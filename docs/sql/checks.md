# Checks

[All parts](README.md) · 408 statements

## checks/actual_plans.py

`log_access` line 386

```sql
SELECT current_setting('is_superuser') = 'on'           AS su,
       current_setting('logging_collector', true)       AS collector,
       current_setting('log_destination', true)         AS dest,
       (SELECT setting FROM pg_settings WHERE name = 'log_directory') AS dir,
       has_function_privilege('pg_catalog.pg_ls_logdir()', 'EXECUTE') AS can_ls,
       has_function_privilege('pg_catalog.pg_read_file(text,bigint,bigint,boolean)', 'EXECUTE') AS can_exec_read,
       pg_has_role(current_user, 'pg_monitor', 'MEMBER') AS monitor
```

`log_access` line 402

```sql
SELECT pg_has_role(current_user, 'pg_read_server_files', 'MEMBER') AS m
```

`_log_files` line 429

```sql
SELECT name, size, extract(epoch FROM modification) AS mod FROM pg_ls_logdir() ORDER BY modification DESC
```

`_read_chunk` line 482

```sql
SELECT pg_read_file(%s, %s, %s, true) AS t
```

`_epochs` line 504

```sql
SELECT s, extract(epoch FROM s::timestamptz) AS e FROM unnest(%s::text[]) AS s
```

`_epochs` line 513

```sql
SELECT extract(epoch FROM %s::timestamptz) AS e
```

`store_plans_state` line 729

```sql
SELECT n.nspname AS schema, e.extversion AS version FROM pg_extension e JOIN pg_namespace n ON n.oid = e.extnamespace WHERE e.extname = 'pg_store_plans'
```

`store_plans_state` line 732

```sql
SELECT current_setting('pg_store_plans.plan_format', true) AS f
```

`store_plans_state` line 734

```sql
SELECT 1 AS x FROM pg_available_extensions WHERE name = 'pg_store_plans'
```

`_qids_by_text` line 752

```sql
SELECT DISTINCT queryid, query FROM {name} WHERE dbid = (SELECT oid FROM pg_database WHERE datname = current_database()) AND query IS NOT NULL LIMIT 20000
```

`read_store_plans` line 778

```sql
SELECT p.queryid, p.planid, {plan_expr} AS plan, p.calls, p.total_time, p.rows,
       p.first_call, p.last_call,
       extract(epoch FROM p.first_call) AS first_epoch,
       extract(epoch FROM p.last_call)  AS last_epoch
  FROM {sch}.pg_store_plans p
 WHERE p.queryid::text = ANY(%s)
   AND p.dbid = (SELECT oid FROM pg_database WHERE datname = current_database())
 ORDER BY p.last_call DESC
 LIMIT 20
```

`grant_sql` line 863

```sql
GRANT pg_monitor TO %s;
```

`grant_sql` line 865

```sql
GRANT EXECUTE ON FUNCTION pg_catalog.pg_read_file(text, bigint, bigint, boolean) TO %s;
```

`auto_explain_config` line 898

```sql
SELECT (SELECT setting FROM pg_settings WHERE name = 'shared_preload_libraries')  AS spl,
       (SELECT setting FROM pg_settings WHERE name = 'session_preload_libraries') AS sesspl,
       current_setting('auto_explain.log_min_duration', true) AS min_duration,
       current_setting('auto_explain.log_format', true)    AS log_format,
       current_setting('auto_explain.log_verbose', true)   AS log_verbose,
       current_setting('auto_explain.log_analyze', true)   AS log_analyze,
       current_setting('auto_explain.log_timing', true)    AS log_timing,
       current_setting('auto_explain.sample_rate', true)   AS sample_rate,
       %s AS compute_query_id
```

`collect_plans` line 964

```sql
SELECT current_setting('TimeZone') AS name, EXTRACT(EPOCH FROM (now() - now() AT TIME ZONE 'UTC'))::float8 AS off
```

## checks/analysis_settings.py

`_setting_rows` line 38

```sql
SELECT name, setting, unit, context, vartype, boot_val, short_desc,
       %s
  FROM pg_settings
 WHERE name = ANY(%%s)
```

`_spl_sql` line 58

```sql
ALTER SYSTEM SET shared_preload_libraries = %s;
```

`_has_extension` line 67

```sql
SELECT 1 FROM pg_extension WHERE extname = %s
```

`_ext_available` line 79

```sql
SELECT 1 FROM pg_available_extensions WHERE name = %s
```

`_data_directory` line 96

```sql
SELECT setting AS d FROM pg_settings WHERE name = 'data_directory'
```

`_is_superuser` line 109

```sql
SELECT current_setting('is_superuser') AS issu
```

`collect` line 281

```sql
CREATE EXTENSION
```

`collect` line 285

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

`_plan_rows` line 504

```sql
SELECT current_user AS u
```

`_plan_rows` line 557

```sql
CREATE EXTENSION
```

`_plan_rows` line 559

```sql
CREATE EXTENSION IF NOT EXISTS pg_store_plans;
```

`_pending_libs` line 583

```sql
SELECT setting AS s FROM pg_file_settings WHERE name = 'shared_preload_libraries' ORDER BY seqno DESC LIMIT 1
```

`_log_read_targets` line 614

```sql
SELECT r.rolname AS r FROM pg_roles r
 WHERE r.rolcanlogin AND NOT r.rolsuper AND r.rolname !~ '^pg_' AND r.rolname <> current_user
   AND NOT (pg_has_role(r.oid, 'pg_monitor', 'USAGE')
            AND has_function_privilege(r.oid, 'pg_catalog.pg_read_file(text,bigint,bigint,boolean)', 'EXECUTE'))
 ORDER BY 1 LIMIT 200
```

`_can_grant_log_read` line 637

```sql
SELECT has_function_privilege('pg_catalog.pg_read_file(text,bigint,bigint,boolean)', 'EXECUTE WITH GRANT OPTION')
   AND pg_has_role(current_user, 'pg_monitor', 'USAGE WITH ADMIN OPTION') AS g
```

## checks/ash.py

`_sample_sql` line 88

```sql
SELECT pid,
       usename AS user_name,
       application_name AS app,
       datname AS db,
       coalesce(client_addr::text, 'local') AS client,
       state,
       wait_event_type,
       wait_event,
       EXTRACT(EPOCH FROM (now() - xact_start)) AS xact_age,
       EXTRACT(EPOCH FROM (now() - query_start)) AS query_age,
       {query_id_col},
       coalesce(pg_blocking_pids(pid), '{}'::int[]) AS blockers,
       regexp_replace(query, '[[:space:]]+', ' ', 'g') AS query
  FROM pg_stat_activity
 WHERE pid <> pg_backend_pid()
   AND state IS NOT NULL
   AND state <> 'idle'
```

`_db_timezone` line 816

```sql
SELECT current_setting('TimeZone'), EXTRACT(EPOCH FROM (now() - now() AT TIME ZONE 'UTC'))
```

`_pgss_for` line 992

```sql
SELECT queryid::text AS qid, sum(calls) AS calls,
       sum({tc['total']}) AS total_ms, sum(rows) AS rows,
       sum(shared_blks_read) AS shared_read,
       sum(temp_blks_written) AS temp_written
  FROM {rel}
 WHERE queryid::text = ANY(%s)
 GROUP BY queryid
```

`_query_size_limit` line 1026

```sql
SELECT current_setting('track_activity_query_size') AS v
```

## checks/awr_snapshot.py

`_capture` line 228

```sql
SELECT datname, datname = current_database() AS here,
       xact_commit, xact_rollback, blks_hit, blks_read,
       tup_returned, tup_fetched, tup_inserted, tup_updated, tup_deleted,
       conflicts, temp_files, temp_bytes, deadlocks{extra}
  FROM pg_stat_database WHERE datname IS NOT NULL
```

`_capture` line 253

```sql
SELECT 1 FROM {_pss} LIMIT 1
```

`_capture` line 267

```sql
SELECT queryid::text AS queryid, dbid::bigint AS dbid, userid::bigint AS userid,
       {_toplevel}{', query, calls,\n                       %(total)s AS total_exec_time,\n                       %(mean)s  AS mean_exec_time,\n                       %(max)s   AS max_exec_time,\n                       rows AS rows_total,\n                       shared_blks_hit, shared_blks_read,\n                       temp_blks_read, temp_blks_written\n                  FROM ' % _tc}{_pss}
 WHERE query IS NOT NULL
```

`_capture` line 282

```sql
SELECT oid::bigint AS oid, datname AS name FROM pg_database
```

`_capture` line 284

```sql
SELECT oid::bigint AS oid, rolname AS name FROM pg_roles
```

`_capture` line 309

```sql
SELECT s.relid::bigint AS relid, s.schemaname, s.relname,
       s.seq_scan, s.seq_tup_read, s.idx_scan, s.idx_tup_fetch,
       s.n_tup_ins, s.n_tup_upd, s.n_tup_del, s.n_tup_hot_upd,
       s.n_live_tup, s.n_dead_tup, s.n_mod_since_analyze, {ins_col},
       s.vacuum_count, s.autovacuum_count, s.analyze_count, s.autoanalyze_count,
       extract(epoch FROM s.last_vacuum)::bigint     AS last_vacuum_epoch,
       extract(epoch FROM s.last_autovacuum)::bigint AS last_autovacuum_epoch,
       c.reltuples::float8 AS reltuples,
       (SELECT array_agg(o) FROM unnest(c.reloptions) o
         WHERE o LIKE 'autovacuum%%') AS av_opts
  FROM pg_stat_user_tables s
  JOIN pg_class c ON c.oid = s.relid
```

`_capture` line 327

```sql
SELECT schemaname, relname,
       seq_scan, idx_scan,
       n_tup_ins, n_tup_upd, n_tup_del, n_tup_hot_upd
  FROM pg_stat_user_tables
```

`_capture` line 340

```sql
SELECT indexrelid::bigint AS indexrelid, relid::bigint AS relid,
       schemaname, relname, indexrelname, idx_scan, idx_tup_read
  FROM pg_stat_user_indexes
```

`_capture` line 355

```sql
SELECT name, setting FROM pg_settings
 WHERE name IN ('autovacuum', 'autovacuum_vacuum_threshold', 'autovacuum_vacuum_scale_factor',
                'autovacuum_vacuum_insert_threshold', 'autovacuum_vacuum_insert_scale_factor',
                'autovacuum_vacuum_max_threshold', 'autovacuum_naptime',
                'autovacuum_max_workers', 'autovacuum_worker_slots')
```

`_capture` line 372

```sql
SELECT pg_database_size(current_database()) AS n
```

`_capture` line 382

```sql
WITH {heap_otta_cte()}
SELECT schemaname, tablename, pg_relation_size(reloid) AS bytes,
       {heap_waste_expr()} AS waste
  FROM heap_bloat_est
 WHERE reltuples IS NOT NULL
```

`_capture` line 407

```sql
SELECT num_timed AS ckpt_timed, num_requested AS ckpt_req,
       write_time AS ckpt_write_ms, sync_time AS ckpt_sync_ms,
       buffers_written AS ckpt_bufs
  FROM pg_stat_checkpointer
```

`_capture` line 414

```sql
SELECT buffers_clean, maxwritten_clean, buffers_alloc
  FROM pg_stat_bgwriter
```

`_capture` line 420

```sql
SELECT checkpoints_timed AS ckpt_timed, checkpoints_req AS ckpt_req,
       checkpoint_write_time AS ckpt_write_ms, checkpoint_sync_time AS ckpt_sync_ms,
       buffers_checkpoint AS ckpt_bufs,
       buffers_clean, maxwritten_clean, buffers_alloc, buffers_backend
  FROM pg_stat_bgwriter
```

`_capture` line 438

```sql
SELECT wal_records, wal_fpi, wal_bytes::bigint AS wal_bytes, wal_buffers_full FROM pg_stat_wal
```

`_capture` line 447

```sql
SELECT backend_type,
       coalesce(sum(reads), 0)::bigint      AS reads,
       coalesce(sum(read_time), 0)::float8  AS read_ms,
       coalesce(sum(writes), 0)::bigint     AS writes,
       coalesce(sum(write_time), 0)::float8 AS write_ms,
       coalesce(sum(hits), 0)::bigint       AS hits
  FROM pg_stat_io
 WHERE object = 'relation'
 GROUP BY backend_type
```

`_capture` line 470

```sql
SELECT extract(epoch FROM pg_postmaster_start_time())::bigint AS started,
                              (SELECT stats_reset FROM pg_stat_database WHERE datname = current_database()) AS reset
```

## checks/bloat.py

`run_bloat_indexes_check` line 130

```sql
WITH {be.index_bloat_cte()}
SELECT idx.nspname AS schema, t.relname AS "table", idx."index" AS index,
       idx.indisprimary AS is_pk, idx.indisunique AS is_unique,
       {be.index_is_fk_sql()},
       idx.idx_bytes AS bytes, idx.waste_bytes AS waste_bytes,
       idx.reltuples AS reltuples,
       COALESCE(si.idx_scan, 0) AS idx_scan,
       NULLIF(t.reltuples, -1)::float8 AS tbl_reltuples, t.relpages::bigint AS tbl_relpages,
       (pg_relation_size(t.oid) / current_setting('block_size')::int)::bigint AS tbl_pages,
       COALESCE(st.n_live_tup, 0) AS tbl_live,
       COALESCE(st.n_mod_since_analyze, 0) AS tbl_mod
  FROM idx
  JOIN pg_class t ON t.oid = idx.indrelid
  LEFT JOIN pg_stat_user_indexes si ON si.indexrelid = idx.index_oid
  LEFT JOIN pg_stat_user_tables st ON st.relid = idx.indrelid
 WHERE idx.idx_bytes > 81920
 ORDER BY idx.waste_bytes DESC, idx.idx_bytes DESC
 LIMIT 50
```

## checks/buffer_cache.py

`run_buffer_cache_check` line 10

```sql
SELECT current_setting('server_version_num')::int AS v
```

`run_buffer_cache_check` line 55

```sql
SELECT buffers_used, buffers_unused, buffers_dirty, buffers_pinned,
       round(usagecount_avg::numeric, 2) AS usagecount_avg
FROM {_bc_summary}()
```

`run_buffer_cache_check` line 73

```sql
SELECT
    n.nspname || '.' || c.relname                         AS relation,
    CASE c.relkind
        WHEN 'r' THEN 'table'
        WHEN 'i' THEN 'index'
        WHEN 'S' THEN 'sequence'
        WHEN 't' THEN 'TOAST table'
        WHEN 'v' THEN 'view'
        WHEN 'm' THEN 'matview'
        WHEN 'c' THEN 'composite'
        WHEN 'f' THEN 'foreign'
        WHEN 'p' THEN 'partitioned table'
        WHEN 'I' THEN 'partitioned index'
        ELSE c.relkind::text
    END                                                   AS relation_type,
    count(*)                                              AS buffers,
    pg_size_pretty(count(*)::bigint * current_setting('block_size')::int) AS buffer_size,
    -- The buffers counted above belong to every fork of the
    -- relation; measured against the main fork alone, a small
    -- table read 101.75 %. The FSM and VM forks are added in.
    pg_size_pretty(pg_relation_size(c.oid, 'main')
                 + pg_relation_size(c.oid, 'fsm')
                 + pg_relation_size(c.oid, 'vm'))           AS relation_size,
    -- Fraction of the whole pool. current_setting('shared_buffers')
    -- is a size string ('128MB'); pg_size_bytes turns it into bytes
    -- so this does not depend on the pool being measured in 8 kB
    -- units.
    -- pg_size_bytes() is PostgreSQL 9.6+. shared_buffers is
    -- reported in blocks by pg_settings, so the same figure is
    -- available everywhere without it: setting * block_size.
    round(100.0 * count(*)
          / NULLIF((SELECT setting::numeric FROM pg_settings
                     WHERE name = 'shared_buffers'), 0), 2)  AS pct_of_buffers,
    CASE WHEN pg_relation_size(c.oid, 'main') > 0
         THEN round(count(*)::numeric * current_setting('block_size')::int * 100
                    / (pg_relation_size(c.oid, 'main')
                       + pg_relation_size(c.oid, 'fsm')
                       + pg_relation_size(c.oid, 'vm')), 2)
    END                                                   AS pct_of_relation
FROM {_bc} b
JOIN pg_database d  ON b.reldatabase = d.oid AND d.datname = current_database()
JOIN pg_class c     ON c.relfilenode = b.relfilenode
JOIN pg_namespace n ON n.oid = c.relnamespace
GROUP BY n.nspname, c.relname, c.oid, c.relkind
ORDER BY count(*) DESC
LIMIT 30
```

## checks/cache_advisor.py

`run_cache_advisor_check` line 8

```sql
SELECT current_setting('block_size')::bigint AS bs, current_setting('shared_buffers') AS sb, current_setting('effective_cache_size') AS ecs
```

`run_cache_advisor_check` line 40

```sql
SELECT pg_database_size(current_database()) AS db_size, (SELECT sum(pg_database_size(d.oid))::bigint FROM pg_database d  WHERE NOT d.datistemplate) AS cluster_size
```

`run_cache_advisor_check` line 51

```sql
SELECT
    EXISTS(SELECT 1 FROM pg_extension WHERE extname='pg_prewarm') AS has_prewarm,
    EXISTS(SELECT 1 FROM pg_extension WHERE extname='pg_buffercache') AS has_buffercache,
    EXISTS(SELECT 1 FROM pg_available_extensions WHERE name='pg_prewarm') AS prewarm_avail,
    EXISTS(SELECT 1 FROM pg_available_extensions WHERE name='pg_buffercache') AS buffercache_avail
```

`run_cache_advisor_check` line 60

```sql
SELECT stats_reset FROM pg_stat_database WHERE datname = current_database()
```

`run_cache_advisor_check` line 70

```sql
SELECT
  COALESCE(SUM(heap_blks_hit),0)::bigint  AS heap_hit,
  COALESCE(SUM(heap_blks_read),0)::bigint AS heap_read,
  COALESCE(SUM(idx_blks_hit),0)::bigint   AS idx_hit,
  COALESCE(SUM(idx_blks_read),0)::bigint  AS idx_read,
  COALESCE(SUM(toast_blks_hit),0)::bigint AS toast_hit,
  COALESCE(SUM(toast_blks_read),0)::bigint AS toast_read
FROM pg_statio_user_tables
```

`run_cache_advisor_check` line 199

```sql
SELECT schemaname AS schema, relname AS "table",
       heap_blks_hit AS hits, heap_blks_read AS reads,
       round(100.0 * heap_blks_hit /
             NULLIF(heap_blks_hit + heap_blks_read, 0), 2) AS hit_pct,
       pg_size_pretty(pg_relation_size(relid)) AS size
FROM pg_statio_user_tables
WHERE heap_blks_hit + heap_blks_read >= %s
ORDER BY heap_blks_read DESC
LIMIT 25
```

`run_cache_advisor_check` line 212

```sql
SELECT schemaname AS schema, relname AS "table",
       indexrelname AS index,
       idx_blks_hit AS hits, idx_blks_read AS reads,
       round(100.0 * idx_blks_hit /
             NULLIF(idx_blks_hit + idx_blks_read, 0), 2) AS hit_pct,
       pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_statio_user_indexes
WHERE idx_blks_hit + idx_blks_read >= %s
ORDER BY idx_blks_read DESC
LIMIT 25
```

`run_cache_advisor_check` line 226

