## Extracting a table

Let's normalize a little.


## Actors gonna act

```sql
SELECT * FROM type_count('events', 'actor');

SELECT COUNT(DISTINCT actor) FROM events;

SELECT * FROM key_count('events', 'actor');
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

Note: without that UNIQUE constraint, this update would have taken _far_
longer.


## Finish the move

```sql
ALTER TABLE events
  ALTER COLUMN actor_id SET NOT NULL,
  DROP COLUMN actor;
```


## Now we can start working on the actors table

```sql
SELECT explode_json_column('actors', 'payload');

SELECT * FROM type_count('actors', 'avatar_url');
SELECT set_concrete_type('actors', 'avatar_url', 'TEXT');

SELECT * FROM type_count('actors', 'id');
SELECT set_concrete_type('actors', 'id', 'BIGINT');
```


```sql
SELECT * FROM type_count('actors', 'login');
SELECT set_concrete_type('actors', 'login', 'TEXT');

SELECT * FROM type_count('actors', 'url');
SELECT set_concrete_type('actors', 'url', 'TEXT');
```


## Clean up data

```sql
SELECT * FROM type_count('actors', 'gravatar_id');
SELECT DISTINCT gravatar_id FROM actors;

ALTER TABLE actors DROP COLUMN gravatar_id;
```

If it's [always
empty](https://developer.github.com/changes/2014-09-05-removing-gravatar-id/),
do we still need it?


## Finish up with actors

```sql
ALTER TABLE actors DROP COLUMN payload;
```
