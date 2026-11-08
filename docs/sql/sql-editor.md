# SQL editor

[All parts](README.md) · 59 statements

## api/sql.py

`_lock_timeout_source` line 40

```sql
SELECT setting, source, COALESCE(sourcefile, '') FROM pg_settings WHERE name = 'lock_timeout'
```

`_ddl_plan` line 227

```sql
SELECT current_setting('server_version_num')::int AS v
```

`_lock_queue_snapshot` line 263

```sql
SELECT wait_event_type, wait_event,
       EXTRACT(EPOCH FROM (now() - state_change))::float AS waited_s
  FROM pg_stat_activity WHERE pid = %s
```

`_lock_queue_snapshot` line 275

```sql
SELECT COALESCE(
         (SELECT l.relation::regclass::text FROM pg_locks l
           WHERE l.pid = %s AND NOT l.granted AND l.relation IS NOT NULL
           LIMIT 1),
         (SELECT c.oid::regclass::text
            FROM pg_locks l JOIN pg_class c ON c.oid = l.relation
           WHERE l.pid = %s AND l.granted
           ORDER BY CASE l.mode WHEN 'AccessExclusiveLock' THEN 1
                                WHEN 'ExclusiveLock' THEN 2 ELSE 9 END
           LIMIT 1)
       ) AS obj
```

`_detect_source` line 357

```sql
SELECT n.nspname, c.relname, c.relkind::text   FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace  WHERE c.oid = %s
```

`_detect_source` line 367

```sql
SELECT a.attname FROM pg_index i   JOIN pg_class c ON c.oid = i.indrelid   JOIN pg_namespace n ON n.oid = c.relnamespace   JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum = ANY(i.indkey)  WHERE i.indisprimary AND n.nspname = %s AND c.relname = %s  ORDER BY (SELECT u.ord FROM unnest(i.indkey::smallint[]) WITH ORDINALITY AS u(k, ord) WHERE u.k = a.attnum LIMIT 1)
```

`_detect_source` line 380

```sql
SELECT attnum, attname FROM pg_attribute  WHERE attrelid = %s AND attnum > 0 AND NOT attisdropped
```

`api_sql_exec` line 563

```sql
SET LOCAL statement_timeout = %s
```

`api_sql_exec` line 565

```sql
SET LOCAL lock_timeout = %s
```

`api_sql_exec` line 574

```sql
SHOW transaction_read_only
```

`api_sql_exec` line 577

```sql
SET LOCAL transaction_read_only = on
```

`api_sql_exec` line 593

```sql
SET application_name = %s
```

`api_sql_exec` line 597

```sql
SET search_path = {quoted}
```

`_search_path_diagnosis` line 956

```sql
SHOW search_path
```

`_resolve_missing_relation` line 985

```sql
SELECT n.nspname
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.relname = %s
   AND n.nspname NOT IN ('pg_catalog','information_schema')
   AND n.nspname NOT LIKE 'pg_toast%%'
   AND n.nspname NOT LIKE 'pg_temp%%'
 ORDER BY n.nspname
```

`api_sql_validate` line 1050

```sql
SET search_path = {_quote_search_path(sp_parts)}
```

`api_sql_validate` line 1053

```sql
EXPLAIN (FORMAT TEXT, GENERIC_PLAN) {sql}
```

`api_sql_actual_plans` line 1270

```sql
SELECT current_database() AS d
```

`api_sql_explain` line 1306

```sql
COPY table directly (without SELECT) cannot be EXPLAINed. Use COPY (SELECT ...) TO ...
```

**error** - `api_sql_explain` line 1319

```sql
EXPLAIN does not support this statement (allowed: {', '.join(_EXPLAINABLE_PREFIXES)})
```

`api_sql_explain._apply_sp` line 1348

```sql
SET search_path = {_quote_search_path(sp_parts)}
```

`api_sql_explain._run_explain` line 1379

```sql
EXPLAIN (FORMAT JSON, GENERIC_PLAN) {sql}
```

`api_sql_explain._run_explain` line 1387

```sql
PREPARE {pname} ({ptypes}) AS {sql}
```

`api_sql_explain._run_explain` line 1389

```sql
EXPLAIN (FORMAT JSON) EXECUTE {pname}({nulls})
```

`api_sql_explain._run_explain` line 1400

```sql
EXPLAIN (ANALYZE, BUFFERS, WAL{io_opt}, VERBOSE, FORMAT JSON) {sql}
```

`api_sql_explain._run_explain` line 1404

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) {sql}
```

`api_sql_explain._run_explain` line 1406

```sql
EXPLAIN (FORMAT JSON) {sql}
```

**extra** - `api_sql_explain` line 1479

```sql
ANALYZE write stmt: executed in tx + ROLLBACK (changes discarded).
```

`api_sql_live_queries` line 1651

```sql
SELECT pid,
       usename AS "user",
       application_name AS app,
       state,
       age(now(), coalesce(query_start, state_change))::text AS age,
       query
FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
  {_btype}
  AND query IS NOT NULL
  AND query <> ''
  AND state IN ('active','idle in transaction','idle in transaction (aborted)')
