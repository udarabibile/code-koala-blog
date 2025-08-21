+++
author = "Udara Bibile"
authors = ["s"]
authorImage = "/img/udarabibile.png"
title = "NodeJS Snippet: Child Process"
date = "2020-01-24"
description = "Offload tasks to a spawned process to unblock the main thread"
tags = ["git", "chmod"]
categories = ["git", "os"]
images  = ["img/2020/nodejs-child-process.png"]
type = "post"
aliases = ["migrate-from-jekyl"]
draft = false
+++

Node.js is single-threaded by design, thereby requiring it to keep the main Node process unblocked to continue execution. Moreover, Node is limited to one thread and 1.76GB (for 64-bit OS) memory usage. Thus, external processes are required to offload CPU intensive processing in parallel. The `child_process` module in Node allows us to spawn processes that can be used for any task to be run in parallel.

* * * * *

### Builtin Node.js module: child_process

The Node module **`child_process`** allows us to create new processes and to offload tasks to be processed in parallel in the background without blocking the single main thread. These child processes can be used for any executable command:

-   System commands: `ls -al`, `pwd`, `node --version`
-   Running Node or JavaScript code: `child.js` or `node calculate.js`
-   Any executable in any other language: `program.exe`, `./script.sh`

Note that system commands may not run in all OS, and this should be handled by the developer.
```
const { platform } = require('os');
const command = platform() === 'win32' ? 'cls' : 'clear';
```
* * * * *

### Creating a new process

**`spawn()`** initiates a new process in the background asynchronously without blocking the main thread.
```
function spawn(command, args?, options?): ChildProcess;
```
Here `command` can be required with optional `arguments` for command and some spawn `options`. This returns a `ChildProcess` instance of the spawned process.

Let's spawn a child process and a execute system command:
```
const cp = require("child_process");\
const child = cp.spawn('pwd');const { spawn } = require("child_process");\
const ls = spawn('ls', ['-al', '/usr']);
```
This would execute `ls -al /usr` in a separate spawned process. Interaction between processes and callbacks can make use of the output of the newly spawned child process. Also note that on Windows OS, a special flag is required to run commands: `spawn('dir', [], { shell:true })`.

* * * * *

### Multiple methods to create a child process

Note following commands are built on top of the `spawn` process for special needs.

-   `spawn()`→ Designed to run **system commands**, where this command is sent to run on new process but nothing is done in the main node process.
-   `fork()`→ Built on top of `spawn()` will **spin up a fresh V8 instance** to be used to run any Node-based code, either of the same program or another. This is useful to execute CPU intensive tasks in parallel or to scale an application. Another difference between `spawn` is that this will have an IPC channel built-in.
-   `exec()`→ It's useful when **only the final result** is the concern and where the child's stdio streams are not required. `spawn()` returns a data stream whereas `exec()` would return buffer data and may encode it to read. By default, the maximum buffer size is set to 200k but can be extended, else it will give error called `maxBuffer exceeded`.
-   `execFile()`→ Unlike in `exec()`, this command executed without creating a new shell. Since the shell is not created, some system commands may not work on all operating systems, `execFile('ls', ...)` works on Linux but the similar command `execFile('dir', ...)` will not work on Windows.

Child processes can be consumed either using callbacks or listeners:
```
// Consume using callbacks

const child = execFile('node', ['--version'],
   (error, stdout, stderr) => {
      if (error) throw error;
      console.log(stdout);
   });
   
// Consuming using listeners

const child = execFile('node', ['--version']);
child.stdout.on('data',(data) => { console.log(data); });
child.stderr.on('data',(data) => { console.log(data); });
```

# TODO REPL.IT SNIPPET


* * * * *

### Asynchronous or synchronous to create a child process

Child process creation can happen either asynchronously or synchronously. Usual methods mentioned above are async methods, where function process created and executed in parallel without blocking the main Node process. In synchronous process creation, these methods are sync and the Node.js event loop is blocked, pausing execution of any code until the spawned process exists.

These synchronous methods includes: `spawnSync`, `execSync`, `execFileSync` which are variations of its asynchronous methods. These methods are useful in loading scenarios or at application startup.

* * * * *

### Inter process communication

The main Node process and spawned child processes don't share memory due to separate space, thereby communication between processes are vital to sharing data and coordinate each process. It's done by **Inter Process Communication or IPC** in the `child_process` module.

As learned before, `spawn()` or `fork()` would return instance of `ChildProcess` and this object implements the Node `EventEmitter` API. Thus **messages can be sent to and from child processes to the main process**.

For example **`'exit'`** is an event emitted from the child process upon termination:
```
const { spawn } = require('child_process');
const child = spawn('ls', ['-al', '/usr']);

child.on('exit', (code) => {
   console.log(`child process exited with code ${code}`);
});
```
Another important event is the **`'message'`** event which is used for communication between child processes and parent processes:
```
// SEND MESSAGES FROM PARENT TO CHILD
In parent → child.send('Hi')
In child  → process.on('message', message =>

// SEND MESSAGES FROM CHILD TO PARENT
In child  → process.send('Hi');
In parent → child.on('message', message =>
```
All notable events emitted by child process: **`child.on('eventType', callback)`**