```sql
SELECT schemaname AS schema, relname AS "table",
       toast_blks_hit AS hits, toast_blks_read AS reads,
       round(100.0 * toast_blks_hit /
             NULLIF(toast_blks_hit + toast_blks_read, 0), 2) AS hit_pct,
       pg_size_pretty(pg_relation_size(relid)) AS size
FROM pg_statio_user_tables
WHERE toast_blks_hit + toast_blks_read >= %s
ORDER BY toast_blks_read DESC
LIMIT 25
```

`run_cache_advisor_check` line 281

```sql
REINDEX CONCURRENTLY may compact bloat (does not block writes; takes 2x disk).
```

`run_cache_advisor_check` line 347

```sql
WITH bufs AS (
    SELECT b.reldatabase, b.relfilenode, count(*) AS bufs
      FROM {_bc} b
     WHERE b.reldatabase IS NOT NULL
     GROUP BY b.reldatabase, b.relfilenode
)
SELECT n.nspname AS schema,
       c.relname AS relname,
       CASE c.relkind
          WHEN 'r' THEN 'table'
          WHEN 'i' THEN 'index'
          WHEN 'm' THEN 'matview'
          WHEN 't' THEN 'toast'
          ELSE c.relkind::text END AS kind,
       sum(b.bufs) AS bufs,
       pg_size_pretty(sum(b.bufs)::bigint * current_setting('block_size')::bigint) AS in_cache_size,
       pg_size_pretty(pg_relation_size(c.oid)) AS object_size
  FROM bufs b
  JOIN pg_class c ON c.relfilenode = b.relfilenode
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname NOT IN ('pg_catalog','information_schema')
 GROUP BY n.nspname, c.relname, c.relkind, c.oid
 ORDER BY sum(b.bufs) DESC
 LIMIT 30
```

**action** - `run_cache_advisor_check` line 392

```sql
CREATE EXTENSION pg_buffercache;  -- low overhead, read-only superuser view
```

## checks/checkpoints.py

**sql** - `run_checkpoints_check` line 13

```sql
SELECT num_timed                                              AS checkpoints_timed,
       num_requested                                          AS checkpoints_req,
       restartpoints_timed,
       restartpoints_req,
       restartpoints_done,
       write_time                                             AS checkpoint_write_ms,
       sync_time                                              AS checkpoint_sync_ms,
       buffers_written                                        AS buffers_checkpoint,
       stats_reset,
       EXTRACT(EPOCH FROM (now() - stats_reset))::bigint      AS seconds_since_reset,
       CASE WHEN num_timed + num_requested > 0
            THEN round(EXTRACT(EPOCH FROM (now() - stats_reset))::numeric
                       / (num_timed + num_requested) / 60, 2)
       END                                                    AS minutes_between_checkpoints
FROM pg_stat_checkpointer
```

**sql** - `run_checkpoints_check` line 31

```sql
SELECT checkpoints_timed,
       checkpoints_req,
       checkpoint_write_time                                  AS checkpoint_write_ms,
       checkpoint_sync_time                                   AS checkpoint_sync_ms,
       buffers_checkpoint,
       buffers_clean,
       buffers_backend,
       maxwritten_clean,
       stats_reset,
       EXTRACT(EPOCH FROM (now() - stats_reset))::bigint      AS seconds_since_reset,
       CASE WHEN checkpoints_timed + checkpoints_req > 0
            THEN round(EXTRACT(EPOCH FROM (now() - stats_reset))::numeric
                       / (checkpoints_timed + checkpoints_req) / 60, 2)
       END                                                    AS minutes_between_checkpoints
FROM pg_stat_bgwriter
```

`run_checkpoints_check` line 79

```sql
SELECT setting::bigint *
       CASE unit WHEN 'kB' THEN 1024
                 WHEN 'MB' THEN 1048576
                 WHEN 'GB' THEN 1073741824
                 ELSE 1 END AS bytes,
       pg_size_pretty(
           setting::bigint *
           CASE unit WHEN 'kB' THEN 1024
                     WHEN 'MB' THEN 1048576
                     WHEN 'GB' THEN 1073741824
                     ELSE 1 END) AS pretty
  FROM pg_settings WHERE name='max_wal_size'
```

`run_checkpoints_check` line 140

```sql
SELECT wal_bytes, wal_fpi_bytes, wal_fpi, wal_records, stats_reset FROM pg_stat_wal
```

## checks/duplicate_indexes.py

**_SQL** - `(module)` line 25

```sql
SELECT i.indexrelid AS oid, i.indrelid AS table_oid,
       n.nspname AS schema, t.relname AS "table", c.relname AS index,
       pg_relation_size(i.indexrelid) AS bytes,
       pg_size_pretty(pg_relation_size(i.indexrelid)) AS size,
       COALESCE(s.idx_scan, 0) AS scans,
       i.indisprimary AS is_pk, i.indisunique AS is_unique, i.indisreplident AS is_replident,
       (SELECT con.contype FROM pg_constraint con
         WHERE con.conindid = i.indexrelid AND con.contype IN ('p', 'u', 'x') LIMIT 1) AS contype,
       EXISTS (SELECT 1 FROM pg_constraint f
                WHERE f.contype = 'f' AND f.conrelid = i.indrelid
                  AND i.indkey[0] = ANY(f.conkey)) AS is_fk,
       i.indkey::int2[] AS keys, i.indclass::oid[] AS classes,
       i.indexprs IS NOT NULL AS has_exprs, i.indpred IS NOT NULL AS is_partial,
       i.indrelid::text || E'\n' || i.indclass::text || E'\n' || i.indkey::text || E'\n'
         || coalesce(i.indexprs::text, '') || E'\n' || coalesce(i.indpred::text, '') AS twin_key,
       pg_get_indexdef(i.indexrelid) AS definition
  FROM pg_index i
  JOIN pg_class c ON c.oid = i.indexrelid
  JOIN pg_class t ON t.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = t.relnamespace
  LEFT JOIN pg_stat_user_indexes s ON s.indexrelid = i.indexrelid
 WHERE n.nspname NOT IN ('pg_catalog', 'information_schema') AND n.nspname NOT LIKE 'pg_toast%%'
   AND i.indisvalid
```

## checks/durability.py

`read_settings` line 89

```sql
SELECT name, setting FROM pg_settings WHERE name = ANY(%s)
```

`sync_standby_state` line 106

```sql
SELECT pg_is_in_recovery() AS r
```

`sync_standby_state` line 110

```sql
SELECT application_name, state, sync_state FROM pg_stat_replication
```

`sync_standby_state` line 116

```sql
SELECT count(*) AS n,
       extract(epoch FROM max(now() - coalesce(query_start, state_change)))::float AS longest
  FROM pg_stat_activity WHERE wait_event = 'SyncRep'
```

`assess` line 139

```sql
ALTER SYSTEM SET fsync = on; SELECT pg_reload_conf(); then flush what was written while it was off (sync, or initdb --sync-only -D <data directory>). For faster commits without that risk use synchronous_commit = off.
```

`assess` line 145

```sql
ALTER SYSTEM SET full_page_writes = on; SELECT pg_reload_conf();
```

`assess` line 174

```sql
GRANT pg_read_all_stats TO <this role>;
```

## checks/errors.py

`_session_reads_server_files` line 36

```sql
SELECT current_setting('is_superuser') = 'on' AS su
```

`_session_reads_server_files` line 43

```sql
SELECT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'pg_read_server_files') AS has_role
```

`_session_reads_server_files` line 45

```sql
SELECT pg_has_role(current_user, 'pg_read_server_files', 'MEMBER') AS m
```

`run_errors_check.snapshot` line 74

```sql
SELECT datname,
       xact_commit, xact_rollback,
       deadlocks, conflicts,
       blks_read, temp_files{extra_cols}
FROM pg_stat_database
WHERE datname = current_database()
```

`run_errors_check` line 230

```sql
SELECT pg_current_logfile(%s) AS f
```

`run_errors_check` line 241

```sql
SELECT pg_current_logfile() AS f
```

`run_errors_check` line 266

```sql
SELECT current_setting('log_line_prefix') AS p
```

`run_errors_check` line 278

```sql
SELECT (pg_stat_file(%s)).size AS sz
```

`run_errors_check` line 281

```sql
SELECT pg_read_file(%s, %s, %s) AS content
```

**next_step** - `run_errors_check` line 297

```sql
GRANT pg_read_server_files TO <role>; or run as superuser
```

## checks/ext_gate.py

`extension_schema` line 59

```sql
SELECT n.nspname AS nsp
  FROM pg_extension e
  JOIN pg_namespace n ON n.oid = e.extnamespace
 WHERE e.extname = %s
```

`ext_relation` line 84

```sql
SELECT n.nspname AS nsp
  FROM pg_class c
  JOIN pg_namespace n  ON n.oid = c.relnamespace
  JOIN pg_depend d     ON d.objid = c.oid
                      AND d.classid = 'pg_class'::regclass
                      AND d.deptype = 'e'
  JOIN pg_extension e  ON e.oid = d.refobjid
 WHERE e.extname = %s AND c.relname = %s
   AND (NOT %s OR has_table_privilege(c.oid, 'SELECT'))
 LIMIT 1
```

`ext_function` line 109

```sql
SELECT n.nspname AS nsp
  FROM pg_proc p
  JOIN pg_namespace n  ON n.oid = p.pronamespace
  JOIN pg_depend d     ON d.objid = p.oid
                      AND d.classid = 'pg_proc'::regclass
                      AND d.deptype = 'e'
  JOIN pg_extension e  ON e.oid = d.refobjid
 WHERE e.extname = %s AND p.proname = %s
   AND (NOT %s OR has_function_privilege(p.oid, 'EXECUTE'))
 LIMIT 1
```

`require_extension` line 171

```sql
SELECT 1 FROM pg_extension WHERE extname = %s
```

`require_extension` line 183

```sql
SELECT default_version, comment FROM pg_available_extensions WHERE name = %s
```

`require_extension` line 209

```sql
CREATE EXTENSION IF NOT EXISTS %s SCHEMA %s
```

## checks/health.py

`_wrap_hint` line 46

```sql
VACUUM (FREEZE) the oldest tables in the oldest database - Vacuum status lists them with their xid age
```

`run_health_check` line 209

```sql
SELECT version() AS v
```

`run_health_check` line 220

```sql
SELECT to_regprocedure('aurora_version()') IS NOT NULL AS aurora
```

`run_health_check` line 222

```sql
SELECT aurora_version() AS v
```

`run_health_check` line 230

```sql
SELECT 1 FROM pg_roles WHERE rolname='rds_superuser'
```

`run_health_check` line 258

```sql
SELECT name, setting, unit
FROM pg_settings
WHERE name IN (
  'max_connections','shared_buffers','effective_cache_size',
  'work_mem','maintenance_work_mem',
  'idle_in_transaction_session_timeout',
  'statement_timeout','lock_timeout',
  'autovacuum_vacuum_scale_factor','autovacuum_analyze_scale_factor',
  'autovacuum_naptime','autovacuum_max_workers',
  'autovacuum_vacuum_cost_limit',
  'hot_standby_feedback','max_standby_streaming_delay',
  'max_wal_size','wal_compression'
)
```

`run_health_check` line 299

```sql
SELECT current_setting('max_connections')::int AS max_c,
       (SELECT count(*) FROM pg_stat_activity
          WHERE pid <> pg_backend_pid() {_client}) AS used,
       (SELECT count(*) FROM pg_stat_activity
          WHERE pid <> pg_backend_pid() {_client}
            AND state = 'active') AS active_n
```

`run_health_check` line 319

```sql
SELECT count(*) AS n,
       extract(epoch FROM max(now() - state_change)) AS max_secs
FROM pg_stat_activity
WHERE state = 'idle in transaction'
```

`run_health_check` line 340

```sql
SELECT count(*) AS n,
       extract(epoch FROM max(now() - xact_start)) AS max_secs
FROM pg_stat_activity
WHERE state = 'active'
  AND xact_start < now() - interval '5 minutes'
```

`run_health_check` line 367

```sql
SELECT count(DISTINCT w.pid) AS n
  FROM pg_locks w
  JOIN pg_locks h
    ON w.locktype = h.locktype
   AND w.database IS NOT DISTINCT FROM h.database
   AND w.relation IS NOT DISTINCT FROM h.relation
   AND w.page IS NOT DISTINCT FROM h.page
   AND w.tuple IS NOT DISTINCT FROM h.tuple
   AND w.virtualxid IS NOT DISTINCT FROM h.virtualxid
   AND w.transactionid IS NOT DISTINCT FROM h.transactionid
   AND w.classid IS NOT DISTINCT FROM h.classid
   AND w.objid IS NOT DISTINCT FROM h.objid
   AND w.objsubid IS NOT DISTINCT FROM h.objsubid
   AND w.pid <> h.pid
 WHERE NOT w.granted AND h.granted AND w.pid <> pg_backend_pid()
```

`run_health_check` line 385

```sql
SELECT count(DISTINCT waiter.pid) AS n
FROM pg_stat_activity waiter
WHERE pg_blocking_pids(waiter.pid) <> '{}'
```

`run_health_check` line 405

```sql
SELECT sum(blks_hit + blks_read) AS total,
       round(100.0 * sum(blks_hit)::numeric /
             nullif(sum(blks_hit + blks_read), 0), 2) AS hit
FROM pg_stat_database
```

`run_health_check` line 430

```sql
SELECT application_name,
       pg_xlog_location_diff(pg_current_xlog_location(),
                             replay_location)::float AS lag_b
FROM pg_stat_replication
ORDER BY lag_b DESC NULLS LAST
```

`run_health_check` line 449

```sql
SELECT application_name,
       extract(epoch FROM coalesce(replay_lag, '0'::interval)) AS lag_s
FROM pg_stat_replication
ORDER BY lag_s DESC NULLS LAST
```

`run_health_check` line 517

```sql
SELECT schemaname, relname,
       n_live_tup, n_dead_tup,
       round(100.0 * n_dead_tup::numeric /
             nullif(n_live_tup + n_dead_tup, 0), 1) AS dead_pct
FROM pg_stat_user_tables
WHERE n_live_tup + n_dead_tup > 1000
ORDER BY dead_pct DESC NULLS LAST
LIMIT 5
```

`run_health_check` line 535

```sql
VACUUM LAG (dead tuples)
```

`run_health_check` line 535

```sql
VACUUM LAG (dead tuples)
```

`run_health_check` line 548

```sql
VACUUM LAG (dead tuples)
```

`run_health_check` line 548

```sql
VACUUM LAG (dead tuples)
```

`run_health_check` line 556

```sql
VACUUM LAG (dead tuples)
```

`run_health_check` line 556

```sql
VACUUM LAG (dead tuples)
```

`run_health_check` line 609

```sql
VACUUM frees heap/TOAST for reuse; REPACK (CONCURRENTLY) returns it to the OS without blocking the table; REINDEX for an index
```

`run_health_check` line 611

```sql
VACUUM frees heap/TOAST for reuse; pg_repack or VACUUM FULL return it to the OS; REINDEX for an index
```

`run_health_check` line 622

```sql
SELECT schemaname, relname, n_dead_tup
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000
  AND n_dead_tup::numeric / nullif(n_live_tup + n_dead_tup, 0) > 0.10
  AND (last_autovacuum IS NULL
       OR last_autovacuum < now() - interval '1 day')
  AND (last_vacuum IS NULL
       OR last_vacuum < now() - interval '1 day')
ORDER BY n_dead_tup DESC
```

`run_health_check` line 657

```sql
SELECT max(age(datfrozenxid))::bigint AS xid_age,
       current_setting('autovacuum_freeze_max_age')::bigint AS xid_max,
       {mx_sel}
  FROM pg_database
```

`run_health_check` line 669

```sql
SELECT members_size FROM pg_get_multixact_stats()
```

`run_health_check` line 751

```sql
SELECT s.schemaname, s.relname, pg_relation_size(s.indexrelid) AS bytes
FROM pg_stat_user_indexes s
JOIN pg_index i ON i.indexrelid = s.indexrelid
WHERE s.idx_scan = 0
  AND NOT i.indisunique
  AND NOT i.indisprimary
  -- Same exclusions as the drilldown below; if the two drift,
  -- the summary promises rows the detail cannot show.
  AND NOT EXISTS (SELECT 1 FROM pg_constraint c
                   WHERE c.conindid = s.indexrelid)
  AND NOT EXISTS (SELECT 1 FROM pg_inherits inh
                   WHERE inh.inhrelid = s.indexrelid)
  AND NOT i.indisreplident
  AND i.indisvalid
  AND i.indisready
  AND pg_relation_size(s.indexrelid) > 1048576
  AND s.last_idx_scan IS NULL
ORDER BY 3 DESC
```

`run_health_check` line 772

```sql
SELECT s.schemaname, s.relname, pg_relation_size(s.indexrelid) AS bytes
FROM pg_stat_user_indexes s
JOIN pg_index i ON i.indexrelid = s.indexrelid
WHERE s.idx_scan = 0
  AND NOT i.indisunique
  AND NOT i.indisprimary
  AND pg_relation_size(s.indexrelid) > 1048576
ORDER BY 3 DESC
```

`run_health_check` line 899

```sql
SELECT extname FROM pg_extension
```

`run_health_check` line 905

```sql
SELECT name FROM pg_available_extensions
```

`run_health_check` line 914

```sql
SELECT setting AS s FROM pg_settings WHERE name = 'shared_preload_libraries'
```

`run_health_check` line 998

```sql
SELECT coalesce(usename,'?') AS user,
       coalesce(application_name,'?') AS app,
       coalesce(client_addr::text,'local') AS client,
       count(*) AS n
FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
GROUP BY usename, application_name, client_addr
ORDER BY n DESC LIMIT 8
```

`run_health_check` line 1074

```sql
SELECT pid, usename AS user, application_name AS app,
       age(now(), xact_start) AS duration,
```

`run_health_check` line 1171

```sql
SELECT datname,
       round(100.0 * blks_hit::numeric /
             nullif(blks_hit + blks_read, 0), 2) AS hit_pct,
       pg_size_pretty(blks_read * 8192::bigint) AS proc_disk
FROM pg_stat_database
WHERE blks_hit + blks_read > 0
ORDER BY hit_pct ASC LIMIT 8
```

`run_health_check` line 1201

```sql
SELECT application_name, client_addr::text AS client, state,
       pg_xlog_location_diff(pg_current_xlog_location(), sent_location)   AS sent_lag_b,
       pg_xlog_location_diff(pg_current_xlog_location(), write_location)  AS write_lag_b,
       pg_xlog_location_diff(pg_current_xlog_location(), flush_location)  AS flush_lag_b,
       pg_xlog_location_diff(pg_current_xlog_location(), replay_location) AS replay_lag_b,
       '-' AS write_lag, '-' AS flush_lag, '-' AS replay_lag
  FROM pg_stat_replication
 ORDER BY replay_location
```

`run_health_check` line 1212

```sql
SELECT application_name, client_addr::text AS client, state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), sent_lsn)   AS sent_lag_b,
       pg_wal_lsn_diff(pg_current_wal_lsn(), write_lsn)  AS write_lag_b,
       pg_wal_lsn_diff(pg_current_wal_lsn(), flush_lsn)  AS flush_lag_b,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS replay_lag_b,
       coalesce(write_lag::text,'-')   AS write_lag,
       coalesce(flush_lag::text,'-')   AS flush_lag,
       coalesce(replay_lag::text,'-')  AS replay_lag
FROM pg_stat_replication
ORDER BY replay_lsn
```

