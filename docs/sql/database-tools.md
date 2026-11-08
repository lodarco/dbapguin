# Database tools

[All parts](README.md) · 526 statements

## api/autovacuum.py

`api_autovacuum_get` line 28

```sql
SELECT reloptions FROM pg_class
WHERE oid = format('%%I.%%I', %s::text, %s::text)::regclass
```

**sql** - `api_autovacuum_set` line 162

```sql
ALTER TABLE {_qq(schema, table)} SET ({', '.join(parts)})
```

**sql** - `api_autovacuum_reset` line 205

```sql
ALTER TABLE {_qq(schema, table)} RESET ({', '.join(keys)})
```

**sql** - `api_autovacuum_reset` line 207

```sql
ALTER TABLE {_qq(schema, table)} RESET ({', '.join(sorted(_AV_KEYS))})
```

## api/constraints.py

`_online_index_sql` line 110

```sql
CREATE UNIQUE INDEX CONCURRENTLY %s ON
```

`_set_search_path` line 191

```sql
SET LOCAL search_path TO %s
```

`_header` line 251

```sql
SET lock_timeout = '3s';
```

`api_constraints_list` line 284

```sql
SELECT n.nspname                       AS schema,
       c.relname                       AS table,
       con.conname                     AS name,
       con.contype                     AS contype,
       con.convalidated                AS validated,
       pg_get_constraintdef(con.oid)   AS def,
       i.relname                       AS index_name,
       pg_get_indexdef(con.conindid)   AS index_def,
       pg_relation_size(con.conindid)  AS index_bytes,
       c.reltuples::bigint             AS approx_rows,
       c.relkind                       AS relkind,
       -- A constraint a partition got from its parent cannot be
       -- dropped on the partition: PostgreSQL answers "cannot drop
       -- inherited constraint". Scripting it would produce a file
       -- that stops on that line.
       (con.coninhcount > 0 AND NOT con.conislocal) AS inherited
  FROM pg_constraint con
  JOIN pg_class c      ON c.oid = con.conrelid
  JOIN pg_namespace n  ON n.oid = c.relnamespace
  LEFT JOIN pg_class i ON i.oid = con.conindid
 WHERE {where}
   AND con.contype = ANY(%s)
 ORDER BY n.nspname, c.relname,
          CASE con.contype WHEN 'p' THEN 0 WHEN 'u' THEN 1
                           WHEN 'x' THEN 2 WHEN 'c' THEN 3
                           ELSE 4 END,
          con.conname
```

`api_constraints_list` line 320

```sql
SELECT cn.nspname                      AS schema,
       cc.relname                      AS table,
       con.conname                     AS name,
       'f'                             AS contype,
       con.convalidated                AS validated,
       pg_get_constraintdef(con.oid)   AS def,
       rn.nspname                      AS references_schema,
       rc.relname                      AS references_table,
       cc.reltuples::bigint            AS approx_rows
  FROM pg_constraint con
  JOIN pg_class cc     ON cc.oid = con.conrelid
  JOIN pg_namespace cn ON cn.oid = cc.relnamespace
  JOIN pg_class rc     ON rc.oid = con.confrelid
  JOIN pg_namespace rn ON rn.oid = rc.relnamespace
 WHERE con.contype = 'f' AND {inbound_where}
   AND NOT (cn.nspname = rn.nspname AND cc.relname = rc.relname)
 ORDER BY cn.nspname, cc.relname, con.conname
```

`api_constraints_script` line 433

```sql
SELECT n.nspname AS schema, c.relname AS table, con.conname AS name,
       con.contype AS contype, pg_get_constraintdef(con.oid) AS def,
       i.relname AS index_name, c.relkind AS relkind,
       -- The index as the server itself defines it. The rebuild
       -- used to derive it from the constraint text by string
       -- surgery, which loses everything that text does not carry
       -- and is a syntax error for everything it carries in the
       -- wrong place.
       pg_get_indexdef(con.conindid) AS index_def,
       con.condeferrable AS deferrable, con.condeferred AS deferred,
       (con.coninhcount > 0 AND NOT con.conislocal) AS inherited
  FROM pg_constraint con
  JOIN pg_class c ON c.oid = con.conrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_class i ON i.oid = con.conindid
 WHERE {where} AND con.contype = ANY(%s)
 ORDER BY n.nspname, c.relname, con.conname
```

`api_constraints_script` line 457

```sql
SELECT cn.nspname AS schema, cc.relname AS table, con.conname AS name,
       'f' AS contype, pg_get_constraintdef(con.oid) AS def,
       NULL::text AS index_name, cc.relkind AS relkind,
       (con.coninhcount > 0 AND NOT con.conislocal) AS inherited
  FROM pg_constraint con
  JOIN pg_class cc ON cc.oid = con.conrelid
  JOIN pg_namespace cn ON cn.oid = cc.relnamespace
  JOIN pg_class rc ON rc.oid = con.confrelid
  JOIN pg_namespace rn ON rn.oid = rc.relnamespace
 WHERE con.contype = 'f' AND {iw}
   AND NOT (cn.nspname = rn.nspname AND cc.relname = rc.relname)
 ORDER BY cn.nspname, cc.relname, con.conname
```

`api_constraints_script` line 479

```sql
SELECT current_setting('server_version_num')::int AS v
```

`api_constraints_script` line 512

```sql
SET search_path TO %s;
```

`api_constraints_script` line 519

```sql
ALTER TABLE %s ADD CONSTRAINT %s %s NOT VALID;
```

`api_constraints_script` line 521

```sql
ALTER TABLE %s VALIDATE CONSTRAINT %s;
```

`api_constraints_script` line 524

```sql
ALTER TABLE %s ADD CONSTRAINT %s %s;
```

`api_constraints_script` line 574

```sql
ALTER TABLE %s ALTER CONSTRAINT %s NOT ENFORCED;
```

`api_constraints_script` line 583

```sql
ALTER TABLE %s DROP CONSTRAINT %s;
```

`api_constraints_script` line 592

```sql
ALTER TABLE %s DROP CONSTRAINT %s;
```

`api_constraints_script` line 635

```sql
ALTER TABLE %s ADD CONSTRAINT %s %s USING INDEX %s%s;
```

`api_constraints_script` line 638

```sql
ALTER TABLE %s ADD CONSTRAINT %s %s NOT VALID;
```

`api_constraints_script` line 640

```sql
ALTER TABLE %s VALIDATE CONSTRAINT %s;
```

`api_constraints_script` line 642

```sql
ALTER TABLE %s ADD CONSTRAINT %s %s;
```

`api_constraints_script` line 651

```sql
ALTER TABLE %s ALTER CONSTRAINT %s ENFORCED;
```

`api_constraints_script` line 659

```sql
ALTER TABLE %s ADD CONSTRAINT %s %s NOT VALID;
```

`api_constraints_script` line 661

```sql
ALTER TABLE %s VALIDATE CONSTRAINT %s;
```

`api_constraints_script` line 663

```sql
ALTER TABLE %s ADD CONSTRAINT %s %s;
```

`api_constraints_script` line 696

```sql
CREATE INDEX CONCURRENTLY cannot run inside a transaction block, so the restore script must not be wrapped in BEGIN/COMMIT. It is also the one statement here that can fail and leave an invalid index behind. The script drops such an index itself before rebuilding, so running it again after a failure is enough - but only an index PostgreSQL has marked invalid is removed, never one in use.
```

## api/dashboard.py

`api_dashboard_overview` line 50

```sql
SELECT current_setting('server_version_num')::int AS pgver, version() AS pgver_str
```

`api_dashboard_overview` line 59

```sql
SELECT setting::int FROM pg_settings WHERE name='max_connections'
```

`api_dashboard_overview` line 68

```sql
SELECT state, count(*) AS n
FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
  %s
GROUP BY state
```

`api_dashboard_overview` line 79

```sql
SELECT count(*) AS n,
       count(*) FILTER (WHERE backend_type = 'autovacuum worker') AS av
  FROM pg_stat_activity
 WHERE backend_type <> 'client backend'
```

`api_dashboard_overview` line 89

```sql
SELECT count(*) AS n,
       coalesce(EXTRACT(EPOCH FROM max(now() - state_change)), 0)::bigint AS max_age_s
  FROM pg_stat_activity
 WHERE state LIKE 'idle in transaction%%'
   AND pid <> pg_backend_pid()
```

`api_dashboard_overview` line 107

```sql
SELECT coalesce(wait_event_type,'CPU') AS wtype,
       coalesce(wait_event,'running')  AS wevent,
       count(*) AS n
FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
  AND state IS NOT NULL
  AND state <> 'idle'
GROUP BY wtype, wevent
ORDER BY n DESC LIMIT 10
```

`api_dashboard_overview` line 126

```sql
SELECT datname,
       blks_hit, blks_read,
       xact_commit, xact_rollback,
       tup_inserted, tup_updated, tup_deleted, tup_fetched,
       deadlocks, conflicts,
       temp_files, temp_bytes,
       pg_database_size(datname) AS db_size_bytes,
       stats_reset
FROM pg_stat_database
WHERE datname = current_database()
```

`api_dashboard_overview` line 148

```sql
SELECT schemaname, relname,
       n_tup_ins, n_tup_upd, n_tup_del,
       n_tup_ins + n_tup_upd + n_tup_del AS dml_total,
       seq_scan, idx_scan,
       n_live_tup, n_dead_tup,
       last_autovacuum, last_autoanalyze,
       pg_total_relation_size(relid) AS total_bytes
FROM pg_stat_user_tables
ORDER BY dml_total DESC NULLS LAST
LIMIT 40
```

`api_dashboard_overview` line 166

```sql
SELECT num_timed AS ckpt_timed, num_requested AS ckpt_req,
       write_time AS ckpt_write_ms, sync_time AS ckpt_sync_ms,
       buffers_written AS ckpt_bufs
FROM pg_stat_checkpointer
```

`api_dashboard_overview` line 173

```sql
SELECT checkpoints_timed AS ckpt_timed, checkpoints_req AS ckpt_req,
       checkpoint_write_time AS ckpt_write_ms, checkpoint_sync_time AS ckpt_sync_ms,
       buffers_checkpoint AS ckpt_bufs
FROM pg_stat_bgwriter
```

`api_dashboard_overview` line 188

```sql
SELECT EXTRACT(EPOCH FROM now() - checkpoint_time)::bigint AS s FROM pg_control_checkpoint()
```

`api_dashboard_overview` line 195

```sql
SELECT count(*) AS blocking_count
FROM pg_stat_activity a
WHERE cardinality(pg_blocking_pids(a.pid)) > 0
```

`api_dashboard_overview` line 211

```sql
SELECT pg_is_in_recovery() AS r
```

**rep_sql** - `api_dashboard_overview` line 215

```sql
SELECT count(*) AS n,
       max(pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn)) AS max_lag,
       EXTRACT(EPOCH FROM max(replay_lag))::bigint AS max_replay_lag_s,
       EXTRACT(EPOCH FROM max(flush_lag))::bigint  AS max_flush_lag_s
FROM pg_stat_replication
```

**rep_sql** - `api_dashboard_overview` line 223

```sql
SELECT count(*) AS n,
       max(pg_xlog_location_diff(pg_current_xlog_location(), replay_location)) AS max_lag,
       NULL::bigint AS max_replay_lag_s,
       NULL::bigint AS max_flush_lag_s
FROM pg_stat_replication
```

`api_dashboard_overview` line 247

```sql
SELECT pg_wal_lsn_diff(CASE WHEN pg_is_in_recovery()
                            THEN pg_last_wal_replay_lsn()
                            ELSE pg_current_wal_lsn() END, '0/0')::bigint AS wal_bytes
```

`api_dashboard_overview` line 258

```sql
SELECT count(*) AS n,
       count(*) FILTER (WHERE NOT active) AS inactive,
       max(pg_wal_lsn_diff(CASE WHEN pg_is_in_recovery()
                                THEN pg_last_wal_replay_lsn()
                                ELSE pg_current_wal_lsn() END,
                           restart_lsn))::bigint AS max_retained_bytes
  FROM pg_replication_slots
```

`api_dashboard_overview` line 270

```sql
SELECT max(age(datfrozenxid)) AS max_age,
       (SELECT datname FROM pg_database ORDER BY age(datfrozenxid) DESC LIMIT 1) AS datname,
       current_setting('autovacuum_freeze_max_age')::bigint AS freeze_max_age
  FROM pg_database
```

`api_dashboard_overview` line 290

```sql
SELECT count(*) AS n FROM pg_settings WHERE pending_restart
```

`api_dashboard_overview` line 295

```sql
SELECT count(*) AS n
  FROM pg_index i
  JOIN pg_class c ON c.oid = i.indexrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE NOT i.indisvalid
   AND n.nspname NOT IN ('pg_catalog', 'information_schema')
```

`api_dashboard_overview` line 305

```sql
SELECT EXTRACT(EPOCH FROM now() - pg_postmaster_start_time())::bigint AS s
```

`api_dashboard_overview` line 343

```sql
SELECT count(*) AS n FROM pg_stat_activity WHERE backend_type = 'autovacuum worker'
```

## api/db.py

`api_db_roles` line 119

```sql
SELECT rolname, rolcanlogin
FROM pg_roles
ORDER BY rolcanlogin DESC, rolname
```

`_detect_special_roles` line 220

```sql
SELECT r.rolname AS name,
       pg_catalog.pg_has_role(current_user, r.oid, 'MEMBER') AS granted
FROM pg_catalog.pg_roles r
WHERE r.rolname = ANY(%s)
```

`_database_sizes` line 259

```sql
SET statement_timeout = 5000
```

`_database_sizes` line 260

```sql
SELECT datname,
       CASE WHEN pg_catalog.has_database_privilege(oid, 'CONNECT')
            THEN pg_catalog.pg_database_size(oid) END
  FROM pg_catalog.pg_database
 WHERE datistemplate = false
```

`_database_sizes` line 272

```sql
RESET statement_timeout
```

`_schema_sizes_estimate` line 291

```sql
SELECT n.nspname,
       (COALESCE(sum(GREATEST(c.relpages, 0)::bigint
                     + COALESCE(GREATEST(t.relpages, 0), 0)::bigint), 0)
        * current_setting('block_size')::bigint)::bigint
  FROM pg_catalog.pg_namespace n
  LEFT JOIN pg_catalog.pg_class c
         ON c.relnamespace = n.oid AND c.relkind IN ('r', 'm', 'i', 'S')
  LEFT JOIN pg_catalog.pg_class t ON t.oid = c.reltoastrelid
 WHERE n.nspname NOT LIKE 'pg_temp%%' AND n.nspname NOT LIKE 'pg_toast%%'
 GROUP BY n.nspname
```

`api_db_schemas` line 335

