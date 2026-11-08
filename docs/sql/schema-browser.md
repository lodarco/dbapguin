# Schema browser

[All parts](README.md) · 349 statements

## api/schema/acl.py

`api_schema_create` line 188

```sql
CREATE SCHEMA. Body: {name, owner?, if_not_exists?, comment?}.
    Quotes the schema and owner via sql.Identifier; comment goes through
    sql.Literal so quotes/backslashes are safe.
```

`api_schema_create` line 204

```sql
CREATE SCHEMA
```

`api_schema_create` line 216

```sql
COMMENT ON SCHEMA {n} IS {c}
```

`api_schema_properties` line 245

```sql
SELECT n.nspname,
       pg_catalog.pg_get_userbyid(n.nspowner) AS owner,
       obj_description(n.oid, 'pg_namespace')  AS comment,
       has_schema_privilege(current_user, n.nspname, 'CREATE') AS can_create,
       has_schema_privilege(current_user, n.nspname, 'USAGE')  AS can_usage,
       -- Ownership as PostgreSQL itself decides it, which is not
       -- "my name is on it or I am a superuser".
       --
       -- That is what this used to ask, and on every managed
       -- platform it was wrong. Cloud SQL owns the schemas with
       -- cloudsqlsuperuser and makes the postgres login a MEMBER
       -- of it; RDS does the same with rds_superuser. Neither
       -- login is rolsuper - managed providers do not hand that
       -- out - and neither name sits in nspowner, so the owner
       -- field greyed out with "not owner and not superuser"
       -- while the server was perfectly willing: the right-click
       -- "change owner" runs the ALTER without pre-judging it,
       -- and it worked. The application contradicted itself, and
       -- the half that was wrong was the half that had decided
       -- the answer in advance.
       --
       -- pg_has_role with USAGE is the same test the server
       -- applies (has_privs_of_role): true for the owner, for a
       -- role that inherits the owner's rights, and for a
       -- superuser, who is a member of everything. USAGE and not
       -- MEMBER, because a NOINHERIT member has to SET ROLE
       -- before the server will let them, and answering yes
       -- before they have is how this lie started.
       pg_catalog.pg_has_role(current_user, n.nspowner, 'USAGE')
           AS can_alter
FROM pg_namespace n
WHERE n.nspname = %s
```

`api_schema_properties` line 283

```sql
SELECT
  count(*) FILTER (WHERE relkind = 'r')       AS tables,
  count(*) FILTER (WHERE relkind = 'p')       AS partitioned,
  count(*) FILTER (WHERE relkind = 'v')       AS views,
  count(*) FILTER (WHERE relkind = 'm')       AS matviews,
  count(*) FILTER (WHERE relkind = 'f')       AS foreign_tables,
  count(*) FILTER (WHERE relkind = 'S')       AS sequences,
  count(*) FILTER (WHERE relkind = 'i' OR relkind = 'I') AS indexes
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s
```

`api_schema_properties` line 299

```sql
SELECT count(*) AS n FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname = %s
   AND NOT EXISTS (SELECT 1 FROM pg_depend d
                    WHERE d.classid = 'pg_proc'::regclass
                      AND d.objid = p.oid AND d.deptype = 'i')
```

`api_schema_properties` line 310

```sql
SELECT count(*) AS n FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace
 WHERE n.nspname = %s AND t.typtype IN ('c','e','d','r')
   AND (t.typtype <> 'c'
        OR EXISTS (SELECT 1 FROM pg_class rc
                    WHERE rc.oid = t.typrelid AND rc.relkind = 'c'))
```

`api_schema_properties` line 319

```sql
SELECT rolname FROM pg_roles WHERE rolname NOT LIKE 'pg\_%' ORDER BY rolname
```

`api_schema_properties_save` line 361

```sql
ALTER SCHEMA {n} OWNER TO {o}
```

`api_schema_properties_save` line 367

```sql
COMMENT ON SCHEMA {n} IS NULL
```

`api_schema_properties_save` line 370

```sql
COMMENT ON SCHEMA {n} IS {c}
```

`api_schema_privileges` line 401

```sql
SELECT rolname,
       has_schema_privilege(rolname, %s, 'USAGE')  AS u,
       has_schema_privilege(rolname, %s, 'CREATE') AS c
FROM pg_roles
ORDER BY rolname
```

`api_schema_privileges_set` line 449

```sql
SELECT has_schema_privilege(%s, %s, 'USAGE')  AS u, has_schema_privilege(%s, %s, 'CREATE') AS c
```

`api_schema_privileges_set` line 457

```sql
GRANT USAGE ON SCHEMA {s} TO {r}
```

`api_schema_privileges_set` line 459

```sql
REVOKE USAGE ON SCHEMA {s} FROM {r}
```

`api_schema_privileges_set` line 461

```sql
GRANT CREATE ON SCHEMA {s} TO {r}
```

`api_schema_privileges_set` line 463

```sql
REVOKE CREATE ON SCHEMA {s} FROM {r}
```

**stmt** - `api_schema_bulk_privileges` line 534

```sql
GRANT {priv_list} ON ALL {tspec['sql']} IN SCHEMA {sch_q} TO {role_q}{' WITH GRANT OPTION' if wgo else ''}
```

**stmt** - `api_schema_bulk_privileges` line 538

```sql
REVOKE {'GRANT OPTION FOR ' if wgo else ''}{priv_list} ON ALL {tspec['sql']} IN SCHEMA {sch_q} FROM {role_q}
```

**stmt** - `api_schema_bulk_privileges` line 544

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA {sch_q} GRANT {priv_list} ON {tspec['sql']} TO {role_q}{' WITH GRANT OPTION' if wgo else ''}
```

**stmt** - `api_schema_bulk_privileges` line 548

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA {sch_q} REVOKE {'GRANT OPTION FOR ' if wgo else ''}{priv_list} ON {tspec['sql']} FROM {role_q}
```

`api_schema_default_privileges` line 586

```sql
SELECT pg_get_userbyid(dacl.defaclrole) AS granter,
       dacl.defaclobjtype             AS target_kind,
       dacl.defaclacl::text[]         AS acl_text,
       dacl.defaclacl                 AS acl_raw
  FROM pg_default_acl dacl
  JOIN pg_namespace n ON n.oid = dacl.defaclnamespace
 WHERE n.nspname = %s
 ORDER BY granter, target_kind
```

`api_schema_default_privileges` line 603

```sql
SELECT CASE WHEN a.grantee = 0 THEN 'PUBLIC'
            ELSE COALESCE(pg_get_userbyid(a.grantee), '(deleted)') END AS grantee,
       a.privilege_type AS privilege,
       a.is_grantable AS with_grant_option,
       COALESCE(pg_get_userbyid(a.grantor), '(deleted)') AS granted_by
  FROM aclexplode(%s::aclitem[]) a
 ORDER BY grantee, privilege
```

**head** - `api_schema_default_privileges_set` line 670

```sql
ALTER DEFAULT PRIVILEGES {granter_clause}IN SCHEMA {_ident_str(schema)}
```

`_seq_relink` line 752

```sql
ALTER SEQUENCE {q} OWNED BY NONE;
ALTER SEQUENCE {q} OWNER TO {_ident_str(new_owner)};
ALTER SEQUENCE {q} OWNED BY {col};
```

`_schema_owner_plan` line 830

```sql
ALTER TABLE {tq} OWNER TO {_ident_str(o.owner)};
```

`_schema_owner_plan` line 831

```sql
ALTER TABLE {tq} OWNER TO {_ident_str(new_owner)};
```

`api_schema_post_import_check` line 1082

```sql
SELECT n.nspname, c.relname, con.conname, con.contype::text
  FROM pg_constraint con
  JOIN pg_class c ON c.oid = con.conrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE NOT con.convalidated {where_ns}
 ORDER BY 1, 2, 3
```

**fix** - `api_schema_post_import_check` line 1095

```sql
ALTER TABLE {_qq(ns, rel)} VALIDATE CONSTRAINT {_qi(con)};
```

`api_schema_post_import_check` line 1097

```sql
SELECT n.nspname, c.relname, ic.relname
  FROM pg_index i
  JOIN pg_class ic ON ic.oid = i.indexrelid
  JOIN pg_class c ON c.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE NOT i.indisvalid {where_ns}
 ORDER BY 1, 2, 3
```

**fix** - `api_schema_post_import_check` line 1111

```sql
REINDEX INDEX {_reindex_conc}{_qq(ns, idx)};  -- or DROP INDEX {_qq(ns, idx)};
```

`api_schema_post_import_check` line 1113

```sql
SELECT n.nspname, c.relname, t.tgname
  FROM pg_trigger t
  JOIN pg_class c ON c.oid = t.tgrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE t.tgenabled = 'D' AND NOT t.tgisinternal {where_ns}
 ORDER BY 1, 2, 3
```

**fix** - `api_schema_post_import_check` line 1126

```sql
ALTER TABLE {_qq(ns, rel)} ENABLE TRIGGER {_qi(tg)};
```

`api_schema_post_import_check` line 1132

```sql
SELECT c.relname, con.conname, fn.nspname, fc.relname
  FROM pg_constraint con
  JOIN pg_class c ON c.oid = con.conrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_class fc ON fc.oid = con.confrelid
  JOIN pg_namespace fn ON fn.oid = fc.relnamespace
 WHERE con.contype = 'f' AND n.nspname = %s AND fn.nspname <> n.nspname
 ORDER BY 1, 2
```

## api/schema/analysis.py

`api_schema_storage` line 84

```sql
SELECT a.attnum,
       a.attname                                              AS name,
       pg_catalog.format_type(a.atttypid, a.atttypmod)       AS type,
       a.attstorage                                           AS storage,
       t.typstorage                                           AS storage_default,
       CASE WHEN t.typlen = -1 THEN 'variable'
            WHEN t.typlen = -2 THEN 'c-string'
            ELSE t.typlen::text || ' B'
       END                                                    AS typlen,
       a.attnotnull                                           AS not_null,
       t.typname                                              AS typname,
       a.atttypmod                                            AS typmod
FROM pg_attribute a
JOIN pg_type t ON t.oid = a.atttypid
WHERE a.attrelid = %s::regclass
  AND a.attnum > 0
  AND NOT a.attisdropped
ORDER BY a.attnum
```

`api_schema_storage` line 117

```sql
SELECT GREATEST(c.reltuples, 0)::bigint AS n FROM pg_class c WHERE c.oid = %s::regclass
```

`api_schema_storage` line 125

```sql
SELECT count(*) AS n, count({qc}) AS n_values,
       avg(pg_column_size({qc}))::bigint AS avg_bytes,
       {toast_expr} AS in_toast
  FROM (SELECT {qc} FROM {qname} LIMIT 1000) s
```

`api_schema_storage` line 149

```sql
SELECT %s::regclass::oid::bigint AS oid
```

`api_schema_storage` line 159

```sql
SELECT c.reltoastrelid                                        AS toast_oid,
       t.relname                                              AS toast_name,
       pg_size_pretty(pg_relation_size(c.oid))                AS table_size,
       CASE WHEN c.reltoastrelid <> 0
            THEN pg_size_pretty(pg_relation_size(c.reltoastrelid))
            ELSE NULL
       END                                                    AS toast_heap_pretty,
       CASE WHEN c.reltoastrelid <> 0
            THEN pg_size_pretty(pg_total_relation_size(c.reltoastrelid))
            ELSE NULL
       END                                                    AS toast_total_pretty,
       CASE WHEN c.reltoastrelid <> 0
            THEN pg_relation_size(c.reltoastrelid)
            ELSE 0
       END                                                    AS toast_heap_bytes
FROM pg_class c
LEFT JOIN pg_class t ON t.oid = c.reltoastrelid
WHERE c.oid = %s::regclass
```

`api_schema_storage` line 186

```sql
SELECT last_vacuum,
       last_autovacuum,
       last_analyze,
       last_autoanalyze,
       n_live_tup,
       n_dead_tup
FROM pg_stat_all_tables
WHERE relid = %s
```

`api_schema_indexes` line 257

```sql
SELECT
  s.relname AS tablica, s.relid::bigint AS relid, {ins_col}
  s.seq_scan, s.seq_tup_read,
  s.idx_scan, s.idx_tup_fetch,
  s.n_live_tup, s.n_dead_tup, s.n_mod_since_analyze,
  -- -1 = never analysed (PG14+); the planner treats it as unknown
  -- and so should the display.
  NULLIF(c.reltuples, -1)::bigint AS planner_tuples,
  pg_size_pretty(pg_relation_size(s.relid)) AS table_size,
  pg_size_pretty(pg_indexes_size(s.relid)) AS indexes_size,
  s.last_vacuum, s.last_autovacuum, s.last_analyze, s.last_autoanalyze,
  {tbl_reset_cols}
  extract(epoch FROM (now() - LEAST(s.last_vacuum, s.last_autovacuum,
                                    s.last_analyze, s.last_autoanalyze)))::float8 AS oldest_mark_s,
  s.vacuum_count, s.autovacuum_count, s.analyze_count, s.autoanalyze_count,
  s.n_tup_ins, s.n_tup_upd, s.n_tup_del, s.n_tup_hot_upd,
  c.relpages::bigint AS relpages, c.relallvisible::bigint AS relallvisible, {frozen_col}
  c.reltuples::float8 AS reltuples_raw,
  (SELECT coalesce(sum(ic.relpages), 0)::bigint FROM pg_index x
     JOIN pg_class ic ON ic.oid = x.indexrelid WHERE x.indrelid = c.oid) AS index_relpages,
  {time_cols}
  extract(epoch FROM (now() - GREATEST(coalesce(s.last_autovacuum, 'epoch'::timestamptz),
                                       coalesce(s.last_vacuum, 'epoch'::timestamptz))))::bigint
      AS sec_since_vacuum,
  age(c.relfrozenxid)::bigint AS xid_age,
  coalesce(c.reloptions, ARRAY[]::text[]) AS reloptions
FROM pg_stat_user_tables s
JOIN pg_class c ON c.oid = s.relid
WHERE s.relid = %s::regclass
```

`api_schema_indexes` line 304

```sql
SELECT name, setting
  FROM pg_settings
 WHERE name IN ('autovacuum',
                'autovacuum_vacuum_scale_factor',
                'autovacuum_vacuum_threshold',
                'autovacuum_analyze_scale_factor',
                'autovacuum_analyze_threshold',
                'autovacuum_vacuum_insert_scale_factor',
                'autovacuum_vacuum_insert_threshold',
                'autovacuum_vacuum_cost_delay',
                'autovacuum_vacuum_cost_limit',
                'autovacuum_naptime',
                'autovacuum_max_workers',
                'vacuum_cost_delay',
                'vacuum_cost_limit',
                'vacuum_freeze_table_age',
                'autovacuum_freeze_max_age',
                'autovacuum_vacuum_max_threshold',
                'autovacuum_worker_slots',
                'vacuum_cost_page_hit',
                'vacuum_cost_page_miss',
                'vacuum_cost_page_dirty',
                'full_page_writes',
                'wal_compression',
                'checkpoint_timeout')
```

`api_schema_indexes` line 458

