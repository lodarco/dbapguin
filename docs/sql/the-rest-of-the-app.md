# The rest of the app

[All parts](README.md) · 785 statements

## app.py

**a** - `_downgrade_check_sql` line 1982

```sql
SELECT unnest(pg_blocking_pids(waiting.pid))
FROM pg_locks waiting
WHERE NOT waiting.granted
```

`_downgrade_check_sql` line 1987

```sql
SELECT blocking.pid
FROM pg_locks blocking
JOIN pg_locks waiting ON blocking.relation = waiting.relation
    AND NOT waiting.granted
    AND blocking.granted
    AND blocking.pid <> waiting.pid
```

**sql** - `_replication_sql_94` line 2035

```sql
SELECT r.pid,
       r.usename                                             AS "user",
       a.datname                                             AS database,
       r.client_addr::text                                   AS replica,
       r.application_name                                    AS app,
       r.state,
       r.sync_state,
       pg_xlog_location_diff(pg_current_xlog_location(), r.replay_location) AS total_lag_bytes,
       pg_xlog_location_diff(pg_current_xlog_location(), r.sent_location)   AS unsent_bytes,
       pg_xlog_location_diff(r.sent_location, r.write_location) AS write_lag_bytes,
       pg_xlog_location_diff(r.sent_location, r.flush_location) AS flush_lag_bytes,
       pg_xlog_location_diff(r.sent_location, r.replay_location) AS replay_lag_bytes
  FROM pg_stat_replication r
  LEFT JOIN pg_stat_activity a ON a.pid = r.pid
 ORDER BY pg_xlog_location_diff(pg_current_xlog_location(), r.replay_location)
          DESC NULLS LAST, r.application_name
```

**sql** - `_connections_sql_pre96` line 2061

```sql
SELECT state,
       CASE WHEN waiting THEN 'Lock' END AS wait_event_type,
       NULL::text                        AS wait_event,
       count(*)                          AS connections
FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
GROUP BY state, waiting
ORDER BY connections DESC
```

**sql** - `_long_queries_sql` line 2080

```sql
SELECT pid,
       usename,
       now() - query_start AS duration,
       {wait_cols}
       query AS _query_full,
       left(query, 120) AS query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '5 seconds'
  AND pid <> pg_backend_pid()
ORDER BY duration DESC
```

**sql** - `_sizes_sql_pre10` line 2104

```sql
SELECT d.datname                                          AS database,
       pg_size_pretty(pg_database_size(d.datname))        AS size,
       pg_database_size(d.datname)                        AS bytes,
       (SELECT count(*) FROM pg_stat_activity sa
        WHERE sa.datname = d.datname)                     AS connections,
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

`_lock_stats_section` line 2145

```sql
SELECT locktype, waits, round(wait_time::numeric, 1) AS wait_time_ms,
       fastpath_exceeded, stats_reset
  FROM pg_stat_lock
 ORDER BY wait_time DESC, waits DESC, fastpath_exceeded DESC
```

`run_db_check` line 2376

```sql
SET application_name = %s
```

`run_db_check` line 2377

```sql
SELECT pg_backend_pid() AS pid
```

`run_db_check` line 2433

```sql
SELECT count(*) AS n FROM pg_stat_activity WHERE pid <> pg_backend_pid() AND query = '<insufficient privilege>'
```

`run_db_check` line 2541

```sql
SELECT 1 FROM pg_roles WHERE rolname = %s
```

`run_db_check` line 2569

```sql
SELECT 1 FROM pg_namespace WHERE nspname = %s
```

`_move_summary` line 2901

```sql
SELECT
  (SELECT count(*) FROM pg_index i WHERE i.indrelid = c.oid)             AS n_idx,
  (SELECT count(*) FROM pg_constraint k
     WHERE k.confrelid = c.oid AND k.contype = 'f')                      AS n_inbound_fk,
  (SELECT count(*) FROM pg_depend d JOIN pg_class s ON s.oid = d.objid
     WHERE d.refobjid = c.oid AND s.relkind = 'S' AND d.deptype = 'a')    AS n_seq,
  c.relkind
FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relname = %s
```

`_search_path_levels` line 2976

```sql
SELECT COALESCE(d.datname, '') AS datname, s.setconfig
  FROM pg_db_role_setting s
  JOIN pg_roles r ON r.oid = s.setrole
  LEFT JOIN pg_database d ON d.oid = s.setdatabase
 WHERE r.rolname = %s
```

`_search_path_levels` line 3001

```sql
SELECT s.setconfig
  FROM pg_db_role_setting s
  JOIN pg_database d ON d.oid = s.setdatabase
 WHERE d.datname = %s AND s.setrole = 0
```

`_search_path_levels` line 3021

```sql
SELECT boot_val FROM pg_settings WHERE name = 'search_path'
```

`_apply_search_path` line 3100

```sql
ALTER ROLE {role} SET search_path TO
```

`api_db_check_cancel` line 3365

```sql
SET statement_timeout = 5000
```

`api_db_check_cancel` line 3367

```sql
SELECT {fn}(%s)
```

`api_db_check_cancel` line 3372

```sql
SELECT pid, {fn}(pid) FROM pg_stat_activity WHERE application_name = %s AND pid <> pg_backend_pid()
```

`api_db_check_cancel` line 3382

```sql
SELECT {fn}({', '.join((str(p) for p in signalled))})
```

`_schema_extensions` line 3694

```sql
WITH owned AS (
    SELECT d.refobjid AS ext_oid, count(*) AS n
      FROM pg_depend d
     WHERE d.deptype = 'e'
       AND d.refclassid = 'pg_extension'::regclass
       AND (
         EXISTS (SELECT 1 FROM pg_proc p JOIN pg_namespace ns ON ns.oid=p.pronamespace
                  WHERE p.oid=d.objid AND d.classid='pg_proc'::regclass AND ns.nspname=%(s)s)
      OR EXISTS (SELECT 1 FROM pg_class c JOIN pg_namespace ns ON ns.oid=c.relnamespace
                  WHERE c.oid=d.objid AND d.classid='pg_class'::regclass AND ns.nspname=%(s)s)
      OR EXISTS (SELECT 1 FROM pg_type t JOIN pg_namespace ns ON ns.oid=t.typnamespace
                  WHERE t.oid=d.objid AND d.classid='pg_type'::regclass AND ns.nspname=%(s)s)
      OR EXISTS (SELECT 1 FROM pg_operator o JOIN pg_namespace ns ON ns.oid=o.oprnamespace
                  WHERE o.oid=d.objid AND d.classid='pg_operator'::regclass AND ns.nspname=%(s)s)
      OR EXISTS (SELECT 1 FROM pg_opclass oc JOIN pg_namespace ns ON ns.oid=oc.opcnamespace
                  WHERE oc.oid=d.objid AND d.classid='pg_opclass'::regclass AND ns.nspname=%(s)s)
      OR EXISTS (SELECT 1 FROM pg_opfamily of2 JOIN pg_namespace ns ON ns.oid=of2.opfnamespace
                  WHERE of2.oid=d.objid AND d.classid='pg_opfamily'::regclass AND ns.nspname=%(s)s)
       )
     GROUP BY d.refobjid
)
SELECT e.extname AS name, e.extversion AS version,
       n.nspname AS ext_schema, owned.n AS objects, true AS lives_here
  FROM owned
  JOIN pg_extension e ON e.oid = owned.ext_oid
  JOIN pg_namespace n ON n.oid = e.extnamespace
 ORDER BY owned.n DESC, e.extname
```

`_schema_extensions` line 3725

```sql
CREATE EXTENSION IF NOT EXISTS %s SCHEMA %s;
```

`_schema_extensions` line 3742

```sql
SELECT DISTINCT e.extname AS name, e.extversion AS version,
       en.nspname AS ext_schema
  FROM pg_depend d_local
  JOIN pg_depend d_ext ON d_ext.classid = d_local.refclassid
                      AND d_ext.objid   = d_local.refobjid
                      AND d_ext.deptype = 'e'
                      AND d_ext.refclassid = 'pg_extension'::regclass
  JOIN pg_extension e  ON e.oid = d_ext.refobjid
  JOIN pg_namespace en ON en.oid = e.extnamespace
 WHERE d_local.deptype IN ('n','a')
   AND (
     EXISTS (SELECT 1 FROM pg_class c JOIN pg_namespace ns ON ns.oid=c.relnamespace
              WHERE c.oid = d_local.objid
                AND d_local.classid = 'pg_class'::regclass AND ns.nspname = %(s)s)
  OR EXISTS (SELECT 1 FROM pg_attrdef ad JOIN pg_class c ON c.oid=ad.adrelid
               JOIN pg_namespace ns ON ns.oid=c.relnamespace
              WHERE ad.oid = d_local.objid
                AND d_local.classid = 'pg_attrdef'::regclass AND ns.nspname = %(s)s)
  OR EXISTS (SELECT 1 FROM pg_proc p JOIN pg_namespace ns ON ns.oid=p.pronamespace
              WHERE p.oid = d_local.objid
                AND d_local.classid = 'pg_proc'::regclass AND ns.nspname = %(s)s)
   )
 ORDER BY 1
```

`_schema_extensions` line 3773

```sql
CREATE EXTENSION IF NOT EXISTS %s SCHEMA %s;
```

`_schema_extension_config_tables` line 3805

```sql
SELECT e.extname                        AS extension,
       n.nspname                        AS table_schema,
       c.relname                        AS "table",
       cond.condition                   AS condition,
       n.nspname = %(s)s                AS lives_here
  FROM pg_extension e
  JOIN LATERAL unnest(e.extconfig, e.extcondition)
            AS cond(objid, condition) ON true
  JOIN pg_class c     ON c.oid = cond.objid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE e.extconfig IS NOT NULL
 ORDER BY e.extname, n.nspname, c.relname
```

`_schema_extension_config_tables` line 3838

```sql
SELECT count(*) AS n FROM %s %s
```

`_schema_extension_config_tables` line 3850

```sql
SELECT * FROM %s %s;
```

`_schema_extension_config_tables` line 3850

```sql
SELECT * FROM %s;
```

`_schema_superuser_objects` line 3877

```sql
SELECT p.proname AS name,
       pg_get_function_identity_arguments(p.oid) AS args,
       l.lanname AS language
  FROM pg_proc p
  JOIN pg_namespace n ON n.oid = p.pronamespace
  JOIN pg_language  l ON l.oid = p.prolang
 WHERE n.nspname = %%s
   AND l.lanname IN ('c', 'internal')
   -- plain functions and procedures only; an aggregate ('a') and
   -- a window function ('w') are not CREATE FUNCTION statements.
   AND %s IN ('f', 'p')
   AND NOT EXISTS (SELECT 1 FROM pg_depend d
                    WHERE d.classid = 'pg_proc'::regclass
                      AND d.objid = p.oid AND d.deptype IN ('i','e'))
 ORDER BY p.proname
```

`_fetch_pg_files` line 3985

```sql
SELECT name, setting FROM pg_settings WHERE name = ANY(%s)
```

`_detect_provider` line 4304

```sql
SELECT
  EXISTS(SELECT 1 FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
         WHERE p.proname = 'aurora_version' AND n.nspname = 'pg_catalog')   AS is_aurora,
  EXISTS(SELECT 1 FROM pg_settings WHERE name LIKE 'rds.%%')                AS is_rds,
  -- AlloyDB: flags only its server build defines, or the roles it
  -- creates, and no superuser beyond the platform ones (see below).
  (EXISTS(SELECT 1 FROM pg_settings WHERE name LIKE 'alloydb.%%'
                                      OR name LIKE 'google_columnar_engine.%%')
   OR EXISTS(SELECT 1 FROM pg_roles
             WHERE rolname IN ('alloydbsuperuser', 'alloydbadmin')))
  AND NOT EXISTS(SELECT 1 FROM pg_roles
                 WHERE rolsuper AND rolname NOT LIKE 'alloydb%%'
                   AND rolname <> 'cloudsqladmin')                  AS is_alloydb,
  EXISTS(SELECT 1 FROM pg_settings WHERE name LIKE 'cloudsql.%%')           AS is_cloudsql,
  EXISTS(SELECT 1 FROM pg_settings WHERE name LIKE 'azure.%%'
                                      OR name = 'azure_storage.allow_network_access') AS is_azure,
  EXISTS(SELECT 1 FROM pg_roles WHERE rolname = 'supabase_admin')           AS is_supabase,
  EXISTS(SELECT 1 FROM pg_roles WHERE rolname = 'neon_superuser')           AS is_neon,
  EXISTS(SELECT 1 FROM pg_roles WHERE rolname = 'doadmin')                  AS is_do,
  EXISTS(SELECT 1 FROM pg_roles WHERE rolname = 'tsdbadmin')                AS is_ts,
  EXISTS(SELECT 1 FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
         WHERE p.proname = 'crunchy_pgbouncer_admin' AND n.nspname = 'pg_catalog') AS is_crunchy,
  EXISTS(SELECT 1 FROM pg_roles WHERE rolname LIKE 'u%%' AND rolname ~ '^u[a-z0-9]{8,}$')
    AND current_setting('cluster_name', true) LIKE 'heroku-postgresql-%%' AS is_heroku,
  -- Not part of the fingerprint: whether the role of THIS session
  -- is a real superuser, from the catalogue. It lifts the refusals
  -- of the provider (see _capability_block), not the key.
  COALESCE((SELECT rolsuper FROM pg_roles WHERE rolname = current_user), false)
                                                                            AS session_superuser
```

`_hba_write_copy` line 4427

```sql
COPY (SELECT nullif(l, '') FROM unnest(%s::text[])         WITH ORDINALITY t(l, n) ORDER BY n) TO %s (FORMAT csv, DELIMITER E'\x01', QUOTE E'\x02', ESCAPE E'\x02')
```

`_hba_write_file_lines` line 4466

```sql
SELECT lo_export(%s, %s)
```

`_hba_write_file_lines` line 4467

```sql
SELECT lo_unlink(%s)
```

`_hba_write_file_lines` line 4475

```sql
SELECT lo_unlink(%s)
```

`_hba_peers` line 4516

```sql
SELECT pg_is_in_recovery() AS rec
```

`_hba_peers` line 4522

```sql
SELECT COALESCE(host(client_addr), 'local') AS client,
       COALESCE(application_name, '')       AS app,
       state, sync_state
  FROM pg_stat_replication
```

`_hba_peers` line 4533

```sql
SELECT slot_name, slot_type, active
  FROM pg_replication_slots
 WHERE slot_type = 'physical'
