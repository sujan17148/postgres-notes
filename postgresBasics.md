# PostgreSQL Basics - CLI Reference

## Installation & Setup (Linux)

```bash
# Install PostgreSQL
sudo apt update
sudo apt install postgresql postgresql-contrib

# Check PostgreSQL status
sudo systemctl status postgresql

# Start/Stop/Restart PostgreSQL
sudo systemctl start postgresql
sudo systemctl stop postgresql
sudo systemctl restart postgresql
```

## PostgreSQL User & Authentication

PostgreSQL uses **roles** for authentication. By default, it creates a `postgres` superuser.

```bash
# Switch to postgres user
sudo -i -u postgres

# Access PostgreSQL prompt directly
sudo -u postgres psql

# Exit PostgreSQL prompt
\q
```

## Essential psql CLI Commands

### Connection & Navigation

```bash
# Connect to specific database
psql -U username -d database_name

# Connect to remote PostgreSQL
psql -h hostname -U username -d database_name -p 5432

# Inside psql:
\l                    # List all databases
\c database_name      # Connect to database
\dt                   # List tables in current database
\d table_name         # Describe table structure
\du                   # List all roles/users
\dn                   # List all schemas
\df                   # List all functions
\dv                   # List all views
\x                    # Toggle expanded display (useful for wide tables)
\timing               # Toggle query execution time display
\q                    # Quit psql
```

### Database Operations

```sql
-- Create database
CREATE DATABASE mydb;

-- Create database with specific owner
CREATE DATABASE mydb OWNER myuser;

-- Drop database (cannot be undone!)
DROP DATABASE mydb;

-- Show current database
SELECT current_database();
```

### User/Role Management

PostgreSQL uses **roles** (users are roles that can login).

```sql
-- Create role with login capability (user)
CREATE ROLE myuser WITH LOGIN PASSWORD 'mypassword';

-- Create superuser
CREATE ROLE admin WITH SUPERUSER LOGIN PASSWORD 'adminpass';

-- Create role with specific privileges
CREATE ROLE developer WITH LOGIN PASSWORD 'devpass' 
    CREATEDB CREATEROLE;

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
GRANT SELECT, INSERT, UPDATE ON table_name TO myuser;

-- Alter role
ALTER ROLE myuser WITH PASSWORD 'newpassword';

-- Drop role
DROP ROLE myuser;
```

## PostgreSQL-Specific Features

### 1. **Schemas** (Namespaces for tables)

PostgreSQL uses schemas to organize tables. Default schema is `public`.

```sql
-- Create schema
CREATE SCHEMA myschema;

-- Create table in specific schema
CREATE TABLE myschema.users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Set search path (default schema)
SET search_path TO myschema, public;

-- View current search path
SHOW search_path;
```

### 2. **SERIAL & Sequences**

PostgreSQL uses `SERIAL` for auto-increment (actually creates a sequence).

```sql
-- Using SERIAL (recommended)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50)
);

-- Equivalent to:
CREATE SEQUENCE users_id_seq;
CREATE TABLE users (
    id INTEGER DEFAULT nextval('users_id_seq') PRIMARY KEY,
    username VARCHAR(50)
);

-- Check sequence value
SELECT currval('users_id_seq');
SELECT nextval('users_id_seq');

-- Reset sequence
ALTER SEQUENCE users_id_seq RESTART WITH 1;
```

### 3. **Data Types** (PostgreSQL-specific)

```sql
-- Array types
CREATE TABLE products (
    id SERIAL,
    tags TEXT[]  -- Array of text
);

INSERT INTO products (tags) VALUES (ARRAY['electronics', 'sale']);
INSERT INTO products (tags) VALUES ('{"books", "fiction"}');

-- JSON & JSONB (binary JSON - faster)
CREATE TABLE events (
    id SERIAL,
    data JSONB
);

INSERT INTO events (data) VALUES ('{"user": "john", "action": "login"}');

-- Query JSON
SELECT data->>'user' FROM events;

-- UUID type
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE TABLE sessions (
    id UUID DEFAULT uuid_generate_v4() PRIMARY KEY
);

-- Enums
CREATE TYPE mood AS ENUM ('happy', 'sad', 'neutral');
CREATE TABLE person (
    name VARCHAR(50),
    current_mood mood
);
```

### 4. **RETURNING Clause**

PostgreSQL can return values after INSERT/UPDATE/DELETE.

```sql
-- Get inserted ID
INSERT INTO users (username) VALUES ('alice') RETURNING id;

-- Get updated rows
UPDATE users SET active = true WHERE id < 10 RETURNING *;

-- Get deleted records
DELETE FROM users WHERE last_login < '2024-01-01' RETURNING username;
```

### 5. **UPSERT (INSERT ... ON CONFLICT)**

```sql
-- Insert or update if conflict
INSERT INTO users (email, name) 
VALUES ('john@example.com', 'John')
ON CONFLICT (email) 
DO UPDATE SET name = EXCLUDED.name;

-- Insert or do nothing
INSERT INTO users (email, name) 
VALUES ('jane@example.com', 'Jane')
ON CONFLICT (email) DO NOTHING;
```

