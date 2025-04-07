# ⚡️ Hasura Live Query Multiplexing Demo

This project demonstrates how [Hasura](https://hasura.io) handles **GraphQL subscriptions (live queries)** efficiently using **multiplexed queries** and **orchestrated polling** to reduce database load.

The demo shows how multiple clients subscribing to similar queries are served with a **single SQL query**, thanks to Hasura's live query engine.

---

## 🚪 What You'll See

- How Hasura batches live queries with different `tenant_id`s
- Query multiplexing in PostgreSQL logs
- Environment variables that control polling and batching behavior

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/wallace-prado/hasura-live-query-demo.git
cd hasura-live-query-demo
```

### 2. Start the Environment

This starts Postgres + Hasura with live query logging enabled:

```bash
docker compose up --build -d
```

---

## 🔍 Try It Yourself

### 1. Open the Hasura Console

Visit [http://localhost:8080](http://localhost:8080) and go to the GraphiQL tab.

### 2. Apply Hasura Metadata (Optional)

If metadata is not already loaded automatically, you can apply it using:

```bash
hasura metadata apply --endpoint http://localhost:8080 --admin-secret <your-secret-if-any>
```

This step is optional depending on your setup. In this project, Hasura metadata should be picked up automatically via volume mount.

### 3. Run a Live Query in Window 1

```graphql
subscription getCustomer {
  customers(
    limit: 5
    where: { tenant_id: { _eq: "1bb9e8c6-3c77-4cf4-9480-3466107e195e" } }
  ) {
    customer_id
    customer_name
  }
}
```

### 4. Run Another Live Query in Window 2

Open an incognito window or a different browser and run:

```graphql
subscription getCustomer {
  customers(
    limit: 5
    where: { tenant_id: { _eq: "46715724-b1d4-429e-9106-8b567c3c4aa2" } }
  ) {
    customer_id
    customer_name
  }
}
```

---

## 🐘 Observe the PostgreSQL Logs

To see the multiplexed SQL queries:

```bash
docker logs -f hasura-pg
```

You’ll see a single SQL query like this (truncated):

```sql
SELECT "__subs"."result_id", "__fld_resp"."root" AS "result"
FROM UNNEST(($1)::uuid[], ($2)::json[]) AS "__subs"("result_id", "result_vars")
...
WHERE "tenant_id" = (("__subs"."result_vars"#>>ARRAY['synthetic', '0'])::uuid)
```

Multiple tenant values are passed in as `UNNEST()` variables — proof of **multiplexing**.

---

## ⚙️ Configuration Highlights

In `docker-compose.yml`, Hasura is tuned with:

```yaml
HASURA_GRAPHQL_LIVE_QUERIES_MULTIPLEXED_REFETCH_INTERVAL: 2000 # Poll every 2s
HASURA_GRAPHQL_LIVE_QUERIES_MULTIPLEXED_BATCH_SIZE: 100 # Batch up to 100
```

Postgres logs all queries:

```yaml
command: postgres -c log_statement=all -c logging_collector=off
```

---

## 🪩 Cleaning Up

To stop the containers:

```bash
docker compose down
```

To reset and re-run everything cleanly:

```bash
docker compose down -v
docker compose up --build -d
```

---

## 📚 References

- [Hasura Live Queries](https://hasura.io/docs/latest/subscriptions/live-queries/)
- [Query Multiplexing Explained](https://hasura.io/blog/graphql-live-queries-with-multiplexed-query-execution/)
