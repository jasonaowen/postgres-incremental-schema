## Correcting scalar data types

PostgreSQL seems not to be able to automatically, implicitly convert between a
JSON[B] type and another scalar.


## TEXT is the great equalizer

```sql
SELECT * FROM type_count('events', 'created_at');
SELECT created_at FROM events LIMIT 5;

ALTER TABLE events
ALTER COLUMN created_at
  TYPE TIMESTAMP
  USING created_at::TEXT::TIMESTAMP;
```


## And again

```sql
SELECT * FROM type_count('events', 'public');
SELECT public FROM events LIMIT 5;

ALTER TABLE events
ALTER COLUMN public
  TYPE BOOLEAN
  USING public::TEXT::BOOLEAN;
```

Note: the `public` field in this dataset is always a `True` value; probably you
could drop it.


## And again...

```sql
SELECT * FROM type_count('events', 'type');
SELECT type FROM events LIMIT 5;

ALTER TABLE events
ALTER COLUMN type
  TYPE TEXT
  USING type::TEXT::TEXT;
```


## What're those quotes doing there?

```sql
SELECT type FROM events LIMIT 5;

UPDATE events
SET type = TRIM(BOTH '"' FROM type);
```


## Maybe let's automate

```sql
CREATE FUNCTION set_concrete_type(
  in table_name text,
  in column_name text,
  in type_name text
) RETURNS void AS
  $func$
  BEGIN
    EXECUTE format(
      'ALTER TABLE "%s" ALTER COLUMN "%s" TYPE %s
         USING TRIM(BOTH ''"'' FROM "%s"::TEXT)::%s',
      table_name, column_name, type_name, column_name, type_name);
  END
  $func$ LANGUAGE plpgsql;
```


## Last scalar

```sql
SELECT * FROM type_count('events', 'github_id');
SELECT github_id FROM events LIMIT 5;

SELECT set_concrete_type('events', 'github_id', 'BIGINT');
```