```sql
SELECT nspname,
       pg_catalog.has_schema_privilege(nspname, 'USAGE') AS usable
  FROM pg_namespace
 WHERE nspname NOT LIKE 'pg_temp%%' AND nspname NOT LIKE 'pg_toast%%'
 ORDER BY nspname
```

`api_db_assumable_roles` line 381

```sql
SELECT r.rolname, r.rolinherit, r.rolsuper, r.rolcanlogin
FROM pg_catalog.pg_roles r
WHERE pg_catalog.pg_has_role(session_user, r.oid, 'MEMBER')
  AND r.rolname NOT LIKE 'pg\_%%'
ORDER BY (r.rolname = session_user) DESC, r.rolname
```

`api_db_ping` line 414

```sql
SELECT current_database(), current_user
```

`_role_privs` line 436

```sql
SELECT rolsuper, rolcreatedb, rolcreaterole, rolreplication, rolcanlogin
FROM pg_roles WHERE rolname = current_user
```

`api_db_acting_role` line 494

```sql
SELECT current_user, session_user, current_database()
```

`_replica_lag_sql` line 527

```sql
SELECT {repl}::text, {recv}::text, {diff}({recv}, {repl}),        pg_last_xact_replay_timestamp(),        extract(epoch FROM now() - pg_last_xact_replay_timestamp())
```

`api_db_databases` line 591

```sql
SELECT datname FROM pg_database WHERE datistemplate = false ORDER BY datname
```

`api_db_databases` line 602

```sql
SELECT current_setting('server_version') AS v, current_setting('server_version_num') AS n
```

`api_db_databases` line 612

```sql
SELECT pg_is_in_recovery()
```

`api_db_databases` line 624

```sql
SELECT current_user, current_database(), session_user
```

`api_db_databases` line 672

```sql
SELECT status, sender_host, sender_port FROM pg_stat_wal_receiver LIMIT 1
```

`api_db_databases` line 684

```sql
SELECT count(*) FROM pg_stat_replication
```

`api_db_sequences_export` line 769

```sql
SELECT current_setting('server_version_num')::int AS n
```

**sql** - `api_db_sequences_export` line 777

```sql
SELECT s.schemaname,
       s.sequencename,
       s.data_type,
       s.start_value,
       s.min_value,
       s.max_value,
       s.increment_by,
       s.cache_size,
       s.cycle,
       s.last_value,
       pg_catalog.pg_get_userbyid(c.relowner) AS owner,
       /* OWNED BY column - pg_depend with auto/internal class for sequence ownership */
       (
         SELECT (n2.nspname || '.' || t.relname || '.' || a.attname)
         FROM pg_depend d
         JOIN pg_class t ON t.oid = d.refobjid
         JOIN pg_namespace n2 ON n2.oid = t.relnamespace
         JOIN pg_attribute a ON a.attrelid = d.refobjid AND a.attnum = d.refobjsubid
         WHERE d.classid = 'pg_class'::regclass
           AND d.objid   = c.oid
           AND d.deptype = 'a'
         LIMIT 1
       ) AS owned_by
FROM pg_sequences s
JOIN pg_class     c  ON c.relname = s.sequencename
JOIN pg_namespace n  ON n.oid = c.relnamespace AND n.nspname = s.schemaname
WHERE s.schemaname NOT IN ('pg_catalog','information_schema')
  AND s.schemaname NOT LIKE 'pg_toast%%'
  AND s.schemaname NOT LIKE 'pg_temp_%%'
```

`api_db_sequences_export` line 840

```sql
CREATE SEQUENCE IF NOT EXISTS {fq}
```

`api_db_sequences_export` line 853

```sql
ALTER SEQUENCE {fq} OWNED BY {q_ident(parts[0])}.{q_ident(parts[1])}.{q_ident(parts[2])};
```

`api_db_sequences_export` line 862

```sql
SELECT setval({lit}::regclass, {str(int(lv))}, true);
```

`api_db_sequences_export` line 866

```sql
SELECT setval({lit}::regclass, {str(int(r['start_value'] or 1))}, false);
```

`api_db_sequences_import_preview` line 958

```sql
SELECT 1 FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s AND a.attname = %s
   AND a.attnum > 0 AND NOT a.attisdropped
```

`api_db_sequences_import_preview` line 966

```sql
SELECT 1 FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
 WHERE c.relname = %s AND a.attname = %s
   AND a.attnum > 0 AND NOT a.attisdropped
```

`api_db_sequences_import` line 1054

```sql
CREATE SEQUENCE IF NOT EXISTS {name}{f' {opts}' if opts else ''}
```

**ddl** - `api_db_create` line 1131

```sql
CREATE DATABASE "{new_dbname}" ENCODING '{encoding}' TEMPLATE "{template}"
```

`api_db_clone` line 1216

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = %s AND pid <> pg_backend_pid()
```

**ddl** - `api_db_clone` line 1220

```sql
CREATE DATABASE "{new_dbname}" TEMPLATE "{src_dbname}"
```

**ddl** - `api_db_drop` line 1261

```sql
DROP DATABASE "{target_dbname}"
```

`api_db_drop` line 1265

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = '{target_dbname}' AND pid <> pg_backend_pid();
```

`api_db_drop` line 1274

```sql
SELECT datistemplate, datname = current_database() AS is_current FROM pg_database WHERE datname = %s
```

`_db_sessions_on` line 1356

```sql
SELECT count(*)::int,   coalesce(array_agg(DISTINCT usename)            FILTER (WHERE usename IS NOT NULL), '{}'),   coalesce(array_agg(DISTINCT application_name)            FILTER (WHERE application_name <> ''), '{}') FROM pg_stat_activity WHERE datname = %s AND pid <> pg_backend_pid()
```

`_terminate_sessions_on` line 1376

```sql
SELECT count(*)::int FROM pg_stat_activity WHERE datname = %s AND pid <> pg_backend_pid()
```

`_terminate_sessions_on` line 1382

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = %s AND pid <> pg_backend_pid()
```

`_terminate_sessions_on` line 1387

```sql
SELECT count(*) FROM pg_stat_activity WHERE datname = %s AND pid <> pg_backend_pid()
```

`api_db_rename` line 1468

```sql
SELECT 1 FROM pg_database WHERE datname = %s
```

`api_db_rename` line 1473

```sql
SELECT 1 FROM pg_database WHERE datname = %s
```

**terminate_sql** - `api_db_rename` line 1494

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = '{old_dbname}' AND pid <> pg_backend_pid();
```

**ddl** - `api_db_rename` line 1496

```sql
ALTER DATABASE "{old_dbname}" RENAME TO "{new_dbname}"
```

`_hba_privs` line 1544

```sql
SELECT current_user AS role, current_setting('is_superuser') = 'on' AS su, current_setting('server_version_num')::int AS pgv
```

`_hba_privs` line 1552

```sql
SELECT pg_has_role(current_user, 'pg_read_all_settings', '{how}') AS settings, has_table_privilege('pg_catalog.pg_hba_file_rules', 'SELECT') AS rules_select, has_function_privilege('pg_catalog.pg_hba_file_rules()', 'EXECUTE') AS rules_exec
```

`_hba_privs` line 1558

```sql
SELECT has_function_privilege('pg_catalog.pg_read_file(text)', 'EXECUTE') AS read_file, pg_has_role(current_user, 'pg_read_server_files', '{how}') AS server_files
```

`_hba_privs` line 1562

```sql
SELECT current_setting('hba_file') AS hba, current_setting('data_directory') AS dd
```

`_hba_rules_needs` line 1610

```sql
SELECT on pg_hba_file_rules
```

`api_db_hba` line 1637

```sql
SELECT row_number() OVER (ORDER BY line_number) AS rb,
       line_number AS line,
       type, database, user_name, address, netmask, auth_method,
       options, error
FROM pg_hba_file_rules
ORDER BY line_number
```

`api_db_hba_get` line 1735

```sql
SELECT current_setting('hba_file') AS hba, current_setting('is_superuser') AS issu
```

`api_db_hba_get` line 1741

```sql
SELECT pg_read_file(%s) AS content
```

`_hba_restrict_mode` line 1792

```sql
COPY (SELECT 1) TO PROGRAM %s
```

`api_db_hba_save` line 1825

```sql
SELECT current_setting('hba_file') AS hba, current_setting('is_superuser') AS issu
```

`api_db_hba_save` line 1834

```sql
SELECT pg_read_file(%s) AS content
```

`api_db_hba_save` line 1845

```sql
SELECT pg_reload_conf()
```

`api_db_hba_save` line 1846

```sql
SELECT line_number AS line, type, database, user_name, address, auth_method, error
FROM pg_hba_file_rules
WHERE error IS NOT NULL
ORDER BY line_number
```

`api_db_hba_restore` line 1894

```sql
SELECT current_setting('hba_file') AS hba, current_setting('is_superuser') AS issu
```

`api_db_hba_restore` line 1913

```sql
SELECT pg_read_file(%s) AS content
```

`api_db_hba_restore` line 1919

```sql
SELECT pg_read_file(%s) AS content
```

`api_db_hba_restore` line 1924

```sql
SELECT pg_reload_conf()
```

`api_db_hba_restore` line 1925

```sql
SELECT line_number AS line, error FROM pg_hba_file_rules
WHERE error IS NOT NULL ORDER BY line_number
```

`api_db_settings` line 1958

```sql
SELECT name, setting, unit, category, short_desc,
       context, vartype, source,
       boot_val, reset_val, min_val, max_val, enumvals,
       {pending_col}, sourcefile, sourceline
FROM pg_settings
ORDER BY category, name
```

`api_db_settings` line 1968

```sql
SELECT current_setting('is_superuser') AS issu
```

`api_settings_undo` line 2031

```sql
SELECT setting, source, sourcefile, sourceline FROM pg_settings WHERE name = %s
```

`api_settings_undo` line 2052

```sql
SELECT pg_reload_conf()
```

`api_settings_undo` line 2061

```sql
SELECT setting, source, pending_restart FROM pg_settings WHERE name = %s
```

`api_restore_target_check` line 2084

```sql
SELECT count(*) FILTER (WHERE c.relkind IN ('r','p')),
       count(*) FILTER (WHERE c.relkind IN ('v','m')),
       count(*) FILTER (WHERE c.relkind IN ('S')),
       count(DISTINCT n.nspname)
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname NOT IN ('pg_catalog', 'information_schema', 'pg_toast')
   AND n.nspname NOT LIKE 'pg_temp%%'
```

`api_restore_target_check` line 2094

```sql
SELECT pg_database_size(current_database()), pg_size_pretty(pg_database_size(current_database()))
```

`api_db_settings_set` line 2117

```sql
ALTER SYSTEM SET / RESET / RESET ALL, optionally followed by pg_reload_conf().
    Writes go to postgresql.auto.conf - never edit user-managed config files. Requires
    superuser (or a role that has been granted SET on the parameter, PG15+).
```

`api_db_settings_set` line 2151

```sql
SELECT pg_reload_conf()
```

`api_db_settings_set` line 2160

```sql
SELECT pg_reload_conf()
```

`api_db_settings_set` line 2181

```sql
SELECT context, vartype, {pending_col} FROM pg_settings WHERE name = %s
```

`api_db_settings_set` line 2191

```sql
SELECT setting, source, sourcefile, sourceline FROM pg_settings WHERE name = %s
```

`api_db_settings_set` line 2224

```sql
SELECT setting FROM pg_file_settings WHERE name = %s ORDER BY seqno DESC LIMIT 1
```

`api_db_settings_set` line 2231

```sql
SELECT current_setting(%s)
```

`api_db_settings_set` line 2268

```sql
SELECT pg_reload_conf()
```

`api_db_settings_set` line 2272

```sql
SELECT setting, source, sourcefile, sourceline, pending_restart, context
FROM pg_settings WHERE name = %s
```

`api_db_analysis_prereqs` line 2375

```sql
SELECT default_version, installed_version FROM pg_available_extensions WHERE name = 'pg_stat_statements'
```

`api_db_analysis_prereqs` line 2390

```sql
SELECT default_version, installed_version FROM pg_available_extensions WHERE name = 'pg_store_plans'
```

`api_db_analysis_prereqs` line 2404

```sql
SELECT setting FROM pg_file_settings WHERE name = 'shared_preload_libraries' ORDER BY seqno DESC LIMIT 1
```

**recommended** - `api_db_analysis_prereqs` line 2419

```sql
CREATE EXTENSION
```

**sql** - `api_db_analysis_prereqs` line 2424

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

`api_db_plan_log_grant` line 2463

```sql
SELECT 1 FROM pg_roles WHERE rolname = %s
```

`api_db_capabilities` line 2522

```sql
SELECT current_database(), current_user, version()
```

`api_db_capabilities` line 2533

```sql
SELECT rolsuper,
       pg_has_role(current_user, 'pg_monitor', 'member')        AS is_monitor,
       pg_has_role(current_user, 'pg_read_all_stats', 'member') AS reads_stats
FROM pg_roles WHERE rolname = current_user
```

`api_db_capabilities` line 2540

```sql
SELECT rolsuper, false AS is_monitor, false AS reads_stats
FROM pg_roles WHERE rolname = current_user
```

`api_db_capabilities` line 2548

```sql
SELECT extname FROM pg_extension
```

`api_db_capabilities` line 2550

```sql
SELECT name FROM pg_available_extensions
```

`api_db_capabilities` line 2560

```sql
SHOW shared_preload_libraries
```

`api_db_capabilities` line 2571

```sql
SELECT setting FROM pg_settings WHERE name = %s
```

`api_db_capabilities` line 2624

```sql
ALTER SYSTEM SET shared_preload_libraries = '{preload_raw + ', ' if preload_raw else ''}{n}';  -- then restart the server
```

`api_db_capabilities` line 2633

```sql
CREATE EXTENSION {n};{'  -- also add to shared_preload_libraries and restart' if e['needs_preload'] else ''}
```

`api_db_capabilities` line 2658

```sql
ALTER SYSTEM SET {s['name']} = '{want_txt}'; SELECT pg_reload_conf();
```

`api_db_capabilities` line 2674

```sql
GRANT pg_monitor TO {user};
```

`_db_compare_props` line 2763

```sql
SELECT pg_catalog.pg_get_userbyid(d.datdba) AS owner,
       pg_catalog.pg_encoding_to_char(d.encoding) AS encoding,
       d.datcollate AS collate, d.datctype AS ctype,
       COALESCE(t.spcname, 'pg_default') AS tablespace,
       d.datconnlimit AS conn_limit,
       d.datallowconn AS allow_conn
FROM pg_catalog.pg_database d
LEFT JOIN pg_catalog.pg_tablespace t ON t.oid = d.dattablespace
WHERE d.datname = %s
```

