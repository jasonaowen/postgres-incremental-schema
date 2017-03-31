## Correcting scalar data types

PostgreSQL seems not to be able to automatically, implicitly convert between a
JSON[B] type and another scalar.


## TEXT is the great equalizer

```sql
SELECT jsonb_typeof(created_at), COUNT(*) FROM events GROUP BY 1;
```

```psql
 jsonb_typeof | count
--------------+-------
 string       | 35591
(1 row)
```

```sql
SELECT created_at FROM events LIMIT 1;
```

```psql
       created_at
------------------------
 "2016-04-24T15:09:48Z"
(1 row)
```

```sql
ALTER TABLE events
ALTER COLUMN created_at
  TYPE TIMESTAMP WITH TIME ZONE
  USING created_at::TEXT::TIMESTAMP WITH TIME ZONE;
```


## And again...

```sql
SELECT jsonb_typeof(type), COUNT(*) FROM events GROUP BY 1;
```
```psql
 jsonb_typeof | count
--------------+-------
 string       | 35591
(1 row)
```
```sql
ALTER TABLE events
ALTER COLUMN type
  TYPE TEXT
  USING type::TEXT;
```


## What're those quotes doing there?

```sql
SELECT type FROM events LIMIT 1;
```
```psql
        type
--------------------
 "PullRequestEvent"
(1 row)
```
```sql
UPDATE events
SET type = TRIM(BOTH '"' FROM type);
```


## Last scalar

```sql
SELECT jsonb_typeof(id), COUNT(*) FROM events GROUP BY 1;
```
```psql
 jsonb_typeof | count
--------------+-------
 string       | 35591
(1 row)
```
```sql
SELECT id FROM events LIMIT 5;
```
```psql
      id
--------------
 "3928878336"
 "3928878670"
(2 rows)
```
```sql
ALTER TABLE events
ALTER COLUMN id
  TYPE BIGINT
  USING TRIM(BOTH '"' FROM id::TEXT)::BIGINT;
```
