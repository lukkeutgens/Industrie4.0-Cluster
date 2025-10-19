# Create the Keycloak Database in PostgreSQL
Keycloak uses PostgreSQL as its backend database. Since PostgreSQL is already running in our cluster, we can manually create the required database and user.

## 1. Open a shell in the PostgreSQL pod
Our PostgreSQL pod is named `timescaledb-0`. Start a shell session:
```bash
kubectl exec -it timescaledb-0 -n postgresql -- bash
```

## 2. Log in to PostgreSQL
Use the credentials defined in your Kubernetes Secret:
```bash
psql -U iotadmin -d postgres
```
If successful, you'll see the prompt:
```bash
postgres=#
```

## 3. Create the Keycloak database and user
```sql
CREATE USER keycloak_user WITH PASSWORD '********';
CREATE DATABASE keycloak OWNER keycloak_user;
GRANT ALL PRIVILEGES ON DATABASE keycloak TO keycloak_user;
```

## 4. Verify the database and user
Use these commands to confirm that the database and user were created:
```sql
\l                     # List all databases
\du                    # List all users (roles)
```

## 5. Exit PostgreSQL and the shell
```sql
\q
exit
```

---

The database is now up and running in PostgreSQL to accept data comming from Keycloak



