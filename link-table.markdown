## Polymorphism

The `payload` type is a little different.

```sql
SELECT * FROM key_count('events', 'payload');
```


## Varies with type

```sql
SELECT type,
       jsonb_object_keys(event),
       COUNT(*)
FROM events
GROUP BY type, jsonb_object_keys(event)
```


## Let's do PushEvent

```sql
CREATE TABLE pushes (
  id SERIAL PRIMARY KEY,
  event_id INTEGER NOT NULL
    REFERENCES events(id),
  payload JSONB NOT NULL);

INSERT INTO pushes (event_id, payload)
SELECT id, payload FROM events WHERE type = 'PushEvent';
```

Note that the foreign key points the other way!


## Most of these are mechanical

```sql
SELECT * FROM key_count('pushes', 'payload');
SELECT explode_json_column('pushes', 'payload');
SELECT set_concrete_type('pushes', 'before', 'TEXT');
SELECT set_concrete_type('pushes', 'distinct_size', 'INTEGER');
SELECT set_concrete_type('pushes', 'head', 'TEXT');
SELECT set_concrete_type('pushes', 'push_id', 'BIGINT');
SELECT set_concrete_type('pushes', 'ref', 'TEXT');
SELECT set_concrete_type('pushes', 'size', 'INTEGER');
```


## But what's that `commits` column?

```sql
SELECT * FROM type_count('pushes', 'commits');
SELECT commits->0 FROM pushes WHERE id = 1;
SELECT jsonb_array_elements(commits) FROM pushes WHERE id = 1;
```


## Sounds like a link table

```sql
CREATE TABLE push_commits (
  id SERIAL PRIMARY KEY,
  push_id INTEGER NOT NULL
    REFERENCES PUSHES(id),
  commit JSONB NOT NULL
);

INSERT INTO push_commits (push_id, commit)
SELECT id, jsonb_array_elements(commits) FROM pushes;
```


## Link target

```sql
CREATE TABLE commits (
  id SERIAL PRIMARY KEY,
  payload JSONB NOT NULL
);

INSERT INTO commits (payload)
SELECT DISTINCT commit FROM push_commits;
```


## Large values are hard

```sql
ALTER TABLE push_commits
  ADD COLUMN commit_id INTEGER
    REFERENCES commits (id);

CREATE UNIQUE INDEX ON commits (payload);
CREATE UNIQUE INDEX ON commits (MD5(payload::TEXT));
CREATE INDEX ON push_commits (MD5(commit::TEXT));
```

Note: we don't need a unique index on `push_commits` because the same commit
can be in multiple pushes.


```sql
UPDATE push_commits SET commit_id =
  (SELECT id FROM commits WHERE MD5(commit::TEXT) = MD5(payload::TEXT));

ALTER TABLE push_commits
  ALTER COLUMN commit_id SET NOT NULL,
  DROP COLUMN commit,
  DROP COLUMN id,
  ADD PRIMARY KEY (push_id, commit_id);
```


## Finish with `commits`

```sql
SELECT * FROM type_count('commits', 'payload');
SELECT * FROM key_count('commits', 'payload');
SELECT explode_json_column('commits', 'payload');
SELECT set_concrete_type('commits', 'distinct', 'BOOLEAN');
SELECT set_concrete_type('commits', 'message', 'TEXT');
SELECT set_concrete_type('commits', 'sha', 'TEXT');
SELECT set_concrete_type('commits', 'url', 'TEXT');
```


## Good break point

We could extract the authors table here.

```sql
SELECT * FROM type_count('commits', 'author');
SELECT * FROM key_count('commits', 'author');
```

But we won't, as it doesn't cover anything new.


## Clean up

```sql
ALTER TABLE commits DROP COLUMN payload;
ALTER TABLE pushes DROP COLUMN payload,
  DROP COLUMN commits;
UPDATE events SET payload = NULL WHERE type = 'PushEvent';
```


## Size comparison

```sql
SELECT nspname || '.' || relname AS "relation",
       pg_size_pretty(pg_relation_size(C.oid)) AS "size"
  FROM pg_class C
    LEFT JOIN pg_namespace N
      ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_relation_size(C.oid) DESC;
```

https://wiki.postgresql.org/wiki/Disk_Usage