`run_health_check` line 1240

```sql
VACUUM LAG (dead tuples)
```

`run_health_check` line 1242

```sql
SELECT schemaname AS schema, relname AS table_name,
       n_live_tup AS live, n_dead_tup AS dead,
       round(100.0 * n_dead_tup::numeric /
             nullif(n_live_tup + n_dead_tup, 0), 1) AS dead_pct,
       pg_size_pretty(pg_total_relation_size(relid)) AS size,
       coalesce(last_autovacuum::text, 'never') AS last_av
FROM pg_stat_user_tables
WHERE n_live_tup + n_dead_tup > 1000
ORDER BY dead_pct DESC NULLS LAST LIMIT 10
```

`run_health_check` line 1304

```sql
SELECT s.schemaname AS schema, s.relname AS "table",
       s.n_live_tup AS live,
       s.n_dead_tup AS dead,
       coalesce(last_autovacuum::text, 'never') AS last_av,
       coalesce(last_vacuum::text, 'never') AS last_manual,
       (SELECT array_agg(opt) FROM unnest(
            coalesce((SELECT reloptions FROM pg_class
                      /* format('%I.%I', ...) quotes identifiers only when
                         needed - case-sensitive names like "MyTable" survive
                         the regclass cast. */
                      WHERE oid = format('%I.%I', s.schemaname, s.relname)::regclass),
                     ARRAY[]::text[])
        ) AS opt WHERE opt LIKE 'autovacuum%' OR opt LIKE 'toast.autovacuum%'
       ) AS per_table_opts
FROM pg_stat_user_tables s
WHERE s.n_dead_tup > 10000
  AND (s.last_autovacuum IS NULL
       OR s.last_autovacuum < now() - interval '1 day')
ORDER BY s.n_dead_tup DESC LIMIT 10
```

`run_health_check` line 1401

```sql
ALTER TABLE x ALTER COLUMN id TYPE bigint;
```

`run_health_check` line 1402

```sql
ALTER SEQUENCE seq AS bigint MAXVALUE 9223372036854775807;
```

`run_health_check` line 1433

```sql
SELECT s.schemaname AS schema, s.relname AS "table",
       s.indexrelname AS index,
       pg_size_pretty(pg_relation_size(s.indexrelid)) AS size,
       i.indisunique  AS unique_idx,
       i.indisprimary AS pk_idx,
       EXISTS (
           SELECT 1 FROM pg_constraint c
           WHERE c.contype = 'f'
             AND c.conrelid = i.indrelid
             AND i.indkey[0] = ANY(c.conkey)
       ) AS pokriva_fk,
       i.indpred IS NOT NULL AS is_partial,
       {last_scan_col} AS last_scan,
       format('DROP INDEX CONCURRENTLY IF EXISTS %I.%I;',
              s.schemaname, s.indexrelname) AS drop_sql
FROM pg_stat_user_indexes s
JOIN pg_index i ON i.indexrelid = s.indexrelid
WHERE s.idx_scan = 0
  AND NOT i.indisunique
  AND NOT i.indisprimary
  -- Everything below is an index PostgreSQL refuses to
  -- drop, or one whose loss breaks something silently.
  -- Offering a statement that errors out is worse than
  -- offering nothing: it teaches the user to ignore the
  -- advisor. All three were reproduced on a live server.
  --
  -- EXCLUDE constraints: the index is not unique and not
  -- primary, so the two filters above let it through, and
  -- DROP INDEX answers "cannot drop ... because constraint
  -- ... requires it".
  AND NOT EXISTS (
      SELECT 1 FROM pg_constraint c
      WHERE c.conindid = s.indexrelid
  )
  -- A partition's index attached to a partitioned parent:
  -- "cannot drop index ... because index ... requires it".
  -- The parent has to be dropped instead.
  AND NOT EXISTS (
      SELECT 1 FROM pg_inherits inh
      WHERE inh.inhrelid = s.indexrelid
  )
  -- REPLICA IDENTITY USING INDEX: dropping it does not
  -- error, it silently breaks logical replication of
  -- UPDATE and DELETE for that table.
  AND NOT i.indisreplident
  -- Still being built, or built and never validated.
  AND i.indisvalid
  AND i.indisready
  AND pg_relation_size(s.indexrelid) > 1048576
ORDER BY pg_relation_size(s.indexrelid) DESC LIMIT 30
```

`run_health_check` line 1512

```sql
DROP statements (CONCURRENTLY does not block the table):
```

`run_health_check` line 1548

```sql
SELECT application_name, client_addr::text AS client, state, sync_state, sync_priority FROM pg_stat_replication ORDER BY sync_priority, application_name
```

## checks/index_advisor.py

`_create_index_sql` line 28

```sql
CREATE INDEX CONCURRENTLY IF NOT EXISTS {_qi(name)} ON {_qq(schema, table)}{using} ({body});
```

`run_index_advisor_check` line 87

```sql
WITH fk_keys AS (
    SELECT con.oid                                AS con_oid,
           con.conrelid                           AS rel_oid,
           n.nspname                              AS schema,
           c.relname                              AS "table",
           con.conkey                             AS cols,
           (
               SELECT array_agg(a.attname ORDER BY u.ord)
                 FROM unnest(con.conkey) WITH ORDINALITY u(attnum, ord)
                 JOIN pg_attribute a
                   ON a.attrelid = con.conrelid AND a.attnum = u.attnum
           )                                      AS col_names
      FROM pg_constraint con
      JOIN pg_class c     ON c.oid = con.conrelid
      JOIN pg_namespace n ON n.oid = c.relnamespace
     WHERE con.contype = 'f'
       AND n.nspname NOT IN ('pg_catalog','information_schema')
),
fk_covered AS (
    -- An FK is "covered" when SOME index's leading-column
    -- prefix equals the FK's column order. Partial indexes
    -- (WHERE clause) DO NOT count: they don't help FK
    -- validation across the whole table.
    -- indkey is an int2vector, and casting it straight to int[]
    -- keeps its 0-based bounds, so [1:n] of it dropped the leading
    -- column and no index ever covered an FK. string_to_array gives
    -- a 1-based array, as the duplicate-index query below does.
    SELECT DISTINCT fk.con_oid
      FROM fk_keys fk
      JOIN pg_index i ON i.indrelid = fk.rel_oid
                     AND i.indpred IS NULL
                     AND array_length(i.indkey::int[], 1) >= array_length(fk.cols::int[], 1)
                     AND (string_to_array(i.indkey::text, ' ')::int[])[1:array_length(fk.cols::int[], 1)]
                         = fk.cols::int[]
),
fk_candidates AS (
    SELECT fk.schema                                            AS shema,
           fk."table"                                           AS tablica,
           array_to_string(fk.col_names, ', ')                  AS kolona,
           'FK without leading index ('
             || array_length(fk.cols::int[], 1) || ' col'
             || CASE WHEN array_length(fk.cols::int[], 1) > 1 THEN 's' ELSE '' END
             || ')'                                             AS razlog,
           -- The statement itself is written in Python, which can
           -- quote the index name and fit it into 63 bytes.
           fk.col_names::text[]                                 AS idx_cols
      FROM fk_keys fk
     WHERE fk.con_oid NOT IN (SELECT con_oid FROM fk_covered)
),
-- FIX #3: For tables with no indexes at all, prefer a PK
-- column shape rather than a random column. Heuristic:
-- pick the column literally named "id" if it exists, else
-- the first int4/int8 column, else the first non-text
-- column. Emit a single recommend per table.
no_index_tables AS (
    SELECT DISTINCT ON (t.oid)
        n.nspname                                              AS shema,
        t.relname                                              AS tablica,
        a.attname                                              AS kolona,
        -- ::text, not a bare literal: fk_candidates' razlog is a
        -- concatenation (already text), this one is a plain string
        -- and PostgreSQL leaves that "unknown" until forced - which
        -- the UNION ALL below needs resolved before it runs, not
        -- deferred. Harmless everywhere; on PostgreSQL 9.4 specifically
        -- the deferred resolution never happens and the UNION fails
        -- with "failed to find conversion function from unknown to
        -- text" on a cluster with no unindexed tables to seed a type
        -- from (see tools/check_pg_version_matrix.py, db-index-advisor).
        'Table has no indexes at all - first candidate'::text  AS razlog,
        ARRAY[a.attname::text]                                  AS idx_cols
    FROM pg_class t
    JOIN pg_namespace n  ON n.oid = t.relnamespace
    JOIN pg_attribute a  ON a.attrelid = t.oid
                       AND a.attnum > 0
                       AND NOT a.attisdropped
    JOIN pg_type ty      ON ty.oid = a.atttypid
    WHERE t.relkind = 'r'
      AND n.nspname NOT IN ('pg_catalog','information_schema')
      AND ty.typname IN ('int2','int4','int8','uuid','varchar','text','timestamp','timestamptz','bool')
      AND t.relpages > 100  -- ≥ ~800KB; skip tiny tables where seqscan is cheaper than index
      AND NOT EXISTS (
          SELECT 1 FROM pg_index i WHERE i.indrelid = t.oid
      )
    ORDER BY t.oid,
             -- prefer "id" name, then int/uuid, then anything else
             CASE WHEN lower(a.attname) = 'id'                 THEN 0
                  WHEN lower(a.attname) LIKE '%%\_id'  ESCAPE '\' THEN 1
                  WHEN ty.typname IN ('int2','int4','int8','uuid') THEN 2
                  ELSE 3
             END,
             a.attnum
)
SELECT * FROM fk_candidates
UNION ALL
SELECT * FROM no_index_tables
ORDER BY shema, tablica, kolona
```

`run_index_advisor_check` line 206

```sql
SELECT 1 FROM {_pss} LIMIT 1
```

`run_index_advisor_check` line 216

```sql
-- COALESCE on idx_scan, and it is the difference between this
-- advisor working and not. pg_stat_user_tables.idx_scan is NULL,
-- not 0, for a table that has no indexes on it - so every
-- comparison below evaluated to NULL, the WHERE clause dropped
-- the row, and the tables with no index at all were the only
-- ones this query could never suggest an index for. Measured:
-- 2000 rows, sixty sequential scans, no index - zero candidates.
SELECT t.schemaname AS schema, t.relname AS table,
       t.seq_scan, COALESCE(t.idx_scan, 0) AS idx_scan,
       t.n_live_tup AS rows,
       CASE WHEN t.seq_scan + COALESCE(t.idx_scan, 0) = 0 THEN 0
            ELSE round(100.0 * t.seq_scan
                       / (t.seq_scan + COALESCE(t.idx_scan, 0)), 1)
       END AS seq_pct
FROM pg_stat_user_tables t
WHERE t.n_live_tup > 1000
  AND t.seq_scan > 50
  AND (t.seq_scan + COALESCE(t.idx_scan, 0)) > 0
  AND (100.0 * t.seq_scan
       / (t.seq_scan + COALESCE(t.idx_scan, 0))) > 30
ORDER BY t.seq_scan DESC
LIMIT 50
```

`run_index_advisor_check` line 249

```sql
SELECT s.queryid::text AS qid, s.calls,
       round(%(total)s::numeric, 1) AS total_ms,
       lower(s.query) AS qtext
FROM
```

`run_index_advisor_check` line 262

```sql
SELECT n.nspname AS schema, t.relname AS table, a.attname AS col
FROM pg_class t
JOIN pg_namespace n  ON n.oid = t.relnamespace
JOIN pg_index    i   ON i.indrelid = t.oid
JOIN pg_attribute a  ON a.attrelid = t.oid
                   AND a.attnum  = i.indkey[0]
WHERE t.relkind = 'r'
```

`run_index_advisor_check` line 297

```sql
SELECT a.attname AS col, ty.typname AS typname, a.atttypmod AS typmod,
       a.attrelid::bigint AS rel_oid
FROM pg_attribute a
JOIN pg_type ty ON ty.oid = a.atttypid
/* format() double-quotes only when needed, so
   case-sensitive identifiers survive. The explanation
   cannot name the format specifier here: psycopg2 scans
   the whole string for its own placeholders, comments
   included, and a bare percent makes it miscount. */
/* Looked up in the catalogue, not cast to regclass: the
   cast needs USAGE on the schema, and its refusal was an
   ERROR line in the server log for every login without it. */
WHERE a.attrelid = (SELECT c.oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
                     WHERE n.nspname = %s AND c.relname = %s)
  AND a.attnum > 0 AND NOT a.attisdropped
```

`run_index_advisor_check` line 327

```sql
SELECT lower(pg_get_indexdef(i.indexrelid)) AS def
  FROM pg_index i
 WHERE i.indrelid = (SELECT c.oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
                      WHERE n.nspname = %s AND c.relname = %s)
```

`run_index_advisor_check` line 497

```sql
WITH idxinfo AS (
    SELECT i.indexrelid::oid       AS oid,
           n.nspname                AS schema,
           t.relname                AS "table",
           c.relname                AS index,
           -- indkey/indclass/indoption are int2vector/oidvector,
           -- and casting them straight to int[] yields an array
           -- with lower bound 0. The prefix test below slices
           -- b.keys[1:len(a)], which on a 0-based array returns
           -- the SECOND column onwards - so leading-prefix
           -- duplicates never matched and this scan silently
           -- reported only exact duplicates. Verified on a live
           -- server: (val) vs (val, id) compared {2} against {1}.
           -- Going through text gives a normal 1-based array.
           string_to_array(i.indkey::text,   ' ')::int[] AS keys,
           string_to_array(i.indclass::text, ' ')::int[] AS classes,
           string_to_array(i.indoption::text,' ')::int[] AS opts,
           i.indpred IS NOT NULL    AS is_partial,
           i.indisunique            AS is_unique,
           i.indisprimary           AS is_primary,
           -- Undroppable for reasons indisunique/indisprimary do
           -- not cover: an EXCLUDE constraint's index is neither,
           -- and DROP INDEX on it fails with "constraint ...
           -- requires it". Same for a partition's index attached
           -- to its partitioned parent. A redundancy report that
           -- suggests statements the server refuses is worse than
           -- no report - both were reproduced on a live server.
           EXISTS (SELECT 1 FROM pg_constraint pc
                    WHERE pc.conindid = i.indexrelid) AS backs_constraint,
           EXISTS (SELECT 1 FROM pg_inherits inh
                    WHERE inh.inhrelid = i.indexrelid) AS is_attached,
           pg_relation_size(i.indexrelid) AS bytes
      FROM pg_index i
      JOIN pg_class c    ON c.oid = i.indexrelid
      JOIN pg_class t    ON t.oid = i.indrelid
      JOIN pg_namespace n ON n.oid = t.relnamespace
     WHERE n.nspname NOT IN ('pg_catalog','information_schema')
       AND t.relkind = 'r'
       AND i.indisvalid AND i.indisready
)
SELECT a.schema, a."table",
       a.index AS dup_index, a.bytes AS dup_bytes,
       b.index AS keep_index,
       -- Just the relationship; the index it is redundant AGAINST
       -- is keep_index, added once by the caller. Appending
       -- "of " || b.index here made the message read
       -- "leading-prefix of X of X".
       CASE WHEN a.keys = b.keys AND a.classes = b.classes
              AND a.opts = b.opts                  THEN 'identical signature'
            WHEN a.keys = b.keys[1:array_length(a.keys,1)] THEN 'leading-prefix'
            ELSE 'overlap'
       END AS reason
  FROM idxinfo a
  JOIN idxinfo b
    ON a.schema = b.schema AND a."table" = b."table"
   AND a.oid <> b.oid
   AND NOT a.is_partial AND NOT b.is_partial
   -- a is leading-prefix of (or identical to) b
   AND a.keys = b.keys[1:array_length(a.keys,1)]
   AND a.classes = b.classes[1:array_length(a.classes,1)]
   AND a.opts    = b.opts[1:array_length(a.opts,1)]
 WHERE NOT a.is_primary AND NOT a.is_unique
   -- 'a' is the index being proposed for removal, so the
   -- undroppable checks apply to it and not to the one kept.
   AND NOT a.backs_constraint AND NOT a.is_attached
   -- Two indexes with the same signature matched in both
   -- directions, so each was offered for DROP with the other
   -- named as the one that covers it - and the generated script
   -- dropped both. One row per pair: when neither side has a
   -- reason to stay, the one later by name goes and the first
   -- by name is kept.
   AND (a.keys <> b.keys OR a.classes <> b.classes OR a.opts <> b.opts
        OR b.is_primary OR b.is_unique OR b.backs_constraint OR b.is_attached
        OR a.index > b.index)
 ORDER BY a.bytes DESC
 LIMIT 50
```

**drop_sql** - `run_index_advisor_check` line 585

```sql
DROP INDEX CONCURRENTLY IF EXISTS {_qq(r['schema'], r['dup_index'])};
```

## checks/index_safety.py

**KEPT_FK_SQL** - `(module)` line 12

```sql
SELECT s.schemaname AS schema, s.relname AS "table", s.indexrelname AS index,
       pg_size_pretty(pg_relation_size(s.indexrelid)) AS size,
       (SELECT string_agg(f.conname, ', ' ORDER BY f.conname)
          FROM pg_constraint f
         WHERE f.contype = 'f' AND f.conrelid = i.indrelid
           AND i.indkey[0] = ANY(f.conkey)) AS foreign_key
  FROM pg_stat_user_indexes s
  JOIN pg_index i ON i.indexrelid = s.indexrelid
 WHERE s.idx_scan = 0
   AND NOT i.indisprimary AND NOT i.indisunique
   AND EXISTS (SELECT 1 FROM pg_constraint f
                WHERE f.contype = 'f' AND f.conrelid = i.indrelid
                  AND i.indkey[0] = ANY(f.conkey))
 ORDER BY pg_relation_size(s.indexrelid) DESC
 LIMIT 50
```

**PROTECTION_SQL** - `(module)` line 62

```sql
SELECT i.indexrelid::bigint AS indexrelid, n.nspname AS schema, t.relname AS "table",
       c.relname AS index, pg_relation_size(i.indexrelid) AS bytes,
       i.indisprimary AS is_pk, i.indisunique AS is_unique, i.indisreplident AS is_replident,
       (SELECT con.contype FROM pg_constraint con
         WHERE con.conindid = i.indexrelid AND con.contype IN ('p', 'u', 'x') LIMIT 1) AS contype,
       EXISTS (SELECT 1 FROM pg_constraint f
                WHERE f.contype = 'f' AND f.conrelid = i.indrelid
                  AND i.indkey[0] = ANY(f.conkey)) AS is_fk,
       EXISTS (SELECT 1 FROM pg_inherits inh WHERE inh.inhrelid = i.indexrelid) AS is_attached,
       (i.indisvalid AND i.indisready) AS in_service
  FROM pg_index i
  JOIN pg_class c ON c.oid = i.indexrelid
  JOIN pg_class t ON t.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = t.relnamespace
 WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND n.nspname NOT LIKE 'pg_toast%'
```

## checks/key_settings.py

`_freeze_clamp_status` line 169

```sql
VACUUM uses {limit}, not {n}: at most {label} of autovacuum_freeze_max_age ({fmax}), whatever is set here.
```

`_failsafe_status` line 182

```sql
VACUUM uses {floor}, not {n}: at least 105% of autovacuum_freeze_max_age ({fmax}).
```

`_load_ext_gucs` line 678

