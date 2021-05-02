This experiment uses [Logstash](https://www.elastic.co/logstash) to replicate data between a MySQL database
and an [Elasticsearch](https://www.elastic.co/elasticsearch/) cluster. Inspired by:

https://www.elastic.co/blog/how-to-keep-elasticsearch-synchronized-with-a-relational-database-using-logstash

---

### Sample data

This sets up a local database called `testing` with username `root` and
password `secret` on port `3600`.

(It imports the `schema.sql` file when the container is run.)

---

| Table | Column | Type |
| ------ | ------ | ------ |
| `foobars` | `id` | `BIGINT` |
| | `baz` | `VARCHAR(255)` |
| | `created_at` | `DATETIME` |
| | `updated_at` | `DATETIME` |

---

### Containers

1. `docker compose up mysql`
2. `docker compose up elasticsearch`
3. `docker compose up logstash`

(Doing it manually this way is quick & dirty to avoid race conditions
between dependencies starting and Logstash trying to interact with them.)

---

### Logstash

Runs every 5 seconds, keeping track of the state of each run:

```sql
SELECT *
FROM foobars
WHERE updated_at > :sql_last_value
  AND updated_at < NOW()
```

and upserts the results in Elasticsearch.

---

### Elasticsearch

  * http://localhost:9200/foobars_index/_search?pretty
  * http://localhost:9200/foobars_index/_doc/3?pretty
