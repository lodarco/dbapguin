# Checks (catalogue)

[All parts](README.md) · 13 statements

## db_queries.py

**db-connections** - `(module)` line 254

```sql
SELECT state,
       wait_event_type,
       wait_event,
       count(*) AS connections
FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
  AND backend_type = 'client backend'
GROUP BY state, wait_event_type, wait_event
ORDER BY connections DESC
```

**db-users** - `(module)` line 265

```sql
SELECT usename, application_name, client_addr::text,
       state, count(*) AS connections
FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
  AND backend_type = 'client backend'
GROUP BY usename, application_name, client_addr, state
ORDER BY connections DESC
```

**db-active-queries** - `(module)` line 274

```sql
SELECT pid,
       usename,
       now() - query_start AS duration,
       state,
       -- The column the table shows is cut to keep the row readable.
       -- This is the same statement, whole, for the row's "Copy SQL"
       -- button: a copy of the first 120 characters is a copy of
       -- nothing anybody can run. Leading underscore means the table
       -- renderer does not draw it as a column.
       query AS _query_full,
       left(query, 120) AS query
FROM pg_stat_activity
WHERE state = 'active'
  AND pid <> pg_backend_pid()
ORDER BY duration DESC NULLS LAST
```

**db-long-queries** - `(module)` line 291

```sql
SELECT pid,
       usename,
       now() - query_start AS duration,
       wait_event_type,
       wait_event,
       -- The column the table shows is cut to keep the row readable.
       -- This is the same statement, whole, for the row's "Copy SQL"
       -- button: a copy of the first 120 characters is a copy of
       -- nothing anybody can run. Leading underscore means the table
       -- renderer does not draw it as a column.
       query AS _query_full,
       left(query, 120) AS query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '5 seconds'
  AND pid <> pg_backend_pid()
  -- A parallel query shows one row per worker, each carrying the
  -- leader's query text, so a single slow statement using four
  -- workers looked like five slow statements. Only the leader is a
  -- query the user can act on. (Reasoned from documented behaviour - 
  -- unlike the other fixes in this pass, it was not reproduced here:
  -- every parallel query tried finished before it could be sampled.)
  AND backend_type = 'client backend'
ORDER BY duration DESC
```

**db-table-sizes** - `(module)` line 320

```sql
-- Rolled up to the partition root. pg_stat_user_tables lists every
-- partition separately and the parent with zeros, so a partitioned
-- table used to appear as "0 bytes, 0 rows" next to a wall of its own
-- partitions. Measured here: a 93 MB / 360k-row table reported as
-- 0 bytes, with its three 31 MB partitions listed instead. On a
-- database with hundreds of partitions the tables a DBA actually
-- thinks about were invisible.
--
-- pg_partition_root() is PostgreSQL 12+, comfortably inside the 13+
-- baseline. Plain tables are their own root, so they pass through
-- unchanged.
WITH rolled AS (
    SELECT COALESCE(pg_partition_root(s.relid), s.relid) AS root_oid,
           s.relid,
           s.n_live_tup,
           s.n_dead_tup,
           greatest(s.last_analyze, s.last_autoanalyze) AS analyzed
    FROM pg_stat_user_tables s
)
SELECT n.nspname                                          AS schema,
       c.relname                                          AS table,
       CASE WHEN c.relkind = 'p'
            THEN count(*) FILTER (WHERE r.relid <> r.root_oid)::text
            ELSE '' END                                   AS partitions,
       pg_size_pretty(sum(pg_total_relation_size(r.relid)))  AS total,
       pg_size_pretty(sum(pg_relation_size(r.relid)))        AS table_only,
       pg_size_pretty(sum(pg_indexes_size(r.relid)))         AS indexes,
       sum(r.n_live_tup)                                  AS live_rows,
       sum(r.n_dead_tup)                                  AS dead_rows,
       -- live_rows is the statistics collector's estimate, not a
       -- count. Without knowing when it was last refreshed the number
       -- means little, so its age travels with it.
       CASE WHEN max(r.analyzed) IS NULL THEN 'never analyzed'
            ELSE age(now(), max(r.analyzed))::text END     AS stats_age
FROM rolled r
JOIN pg_class c     ON c.oid = r.root_oid
JOIN pg_namespace n ON n.oid = c.relnamespace
GROUP BY n.nspname, c.relname, c.relkind
ORDER BY sum(pg_total_relation_size(r.relid)) DESC
LIMIT 1000
```

**db-vacuum** - `(module)` line 362