```sql
SELECT '[1]'::{_qi(vec['schema'])}.vector IS NOT NULL
```

`_load_ext_gucs` line 680

```sql
SELECT ic.relname AS name, pg_relation_size(ic.oid) AS bytes
  FROM pg_class ic JOIN pg_am am ON am.oid = ic.relam
 WHERE am.amname = 'hnsw' AND ic.relkind = 'i'
 ORDER BY 2 DESC LIMIT 1
```

`_load_ext_gucs` line 697

```sql
SELECT {_qi(trgm['schema'])}.show_limit()
```

`run_key_settings_check` line 765

```sql
SELECT name, setting, unit, category, short_desc,
       context, vartype, source,
       boot_val, reset_val, min_val, max_val, enumvals,
       {pending_col}, sourcefile, sourceline
FROM pg_settings
WHERE name = ANY(%s)
ORDER BY category, name
```

`run_key_settings_check` line 787

```sql
SELECT current_setting('is_superuser') AS issu
```

`run_key_settings_check` line 809

```sql
SELECT setting FROM pg_settings WHERE name = %s
```

## checks/logical_replication.py

`run_logical_replication_check` line 34

```sql
SELECT has_column_privilege('pg_catalog.pg_subscription', 'oid', 'SELECT') AS ok
```

`run_logical_replication_check` line 37

```sql
SELECT s.subname,
       s.subenabled                                       AS enabled,
       array_to_string(s.subpublications, ', ')           AS publications,
       st.pid                                             AS apply_pid,
       st.received_lsn::text                              AS received_lsn,
       st.latest_end_lsn::text                            AS latest_end_lsn,
       pg_wal_lsn_diff(st.latest_end_lsn, st.received_lsn) AS lag_bytes,
       CASE
           WHEN st.received_lsn IS NULL THEN 'NOT STARTED'
           WHEN st.received_lsn = st.latest_end_lsn THEN 'OK'
           ELSE 'LAG'
       END                                                AS status,
       st.last_msg_send_time,
       st.last_msg_receipt_time,
       st.latest_end_time
FROM pg_subscription s
LEFT JOIN pg_stat_subscription st ON {_sub_join}
ORDER BY s.subname
```

`run_logical_replication_check` line 96

```sql
SELECT s.subname,
       sr.srrelid::regclass::text                         AS table_name,
       sr.srsubstate                                      AS state_code,
       CASE sr.srsubstate
           WHEN 'r' THEN 'READY (replication OK)'
           WHEN 'd' THEN 'DATA COPY (initial sync)'
           WHEN 's' THEN 'SYNC (catch-up)'
           WHEN 'f' THEN 'FINISHED COPY'
           WHEN 'i' THEN 'INIT'
           ELSE 'UNKNOWN'
       END                                                AS meaning,
       sr.srsublsn::text                                  AS sublsn
FROM pg_subscription_rel sr
JOIN pg_subscription s ON s.oid = sr.srsubid
ORDER BY s.subname, sr.srrelid::regclass::text
```

## checks/migrations.py

`_flyway` line 98

```sql
SELECT installed_rank, version, description, type, script, installed_by,
       installed_on::text AS installed_on, execution_time
  FROM {tq} WHERE NOT success ORDER BY installed_rank
```

`_flyway` line 102

```sql
SELECT installed_rank, version, description, installed_on::text AS installed_on
  FROM {tq} WHERE success ORDER BY installed_rank DESC LIMIT 1
```

`_flyway` line 105

```sql
SELECT installed_rank, version, description, installed_on::text AS installed_on
  FROM {tq} WHERE success AND version IS NOT NULL ORDER BY installed_rank LIMIT 20000
```

`_flyway` line 118

```sql
SELECT count(*) AS n FROM {tq}
```

`_liquibase_lock` line 149

```sql
SELECT id, locked, lockgranted::text AS lockgranted, lockedby,
       EXTRACT(EPOCH FROM (localtimestamp - lockgranted))::float AS held_s
  FROM {tq} ORDER BY id
```

`_liquibase` line 175

```sql
SELECT id, author, filename, dateexecuted::text AS dateexecuted, orderexecuted, exectype
  FROM {tq} ORDER BY orderexecuted DESC LIMIT 1
```

`_liquibase` line 178

```sql
SELECT id, author, filename, dateexecuted::text AS dateexecuted, orderexecuted, exectype
  FROM {tq} WHERE exectype = 'FAILED' ORDER BY orderexecuted
```

`_liquibase` line 181

```sql
SELECT count(*) AS n FROM {tq}
```

`_alembic` line 201

```sql
SELECT version_num FROM {tq} ORDER BY version_num
```

`_golang_migrate` line 217

```sql
SELECT version, dirty FROM {tq} ORDER BY version DESC
```

`_rails` line 236

```sql
SELECT count(*) AS n, max(version) AS v FROM {tq}
```

`_django` line 248

```sql
SELECT app, name, applied::text AS applied FROM {tq} ORDER BY applied DESC, id DESC LIMIT 1
```

`_django` line 249

```sql
SELECT count(*) AS n FROM {tq}
```

`_prisma` line 258

```sql
SELECT migration_name, started_at::text AS started_at, applied_steps_count,
       left(coalesce(logs, ''), 400) AS logs,
       EXTRACT(EPOCH FROM (now() - started_at))::float AS since_s
  FROM {tq}
 WHERE finished_at IS NULL AND rolled_back_at IS NULL
 ORDER BY started_at
```

`_prisma` line 265

```sql
SELECT migration_name, finished_at::text AS finished_at FROM {tq}
 WHERE finished_at IS NOT NULL ORDER BY finished_at DESC LIMIT 1
```

`_prisma` line 268

```sql
SELECT count(*) AS n FROM {tq} WHERE finished_at IS NOT NULL
```

`_knex_lock` line 296

```sql
SELECT * FROM {tq}
```

`_knex` line 308

```sql
SELECT name, batch, migration_time::text AS migration_time FROM {tq} ORDER BY id DESC LIMIT 1
```

`_knex` line 309

```sql
SELECT count(*) AS n FROM {tq}
```

`_efcore` line 316

```sql
SELECT "MigrationId" AS migration_id, "ProductVersion" AS product_version FROM {tq} ORDER BY "MigrationId" DESC LIMIT 1
```

`_efcore` line 318

```sql
SELECT count(*) AS n FROM {tq}
```

`detect` line 334

```sql
SELECT n.nspname AS schema, c.relname AS name,
       pg_catalog.has_table_privilege(c.oid, 'SELECT') AS readable,
       array_agg(a.attname::text ORDER BY a.attnum) AS cols
  FROM pg_catalog.pg_class c
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_catalog.pg_attribute a ON a.attrelid = c.oid AND a.attnum > 0 AND NOT a.attisdropped
 WHERE c.relkind IN ('r', 'p')
   AND c.relname = ANY(%s)
   AND n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND n.nspname NOT LIKE 'pg\_toast%%' AND n.nspname NOT LIKE 'pg\_temp%%'
 GROUP BY n.nspname, c.relname, c.oid
 ORDER BY n.nspname, c.relname
```

## checks/ownership.py

`run_ownership_check` line 79

```sql
SELECT rolname FROM pg_roles WHERE rolname !~ '^pg_' ORDER BY rolname
```

`run_ownership_check` line 88

```sql
SELECT
    n.nspname AS schema,
    pg_get_userbyid(n.nspowner) AS owner,
    obj_description(n.oid, 'pg_namespace') AS comment
FROM pg_namespace n
WHERE n.nspname NOT LIKE 'pg_%'
  AND n.nspname <> 'information_schema'
ORDER BY n.nspname
```

**sql** - `run_ownership_check` line 104

```sql
SELECT
    n.nspname AS schema,
    c.relname AS name,
    CASE c.relkind
        WHEN 'r' THEN 'table' WHEN 'v' THEN 'view'
        WHEN 'm' THEN 'matview' WHEN 'p' THEN 'parted'
        WHEN 'f' THEN 'foreign' END AS kind,
    pg_get_userbyid(c.relowner) AS owner,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,
    obj_description(c.oid, 'pg_class') AS comment
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','v','m','f','p')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND ({flt})
ORDER BY n.nspname, c.relname
```

**sql** - `run_ownership_check` line 136

```sql
SELECT
    n.nspname AS schema,
    c.relname AS sequence,
    pg_get_userbyid(c.relowner) AS owner
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind = 'S'
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND ({flt})
ORDER BY n.nspname, c.relname
```

**sql** - `run_ownership_check` line 170

```sql
SELECT
    n.nspname AS schema,
    p.proname AS name,
    pg_get_function_arguments(p.oid) AS args,
    {kind_expr} AS kind,
    pg_get_userbyid(p.proowner) AS owner
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND NOT {_PART_OF_ANOTHER_OBJECT}
  AND ({flt})
ORDER BY n.nspname, p.proname
```

**sql** - `run_ownership_check` line 199

```sql
SELECT
    n.nspname AS schema,
    t.typname AS type,
    CASE t.typtype
        WHEN 'c' THEN 'composite'
        WHEN 'e' THEN 'enum' WHEN 'd' THEN 'domain'
        WHEN 'r' THEN 'range' WHEN 'm' THEN 'multirange' END AS kind,
    pg_get_userbyid(t.typowner) AS owner
FROM pg_type t
JOIN pg_namespace n ON n.oid = t.typnamespace
WHERE {_OWNED_TYPE}
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND ({flt})
ORDER BY n.nspname, t.typname
```

**sql_extra** - `run_ownership_check` line 244

```sql
SELECT n.nspname AS schema,
       x.{name_col} AS name,
       pg_get_userbyid(x.{owner_col}) AS owner
FROM {catalog} x
JOIN pg_namespace n ON n.oid = x.{ns_col}
WHERE n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND ({flt})
ORDER BY n.nspname, x.{name_col}
```

**sql_ext** - `run_ownership_check` line 273

```sql
SELECT n.nspname AS schema, s.stxname AS name,
       pg_get_userbyid(s.stxowner) AS owner
FROM pg_statistic_ext s
JOIN pg_namespace n ON n.oid = s.stxnamespace
WHERE n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND ({flt})
ORDER BY n.nspname, s.stxname
```

`run_ownership_check` line 296

```sql
SELECT n.nspname AS schema, c.relname AS "table",
       p.polname AS policy,
       CASE p.polcmd WHEN 'r' THEN 'SELECT' WHEN 'a' THEN 'INSERT'
                     WHEN 'w' THEN 'UPDATE' WHEN 'd' THEN 'DELETE'
                     WHEN '*' THEN 'ALL' END AS cmd,
       CASE WHEN p.polroles = '{0}'::oid[]
            THEN 'PUBLIC'
            ELSE array_to_string(ARRAY(
                 SELECT pg_get_userbyid(rid)
                 FROM unnest(p.polroles) AS rid
            ), ', ')
       END AS roles
FROM pg_policy p
JOIN pg_class c ON c.oid = p.polrelid
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND ({flt})
ORDER BY n.nspname, c.relname, p.polname
```

`run_ownership_check` line 330

```sql
SELECT fdwname AS name, pg_get_userbyid(fdwowner) AS owner
FROM pg_foreign_data_wrapper ORDER BY fdwname
```

`run_ownership_check` line 343

```sql
SELECT srvname AS name, srvtype AS type, srvversion AS version,
       pg_get_userbyid(srvowner) AS owner,
       (SELECT fdwname FROM pg_foreign_data_wrapper f
        WHERE f.oid = s.srvfdw) AS fdw
FROM pg_foreign_server s ORDER BY srvname
```

`run_ownership_check` line 359

```sql
SELECT lanname AS name, pg_get_userbyid(lanowner) AS owner,
       lanispl AS is_pl, lanpltrusted AS trusted
FROM pg_language WHERE lanispl OR lanname NOT IN ('internal','c','sql')
ORDER BY lanname
```

`run_ownership_check` line 375

```sql
SELECT pubname AS name, pg_get_userbyid(pubowner) AS owner,
       puballtables AS all_tables, pubinsert AS ins,
       pubupdate AS upd, pubdelete AS del
FROM pg_publication ORDER BY pubname
```

`run_ownership_check` line 390

```sql
SELECT subname AS name, pg_get_userbyid(subowner) AS owner,
       subenabled AS enabled
FROM pg_subscription ORDER BY subname
```

`run_ownership_check` line 407

```sql
WITH all_owned AS (
    SELECT pg_get_userbyid(c.relowner) AS owner,
           CASE c.relkind WHEN 'r' THEN 'table' WHEN 'p' THEN 'parted'
                          WHEN 'v' THEN 'view'  WHEN 'm' THEN 'matview'
                          WHEN 'f' THEN 'foreign' WHEN 'S' THEN 'sequence' END AS kind
    FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE c.relkind IN ('r','p','v','m','f','S')
      AND n.nspname NOT LIKE 'pg_%'
      AND n.nspname <> 'information_schema'
    UNION ALL
    SELECT pg_get_userbyid(p.proowner), 'function'
    FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
    WHERE n.nspname NOT LIKE 'pg_%'
      AND n.nspname <> 'information_schema'
      AND NOT {_PART_OF_ANOTHER_OBJECT}
    UNION ALL
    SELECT pg_get_userbyid(t.typowner), 'type'
    FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace
    WHERE {_OWNED_TYPE}
      AND n.nspname NOT LIKE 'pg_%'
      AND n.nspname <> 'information_schema'
)
SELECT owner, kind, count(*)::int AS count
FROM all_owned
GROUP BY owner, kind
ORDER BY owner, kind
```

**sql** - `run_ownership_check` line 447

```sql
SELECT
    n.nspname AS schema,
    pg_get_userbyid(n.nspowner) AS schema_owner,
    c.relname AS object,
    CASE c.relkind WHEN 'r' THEN 'table' WHEN 'p' THEN 'parted'
                   WHEN 'v' THEN 'view'  WHEN 'm' THEN 'matview'
                   WHEN 'S' THEN 'sequence' END AS kind,
    pg_get_userbyid(c.relowner) AS object_owner
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','p','v','m','S')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND c.relowner <> n.nspowner
  AND ({flt})
ORDER BY n.nspname, c.relname
```

`run_ownership_check` line 494

```sql
SELECT pg_get_userbyid(c.relowner) AS owner,
       count(*)                    AS n
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
JOIN pg_roles ro ON ro.oid = c.relowner
WHERE c.relkind IN ('r','p','v','m','S','f')
  AND ro.rolsuper = true
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND ({flt})
GROUP BY ro.rolname, c.relowner
ORDER BY n DESC
```

`run_ownership_check` line 521

```sql
SELECT pg_get_userbyid(p.proowner) AS owner,
       sum(CASE WHEN p.prosecdef THEN 1 ELSE 0 END)::int AS n_secdef,
       count(*)                                          AS n_total
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
JOIN pg_roles ro ON ro.oid = p.proowner
WHERE ro.rolsuper = true
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND NOT {_PART_OF_ANOTHER_OBJECT}
  AND ({flt})
GROUP BY p.proowner, ro.rolname
ORDER BY n_total DESC
```

`run_ownership_check` line 539

```sql
SECURITY DEFINER routine(s) in scope owned by superuser '
```

`run_ownership_check` line 554

```sql
SELECT n.nspname AS schema,
       c.relname AS object,
       c.relkind,
       c.relowner AS dangling_owner_oid
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','p','v','m','S','f')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND NOT EXISTS (SELECT 1 FROM pg_roles ro WHERE ro.oid = c.relowner)
  AND ({flt})
ORDER BY n.nspname, c.relname
```

`run_ownership_check` line 584

```sql
SELECT pg_get_userbyid(n.nspowner) AS owner,
       r.rolsuper                  AS owner_is_super
FROM pg_namespace n
JOIN pg_roles r ON r.oid = n.nspowner
WHERE n.nspname = 'public'
```

`run_ownership_check` line 604

```sql
SELECT n.nspname AS schema,
       count(DISTINCT c.relowner) AS distinct_owners
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','p','v','m','S','f')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND ({flt})
GROUP BY n.nspname
HAVING count(DISTINCT c.relowner) >= 3
ORDER BY count(DISTINCT c.relowner) DESC
LIMIT 20
```

## checks/partition_runway.py

`_partman` line 81

```sql
SELECT n.nspname AS nsp, e.extversion AS ver
  FROM pg_extension e JOIN pg_namespace n ON n.oid = e.extnamespace
 WHERE e.extname = 'pg_partman'
```

`_partman` line 95

```sql
SELECT * FROM {q}
```

`_partman` line 115

```sql
SELECT extract(epoch FROM %s::interval) / 86400.0 AS d
```

`_partman` line 122

```sql
SELECT setting AS spl FROM pg_settings WHERE name = 'shared_preload_libraries'
```

`_partman` line 128

```sql
SELECT current_setting('pg_partman_bgw.dbname', true) AS d
```

`_partman` line 135

```sql
SELECT count(*) AS n FROM cron.job WHERE command ILIKE '%run_maintenance%' AND active
```

`collect` line 166

```sql
SELECT c.oid::bigint AS oid, n.nspname AS schema, c.relname AS name,
       pt.partstrat AS strategy, pg_get_partkeydef(c.oid) AS partkey,
       {defcol}::bigint AS default_oid, c.relispartition AS is_partition,
       (SELECT format_type(oc.opcintype, NULL) FROM pg_opclass oc
         WHERE oc.oid = pt.partclass[0]) AS key_type
  FROM pg_partitioned_table pt
  JOIN pg_class c ON c.oid = pt.partrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND n.nspname NOT LIKE 'pg_toast%%'
 ORDER BY n.nspname, c.relname
```

`collect` line 182

```sql
SELECT i.inhparent::bigint AS parent, c.oid::bigint AS oid, n.nspname AS schema,
       c.relname AS name, pg_get_expr(c.relpartbound, c.oid) AS bound,
       greatest(c.reltuples, 0)::bigint AS reltuples,
       coalesce(s.n_live_tup, 0)::bigint AS live
  FROM pg_inherits i
  JOIN pg_class c ON c.oid = i.inhrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_stat_all_tables s ON s.relid = c.oid
 WHERE i.inhparent = ANY(%s::oid[])
```

`collect` line 232

```sql
SELECT x.k::bigint AS k, x.u AS u,
       CASE x.kind
         WHEN 'date' THEN extract(epoch FROM (x.u::date::timestamp - localtimestamp))
         WHEN 'ts'   THEN extract(epoch FROM (x.u::timestamp - localtimestamp))
         ELSE             extract(epoch FROM (x.u::timestamptz - now()))
       END / 86400.0 AS days
  FROM unnest(%s::text[], %s::text[], %s::text[]) AS x(k, u, kind)
```

## checks/pgss_state.py

`pgss_state` line 32

```sql
SELECT 1 FROM pg_extension WHERE extname = 'pg_stat_statements'
```

`pgss_state` line 37

```sql
SELECT 1 FROM pg_available_extensions WHERE name = 'pg_stat_statements'
```

`pgss_state` line 44

```sql
SELECT setting FROM pg_settings WHERE name = 'shared_preload_libraries'
```

`pgss_state` line 52

```sql
SELECT usesuper FROM pg_user WHERE usename = current_user
```

`pgss_advice` line 90

```sql
CREATE EXTENSION again will not change anything.
```

`_pgss_columns_now` line 193

```sql
SELECT column_name FROM information_schema.columns WHERE table_name = 'pg_stat_statements'
```