### 6. **COPY Command** (Fast bulk import/export)

```bash
# Export table to CSV
\copy users TO '/tmp/users.csv' CSV HEADER

# Import CSV to table
\copy users FROM '/tmp/users.csv' CSV HEADER

# Export query result
\copy (SELECT * FROM users WHERE active = true) TO '/tmp/active_users.csv' CSV
```

## Configuration Files

```bash
# Find PostgreSQL config files
sudo -u postgres psql -c "SHOW config_file;"
sudo -u postgres psql -c "SHOW hba_file;"

# Main config: postgresql.conf
# Location: /etc/postgresql/{version}/main/postgresql.conf

# Authentication config: pg_hba.conf
# Location: /etc/postgresql/{version}/main/pg_hba.conf
```

### Important postgresql.conf Settings

```conf
listen_addresses = 'localhost'    # IP addresses to listen on
port = 5432                        # TCP port
max_connections = 100              # Maximum concurrent connections
shared_buffers = 128MB             # Memory for caching
```

### pg_hba.conf (Client Authentication)

```conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             postgres                                peer
local   all             all                                     md5
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

After changing config files:
```bash
sudo systemctl restart postgresql
```

## Backup & Restore

```bash
# Backup single database
pg_dump -U postgres database_name > backup.sql

# Backup with custom format (compressed)
pg_dump -U postgres -Fc database_name > backup.dump

# Backup all databases
pg_dumpall -U postgres > all_backup.sql

# Restore from SQL backup
psql -U postgres database_name < backup.sql

# Restore from custom format
pg_restore -U postgres -d database_name backup.dump

# Backup specific table
pg_dump -U postgres -t table_name database_name > table_backup.sql
```

## Useful Queries

```sql
-- Show table size
SELECT pg_size_pretty(pg_total_relation_size('table_name'));

-- Show database size
SELECT pg_size_pretty(pg_database_size('database_name'));

-- Show active connections
SELECT * FROM pg_stat_activity;

-- Kill specific connection
SELECT pg_terminate_backend(pid) FROM pg_stat_activity 
WHERE datname = 'mydb' AND pid <> pg_backend_pid();

-- Show running queries
SELECT pid, age(clock_timestamp(), query_start), usename, query 
FROM pg_stat_activity 
WHERE query != '<IDLE>' AND query NOT ILIKE '%pg_stat_activity%'
ORDER BY query_start DESC;

-- Show table indexes
SELECT * FROM pg_indexes WHERE tablename = 'users';

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM users WHERE id = 1;
```

## Transaction Management

```sql
-- Begin transaction
BEGIN;

-- Your queries here
INSERT INTO accounts (balance) VALUES (100);
UPDATE accounts SET balance = balance - 50 WHERE id = 1;

-- Commit or rollback
COMMIT;
-- or
ROLLBACK;

-- Savepoints
BEGIN;
INSERT INTO users (name) VALUES ('Alice');
SAVEPOINT sp1;
INSERT INTO users (name) VALUES ('Bob');
ROLLBACK TO SAVEPOINT sp1;  -- Bob not inserted, Alice still pending
COMMIT;  -- Alice inserted
```

## Environment Variables

```bash
# Set default PostgreSQL connection variables
export PGHOST=localhost
export PGPORT=5432
export PGDATABASE=mydb
export PGUSER=myuser
export PGPASSWORD=mypassword

# Now you can connect simply with:
psql
```

## Quick Tips

1. **Tab completion**: psql supports tab completion for SQL commands and table names
2. **Command history**: Use ↑/↓ arrows to navigate command history
3. **Multi-line commands**: Press Enter to continue command on next line; end with `;`
4. **Cancel query**: Press `Ctrl+C` to cancel running query
5. **Clear screen**: `\! clear` or `Ctrl+L`
6. **Execute system commands**: `\! ls -la`
7. **Edit in external editor**: `\e` opens last query in $EDITOR
8. **Watch query**: `\watch 2` repeats query every 2 seconds

## Common Issues

### Issue: "peer authentication failed"
**Solution**: Edit `pg_hba.conf` and change `peer` to `md5` for local connections

### Issue: Can't connect to PostgreSQL
```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql

# Check port is listening
sudo netstat -plnt | grep 5432

# Check logs
sudo tail -f /var/log/postgresql/postgresql-*-main.log
```

### Issue: Permission denied for database
```sql
-- Grant necessary permissions
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO myuser;
```

---

## Quick Reference Cheat Sheet

| Task | Command |
|------|---------|
| Connect to DB | `psql -U user -d database` |
| List databases | `\l` |
| Switch database | `\c dbname` |
| List tables | `\dt` |
| Describe table | `\d tablename` |
| Execute SQL file | `psql -U user -d db -f file.sql` |
| Backup database | `pg_dump dbname > backup.sql` |
| Restore database | `psql dbname < backup.sql` |
| Show current user | `SELECT current_user;` |
| Show version | `SELECT version();` |

---

**Note**: Replace `username`, `database_name`, `table_name` with your actual names when using these commands.