```sql
-- Dead tuples on their own do not answer the question the user has,
-- which is "will autovacuum deal with this or do I have to". That
-- depends on a threshold computed per table, and per-table reloptions
-- override the global settings: measured here, a table with
-- autovacuum_vacuum_scale_factor=0.001 had an effective threshold of
-- 117 where the global settings implied 13,383 - a 114x difference,
-- and the check showed neither number.
--
-- autovacuum_enabled=false matters even more: those tables are never
-- vacuumed at all, and their dead tuples accumulate until someone
-- notices. They are called out rather than left to be inferred.
WITH eff AS (
    SELECT s.schemaname, s.relname, s.relid,
           s.n_live_tup, s.n_dead_tup,
           s.last_vacuum, s.last_autovacuum,
           s.last_analyze, s.last_autoanalyze,
           s.vacuum_count, s.autovacuum_count,
           COALESCE((SELECT option_value FROM pg_options_to_table(c.reloptions)
                      WHERE option_name = 'autovacuum_enabled'), 'true')
               AS av_enabled,
           (COALESCE((SELECT option_value FROM pg_options_to_table(c.reloptions)
                       WHERE option_name = 'autovacuum_vacuum_threshold'),
                     current_setting('autovacuum_vacuum_threshold'))::numeric
            + COALESCE((SELECT option_value FROM pg_options_to_table(c.reloptions)
                         WHERE option_name = 'autovacuum_vacuum_scale_factor'),
                       current_setting('autovacuum_vacuum_scale_factor'))::numeric
              * s.n_live_tup)::bigint AS av_threshold
    FROM pg_stat_user_tables s
    JOIN pg_class c ON c.oid = s.relid
)
SELECT schemaname AS schema,
       relname    AS table,
       n_live_tup AS live,
       n_dead_tup AS dead,
       av_threshold AS autovacuum_at,
       CASE WHEN av_enabled <> 'true' THEN 'DISABLED on this table'
            WHEN n_dead_tup >= av_threshold THEN 'due now'
            WHEN n_live_tup = 0 THEN ''
            ELSE round(100.0 * n_dead_tup / NULLIF(av_threshold, 0), 0)::text || '% of threshold'
       END AS autovacuum,
       last_vacuum::text,
       last_autovacuum::text,
       last_analyze::text,
       last_autoanalyze::text,
       vacuum_count,
       autovacuum_count
FROM eff
ORDER BY (av_enabled <> 'true') DESC,
         CASE WHEN av_threshold > 0 THEN n_dead_tup::numeric / av_threshold ELSE 0 END DESC,
         n_dead_tup DESC NULLS LAST
-- Was LIMIT 25, with no way to see the 26th. The section now pages:
-- twenty-five on screen, "Load more" below, and the rest in the same
-- height once there is more to show. 1000 is a ceiling against a
-- catalogue with tens of thousands of tables, not a page size - the
-- ordering puts every table that needs attention first, so what the
-- ceiling would ever cut is the quietest tail.
LIMIT 1000
```

**db-cache-hit** - `(module)` line 421

```sql
-- Counts index and TOAST blocks, not only heap. A table read entirely
-- through an index whose pages miss cache showed a heap ratio near
-- 100% while the reads it actually performed were mostly from disk.
-- The components are kept as separate columns so a low overall figure
-- can be attributed rather than guessed at.
SELECT schemaname AS schema,
       relname    AS table,
       heap_blks_read                       AS heap_reads,
       heap_blks_hit                        AS heap_hits,
       COALESCE(idx_blks_read, 0)           AS idx_reads,
       COALESCE(idx_blks_hit, 0)            AS idx_hits,
       COALESCE(toast_blks_read, 0) + COALESCE(tidx_blks_read, 0) AS toast_reads,
       heap_blks_read + COALESCE(idx_blks_read, 0)
         + COALESCE(toast_blks_read, 0) + COALESCE(tidx_blks_read, 0) AS total_reads,
       CASE WHEN heap_blks_hit + heap_blks_read
               + COALESCE(idx_blks_hit, 0) + COALESCE(idx_blks_read, 0)
               + COALESCE(toast_blks_hit, 0) + COALESCE(toast_blks_read, 0)
               + COALESCE(tidx_blks_hit, 0) + COALESCE(tidx_blks_read, 0) = 0
            THEN NULL
            ELSE round(100.0 *
                 (heap_blks_hit + COALESCE(idx_blks_hit, 0)
                  + COALESCE(toast_blks_hit, 0) + COALESCE(tidx_blks_hit, 0))
               / (heap_blks_hit + heap_blks_read
                  + COALESCE(idx_blks_hit, 0) + COALESCE(idx_blks_read, 0)
                  + COALESCE(toast_blks_hit, 0) + COALESCE(toast_blks_read, 0)
                  + COALESCE(tidx_blks_hit, 0) + COALESCE(tidx_blks_read, 0)), 2)
       END AS cache_hit_pct
FROM pg_statio_user_tables
WHERE heap_blks_hit + heap_blks_read
    + COALESCE(idx_blks_hit, 0) + COALESCE(idx_blks_read, 0) > 0
ORDER BY heap_blks_read + COALESCE(idx_blks_read, 0)
       + COALESCE(toast_blks_read, 0) + COALESCE(tidx_blks_read, 0) DESC
LIMIT 1000
```