`_db_compare_props` line 2826

```sql
SELECT COALESCE(array_to_string(setconfig, E'\n'), '')
FROM pg_catalog.pg_db_role_setting s
JOIN pg_catalog.pg_database d ON d.oid = s.setdatabase
WHERE d.datname = %s AND s.setrole = 0
```

`_db_compare_extensions` line 2847

```sql
SELECT e.extname, e.extversion, n.nspname
FROM pg_catalog.pg_extension e
LEFT JOIN pg_catalog.pg_namespace n ON n.oid = e.extnamespace
ORDER BY e.extname
```

`_db_compare_roles` line 2926

```sql
SELECT r.rolname, r.rolsuper, r.rolinherit, r.rolcreatedb,
       r.rolcreaterole, r.rolcanlogin, r.rolreplication,
       {bypass_col} AS rolbypassrls, r.rolconnlimit,
       COALESCE(to_char(r.rolvaliduntil, 'YYYY-MM-DD HH24:MI:SSOF'), '') AS valid_until,
       COALESCE((SELECT array_to_string(array_agg(m.rolname ORDER BY m.rolname), ',')
                 FROM pg_catalog.pg_auth_members am
                 JOIN pg_catalog.pg_roles m ON m.oid = am.roleid
                 WHERE am.member = r.oid), '') AS member_of
FROM pg_catalog.pg_roles r
WHERE r.rolname NOT LIKE 'pg\_%'
ORDER BY r.rolname
```

`_role_sync_sql` line 2978

```sql
CREATE ROLE {q} {attrs(ra)};
```

`_role_sync_sql` line 2985

```sql
ALTER ROLE {q} {attrs(ra)};
```

`_role_sync_sql` line 2997

```sql
GRANT "{grp.replace('"', '""')}" TO {q};
```

`_role_sync_sql` line 2999

```sql
REVOKE "{grp.replace('"', '""')}" FROM {q};
```

`_db_compare_publications` line 3025

```sql
SELECT p.pubname, pg_catalog.pg_get_userbyid(p.pubowner) AS owner,
       p.puballtables, p.pubinsert, p.pubupdate, p.pubdelete,
       %s AS pubtruncate,
       (SELECT COALESCE(string_agg(pt.schemaname || '.' || pt.tablename, ', '
                                    ORDER BY pt.schemaname, pt.tablename), '')
          FROM pg_catalog.pg_publication_tables pt WHERE pt.pubname = p.pubname) AS tables
  FROM pg_catalog.pg_publication p
 ORDER BY p.pubname
```

`_db_compare_subscriptions` line 3049

```sql
SELECT s.subname, pg_catalog.pg_get_userbyid(s.subowner) AS owner,
       s.subenabled, s.subslotname,
       array_to_string(s.subpublications, ', ') AS publications
  FROM pg_catalog.pg_subscription s
 WHERE s.subdbid = (SELECT oid FROM pg_catalog.pg_database WHERE datname = current_database())
 ORDER BY s.subname
```

`_db_compare_servers` line 3065

```sql
SELECT s.srvname, w.fdwname, pg_catalog.pg_get_userbyid(s.srvowner) AS owner,
       s.srvoptions,
       (SELECT count(*) FROM pg_catalog.pg_user_mappings um WHERE um.srvid = s.oid) AS mappings
  FROM pg_catalog.pg_foreign_server s
  JOIN pg_catalog.pg_foreign_data_wrapper w ON w.oid = s.srvfdw
 ORDER BY s.srvname
```

`_db_compare_event_triggers` line 3088

```sql
SELECT e.evtname, e.evtevent, e.evtfoid::regproc::text AS fn,
       e.evtenabled::text, array_to_string(e.evttags, ', ') AS tags,
       pg_catalog.pg_get_userbyid(e.evtowner) AS owner
  FROM pg_catalog.pg_event_trigger e
 ORDER BY e.evtname
```

`_db_compare_casts` line 3115

```sql
SELECT pg_catalog.format_type(c.castsource, NULL) AS source,
       pg_catalog.format_type(c.casttarget, NULL) AS target,
       c.castmethod::text AS method, c.castcontext::text AS context,
       CASE WHEN c.castfunc <> 0 THEN c.castfunc::pg_catalog.regprocedure::text END AS func
  FROM pg_catalog.pg_cast c
 WHERE c.oid >= 16384
   AND NOT EXISTS (SELECT 1 FROM pg_catalog.pg_depend d
                    WHERE d.classid = 'pg_catalog.pg_cast'::pg_catalog.regclass
                      AND d.objid = c.oid AND d.deptype = 'e')
 ORDER BY 1, 2
```

`_cast_sync_sql.create` line 3149

```sql
CREATE CAST ({v['source']} AS {v['target']}) {how}{ctx};
```

**drop** - `_cast_sync_sql` line 3151

```sql
DROP CAST IF EXISTS ({v['source']} AS {v['target']});
```

`api_db_compare.collect` line 3247

```sql
SELECT current_setting('server_version'), current_setting('server_version_num')::int
```

`api_db_compare_sync` line 3522

```sql
SELECT rolname FROM pg_catalog.pg_roles
```

`api_db_compare_sync` line 3524

```sql
SELECT current_setting('server_version_num')::int
```

`api_db_compare_sync` line 3597

```sql
CREATE SCHEMA IF NOT EXISTS "{name.replace('"', '""')}";
```

**sql** - `api_db_compare_sync` line 3599

```sql
CREATE SCHEMA IF NOT EXISTS "{name.replace('"', '""')}"
```

`_role_object_grants` line 3724

```sql
SELECT oid FROM pg_catalog.pg_roles WHERE rolname = %s
```

`_role_object_grants` line 3762

```sql
SELECT r.rolname, CASE WHEN am.admin_option THEN 'ADMIN' ELSE 'MEMBER' END
                       FROM pg_catalog.pg_auth_members am
                       JOIN pg_catalog.pg_roles r ON r.oid = am.roleid
                      WHERE am.member = %s
```

`_role_object_grants` line 3769

```sql
SELECT datdba FROM pg_catalog.pg_database WHERE datname = current_database()
```

`_grant_stmt` line 3867

```sql
ALTER {target} OWNER TO {who};
```

`_grant_stmt` line 3880

```sql
REVOKE {priv} ({q(col)}) ON {target} FROM {who};
```

`_grant_stmt` line 3882

```sql
GRANT {priv} ({q(col)}) ON {target} TO {who};
```

**head** - `_grant_stmt` line 3890

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE {q(owner)}{where}
```

`_grant_stmt` line 3891

```sql
REVOKE {priv} ON {objtype} FROM
```

`_grant_stmt` line 3892

```sql
GRANT {priv} ON {objtype} TO
```

`_grant_stmt` line 3910

```sql
GRANT {q(name)} TO {who}{' WITH ADMIN OPTION' if priv == 'ADMIN' else ''};
```

`_grant_stmt` line 3930

```sql
REVOKE {priv} ON {target} FROM {who};
```

`_grant_stmt` line 3931

```sql
GRANT {priv} ON {target} TO {who};
```

`api_role_privileges_compare._exists` line 4000

```sql
SELECT 1 FROM pg_roles WHERE rolname = %s
```

`api_role_privileges_compare_sync` line 4105

```sql
SELECT 1 FROM pg_roles WHERE rolname = %s
```

`api_db_compare_role_privs_apply` line 4247

```sql
SELECT 1 FROM pg_catalog.pg_roles WHERE rolname = %s
```

`api_db_compare_role_privs_apply._exists_on_target` line 4268

```sql
SELECT 1 FROM pg_catalog.pg_namespace WHERE nspname = %s
```

`api_db_compare_role_privs_apply._exists_on_target` line 4273

```sql
SELECT to_regprocedure(%s) IS NOT NULL
```

`api_db_compare_role_privs_apply._exists_on_target` line 4276

```sql
SELECT to_regtype(%s) IS NOT NULL
```

`api_db_compare_role_privs_apply._exists_on_target` line 4280

```sql
SELECT 1 FROM pg_catalog.pg_attribute
                                      WHERE attrelid = to_regclass(%s) AND attname = %s
                                        AND attnum > 0 AND NOT attisdropped
```

`api_db_compare_role_privs_apply._exists_on_target` line 4285

```sql
SELECT 1 FROM pg_catalog.pg_language WHERE lanname = %s
```

`api_db_compare_role_privs_apply._exists_on_target` line 4287

```sql
SELECT 1 FROM pg_catalog.pg_foreign_data_wrapper WHERE fdwname = %s
```

`api_db_compare_role_privs_apply._exists_on_target` line 4289

```sql
SELECT 1 FROM pg_catalog.pg_foreign_server WHERE srvname = %s
```

`api_db_compare_role_privs_apply._exists_on_target` line 4291

```sql
SELECT 1 FROM pg_catalog.pg_largeobject_metadata WHERE oid = %s::oid
```

`api_db_compare_role_privs_apply._exists_on_target` line 4294

```sql
SELECT to_regclass(%s) IS NOT NULL
```

## api/exts.py

`api_exts_inventory` line 37

```sql
SELECT a.name,
       a.default_version,
       a.installed_version,
       a.comment,
       COALESCE(n.nspname, '')        AS schema,
       COALESCE(e.extrelocatable, false) AS relocatable,
       COALESCE(e.extconfig::text[], ARRAY[]::text[]) AS config_tables
  FROM pg_available_extensions a
  LEFT JOIN pg_extension e ON e.extname = a.name
  LEFT JOIN pg_namespace n ON n.oid = e.extnamespace
 ORDER BY a.name
```

`api_exts_inventory` line 58

```sql
SELECT name, version, superuser, {_trusted}, requires
  FROM pg_available_extension_versions
 ORDER BY name, version
```

**sql** - `api_exts_install` line 107

```sql
CREATE EXTENSION
```

`api_exts_install` line 126

```sql
SELECT 1 FROM pg_available_extensions WHERE name = %s
```

**sql** - `api_exts_update` line 162

```sql
ALTER EXTENSION {_ident(name)} UPDATE
```

**sql** - `api_exts_drop` line 187

```sql
DROP EXTENSION IF EXISTS {_ident(name)}
```

**sql** - `api_exts_set_schema` line 213

```sql
ALTER EXTENSION {_ident(name)} SET SCHEMA {_ident(schema)}
```

## api/fdw.py

`api_fdw_catalog` line 33

```sql
SELECT f.fdwname, pg_get_userbyid(f.fdwowner),
       COALESCE(h.proname::text, ''),
       COALESCE((SELECT extname FROM pg_extension e
                  JOIN pg_depend d ON d.refobjid = e.oid
                  WHERE d.objid = f.oid AND d.deptype = 'e'
                  LIMIT 1), '')
  FROM pg_foreign_data_wrapper f
  LEFT JOIN pg_proc h ON h.oid = f.fdwhandler
 ORDER BY f.fdwname
```

`api_fdw_catalog` line 48

```sql
SELECT name FROM pg_available_extensions
 WHERE name = ANY(%s)
```

`api_fdw_inventory` line 84

```sql
SELECT s.srvname,
       pg_get_userbyid(s.srvowner),
       (SELECT fdwname FROM pg_foreign_data_wrapper f WHERE f.oid = s.srvfdw),
       COALESCE(s.srvoptions, ARRAY[]::text[])
  FROM pg_foreign_server s
 ORDER BY s.srvname
```

`api_fdw_inventory` line 105

```sql
SELECT srv.srvname,
       CASE WHEN um.umuser = 0 THEN 'PUBLIC' ELSE pg_get_userbyid(um.umuser) END,
       COALESCE(um.umoptions, ARRAY[]::text[])
  FROM pg_user_mappings um
  JOIN pg_foreign_server srv ON srv.oid = um.srvid
 ORDER BY srv.srvname, um.umuser
```

`api_fdw_create_server` line 232

```sql
CREATE EXTENSION IF NOT EXISTS {_ident(ext)}
```

**sql** - `api_fdw_create_server` line 236

```sql
CREATE SERVER {_ident(srv_name)} FOREIGN DATA WRAPPER {_ident(fdw)}
```

**head** - `api_fdw_create_server` line 251

```sql
CREATE USER MAPPING FOR {whom} SERVER {_ident(srv_name)}
```

**head** - `api_fdw_create_user_mapping` line 304

```sql
CREATE USER MAPPING FOR {whom} SERVER {_ident(srv_name)}
```

`_srv_option_keys` line 334

```sql
SELECT COALESCE(srvoptions, ARRAY[]::text[])
  FROM pg_foreign_server WHERE srvname = %s
```

`api_fdw_alter_server` line 423

```sql
ALTER SERVER {srv} OPTIONS ({', '.join(clauses)})
```

`api_fdw_alter_server` line 424

```sql
ALTER SERVER {srv} OPTIONS ({', '.join(shown)})
```

**st** - `api_fdw_alter_server` line 426

```sql
ALTER SERVER {srv} OWNER TO {_ident(owner)}
```

**sql** - `api_fdw_drop_server` line 467

```sql
DROP SERVER IF EXISTS {_ident(srv_name)}
```

`api_fdw_alter_user_mapping` line 515

```sql
SELECT COALESCE(um.umoptions, ARRAY[]::text[])
  FROM pg_user_mappings um
  JOIN pg_foreign_server s ON s.oid = um.srvid
 WHERE s.srvname = %s
   AND CASE WHEN um.umuser = 0 THEN 'PUBLIC' ELSE pg_get_userbyid(um.umuser) END = %s
```

**sql** - `api_fdw_alter_user_mapping` line 532

```sql
ALTER USER MAPPING FOR {whom} SERVER {srv} OPTIONS ({', '.join(clauses)})
```

**shown_sql** - `api_fdw_alter_user_mapping` line 533

```sql
ALTER USER MAPPING FOR {whom} SERVER {srv} OPTIONS ({', '.join(shown)})
```

**sql** - `api_fdw_drop_user_mapping` line 567

```sql
DROP USER MAPPING IF EXISTS FOR {whom} SERVER {_ident(srv_name)}
```

`api_fdw_drop_user_mapping` line 582

```sql
SELECT 1 FROM pg_user_mappings um
  JOIN pg_foreign_server srv ON srv.oid = um.srvid
 WHERE srv.srvname = %s
   AND CASE WHEN um.umuser = 0 THEN 'PUBLIC'
            ELSE pg_get_userbyid(um.umuser) END = %s
