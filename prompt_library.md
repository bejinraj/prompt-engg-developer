# Input Prompt:

- CAPACITY: Act as a Senior Backend Software Engineer.

- RECIPIENT: Junior QA and developer team.

- INSTRUCTION: Write API route controller logic in Node.js (using Express) or Python (using FastAPI) to handle:

1. POST /api/v1/tasks (Create a new task, extracting title, description, due_date).
2. GET /api/v1/tasks/:id (Read a specific task by ID).

- STYLE: Follow DRY and SOLID design principles. Include clean error handling.

- PARAMETERS: Use raw SQL queries matching the schemas created in Step 3. Do not use an
  ORM.

- EVALUATION: Write inline comments explaining how parameters are parsed.

# Chat-GPT Output:

Sure. Below is the **Node.js + Express** version using **raw SQL** and PostgreSQL, with clean error handling and inline comments explaining parameter parsing.

```javascript
const express = require("express");
const { Pool } = require("pg");

const app = express();

app.use(express.json());

// Database connection pool
const pool = new Pool({
  host: process.env.DB_HOST || "localhost",
  database: process.env.DB_NAME || "task_tracker",
  user: process.env.DB_USER || "postgres",
  password: process.env.DB_PASSWORD || "password",
  port: process.env.DB_PORT || 5432,
});

/*
 * POST /api/v1/tasks
 * Create a new task
 */
app.post("/api/v1/tasks", async (req, res) => {
  // express.json() parses the JSON request body.
  // Destructuring extracts the required parameters from req.body.
  const { user_id, title, description, due_date } = req.body;

  // Basic input validation
  if (!user_id || !title || !title.trim()) {
    return res.status(400).json({
      error: "user_id and title are required",
    });
  }

  try {
    // Raw SQL query using parameterized placeholders ($1, $2, etc.).
    // This prevents SQL injection.
    const query = `
      INSERT INTO tasks
        (user_id, title, description, due_date, created_at)
      VALUES
        ($1, $2, $3, $4, CURRENT_TIMESTAMP)
      RETURNING
        id,
        user_id,
        title,
        description,
        status,
        due_date,
        created_at;
    `;

    const values = [user_id, title, description || null, due_date || null];

    const result = await pool.query(query, values);

    return res.status(201).json({
      message: "Task created successfully",
      task: result.rows[0],
    });
  } catch (error) {
    console.error("Create task error:", error);

    // Foreign-key or other constraint violation
    if (error.code === "23503" || error.code === "23505") {
      return res.status(400).json({
        error: "Invalid task data or user_id",
      });
    }

    return res.status(500).json({
      error: "Internal server error",
    });
  }
});

/*
 * GET /api/v1/tasks/:id
 * Retrieve a specific task by ID
 */
app.get("/api/v1/tasks/:id", async (req, res) => {
  // Express parses the URL parameter as a string.
  // Number() converts it into an integer for the SQL query.
  const taskId = Number(req.params.id);

  // Validate the parsed ID
  if (!Number.isInteger(taskId) || taskId <= 0) {
    return res.status(400).json({
      error: "Task ID must be a positive integer",
    });
  }

  try {
    // Raw SQL query with a parameterized placeholder.
    const query = `
      SELECT
        id,
        user_id,
        title,
        description,
        status,
        due_date,
        created_at
      FROM tasks
      WHERE id = $1;
    `;

    const result = await pool.query(query, [taskId]);

    // No matching task found
    if (result.rows.length === 0) {
      return res.status(404).json({
        error: "Task not found",
      });
    }

    return res.status(200).json({
      task: result.rows[0],
    });
  } catch (error) {
    console.error("Get task error:", error);

    return res.status(500).json({
      error: "Internal server error",
    });
  }
});

/*
 * Start server
 */
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Task Tracker API running on port ${PORT}`);
});
```

### Required packages

```bash
npm install express pg
```

### Example POST request

**Endpoint:**

```text
POST /api/v1/tasks
```

**Body:**

```json
{
  "user_id": 1,
  "title": "Complete API testing",
  "description": "Test task creation and retrieval",
  "due_date": "2026-09-25"
}
```

### Example GET request

```text
GET /api/v1/tasks/10
```

### Key implementation points

- `req.body` → extracts `user_id`, `title`, `description`, and `due_date`.
- `req.params.id` → extracts the task ID from `/tasks/:id`.
- `Number(req.params.id)` → converts the URL parameter from string to integer.
- `$1`, `$2`, `$3`, `$4` → parameterized SQL values that protect against SQL injection.
- `201` → task successfully created.
- `400` → invalid input.
- `404` → task does not exist.
- `500` → unexpected/database error.
- No ORM is used; queries directly access the `tasks` table.