```

`_hba_list_backups` line 4554

```sql
SELECT n.name AS name, (pg_stat_file(%s || '/' || n.name)).size AS size FROM pg_ls_dir(%s) AS n(name)
```

`_hba_list_backups` line 4565

```sql
SELECT pg_ls_dir(%s) AS name
```

## archive_scan.py

`DataScan.feed` line 507

```sql
SELECT pg_catalog.setval(
```

## archiver_status.py

`archiver_status` line 64

```sql
SELECT current_setting('archive_mode')        AS mode,
       current_setting('archive_command')     AS command,
       {lib}                                  AS library,
       pg_is_in_recovery()                    AS in_recovery,
       archived_count, last_archived_wal, last_archived_time,
       failed_count, last_failed_wal, last_failed_time,
       (last_failed_time IS NOT NULL
        AND (last_archived_time IS NULL
             OR last_failed_time > last_archived_time)) AS failing_now,
       EXTRACT(epoch FROM now() - last_archived_time)::bigint AS archived_age_s,
       EXTRACT(epoch FROM now() - last_failed_time)::bigint   AS failed_age_s
  FROM pg_stat_archiver
```

`archiver_status` line 115

```sql
SELECT (SELECT count(*) FROM pg_ls_archive_statusdir()
         WHERE name LIKE '%.ready'
           AND name NOT LIKE '%.backup.ready'
           AND name NOT LIKE '%.history.ready')        AS n,
       (SELECT setting::bigint FROM pg_settings
         WHERE name = 'wal_segment_size')               AS seg
```

## bloat_estimate.py

**reindex_sql** - `recommend_index` line 437

```sql
REINDEX INDEX{conc} {iq};
```

`recommend_table` line 520

```sql
VACUUM (VERBOSE, ANALYZE) {fq};
```

`recommend_table` line 551

```sql
VACUUM FULL {fq};  -- rewrites heap + TOAST
```

`recommend_table` line 594

```sql
VACUUM ANALYZE {fq};
```

`recommend_table` line 597

```sql
VACUUM ANALYZE {fq};
```

`recommend_table` line 612

```sql
ANALYZE {fq};
```

`recommend_table` line 617

```sql
ANALYZE {fq};
```

`recommend_table` line 630

```sql
ANALYZE {fq};
```

**sql** - `_fetch_heap_and_toast` line 648

```sql
WITH {heap_otta_cte(where=heap_where)}
SELECT n.nspname AS schema, c.relname AS "table", c.oid AS reloid, c.relkind::text AS relkind,
       pg_relation_size(c.oid) AS heap_bytes,
       he.bs AS bs, he.otta AS otta,
       {heap_waste_expr('c.oid', 'he.bs', 'he.otta')}::bigint AS heap_waste_bytes,
       COALESCE(pg_total_relation_size(c.reltoastrelid), 0) AS toast_total_bytes,
       COALESCE(pg_relation_size(c.reltoastrelid), 0) AS toast_heap_bytes,
       COALESCE(ts.n_live_tup, 0) AS toast_live, COALESCE(ts.n_dead_tup, 0) AS toast_dead,
       pg_indexes_size(c.oid) AS index_bytes,
       pg_total_relation_size(c.oid) AS total_bytes,
       COALESCE(s.n_live_tup, 0) AS n_live, COALESCE(s.n_dead_tup, 0) AS n_dead,
       COALESCE(s.n_mod_since_analyze, 0) AS n_mod,
       NULLIF(c.reltuples, -1)::float8 AS reltuples, c.relpages::bigint AS relpages,
       GREATEST(s.last_vacuum, s.last_autovacuum) AS last_vacuum,
       GREATEST(s.last_analyze, s.last_autoanalyze) AS last_analyze
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN heap_bloat_est he ON he.reloid = c.oid
  LEFT JOIN pg_stat_user_tables s ON s.relid = c.oid
  LEFT JOIN pg_stat_all_tables ts ON ts.relid = c.reltoastrelid
 WHERE c.relkind IN ('r', 'p', 'm')
   AND n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND n.nspname !~ '^pg_toast'
   {table_filter}
   AND pg_total_relation_size(c.oid) > {int(min_bytes)}
 ORDER BY pg_total_relation_size(c.oid) DESC
 LIMIT {int(limit)}
```

`_fetch_indexes` line 691

```sql
WITH {index_bloat_cte()}
SELECT idx.indrelid AS reloid, idx."index" AS name, idx.idx_bytes AS bytes,
       idx.waste_bytes AS waste_bytes, idx.reltuples AS reltuples,
       idx.indisprimary AS is_pk, idx.indisunique AS is_unique,
       {index_is_fk_sql()},
       COALESCE(si.idx_scan, 0) AS idx_scan
  FROM idx
  LEFT JOIN pg_stat_user_indexes si ON si.indexrelid = idx.index_oid
 WHERE idx.indrelid = ANY(%s)
```

`_fetch_indexes` line 717

```sql
SELECT x.indrelid AS reloid, i.relname AS name, am.amname AS method,
       pg_relation_size(i.oid) AS bytes,
       x.indisprimary AS is_pk, x.indisunique AS is_unique,
       COALESCE(si.idx_scan, 0) AS idx_scan
  FROM pg_index x
  JOIN pg_class i ON i.oid = x.indexrelid
  JOIN pg_am am ON am.oid = i.relam
  LEFT JOIN pg_stat_user_indexes si ON si.indexrelid = i.oid
 WHERE x.indrelid = ANY(%s) AND am.amname <> 'btree'
```

## compare_apply.py

`Catalog._one` line 280

```sql
ROLLBACK TO SAVEPOINT dbap_classify
```

`Catalog.fk_referenced` line 306

```sql
SELECT pg_catalog.to_regclass(%s) IS NOT NULL, (SELECT pg_catalog.format('%%I.%%I', n.nspname, r.relname)    FROM pg_catalog.pg_constraint con    JOIN pg_catalog.pg_class r ON r.oid = con.confrelid    JOIN pg_catalog.pg_namespace n ON n.oid = r.relnamespace   WHERE con.conrelid = pg_catalog.to_regclass(%s) AND con.conname = %s     AND con.contype = 'f' AND con.confrelid <> con.conrelid)
```

`Catalog.fk_tables` line 343

```sql
SELECT pg_catalog.to_regclass(%s) IS NOT NULL
```

`Catalog.fk_tables` line 350

```sql
SELECT 'referenced' AS side, pg_catalog.format('%%I.%%I', n.nspname, r.relname) AS other  FROM pg_catalog.pg_constraint con  JOIN pg_catalog.pg_class r ON r.oid = con.confrelid  JOIN pg_catalog.pg_namespace n ON n.oid = r.relnamespace WHERE con.contype = 'f' AND con.conrelid = pg_catalog.to_regclass(%s)   AND con.confrelid <> con.conrelid   AND (%s = '' OR EXISTS (SELECT 1 FROM pg_catalog.pg_attribute a                            WHERE a.attrelid = con.conrelid AND a.attname = %s                              AND a.attnum = ANY (con.conkey))) UNION SELECT 'referencing', pg_catalog.format('%%I.%%I', n.nspname, r.relname)  FROM pg_catalog.pg_constraint con  JOIN pg_catalog.pg_class r ON r.oid = con.conrelid  JOIN pg_catalog.pg_namespace n ON n.oid = r.relnamespace WHERE con.contype = 'f' AND con.confrelid = pg_catalog.to_regclass(%s)   AND con.confrelid <> con.conrelid   AND (%s = '' OR EXISTS (SELECT 1 FROM pg_catalog.pg_attribute a                            WHERE a.attrelid = con.confrelid AND a.attname = %s                              AND a.attnum = ANY (con.confkey))) ORDER BY 1, 2
```

`Catalog.column_type` line 384

```sql
SELECT pg_catalog.format_type(a.atttypid, a.atttypmod) FROM pg_catalog.pg_attribute a WHERE a.attrelid = pg_catalog.to_regclass(%s) AND a.attname = %s AND a.attnum > 0 AND NOT a.attisdropped
```

`Catalog.volatile` line 397

```sql
SELECT DISTINCT proname::text FROM pg_catalog.pg_proc WHERE proname = ANY(%s) AND provolatile = 'v'
```

`copy_targets` line 1099

```sql
SELECT pg_catalog.current_setting('search_path')
```

`copy_targets` line 1101

```sql
SELECT pg_catalog.set_config('search_path', '', false)
```

`copy_targets` line 1119

```sql
SELECT c.relkind FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname = %s
```

`copy_targets` line 1128

```sql
SELECT 1 FROM pg_namespace WHERE nspname = %s
```

`copy_targets` line 1136

```sql
SELECT c.relname FROM pg_trigger t JOIN pg_class c ON c.oid = t.tgrelid JOIN pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname || '.' || t.tgname = %s
```

`copy_targets` line 1140

```sql
SELECT c.relname FROM pg_rewrite r JOIN pg_class c ON c.oid = r.ev_class JOIN pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname || '.' || r.rulename = %s
```

`copy_targets` line 1162

```sql
SELECT p.proname, pg_catalog.pg_get_function_identity_arguments(p.oid)
  FROM pg_catalog.pg_proc p JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname = %s
   AND p.proname || '(' || pg_catalog.pg_get_function_identity_arguments(p.oid) || ')' = %s
 LIMIT 1
```

`copy_targets` line 1176

```sql
SELECT 1 FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace WHERE n.nspname = %s AND t.typname = %s
```

`copy_targets` line 1179

```sql
SELECT 1 FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname = %s
```

`copy_targets` line 1187

```sql
SELECT pg_catalog.set_config('search_path', %s, false)
```

`permission_problems` line 1200

```sql
SELECT current_setting('server_version_num')::int
```

`permission_problems` line 1210

```sql
SELECT pg_catalog.pg_get_userbyid(c.relowner), pg_catalog.pg_has_role(c.relowner, 'USAGE') FROM pg_catalog.pg_class c JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname = %s
```

`permission_problems` line 1215

```sql
SELECT pg_catalog.pg_get_userbyid(t.typowner), pg_catalog.pg_has_role(t.typowner, 'USAGE') FROM pg_catalog.pg_type t JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace WHERE n.nspname = %s AND t.typname = %s
```

`permission_problems` line 1220

```sql
SELECT pg_catalog.pg_get_userbyid(p.proowner), pg_catalog.pg_has_role(p.proowner, 'USAGE') FROM pg_catalog.pg_proc p JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace WHERE n.nspname = %s AND p.proname || '(' || pg_catalog.pg_get_function_identity_arguments(p.oid) || ')' = %s
```

`permission_problems` line 1227

```sql
SELECT pg_catalog.pg_get_userbyid(c.relowner), pg_catalog.pg_has_role(c.relowner, 'USAGE') FROM pg_catalog.pg_class c JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname = %s
```

`permission_problems` line 1232

```sql
SELECT pg_catalog.pg_get_userbyid(nspowner), pg_catalog.pg_has_role(nspowner, 'USAGE') FROM pg_catalog.pg_namespace WHERE nspname = %s
```

`permission_problems` line 1239

```sql
SELECT pg_catalog.has_schema_privilege(n.oid, 'CREATE') FROM pg_catalog.pg_namespace n WHERE n.nspname = %s
```

`permission_problems` line 1255

```sql
SELECT pg_catalog.pg_has_role(r.oid, %s) FROM pg_catalog.pg_roles r WHERE r.rolname = %s
```

`permission_problems` line 1264

```sql
SELECT rolcreaterole OR rolsuper FROM pg_catalog.pg_roles WHERE rolname = current_user
```

`execute` line 1309

```sql
ROLLBACK TO SAVEPOINT dbap_cmp_dry
```

## dump_scope.py

`_resolve_schemas` line 308

```sql
SELECT nspname FROM pg_namespace WHERE lower(nspname) = ANY(%s)
```

`_resolve_rels` line 330

```sql
SELECT n.nspname, c.relname, c.oid, c.relkind   FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace  WHERE c.relkind = ANY(%s) AND lower(c.relname) = ANY(%s) AND {_SYS_SCHEMA_SQL}
```

`resolve` line 419

```sql
SELECT extname FROM pg_extension
```

## object_scope.py

`server_version` line 156

```sql
SHOW server_version_num
```

`extension_members` line 184

```sql
SELECT d.classid::regclass::text, d.objid, e.extname
                   FROM pg_depend d JOIN pg_extension e ON e.oid = d.refobjid
                  WHERE d.refclassid = 'pg_extension'::regclass AND d.deptype = 'e'
```

`extension_schemas` line 211

```sql
SELECT n.nspname, e.extname FROM pg_depend d
                   JOIN pg_namespace n ON n.oid = d.objid
                   JOIN pg_extension e ON e.oid = d.refobjid
                  WHERE d.classid = 'pg_namespace'::regclass AND d.refclassid = 'pg_extension'::regclass
                    AND d.deptype = 'e'
```

`simple_in_schema` line 274

```sql
SELECT x.oid, n.nspname, x.{ncol}, {disp or 'x.' + ncol},
       {'pg_get_userbyid(x.' + ocol + ')' if ocol else 'NULL::name'},
       {extra[1] if extra else 'NULL::text'}
  FROM {cat} x JOIN pg_namespace n ON n.oid = x.{nspcol}
 WHERE {sf} ORDER BY n.nspname, x.{ncol}
```

`namespaces` line 298

```sql
SELECT n.oid, n.nspname, pg_get_userbyid(n.nspowner) FROM pg_namespace n
                   WHERE {where} ORDER BY n.nspname
```

`objects` line 370

```sql
SELECT e.oid, n.nspname, e.extname, e.extversion, pg_get_userbyid(e.extowner)
                        FROM pg_extension e JOIN pg_namespace n ON n.oid = e.extnamespace
                       WHERE {sf} ORDER BY n.nspname, e.extname
```

`relations` line 395

```sql
SELECT c.oid, n.nspname, c.relname, c.relkind::text, pg_get_userbyid(c.relowner),
       {ispart}, c.relpersistence::text, quote_ident(n.nspname) || '.' || quote_ident(c.relname)
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.relkind IN ('r', 'p', 'v', 'm', 'S', 'f', 'i', 'I') AND {sf}
 ORDER BY n.nspname, c.relname
```

`relations` line 420

```sql
SELECT i.inhrelid, i.inhparent, i.inhparent::regclass::text FROM pg_inherits i
                        JOIN pg_class c ON c.oid = i.inhrelid {nsp} AND c.relispartition
                         AND c.relkind IN ('r', 'p', 'f')
```

`relations` line 429

```sql
SELECT d.objid, d.refobjid, t.relname, a.attname, d.deptype::text, t.oid::regclass::text
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_depend d ON d.classid = 'pg_class'::regclass AND d.objid = c.oid
  JOIN pg_class t ON t.oid = d.refobjid
  JOIN pg_attribute a ON a.attrelid = d.refobjid AND a.attnum = d.refobjsubid
 WHERE {sf} AND c.relkind = 'S' AND d.refclassid = 'pg_class'::regclass
   AND d.objsubid = 0 AND d.refobjsubid > 0 AND d.deptype IN ('a', 'i')
```

`relations` line 448

```sql
SELECT x.indexrelid, x.indrelid, t.relname, x.indisprimary, x.indisunique,
       x.indisvalid,
       (SELECT k.conname FROM pg_constraint k WHERE k.conindid = x.indexrelid
           AND k.contype IN ('p', 'u', 'x') AND k.conrelid = x.indrelid LIMIT 1),
       {clone}
  FROM pg_index x JOIN pg_class c ON c.oid = x.indexrelid
  JOIN pg_class t ON t.oid = x.indrelid {nsp}
```

`routines` line 479

```sql
SELECT p.oid, n.nspname, p.proname, {kind}, pg_get_function_identity_arguments(p.oid),
       pg_get_userbyid(p.proowner)
  FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE {sf} AND NOT {_internal('pg_proc', 'p')}
 ORDER BY n.nspname, p.proname, 5
```

`types` line 502

```sql
SELECT t.oid, n.nspname, t.typname, t.typtype::text, pg_get_userbyid(t.typowner)
  FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace
 WHERE {sf} AND t.typtype IN ('b', 'c', 'd', 'e', 'r', 'm') AND t.typisdefined
   AND (t.typtype <> 'c' OR EXISTS (SELECT 1 FROM pg_class r
                                     WHERE r.oid = t.typrelid AND r.relkind = 'c'))
   AND (t.typtype = 'm' OR NOT {_internal('pg_type', 't')})
 ORDER BY n.nspname, t.typname
```

`types` line 519

```sql
SELECT rngmultitypid, rngtypid, rngtypid::regtype::text FROM pg_range WHERE rngmultitypid = ANY(%s)
```

`triggers` line 556

```sql
SELECT t.oid, n.nspname, c.relname, c.oid, t.tgname, t.tgenabled::text, {clone},
       t.tgfoid::regprocedure::text
  FROM pg_trigger t JOIN pg_class c ON c.oid = t.tgrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE {where} AND (NOT t.tgisinternal OR {clone} IS NOT NULL)
 ORDER BY n.nspname, c.relname, t.tgname
```

`rules` line 582

```sql
SELECT r.oid, n.nspname, c.relname, c.oid, r.rulename
  FROM pg_rewrite r JOIN pg_class c ON c.oid = r.ev_class
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE r.rulename <> '_RETURN' AND {where}
 ORDER BY n.nspname, c.relname, r.rulename
```

`policies` line 601

```sql
SELECT p.oid, n.nspname, c.relname, c.oid, p.polname
  FROM pg_policy p JOIN pg_class c ON c.oid = p.polrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE {where} ORDER BY n.nspname, c.relname, p.polname
```

`database_wide` line 638

```sql
SELECT oid, evtname, evtevent, evtenabled::text, pg_get_userbyid(evtowner)
                        FROM pg_event_trigger WHERE true {f} ORDER BY evtname
```

`database_wide` line 646

```sql
SELECT oid, lanname, pg_get_userbyid(lanowner), lanispl FROM pg_language
                       WHERE (lanispl OR lanname NOT IN ('internal', 'c', 'sql')) {f}
                       ORDER BY lanname
```

`database_wide` line 657

```sql
SELECT c.oid, format_type(c.castsource, NULL), format_type(c.casttarget, NULL)
  FROM pg_cast c
  JOIN pg_type ts ON ts.oid = c.castsource JOIN pg_namespace ns ON ns.oid = ts.typnamespace
  JOIN pg_type tt ON tt.oid = c.casttarget JOIN pg_namespace nt ON nt.oid = tt.typnamespace
 WHERE (c.oid >= %(first)s OR ns.nspname NOT IN ('pg_catalog', 'information_schema')
        OR nt.nspname NOT IN ('pg_catalog', 'information_schema'))
   -- the range -> multirange cast CREATE TYPE ... AS RANGE makes
   -- (14+) is part of the type, like its constructors
   AND NOT {_internal('pg_cast', 'c')} {f}
 ORDER BY 2, 3
```

`database_wide` line 673

```sql
SELECT oid, fdwname, pg_get_userbyid(fdwowner)
                        FROM pg_foreign_data_wrapper WHERE true {f} ORDER BY fdwname
```

`database_wide` line 679

```sql
SELECT s.oid, s.srvname, s.srvversion, pg_get_userbyid(s.srvowner), w.fdwname
                        FROM pg_foreign_server s JOIN pg_foreign_data_wrapper w ON w.oid = s.srvfdw
                       WHERE true {f} ORDER BY s.srvname
```

`database_wide` line 689

```sql
SELECT umid, CASE WHEN umuser = 0 THEN 'PUBLIC' ELSE usename END, srvname
                        FROM pg_user_mappings WHERE true {f} ORDER BY 3, 2
```

`database_wide` line 696

```sql
SELECT oid, pubname, puballtables, pg_get_userbyid(pubowner)
                        FROM pg_publication WHERE true {f} ORDER BY pubname
```

`database_wide` line 708

```sql
SELECT bool_and(has_column_privilege('pg_catalog.pg_subscription', col, 'SELECT'))
                       FROM unnest(ARRAY['oid', 'subdbid', 'subname', 'subowner', 'subenabled']) col
```

`database_wide` line 712

```sql
SELECT oid, subname, subenabled, pg_get_userbyid(subowner) FROM pg_subscription
                           WHERE subdbid = (SELECT oid FROM pg_database WHERE datname = current_database())
                             {f} ORDER BY subname
```

`database_wide` line 722

```sql
SELECT count(*) FROM pg_largeobject_metadata
```

`database_wide` line 727

```sql
SELECT oid, pg_get_userbyid(lomowner) FROM pg_largeobject_metadata
                       WHERE true {f} ORDER BY oid LIMIT %(lim)s
```

`database_wide` line 737

```sql
SELECT oid, spcname, pg_get_userbyid(spcowner), pg_tablespace_location(oid)
                        FROM pg_tablespace WHERE true {f} ORDER BY spcname
```

`inheritance` line 774

```sql
SELECT i.inhrelid, i.inhparent, i.inhseqno, {ispart}, {bound}
  FROM pg_inherits i
  JOIN pg_class c ON c.oid = i.inhrelid
  JOIN pg_class p ON p.oid = i.inhparent
 WHERE c.relkind IN ('r', 'p', 'f') AND {where}
 ORDER BY i.inhrelid, i.inhseqno
```

`inheritance` line 794

```sql
SELECT i.inhrelid, i.inhparent, {ispart}, {bound}
                        FROM pg_inherits i JOIN pg_class c ON c.oid = i.inhrelid
                       WHERE i.inhrelid = ANY(%s) AND c.relkind IN ('r', 'p', 'f')
                       ORDER BY i.inhrelid, i.inhseqno
```

`partition_keys` line 822

```sql
SELECT c.oid, pg_get_partkeydef(c.oid) FROM pg_class c WHERE c.oid = ANY(%s) AND c.relkind = 'p'
```

`children` line 858

```sql
SELECT c.relkind::text, n.nspname, c.relname FROM pg_class c
                   JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = %s
```

`children` line 874

```sql
SELECT c.oid, n.nspname, c.relname, c.relkind::text, {ispart}, {bound},
       pg_get_userbyid(c.relowner),
       EXISTS (SELECT 1 FROM pg_inherits g WHERE g.inhparent = c.oid)
  FROM pg_inherits i JOIN pg_class c ON c.oid = i.inhrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE i.inhparent = %s AND c.relkind IN ('r', 'p', 'f')
 ORDER BY c.relname
```

`children` line 895

```sql
SELECT x.indexrelid, i.relname, i.relkind::text, x.indisprimary, x.indisunique, x.indisvalid,
       am.amname, pg_get_indexdef(x.indexrelid),
       (SELECT k.conname FROM pg_constraint k WHERE k.conindid = x.indexrelid
           AND k.conrelid = x.indrelid AND k.contype IN ('p', 'u', 'x') LIMIT 1),
       {clone}
  FROM pg_index x JOIN pg_class i ON i.oid = x.indexrelid
  JOIN pg_am am ON am.oid = i.relam
 WHERE x.indrelid = %s ORDER BY x.indisprimary DESC, i.relname
```

`children` line 917

```sql
SELECT k.oid, k.conname, k.contype::text, pg_get_constraintdef(k.oid), k.conislocal,
       k.coninhcount, {conparent},
       CASE WHEN k.contype = 'f' THEN k.confrelid::regclass::text END
  FROM pg_constraint k
 WHERE k.conrelid = %s AND k.contype <> 'n'
 ORDER BY CASE k.contype WHEN 'p' THEN 0 WHEN 'u' THEN 1 WHEN 'f' THEN 2 ELSE 3 END,
          k.conname
```

`children` line 942

```sql
SELECT t.oid, t.tgname, c.relname FROM pg_trigger t JOIN pg_class c ON c.oid = t.tgrelid WHERE t.oid = ANY(%s)
```

`children` line 961

```sql
SELECT s.oid, n.nspname, s.relname, a.attname, d.deptype::text, pg_get_userbyid(s.relowner)
  FROM pg_depend d
  JOIN pg_class s ON s.oid = d.objid AND s.relkind = 'S'
  JOIN pg_namespace n ON n.oid = s.relnamespace
  JOIN pg_attribute a ON a.attrelid = d.refobjid AND a.attnum = d.refobjsubid
 WHERE d.classid = 'pg_class'::regclass AND d.refclassid = 'pg_class'::regclass
   AND d.refobjid = %s AND d.refobjsubid > 0 AND d.deptype IN ('a', 'i')
 ORDER BY a.attnum
```

`children` line 976

```sql
SELECT st.oid, n.nspname, st.stxname FROM pg_statistic_ext st
                       JOIN pg_namespace n ON n.oid = st.stxnamespace
                      WHERE st.stxrelid = %s ORDER BY st.stxname
```

`children` line 989

```sql
SELECT p.oid, p.pubname, 'table' FROM pg_publication p
  JOIN pg_publication_rel r ON r.prpubid = p.oid WHERE r.prrelid = %(r)s
{by_schema}
UNION SELECT p.oid, p.pubname, 'all' FROM pg_publication p
 WHERE p.puballtables AND %(rk)s IN ('r', 'p')
ORDER BY 2
```

`edges` line 1022

```sql
SELECT x.indexrelid, x.indrelid, {iclone},
                         (SELECT k.oid FROM pg_constraint k WHERE k.conindid = x.indexrelid
                             AND k.conrelid = x.indrelid AND k.contype IN ('p', 'u', 'x') LIMIT 1)
                    FROM pg_index x JOIN pg_class t ON t.oid = x.indrelid
                    JOIN pg_namespace n ON n.oid = t.relnamespace WHERE {sf}
```

`edges` line 1034

```sql
SELECT k.oid, k.conrelid, {cparent}, k.conislocal, k.contypid
                    FROM pg_constraint k JOIN pg_namespace n ON n.oid = k.connamespace
                   WHERE {sf} AND k.contype <> 'n'
```

`edges` line 1055

```sql
SELECT st.oid, st.stxrelid FROM pg_statistic_ext st
                        JOIN pg_namespace n ON n.oid = st.stxnamespace WHERE {sf}
```

`edges` line 1059

```sql
SELECT d.objid, d.refobjid, d.deptype::text
  FROM pg_depend d JOIN pg_class s ON s.oid = d.objid AND s.relkind = 'S'
  JOIN pg_namespace n ON n.oid = s.relnamespace
 WHERE d.classid = 'pg_class'::regclass AND d.refclassid = 'pg_class'::regclass
   AND d.refobjsubid > 0 AND d.deptype IN ('a', 'i') AND {sf}
```

`edges` line 1076

```sql
SELECT pn.pnnspid, pn.pnpubid FROM pg_publication_namespace pn
                            JOIN pg_namespace n ON n.oid = pn.pnnspid WHERE {sf}
```

`extension_links` line 1098

```sql
SELECT d.classid::regclass::text, d.objid, d.refobjid, e.extname, d.deptype::text
                    FROM pg_depend d JOIN pg_extension e ON e.oid = d.refobjid
                   WHERE d.refclassid = 'pg_extension'::regclass AND d.deptype IN {deptypes}{where}
                   ORDER BY e.extname, 1, 2
```

`publication_members` line 1127

```sql
SELECT r.prrelid, r.prpubid, n.nspname, t.relname, {cols}, {qual}
                    FROM pg_publication_rel r
                    JOIN pg_class t ON t.oid = r.prrelid
                    JOIN pg_namespace n ON n.oid = t.relnamespace
                   WHERE {sf} ORDER BY r.prpubid, t.relname
```

`column_sequences` line 1225

```sql
SELECT s.oid, n.nspname, s.relname, d.refobjid, tn.nspname, t.relname, a.attname, a.attnum,
       d.deptype::text
  FROM pg_depend d
  JOIN pg_class s ON s.oid = d.objid AND s.relkind = 'S'
  JOIN pg_namespace n ON n.oid = s.relnamespace
  JOIN pg_class t ON t.oid = d.refobjid
  JOIN pg_namespace tn ON tn.oid = t.relnamespace
  JOIN pg_attribute a ON a.attrelid = d.refobjid AND a.attnum = d.refobjsubid
 WHERE d.classid = 'pg_class'::regclass AND d.refclassid = 'pg_class'::regclass
   AND d.objsubid = 0 AND d.refobjsubid > 0 AND d.deptype IN ('a', 'i') AND {where}
 ORDER BY d.refobjid, a.attnum, s.relname
```

**_COLUMN_USERS_SQL** - `(module)` line 1249

```sql
WITH RECURSIVE
cols AS (
    SELECT a.attrelid AS tbl, a.attnum
      FROM pg_attribute a JOIN pg_class t ON t.oid = a.attrelid
      JOIN pg_namespace n ON n.oid = t.relnamespace
     WHERE n.nspname = ANY(%(s)s) AND t.relkind IN ('r', 'p') AND a.attnum > 0
       AND NOT a.attisdropped),
direct AS (
    SELECT cols.tbl, cols.attnum, d.classid, d.objid
      FROM cols JOIN pg_depend d
        ON d.refclassid = 'pg_class'::regclass AND d.refobjid = cols.tbl
       AND d.refobjsubid = cols.attnum AND d.deptype = 'n'
     WHERE d.classid <> 'pg_class'::regclass OR d.objid <> cols.tbl),
-- A view's or materialised view's own _RETURN rule stands for the view;
-- the views that read it are reached through its columns, at any depth.
rel(tbl, attnum, oid, depth, path) AS (
    SELECT DISTINCT x.tbl, x.attnum, r.ev_class, 1, ARRAY[r.ev_class]
      FROM direct x JOIN pg_rewrite r ON r.oid = x.objid
      JOIN pg_class v ON v.oid = r.ev_class
     WHERE x.classid = 'pg_rewrite'::regclass AND r.rulename = '_RETURN' AND v.relkind IN ('v', 'm')
  UNION ALL
    SELECT rel.tbl, rel.attnum, r.ev_class, rel.depth + 1, rel.path || r.ev_class
      FROM rel JOIN pg_depend d
        ON d.refclassid = 'pg_class'::regclass AND d.refobjid = rel.oid AND d.refobjsubid > 0
       AND d.classid = 'pg_rewrite'::regclass AND d.deptype = 'n'
      JOIN pg_rewrite r ON r.oid = d.objid AND r.rulename = '_RETURN'
      JOIN pg_class v ON v.oid = r.ev_class AND v.relkind IN ('v', 'm')
     WHERE r.ev_class <> ALL(rel.path) AND rel.depth < 50)
SELECT rel.tbl, rel.attnum, 'pg_class', rel.oid, max(rel.depth), NULL::oid
  FROM rel GROUP BY rel.tbl, rel.attnum, rel.oid
UNION ALL
-- Rules other than a view's own (ON INSERT DO ALSO ...), on whatever
-- table or view they hang, reading the column or a view above it.
SELECT u.tbl, u.attnum, 'pg_rewrite', r.oid, 0, r.ev_class
  FROM (SELECT tbl, attnum, objid FROM direct WHERE classid = 'pg_rewrite'::regclass
        UNION
        SELECT rel.tbl, rel.attnum, d.objid FROM rel JOIN pg_depend d
            ON d.refclassid = 'pg_class'::regclass AND d.refobjid = rel.oid AND d.refobjsubid > 0
           AND d.classid = 'pg_rewrite'::regclass AND d.deptype = 'n') u
  JOIN pg_rewrite r ON r.oid = u.objid AND r.rulename <> '_RETURN'
UNION ALL
-- Everything else that reads the column and that PostgreSQL will not
-- rebuild by itself.
SELECT x.tbl, x.attnum, x.classid::regclass::text, x.objid, 0, NULL::oid
  FROM direct x
 WHERE x.classid::regclass::text IN ('pg_trigger', 'pg_policy', 'pg_proc', 'pg_publication_rel')
    OR (x.classid = 'pg_attrdef'::regclass AND EXISTS (
            SELECT 1 FROM pg_attrdef ad WHERE ad.oid = x.objid
               AND (ad.adrelid <> x.tbl OR ad.adnum <> x.attnum)))
```

`column_users` line 1335

```sql
SELECT oid, rulename FROM pg_rewrite WHERE oid = ANY(%s)
```

`column_users` line 1340

```sql
SELECT u.c::text, u.o, pg_describe_object(u.c, u.o, 0)
                       FROM unnest(%s::regclass[], %s::oid[]) u(c, o)
```

`column_users` line 1344

```sql
SELECT a.attrelid, a.attnum, a.attname FROM pg_attribute a
                   WHERE (a.attrelid, a.attnum) IN (SELECT * FROM unnest(%s::oid[], %s::int2[]))
```

`partition_clones` line 1384

```sql
SELECT d.classid::regclass::text, d.objid, d.refobjid
                   FROM pg_depend d
                  WHERE (d.classid, d.objid) IN (SELECT c, o FROM unnest(%s::regclass[], %s::oid[]) u(c, o))
                    AND d.refclassid = d.classid AND d.refobjid <> d.objid AND d.objsubid = 0
                    AND d.deptype IN ('P', 'I')
```

`relation_names` line 1401

```sql
SELECT c.oid, n.nspname, c.relname, c.relkind::text,
                        quote_ident(n.nspname) || '.' || quote_ident(c.relname) FROM pg_class c
                   JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = ANY(%s)
```

**_DEPENDENTS_SQL** - `(module)` line 1420

```sql
WITH RECURSIVE r(classid, objid, objsubid) AS (
    SELECT c, o, 0 FROM unnest(%(c)s::regclass[], %(o)s::oid[]) AS u(c, o)
  UNION
    SELECT CASE WHEN d.refclassid = r.classid AND d.refobjid = r.objid THEN d.classid ELSE d.refclassid END,
           CASE WHEN d.refclassid = r.classid AND d.refobjid = r.objid THEN d.objid ELSE d.refobjid END,
           CASE WHEN d.refclassid = r.classid AND d.refobjid = r.objid THEN d.objsubid ELSE 0 END
      FROM r JOIN pg_depend d
        ON (d.refclassid = r.classid AND d.refobjid = r.objid
            AND (r.objsubid = 0 OR d.refobjsubid = r.objsubid OR d.refobjsubid = 0)
            AND d.deptype = ANY(%(down)s))
        OR (d.classid = r.classid AND d.objid = r.objid AND d.objsubid = 0
            AND d.deptype = ANY(%(up)s) AND NOT (d.refclassid = d.classid AND d.refobjid = d.objid))
)
SELECT DISTINCT r.classid::regclass::text, r.objid, r.objsubid,
       pg_describe_object(r.classid, r.objid, r.objsubid)
  FROM r
 WHERE r.classid <> 'pg_namespace'::regclass OR r.objid NOT IN (
       SELECT oid FROM pg_namespace WHERE nspname IN ('pg_catalog', 'information_schema'))
```

`outside` line 1484

```sql
SELECT DISTINCT d.refclassid::regclass::text, d.refobjid,
       pg_describe_object(d.classid, d.objid, d.objsubid)
  FROM pg_depend d
 WHERE (d.classid, d.objid) IN (SELECT c, o FROM unnest(%s::regclass[], %s::oid[]) u(c, o))
   AND d.deptype = 'n' AND d.refobjid >= %s
   AND d.refclassid <> 'pg_namespace'::regclass
```

`outside` line 1501

```sql
SELECT d.classid::regclass::text, d.objid, d.refclassid::regclass::text, d.refobjid
                       FROM pg_depend d
                      WHERE (d.classid, d.objid) IN (SELECT c, o FROM unnest(%s::regclass[], %s::oid[]) u(c, o))
                        AND d.deptype = 'i' AND d.objsubid = 0
                        AND NOT (d.refclassid = d.classid AND d.refobjid = d.objid)
```

`outside` line 1512

```sql
SELECT 'pg_extension'::regclass::text, oid FROM pg_extension WHERE extname = %s
```

`outside` line 1524

```sql
SELECT d.classid::regclass::text, d.objid FROM pg_depend d
                      WHERE (d.classid, d.objid) IN (SELECT c, o FROM unnest(%s::regclass[], %s::oid[]) u(c, o))
                        AND d.deptype IN ('i', 'P') AND d.objsubid = 0
                        AND NOT (d.refclassid = d.classid AND d.refobjid = d.objid)
                        AND ((d.refclassid::regclass::text, d.refobjid) IN
                             (SELECT c::text, o FROM unnest(%s::regclass[], %s::oid[]) u(c, o))
                             OR (d.refclassid::regclass::text, d.refobjid) IN
                             (SELECT c::text, o FROM unnest(%s::regclass[], %s::oid[]) u(c, o)))
```

`outside` line 1536

```sql
SELECT oid FROM pg_trigger WHERE oid = ANY(%s) AND tgisinternal
```

`outside` line 1540

```sql
SELECT oid FROM pg_constraint WHERE oid = ANY(%s) AND conparentid <> 0
```

`outside.described` line 1548

```sql
SELECT u.c::text, u.o, io.type, io.schema, io.name, io.identity
                       FROM unnest(%s::regclass[], %s::oid[]) u(c, o),
                            LATERAL pg_identify_object(u.c, u.o, 0) io
```

**table** - `(module)` line 1596

```sql
ALTER TABLE {fq} OWNER TO {owner}
```

**partitioned** - `(module)` line 1597

```sql
ALTER TABLE {fq} OWNER TO {owner}
```

**view** - `(module)` line 1598

```sql
ALTER VIEW {fq} OWNER TO {owner}
```

**matview** - `(module)` line 1599

```sql
ALTER MATERIALIZED VIEW {fq} OWNER TO {owner}
```

**sequence** - `(module)` line 1600

```sql
ALTER SEQUENCE {fq} OWNER TO {owner}
```

**foreign_table** - `(module)` line 1601

```sql
ALTER FOREIGN TABLE {fq} OWNER TO {owner}
```

**index** - `(module)` line 1604

```sql
ALTER INDEX {fq} OWNER TO {owner}
```

**schema** - `(module)` line 1605

```sql
ALTER SCHEMA {bare} OWNER TO {owner}
```

**function** - `(module)` line 1606

```sql
ALTER FUNCTION {fq}({args}) OWNER TO {owner}
```

**procedure** - `(module)` line 1607

```sql
ALTER PROCEDURE {fq}({args}) OWNER TO {owner}
```

**aggregate** - `(module)` line 1608

```sql
ALTER AGGREGATE {fq}({args}) OWNER TO {owner}
```

**window_function** - `(module)` line 1609

```sql
ALTER FUNCTION {fq}({args}) OWNER TO {owner}
```

**routine** - `(module)` line 1610

```sql
ALTER ROUTINE {fq}({args}) OWNER TO {owner}
```

**type** - `(module)` line 1611

```sql
ALTER TYPE {fq} OWNER TO {owner}
```

**domain** - `(module)` line 1612

```sql
ALTER DOMAIN {fq} OWNER TO {owner}
```

**operator** - `(module)` line 1613

```sql
ALTER OPERATOR {op} ({args}) OWNER TO {owner}
```

**opclass** - `(module)` line 1614

```sql
ALTER OPERATOR CLASS {fq} USING {am} OWNER TO {owner}
```

**opfamily** - `(module)` line 1615

```sql
ALTER OPERATOR FAMILY {fq} USING {am} OWNER TO {owner}
```

**collation** - `(module)` line 1616

```sql
ALTER COLLATION {fq} OWNER TO {owner}
```

**conversion** - `(module)` line 1617

```sql
ALTER CONVERSION {fq} OWNER TO {owner}
```

**ts_config** - `(module)` line 1618

```sql
ALTER TEXT SEARCH CONFIGURATION {fq} OWNER TO {owner}
```

**ts_dict** - `(module)` line 1619

```sql
ALTER TEXT SEARCH DICTIONARY {fq} OWNER TO {owner}
```

**extended_stats** - `(module)` line 1620

```sql
ALTER STATISTICS {fq} OWNER TO {owner}
```

**fdw** - `(module)` line 1621

```sql
ALTER FOREIGN DATA WRAPPER {bare} OWNER TO {owner}
```

**foreign_server** - `(module)` line 1622

```sql
ALTER SERVER {bare} OWNER TO {owner}
```

**language** - `(module)` line 1623

```sql
ALTER LANGUAGE {bare} OWNER TO {owner}
```

**publication** - `(module)` line 1624

```sql
ALTER PUBLICATION {bare} OWNER TO {owner}
```

**subscription** - `(module)` line 1625

```sql
ALTER SUBSCRIPTION {bare} OWNER TO {owner}
```

**event_trigger** - `(module)` line 1626

```sql
ALTER EVENT TRIGGER {bare} OWNER TO {owner}
```

**large_object** - `(module)` line 1627

```sql
ALTER LARGE OBJECT {loid} OWNER TO {owner}
```

**database** - `(module)` line 1628

```sql
ALTER DATABASE {bare} OWNER TO {owner}
```

**tablespace** - `(module)` line 1629

```sql
ALTER TABLESPACE {bare} OWNER TO {owner}
```

`describe` line 1779

```sql
SELECT e.oid, n.nspname, e.extname, e.extversion, pg_get_userbyid(e.extowner)
                           FROM pg_extension e JOIN pg_namespace n ON n.oid = e.extnamespace
                          WHERE e.oid = ANY(%s)
```

`describe` line 1789

```sql
SELECT oid, datname, pg_get_userbyid(datdba) FROM pg_database WHERE oid = ANY(%s)
```

`describe` line 1802

```sql
SELECT io.type, io.schema, io.name, io.identity FROM pg_identify_object(%s::regclass, %s, 0) io
```

`owned_by` line 1854

```sql
SELECT oid, rolname FROM pg_roles
```

`owned_by` line 1856

```sql
SELECT oid, rolname FROM pg_roles WHERE rolname = ANY(%s)
```

`owned_by` line 1862

```sql
SELECT oid FROM pg_database WHERE datname = current_database()
```

`owned_by` line 1871

```sql
SELECT sd.classid::regclass::text, sd.objid, io.type, io.schema, io.name, io.identity,
       sd.refobjid
  FROM (SELECT classid, objid, refobjid FROM pg_shdepend
         WHERE deptype = 'o' AND refobjid = ANY(%s) AND dbid = ANY(%s) OFFSET 0) sd,
       LATERAL pg_identify_object(sd.classid, sd.objid, 0) io
 ORDER BY 1, 6
```

`owners` line 1944

```sql
SELECT x.oid, pg_get_userbyid(x.{col}) FROM {cat} x WHERE x.oid = ANY(%s)
```

`_bootstrap_owned` line 1961

```sql
SELECT x.oid, io.type, io.schema, io.name, io.identity
  FROM {cat} x, LATERAL pg_identify_object('{addr}'::regclass, x.oid, 0) io
 WHERE x.{col} = %s AND x.oid >= %s AND NOT {_internal(addr, 'x')}
```

`find` line 2026

```sql
SELECT oid FROM pg_largeobject_metadata WHERE oid::text = %s
```

`find` line 2035

```sql
SELECT x.oid FROM {cat} x WHERE {where}
```

`partitions` line 2062

```sql
WITH RECURSIVE tree AS (
    SELECT i.inhrelid AS oid, 1 AS lvl FROM pg_inherits i WHERE i.inhparent = %s
    UNION ALL
    SELECT i.inhrelid, t.lvl + 1 FROM pg_inherits i JOIN tree t ON i.inhparent = t.oid)
SELECT c.oid, n.nspname, c.relname, c.relkind::text, pg_get_userbyid(c.relowner), tree.lvl
  FROM tree JOIN pg_class c ON c.oid = tree.oid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.relispartition AND c.relkind IN ('r', 'p', 'f')
 ORDER BY tree.lvl, n.nspname, c.relname
```

`cluster_objects_of` line 2083

```sql
SELECT DISTINCT ft.ftserver FROM pg_foreign_table ft
                   JOIN pg_class t ON t.oid = ft.ftrelid
                   JOIN pg_namespace n ON n.oid = t.relnamespace WHERE n.nspname = ANY(%s)
```

`cluster_objects_of` line 2092

```sql
SELECT oid FROM pg_publication WHERE puballtables
                      UNION SELECT pr.prpubid FROM pg_publication_rel pr
                              JOIN pg_class t ON t.oid = pr.prrelid
                              JOIN pg_namespace n ON n.oid = t.relnamespace WHERE n.nspname = ANY(%(s)s)
                      {by_schema}
```

**schema** - `acl_sources` line 2178

```sql
SELECT 'SCHEMA' AS kw, quote_ident(n.nspname) AS obj, NULL::text AS col, 0 AS colnum,
       n.nspname::text AS nsp, n.nspname::text AS name, ''::text AS args,
       {acl('n.nspacl', "acldefault('n', n.nspowner)")} AS acl,
       COALESCE({ip(ver, 'pg_namespace', 'n.oid')}, acldefault('n', n.nspowner)) AS base,
       'pg_namespace'::text AS classid, n.oid AS objoid, n.nspowner AS owner,
       n.nspacl IS NOT NULL AS touched, NULL::text AS subkind
  FROM pg_catalog.pg_namespace n
 WHERE {isset('n.nspacl')} AND {ok('n.nspname')}
```

**rel** - `acl_sources` line 2187

```sql
SELECT CASE WHEN c.relkind = 'S' THEN 'SEQUENCE' ELSE 'TABLE' END, {rel_name}, NULL::text, 0,
       n.nspname::text, c.relname::text, ''::text,
       {acl('c.relacl', f'acldefault({rel_letter}, c.relowner)')},
       COALESCE({ip(ver, 'pg_class', 'c.oid')}, acldefault({rel_letter}, c.relowner)),
       'pg_class'::text, c.oid, c.relowner, c.relacl IS NOT NULL, c.relkind::text
  FROM pg_catalog.pg_class c JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
 WHERE {isset('c.relacl')} AND c.relkind IN ('r', 'p', 'v', 'm', 'f', 'S')
   AND {ok('n.nspname')}
```

**col** - `acl_sources` line 2199

```sql
SELECT 'TABLE', {rel_name}, quote_ident(a.attname), a.attnum::int,
       n.nspname::text, c.relname::text, ''::text, a.attacl,
       {ip(ver, 'pg_class', 'c.oid', 'a.attnum')},
       'pg_class'::text, c.oid, c.relowner, true, c.relkind::text
  FROM pg_catalog.pg_attribute a
  JOIN pg_catalog.pg_class c ON c.oid = a.attrelid
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
 WHERE a.attacl IS NOT NULL AND a.attnum > 0 AND NOT a.attisdropped
   AND c.relkind IN ('r', 'p', 'v', 'm', 'f') AND {ok('n.nspname')}
```

**routine** - `acl_sources` line 2211

```sql
SELECT {routine_kw},
       quote_ident(n.nspname) || '.' || quote_ident(p.proname)
         || '(' || pg_get_function_identity_arguments(p.oid) || ')', NULL::text, 0,
       n.nspname::text, p.proname::text, pg_get_function_identity_arguments(p.oid),
       {acl('p.proacl', "acldefault('f', p.proowner)")},
       COALESCE({ip(ver, 'pg_proc', 'p.oid')}, acldefault('f', p.proowner)),
       'pg_proc'::text, p.oid, p.proowner, p.proacl IS NOT NULL, {prokind}
  FROM pg_catalog.pg_proc p JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace
 WHERE {isset('p.proacl')} AND {ok('n.nspname')}
```

**type** - `acl_sources` line 2223

```sql
SELECT CASE WHEN t.typtype = 'd' THEN 'DOMAIN' ELSE 'TYPE' END,
       quote_ident(n.nspname) || '.' || quote_ident(t.typname), NULL::text, 0,
       n.nspname::text, t.typname::text, ''::text,
       {acl('t.typacl', "acldefault('T', t.typowner)")},
       COALESCE({ip(ver, 'pg_type', 't.oid')}, acldefault('T', t.typowner)),
       'pg_type'::text, t.oid, t.typowner, t.typacl IS NOT NULL, t.typtype::text
  FROM pg_catalog.pg_type t JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace
 WHERE {isset('t.typacl')} AND NOT (t.typcategory = 'A' AND t.typelem <> 0)
   AND {type_filter} AND {ok('n.nspname')}
```

**language** - `acl_sources` line 2233

```sql
SELECT 'LANGUAGE', quote_ident(l.lanname), NULL::text, 0, NULL::text, l.lanname::text, ''::text,
       {acl('l.lanacl', "acldefault('l', l.lanowner)")},
       COALESCE({ip(ver, 'pg_language', 'l.oid')}, acldefault('l', l.lanowner)),
       'pg_language'::text, l.oid, l.lanowner, l.lanacl IS NOT NULL, NULL::text
  FROM pg_catalog.pg_language l WHERE {isset('l.lanacl')}
```

**fdw** - `acl_sources` line 2239

```sql
SELECT 'FOREIGN DATA WRAPPER', quote_ident(w.fdwname), NULL::text, 0, NULL::text, w.fdwname::text,
       ''::text, {acl('w.fdwacl', "acldefault('F', w.fdwowner)")},
       COALESCE({ip(ver, 'pg_foreign_data_wrapper', 'w.oid')}, acldefault('F', w.fdwowner)),
       'pg_foreign_data_wrapper'::text, w.oid, w.fdwowner, w.fdwacl IS NOT NULL, NULL::text
  FROM pg_catalog.pg_foreign_data_wrapper w WHERE {isset('w.fdwacl')}
```

**server** - `acl_sources` line 2245

```sql
SELECT 'FOREIGN SERVER', quote_ident(s.srvname), NULL::text, 0, NULL::text, s.srvname::text,
       ''::text, {acl('s.srvacl', "acldefault('S', s.srvowner)")},
       COALESCE({ip(ver, 'pg_foreign_server', 's.oid')}, acldefault('S', s.srvowner)),
       'pg_foreign_server'::text, s.oid, s.srvowner, s.srvacl IS NOT NULL, NULL::text
  FROM pg_catalog.pg_foreign_server s WHERE {isset('s.srvacl')}
```

**lo** - `acl_sources` line 2251

```sql
SELECT 'LARGE OBJECT', m.oid::text, NULL::text, 0, NULL::text, m.oid::text, ''::text,
       {acl('m.lomacl', "acldefault('L', m.lomowner)")}, acldefault('L', m.lomowner),
       'pg_largeobject'::text, m.oid, m.lomowner, m.lomacl IS NOT NULL, NULL::text
  FROM pg_catalog.pg_largeobject_metadata m WHERE {isset('m.lomacl')}
```

**database** - `acl_sources` line 2257

```sql
SELECT 'DATABASE', quote_ident(d.datname), NULL::text, 0, NULL::text, d.datname::text, ''::text,
       {acl('d.datacl', "acldefault('d', d.datdba)")}, acldefault('d', d.datdba),
       'pg_database'::text, d.oid, d.datdba, d.datacl IS NOT NULL, NULL::text
  FROM pg_catalog.pg_database d WHERE {isset('d.datacl')}
```

**tablespace** - `acl_sources` line 2262

```sql
SELECT 'TABLESPACE', quote_ident(t.spcname), NULL::text, 0, NULL::text, t.spcname::text, ''::text,
       {acl('t.spcacl', "acldefault('t', t.spcowner)")}, acldefault('t', t.spcowner),
       'pg_tablespace'::text, t.oid, t.spcowner, t.spcacl IS NOT NULL, NULL::text
  FROM pg_catalog.pg_tablespace t WHERE {isset('t.spcacl')}
```

`acl_sources` line 2270

```sql
SELECT 'PARAMETER', p.parname::text, NULL::text, 0, NULL::text, p.parname::text, ''::text,
       p.paracl, acldefault('p', 10::oid), 'pg_parameter_acl'::text, p.oid, 10::oid,
       true, NULL::text
  FROM pg_catalog.pg_parameter_acl p WHERE p.paracl IS NOT NULL
```

`default_acl_changes` line 2314

```sql
SELECT d.defaclrole, quote_ident(n.nspname), d.defaclobjtype::text, '{side}'::text, x.grantee, x.privilege_type, x.is_grantable, n.nspname::text FROM pg_catalog.pg_default_acl d LEFT JOIN pg_catalog.pg_namespace n ON n.oid = d.defaclnamespace, aclexplode({acl}) x WHERE ({where})
```

`_role_names` line 2341

```sql
SELECT oid, rolname FROM pg_catalog.pg_roles
```

`acl_changes` line 2383

```sql
SELECT '{name}'::text, o.kw, o.obj, o.col, o.colnum, '{side}'::text, x.grantee, x.privilege_type, x.is_grantable, o.classid, o.objoid, o.owner, o.nsp, o.name FROM ({every[name]}) o({ACL_COLS}), aclexplode(o.{column}) x WHERE ({where}){only}
```

`held` line 2438

```sql
SELECT '{name}'::text, o.kw, o.obj, o.col, o.nsp, o.name, o.args, o.classid, o.objoid, o.owner, o.touched, o.subkind, x.grantee, x.grantor, x.privilege_type, x.is_grantable FROM ({every[name]}) o({ACL_COLS}), aclexplode(o.acl) x WHERE x.grantee = ANY(%(grantee)s) AND ({where})
```

`object_acls` line 2572

```sql
SELECT '{name}'::text, '{side}'::text, o.kw, o.obj, o.col, o.nsp, o.name, o.args, o.classid, o.objoid, o.owner, o.touched, o.subkind, x.grantee, x.privilege_type, x.is_grantable FROM ({every[name]}) o({ACL_COLS}) LEFT JOIN LATERAL aclexplode(o.{column}) x ON true WHERE ({where})
```

`acl_copy_sql.plan` line 2649

```sql
GRANT {priv} ON {target} TO {who(g)}{' WITH GRANT OPTION' if w else ''}
```

`held_defaults` line 2679

```sql
SELECT pg_get_userbyid(d.defaclrole), COALESCE(n.nspname, ''),
                        d.defaclobjtype::text, x.privilege_type
                   FROM pg_catalog.pg_default_acl d
                   LEFT JOIN pg_catalog.pg_namespace n ON n.oid = d.defaclnamespace,
                        aclexplode(d.defaclacl) x
                  WHERE x.grantee = %s
```

`default_acls` line 2699

```sql
SELECT d.oid, d.defaclrole, n.nspname, quote_ident(n.nspname), d.defaclobjtype::text
                    FROM pg_default_acl d LEFT JOIN pg_namespace n ON n.oid = d.defaclnamespace
                   WHERE {where} ORDER BY 2, 3 NULLS FIRST, 5
```

**head** - `default_acls` line 2711

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE {_ident_str(nm(role))}{f' IN SCHEMA {qn}' if qn else ''}
```

## patroni.py

`detect` line 79

```sql
SELECT pg_ls_dir('.') AS f
```

`detect` line 96

```sql
SELECT current_setting('cluster_name', true)                       AS cluster_name,
       (SELECT count(*) FROM pg_replication_slots)                 AS slots,
       (SELECT count(*) FROM pg_stat_replication)                  AS standbys
```

## pgconn.py

`_apply_conn_role` line 253

```sql
SET ROLE {_qi(role)}
```

`_server_version_num` line 291

```sql
SHOW server_version_num
```

`_existing_catalog_views` line 310

```sql
SELECT v FROM unnest(%s::text[]) v WHERE to_regclass('pg_catalog.' || v) IS NOT NULL
```

`_has_table_priv` line 336

```sql
SELECT COALESCE(has_table_privilege(to_regclass(%s), %s), false)
```

`_has_func_priv` line 341

```sql
SELECT COALESCE(has_function_privilege(to_regprocedure(%s), 'EXECUTE'), false)
```

`_has_role` line 348

```sql
SELECT current_setting('is_superuser') = 'on' OR COALESCE((SELECT pg_has_role(current_user, oid, 'USAGE') FROM pg_roles WHERE rolname = %s), false)
```

**_TEMPLATE_PROBE_SQL** - `(module)` line 495

```sql
SELECT d.datistemplate FROM pg_catalog.pg_database d WHERE d.datname = pg_catalog.current_database()
```

`_conn_in_recovery` line 623

```sql
SELECT pg_is_in_recovery()
```

`_connect` line 733

```sql
SET lock_timeout = '{int(lock_timeout_ms)}ms'
```

`_connect` line 734

```sql
SET statement_timeout = '{_stmt_to}'
```

`_connect` line 735

```sql
SET application_name = 'DBApguin'
```

`_connect` line 755

```sql
RESET search_path
```

`_connect` line 767

```sql
SET ROLE {_qi(ctx_role)}
```

`_connect` line 769

```sql
SET ROLE {_qi(conn_role)}
```

`_connect` line 791

```sql
SELECT pg_catalog.set_config('search_path', {_literal_str(quoted)} || ', ' || pg_catalog.current_setting('search_path'), false)
```

## plain_dump.py

`rewrite` line 214

```sql
SET default_tablespace = '';
```

## preimage.py

`_dump_relations` line 157

```sql
SELECT pg_catalog.set_config('search_path'
```

`_dump_relations` line 161

```sql
SET default_tablespace = '';
```

`_acl_lines` line 209

```sql
REVOKE {a.privilege} ON {kw} {ref} FROM {_grantee(a.grantee)};
```

`_acl_lines` line 210

```sql
GRANT {a.privilege} ON {kw} {ref} TO {_grantee(a.grantee)}{' WITH GRANT OPTION' if a.grantable else ''};
```

`_acl_lines` line 213

```sql
GRANT {a.privilege} ({_ident(col)}) ON {kw} {ref} TO {_grantee(a.grantee)}{' WITH GRANT OPTION' if a.grantable else ''};
```

`_server_version_num` line 231

```sql
SHOW server_version_num
```

`_seq_params` line 240

```sql
SELECT format_type(seqtypid, NULL), seqstart, seqmin, seqmax, seqincrement, seqcycle, seqcache FROM pg_sequence WHERE seqrelid = %s
```

`_seq_params` line 245

```sql
SELECT 'bigint', start_value, min_value, max_value, increment_by, is_cycled, cache_value FROM {ref}
```

`_setval` line 255

```sql
SELECT has_sequence_privilege(%s, 'SELECT')
```

`_setval` line 258

```sql
SELECT last_value, is_called FROM {ref}
```

`_setval` line 260

```sql
SELECT setval('{ref}'::regclass, {last}, {'true' if called else 'false'});
```

`_sequence_lines` line 267

```sql
SELECT c.oid, pg_get_userbyid(c.relowner), c.relpersistence, obj_description(c.oid, 'pg_class') FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname = %s AND c.relkind = 'S'
```

`_sequence_lines` line 280

```sql
CREATE {'UNLOGGED ' if persistence == 'u' else ''}SEQUENCE {ref}{astype} START WITH {start} INCREMENT BY {inc} MINVALUE {mn} MAXVALUE {mx} CACHE {cache}{' CYCLE' if cyc else ''};
```

`_sequence_lines` line 283

```sql
ALTER SEQUENCE {ref} OWNER TO {_ident(owner)};
```

`_sequence_lines` line 285

```sql
COMMENT ON SEQUENCE {ref} IS {_lit(comment)};
```

`_catalog_relation_defs` line 350

```sql
SELECT c.oid, c.relkind, pg_get_userbyid(c.relowner),
       obj_description(c.oid, 'pg_class'),
       c.reloptions, c.relpersistence, c.relreplident, {am_col}
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
```

`_catalog_relation_defs` line 396

```sql
SELECT a.attname, pg_get_expr(d.adbin, d.adrelid), a.attnotnull, a.attstorage, {compression_col}
  FROM pg_attribute a LEFT JOIN pg_attrdef d ON d.adrelid = a.attrelid AND d.adnum = a.attnum
 WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped
```

`_catalog_relation_defs` line 408

```sql
SELECT obj_description(%s, 'pg_class')
```

`_catalog_relation_defs` line 411

```sql
SELECT a.attnum, a.attname, format_type(a.atttypid, a.atttypmod), a.attnotnull,
       pg_get_expr(d.adbin, d.adrelid), {identity_col}, {generated_col},
       CASE WHEN a.attcollation <> 0 AND a.attcollation <> t.typcollation
            THEN {_COLL_SQL.format(oid='a.attcollation')} END,
       col_description(a.attrelid, a.attnum), a.attislocal, a.attinhcount,
       a.attstorage, t.typstorage, CASE WHEN a.attstattarget >= 0 THEN a.attstattarget END,
       a.attoptions, a.attfdwoptions, {compression_col}
  FROM pg_attribute a JOIN pg_type t ON t.oid = a.atttypid
  LEFT JOIN pg_attrdef d ON d.adrelid = a.attrelid AND d.adnum = a.attnum
 WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

`_catalog_relation_defs` line 456

```sql
ALTER {word} {cref} {f'SET DEFAULT {default};' if default else 'DROP DEFAULT;'}
```

`_catalog_relation_defs` line 458

```sql
ALTER {word} {cref} SET NOT NULL;
```

`_catalog_relation_defs` line 464

```sql
ALTER TABLE {cref} SET STORAGE {_STORAGE.get(storage, 'EXTENDED')};
```

`_catalog_relation_defs` line 467

```sql
ALTER TABLE {cref} SET COMPRESSION {_COMPRESSION[compression]};
```

`_catalog_relation_defs` line 469

```sql
ALTER {word} {cref} SET STATISTICS {stattarget};
```

`_catalog_relation_defs` line 471

```sql
ALTER {word} {cref} SET ({', '.join(attoptions)});
```

`_catalog_relation_defs` line 473

```sql
ALTER FOREIGN TABLE {cref} OPTIONS ({_fdw_options(fdwoptions)});
```

`_catalog_relation_defs` line 475

```sql
COMMENT ON COLUMN {ref}.{_ident(cn)} IS {_lit(ccomment)};
```

**head** - `_catalog_relation_defs` line 476

```sql
CREATE {'UNLOGGED ' if persistence == 'u' else ''}{'FOREIGN TABLE' if kind == 'f' else 'TABLE'} {ref}
```

`_catalog_relation_defs` line 481

```sql
WITH ({', '.join(reloptions)})
```

`_catalog_relation_defs` line 484

```sql
SELECT s.srvname, f.ftoptions FROM pg_foreign_table f JOIN pg_foreign_server s ON s.oid = f.ftserver WHERE f.ftrelid = %s
```

`_catalog_relation_defs` line 497

```sql
SELECT conname, pg_get_constraintdef(oid, true), conislocal FROM pg_constraint WHERE conrelid = %s AND contype IN ('p','u','c','f','x') {own}ORDER BY contype DESC, conname
```

**line** - `_catalog_relation_defs` line 502

```sql
ALTER {word} {ref} ADD CONSTRAINT {_ident(conname)} {condef};
```

`_catalog_relation_defs` line 506

```sql
SELECT pg_get_indexdef(i.indexrelid), obj_description(i.indexrelid, 'pg_class')
  FROM pg_index i
 WHERE i.indrelid = %s
   AND NOT EXISTS (SELECT 1 FROM pg_constraint k WHERE k.conindid = i.indexrelid AND k.contype IN ('p','u','x'))
   AND NOT EXISTS (SELECT 1 FROM pg_inherits h WHERE h.inhrelid = i.indexrelid)
 ORDER BY 1
```

`_catalog_relation_defs` line 518

```sql
COMMENT ON INDEX {_ident(schema)}.{iname} IS {_lit(icomment)};
```

`_catalog_relation_defs` line 523

```sql
SELECT t.tgname, pg_get_triggerdef(t.oid, true), t.tgenabled, {tgparent_col},
       (SELECT pt.tgenabled FROM pg_trigger pt WHERE pt.oid = {tgparent_col})
  FROM pg_trigger t WHERE t.tgrelid = %s AND (NOT t.tgisinternal OR {tgparent_col} <> 0)
 ORDER BY t.tgname
```

`_catalog_relation_defs` line 532

```sql
ALTER {word} {ref} {_TRIGGER_STATE.get(enabled, 'ENABLE TRIGGER')} {_ident(tgname)};
```

`_catalog_relation_defs` line 536

```sql
SELECT relrowsecurity, relforcerowsecurity FROM pg_class WHERE oid = %s
```

`_catalog_relation_defs` line 539

```sql
ALTER TABLE {ref} ENABLE ROW LEVEL SECURITY;
```

`_catalog_relation_defs` line 541

```sql
ALTER TABLE {ref} FORCE ROW LEVEL SECURITY;
```

`_catalog_relation_defs` line 545

```sql
SELECT polname, {permissive_col}, polcmd,
       ARRAY(SELECT CASE WHEN r = 0 THEN 'public' ELSE pg_get_userbyid(r) END FROM unnest(polroles) r),
       pg_get_expr(polqual, polrelid), pg_get_expr(polwithcheck, polrelid)
  FROM pg_policy WHERE polrelid = %s ORDER BY polname
```

**st** - `_catalog_relation_defs` line 553

```sql
CREATE POLICY {_ident(pname)} ON {ref}{'' if permissive else ' AS RESTRICTIVE'} FOR {cmd.get(pcmd, 'ALL')}
```

`_catalog_relation_defs` line 559

```sql
WITH CHECK ({check})
```

`_catalog_relation_defs` line 563

```sql
SELECT pg_get_ruledef(oid, true) FROM pg_rewrite WHERE ev_class = %s AND rulename <> '_RETURN' ORDER BY rulename
```

`_catalog_relation_defs` line 572

```sql
ALTER {word} {ref} REPLICA IDENTITY FULL;
```

`_catalog_relation_defs` line 574

```sql
ALTER {word} {ref} REPLICA IDENTITY NOTHING;
```

`_catalog_relation_defs` line 576

```sql
SELECT ic.relname FROM pg_index i JOIN pg_class ic ON ic.oid = i.indexrelid WHERE i.indrelid = %s AND i.indisreplident
```

`_catalog_relation_defs` line 580

```sql
ALTER TABLE {ref} REPLICA IDENTITY USING INDEX {_ident(ri[0])};
```

`_catalog_relation_defs` line 584

```sql
SELECT t.spcname FROM pg_class c JOIN pg_tablespace t ON t.oid = c.reltablespace WHERE c.oid = %s
```

`_catalog_relation_defs` line 587

```sql
ALTER TABLE {ref} SET TABLESPACE {_ident(spc[0])};
```

`_catalog_relation_defs` line 588

```sql
SELECT ic.relname, t.spcname FROM pg_index i JOIN pg_class ic ON ic.oid = i.indexrelid
  JOIN pg_tablespace t ON t.oid = ic.reltablespace
 WHERE i.indrelid = %s ORDER BY ic.relname
```

`_catalog_relation_defs` line 593

```sql
ALTER INDEX {_ident(schema)}.{_ident(iname)} SET TABLESPACE {_ident(ispc)};
```

`_catalog_relation_defs` line 594

```sql
ALTER {word} {ref} OWNER TO {_ident(owner)};
```

`_catalog_relation_defs` line 596

```sql
COMMENT ON {word} {ref} IS {_lit(comment)};
```

`_catalog_relation_defs` line 607

```sql
ALTER SEQUENCE {_ident(sschema)}.{_ident(sname)} OWNED BY {ref}.{_ident(col)};
```

`_catalog_relation_defs` line 613

```sql
COMMENT ON SEQUENCE {sref} IS {_lit(scomment)};
```

`_catalog_relation_defs` line 617

```sql
SELECT pg_get_viewdef(%s, true)
```

`_catalog_relation_defs` line 620

```sql
WITH ({', '.join(reloptions)})
```

`_catalog_relation_defs` line 621

```sql
SELECT t.spcname FROM pg_class c JOIN pg_tablespace t ON t.oid = c.reltablespace WHERE c.oid = %s
```

`_catalog_relation_defs` line 623

```sql
CREATE {word} {ref}{vopts}{f' TABLESPACE {_ident(vspc[0])}' if vspc else ''} AS
{body}{' WITH NO DATA' if kind == 'm' else ''};
```

`_catalog_relation_defs` line 626

```sql
SELECT pg_get_indexdef(indexrelid) FROM pg_index WHERE indrelid = %s ORDER BY 1
```

`_catalog_relation_defs` line 631

```sql
SELECT pg_get_ruledef(oid, true) FROM pg_rewrite WHERE ev_class = %s AND rulename <> '_RETURN' ORDER BY rulename
```

`_catalog_relation_defs` line 639

```sql
SELECT a.attname, pg_get_expr(d.adbin, d.adrelid), col_description(a.attrelid, a.attnum)
                             FROM pg_attribute a LEFT JOIN pg_attrdef d ON d.adrelid = a.attrelid AND d.adnum = a.attnum
                            WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped
                            ORDER BY a.attnum
```

`_catalog_relation_defs` line 646

```sql
ALTER VIEW {ref} ALTER COLUMN {_ident(cn)} SET DEFAULT {vdefault};
```

`_catalog_relation_defs` line 648

```sql
COMMENT ON COLUMN {ref}.{_ident(cn)} IS {_lit(ccomment)};
```

`_catalog_relation_defs` line 649

```sql
SELECT pg_get_triggerdef(oid, true) FROM pg_trigger WHERE tgrelid = %s AND NOT tgisinternal ORDER BY tgname
```

`_catalog_relation_defs` line 653

```sql
ALTER {word} {ref} OWNER TO {_ident(owner)};
```

`_catalog_relation_defs` line 655

```sql
COMMENT ON {word} {ref} IS {_lit(comment)};
```

`_index_defs` line 691

```sql
SELECT i.indexrelid, pg_get_indexdef(i.indexrelid), obj_description(i.indexrelid, 'pg_class'),
       k.conname, pg_get_constraintdef(k.oid, true), tn.nspname, t.relname
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_index i ON i.indexrelid = c.oid
  JOIN pg_class t ON t.oid = i.indrelid JOIN pg_namespace tn ON tn.oid = t.relnamespace
  LEFT JOIN pg_constraint k ON k.conindid = i.indexrelid AND k.contype IN ('p', 'u', 'x')
 WHERE n.nspname = %s AND c.relname = %s
```

`_index_defs` line 706

```sql
ALTER TABLE {_ident(tschema)}.{_ident(tname)} ADD CONSTRAINT {_ident(conname)} {condef};
```

`_index_defs` line 710

```sql
COMMENT ON INDEX {_ident(schema)}.{_ident(name)} IS {_lit(comment)};
```

`_type_defs` line 722

```sql
SELECT t.oid, t.typtype, pg_get_userbyid(t.typowner), obj_description(t.oid, 'pg_type'),
       t.typrelid, format_type(t.typbasetype, t.typtypmod), t.typnotnull, t.typdefault,
       CASE WHEN t.typcollation <> 0 AND t.typcollation <> 100 THEN {_COLL_SQL.format(oid='t.typcollation')} END
  FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace
 WHERE n.nspname = %s AND t.typname = %s
```

`_type_defs` line 736

```sql
SELECT enumlabel FROM pg_enum WHERE enumtypid = %s ORDER BY enumsortorder
```

`_type_defs` line 738

```sql
CREATE TYPE {ref} AS ENUM ({labels});
```

**line** - `_type_defs` line 741

```sql
CREATE DOMAIN {ref} AS {basetype}
```

`_type_defs` line 750

```sql
SELECT conname, pg_get_constraintdef(oid, true) FROM pg_constraint WHERE contypid = %s AND contype <> 'n' ORDER BY conname
```

`_type_defs` line 756

```sql
SELECT a.attname, format_type(a.atttypid, a.atttypmod),
       CASE WHEN a.attcollation <> 0 AND a.attcollation <> ty.typcollation
            THEN {_COLL_SQL.format(oid='a.attcollation')} END
  FROM pg_attribute a JOIN pg_type ty ON ty.oid = a.atttypid
 WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped ORDER BY a.attnum
```

`_type_defs` line 764

```sql
CREATE TYPE {ref} AS (
{attrs}
);
```

`_type_defs` line 773

```sql
SELECT format_type(r.rngsubtype, NULL),
       (SELECT CASE WHEN NOT oc.opcdefault THEN quote_ident(ocn.nspname) || '.' || quote_ident(oc.opcname) END
          FROM pg_opclass oc JOIN pg_namespace ocn ON ocn.oid = oc.opcnamespace WHERE oc.oid = r.rngsubopc),
       CASE WHEN r.rngcollation <> 0 THEN {_COLL_SQL.format(oid='r.rngcollation')} END,
       CASE WHEN r.rngcanonical <> 0 THEN r.rngcanonical::regproc::text END,
       CASE WHEN r.rngsubdiff <> 0 THEN r.rngsubdiff::regproc::text END,
       {multi}
  FROM pg_range r WHERE r.rngtypid = %s
```

`_type_defs` line 795

```sql
CREATE TYPE {ref} AS RANGE ({', '.join(parts)});
```

`_type_defs` line 799

```sql
ALTER {word} {ref} OWNER TO {_ident(owner)};
```

`_type_defs` line 803

```sql
COMMENT ON {word} {ref} IS {_lit(comment)};
```

`_multirange_owners` line 817

```sql
SELECT quote_ident(n.nspname) || '.' || quote_ident(t.typname), pg_get_userbyid(t.typowner)
  FROM pg_range r JOIN pg_type t ON t.oid = r.rngmultitypid JOIN pg_namespace n ON n.oid = t.typnamespace
 WHERE r.rngtypid = ANY(%s::oid[]) ORDER BY t.oid
```

`_multirange_owners` line 821

```sql
ALTER TYPE {mref} OWNER TO {_ident(owner)};
```

`_schema_contents` line 830

```sql
SELECT oid, pg_get_userbyid(nspowner), obj_description(oid, 'pg_namespace') FROM pg_namespace WHERE nspname = %s
```

`_schema_contents` line 851

```sql
SELECT DISTINCT d.objid FROM pg_depend d JOIN pg_type t ON t.oid = d.refobjid
  LEFT JOIN pg_type te ON te.oid = t.typelem
  JOIN pg_class c ON c.oid = CASE WHEN t.typrelid <> 0 THEN t.typrelid ELSE te.typrelid END
 WHERE d.classid = 'pg_proc'::regclass AND d.objid = ANY(%s) AND d.refclassid = 'pg_type'::regclass
   AND c.relkind <> 'c' AND c.relnamespace = %s
```

`_schema_contents` line 867

```sql
CREATE SCHEMA {_ident(schema)} AUTHORIZATION {_ident(owner)};
```

`_schema_contents` line 869

```sql
COMMENT ON SCHEMA {_ident(schema)} IS {_lit(comment)};
```

`_collation_defs` line 887

```sql
SELECT c.collname, pg_get_userbyid(c.collowner), obj_description(c.oid, 'pg_collation'),
       {prov}, c.collcollate, c.collctype, {loc}, {det}, {rules}
  FROM pg_collation c JOIN pg_namespace n ON n.oid = c.collnamespace
 WHERE n.nspname = %s ORDER BY c.oid
```

`_collation_defs` line 908

```sql
CREATE COLLATION {ref} ({', '.join(opts)});
```

`_collation_defs` line 909

```sql
ALTER COLLATION {ref} OWNER TO {_ident(owner)};
```

`_collation_defs` line 911

```sql
COMMENT ON COLLATION {ref} IS {_lit(comment)};
```

**head** - `_default_acl_lines` line 925

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE {_ident(d['role'])} IN SCHEMA {_ident(schema)}
```

`_aggregate_def` line 942

```sql
SELECT pg_get_function_arguments(a.aggfnoid), p.pronargs, a.aggkind,
       {fn('aggtransfn')}, format_type(a.aggtranstype, NULL), a.aggtransspace, a.agginitval,
       {fn('aggfinalfn')}, a.aggfinalextra, {'a.aggfinalmodify' if v11 else 'NULL'},
       {fn('aggcombinefn') if v96 else 'NULL'}, {fn('aggserialfn') if v96 else 'NULL'},
       {fn('aggdeserialfn') if v96 else 'NULL'},
       {fn('aggmtransfn')}, {fn('aggminvtransfn')},
       CASE WHEN a.aggmtranstype <> 0 THEN format_type(a.aggmtranstype, NULL) END, a.aggmtransspace,
       a.aggminitval, {fn('aggmfinalfn')}, a.aggmfinalextra, {'a.aggmfinalmodify' if v11 else 'NULL'},
       (SELECT 'OPERATOR(' || quote_ident(n.nspname) || '.' || o.oprname || ')'
          FROM pg_operator o JOIN pg_namespace n ON n.oid = o.oprnamespace WHERE o.oid = a.aggsortop),
       {'p.proparallel' if v96 else "'u'"}
  FROM pg_aggregate a JOIN pg_proc p ON p.oid = a.aggfnoid WHERE a.aggfnoid = %s
```

`_aggregate_def` line 997

```sql
CREATE AGGREGATE {ref}({'*' if nargs == 0 else args}) (
    {',\n    '.join(opts)}
);
```

`_routine_defs` line 1012

```sql
SELECT p.oid, {kind_col}, pg_get_userbyid(p.proowner), obj_description(p.oid, 'pg_proc'),
       pg_get_function_identity_arguments(p.oid), p.pronargs, oidvectortypes(p.proargtypes)
  FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname = %s AND p.proname = %s
   AND (%s = '' OR pg_get_function_identity_arguments(p.oid) = %s)
 ORDER BY p.oid
```

`_routine_defs` line 1036

```sql
SELECT pg_get_functiondef(%s)
```

`_routine_defs` line 1041

```sql
ALTER {word} {rref} OWNER TO {_ident(owner)};
```

`_routine_defs` line 1043

```sql
COMMENT ON {word} {rref} IS {_lit(comment)};
```

`_schema_defs` line 1054

```sql
SELECT pg_get_userbyid(nspowner), nspacl::text FROM pg_namespace WHERE nspname = %s
```

`_schema_defs` line 1059

```sql
ALTER SCHEMA {_ident(name)} OWNER TO {_ident(row[0])};
```

`_owner_defs` line 1076

```sql
SELECT pg_get_userbyid(x.{ocol}) FROM {cat} x
  JOIN pg_namespace n ON n.oid = x.{nspcol}
 WHERE n.nspname = %s AND x.{ncol} = %s
```

`_inbound_fks` line 1098

```sql
SELECT DISTINCT tn.nspname, t.relname, k.conname, pg_get_constraintdef(k.oid, true)
  FROM pg_constraint k
  JOIN pg_class r ON r.oid = k.confrelid JOIN pg_namespace rn ON rn.oid = r.relnamespace
  JOIN pg_class t ON t.oid = k.conrelid JOIN pg_namespace tn ON tn.oid = t.relnamespace
 WHERE k.contype = 'f' AND (rn.nspname, r.relname) IN %s AND (tn.nspname, t.relname) NOT IN %s
 ORDER BY 1, 2, 3
```

`_inbound_fks` line 1111

```sql
ALTER TABLE {_ident(tschema)}.{_ident(tname)} ADD CONSTRAINT {_ident(conname)} {condef};
```

`_rel_oids` line 1120

```sql
SELECT n.nspname, c.relname, c.oid, c.relkind::text
                     FROM unnest(%s::text[], %s::text[]) u(s, r)
                     JOIN pg_namespace n ON n.nspname = u.s
                     JOIN pg_class c ON c.relnamespace = n.oid AND c.relname = u.r
```

`_written_after_all` line 1219

```sql
SELECT refclassid::regclass::text, refobjid FROM pg_depend WHERE classid = %s::regclass AND objid = %s AND deptype = 'i'
```

`_written_after_all` line 1225

```sql
SELECT n.nspname FROM pg_default_acl d JOIN pg_namespace n ON n.oid = d.defaclnamespace WHERE d.oid = %s
```

`_written_after_all` line 1228

```sql
SELECT schema FROM pg_identify_object(%s::regclass, %s, %s)
```

`server_identity` line 1254

```sql
SELECT current_database()
```

`server_identity` line 1258

```sql
SHOW server_version_num
```

`server_identity` line 1261

```sql
SELECT system_identifier::text FROM pg_control_system()
```

`capture` line 1412

```sql
SHOW search_path
```

`capture` line 1416

```sql
SELECT pg_catalog.set_config('search_path', 'pg_catalog', false)
```

`capture` line 1422

```sql
SELECT to_regclass(%s)::oid
```

`capture` line 1478

```sql
SELECT r.rngtypid FROM pg_range r JOIN pg_type t ON t.oid = r.rngtypid JOIN pg_namespace n ON n.oid = t.typnamespace WHERE n.nspname = ANY(%s)
```

`capture` line 1494

```sql
SELECT s.oid FROM pg_statistic_ext s JOIN pg_namespace n ON n.oid = s.stxnamespace WHERE s.oid = ANY(%s) AND n.nspname = ANY(%s)
```

`capture` line 1521

```sql
SET check_function_bodies = false;
{_routine_defs(cur, cascaded_routines)}
```

`capture` line 1534

```sql
SELECT relkind FROM pg_class WHERE oid = to_regclass(%s)
```

`capture` line 1539

```sql
REFRESH MATERIALIZED VIEW {_ident(schema)}.{_ident(name)};
```

`capture` line 1559

```sql
SELECT pg_catalog.set_config('search_path', %s, false)
```

`capture` line 1577

```sql
SET check_function_bodies = false;
```

## preimage_cascade.py

`roots` line 115

```sql
SELECT 'pg_namespace', oid FROM pg_namespace WHERE nspname = %s
```

`roots` line 120

```sql
SELECT 'pg_proc', p.oid FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname = %s AND p.proname = %s
   AND (pg_get_function_identity_arguments(p.oid) = %s
        OR oidvectortypes(p.proargtypes) = %s)
 ORDER BY p.oid LIMIT 1
```

`roots` line 127

```sql
SELECT 'pg_type', to_regtype(%s)::oid
```

`roots` line 131

```sql
SELECT %s, oid FROM pg_catalog.{cat} WHERE {rel} = to_regclass(%s) AND {col} = %s
```

`roots` line 136

```sql
SELECT %s, x.oid FROM pg_catalog.{cat} x JOIN pg_namespace n ON n.oid = x.{nsp} WHERE n.nspname = %s AND x.{col} = %s
```

`roots` line 139

```sql
SELECT %s, oid FROM pg_catalog.{cat} WHERE {col} = %s
```

`roots` line 141

```sql
SELECT 'pg_class', to_regclass(%s)::oid
```

`_gone` line 171

```sql
SELECT %s::text, u.o, 0 FROM unnest(%s::oid[]) AS u(o) WHERE NOT EXISTS (SELECT 1 FROM pg_catalog.{cat} x WHERE x.oid = u.o)
```

`_gone` line 175

```sql
SELECT 'pg_class', u.r, u.a FROM unnest(%s::oid[], %s::int[]) AS u(r, a) WHERE NOT EXISTS (SELECT 1 FROM pg_catalog.pg_attribute x WHERE x.attrelid = u.r AND x.attnum = u.a AND NOT x.attisdropped)
```

`closure` line 215

```sql
SELECT count(*) FROM pg_event_trigger WHERE evtenabled <> 'D' AND evtevent IN ('ddl_command_start', 'ddl_command_end', 'sql_drop')
```

`closure` line 222

```sql
SET LOCAL lock_timeout = '3s'
```

`closure` line 226

```sql
SET LOCAL statement_timeout = '10s'
```

`closure` line 228

```sql
SET LOCAL idle_in_transaction_session_timeout = '5s'
```

**_WHOLE_SQL** - `(module)` line 252

```sql
WITH s(c, o, sub) AS (SELECT * FROM unnest(%s::regclass[], %s::oid[], %s::int[]))
SELECT s.c::text, s.o, s.sub FROM s
 WHERE NOT (s.sub = 0 AND EXISTS (
         SELECT 1 FROM pg_depend d JOIN s p ON p.c = d.refclassid AND p.o = d.refobjid AND p.sub = 0
          WHERE d.classid = s.c AND d.objid = s.o AND d.objsubid = 0
            AND d.deptype IN ('a', 'i', 'P', 'S', 'e', 'x')
            AND NOT (d.refclassid = d.classid AND d.refobjid = d.objid)))
   AND NOT (s.sub <> 0 AND EXISTS (SELECT 1 FROM s p WHERE p.c = s.c AND p.o = s.o AND p.sub = 0))
```

**_RELS_SQL** - `(module)` line 269

```sql
SELECT c.oid, n.nspname, c.relname, c.relkind::text, greatest(c.reltuples, 0)::float8,
       (c.relpages::bigint + coalesce(t.relpages, 0)) * current_setting('block_size')::bigint,
       c.relkind = 'r' AND (c.reltuples < 0 OR (c.relpages = 0 AND (coalesce(st.live, 0) > 0
                                                                 OR (%s AND st.last IS NULL))))
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_class t ON t.oid = c.reltoastrelid
  LEFT JOIN LATERAL (SELECT greatest(last_vacuum, last_autovacuum, last_analyze, last_autoanalyze) AS last,
                            n_live_tup AS live
                       FROM pg_stat_all_tables WHERE relid = c.oid) st ON true
 WHERE c.oid = ANY(%s) AND c.relkind IN ('r', 'p', 'v', 'm', 'f', 'S')
```

`classify` line 358

```sql
SELECT c.oid, n.nspname, c.relname, c.relkind::text, i.indrelid, n.nspname = 'pg_toast',
       EXISTS (SELECT 1 FROM pg_constraint k WHERE k.conindid = c.oid AND k.contype IN ('p', 'u', 'x'))
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_index i ON i.indexrelid = c.oid
 WHERE c.oid = ANY(%s)
```

`classify` line 388

```sql
SELECT oid, nspname FROM pg_namespace WHERE oid = ANY(%s)
```

`classify` line 397

```sql
SELECT p.oid, n.nspname, p.proname, pg_get_function_identity_arguments(p.oid),
       EXISTS (SELECT 1 FROM pg_depend d WHERE d.classid = 'pg_proc'::regclass AND d.objid = p.oid
                  AND d.deptype = 'i')
  FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace WHERE p.oid = ANY(%s)
```

`classify` line 409

```sql
SELECT t.oid, n.nspname, t.typname, t.typtype::text, t.typelem, t.typrelid,
       (SELECT relkind::text FROM pg_class WHERE oid = t.typrelid)
  FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace WHERE t.oid = ANY(%s)
```

`classify` line 429

```sql
SELECT oid, conrelid, contypid, contype, conname, conislocal, coninhcount FROM pg_constraint WHERE oid = ANY(%s)
```

`classify` line 454

```sql
SELECT oid, tgrelid, tgisinternal, tgname, tgenabled::text FROM pg_trigger WHERE oid = ANY(%s)
```

`classify` line 457

```sql
SELECT oid, tgenabled::text FROM pg_trigger WHERE oid = ANY(%s)
```

`classify` line 474

```sql
SELECT oid, adrelid, adnum FROM pg_attrdef WHERE oid = ANY(%s)
```

`classify` line 480

```sql
SELECT oid, polrelid FROM pg_policy WHERE oid = ANY(%s)
```

`classify` line 486

```sql
SELECT oid, ev_class, rulename FROM pg_rewrite WHERE oid = ANY(%s)
```

`piece_defs.rel_ref` line 511

```sql
SELECT quote_ident(n.nspname) || '.' || quote_ident(c.relname) FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = %s
```

`piece_defs` line 518

```sql
SELECT attname, format_type(atttypid, atttypmod), attnotnull FROM pg_attribute WHERE attrelid = %s AND attnum = %s
```

`piece_defs` line 521

```sql
ALTER TABLE {rel_ref(relid)} ADD COLUMN {_ident(name)} {typ};
```

`piece_defs` line 527

```sql
SELECT adrelid, (SELECT attname FROM pg_attribute WHERE attrelid = adrelid AND attnum = adnum), pg_get_expr(adbin, adrelid) FROM pg_attrdef WHERE oid = %s
```

`piece_defs` line 530

```sql
ALTER TABLE {rel_ref(relid)} ALTER COLUMN {_ident(col)} SET DEFAULT {expr};
```

`piece_defs` line 535

```sql
SELECT conname, conrelid, contypid, pg_get_constraintdef(oid, true), contype FROM pg_constraint WHERE oid = %s
```

`piece_defs` line 542

```sql
SELECT array_agg(quote_ident(a.attname)) FROM pg_constraint k JOIN pg_attribute a ON a.attrelid = k.conrelid AND a.attnum = ANY(k.conkey) WHERE k.oid = %s
```

`piece_defs` line 547

```sql
SELECT EXISTS (SELECT 1 FROM {rel_ref(relid)} WHERE {cond})
```

`piece_defs` line 555

```sql
ALTER TABLE {ref} ADD CONSTRAINT {_ident(name)} {defn} NOT VALID;
```

`piece_defs` line 556

```sql
ALTER TABLE {ref} VALIDATE CONSTRAINT {_ident(name)};
```

`piece_defs` line 558

```sql
ALTER TABLE {rel_ref(relid)} ADD CONSTRAINT {_ident(name)} {defn};
```

`piece_defs` line 560

```sql
SELECT %s::regtype::text
```

`piece_defs` line 561

```sql
ALTER DOMAIN {cur.fetchone()[0]} ADD CONSTRAINT {_ident(name)} {defn};
```

`piece_defs` line 565

```sql
SELECT pg_get_indexdef(%s), (SELECT spcname FROM pg_tablespace WHERE oid = (SELECT reltablespace FROM pg_class WHERE oid = %s))
```

`piece_defs` line 579

```sql
ALTER INDEX {rel_ref(p)} ATTACH PARTITION {rel_ref(c)};
```

`piece_defs` line 584

```sql
SELECT pg_get_triggerdef(%s, true), tgrelid, tgname, tgenabled FROM pg_trigger WHERE oid = %s
```

`piece_defs` line 589

```sql
ALTER TABLE {rel_ref(relid)} {_TRIGGER_STATE.get(enabled, 'ENABLE TRIGGER')} {_ident(name)};
```

`piece_defs` line 594

```sql
ALTER TABLE {rel_ref(relid)} {_TRIGGER_STATE.get(enabled, 'ENABLE TRIGGER')} {_ident(name)};
```

`piece_defs` line 599

```sql
SELECT polname, polrelid, polpermissive, polcmd,
       ARRAY(SELECT CASE WHEN r = 0 THEN 'public' ELSE pg_get_userbyid(r) END FROM unnest(polroles) r),
       pg_get_expr(polqual, polrelid), pg_get_expr(polwithcheck, polrelid)
  FROM pg_policy WHERE oid = %s
```

**st** - `piece_defs` line 605

```sql
CREATE POLICY {_ident(pname)} ON {rel_ref(relid)}{'' if permissive else ' AS RESTRICTIVE'} FOR {cmd.get(pcmd, 'ALL')}
```

`piece_defs` line 612

```sql
WITH CHECK ({check})
```

`piece_defs` line 617

```sql
SELECT pg_get_ruledef(%s, true)
```

`piece_defs` line 623

```sql
SELECT pg_get_statisticsobjdef(s.oid), quote_ident(n.nspname) || '.' || quote_ident(s.stxname), quote_ident(pg_get_userbyid(s.stxowner)) FROM pg_statistic_ext s JOIN pg_namespace n ON n.oid = s.stxnamespace WHERE s.oid = %s
```

`piece_defs` line 629

```sql
ALTER STATISTICS {sname} OWNER TO {sowner};
```

`piece_defs` line 637

```sql
SELECT p.pubname, pr.prrelid FROM pg_publication_rel pr JOIN pg_publication p ON p.oid = pr.prpubid WHERE pr.oid = %s
```

`piece_defs` line 640

```sql
ALTER PUBLICATION {_ident(pub)} ADD TABLE {rel_ref(relid)};
```

`rows_size` line 657

```sql
SELECT pg_table_size(to_regclass(%s))
```

`_save_rows` line 697

```sql
SELECT n.nspname, c.relname, array_agg(a.attname::text ORDER BY a.attnum) FROM unnest(%s::text[], %s::text[]) AS u(s, r) JOIN pg_namespace n ON n.nspname = u.s JOIN pg_class c ON c.relnamespace = n.oid AND c.relname = u.r JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum > 0 AND NOT a.attisdropped {gen} GROUP BY n.nspname, c.relname
```

`_save_rows` line 717

```sql
COPY {ref} ({collist}) TO STDOUT
```

`_save_rows` line 722

```sql
SELECT count(*) FROM {ref}
```

`_save_rows` line 729

```sql
SELECT n.nspname, c.relname, a.attname FROM pg_attribute a JOIN pg_class c ON c.oid = a.attrelid JOIN pg_namespace n ON n.oid = c.relnamespace WHERE a.attrelid = %s AND a.attnum = %s
```

`_save_rows` line 732

```sql
SELECT a.attname FROM pg_index i JOIN pg_attribute a ON a.attrelid = i.indrelid
                        AND a.attnum = ANY(i.indkey) WHERE i.indrelid = %s AND i.indisprimary ORDER BY a.attnum
```

`_save_rows` line 746

```sql
COPY (SELECT {sel} FROM {ref}) TO STDOUT
```

`_pgver` line 760

```sql
SHOW server_version_num
```

`restore_rows` line 790

```sql
SELECT to_regclass(%s)::oid
```

`restore_rows` line 796

```sql
SELECT EXISTS (SELECT 1 FROM {ref})
```

`restore_rows` line 800

```sql
SELECT 1 FROM pg_attribute WHERE attrelid = to_regclass(%s) AND attname = %s AND NOT attisdropped
```

`restore_rows` line 810

```sql
SELECT conrelid, confrelid FROM pg_constraint WHERE contype = 'f' AND conrelid = ANY(%s) AND confrelid = ANY(%s) AND conrelid <> confrelid
```

`restore_rows` line 831

```sql
COPY {ref} ({collist}) FROM STDIN
```

`restore_rows` line 835

```sql
SELECT format_type(atttypid, atttypmod) FROM pg_attribute WHERE attrelid = to_regclass(%s) AND attname = %s
```

`restore_rows` line 839

```sql
CREATE TEMP TABLE _dbap_colvals ON COMMIT DROP AS SELECT {keys} FROM {ref} WITH NO DATA
```

`restore_rows` line 840

```sql
ALTER TABLE _dbap_colvals ADD COLUMN _v text
```

`restore_rows` line 842

```sql
COPY _dbap_colvals ({keys}, _v) FROM STDIN
```

`restore_rows` line 844

```sql
UPDATE {ref} t SET {_ident(c['column'])} = v._v::{typ} FROM _dbap_colvals v WHERE {match}
```

`restore_rows` line 846

```sql
DROP TABLE _dbap_colvals
```

## remap_restore.py

`capabilities` line 72

```sql
SELECT COALESCE(bool_or(r.rolcreatedb OR r.rolsuper), false)
  FROM pg_roles r
 WHERE pg_has_role(current_user, r.oid, 'USAGE')
```

`capabilities` line 79

```sql
SELECT COALESCE(bool_or(pg_has_role(current_user, n.nspowner, 'USAGE')), false)
  FROM pg_namespace n WHERE n.nspname = %s
```

`capabilities` line 84

```sql
SELECT has_database_privilege(current_user, current_database(), 'CREATE')
```

`capabilities` line 90

```sql
SELECT COALESCE(SUM(pg_total_relation_size(c.oid)), 0)
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind IN ('r','m','p')
```

`run.count_objects` line 243

```sql
SELECT (SELECT count(*) FROM pg_class c
          JOIN pg_namespace n ON n.oid = c.relnamespace
         WHERE n.nspname = %s)
     + (SELECT count(*) FROM pg_proc p
          JOIN pg_namespace n ON n.oid = p.pronamespace
         WHERE n.nspname = %s)
```

`run.drop_scratch` line 270

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = %s AND pid <> pg_backend_pid()
```

`run.drop_scratch` line 275

```sql
DROP DATABASE IF EXISTS %s
```

`run` line 286

```sql
CREATE DATABASE %s
```

`run` line 294

```sql
CREATE SCHEMA %s
```

`run` line 304

```sql
ALTER SCHEMA %s RENAME TO %s
```

`run` line 335

```sql
ALTER SCHEMA %s RENAME TO %s
```

`run` line 338

```sql
CREATE SCHEMA %s
```

`run` line 348

```sql
ALTER SCHEMA %s RENAME TO %s
```

`run` line 350

```sql
ALTER SCHEMA %s RENAME TO %s
```

`run` line 361

```sql
DROP SCHEMA IF EXISTS %s CASCADE
```

`run` line 362

```sql
ALTER SCHEMA %s RENAME TO %s
```

## restore_checks.py

`target_facts` line 74

```sql
SELECT rolname FROM pg_roles
```

`target_facts` line 76

```sql
SELECT name FROM pg_available_extensions
```

`target_facts` line 78

```sql
SELECT extname, extversion FROM pg_extension
```

`target_facts` line 82

```sql
SELECT default_version FROM pg_available_extensions WHERE name = 'timescaledb'
```

`target_facts` line 85

```sql
SELECT nspname FROM pg_namespace
```

`target_facts` line 87

```sql
SELECT pg_encoding_to_char(encoding), datcollate, datctype FROM pg_database WHERE datname = current_database()
```

`target_facts` line 97

```sql
SELECT datlocprovider, %s FROM pg_database WHERE datname = current_database()
```

`_rel_of` line 181

```sql
SELECT format('%%I.%%I', n.nspname, c.relname), c.relkind FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.relname = %s AND pg_table_is_visible(c.oid)
```

`compare_rows` line 208

```sql
SELECT count(*) FROM %s
```

`compare_rows` line 209

```sql
SELECT count(*) FROM ONLY %s
```

`compare_sequences` line 229

```sql
SELECT last_value, is_called FROM %s
```

`_catalog` line 295

```sql
SELECT n.nspname, pg_get_userbyid(n.nspowner) FROM pg_namespace n WHERE {sysw}
```

`_catalog` line 299

```sql
SELECT '{name}'::text, o.nsp, o.name, o.args, o.col, COALESCE(r.rolname, 'PUBLIC'), x.privilege_type FROM ({sources[name]}) o({osc.ACL_COLS}), aclexplode(o.acl) x LEFT JOIN pg_catalog.pg_roles r ON r.oid = x.grantee
```

`_catalog` line 319

```sql
SELECT l, array_agg(x.privilege_type) FROM unnest(%s::text[]) l, aclexplode(acldefault(l::"char", 10::oid)) x GROUP BY l
```

## role_privs_export.py

`_server_version` line 98

```sql
SHOW server_version_num
```

**line** - `_grant_lines` line 139

```sql
GRANT {', '.join((f'{p} ({col})' for p in sorted(set(privs))))} ON TABLE {rel} TO {to}{tail};
```

**line** - `_grant_lines` line 141

```sql
GRANT {', '.join(sorted(set(privs)))} ON {kind} {obj} TO {to}{tail};
```

`_role_oids` line 149

```sql
SELECT oid, rolname FROM pg_roles WHERE rolname = ANY(%s)
```

`_role_oids` line 151

```sql
SELECT oid, rolname FROM pg_roles WHERE rolname NOT LIKE 'pg\_%%'
```

`_owner_diff_lines` line 176

```sql
GRANT {', '.join((f'{p} ({col})' for p in sorted(privs)))} ON TABLE {rel} TO {_who(grantee, names)}{' WITH GRANT OPTION' if grantable else ''};
```

`_owner_diff_lines` line 180

```sql
REVOKE {', '.join(sorted(privs))} ON {kw} {obj} FROM {_who(grantee, names)};
```

`_owner_diff_lines` line 182

```sql
GRANT {', '.join(sorted(privs))} ON {kw} {obj} TO {_who(grantee, names)}{' WITH GRANT OPTION' if grantable else ''};
```

**stmt** - `_ownership` line 350

```sql
ALTER TYPE {ident} OWNER TO {who};
```

`_ownership` line 354

```sql
DO {tag} BEGIN IF current_setting('server_version_num')::int < 170000 THEN {stmt} END IF; END {tag}; -- {_one_line(f'multirange of {o.get('range') or '?'}: PostgreSQL 14-16 keep its owner apart, 17+ moves it with the range type')}
```

`_default_privilege_lines` line 416

```sql
SELECT oid, rolname FROM pg_roles
```

**head** - `_default_privilege_lines` line 435

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE {who(owner)}{f' IN SCHEMA {_ident_str(schema)}' if schema else ''}
```

`export` line 514

```sql
SELECT datname FROM pg_database
                        WHERE datallowconn AND NOT datistemplate
                          AND has_database_privilege(datname, 'CONNECT') ORDER BY datname
```

`export` line 518

```sql
SELECT datname FROM pg_database
                        WHERE datallowconn AND NOT datistemplate
                          AND NOT has_database_privilege(datname, 'CONNECT') ORDER BY datname
```

## schema_engine.py

`_schema_other_objects._rows` line 200

```sql
ROLLBACK TO SAVEPOINT _dbap_other_objs
```

`_schema_other_objects` line 225

```sql
SELECT p.proname::text, (p.proname || '(' ||
       pg_get_function_identity_arguments(p.oid) || ')')::text,
       p.oid::regprocedure::text, ''::text, {kw},
       true, ''::text,
       EXISTS (SELECT 1 FROM pg_proc p2
                 JOIN pg_namespace n2 ON n2.oid = p2.pronamespace
                WHERE n2.nspname = %(t)s AND p2.proname = p.proname
                  AND p2.proargtypes = p.proargtypes),
       ARRAY[]::text[]
  FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname = %(s)s
   AND NOT EXISTS (SELECT 1 FROM pg_depend d WHERE d.objid = p.oid
                    AND d.classid = 'pg_proc'::regclass
                    AND d.deptype IN ('i','e','a'))
 ORDER BY 1
```

`_schema_other_objects` line 278

```sql
SELECT t.typname::text, t.typname::text, t.oid::regtype::text, ''::text,
       CASE WHEN t.typtype = 'd' THEN 'DOMAIN' ELSE 'TYPE' END,
       true, ''::text,
       EXISTS (SELECT 1 FROM pg_type t2
                 JOIN pg_namespace n2 ON n2.oid = t2.typnamespace
                WHERE n2.nspname = %(t)s AND t2.typname = t.typname),
       {multi}
  FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace
 WHERE n.nspname = %(s)s
   AND t.typtype IN ('e','d','c','r')
   AND (t.typrelid = 0 OR EXISTS (SELECT 1 FROM pg_class rc
                                   WHERE rc.oid = t.typrelid
                                     AND rc.relkind = 'c'))
   AND NOT EXISTS (SELECT 1 FROM pg_type el WHERE el.typarray = t.oid)
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.objid = t.oid
                    AND dx.classid = 'pg_type'::regclass
                    AND dx.deptype IN ('i','e'))
 ORDER BY 1
```

`_schema_other_objects` line 298

```sql
SELECT c.collname::text, c.collname::text,
       (quote_ident(n.nspname) || '.' || quote_ident(c.collname))::text,
       ''::text, 'COLLATION', true, ''::text,
       EXISTS (SELECT 1 FROM pg_collation c2
                 JOIN pg_namespace n2 ON n2.oid = c2.collnamespace
                WHERE n2.nspname = %(t)s AND c2.collname = c.collname
                  AND c2.collencoding = c.collencoding),
       ARRAY[]::text[]
  FROM pg_collation c JOIN pg_namespace n ON n.oid = c.collnamespace
 WHERE n.nspname = %(s)s
   AND NOT EXISTS (SELECT 1 FROM pg_depend d WHERE d.objid = c.oid
                    AND d.classid = 'pg_collation'::regclass
                    AND d.deptype IN ('i','e'))
 ORDER BY 1
```

`_schema_other_objects` line 314

```sql
SELECT c.conname::text, c.conname::text,
       (quote_ident(n.nspname) || '.' || quote_ident(c.conname))::text,
       ''::text, 'CONVERSION', true, ''::text,
       EXISTS (SELECT 1 FROM pg_conversion c2
                 JOIN pg_namespace n2 ON n2.oid = c2.connamespace
                WHERE n2.nspname = %(t)s AND c2.conname = c.conname),
       ARRAY[]::text[]
  FROM pg_conversion c JOIN pg_namespace n ON n.oid = c.connamespace
 WHERE n.nspname = %(s)s
   AND NOT EXISTS (SELECT 1 FROM pg_depend d WHERE d.objid = c.oid
                    AND d.classid = 'pg_conversion'::regclass
                    AND d.deptype IN ('i','e'))
 ORDER BY 1
```

`_schema_other_objects` line 329

```sql
SELECT o.oprname::text,
       (o.oprname || '(' || COALESCE(format_type(o.oprleft, NULL), 'NONE')
                  || ', ' || COALESCE(format_type(o.oprright, NULL), 'NONE')
                  || ')')::text,
       o.oid::regoperator::text, ''::text, 'OPERATOR', true, ''::text,
       EXISTS (SELECT 1 FROM pg_operator o2
                 JOIN pg_namespace n2 ON n2.oid = o2.oprnamespace
                WHERE n2.nspname = %(t)s AND o2.oprname = o.oprname
                  AND o2.oprleft = o.oprleft
                  AND o2.oprright = o.oprright),
       ARRAY[]::text[]
  FROM pg_operator o JOIN pg_namespace n ON n.oid = o.oprnamespace
 WHERE n.nspname = %(s)s
   AND NOT EXISTS (SELECT 1 FROM pg_depend d WHERE d.objid = o.oid
                    AND d.classid = 'pg_operator'::regclass
                    AND d.deptype IN ('i','e'))
 ORDER BY 1
```

`_schema_other_objects` line 348

```sql
SELECT oc.opcname::text, (oc.opcname || ' USING ' || am.amname)::text,
       (quote_ident(n.nspname) || '.' || quote_ident(oc.opcname))::text,
       ('USING ' || quote_ident(am.amname))::text,
       'OPERATOR CLASS', true, ''::text,
       EXISTS (SELECT 1 FROM pg_opclass o2
                 JOIN pg_namespace n2 ON n2.oid = o2.opcnamespace
                WHERE n2.nspname = %(t)s AND o2.opcname = oc.opcname
                  AND o2.opcmethod = oc.opcmethod),
       ARRAY[]::text[]
  FROM pg_opclass oc JOIN pg_namespace n ON n.oid = oc.opcnamespace
  JOIN pg_am am ON am.oid = oc.opcmethod
 WHERE n.nspname = %(s)s
   AND NOT EXISTS (SELECT 1 FROM pg_depend d WHERE d.objid = oc.oid
                    AND d.classid = 'pg_opclass'::regclass
                    AND d.deptype IN ('i','e'))
 ORDER BY 1
```

`_schema_other_objects` line 366

```sql
SELECT opf.opfname::text, (opf.opfname || ' USING ' || am.amname)::text,
       (quote_ident(n.nspname) || '.' || quote_ident(opf.opfname))::text,
       ('USING ' || quote_ident(am.amname))::text,
       'OPERATOR FAMILY', true, ''::text,
       EXISTS (SELECT 1 FROM pg_opfamily o2
                 JOIN pg_namespace n2 ON n2.oid = o2.opfnamespace
                WHERE n2.nspname = %(t)s AND o2.opfname = opf.opfname
                  AND o2.opfmethod = opf.opfmethod),
       ARRAY[]::text[]
  FROM pg_opfamily opf JOIN pg_namespace n ON n.oid = opf.opfnamespace
  JOIN pg_am am ON am.oid = opf.opfmethod
 WHERE n.nspname = %(s)s
   AND NOT EXISTS (SELECT 1 FROM pg_depend d WHERE d.objid = opf.oid
                    AND d.classid = 'pg_opfamily'::regclass
                    AND d.deptype IN ('i','e'))
 ORDER BY 1
```

`_schema_other_objects` line 389

```sql
SELECT x.{c}name::text, x.{c}name::text,
       (quote_ident(n.nspname) || '.' || quote_ident(x.{c}name))::text,
       ''::text, '{k}', true, ''::text,
       EXISTS (SELECT 1 FROM {t} x2
                 JOIN pg_namespace n2 ON n2.oid = x2.{c}namespace
                WHERE n2.nspname = %(t)s AND x2.{c}name = x.{c}name),
       ARRAY[]::text[]
  FROM {t} x JOIN pg_namespace n ON n.oid = x.{c}namespace
 WHERE n.nspname = %(s)s
   AND NOT EXISTS (SELECT 1 FROM pg_depend d WHERE d.objid = x.oid
                    AND d.classid = '{t}'::regclass
                    AND d.deptype IN ('i','e'))
 ORDER BY 1
```

`_schema_other_objects` line 406

```sql
SELECT s.stxname::text, s.stxname::text,
       (quote_ident(n.nspname) || '.' || quote_ident(s.stxname))::text,
       ''::text, 'STATISTICS', true, ''::text,
       EXISTS (SELECT 1 FROM pg_statistic_ext s2
                 JOIN pg_namespace n2 ON n2.oid = s2.stxnamespace
                WHERE n2.nspname = %(t)s AND s2.stxname = s.stxname),
       ARRAY[]::text[]
  FROM pg_statistic_ext s JOIN pg_namespace n ON n.oid = s.stxnamespace
 WHERE n.nspname = %(s)s ORDER BY 1
```

`_schema_other_objects` line 420

```sql
SELECT c.relname::text, c.relname::text,
       (quote_ident(n.nspname) || '.' || quote_ident(c.relname))::text,
       ''::text, 'FOREIGN TABLE', true, ''::text,
       EXISTS (SELECT 1 FROM pg_class c2
                 JOIN pg_namespace n2 ON n2.oid = c2.relnamespace
                WHERE n2.nspname = %(t)s AND c2.relname = c.relname),
       ARRAY[]::text[]
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %(s)s AND c.relkind = 'f' ORDER BY 1
```

`_schema_other_objects` line 435

```sql
SELECT e.extname::text, e.extname::text, quote_ident(e.extname)::text,
       ''::text, 'EXTENSION', e.extrelocatable,
       CASE WHEN e.extrelocatable THEN ''
            ELSE 'the extension declares itself non-relocatable, so '
                 'PostgreSQL refuses to move it' END::text,
       false, ARRAY[]::text[]
  FROM pg_extension e JOIN pg_namespace n ON n.oid = e.extnamespace
 WHERE n.nspname = %(s)s ORDER BY 1
```

`_move_other_object_sql` line 456

```sql
SET SCHEMA %s
```

`_move_other_object_sql` line 464

```sql
ALTER %s SET SCHEMA %s
```

**sql** - `_retained_schema_deps` line 498

```sql
WITH moving AS (
  SELECT c.oid, c.relname FROM pg_class c
    JOIN pg_namespace n ON n.oid = c.relnamespace
   WHERE n.nspname = %(s)s AND c.relkind = ANY(%(k)s)
), dep AS (
  SELECT m.relname AS rel, 'trigger ' || t.tgname AS what,
         'function' AS dep_kind, p.proname || '()' AS dep_name,
         p.proname::text AS dep_base
    FROM moving m
    JOIN pg_trigger t ON t.tgrelid = m.oid AND NOT t.tgisinternal
    JOIN pg_proc p ON p.oid = t.tgfoid
    JOIN pg_namespace pn ON pn.oid = p.pronamespace
   WHERE pn.nspname = %(s)s
  UNION ALL
  SELECT m.relname, 'column ' || a.attname, 'type', ty.typname::text,
         ty.typname::text
    FROM moving m
    JOIN pg_attribute a ON a.attrelid = m.oid AND a.attnum > 0 AND NOT a.attisdropped
    JOIN pg_type ty ON ty.oid = a.atttypid
    JOIN pg_namespace tn ON tn.oid = ty.typnamespace
   WHERE tn.nspname = %(s)s
  UNION ALL
  SELECT m.relname, 'column ' || a.attname, 'collation', co.collname::text,
         co.collname::text
    FROM moving m
    JOIN pg_attribute a ON a.attrelid = m.oid AND a.attnum > 0 AND NOT a.attisdropped
    JOIN pg_collation co ON co.oid = a.attcollation
    JOIN pg_namespace cn ON cn.oid = co.collnamespace
   WHERE cn.nspname = %(s)s
)
SELECT rel, what, dep_kind, dep_name, dep_base FROM dep ORDER BY 1, 2
```

`_retained_schema_deps` line 546

```sql
ROLLBACK TO SAVEPOINT _dbap_retained
```

`_cross_schema_text_risks` line 608

```sql
SELECT n.nspname, p.proname, p.oid,
       pg_get_function_identity_arguments(p.oid) AS args,
       p.prosrc AS body
  FROM pg_proc p
  JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND p.prosrc ILIKE %s
 ORDER BY n.nspname, p.proname
```

`_cross_schema_text_risks` line 631

```sql
SELECT t.tgfoid, tn.nspname, tc.relname, t.tgname
  FROM pg_trigger t
  JOIN pg_class tc ON tc.oid = t.tgrelid
  JOIN pg_namespace tn ON tn.oid = tc.relnamespace
 WHERE t.tgfoid = ANY(%s) AND NOT t.tgisinternal
```

`_cross_schema_text_risks` line 679

```sql
SELECT schemaname, tablename, policyname, qual, with_check FROM pg_policies
```

`_firing_sql` line 708

```sql
ALTER TABLE {table_ref} {word} {kind} {_qi(name)};
```

`_clone_plan` line 749

```sql
SELECT c.oid, c.relkind FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relname = %s
```

`_clone_plan` line 762

```sql
SELECT 1 FROM pg_namespace WHERE nspname = %s
```

`_clone_plan` line 767

```sql
SELECT 1 FROM pg_roles WHERE rolname = %s
```

`_clone_plan` line 772

```sql
SELECT 1 FROM pg_tablespace WHERE spcname = %s
```

`_clone_plan` line 782

```sql
SELECT DISTINCT k.confrelid
FROM pg_constraint k JOIN pg_class rc ON rc.oid = k.confrelid
WHERE k.conrelid = %s AND k.contype = 'f'
  AND k.confrelid <> k.conrelid AND rc.relkind IN ('r','p')
```

`_clone_plan` line 805

```sql
SELECT c.relkind FROM pg_class c WHERE c.oid = %s
```

`_clone_plan` line 810

```sql
SELECT i.inhrelid, pg_get_expr(c.relpartbound, c.oid)
  FROM pg_inherits i
  JOIN pg_class c ON c.oid = i.inhrelid
 WHERE i.inhparent = %s
 ORDER BY c.relname
```

`_clone_plan` line 833

```sql
SELECT n.nspname, c.relname, {_rls_col}, c.relacl, c.relkind
FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.oid = %s
```

`_clone_plan` line 858

```sql
SELECT 1 FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relname = %s
```

`_clone_plan` line 891

```sql
CREATE TABLE {_qq(ts, tn)} PARTITION OF {_qq(parent_ts, parent_tn)} {bound or 'DEFAULT'}{ts_clause};
```

`_clone_plan` line 901

```sql
SELECT pg_catalog.pg_get_partkeydef(%s)
```

`_clone_plan` line 904

```sql
CREATE TABLE {_qq(ts, tn)} (LIKE {_qq(si['schema'], si['name'])} {incl}){partby}{ts_clause};
```

`_clone_plan` line 946

```sql
SELECT attname, {_ident} FROM pg_attribute WHERE attrelid = %s AND attnum > 0 AND NOT attisdropped {_gen} ORDER BY attnum
```

**st** - `_clone_plan` line 955

```sql
INSERT INTO {_qq(ts, tn)} ({collist}){ovr} SELECT {collist} FROM {_qq(si['schema'], si['name'])};
```

`_clone_plan` line 966

```sql
SET LOCAL row_security = off;
```

`_clone_plan` line 969

```sql
RESET row_security;
```

`_clone_plan` line 981

```sql
SELECT k.conname, k.confrelid, k.confupdtype, k.confdeltype, k.confmatchtype,
       k.condeferrable, k.condeferred, k.convalidated, {_enf} AS enforced,
       (SELECT array_agg(a.attname ORDER BY u.ord)
          FROM unnest(k.conkey) WITH ORDINALITY u(attnum, ord)
          JOIN pg_attribute a ON a.attrelid = k.conrelid AND a.attnum = u.attnum) AS cols,
       (SELECT array_agg(a.attname ORDER BY u.ord)
          FROM unnest(k.confkey) WITH ORDINALITY u(attnum, ord)
          JOIN pg_attribute a ON a.attrelid = k.confrelid AND a.attnum = u.attnum) AS refcols
FROM pg_constraint k
WHERE k.conrelid = %s AND k.contype = 'f'
```

`_clone_plan` line 998

```sql
SELECT n.nspname, c.relname FROM pg_class c
                               JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = %s
```

`_clone_plan` line 1019

```sql
ALTER TABLE {_qq(ts, tn)} ADD CONSTRAINT {_qi(cn)} {' '.join(parts)};
```

`_clone_plan` line 1027

```sql
SELECT pg_get_triggerdef(t.oid), %s::regclass::text, t.tgname, t.tgenabled::text
FROM pg_trigger t WHERE t.tgrelid = %s AND NOT t.tgisinternal
```

`_clone_plan` line 1045

```sql
ALTER TABLE {_qq(ts, tn)} ENABLE ROW LEVEL SECURITY;
```

`_clone_plan` line 1046

```sql
SELECT pol.polname, pol.polcmd, pol.polpermissive,
       pg_get_expr(pol.polqual, pol.polrelid),
       pg_get_expr(pol.polwithcheck, pol.polrelid),
       (SELECT string_agg(quote_ident(rolname), ', ')
          FROM pg_roles WHERE oid = ANY(pol.polroles)) AS roles,
       (0 = ANY(pol.polroles)) AS to_public
FROM pg_policy pol WHERE pol.polrelid = %s
```

`_clone_plan` line 1057

```sql
CREATE POLICY {_qi(pn)} ON {_qq(ts, tn)}
```

`_clone_plan` line 1062

```sql
WITH CHECK ({wcheck})
```

`_clone_plan` line 1072

```sql
SELECT COALESCE(grantee.rolname, 'PUBLIC') AS grantee,
       acl.privilege_type, acl.is_grantable
FROM aclexplode(%s::aclitem[]) acl
LEFT JOIN pg_roles grantee ON grantee.oid = acl.grantee
```

`_clone_plan` line 1080

```sql
GRANT {priv} ON {_qq(ts, tn)} TO {who}{' WITH GRANT OPTION' if grantable else ''};
```

`_clone_plan` line 1092

```sql
ALTER TABLE {_qq(ts, tn)} OWNER TO {_qi(new_owner)};
```

`_clone_plan` line 1102

```sql
SELECT a.attname, pg_get_serial_sequence(%s, a.attname) AS seq
  FROM pg_attribute a
 WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped
```

`_clone_plan` line 1127

```sql
SELECT setval(pg_get_serial_sequence({_fqn_lit}, {_col_lit}), (SELECT COALESCE(max({_qi(col)}), 0) + 1 FROM {_qq(ts, tn)}), false);
```

`_clone_plan` line 1138

```sql
ANALYZE {_qq(ts, tn)};
```

`_clone_plan` line 1155

```sql
ANALYZE statement(s) appended
```

`_clone_rows_blocked` line 1189

```sql
SELECT c.relname AS name, %s AS rls, %s AS forced,
       pg_catalog.pg_has_role(current_user, c.relowner, 'USAGE') AS owner,
       (SELECT %s FROM pg_catalog.pg_roles r WHERE r.rolname = current_user) AS bypass,
       current_user AS me,
       ARRAY(SELECT a.attname::text FROM pg_catalog.pg_attribute a
              WHERE a.attrelid = c.oid AND a.attnum > 0 AND NOT a.attisdropped %s
                AND NOT pg_catalog.has_column_privilege(c.oid, a.attnum, 'SELECT')
              ORDER BY a.attnum) AS denied
  FROM pg_catalog.pg_class c
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %%s AND c.relname = ANY(%%s)
 ORDER BY c.relname
```

`_failing_source_rows` line 1307

```sql
SET LOCAL statement_timeout = 15000
```

`_failing_source_rows` line 1317

```sql
SELECT pg_catalog.pg_get_constraintdef(oid) FROM pg_catalog.pg_constraint WHERE conrelid = %s::regclass AND conname = %s AND contype = 'c'
```

`_failing_source_rows` line 1330

```sql
SELECT c.relkind::text FROM pg_catalog.pg_class c WHERE c.oid = %s::regclass
```

`_failing_source_rows` line 1334

```sql
SELECT a.attnum, a.attname, i.indkey::int2[] FROM pg_catalog.pg_index i
  JOIN pg_catalog.pg_attribute a ON a.attrelid = i.indrelid
                                AND a.attnum = ANY(i.indkey)
 WHERE i.indrelid = %s::regclass AND i.indisprimary
```

`_failing_source_rows` line 1343

```sql
SELECT %s FROM %s%s WHERE %s LIMIT %d
```

`_like_fixups` line 1388

```sql
SELECT conname, pg_catalog.pg_get_constraintdef(oid) FROM pg_catalog.pg_constraint WHERE conrelid = %s AND contype = 'c' AND NOT convalidated {enforced} ORDER BY conname
```

`_like_fixups` line 1392

```sql
ALTER TABLE %s DROP CONSTRAINT %s;
```

`_like_fixups` line 1393

```sql
ALTER TABLE %s ADD CONSTRAINT %s %s;
```

`_like_fixups` line 1396

```sql
SELECT co.conname, a.attname FROM pg_catalog.pg_constraint co
  JOIN pg_catalog.pg_attribute a ON a.attrelid = co.conrelid AND a.attnum = co.conkey[1]
 WHERE co.conrelid = %s AND co.contype = 'n' AND NOT co.convalidated
 ORDER BY co.conname
```

`_like_fixups` line 1402

```sql
ALTER TABLE %s ALTER COLUMN %s DROP NOT NULL;
```

`_like_fixups` line 1404

```sql
ALTER TABLE %s ADD CONSTRAINT %s NOT NULL %s NOT VALID;
```

`_like_fixups` line 1410

```sql
SELECT i.relname, x.indisunique,
       regexp_replace(pg_catalog.pg_get_indexdef(x.indexrelid), '^.*? USING ', '')
  FROM pg_catalog.pg_index x JOIN pg_catalog.pg_class i ON i.oid = x.indexrelid
 WHERE x.indrelid = %s AND NOT x.indisvalid
 ORDER BY i.relname
```

`_like_fixups` line 1418

```sql
DO $$DECLARE ix regclass; BEGIN
  SELECT x.indexrelid::regclass INTO ix FROM pg_catalog.pg_index x
   WHERE x.indrelid = {_literal_str(new_q)}::regclass AND x.indisunique = {'true' if uniq else 'false'}
     AND regexp_replace(pg_catalog.pg_get_indexdef(x.indexrelid), '^.*? USING ', '') = {_literal_str(tail)}
   LIMIT 1;
  IF ix IS NOT NULL THEN EXECUTE 'DROP INDEX ' || ix; END IF;
END $$;
```

`_forced_qualification` line 1630

```sql
SELECT pg_catalog.current_setting('search_path') AS sp
```

`_forced_qualification` line 1639

```sql
SELECT pg_catalog.set_config('search_path', '', false)
```

`_restore_qualification` line 1650

```sql
SELECT pg_catalog.set_config('search_path', %s, false)
```

`_aggregate_def` line 1669

```sql
SELECT a.aggtransfn::regprocedure::text   AS sfunc,
       pg_catalog.format_type(a.aggtranstype, NULL) AS stype,
       NULLIF(a.aggtransspace, 0)          AS sspace,
       NULLIF(a.aggfinalfn, 0)::regprocedure::text  AS finalfunc,
       a.aggfinalextra                     AS finalfunc_extra,
       NULLIF(a.aggcombinefn, 0)::regprocedure::text AS combinefunc,
       NULLIF(a.aggserialfn, 0)::regprocedure::text  AS serialfunc,
       NULLIF(a.aggdeserialfn, 0)::regprocedure::text AS deserialfunc,
       a.agginitval                        AS initcond,
       NULLIF(a.aggmtransfn, 0)::regprocedure::text  AS msfunc,
       NULLIF(a.aggminvtransfn, 0)::regprocedure::text AS minvfunc,
       CASE WHEN a.aggmtranstype = 0 THEN NULL
            ELSE pg_catalog.format_type(a.aggmtranstype, NULL) END AS mstype,
       a.aggmtransspace                    AS msspace,
       NULLIF(a.aggmfinalfn, 0)::regprocedure::text  AS mfinalfunc,
       a.aggminitval                       AS minitcond,
       NULLIF(a.aggsortop, 0)::regoperator::text     AS sortop,
       a.aggkind::text                     AS aggkind,
       p.proparallel::text                 AS parallel
  FROM pg_aggregate a
  JOIN pg_proc p ON p.oid = a.aggfnoid
 WHERE a.aggfnoid = %s
```

`_extension_owned_names` line 1745

```sql
SELECT lower(p.proname) AS n FROM pg_proc p
  JOIN pg_namespace ns ON ns.oid = p.pronamespace
  JOIN pg_depend d ON d.classid='pg_proc'::regclass AND d.objid=p.oid AND d.deptype='e'
 WHERE ns.nspname = %(s)s
UNION
SELECT lower(t.typname) FROM pg_type t
  JOIN pg_namespace ns ON ns.oid = t.typnamespace
  JOIN pg_depend d ON d.classid='pg_type'::regclass AND d.objid=t.oid AND d.deptype='e'
 WHERE ns.nspname = %(s)s
UNION
SELECT lower(c.relname) FROM pg_class c
  JOIN pg_namespace ns ON ns.oid = c.relnamespace
  JOIN pg_depend d ON d.classid='pg_class'::regclass AND d.objid=c.oid AND d.deptype='e'
 WHERE ns.nspname = %(s)s
UNION
SELECT lower(o.opcname) FROM pg_opclass o
  JOIN pg_namespace ns ON ns.oid = o.opcnamespace
  JOIN pg_depend d ON d.classid='pg_opclass'::regclass AND d.objid=o.oid AND d.deptype='e'
 WHERE ns.nspname = %(s)s
UNION
SELECT lower(f.opfname) FROM pg_opfamily f
  JOIN pg_namespace ns ON ns.oid = f.opfnamespace
  JOIN pg_depend d ON d.classid='pg_opfamily'::regclass AND d.objid=f.oid AND d.deptype='e'
 WHERE ns.nspname = %(s)s
UNION
SELECT lower(op.oprname) FROM pg_operator op
  JOIN pg_namespace ns ON ns.oid = op.oprnamespace
  JOIN pg_depend d ON d.classid='pg_operator'::regclass AND d.objid=op.oid AND d.deptype='e'
 WHERE ns.nspname = %(s)s
```

`_schema_object_defs_collect` line 1885

```sql
SELECT t.oid, t.typname AS name, t.typtype::text AS typtype,
       pg_catalog.format_type(t.typbasetype, t.typtypmod) AS base,
       t.typnotnull, t.typdefault, t.typrelid
  FROM pg_type t
  JOIN pg_namespace n ON n.oid = t.typnamespace
 WHERE n.nspname = %s
   AND t.typtype IN ('e','d','c','r')
   -- The row type every table and view carries is not a type
   -- somebody declared; only a standalone composite is.
   AND (t.typrelid = 0 OR EXISTS (SELECT 1 FROM pg_class rc
                                   WHERE rc.oid = t.typrelid
                                     AND rc.relkind = 'c'))
   -- Nor is the array type PostgreSQL makes for each of these.
   AND NOT EXISTS (SELECT 1 FROM pg_type el WHERE el.typarray = t.oid)
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx
                    WHERE dx.classid = 'pg_type'::regclass
                      AND dx.objid = t.oid AND dx.deptype = 'e')
 ORDER BY t.typname
```

`_schema_object_defs_collect` line 1909

```sql
SELECT enumlabel FROM pg_enum WHERE enumtypid = %s ORDER BY enumsortorder
```

`_schema_object_defs_collect` line 1921

```sql
SELECT conname, pg_get_constraintdef(oid) AS d, convalidated FROM pg_constraint WHERE contypid = %s AND contype <> 'n' ORDER BY conname
```

`_schema_object_defs_collect` line 1946

```sql
SELECT pg_catalog.format_type(rng.rngsubtype, NULL) AS subtype,
       (SELECT opc.opcname FROM pg_opclass opc
         WHERE opc.oid = rng.rngsubopc)             AS subtype_opclass,
       NULLIF(rng.rngcollation, 0)::regcollation::text AS collation,
       NULLIF(rng.rngcanonical, 0)::regprocedure::text AS canonical,
       NULLIF(rng.rngsubdiff, 0)::regprocedure::text  AS subtype_diff
  FROM pg_range rng WHERE rng.rngtypid = %s
```

`_schema_object_defs_collect` line 1971

```sql
SELECT a.attname,
       pg_catalog.format_type(a.atttypid, a.atttypmod) AS typ
  FROM pg_attribute a
 WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

`_schema_object_defs_collect` line 2013

```sql
SELECT seqrelid AS oid, seqstart AS start, seqincrement AS inc,
                                      seqmin AS min, seqmax AS max, seqcache AS cache, seqcycle AS cycle
                                 FROM pg_catalog.pg_sequence WHERE seqrelid = ANY(%s)
```

`_schema_object_defs_collect` line 2028

```sql
SELECT c.relname AS name, c.oid AS oid, c.relkind::text AS relkind,
       c.relpersistence::text AS persistence,
       -- Where the table lives and how it is stored. 0 is the
       -- database's default tablespace, which is not the
       -- table's own setting and is compared with the database.
       (SELECT ts.spcname FROM pg_catalog.pg_tablespace ts
         WHERE ts.oid = c.reltablespace AND c.reltablespace <> 0) AS tablespace,
       {_am_expr} AS amname,
       -- The TOAST table's own storage parameters, as the
       -- toast.* names ALTER TABLE ... SET takes: a table whose
       -- toast.autovacuum_enabled differed compared as the same
       -- (only the deep comparison looked there).
       COALESCE(c.reloptions, '{}'::text[])
       || COALESCE((SELECT array_agg('toast.' || o ORDER BY o)
                      FROM pg_class tc, unnest(tc.reloptions) o
                     WHERE tc.oid = c.reltoastrelid), '{}'::text[]) AS reloptions,
       {_rls_col} AS rls,
       {_rls_forced_col} AS rls_forced,
       c.relreplident::text AS replident,
       (SELECT i.relname FROM pg_index x JOIN pg_class i ON i.oid = x.indexrelid
         WHERE x.indrelid = c.oid AND x.indisreplident) AS replident_index
FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relkind IN ('r','p')
  AND NOT EXISTS (SELECT 1 FROM pg_depend dx
                   WHERE dx.classid = 'pg_class'::regclass
                     AND dx.objid = c.oid AND dx.deptype = 'e')
ORDER BY c.relname
```

`_schema_object_defs_collect` line 2084

```sql
SELECT a.attname, pg_catalog.format_type(a.atttypid, a.atttypmod) AS typ,
       a.attnotnull, pg_get_expr(d.adbin, d.adrelid) AS dflt,
       %s AS identity, %s AS generated,
       CASE WHEN a.attcollation <> 0
             AND a.attcollation <> t.typcollation
            THEN (SELECT co.collname FROM pg_collation co
                   WHERE co.oid = a.attcollation) END AS collname,
       CASE WHEN a.attcollation <> 0
             AND a.attcollation <> t.typcollation
            THEN (SELECT cn.nspname FROM pg_collation co
                    JOIN pg_namespace cn ON cn.oid = co.collnamespace
                   WHERE co.oid = a.attcollation) END AS collschema,
       a.attstorage::text AS storage, t.typstorage::text AS typstorage,
       %s AS compression, %s AS stattarget,
       a.attislocal AS islocal
FROM pg_attribute a
JOIN pg_type t ON t.oid = a.atttypid
LEFT JOIN pg_attrdef d ON d.adrelid = a.attrelid AND d.adnum = a.attnum
WHERE a.attrelid = %%s AND a.attnum > 0 AND NOT a.attisdropped
ORDER BY a.attnum
```

`_schema_object_defs_collect` line 2161

```sql
SELECT conname, pg_get_constraintdef(oid) AS d, convalidated FROM pg_constraint WHERE conrelid = %s {_parent_filter} AND contype <> 'n' AND (contype <> 'c' OR conislocal) ORDER BY conname
```

`_schema_object_defs_collect` line 2183

```sql
SELECT co.conname, a.attname
  FROM pg_constraint co
  JOIN pg_attribute a ON a.attrelid = co.conrelid
                     AND a.attnum = co.conkey[1]
 WHERE co.conrelid = %s AND co.contype = 'n'
   AND NOT co.convalidated AND co.conislocal
```

`_schema_object_defs_collect` line 2204

```sql
SELECT a.attname, %s AS conname
  FROM pg_attribute a
 WHERE a.attrelid = %%s AND a.attnum > 0 AND NOT a.attisdropped
   AND a.attnotnull
   AND NOT EXISTS (SELECT 1 FROM pg_inherits i
                     JOIN pg_attribute pa ON pa.attrelid = i.inhparent
                                         AND pa.attname = a.attname
                    WHERE i.inhrelid = a.attrelid AND pa.attnotnull)
```

`_schema_object_defs_collect` line 2234

```sql
SELECT i.relname AS indexname,
       pg_catalog.pg_get_indexdef(i.oid) AS indexdef,
       x.indisvalid AS valid
  FROM pg_index x
  JOIN pg_class i ON i.oid = x.indexrelid
 WHERE x.indrelid = %s
   AND NOT EXISTS (SELECT 1 FROM pg_inherits ih
                    WHERE ih.inhrelid = i.oid)
 ORDER BY i.relname
```

`_schema_object_defs_collect` line 2277

```sql
SELECT pol.polname AS name, pol.polcmd::text AS cmd,
       pol.polpermissive AS permissive,
       pg_catalog.pg_get_expr(pol.polqual, pol.polrelid, true) AS using_expr,
       pg_catalog.pg_get_expr(pol.polwithcheck, pol.polrelid, true) AS check_expr,
       ARRAY(SELECT rolname FROM pg_catalog.pg_roles
              WHERE oid = ANY(pol.polroles) ORDER BY rolname) AS roles
  FROM pg_catalog.pg_policy pol
 WHERE pol.polrelid = %s
 ORDER BY pol.polname
```

`_schema_object_defs_collect._view_colnames` line 2465

```sql
SELECT attname FROM pg_catalog.pg_attribute WHERE attrelid = %s AND attnum > 0 AND NOT attisdropped ORDER BY attnum
```

`_schema_object_defs_collect._view_deps` line 2479

```sql
SELECT DISTINCT dep.relname AS name, dep.relkind AS kind
  FROM pg_depend d
  JOIN pg_rewrite r  ON r.oid = d.objid
  JOIN pg_class   dep ON dep.oid = d.refobjid
  JOIN pg_namespace dn ON dn.oid = dep.relnamespace
 WHERE d.classid = 'pg_rewrite'::regclass
   AND d.refclassid = 'pg_class'::regclass
   AND r.ev_class = %s AND dep.oid <> %s
   AND dn.nspname = %s
   AND dep.relkind IN ('r','p','v','m','f')
```

`_schema_object_defs_collect` line 2499

```sql
SELECT c.relname AS name, c.relkind AS kind, c.oid AS oid
FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relkind = 'v'
  AND NOT EXISTS (SELECT 1 FROM pg_depend dx
                   WHERE dx.classid = 'pg_class'::regclass
                     AND dx.objid = c.oid AND dx.deptype = 'e')
ORDER BY c.relname
```

`_schema_object_defs_collect` line 2510

```sql
SELECT pg_get_viewdef(%s, true) AS d
```

`_schema_object_defs_collect` line 2526

```sql
SELECT c.relname AS name, c.oid AS oid,
       c.relispopulated AS populated
FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relkind = 'm'
  AND NOT EXISTS (SELECT 1 FROM pg_depend dx
                   WHERE dx.classid = 'pg_class'::regclass
                     AND dx.objid = c.oid AND dx.deptype = 'e')
ORDER BY c.relname
```

`_schema_object_defs_collect` line 2538

```sql
SELECT pg_get_viewdef(%s, true) AS d
```

`_schema_object_defs_collect` line 2580

```sql
SELECT s.sequencename AS name, s.data_type, s.start_value,
       s.increment_by, s.min_value, s.max_value, s.cycle,
       s.cache_size, c.relpersistence::text AS persistence
  FROM pg_sequences s
  JOIN pg_class c ON c.relname = s.sequencename
  JOIN pg_namespace n ON n.oid = c.relnamespace
                     AND n.nspname = s.schemaname
 WHERE s.schemaname = %s AND c.relkind = 'S'
 ORDER BY s.sequencename
```

`_schema_object_defs_collect` line 2593

```sql
SELECT c.relname AS name, c.relpersistence::text AS persistence
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind = 'S'
 ORDER BY c.relname
```

`_schema_object_defs_collect` line 2603

```sql
SELECT start_value, increment_by, min_value, max_value, is_cycled AS cycle, cache_value AS cache_size FROM {fq}
```

`_schema_object_defs_collect` line 2673

```sql
SELECT p.oid AS oid, p.proname AS name, %s AS prokind,
       pg_get_function_identity_arguments(p.oid) AS args
FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname = %%s
  -- 'i' = a function the server made for something else (a range
  -- type's constructors); 'e' = a function that belongs to an
  -- EXTENSION. Neither is an object anybody declared here, and
  -- an extension's are the ones that broke cloning outright:
  -- CREATE FUNCTION ... LANGUAGE c needs superuser, which no
  -- managed-instance user has, so a schema holding pg_trgm or
  -- vector could not be cloned at all. They arrive with
  -- CREATE EXTENSION on the target instead - see the clone
  -- preflight, which lists the extensions to install.
  AND NOT EXISTS (SELECT 1 FROM pg_depend d
                   WHERE d.classid = 'pg_proc'::regclass
                     AND d.objid = p.oid AND d.deptype IN ('i','e'))
ORDER BY p.proname, args
```

`_schema_object_defs_collect` line 2721

```sql
SELECT pg_get_functiondef(%s) AS d
```

`_schema_object_defs_collect` line 2749

```sql
SELECT c.relname AS name, c.oid AS oid, s.srvname AS server,
       COALESCE(array_to_string(ft.ftoptions, ', '), '') AS options
  FROM pg_foreign_table ft
  JOIN pg_class c ON c.oid = ft.ftrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_foreign_server s ON s.oid = ft.ftserver
 WHERE n.nspname = %s
   -- An extension's own objects arrive with CREATE
   -- EXTENSION and are not compared one by one (here and in
   -- the four collectors below, as in the ones above).
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.classid = 'pg_class'::regclass
                    AND dx.objid = c.oid AND dx.deptype = 'e')
 ORDER BY c.relname
```

`_schema_object_defs_collect` line 2766

```sql
SELECT a.attname,
       pg_catalog.format_type(a.atttypid, a.atttypmod) AS typ,
       a.attnotnull,
       COALESCE(array_to_string(a.attfdwoptions, ', '), '') AS opts
  FROM pg_attribute a
 WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

`_schema_object_defs_collect` line 2796

```sql
SELECT c.relname || '.' || r.rulename AS name,
       pg_catalog.pg_get_ruledef(r.oid, true) AS def,
       r.ev_enabled::text AS en, c.relkind::text AS relkind
  FROM pg_rewrite r
  JOIN pg_class c ON c.oid = r.ev_class
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND r.rulename <> '_RETURN'
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.classid = 'pg_class'::regclass
                    AND dx.objid = c.oid AND dx.deptype = 'e')
 ORDER BY 1
```

`_schema_object_defs_collect` line 2824

```sql
SELECT st.stxname AS name,
       pg_catalog.pg_get_statisticsobjdef(st.oid) AS def
  FROM pg_statistic_ext st
  JOIN pg_namespace n ON n.oid = st.stxnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.classid = 'pg_statistic_ext'::regclass
                    AND dx.objid = st.oid AND dx.deptype = 'e')
 ORDER BY st.stxname
```

`_schema_object_defs_collect` line 2848

```sql
SELECT c.collname AS name, c.collprovider::text AS provider,
       c.collcollate AS lc_collate, c.collctype AS lc_ctype,
       c.collisdeterministic AS deterministic
  FROM pg_collation c
  JOIN pg_namespace n ON n.oid = c.collnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.classid = 'pg_collation'::regclass
                    AND dx.objid = c.oid AND dx.deptype = 'e')
 ORDER BY c.collname
```

`_schema_object_defs_collect` line 2877

```sql
SELECT c.cfgname AS name,
       (SELECT pn.nspname || '.' || p.prsname FROM pg_ts_parser p
          JOIN pg_namespace pn ON pn.oid = p.prsnamespace
         WHERE p.oid = c.cfgparser) AS detail
  FROM pg_ts_config c
  JOIN pg_namespace n ON n.oid = c.cfgnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.classid = 'pg_ts_config'::regclass
                    AND dx.objid = c.oid AND dx.deptype = 'e') ORDER BY c.cfgname
```

`_schema_object_defs_collect` line 2887

```sql
SELECT d.dictname AS name,
       (SELECT tn.nspname || '.' || t.tmplname FROM pg_ts_template t
          JOIN pg_namespace tn ON tn.oid = t.tmplnamespace
         WHERE t.oid = d.dicttemplate)
       || ' ' || COALESCE(d.dictinitoption, '') AS detail
  FROM pg_ts_dict d
  JOIN pg_namespace n ON n.oid = d.dictnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.classid = 'pg_ts_dict'::regclass
                    AND dx.objid = d.oid AND dx.deptype = 'e') ORDER BY d.dictname
```

`_schema_object_defs_collect` line 2898

```sql
SELECT p.prsname AS name,
       p.prsstart::regprocedure::text || ' '
       || p.prstoken::regprocedure::text AS detail
  FROM pg_ts_parser p
  JOIN pg_namespace n ON n.oid = p.prsnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.classid = 'pg_ts_parser'::regclass
                    AND dx.objid = p.oid AND dx.deptype = 'e') ORDER BY p.prsname
```

`_schema_object_defs_collect` line 2907

```sql
SELECT t.tmplname AS name,
       COALESCE(NULLIF(t.tmplinit, 0)::regprocedure::text, '')
       || ' ' || t.tmpllexize::regprocedure::text AS detail
  FROM pg_ts_template t
  JOIN pg_namespace n ON n.oid = t.tmplnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx WHERE dx.classid = 'pg_ts_template'::regclass
                    AND dx.objid = t.oid AND dx.deptype = 'e') ORDER BY t.tmplname
```

`_schema_object_defs_collect` line 2933

```sql
SELECT o.oprname || ' ('
       || COALESCE(pg_catalog.format_type(NULLIF(o.oprleft, 0), NULL), 'NONE') || ', '
       || COALESCE(pg_catalog.format_type(NULLIF(o.oprright, 0), NULL), 'NONE') || ')' AS name,
       o.oprcode::regprocedure::text AS detail
  FROM pg_operator o
  JOIN pg_namespace n ON n.oid = o.oprnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx
                    WHERE dx.classid = 'pg_operator'::regclass
                      AND dx.objid = o.oid AND dx.deptype = 'e') ORDER BY 1
```

`_schema_object_defs_collect` line 2944

```sql
SELECT c.opcname || ' (' || am.amname || ')' AS name,
       pg_catalog.format_type(c.opcintype, NULL)
       || CASE WHEN c.opcdefault THEN ' default' ELSE '' END AS detail,
       am.amname AS am,
       c.opcname AS bare
  FROM pg_opclass c
  JOIN pg_am am ON am.oid = c.opcmethod
  JOIN pg_namespace n ON n.oid = c.opcnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx
                    WHERE dx.classid = 'pg_opclass'::regclass
                      AND dx.objid = c.oid AND dx.deptype = 'e') ORDER BY 1
```

`_schema_object_defs_collect` line 2957

```sql
SELECT f.opfname || ' (' || am.amname || ')' AS name, '' AS detail,
       am.amname AS am,
       f.opfname AS bare
  FROM pg_opfamily f
  JOIN pg_am am ON am.oid = f.opfmethod
  JOIN pg_namespace n ON n.oid = f.opfnamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx
                    WHERE dx.classid = 'pg_opfamily'::regclass
                      AND dx.objid = f.oid AND dx.deptype = 'e') ORDER BY 1
```

`_schema_object_defs_collect` line 2988

```sql
SELECT c.conname AS name,
       pg_catalog.pg_encoding_to_char(c.conforencoding) || ' -> '
       || pg_catalog.pg_encoding_to_char(c.contoencoding)
       || ' ' || c.conproc::regprocedure::text
       || CASE WHEN c.condefault THEN ' default' ELSE '' END AS detail
  FROM pg_conversion c
  JOIN pg_namespace n ON n.oid = c.connamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend dx
                    WHERE dx.classid = 'pg_conversion'::regclass
                      AND dx.objid = c.oid AND dx.deptype = 'e') ORDER BY c.conname
```

`_schema_object_defs_collect` line 3025

```sql
SELECT c.relname AS tab, t.tgname AS name,
       pg_catalog.pg_get_triggerdef(t.oid, true) AS d, t.tgenabled::text AS en
  FROM pg_trigger t
  JOIN pg_class c ON c.oid = t.tgrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %%s AND NOT t.tgisinternal %s
 ORDER BY c.relname, t.tgname
```

`_schema_object_defs_collect` line 3052

```sql
SELECT c.relname AS name, c.relkind::text AS relkind,
       pg_catalog.obj_description(c.oid, 'pg_class') AS cmt,
       a.attname AS col,
       pg_catalog.col_description(c.oid, a.attnum) AS colcmt
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum > 0
                          AND NOT a.attisdropped
                          AND pg_catalog.col_description(c.oid, a.attnum) IS NOT NULL
 WHERE n.nspname = %s AND c.relkind IN ('r','p','v','m','S')
```

`_schema_object_defs_collect` line 3075

```sql
SELECT p.proname AS name,
       pg_catalog.pg_get_function_identity_arguments(p.oid) AS args,
       pg_catalog.obj_description(p.oid, 'pg_proc') AS cmt
  FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname = %s
   AND pg_catalog.obj_description(p.oid, 'pg_proc') IS NOT NULL
```

`_schema_object_defs_collect` line 3087

```sql
SELECT t.typname AS name,
       pg_catalog.obj_description(t.oid, 'pg_type') AS cmt
  FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace
 WHERE n.nspname = %s
   AND pg_catalog.obj_description(t.oid, 'pg_type') IS NOT NULL
```

`_schema_object_defs_collect` line 3127

```sql
SELECT c.relname AS tab, t.tgname AS name,
       pg_catalog.obj_description(t.oid, 'pg_trigger') AS cmt
  FROM pg_trigger t
  JOIN pg_class c ON c.oid = t.tgrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND NOT t.tgisinternal
   AND pg_catalog.obj_description(t.oid, 'pg_trigger') IS NOT NULL
```

`_schema_object_defs_collect` line 3141

```sql
SELECT c.relname AS tab, rw.rulename AS name,
       pg_catalog.obj_description(rw.oid, 'pg_rewrite') AS cmt
  FROM pg_rewrite rw
  JOIN pg_class c ON c.oid = rw.ev_class
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND rw.rulename <> '_RETURN'
   AND pg_catalog.obj_description(rw.oid, 'pg_rewrite') IS NOT NULL
```

`_schema_object_defs_collect` line 3155

```sql
SELECT c.relname AS tab, pol.polname AS name,
       pg_catalog.obj_description(pol.oid, 'pg_policy') AS cmt
  FROM pg_policy pol
  JOIN pg_class c ON c.oid = pol.polrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s
   AND pg_catalog.obj_description(pol.oid, 'pg_policy') IS NOT NULL
```

`_schema_object_defs_collect` line 3169

```sql
SELECT s.stxname AS name,
       pg_catalog.obj_description(s.oid, 'pg_statistic_ext') AS cmt
  FROM pg_statistic_ext s
  JOIN pg_namespace n ON n.oid = s.stxnamespace
 WHERE n.nspname = %s
   AND pg_catalog.obj_description(s.oid, 'pg_statistic_ext') IS NOT NULL
```

`_schema_object_defs_collect` line 3182

```sql
SELECT co.collname AS name,
       pg_catalog.obj_description(co.oid, 'pg_collation') AS cmt
  FROM pg_collation co
  JOIN pg_namespace n ON n.oid = co.collnamespace
 WHERE n.nspname = %s
   AND pg_catalog.obj_description(co.oid, 'pg_collation') IS NOT NULL
```

`_tsdb_annotate` line 3247

```sql
SELECT c.relname, c.relkind::text FROM pg_catalog.pg_class c
                           JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
                          WHERE n.nspname = %s AND c.oid = ANY(%s)
```

`_tsdb_annotate` line 3256

```sql
SELECT hypertable_name::text, attname::text, segmentby_column_index,
                                orderby_column_index, orderby_asc
                           FROM timescaledb_information.compression_settings
                          WHERE hypertable_schema = %s
                          ORDER BY 1, segmentby_column_index NULLS LAST, orderby_column_index NULLS LAST
```

`_read_seq_values` line 3578

```sql
SELECT c.relname AS name FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind = 'S' ORDER BY c.relname
```

`_read_seq_values` line 3587

```sql
SELECT last_value, is_called FROM {_qrel(schema, nm)}
```

`_read_row_counts` line 3606

```sql
SELECT c.relname, c.relkind::text AS relkind,
       CASE WHEN c.reltuples < 0 THEN NULL ELSE c.reltuples::bigint END AS estimated
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind IN ('r','p','m')
 ORDER BY c.relname
```

`_read_row_counts` line 3620

```sql
SELECT count(*) AS n FROM {}.{}
```

`_read_sizes` line 3636

```sql
SELECT c.relname, pg_catalog.pg_total_relation_size(c.oid) AS bytes
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind IN ('r','m')
```

`_read_index_state` line 3649

```sql
SELECT t.relname AS tab, i.relname AS idx, x.indisvalid AS valid,
       COALESCE(s.idx_scan, 0) AS scans,
       COALESCE(s.idx_tup_read, 0) AS tup_read,
       COALESCE(s.idx_tup_fetch, 0) AS tup_fetch
  FROM pg_index x
  JOIN pg_class i ON i.oid = x.indexrelid
  JOIN pg_class t ON t.oid = x.indrelid
  JOIN pg_namespace n ON n.oid = t.relnamespace
  LEFT JOIN pg_stat_user_indexes s ON s.indexrelid = x.indexrelid
 WHERE n.nspname = %s
```

**_READS_SQL** - `(module)` line 3893

```sql
SELECT n.nspname, c.relname, a.attname
  FROM pg_rewrite r
  JOIN pg_depend d ON d.classid = 'pg_rewrite'::regclass AND d.objid = r.oid
                  AND d.refclassid = 'pg_class'::regclass AND d.refobjsubid > 0
  JOIN pg_class c ON c.oid = d.refobjid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_attribute a ON a.attrelid = d.refobjid AND a.attnum = d.refobjsubid
 WHERE {where}
 ORDER BY 1, 2, 3
```

`_restore_user` line 3911

```sql
SELECT pg_get_ruledef(oid, true), obj_description(oid, 'pg_rewrite') FROM pg_rewrite WHERE oid = %s
```

`_restore_user` line 3917

```sql
COMMENT ON RULE {_qi(u['name'])} ON {on} IS {_literal_str(cmt)}
```

**drop** - `_restore_user` line 3921

```sql
DROP RULE IF EXISTS {_qi(u['name'])} ON {on}
```

`_restore_user` line 3927

```sql
SELECT relispopulated FROM pg_class WHERE oid = %s
```

**drop** - `_restore_user` line 3934

```sql
DROP {word} IF EXISTS {q}
```

`_attachment_column` line 3945

```sql
ALTER (?:MATERIALIZED )?VIEW {re.escape(ref)} ALTER COLUMN {_QID}
```

`_attachment_column` line 3946

```sql
COMMENT ON COLUMN {re.escape(ref)}\.{_QID}
```

`_attachment_column` line 3947

```sql
GRANT [A-Z ]+ \({_QID}\) ON TABLE {re.escape(ref)}
```

`_build_object_ddl` line 4282

```sql
CREATE TABLE {qi(schema)}.{qi(name)} (
{cols_part}
);
```

`_build_object_ddl` line 4284

```sql
ALTER TABLE {qi(schema)}.{qi(name)} ADD CONSTRAINT {qi(cn)} {sub(m['cons'][cn])};
```

`_build_object_ddl` line 4292

```sql
CREATE {kw} {qi(schema)}.{qi(name)} AS
{sub(m.get('viewdef') or '').rstrip().rstrip(';')};
```

`_build_object_ddl` line 4295

```sql
CREATE {'UNLOGGED ' if m.get('persistence') == 'u' else ''}SEQUENCE {qi(schema)}.{qi(name)}{' AS ' + (m.get('data_type') or '') if _typed_seq else ''} START {str(m.get('start'))} INCREMENT {str(m.get('inc'))} MINVALUE {str(m.get('min'))} MAXVALUE {str(m.get('max'))}{' CACHE ' + str(m.get('cache')) if m.get('cache') else ''}{' CYCLE' if m.get('cycle') else ''};
```

`_rls_sql` line 4330

```sql
ALTER TABLE {t} ENABLE ROW LEVEL SECURITY;
```

`_rls_sql` line 4332

```sql
ALTER TABLE {t} FORCE ROW LEVEL SECURITY;
```

`_rls_sql` line 4337

```sql
DROP POLICY IF EXISTS {qi(nm)} ON {t};
```

`_rls_sql` line 4343

```sql
CREATE POLICY {qi(nm)} ON {t}{'' if p.get('permissive', True) else ' AS RESTRICTIVE'}{' FOR ' + cmd if cmd else ''}{' TO ' + ', '.join((qi(rr) for rr in p['roles'])) if p.get('roles') else ''}{' USING (' + sub(p['using']) + ')' if p.get('using') else ''}{' WITH CHECK (' + sub(p['check']) + ')' if p.get('check') else ''};
```

**st** - `_build_sync_sql._detach` line 4427

```sql
ALTER TABLE {qi(sb)}.{qi(parent)} DETACH PARTITION {t}
```

**st** - `_build_sync_sql._attach` line 4450

```sql
ALTER TABLE {qi(sb)}.{qi(parent)} ATTACH PARTITION {qi(sb)}.{qi(name)} {bound or 'DEFAULT'};
```

`_build_sync_sql._hypertable_note` line 4481

```sql
SELECT set_chunk_time_interval({lit}, {iv_sql});
```

`_build_sync_sql._hypertable_note` line 4483

```sql
 SET (timescaledb.compress ...)
-- after decompressing its compressed chunks.
```

`_build_sync_sql._storage_sql` line 4551

```sql
ALTER TABLE {t} SET TABLESPACE {qi(ts_a)};
```

`_build_sync_sql._storage_sql` line 4555

```sql
DO $$BEGIN EXECUTE format('ALTER TABLE %s SET TABLESPACE %I', {_literal_str(t)}, (SELECT ts.spcname FROM pg_catalog.pg_tablespace ts JOIN pg_catalog.pg_database d ON d.dattablespace = ts.oid WHERE d.datname = current_database())); END$$;
```

`_build_sync_sql._storage_sql` line 4564

```sql
ALTER TABLE {t} SET ACCESS METHOD {qi(am_a or 'heap')};
```

`_build_sync_sql._nn_not_valid_sql` line 4620

```sql
ALTER TABLE {t} ADD CONSTRAINT {qi(c['notnull_invalid'])} NOT NULL {qi(c['name'])} NOT VALID;
```

**head** - `_build_sync_sql._identity_alter` line 4630

```sql
ALTER TABLE {t} ALTER COLUMN {qi(c['name'])}
```

`_build_sync_sql._identity_alter` line 4636

```sql
DROP IDENTITY IF EXISTS;
```

`_build_sync_sql._identity_alter` line 4646

```sql
SET {kw} {str(sa[k])}
```

**head** - `_build_sync_sql._col_props_alter` line 4661

```sql
ALTER TABLE {t} ALTER COLUMN {qi(c['name'])}
```

`_build_sync_sql._add_constraint_idempotent` line 4732

```sql
ALTER TABLE {t} ADD CONSTRAINT {qi(cn)} {sub(cdef)};
```

`_build_sync_sql._add_constraint_idempotent` line 4734

```sql
DO $$
BEGIN
  ALTER TABLE {t} ADD CONSTRAINT {qi(cn)} {sub(cdef)};
EXCEPTION WHEN duplicate_object OR duplicate_table OR invalid_table_definition THEN NULL;
END $$;
```

`_build_sync_sql._idempotent_type` line 4752

```sql
DO $$
BEGIN
  {stmt.rstrip().rstrip(';')};
EXCEPTION WHEN duplicate_object THEN NULL;
END $$;
```

`_build_sync_sql.create_one` line 4760

```sql
CREATE TYPE {n} AS ENUM ({', '.join((_literal_str(l) for l in m.get('labels') or []))})
```

`_build_sync_sql.create_one` line 4769

```sql
CREATE DOMAIN {n} AS {sub(m.get('base') or 'text')}{' DEFAULT ' + m['typdefault'] if m.get('typdefault') else ''}{' NOT NULL' if m.get('notnull') else ''}{''.join((' CONSTRAINT ' + qi(c) + ' ' + sub(m['checks'][c]) for c in sorted(m.get('checks') or {}) if c not in nv))}
```

`_build_sync_sql.create_one` line 4776

```sql
ALTER DOMAIN {n} ADD CONSTRAINT {qi(c)} {sub(m['checks'][c])}
```

`_build_sync_sql.create_one` line 4797

```sql
CREATE TYPE {n} AS RANGE ({', '.join(opts)})
```

`_build_sync_sql.create_one` line 4800

```sql
CREATE TYPE {n} AS ({', '.join((qi(f['name']) + ' ' + sub(f['type']) for f in m.get('fields') or []))})
```

`_build_sync_sql.create_one` line 4810

```sql
CREATE FOREIGN TABLE IF NOT EXISTS {qi(sb)}.{qi(name)} (
{cols}
) SERVER {qi(m.get('server') or '')}{' OPTIONS (' + m['options'] + ')' if m.get('options') else ''};
```

`_build_sync_sql.create_one` line 4829

```sql
CREATE COLLATION {qi(sb)}.{qi(name)} (provider = {prov}, locale = {_literal_str(m.get('lc_collate') or '')}{'' if m.get('deterministic', True) else ', deterministic = false'})
```

`_build_sync_sql.create_one` line 4851

```sql
DROP TRIGGER IF EXISTS {qi(m.get('trigger') or name)} ON {qi(sb)}.{qi(m.get('table') or '')};
{sub(m.get('trigdef') or '').rstrip().rstrip(';')};{'\n' + fire if fire else ''}
```

`_build_sync_sql.create_one` line 4866

```sql
CREATE {_unlogged(m)}TABLE IF NOT EXISTS {t} PARTITION OF {qi(sb)}.{qi(m['parent'])} {m.get('partbound') or 'DEFAULT'}{' PARTITION BY ' + sub(m['partkey']) if m.get('partkey') else ''}{storage};
```

`_build_sync_sql.create_one` line 4878

```sql
CREATE {_unlogged(m)}TABLE IF NOT EXISTS {t} (
{',\n'.join((col_line(c) for c in own))}
){' INHERITS (' + ', '.join((sub(p) for p in m['inherits'])) + ')' if m.get('inherits') else ''}{' PARTITION BY ' + sub(m['partkey']) if m.get('partkey') else ''}{storage};
```

`_build_sync_sql.create_one` line 4909

```sql
ALTER TABLE {t} ADD CONSTRAINT {qi(c['notnull_name'])} NOT NULL {qi(c['name'])};
```

`_build_sync_sql.create_one` line 4912

```sql
ALTER TABLE {t} ALTER COLUMN {qi(c['name'])} SET NOT NULL;
```

`_build_sync_sql.create_one` line 4918

```sql
ALTER TABLE {t} SET ({', '.join(m['reloptions'])});
```

`_build_sync_sql.create_one` line 4933

```sql
ALTER TABLE {t} REPLICA IDENTITY {_replident_sql(m)};
```

`_build_sync_sql.create_one` line 4942

```sql
CREATE OR REPLACE VIEW {qi(sb)}.{qi(name)} AS
{sub(m['viewdef']).rstrip().rstrip(';')};
```

`_build_sync_sql.create_one` line 4945

```sql
CREATE MATERIALIZED VIEW IF NOT EXISTS {qi(sb)}.{qi(name)} AS
{sub(m['viewdef']).rstrip().rstrip(';')}{'' if m.get('populated', True) else '\n  WITH NO DATA'};
```

`_build_sync_sql.create_one` line 4949

```sql
CREATE {'UNLOGGED ' if m.get('persistence') == 'u' else ''}SEQUENCE {'IF NOT EXISTS ' if _seq_ine else ''}{qi(sb)}.{qi(name)}{' AS ' + m['data_type'] if _typed_seq else ''} START {str(m['start'])} INCREMENT {str(m['inc'])} MINVALUE {str(m['min'])} MAXVALUE {str(m['max'])}{' CACHE ' + str(m['cache']) if m.get('cache') else ''}{' CYCLE' if m['cycle'] else ''};
```

`_build_sync_sql.create_one` line 4962

```sql
CREATE AGGREGATE {_routine_ref(m)} {sub(m['aggbody'])}
```

`_build_sync_sql.drop_one` line 4969

```sql
DROP {_routine_kw(m or {})} IF EXISTS {_routine_ref(m or {})};
```

`_build_sync_sql.drop_one` line 4973

```sql
DROP TRIGGER IF EXISTS {qi(m.get('trigger') or name)} ON {qi(sb)}.{qi(m.get('table') or '')};
```

`_build_sync_sql.drop_one` line 4977

```sql
DROP {kw} IF EXISTS {qi(sb)}.{qi(name)};
```

`_build_sync_sql.drop_one` line 4980

```sql
DROP RULE IF EXISTS {qi(rl)} ON {qi(sb)}.{qi(tbl)};
```

`_build_sync_sql.drop_one` line 4983

```sql
DROP FOREIGN TABLE IF EXISTS {qi(sb)}.{qi(name)};
```

`_build_sync_sql.drop_one` line 4985

```sql
DROP {{'statistics': 'STATISTICS', 'collation': 'COLLATION', 'conversion': 'CONVERSION'}[otype]} IF EXISTS {qi(sb)}.{qi(name)};
```

`_build_sync_sql.drop_one` line 4998

```sql
DROP {kw} IF EXISTS {qi(sb)}.{qi(name)};
```

`_build_sync_sql._drop_column_sql` line 5107

```sql
ALTER TABLE {t} DROP COLUMN {qi(c)};  -- DESTRUCTIVE
```

`_build_sync_sql.alter_one` line 5169

```sql
ALTER TYPE {qi(sb)}.{qi(name)} ADD VALUE IF NOT EXISTS {_literal_str(l)}{' AFTER ' + _literal_str(prev) if prev else ''};
```

`_build_sync_sql.alter_one` line 5182

```sql
ALTER DOMAIN {qi(sb)}.{qi(name)} DROP CONSTRAINT IF EXISTS {qi(c)};
```

`_build_sync_sql.alter_one` line 5186

```sql
ALTER DOMAIN {qi(sb)}.{qi(name)} DROP CONSTRAINT IF EXISTS {qi(c)};
```

`_build_sync_sql.alter_one` line 5188

```sql
ALTER DOMAIN {qi(sb)}.{qi(name)} ADD CONSTRAINT {qi(c)} {sub(ma['checks'][c])};
```

`_build_sync_sql.alter_one` line 5200

```sql
CREATE OR REPLACE VIEW {qi(sb)}.{qi(name)} AS
{sub(ma['viewdef']).rstrip().rstrip(';')};
```

`_build_sync_sql.alter_one` line 5207

```sql
DROP AGGREGATE IF EXISTS {_routine_ref(ma)};
{create_one('function', name, ma)}
```

`_build_sync_sql.alter_one` line 5211

```sql
DROP MATERIALIZED VIEW IF EXISTS {qi(sb)}.{qi(name)};
CREATE MATERIALIZED VIEW {qi(sb)}.{qi(name)} AS
{sub(ma['viewdef']).rstrip().rstrip(';')};
```

`_build_sync_sql.alter_one` line 5215

```sql
ALTER SEQUENCE {qi(sb)}.{qi(name)}{' AS ' + ma['data_type'] if _typed_seq and ma.get('data_type') != mb.get('data_type') else ''} INCREMENT {str(ma['inc'])} MINVALUE {str(ma['min'])} MAXVALUE {str(ma['max'])}{' CACHE ' + str(ma['cache']) if ma.get('cache') else ''}{' CYCLE' if ma['cycle'] else ' NO CYCLE'};
```

`_build_sync_sql.alter_one` line 5227

```sql
ALTER SEQUENCE {qi(sb)}.{qi(name)} OWNED BY {qi(sb)}.{qi(tab)}.{qi(col)};
```

`_build_sync_sql.alter_one` line 5230

```sql
ALTER SEQUENCE {qi(sb)}.{qi(name)} OWNED BY NONE;
```

`_build_sync_sql.alter_one` line 5259

```sql
ALTER TABLE {t} NO INHERIT {sub(p)};
```

`_build_sync_sql.alter_one` line 5260

```sql
ALTER TABLE {t} INHERIT {sub(p)};
```

`_build_sync_sql.alter_one` line 5285

```sql
ALTER TABLE {t} ADD COLUMN IF NOT EXISTS {col_line(c).strip()};
```

`_build_sync_sql.alter_one` line 5299

```sql
ALTER TABLE {t} ALTER COLUMN {qi(n)} TYPE {c['type']};
```

`_build_sync_sql.alter_one` line 5302

```sql
ALTER TABLE {t} ALTER COLUMN {qi(n)}{' SET DEFAULT ' + sub(c['default']) if c['default'] else ' DROP DEFAULT'};
```

`_build_sync_sql.alter_one` line 5309

```sql
ALTER TABLE {t} ALTER COLUMN {qi(n)}{' SET' if c['notnull'] else ' DROP'} NOT NULL;
```

`_build_sync_sql.alter_one` line 5357

```sql
ALTER TABLE {t} DROP CONSTRAINT IF EXISTS {qi(cn)};
```

`_build_sync_sql.alter_one` line 5364

```sql
ALTER TABLE {t} DROP CONSTRAINT IF EXISTS {qi(cn)};
```

`_build_sync_sql.alter_one` line 5369

```sql
DROP INDEX IF EXISTS {qi(sb)}.{qi(ixn)};
```

`_build_sync_sql.alter_one` line 5374

```sql
DROP INDEX IF EXISTS {qi(sb)}.{qi(ixn)};
```

`_build_sync_sql.alter_one` line 5386

```sql
ALTER TABLE {t} RESET ({', '.join(gone)});
```

`_build_sync_sql.alter_one` line 5388

```sql
ALTER TABLE {t} SET ({', '.join(ma['reloptions'])});
```

`_build_sync_sql.alter_one` line 5396

```sql
DROP POLICY IF EXISTS {qi(nm)} ON {t};
```

`_build_sync_sql.alter_one` line 5398

```sql
ALTER TABLE {t} DISABLE ROW LEVEL SECURITY;
```

`_build_sync_sql.alter_one` line 5400

```sql
ALTER TABLE {t} NO FORCE ROW LEVEL SECURITY;
```

`_build_sync_sql.alter_one` line 5405

```sql
ALTER TABLE {t} REPLICA IDENTITY {_replident_sql(ma)};
```

`_build_sync_sql._emit_creates` line 5638

```sql
ALTER SEQUENCE {qi(sb)}.{qi(n)} OWNED BY {qi(sb)}.{qi(tab)}.{qi(col)};
```

`_build_sync_sql._emit_comments` line 5741

```sql
COMMENT ON {target} IS {_literal_str(m['comment'])};
```

`_build_sync_sql._emit_comments` line 5744

```sql
COMMENT ON COLUMN {qi(sb)}.{qi(name)}.{qi(col)} IS {_literal_str(txt)};
```

`_build_sync_sql._emit_comments` line 5751

```sql
COMMENT ON POLICY {qi(pn)} ON {qi(sb)}.{qi(name)} IS {_literal_str(pc)};
```

`_build_sync_sql._acl_sync` line 5866

```sql
REVOKE {', '.join(_words(revoke, col))} ON {t} FROM {who};
```

`_build_sync_sql._acl_sync` line 5869

```sql
REVOKE GRANT OPTION FOR {', '.join(_words(revoke_go, col))} ON {t} FROM {who};
```

`_build_sync_sql._acl_sync` line 5872

```sql
GRANT {', '.join(_words(grant, col))} ON {t} TO {who};
```

`_build_sync_sql._acl_sync` line 5875

```sql
GRANT {', '.join(_words(grant_go, col))} ON {t} TO {who} WITH GRANT OPTION;
```

**head** - `_build_sync_sql._defacl_sync` line 5903

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE {qi(role)} IN SCHEMA {qi(sb)}
```

`_build_sync_sql._defacl_sync` line 5924

```sql
WITH GRANT OPTION;
```

`_build_sync_sql._grants_one` line 5950

```sql
ALTER {t} OWNER TO {qi(ma['owner'])};
```

`_build_sync_sql._setval` line 6040

```sql
SELECT setval({seq_expr}, COALESCE({agg}, {start}), {agg} IS NOT NULL) FROM {qi(sb)}.{qi(tab)};
```

`_build_sync_sql._setval` line 6071

```sql
SELECT setval({seq_expr}, {str(int(val[0]))}, {'true' if val[1] else 'false'});
```

`_build_sync_sql._state_one` line 6100

```sql
REFRESH MATERIALIZED VIEW {qi(sb)}.{qi(name)}{'' if d['a'] else ' WITH NO DATA'};
```

`_compare_collect` line 6269

```sql
SELECT c.relname, c.relkind::text,
       COALESCE(c.reloptions, ARRAY[]::text[]) AS opts,
       COALESCE((SELECT array_agg(opt ORDER BY opt)
                  FROM unnest(tc.reloptions) opt) , ARRAY[]::text[])
         AS toast_opts
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_class tc ON tc.oid = c.reltoastrelid
 WHERE n.nspname = %s AND c.relkind IN ('r','p','m')
```

`_compare_collect` line 6289

```sql
SELECT e.extname, e.extversion,
       n.nspname AS schema
  FROM pg_extension e
  LEFT JOIN pg_namespace n ON n.oid = e.extnamespace
```

`_compare_diff` line 6367

```sql
SELECT setval({_lit(tgt)}, (SELECT last_value FROM {src}), (SELECT is_called FROM {src}));
```

`_copy_table_cross` line 6557

```sql
SELECT to_regclass(%s) IS NOT NULL
```

`_copy_table_cross` line 6562

```sql
SELECT current_setting('server_version_num')::int
```

`_copy_table_cross` line 6574

```sql
SELECT a.attname, format_type(a.atttypid, a.atttypmod),
       a.attnotnull, pg_get_expr(d.adbin, d.adrelid), %s, %s
  FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_attrdef d ON d.adrelid = c.oid AND d.adnum = a.attnum
 WHERE n.nspname = %%s AND c.relname = %%s
   AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

`_copy_table_cross` line 6592

```sql
SELECT to_regclass(%s) IS NOT NULL
```

`_copy_table_cross` line 6598

```sql
SELECT current_setting('server_version_num')::int
```

`_copy_table_cross` line 6614

```sql
ALTER TABLE {dst_q} ADD CONSTRAINT {_qi(nn_nv)} NOT NULL "{name}" NOT VALID
```

**ddl** - `_copy_table_cross` line 6618

```sql
CREATE TABLE {dst_q} ({', '.join(col_defs)})
```

`_copy_table_cross` line 6633

```sql
SELECT set_config('dbapguin.anon_salt', %s, false)
```

**select_sql** - `_copy_table_cross` line 6652

```sql
SELECT {', '.join(select_parts)} FROM {from_sql}
```

`_copy_table_cross` line 6677

```sql
SET LOCAL row_security = off
```

`_copy_table_cross` line 6680

```sql
COPY ({select_sql}) TO STDOUT WITH (FORMAT csv, HEADER false)
```

`_copy_table_cross` line 6684

```sql
COPY {dst_q} ({dst_cols_sql}) FROM STDIN WITH (FORMAT csv, HEADER false)
```

`_copy_table_cross` line 6686

```sql
COPY {dst_q} FROM {src_q}
```

## smoke_test_checks.py

`_resolve_target_role` line 77

```sql
SELECT current_user AS u
```

## timescaledb.py

`detect` line 80

```sql
SELECT e.extversion, n.nspname FROM pg_extension e
                   JOIN pg_namespace n ON n.oid = e.extnamespace WHERE e.extname = %s
```

`detect` line 87

```sql
SELECT current_setting('timescaledb.license', true),       current_setting('timescaledb.license_key', true)
```

`detect` line 97

```sql
SELECT default_version FROM pg_available_extensions WHERE name = %s
```

`detect` line 114

```sql
SELECT n.nspname FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
                  WHERE p.proname = 'cagg_watermark'
                    AND n.nspname IN ('_timescaledb_functions', '_timescaledb_internal')
                  ORDER BY n.nspname = '_timescaledb_functions' DESC LIMIT 1
```

`_has_rel` line 125

```sql
SELECT to_regclass(%s) IS NOT NULL
```

`_columns` line 130

```sql
SELECT a.attname FROM pg_attribute a WHERE a.attrelid = to_regclass(%s)
                    AND a.attnum > 0 AND NOT a.attisdropped
```

`managed` line 160

```sql
SELECT h.id, c.oid, h.compressed_hypertable_id
                   FROM _timescaledb_catalog.hypertable h
                   JOIN pg_namespace n ON n.nspname = h.schema_name
                   JOIN pg_class c ON c.relnamespace = n.oid AND c.relname = h.table_name
```

`managed` line 177

```sql
SELECT ca.mat_hypertable_id, ca.raw_hypertable_id,
                             to_regclass(format('%I.%I', ca.user_view_schema, ca.user_view_name))::oid,
                             to_regclass(format('%I.%I', ca.partial_view_schema, ca.partial_view_name))::oid,
                             {"to_regclass(format('%I.%I', ca.direct_view_schema, ca.direct_view_name))::oid" if direct else 'NULL::oid'}
                        FROM _timescaledb_catalog.continuous_agg ca
```

`managed` line 196

```sql
SELECT k.relid::oid, k.hypertable_id, k.id FROM _timescaledb_catalog.chunk k WHERE true {dropped}
```

`managed` line 198

```sql
SELECT c.oid, k.hypertable_id, k.id FROM _timescaledb_catalog.chunk k
                        JOIN pg_namespace n ON n.nspname = k.schema_name
                        JOIN pg_class c ON c.relnamespace = n.oid AND c.relname = k.table_name
                       WHERE true {dropped}
```

`managed` line 222

```sql
SELECT s.relid::oid, s.compress_relid::oid FROM _timescaledb_catalog.compression_settings s
                      WHERE s.compress_relid IS NOT NULL
```

`hypertables` line 274

```sql
SELECT format('%%I.%%I', h.hypertable_schema, h.hypertable_name)::regclass::oid,
                             h.hypertable_schema, h.hypertable_name, h.owner::text, h.num_chunks,
                             h.compression_enabled
                        FROM timescaledb_information.hypertables h WHERE {where}
```

`hypertables` line 284

```sql
SELECT format('%%I.%%I', h.hypertable_schema, h.hypertable_name)::regclass::oid,
                             h.dimension_number, h.column_name::text, h.column_type::text,
                             h.dimension_type, h.time_interval, h.integer_interval, h.num_partitions
                        FROM timescaledb_information.dimensions h WHERE {where}
                       ORDER BY 1, 2
```

`hypertables` line 298

```sql
SELECT format('%%I.%%I', h.hypertable_schema, h.hypertable_name)::regclass::oid,
                             count(*) FILTER (WHERE h.is_compressed)
                        FROM timescaledb_information.chunks h WHERE {where} GROUP BY 1
```

`hypertables` line 306

```sql
SELECT format('%%I.%%I', h.table_schema, h.table_name)::regclass::oid,
                             h.table_schema, h.table_name, h.table_owner::text, h.num_chunks
                        FROM timescaledb_information.hypertable h WHERE {where1}
```

`hypertables` line 313

```sql
SELECT format('%I.%I', h.schema_name, h.table_name)::regclass::oid,
                            d.column_name::text, format_type(d.column_type, NULL),
                            d.interval_length, d.num_slices
                       FROM _timescaledb_catalog.dimension d
                       JOIN _timescaledb_catalog.hypertable h ON h.id = d.hypertable_id
                      ORDER BY 1, d.id
```

`row_estimates` line 355

```sql
SELECT chunk_id, {num} FROM _timescaledb_catalog.compression_chunk_size
```

`chunks` line 392

```sql
SELECT format('%%I.%%I', n.nspname, c.relname) FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = %s
```

`chunks` line 401

```sql
SELECT format('%%I.%%I', k.chunk_schema, k.chunk_name)::regclass::oid,
                            k.chunk_schema, k.chunk_name, k.range_start, k.range_end,
                            k.range_start_integer, k.range_end_integer, k.is_compressed
                       FROM timescaledb_information.chunks k
                      WHERE k.hypertable_schema = %s AND k.hypertable_name = %s
                      ORDER BY coalesce(k.range_end, '-infinity'::timestamptz) DESC,
                               k.range_end_integer DESC NULLS LAST, k.chunk_name DESC
```

`chunks` line 410

```sql
SELECT chunk_schema, chunk_name, total_bytes FROM {fs}.chunks_detailed_size(%s::regclass)
```

`chunks` line 415

```sql
SELECT chunk_schema, chunk_name, before_compression_total_bytes,
                                 after_compression_total_bytes
                            FROM {fs}.chunk_compression_stats(%s::regclass)
```

`chunks` line 442

```sql
SELECT c.oid, n.nspname, c.relname, pg_total_relation_size(c.oid)
                       FROM pg_inherits i JOIN pg_class c ON c.oid = i.inhrelid
                       JOIN pg_namespace n ON n.oid = c.relnamespace
                      WHERE i.inhparent = %s ORDER BY c.oid DESC
```

`_split_qname` line 461

```sql
SELECT n.nspname, c.relname FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = %s
```

`_jobs` line 499

```sql
SELECT j.job_id, j.application_name, j.proc_schema::text, j.proc_name::text,
                        j.owner::text, j.scheduled, j.schedule_interval, j.max_runtime, j.max_retries,
                        j.retry_period, j.config, j.next_start, j.hypertable_schema::text,
                        j.hypertable_name::text,
                        s.last_run_started_at, s.last_successful_finish, s.last_run_status,
                        s.job_status, s.last_run_duration, s.total_runs, s.total_successes,
                        s.total_failures, now(), pg_has_role(j.owner, 'USAGE')
                   FROM timescaledb_information.jobs j
                   LEFT JOIN timescaledb_information.job_stats s ON s.job_id = j.job_id
                  ORDER BY j.job_id
```

`_jobs` line 526

```sql
SELECT e.job_id, count(*), max(e.start_time),
                            (array_agg(e.err_message ORDER BY e.start_time DESC))[1],
                            (array_agg(e.sqlerrcode ORDER BY e.start_time DESC))[1]
                       FROM timescaledb_information.job_errors e
                       LEFT JOIN timescaledb_information.job_stats s ON s.job_id = e.job_id
                      WHERE e.start_time > coalesce(s.last_successful_finish, '-infinity')
                      GROUP BY e.job_id
```

`_scheduler_running` line 605

```sql
SELECT count(*) FROM pg_stat_activity
                      WHERE datname = current_database()
                        AND application_name = 'TimescaleDB Background Worker Scheduler'
```

`caggs` line 616

```sql
SELECT format('%I.%I', a.view_schema, a.view_name)::regclass::oid,
                        a.view_schema, a.view_name, a.hypertable_schema, a.hypertable_name,
                        a.materialized_only, a.compression_enabled, pg_has_role(a.view_owner, 'USAGE')
                   FROM timescaledb_information.continuous_aggregates a ORDER BY 2, 3
```

`caggs` line 643

```sql
SELECT w, CASE WHEN t IN ('timestamp with time zone', 'timestamp without time zone', 'date')
                                             THEN {_ident_str(fn)}.to_timestamp(w) END, now()
                                FROM (SELECT {_ident_str(fn)}.cagg_watermark(%s) AS w,
                                             (SELECT d.column_type::text FROM timescaledb_information.dimensions d
                                               WHERE d.hypertable_schema = %s AND d.hypertable_name = %s
                                               ORDER BY d.dimension_number LIMIT 1) AS t) x
```

`compression` line 717

```sql
SELECT total_chunks, number_compressed_chunks, before_compression_total_bytes,
                             after_compression_total_bytes FROM {fs}.hypertable_compression_stats(%s::regclass)
```

`compression` line 727

```sql
SELECT attname::text, segmentby_column_index, orderby_column_index, orderby_asc
                       FROM timescaledb_information.compression_settings
                      WHERE hypertable_schema = %s AND hypertable_name = %s
                      ORDER BY segmentby_column_index NULLS LAST, orderby_column_index NULLS LAST
```

`compression` line 744

```sql
SELECT chunk_schema::text, chunk_name::text, range_start::text
                               FROM timescaledb_information.chunks
                              WHERE hypertable_schema = %s AND hypertable_name = %s
                                AND NOT is_compressed AND range_end < now() - make_interval(secs => %s)
                              ORDER BY range_start
```

`compression` line 751

```sql
SELECT chunk_schema::text, chunk_name::text, range_start_integer::text
                               FROM timescaledb_information.chunks k
                              WHERE hypertable_schema = %s AND hypertable_name = %s AND NOT is_compressed
                                AND range_end_integer < (SELECT max(range_end_integer)
                                       FROM timescaledb_information.chunks k2
                                      WHERE k2.hypertable_schema = k.hypertable_schema
                                        AND k2.hypertable_name = k.hypertable_name) - %s
                              ORDER BY range_start_integer
```

`retention` line 792

```sql
SELECT count(*) FROM timescaledb_information.chunks
                          WHERE hypertable_schema = %s AND hypertable_name = %s
                            AND range_end < now() - make_interval(secs => %s)
```

`chunk_sizing` line 807

```sql
SELECT setting::bigint * current_setting('block_size')::bigint FROM pg_settings WHERE name = 'shared_buffers'
```

`chunk_sizing` line 812

```sql
WITH k AS (
                       SELECT format('%%I.%%I', chunk_schema, chunk_name)::regclass AS rel,
                              coalesce(range_start::text, range_start_integer::text) AS g,
                              coalesce(extract(epoch FROM range_start), range_start_integer) AS gs,
                              coalesce(range_end < now(), false) AS closed, is_compressed
                         FROM timescaledb_information.chunks
                        WHERE hypertable_schema = %s AND hypertable_name = %s),
                     pick AS (
                       SELECT g, bool_or(closed) AS closed FROM k
                        WHERE NOT is_compressed GROUP BY g, gs
                        ORDER BY bool_or(closed) DESC, gs DESC LIMIT 1)
                     SELECT pick.g, pick.closed, sum(pg_total_relation_size(k.rel)), count(*),
                            (SELECT max(gs) FROM k) >= extract(epoch FROM now() - interval '7 days')
                       FROM pick JOIN k ON k.g = pick.g AND NOT k.is_compressed
                      GROUP BY pick.g, pick.closed
```

`report` line 883

```sql
SELECT default_version FROM pg_available_extensions WHERE name = %s
```

`report` line 894

```sql
SELECT pg_has_role(c.relowner, 'USAGE') FROM pg_class c WHERE c.oid = %s
```

`report` line 903

```sql
SELECT total_bytes FROM {fs}.hypertable_detailed_size(%s::regclass)
```

`report` line 910

```sql
SELECT current_setting('timescaledb.max_background_workers', true), current_setting('max_worker_processes')
```

`add_compression_policy_sql` line 952

```sql
SELECT {_fn(info, 'add_compression_policy')}({_literal_str(_ident_str(schema, table))}::regclass, {_literal_str(compress_after)}::interval);
```

`remove_compression_policy_sql` line 957

```sql
SELECT {_fn(info, 'remove_compression_policy')}({_literal_str(_ident_str(schema, table))}::regclass);
```

`add_retention_policy_sql` line 961

```sql
SELECT {_fn(info, 'add_retention_policy')}({_literal_str(_ident_str(schema, table))}::regclass, {_literal_str(drop_after)}::interval);
```

`remove_retention_policy_sql` line 966

```sql
SELECT {_fn(info, 'remove_retention_policy')}({_literal_str(_ident_str(schema, table))}::regclass);
```

`compress_chunk_sql` line 970

```sql
SELECT {_fn(info, 'compress_chunk')}({_literal_str(_ident_str(schema, chunk))}::regclass);
```

`decompress_chunk_sql` line 974

```sql
SELECT {_fn(info, 'decompress_chunk')}({_literal_str(_ident_str(schema, chunk))}::regclass);
```

`refresh_cagg_sql` line 980

```sql
CALL {_fn(info, 'refresh_continuous_aggregate')}({_literal_str(_ident_str(schema, view))}::regclass, {s}, {e});
```

`run_job_sql` line 984

```sql
CALL {_fn(info, 'run_job')}({int(job_id)});
```

`alter_job_sql` line 993

```sql
SELECT {_fn(info, 'alter_job')}({', '.join(args)});
```

`relation_labels` line 1042

```sql
SELECT c.oid, n.nspname, c.relname FROM pg_class c
                   JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = ANY(%s)
```

## vacuum_advice.py

`stats_window` line 215

```sql
SELECT current_setting('server_version_num')::int AS v
```

`stats_window` line 228

```sql
SELECT extract(epoch FROM (now() - w.start))::numeric(18,3) AS s,
       to_char(w.start, 'YYYY-MM-DD HH24:MI TZ') AS start, w.src
  FROM (SELECT coalesce(d.stats_reset, c.cl, pg_postmaster_start_time()) AS start,
               CASE WHEN d.stats_reset IS NOT NULL THEN 'database'
                    WHEN c.cl IS NOT NULL THEN 'cluster' ELSE 'server' END AS src
          FROM pg_stat_database d, (SELECT {least} AS cl) c
         WHERE d.datname = current_database()) w
```

`horizon_holder` line 400

```sql
SELECT pid, usename, application_name, state, age({SESSION_XMIN_SQL}) AS age,
                    extract(epoch FROM (now() - xact_start))::bigint AS open_s
               FROM pg_stat_activity
              WHERE {SESSION_XMIN_SQL} IS NOT NULL AND pid <> pg_backend_pid()
                AND pid NOT IN (SELECT pid FROM pg_stat_replication WHERE pid IS NOT NULL)
              ORDER BY 5 DESC NULLS LAST LIMIT 1
```

`horizon_holder` line 413

```sql
SELECT gid, age(transaction) AS age FROM pg_prepared_xacts ORDER BY age(transaction) DESC LIMIT 1
```

`horizon_holder` line 423

```sql
SELECT s.slot_name, s.slot_type, s.active, s.xmin IS NOT NULL AS has_xmin,
                     {who_sql} AS standby,
                     greatest(coalesce(age(s.xmin), 0), coalesce(age(s.catalog_xmin), 0)) AS age
                FROM pg_replication_slots s WHERE s.xmin IS NOT NULL OR s.catalog_xmin IS NOT NULL
               ORDER BY 6 DESC LIMIT 1
```

`horizon_holder` line 435

```sql
SELECT application_name, client_addr::text AS client, age(backend_xmin) AS age
               FROM pg_stat_replication WHERE backend_xmin IS NOT NULL
              ORDER BY age(backend_xmin) DESC LIMIT 1
```

`lock_blockers` line 460

```sql
SELECT l.relation::bigint AS relid, l.mode, a.pid, a.usename, a.application_name,
       COALESCE(host(a.client_addr), 'local') AS client, a.state,
       EXTRACT(epoch FROM now() - a.xact_start)::bigint AS held_s,
       left(a.query, 200) AS query
  FROM pg_locks l JOIN pg_stat_activity a ON a.pid = l.pid
 WHERE l.locktype = 'relation' AND l.granted
   AND l.database = (SELECT oid FROM pg_database WHERE datname = current_database())
   AND l.mode IN ({', '.join(("'%s'" % m for m in VACUUM_CONFLICTS))})
   AND a.pid <> pg_backend_pid() {not_av}
   {not_vac}
   {one}
 ORDER BY a.xact_start NULLS LAST
```

`lock_blockers` line 477

```sql
SELECT l.relation::bigint AS relid, count(*)::int AS n
  FROM pg_locks l JOIN pg_stat_activity a ON a.pid = l.pid
 WHERE l.locktype = 'relation' AND NOT l.granted
   AND a.backend_type = 'autovacuum worker' {one}
 GROUP BY 1
```

**prepared** - `findings` line 1134

```sql
COMMIT PREPARED or ROLLBACK PREPARED it; its coordinator has lost track of it.
```

`autovacuum_workers` line 1258

```sql
SELECT count(*)::int AS v FROM pg_stat_activity WHERE backend_type = 'autovacuum worker'
```

`autovacuum_workers` line 1259

```sql
SELECT count(*)::int AS v FROM pg_database WHERE datallowconn
```

`autovacuum_workers` line 1261

```sql
SELECT (coalesce(sum(total_autovacuum_time), 0) + coalesce(sum(total_autoanalyze_time), 0))::float8
                    AS v FROM pg_stat_all_tables
```

`autovacuum_workers` line 1268

```sql
SELECT count(*)::int AS v FROM pg_stat_autovacuum_scores WHERE do_vacuum
```

`cluster_io` line 1341

```sql
SELECT (backend_type = 'autovacuum worker') AS av,
       sum(coalesce({rb}, 0))::bigint AS read_b,
       sum(coalesce({wb}, 0))::bigint AS write_b,
       sum(coalesce(read_time, 0))::numeric AS read_ms,
       sum(coalesce(write_time, 0))::numeric AS write_ms,
       min(stats_reset) AS since
  FROM pg_stat_io
 WHERE object <> 'wal'
 GROUP BY 1
```

`cluster_io` line 1354

```sql
SELECT (backend_type = 'autovacuum worker') AS av,
       sum(coalesce(write_bytes, 0))::bigint AS write_b
  FROM pg_stat_io WHERE object = 'wal' GROUP BY 1
```

`cluster_io` line 1359

```sql
SELECT current_setting('track_io_timing') = 'on' AS timing
```

`cluster_io` line 1375

```sql
SELECT extract(epoch FROM (now() - %s::timestamptz))::bigint AS s
```

`cluster_wal` line 1415

```sql
SELECT wal_bytes::numeric AS b, wal_fpi::bigint AS fpi,
                    extract(epoch FROM (now() - coalesce(stats_reset, pg_postmaster_start_time())))::numeric(18,3) AS s
               FROM pg_stat_wal
```

`cluster_wal` line 1422

```sql
SELECT num_done::bigint AS n, extract(epoch FROM (now() - coalesce(stats_reset, pg_postmaster_start_time())))::numeric(18,3) AS s FROM pg_stat_checkpointer
```

`cluster_wal` line 1426

```sql
SELECT (num_timed + num_requested)::bigint AS n, extract(epoch FROM (now() - coalesce(stats_reset, pg_postmaster_start_time())))::numeric(18,3) AS s FROM pg_stat_checkpointer
```

`cluster_wal` line 1430

```sql
SELECT (checkpoints_timed + checkpoints_req)::bigint AS n, extract(epoch FROM (now() - coalesce(stats_reset, pg_postmaster_start_time())))::numeric(18,3) AS s FROM pg_stat_bgwriter
```

## vector_search.py

`ext_info` line 92

```sql
SELECT a.name, a.default_version, e.extversion, n.nspname
  FROM pg_available_extensions a
  LEFT JOIN pg_extension e ON e.extname = a.name
  LEFT JOIN pg_namespace n ON n.oid = e.extnamespace
 WHERE a.name IN ('vector', 'pg_trgm')
```

`ann_opclasses` line 121

```sql
SELECT am.amname AS am, opc.opcname AS opclass, n.nspname AS schema,
       t.typname AS type, o.oprname AS op, ao.amoppurpose::text AS purpose,
       opc.opcdefault AS is_default
  FROM pg_opclass opc
  JOIN pg_am am       ON am.oid = opc.opcmethod
  JOIN pg_namespace n ON n.oid = opc.opcnamespace
  JOIN pg_type t      ON t.oid = opc.opcintype
  JOIN pg_amop ao     ON ao.amopfamily = opc.opcfamily
                     AND ao.amoplefttype = opc.opcintype
  JOIN pg_operator o  ON o.oid = ao.amopopr
 WHERE am.amname IN ('hnsw', 'ivfflat')
    OR opc.opcname IN ('gin_trgm_ops', 'gist_trgm_ops')
 ORDER BY 1, 2, 5
```

`index_columns` line 160

```sql
SELECT i.indexrelid::bigint AS indexrelid, ic.relname AS index, n.nspname AS schema,
       t.relname AS "table", t.oid::bigint AS table_oid, am.amname AS am,
       pg_relation_size(i.indexrelid) AS bytes, i.indpred IS NOT NULL AS partial,
       i.indisvalid AS valid, ic.reloptions AS reloptions,
       k.ord::int AS ord,
       pg_get_indexdef(i.indexrelid, k.ord::int, false) AS expr,
       opc.opcname AS opclass,
       ARRAY(SELECT DISTINCT o.oprname::text FROM pg_amop ao
               JOIN pg_operator o ON o.oid = ao.amopopr
              WHERE ao.amopfamily = opc.opcfamily AND ao.amoppurpose = 'o') AS order_ops,
       ARRAY(SELECT DISTINCT o.oprname::text FROM pg_amop ao
               JOIN pg_operator o ON o.oid = ao.amopopr
              WHERE ao.amopfamily = opc.opcfamily AND ao.amoppurpose = 's'
                AND ao.amoplefttype = opc.opcintype) AS search_ops
  FROM pg_index i
  JOIN pg_class ic    ON ic.oid = i.indexrelid
  JOIN pg_class t     ON t.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = t.relnamespace
  JOIN pg_am am       ON am.oid = ic.relam
  CROSS JOIN LATERAL unnest(string_to_array(i.indclass::text, ' ')::oid[])
             WITH ORDINALITY AS k(opc_oid, ord)
  JOIN pg_opclass opc ON opc.oid = k.opc_oid
 WHERE i.indrelid = ANY(%s)
 ORDER BY ic.relname, k.ord
```

**prereq** - `advisor_recs` line 398

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

`advisor_recs` line 459

```sql
WITH (lists = {lists});
```

`explain_context` line 518

```sql
SELECT c.oid::bigint AS oid, n.nspname AS schema, c.relname AS name,
       c.reltuples::bigint AS reltuples, c.relkind::text AS kind
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.relname = ANY(%s) AND c.relkind IN ('r', 'p', 'm', 'f')
   AND n.nspname NOT IN ('pg_catalog', 'information_schema')
```

`explain_context` line 529

```sql
SELECT oid::bigint FROM pg_class WHERE oid = ANY(%s) AND pg_table_is_visible(oid)
```

`explain_context` line 535

```sql
SELECT DISTINCT i.indrelid::bigint AS oid
  FROM pg_index i JOIN pg_class ic ON ic.oid = i.indexrelid
 WHERE ic.relname = ANY(%s)
   AND (pg_table_is_visible(ic.oid) OR NOT EXISTS (
          SELECT 1 FROM pg_class o WHERE o.relname = ic.relname AND o.oid <> ic.oid))
```

`explain_context` line 546

```sql
SELECT c.oid::bigint AS oid, n.nspname AS schema, c.relname AS name,
       c.reltuples::bigint AS reltuples, c.relkind::text AS kind
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.oid = ANY(%s)
```

`explain_context` line 557

```sql
SELECT a.attrelid::bigint AS oid, a.attname AS name,
       format_type(a.atttypid, a.atttypmod) AS type, t.typname AS typname,
       a.atttypmod AS typmod
  FROM pg_attribute a JOIN pg_type t ON t.oid = a.atttypid
 WHERE a.attrelid = ANY(%s) AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attrelid, a.attnum
```

`explain_context` line 577

```sql
SELECT name, setting FROM pg_settings WHERE name = ANY(%s)
```

`explain_context` line 587

```sql
SELECT relid::bigint AS oid, n_live_tup::bigint AS n_live,
       n_mod_since_analyze::bigint AS n_mod,
       extract(epoch FROM now() - GREATEST(last_analyze, last_autoanalyze))::float8 AS analyze_age_s,
       (last_analyze IS NOT NULL OR last_autoanalyze IS NOT NULL) AS analyzed
  FROM pg_stat_all_tables WHERE relid = ANY(%s)
```
