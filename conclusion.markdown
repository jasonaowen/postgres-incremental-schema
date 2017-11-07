## Answering the question

What are the first commits added
to repos newly made public?


```sql
WITH public_repos AS (
  SELECT events.repo_id
  FROM public_events
    JOIN events ON public_events.event_id = events.event_id
)
SELECT repos.name, authors.name, commits.message
FROM push_events
  JOIN events ON push_events.event_id = events.event_id
  JOIN public_repos ON events.repo_id = public_repos.repo_id
  JOIN push_commits ON push_commits.push_id = push_events.id
  JOIN commits ON commits.id = push_commits.commit_id
  JOIN repos ON repos.repo_id = events.repo_id
  JOIN authors ON authors.author_id = commits.author_id;
```


## Thank you

@jasonaowen

https://jasonaowen.github.io/postgres-incremental-schema
https://github.com/jasonaowen/postgres-incremental-schema
