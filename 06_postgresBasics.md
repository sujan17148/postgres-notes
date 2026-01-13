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
# Connect to a local database (no -h)
psql -U username -d database_name
# Usually connects via the local Unix socket.
# On many Linux setups, local socket auth is "peer" (often no password prompt),
# BUT it only works if your Linux username matches the PostgreSQL role.
# Example: `sudo -u postgres psql` works because Linux user "postgres" matches DB role "postgres".

# Connect using TCP (local or remote)
psql -h hostname -U username -d database_name -p 5432
# Using -h forces TCP/IP.
# With typical configs (like yours), TCP uses scram-sha-256, so it will prompt for a password.
# Local TCP example:  psql -h 127.0.0.1 -U dev_user -d myapp_db
# Remote example:     psql -h your.server.ip -U dev_user -d myapp_db
```

### Basic CLI Commands
```bash 
\l                          -- List all databases
\c database_name            -- Connect to database (keeps current user)
\c database_name user_name  -- Connect to database as user_name (auth rules apply)

\dt                         -- List tables in current database
\d table_name               -- Describe table structure

\du                         -- List all roles/users
\dn                         -- List all schemas
\df                         -- List all functions
\dv                         -- List all views

\x                          -- Toggle expanded display (better for wide output)
\timing                     -- Toggle query execution time display

\q                          -- Quit psql
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