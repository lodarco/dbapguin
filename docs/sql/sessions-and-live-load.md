# Sessions and live load

[All parts](README.md) · 49 statements

## api/sessions.py

`_psa._has` line 33

```sql
SELECT 1 FROM information_schema.columns WHERE table_name='pg_stat_activity' AND column_name=%s
```

`_psa._fn` line 38

```sql
SELECT 1 FROM pg_proc WHERE proname=%s
```

`_xmin_holders` line 109

```sql
SELECT pid, usename, application_name,
       COALESCE(host(client_addr), 'local') AS client, datname, state,
       age({_SESSION_XMIN})::bigint AS xmin_age,
       EXTRACT(epoch FROM now() - xact_start)::bigint AS xact_age_s,
       left(query, 200) AS query
  FROM pg_stat_activity
 WHERE {_SESSION_XMIN} IS NOT NULL AND pid <> pg_backend_pid() {not_walsender}
 ORDER BY 7 DESC LIMIT %s
```

`_xmin_holders` line 119

```sql
SELECT gid, owner, database, age(transaction)::bigint AS xmin_age,
       EXTRACT(epoch FROM now() - prepared)::bigint AS prepared_age_s
  FROM pg_prepared_xacts ORDER BY age(transaction) DESC LIMIT %s
```

`_xmin_holders` line 124

```sql
SELECT slot_name, slot_type, database, active,
       GREATEST(age(xmin), age(catalog_xmin))::bigint AS xmin_age
  FROM pg_replication_slots
 WHERE xmin IS NOT NULL OR catalog_xmin IS NOT NULL
 ORDER BY 5 DESC NULLS LAST LIMIT %s
```

`_xmin_holders` line 131

```sql
SELECT pid, application_name, COALESCE(host(client_addr), 'local') AS client,
       age(backend_xmin)::bigint AS xmin_age
  FROM pg_stat_replication WHERE backend_xmin IS NOT NULL
 ORDER BY age(backend_xmin) DESC LIMIT %s
```

`_xmin_horizon` line 154

```sql
WITH holders AS (
    SELECT 'session'::text AS kind, pid::text AS id, {_SESSION_XMIN} AS xmin
      FROM pg_stat_activity
     WHERE {_SESSION_XMIN} IS NOT NULL AND pid <> pg_backend_pid()
    UNION ALL
    SELECT 'prepared transaction', gid, transaction
      FROM pg_prepared_xacts
    UNION ALL
    SELECT 'replication slot', slot_name, xmin
      FROM pg_replication_slots WHERE xmin IS NOT NULL
    UNION ALL
    SELECT 'replication slot (catalog)', slot_name, catalog_xmin
      FROM pg_replication_slots WHERE catalog_xmin IS NOT NULL
)
SELECT kind, id, xmin::text AS xmin, age(xmin)::bigint AS xmin_age
  FROM holders
 ORDER BY age(xmin) DESC
 LIMIT 1
```

`_xmin_horizon` line 184

```sql
SELECT 'session'::text AS kind, pid::text AS id,
       xmin::text AS xmin, age(xmin)::bigint AS xmin_age
  FROM (SELECT pid, %s AS xmin FROM pg_stat_activity
         WHERE pid <> pg_backend_pid()) s
 WHERE xmin IS NOT NULL
 ORDER BY age(xmin) DESC
 LIMIT 1
```

`api_sessions_list` line 214

```sql
SELECT pid,
       datname,
       usename AS "user",
       application_name AS app,
       coalesce(client_addr::text, 'local') AS client,
       state,
```

`api_sessions_detail` line 274

```sql
SELECT a.pid, a.datname, a.usename AS "user", a.application_name AS app,
       coalesce(a.client_addr::text, 'local') AS client, a.client_port,
       {df['backend_type_expr']} AS backend_type, a.state,
       {df['wait_type_expr']} AS wait_event_type,
       {df['wait_expr']} AS wait_event,
       a.backend_start, a.xact_start, a.query_start, a.state_change,
       age(now(), a.backend_start)::text AS backend_age,
       date_trunc('second', age(now(), a.xact_start))::text    AS xact_age,
       age(now(), a.query_start)::text   AS query_age,
       date_trunc('second', age(now(), a.state_change))::text  AS state_age,
       a.backend_xid::text               AS backend_xid,
       a.backend_xmin::text              AS backend_xmin,
       age(a.backend_xmin)::bigint       AS xmin_age,
       a.query, {df['leader_pid_expr']} AS leader_pid
  FROM pg_stat_activity a
 WHERE a.pid = %s
```

`api_sessions_detail` line 345

