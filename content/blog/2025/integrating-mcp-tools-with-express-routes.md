+++
author = "Udara Bibile"
authors = ["s"]
authorImage = "/img/udarabibile-thumbnail.png"
title = "Integrating MCP Tools into Express with Minimal Changes"
date = "2025-08-28"
description = "Share validation schemas and handler logic from Express Routes to MCP Tools"
tags = ["expressjs", "nodejs", "mcp"]
categories = ["ai", "javascript"]
images  = ["img/2025/mcp-express.png"]
type = "post"
aliases = []
draft = false
+++

This guide walks through setting up an **Express HTTP server** and extending it with **MCP tools**, using `Zod` for schema validation. By the end, you’ll have a server that supports both REST-style routes and MCP-compatible tools with shared logic.

Let’s create new project with Nodejs v23+ and install dependencies:

```
npm install express zod @modelcontextprotocol/sdk
```

Then create `index.ts` and it can be run via `node index.ts`

---

## Basic Express HTTP Route

This route accepts a JSON body with `name`, validates it with Zod, and replies with a greeting.

```
import express from "express";
import { z } from "zod";

const app = express();
app.use(express.json());

app.post('/greet-user', async (req, res) => {
    try {
        const { name } = z.object({
            name: z.string().describe("Name of the user"),
        }).parse(req.body);
        res.json({ text: `Hello, ${name}!` });
    } catch (error) {
        if (error instanceof z.ZodError) {
            return res.status(400).json({ error: error.errors });
        }
        res.status(500).json({ error: 'Internal server error' });
    }
});

const PORT = 4100;
app.listen(PORT, () => {
    console.log(`Express HTTP server connected and running on port ${PORT}`);
});
```

This can be verified via
```
curl -X POST http://localhost:4100/greet-user \
  -H "Content-Type: application/json" \
  -d '{"name": "Tommy Shelby"}'
```

---

## Basic MCP Tool Handler

Next, let’s expose similar functionality, but as an MCP tool available over HTTP.

We set up an MCP server and register a tool named `greet-user`, with the same Zod validation and response format.

```
import express from "express";
import { z } from "zod";
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";

const server = new McpServer({
  name: "Greet Server",
  version: "1.0.0",
  capabilities: { tools: {} }
});

server.tool(
  'greet-user',
  'Greet user by name',
  {
    name: z.string().describe("Name of the user"),
  },
  async ({ name }) => {
    return {
      content: [{ type: "text", text: `Hello, ${name}!` }]
    };
  }
)

const app = express();
app.use(express.json());

const transport = new StreamableHTTPServerTransport({
  sessionIdGenerator: undefined,
});

app.post('/mcp', async (req, res) => {
  await transport.handleRequest(req, res, req.body);
});

const PORT = 4100;
server.connect(transport)
  .then(() => {
    app.listen(PORT, () => {
      console.log(`MCP HTTP server connected and running on port ${PORT}`);
    });
  })
  .catch(error => {
    console.error("Failed to connect MCP server to transport:", error);
    process.exit(1);
  });
```

This can be verified via

```
curl -X POST http://localhost:4100/mcp \
-H "Content-Type: application/json" \
-H "Accept: application/json, text/event-stream" \
-d '{   "jsonrpc": "2.0",                  
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "greet-user",
    "arguments": { "name": "Tommy Shelby" },
    "_meta": { "progressToken": 0 }
  }
}'
```

## Unifying Express Routes & MCP Tools

Notice both the Express route and the MCP tool share common elements:

- Validation schema (Zod)

- Handler function

The only difference is output format — Express returns `JSON`, MCP requires the `{ type: "text"; text: string }[]` structure.

To avoid duplication, we create wrapper functions that adapt a schema + handler into either:

- an Express request handler (`createExpressHandler`)

- an MCP tool function (`createMcpTool`)

---

As a practical example, let’s build a mini book management service.
We’ll store books in a local file (`books.json`) and expose two operations: `createBook` and `getBookById`

#### Managing Book Data with File Storage
`bookService.ts` → define read/write logic using Node’s `fs/promises`

```
import fs from "fs/promises";
import path from "path";

export interface IBook { id: string; title: string; author: string; year?: number; }

const DATA_FILE = path.join(process.cwd(), "books.json");

export async function readBooks(): Promise<IBook[]> {
  try {
    const data = await fs.readFile(DATA_FILE, "utf8");
    return JSON.parse(data) as IBook[];
  } catch (e) {
    if (e.code === "ENOENT") return [];
    throw e;
  }
}

export async function writeBooks(books: IBook[]) {
  await fs.writeFile(DATA_FILE, JSON.stringify(books, null, 2), "utf8");
}

export async function getAllBooks() {
  return await readBooks();
}

export async function getBookById(id: string) {
  const books = await readBooks();
  const book = books.find(b => b.id === id);
  if (!book) throw new Error(`Book with ID ${id} not found.`);
  return book;
}

export async function createBook(data: Omit<IBook, "id">) {
  const books = await readBooks();
  const id = books.length ? String(Math.max(...books.map(b => Number(b.id))) + 1) : "1";
  const newBook = { id, ...data };
  books.push(newBook);
  await writeBooks(books);
  return newBook;
}
```

#### Reusable Validation and Handler Wrappers
`wrappers.ts` → create the helper wrappers for Express and MCP, both powered by `Zod` validation.

