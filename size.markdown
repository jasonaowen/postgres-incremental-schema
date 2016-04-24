## Size comparison

```sql
SELECT nspname || '.' || relname AS "relation",
       pg_size_pretty(pg_total_relation_size(C.oid)) AS "size"
  FROM pg_class C
    LEFT JOIN pg_namespace N
      ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
  AND C.relkind = 'r'
ORDER BY pg_total_relation_size(C.oid) DESC;
```


## Springtime

```sql
VACUUM FULL VERBOSE;
```

http://www.postgresql.org/docs/current/static/sql-vacuum.html


## Size comparison again

```sql
SELECT nspname || '.' || relname AS "relation",
       pg_size_pretty(pg_total_relation_size(C.oid)) AS "size"
  FROM pg_class C
    LEFT JOIN pg_namespace N
      ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
  AND C.relkind = 'r'
ORDER BY pg_total_relation_size(C.oid) DESC;
```

https://wiki.postgresql.org/wiki/Disk_Usage
