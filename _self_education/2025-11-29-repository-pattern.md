---
layout: single
title: "What's the repository pattern in programming and why do we use it?"
date: 2025-11-29 15:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /ai_workflows/2025-11-29-repository-pattern
---

# What's the repository pattern in programming and why do we use it?

Recently I encountered a problem in my codebase where I wanted to swap out a backend (from the SQLite that I had been using for prototyping to a more mature Postgres) and I found that I had to rewrite a lot of business logic. As it turns out, I had coupled the business logic too closely to the database. This also made testing a huge pain, because to test any of the business logic relatd to DBs, I had to go through the effort of mocking the entire database.

I learned about the `Repository` pattern and it ended up solving my problem!

## What is the `Repository` pattern?

The Repository Pattern abstracts access to persistent storage (SQL, NoSQL, files, external APIs) behind a clean interface so that:

- Your domain logic doesn’t know anything about SQL, schemas, or connection details.
- You can swap storage implementations (SQLite → Postgres → in-memory for tests) without changing application logic.
- You enforce a boundary between business logic and persistence, improving correctness, testability, and evolution.

This means that the app depends on an interface, not on the specifics of database implementation.

## Example anti-pattern

Let's imagine that we have the following code:

```python
import sqlite3
from dataclasses import dataclass

@dataclass
class User:
    id: int | None
    name: str
    email: str


def create_user_table():
    conn = sqlite3.connect("example.db")
    cur = conn.cursor()
    cur.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL
        )
    """)
    conn.commit()
    conn.close()


def create_user(name: str, email: str) -> User:
    conn = sqlite3.connect("example.db")
    cur = conn.cursor()
    cur.execute("INSERT INTO users (name, email) VALUES (?, ?)", (name, email))
    conn.commit()
    user_id = cur.lastrowid
    conn.close()
    return User(id=user_id, name=name, email=email)


def find_user_by_email(email: str) -> User | None:
    conn = sqlite3.connect("example.db")
    cur = conn.cursor()
    cur.execute("SELECT id, name, email FROM users WHERE email = ?", (email,))
    row = cur.fetchone()
    conn.close()
    if not row:
        return None
    return User(id=row[0], name=row[1], email=row[2])


# Business logic tightly coupled to SQL
def register_user(email: str, name: str):
    existing = find_user_by_email(email)
    if existing:
        raise ValueError("Email already registered")

    return create_user(name, email)
```

In this implementation, the business logic (`register_user`) makes a call to `find_user_by_email`, which directly accesses SQLite. If you migrate from SQLite to Postgres, you would also need to change the business logic, which isn't ideal. Ditto for testing, as if you wanted to test `register_user`, you'd need a full mock of the database functionality and it would be specific to your database implementation. Any database migrations or schema changes or refactors would also touch business logic.

In short, here are some of the fallbacks with this:

- Business logic directly knows SQL details.
- Hard to test because you need a real DB.
- SQL queries spread across the codebase.
- Any migration or schema change forces refactors everywhere.

## A better version, using the Repository pattern

We can create a `UserRepository` interface that serves as an intermediary between the database and business layers:

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class User:
    id: int | None
    name: str
    email: str


class UserRepository(ABC):
    @abstractmethod
    def create(self, user: User) -> User: ...
    
    @abstractmethod
    def find_by_email(self, email: str) -> User | None: ...
```

Our SQLite database becomes the following:

```python
import sqlite3

class SQLiteUserRepository(UserRepository):
    def __init__(self, db_path: str = "example.db"):
        self.db_path = db_path
        self._init_schema()

    def _connect(self):
        return sqlite3.connect(self.db_path)

    def _init_schema(self):
        conn = self._connect()
        cur = conn.cursor()
        cur.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT NOT NULL
            )
        """)
        conn.commit()
        conn.close()

    def create(self, user: User) -> User:
        conn = self._connect()
        cur = conn.cursor()
        cur.execute("INSERT INTO users (name, email) VALUES (?, ?)",
                    (user.name, user.email))
        conn.commit()
        user.id = cur.lastrowid
        conn.close()
        return user

    def find_by_email(self, email: str) -> User | None:
        conn = self._connect()
        cur = conn.cursor()
        cur.execute("SELECT id, name, email FROM users WHERE email = ?", (email,))
        row = cur.fetchone()
        conn.close()
        if not row:
            return None
        return User(id=row[0], name=row[1], email=row[2])
```

However, we can, as is implied here, create, in addition to `SQLiteUserRepository`, implementations for things like `PostgresUserRepository`, `LocalUserRepository`, `MongoDBUserRepository`, and the like, as long as they expose the `create` and `find_by_email` methods enforced by the `UserRepository` dataclass. This allows us to swap out the backend and to expose a single consistent interface for business logic to use.

The business logic layer now becomes decoupled:

```python
class UserService:
    def __init__(self, repo: UserRepository):
        self.repo = repo

    def register_user(self, name: str, email: str) -> User:
        if self.repo.find_by_email(email):
            raise ValueError("Email already registered")
        return self.repo.create(User(id=None, name=name, email=email))
```

We can now use it like this:

```python
repo = SQLiteUserRepository()
service = UserService(repo)

new_user = service.register_user("Alice", "alice@example.com")
print(new_user)
```

This version is agnostic of the underlying database implementation and can be easily tested or mocked. We can swap out or migrate the database layer independent of the business layer.

In our original code:

```bash
register_user() -> find_user_by_email() -> sqlite3 -> SQL
```

The business rule is fully tied to SQLite implementation details.

In the refactored version:

```bash
UserService.register_user() -> repo.find_by_email()
```

The service has no idea how the repo works. It only knows about the contract: “give me a user by email”.

Some of the benefits of this include:

- The application logic never touches SQL strings, DB connections, or schema details.
- No need for a live database to test business rules. We can use a bare minimal `FakeUserRepository` mock.
- All data access concerns are centralized and consistent.
- We can easily swap out backends.
- Concise business logic means easier onboarding for engineers.

In addition, our implementation uses the ABC class, which communicates intent and prevents misuse (by requiring that specific methods exist)

We see in our new version a clean separation of concerns:

- `UserService` handles business rules.
- `SQLiteUserRepository` handles persistence.
- `UserRepository` defines a stable boundary.

Because the code is set up like this, we get dependency inversion.

High-level module

- UserService
  - Handles application rules (“emails must be unique”).
  - Should not do SQL or know how the DB works.

Low-level module

- SQLiteUserRepository
  - Handles SQL queries, connections, schema.

Abstraction that both depend on

- UserRepository interface
  - Defines the contract expected by the business layer.
  - Doesn’t know anything about SQLite or any specific DB.

The dependency inversion that we get from the Repository Pattern supports all of the following considerations for long-term maintainable code:

- Business domain outlives any particular datastore.
- Teams swap datastores over time.
- Tests run on lightweight in-memory mocks.
- Services talk to other microservices via interfaces, not implementations.

## Learnings

The Repository Pattern was a fix for an anti-pattern that was causing bugs in my codebase. I understand now what it's for and why it's used and the benefits of it, and I'll keep applying it to my codebase over time.
