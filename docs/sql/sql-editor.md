# SQL editor

[All parts](README.md) · 62 statements

## api/sql.py

`_lock_timeout_source` line 99

```sql
SELECT setting, source, COALESCE(sourcefile, '') FROM pg_settings WHERE name = 'lock_timeout'
```

`_ddl_plan` line 309

```sql
SELECT current_setting('server_version_num')::int AS v
```

`_lock_queue_snapshot` line 345

```sql
SELECT wait_event_type, wait_event,
       EXTRACT(EPOCH FROM (now() - state_change))::float AS waited_s
  FROM pg_stat_activity WHERE pid = %s
```

`_lock_queue_snapshot` line 357

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

`_lock_queue_snapshot` line 377

```sql
SELECT locktype, mode FROM pg_locks WHERE pid = %s AND NOT granted LIMIT 1
```

`_detect_source` line 454

```sql
SELECT n.nspname, c.relname, c.relkind::text   FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace  WHERE c.oid = %s
```

`_detect_source` line 464

```sql
SELECT a.attname FROM pg_index i   JOIN pg_class c ON c.oid = i.indrelid   JOIN pg_namespace n ON n.oid = c.relnamespace   JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum = ANY(i.indkey)  WHERE i.indisprimary AND n.nspname = %s AND c.relname = %s  ORDER BY (SELECT u.ord FROM unnest(i.indkey::smallint[]) WITH ORDINALITY AS u(k, ord) WHERE u.k = a.attnum LIMIT 1)
```

`_detect_source` line 477

```sql
SELECT attnum, attname FROM pg_attribute  WHERE attrelid = %s AND attnum > 0 AND NOT attisdropped
```

`_detect_source` line 497

```sql
SELECT has_table_privilege(%s, 'INSERT'),        has_table_privilege(%s, 'UPDATE'),        has_table_privilege(%s, 'DELETE'),        current_setting('transaction_read_only') = 'on'        AND pg_is_in_recovery()
```

`api_sql_exec` line 687

```sql
SET LOCAL statement_timeout = %s
```

`api_sql_exec` line 689

```sql
SET LOCAL lock_timeout = %s
```

`api_sql_exec` line 698

```sql
SHOW transaction_read_only
```

`api_sql_exec` line 701

```sql
SET LOCAL transaction_read_only = on
```

`api_sql_exec` line 717

```sql
SET application_name = %s
```

`api_sql_exec` line 721

```sql
SET search_path = {quoted}
```

`api_sql_exec` line 966

```sql
COPY ... TO STDOUT and COPY ... FROM STDIN send the data over the client connection, which the editor does not carry. Run the SELECT and use Export CSV, or COPY to or from a file on the server.
```

`_search_path_diagnosis` line 1088

```sql
SHOW search_path
```

`_resolve_missing_relation` line 1117

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

`api_sql_validate` line 1182

```sql
SET search_path = {_quote_search_path(sp_parts)}
```

`api_sql_validate` line 1185

```sql
EXPLAIN (FORMAT TEXT, GENERIC_PLAN) {sql}
```

`api_sql_actual_plans` line 1402

```sql
SELECT current_database() AS d
```

`api_sql_explain` line 1438

```sql
COPY table directly (without SELECT) cannot be EXPLAINed. Use COPY (SELECT ...) TO ...
```

**error** - `api_sql_explain` line 1451

```sql
EXPLAIN does not support this statement (allowed: {', '.join(_EXPLAINABLE_PREFIXES)})
```

`api_sql_explain._apply_sp` line 1480

```sql
SET search_path = {_quote_search_path(sp_parts)}
```

`api_sql_explain._run_explain` line 1511

```sql
EXPLAIN (FORMAT JSON, GENERIC_PLAN) {sql}
```

`api_sql_explain._run_explain` line 1519

```sql
PREPARE {pname} ({ptypes}) AS {sql}
```

`api_sql_explain._run_explain` line 1521

