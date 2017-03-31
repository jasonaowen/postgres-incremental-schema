## What's in our database?

```sql
SELECT jsonb_typeof(event),
       COUNT(*)
FROM events
GROUP BY 1;
```
```psql
 jsonb_typeof | count
 --------------+-------
  object       | 35591
  (1 row)
```


## What are those objects?

```sql
SELECT jsonb_object_keys(event),
       COUNT(*)
FROM events
GROUP BY 1;
```
```psql
 jsonb_object_keys | count
-------------------+-------
 public            | 35591
 repo              | 35591
 id                | 35591
 org               |  8615
 actor             | 35591
 payload           | 35591
 created_at        | 35591
 type              | 35591
(8 rows)

```
