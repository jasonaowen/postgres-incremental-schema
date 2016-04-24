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


## Postgres Types

```sql
CREATE TABLE json_demo (
  id SERIAL PRIMARY KEY,
  interpreted JSON,
  compiled JSONB
);
```

http://www.postgresql.org/docs/current/static/datatype-json.html


A single data point:
<table>
  <thead>
    <tr>
      <th>Type</th>
      <th>Load time</th>
      <th>Size on disk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>JSON</td>
      <td>47.85 s</td>
      <td>127 MB</td>
    </tr>
    <tr>
      <td>JSONB</td>
      <td>53.71 s</td>
      <td>143 MB</td>
    </tr>
  </tbody>
</table>

Note: timed on my laptop, using the loading script introduced later on the data
sets `2016-01-01-1.json.gz`, `2016-01-15-15.json.gz`, and
`2016-02-18-23.json.gz`.


## Tradeoffs

<table>
  <thead>
    <tr>
      <th>Type</th>
      <th>Query time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>JSON</td>
      <td>2.044 s</td>
    </tr>
    <tr>
      <td>JSONB</td>
      <td>1.359 s</td>
    </tr>
  </tbody>
</table>

tl;dr: use JSONB.

Note: queries will be introduced later, but were `SELECT
JSON_OBJECT_KEYS(data), COUNT(*) FROM interpreted GROUP BY 1` and `SELECT
JSONB_OBJECT_KEYS(data), COUNT(*) FROM compiled GROUP BY 1`, run twice, and the
lower score reported.


## How to work with it

```sql
SELECT * FROM demo;
SELECT data->'bar' FROM demo WHERE id = 1;
SELECT data->1 FROM demo WHERE id = 2;
```

http://www.postgresql.org/docs/current/static/functions-json.html

Note: `CREATE TABLE demo (id SERIAL PRIMARY KEY, data JSONB);`
`INSERT INTO demo (data) VALUES ('{"bar": "baz", "balance": 7.77, "active": false}'::json);`
`INSERT INTO demo (data) VALUES ('["first", "second", 3, true]'::json);`


## [Set returning functions](http://www.postgresql.org/docs/current/static/functions-srf.html)

```sql
SELECT generate_series(1, 5);
SELECT 'a', generate_series(1, 5);

SELECT id, jsonb_each(data) FROM demo WHERE id = 1;
SELECT id, jsonb_array_elements(data) FROM demo WHERE id = 2;
```