```sql
EXPLAIN (FORMAT JSON) EXECUTE {pname}({nulls})
```

`api_sql_explain._run_explain` line 1532

```sql
EXPLAIN (ANALYZE, BUFFERS, WAL{io_opt}, VERBOSE, FORMAT JSON) {sql}
```

`api_sql_explain._run_explain` line 1536

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) {sql}
```

`api_sql_explain._run_explain` line 1538

```sql
EXPLAIN (FORMAT JSON) {sql}
```

**extra** - `api_sql_explain` line 1611

```sql
ANALYZE write stmt: executed in tx + ROLLBACK (changes discarded).
```

`api_sql_live_queries` line 1790

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

`api_sql_stat_queries` line 1825

```sql
SELECT 1 FROM pg_extension WHERE extname='pg_stat_statements'
```

**remedy** - `api_sql_stat_queries` line 1839

```sql
CREATE EXTENSION pg_stat_statements;  -- also add it to shared_preload_libraries and restart
```

`api_sql_stat_queries` line 1859

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

**remedy** - `api_sql_stat_queries` line 1891

```sql
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';  -- then restart the cluster
```

`api_sql_hints` line 2058

```sql
SELECT current_schemas(false) AS sp
```

`api_sql_hints` line 2061

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

`api_sql_hints` line 2074

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

`api_sql_hints` line 2093

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

`api_sql_hints` line 2118

```sql
SELECT DISTINCT n.nspname AS schema, p.proname AS name
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
ORDER BY n.nspname, p.proname
```

`api_sql_index_tryout` line 2194

```sql
SELECT 1 FROM pg_extension WHERE extname = 'hypopg'
```

`api_sql_index_tryout` line 2196

```sql
SELECT 1 FROM pg_available_extensions WHERE name = 'hypopg'
```

`api_sql_index_tryout.plan` line 2211

```sql
EXPLAIN (FORMAT JSON) {inner}
```

`api_sql_index_tryout` line 2225

```sql
SELECT indexname FROM {_hypo_create}(%s)
```

`api_sql_index_tryout` line 2234

```sql
SELECT pg_size_pretty({_hypo_size}(indexrelid)) FROM {_hypo_list}() WHERE indexname = %s
```

`api_sql_index_tryout` line 2241

```sql
SELECT {_hypo_reset}()
```

`(module)` line 2292

```sql
CREATE INDEX
```

`api_sql_running` line 2332

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

`api_sql_running` line 2378

```sql
SELECT row_to_json(p)::text AS j FROM {view} p WHERE pid = %s
```

`api_sql_ddl_preflight` line 2488

```sql
SELECT c.oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind IN ('r','p','m','i')
```

`api_sql_ddl_preflight` line 2493

```sql
SELECT c.oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
```

`api_sql_ddl_preflight` line 2498

```sql
SELECT c.oid FROM pg_class c WHERE c.relname = %s AND pg_table_is_visible(c.oid)
```

`api_sql_cancel` line 2587

```sql
SELECT pid, {fn}(pid) AS signalled
  FROM pg_stat_activity
 WHERE application_name = %s AND pid <> pg_backend_pid()
```

`api_sql_cancel` line 2600

```sql
SELECT %s(%s)
```

`_page_count` line 2941

```sql
SET LOCAL statement_timeout = %d
```

`_page_count` line 2946

```sql
ROLLBACK TO SAVEPOINT dbapguin_pc
```

`_page_count` line 2947

```sql
SET LOCAL statement_timeout = 0
```

`_page_count` line 2951

```sql
SET LOCAL statement_timeout = 0
```

`api_sql_page_open` line 3050

```sql
SET LOCAL search_path = {_quote_search_path(sp_parts)}
```

`api_sql_page_open` line 3051

```sql
SET LOCAL statement_timeout = %d
```

`api_sql_page_open` line 3059

```sql
SET LOCAL application_name = %s
```