`pgss_info` line 267

```sql
SELECT current_setting('pg_stat_statements.max', true) AS v
```

`pgss_info` line 270

```sql
SELECT count(*)::bigint AS n FROM {pss_name}
```

`pgss_info` line 283

```sql
SELECT dealloc::bigint AS dealloc, stats_reset::text AS stats_reset FROM {info}
```

## checks/pgstattuple.py

`run_pgstattuple_check` line 32

```sql
SELECT extversion FROM pg_extension WHERE extname = 'pgstattuple'
```

`run_pgstattuple_check` line 103

```sql
SELECT n.nspname AS schema, c.relname AS "table",
       pg_total_relation_size(c.oid) AS bytes,
       pg_size_pretty(pg_total_relation_size(c.oid)) AS size
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relname = %s
  AND c.relkind IN ('r','m')
```

`run_pgstattuple_check` line 113

```sql
SELECT n.nspname AS schema, c.relname AS "table",
       pg_total_relation_size(c.oid) AS bytes,
       pg_size_pretty(pg_total_relation_size(c.oid)) AS size
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','m')
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
  AND pg_total_relation_size(c.oid) >= %s
  AND (%s IS NULL OR pg_total_relation_size(c.oid) <= %s)
ORDER BY pg_total_relation_size(c.oid) DESC
LIMIT %s
```

`run_pgstattuple_check` line 179

```sql
SELECT * FROM {_pgstattuple}(format('%%I.%%I', %s, %s)::regclass)
```

`run_pgstattuple_check` line 257

```sql
SELECT i.relname AS index, i.oid AS index_oid,
       pg_relation_size(i.oid) AS index_bytes,
       x.indisprimary AS is_pk, x.indisunique AS is_unique,
       COALESCE(s.idx_scan, 0) AS idx_scan
  FROM pg_index x
  JOIN pg_class i ON i.oid = x.indexrelid
  JOIN pg_am am ON am.oid = i.relam
  LEFT JOIN pg_stat_user_indexes s ON s.indexrelid = i.oid
 WHERE x.indrelid = format('%%I.%%I', %s, %s)::regclass
   AND am.amname = 'btree'
 ORDER BY pg_relation_size(i.oid) DESC
```

`run_pgstattuple_check` line 281

```sql
SELECT avg_leaf_density, leaf_fragmentation, leaf_pages, internal_pages, deleted_pages FROM {_pgstatindex}(%s::regclass)
```

## checks/prepared_xacts.py

`run_prepared_xacts_check` line 27

```sql
SELECT current_setting('max_prepared_transactions')::int AS m
```

`run_prepared_xacts_check` line 53

```sql
SELECT gid,
       owner,
       database,
       prepared,
       age(now(), prepared)::text                    AS prepared_age,
       EXTRACT(EPOCH FROM now() - prepared)::bigint   AS prepared_age_s,
       transaction::text                             AS xid,
       age(transaction)::bigint                      AS xid_age
  FROM pg_prepared_xacts
 ORDER BY prepared
```

`run_prepared_xacts_check` line 92

```sql
SELECT px.gid,
       count(*)::int                                       AS n_locks,
       coalesce(array_agg(DISTINCT l.relation::regclass::text)
                FILTER (WHERE l.relation IS NOT NULL), '{}') AS relations
  FROM pg_locks l
  JOIN LATERAL (
    SELECT p.gid
      FROM pg_locks tl
      JOIN pg_prepared_xacts p ON p.transaction = tl.transactionid
     WHERE tl.pid IS NULL AND tl.locktype = 'transactionid'
       AND tl.virtualtransaction = l.virtualtransaction
     LIMIT 1
  ) px ON l.pid IS NULL
 GROUP BY px.gid
```

## checks/query_history.py

`run_query_history_check` line 113

```sql
SELECT count(*) AS n FROM {_pss}
```

`run_query_history_check` line 134

```sql
SELECT stats_reset FROM pg_stat_database WHERE datname = current_database()
```

`run_query_history_check` line 192

```sql
SELECT s.queryid::text AS query_id,
       r.rolname       AS "user",
       s.calls,
       {_total}  AS total_ms,
       {_mean}   AS mean_ms,
       {_max}    AS max_ms,
       {_stddev} AS stddev_ms,
       s.rows,
       s.query
  FROM {_pss} s
  LEFT JOIN pg_roles r ON r.oid = s.userid
 WHERE {' AND '.join(where)}
 ORDER BY {order_sql}
 LIMIT %s
```

`run_query_history_check` line 276

```sql
SHOW server_version_num
```

`run_query_history_check` line 304

```sql
EXPLAIN ({opts}) {q}
```

## checks/queues.py

`_known_tables` line 128

```sql
SELECT n.nspname AS schema, c.relname AS name,
       array_agg(a.attname::text ORDER BY a.attnum) AS cols
  FROM pg_catalog.pg_class c
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_catalog.pg_attribute a ON a.attrelid = c.oid AND a.attnum > 0 AND NOT a.attisdropped
 WHERE c.relkind IN ('r', 'p')
   AND ((n.nspname = 'pgmq' AND c.relname LIKE 'q\_%')
     OR (n.nspname = 'graphile_worker' AND c.relname IN ('jobs', '_private_jobs'))
     OR c.relname IN ('que_jobs', 'river_job'))
 GROUP BY n.nspname, c.relname
 ORDER BY 1, 2
```

`_guarded` line 183

```sql
SELECT current_setting('statement_timeout') AS t
```

`_guarded` line 185

```sql
SET statement_timeout = %s
```

`_guarded` line 197

```sql
SELECT set_config('statement_timeout', %s, false)
```

`_table_stats` line 203

```sql
SELECT c.oid, coalesce(s.n_live_tup, 0) AS n_live, coalesce(s.n_dead_tup, 0) AS n_dead,
       coalesce(s.n_tup_ins, 0) AS inserts, coalesce(s.n_tup_del, 0) AS deletes,
       coalesce(s.n_tup_upd, 0) AS updates,
       pg_catalog.pg_total_relation_size(c.oid) AS bytes,
       greatest(s.last_autovacuum, s.last_vacuum)::text AS last_vacuum,
       EXTRACT(EPOCH FROM now() - greatest(s.last_autovacuum, s.last_vacuum))::float AS vacuum_age_s,
       pg_catalog.has_table_privilege(c.oid, 'SELECT') AS readable
  FROM pg_catalog.pg_class c
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_catalog.pg_stat_all_tables s ON s.relid = c.oid
 WHERE n.nspname = %s AND c.relname = %s
```

`_measure` line 239

```sql
SELECT count(*) AS n FROM (SELECT 1 FROM {tq}{where} LIMIT {COUNT_CAP + 1}) s
```

`_measure` line 245

```sql
SELECT EXTRACT(EPOCH FROM now() - min({_qi(due)}))::float AS s FROM {tq}{where}
```

`_measure` line 251

```sql
SELECT count(*) AS n FROM (SELECT 1 FROM {tq} WHERE {failed} LIMIT {COUNT_CAP + 1}) s
```

`_pgq_consumers` line 267

```sql
SELECT queue_name, consumer_name, EXTRACT(EPOCH FROM lag)::float AS lag_s,
       EXTRACT(EPOCH FROM last_seen)::float AS last_seen_s, pending_events
  FROM pgq.get_consumer_info()
```

`find_queues` line 288

```sql
SELECT 1 FROM pg_catalog.pg_proc p JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace WHERE n.nspname = 'pgq' AND p.proname = 'get_consumer_info' LIMIT 1
```

`find_queues` line 300

```sql
SELECT query, calls, rows
  FROM pg_stat_statements
 WHERE dbid = (SELECT oid FROM pg_catalog.pg_database WHERE datname = current_database())
   AND query ~* 'skip[[:space:]]+locked'
 ORDER BY calls DESC LIMIT 50
```

`find_queues` line 321

```sql
SELECT n.nspname AS schema, c.relname AS name FROM pg_catalog.pg_class c
                                 JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
                                WHERE n.nspname = %s AND c.relname = %s AND c.relkind IN ('r', 'p')
```

`find_queues` line 329

```sql
SELECT n.nspname AS schema, c.relname AS name,
                                      pg_catalog.pg_table_is_visible(c.oid) AS visible
                                 FROM pg_catalog.pg_class c
                                 JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
                                WHERE c.relname = %s AND c.relkind IN ('r', 'p')
                                  AND n.nspname NOT IN ('pg_catalog', 'information_schema')
                                ORDER BY 3 DESC
```

`find_queues` line 344

```sql
SELECT array_agg(a.attname::text) AS cols FROM pg_catalog.pg_attribute a
                            WHERE a.attrelid = (SELECT c.oid FROM pg_catalog.pg_class c
                                                  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
                                                 WHERE n.nspname = %s AND c.relname = %s)
                              AND a.attnum > 0 AND NOT a.attisdropped
```

`notify_state` line 364

```sql
SELECT pg_catalog.pg_notification_queue_usage() AS u
```

`notify_state` line 368

```sql
SELECT current_setting('max_notify_queue_pages', true) AS p, current_setting('block_size')::int AS b
```

## checks/repl_slots.py

**sql** - `run_repl_slots_check` line 39

```sql
SELECT slot_name,
       plugin,
       slot_type,
       database,
{temp_col}       active,
{active_pid_col}       restart_lsn::text                                  AS restart_lsn,
{conf_flush_col}       CASE
           WHEN restart_lsn IS NULL OR pg_is_in_recovery()
               THEN NULL
           ELSE {lsn_diff}({cur_lsn}, restart_lsn)
       END                                                AS retained_wal_bytes,
       CASE
           WHEN restart_lsn IS NULL OR pg_is_in_recovery()
               THEN NULL
           ELSE pg_size_pretty(
                    {lsn_diff}({cur_lsn}, restart_lsn))
       END                                                AS retained_wal,
{extra_cols}       0 AS _sort_pad
FROM pg_replication_slots
ORDER BY active ASC, retained_wal_bytes DESC NULLS LAST
```

## checks/repl_stats.py

`_has_view` line 33

```sql
SELECT to_regclass(%s) IS NOT NULL AS ok
```

`subscription_stats_section` line 52

```sql
SELECT row_to_json(s)::text AS j FROM pg_stat_subscription_stats s ORDER BY subname
```

`slot_stats_section` line 95

```sql
SELECT row_to_json(s)::text AS j FROM pg_stat_replication_slots s ORDER BY slot_name
```

`slot_stats_section` line 97

```sql
SELECT row_to_json(s)::text AS j FROM pg_replication_slots s
```

`recovery_section` line 144

```sql
SELECT row_to_json(s)::text AS j FROM pg_stat_recovery s
```

## checks/replica_identity.py

`run_replica_identity_check` line 22

```sql
SELECT schemaname, tablename FROM pg_publication_tables
```

`run_replica_identity_check` line 30

```sql
SELECT n.nspname                                           AS schema,
       c.relname                                           AS table,
       c.relkind                                           AS relkind,
       c.relreplident                                      AS replident,
       CASE c.relreplident
           WHEN 'd' THEN 'DEFAULT'
           WHEN 'n' THEN 'NOTHING'
           WHEN 'f' THEN 'FULL'
           WHEN 'i' THEN 'INDEX'
           ELSE c.relreplident::text
       END                                                 AS identity_mode,
       pg_size_pretty(pg_relation_size(c.oid))             AS size,
       -- best unique non-null index candidate for REPLICA IDENTITY INDEX
       (
           SELECT ci.relname
             FROM pg_index i2
             JOIN pg_class ci ON ci.oid = i2.indexrelid
            WHERE i2.indrelid = c.oid
              AND i2.indisunique
              AND NOT i2.indisprimary
              AND i2.indpred IS NULL  -- not partial
              AND NOT EXISTS (
                  -- all indexed cols must be NOT NULL
                  SELECT 1 FROM unnest(i2.indkey) AS k(attnum)
                    JOIN pg_attribute a
                      ON a.attrelid = c.oid AND a.attnum = k.attnum
                   WHERE NOT a.attnotnull OR k.attnum = 0
              )
            ORDER BY ci.relname
            LIMIT 1
       )                                                   AS suggested_unique_idx,
       format('ALTER TABLE %I.%I REPLICA IDENTITY FULL;',
              n.nspname, c.relname)                        AS alter_to_full,
       format('ALTER TABLE %I.%I REPLICA IDENTITY DEFAULT;',
              n.nspname, c.relname)                        AS alter_to_default
FROM pg_class c
JOIN pg_namespace n   ON n.oid = c.relnamespace
LEFT JOIN pg_constraint con ON con.conrelid = c.oid AND con.contype = 'p'
WHERE c.relkind IN ('r', 'p')
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
  AND con.oid IS NULL
ORDER BY n.nspname, c.relname
```

`run_replica_identity_check` line 135

```sql
SELECT n.nspname                                           AS schema,
       c.relname                                           AS table,
       c.relkind                                           AS relkind,
       CASE c.relreplident
           WHEN 'n' THEN 'NOTHING'
           WHEN 'f' THEN 'FULL'
           WHEN 'i' THEN 'INDEX'
           ELSE c.relreplident::text
       END                                                 AS identity_mode,
       pg_size_pretty(pg_relation_size(c.oid))             AS size,
       format('ALTER TABLE %I.%I REPLICA IDENTITY DEFAULT;',
              n.nspname, c.relname)                        AS revert_command
FROM pg_class c
JOIN pg_namespace n   ON n.oid = c.relnamespace
WHERE c.relkind IN ('r', 'p')
  AND c.relreplident <> 'd'
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
ORDER BY n.nspname, c.relname
```

## checks/role_perms.py

**_SQL_ROLE_OVERVIEW** - `(module)` line 19

```sql
SELECT
    r.rolname AS name,
    shobj_description(r.oid, 'pg_authid') AS comment,
    r.rolcanlogin    AS can_login,
    r.rolinherit     AS inherits,
    r.rolsuper       AS is_superuser,
    r.rolcreatedb    AS create_db,
    r.rolcreaterole  AS create_role,
    r.rolreplication AS replication,
    {BYPASS_RLS}     AS bypass_rls,
    r.rolconnlimit   AS conn_limit,
    r.rolvaliduntil  AS valid_until
FROM pg_roles r
WHERE r.rolname = %s
```

**_SQL_ROLE_DIRECT_MEMBERSHIPS** - `(module)` line 36

```sql
SELECT
    g.rolname AS granted_role,
    shobj_description(g.oid, 'pg_authid') AS role_comment,
    am.admin_option,
    pg_has_role(%s, g.oid, 'USAGE') AS effectively_inherits
FROM pg_auth_members am
JOIN pg_roles m ON m.oid = am.member
JOIN pg_roles g ON g.oid = am.roleid
WHERE m.rolname = %s
ORDER BY g.rolname
```

`_sql_role_direct_memberships` line 53

```sql
SELECT
    g.rolname AS granted_role,
    shobj_description(g.oid, 'pg_authid') AS role_comment,
    am.admin_option,
    {inh_col}
    pg_has_role(%s, g.oid, 'USAGE') AS effectively_inherits
FROM pg_auth_members am
JOIN pg_roles m ON m.oid = am.member
JOIN pg_roles g ON g.oid = am.roleid
WHERE m.rolname = %s
ORDER BY g.rolname
```

**_SQL_ROLE_TRANSITIVE** - `(module)` line 69

```sql
WITH RECURSIVE chain AS (
    SELECT
        m.rolname AS root,
        g.rolname AS role,
        1 AS depth,
        ARRAY[m.rolname, g.rolname] AS path,
        pg_has_role(m.rolname, g.oid, 'USAGE') AS effectively_inherits
    FROM pg_auth_members am
    JOIN pg_roles m ON m.oid = am.member
    JOIN pg_roles g ON g.oid = am.roleid
    WHERE m.rolname = %s
    UNION ALL
    SELECT
        chain.root,
        g2.rolname,
        chain.depth + 1,
        chain.path || g2.rolname,
        pg_has_role(chain.root, g2.oid, 'USAGE')
    FROM chain
    JOIN pg_roles m2 ON m2.rolname = chain.role
    JOIN pg_auth_members am2 ON am2.member = m2.oid
    JOIN pg_roles g2 ON g2.oid = am2.roleid
    WHERE NOT (g2.rolname = ANY(chain.path)) AND chain.depth < 10
)
SELECT
    role AS granted_role,
    depth,
    array_to_string(path, ' → ') AS path,
    effectively_inherits
FROM chain
ORDER BY depth, role
```

**_SQL_ROLE_DB_PRIVS** - `(module)` line 103

```sql
SELECT
    d.datname AS database,
    has_database_privilege(%s, d.oid, 'CONNECT')   AS connect,
    has_database_privilege(%s, d.oid, 'CREATE')    AS create,
    has_database_privilege(%s, d.oid, 'TEMPORARY') AS temporary
FROM pg_database d
WHERE d.datistemplate = false
ORDER BY d.datname
```

**_SQL_ROLE_SCHEMA_PRIVS** - `(module)` line 121

```sql
SELECT
    n.nspname AS schema,
    pg_get_userbyid(n.nspowner) AS owner,
    has_schema_privilege(%s, n.oid, 'USAGE')  AS usage,
    has_schema_privilege(%s, n.oid, 'CREATE') AS create
FROM pg_namespace n
WHERE n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND (
        has_schema_privilege(%s, n.oid, 'USAGE')
     OR has_schema_privilege(%s, n.oid, 'CREATE')
     OR pg_get_userbyid(n.nspowner) = %s
  )
ORDER BY n.nspname
```

**_USER_ROLES_CTE** - `(module)` line 140

```sql
WITH user_roles AS (
    SELECT r.rolname
    FROM pg_roles r
    WHERE r.rolname = %s OR pg_has_role(%s, r.oid, 'USAGE')
)
```

`(module)` line 148

```sql
SELECT
    n.nspname AS schema,
    c.relname AS name,
    CASE c.relkind
        WHEN 'r' THEN 'table' WHEN 'v' THEN 'view'
        WHEN 'm' THEN 'matview' WHEN 'p' THEN 'parted'
        WHEN 'f' THEN 'foreign' END AS kind,
    pg_get_userbyid(c.relowner) AS owner,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role oid=' || a.grantee || ')')
    END AS via_role,
    a.privilege_type AS privilege,
    a.is_grantable
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
CROSS JOIN LATERAL aclexplode(c.relacl) a
WHERE c.relkind IN ('r','v','m','f','p')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND (
        a.grantee = 0
     OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles)
  )
ORDER BY schema, name, privilege
```

**_SQL_ROLE_OWNED_TABLES** - `(module)` line 175

```sql
SELECT
    n.nspname AS schema,
    c.relname AS name,
    CASE c.relkind
        WHEN 'r' THEN 'table' WHEN 'v' THEN 'view'
        WHEN 'm' THEN 'matview' WHEN 'p' THEN 'parted'
        WHEN 'f' THEN 'foreign' WHEN 'S' THEN 'sequence' END AS kind
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
JOIN pg_roles r ON r.oid = c.relowner
WHERE c.relkind IN ('r','v','m','f','p','S')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  -- DIRECT ownership only - PG ownership doesn't inherit. Earlier this
  -- query included objects owned by any role the target was a member of
  -- (via pg_has_role USAGE), which made the list look like the whole
  -- cluster when the target was a member of a superuser/admin role.
  -- The cluster-wide owned-by-anyone view lives in Object ownership.
  AND r.rolname = %s
ORDER BY schema, name
```