```sql
SELECT l.locktype, l.mode, l.granted,
       coalesce(c.relname, l.locktype) AS object,
       n.nspname AS schema
  FROM pg_locks l
  LEFT JOIN pg_class c     ON c.oid = l.relation
  LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
 WHERE l.pid = %s
 ORDER BY l.granted, l.locktype
```

`api_sessions_detail` line 364

```sql
SELECT query_id FROM pg_stat_activity WHERE pid = %s
```

`api_sessions_detail` line 374

```sql
SELECT calls, round(%(total)s::numeric, 1) AS total_ms,
       round(%(mean)s::numeric, 2)  AS mean_ms,
       rows
  FROM
```

`api_sessions_detail` line 405

```sql
EXPLAIN (FORMAT TEXT, VERBOSE, COSTS) {q}
```

`_current_db_name` line 421

```sql
SELECT current_database() AS d
```

`_oldest_relation` line 471

```sql
SELECT n.nspname AS schema, c.relname AS name,        GREATEST(age(c.relfrozenxid),                 COALESCE(age(t.relfrozenxid), 0)) AS age,        c.relisshared AS shared,        {can_vac} AS can_vacuum   FROM pg_class c   JOIN pg_namespace n ON n.oid = c.relnamespace   LEFT JOIN pg_class t ON t.oid = c.reltoastrelid  WHERE c.relkind IN ('r','m')    AND c.relfrozenxid <> '0'::xid {'   AND GREATEST(age(c.relfrozenxid),                  COALESCE(age(t.relfrozenxid), 0)) >= %d ' % int(min_age) if min_age else ''}{' ORDER BY GREATEST(age(c.relfrozenxid),                    COALESCE(age(t.relfrozenxid), 0)) DESC,           n.nspname, c.relname  LIMIT %d' % int(limit)}
```

`_cluster_alarm_facts` line 546

```sql
SELECT (SELECT count(*) FROM pg_stat_activity {client_only})    AS used,
       current_setting('max_connections')::int                  AS max_conn,
       current_setting('superuser_reserved_connections')::int{role_res} AS reserved
```

`_cluster_alarm_facts` line 566

```sql
SELECT count(*)                                              AS blocked,
       COALESCE(MAX(EXTRACT(epoch FROM (now() - a.state_change))), 0) AS longest_s
  FROM pg_stat_activity a
 WHERE cardinality(pg_blocking_pids(a.pid)) > 0
```

`_cluster_alarm_facts` line 592

```sql
SELECT r.pid, r.usename, a.datname,
       r.application_name, r.state, r.sync_state,
       r.client_addr::text AS client_addr,
       {slot_sel} AS slot_name,
       EXTRACT(epoch FROM {lag_col}) AS replay_lag_s
  FROM pg_stat_replication r
  LEFT JOIN pg_stat_activity a ON a.pid = r.pid
  {slot_join}
```

`_cluster_alarm_facts` line 617

```sql
SELECT slot_name, slot_type, active,
       CASE WHEN restart_lsn IS NULL THEN NULL
            ELSE {retained} END AS retained_bytes
  FROM pg_replication_slots
```

`_cluster_alarm_facts` line 640

```sql
SELECT datname                                       AS oldest_db,
       age(datfrozenxid)                             AS max_age,
       datallowconn                                  AS allows_conn,
       datname = current_database()                  AS connected,
       current_setting('autovacuum_freeze_max_age')::bigint AS freeze_max
  FROM pg_database
 ORDER BY age(datfrozenxid) DESC, datname
 LIMIT 1
```

`_cluster_alarm_facts._caps` line 850

```sql
SELECT name, setting FROM pg_settings
 WHERE name IN ('archive_mode', 'max_prepared_transactions')
```

`_cluster_alarm_facts._has_tsdb` line 874

```sql
SELECT extversion FROM pg_extension WHERE extname = 'timescaledb'
```

`_cluster_alarm_facts` line 893

```sql
SELECT max(mxid_age(datminmxid))::bigint                          AS age,
       current_setting('autovacuum_multixact_freeze_max_age')::bigint AS freeze_max
  FROM pg_database
```

`_cluster_alarm_facts` line 940

```sql
SELECT count(*) AS n,
       COALESCE(MAX(EXTRACT(epoch FROM (now() - xact_start))), 0) AS oldest_s,
       (SELECT count(*) FROM pg_stat_activity
         WHERE state LIKE 'idle in transaction%'
           AND xact_start IS NOT NULL) AS idle_in_tx
  FROM pg_stat_activity
 WHERE xact_start IS NOT NULL
   AND pid <> pg_backend_pid()
   {backend_filter}
```

`_cluster_alarm_facts` line 958