```sql
SELECT
  i.relname AS index_name,
  pg_get_indexdef(x.indexrelid) AS def,
  pg_size_pretty(pg_relation_size(x.indexrelid)) AS size,
  s.idx_scan AS scans,
  s.idx_tup_read AS tup_read,
  s.idx_tup_fetch AS tup_fetch,
  x.indisprimary AS is_pk,
  x.indisunique AS is_unique,
  x.indisvalid AS is_valid,
  (SELECT string_agg(att.attname, ', ' ORDER BY u.ord)
     FROM unnest(x.indkey::int[]) WITH ORDINALITY u(attnum, ord)
     LEFT JOIN pg_attribute att ON att.attrelid=x.indrelid AND att.attnum=u.attnum
  ) AS kolone,
  EXISTS (
    SELECT 1 FROM pg_constraint c
    WHERE c.contype='f' AND c.conrelid=x.indrelid AND x.indkey[0] = ANY(c.conkey)
  ) AS pokriva_fk
FROM pg_index x
JOIN pg_class i ON i.oid = x.indexrelid
LEFT JOIN pg_stat_user_indexes s ON s.indexrelid = x.indexrelid
WHERE x.indrelid = %s::regclass
ORDER BY x.indisprimary DESC, s.idx_scan DESC NULLS LAST
```

`(module)` line 569

```sql
CREATE INDEX
```

`run_maint` line 686

```sql
SELECT pg_backend_pid()
```

`run_maint` line 700

```sql
SELECT current_setting('server_version_num')::int
```

`run_maint` line 758

```sql
SELECT n.nspname, c.relname
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relkind IN ({kinds}) {persist}
ORDER BY c.relname
```

`run_maint.build_sql` line 874

```sql
VACUUM (ONLY_DATABASE_STATS)
```

`api_schema_active_on_table` line 1062

```sql
SELECT c.oid::int AS oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname = %s LIMIT 1
```

`api_schema_active_on_table` line 1077

```sql
SELECT a.pid,
       a.datname                        AS database,
       a.usename                        AS "user",
       a.application_name               AS app,
       coalesce(a.client_addr::text,'local') AS client,
       a.state,
```

`api_schema_active_on_table` line 1101

```sql
SELECT a.pid,
       a.datname                        AS database,
       a.usename                        AS "user",
       a.application_name               AS app,
       coalesce(a.client_addr::text,'local') AS client,
       a.state,
       {_f['wait_expr']} AS wait,
       EXTRACT(EPOCH FROM now() - a.query_start) AS elapsed_s,
       EXTRACT(EPOCH FROM now() - a.xact_start)  AS tx_age_s,
       regexp_replace(a.query, '[[:space:]]+', ' ', 'g') AS query
FROM pg_stat_activity a
WHERE a.pid <> pg_backend_pid()
  AND a.state IS NOT NULL AND a.state <> 'idle'
  AND a.query ~* (%s)
```

`api_schema_deep_analysis` line 1273

```sql
SELECT 1 FROM pg_extension WHERE extname='pg_stat_statements'
```

`api_schema_deep_analysis` line 1289

```sql
SELECT s.queryid::text AS qid,
       s.calls,
       round(s.total_exec_time::numeric, 1) AS total_ms,
       round(s.mean_exec_time::numeric, 2) AS mean_ms,
       s.rows AS rows_total,
       s.query
FROM {_pss} s
WHERE s.query ~* %s
ORDER BY s.total_exec_time DESC
LIMIT 15
```

`api_schema_deep_analysis` line 1306

```sql
SELECT a.attname AS kolona,
       pg_catalog.format_type(a.atttypid, a.atttypmod) AS tip,
       a.attnotnull AS not_null,
       ps.n_distinct,
       ps.null_frac,
       ps.correlation,
       array_to_string(ps.most_common_vals, ', ') AS mcv_sample,
       EXISTS (
         SELECT 1 FROM pg_index i
         WHERE i.indrelid = a.attrelid
           AND a.attnum = ANY(i.indkey)
           AND i.indkey[0] = a.attnum
       ) AS indexed_lead
FROM pg_attribute a
LEFT JOIN pg_stats ps ON ps.schemaname=%s AND ps.tablename=%s AND ps.attname=a.attname
WHERE a.attrelid = %s::regclass AND a.attnum > 0 AND NOT a.attisdropped
ORDER BY a.attnum
```

`api_schema_deep_analysis` line 1330

```sql
SELECT a.attname AS kolona
FROM pg_constraint con
JOIN pg_attribute a ON a.attrelid = con.conrelid AND a.attnum = ANY(con.conkey)
WHERE con.contype='f' AND con.conrelid = %s::regclass
  AND NOT EXISTS (
      SELECT 1 FROM pg_index i
      WHERE i.indrelid = con.conrelid
        AND a.attnum = ANY(i.indkey)
        AND i.indkey[0] = a.attnum
  )
ORDER BY a.attnum
```

**create_sql** - `api_schema_deep_analysis` line 1349

```sql
CREATE INDEX CONCURRENTLY idx_{name}_{col} ON "{schema}"."{name}" ("{col}");
```

`api_schema_deep_analysis` line 1373

```sql
SELECT coalesce(n_live_tup, 0) AS n FROM pg_stat_user_tables WHERE relid=%s::regclass
```

**create_sql** - `api_schema_deep_analysis` line 1401

```sql
CREATE INDEX CONCURRENTLY idx_{name}_{c} ON "{schema}"."{name}" ("{c}");
```

`api_schema_deep_analysis` line 1409

```sql
SELECT
  s.n_live_tup, s.n_dead_tup, s.n_mod_since_analyze,
  s.last_vacuum, s.last_autovacuum, s.last_analyze, s.last_autoanalyze,
  s.vacuum_count, s.autovacuum_count, s.analyze_count, s.autoanalyze_count,
  round(100.0 * s.n_dead_tup::numeric /
        nullif(s.n_live_tup + s.n_dead_tup, 0), 1) AS dead_pct,
  EXTRACT(EPOCH FROM (now() - coalesce(greatest(s.last_analyze, s.last_autoanalyze), 'epoch'::timestamp))) AS analyze_age_s,
  EXTRACT(EPOCH FROM (now() - coalesce(greatest(s.last_vacuum, s.last_autovacuum), 'epoch'::timestamp))) AS vacuum_age_s,
  pg_size_pretty(pg_relation_size(s.relid)) AS table_size,
  pg_relation_size(s.relid) AS table_bytes,
  pg_size_pretty(pg_total_relation_size(s.relid)) AS total_size,
  (SELECT setting FROM pg_settings WHERE name='autovacuum_vacuum_threshold') AS av_vac_thr,
  (SELECT setting FROM pg_settings WHERE name='autovacuum_vacuum_scale_factor') AS av_vac_sf,
  (SELECT setting FROM pg_settings WHERE name='autovacuum_analyze_threshold') AS av_ana_thr,
  (SELECT setting FROM pg_settings WHERE name='autovacuum_analyze_scale_factor') AS av_ana_sf,
  (SELECT setting FROM pg_settings WHERE name='autovacuum_vacuum_max_threshold') AS av_vac_max,
  (SELECT reltuples FROM pg_class WHERE oid=s.relid)::float8 AS reltuples,
  (SELECT array_agg(opt) FROM unnest(
        coalesce((SELECT reloptions FROM pg_class WHERE oid=s.relid), ARRAY[]::text[])
   ) AS opt WHERE opt LIKE 'autovacuum%%' OR opt LIKE 'toast.autovacuum%%') AS per_table_opts
FROM pg_stat_user_tables s
WHERE s.relid = %s::regclass
```

**create_sql** - `api_schema_deep_analysis` line 1495

```sql
VACUUM (VERBOSE, ANALYZE) "{schema}"."{name}";
```

**create_sql** - `api_schema_deep_analysis` line 1503

```sql
VACUUM (ANALYZE) "{schema}"."{name}";
```

**create_sql** - `api_schema_deep_analysis` line 1512

```sql
ANALYZE "{schema}"."{name}";
```

**create_sql** - `api_schema_deep_analysis` line 1529

```sql
ANALYZE "{schema}"."{name}";
```

**create_sql** - `api_schema_deep_analysis` line 1541

```sql
ANALYZE "{schema}"."{name}";
```

`api_schema_index_advisor` line 1655

```sql
SELECT n.nspname                     AS schema,
       t.relname                     AS "table",
       i.relname                     AS index,
       am.amname                     AS method,
       pg_get_indexdef(x.indexrelid) AS def,
       x.indisprimary, x.indisunique, x.indisvalid, x.indisready,
       x.indisreplident,
       x.indpred IS NOT NULL         AS is_partial,
       x.indexprs IS NOT NULL        AS has_expression,
       x.indnatts, {_indnkeyatts} AS indnkeyatts,
       pg_relation_size(x.indexrelid)  AS index_bytes,
       pg_relation_size(x.indrelid)    AS table_bytes,
       pg_size_pretty(pg_relation_size(x.indexrelid)) AS index_size,
       pg_size_pretty(pg_relation_size(x.indrelid))   AS table_size,
       COALESCE(ts.spcname, '')      AS tablespace,
       c.conname                     AS constraint_name,
       c.contype                     AS constraint_type,
       COALESCE(s.idx_scan, 0)       AS idx_scan,
       COALESCE(s.idx_tup_read, 0)   AS idx_tup_read,
       COALESCE(s.idx_tup_fetch, 0)  AS idx_tup_fetch,
       NULLIF(t.reltuples, -1)::float8 AS tbl_reltuples, t.relpages::bigint AS tbl_relpages,
       (pg_relation_size(t.oid) / current_setting('block_size')::int)::bigint AS tbl_pages,
       COALESCE(tst.n_live_tup, 0)   AS tbl_live,
       COALESCE(tst.n_mod_since_analyze, 0) AS tbl_mod,
       (SELECT string_agg(att.attname, ', ' ORDER BY u.ord)
          FROM unnest(x.indkey::int[]) WITH ORDINALITY u(attnum, ord)
          LEFT JOIN pg_attribute att
                 ON att.attrelid = x.indrelid AND att.attnum = u.attnum
        ) AS columns,
       x.indkey::text                AS indkey
  FROM pg_index x
  JOIN pg_class     i  ON i.oid = x.indexrelid
  JOIN pg_class     t  ON t.oid = x.indrelid
  JOIN pg_namespace n  ON n.oid = i.relnamespace
  JOIN pg_am        am ON am.oid = i.relam
  LEFT JOIN pg_tablespace ts ON ts.oid = i.reltablespace
  LEFT JOIN pg_constraint c  ON c.conindid = x.indexrelid
  LEFT JOIN pg_stat_user_indexes s ON s.indexrelid = x.indexrelid
  LEFT JOIN pg_stat_user_tables tst ON tst.relid = x.indrelid
 WHERE n.nspname = %s AND i.relname = %s
```

`api_schema_index_advisor` line 1706

```sql
SELECT stats_reset FROM pg_stat_database WHERE datname = current_database()
```

`api_schema_index_advisor` line 1711

```sql
SELECT i.relname AS name, pg_get_indexdef(x.indexrelid) AS def,
       am.amname AS method, x.indkey::text AS indkey,
       x.indisprimary, x.indisunique,
       pg_size_pretty(pg_relation_size(x.indexrelid)) AS size,
       COALESCE(s.idx_scan, 0) AS idx_scan
  FROM pg_index x
  JOIN pg_class     i  ON i.oid = x.indexrelid
  JOIN pg_namespace n  ON n.oid = i.relnamespace
  JOIN pg_am        am ON am.oid = i.relam
  LEFT JOIN pg_stat_user_indexes s ON s.indexrelid = x.indexrelid
 WHERE x.indrelid = (SELECT indrelid FROM pg_index WHERE indexrelid =
                      format('%%I.%%I', %s::text, %s::text)::regclass)
   AND i.relname <> %s
```

`_index_findings` line 1880

```sql
REINDEX INDEX CONCURRENTLY %s;
```

`_index_findings` line 1886

```sql
REINDEX INDEX %s;
```

`_index_findings` line 1897

```sql
DROP INDEX CONCURRENTLY
```

`_index_findings` line 1898

```sql
DROP INDEX CONCURRENTLY %s;
```

## api/schema/data.py

`api_schema_row_modify` line 69

```sql
SELECT a.attname,
       {gen_col} AS generated
  FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
   AND a.attnum > 0 AND NOT a.attisdropped
```

`api_schema_row_modify` line 83

```sql
SELECT a.attname FROM pg_index i
  JOIN pg_class c ON c.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum = ANY(i.indkey)
 WHERE i.indisprimary AND n.nspname = %s AND c.relname = %s
```

`api_schema_row_modify` line 125

```sql
INSERT INTO {s}.{t} (
```

`api_schema_row_modify` line 135

```sql
INSERT INTO {s}.{t} DEFAULT VALUES RETURNING *
```

`api_schema_row_modify` line 168

```sql
DELETE FROM {s}.{t} WHERE
```

`api_schema_distinct` line 227

```sql
SELECT 1 FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s AND a.attname = %s
   AND a.attnum > 0 AND NOT a.attisdropped
```

`api_schema_distinct` line 237

```sql
SELECT v::text AS v, count(*)::bigint AS c   FROM (SELECT {col} AS v FROM {sch}.{tbl} LIMIT {n}) s  GROUP BY v  ORDER BY c DESC NULLS LAST, v  LIMIT {l}
```

`api_schema_sample` line 321

```sql
SELECT a.attname
FROM pg_attribute a
JOIN pg_class c     ON c.oid = a.attrelid
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = %s AND c.relname = %s
  AND a.attnum > 0 AND NOT a.attisdropped
```

**wrapped** - `_sample_raw` line 401

```sql
SELECT * FROM (
{stmt}
) AS dbapguin_raw{'' if limit is None else ' LIMIT %d' % limit}
```

`api_schema_copy_table` line 514

```sql
SELECT to_regclass(%s) IS NOT NULL
```

`api_schema_copy_table` line 518

```sql
SELECT to_regclass(%s) IS NOT NULL
```

`api_schema_copy_table` line 529

```sql
SELECT a.attname, %s, %s
  FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %%s AND c.relname = %%s
   AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

**ddl** - `api_schema_copy_table` line 544

```sql
CREATE TABLE {dst_q} (LIKE {src_q} INCLUDING ALL)
```

`api_schema_copy_table` line 553

```sql
SELECT %s::regclass::oid
```

`api_schema_copy_table` line 570

```sql
SELECT set_config('dbapguin.anon_salt', %s, true)
```

**data_sql** - `api_schema_copy_table` line 604

```sql
INSERT INTO {dst_q} ({dst_cols_sql}){_ovr} SELECT {select_sql} FROM {from_sql}
```

`api_schema_copy_table` line 620

```sql
SET LOCAL row_security = off
```

`api_schema_copy_table` line 635

```sql
RESET row_security
```

`api_schema_copy_table` line 643

```sql
SELECT quote_ident(n.nspname) || '.' || quote_ident(ic.relname)
  FROM pg_index i
  JOIN pg_class ic    ON ic.oid = i.indexrelid
  JOIN pg_namespace n ON n.oid = ic.relnamespace
  JOIN pg_am am       ON am.oid = ic.relam
 WHERE i.indrelid = %s::regclass AND am.amname = 'ivfflat'
 ORDER BY 1