`(module)` line 198

```sql
SELECT
    n.nspname AS schema,
    c.relname AS table,
    a.attname AS column,
    CASE WHEN ae.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(ae.grantee), '(deleted role)')
    END AS via_role,
    ae.privilege_type AS privilege,
    ae.is_grantable
FROM pg_attribute a
JOIN pg_class c ON c.oid = a.attrelid
JOIN pg_namespace n ON n.oid = c.relnamespace
CROSS JOIN LATERAL aclexplode(a.attacl) ae
WHERE a.attacl IS NOT NULL
  AND a.attnum > 0
  AND NOT a.attisdropped
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND (
        ae.grantee = 0
     OR pg_get_userbyid(ae.grantee) IN (SELECT rolname FROM user_roles)
  )
ORDER BY schema, "table", "column", privilege
```

`(module)` line 224

```sql
SELECT
    n.nspname AS schema,
    c.relname AS name,
    pg_get_userbyid(c.relowner) AS owner,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)')
    END AS via_role,
    a.privilege_type AS privilege,
    a.is_grantable
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
CROSS JOIN LATERAL aclexplode(c.relacl) a
WHERE c.relkind = 'S'
  AND n.nspname NOT LIKE 'pg_%%'
  AND (
        a.grantee = 0
     OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles)
  )
ORDER BY schema, name, privilege
```

`_sql_role_function_privs` line 255

```sql
SELECT
    n.nspname AS schema,
    p.proname AS function,
    pg_get_function_arguments(p.oid) AS args,
    {kind_expr} AS kind,
    pg_get_userbyid(p.proowner) AS owner,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)')
    END AS via_role,
    a.privilege_type AS privilege,
    a.is_grantable
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
CROSS JOIN LATERAL aclexplode(p.proacl) a
WHERE n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND (
        a.grantee = 0
     OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles)
  )
ORDER BY schema, function, privilege
```

`(module)` line 283

```sql
SELECT
    pg_get_userbyid(d.defaclrole) AS owner,
    COALESCE(n.nspname, '(global)') AS schema,
    CASE d.defaclobjtype
        WHEN 'r' THEN 'tables'
        WHEN 'S' THEN 'sequences'
        WHEN 'f' THEN 'functions'
        WHEN 'T' THEN 'types'
        WHEN 'n' THEN 'schemas'
    END AS object_type,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)')
    END AS grantee,
    a.privilege_type AS privilege,
    a.is_grantable
FROM pg_default_acl d
LEFT JOIN pg_namespace n ON n.oid = d.defaclnamespace
CROSS JOIN LATERAL aclexplode(d.defaclacl) a
WHERE a.grantee = 0
   OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles)
ORDER BY owner, schema, object_type, privilege
```

`(module)` line 307

```sql
SELECT
    n.nspname AS schema,
    c.relname AS "table",
    pol.polname AS policy,
    CASE pol.polcmd
        WHEN '*' THEN 'ALL' WHEN 'r' THEN 'SELECT' WHEN 'a' THEN 'INSERT'
        WHEN 'w' THEN 'UPDATE' WHEN 'd' THEN 'DELETE' END AS cmd,
    {POLPERMISSIVE} AS permissive,
    (
        SELECT array_to_string(array_agg(
            CASE WHEN role_oid = 0 THEN 'PUBLIC'
                 ELSE COALESCE(pg_get_userbyid(role_oid), '(deleted)')
            END
        ), ', ')
        FROM unnest(pol.polroles) AS role_oid
    ) AS applied_to_roles,
    pg_get_expr(pol.polqual, pol.polrelid) AS using_expr,
    pg_get_expr(pol.polwithcheck, pol.polrelid) AS check_expr
FROM pg_policy pol
JOIN pg_class c ON c.oid = pol.polrelid
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE 0 = ANY(pol.polroles)
   OR EXISTS (
       SELECT 1 FROM unnest(pol.polroles) AS role_oid
       WHERE pg_get_userbyid(role_oid) IN (SELECT rolname FROM user_roles)
   )
ORDER BY schema, "table", policy
```

`(module)` line 339

```sql
SELECT
    n.nspname AS schema,
    t.typname AS name,
    CASE t.typtype WHEN 'c' THEN 'composite' WHEN 'e' THEN 'enum'
                   WHEN 'd' THEN 'domain'    WHEN 'r' THEN 'range'
                   WHEN 'm' THEN 'multirange' END AS kind,
    pg_get_userbyid(t.typowner) AS owner,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)')
    END AS via_role,
    a.privilege_type AS privilege
FROM pg_type t
JOIN pg_namespace n ON n.oid = t.typnamespace
CROSS JOIN LATERAL aclexplode(t.typacl) a
WHERE t.typacl IS NOT NULL
  AND t.typtype IN ('c','e','d','r','m')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND (a.grantee = 0
       OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles))
ORDER BY n.nspname, t.typname, a.privilege_type
```

`(module)` line 364

```sql
SELECT
    f.fdwname AS name,
    pg_get_userbyid(f.fdwowner) AS owner,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)')
    END AS via_role,
    a.privilege_type AS privilege
FROM pg_foreign_data_wrapper f
CROSS JOIN LATERAL aclexplode(f.fdwacl) a
WHERE f.fdwacl IS NOT NULL
  AND (a.grantee = 0
       OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles))
ORDER BY f.fdwname, a.privilege_type
```

`(module)` line 381

```sql
SELECT
    s.srvname AS name,
    pg_get_userbyid(s.srvowner) AS owner,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)')
    END AS via_role,
    a.privilege_type AS privilege
FROM pg_foreign_server s
CROSS JOIN LATERAL aclexplode(s.srvacl) a
WHERE s.srvacl IS NOT NULL
  AND (a.grantee = 0
       OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles))
ORDER BY s.srvname, a.privilege_type
```

`(module)` line 398

```sql
SELECT
    l.lanname AS name,
    pg_get_userbyid(l.lanowner) AS owner,
    l.lanpltrusted AS trusted,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)')
    END AS via_role,
    a.privilege_type AS privilege
FROM pg_language l
CROSS JOIN LATERAL aclexplode(l.lanacl) a
WHERE l.lanacl IS NOT NULL
  AND (a.grantee = 0
       OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles))
ORDER BY l.lanname, a.privilege_type
```

`(module)` line 416

```sql
SELECT
    t.spcname AS name,
    pg_get_userbyid(t.spcowner) AS owner,
    CASE WHEN a.grantee = 0 THEN 'PUBLIC'
         ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)')
    END AS via_role,
    a.privilege_type AS privilege
FROM pg_tablespace t
CROSS JOIN LATERAL aclexplode(t.spcacl) a
WHERE t.spcacl IS NOT NULL
  AND (a.grantee = 0
       OR pg_get_userbyid(a.grantee) IN (SELECT rolname FROM user_roles))
ORDER BY t.spcname, a.privilege_type
```

**_SQL_ROLE_CLUSTER_OWNED** - `(module)` line 435

```sql
SELECT 'database' AS kind, datname AS name
  FROM pg_database d JOIN pg_roles r ON r.oid = d.datdba
 WHERE r.rolname = %s
UNION ALL
SELECT 'tablespace', spcname
  FROM pg_tablespace t JOIN pg_roles r ON r.oid = t.spcowner
 WHERE r.rolname = %s
UNION ALL
SELECT 'fdw', fdwname
  FROM pg_foreign_data_wrapper f JOIN pg_roles r ON r.oid = f.fdwowner
 WHERE r.rolname = %s
UNION ALL
SELECT 'foreign server', srvname
  FROM pg_foreign_server s JOIN pg_roles r ON r.oid = s.srvowner
 WHERE r.rolname = %s
UNION ALL
SELECT 'language', lanname
  FROM pg_language l JOIN pg_roles r ON r.oid = l.lanowner
 WHERE r.rolname = %s
ORDER BY 1, 2
```

**_SQL_ROLE_PUBSUB_OWNED** - `(module)` line 460

```sql
SELECT 'publication' AS kind, pubname AS name
  FROM pg_publication p JOIN pg_roles r ON r.oid = p.pubowner
 WHERE r.rolname = %s
UNION ALL
SELECT 'subscription', subname
  FROM pg_subscription s JOIN pg_roles r ON r.oid = s.subowner
 WHERE r.rolname = %s
ORDER BY 1, 2
```

`_build_role_audit_ctx` line 513

```sql
SELECT rolsuper, {BYPASS} AS rolbypassrls, rolcreaterole, rolreplication,
       rolcanlogin, rolinherit, rolconnlimit, rolvaliduntil
FROM pg_roles WHERE rolname = %s
```

`_build_role_audit_ctx` line 535

```sql
SELECT n.nspname AS schema,
       has_schema_privilege(%s, n.oid, 'USAGE')  AS u,
       has_schema_privilege(%s, n.oid, 'CREATE') AS c,
       pg_get_userbyid(n.nspowner) AS owner
FROM pg_namespace n
WHERE n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
```

`_build_role_audit_ctx` line 558

```sql
SELECT n.nspname AS schema, c.relname AS name
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
JOIN pg_roles ro ON ro.oid = c.relowner
WHERE ro.rolname = %s
  AND c.relkind IN ('r','p','v','m','S','f')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
```

`_build_role_audit_ctx` line 579

```sql
SELECT
  (SELECT r.rolname FROM pg_roles r
    WHERE r.rolsuper = true AND r.rolname <> %s
      AND pg_has_role(%s, r.oid, 'USAGE')
    LIMIT 1) AS super_via,
  (SELECT r.rolname FROM pg_roles r
    WHERE {BYPASS_VIA} = true AND r.rolname <> %s
      AND pg_has_role(%s, r.oid, 'USAGE')
    LIMIT 1) AS bypass_via,
  (SELECT r.rolname FROM pg_roles r
    WHERE r.rolreplication = true AND r.rolname <> %s
      AND pg_has_role(%s, r.oid, 'USAGE')
    LIMIT 1) AS repl_via
```

`_build_role_audit_ctx` line 615

```sql
SELECT CASE
    WHEN passwd IS NULL                THEN NULL
    WHEN passwd LIKE 'md5%%'           THEN 'MD5'
    WHEN passwd LIKE 'SCRAM-SHA-256%%' THEN 'SCRAM-SHA-256'
    ELSE 'Other'
END AS m
FROM pg_shadow WHERE usename = %s
```

`_interp_db_priv` line 699

```sql
CREATE (can create schemas inside)
```

`_collect_role_findings` line 903

```sql
REVOKE the membership, or grant with INHERIT FALSE (PG16+), or NOINHERIT on '{p0}'
```

`_collect_role_findings` line 909

```sql
ALTER ROLE {p0} NOBYPASSRLS if not required
```

`_collect_role_findings` line 931

```sql
SET password_encryption = 'scram-sha-256'; then \password {p0} (or ALTER ROLE)
```

`_collect_role_findings` line 974

```sql
GRANT USAGE ON SCHEMA <schema> TO {p0} for each, or REVOKE the dangling table grants
```

## checks/roles_list.py

`_sql_roles_login` line 23

```sql
SELECT
    r.rolname AS username,
    shobj_description(r.oid, 'pg_authid') AS comment,
    CASE
        WHEN s.passwd IS NULL THEN 'No password (or not visible)'
        WHEN s.passwd LIKE 'md5%%' THEN 'MD5'
        WHEN s.passwd LIKE 'SCRAM-SHA-256%%' THEN 'SCRAM-SHA-256'
        ELSE 'Other'
    END AS auth_method,
    r.rolsuper      AS is_superuser,
    r.rolinherit    AS inherits,
    r.rolcreatedb   AS create_db,
    r.rolcreaterole AS create_role,
    r.rolreplication AS replication,
    {_bypass_rls_col(server_vnum)} AS bypass_rls,
    r.rolconnlimit  AS conn_limit,
    r.rolvaliduntil AS valid_until,
    -- A login user is still a role: other roles can be GRANTed into it.
    -- Surface its members so the UI shows the same expandable member
    -- list that group roles have.
    (SELECT count(*) FROM pg_auth_members am WHERE am.roleid = r.oid)::int AS direct_members,
    (SELECT array_agg(rolname ORDER BY rolname)
     FROM (
         SELECT m.rolname
         FROM pg_auth_members am
         JOIN pg_roles m ON m.oid = am.member
         WHERE am.roleid = r.oid
         ORDER BY m.rolname
         LIMIT 500
     ) cap) AS member_names
FROM pg_roles r
LEFT JOIN pg_shadow s ON s.usename = r.rolname
WHERE r.rolcanlogin = true
ORDER BY r.rolname
```

`_sql_roles_login_no_shadow` line 64

```sql
SELECT
    r.rolname AS username,
    shobj_description(r.oid, 'pg_authid') AS comment,
    '(restricted: pg_shadow not readable)' AS auth_method,
    r.rolsuper      AS is_superuser,
    r.rolinherit    AS inherits,
    r.rolcreatedb   AS create_db,
    r.rolcreaterole AS create_role,
    r.rolreplication AS replication,
    {_bypass_rls_col(server_vnum)} AS bypass_rls,
    r.rolconnlimit  AS conn_limit,
    r.rolvaliduntil AS valid_until,
    (SELECT count(*) FROM pg_auth_members am WHERE am.roleid = r.oid)::int AS direct_members,
    (SELECT array_agg(rolname ORDER BY rolname)
     FROM (
         SELECT m.rolname
         FROM pg_auth_members am
         JOIN pg_roles m ON m.oid = am.member
         WHERE am.roleid = r.oid
         ORDER BY m.rolname
         LIMIT 500
     ) cap) AS member_names
FROM pg_roles r
WHERE r.rolcanlogin = true
ORDER BY r.rolname
```

`_sql_roles_group` line 94

```sql
SELECT
    r.rolname AS role_name,
    shobj_description(r.oid, 'pg_authid') AS comment,
    -- Always false here by definition - this section is "no login" - but
    -- it is listed so the editor renders it as an editable true/false,
    -- which is the only way to turn a group role into a login role after
    -- the fact. Without it the change was one-way: login users have a
    -- Lock button that sets NOLOGIN, and nothing brought it back.
    r.rolcanlogin    AS can_login,
    r.rolsuper       AS is_superuser,
    r.rolinherit     AS inherits,
    r.rolinherit     AS member_role_inherit,
    r.rolcreatedb    AS create_db,
    r.rolcreaterole  AS create_role,
    r.rolreplication AS replication,
    {_bypass_rls_col(server_vnum)} AS bypass_rls,
    (SELECT count(*) FROM pg_auth_members am WHERE am.roleid = r.oid)::int AS direct_members,
    -- Per-role member list. The UI uses this for the inline expand
    -- under the "direct_members" count - each name becomes a clickable
    -- link to a role-perms audit. Cap inside a subquery (LIMIT on
    -- aggregate input rows, not on array_agg directly).
    (SELECT array_agg(rolname ORDER BY rolname)
     FROM (
         SELECT m.rolname
         FROM pg_auth_members am
         JOIN pg_roles m ON m.oid = am.member
         WHERE am.roleid = r.oid
         ORDER BY m.rolname
         LIMIT 500
     ) cap) AS member_names
FROM pg_roles r
WHERE r.rolcanlogin = false
ORDER BY r.rolname
```

**_SQL_ROLES_MEMBERSHIPS** - `(module)` line 134

```sql
SELECT
    m.rolname AS member,
    g.rolname AS granted_role,
    am.admin_option,
    m.rolinherit AS member_rolinherit,
    pg_has_role(m.rolname, g.oid, 'USAGE') AS effectively_inherits
FROM pg_auth_members am
JOIN pg_roles m ON m.oid = am.member
JOIN pg_roles g ON g.oid = am.roleid
ORDER BY m.rolname, g.rolname
```

`_sql_roles_memberships` line 151

```sql
SELECT
    m.rolname AS member,
    g.rolname AS granted_role,
    am.admin_option,
    {inh_col}
    pg_has_role(m.rolname, g.oid, 'USAGE') AS effectively_inherits
FROM pg_auth_members am
JOIN pg_roles m ON m.oid = am.member
JOIN pg_roles g ON g.oid = am.roleid
ORDER BY m.rolname, g.rolname
```

`run_roles_list_check` line 257

```sql
SELECT rolname FROM pg_roles
WHERE rolsuper = true AND rolcanlogin = true
  AND rolname !~ '^pg_'
ORDER BY rolname
```

`run_roles_list_check` line 279

```sql
SELECT rolname FROM pg_roles
WHERE rolbypassrls = true AND rolsuper = false
  AND rolname !~ '^pg_'
ORDER BY rolname
```

`run_roles_list_check` line 298

```sql
SELECT rolname FROM pg_roles
WHERE rolcreaterole = true AND rolsuper = false
  AND rolname !~ '^pg_'
ORDER BY rolname
```

`run_roles_list_check` line 316

```sql
SELECT rolname FROM pg_roles
WHERE rolcanlogin = true
  AND rolconnlimit = -1
  AND rolvaliduntil IS NULL
  AND rolname !~ '^pg_'
  AND rolname <> current_user
ORDER BY rolname
```

`run_roles_list_check` line 337

```sql
SELECT rolname,
       to_char(rolvaliduntil, 'YYYY-MM-DD') AS expired_at
FROM pg_roles
WHERE rolcanlogin = true
  AND rolvaliduntil IS NOT NULL
  AND rolvaliduntil < now()
  AND rolname !~ '^pg_'
ORDER BY rolvaliduntil
```

`run_roles_list_check` line 360

```sql
SELECT coalesce((SELECT setting::bigint FROM pg_settings
                  WHERE name = 'password_expiration_warning_threshold'), 604800) AS s
```

`run_roles_list_check` line 365

```sql
SELECT rolname, to_char(rolvaliduntil, 'YYYY-MM-DD HH24:MI') AS expires_at
FROM pg_roles
WHERE rolcanlogin = true
  AND rolvaliduntil IS NOT NULL
  AND rolvaliduntil >= now()
  AND rolvaliduntil < now() + %s * interval '1 second'
  AND rolname !~ '^pg_'
ORDER BY rolvaliduntil
```

`run_roles_list_check` line 378

```sql
ALTER ROLE ... VALID UNTIL '...' (or set a new password) before it lapses and the login stops working
```

`run_roles_list_check` line 397

```sql
SELECT rolname FROM pg_roles
WHERE rolreplication = true AND rolcanlogin = true
  AND rolsuper = false
  AND rolname !~ '^pg_'
ORDER BY rolname
```

`run_roles_list_check` line 416

```sql
SELECT d.datname
FROM pg_database d
WHERE d.datistemplate = false
  AND has_database_privilege('public', d.oid, 'CONNECT')
ORDER BY d.datname
```

`run_roles_list_check` line 429

```sql
REVOKE CONNECT ON DATABASE x FROM PUBLIC; then GRANT to specific roles
```

`run_roles_list_check` line 440

```sql
SELECT n.nspname
FROM pg_namespace n
WHERE n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
  AND has_schema_privilege('public', n.oid, 'CREATE')
ORDER BY n.nspname
```

`run_roles_list_check` line 453

```sql
REVOKE CREATE ON SCHEMA {s} FROM PUBLIC;
```

`run_roles_list_check` line 462

```sql
SELECT n.nspname AS schema,
       p.proname AS function,
       pg_get_userbyid(p.proowner) AS owner
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
JOIN pg_roles ow ON ow.oid = p.proowner
WHERE p.prosecdef = true
  AND ow.rolsuper = true
  AND has_function_privilege('public', p.oid, 'EXECUTE')
  AND n.nspname NOT LIKE 'pg_%%'
  AND n.nspname <> 'information_schema'
ORDER BY n.nspname, p.proname
LIMIT 20
```

