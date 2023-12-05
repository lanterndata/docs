# Migrate from Postgres to Lantern Cloud

This guide assumes that you are currently using Postgres without `pgvector`. If you are using `pgvector`, please reference [this guide](/docs/migrate/pgvector-cloud) instead.

## Steps

1. Create a Lantern Cloud database

   Sign up for [Lantern Cloud](/) and create a database. Obtain a database URL. We'll call this `LANTERN_DATABASE_URL`.

2. Backup the Source Database

   Use the `pgdump` utility to create a database backup.

   ```bash
   pg_dump $OLD_DATABASE_URL > backup.sql
   ```

3. (Optional) Stop the old database

   You may want to disable the old database at this time, to prevent data from being dropped.

4. Transfer the Data

   ```sql
   psql $LANTERN_DATABASE_URL < backup.sql
   ```

5. Use the new database

   Update any applications, scripts, or services to point to the new database. You're done!