```

**stmt** - `api_schema_copy_table` line 653

```sql
REINDEX INDEX {ix}
```

## api/schema/ddl.py

`api_schema_ddl` line 123

```sql
SELECT {_part_col} AS is_partition,
       {_partkeydef_expr} AS partkeydef,
       {_partbound_expr} AS partbound,
       (SELECT pn.nspname || '.' || pc.relname
          FROM pg_inherits i JOIN pg_class pc ON pc.oid = i.inhparent
          JOIN pg_namespace pn ON pn.oid = pc.relnamespace
         WHERE i.inhrelid = c.oid LIMIT 1) AS parent_fqn
  FROM pg_class c WHERE c.oid = %s::regclass
```

**head** - `api_schema_ddl` line 151

```sql
CREATE TABLE {qname} PARTITION OF {pq} {pinfo.get('partbound') or 'DEFAULT'}
```

`api_schema_ddl` line 157

```sql
SELECT a.attname,
       pg_catalog.format_type(a.atttypid, a.atttypmod) AS type,
       a.attnotnull,
       pg_get_expr(d.adbin, d.adrelid) AS default_expr
FROM pg_attribute a
LEFT JOIN pg_attrdef d ON d.adrelid=a.attrelid AND d.adnum=a.attnum
WHERE a.attrelid = %s::regclass AND a.attnum > 0 AND NOT a.attisdropped
ORDER BY a.attnum
```

`api_schema_ddl` line 180

```sql
CREATE {kw} {qname} (
{',\n'.join(col_lines)}
){tail};
```

`api_schema_ddl` line 186

```sql
SELECT conname, pg_get_constraintdef(oid) AS def
FROM pg_constraint WHERE conrelid = %s::regclass{_con_filter}
ORDER BY contype, conname
```

`api_schema_ddl` line 193

```sql
ALTER TABLE {qname} ADD CONSTRAINT {_qi(c['conname'])} {c['def']};
```

`api_schema_ddl` line 197

```sql
SELECT ic.relname AS indexname, pg_get_indexdef(x.indexrelid) AS indexdef
  FROM pg_index x
  JOIN pg_class ic ON ic.oid = x.indexrelid
 WHERE x.indrelid = %s::regclass
   AND NOT EXISTS (SELECT 1 FROM pg_constraint co WHERE co.conindid = x.indexrelid)
   AND NOT EXISTS (SELECT 1 FROM pg_inherits pi WHERE pi.inhrelid = x.indexrelid)
 ORDER BY ic.relname
```

`api_schema_ddl` line 209

```sql
SELECT pg_get_viewdef(%s::regclass, true) AS def
```

`api_schema_ddl` line 210

```sql
CREATE OR REPLACE VIEW {qname} AS
{cur.fetchone()['def']}
```

`api_schema_ddl` line 212

```sql
SELECT pg_get_viewdef(%s::regclass, true) AS def
```

`api_schema_ddl` line 213

```sql
CREATE MATERIALIZED VIEW {qname} AS
{cur.fetchone()['def']}
```

`api_schema_ddl` line 216

```sql
SELECT 'CREATE SEQUENCE ' || quote_ident(schemaname) || '.' || quote_ident(sequencename)
    || ' AS ' || data_type
    || ' START ' || start_value
    || ' INCREMENT ' || increment_by
    || ' MINVALUE ' || min_value
    || ' MAXVALUE ' || max_value
    || CASE WHEN cycle THEN ' CYCLE' ELSE '' END
    || ';' AS def
FROM pg_sequences WHERE schemaname=%s AND sequencename=%s
```

`api_schema_ddl` line 234

```sql
SELECT start_value, increment_by, min_value, max_value, is_cycled FROM "{schema}"."{name}"
```

`api_schema_ddl` line 239

```sql
CREATE SEQUENCE "{schema}"."{name}" START {r['start_value']} INCREMENT {r['increment_by']} MINVALUE {r['min_value']} MAXVALUE {r['max_value']}{' CYCLE' if r['is_cycled'] else ''};
```

`api_schema_ddl` line 250

```sql
SELECT pg_get_functiondef(%s::oid) AS def
```

`api_schema_ddl` line 258

```sql
SELECT pg_get_triggerdef(%s::oid, true) AS def
```

`api_schema_ddl` line 266

```sql
SELECT t.typname, t.typtype, n.nspname, t.typbasetype,
       pg_catalog.format_type(t.typbasetype, t.typtypmod) AS basetype,
       (SELECT array_agg(enumlabel ORDER BY enumsortorder)
        FROM pg_enum WHERE enumtypid = t.oid) AS enum_labels
FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace
WHERE t.oid = %s::oid
```

`api_schema_ddl` line 280

```sql
CREATE TYPE "{r['nspname']}"."{r['typname']}" AS ENUM ({lits});
```

`api_schema_ddl` line 282

```sql
CREATE DOMAIN "{r['nspname']}"."{r['typname']}" AS {r['basetype']};
```

`api_schema_ddl` line 284

```sql
SELECT a.attname, pg_catalog.format_type(a.atttypid, a.atttypmod) AS type
FROM pg_attribute a
JOIN pg_type t ON t.typrelid = a.attrelid
WHERE t.oid = %s::oid AND a.attnum > 0 AND NOT a.attisdropped
ORDER BY a.attnum
```

`api_schema_ddl` line 293

```sql
CREATE TYPE "{r['nspname']}"."{r['typname']}" AS (
{lines}
);
```

`api_schema_ddl` line 295

```sql
SELECT extname, extversion FROM pg_extension WHERE extname=%s
```

`api_schema_ddl` line 298

```sql
-- extension installed: {r['extname']} {r['extversion']}
CREATE EXTENSION IF NOT EXISTS "{r['extname']}";
```

`api_schema_ddl` line 308

```sql
SELECT pg_get_indexdef(c.oid) AS def
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE c.relkind IN ('i','I') AND n.nspname = %s AND c.relname = %s
```

`api_schema_ddl` line 320

```sql
SELECT pg_get_ruledef(r.oid, true) AS def
  FROM pg_rewrite r
  JOIN pg_class c ON c.oid = r.ev_class
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND r.rulename = %s
   AND (%s = '' OR c.relname = %s)
```

`api_schema_ddl` line 332

```sql
SELECT pg_get_statisticsobjdef(st.oid) AS def
  FROM pg_statistic_ext st JOIN pg_namespace n ON n.oid = st.stxnamespace
 WHERE n.nspname = %s AND st.stxname = %s
```

`api_schema_ddl` line 341

```sql
SELECT pg_catalog.format_type(t.typbasetype, t.typtypmod) AS basetype,
       t.typnotnull, t.typdefault,
       co.collname, cn.nspname AS collschema,
       (SELECT array_agg(conname || ' ' || pg_get_constraintdef(oid)
                         ORDER BY conname)
          FROM pg_constraint WHERE contypid = t.oid) AS checks
  FROM pg_type t
  JOIN pg_namespace n ON n.oid = t.typnamespace
  LEFT JOIN pg_collation co ON co.oid = t.typcollation
  LEFT JOIN pg_namespace cn ON cn.oid = co.collnamespace
 WHERE t.typtype = 'd' AND n.nspname = %s AND t.typname = %s
```

**sql_txt** - `api_schema_ddl` line 356

```sql
CREATE DOMAIN {qname} AS {r['basetype']}
```

`api_schema_ddl` line 373

```sql
SELECT pg_get_functiondef(%s::oid) AS def
```

`api_schema_ddl` line 378

```sql
SELECT p.proname,
       pg_catalog.format_type(a.aggtranstype, NULL) AS stype,
       a.aggtransfn::regproc::text  AS sfunc,
       NULLIF(a.aggfinalfn, 0)::regproc::text AS finalfunc,
       a.agginitval,
       pg_get_function_arguments(p.oid) AS args
  FROM pg_aggregate a
  JOIN pg_proc p      ON p.oid = a.aggfnoid
  JOIN pg_namespace n ON n.oid = p.pronamespace
 WHERE n.nspname = %s AND p.proname = %s
```

`api_schema_ddl` line 397

```sql
CREATE AGGREGATE {qname}({r['args']}) (
    {',\n    '.join(parts)}
);
```

`api_schema_ddl` line 400

```sql
SELECT collprovider, collcollate, collctype, collisdeterministic
  FROM pg_collation c JOIN pg_namespace n ON n.oid = c.collnamespace
 WHERE n.nspname = %s AND c.collname = %s
```

`api_schema_ddl` line 415

```sql
CREATE COLLATION {qname} ({', '.join(opts)});
```

`api_schema_ddl` line 420

```sql
SELECT tablename, policyname, permissive, roles, cmd, qual, with_check
  FROM pg_policies
 WHERE schemaname = %s AND policyname = %s
   AND (%s = '' OR tablename = %s)
```

**sql_txt** - `api_schema_ddl` line 428

```sql
CREATE POLICY "{r['policyname']}" ON {schema and chr(34) + schema + chr(34) + '.'}"{r['tablename']}"
```

`api_schema_ddl` line 440

```sql
WITH CHECK ({r['with_check']})
```

`api_schema_ddl` line 443

```sql
SELECT evtname, evtevent, evtfoid::regproc::text AS fn, evtenabled, evttags
  FROM pg_event_trigger WHERE evtname = %s
```

**sql_txt** - `api_schema_ddl` line 449

```sql
CREATE EVENT TRIGGER "{r['evtname']}" ON {r['evtevent']}
```

`api_schema_ddl` line 456

```sql
SELECT lanname, lanpltrusted,
       NULLIF(lanplcallfoid,0)::regproc::text AS handler,
       NULLIF(lanvalidator,0)::regproc::text  AS validator
  FROM pg_language WHERE lanname = %s
```

`api_schema_ddl` line 475

```sql
SELECT pg_catalog.format_type(castsource, NULL) AS src,
       pg_catalog.format_type(casttarget, NULL) AS tgt,
       NULLIF(castfunc,0)::regprocedure::text AS fn, castcontext
  FROM pg_cast WHERE oid = %s::oid
```

`api_schema_ddl` line 485

```sql
CREATE CAST ({r['src']} AS {r['tgt']}) {how}{ctx};
```

`api_schema_ddl` line 487

```sql
SELECT conname, condefault,
       pg_encoding_to_char(conforencoding) AS src,
       pg_encoding_to_char(contoencoding)  AS tgt,
       conproc::regproc::text AS fn
  FROM pg_conversion c JOIN pg_namespace n ON n.oid = c.connamespace
 WHERE n.nspname = %s AND c.conname = %s
```

`api_schema_ddl` line 497

```sql
CREATE {'DEFAULT ' if r['condefault'] else ''}CONVERSION {qname} FOR '{r['src']}' TO '{r['tgt']}' FROM {r['fn']};
```

`api_schema_ddl` line 501

```sql
SELECT oprname, oprcode::regproc::text AS fn,
       NULLIF(oprleft,0)::regtype::text  AS lft,
       NULLIF(oprright,0)::regtype::text AS rgt,
       NULLIF(oprcom,0)::regoperator::text AS com,
       NULLIF(oprnegate,0)::regoperator::text AS neg
  FROM pg_operator o JOIN pg_namespace n ON n.oid = o.oprnamespace
 WHERE n.nspname = %s AND o.oprname = %s
```

`api_schema_ddl` line 517

```sql
CREATE OPERATOR "{schema}".{r['oprname']} (
    {',\n    '.join(opts)}
);
```

`api_schema_ddl` line 520

```sql
SELECT t.typname,
       pg_catalog.format_type(r.rngsubtype, NULL) AS subtype,
       NULLIF(r.rngsubopc,0)::regclass::text AS opclass,
       NULLIF(r.rngcollation,0)::regcollation::text AS coll,
       NULLIF(r.rngcanonical,0)::regproc::text AS canonical,
       NULLIF(r.rngsubdiff,0)::regproc::text AS subdiff
  FROM pg_range r
  JOIN pg_type t      ON t.oid = r.rngtypid
  JOIN pg_namespace n ON n.oid = t.typnamespace
 WHERE n.nspname = %s AND t.typname = %s
```

`api_schema_ddl` line 538

```sql
CREATE TYPE {qname} AS RANGE (
    {',\n    '.join(opts)}
);
```

`api_schema_ddl` line 541

```sql
SELECT fdwname, NULLIF(fdwhandler,0)::regproc::text AS handler,
       NULLIF(fdwvalidator,0)::regproc::text AS validator, fdwoptions
  FROM pg_foreign_data_wrapper WHERE fdwname = %s
```

`api_schema_ddl` line 557

```sql
SELECT s.srvname, s.srvtype, s.srvversion, s.srvoptions, w.fdwname
  FROM pg_foreign_server s
  JOIN pg_foreign_data_wrapper w ON w.oid = s.srvfdw
 WHERE s.srvname = %s
```

`api_schema_ddl` line 575

```sql
SELECT pubname, puballtables, pubinsert, pubupdate, pubdelete, pubtruncate
  FROM pg_publication WHERE pubname = %s
```

`api_schema_ddl` line 584

```sql
SELECT schemaname, tablename FROM pg_publication_tables
 WHERE pubname = %s ORDER BY schemaname, tablename
```

**with_txt** - `api_schema_ddl` line 594

```sql
WITH (publish = '{', '.join(ops)}')
```

`api_schema_ddl` line 595

```sql
CREATE PUBLICATION "{r['pubname']}"{target}{with_txt};
```

`api_schema_ddl` line 597

```sql
SELECT subname, subpublications, subenabled, subslotname
  FROM pg_subscription WHERE subname = %s
```

`api_schema_ddl` line 607

```sql
CREATE SUBSCRIPTION "{r['subname']}"
    CONNECTION '<connection string - not exported>'
    PUBLICATION {', '.join((chr(34) + p + chr(34) for p in r['subpublications'] or []))}
    WITH (slot_name = '{(r['subslotname'] or '').replace(chr(39), chr(39) * 2)}', enabled = {'true' if r['subenabled'] else 'false'});
```

`api_schema_ddl` line 624

```sql
SELECT n.nspname, o.opcname, am.amname, o.opcdefault,
       pg_catalog.format_type(o.opcintype, NULL) AS input_type,
       fn.nspname AS family_schema, of.opfname AS family_name,
       CASE WHEN o.opckeytype != 0
            THEN pg_catalog.format_type(o.opckeytype, NULL) END AS storage_type
  FROM pg_opclass o
  JOIN pg_am am ON am.oid = o.opcmethod
  JOIN pg_namespace n ON n.oid = o.opcnamespace
  JOIN pg_opfamily of ON of.oid = o.opcfamily
  JOIN pg_namespace fn ON fn.oid = of.opfnamespace
 WHERE o.oid = %s::oid
