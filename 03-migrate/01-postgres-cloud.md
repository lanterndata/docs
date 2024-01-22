# Migrate from Postgres to Lantern Cloud

This guide assumes that you are currently using Postgres without `pgvector`. If you are using `pgvector`, please reference [this guide](/docs/migrate/pgvector-cloud) instead. A few options are available for migrating from Postgres to Lantern Cloud.

## Steps (bash)

1. Create a Lantern Cloud database

   Sign up for [Lantern Cloud](/) and create a database. Obtain a database URL. We'll call this `LANTERN_DATABASE_URL`.

2. Backup the old database using `pg_dump`

   Use the `pgdump` utility to create a database backup. We assume below that your old database URL is stored in the environment variable `OLD_DATABASE_URL`.

   ```bash
   pg_dump $OLD_DATABASE_URL > backup.sql
   ```

3. (Optional) Stop the old database

   You may want to disable the old database at this time, to prevent missing data.

4. Transfer the data to Lantern Cloud database using `psql`

   ```sql
   psql $LANTERN_DATABASE_URL < backup.sql
   ```

5. Use the new database

   Update any applications, scripts, or services to point to the new database. You're done!

## Steps (TablePlus)

[TablePlus](https://tableplus.com) is a GUI for managing databases. It is available for Mac, Windows, and Linux.

1. Create a Lantern Cloud database

   Sign up for [Lantern Cloud](/) and create a database.

2. Backup the old database

   Add the old database to TablePlus. Export the database to a CSV, JSON or SQL file using steps defined [here](https://docs.tableplus.com/gui-tools/import-and-export#export-data)

3. (Optional) Stop the old database

   You may want to disable the old database at this time, to prevent missing data.

4. Transfer the data to Lantern Cloud database

   Add the Lantern Cloud database to TablePlus. Import the data using steps defined [here](https://docs.tableplus.com/gui-tools/import-and-export#import-data)

5. Use the new database

   Update any applications, scripts, or services to point to the new database. You're done!
