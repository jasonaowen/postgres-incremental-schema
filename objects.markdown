## What's in a JSON?

Scalars:
- Boolean
- Number
- String
- null

Containers:
- Array
- Object


## What's in an object?

```json
{
  "created_at": "2011-09-06T17:26:27Z",
  "id": 12345
}
```

Note: keys and values. Keys are always strings; values can be any JSON type.
Keys may _not_ be unique!


## What's in our database?

```sql
SELECT jsonb_typeof(event),
       COUNT(*)
FROM events
GROUP BY jsonb_typeof(event);
```


## We'll be doing that a lot

```sql
CREATE FUNCTION type_count(in table_name text, in column_name text)
  RETURNS TABLE (json_type text, count bigint) AS
  $func$
    BEGIN
      RETURN QUERY EXECUTE format(
        'SELECT jsonb_typeof("%s"),
                COUNT(*)
         FROM "%s"
         GROUP BY jsonb_typeof("%s")',
      column_name, table_name, column_name);
    END
  $func$ LANGUAGE plpgsql STABLE;

select * from type_count('events', 'event');
```


## What are those objects?

```sql
SELECT jsonb_object_keys(event),
       COUNT(*)
FROM events
GROUP BY jsonb_object_keys(event);
```


## We'll be doing that a lot, too

```sql
CREATE FUNCTION key_count(in table_name text, in column_name text)
  RETURNS TABLE (key text, count bigint) AS
  $func$
    BEGIN
      RETURN QUERY EXECUTE format(
        'SELECT jsonb_object_keys("%s"),
                COUNT(*)
         FROM "%s"
         GROUP BY jsonb_object_keys("%s")',
      column_name, table_name, column_name);
    END
  $func$ LANGUAGE plpgsql STABLE;

select * from key_count('events', 'event');
```