```

`api_fdw_import_schema` line 605

```sql
IMPORT FOREIGN SCHEMA %s
```

**sql** - `api_fdw_import_schema` line 637

```sql
IMPORT FOREIGN SCHEMA {_ident(remote_sch)}
```

`api_fdw_import_schema` line 649

```sql
SELECT count(*) FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind = 'f'
```

## api/maint.py

`api_maint_progress` line 124

```sql
CREATE INDEX
```

`api_maint_progress` line 129

```sql
SELECT row_to_json(p)::text AS j FROM {view} p WHERE pid = %s
```

`api_maint_table_ops` line 167

```sql
SELECT c.oid::int AS oid FROM pg_class c JOIN pg_namespace n ON n.oid=c.relnamespace WHERE n.nspname=%s AND c.relname=%s
```

`api_maint_table_ops` line 179

```sql
CREATE INDEX
```

`api_pg_repack_info` line 238

```sql
SELECT extversion FROM pg_extension WHERE extname = 'pg_repack'
```

## api/object.py

`api_object_details` line 163

```sql
SELECT rolname FROM pg_roles WHERE rolname !~ '^pg_' ORDER BY rolname
```

`api_object_details` line 165

```sql
SELECT spcname FROM pg_tablespace ORDER BY spcname
```

`api_object_details` line 168

```sql
SELECT pg_get_userbyid(n.nspowner) AS owner,
       obj_description(n.oid, 'pg_namespace') AS comment,
       (SELECT count(*) FROM pg_class c
          WHERE c.relnamespace = n.oid AND c.relkind IN ('r','p'))::int AS n_tables
FROM pg_namespace n WHERE n.nspname = %s
```

`api_object_details` line 181

```sql
SELECT pg_get_userbyid(c.relowner) AS owner,
       obj_description(c.oid, 'pg_class') AS comment,
       COALESCE(t.spcname,
           (SELECT spcname FROM pg_tablespace
            WHERE oid = (SELECT dattablespace FROM pg_database
                         WHERE datname = current_database()))) AS tablespace,
       pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,
       CASE c.relpersistence WHEN 'u' THEN 'unlogged'
            WHEN 't' THEN 'temporary' ELSE 'permanent' END AS persistence
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
LEFT JOIN pg_tablespace t ON t.oid = c.reltablespace
WHERE n.nspname = %s AND c.relname = %s
```

`api_object_tablespace` line 225

```sql
ALTER TABLE {} SET TABLESPACE {}
```

`api_object_comment` line 260

```sql
COMMENT ON {kw} {} IS NULL
```

`api_object_comment` line 262

```sql
COMMENT ON {kw} {} IS {}
```

**sql** - `api_object_move` line 307

```sql
ALTER {verb} {ref} SET SCHEMA {_qi(target)};
```

`api_object_drop_multi` line 385

```sql
DROP {kw} IF EXISTS {ref}{tail}
```

`api_object_move_multi` line 457

```sql
SELECT 1 FROM pg_namespace WHERE nspname = %s
```

`api_object_move_multi` line 488

```sql
SELECT 1 FROM pg_class c JOIN pg_namespace n
                                         ON n.oid = c.relnamespace
                                        WHERE n.nspname = %s AND c.relname = %s
```

`api_object_move_multi` line 494

```sql
SELECT 1 FROM pg_proc p JOIN pg_namespace n
                                         ON n.oid = p.pronamespace
                                        WHERE n.nspname = %s AND p.proname = %s
                                          AND pg_get_function_identity_arguments(p.oid) = %s
```

`api_object_move_multi` line 501

```sql
SELECT 1 FROM pg_type t JOIN pg_namespace n
                                         ON n.oid = t.typnamespace
                                        WHERE n.nspname = %s AND t.typname = %s
```

`api_object_move_multi` line 513

```sql
ALTER {verb} {ref} SET SCHEMA {_qi(target)};
```

`api_object_clone_estimate` line 628

```sql
SELECT c.oid FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s AND c.relkind IN ('r','p')
```

`api_object_clone_estimate` line 648

```sql
SELECT DISTINCT k.confrelid
  FROM pg_constraint k JOIN pg_class rc ON rc.oid = k.confrelid
 WHERE k.conrelid = %s AND k.contype = 'f'
   AND k.confrelid <> k.conrelid AND rc.relkind IN ('r','p')
```

`api_object_clone_estimate` line 668

```sql
SELECT n.nspname || '.' || c.relname AS fq,
       pg_total_relation_size(c.oid) AS total_bytes,
       pg_relation_size(c.oid)        AS heap_bytes,
       pg_indexes_size(c.oid)         AS indexes_bytes,
       -- -1 means never analysed (PG14+), not empty: a 50,000-row
       -- table that was never analysed reported "-1 rows". NULL is
       -- the honest answer, and the UI already renders it as unknown.
       -- Same trap as the schema tree: n_live_tup is 0, not
       -- NULL, after a stats reset, so a table with rows can
       -- read as empty. Take the better of the two estimates
       -- and let the file size say whether zero means empty.
       GREATEST(NULLIF(s.n_live_tup, 0),
                NULLIF(c.reltuples, -1)::bigint) AS row_estimate,
       CASE WHEN coalesce(GREATEST(NULLIF(s.n_live_tup, 0),
                                   NULLIF(c.reltuples, -1)::bigint), 0) = 0
            THEN pg_relation_size(c.oid) > 0
            ELSE false
       END AS rows_unknown,
       (SELECT count(*) FROM pg_index i WHERE i.indrelid = c.oid)
          AS index_count,
       (SELECT count(*) FROM pg_trigger t WHERE t.tgrelid = c.oid AND NOT t.tgisinternal)
          AS trigger_count,
       (SELECT count(*) FROM pg_constraint k WHERE k.conrelid = c.oid AND k.contype = 'f')
          AS fk_count
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_stat_user_tables s ON s.relid = c.oid
 WHERE c.oid = %s
```

`api_object_privileges` line 945

```sql
SELECT p.oid, pg_get_userbyid(p.proowner) AS owner, p.proacl::text AS acl_text
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname = %s AND p.proname = %s
  AND pg_get_function_identity_arguments(p.oid) = %s
```

`api_object_privileges` line 955

```sql
SELECT CASE WHEN a.grantee = 0 THEN 'PUBLIC'
            ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)') END AS grantee,
       CASE WHEN a.grantor = 0 THEN 'PUBLIC'
            ELSE COALESCE(pg_get_userbyid(a.grantor), '(deleted)') END AS granted_by,
       a.privilege_type AS privilege,
       a.is_grantable AS with_grant_option
FROM aclexplode((SELECT proacl FROM pg_proc WHERE oid = %s)) a
ORDER BY grantee, privilege
```

`api_object_privileges` line 984

```sql
SELECT x.oid::oid AS oid,
       pg_get_userbyid(x.{spec['owner']}) AS owner,
       x.{spec['acl']} AS acl
FROM {spec['catalog']} x
JOIN pg_namespace n ON n.oid = x.{spec['nsp']}
WHERE n.nspname = %s AND x.{spec['name']} = %s
```

`api_object_privileges` line 993

```sql
SELECT x.oid::oid AS oid,
       pg_get_userbyid(x.{spec['owner']}) AS owner,
       x.{spec['acl']} AS acl
FROM {spec['catalog']} x
WHERE x.{spec['name']} = %s
```

`api_object_privileges` line 1005

```sql
SELECT CASE WHEN a.grantee = 0 THEN 'PUBLIC'
            ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted role)') END AS grantee,
       CASE WHEN a.grantor = 0 THEN 'PUBLIC'
            ELSE COALESCE(pg_get_userbyid(a.grantor), '(deleted)') END AS granted_by,
       a.privilege_type AS privilege,
       a.is_grantable AS with_grant_option
FROM aclexplode((SELECT {spec['acl']} FROM {spec['catalog']} WHERE oid = %s)) a
ORDER BY grantee, privilege
```

`api_object_privileges` line 1023

```sql
SELECT rolname,
       (SELECT array_agg(g.rolname)
          FROM pg_auth_members am
          JOIN pg_roles g ON g.oid = am.roleid
         WHERE am.member = r.oid) AS member_of
  FROM pg_roles r
 WHERE rolname IN %s
```

**stmt** - `api_object_privileges_apply` line 1112

```sql
GRANT {priv_list} ON {target_str} TO {grantee_sql}{tail}
```

**stmt** - `api_object_privileges_apply` line 1114

```sql
REVOKE {priv_list} ON {target_str} FROM {grantee_sql}
```

`api_object_column_privileges` line 1159

```sql
SELECT a.attname AS col, a.attacl::text AS acl_text, a.attacl AS acl_raw
  FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
   AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

`api_object_column_privileges` line 1173

```sql
SELECT CASE WHEN a.grantee = 0 THEN 'PUBLIC'
            ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted)') END AS grantee,
       a.privilege_type AS privilege,
       a.is_grantable AS with_grant_option
  FROM aclexplode(%s::aclitem[]) a
 ORDER BY grantee, privilege
```

**stmt** - `api_object_column_privileges_apply` line 1233

```sql
GRANT {priv} ({col_q}) ON TABLE {sch_q} TO {role_q}{' WITH GRANT OPTION' if wgo else ''}
```

**stmt** - `api_object_column_privileges_apply` line 1235

```sql
REVOKE {'GRANT OPTION FOR ' if wgo else ''}{priv} ({col_q}) ON TABLE {sch_q} FROM {role_q}
```

`api_object_privileges_copy` line 1335

```sql
SELECT attname FROM pg_catalog.pg_attribute WHERE attrelid = %s AND attnum > 0 AND NOT attisdropped
```

`api_object_function_overloads` line 1376

```sql
SELECT pg_get_function_identity_arguments(p.oid) AS args,
       {_kind_col} AS kind,
       pg_get_function_result(p.oid) AS return_type,
       l.lanname AS language,
       pg_get_userbyid(p.proowner) AS owner,
       p.pronargs AS n_args
  FROM pg_proc p
  JOIN pg_namespace n ON n.oid = p.pronamespace
  JOIN pg_language l ON l.oid = p.prolang
 WHERE n.nspname = %s AND p.proname = %s
 ORDER BY p.pronargs, args
```

**stmt** - `api_object_privileges_bulk_apply` line 1446

```sql
GRANT {priv_list} ON {target_sql} TO {ge_q}{' WITH GRANT OPTION' if wgo else ''}
```

**stmt** - `api_object_privileges_bulk_apply` line 1448

```sql
REVOKE {'GRANT OPTION FOR ' if wgo else ''}{priv_list} ON {target_sql} FROM {ge_q}
```

## api/partitions.py

`api_partitions_list` line 103

```sql
SELECT c.oid                          AS oid,
       p.partstrat                    AS strategy,
       pg_get_partkeydef(c.oid)       AS partkey,
       c.relispartition               AS is_itself_a_partition
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_partitioned_table p ON p.partrelid = c.oid
 WHERE n.nspname = %s AND c.relname = %s
```

`api_partitions_list` line 122

```sql
SELECT cn.nspname                              AS schema,
       cc.relname                              AS name,
       pg_get_expr(cc.relpartbound, cc.oid)    AS bounds,
       cc.relkind                              AS relkind,
       cc.reltuples::bigint                    AS approx_rows,
       pg_total_relation_size(cc.oid)          AS total_bytes,
       pg_size_pretty(pg_total_relation_size(cc.oid)) AS total,
       COALESCE(ts.spcname, '')                AS tablespace,
       {_detach_col}                           AS detach_pending
  FROM pg_inherits i
  JOIN pg_class cc     ON cc.oid = i.inhrelid
  JOIN pg_namespace cn ON cn.oid = cc.relnamespace
  LEFT JOIN pg_tablespace ts ON ts.oid = cc.reltablespace
 WHERE i.inhparent = %s
 ORDER BY cn.nspname, cc.relname
```

`api_partitions_recipe` line 236

```sql
SET search_path = pg_catalog
```

`api_partitions_recipe` line 238

```sql
SELECT c.oid, c.relkind, c.relpersistence, c.relispartition,
       c.relrowsecurity, c.relforcerowsecurity, c.relreplident,
       c.reloptions, c.reltype,
       pg_get_userbyid(c.relowner) AS owner,
       obj_description(c.oid, 'pg_class') AS comment,
       quote_ident(n.nspname) || '.' || quote_ident(c.relname) AS qname,
       pg_size_pretty(pg_total_relation_size(c.oid)) AS size,
       c.reltuples::bigint AS approx_rows,
       EXISTS (SELECT 1 FROM pg_inherits i WHERE i.inhrelid = c.oid) AS has_parent,
       EXISTS (SELECT 1 FROM pg_inherits i WHERE i.inhparent = c.oid) AS has_children
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
```

`api_partitions_recipe` line 281

```sql
SELECT a.attname, a.attnum,
       format_type(a.atttypid, a.atttypmod) AS type,
       a.attidentity AS identity, {gen_col} AS generated,
       pg_get_serial_sequence(%s, a.attname) AS seq,
       CASE WHEN a.attstattarget > 0 THEN a.attstattarget END AS stattarget,
       col_description(a.attrelid, a.attnum) AS comment
  FROM pg_attribute a
 WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

`api_partitions_recipe` line 294

```sql
SELECT CASE WHEN x.grantee = 0 THEN 'PUBLIC'
            ELSE quote_ident(pg_get_userbyid(x.grantee)) END AS grantee,
       x.privilege_type, x.is_grantable
  FROM pg_class c, aclexplode(c.relacl) x
 WHERE c.oid = %s AND x.grantee <> c.relowner
 ORDER BY 1, 2
```

`api_partitions_recipe` line 303

```sql
SELECT a.attname,
       CASE WHEN x.grantee = 0 THEN 'PUBLIC'
            ELSE quote_ident(pg_get_userbyid(x.grantee)) END AS grantee,
       x.privilege_type, x.is_grantable
  FROM pg_attribute a, aclexplode(a.attacl) x
 WHERE a.attrelid = %s AND a.attnum > 0 AND NOT a.attisdropped
   AND a.attacl IS NOT NULL
 ORDER BY a.attnum, 2, 3
```

`api_partitions_recipe` line 317

```sql
SELECT c.conname, pg_get_constraintdef(c.oid) AS def, c.contype,
       obj_description(c.oid, 'pg_constraint') AS comment
  FROM pg_constraint c
 WHERE c.conrelid = %s AND c.contype IN ('p','u','x')
 ORDER BY (c.contype = 'p') DESC, c.conname
```

`api_partitions_recipe` line 326

```sql
SELECT conname, pg_get_constraintdef(oid) AS def, convalidated
  FROM pg_constraint WHERE conrelid = %s AND contype = 'f'
 ORDER BY conname
```

`api_partitions_recipe` line 337

```sql
SELECT conname, pg_get_constraintdef(oid) AS def, convalidated, connoinherit,
       obj_description(oid, 'pg_constraint') AS comment
  FROM pg_constraint WHERE conrelid = %s AND contype = 'c'
 ORDER BY conname