```

`api_schema_ddl` line 639

```sql
SELECT amop.amopstrategy, op.oprname,
       pg_catalog.format_type(amop.amoplefttype, NULL) AS lt,
       pg_catalog.format_type(amop.amoprighttype, NULL) AS rt
  FROM pg_depend d
  JOIN pg_amop amop ON amop.oid = d.objid
  JOIN pg_operator op ON op.oid = amop.amopopr
 WHERE d.refclassid = 'pg_opclass'::regclass AND d.refobjid = %s::oid
   AND d.classid = 'pg_amop'::regclass
 ORDER BY amop.amopstrategy
```

`api_schema_ddl` line 652

```sql
SELECT amproc.amprocnum, amproc.amproc::regprocedure AS fn,
       pg_catalog.format_type(amproc.amproclefttype, NULL) AS lt,
       pg_catalog.format_type(amproc.amprocrighttype, NULL) AS rt
  FROM pg_depend d
  JOIN pg_amproc amproc ON amproc.oid = d.objid
 WHERE d.refclassid = 'pg_opclass'::regclass AND d.refobjid = %s::oid
   AND d.classid = 'pg_amproc'::regclass
 ORDER BY amproc.amprocnum
```

`api_schema_ddl` line 675

```sql
CREATE OPERATOR CLASS {_qi(r['nspname'])}.{_qi(r['opcname'])}{default_clause}
    FOR TYPE {r['input_type']} USING {r['amname']}{family_clause} AS
    {',\n    '.join(members)};
```

`api_schema_ddl` line 682

```sql
SELECT n.nspname, opf.opfname, am.amname
  FROM pg_opfamily opf
  JOIN pg_am am ON am.oid = opf.opfmethod
  JOIN pg_namespace n ON n.oid = opf.opfnamespace
 WHERE opf.oid = %s::oid
```

`api_schema_ddl` line 692

```sql
CREATE OPERATOR FAMILY {qname_f} USING {r['amname']};
```

`api_schema_ddl` line 703

```sql
SELECT amop.amopstrategy, op.oprname,
       pg_catalog.format_type(amop.amoplefttype, NULL) AS lt,
       pg_catalog.format_type(amop.amoprighttype, NULL) AS rt
  FROM pg_amop amop JOIN pg_operator op ON op.oid = amop.amopopr
 WHERE amop.amopfamily = %s::oid
   AND NOT EXISTS (
       SELECT 1 FROM pg_depend d
        WHERE d.classid = 'pg_amop'::regclass AND d.objid = amop.oid
          AND d.refclassid = 'pg_opclass'::regclass)
 ORDER BY amop.amoplefttype, amop.amoprighttype, amop.amopstrategy
```

`api_schema_ddl` line 717

```sql
SELECT amproc.amprocnum, amproc.amproc::regprocedure AS fn,
       pg_catalog.format_type(amproc.amproclefttype, NULL) AS lt,
       pg_catalog.format_type(amproc.amprocrighttype, NULL) AS rt
  FROM pg_amproc amproc
 WHERE amproc.amprocfamily = %s::oid
   AND NOT EXISTS (
       SELECT 1 FROM pg_depend d
        WHERE d.classid = 'pg_amproc'::regclass AND d.objid = amproc.oid
          AND d.refclassid = 'pg_opclass'::regclass)
 ORDER BY amproc.amproclefttype, amproc.amprocrighttype, amproc.amprocnum
```

`api_schema_ddl` line 732

```sql
ALTER OPERATOR FAMILY {qname_f} USING {r['amname']} ADD
    {',\n    '.join(add_members)};
```

`api_schema_ddl` line 739

```sql
SELECT n.nspname, c.cfgname, pn.nspname AS parser_schema, p.prsname AS parser_name
  FROM pg_ts_config c
  JOIN pg_namespace n ON n.oid = c.cfgnamespace
  JOIN pg_ts_parser p ON p.oid = c.cfgparser
  JOIN pg_namespace pn ON pn.oid = p.prsnamespace
 WHERE c.oid = %s::oid
```

`api_schema_ddl` line 750

```sql
CREATE TEXT SEARCH CONFIGURATION {qname_c} (PARSER = {_qi(r['parser_schema'])}.{_qi(r['parser_name'])});
```

`api_schema_ddl` line 755

```sql
SELECT tt.alias,
       array_agg(dn.nspname || '.' || d.dictname ORDER BY m.mapseqno) AS dicts
  FROM pg_ts_config_map m
  JOIN pg_ts_dict d ON d.oid = m.mapdict
  JOIN pg_namespace dn ON dn.oid = d.dictnamespace,
  LATERAL ts_token_type((SELECT cfgparser FROM pg_ts_config WHERE oid = %s::oid)) tt
 WHERE m.mapcfg = %s::oid AND tt.tokid = m.maptokentype
 GROUP BY tt.alias, tt.tokid
 ORDER BY tt.tokid
```

`api_schema_ddl` line 768

```sql
ALTER TEXT SEARCH CONFIGURATION {qname_c} ADD MAPPING FOR {m['alias']} WITH {dicts};
```

`api_schema_ddl` line 775

```sql
SELECT n.nspname, d.dictname, tn.nspname AS tmpl_schema, t.tmplname AS tmpl_name,
       d.dictinitoption
  FROM pg_ts_dict d
  JOIN pg_namespace n ON n.oid = d.dictnamespace
  JOIN pg_ts_template t ON t.oid = d.dicttemplate
  JOIN pg_namespace tn ON tn.oid = t.tmplnamespace
 WHERE d.oid = %s::oid
```

`api_schema_ddl` line 791

```sql
CREATE TEXT SEARCH DICTIONARY {_qi(r['nspname'])}.{_qi(r['dictname'])} (TEMPLATE = {_qi(r['tmpl_schema'])}.{_qi(r['tmpl_name'])}{opts});
```

`api_schema_ddl` line 797

```sql
SELECT n.nspname, p.prsname,
       p.prsstart::regprocedure AS start_fn,
       p.prstoken::regprocedure AS token_fn,
       p.prsend::regprocedure AS end_fn,
       p.prslextype::regprocedure AS lextypes_fn,
       NULLIF(p.prsheadline,0)::regprocedure AS headline_fn
  FROM pg_ts_parser p JOIN pg_namespace n ON n.oid = p.prsnamespace
 WHERE p.oid = %s::oid
```

`api_schema_ddl` line 825

```sql
CREATE TEXT SEARCH PARSER {_qi(r['nspname'])}.{_qi(r['prsname'])} (
    {',\n    '.join(lines)}
);
```

`api_schema_ddl` line 831

```sql
SELECT n.nspname, t.tmplname,
       NULLIF(t.tmplinit,0)::regprocedure AS init_fn,
       t.tmpllexize::regprocedure AS lexize_fn
  FROM pg_ts_template t JOIN pg_namespace n ON n.oid = t.tmplnamespace
 WHERE t.oid = %s::oid
```

`api_schema_ddl` line 846

```sql
CREATE TEXT SEARCH TEMPLATE {_qi(r['nspname'])}.{_qi(r['tmplname'])} (
    {',\n    '.join(lines)}
);
```

`api_schema_ddl` line 852

```sql
SELECT um.umuser, pg_catalog.pg_get_userbyid(um.umuser) AS user_name,
       s.srvname, um.umoptions
  FROM pg_user_mapping um
  JOIN pg_foreign_server s ON s.oid = um.umserver
 WHERE um.oid = %s::oid
```

`api_schema_ddl` line 874

```sql
CREATE USER MAPPING FOR {who} SERVER {_qi(r['srvname'])}{opts};
```

`api_schema_ddl` line 886

```sql
SELECT DISTINCT s.relname AS seq_name,
       n.nspname       AS seq_schema,
       a.attname       AS col_name
  FROM pg_class s
  JOIN pg_namespace n   ON n.oid = s.relnamespace
  JOIN pg_depend d      ON d.objid = s.oid AND d.classid = 'pg_class'::regclass
  JOIN pg_class t       ON t.oid = d.refobjid
  JOIN pg_namespace tn  ON tn.oid = t.relnamespace
  JOIN pg_attribute a   ON a.attrelid = t.oid AND a.attnum = d.refobjsubid
 WHERE s.relkind = 'S' AND d.deptype = 'a'
   AND tn.nspname = %s AND t.relname = %s
 ORDER BY 1
```

`api_schema_ddl` line 909

```sql
CREATE SEQUENCE IF NOT EXISTS {fq};
```

`api_schema_ddl` line 912

```sql
ALTER SEQUENCE {fq} OWNED BY {qname}.{_qi(sq['col_name'])};
```

**_DEPEND_OBJECTS_BASE** - `(module)` line 1035

```sql
SELECT 'pg_class'::regclass AS cls, c.oid AS id, c.relnamespace AS ns,
       c.relname::text AS name
  FROM pg_catalog.pg_class c
UNION ALL
SELECT 'pg_constraint'::regclass, co.oid, co.connamespace,
       co.conname::text
  FROM pg_catalog.pg_constraint co
UNION ALL
SELECT 'pg_attrdef'::regclass, ad.oid, c.relnamespace,
       (c.relname || '.' || a.attname)::text
  FROM pg_catalog.pg_attrdef ad
  JOIN pg_catalog.pg_class c ON c.oid = ad.adrelid
  JOIN pg_catalog.pg_attribute a
    ON a.attrelid = ad.adrelid AND a.attnum = ad.adnum
UNION ALL
SELECT 'pg_proc'::regclass, p.oid, p.pronamespace, p.proname::text
  FROM pg_catalog.pg_proc p
UNION ALL
SELECT 'pg_type'::regclass, t.oid, t.typnamespace, t.typname::text
  FROM pg_catalog.pg_type t
UNION ALL
SELECT 'pg_rewrite'::regclass, r.oid, c.relnamespace, c.relname::text
  FROM pg_catalog.pg_rewrite r
  JOIN pg_catalog.pg_class c ON c.oid = r.ev_class
UNION ALL
SELECT 'pg_trigger'::regclass, tg.oid, c.relnamespace,
       (c.relname || '.' || tg.tgname)::text
  FROM pg_catalog.pg_trigger tg
  JOIN pg_catalog.pg_class c ON c.oid = tg.tgrelid
```

`_depends_on_sql` line 1084

```sql
WITH obj AS (%s)
SELECT DISTINCT
       d.classid::regclass::text AS dep_class,
       o.name  AS dep_name,
       d.refclassid::regclass::text AS ref_class,
       ro.name AS ref_name
  FROM pg_catalog.pg_depend d
  JOIN obj o  ON o.cls  = d.classid    AND o.id  = d.objid
  JOIN obj ro ON ro.cls = d.refclassid AND ro.id = d.refobjid
  JOIN pg_catalog.pg_namespace dn ON dn.oid = o.ns
  JOIN pg_catalog.pg_namespace rn ON rn.oid = ro.ns
 WHERE dn.nspname = %%s AND rn.nspname = %%s
 ORDER BY 1, 2, 3, 4
```

`(module)` line 1106

```sql
SELECT c.conname AS name,
                             pg_catalog.pg_get_constraintdef(c.oid) AS def
                        FROM pg_catalog.pg_constraint c
                        JOIN pg_catalog.pg_namespace n ON n.oid = c.connamespace
                       WHERE n.nspname = %s
```

`(module)` line 1111

```sql
SELECT c.relname || '.' || a.attname AS name,
                                 pg_catalog.pg_get_expr(d.adbin, d.adrelid) AS def
                            FROM pg_catalog.pg_attrdef d
                            JOIN pg_catalog.pg_class c ON c.oid = d.adrelid
                            JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
                            JOIN pg_catalog.pg_attribute a
                              ON a.attrelid = d.adrelid AND a.attnum = d.adnum
                           WHERE n.nspname = %s
```

`(module)` line 1119

```sql
SELECT i.relname AS name,
                        pg_catalog.pg_get_indexdef(i.oid) AS def
                   FROM pg_catalog.pg_class i
                   JOIN pg_catalog.pg_namespace n ON n.oid = i.relnamespace
                  WHERE n.nspname = %s AND i.relkind IN ('i', 'I')
```

`(module)` line 1124

```sql
SELECT c.relname AS name,
                       pg_catalog.pg_get_viewdef(c.oid, true) AS def
                  FROM pg_catalog.pg_class c
                  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
                 WHERE n.nspname = %s AND c.relkind IN ('v', 'm')
```

`(module)` line 1133

```sql
SELECT p.proname AS name,
                           pg_catalog.pg_get_functiondef(p.oid) AS def
                      FROM pg_catalog.pg_proc p
                      JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace
                     WHERE n.nspname = %s AND {prokind_where}
```

`(module)` line 1138

```sql
SELECT t.tgname AS name,
                          pg_catalog.pg_get_triggerdef(t.oid) AS def
                     FROM pg_catalog.pg_trigger t
                     JOIN pg_catalog.pg_class c ON c.oid = t.tgrelid
                     JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
                    WHERE n.nspname = %s AND NOT t.tgisinternal
```

`(module)` line 1144

```sql
SELECT c.relname || '.' || a.attname AS name,
                              pg_catalog.format_type(a.atttypid, a.atttypmod) AS def
                         FROM pg_catalog.pg_attribute a
                         JOIN pg_catalog.pg_class c ON c.oid = a.attrelid
                         JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
                        WHERE n.nspname = %s AND a.attnum > 0 AND NOT a.attisdropped
                          AND c.relkind IN ('r', 'p', 'v', 'm')
```

`(module)` line 1151

```sql
SELECT t.typname AS name,
                         pg_catalog.format_type(t.typbasetype, t.typtypmod) AS def
                    FROM pg_catalog.pg_type t
                    JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace
                   WHERE n.nspname = %s AND t.typtype = 'd'
