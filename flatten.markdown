## Getting data out of a JSON object

Now that we know what's in those objects, let's get them into columns.


## So much DDL

```sql
ALTER TABLE events
ADD COLUMN actor JSONB,
ADD COLUMN created_at JSONB,
ADD COLUMN id JSONB,
ADD COLUMN org JSONB,
ADD COLUMN payload JSONB,
ADD COLUMN public JSONB,
ADD COLUMN repo JSONB,
ADD COLUMN type JSONB;
```


```sql
UPDATE events
SET actor = event->'actor',
    created_at = event->'created_at',
    id = event->'id',
    org = event->'org',
    payload = event->'payload',
    public = event->'public',
    repo = event->'repo',
    type = event->'type';
```
