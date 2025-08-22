+++
author = "Udara Bibile"
authorImage = "/img/udarabibile-thumbnail.png"
title = "SQL Snippet: Foreign Keys"
date = "2019-09-24"
description = "MANAGING RELATIONSHIPS BETWEEN DATABASE ATTRIBUTES."
tags = ["sql", "syntax"]
categories = ["database", "software"]
images  = ["img/2019/sql-foreign-keys.png"]
type = "post"
aliases = ["migrate-from-jekyl"]
+++

If you have worked with SQL based database, I guess y’all are familiar with **usage of foreign keys to manage relationship between tables.**

## Relationship Types
Lets dive into a famous example involving book — author connection to gain insight. There are variety of connection between these entities, in a graphical:

<img class="image featured" src="/img/2019/sql-foreign-keys.png" alt="" />

* **one to one** →an author can write only one book, and a book only have one author.
* **one to many** →an author can write multiple books, but book can only have one author.
* **many to many** →an author can write multiple books, and also a book can be written by multiple authors.

Lets look into above scenarios for ease of this tutorial, even though some scenarios doesn't seem accurate in real world.

SQL snippets found in this article is written to support SQLite and can be tested in https://sqliteonline.com/.

<hr/>

### One-to-One relationship
Each of author and book table will have **primary keys** to identify each value uniquely. **Author** table will have `Pk_author_id`, and **Book** table will have `Pk_book_Id` as unique identifier for each table. To form relationship between two tables, foreign key references should be added. Here each entry in **Book** table, it will refer `author_Id` to keep track of connected entry identifier in **Author** table.

Lets dig into SQL code as to how it can be done:

```
CREATE TABLE Author (
   Pk_author_id INTEGER PRIMARY KEY AUTOINCREMENT,
   Name TEXT,
   Country TEXT
);
CREATE TABLE Book (
   Pk_book_Id INTEGER PRIMARY KEY AUTOINCREMENT,
   Title TEXT,
   Fk_author_Id INTEGER UNIQUE,
   FOREIGN KEY (Fk_author_Id) REFERENCES Author(Pk_author_id)
);
```

<mark>TODO: HIGHLIGHT WITHIN CODE BLOCK</mark>


Here when setting up Book table, a connection to author is attached via an id. Lets add some values to test one-to-one relationship.

```
INSERT INTO Author (Name, Country) VALUES ('Leo Tolstoy', 'Russia');
INSERT INTO Book (Title, Fk_author_Id) VALUES ('War & Peace', 1);
```

Thereby after inserting these values, formed relationship can be queried:

```
SELECT Book.Title, Author.Name, Author.Country
FROM Book INNER JOIN Author
ON Author.Pk_author_id = Book.Fk_author_Id
```
<br/>

   Title | Name  | Country
--------|------|------
    War & Peace | Leo Tolstoy | Russia

However since this is one-to-one relationship implementation, there should be restriction of adding another book for this author.

```
INSERT INTO Book (Title, Fk_author_Id) VALUES ('Anna Karenina', 1);
→ Results in UNIQUE constraint failed: Book.Fk_author_Id
```

Due to **UNIQUE** keyword introduced to `Fk_author_Id` attribute in Book table, it would restrict adding any record by used author_Id. Thereby making it one-to-one relationship.

<hr/>

### One-to-Many relationship
Building upon previous example, this scenario should allow an author to write multiple books. By looking into how one-to-one relationship was implemented using UNIQUE keyword, it can be seen that **removing UNIQUE keyword** would make it one-to-many relationship. Book table should have been created as such.

```
CREATE TABLE Book (
   Pk_book_Id INTEGER PRIMARY KEY AUTOINCREMENT,
   Title TEXT,
   Fk_author_Id INTEGER,
   FOREIGN KEY (Fk_author_Id) REFERENCES Author(Pk_author_id)
);
```
And add values as such without facing an issue like previously.