```

**_CENSUS_SQL_BASE** - `(module)` line 1170

```sql
SELECT kind, name FROM (
  SELECT CASE c.relkind WHEN 'r' THEN 'table' WHEN 'p' THEN 'table'
                        WHEN 'f' THEN 'foreign table'
                        WHEN 'v' THEN 'view'  WHEN 'm' THEN 'matview'
                        WHEN 'S' THEN 'sequence' WHEN 'i' THEN 'index'
                        WHEN 'I' THEN 'index' ELSE 'relation' END AS kind,
         c.relname::text AS name
    FROM pg_catalog.pg_class c
    JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
   WHERE n.nspname = %s AND c.relkind <> 'c'
  UNION ALL
  SELECT 'function',
         p.proname || '(' ||
         pg_catalog.pg_get_function_identity_arguments(p.oid) || ')'
    FROM pg_catalog.pg_proc p
    JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace
   WHERE n.nspname = %s
  UNION ALL
  SELECT 'type', t.typname::text
    FROM pg_catalog.pg_type t
    JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace
   WHERE n.nspname = %s AND t.typtype IN ('e', 'd', 'c')
     AND NOT EXISTS (SELECT 1 FROM pg_catalog.pg_type el WHERE el.typarray = t.oid)
     AND (t.typrelid = 0 OR EXISTS (SELECT 1 FROM pg_catalog.pg_class rc
                                     WHERE rc.oid = t.typrelid AND rc.relkind = 'c'))
  UNION ALL
  SELECT 'trigger', c.relname || '.' || tg.tgname
    FROM pg_catalog.pg_trigger tg
    JOIN pg_catalog.pg_class c ON c.oid = tg.tgrelid
    JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
   WHERE n.nspname = %s AND NOT tg.tgisinternal
  UNION ALL
  SELECT 'constraint', c.relname || '.' || co.conname
    FROM pg_catalog.pg_constraint co
    JOIN pg_catalog.pg_class c ON c.oid = co.conrelid
    JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
   WHERE n.nspname = %s
```

`_schema_sequences` line 1331

```sql
SELECT sequencename AS name, data_type::text AS data_type,
       increment_by, min_value, max_value, cycle,
       cache_size, last_value
  FROM pg_catalog.pg_sequences
 WHERE schemaname = %s
```

`_schema_sequences` line 1341

```sql
SELECT c.relname AS name
  FROM pg_catalog.pg_class c
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind = 'S'
```

`_schema_sequences` line 1348

```sql
SELECT increment_by, min_value, max_value, is_cycled AS cycle, cache_value AS cache_size, last_value FROM {fq}
```

`_verify_schema_copy._not_valid` line 1519

```sql
SELECT COALESCE(c.relname, t.typname)::text AS owner, co.conname::text AS con
  FROM pg_catalog.pg_constraint co
  JOIN pg_catalog.pg_namespace n ON n.oid = co.connamespace
  LEFT JOIN pg_catalog.pg_class c ON c.oid = co.conrelid AND co.conrelid <> 0
  LEFT JOIN pg_catalog.pg_type t ON t.oid = co.contypid AND co.contypid <> 0
 WHERE n.nspname = %s AND NOT co.convalidated
 ORDER BY 1, 2
```

`_verify_schema_copy` line 1541

```sql
SELECT c.relname AS name FROM pg_catalog.pg_class c
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s
UNION
SELECT t.typname FROM pg_catalog.pg_type t
  JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace WHERE n.nspname = %s
```

`_verify_schema_copy` line 1561

```sql
SELECT s.relname AS name
  FROM pg_catalog.pg_class s
  JOIN pg_catalog.pg_namespace sn ON sn.oid = s.relnamespace
  LEFT JOIN pg_catalog.pg_class t
         ON t.relname = s.relname
        AND t.relnamespace = (SELECT oid FROM pg_catalog.pg_namespace
                               WHERE nspname = %s)
 WHERE sn.nspname = %s AND s.relkind IN ('r', 'p')
   AND (s.relrowsecurity IS DISTINCT FROM t.relrowsecurity
     OR s.relforcerowsecurity IS DISTINCT FROM t.relforcerowsecurity)
 ORDER BY 1
```

`_verify_schema_copy` line 1580

```sql
SELECT i.relname AS name
  FROM pg_catalog.pg_index x
  JOIN pg_catalog.pg_class i ON i.oid = x.indexrelid
  JOIN pg_catalog.pg_namespace n ON n.oid = i.relnamespace
 WHERE n.nspname = %s AND (NOT x.indisvalid OR NOT x.indisready)
 ORDER BY 1
```

`_verify_schema_copy` line 1600

```sql
SELECT c.relname AS name
  FROM pg_catalog.pg_class c
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relkind IN ('r', 'p')
   {_not_partition}
 ORDER BY c.relname
```

`_verify_schema_copy` line 1614

```sql
SELECT (SELECT count(*) FROM %s.%s%s) AS a, (SELECT count(*) FROM %s.%s) AS b
```

`_outside_partitions` line 1746

```sql
WITH RECURSIVE t(oid) AS (
    SELECT i.inhrelid FROM pg_catalog.pg_inherits i
     WHERE i.inhparent = pg_catalog.to_regclass(%s)
  UNION ALL
    SELECT i.inhrelid FROM pg_catalog.pg_inherits i JOIN t ON i.inhparent = t.oid)
SELECT c.oid::regclass::text AS rel FROM t
  JOIN pg_catalog.pg_class c ON c.oid = t.oid
  JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname <> %s AND c.relkind IN ('r', 'f')
 ORDER BY 1
```

`api_schema_clone` line 1882

```sql
SELECT 1 FROM pg_namespace WHERE nspname = %s
```

`api_schema_clone` line 1885

```sql
SELECT 1 FROM pg_namespace WHERE nspname = %s
```

`api_schema_clone` line 1919

```sql
SELECT usesuper FROM pg_user WHERE usename = current_user
```

`api_schema_clone` line 1998

```sql
CREATE SCHEMA {qi(tgt)};
```

**st** - `api_schema_clone` line 2087

```sql
INSERT INTO {qi(tgt)}.{qi(t)} ({collist}){override} SELECT {', '.join(sel)} FROM {only}{qi(src)}.{qi(t)}{where};
```

`api_schema_clone` line 2101

```sql
SET LOCAL row_security = off;
```

`api_schema_clone` line 2103

```sql
RESET row_security;
```

`api_schema_clone` line 2111

```sql
SELECT setval({_literal_str(qi(tgt) + '.' + qi(s))}, (SELECT last_value FROM {qi(src)}.{qi(s)}), (SELECT is_called FROM {qi(src)}.{qi(s)}));
```

`api_schema_clone` line 2132

```sql
SELECT setval({tgt_seq}, COALESCE({src_last}, 1), {src_last} IS NOT NULL);
```

**st** - `api_schema_clone` line 2143

```sql
ALTER TABLE {qi(tgt)}.{qi(t)} ADD CONSTRAINT {qi(cn)} {cdef.replace('{SCHEMA}.', qi(tgt) + '.').rstrip().rstrip(';')};
```

**st** - `api_schema_clone` line 2148

```sql
ALTER TABLE {qi(tgt)}.{qi(t)} ADD CONSTRAINT {qi(cn)} NOT NULL {qi(col)} NOT VALID;
```

**st** - `api_schema_clone` line 2155

```sql
ALTER DOMAIN {qi(tgt)}.{qi(dn)} ADD CONSTRAINT {qi(cn)} {cdef.replace('{SCHEMA}.', qi(tgt) + '.').rstrip().rstrip(';')};
```

`api_schema_clone` line 2199

```sql
COMMENT ON POLICY {qi(pn)} ON {qi(tgt)}.{qi(t)} IS {_literal_str(pc)};
```

`api_schema_clone` line 2214

```sql
REFRESH MATERIALIZED VIEW {qi(tgt)}.{qi(n)};
```

`api_schema_compare` line 2400

```sql
SELECT rolname FROM pg_catalog.pg_roles
```

`api_schema_apply_sync` line 2655

```sql
SELECT current_setting('server_version_num')::int, split_part(current_setting('server_version'), ' ', 1)
```

`api_schema_apply_sync` line 2666

```sql
SELECT pg_catalog.current_setting('search_path')
```

`api_schema_apply_sync` line 2668

```sql
SELECT pg_catalog.set_config('search_path', '', false)
```

`api_schema_apply_sync` line 2679

```sql
SELECT to_regclass(%s) IS NOT NULL
```

`api_schema_apply_sync` line 2688

```sql
SELECT pg_catalog.set_config('search_path', %s, false)
```

## api/schema/objects.py

`api_schema_trigger_functions` line 110

```sql
SELECT n.nspname            AS schema,
       p.proname            AS name,
       pg_get_function_identity_arguments(p.oid) AS args,
       t.typname            AS rettype,
       pg_get_function_arguments(p.oid)         AS sig
  FROM pg_proc p
  JOIN pg_namespace n ON n.oid = p.pronamespace
  JOIN pg_type t      ON t.oid = p.prorettype
 WHERE t.typname IN ('trigger', 'event_trigger')
   AND n.nspname NOT IN ('pg_catalog', 'information_schema')
 ORDER BY n.nspname, p.proname
```

`api_schema_create_trigger` line 200

```sql
UPDATE OF {', '.join((_ident(c) for c in upd_cols))}
```

**sql** - `api_schema_alter_trigger` line 295

```sql
ALTER TABLE {tbl} ENABLE TRIGGER {_ident(name)}
```

**sql** - `api_schema_alter_trigger` line 297

```sql
ALTER TABLE {tbl} DISABLE TRIGGER {_ident(name)}
```

**sql** - `api_schema_alter_trigger` line 299

```sql
ALTER TABLE {tbl} ENABLE REPLICA TRIGGER {_ident(name)}
```

**sql** - `api_schema_alter_trigger` line 301

```sql
ALTER TABLE {tbl} ENABLE ALWAYS TRIGGER {_ident(name)}
```

**sql** - `api_schema_alter_trigger` line 306

```sql
ALTER TRIGGER {_ident(name)} ON {tbl} RENAME TO {_ident(new_name)}
```

**q** - `api_schema_policies` line 350

```sql
SELECT schemaname AS schema, tablename AS table,
       policyname AS name,
       permissive::text,
       cmd,
       roles::text[] AS roles,
       qual          AS using_expr,
       with_check    AS check_expr
  FROM pg_policies
 WHERE schemaname = %s
```

`api_schema_policies` line 369

```sql
SELECT n.nspname  AS schema, c.relname AS table,
       c.relrowsecurity AS rls_enabled,
       c.relforcerowsecurity AS rls_forced
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s
   AND c.relkind IN ('r','p')
```

**sql** - `api_schema_create_policy` line 445

```sql
CREATE POLICY {_ident(name)} ON {_ident(schema)}.{_ident(table)} AS {perm} FOR {cmd} TO {', '.join(role_sql_parts)}
```

`api_schema_create_policy` line 448

```sql
WITH CHECK ({check_})
```

`api_schema_alter_policy` line 508

```sql
WITH CHECK ({new_check.strip() or 'true'})
```

`api_schema_alter_policy` line 514

```sql
ALTER POLICY {_ident(name)} ON {_ident(schema)}.{_ident(table)} {' '.join(parts)}
```

`api_schema_alter_policy` line 519

```sql
ALTER POLICY {_ident(name)} ON {_ident(schema)}.{_ident(table)} RENAME TO {_ident(new_name)}
```

`api_schema_drop_policy` line 541

```sql
DROP POLICY name ON schema.table. Body: { conn params, schema, table, name }
```

**sql** - `api_schema_drop_policy` line 552

```sql
DROP POLICY {_ident(name)} ON {_ident(schema)}.{_ident(table)}
```

**sql** - `api_schema_rls_toggle` line 600

```sql
ALTER TABLE {tbl} {clause}
```

**sql** - `(module)` line 621

```sql
DROP TABLE {n}{cascade}
```

**sql** - `(module)` line 622

```sql
DROP TABLE {n}{cascade}
```

**sql** - `(module)` line 623

```sql
DROP TABLE {n}{cascade}
```

**sql** - `(module)` line 630

```sql
DROP FOREIGN TABLE {n}{cascade}
```

**sql** - `(module)` line 631

```sql
DROP FOREIGN TABLE {n}{cascade}
```

**sql** - `(module)` line 632

```sql
DROP INDEX {n}{cascade}
```

**sql** - `(module)` line 633

```sql
DROP SCHEMA {n}{cascade}
```

**sql** - `(module)` line 634

```sql
DROP FUNCTION {n}({args}){cascade}
```

**sql** - `(module)` line 635

```sql
DROP PROCEDURE {n}({args}){cascade}
```

**sql** - `(module)` line 636

```sql
DROP AGGREGATE {n}({args}){cascade}
```

**sql** - `(module)` line 637

```sql
DROP FUNCTION {n}({args}){cascade}
```

**sql** - `(module)` line 638

```sql
DROP TRIGGER {tgname} ON {n}{cascade}
```

**sql** - `(module)` line 639

```sql
DROP RULE {rulename} ON {n}{cascade}
```

**sql** - `(module)` line 640

```sql
DROP POLICY {polname} ON {n}{cascade}
```

**sql** - `(module)` line 653

```sql
DROP EXTENSION {n}{cascade}
```

`api_schema_create_view` line 854

```sql
CREATE [OR REPLACE] VIEW. Body:
       {schema, name, sql_body, or_replace?, materialized?, with_data?}
    sql_body must be a SELECT - we wrap it as `CREATE VIEW … AS <sql_body>`.
    A pre-flight EXPLAIN catches syntax errors with a clear message before
    actually creating the view.
```

`api_schema_create_view` line 888

```sql
EXPLAIN (FORMAT TEXT) {sql_body}
```

`api_schema_function_detail` line 1052

```sql
SELECT p.oid,
       {kind_expr} AS prokind,
       pg_get_userbyid(p.proowner) AS owner,
       l.lanname AS language,
       p.provolatile::text AS volatile,
       p.prosecdef AS security_definer,
       p.proleakproof AS leakproof,
       {par_expr} AS parallel,
       p.procost::float  AS cost,
       CASE WHEN p.proretset THEN p.prorows::float END AS rows,
       p.proisstrict  AS strict,
       p.proretset    AS returns_set,
       pg_get_function_result(p.oid)                AS return_type,
       pg_get_function_arguments(p.oid)             AS arguments,
       pg_get_function_identity_arguments(p.oid)    AS identity_arguments,
       pg_get_functiondef(p.oid)                    AS def
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
JOIN pg_language l ON l.oid = p.prolang
WHERE n.nspname = %s AND p.proname = %s
  AND pg_get_function_identity_arguments(p.oid) = %s
```

`api_schema_create_function` line 1094

```sql
CREATE OR REPLACE FUNCTION / PROCEDURE. Body:
       {schema, name, or_replace?, is_procedure?,
        args: "raw arg list e.g. p1 integer, p2 text DEFAULT 'x'",
        returns: "type or 'TABLE (...)' or 'SETOF type'", (omitted for procedure)
        language: "plpgsql"|"sql"|...,
        body: "function body as $body$ ... $body$ contents",
        volatility: "VOLATILE"|"STABLE"|"IMMUTABLE",  (optional, function only)
        security_definer?: bool,
        strict?: bool,
        cost?: number, rows?: number,
        set_params: ["search_path = public, mysch", ...],
       }

    SAFETY: we pre-flight with `PREPARE _dbg_…` AS… on plain SQL functions
    where possible; for plpgsql we just attempt CREATE OR REPLACE and let
    Postgres do the parse. Catalog still rolls back cleanly if it fails.
```

`api_schema_create_function` line 1194

```sql
SELECT p.oid, p.prokind, p.procost, p.prorows, p.proretset,
       p.proparallel, p.proleakproof, p.proconfig,
       pg_get_function_identity_arguments(p.oid) AS ident,
       (SELECT lanname FROM pg_language l WHERE l.oid = p.prolang) AS lang
FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname = %s AND p.proname = %s
```

`api_schema_create_function` line 1226

```sql
ALTER {what} {target} {' '.join(keep)}
```

`api_schema_create_function` line 1232

```sql
ALTER {what} {target} SET {} = {}
```

`api_schema_create_function` line 1234

```sql
SET {k} = {v}
```

`api_schema_alter_function` line 1249

```sql
ALTER FUNCTION / PROCEDURE. Body:
       {schema, name, args, action, ...action-specific}

    Actions:
      rename       new_name
      set_schema   new_schema
      owner        owner_to
      set_search_path  search_path  (e.g. "public, myschema")
      reset_search_path
      security     security_definer: bool
      volatility   volatility: VOLATILE|STABLE|IMMUTABLE
      strict       strict: bool
      set_cost     cost: number
      set_rows     rows: number
      leakproof    leakproof: bool           (superuser only)
      parallel     parallel: UNSAFE|RESTRICTED|SAFE   (PG 9.6+)
```

`api_schema_alter_function` line 1340

```sql
SET search_path =
```

`api_schema_alter_function` line 1342

```sql
RESET search_path
```

`api_schema_type_detail` line 1390

```sql
SELECT t.oid, t.typtype::text AS typtype,
       pg_get_userbyid(t.typowner) AS owner,
       t.typbasetype, t.typnotnull AS not_null,
       t.typdefault AS default_expr,
       format_type(t.typbasetype, t.typtypmod) AS base_type
FROM pg_type t
JOIN pg_namespace n ON n.oid = t.typnamespace
WHERE n.nspname = %s AND t.typname = %s
```

`api_schema_type_detail` line 1406

```sql
SELECT enumlabel, enumsortorder
FROM pg_enum WHERE enumtypid = %s
ORDER BY enumsortorder
```

`api_schema_type_detail` line 1415

```sql
SELECT a.attname,
       format_type(a.atttypid, a.atttypmod) AS type,
       a.attnotnull AS not_null,
       pg_get_expr(d.adbin, d.adrelid) AS default_expr
FROM pg_type t
JOIN pg_attribute a ON a.attrelid = t.typrelid
LEFT JOIN pg_attrdef d ON d.adrelid = a.attrelid AND d.adnum = a.attnum
WHERE t.oid = %s
  AND a.attnum > 0 AND NOT a.attisdropped
ORDER BY a.attnum
```

`api_schema_type_detail` line 1430

```sql
SELECT conname,
       pg_get_constraintdef(oid) AS def
FROM pg_constraint
WHERE contypid = %s
ORDER BY conname
```

`api_schema_type_detail` line 1444

```sql
SELECT format_type(rngsubtype, NULL) AS subtype
FROM pg_range WHERE rngtypid = %s
```

`api_schema_create_type` line 1469

```sql
CREATE TYPE for enum / composite / domain.
    Body shape per kind:
      enum:      {kind: "enum",      schema, name, values: ["v1","v2",...]}
      composite: {kind: "composite", schema, name,
                  attributes: [{name, type, not_null?, default?}, ...]}
      domain:    {kind: "domain",    schema, name, base_type,
                  not_null?, default?, checks: ["expr1", "expr2"]}

    Identifiers go through psycopg2.sql.Identifier; expression literals
    (default, check expressions, enum values) are bound or quoted to
    prevent injection. PG has no `CREATE TYPE IF NOT EXISTS`, so a
    name clash is surfaced verbatim from the server.
```

**stmt** - `api_schema_create_type` line 1503

```sql
CREATE TYPE {fq} AS ENUM ({vals_sql})
```

**stmt** - `api_schema_create_type` line 1518

```sql
CREATE TYPE {fq} AS ({', '.join(cols)})
```

`api_schema_create_type` line 1529

```sql
CREATE DOMAIN {fq} AS {base}
```

`api_schema_alter_type` line 1565

```sql
ALTER TYPE / ALTER DOMAIN. One endpoint, many actions; the `action`
    field discriminates. Body shapes:

      Generic (any custom type):
        {action: "rename",      schema, name, new_name}
        {action: "set_schema",  schema, name, new_schema}
        {action: "owner",       schema, name, owner_to}

      ENUM only:
        {action: "add_value",   schema, name, value, before?: "x" | after?: "y", if_not_exists?: bool}
        {action: "rename_value",schema, name, old_value, new_value}  # PG10+

      COMPOSITE only:
        {action: "add_attr",    schema, name, attr_name, attr_type}
        {action: "drop_attr",   schema, name, attr_name, cascade?: bool}
        {action: "alter_attr",  schema, name, attr_name, new_type}
        {action: "rename_attr", schema, name, attr_name, new_attr_name}

      DOMAIN only:
        {action: "domain_set_default",  schema, name, default}
        {action: "domain_drop_default", schema, name}
        {action: "domain_set_notnull",  schema, name, not_null: bool}
        {action: "domain_add_check",    schema, name, constraint_name, expr}
        {action: "domain_drop_constraint", schema, name, constraint_name}
        {action: "domain_rename_constraint", schema, name, old, new}

    All identifiers / new names validated; expressions accept arbitrary SQL
    (psycopg2 wouldn't quote them anyway), with semicolon-block defence.
```

**stmt** - `api_schema_alter_type` line 1619

```sql
ALTER TYPE {fq} RENAME TO {_id(new_name, 'new_name')}
```

**stmt** - `api_schema_alter_type` line 1622

```sql
ALTER TYPE {fq} SET SCHEMA {_id(new_schema, 'new_schema')}
```

**stmt** - `api_schema_alter_type` line 1627

```sql
ALTER TYPE {fq} OWNER TO {_id(owner_to, 'owner_to')}
```

**stmt** - `api_schema_alter_type` line 1643

```sql
ALTER TYPE {fq} ADD VALUE {'IF NOT EXISTS ' if ife else ''}{_lit(v)}{pos}
```

**stmt** - `api_schema_alter_type` line 1649

```sql
ALTER TYPE {fq} RENAME VALUE {_lit(ov)} TO {_lit(nv)}
```

**stmt** - `api_schema_alter_type` line 1656

```sql
ALTER TYPE {fq} ADD ATTRIBUTE {_id(an, 'attr_name')} {at}
```

**stmt** - `api_schema_alter_type` line 1660

```sql
ALTER TYPE {fq} DROP ATTRIBUTE {_id(an, 'attr_name')}{cas}
```

**stmt** - `api_schema_alter_type` line 1665

```sql
ALTER TYPE {fq} ALTER ATTRIBUTE {_id(an, 'attr_name')} SET DATA TYPE {nt}
```

**stmt** - `api_schema_alter_type` line 1669

```sql
ALTER TYPE {fq} RENAME ATTRIBUTE {_id(an, 'attr_name')} TO {_id(nn, 'new_attr_name')}
```

**stmt** - `api_schema_alter_type` line 1679

```sql
ALTER DOMAIN {fq} {'SET' if nn else 'DROP'} NOT NULL
```

**stmt** - `api_schema_alter_type` line 1683

```sql
ALTER DOMAIN {fq} ADD CONSTRAINT {_id(cn, 'constraint_name')} CHECK ({ex})
```

**stmt** - `api_schema_alter_type` line 1686

```sql
ALTER DOMAIN {fq} DROP CONSTRAINT {_id(cn, 'constraint_name')}
```

**stmt** - `api_schema_alter_type` line 1690

```sql
ALTER DOMAIN {fq} RENAME CONSTRAINT {_id(old, 'old')} TO {_id(new_, 'new')}
```

`api_table_create` line 1717

```sql
CREATE TABLE from a structured spec OR raw SQL.
    Body modes:
      A) {mode:"sql", sql: "CREATE TABLE ..."} - passed through verbatim,
          executed in autocommit. Safer for power users with custom DDL.
      B) {mode:"struct", schema, name, columns:[{name,type,not_null,default,
          primary_key,unique,fk:{schema,table,column,on_delete,on_update}}],
          constraints:{check:[...]}, owner?, tablespace?, if_not_exists?,
          comment?} - built via sql.Identifier/Literal, identifiers regex-checked.
