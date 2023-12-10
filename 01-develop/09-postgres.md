# Postgres Notes

## Changing runtime parameters in Postgres

There are several ways to change runtime parameters in Postgres.

1. **Using the `SET` Command:** The `SET` command changes the value of a configuration parameter for the duration of the current session or transaction. These changes do not persist after the session or transaction ends.

For example:

```sql
SET parameter_name = 'value';
```

2. **Using the `ALTER SYSTEM` Command:** This command is used to change the value of a configuration parameter in the `postgresql.auto.conf` file. This file is read at server start-up and after the main `postgresql.conf` file. Changes made with `ALTER SYSTEM` are applied globally and persist across server restarts.
   Example:

```sql
ALTER SYSTEM SET parameter_name TO 'value';
```

3. **Modifying the `postgresql.conf` File Directly:** You can directly edit the `postgresql.conf` file to change the default values of parameters. After editing, you usually need to reload the PostgreSQL server for the changes to take effect (a full restart is not always necessary, depending on the parameter).
   To reload the configuration, after modifying `postgres.conf`, use:

```sql
SELECT pg_reload_conf();
```
