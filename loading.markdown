## Loading into Postgres

Python is pretty cool

https://github.com/jasonaowen/document-to-postgres


## Expected format

```sql
CREATE TABLE raw_events (
  event_id SERIAL PRIMARY KEY,
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
zcat 2016-04-24-15.json.gz | grep '\\u0000' | wc -l
```

The double backslash matters.


## Loading

```
zcat 2016-04-24-15.json.gz |
  sed -e 's/\\u0000//g' |
  python json-to-postgres.py owenja demo raw_events event
```

Note: other approaches are valid, here; you could halt the import, or
filter out lines with grep to parse or inspect separately.


## Size comparison

```sql
SELECT nspname || '.' || relname AS "relation",
       pg_size_pretty(pg_total_relation_size(C.oid)) AS "size"
  FROM pg_class C
    LEFT JOIN pg_namespace N
      ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
  AND C.relkind = 'r'
ORDER BY pg_total_relation_size(C.oid) DESC;
```

https://wiki.postgresql.org/wiki/Disk_Usage


## Keep a copy for comparison

```sql
CREATE TABLE events (
  event_id INTEGER PRIMARY KEY
    REFERENCES raw_events(event_id),
  event JSONB NOT NULL
);

INSERT INTO events (event_id, event)
SELECT event_id, event
FROM raw_events;
```