```

`api_table_create` line 1824

```sql
CREATE TABLE
```

`api_table_create` line 1861

```sql
ALTER TABLE {s}.{t} OWNER TO {o}
```

`api_table_create` line 1867

```sql
COMMENT ON TABLE {s}.{t} IS {c}
```

`api_schema_alter_column` line 2043

```sql
SET NOT NULL
```

`api_schema_alter_column` line 2045

```sql
DROP NOT NULL
```

`api_schema_index_detail` line 2134

```sql
SELECT ic.relname                             AS name,
       am.amname                              AS am,
       tn.nspname                             AS table_schema,
       tc.relname                             AS table_name,
       ic.relkind::text                       AS relkind,
       i.indisvalid                           AS is_valid,
       i.indisprimary                         AS is_primary,
       i.indisunique                          AS is_unique,
       i.indisexclusion                       AS is_exclusion,
       COALESCE(ts.spcname, 'pg_default')     AS tablespace,
       ic.reloptions                          AS reloptions,
       pg_get_indexdef(i.indexrelid)          AS definition,
       pg_size_pretty(pg_relation_size(i.indexrelid)) AS size,
       i.indkey::int2[]                       AS indkey,
       (SELECT c.conname FROM pg_constraint c
         WHERE c.conindid = i.indexrelid LIMIT 1) AS owning_constraint
  FROM pg_index i
  JOIN pg_class ic     ON ic.oid = i.indexrelid
  JOIN pg_namespace n  ON n.oid = ic.relnamespace
  JOIN pg_class tc     ON tc.oid = i.indrelid
  JOIN pg_namespace tn ON tn.oid = tc.relnamespace
  JOIN pg_am am        ON am.oid = ic.relam
  LEFT JOIN pg_tablespace ts ON ts.oid = ic.reltablespace
 WHERE n.nspname = %s AND ic.relname = %s
```

`api_schema_alter_index` line 2219

```sql
SELECT am.amname FROM pg_class c
                             JOIN pg_namespace n ON n.oid = c.relnamespace
                             JOIN pg_am am ON am.oid = c.relam
                            WHERE n.nspname = %s AND c.relname = %s
```

**sql** - `api_schema_alter_index` line 2238

```sql
ALTER INDEX {fq} RENAME TO {_qi(new_name)}
```

**sql** - `api_schema_alter_index` line 2243

```sql
ALTER INDEX {fq} SET TABLESPACE {_qi(ts)}
```

**sql** - `api_schema_alter_index` line 2264

```sql
ALTER INDEX {fq} SET ({', '.join(parts)})
```

**sql** - `api_schema_alter_index` line 2272

```sql
ALTER INDEX {fq} RESET ({', '.join((_qi(str(k).strip()) for k in names))})
```

**sql** - `api_schema_alter_index` line 2284

```sql
ALTER INDEX {fq} ALTER COLUMN {colnum} SET STATISTICS {value}
```

`api_schema_relation_detail` line 2363

```sql
SELECT c.oid,
       c.relkind::text                    AS relkind,
       c.relpersistence::text             AS persistence,
       c.reloptions                       AS reloptions,
       COALESCE(ts.spcname, 'pg_default') AS tablespace,
       -- relrowsecurity is 9.5+; selecting it on 9.4 fails the
       -- whole query, which would take the menu's detail read
       -- down with it rather than just this one field.
       {'c.relrowsecurity' if pgv >= 90500 else 'false'} AS rls_enabled,
       pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,
       pg_size_pretty(pg_relation_size(c.oid))       AS heap_size
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_tablespace ts ON ts.oid = c.reltablespace
 WHERE n.nspname = %s AND c.relname = %s
```

`api_schema_relation_detail` line 2391

```sql
SELECT ic.relname AS name, i.indisclustered AS clustered,
       i.indisvalid AS is_valid
  FROM pg_index i JOIN pg_class ic ON ic.oid = i.indexrelid
 WHERE i.indrelid = %s ORDER BY ic.relname
```

`api_schema_relation_detail` line 2407

```sql
SELECT relispartition FROM pg_class WHERE oid = %s
```

`api_schema_relation_detail` line 2410

```sql
SELECT count(*) AS n FROM pg_inherits WHERE inhparent = %s
```

`api_schema_alter_relation` line 2466

```sql
SELECT c.relkind::text, c.relpersistence::text
                         FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
                        WHERE n.nspname = %s AND c.relname = %s
```

**sql** - `api_schema_alter_sequence` line 2687

```sql
ALTER SEQUENCE {fq} {' '.join(parts)}
```

## api/schema/tree.py

`api_schema_sequence_detail` line 53

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
       pg_catalog.pg_get_userbyid(c.relowner) AS owner,
       obj_description(c.oid, 'pg_class')      AS comment
FROM pg_sequences s
JOIN pg_class     c ON c.relname = s.sequencename
JOIN pg_namespace n ON n.oid = c.relnamespace AND n.nspname = s.schemaname
WHERE s.schemaname = %s AND s.sequencename = %s
```

`api_schema_sequence_detail` line 72

```sql
SELECT %s::text                       AS schemaname,
       %s::text                       AS sequencename,
       'bigint'::text                 AS data_type,
       q.start_value, q.min_value, q.max_value,
       q.increment_by,
       q.cache_value                  AS cache_size,
       q.is_cycled                    AS cycle,
       pg_catalog.pg_get_userbyid(c.relowner) AS owner,
       obj_description(c.oid, 'pg_class')     AS comment
  FROM {q} q,
       pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
```

`api_schema_sequence_detail` line 97

```sql
SELECT last_value, is_called FROM "{schema.replace('"', '""')}"."{name.replace('"', '""')}"
```

`api_schema_sequence_detail` line 107

```sql
SELECT n.nspname AS schema, t.relname AS "table", a.attname AS column
FROM pg_depend d
JOIN pg_class    c ON c.oid = d.objid
JOIN pg_namespace n2 ON n2.oid = c.relnamespace
JOIN pg_class    t ON t.oid = d.refobjid
JOIN pg_namespace n ON n.oid = t.relnamespace
JOIN pg_attribute a ON a.attrelid = d.refobjid AND a.attnum = d.refobjsubid
WHERE d.classid = 'pg_class'::regclass
  AND d.deptype IN ('a', 'i')
  AND n2.nspname = %s AND c.relname = %s
LIMIT 1
```

`api_schema_sequence_detail` line 124

```sql
SELECT n.nspname AS schema, c.relname AS "table", a.attname AS column,
       pg_get_expr(ad.adbin, ad.adrelid) AS default_expr
FROM pg_attrdef ad
JOIN pg_attribute a ON a.attrelid = ad.adrelid AND a.attnum = ad.adnum
JOIN pg_class    c ON c.oid = ad.adrelid
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE pg_get_expr(ad.adbin, ad.adrelid) LIKE %s
   OR pg_get_expr(ad.adbin, ad.adrelid) LIKE %s
ORDER BY 1, 2, 3
```

`api_schema_sequence_setval` line 205

```sql
SELECT setval('{fq}', {value}, {str(is_called).lower()})
```

`api_schema_sequence_setval` line 207

```sql
SELECT setval(%s::regclass, %s, %s)
```

`_tree_build` line 264

```sql
SELECT n.nspname FROM pg_namespace n
                  WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
                    AND n.nspname NOT LIKE 'pg\_toast%%' AND n.nspname NOT LIKE 'pg\_temp\_%%'
                  ORDER BY 1
```

`_tree_build` line 296

```sql
SELECT c.oid,
       GREATEST(NULLIF(s.n_live_tup, 0), NULLIF(c.reltuples, -1)::bigint, 0),
       coalesce(GREATEST(NULLIF(s.n_live_tup, 0), NULLIF(c.reltuples, -1)::bigint), 0) = 0
         AND pg_relation_size(c.oid) > 0
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_stat_user_tables s ON s.relid = c.oid
 WHERE c.relkind IN ('r', 'p')
   AND n.nspname NOT IN ('pg_catalog', 'information_schema')
   AND n.nspname NOT LIKE 'pg\_toast%%' AND n.nspname NOT LIKE 'pg\_temp\_%%'
```