**db-schema-sizes** - `(module)` line 456

```sql
-- Between "Database sizes" and "Table sizes" there was nothing, so
-- "which schema is eating the disk" meant reading a table list and
-- adding up by eye.
--
-- Only THIS database: pg_namespace is per-database, and listing the
-- schemas of every database would need a connection to each one. The
-- description says so rather than quietly showing one database's
-- schemas under a cluster-wide heading.
--
-- Everything in the schema that occupies disk, and each byte once.
--
-- pg_total_relation_size is heap + TOAST + the relation's own indexes,
-- so summing it over index relations as well would count every index
-- twice. Summing it only over tables would instead LOSE the sequences:
-- each one is 8 kB, which is nothing until a schema has five hundred
-- of them. Measured on a live cluster before this was fixed: a schema
-- holding 1152 kB was reported as 752 kB, the 400 kB missing being
-- exactly its fifty sequences.
--
-- So: r (tables), p (partitioned parents, no storage of their own - 
-- their partitions are ordinary relations counted where they live),
-- m (materialised views, which do hold data), S (sequences). Views,
-- foreign tables and composite types hold nothing locally, and TOAST
-- tables live in pg_toast and are already inside pg_table_size of the
-- table they belong to.
WITH rel AS (
    SELECT n.nspname                                       AS schema,
           c.oid, c.relname, c.relkind,
           pg_total_relation_size(c.oid)                   AS total,
           pg_table_size(c.oid)                            AS heap_toast,
           pg_indexes_size(c.oid)                          AS idx
      FROM pg_class c
      JOIN pg_namespace n ON n.oid = c.relnamespace
     WHERE c.relkind IN ('r','p','m','S')
       AND n.nspname NOT IN ('pg_catalog','information_schema')
       AND n.nspname NOT LIKE 'pg_toast%'
       AND n.nspname NOT LIKE 'pg_temp%'
),
-- Driven from pg_namespace, not from pg_class: a schema with nothing
-- in it has no rows there at all, and grouping pg_class would drop it
-- silently. An empty schema is a fact worth seeing - usually one
-- somebody created and forgot, or one a migration never filled.
ns AS (
    SELECT n.oid, n.nspname AS schema, n.nspowner
      FROM pg_namespace n
     WHERE n.nspname NOT IN ('pg_catalog','information_schema')
       AND n.nspname NOT LIKE 'pg_toast%'
       AND n.nspname NOT LIKE 'pg_temp%'
),
counts AS (
    SELECT ns.schema,
           count(c.oid) FILTER (WHERE c.relkind IN ('r','p'))  AS tables,
           count(c.oid) FILTER (WHERE c.relkind = 'v')         AS views,
           count(c.oid) FILTER (WHERE c.relkind = 'm')         AS matviews,
           count(c.oid) FILTER (WHERE c.relkind = 'i')         AS indexes,
           count(c.oid) FILTER (WHERE c.relkind = 'S')         AS sequences
      FROM ns
      LEFT JOIN pg_class c ON c.relnamespace = ns.oid
     GROUP BY ns.schema
),
agg AS (
    SELECT schema,
           sum(total)                                      AS total,
           sum(heap_toast)                                 AS heap_toast,
           sum(idx)                                        AS idx,
           -- Tables and matviews only: "the biggest thing here is a
           -- sequence" is true and useless.
           (SELECT r2.relname FROM rel r2
             WHERE r2.schema = rel.schema AND r2.relkind IN ('r','p','m')
             ORDER BY r2.total DESC LIMIT 1)               AS biggest,
           (SELECT max(r2.total) FROM rel r2
             WHERE r2.schema = rel.schema AND r2.relkind IN ('r','p','m'))
                                                           AS biggest_bytes
      FROM rel GROUP BY schema
)
-- Every schema, including the empty ones: a schema with nothing in it
-- is a fact worth seeing, and an inner join would hide it.
SELECT c.schema                                            AS schema,
       pg_size_pretty(COALESCE(a.total, 0))                AS total,
       pg_size_pretty(COALESCE(a.heap_toast, 0))           AS data_toast_seq,
       pg_size_pretty(COALESCE(a.idx, 0))                  AS indexes_size,
       CASE WHEN pg_database_size(current_database()) > 0
            THEN round(100.0 * COALESCE(a.total, 0)
                       / pg_database_size(current_database()), 1)
       END                                                 AS pct_of_db,
       c.tables, c.views, c.matviews, c.indexes, c.sequences,
       COALESCE(a.biggest, '-')                            AS biggest_table,
       pg_size_pretty(COALESCE(a.biggest_bytes, 0))        AS biggest_size,
       pg_get_userbyid(n.nspowner)                         AS owner
  FROM counts c
  LEFT JOIN agg a ON a.schema = c.schema
  JOIN ns n ON n.schema = c.schema
 ORDER BY COALESCE(a.total, 0) DESC, c.schema
```