```

`api_partitions_recipe` line 349

```sql
SELECT quote_ident(n.nspname) || '.' || quote_ident(cl.relname) AS qtable,
       c.conname, pg_get_constraintdef(c.oid) AS def
  FROM pg_constraint c
  JOIN pg_class cl ON cl.oid = c.conrelid
  JOIN pg_namespace n ON n.oid = cl.relnamespace
 WHERE c.confrelid = %s AND c.contype = 'f' {top_only}
 ORDER BY n.nspname, cl.relname, c.conname
```

`api_partitions_recipe` line 362

```sql
SELECT i.indexrelid, ic.relname AS name,
       quote_ident(n.nspname) || '.' || quote_ident(ic.relname) AS qname,
       pg_get_indexdef(i.indexrelid) AS def,
       EXISTS (SELECT 1 FROM pg_constraint k
                WHERE k.conindid = i.indexrelid AND k.conrelid = i.indrelid
                  AND k.contype IN ('p','u','x')) AS backs_constraint,
       obj_description(i.indexrelid, 'pg_class') AS comment
  FROM pg_index i
  JOIN pg_class ic ON ic.oid = i.indexrelid
  JOIN pg_namespace n ON n.oid = ic.relnamespace
 WHERE i.indrelid = %s
 ORDER BY ic.relname
```

`api_partitions_recipe` line 379

```sql
SELECT ic.relname FROM pg_index i JOIN pg_class ic ON ic.oid = i.indexrelid WHERE i.indrelid = %s AND i.indisreplident
```

`api_partitions_recipe` line 387

```sql
SELECT tgname, pg_get_triggerdef(oid) AS def,
       obj_description(oid, 'pg_trigger') AS comment
  FROM pg_trigger WHERE tgrelid = %s AND NOT tgisinternal {clone_filter}
 ORDER BY tgname
```

`api_partitions_recipe` line 395

```sql
SELECT p.polname,
       CASE p.polcmd WHEN 'r' THEN 'SELECT' WHEN 'a' THEN 'INSERT'
                     WHEN 'w' THEN 'UPDATE' WHEN 'd' THEN 'DELETE' ELSE 'ALL' END AS cmd,
       p.polpermissive AS permissive,
       (SELECT string_agg(CASE WHEN r = 0 THEN 'PUBLIC'
                               ELSE quote_ident(pg_get_userbyid(r)) END, ', ')
          FROM unnest(p.polroles) r) AS roles,
       pg_get_expr(p.polqual, p.polrelid) AS qual,
       pg_get_expr(p.polwithcheck, p.polrelid) AS withcheck
  FROM pg_policy p WHERE p.polrelid = %s ORDER BY p.polname
```

`api_partitions_recipe` line 412

```sql
WITH RECURSIVE tree AS (
    SELECT inhrelid AS oid FROM pg_inherits WHERE inhparent = %s
    UNION ALL
    SELECT i.inhrelid FROM pg_inherits i JOIN tree t ON i.inhparent = t.oid)
SELECT t.oid, quote_ident(n.nspname) || '.' || quote_ident(c.relname) AS qname
  FROM tree t JOIN pg_class c ON c.oid = t.oid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 ORDER BY 2
```

`api_partitions_recipe` line 428

```sql
SELECT count(*) AS n FROM pg_index i WHERE i.indrelid = ANY(%s) {parent_idx}
```

`api_partitions_recipe` line 432

```sql
SELECT count(*) AS n FROM pg_constraint WHERE conrelid = ANY(%s) AND contype IN ('p','u','x','f') {con_own}
```

`api_partitions_recipe` line 435

```sql
SELECT count(*) AS n FROM pg_trigger WHERE tgrelid = ANY(%s) AND NOT tgisinternal {clone_filter}
```

`api_partitions_recipe` line 443

```sql
SELECT DISTINCT v.oid, v.relkind, v.reloptions,
       quote_ident(n.nspname) || '.' || quote_ident(v.relname) AS qname,
       pg_get_viewdef(v.oid) AS def,
       (SELECT string_agg(DISTINCT d2.refobjid::text, ',') FROM pg_depend d2
         WHERE d2.objid = r.oid AND d2.classid = 'pg_rewrite'::regclass
           AND d2.refobjid = ANY(%s)) AS reads
  FROM pg_depend d
  JOIN pg_rewrite r ON d.classid = 'pg_rewrite'::regclass AND d.objid = r.oid
  JOIN pg_class v ON v.oid = r.ev_class
  JOIN pg_namespace n ON n.oid = v.relnamespace
 WHERE d.refclassid = 'pg_class'::regclass AND d.refobjid = ANY(%s)
   AND v.oid <> ALL(%s)
 ORDER BY 4
```

`api_partitions_recipe` line 459

```sql
SELECT DISTINCT p.oid::regprocedure::text AS fn
  FROM pg_depend d JOIN pg_proc p ON d.classid = 'pg_proc'::regclass AND d.objid = p.oid
 WHERE d.refclassid = 'pg_class'::regclass AND d.refobjid = ANY(%s)
 ORDER BY 1
```

`api_partitions_recipe` line 469

```sql
SELECT DISTINCT CASE WHEN d.classid = 'pg_proc'::regclass THEN d.objid::regprocedure::text
                     ELSE d.objid::regclass::text END AS obj
  FROM pg_depend d
 WHERE d.refclassid = 'pg_type'::regclass AND d.refobjid = %s
   AND d.classid IN ('pg_proc'::regclass, 'pg_class'::regclass) AND d.objid <> %s
 ORDER BY 1
```

`api_partitions_recipe` line 478

```sql
SELECT count(*) AS n FROM pg_rewrite WHERE ev_class = %s AND rulename <> '_RETURN'
```

`api_partitions_recipe` line 481

```sql
SELECT DISTINCT quote_ident(p.pubname) AS pub, pr.prrelid = %s AS on_table
  FROM pg_publication_rel pr JOIN pg_publication p ON p.oid = pr.prpubid
 WHERE pr.prrelid = ANY(%s) ORDER BY 1
```

`api_partitions_recipe` line 565

```sql
LOCK TABLE {q} IN SHARE MODE;
```

`api_partitions_recipe` line 583

```sql
CREATE TABLE {tmp} (LIKE {q} INCLUDING ALL EXCLUDING INDEXES EXCLUDING CONSTRAINTS)
```

`api_partitions_recipe` line 592

```sql
CREATE TABLE {_qq(schema, _fit_name(name, '_pdefault'))} PARTITION OF {tmp} DEFAULT;
```

`api_partitions_recipe` line 598

```sql
CREATE TABLE {tmp} (LIKE {q} INCLUDING ALL EXCLUDING INDEXES EXCLUDING CONSTRAINTS);
```

**stmt** - `api_partitions_recipe` line 606

```sql
ALTER TABLE {tmp} ADD CONSTRAINT {_qi_col(c['conname'])} {c['def']};
```

`api_partitions_recipe` line 621

```sql
INSERT INTO {tmp} ({col_list}){' OVERRIDING SYSTEM VALUE' if always_identity else ''}
```

`api_partitions_recipe` line 622

```sql
SELECT {col_list} FROM {q};
```

`api_partitions_recipe` line 633

```sql
SELECT setval(pg_get_serial_sequence({_lit(tmp)}, {_lit(c['attname'])}), s.last_value, s.is_called) FROM {c['seq']} s;
```

`api_partitions_recipe` line 638

```sql
ALTER SEQUENCE {c['seq']} OWNED BY {tmp}.{_qi_col(c['attname'])};
```

`api_partitions_recipe` line 650

```sql
ALTER INDEX {i['qname']} RENAME TO {_qi_col(_fit_name(i['name'], '_old'))};
```

**stmt** - `api_partitions_recipe` line 657

```sql
ALTER TABLE {tmp} ADD CONSTRAINT {_qi_col(c['conname'])} {c['def']};
```

`api_partitions_recipe` line 675

```sql
ALTER TABLE {tmp} ADD CONSTRAINT {_qi_col(f['conname'])} {fdef};
```

**stmt** - `api_partitions_recipe` line 681

```sql
ALTER TABLE {tmp} ADD CONSTRAINT {_qi_col(c['conname'])} {c['def']};
```

`api_partitions_recipe` line 689

```sql
ALTER TABLE {tmp} ALTER COLUMN {_qi_col(c['attname'])} SET STATISTICS {int(c['stattarget'])};
```

`api_partitions_recipe` line 692

```sql
COMMENT ON CONSTRAINT {_qi_col(c['conname'])} ON {tmp} IS {_lit(c['comment'])};
```

`api_partitions_recipe` line 695

```sql
COMMENT ON INDEX {_qq(schema, i['name'])} IS {_lit(i['comment'])};
```

`api_partitions_recipe` line 697

```sql
ALTER TABLE {tmp} REPLICA IDENTITY USING INDEX {_qi_col(replident_idx)};
```

`api_partitions_recipe` line 699

```sql
ALTER TABLE {tmp} REPLICA IDENTITY FULL;
```

`api_partitions_recipe` line 701

```sql
ALTER TABLE {tmp} REPLICA IDENTITY NOTHING;
```

`api_partitions_recipe` line 709

```sql
COMMENT ON TRIGGER {_qi_col(tg['tgname'])} ON {tmp} IS {_lit(tg['comment'])};
```

`api_partitions_recipe` line 713

```sql
ALTER TABLE {tmp} OWNER TO {_qi_col(rel['owner'])};
```

`api_partitions_recipe` line 715

```sql
ALTER TABLE {_qq(schema, _fit_name(name, '_pdefault'))} OWNER TO {_qi_col(rel['owner'])};
```

`api_partitions_recipe` line 718

```sql
GRANT {g_['privilege_type']} ON {tmp} TO {g_['grantee']}{' WITH GRANT OPTION' if g_['is_grantable'] else ''};
```

`api_partitions_recipe` line 721

```sql
GRANT {g_['privilege_type']} ({_qi_col(g_['attname'])}) ON {tmp} TO {g_['grantee']}{' WITH GRANT OPTION' if g_['is_grantable'] else ''};
```

`api_partitions_recipe` line 724

```sql
COMMENT ON TABLE {tmp} IS {_lit(rel['comment'])};
```

`api_partitions_recipe` line 726

```sql
ALTER TABLE {tmp} ENABLE ROW LEVEL SECURITY;
```

`api_partitions_recipe` line 728

```sql
ALTER TABLE {tmp} FORCE ROW LEVEL SECURITY;
```

**s_** - `api_partitions_recipe` line 730

```sql
CREATE POLICY {_qi_col(p_['polname'])} ON {tmp} AS {'PERMISSIVE' if p_['permissive'] else 'RESTRICTIVE'} FOR {p_['cmd']} TO {p_['roles'] or 'PUBLIC'}
```

`api_partitions_recipe` line 736

```sql
WITH CHECK ({p_['withcheck']})
```

`api_partitions_recipe` line 744

```sql
ALTER TABLE {f['qtable']} DROP CONSTRAINT {_qi_col(f['conname'])};
```

`api_partitions_recipe` line 750

```sql
ALTER TABLE {q} RENAME TO {_qi_col(old_name)};
```

`api_partitions_recipe` line 751

```sql
ALTER TABLE {tmp} RENAME TO {_qi_col(name)};
```

`api_partitions_recipe` line 757

```sql
ALTER TABLE {f['qtable']} ADD CONSTRAINT {_qi_col(f['conname'])} {f['def']};
```

`api_partitions_recipe` line 766

```sql
WITH ({', '.join(v['reloptions'])})
```

`api_partitions_recipe` line 767

```sql
CREATE OR REPLACE VIEW {v['qname']}{opts} AS
```

`api_partitions_recipe` line 773

```sql
ALTER PUBLICATION {p_['pub']} ADD TABLE {q};
```

**sql** - `api_partitions_ddl` line 903

```sql
CREATE TABLE {_qq(cs, cn)} PARTITION OF {_qq(ps, pn)} {bounds}{f' TABLESPACE {_qi(tsp)}' if tsp else ''};
```

**sql** - `api_partitions_ddl` line 906

```sql
ALTER TABLE {_qq(ps, pn)} ATTACH PARTITION {_qq(cs, cn)} {bounds};
```

**sql** - `api_partitions_ddl` line 912

```sql
ALTER TABLE {_qq(ps, pn)} DETACH PARTITION {_qq(cs, cn)} FINALIZE;
```

**sql** - `api_partitions_ddl` line 917

```sql
ALTER TABLE {_qq(ps, pn)} DETACH PARTITION {_qq(cs, cn)} CONCURRENTLY;
```

**sql** - `api_partitions_ddl` line 920

```sql
ALTER TABLE {_qq(ps, pn)} DETACH PARTITION {_qq(cs, cn)};
```

**sql** - `api_partitions_ddl` line 922

```sql
DROP TABLE {_qq(cs, cn)};
```

`api_partitions_ddl` line 935

```sql
SELECT c.reltuples::bigint AS approx_rows,
       pg_size_pretty(pg_total_relation_size(c.oid)) AS size,
       pg_total_relation_size(c.oid) AS bytes,
       EXISTS (SELECT 1 FROM pg_constraint k
                WHERE k.conrelid = c.oid AND k.contype = 'c'
                  AND k.convalidated) AS has_valid_check,
       c.relispartition AS already_partition
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
```

## api/privileges.py

`_server_version` line 78

```sql
SELECT current_setting('server_version_num')::int
```

`_role_names` line 83

```sql
SELECT oid, rolname FROM pg_catalog.pg_roles
```

`_statement` line 103

```sql
GRANT {privs} ON {kw} {obj} TO {who}{' WITH GRANT OPTION' if wgo else ''};
```

**head** - `_render_defacl` line 161

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE {owner}{f' IN SCHEMA {nsp}' if nsp else ''}
```

`_export_schema` line 229

```sql
SELECT current_database(), EXISTS (SELECT 1 FROM pg_catalog.pg_namespace WHERE nspname = %s)
```

`_export_schema` line 238

```sql
SELECT count(*) FROM pg_catalog.pg_default_acl WHERE defaclnamespace = 0
```

`_export_role` line 264

```sql
SELECT oid FROM pg_catalog.pg_roles WHERE rolname = %s
```

`_export_role` line 269

```sql
SELECT current_database()
```

`_export_role` line 271

```sql
SELECT datname, has_database_privilege(datname, 'CONNECT')
                     FROM pg_catalog.pg_database WHERE datallowconn AND NOT datistemplate ORDER BY datname
```

`_export_role` line 284

```sql
SELECT g.rolname FROM pg_catalog.pg_auth_members m
                     JOIN pg_catalog.pg_roles g ON g.oid = m.roleid
                    WHERE m.member = %s ORDER BY 1
```

`api_privileges_export` line 371

```sql
SELECT current_database()
```

`api_privileges_revoke_impact` line 447

```sql
SELECT {priv_fn}(%s, %s, %s) AS has
```

