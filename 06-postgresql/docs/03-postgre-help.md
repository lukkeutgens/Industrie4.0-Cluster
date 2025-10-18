# PostgreSQL + timescaleDB Quick Help
This guide provides quick reference commands for interacting with your PostgreSQL + TimescaleDB instance running in Kubernetes.

## Accessing the PostgreSQL Console
### Open a shell in the pod
Use the following command to open the shell:
```bash
kubectl exec -it timescaledb-0 -n postgresql -- bash
```
Exiting the shell:
```bash
exit
```

### Log in to PostgreSQL
Use the credentials defined in your Kubernetes Secret (POSTGRES_USER, POSTGRES_DB):
```bash
psql -U iotadmin -d postgres
```
If successful, you'll see the prompt:
```bash
postgres=#
```
Exiting the Console:
```bash
\q
```

---

## Useful Commands
```sql
\conninfo              # Show current connection info
\l                     # List all databases
\du                    # List all users (roles)
\c your_database_name  # Switch to another database
\dt                    # List all tables in current database
\d tablename           # Describe a specific table
```

---

## Basic SQL Examples
Create a new database:
```sql
CREATE DATABASE iotdb;
```
Connect to the new database:
```bash
\c iotdb
```
Create a table;
```sql
CREATE TABLE sensors (
  id SERIAL PRIMARY KEY,
  name TEXT,
  reading DOUBLE PRECISION,
  created_at TIMESTAMPTZ DEFAULT now()
);
```
Insert a row:
```sql
INSERT INTO sensors (name, reading) VALUES ('sensor-1', 42.7);
```
Query data:
```sql
SELECT * FROM sensors;
```

---

## TimescaleDB Specific
Check TimescaleDB extension
```sql
\dx
```
If not installed, enable it:
```sql
CREATE EXTENSION IF NOT EXISTS timescaledb;
```