**db-sizes** - `(module)` line 551

```sql
SELECT d.datname                                          AS database,
       pg_size_pretty(pg_database_size(d.datname))        AS size,
       pg_database_size(d.datname)                        AS bytes,
       -- Client backends only: autovacuum workers carry a datname
       -- too, and counting them makes an idle database look busy.
       (SELECT count(*) FROM pg_stat_activity sa
        WHERE sa.datname = d.datname
          AND sa.backend_type = 'client backend')         AS connections,
       CASE WHEN s.blks_hit + s.blks_read > 0
            THEN round(100.0 * s.blks_hit /
                       (s.blks_hit + s.blks_read), 2)
       END                                                AS cache_hit_pct,
       s.temp_files                                       AS temp_files,
       pg_size_pretty(s.temp_bytes)                       AS temp_size
FROM pg_database d
JOIN pg_stat_database s ON s.datid = d.oid
WHERE d.datistemplate = false
ORDER BY pg_database_size(d.datname) DESC
```

**db-conn-limits** - `(module)` line 613

```sql
WITH s AS (
    SELECT setting::int AS max_conn FROM pg_settings WHERE name='max_connections'
),
r AS (
    -- The slots an ordinary login cannot take, each shown on its own:
    -- superuser_reserved_connections, and reserved_connections
    -- (PostgreSQL 16+, for roles with pg_use_reserved_connections; NULL
    -- before 16, where the setting does not exist). Counting only the
    -- first read "6 free" on a 16 where an application had 1 left
    -- (2.11.2026).
    SELECT (SELECT setting::int FROM pg_settings WHERE name = 'superuser_reserved_connections') AS reserved,
           (SELECT setting::int FROM pg_settings WHERE name = 'reserved_connections') AS role_reserved
),
c AS (
    -- Only client backends consume a max_connections slot. Since
    -- PostgreSQL 10 pg_stat_activity also lists checkpointer, walwriter,
    -- the autovacuum and logical replication launchers and other
    -- background processes, which have their own limits - counting them
    -- here reported 7 connections on a cluster with 2, and understated
    -- the free slots by the same amount.
    SELECT count(*)::int AS used,
           count(*) FILTER (WHERE state='active')::int            AS active,
           count(*) FILTER (WHERE state='idle')::int              AS idle,
           count(*) FILTER (WHERE state LIKE 'idle in trans%')::int AS idle_in_tx,
           -- Waiting means an active statement held up by something
           -- other than its client: a lock, an LWLock, I/O, IPC. Every
           -- idle session also has a wait event - Client:ClientRead,
           -- it is waiting for its next command - and counting those
           -- read "waiting 7" on a server with 3 lock waits (measured
           -- on 18, 31.10.2026). ClientWrite, an active statement the
           -- client is not reading, is the client's wait too.
           count(*) FILTER (WHERE state = 'active'
                              AND wait_event_type IS NOT NULL
                              AND wait_event_type <> 'Client')::int AS waiting
    FROM pg_stat_activity
    WHERE backend_type = 'client backend'
),
bg AS (
    SELECT count(*)::int AS background
    FROM pg_stat_activity
    WHERE backend_type <> 'client backend'
)
SELECT s.max_conn,
       r.reserved,
       r.role_reserved,
       bg.background,
       c.used,
       c.active,
       c.idle,
       c.idle_in_tx,
       c.waiting,
       -- free for an ordinary login: the reserved slots are not its to take
       greatest(s.max_conn - r.reserved - COALESCE(r.role_reserved, 0) - c.used, 0) AS free,
       round(100.0 * c.used / s.max_conn, 1)              AS used_pct
FROM s, r, c, bg
```