`api_privileges_revoke_impact` line 453

```sql
SELECT g.rolname FROM pg_auth_members am
  JOIN pg_roles g ON g.oid = am.roleid
 WHERE am.member = (SELECT oid FROM pg_roles WHERE rolname = %s)
```

`api_privileges_revoke_impact` line 461

```sql
SELECT {priv_fn}(%s, %s, %s) AS h
```

**schema** - `api_privileges_revoke_impact` line 474

```sql
SELECT nspacl::text[] FROM pg_catalog.pg_namespace WHERE nspname = %s
```

**database** - `api_privileges_revoke_impact` line 475

```sql
SELECT datacl::text[] FROM pg_catalog.pg_database  WHERE datname = %s
```

`api_privileges_revoke_impact` line 477

```sql
SELECT relacl::text[] FROM pg_catalog.pg_class WHERE oid = %s::regclass
```

`api_privileges_revoke_impact` line 486

```sql
SELECT EXISTS (SELECT 1 FROM aclexplode(%s::aclitem[]) a WHERE a.grantee = 0 AND a.privilege_type = %s) AS h
```

## api/repl.py

`api_repl_publications` line 69

```sql
SELECT p.pubname            AS name,
       pg_get_userbyid(p.pubowner) AS owner,
       p.puballtables       AS all_tables,
       p.pubinsert, p.pubupdate, p.pubdelete, p.pubtruncate,
       p.pubviaroot         AS via_partition_root,
       (SELECT count(*) FROM pg_publication_rel r WHERE r.prpubid = p.oid) AS table_count
  FROM pg_publication p
 ORDER BY p.pubname
```

`api_repl_publications` line 82

```sql
SELECT p.pubname AS pub, n.nspname AS schema
  FROM pg_publication_namespace pn
  JOIN pg_publication p ON p.oid = pn.pnpubid
  JOIN pg_namespace   n ON n.oid = pn.pnnspid
 ORDER BY 1, 2
```

`api_repl_publications` line 95

```sql
SELECT p.pubname AS pub,
       n.nspname || '.' || c.relname AS rel
  FROM pg_publication_rel r
  JOIN pg_publication p ON p.oid = r.prpubid
  JOIN pg_class c       ON c.oid = r.prrelid
  JOIN pg_namespace n   ON n.oid = c.relnamespace
 ORDER BY 1, 2
```

`api_repl_subscriptions` line 135

```sql
SELECT s.subname                          AS name,
       pg_get_userbyid(s.subowner)        AS owner,
       s.subenabled                       AS enabled,
       s.subslotname                      AS slot,
       s.subsynccommit                    AS sync_commit,
       array_to_string(s.subpublications, ', ') AS publications,
       (SELECT count(*) FROM pg_subscription_rel sr WHERE sr.srsubid = s.oid) AS rel_count,
       (SELECT count(*) FROM pg_subscription_rel sr
         WHERE sr.srsubid = s.oid AND sr.srsubstate <> 'r')                AS rel_not_ready
  FROM pg_subscription s
 ORDER BY s.subname
```

`api_repl_subscriptions` line 150

```sql
SELECT subname,
       pid,
       received_lsn::text  AS received_lsn,
       latest_end_lsn::text AS latest_end_lsn,
       last_msg_send_time::text  AS last_msg_send_time,
       last_msg_receipt_time::text AS last_msg_receipt_time
  FROM pg_stat_subscription
 ORDER BY subname
```

**sql** - `api_repl_create_publication` line 196

```sql
CREATE PUBLICATION {_ident(name)}
```

`api_repl_create_publication` line 232

```sql
WITH ({', '.join(with_parts)})
```

**s** - `api_repl_create_subscription._stmt` line 295

```sql
CREATE SUBSCRIPTION {_ident(name)} CONNECTION {_quote_pgsl(conn_str)} PUBLICATION {', '.join((_ident(p) for p in pubs))}
```

`api_repl_create_subscription._stmt` line 299

```sql
WITH ({', '.join(with_parts)})
```

`api_repl_alter_subscription` line 370

```sql
ALTER SUBSCRIPTION {qn} SET (slot_name = NONE)
```

**sql** - `api_repl_drop_publication` line 408

```sql
DROP PUBLICATION "{name.replace(chr(34), chr(34) * 2)}"
```

## api/replslots.py

`_slot_columns` line 66

```sql
SELECT attname FROM pg_attribute
 WHERE attrelid = 'pg_replication_slots'::regclass
   AND attnum > 0 AND NOT attisdropped
```

`_fn_args` line 83

```sql
SELECT proname, max(pronargs)
  FROM pg_proc
 WHERE proname IN ('pg_create_physical_replication_slot',
                   'pg_create_logical_replication_slot',
                   'pg_drop_replication_slot',
                   'pg_copy_physical_replication_slot',
                   'pg_copy_logical_replication_slot',
                   'pg_replication_slot_advance',
                   'pg_sync_replication_slots',
                   'pg_logical_slot_peek_changes')
 GROUP BY proname
```

`_privileges` line 107

```sql
SELECT COALESCE(bool_or(r.rolsuper), false)       AS is_super,
       COALESCE(bool_or(r.rolreplication), false) AS has_replication
  FROM pg_roles r
 WHERE pg_has_role(current_user, r.oid, 'USAGE')
```

`_caps` line 146

```sql
SHOW wal_level
```

`_caps` line 151

```sql
SHOW max_replication_slots
```

`_caps` line 161

```sql
SELECT setting FROM pg_settings WHERE name = 'effective_wal_level'
```

`_output_plugins` line 185

```sql
SELECT extname FROM pg_extension ORDER BY extname
```

`_output_plugins` line 189

```sql
SHOW server_version_num
```

`_output_plugins` line 201

```sql
SELECT DISTINCT plugin FROM pg_replication_slots WHERE plugin IS NOT NULL
```

`api_replslots_list` line 251

```sql
SELECT pg_is_in_recovery()
```

`api_replslots_list` line 258

```sql
SELECT {', '.join(sel)} FROM pg_replication_slots ORDER BY slot_name
```

`api_replslots_peek` line 308

```sql
SELECT slot_type FROM pg_replication_slots WHERE slot_name = %s
```

`api_replslots_peek` line 320

```sql
SELECT lsn::text, xid::text, data FROM pg_logical_slot_peek_changes(%s, NULL, %s)
```

`_sees_all_sessions` line 397

```sql
SELECT pg_has_role('pg_read_all_stats', 'USAGE')
```

`_sees_all_sessions` line 399

```sql
SELECT COALESCE(bool_or(rolsuper), false) FROM pg_roles WHERE pg_has_role(current_user, oid, 'USAGE')
```

**sql** - `api_replslots_create` line 530

```sql
SELECT * FROM pg_create_physical_replication_slot({', '.join(args)})
```

**sql** - `api_replslots_create` line 557

```sql
SELECT * FROM pg_create_logical_replication_slot({', '.join(args)})
```

`api_replslots_create` line 561

```sql
SELECT clock_timestamp()
```

`api_replslots_drop` line 636

```sql
SELECT active{', active_pid' if has_pid else ', NULL'} FROM pg_replication_slots WHERE slot_name = %s
```

`api_replslots_drop` line 659

```sql
SELECT pg_terminate_backend(%s)
```

`api_replslots_drop` line 660

```sql
SELECT pg_terminate_backend(%s)
```

`api_replslots_drop` line 664

```sql
SELECT pg_sleep(0.5)
```

`api_replslots_drop` line 665

```sql
SELECT pg_drop_replication_slot(%s)
```

`api_replslots_drop` line 667

```sql
SELECT pg_drop_replication_slot(%s)
```

`api_replslots_advance` line 709

```sql
SELECT * FROM pg_replication_slot_advance(%s, %s::pg_lsn)
```

`api_replslots_advance` line 711

```sql
SELECT * FROM pg_replication_slot_advance(%s, pg_current_wal_lsn())
```

`api_replslots_copy` line 745

```sql
SELECT slot_type FROM pg_replication_slots WHERE slot_name = %s
```

`api_replslots_copy` line 759

```sql
SELECT * FROM {fn}(%s, %s)
```

`api_replslots_copy` line 760

```sql
SELECT * FROM {fn}(%s, %s)
```

## api/role.py

`api_role_alter` line 118

```sql
ALTER ROLE {role}
```

`api_role_alter` line 125

```sql
COMMENT ON ROLE {role} IS NULL
```

`api_role_alter` line 127

```sql
COMMENT ON ROLE {role} IS {c}
```

`api_role_create` line 162

```sql
CREATE ROLE with all attributes + initial comment, group memberships
    and search_path. Body: {role, login, password, capabilities:{}, conn_limit,
    valid_until, in_role:[], comment, search_path:[]}.
    Password is parameter-bound (sql.Literal) so it never reaches logs/text.
    Returns the new role's normalized state from pg_roles.
```

`api_role_create` line 231

```sql
CREATE ROLE {role} WITH
```

`api_role_create` line 237

```sql
COMMENT ON ROLE {role} IS {c}
```

`api_role_create` line 253

```sql
ALTER ROLE {role} SET search_path TO
```

`api_role_create` line 261

```sql
SELECT rolname, rolsuper, rolinherit, rolcreatedb, rolcreaterole,
       rolreplication, {_bypass_rls_col(vnum, 'pg_roles')} AS rolbypassrls,
       rolcanlogin, rolconnlimit,
       rolvaliduntil
FROM pg_roles WHERE rolname = %s
```

`api_role_privileges` line 331

```sql
SELECT n.nspname AS schema,
       has_schema_privilege(%s, n.oid, 'USAGE')  AS usage,
       has_schema_privilege(%s, n.oid, 'CREATE') AS create,
       pg_get_userbyid(n.nspowner)               AS owner
FROM pg_namespace n
WHERE n.nspname NOT IN ('pg_catalog','information_schema')
  AND n.nspname NOT LIKE 'pg_toast%%' AND n.nspname NOT LIKE 'pg_temp%%'
  AND (%s::text IS NULL OR n.nspname = %s)
ORDER BY n.nspname
```

`api_role_privileges` line 346

```sql
WITH rels AS (
    SELECT c.oid, n.nspname AS schema, c.relname AS name, c.relkind
    FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE c.relkind IN ('r','v','m','p','f')
      AND n.nspname NOT IN ('pg_catalog','information_schema')
      AND (%s::text IS NULL OR n.nspname = %s)
)
SELECT schema,
       count(*)                                            AS total,
       sum((has_table_privilege(%s, oid, 'SELECT'))::int)   AS sel,
       sum((has_table_privilege(%s, oid, 'INSERT'))::int)   AS ins,
       sum((has_table_privilege(%s, oid, 'UPDATE'))::int)   AS upd,
       sum((has_table_privilege(%s, oid, 'DELETE'))::int)   AS del,
       sum((has_table_privilege(%s, oid, 'TRUNCATE'))::int) AS trunc,
       sum((has_table_privilege(%s, oid, 'REFERENCES'))::int) AS refs,
       sum((has_table_privilege(%s, oid, 'TRIGGER'))::int)  AS trig
FROM rels
GROUP BY schema ORDER BY schema
```

`api_role_privileges` line 368

```sql
SELECT n.nspname AS schema,
       count(*) AS total,
       sum((has_sequence_privilege(%s, c.oid, 'USAGE'))::int) AS usage,
       sum((has_sequence_privilege(%s, c.oid, 'SELECT'))::int) AS sel,
       sum((has_sequence_privilege(%s, c.oid, 'UPDATE'))::int) AS upd
FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind = 'S'
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND (%s::text IS NULL OR n.nspname = %s)
GROUP BY n.nspname ORDER BY n.nspname
```

`api_role_privileges` line 382

```sql
SELECT n.nspname AS schema,
       count(*) AS total,
       sum((has_function_privilege(%s, p.oid, 'EXECUTE'))::int) AS execute
FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname NOT IN ('pg_catalog','information_schema')
  AND (%s::text IS NULL OR n.nspname = %s)
GROUP BY n.nspname ORDER BY n.nspname
```

`api_role_privileges` line 397

```sql
SELECT n.nspname AS schema,
       count(*) AS total,
       sum((has_type_privilege(%s, t.oid, 'USAGE'))::int) AS usage
FROM pg_type t
JOIN pg_namespace n ON n.oid = t.typnamespace
WHERE t.typtype IN ('c','e','d','r','m')
  AND (t.typtype <> 'c'
       OR EXISTS (SELECT 1 FROM pg_class r
                   WHERE r.oid = t.typrelid AND r.relkind = 'c'))
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND (%s::text IS NULL OR n.nspname = %s)
GROUP BY n.nspname ORDER BY n.nspname
```

`api_role_privileges` line 423

```sql
SELECT n.nspname                       AS schema,
       dacl.defaclobjtype::text        AS objtype,
       pg_get_userbyid(dacl.defaclrole) AS granter,
       dacl.defaclacl::text            AS acl
FROM pg_default_acl dacl
LEFT JOIN pg_namespace n ON n.oid = dacl.defaclnamespace
WHERE n.nspname IS NOT NULL
  AND (%s::text IS NULL OR n.nspname = %s)
ORDER BY n.nspname
```

`api_role_privileges` line 458

```sql
SELECT n.nspname AS schema,
       sum((has_table_privilege(%s, c.oid, 'MAINTAIN'))::int) AS maint
FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','v','m','p','f')
  AND n.nspname NOT IN ('pg_catalog','information_schema')
  AND (%s::text IS NULL OR n.nspname = %s)
GROUP BY n.nspname
```

`api_role_search_path_overview` line 541

```sql
SELECT COALESCE(d.datname, '') AS datname,
       COALESCE(r.rolname, '') AS rolname,
       s.setconfig
  FROM pg_db_role_setting s
  LEFT JOIN pg_database d ON d.oid = s.setdatabase
  LEFT JOIN pg_roles    r ON r.oid = s.setrole
```

`api_role_search_path_overview` line 561

```sql
SELECT boot_val FROM pg_settings WHERE name = 'search_path'
```

`api_role_search_path_overview` line 565

```sql
SELECT datname FROM pg_database
 WHERE datallowconn AND NOT datistemplate ORDER BY datname
```

`api_role_cluster_privs` line 627

```sql
SELECT lanname AS name,
       pg_get_userbyid(lanowner) AS owner,
       has_language_privilege(%s, oid, 'USAGE') AS usage,
       lanpltrusted AS trusted
FROM pg_language
WHERE lanispl OR lanname NOT IN ('internal','c','sql')
ORDER BY lanname
```

`api_role_cluster_privs` line 641

```sql
SELECT fdwname AS name,
       pg_get_userbyid(fdwowner) AS owner,
       has_foreign_data_wrapper_privilege(%s, oid, 'USAGE') AS usage
FROM pg_foreign_data_wrapper ORDER BY fdwname
```

