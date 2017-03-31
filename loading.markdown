## Loading into Postgres

```sql
CREATE TABLE events (
  event_id SERIAL PRIMARY KEY,
  event JSONB NOT NULL
)
```

https://github.com/jasonaowen/document-to-postgres
