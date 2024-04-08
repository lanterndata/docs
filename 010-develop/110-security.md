# Security

## Keep your connection secure

When connecting to Lantern with Postgres clients such as Python's [psycopg2](https://pypi.org/project/psycopg2/) or shell's `psql` command line tool, you can manually verify SSL certificates presented by your Lantern Cloud database.

By default, postgres clients run in `sslmode=prefer` which means the connection will be fully encrypted but your client will not verify the certificates provided by the server.

You can enforce certificate verification by specifying `sslmode=verify-full`. You can read more about supported Postgres SSL modes [here](https://www.postgresql.org/docs/current/libpq-ssl.html)

When verifying certificates, you can use the trusted certificate authority (CA) list to verify the server certificates. Alternatively, you can use this [Lantern Cloud Root Certificate list](https://storage.googleapis.com/lantern-web/root.crt)