**db-replication** - `(module)` line 670

```sql
-- The lag columns below were all measured from sent_lsn, which only
-- describes the part of the WAL that already left this server. When
-- sending itself stalls - a saturated link, a replica that stopped
-- reading, a paused walsender - sent_lsn falls behind the current WAL
-- position and every one of those numbers stays small while the
-- replica drifts arbitrarily far behind.
--
-- total_lag_bytes measures against pg_current_wal_lsn(), which is the
-- honest answer to "how far behind is this replica". The per-stage
-- numbers are kept because they say WHERE the delay is: sending,
-- writing, flushing or replaying.
--
-- Reasoned, not measured: verifying this needs a second server, and
-- no streaming replica was set up for the audit.
--
-- pid, user and database identify the walsender itself, which is what
-- you need to act on one: the pid is what pg_terminate_backend takes,
-- and the role is what a replication login is granted or revoked on.
-- database is joined in from pg_stat_activity because
-- pg_stat_replication does not carry it (checked on 13, 14 and 18) -
-- and it is empty for a physical standby on purpose, because a
-- physical walsender streams the whole cluster and belongs to no one
-- database. A name there means logical replication.
SELECT r.pid,
       r.usename                                          AS "user",
       a.datname                                          AS database,
       r.client_addr::text                                AS replica,
       r.application_name                                 AS app,
       r.state,
       r.sync_state,
       r.write_lag,
       r.flush_lag,
       r.replay_lag,
       pg_wal_lsn_diff(pg_current_wal_lsn(), r.replay_lsn)  AS total_lag_bytes,
       pg_wal_lsn_diff(pg_current_wal_lsn(), r.sent_lsn)    AS unsent_bytes,
       pg_wal_lsn_diff(r.sent_lsn,    r.write_lsn)          AS write_lag_bytes,
       pg_wal_lsn_diff(r.sent_lsn,    r.flush_lsn)          AS flush_lag_bytes,
       pg_wal_lsn_diff(r.sent_lsn,    r.replay_lsn)         AS replay_lag_bytes
FROM pg_stat_replication r
LEFT JOIN pg_stat_activity a ON a.pid = r.pid
ORDER BY pg_wal_lsn_diff(pg_current_wal_lsn(), r.replay_lsn) DESC NULLS LAST,
         r.application_name
```

**db-settings** - `(module)` line 714

```sql
SELECT name,
       setting,
       unit,
       category,
       short_desc
FROM pg_settings
WHERE name IN (
    'shared_buffers','effective_cache_size','work_mem','maintenance_work_mem',
    'max_connections','max_wal_size','min_wal_size','checkpoint_timeout',
    'checkpoint_completion_target','wal_buffers','random_page_cost',
    'effective_io_concurrency','default_statistics_target',
    'autovacuum','autovacuum_max_workers','autovacuum_naptime',
    'autovacuum_vacuum_scale_factor','autovacuum_analyze_scale_factor',
    'log_min_duration_statement','statement_timeout','idle_in_transaction_session_timeout',
    'max_parallel_workers','max_parallel_workers_per_gather',
    'shared_preload_libraries','rds.force_ssl'
)
ORDER BY category, name
```

**db-index-sizes** - `(module)` line 743

```sql
SELECT n.nspname || '.' || c.relname                      AS "table",
       i.relname                                          AS index_name,
       am.amname || ' (' || COALESCE((
           SELECT string_agg(opc.opcname, ', ' ORDER BY k.ord)
             FROM unnest(string_to_array(x.indclass::text, ' ')::oid[])
                  WITH ORDINALITY AS k(opc_oid, ord)
             JOIN pg_opclass opc ON opc.oid = k.opc_oid), '') || ')' AS method,
       pg_size_pretty(pg_relation_size(x.indrelid))       AS table_size,
       CASE WHEN c.reltoastrelid <> 0
            THEN pg_size_pretty(pg_total_relation_size(c.reltoastrelid)) END AS toast_size,
       pg_size_pretty(pg_relation_size(x.indexrelid))     AS index_size,
       pg_size_pretty(pg_total_relation_size(x.indrelid)) AS total_size,
       pg_relation_size(x.indexrelid)                     AS index_bytes
FROM pg_class c
JOIN pg_index x      ON c.oid = x.indrelid
JOIN pg_class i      ON i.oid = x.indexrelid
JOIN pg_am am        ON am.oid = i.relam
LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind = ANY (ARRAY['r','t','p'])
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname !~ '^pg_toast'
ORDER BY pg_relation_size(x.indexrelid) DESC
LIMIT 1000
```