```
import { z } from "zod";

export function createMcpTool<TInput>(
  schema: z.ZodType<TInput>,
  handler: (input: TInput) => Promise<any>
): (input: TInput) => Promise<{ content: { type: "text"; text: string }[] }> {
  return async (input: TInput) => {
    const parseResult = schema.safeParse(input);
    if (!parseResult.success) {
      return {
        content: [{ type: "text", text: `Validation failed: ${JSON.stringify(parseResult.error.errors)}` }]
      };
    }
    try {
      const result = await handler(parseResult.data);
      return {
        content: [{ type: "text", text: JSON.stringify(result) }]
      };
    } catch (err: any) {
      return {
        content: [{ type: "text", text: `Error: ${err.message}` }]
      };
    }
  };
}

export function createExpressHandler<TInput>(
  schema: z.ZodType<TInput>,
  handler: (input: TInput) => Promise<any>
) {
  return async (req, res) => {
    const parseResult = schema.safeParse(req.body || req.params);
    if (!parseResult.success) {
      return res.status(400).json({ error: parseResult.error.errors });
    }
    try {
      const result = await handler(parseResult.data);
      res.json(result);
    } catch (err: any) {
      const isNotFound = err.message.includes("not found");
      res.status(isNotFound ? 404 : 500).json({ error: err.message });
    }
  };
}
```

#### Integrating Express Routes with MCP Tools
`index.ts` → This starts an Express server that registers multiple routes, and serves MCP tools over HTTP through the `/mcp` endpoint.

```
import express from "express";
import { z } from "zod";
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import { createExpressHandler, createMcpTool } from "./wrappers";
import { getBookById, createBook } from "./bookService";

const expressServer = express();
expressServer.use(express.json());

// Schema and Handler for Get Book by ID
const getBookByIdSchema = z.object({
  id: z.string().describe("Book ID"),
});
const getBookByIdFunc = async ({ id }) => {
  const book = await getBookById(id);
  if (!book) throw new Error(`Book with ID ${id} not found.`);
  return book;
}
// Schema and Handler for Create Book
const createBookSchema = z.object({
  title: z.string(),
  author: z.string(),
  year: z.number().min(0).optional(),
});
const createBookFunc = async ({ title, author, year }: { title: string; author: string; year?: number }) => {
  const book = await createBook({ title, author, year });
  if (!book) throw new Error("Failed to create book.");
  return book;
}

// Express Route
expressServer.get(
  "/books/:id",
  createExpressHandler(getBookByIdSchema, getBookByIdFunc)
);

expressServer.post(
  "/books",
  createExpressHandler(createBookSchema, createBookFunc)
);

// MCP Tool
const mcpServer = new McpServer({
  name: "Books CRUD MCP Server",
  version: "1.0.0",
  capabilities: { tools: {} }
});

mcpServer.tool(
  "get-book-by-id",
  "Get book by ID",
  getBookByIdSchema.shape,
  createMcpTool(getBookByIdSchema, getBookByIdFunc)
);

mcpServer.tool(
  "create-book",
  "Create a new book",
  createBookSchema.shape,
  createMcpTool(createBookSchema, createBookFunc)
);

const transport = new StreamableHTTPServerTransport({
  sessionIdGenerator: undefined,
});

expressServer.post('/mcp', async (req, res) => {
  await transport.handleRequest(req, res, req.body);
});

const PORT = 4100;
mcpServer.connect(transport)
  .then(() => {
    expressServer.listen(PORT, () => {
      console.log(`MCP HTTP mcpServer connected and running on port ${PORT}`);
    });
  })
  .catch(error => {
    console.error("Failed to connect MCP mcpServer to transport:", error);
    process.exit(1);
  });
```

Finally, `index.ts` ties everything together:

- Create Zod schemas for `getBookById` and `createBook`

- Define handler functions for `getBookById` and `createBook`

- Register Express routes for HTTP calls via `createExpressHandler`

- Register MCP tools for MCP calls via `createMcpTool`

- Wrapper functions would be reusing the same schemas & handlers


#### Verify Express Routes and MCP Tools
Let’s test Express routes and MCP tools by running the server.

Lets install `tsx` via `npm install --save-dev tsx` and update `package.json` to use: `"dev": "tsx index.ts"` and run the server using `npm run dev`.

Let’s verify both Express routes and MCP tools works fine using curls

```
// Create Book via Express Route

curl -X POST http://localhost:4100/books \
  -H "Content-Type: application/json" \
  -d '{"title": "The Brothers Karamazov", "author": "Fyodor Dostoevsky", "year": 1880}'
```
```
// Create Book via MCP Tool

curl -X POST http://localhost:4100/mcp \
-H "Content-Type: application/json" \
-H "Accept: application/json, text/event-stream" \
-d '{   "jsonrpc": "2.0",                  
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "create-book",
    "arguments": { "title": "War and Peace", "author": "Leo Tolstoy", "year": 1867 },
    "_meta": { "progressToken": 0 }
  }
}'
```
```
// Get Book via Express Route

curl -X GET http://localhost:4100/books/2
```
```
// Get Book via MCP Tool

curl -X POST http://localhost:4100/mcp \
-H "Content-Type: application/json" \
-H "Accept: application/json, text/event-stream" \
-d '{   "jsonrpc": "2.0",                  
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get-book-by-id",
    "arguments": { "id": "1" },
    "_meta": { "progressToken": 0 }
  }
}'
```

MCP tools can also be verified via `@modelcontextprotocol/inspector` by
```
npx @modelcontextprotocol/inspector npm run dev
```


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1xf54z3omw22rra7577e.png)

---
This approach minimizes code duplication by extracting validation schemas and core logic from Express routes and reusing them seamlessly in MCP tools. It enables rapid exposure of MCP functionality for existing Express servers without rewriting business logic. With this structure, you achieve a maintainable, efficient setup where you build your application logic once and expose it everywhere — supporting both traditional REST APIs and modern MCP tools with ease and consistency.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qxtgm4ykmt3xoo3x5liv.png)

https://github.com/udarabibile/mcp-using-javascript/tree/main/mcp-tools-express-routes