```
INSERT INTO Author (Name, Country) VALUES ('Leo Tolstoy', 'Russia');
INSERT INTO Book (Title, Fk_author_Id) VALUES ('War & Peace', 1);
INSERT INTO Book (Title, Fk_author_Id) VALUES ('Anna Karenina', 1);
```

Lets query and see that one-to-many relationship exists. Here an author have been attached to multiple books.
<br/>

   Title | Name  | Country
--------|------|------
    War & Peace | Leo Tolstoy | Russia
    Anna Karenina | Leo Tolstoy | Russia

<hr/>


### Many-to-Many relationship
Simply in this occasion an author can be attached to multiple books, and a book can be attached to multiple authors. This is more closer to real world.

In document based databases such as MongoDB or such this can be simple due to usage of array. Simple example can be as follows:

```
// Authors
{ Id: 1, Name: 'Larry Bird', BookIds: [1] }
{ Id: 2, Name: 'Magic Johnson', BookIds: [1,2] }

// Books
{ Id: 1, Title: 'When the game was ours', AuthorIds: [1, 2] }
{ Id: 2, Title: 'Drive', AuthorIds: [1] }
```

With just using flexibility of NoSQL and arrays many-to-many relationship is simply made. However due to unavailability of array in SQL, different approach is taken to handle such relationship.

In order to create many-to-many relationship, a join table is created to record unique identifiers of connected values. As per example in order to join **Book** and **Author** tables, another table called **Author_Book** is created.

<img class="image featured" src="/img/2019/many-to-many.png" alt="" />

Lets create two simple tables without any relationship for **Book** and **Author**.

```
CREATE TABLE Author (
   Pk_author_id INTEGER PRIMARY KEY AUTOINCREMENT,
   Name TEXT,
   Country TEXT
);

CREATE TABLE Book (
   Pk_book_Id INTEGER PRIMARY KEY AUTOINCREMENT,
   Title TEXT
);
```

Then create joining table, adding parent tables’ primary keys as foreign keys in join table. Additional property related to relationship can also be stored here such as **Extra_value**.

```
CREATE TABLE Author_Book (
   Fk_author_id INTEGER NOT NULL,
   Fk_book_id INTEGER NOT NULL,
   Extra_value TEXT,
   FOREIGN KEY (Fk_author_id) REFERENCES Author(Pk_author_id),
   FOREIGN KEY (Fk_book_id) REFERENCES Book(Pk_book_Id),
   PRIMARY KEY (Fk_author_id, Fk_book_id)
);
```

Each entry in this join table, will include primary keys of parent tables. As an example, `Fk_author_id` is unique identifier in **Author** table, and `Fk_book_id` is unique identifier in **Book** table. Thereby (`Fk_author_id`, `Fk_book_id`) tuple or composite key will be unique itself, and thereby separate unique identifier or primary key for **Author_Book** is not essential. However integer auto incrementing value can be added as primary key, but it’ll be redundant.

Here this table just contains each combination of given many-to-many relationship. Lets enter some data and query.

```
INSERT INTO Author (Pk_author_id, name, country)
VALUES (1, 'Larry Bird', 'USA');

INSERT INTO Author (Pk_author_id, name, country)
VALUES (2, 'Magic Johnson', 'USA');

INSERT INTO Book (Pk_book_Id, Title)
VALUES (1, 'When the game was ours');

INSERT INTO Book (Pk_book_Id, Title)
VALUES (2, 'Drive');
```

Lets add many-to-many relationship between them:

```
INSERT INTO Author_Book(Fk_author_id, Fk_book_id) VALUES (1, 1);
INSERT INTO Author_Book(Fk_author_id, Fk_book_id) VALUES (2, 1);
INSERT INTO Author_Book(Fk_author_id, Fk_book_id) VALUES (1, 2);
```

Lets query the database by join all three tables and extract required values:

```
SELECT Author.Name AS 'Name', Book.Title AS 'Title'
FROM Author, Book, Author_Book
WHERE Author.Pk_author_id = Author_Book.Fk_author_id
   AND Book.Pk_book_Id = Author_Book.Fk_book_id;
```

   Name | Title