ORDER BY (state = 'active') DESC, query_start NULLS LAST
LIMIT 50
```

`api_sql_stat_queries` line 1686

```sql
SELECT 1 FROM pg_extension WHERE extname='pg_stat_statements'
```

**remedy** - `api_sql_stat_queries` line 1700

```sql
CREATE EXTENSION pg_stat_statements;  -- also add it to shared_preload_libraries and restart
```

`api_sql_stat_queries` line 1720

```sql
SELECT s.queryid::text AS qid,
       r.rolname AS "user",
       d.datname AS db,
       s.calls,
       round(%(total)s::numeric, 1) AS total_ms,
       round(%(mean)s::numeric, 2) AS mean_ms,
       s.rows AS rows_total,
       s.query
FROM
```

**remedy** - `api_sql_stat_queries` line 1752

```sql
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';  -- then restart the cluster
```

`api_sql_hints` line 1919

```sql
SELECT current_schemas(false) AS sp
```

`api_sql_hints` line 1922

```sql
SELECT n.nspname AS schema
FROM pg_namespace n
WHERE n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
  AND n.nspname NOT LIKE 'pg_temp%%'
  AND has_schema_privilege(n.oid, 'USAGE')
ORDER BY
  CASE WHEN n.nspname = ANY(current_schemas(false)) THEN 0 ELSE 1 END,
  n.nspname
```

`api_sql_hints` line 1935

```sql
SELECT n.nspname AS schema,
       c.relname AS name,
       c.relkind AS kind,
       c.oid::bigint AS oid
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','v','m','p','f')
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
  AND n.nspname NOT LIKE 'pg_temp%%'
  AND has_table_privilege(c.oid, 'SELECT')
ORDER BY n.nspname, c.relname
```

`api_sql_hints` line 1954

```sql
SELECT a.attrelid::bigint AS oid,
       a.attname          AS col,
       pg_catalog.format_type(a.atttypid, a.atttypmod) AS typ
FROM pg_attribute a
WHERE a.attrelid = ANY(%s)
  AND a.attnum > 0
  AND NOT a.attisdropped
ORDER BY a.attrelid, a.attnum
```

`api_sql_hints` line 1979

```sql
SELECT DISTINCT n.nspname AS schema, p.proname AS name
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
ORDER BY n.nspname, p.proname
```

`api_sql_index_tryout` line 2055

```sql
SELECT 1 FROM pg_extension WHERE extname = 'hypopg'
```

`api_sql_index_tryout` line 2057

```sql
SELECT 1 FROM pg_available_extensions WHERE name = 'hypopg'
```

`api_sql_index_tryout.plan` line 2072

```sql
EXPLAIN (FORMAT JSON) {inner}
```

`api_sql_index_tryout` line 2086

```sql
SELECT indexname FROM {_hypo_create}(%s)
```

`api_sql_index_tryout` line 2095

```sql
SELECT pg_size_pretty({_hypo_size}(indexrelid)) FROM {_hypo_list}() WHERE indexname = %s
```

`api_sql_index_tryout` line 2102

```sql
SELECT {_hypo_reset}()
```

`(module)` line 2153

```sql
CREATE INDEX
```

`api_sql_running` line 2193

```sql
SELECT pid, state, wait_event_type, wait_event,
       EXTRACT(EPOCH FROM (now() - query_start))::float AS elapsed_s,
       EXTRACT(EPOCH FROM (now() - state_change))::float AS in_state_s,
       query AS _query_full,
       left(query, 300) AS query
  FROM pg_stat_activity
 WHERE application_name = %s AND pid <> pg_backend_pid()
 ORDER BY query_start
 LIMIT 1
```

`api_sql_running` line 2233

```sql
SELECT row_to_json(p)::text AS j FROM {view} p WHERE pid = %s
```

`api_sql_ddl_preflight` line 2343

```sql
SELECT c.oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind IN ('r','p','m','i')
```

`api_sql_ddl_preflight` line 2348

```sql
SELECT c.oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
```

`api_sql_ddl_preflight` line 2353

```sql
SELECT c.oid FROM pg_class c WHERE c.relname = %s AND pg_table_is_visible(c.oid)
```

`api_sql_cancel` line 2442

```sql
SELECT pid, {fn}(pid) AS signalled
  FROM pg_stat_activity
 WHERE application_name = %s AND pid <> pg_backend_pid()
```

`api_sql_cancel` line 2455

```sql
SELECT %s(%s)
```

`_page_count` line 2737

```sql
SET LOCAL statement_timeout = %d
```

`_page_count` line 2742

```sql
ROLLBACK TO SAVEPOINT dbapguin_pc
```

`_page_count` line 2743

```sql
SET LOCAL statement_timeout = 0
```

`_page_count` line 2747

```sql
SET LOCAL statement_timeout = 0
```

`api_sql_page_open` line 2800

```sql
SET LOCAL search_path = {_quote_search_path(sp_parts)}
```

`api_sql_page_open` line 2801

```sql
SET LOCAL statement_timeout = %d
```

`api_sql_page_open` line 2809

```sql
SET LOCAL application_name = %s
```