`api_role_cluster_privs` line 652

```sql
SELECT srvname AS name,
       pg_get_userbyid(srvowner) AS owner,
       has_server_privilege(%s, oid, 'USAGE') AS usage,
       (SELECT fdwname FROM pg_foreign_data_wrapper f
        WHERE f.oid = s.srvfdw) AS fdw
FROM pg_foreign_server s ORDER BY srvname
```

`api_role_cluster_privs` line 665

```sql
SELECT spcname AS name,
       pg_get_userbyid(spcowner) AS owner,
       has_tablespace_privilege(%s, oid, 'CREATE') AS create
FROM pg_tablespace ORDER BY spcname
```

`api_role_grantable` line 798

```sql
SELECT current_user, pg_catalog.pg_has_role(current_user, 'pg_monitor', 'MEMBER') AS is_monitor
```

`api_role_grantable` line 815

```sql
SELECT
  (SELECT pg_catalog.pg_has_role(%s, nspowner, 'USAGE')
     FROM pg_catalog.pg_namespace WHERE nspname = %s)    AS is_owner,
  has_schema_privilege(%s, %s, 'USAGE WITH GRANT OPTION') AS usage_grantable,
  has_schema_privilege(%s, %s, 'CREATE WITH GRANT OPTION') AS create_grantable
```

`api_role_grantable` line 829

```sql
SELECT current_setting('server_version_num')::int
```

`api_role_grantable` line 843

```sql
SELECT COUNT(*) = SUM(CASE WHEN has_table_privilege(%s, c.oid, %s) THEN 1 ELSE 0 END)
FROM pg_catalog.pg_class c
JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relkind IN ('r','p')
```

`api_role_grantable` line 859

```sql
SELECT COUNT(*) = SUM(CASE WHEN has_sequence_privilege(%s, c.oid, %s) THEN 1 ELSE 0 END)
FROM pg_catalog.pg_class c
JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relkind = 'S'
```

`api_role_grantable` line 874

```sql
SELECT COUNT(*) = SUM(CASE WHEN has_function_privilege(%s, p.oid, 'EXECUTE WITH GRANT OPTION') THEN 1 ELSE 0 END)
FROM pg_catalog.pg_proc p
JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname = %s
```

`api_role_grantable` line 895

```sql
SELECT g.rolname
FROM pg_catalog.pg_roles g, pg_catalog.pg_roles me
WHERE me.rolname = %s
  AND (me.rolsuper
       OR pg_catalog.pg_has_role(me.oid, g.oid, 'MEMBER WITH ADMIN OPTION')
       OR (%s < 160000 AND me.rolcreaterole AND NOT g.rolsuper))
ORDER BY g.rolname
```

`api_role_privileges_apply` line 1064

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE {fr} IN SCHEMA {s}
```

`api_role_privileges_apply` line 1069

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA {s}
```

`api_role_privileges_apply` line 1091

```sql
SELECT t.typname
FROM pg_type t
JOIN pg_namespace n ON n.oid = t.typnamespace
WHERE t.typtype IN ('c','e','d','r','m')
  AND (t.typtype <> 'c'
       OR EXISTS (SELECT 1 FROM pg_class r
                   WHERE r.oid = t.typrelid AND r.relkind = 'c'))
  AND (t.typtype <> 'm'
       OR current_setting('server_version_num')::int < 170000)
  AND n.nspname = %s
```

`api_role_audit_summary` line 1204

```sql
SELECT r.oid, r.rolname FROM pg_roles r
 WHERE r.rolname <> %s
   AND pg_has_role(%s, r.oid, 'USAGE')
 ORDER BY r.rolname
```

`api_role_audit_summary` line 1212

```sql
SELECT oid, rolname FROM pg_roles
```

**sql** - `api_role_privileges_copy` line 1298

```sql
SELECT n.nspname AS schema, c.relname AS name, c.relkind AS relkind,
       a.privilege_type AS privilege, a.is_grantable AS wgo
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN aclexplode(c.relacl) a ON true
 WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND c.relkind IN ('r','p','v','m','S','f')
   AND a.grantee = (SELECT oid FROM pg_roles WHERE rolname = %s)
```

**stmt** - `api_role_privileges_copy` line 1323

```sql
GRANT {r['privilege']} ON {kw} {fq} TO {dst_q}{' WITH GRANT OPTION' if r['wgo'] else ''}
```

`api_role_privileges_copy` line 1328

```sql
SELECT n.nspname AS schema,
       a.privilege_type AS priv,
       a.is_grantable AS wgo
  FROM pg_namespace n
  JOIN aclexplode(n.nspacl) a ON true
 WHERE a.grantee = (SELECT oid FROM pg_roles WHERE rolname = %s)
   AND n.nspname NOT IN ('pg_catalog', 'information_schema')
```

**stmt** - `api_role_privileges_copy` line 1339

```sql
GRANT {r['priv']} ON SCHEMA {_ident_str(r['schema'])} TO {dst_q}{' WITH GRANT OPTION' if r['wgo'] else ''}
```

`api_role_check_drop` line 1433

```sql
SELECT coalesce(d.datname, '(shared)') AS database,
       count(*) AS deps
  FROM pg_shdepend s
  LEFT JOIN pg_database d ON d.oid = s.dbid
  JOIN pg_authid a ON a.oid = s.refobjid
 WHERE a.rolname = %s
 GROUP BY d.datname
 ORDER BY 2 DESC
```

`api_role_check_drop` line 1453

```sql
SELECT oid FROM pg_roles WHERE rolname = %s
```

`api_role_check_drop` line 1484

```sql
SELECT m.rolname AS member, g.rolname AS granted_role,
       am.admin_option
  FROM pg_auth_members am
  JOIN pg_roles m ON m.oid = am.member
  JOIN pg_roles g ON g.oid = am.roleid
 WHERE m.rolname = %s OR g.rolname = %s
 ORDER BY m.rolname, g.rolname
```

`api_role_check_drop` line 1497

```sql
SELECT rolname FROM pg_roles
 WHERE rolname <> %s AND rolname NOT LIKE 'pg_%%'
 ORDER BY rolcanlogin DESC, rolname
```

`api_role_check_drop` line 1506

```sql
SELECT current_user AS me, current_database() AS db
```

`api_role_drop` line 1688

```sql
DROP ROLE {r}
```

`api_role_drop` line 1689

```sql
DROP ROLE {role}
```

`api_role_memberships` line 1735

```sql
SELECT g.rolname, am.admin_option, am.inherit_option, am.set_option
  FROM pg_auth_members am
  JOIN pg_roles g ON g.oid = am.roleid
  JOIN pg_roles m ON m.oid = am.member
 WHERE m.rolname = %s
 ORDER BY g.rolname
```

`api_role_memberships` line 1744

```sql
SELECT g.rolname, am.admin_option, m.rolinherit, TRUE
  FROM pg_auth_members am
  JOIN pg_roles g ON g.oid = am.roleid
  JOIN pg_roles m ON m.oid = am.member
 WHERE m.rolname = %s
 ORDER BY g.rolname
```

`api_role_template` line 1792

```sql
SELECT rolname, rolsuper, rolinherit, rolcreatedb, rolcreaterole,
       rolreplication, {_bypass_rls_col(vnum, 'pg_roles')} AS rolbypassrls,
       rolcanlogin, rolconnlimit,
       rolvaliduntil,
       pg_catalog.shobj_description(oid, 'pg_authid') AS comment
  FROM pg_roles WHERE rolname = %s
```

`api_role_template` line 1805

```sql
SELECT g.rolname AS name
  FROM pg_auth_members am
  JOIN pg_roles g ON g.oid = am.roleid
  JOIN pg_roles m ON m.oid = am.member
 WHERE m.rolname = %s
 ORDER BY g.rolname
```

`api_role_template` line 1819

```sql
SELECT count(*) AS n
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN aclexplode(c.relacl) a ON true
 WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND c.relkind IN ('r','p','v','m','S','f')
   AND a.grantee = (SELECT oid FROM pg_roles WHERE rolname = %s)
```

`api_role_template` line 1829

```sql
SELECT count(*) AS n
  FROM pg_namespace n
  JOIN aclexplode(n.nspacl) a ON true
 WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND a.grantee = (SELECT oid FROM pg_roles WHERE rolname = %s)
```

`_grant_membership_sql` line 1883

```sql
GRANT {parent} TO {member}
```

`api_role_membership_apply` line 1964

```sql
REVOKE {parent} FROM {member}
```

`api_role_search_path` line 2021

```sql
ALTER ROLE {r}
```

`api_role_search_path` line 2021

```sql
RESET search_path
```

`api_role_search_path` line 2024

```sql
ALTER ROLE {r}
```

`api_role_search_path` line 2026

```sql
SET search_path TO
```

`api_role_timeouts` line 2094

```sql
SELECT COALESCE(d.datname, '') AS datname,
       COALESCE(r.rolname, '') AS rolname,
       s.setconfig
  FROM pg_db_role_setting s
  LEFT JOIN pg_database d ON d.oid = s.setdatabase
  LEFT JOIN pg_roles    r ON r.oid = s.setrole
```

`api_role_timeouts` line 2131

```sql
SELECT name, setting FROM pg_file_settings
                            WHERE applied AND name = ANY(%s) ORDER BY seqno
```

`api_role_timeouts` line 2141

```sql
SELECT name, reset_val, boot_val, source, unit
                         FROM pg_settings WHERE name = ANY(%s)
```

`api_role_timeouts` line 2156

```sql
SELECT rolconnlimit FROM pg_roles WHERE rolname = %s
```

`api_role_timeouts_apply` line 2233

```sql
ALTER ROLE {r}
```

`api_role_ddl` line 2309

```sql
WITH RECURSIVE up(oid) AS (
    SELECT am.roleid FROM pg_auth_members am JOIN pg_roles m ON m.oid = am.member
     WHERE m.rolname = %s
    UNION
    SELECT am.roleid FROM pg_auth_members am JOIN up ON am.member = up.oid)
SELECT r.rolname FROM pg_roles r JOIN up ON up.oid = r.oid
 WHERE r.rolname NOT LIKE 'pg\_%%' AND r.rolname <> %s
 ORDER BY 1
```

`api_role_ddl` line 2334

```sql
SELECT rolpassword FROM pg_authid LIMIT 1
```

`api_role_ddl` line 2352

```sql
SELECT r.rolname, r.rolsuper, r.rolinherit, r.rolcreaterole,
       r.rolcreatedb, r.rolcanlogin, r.rolreplication,
       {_bypass_rls_col(vnum)} AS rolbypassrls, r.rolconnlimit, r.rolvaliduntil,
       {pw_col},
       pg_catalog.shobj_description(r.oid, 'pg_authid') AS comment
  FROM pg_roles r {pw_join} {where}
 ORDER BY r.rolname
```

`api_role_ddl` line 2411

```sql
CREATE ROLE {name} {' '.join(opts)};
```

`api_role_ddl` line 2415

```sql
COMMENT ON ROLE {name} IS {_literal_str(r['comment'])};
```

`api_role_ddl` line 2432

```sql
SELECT g.rolname AS grp, m.rolname AS member, am.admin_option, {opt16}
  FROM pg_auth_members am
  JOIN pg_roles g ON g.oid = am.roleid
  JOIN pg_roles m ON m.oid = am.member
 WHERE m.rolname NOT LIKE 'pg\_%%'
   AND (%s OR m.rolname = ANY(%s) OR g.rolname = %s)
 ORDER BY g.rolname, m.rolname
```

**stmt** - `api_role_ddl` line 2459

```sql
GRANT {_ident_str(grp)} TO {_ident_str(member)}{' WITH ' + ', '.join(opts) if opts else ''};
```

`api_role_ddl` line 2492

```sql
SELECT r.rolname, COALESCE(d.datname, '') AS dbname, s.setconfig
  FROM pg_db_role_setting s
  JOIN pg_roles r        ON r.oid = s.setrole
  LEFT JOIN pg_database d ON d.oid = s.setdatabase
 WHERE r.rolname NOT LIKE 'pg\_%%'
   AND (%s OR r.rolname = ANY(%s))
 ORDER BY r.rolname, 2
```

`api_role_ddl` line 2511

```sql
ALTER ROLE %s%s SET %s = %s;
```

## api/sequences.py

**q** - `api_sequences_health` line 35

```sql
SELECT s.schemaname AS schema, s.sequencename AS name,
       s.last_value, s.max_value, s.min_value, s.start_value,
       s.increment_by, s.cycle, s.cache_size
  FROM pg_sequences s
 WHERE s.schemaname NOT IN ('pg_catalog','information_schema')
   AND s.schemaname NOT LIKE 'pg_toast%%'
   AND s.schemaname NOT LIKE 'pg_temp_%%'
```

**q** - `api_sequences_health` line 52

```sql
SELECT n.nspname AS schema, c.relname AS name,
       NULL::bigint AS last_value, NULL::bigint AS max_value,
       NULL::bigint AS min_value, NULL::bigint AS start_value,
       NULL::bigint AS increment_by, NULL::bool   AS cycle,
       NULL::bigint AS cache_size
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.relkind = 'S'
   AND n.nspname NOT IN ('pg_catalog','information_schema')
```

`api_sequences_health` line 86

```sql
SELECT n.nspname || '.' || c.relname AS sequence,
       tn.nspname AS owner_schema,
       tc.relname AS owner_table,
       a.attname  AS owner_column,
       {_identity_col} AS identity_kind
  FROM pg_depend d
  JOIN pg_class c     ON c.oid = d.objid AND c.relkind = 'S'
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_class tc    ON tc.oid = d.refobjid
  JOIN pg_namespace tn ON tn.oid = tc.relnamespace
  JOIN pg_attribute a ON a.attrelid = d.refobjid AND a.attnum = d.refobjsubid
 WHERE d.classid = 'pg_class'::regclass
   AND d.refclassid = 'pg_class'::regclass
   AND d.deptype IN ('a', 'i')
   AND tc.relkind IN ('r', 'p')
```

`api_sequences_health` line 120

```sql
SELECT MAX({}) AS mx FROM {}.{}
```

`api_sequences_health` line 161

```sql
SELECT setval(format('%I.%I', '{s['schema'].replace("'", "''")}', '{s['name'].replace("'", "''")}')::regclass, {str(mx) if mx is not None else '1'}, true);
```

`api_sequences_repair` line 222

```sql
SELECT setval({}::regclass, %s, true)
```

`api_sequences_find_serial` line 272