--------|------
    Larry Bird | When the game was ours
    Magic Johnson | When the game was ours
    Larry Bird | Drive

<hr/>

## Foreign Key Constraints

Here connections are made between tables, and now records are connected. Now lets try to remove these connection:

```
DELETE FROM Author WHERE Name="Larry Bird";
```

Here **Author** value is deleted without any issue, but these raises concern about relationships made. Thereby inconsistent data is still available, such as this author already referred in **Author_Book**.

SQL allows functionality to avoid such inconsistent data, by disabling or handling such deletion. **Foreign key constraints** are there to handle it, and lets create new tables to test these.

<hr/>

### RESTRICT

Restricting any deletion of references can be an approach foreign keys constraints can be added.

```
DROP TABLE IF EXISTS Book;
DROP TABLE IF EXISTS Author;
CREATE TABLE Author(AuthorId INTEGER PRIMARY KEY, Name TEXT);
CREATE TABLE Book(BookId INTEGER PRIMARY KEY, Title TEXT,
    AuthorId INTEGER, 
    FOREIGN KEY(AuthorId) REFERENCES Author(AuthorId));
INSERT INTO Author VALUES (1, 'Leo Tolstoy');
INSERT INTO Book VALUES (1,'War & Peace',1);
INSERT INTO Book VALUES (2,'Anna Karenia',1);
```

Enable foreign key constraints and try to delete connected record.

```
PRAGMA foreign_keys=1;
   → Enable foreign keys constraints in SQLite.
DELETE FROM Author WHERE AuthorId=1;
   → FOREIGN KEY constraint failed, Due to Books for that Author.
DELETE FROM Book WHERE AuthorId=1;
   → Books related to that Author deleted without an issue.
DELETE FROM Author WHERE AuthorId=1;
   → Author can be deleted now, as related books are deleted.
```

As seen when foreign keys constraints are enforced, any deletion or updates are not allowed. That behavior is called **RESTRICT**, added by default.

<hr/>

### ON DELETE CASCADE

However when creating the table, required behavior for foreign key constraints can be decided. Such action is **ON DELETE CASCADE**, which indicates that action (DELETE) should be propagated by related foreign keys from parent table (Author) to child table (Book). Simply if parent value is deleted, all other records keeping reference to that parent value is to be deleted.

```
CREATE TABLE Book(BookId INTEGER PRIMARY KEY, Title TEXT,
    AuthorId INTEGER, FOREIGN KEY(AuthorId) REFERENCES Author(AuthorId) ON DELETE CASCADE);
   → CREATE Book table as above with condition
SELECT Title, Name FROM Author, Book;
   → Add data as added previously.
DELETE FROM Author WHERE AuthorId=1;
   → No foreign key violation invoked. Deleting from Author & Book
```
<hr/>

### ON DELETE SET NULL

If values on child tables, are **not to be deleted but rather be altered** according to deleted values in parent table this approach can be used. Here references in child table to parent table will be set to null, and row wouldn't be deleted.

```
CREATE TABLE Book(BookId INTEGER PRIMARY KEY, Title TEXT,
    AuthorId INTEGER, FOREIGN KEY(AuthorId) REFERENCES Author(AuthorId) ON DELETE SET NULL);
DELETE FROM Author WHERE AuthorId=1;
   → No foreign key violation invoked.
   → Deleting from Author. Set null in Book.
SELECT * FROM Book;
```

   BookId | Title  | AuthorId
--------|------|------
    1 | War & Peace | null
    2 | Anna Karenina | null


Moreover, it should be noted that restriction happened only when deleting from Author table, and not from Book table. This is the expected behavior where **foreign key relationship is only one way**, from parent table (Author) towards child table (Book). Here Book can’t be created unless Author is explicitly connected, and at deletion Book wouldn't be referred in any other tables. However when trying to delete Author, it could be referred elsewhere in Book table, so constraints applied.

<hr/>

From this **SQL Snippet**, I think y’all are able to insight into managing data relationship in SQL database tables. Note that SQLite code segments are used here, and other implementation might vary little.

<br/><br/>