`run_roles_list_check` line 481

```sql
REVOKE EXECUTE ... FROM PUBLIC and grant only to the roles that need it; pin search_path in the function body
```

## checks/seq_capacity.py

**_FED_SQL** - `(module)` line 39

```sql
SELECT d.objid AS seq_oid, t.oid AS table_oid, a.attnum,
       tn.nspname AS table_schema, t.relname AS table_name, a.attname AS column_name,
       format_type(a.atttypid, NULL) AS column_type
  FROM pg_depend d
  JOIN pg_class s      ON s.oid = d.objid AND s.relkind = 'S'
  JOIN pg_class t      ON t.oid = d.refobjid
  JOIN pg_namespace tn ON tn.oid = t.relnamespace
  JOIN pg_attribute a  ON a.attrelid = t.oid AND a.attnum = d.refobjsubid
 WHERE d.classid = 'pg_class'::regclass AND d.refclassid = 'pg_class'::regclass
   AND d.deptype IN ('a', 'i') AND d.refobjsubid > 0
UNION
SELECT dep.refobjid, t.oid, a.attnum, tn.nspname, t.relname, a.attname,
       format_type(a.atttypid, NULL)
  FROM pg_depend dep
  JOIN pg_class s      ON s.oid = dep.refobjid AND s.relkind = 'S'
  JOIN pg_attrdef ad   ON ad.oid = dep.objid
  JOIN pg_attribute a  ON a.attrelid = ad.adrelid AND a.attnum = ad.adnum
  JOIN pg_class t      ON t.oid = ad.adrelid
  JOIN pg_namespace tn ON tn.oid = t.relnamespace
 WHERE dep.classid = 'pg_attrdef'::regclass AND dep.refclassid = 'pg_class'::regclass
```

**_FK_SQL** - `(module)` line 63

```sql
SELECT c.confrelid AS ref_oid, c.confkey[1] AS ref_attnum,
       tn.nspname AS table_schema, t.relname AS table_name, a.attname AS column_name,
       format_type(a.atttypid, NULL) AS column_type
  FROM pg_constraint c
  JOIN pg_class t      ON t.oid = c.conrelid
  JOIN pg_namespace tn ON tn.oid = t.relnamespace
  JOIN pg_attribute a  ON a.attrelid = c.conrelid AND a.attnum = c.conkey[1]
 WHERE c.contype = 'f' AND array_length(c.conkey, 1) = 1
```

**_SEQ_SQL** - `(module)` line 77

```sql
SELECT c.oid AS seq_oid, n.nspname AS schema, c.relname AS sequence,
       format_type(s.seqtypid, NULL) AS seq_type,
       s.seqstart AS start_value, s.seqmin AS min_value, s.seqmax AS max_value,
       s.seqincrement AS increment, s.seqcycle AS cycles,
       CASE WHEN has_sequence_privilege(c.oid, 'SELECT,USAGE')
            THEN pg_sequence_last_value(c.oid) END AS last_value
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_sequence s  ON s.seqrelid = c.oid
 WHERE c.relkind = 'S' AND n.nspname NOT IN ('pg_catalog', 'information_schema')
```

**_PRE10_LIST_SQL** - `(module)` line 94

```sql
SELECT c.oid AS seq_oid, n.nspname AS schema, c.relname AS sequence
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.relkind = 'S' AND n.nspname NOT IN ('pg_catalog', 'information_schema')
   -- CASE: the planner may test this before relkind, and on a table it errors
   AND CASE WHEN c.relkind = 'S' THEN has_sequence_privilege(c.oid, 'SELECT') ELSE false END
```

`_ver` line 113

```sql
SELECT current_setting('server_version_num')::int AS v
```

`_sequences` line 140

```sql
SELECT %d::oid AS seq_oid, 'bigint'::text AS seq_type, start_value, min_value, max_value, increment_by AS increment, is_cycled AS cycles, CASE WHEN is_called THEN last_value END AS last_value FROM %s.%s
```

`_table_rates` line 169

```sql
SELECT s.relid AS table_oid, s.n_tup_ins AS tup_ins, {reset_cols}
       extract(epoch FROM (now() - LEAST(s.last_vacuum, s.last_autovacuum,
                                         s.last_analyze, s.last_autoanalyze)))::float8 AS oldest_mark_s
  FROM pg_stat_user_tables s
 WHERE s.relid = ANY(%s)
```

## checks/seq_sync.py

`run_seq_sync_check` line 16

```sql
SELECT schemaname || '.' || sequencename AS sequence,
       last_value, start_value,
       CASE
           WHEN last_value IS NULL
               /* format('%I.%I', ...) double-quotes only when needed - 
                  the generated setval(...) line works for both lowercase
                  ("public"."seq" stays unquoted internally) and case-
                  sensitive ("MySchema"."MySeq") names without breakage. */
               THEN format('SELECT setval(%L::regclass, %s, false);',
                           format('%I.%I', schemaname, sequencename), start_value)
           ELSE format('SELECT setval(%L::regclass, %s);',
                       format('%I.%I', schemaname, sequencename), last_value)
       END AS setval_command
FROM pg_sequences
WHERE schemaname NOT IN ('pg_catalog','information_schema')
  AND schemaname NOT LIKE 'pg_toast%%'
  AND schemaname NOT LIKE 'pg_temp_%%'
ORDER BY schemaname, sequencename
```

`run_seq_sync_check` line 45

```sql
SELECT n.nspname AS schemaname, c.relname AS sequencename
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind = 'S'
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
  AND n.nspname NOT LIKE 'pg_temp_%%'
ORDER BY n.nspname, c.relname
```

`run_seq_sync_check` line 64

```sql
SELECT last_value, start_value, is_called FROM {fq}
```

**cmd** - `run_seq_sync_check` line 76

```sql
SELECT setval('{fq_lit}'::regclass, {int(sv) if sv is not None else 1}, false);
```

**cmd** - `run_seq_sync_check` line 78

```sql
SELECT setval('{fq_lit}'::regclass, {int(lv)});
```

## checks/seq_usage.py

**_SQL_SEQ_USAGE** - `(module)` line 20

```sql
SELECT
    sn.nspname                                              AS schema,
    sc.relname                                              AS sequence,
    COALESCE(NULLIF(ts.spcname, ''), 'pg_default')          AS tablespace,
    CASE
        WHEN sc.reltablespace = 0
            THEN '(database default - see pg_database.dattablespace)'
        ELSE COALESCE(NULLIF(pg_tablespace_location(sc.reltablespace), ''),
                      '(under data_directory)')
    END                                                     AS location,
    format_type(s.seqtypid, NULL)                           AS data_type,
    s.seqstart                                              AS start,
    s.seqmin                                                AS min_value,
    s.seqmax                                                AS max_value,
    s.seqincrement                                          AS increment,
    s.seqcycle                                              AS cycles,
    (CASE WHEN has_sequence_privilege(sc.oid, 'SELECT, USAGE') THEN pg_sequence_last_value(sc.oid) END)                          AS current_value,
    -- Consumption measured across the sequence's actual range and in its
    -- actual direction. last_value / seqmax was wrong three ways, all
    -- reproduced on a live server:
    --   * a CYCLE sequence at 95 of 100 reported 95% used, when it wraps
    --     rather than failing and can never run out at all;
    --   * a descending sequence (INCREMENT -1, range -1 .. -100) sitting
    --     at -50 reported 5000%, having consumed half its range;
    --   * a sequence spanning -1000 .. 1000 sitting at 0 reported 0%,
    --     having also consumed half. That one is the dangerous direction:
    --     the alarm was silent, not noisy.
    CASE
        WHEN (CASE WHEN has_sequence_privilege(sc.oid, 'SELECT, USAGE') THEN pg_sequence_last_value(sc.oid) END) IS NULL THEN NULL
        WHEN s.seqcycle THEN NULL          -- wraps; exhaustion is not a thing
        WHEN s.seqmax = s.seqmin THEN NULL -- degenerate, avoid /0
        WHEN s.seqincrement > 0 THEN
            round(100.0 * ((CASE WHEN has_sequence_privilege(sc.oid, 'SELECT, USAGE') THEN pg_sequence_last_value(sc.oid) END)::numeric - s.seqmin)
                         / (s.seqmax::numeric - s.seqmin), 2)
        ELSE
            round(100.0 * (s.seqmax::numeric - (CASE WHEN has_sequence_privilege(sc.oid, 'SELECT, USAGE') THEN pg_sequence_last_value(sc.oid) END))
                         / (s.seqmax::numeric - s.seqmin), 2)
    END                                                     AS used_pct,
    -- The owning column. A SERIAL column (or OWNED BY) records the
    -- dependency as deptype 'a'; an IDENTITY column (PostgreSQL 10+)
    -- records it as 'i' and has no DEFAULT expression at all. With 'a'
    -- alone every identity sequence in a modern schema read as an
    -- orphan.
    (SELECT t_ns.nspname || '.' || t.relname || '.' || a.attname
       FROM pg_depend d
       JOIN pg_class t        ON t.oid = d.refobjid
       JOIN pg_namespace t_ns ON t_ns.oid = t.relnamespace
       JOIN pg_attribute a    ON a.attrelid = d.refobjid AND a.attnum = d.refobjsubid
      WHERE d.classid     = 'pg_class'::regclass
        AND d.objid       = sc.oid
        AND d.refclassid  = 'pg_class'::regclass
        AND d.deptype     IN ('a', 'i')
      LIMIT 1)                                              AS owned_by_column,
    (SELECT CASE d.deptype WHEN 'i' THEN 'identity' ELSE 'owned by' END
       FROM pg_depend d
      WHERE d.classid     = 'pg_class'::regclass
        AND d.objid       = sc.oid
        AND d.refclassid  = 'pg_class'::regclass
        AND d.refobjsubid > 0
        AND d.deptype     IN ('a', 'i')
      LIMIT 1)                                              AS ownership,
    (SELECT string_agg(t_ns.nspname || '.' || t.relname || '.' || a.attname, ', '
                       ORDER BY t_ns.nspname, t.relname, a.attname)
       FROM pg_depend dep
       JOIN pg_attrdef ad   ON ad.oid = dep.objid
       JOIN pg_attribute a  ON a.attrelid = ad.adrelid AND a.attnum = ad.adnum
       JOIN pg_class t      ON t.oid = ad.adrelid
       JOIN pg_namespace t_ns ON t_ns.oid = t.relnamespace
      WHERE dep.classid    = 'pg_attrdef'::regclass
        AND dep.refclassid = 'pg_class'::regclass
        AND dep.refobjid   = sc.oid
        AND dep.deptype IN ('n','a'))                       AS used_in_columns,
    pg_get_userbyid(sc.relowner)                            AS owner
FROM pg_class sc
JOIN pg_namespace sn ON sn.oid = sc.relnamespace
JOIN pg_sequence s   ON s.seqrelid = sc.oid
LEFT JOIN pg_tablespace ts ON ts.oid = sc.reltablespace
WHERE sc.relkind = 'S'
  AND sn.nspname NOT IN ('pg_catalog','information_schema')
ORDER BY sn.nspname, sc.relname
```

`run_seq_usage_check` line 230

```sql
ALTER SEQUENCE s AS bigint MAXVALUE 9223372036854775807;
```

## checks/sequences.py

`unreadable_sequences` line 10

```sql
SELECT n.nspname || '.' || c.relname AS s
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.relkind = 'S' AND n.nspname NOT IN ('pg_catalog', 'information_schema')
   -- In a CASE: the planner may test the WHERE terms in any order,
   -- and has_sequence_privilege() on a view is an error.
   AND CASE WHEN c.relkind = 'S' THEN NOT has_sequence_privilege(c.oid, 'SELECT, USAGE') END
 ORDER BY 1
```

`run_sequences_check` line 37

```sql
SELECT schemaname || '.' || sequencename                  AS sequence,
       start_value                                        AS start,
       min_value                                          AS min_value,
       max_value                                          AS max_value,
       increment_by                                       AS increment,
       cycle                                              AS cycles,
       last_value                                         AS current,
       -- Measured across the sequence's actual range and in
       -- its actual direction. last_value / max_value is
       -- wrong three ways, all reproducible:
       --   * a CYCLE sequence at 95 of 100 read 95% used,
       --     when it wraps and can never run out at all;
       --   * a descending sequence over -1 .. -100 sitting
       --     at -50 read 5000%, and sorted itself to the top
       --     of the list as the most urgent thing on it;
       --   * a sequence over -1000 .. 1000 sitting at 0 read
       --     0%, having consumed half its range. That is the
       --     dangerous direction: the alarm was silent.
       -- db-seq-usage already computes it this way; this
       -- check is the other half of the same question and
       -- was still using the old formula.
       CASE
           WHEN last_value IS NULL THEN NULL
           WHEN cycle THEN NULL
           WHEN max_value = min_value THEN NULL
           WHEN increment_by > 0 THEN
               round(100.0 * (last_value::numeric - min_value)
                            / (max_value::numeric - min_value), 2)
           ELSE
               round(100.0 * (max_value::numeric - last_value)
                            / (max_value::numeric - min_value), 2)
       END                                                AS used_pct
FROM pg_sequences
WHERE schemaname NOT IN ('pg_catalog','information_schema')
  AND last_value IS NOT NULL
ORDER BY used_pct DESC NULLS LAST, sequence
LIMIT 50
```

`run_sequences_check` line 85

```sql
SELECT n.nspname AS schemaname, c.relname AS sequencename
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind = 'S'
  AND n.nspname NOT IN ('pg_catalog','information_schema')
ORDER BY n.nspname, c.relname
```

`run_sequences_check` line 102

```sql
SELECT last_value, start_value, min_value, max_value, increment_by, is_cycled FROM {fq}
```

`run_sequences_check` line 196

```sql
ALTER SEQUENCE s AS bigint MAXVALUE 9223372036854775807;
```

## checks/setup_advisor.py

`_setting_dict` line 60

```sql
SELECT name, setting, unit, context FROM pg_settings WHERE name = ANY(%s)
```

`_has_extension` line 72

```sql
SELECT 1 FROM pg_extension WHERE extname = %s
```

`_has_extension` line 77

```sql
SELECT 1 FROM pg_available_extensions WHERE name = %s
```

`_has_role_membership` line 112

```sql
SELECT 1 FROM pg_roles WHERE rolname = %s
```

`_has_role_membership` line 115

```sql
SELECT pg_has_role(current_user, %s, 'USAGE') AS ok
```

`_is_super` line 124

```sql
SELECT rolsuper FROM pg_roles WHERE rolname = current_user
```

`run_setup_advisor_check.spl_fix` line 253

```sql
ALTER SYSTEM SET shared_preload_libraries = {guc_value_sql('shared_preload_libraries', new)}; then RESTART the cluster (this GUC is postmaster-only).
```

`run_setup_advisor_check` line 267

```sql
CREATE EXTENSION pg_stat_statements;
```

`run_setup_advisor_check` line 378

```sql
CREATE EXTENSION pg_buffercache;
```

`run_setup_advisor_check` line 392

```sql
CREATE EXTENSION pg_repack; (requires superuser). Also install the pg_repack client binary on the host running DBApguin.
```

`run_setup_advisor_check` line 430

```sql
GRANT pg_read_server_files TO <role>;  -- or connect as a superuser.
```

`run_setup_advisor_check` line 499

```sql
GRANT pg_monitor TO <your_role>;  -- includes pg_read_all_stats, pg_read_all_settings.
```

## checks/stat_statements.py

`run_stat_statements_check` line 42

```sql
SELECT stats_reset FROM {_pss_info}
```

`run_stat_statements_check` line 52

```sql
SELECT count(*) FROM {_pss}
```

`run_stat_statements_check` line 89

```sql
SELECT column_name FROM information_schema.columns WHERE table_name='pg_stat_statements'
```

`run_stat_statements_check` line 172

```sql
SELECT setting::bigint * CASE unit WHEN 'kB' THEN 1024 WHEN 'MB' THEN 1048576 ELSE 1 END AS v FROM pg_settings WHERE name='work_mem'
```

`run_stat_statements_check` line 283

```sql
SELECT queryid AS qid, query, calls, rows, {time_total} AS total_ms
  FROM {_pss}
 WHERE {_ol.SQL_FILTER}
 ORDER BY calls DESC LIMIT 500
```

`run_stat_statements_check` line 324

```sql
UPDATE and DELETE statements whose WHERE compares a version column (version, lock_version, xmin, a rowversion or ConcurrencyStamp, updated_at...) with a parameter. A call that wrote no row lost the race: someone changed the row (or deleted it) after it was read, and the application reloads and retries. stale_% = 1 - rows / calls since the statistics were reset.
```

## checks/table_stats.py

**sql** - `run_table_stats_check` line 21

```sql
SELECT t.schemaname || '.' || t.relname                       AS table_name,
       pg_size_pretty(pg_total_relation_size(t.relid))        AS total_size,
       t.seq_scan                                             AS seq_scan_cnt,
       t.idx_scan                                             AS idx_scan_cnt,
{extra_cols}       to_char(t.last_analyze,     'YYYY-MM-DD HH24:MI:SS')   AS last_analyze,
       to_char(t.last_autoanalyze, 'YYYY-MM-DD HH24:MI:SS')   AS last_autoanalyze,
       t.autoanalyze_count,
       t.n_mod_since_analyze                                  AS mods_since_analyze,
       CASE WHEN c.reltuples > 0
            THEN round(t.n_mod_since_analyze::numeric / c.reltuples::numeric * 100, 1)
       END                                                    AS mod_pct
FROM pg_stat_all_tables t
JOIN pg_class c ON c.oid = t.relid
WHERE t.schemaname NOT IN ('pg_catalog','information_schema','pg_toast')
  AND t.n_mod_since_analyze > 0
ORDER BY {'t.last_seq_scan DESC NULLS LAST' if has_last_scan else 't.seq_scan DESC'}
LIMIT 100
```

## checks/tablespaces.py

`run_tablespaces_check` line 19

```sql
SELECT
    t.spcname                                                  AS name,
    pg_get_userbyid(t.spcowner)                                AS owner,
    CASE WHEN t.spcname IN ('pg_default','pg_global')
         THEN '(under data_directory: ' ||
              COALESCE((SELECT setting FROM pg_settings WHERE name='data_directory'), 'data_directory') ||
              CASE WHEN t.spcname='pg_global' THEN '/global)' ELSE '/base)' END
         ELSE COALESCE(NULLIF(pg_tablespace_location(t.oid), ''), '(unknown)')
    END                                                        AS location,
    -- The size needs CREATE on the tablespace, pg_read_all_stats, or
    -- being this database's default; without it the whole section
    -- was lost to "permission denied for tablespace pg_global".
    CASE WHEN (has_tablespace_privilege(t.oid, 'CREATE') OR pg_has_role(current_user, 'pg_read_all_stats', 'USAGE') OR t.oid = (SELECT dattablespace FROM pg_database WHERE datname = current_database())) THEN pg_size_pretty(pg_tablespace_size(t.oid)) END AS size,
    CASE WHEN (has_tablespace_privilege(t.oid, 'CREATE') OR pg_has_role(current_user, 'pg_read_all_stats', 'USAGE') OR t.oid = (SELECT dattablespace FROM pg_database WHERE datname = current_database())) THEN pg_tablespace_size(t.oid) END        AS size_bytes,
    t.spcoptions                                               AS options,
    t.spcacl::text                                             AS acl
FROM pg_tablespace t
ORDER BY CASE WHEN (has_tablespace_privilege(t.oid, 'CREATE') OR pg_has_role(current_user, 'pg_read_all_stats', 'USAGE') OR t.oid = (SELECT dattablespace FROM pg_database WHERE datname = current_database())) THEN pg_tablespace_size(t.oid) END DESC NULLS LAST
```

