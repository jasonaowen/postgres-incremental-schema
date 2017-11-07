## Loading into Postgres

Python is pretty cool

https://github.com/jasonaowen/document-to-postgres


## Expected format

```sql
CREATE TABLE events (
  event_id SERIAL PRIMARY KEY,
  event JSONB NOT NULL
);
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

https://www.postgresql.org/docs/current/static/datatype-json.html


## Need to filter it

```
zcat 2017-11-01-11.json.gz | grep '\\u0000' | wc -l
```

The double backslash matters.


## Loading

```
zcat 2017-11-01-11.json.gz |
  sed -e 's/\\u0000//g' |
  python json-to-postgres.py owenja demo events event
```

Note: other approaches are valid, here; you could halt the import, or
filter out lines with grep to parse or inspect separately.