```sql
SELECT pid, usename, application_name,
       COALESCE(host(client_addr), 'local') AS client, datname, state,
       EXTRACT(epoch FROM now() - xact_start)::bigint AS xact_age_s,
       left(query, 200) AS query
  FROM pg_stat_activity
 WHERE xact_start IS NOT NULL AND pid <> pg_backend_pid() {backend_filter}
 ORDER BY xact_start LIMIT 1
```

`_cluster_alarm_facts` line 977

```sql
SELECT count(*) AS n,
       COALESCE(MAX(EXTRACT(epoch FROM (now() - prepared))), 0) AS oldest_s
  FROM pg_prepared_xacts
```

`_cluster_alarm_facts` line 986

```sql
SELECT gid, owner, database,
       EXTRACT(epoch FROM now() - prepared)::bigint AS prepared_age_s
  FROM pg_prepared_xacts ORDER BY prepared LIMIT 10
```

`_cluster_alarm_facts._lim` line 1009

```sql
SELECT name, setting::int AS v
  FROM pg_settings
 WHERE name IN ('max_worker_processes', 'max_parallel_workers',
                'autovacuum_max_workers')
```

`_cluster_alarm_facts` line 1022

```sql
SELECT
  count(*) FILTER (WHERE backend_type = 'parallel worker')    AS parallel_used,
  count(*) FILTER (WHERE backend_type = 'autovacuum worker')  AS av_used,
  -- Everything the postmaster started as a background worker:
  -- parallel workers, logical replication, and whatever an
  -- extension registered under its own name. The fixed
  -- auxiliary processes are not workers and do not come out
  -- of max_worker_processes.
  count(*) FILTER (WHERE backend_type NOT IN (
      'client backend', 'autovacuum launcher', 'autovacuum worker',
      'background writer', 'checkpointer', 'startup', 'walwriter',
      'walsender', 'walreceiver', 'logger', 'archiver',
      'slotsync worker', 'io worker'))                        AS bg_used
  FROM pg_stat_activity
```

`_cluster_alarm_facts` line 1043

```sql
SELECT count(*) AS av_used
  FROM pg_stat_activity
 WHERE query LIKE 'autovacuum: %'
```

`_cluster_alarm_facts` line 1066

```sql
SELECT COALESCE(SUM(temp_files), 0) AS temp_files,
       COALESCE(SUM(temp_bytes), 0) AS temp_bytes,
       EXTRACT(epoch FROM now())    AS at
  FROM pg_stat_database
```

`_cluster_alarm_facts` line 1084

```sql
SELECT (SELECT count(*) FROM pg_stat_progress_create_index) AS index_builds,
       (SELECT count(*) FROM pg_stat_progress_cluster)      AS clusters
```

`_cluster_alarm_facts._nmax` line 1109

```sql
SELECT current_setting('max_notify_queue_pages', true) AS p, current_setting('block_size')::int AS b
```

`_cluster_alarm_facts` line 1114

```sql
SELECT pg_notification_queue_usage() AS usage,
       (SELECT count(*) FROM pg_stat_activity
         WHERE query ~* '^[[:space:]]*listen[[:space:]]') AS listeners
```

`api_sessions_load._feats` line 1191

```sql
SELECT
  EXISTS (SELECT 1 FROM information_schema.columns
          WHERE table_name='pg_stat_activity' AND column_name='query_id') AS has_qid,
  EXISTS (SELECT 1 FROM pg_extension WHERE extname='pg_stat_statements') AS has_pss,
  EXISTS (SELECT 1 FROM information_schema.columns
          WHERE table_name='pg_stat_activity' AND column_name='leader_pid') AS has_leader,
  to_regclass('pg_stat_progress_vacuum')        IS NOT NULL AS has_p_vac,
  to_regclass('pg_stat_progress_analyze')       IS NOT NULL AS has_p_ana,
  to_regclass('pg_stat_progress_create_index')  IS NOT NULL AS has_p_idx,
  to_regclass('pg_stat_progress_cluster')       IS NOT NULL AS has_p_cls,
  to_regclass('pg_stat_progress_copy')          IS NOT NULL AS has_p_cp,
  to_regclass('pg_stat_progress_basebackup')    IS NOT NULL AS has_p_bb,
  -- 19: pg_stat_progress_vacuum says how hard the vacuum works
  -- (mode) and who started it (started_by)
  EXISTS (SELECT 1 FROM information_schema.columns
          WHERE table_name='pg_stat_progress_vacuum' AND column_name='mode') AS has_p_vac_mode
```

`api_sessions_load` line 1227

```sql
SELECT pg_is_in_recovery() AS in_recovery
```

`api_sessions_load._pss_loaded` line 1325

```sql
SELECT 1 FROM {_pss_rel} LIMIT 1
```

**sql** - `api_sessions_load` line 1436