`run_tablespaces_check` line 58

```sql
SELECT
    t.spcname                              AS tablespace,
    d.datname                              AS database
FROM pg_tablespace t
CROSS JOIN LATERAL pg_tablespace_databases(t.oid) AS db_oid(oid)
JOIN pg_database d ON d.oid = db_oid.oid
ORDER BY t.spcname, d.datname
```

`run_tablespaces_check` line 85

```sql
WITH this_db AS (
    SELECT dattablespace AS default_tsoid
    FROM pg_database
    WHERE datname = current_database()
)
SELECT
    COALESCE(NULLIF(ts.spcname, ''),
             (SELECT spcname FROM pg_tablespace
              WHERE oid = (SELECT default_tsoid FROM this_db))) AS tablespace,
    n.nspname                                                   AS schema,
    c.relname                                                   AS object,
    CASE c.relkind
        WHEN 'r' THEN 'table'
        WHEN 'p' THEN 'partitioned table'
        WHEN 'i' THEN 'index'
        WHEN 'I' THEN 'partitioned index'
        WHEN 'S' THEN 'sequence'
        WHEN 'm' THEN 'matview'
        WHEN 't' THEN 'toast'
        WHEN 'v' THEN 'view'
        WHEN 'f' THEN 'foreign'
        ELSE c.relkind::text
    END                                                         AS kind,
    pg_get_userbyid(c.relowner)                                 AS owner,
    CASE WHEN c.relkind IN ('r','p','m','i','I','t','S')
         THEN pg_size_pretty(pg_relation_size(c.oid))
         ELSE NULL
    END                                                         AS size,
    CASE WHEN c.relkind IN ('r','p','m','i','I','t','S')
         THEN pg_relation_size(c.oid)
         ELSE NULL
    END                                                         AS size_bytes,
    CASE WHEN c.reltablespace = 0 THEN 'inherited (db default)'
         ELSE 'explicit'
    END                                                         AS placement
FROM pg_class c
JOIN pg_namespace n  ON n.oid = c.relnamespace
LEFT JOIN pg_tablespace ts ON ts.oid = NULLIF(c.reltablespace, 0)
WHERE c.relkind IN ('r','p','m','i','I','S','t','f')
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%'
ORDER BY tablespace, n.nspname, c.relname
```

## checks/temp_files.py

`_tablespaces_to_scan` line 45

```sql
SELECT oid, spcname
  FROM pg_tablespace
 WHERE spcname NOT IN ('pg_default', 'pg_global')
 ORDER BY spcname
```

`_read_temp_files` line 100

```sql
SELECT name, size, modification FROM pg_ls_tmpdir(%s)
```

`_activity_for` line 160

```sql
SELECT pid, {leader}, {btype},
       usename, datname, application_name,
       COALESCE(host(client_addr), 'local') AS client,
       state, wait_event_type, wait_event,
       query,
       EXTRACT(epoch FROM now() - query_start)::bigint AS query_age_s,
       EXTRACT(epoch FROM now() - xact_start)::bigint  AS xact_age_s
  FROM pg_stat_activity
 WHERE pid = ANY(%s)
```

`_relations_for` line 191

```sql
SELECT l.pid,
       n.nspname AS schema,
       c.relname AS name,
       c.relkind,
       l.mode
  FROM pg_locks l
  JOIN pg_class c     ON c.oid = l.relation
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE l.pid = ANY(%s)
   AND l.locktype = 'relation'
   AND c.relkind IN ('r', 'p', 'm', 'f', 'v')
   AND n.nspname NOT IN ('pg_catalog', 'information_schema')
 ORDER BY l.pid, (l.mode <> 'AccessShareLock') DESC, n.nspname, c.relname
```

`_work_mem` line 363

```sql
SELECT name,
       setting,
       unit,
       setting || COALESCE(' ' || unit, '') AS pretty,
       -- Guarded, because this list is not all numbers:
       -- temp_tablespaces is a name list and is usually empty,
       -- and setting::bigint on '' fails - taking the whole
       -- query with it and leaving the panel with no settings
       -- at all rather than one missing row.
       CASE WHEN setting ~ '^-?[0-9]+$' THEN
         CASE unit WHEN 'kB'  THEN setting::bigint * 1024
                   WHEN 'MB'  THEN setting::bigint * 1048576
                   WHEN '8kB' THEN setting::bigint * 8192
                   ELSE setting::bigint END
       END AS bytes
  FROM pg_settings
 WHERE name IN ('work_mem', 'hash_mem_multiplier',
                'maintenance_work_mem', 'temp_file_limit',
                'temp_tablespaces', 'log_temp_files')
```

`(module)` line 430

```sql
CREATE INDEX, REINDEX, CLUSTER, ALTER TABLE rewrites and VACUUM are bounded by maintenance_work_mem, never by work_mem. A spill during an index build is not a work_mem problem and raising work_mem does nothing for it.
```

## checks/temp_queries.py

`run_temp_queries_check` line 76

```sql
SELECT setting::bigint *
       CASE unit WHEN 'kB' THEN 1024 WHEN 'MB' THEN 1048576 ELSE 1 END AS bytes,
       setting || COALESCE(' ' || unit, '') AS pretty
  FROM pg_settings WHERE name='work_mem'
```

## checks/trace_session.py

`_resolve_targets` line 82

```sql
SELECT pid FROM pg_stat_activity
 WHERE application_name ~ %s AND pid <> pg_backend_pid()
```

`_resolve_targets` line 87

```sql
SELECT pid FROM pg_stat_activity
 WHERE usename = %s AND pid <> pg_backend_pid()
```

`run_trace_session_check` line 242

```sql
SELECT name, setting
  FROM pg_settings
 WHERE name IN ('auto_explain.log_min_duration',
                'auto_explain.log_analyze',
                'auto_explain.log_buffers',
                'auto_explain.log_timing',
                'auto_explain.log_format',
                'auto_explain.log_nested_statements',
                'shared_preload_libraries')
```

`run_trace_session_check` line 281

```sql
SELECT 1 FROM {_pss} LIMIT 1
```

`run_trace_session_check` line 284

```sql
SELECT queryid, query, calls,
       %(total)s AS total_exec_time,
       shared_blks_read, temp_blks_written, rows AS rows_total
  FROM
```

**_sample_sql** - `run_trace_session_check` line 305

```sql
SELECT pid, state, {_qid_col}, wait_event_type, wait_event,
       regexp_replace(query, '[[:space:]]+', ' ', 'g') AS query
  FROM pg_stat_activity
 WHERE pid = ANY(%s)
```

`run_trace_session_check` line 446

```sql
SELECT queryid, query, calls,
       %(total)s AS total_exec_time,
       shared_blks_read, temp_blks_written, rows AS rows_total
  FROM
```

## checks/tuning_advice.py

`_read_file` line 117

```sql
SELECT pg_read_file(%s, 0, 1048576, true) AS t
```

`facts` line 159

```sql
SELECT name, setting, unit, context, source FROM pg_settings WHERE name = ANY(%s)
```

`facts` line 192

```sql
SELECT coalesce(sum(temp_files), 0) AS temp_files,
                            coalesce(sum(temp_bytes), 0) AS temp_bytes,
                            extract(epoch FROM now() - min(stats_reset)) AS age_s
                       FROM pg_stat_database
```

`facts` line 199

```sql
SELECT count(*) AS n, count(*) FILTER (WHERE state = 'active') AS active
                          FROM pg_stat_activity WHERE backend_type = 'client backend'
```

`facts` line 212

```sql
SELECT s.query, s.calls, s.temp_blks_written, d.datname AS db, r.rolname AS "user"
                            FROM {pss} s
                            LEFT JOIN pg_database d ON d.oid = s.dbid
                            LEFT JOIN pg_roles r ON r.oid = s.userid
                           WHERE s.temp_blks_written > 0 AND s.calls > 0
                           ORDER BY s.temp_blks_written DESC LIMIT 10
```

`facts` line 226

```sql
SELECT num_timed AS timed, num_requested AS req,
                                extract(epoch FROM now() - stats_reset) AS age_s
                           FROM pg_stat_checkpointer
```

`facts` line 230

```sql
SELECT checkpoints_timed AS timed, checkpoints_req AS req,
                                extract(epoch FROM now() - stats_reset) AS age_s
                           FROM pg_stat_bgwriter
```

`facts` line 240

```sql
SELECT wal_bytes, extract(epoch FROM now() - stats_reset) AS age_s FROM pg_stat_wal
```

`facts` line 248

```sql
SELECT buffers_clean, maxwritten_clean FROM pg_stat_bgwriter
```

`facts` line 253

```sql
SELECT coalesce(sum(writes), 0) AS w FROM pg_stat_io
                           WHERE backend_type = 'client backend' AND object = 'relation'
                             AND context = 'normal'
```

`facts` line 259

```sql
SELECT buffers_backend FROM pg_stat_bgwriter
```

`facts` line 271

```sql
SELECT coalesce(sum(reads), 0)::float8 AS n, coalesce(sum(read_time), 0)::float8 AS ms
                            FROM pg_stat_io WHERE object = 'relation'
```

## checks/upgrade_readiness.py

`run_upgrade_readiness_check._mule` line 87

```sql
SELECT datname FROM pg_database WHERE pg_encoding_to_char(encoding) = 'MULE_INTERNAL' ORDER BY 1
```

`run_upgrade_readiness_check._crlf` line 98

```sql
SELECT datname AS n FROM pg_database
```

`run_upgrade_readiness_check._crlf` line 99

```sql
SELECT rolname AS n FROM pg_roles
```

`run_upgrade_readiness_check._crlf` line 100

```sql
SELECT spcname AS n FROM pg_tablespace
```

`run_upgrade_readiness_check._gist_inet` line 113

```sql
SELECT n.nspname AS schema, ic.relname AS index, tc.relname AS "table", opc.opcname
  FROM pg_index i
  JOIN pg_class ic ON ic.oid = i.indexrelid
  JOIN pg_class tc ON tc.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = ic.relnamespace
  JOIN LATERAL unnest(i.indclass::oid[]) AS u(opc) ON true
  JOIN pg_opclass opc ON opc.oid = u.opc
 WHERE opc.opcname IN ('gist_inet_ops', 'gist_cidr_ops')
 ORDER BY 1, 2
```

`run_upgrade_readiness_check._radius` line 139

```sql
SELECT line_number FROM pg_hba_file_rules WHERE auth_method = 'radius' ORDER BY 1
```

`run_upgrade_readiness_check._esc` line 155

```sql
SELECT sourcefile, sourceline FROM pg_file_settings WHERE name = 'escape_string_warning'
```

`run_upgrade_readiness_check._esc` line 157

```sql
SELECT coalesce(d.datname, '*') AS db, coalesce(r.rolname, '*') AS role
  FROM pg_db_role_setting s
  LEFT JOIN pg_database d ON d.oid = s.setdatabase
  LEFT JOIN pg_roles r ON r.oid = s.setrole
 WHERE EXISTS (SELECT 1 FROM unnest(s.setconfig) c WHERE c LIKE 'escape_string_warning=%')
```

`run_upgrade_readiness_check._esc` line 163

```sql
ALTER ROLE/DATABASE SET (db {x['db']}, role {x['role']})
```

`run_upgrade_readiness_check._scs` line 174

```sql
SHOW standard_conforming_strings
```

`run_upgrade_readiness_check._scs` line 176

```sql
SELECT coalesce(d.datname, '*') AS db, coalesce(r.rolname, '*') AS role
  FROM pg_db_role_setting s
  LEFT JOIN pg_database d ON d.oid = s.setdatabase
  LEFT JOIN pg_roles r ON r.oid = s.setrole
 WHERE EXISTS (SELECT 1 FROM unnest(s.setconfig) c WHERE c = 'standard_conforming_strings=off')
```

`run_upgrade_readiness_check._md5` line 192

```sql
SELECT count(*) AS n FROM pg_authid WHERE rolpassword LIKE 'md5%'
```

`run_upgrade_readiness_check._locks` line 204

```sql
SELECT setting, source FROM pg_settings WHERE name = 'max_locks_per_transaction'
```

`run_upgrade_readiness_check._jit` line 216

```sql
SELECT setting, source FROM pg_settings WHERE name = 'jit'
```

`run_upgrade_readiness_check._jit` line 223

```sql
SELECT 1 FROM pg_attribute
 WHERE attrelid = to_regclass('pg_stat_statements') AND attname = 'jit_functions' AND NOT attisdropped
   AND EXISTS (SELECT 1 FROM pg_settings WHERE name = 'pg_stat_statements.max')
```

`run_upgrade_readiness_check._jit` line 229

```sql
SELECT sum(jit_functions)::bigint AS n FROM pg_stat_statements
```

`run_upgrade_readiness_check._ext` line 244

```sql
SELECT extname, extversion FROM pg_extension ORDER BY 1
```

## checks/vacuum.py

`run_vacuum_check` line 74

```sql
SELECT name, setting
  FROM pg_settings
 WHERE name IN ('autovacuum', 'autovacuum_vacuum_scale_factor',
                'autovacuum_vacuum_threshold',
                'autovacuum_analyze_scale_factor',
                'autovacuum_analyze_threshold',
                'autovacuum_vacuum_insert_scale_factor',
                'autovacuum_vacuum_insert_threshold',
                'autovacuum_freeze_min_age',
                'autovacuum_freeze_max_age',
                'autovacuum_vacuum_cost_limit',
                'autovacuum_vacuum_cost_delay',
                'vacuum_cost_limit',
                'vacuum_cost_delay',
                'vacuum_freeze_table_age',
                'autovacuum_max_parallel_workers',
                'vacuum_cost_page_hit',
                'vacuum_cost_page_miss',
                'vacuum_cost_page_dirty',
                'autovacuum_vacuum_max_threshold',
                'autovacuum_max_workers',
                'autovacuum_worker_slots',
                'autovacuum_naptime',
                'full_page_writes',
                'wal_compression',
                'checkpoint_timeout')
```

`run_vacuum_check` line 116

```sql
SELECT setting::bigint AS v FROM pg_settings WHERE name='autovacuum_freeze_max_age'
```

`run_vacuum_check` line 127

```sql
SELECT setting FROM pg_settings WHERE name='track_counts'
```

`run_vacuum_check` line 148

```sql
SELECT (schemaname || '.' || relname) AS fq {extra}
  FROM pg_stat_progress_vacuum p
  JOIN pg_stat_user_tables  s ON s.relid = p.relid
```

`run_vacuum_check` line 176

```sql
SELECT s.schemaname AS schema,
       s.relname    AS table,
       s.n_live_tup AS live,
       s.n_dead_tup AS dead,
       s.n_mod_since_analyze AS mods_since_analyze,
       s.n_tup_ins  AS tup_ins,
       s.n_tup_upd  AS tup_upd,
       s.n_tup_del  AS tup_del,
       s.n_tup_hot_upd AS tup_hot_upd,
       {ins_col}
       s.vacuum_count, s.autovacuum_count,
       s.analyze_count, s.autoanalyze_count,
       {time_cols}
       s.last_vacuum, s.last_autovacuum,
       s.last_analyze, s.last_autoanalyze,
       {tbl_reset_cols}
       -- the oldest mark a reset would have cleared (table_window)
       extract(epoch FROM (now() - LEAST(s.last_vacuum, s.last_autovacuum,
                                         s.last_analyze, s.last_autoanalyze)))::float8 AS oldest_mark_s,
       extract(epoch FROM (now() - GREATEST(
           coalesce(s.last_autovacuum, 'epoch'::timestamptz),
           coalesce(s.last_vacuum,     'epoch'::timestamptz)
       )))::bigint AS sec_since_vacuum,
       extract(epoch FROM (now() - GREATEST(
           coalesce(s.last_autoanalyze, 'epoch'::timestamptz),
           coalesce(s.last_analyze,     'epoch'::timestamptz)
       )))::bigint AS sec_since_analyze,
       c.relpages::bigint AS pages,
       c.relallvisible::bigint AS allvisible,
       {frozen_col}
       c.reltuples::float8 AS reltuples,
       (SELECT coalesce(sum(ic.relpages), 0)::bigint FROM pg_index x
          JOIN pg_class ic ON ic.oid = x.indexrelid WHERE x.indrelid = c.oid) AS index_pages,
       pg_total_relation_size(c.oid)::bigint AS total_bytes,
       age(c.relfrozenxid)::bigint AS xid_age,
       (SELECT array_agg(opt) FROM unnest(
            coalesce(c.reloptions, ARRAY[]::text[])
        ) AS opt WHERE opt LIKE 'autovacuum%%' OR opt LIKE 'toast.autovacuum%%'
       ) AS per_table_opts,
       c.reltoastrelid::oid AS toast_oid,
       c.oid::bigint AS relid
FROM pg_stat_user_tables s
JOIN pg_class c ON c.oid = s.relid
ORDER BY GREATEST(s.n_dead_tup, s.n_mod_since_analyze) DESC NULLS LAST
LIMIT 200
```

**alter_recommended** - `run_vacuum_check` line 556

```sql
ALTER TABLE {fq} SET ({kv_pairs});
```

**alter_reset** - `run_vacuum_check` line 557

```sql
ALTER TABLE {fq} RESET ({', '.join(recommended.keys())});
```

**measure** - `run_vacuum_check` line 665

```sql
cluster WAL (pg_stat_wal)
```

`run_vacuum_check` line 786

```sql
SELECT schemaname AS schema, relname AS "table",
       round(score::numeric, 3)               AS score,
       round(xid_score::numeric, 3)           AS xid_score,
       round(mxid_score::numeric, 3)          AS mxid_score,
       round(vacuum_score::numeric, 3)        AS vacuum_score,
       round(vacuum_insert_score::numeric, 3) AS insert_score,
       round(analyze_score::numeric, 3)       AS analyze_score,
       do_vacuum, do_analyze, for_wraparound
  FROM pg_stat_autovacuum_scores
 WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
   AND schemaname NOT LIKE 'pg_toast%%'
 ORDER BY score DESC NULLS LAST
 LIMIT 50
```

## checks/workload_report.py

`run_workload_report_check` line 65

```sql
SELECT 1 FROM {_pss} LIMIT 1
```

**counts_sql** - `run_workload_report_check` line 100

```sql
SELECT queryid::text AS queryid, dbid::bigint AS dbid, userid::bigint AS userid, {_tl}, calls, rows AS rows_total FROM {_pss} WHERE query IS NOT NULL
```

`run_workload_report_check` line 124

```sql
SELECT pid, usename, application_name AS app,
       state, wait_event_type, wait_event,
       EXTRACT(EPOCH FROM now() - xact_start) AS xact_age,
       regexp_replace(query, '[[:space:]]+', ' ', 'g') AS query,
       coalesce(pg_blocking_pids(pid), '{}'::int[]) AS blockers
  FROM pg_stat_activity
 WHERE pid <> pg_backend_pid()
```
