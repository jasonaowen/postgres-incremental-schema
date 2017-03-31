## Extracting a table

Let's normalize a little.


## Actors gonna act

```sql
SELECT jsonb_typeof(actor), COUNT(*) FROM events GROUP BY 1;
```
```psql
 jsonb_typeof | count
--------------+-------
 object       | 35591
(1 row)
```
```sql
SELECT COUNT(DISTINCT actor) FROM events;
```
```psql
 count
-------
 15228
(1 row)
```


## Give them their own place

```sql
CREATE TABLE actors (
  actor_id SERIAL PRIMARY KEY,
  payload JSONB UNIQUE NOT NULL
);

INSERT INTO actors (payload)
SELECT DISTINCT actor FROM events;
```


## But keep them connected

```sql
ALTER TABLE events
  ADD COLUMN actor_id INTEGER
  REFERENCES actors (actor_id);

UPDATE events
  SET actor_id =
    (SELECT actor_id FROM actors WHERE actor = payload);
```

Note: without that UNIQUE constraint on payload, this update would have taken _far_
longer.