`_tree_build` line 328

```sql
SELECT c.oid, n.nspname, c.relname FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = ANY(%s)
```

`_tree_build.build` line 345

```sql
SELECT n.nspname, c.relname FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = %s
```

`_tree_build` line 463

```sql
SELECT c.relname, c.relkind::text, c.oid FROM pg_class c
                       JOIN pg_namespace n ON n.oid = c.relnamespace
                      WHERE n.nspname = %s AND c.relkind IN ('r','v','m','S','i') ORDER BY c.relname
```

`_tree_build` line 471

```sql
SELECT p.proname, p.oid, {rk}, pg_get_function_identity_arguments(p.oid)
                        FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
                       WHERE n.nspname = %s ORDER BY p.proname
```

`_tree_build` line 478

```sql
SELECT t.typname, t.oid, CASE t.typtype WHEN 'd' THEN 'D' WHEN 'r' THEN 'R'
                                                     WHEN 'm' THEN 'MR' ELSE 'Y' END
                       FROM pg_type t JOIN pg_namespace n ON n.oid = t.typnamespace
                      WHERE n.nspname = %s AND t.typtype IN ('b','e','d','r','m')
                        AND t.typname NOT LIKE '\_%%' ORDER BY t.typname
```

`_tree_children_timescale` line 555

```sql
SELECT n.nspname, c.relname FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = %s
```

`api_schema_tree_children` line 592

```sql
SELECT c.oid FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE n.nspname = %s AND c.relname = %s
```

`api_schema_columns` line 636

```sql
SELECT a.attname                          AS name,
       format_type(a.atttypid, a.atttypmod) AS type,
       a.attnotnull                       AS not_null,
       pg_get_expr(d.adbin, d.adrelid)    AS default,
       t.typcategory::text                AS cat,
       t.typname                          AS type_name,
       t.typtype::text                    AS typtype,
       {ident_col}                        AS identity,
       {gen_col}                          AS generated
  FROM pg_attribute a
  JOIN pg_class c     ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_attrdef d ON d.adrelid = c.oid AND d.adnum = a.attnum
  JOIN pg_type t      ON t.oid = a.atttypid
 WHERE n.nspname = %s AND c.relname = %s
   AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY a.attnum
```

`api_schema_columns` line 660

```sql
SELECT a.attname AS col
  FROM pg_index i
  JOIN pg_class c ON c.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum = ANY(i.indkey)
 WHERE i.indisprimary AND n.nspname = %s AND c.relname = %s
 ORDER BY (SELECT u.ord FROM unnest(i.indkey::smallint[]) WITH ORDINALITY AS u(k, ord) WHERE u.k = a.attnum LIMIT 1)
```

`api_schema_columns` line 671

```sql
SELECT c.relkind::text AS k
  FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND c.relname = %s
```

`_leaf_relids_sql` line 695

```sql
-- pg_partition_tree() returns NOTHING for a relation that is
-- neither a partition nor partitioned - not "the relation
-- itself", which is what the code here used to assume. Measured
-- on 13, 14 and 18: zero rows every time, so every ordinary
-- table summed over an empty set and reported 0 bytes and 0
-- rows. A 2,000,000-row table displayed as "0 bytes".
SELECT t.relid
  FROM pg_partition_tree(c.oid) t
 WHERE c.relkind = 'p' AND t.isleaf
UNION ALL
SELECT c.oid
 WHERE c.relkind <> 'p'
```

`_leaf_relids_sql` line 709

```sql
WITH RECURSIVE tree(relid) AS (
    SELECT c.oid
    UNION ALL
    SELECT i.inhrelid
      FROM pg_inherits i JOIN tree ON i.inhparent = tree.relid
)
SELECT tree.relid
  FROM tree JOIN pg_class lc ON lc.oid = tree.relid
 WHERE c.relkind = 'p' AND lc.relkind <> 'p'
UNION ALL
SELECT c.oid
 WHERE c.relkind <> 'p'
```

`_leaf_relids_sql` line 722

```sql
SELECT c.oid AS relid
```

`api_schema_tables_overview` line 756

```sql
SELECT
    c.relname                                      AS tablica,  -- kept: JS sort key 'tablica' depends on this
    c.relkind                                      AS kind,
    -- A partitioned parent owns no storage and collects no
    -- statistics of its own, so reading pg_class/pg_stat directly
    -- showed it as 0 rows and 0 bytes next to its own partitions.
    -- Measured: a table with 400,000 rows across two 10 MB
    -- partitions displayed as "0 bytes". The totals below are
    -- summed over pg_partition_tree() for relkind='p'; every other
    -- relkind is its own single-member tree, so the same
    -- expression serves both and there is no branch to keep in
    -- sync.
    COALESCE(agg.n_live, 0)                        AS n_live,
    COALESCE(agg.n_dead, 0)                        AS n_dead,
    CASE WHEN COALESCE(agg.n_live,0) > 0
         THEN round((agg.n_dead::numeric / NULLIF(agg.n_live,0)) * 100, 1)
         ELSE 0 END                                AS dead_pct,
    -- NULL, not 0, when the sizes were not asked for: "unknown"
    -- and "empty" are different answers and the UI draws them
    -- differently. COALESCE here is what would turn a table whose
    -- size nobody measured into a table of zero bytes.
    agg.total_bytes                                AS total_bytes,
    CASE WHEN agg.total_bytes IS NULL THEN NULL
         ELSE pg_size_pretty(agg.total_bytes) END  AS total_size,
    CASE WHEN agg.heap_bytes IS NULL THEN NULL
         ELSE pg_size_pretty(agg.heap_bytes) END   AS table_size,
    CASE WHEN agg.index_bytes IS NULL THEN NULL
         ELSE pg_size_pretty(agg.index_bytes) END  AS indexes_size,
    (SELECT count(*) FROM pg_index i WHERE i.indrelid = c.oid) AS indexes_count,
    COALESCE(s.seq_scan, 0)                        AS seq_scan,
    COALESCE(s.idx_scan, 0)                        AS idx_scan,
    GREATEST(s.last_vacuum, s.last_autovacuum)::text   AS last_vacuum,
    GREATEST(s.last_analyze, s.last_autoanalyze)::text AS last_analyze
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
LEFT JOIN pg_stat_user_tables s ON s.schemaname = n.nspname AND s.relname = c.relname
-- Totals over the partition tree. pg_partition_tree() returns the
-- relation itself for anything unpartitioned, so plain tables,
-- matviews and foreign tables come out unchanged; foreign tables
-- report no size, hence the COALESCEs above.
LEFT JOIN LATERAL (
    SELECT {SIZES}
           sum(COALESCE(ps.n_live_tup, 0))               AS n_live,
           sum(COALESCE(ps.n_dead_tup, 0))               AS n_dead
      FROM ( {LEAVES} ) x
      LEFT JOIN pg_stat_user_tables ps ON ps.relid = x.relid
) agg ON true
WHERE n.nspname = %s
  AND c.relkind IN ('r','p','m','f')
ORDER BY c.relname
```

`api_schema_table_sizes` line 869

```sql
SET statement_timeout = %s
```

`api_schema_table_sizes` line 870

```sql
SELECT sum(pg_total_relation_size(x.relid)) AS total_bytes,
       sum(pg_relation_size(x.relid))       AS heap_bytes,
       sum(pg_indexes_size(x.relid))        AS index_bytes,
       pg_size_pretty(sum(pg_total_relation_size(x.relid))) AS total_size,
       pg_size_pretty(sum(pg_relation_size(x.relid)))       AS table_size,
       pg_size_pretty(sum(pg_indexes_size(x.relid)))        AS indexes_size
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  -- Partitioned parents own no storage, so they are
  -- summed over their leaves; everything else is its
  -- own single entry (see _leaf_relids_sql).
  CROSS JOIN LATERAL ( {LEAVES} ) x
 WHERE n.nspname = %s AND c.relname = %s
```

`api_schema_table_sizes` line 897

```sql
SET statement_timeout = 0
```

`api_schema_relations` line 923

```sql
SELECT
  c.conname AS ime,
  (SELECT string_agg(att.attname, ', ' ORDER BY u.ord)
     FROM unnest(c.conkey) WITH ORDINALITY u(attnum, ord)
     JOIN pg_attribute att ON att.attrelid=c.conrelid AND att.attnum=u.attnum
  ) AS kolone,
  fns.nspname || '.' || fc.relname AS ref_tablica,
  (SELECT string_agg(att.attname, ', ' ORDER BY u.ord)
     FROM unnest(c.confkey) WITH ORDINALITY u(attnum, ord)
     JOIN pg_attribute att ON att.attrelid=c.confrelid AND att.attnum=u.attnum
  ) AS ref_kolone,
  CASE c.confdeltype WHEN 'a' THEN 'NO ACTION' WHEN 'r' THEN 'RESTRICT'
                      WHEN 'c' THEN 'CASCADE' WHEN 'n' THEN 'SET NULL'
                      WHEN 'd' THEN 'SET DEFAULT' END AS on_delete,
  CASE c.confupdtype WHEN 'a' THEN 'NO ACTION' WHEN 'r' THEN 'RESTRICT'
                      WHEN 'c' THEN 'CASCADE' WHEN 'n' THEN 'SET NULL'
                      WHEN 'd' THEN 'SET DEFAULT' END AS on_update
FROM pg_constraint c
JOIN pg_class fc ON fc.oid = c.confrelid
JOIN pg_namespace fns ON fns.oid = fc.relnamespace
WHERE c.conrelid = %s::regclass AND c.contype = 'f'
ORDER BY c.conname
```

`api_schema_relations` line 949

```sql
SELECT
  c.conname AS ime,
  cns.nspname || '.' || cc.relname AS izvor_tablica,
  (SELECT string_agg(att.attname, ', ' ORDER BY u.ord)
     FROM unnest(c.conkey) WITH ORDINALITY u(attnum, ord)
     JOIN pg_attribute att ON att.attrelid=c.conrelid AND att.attnum=u.attnum
  ) AS izvor_kolone,
  (SELECT string_agg(att.attname, ', ' ORDER BY u.ord)
     FROM unnest(c.confkey) WITH ORDINALITY u(attnum, ord)
     JOIN pg_attribute att ON att.attrelid=c.confrelid AND att.attnum=u.attnum
  ) AS moje_kolone,
  CASE c.confdeltype WHEN 'a' THEN 'NO ACTION' WHEN 'r' THEN 'RESTRICT'
                      WHEN 'c' THEN 'CASCADE' WHEN 'n' THEN 'SET NULL'
                      WHEN 'd' THEN 'SET DEFAULT' END AS on_delete
FROM pg_constraint c
JOIN pg_class cc ON cc.oid = c.conrelid
JOIN pg_namespace cns ON cns.oid = cc.relnamespace
WHERE c.confrelid = %s::regclass AND c.contype = 'f'
ORDER BY cns.nspname, cc.relname, c.conname
```

`api_schema_relations` line 972

```sql
SELECT a.attname, pg_catalog.format_type(a.atttypid, a.atttypmod) AS tip,
       a.attnotnull AS not_null
FROM pg_attribute a
WHERE a.attrelid = %s::regclass AND a.attnum > 0 AND NOT a.attisdropped
ORDER BY a.attnum
```

`_relmap_timescale` line 1076

```sql
SELECT c.oid, n.nspname, c.relname FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.oid = ANY(%s)
```

`api_schema_relations_map` line 1191

```sql
SELECT c.oid, c.relname AS name, c.relkind AS kind,
       coalesce(s.n_live_tup, 0) AS n_live,
       GREATEST(coalesce(s.n_live_tup, 0), coalesce(c.reltuples, 0)::bigint) AS est_rows,
       pg_get_userbyid(c.relowner) AS owner, {rls}
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_stat_user_tables s ON s.relid = c.oid
 WHERE n.nspname = %s AND c.relkind IN ('r','p','m','v','f','S')
 ORDER BY c.relname
```

`api_schema_relations_map` line 1246

```sql
SELECT c.relname AS rel, c.relkind, a.attnum, a.attname AS name,
       format_type(a.atttypid, a.atttypmod) AS type,
       a.attnotnull AS nn,
       bt.typtype AS base_typtype, btn.nspname AS base_type_schema,
       bt.typname AS base_type_name, bt.oid AS base_type_oid,
       (btr.relkind = 'c') AS base_is_free_composite
  FROM pg_attribute a
  JOIN pg_class c ON c.oid = a.attrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_type t ON t.oid = a.atttypid
  JOIN pg_type bt ON bt.oid = CASE WHEN t.typcategory = 'A' AND t.typelem <> 0
                                   THEN t.typelem ELSE t.oid END
  JOIN pg_namespace btn ON btn.oid = bt.typnamespace
  LEFT JOIN pg_class btr ON btr.oid = bt.typrelid
 WHERE n.nspname = %s AND c.relkind IN ('r','p','v','m','f')
   AND a.attnum > 0 AND NOT a.attisdropped
 ORDER BY c.relname, a.attnum
```

`api_schema_relations_map` line 1290

```sql
SELECT c.relname AS rel, k.contype,
       ARRAY(SELECT a.attname FROM pg_attribute a
              WHERE a.attrelid = k.conrelid AND a.attnum = ANY(k.conkey)) AS cols
  FROM pg_constraint k
  JOIN pg_class c ON c.oid = k.conrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND k.contype IN ('p','u','f')
```

`api_schema_relations_map` line 1303

```sql
SELECT c.conname                                            AS name,
       src_n.nspname                                        AS src_schema,
       src.relname                                          AS src_table,
       ARRAY(SELECT a.attname FROM pg_attribute a
              WHERE a.attrelid = c.conrelid AND a.attnum = ANY(c.conkey)
              ORDER BY (SELECT min(g) FROM generate_subscripts(c.conkey, 1) g
                         WHERE c.conkey[g] = a.attnum))  AS src_cols,
       tgt_n.nspname                                        AS tgt_schema,
       tgt.relname                                          AS tgt_table,
       ARRAY(SELECT a.attname FROM pg_attribute a
              WHERE a.attrelid = c.confrelid AND a.attnum = ANY(c.confkey)
              ORDER BY (SELECT min(g) FROM generate_subscripts(c.confkey, 1) g
                         WHERE c.confkey[g] = a.attnum)) AS tgt_cols,
       CASE c.confdeltype WHEN 'a' THEN 'NO ACTION' WHEN 'r' THEN 'RESTRICT'
            WHEN 'c' THEN 'CASCADE' WHEN 'n' THEN 'SET NULL' WHEN 'd' THEN 'SET DEFAULT'
            ELSE c.confdeltype::text END                     AS on_delete,
       CASE c.confupdtype WHEN 'a' THEN 'NO ACTION' WHEN 'r' THEN 'RESTRICT'
            WHEN 'c' THEN 'CASCADE' WHEN 'n' THEN 'SET NULL' WHEN 'd' THEN 'SET DEFAULT'
            ELSE c.confupdtype::text END                     AS on_update
FROM pg_constraint c
JOIN pg_class src ON src.oid = c.conrelid
JOIN pg_namespace src_n ON src_n.oid = src.relnamespace
JOIN pg_class tgt ON tgt.oid = c.confrelid
JOIN pg_namespace tgt_n ON tgt_n.oid = tgt.relnamespace
WHERE c.contype = 'f'
  AND (src_n.nspname = %s OR tgt_n.nspname = %s)
  {no_inherited}
ORDER BY src_n.nspname, src.relname, c.conname
```

