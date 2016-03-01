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


## We might do that a few times, too

```sql
CREATE FUNCTION explode_json_column(
  in table_name text,
  in column_name text
) RETURNS void AS
  $func$
    DECLARE
      record RECORD;
    BEGIN
      FOR record IN SELECT key, count
                    FROM key_count(table_name, column_name) LOOP
        EXECUTE format('ALTER TABLE "%s" ADD COLUMN "%s" JSONB',
                       table_name, record.key);
        EXECUTE format('UPDATE "%s" SET "%s" = "%s"->''%s''',
                       table_name, record.key, column_name, record.key);
      END LOOP;
    END
  $func$ LANGUAGE plpgsql;
```
