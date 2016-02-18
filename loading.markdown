## Loading into Postgres

Python is pretty cool

https://github.com/jasonaowen/document-to-postgres


## Expected format

```sql
CREATE TABLE raw_events (
  id SERIAL PRIMARY KEY,
  event JSONB NOT NULL
)
```


## Not lossless

Unicode allows escaped 0 bytes, but if you try to look at a JSON object with one:

```
unsupported Unicode escape sequence
DETAIL:  \u0000 cannot be converted to text.
CONTEXT:  JSON data, line 1: ...yload": {"action": "created", "comment": {"body":...

```


## JSONB, too

> The jsonb type also rejects \u0000 (because that cannot be represented in PostgreSQL's text type)

http://www.postgresql.org/docs/9.5/static/datatype-json.html


## Need to filter it

```
zcat 2016-02-19-01.json.gz | grep '\\u0000' | wc -l
```

The double backslash matters.

Note: other approaches are valid, here; you could halt the import, or
substitute a placeholder using sed.


## Loading

```
zcat 2016-02-19-01.json.gz |
  grep -v '\\u0000' |
  python json-to-postgres.py owenja pdxpug raw_events event
```


## Size comparison

```sql
SELECT nspname || '.' || relname AS "relation",
       pg_size_pretty(pg_relation_size(C.oid)) AS "size"
  FROM pg_class C
    LEFT JOIN pg_namespace N
      ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_relation_size(C.oid) DESC;
```

https://wiki.postgresql.org/wiki/Disk_Usage

Note: will someone write this down?


## Keep a copy for comparison

```sql
CREATE TABLE events (
  id INTEGER PRIMARY KEY
    REFERENCES raw_events(id),
  event JSONB NOT NULL
);

INSERT INTO events (id, event)
SELECT id, event
FROM raw_events;
```