```sql
    SELECT n.nspname AS schema, c.relname AS table, a.attname AS column,
           pg_get_expr(d.adbin, d.adrelid) AS default_expr,
           sn.nspname AS seq_schema, sc.relname AS seq_name,
           COALESCE(s.last_value, 0) AS last_value
      FROM pg_attribute a
      JOIN pg_class c ON c.oid = a.attrelid
      JOIN pg_namespace n ON n.oid = c.relnamespace
      JOIN pg_attrdef d ON d.adrelid = c.oid AND d.adnum = a.attnum
      LEFT JOIN pg_depend dep ON dep.refobjid = c.oid
                             AND dep.refobjsubid = a.attnum
                             AND dep.deptype = 'a'
      LEFT JOIN pg_class sc ON sc.oid = dep.objid AND sc.relkind = 'S'
      LEFT JOIN pg_namespace sn ON sn.oid = sc.relnamespace
      LEFT JOIN pg_sequences s ON s.schemaname = sn.nspname AND s.sequencename = sc.relname
     WHERE c.relkind IN ('r','p')
       AND n.nspname NOT IN ('pg_catalog','information_schema')
       AND a.attidentity = ''
       AND pg_get_expr(d.adbin, d.adrelid) LIKE 'nextval(%%'
       AND sc.oid IS NOT NULL
{' AND n.nspname = %s' if schema_filter else ''}
```

`api_sequences_find_serial` line 302

```sql
ALTER TABLE {ts} ALTER COLUMN {col} DROP DEFAULT;
ALTER TABLE {ts} ALTER COLUMN {col} ADD GENERATED BY DEFAULT AS IDENTITY (START WITH {start});
DROP SEQUENCE "{r['seq_schema']}"."{r['seq_name']}";
```

`api_sequences_serial_to_identity` line 357

```sql
SELECT pg_get_serial_sequence(%s, %s)
```

`api_sequences_serial_to_identity` line 362

```sql
SELECT last_value, is_called FROM {}
```

`api_sequences_serial_to_identity` line 366

```sql
SELECT max({}) FROM {}.{}
```

`api_sequences_serial_to_identity` line 371

```sql
ALTER TABLE {}.{} ALTER COLUMN {} DROP DEFAULT
```

`api_sequences_serial_to_identity` line 374

```sql
ALTER TABLE {}.{} ALTER COLUMN {} ADD GENERATED {mode} AS IDENTITY (START WITH {str(start_with)})
```

`api_sequences_serial_to_identity` line 382

```sql
ALTER TABLE %s ALTER COLUMN %s DROP DEFAULT
```

`api_sequences_serial_to_identity` line 383

```sql
ALTER TABLE %s ALTER COLUMN %s ADD GENERATED %s AS IDENTITY (START WITH %d)
```

`api_sequences_drop_orphans` line 420

```sql
SELECT n.nspname, c.relname FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_depend d ON d.objid = c.oid AND d.deptype IN ('a', 'i')
 WHERE c.relkind = 'S'
   AND n.nspname NOT IN ('pg_catalog','information_schema')
   AND d.objid IS NULL
```

## api/subset.py

`_resolve_oid` line 18

```sql
SELECT c.oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s AND c.relkind IN ('r','p')
```

`_fk_edges` line 34

```sql
SELECT k.conname,
       k.conrelid, k.confrelid, k.confdeltype,
       (SELECT array_agg(a.attname ORDER BY u.ord)
          FROM unnest(k.conkey) WITH ORDINALITY u(attnum, ord)
          JOIN pg_attribute a ON a.attrelid = k.conrelid AND a.attnum = u.attnum) AS cols,
       (SELECT array_agg(a.attname ORDER BY u.ord)
          FROM unnest(k.confkey) WITH ORDINALITY u(attnum, ord)
          JOIN pg_attribute a ON a.attrelid = k.confrelid AND a.attnum = u.attnum) AS refcols
  FROM pg_constraint k
 WHERE k.contype = 'f' AND (k.conrelid = %s OR k.confrelid = %s)
```

`_table_meta` line 64

```sql
SELECT n.nspname, c.relname,
       (SELECT array_agg(a.attname ORDER BY a.attnum)
          FROM pg_attribute a
         WHERE a.attrelid = c.oid AND a.attnum > 0 AND NOT a.attisdropped) AS cols,
       (SELECT array_agg(a.attname ORDER BY a.attnum)
          FROM pg_index i
          JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum = ANY(i.indkey)
         WHERE i.indrelid = c.oid AND i.indisprimary) AS pk_cols,
       {_identity_expr} AS has_identity
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.oid = %s
```

`_subset_walk` line 130

```sql
SELECT {} FROM {}.{} WHERE {where_sql} LIMIT %s
```

`_subset_walk` line 198

```sql
SELECT {} FROM {}.{} WHERE ({}) IN ({placeholders}) LIMIT %s
```

`api_subset_export` line 366

```sql
SET CONSTRAINTS ALL DEFERRED;
```

`api_subset_export` line 378

```sql
SELECT attname FROM pg_attribute WHERE attrelid = %s AND attgenerated <> ''
```

`api_subset_export` line 393

```sql
SELECT {} FROM {}.{} WHERE ({}) IN ({placeholders})
```

`api_subset_export` line 399

```sql
COPY {_qq(sch, nm)} ({col_list}) FROM stdin;
```

`api_subset_export` line 413

```sql
INSERT INTO {_qq(sch, nm)} ({col_list}){ovr} VALUES ({vals});
```

## api/tablespaces.py

`api_tablespaces_inventory` line 23

```sql
SELECT t.spcname AS name,
       pg_get_userbyid(t.spcowner) AS owner,
       pg_tablespace_location(t.oid) AS location,
       -- pg_tablespace_size() needs CREATE on the tablespace (or
       -- superuser). A plain role gets "permission denied for
       -- tablespace pg_global" - which is every non-superuser,
       -- including every managed-instance user - and that refusal
       -- took the whole inventory down rather than one column of
       -- one row. Gate it per row: the listing is the point, the
       -- size is a bonus, and NULL says "not visible to you".
       CASE WHEN has_tablespace_privilege(t.oid, 'CREATE')
            THEN pg_size_pretty(pg_tablespace_size(t.oid)) END AS size,
       CASE WHEN has_tablespace_privilege(t.oid, 'CREATE')
            THEN pg_tablespace_size(t.oid) END AS size_bytes,
       COALESCE(t.spcoptions, ARRAY[]::text[]) AS options,
       COALESCE(t.spcacl::text[], ARRAY[]::text[]) AS acl
  FROM pg_tablespace t
 ORDER BY t.spcname
```

`api_tablespaces_create` line 53

```sql
CREATE TABLESPACE name [OWNER role] LOCATION '/path' [WITH (opt=val,...)]
    Note: requires superuser; the directory must already exist and be owned
    by the postgres OS user.
```

**sql** - `api_tablespaces_create` line 76

```sql
CREATE TABLESPACE {_ident(name)}
```

`api_tablespaces_create` line 86

```sql
WITH ({', '.join(opt_pairs)})
```

`api_tablespaces_alter` line 130

```sql
ALTER TABLESPACE {_ident(name)} RENAME TO {_ident(nn)}
```

`api_tablespaces_alter` line 135

```sql
ALTER TABLESPACE {_ident(name)} OWNER TO {_ident(no)}
```

`api_tablespaces_alter` line 144

```sql
ALTER TABLESPACE {_ident(name)} SET ({', '.join(pairs)})
```

`api_tablespaces_alter` line 153

```sql
ALTER TABLESPACE {_ident(name)} RESET ({', '.join(ks)})
```

**sql** - `api_tablespaces_drop` line 177

```sql
DROP TABLESPACE {_ident(name)}
```

## api/timescaledb.py

`_rel` line 53

```sql
SELECT c.oid, pg_has_role(c.relowner, 'USAGE'), pg_get_userbyid(c.relowner)
                     FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
                    WHERE n.nspname = %s AND c.relname = %s
```

`_interval` line 64

```sql
SELECT %s::interval::text
```

`plan` line 129

```sql
SELECT compression_enabled FROM timescaledb_information.hypertables
                                WHERE hypertable_schema = %s AND hypertable_name = %s
```

**undo** - `plan` line 151

```sql
SELECT {tsdb._fn(info, 'add_' + kind + '_policy')}({tsdb._literal_str(tsdb._ident_str(schema, name))}::regclass, {was_sql});
```

`plan` line 169

```sql
SELECT %s::timestamptz
```

`plan` line 179

```sql
SELECT pg_has_role(owner, 'USAGE'), owner::text, schedule_interval::text, scheduled,
                              application_name
                         FROM timescaledb_information.jobs WHERE job_id = %s
```

## api/tools.py

`server_timeouts` line 222

```sql
SELECT name, reset_val, unit FROM pg_settings WHERE name = ANY(%s)
```

`server_timeouts` line 235

```sql
SELECT COALESCE(d.datname, '')  AS datname,
       COALESCE(r.rolname, '')  AS rolname,
       s.setconfig
  FROM pg_db_role_setting s
  LEFT JOIN pg_database d ON d.oid = s.setdatabase
  LEFT JOIN pg_roles    r ON r.oid = s.setrole
```

`_settings_this_server_lacks` line 330

```sql
SELECT name FROM pg_settings WHERE name = ANY(%s)
```

`_settings_this_server_lacks` line 333

```sql
SHOW server_version
```

`_server_tablespaces` line 953

```sql
SELECT spcname FROM pg_tablespace WHERE spcname <> 'pg_global' ORDER BY spcname
```

`_restore_selection` line 1537

```sql
SELECT nspname FROM pg_namespace WHERE nspname = ANY(%s)
```

`_needed_the_target_lacks` line 1585

```sql
SELECT 1 FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname = %s
```

`_needed_the_target_lacks` line 1590

```sql
SELECT 1 FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace WHERE n.nspname = %s AND p.proname || '(' || oidvectortypes(p.proargtypes) || ')' = %s
```

`api_db_restore` line 1793

```sql
SET ROLE {dump_scope.quote_ident(role)}
```

`api_db_restore` line 1899

```sql
CREATE SCHEMA IF NOT EXISTS {dump_scope.quote_ident(n)}
```

`api_db_restore` line 1964

```sql
SET ROLE {dump_scope.quote_ident(role)}
```

`api_db_restore` line 1966

```sql
CREATE EXTENSION IF NOT EXISTS timescaledb
```

`api_db_restore` line 1967

```sql
SELECT timescaledb_pre_restore()
```

`api_db_restore` line 1981

```sql
SET ROLE {dump_scope.quote_ident(role)}
```

`api_db_restore` line 1982

```sql
SELECT timescaledb_post_restore()
```

`api_db_remap_schema` line 2280

```sql
SELECT nspname FROM pg_namespace WHERE nspname = ANY(%s)
```

`api_db_remap_schema` line 2320

```sql
SELECT n.nspname || '.' || p.proname AS routine,
       pg_get_function_identity_arguments(p.oid) AS args,
       p.prosrc                                  AS body,
       COALESCE(p.proconfig, ARRAY[]::text[])    AS config
  FROM pg_proc p
  JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname NOT IN ('pg_catalog','information_schema')
   AND (p.prosrc ILIKE %s
     OR COALESCE(array_to_string(p.proconfig, ' '), '') ILIKE %s)
 ORDER BY 1
```

**stmt** - `api_db_remap_schema` line 2363

```sql
ALTER SCHEMA {_ident_str(src)} RENAME TO {_ident_str(dst)}
```

`_schema_objects` line 2405

```sql
SELECT c.relname, c.relkind::text
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind = ANY(%s)
 ORDER BY c.relname
```

`_schema_objects` line 2416

```sql
SELECT c.relname, a.attname, format_type(a.atttypid, a.atttypmod)
  FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY c.relname, a.attnum
```

`_schema_objects` line 2432

```sql
SELECT s.relname, t.relname
  FROM pg_depend d
  JOIN pg_class s ON s.oid = d.objid AND s.relkind = 'S'
  JOIN pg_class t ON t.oid = d.refobjid
  JOIN pg_namespace n ON n.oid = s.relnamespace
 WHERE d.classid = 'pg_class'::regclass
   AND d.refclassid = 'pg_class'::regclass
   AND d.deptype IN ('a', 'i')
   AND n.nspname = %s
```

`_pk_columns` line 2450

```sql
SELECT a.attname
  FROM pg_index i
  JOIN pg_class c ON c.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum = ANY(i.indkey)
 WHERE i.indisprimary AND n.nspname = %s AND c.relname = %s
 ORDER BY (SELECT u.ord FROM unnest(i.indkey::smallint[]) WITH ORDINALITY AS u(k, ord) WHERE u.k = a.attnum LIMIT 1)
```

`api_db_merge_schema` line 2581

```sql
SELECT nspname FROM pg_namespace WHERE nspname = ANY(%s)
```

`api_db_merge_schema._copy_rows` line 2721

```sql
SELECT a.attname, {_ident}, {_gen} AS generated
  FROM pg_attribute a
 WHERE a.attrelid = %s::regclass AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

**ins** - `api_db_merge_schema._copy_rows` line 2740

```sql
INSERT INTO {q_dst} ({collist}){' OVERRIDING SYSTEM VALUE' if always else ''} SELECT {collist} FROM {q_src}{tail}
```

`api_db_merge_schema._copy_rows` line 2757

```sql
SELECT pg_get_serial_sequence(%s, %s)
```

`api_db_merge_schema._copy_rows` line 2761

```sql
SELECT pg_get_serial_sequence(%s, %s)
```

`api_db_merge_schema._copy_rows` line 2769

```sql
SELECT setval({_literal(seq)}, v) FROM (SELECT GREATEST({', '.join(bound)}) AS v) x WHERE v IS NOT NULL
```

`api_db_merge_schema` line 2802

```sql
SELECT setval({_literal(q_dst)}, last_value, is_called) FROM {q_src}
```

`api_db_merge_schema` line 2808

```sql
CREATE TABLE {q_dst} (LIKE {q_src} INCLUDING ALL)
```

`api_db_merge_schema` line 2815

```sql
SELECT %s::regclass::oid
```

`api_db_merge_schema` line 2822

```sql
SET LOCAL row_security = off
```

`api_db_merge_schema` line 2825

```sql
RESET row_security
```

`api_db_merge_schema` line 2830

```sql
ALTER {KINDWORD[r['kind']]} {q_src} SET SCHEMA {_ident_str(dst)}
```

`api_db_merge_schema` line 2846

```sql
ALTER {kindword} {q_src} SET SCHEMA {_ident_str(dst)}
```

`api_db_merge_schema` line 2890

```sql
SELECT count(*) FROM pg_class c JOIN pg_namespace n ON n.oid=c.relnamespace
                            WHERE n.nspname=%s AND c.relkind = ANY(%s)
```

`api_db_merge_schema` line 2902

```sql
DROP SCHEMA {_ident_str(src)}
```