-   **`'exit'`** → the child process exits. This `signal` variable is null when the child process exits normally.
-   **`'disconnect'`** → parent process manually calls the `child.disconnect` function
-   **`'error'`** → if the process could not be spawned or killed
-   **`'close'`** → the `stdio` streams of a child process get closed. This `close` event is different than the `exit` event because multiple child processes might share the same `stdio` streams and so one child process exiting does not mean that the streams got closed.
-   **`'message'`** → child process uses the `process.send()` function to send messages to the parent process.

# TODO:  GIT SNIPPET

IPC communication between parent & child processes

* * * * *

### Standard stdio streams

Moreover `ChildProcess` also contains standard `stdio` streams for I/O called `stdin` (writeable), `stdout` (readable), `stderr` (readable) which establishes communication pipeline between parent Node.js process and the spawned child process. These streams also implements `EventEmitter` API in nodejs.

On `readable` streams, the parent process can listen to `data` of the event:
```
const { spawn } = require('child_process');
const child = spawn('pwd');

child.stdout.on('data', (data) => {
    console.log(`stdout: ${data}`);
});
```

Output of the `pwd` command gets printed and child process exits with `code 0`.

If there is an error in command, the `data` event handler of `child.stderr` is triggered and `exit` event handler will exit with `code 1`.
```
const { exec } = require('child_process');
const child = exec('pwdsss');

child.stderr.on('data', (data) => {
   console.log(`stdout: ${data}`);
});
```

If the invalid command of `pwdsss` is executed and `stderr` stream is activated. **When all `stdio` streams gets closed, child process will emit `close` event**.

### Pipeline between parent process and child process streams

On the main process, `stdin` is a `readable` stream but on the child process, it's a `writable` stream. Thereby inverse is found in the main process, here the child process writes into the stream, where from the main process point of view stream it can be read.

If there is input that needs to pass to the child from the parent, a simple pipe can `stdin` of parent to input stream `stdin` of child.

```
const { spawn } = require('child_process');
const child = spawn('wc');

process.stdin.pipe(child.stdin);

child.stdout.on('data', (data) => {
   console.log(`\nFrom Child:\n${data}`);
});
```

After executing this, in an idle terminal add random texts and lines and then enter `Ctrl + D` to finish parent's input stream. This input from parent process will be piped to child process. This `wc` just counts lines, characters of the input.

Lets use a pipeline to connect child process's `stdout` to parent process:
```
var ls = child_process.spawn('ls', ['-a', '/home']);
ls.stdout.pipe(process.stdout);
```
Here it will pipe the child's output stream into parent to list directories. Like this, all streams of the child can be linked to parent straight away using spawn options.
```
var cp = require('child_process');

var command = 'echo';
var args = ['hello', 'world'];

var childProcess = cp.spawnSync(command, args, {
   cwd: process.cwd(),
   env: process.env,
   stdio: [ process.stdin, process.stdout, process.stderr ],
   // stdio: 'inherit' can also be used.
   encoding: 'utf-8'
});
```
Another use case is to make pipelines between child processes:
```
const { spawn } = require('child_process');
const find = spawn('find', ['.', '-type', 'f']);
const wc = spawn('wc', ['-l']);

find.stdout.pipe(process.stdout);
find.stdout.pipe(wc.stdin);wc.stdout.on('data', (data) => {
   console.log(`Number of files ${data}`);
});
```

Here the output of `find` child process into `wc` child process to list of number of files in current directory.

* * * * *

### Summary of Child Process object

When spawning up the child process, we saw that an object of type `ChildProcess` is returned. Summary of this object is as follows:

-   Events → `message`, `error`, `exit`, `disconnect`, `close`
-   Attributes → `stdin`, `stdout`, `stderr`, `stdio`, `pid`, `connected`, `kill`, `send()`, `disconnect()`

For example, when creating child processes using `spawn()` or `exec()`, options can be added such as

-   `SpawnOptions` → `cwd`, `env`, `stdio`, `detached`, `uid`, `gid`, `shell`
-   `ExecOptions` → `encoding`, `timeout`, `shell`, `maxBuffer`, `killSignal`

* * * * *

![](/img/2020/nodejs-child-process.png)

This aimed to give a brief overview of using processes form `child_process` to make most out of nodejs. Find out more on these here:

# TODO WEB THUMBNAIL
[Node.js Documentation](https://nodejs.org/api/child_process.html)

Note: Nodejs built-in module **`cluster`** also spins up new process also using `child_process` method `fork`. This allows us to enable the process to share the same port, thereby it can be mostly used for **nodejs http server scaling**.

Note: Nodejs built-in module `worker_threads` to is also used for processing intensive tasks with light weight threads. However use cases and requirements for these two modules might be varied.