```sql
SELECT {', '.join(select_lines)} FROM pg_stat_activity a {' '.join(joins)} WHERE a.pid <> pg_backend_pid()    AND (a.state IS NOT NULL AND a.state <> 'idle')  ORDER BY elapsed_s DESC NULLS LAST, a.pid
```

`api_sessions_blocking` line 1513

```sql
SELECT EXISTS (SELECT 1 FROM pg_proc WHERE proname='pg_blocking_pids') AS ok
```

`api_sessions_blocking` line 1522

```sql
SELECT a.pid AS waiter, b.blocker::int AS blocker
FROM pg_stat_activity a
CROSS JOIN LATERAL unnest(pg_blocking_pids(a.pid)) AS b(blocker)
WHERE a.pid <> pg_backend_pid() AND b.blocker <> 0
```

`api_sessions_blocking` line 1536

```sql
SELECT DISTINCT w.pid AS waiter, h.pid AS blocker
  FROM pg_locks w
  JOIN pg_locks h
    ON w.locktype = h.locktype
   AND w.database   IS NOT DISTINCT FROM h.database
   AND w.relation   IS NOT DISTINCT FROM h.relation
   AND w.page       IS NOT DISTINCT FROM h.page
   AND w.tuple      IS NOT DISTINCT FROM h.tuple
   AND w.virtualxid IS NOT DISTINCT FROM h.virtualxid
   AND w.transactionid IS NOT DISTINCT FROM h.transactionid
   AND w.classid    IS NOT DISTINCT FROM h.classid
   AND w.objid      IS NOT DISTINCT FROM h.objid
   AND w.objsubid   IS NOT DISTINCT FROM h.objsubid
   AND w.pid <> h.pid
 WHERE NOT w.granted AND h.granted
   AND w.pid IS NOT NULL AND h.pid IS NOT NULL
   AND w.pid <> pg_backend_pid()
```

`api_sessions_blocking` line 1565

```sql
SELECT DISTINCT w.pid AS waiter, px.gid,
       px.transaction::text                      AS xid,
       px.owner, px.database,
       EXTRACT(EPOCH FROM now() - px.prepared)::float AS prepared_age_s,
       age(px.transaction)::bigint               AS xid_age
  FROM pg_locks w
  JOIN pg_locks h ON h.locktype = 'transactionid' AND h.granted
   AND h.transactionid = w.transactionid AND h.pid IS NULL
  JOIN pg_prepared_xacts px ON px.transaction = h.transactionid
 WHERE w.locktype = 'transactionid' AND NOT w.granted
   AND w.pid IS NOT NULL AND w.pid <> pg_backend_pid()
```

`api_sessions_blocking` line 1621

```sql
SELECT pid,
       datname                                   AS database,
       usename                                   AS "user",
       nullif(application_name, '')              AS app,
       coalesce(client_addr::text, 'local')      AS client_addr,
       client_hostname,
       client_port,
       state,
```

`api_sessions_blocking` line 1657

```sql
WITH ungranted AS (
    SELECT DISTINCT ON (pid)
           pid, locktype, mode, relation, transactionid
    FROM pg_locks
    WHERE NOT granted AND pid = ANY(%s)
    ORDER BY pid, (relation IS NOT NULL) DESC
)
SELECT u.pid,
       u.locktype,
       u.mode,
       u.transactionid::text AS transactionid,
       COALESCE(
         CASE WHEN u.relation IS NOT NULL
              THEN u.relation::regclass::text END,
         g.rel
       ) AS relation,
       (u.relation IS NULL AND g.rel IS NOT NULL) AS relation_inferred
FROM ungranted u
LEFT JOIN LATERAL (
    SELECT c.oid::regclass::text AS rel
    FROM pg_locks l2
    JOIN pg_class c     ON c.oid = l2.relation
    JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE l2.pid = u.pid
      AND l2.granted
      AND n.nspname NOT IN ('pg_catalog', 'information_schema', 'pg_toast')
    ORDER BY CASE l2.mode
               WHEN 'AccessExclusiveLock'    THEN 1
               WHEN 'ExclusiveLock'          THEN 2
               WHEN 'ShareRowExclusiveLock'  THEN 3
               WHEN 'ShareLock'              THEN 4
               WHEN 'RowExclusiveLock'       THEN 5
               ELSE 9
             END
    LIMIT 1
) g ON u.relation IS NULL
```

`api_sessions_kill_many` line 1792

```sql
SELECT {fn}({int(x)})
```

`api_sessions_kill_many` line 1799

```sql
SELECT {fn}(%s)
```

`api_sessions_kill` line 1823

```sql
SELECT {fn}({int(pid)})
```

`api_sessions_kill` line 1827

```sql
SELECT {fn}(%s)
```