`api_schema_relations_map` line 1350

```sql
SELECT vn.nspname AS src_schema, v.relname AS src,
       rn.nspname AS tgt_schema, r.relname AS tgt, r.relkind AS tgt_kind,
       array_remove(array_agg(DISTINCT a.attname::text), NULL) AS cols
  FROM pg_depend d
  JOIN pg_rewrite w ON w.oid = d.objid
  JOIN pg_class v ON v.oid = w.ev_class
  JOIN pg_namespace vn ON vn.oid = v.relnamespace
  JOIN pg_class r ON r.oid = d.refobjid
  JOIN pg_namespace rn ON rn.oid = r.relnamespace
  LEFT JOIN pg_attribute a ON a.attrelid = r.oid AND a.attnum = d.refobjsubid
                          AND d.refobjsubid > 0
 WHERE d.classid = 'pg_rewrite'::regclass
   AND d.refclassid = 'pg_class'::regclass
   AND w.rulename = '_RETURN'
   AND v.relkind IN ('v','m') AND r.oid <> v.oid
   AND (vn.nspname = %s OR rn.nspname = %s)
 GROUP BY vn.nspname, v.relname, rn.nspname, r.relname, r.relkind
```

`api_schema_relations_map` line 1391

```sql
SELECT sn.nspname AS seq_schema, s.relname AS seq, tn.nspname AS tbl_schema,
       t.relname AS tbl, a.attname AS col, 'default' AS how
  FROM pg_depend d
  JOIN pg_attrdef ad ON ad.oid = d.objid
  JOIN pg_class s ON s.oid = d.refobjid AND s.relkind = 'S'
  JOIN pg_namespace sn ON sn.oid = s.relnamespace
  JOIN pg_class t ON t.oid = ad.adrelid
  JOIN pg_namespace tn ON tn.oid = t.relnamespace
  JOIN pg_attribute a ON a.attrelid = t.oid AND a.attnum = ad.adnum
 WHERE d.classid = 'pg_attrdef'::regclass AND d.refclassid = 'pg_class'::regclass
   AND (sn.nspname = %s OR tn.nspname = %s)
   AND NOT EXISTS (
       SELECT 1 FROM pg_inherits i
         JOIN pg_attribute pa ON pa.attrelid = i.inhparent AND pa.attname = a.attname
         JOIN pg_attrdef pad ON pad.adrelid = pa.attrelid AND pad.adnum = pa.attnum
         JOIN pg_depend pd ON pd.classid = 'pg_attrdef'::regclass
                          AND pd.objid = pad.oid
                          AND pd.refclassid = 'pg_class'::regclass
                          AND pd.refobjid = s.oid
        WHERE i.inhrelid = t.oid)
```

`api_schema_relations_map` line 1433

```sql
SELECT c.relname AS rel, s.oid AS srv_oid, s.srvname, w.fdwname,
       s.srvoptions, ft.ftoptions
  FROM pg_foreign_table ft
  JOIN pg_class c ON c.oid = ft.ftrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_foreign_server s ON s.oid = ft.ftserver
  JOIN pg_foreign_data_wrapper w ON w.oid = s.srvfdw
 WHERE n.nspname = %s
```

`api_schema_relations_map` line 1472

```sql
SELECT t.oid, n.nspname AS schema, t.typname AS name, t.typtype,
       format_type(t.typbasetype, t.typtypmod) AS base,
       format_type(rg.rngsubtype, NULL) AS subtype,
       CASE WHEN t.typtype = 'e' THEN
            ARRAY(SELECT e.enumlabel::text FROM pg_enum e
                   WHERE e.enumtypid = t.oid ORDER BY e.enumsortorder)
       END AS labels,
       CASE WHEN t.typtype = 'c' THEN
            ARRAY(SELECT a.attname || ' ' || format_type(a.atttypid, a.atttypmod)
                    FROM pg_attribute a
                   WHERE a.attrelid = t.typrelid AND a.attnum > 0
                     AND NOT a.attisdropped ORDER BY a.attnum)
       END AS attrs,
       ARRAY(SELECT pg_get_constraintdef(k.oid) FROM pg_constraint k
              WHERE k.contypid = t.oid) AS checks
  FROM pg_type t
  JOIN pg_namespace n ON n.oid = t.typnamespace
  LEFT JOIN pg_class r ON r.oid = t.typrelid
  LEFT JOIN pg_range rg ON rg.rngtypid = t.oid
 WHERE (n.nspname = %s OR t.oid = ANY(%s::oid[]))
   AND (t.typtype IN ('e','d','r') OR (t.typtype = 'c' AND r.relkind = 'c'))
```

`api_schema_relations_map` line 1534

```sql
SELECT c.relname AS rel, t.tgname, t.tgtype, p.oid AS fn_oid,
       pn.nspname AS fn_schema, p.proname AS fn_name,
       pg_get_function_identity_arguments(p.oid) AS fn_args
  FROM pg_trigger t
  JOIN pg_class c ON c.oid = t.tgrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_proc p ON p.oid = t.tgfoid
  JOIN pg_namespace pn ON pn.oid = p.pronamespace
 WHERE n.nspname = %s AND NOT t.tgisinternal {clones}
 ORDER BY c.relname, t.tgname
```

`api_schema_relations_map._dep_relations` line 1568

```sql
SELECT d.objid, c.oid AS rel_oid, c.relname AS rel, n.nspname AS rel_schema,
       a.attname AS col
  FROM pg_depend d
  JOIN pg_class c ON c.oid = d.refobjid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  LEFT JOIN pg_attribute a ON a.attrelid = c.oid AND a.attnum = d.refobjsubid
                          AND d.refobjsubid > 0
 WHERE d.classid = %s::regclass AND d.refclassid = 'pg_class'::regclass
   AND d.objid = ANY(%s::oid[])
 ORDER BY d.objid, c.relname, a.attnum
```

`api_schema_relations_map` line 1587

```sql
SELECT p.oid, p.polname AS name, c.oid AS rel_oid, c.relname AS tbl,
       n.nspname AS tbl_schema,
       CASE p.polcmd WHEN 'r' THEN 'SELECT' WHEN 'a' THEN 'INSERT'
            WHEN 'w' THEN 'UPDATE' WHEN 'd' THEN 'DELETE' ELSE 'ALL' END AS cmd,
       {permissive} AS permissive,
       ARRAY(SELECT CASE WHEN r = 0 THEN 'PUBLIC' ELSE pg_get_userbyid(r) END
               FROM unnest(p.polroles) r ORDER BY 1) AS roles,
       pg_get_expr(p.polqual, p.polrelid) AS using_expr,
       pg_get_expr(p.polwithcheck, p.polrelid) AS check_expr,
       c.relrowsecurity AS rls
  FROM pg_policy p
  JOIN pg_class c ON c.oid = p.polrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s
 ORDER BY c.relname, p.polname
```

`api_schema_relations_map` line 1632

```sql
SELECT w.oid, w.rulename AS name, c.oid AS rel_oid, c.relname AS rel,
       n.nspname AS rel_schema,
       CASE w.ev_type WHEN '1' THEN 'SELECT' WHEN '2' THEN 'UPDATE'
            WHEN '3' THEN 'INSERT' WHEN '4' THEN 'DELETE' END AS event,
       w.is_instead, pg_get_ruledef(w.oid, true) AS definition
  FROM pg_rewrite w
  JOIN pg_class c ON c.oid = w.ev_class
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s AND w.rulename <> '_RETURN'
 ORDER BY c.relname, w.rulename
```

`api_schema_relations_map` line 1666

```sql
SELECT i.indexrelid AS oid, ic.relname AS name, n.nspname AS schema,
       c.relname AS tbl, am.amname AS method,
       i.indisunique AS uniq, i.indisprimary AS prim,
       {nkey} AS nkey, i.indnatts AS natts,
       pg_get_expr(i.indpred, i.indrelid) AS pred,
       ARRAY(SELECT pg_get_indexdef(i.indexrelid, k, true)
               FROM generate_series(1, i.indnatts) k ORDER BY k) AS keys,
       ARRAY(SELECT coalesce(a.attname::text, '')
               FROM generate_series(1, i.indnatts) k
               LEFT JOIN pg_attribute a ON a.attrelid = i.indrelid
                                       AND a.attnum = i.indkey[k - 1]
                                       AND i.indkey[k - 1] <> 0
              ORDER BY k) AS cols,
       con.conname AS constraint_name
  FROM pg_index i
  JOIN pg_class ic ON ic.oid = i.indexrelid
  JOIN pg_class c ON c.oid = i.indrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_am am ON am.oid = ic.relam
  LEFT JOIN pg_constraint con ON con.conindid = i.indexrelid
                             AND con.contype IN ('p','u','x')
 WHERE n.nspname = %s AND c.relkind IN ('r','p','m')
 ORDER BY c.relname, ic.relname
```

`api_schema_relations_map` line 1731

```sql
SELECT x.oid, x.stxname AS name, xn.nspname AS schema,
       c.relname AS tbl, n.nspname AS tbl_schema,
       pg_get_userbyid(x.stxowner) AS owner, x.stxkind::text[] AS kinds,
       ARRAY(SELECT a.attname::text
               FROM unnest(x.stxkeys::int2[]) WITH ORDINALITY k(attnum, ord)
               JOIN pg_attribute a ON a.attrelid = x.stxrelid AND a.attnum = k.attnum
              ORDER BY k.ord) AS cols,
       {exprs} AS exprs
  FROM pg_statistic_ext x
  JOIN pg_namespace xn ON xn.oid = x.stxnamespace
  JOIN pg_class c ON c.oid = x.stxrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE n.nspname = %s OR xn.nspname = %s
 ORDER BY x.stxname
```

`api_schema_relations_map` line 1777

```sql
SELECT CASE WHEN w.rulename = '_RETURN' THEN 'view' ELSE 'rule' END AS how,
       w.oid AS objid, w.ev_class AS rel_oid, NULL::name AS col, NULL::name AS detail, {fc}
  FROM pg_depend d JOIN pg_rewrite w ON w.oid = d.objid
  JOIN pg_class c ON c.oid = w.ev_class
  JOIN pg_namespace n ON n.oid = c.relnamespace {fj}
 WHERE d.classid = 'pg_rewrite'::regclass AND {fw} AND n.nspname = %(s)s
```

`api_schema_relations_map` line 1784

```sql
SELECT CASE WHEN {gen} THEN 'generated' ELSE 'default' END,
       ad.oid, ad.adrelid, a.attname, NULL::name, {fc}
  FROM pg_depend d JOIN pg_attrdef ad ON ad.oid = d.objid
  JOIN pg_class c ON c.oid = ad.adrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace
  JOIN pg_attribute a ON a.attrelid = ad.adrelid AND a.attnum = ad.adnum {fj}
 WHERE d.classid = 'pg_attrdef'::regclass AND {fw} AND n.nspname = %(s)s
   AND NOT EXISTS (
       SELECT 1 FROM pg_inherits i
         JOIN pg_attribute pa ON pa.attrelid = i.inhparent AND pa.attname = a.attname
         JOIN pg_attrdef pad ON pad.adrelid = pa.attrelid AND pad.adnum = pa.attnum
         JOIN pg_depend pd ON pd.classid = 'pg_attrdef'::regclass
                          AND pd.objid = pad.oid
                          AND pd.refclassid = 'pg_proc'::regclass
                          AND pd.refobjid = d.refobjid
        WHERE i.inhrelid = ad.adrelid)
```

`api_schema_relations_map` line 1801

```sql
SELECT 'check', k.oid, k.conrelid,
       CASE WHEN array_length(k.conkey, 1) = 1 THEN
            (SELECT a.attname FROM pg_attribute a
              WHERE a.attrelid = k.conrelid AND a.attnum = k.conkey[1]) END,
       k.conname, {fc}
  FROM pg_depend d JOIN pg_constraint k ON k.oid = d.objid
  JOIN pg_class c ON c.oid = k.conrelid
  JOIN pg_namespace n ON n.oid = c.relnamespace {fj}
 WHERE d.classid = 'pg_constraint'::regclass AND {fw} AND n.nspname = %(s)s
   AND k.contype = 'c' AND k.conislocal
```

`api_schema_relations_map` line 1812

```sql
SELECT 'domain', k.oid, k.contypid, NULL::name, k.conname, {fc}
  FROM pg_depend d JOIN pg_constraint k ON k.oid = d.objid {fj}
 WHERE d.classid = 'pg_constraint'::regclass AND {fw}
   AND k.contypid = ANY(%(types)s::oid[])
```

`api_schema_relations_map` line 1817

```sql
SELECT 'index', d.objid, NULL::oid, NULL::name, NULL::name, {fc}
  FROM pg_depend d {fj}
 WHERE d.classid = 'pg_class'::regclass AND {fw}
   AND d.objid = ANY(%(idx)s::oid[])
```

`api_schema_relations_map` line 1824

```sql
SELECT 'policy', d.objid, NULL::oid, NULL::name, NULL::name, {fc}
  FROM pg_depend d {fj}
 WHERE d.classid = 'pg_policy'::regclass AND {fw}
   AND d.objid = ANY(%(pol)s::oid[])
```

`api_schema_relations_map` line 1831

```sql
SELECT 'statistics', d.objid, NULL::oid, NULL::name, NULL::name, {fc}
  FROM pg_depend d {fj}
 WHERE d.classid = 'pg_statistic_ext'::regclass AND {fw}
   AND d.objid = ANY(%(stx)s::oid[])
```

`api_schema_relations_map` line 1887

```sql
SELECT p.oid, p.pubname AS name, pg_get_userbyid(p.pubowner) AS owner,
       p.puballtables AS all_tables, p.pubinsert, p.pubupdate, p.pubdelete,
       {trunc} AS pubtruncate, {viaroot} AS via_root, {pschemas} AS schemas
  FROM pg_publication p
 WHERE p.puballtables
    OR EXISTS (SELECT 1 FROM pg_publication_rel pr
                 JOIN pg_class c ON c.oid = pr.prrelid
                 JOIN pg_namespace n ON n.oid = c.relnamespace
                WHERE pr.prpubid = p.oid AND n.nspname = %s)
    OR %s = ANY({pschemas})
 ORDER BY p.pubname
```

`api_schema_relations_map` line 1943

```sql
SELECT e.extname AS name, n.nspname AS schema, e.extversion AS version,
       pg_get_userbyid(e.extowner) AS owner
  FROM pg_extension e
  JOIN pg_namespace n ON n.oid = e.extnamespace
 WHERE n.nspname = %s OR e.extname = ANY(%s::name[])
 ORDER BY e.extname
```
