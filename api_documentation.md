# Input Prompt:

[CONTEXT]: We are building the database layer for our Task Tracker API.

[ACTION]: Generate relational SQL table schemas matching the database requirements:

- Table 'users': columns for id (primary key), name, email (unique), password_hash, and
  created_at.

- Table 'tasks': columns for id (primary key), user_id (foreign key referencing users),
  title, description, status, due_date, and created_at.

[REQUIREMENTS]:

- Include SQL constraints (NOT NULL, UNIQUE, FOREIGN KEY, PRIMARY KEY).
- Provide raw DDL SQL code block.

# Chat-GPT Output:

Below is the raw relational DDL for the `users` and `tasks` tables, including the requested constraints.

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL
);

CREATE TABLE tasks (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) NOT NULL,
    due_date TIMESTAMP,
    created_at TIMESTAMP NOT NULL,

    CONSTRAINT fk_tasks_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
);
```
