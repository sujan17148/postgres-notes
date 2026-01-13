Here's your content reformatted into a well-structured Markdown note:

```markdown
# PostgreSQL Users, DB Ownership, and Express Connections (Minimal Notes)

## Why Create a Separate DB Owner User (Instead of Using `postgres`)?

- The `postgres` role is typically a **superuser**, which means it has full access to modify and drop almost anything in the Postgres cluster.
- **Best practice**: Create a **separate user/role** to own each project database, or at least assign a dedicated DB user for your app. This limits permissions and avoids running the app with superuser privileges.

---

## Create a New User (Role)

### Create User and Set Password

```sql
CREATE USER user_name WITH PASSWORD 'strong_password';
```

- Replace `user_name` with the desired username for your app.
- This user will be used in your Express app's Pool/Client configuration, instead of the `postgres` superuser.

### Change User Password

```sql
sudo -u postgres psql
ALTER USER user_name WITH PASSWORD 'new_password';
```

- You can reset passwords for any user if you have access to the `postgres` superuser.

### Create a Database and Set the Owner

```sql
CREATE DATABASE my_db;
-- By default, owner will be 'postgres' (superuser)
```

```sql
CREATE DATABASE my_db OWNER user_name;
-- Now user_name is the owner of 'my_db'
```

- You can assign a specific user as the owner of the database, improving security by limiting superuser access.

---

## Privileges (Very Short)

- **postgres** (superuser) has all privileges by default.
- For other users, you can grant permissions as needed, such as:
  - Connect
  - Create new tables or schemas
  - CRUD operations on specific tables, etc.

---

## Express + PostgreSQL: Client vs Pool

### Client (Single Connection)

- A single connection to Postgres.
- Suitable for scripts or one-off tasks.
- Not ideal for handling multiple concurrent HTTP requests (it can become a bottleneck).

### Pool (Recommended for Express Apps)

- Manages a pool of reusable connections.
- Handles concurrent requests more efficiently.
- **Most Express apps should use Pool**.

### Client Example

```js
const { Client } = require("pg");

const client = new Client({
  host: "127.0.0.1",      // or "localhost"
  user: "user_name",
  password: "strong_password",
  database: "my_db",
  port: 5432,
});

await client.connect();

const result = await client.query("SELECT NOW()");
console.log(result.rows);

await client.end();
```

### Pool Example

```js
const { Pool } = require("pg");

const pool = new Pool({
  host: "127.0.0.1",      // or "localhost"
  user: "user_name",
  password: "strong_password",
  database: "my_db",
  port: 5432,

  // Optional tuning (safe defaults if you omit)
  max: 10,                 // max connections in pool
  idleTimeoutMillis: 30000, // close idle clients after 30s
  connectionTimeoutMillis: 2000, // fail if can't connect in 2s
});

// Simplest usage
const result = await pool.query("SELECT NOW()");
console.log(result.rows);

// When shutting down the app (optional but good practice)
await pool.end();
```