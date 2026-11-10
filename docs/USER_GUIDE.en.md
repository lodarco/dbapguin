# DBApguin User Guide

**For DBApguin 1.0.1**

DBApguin is a desktop PostgreSQL DBA console. This guide covers every feature
from a user's point of view, and ships with the application, so it always
describes the build you are running.

Open it any time from **Help → User guide**.

## 1. Installation

### Linux

1. Download `DBApguin-<version>-x86_64.AppImage`.
2. Make executable: `chmod +x DBApguin-*.AppImage`
3. Double-click or run it from a terminal.

The AppImage is self-contained (~27 MB) and stores its data under `~/.local/share/DBApguin/`.

If the embedded window does not appear, the AppImage shows a setup dialog with installation instructions for every major package manager (apt, dnf, pacman, zypper, apk, emerge). The dialog auto-selects the tab that matches your distribution and offers a one-click "Copy command" button. For Debian/Ubuntu the command is:

```
sudo apt-get install python3-gi python3-gi-cairo gir1.2-gtk-3.0 gir1.2-webkit2-4.1
```

DBApguin is a standalone desktop application: if WebKit2GTK is not available it will NOT silently open a tab in your web browser. Install the package, then relaunch the AppImage.

### Windows

1. Unpack the DBApguin zip anywhere (Desktop, `C:\Tools\`, etc.).
2. Open the folder.
3. Double-click `DBApguin.exe`.

The first launch may show SmartScreen ("Windows protected your PC"). Click "More info" then "Run anyway". DBApguin is not yet code-signed for Windows.

If Microsoft Edge WebView2 Runtime is missing, the app opens a setup dialog with three install paths: (1) launch the bundled `MicrosoftEdgeWebView2Setup.exe` from the app folder, (2) download the online bootstrapper from `developer.microsoft.com/microsoft-edge/webview2/`, or (3) deploy the standalone installer through GPO / SCCM / Intune for managed environments. Each block has a "Copy" button for paths and URLs.

The app does not install into the system; it runs from its folder. To remove it, delete the folder.

The setup dialog reads its language from your DBApguin preferences (`app_prefs.json` -> `language`); on first launch it follows the system locale (Croatian for `hr_*`, English otherwise).

## 1.4 Supported PostgreSQL versions

DBApguin targets **PostgreSQL 13 and newer**. That is the version it is tested against and the one every check is written for.

It also runs against older servers, and the rule is **do what the version allows, do not refuse**. A check that can leave out a column the server lacks, or skip a section that needs a view it does not have, does exactly that and prints a short note saying what it left out. Only a check whose *entire* subject is a feature the server does not have - logical replication, which arrived whole in PostgreSQL 10 - shows a "Needs PostgreSQL N" card instead of running.

Concretely, on a server as old as **9.4**: the session list, live load and the blocking view all work - the blocking view falls back to a `pg_locks` self-join where `pg_blocking_pids()` (9.6) is missing, and the session list shows the `waiting` flag where wait events (9.6) are missing. `Blocking pairs`, `Long transactions`, `Connection limits`, `Autovacuum activity`, `Replication`, `Table sizes`, `Replication slots`, the bloat estimates, replica identity and the health overview all build a version-appropriate query and note what is coarser. Declarative partitioning is 10+, so on an older server the schema tree shows inheritance children as tables under the table they inherit from, never as partitions.

**PostgreSQL 10, 11 and 12** are fully connectable and most of the app works unchanged; a handful of dashboard panels that read newer statistics views are quietly left blank.

Below **PostgreSQL 10** you are past what is tested, but the checks that can adapt do, and the connect-time notice says what is coarser. Below **9.4** the catalogs have diverged far enough that DBApguin does not claim to work at all.

You never see a raw `column "..." does not exist` from a check: when the cause is the server being older than the check expects, the check says that in words instead.

### PostgreSQL 19

DBApguin is measured against PostgreSQL 19 as well: every check runs on it,
with and without the contrib extensions, as a superuser and as a plain role.
A beta or a release candidate says so when you connect, because details can
still change before the final release. A major version newer than 19
connects and runs, with a note saying it is newer than anything measured.

On 19 the application uses what the release added:

- **REPACK** in the Maintenance panel and in the bloat advice. It rewrites a
  table to give the space back, and with CONCURRENTLY it does that while
  reads and writes go on. Older servers keep VACUUM FULL and CLUSTER.
- **The server's own autovacuum queue** in Vacuum status: which tables
  autovacuum takes next and why, including wraparound prevention.
- **Lock statistics** under Locks: waits per lock type since the last reset,
  and how often the fast path ran out of slots.
- **How a vacuum was started and how hard it works** (aggressive, failsafe,
  to prevent wraparound), in the live load and in the vacuum advisor.
- **Per-index reset times** in Unused indexes, so an index whose counters
  were just reset is not called unused.
- **Generic and custom plan counts** per statement in Top queries.
- **EXPLAIN ANALYZE with I/O**: the requests a scan issued, its waits and how
  far ahead it read.
- **Replication**: errors and conflicts per subscription, how often logical
  decoding outgrew its memory, and the recovery state of a standby.
- **Logical slots on wal_level = replica**: 19 raises the WAL level by itself,
  so the slot dialog allows them and says what that costs.
- **Switching constraints off**: the constraint manager can set foreign keys
  (from 18) and checks (from 19) NOT ENFORCED instead of dropping them.
- **Settings advice that follows 19's defaults**: JIT off, a lock table sized
  differently, lock-wait logging on, MD5 logins warned, RADIUS gone from
  pg_hba.conf.

A backup checks first that the pg_dump on this machine is not older than the
server, which is the usual state of a machine on the day a new major version
appears; it names the client package to install. Before you upgrade, run
**Ready for PostgreSQL 19?** (see [the readiness check](#ready-for-postgresql-19)).

## 1.5 Server-side prerequisites

DBApguin works against a stock PostgreSQL cluster with no special setup. Some advanced features require extensions or GUCs to be enabled on the server. The fastest way to find out what is missing is to run the built-in setup advisor:

> Top menu -> Cluster wide -> "DBApguin setup advisor"

It probes the cluster and prints, per feature, a status (OK / PARTIAL / MISSING) plus the exact SQL or postgresql.conf line you need to enable each missing bit. On managed clouds (RDS, Aurora, Cloud SQL, Azure, Supabase, Neon) it switches the remediation hint to the provider parameter group since ALTER SYSTEM is blocked there.

Quick reference, ordered by impact:

| Feature you want to use | Server requirement | How to enable |
|--|--|--|
| Top queries, query history, AWR-style reports | pg_stat_statements ext loaded via `shared_preload_libraries` | Add `pg_stat_statements` to `shared_preload_libraries` + restart, then `CREATE EXTENSION pg_stat_statements` in each DB you want stats for |
| Plan capture during trace | auto_explain ext + `log_min_duration` >= 0 | Add `auto_explain` to `shared_preload_libraries` + restart, then `SET auto_explain.log_min_duration = 500` (or 0 for everything) |
| Query_id in pg_stat_activity, ASH correlation with pg_stat_statements | `compute_query_id = on` | `ALTER SYSTEM SET compute_query_id = on; SELECT pg_reload_conf();` (PG14+) |
| Realistic IO time in pg_stat_statements / EXPLAIN (BUFFERS) | `track_io_timing = on` | `ALTER SYSTEM SET track_io_timing = on; SELECT pg_reload_conf();` |
| Full query text in pg_stat_activity + ASH | `track_activity_query_size >= 4096` | `ALTER SYSTEM SET track_activity_query_size = 4096;` then RESTART |
| Complete Top SQL in AWR (no statements evicted between snapshots) | `pg_stat_statements.max` large enough for the distinct statements | `ALTER SYSTEM SET pg_stat_statements.max = 10000;` then RESTART - the AWR compare says when statements were evicted |
| Function call stats | `track_functions = 'pl'` | `ALTER SYSTEM SET track_functions = 'pl'; SELECT pg_reload_conf();` |
| Buffer cache snapshot in db-cache-advisor | pg_buffercache ext | `CREATE EXTENSION pg_buffercache;` (superuser) |
| Online table rewrite (Maintenance -> pg_repack) | pg_repack ext + client binary | Install pg_repack package on server + DBApguin host, then `CREATE EXTENSION pg_repack;` |
| Server log tail in db-errors | `logging_collector = on`, `log_destination` includes stderr, `pg_read_server_files` granted | Self-hosted only. Managed clouds block pg_read_file. |
| Lock-wait detail in server log | `log_lock_waits = on`, `deadlock_timeout = '1s'` | `ALTER SYSTEM SET log_lock_waits = on;` |
| Slow query log | `log_min_duration_statement >= 0` | `ALTER SYSTEM SET log_min_duration_statement = '1s';` (or `'100ms'` for strict) |
| Server-side audit log (complements client audit) | pgaudit ext or `log_statement = 'ddl'` | Add `pgaudit` to `shared_preload_libraries`, restart, `CREATE EXTENSION pgaudit;` |
| Non-superuser can run DBA checks | Grant `pg_monitor` to the role | `GRANT pg_monitor TO <role>;` (PG10+) |

If you only enable one thing, enable `pg_stat_statements`. Most DBApguin features become significantly more useful once it is loaded.

## 2. First-time setup

On first launch you see an empty Servers sidebar on the left.

1. Click `+ New` in the Servers toolbar.
2. Fill in:
   - Alias (display name)
   - Group (defaults to "default")
   - Optional color tag (helps you spot prod vs staging at a glance)
   - Host, Port, User, Password, Database, sslmode
   - Optional sslrootcert (upload via Settings -> Certificates)
3. Click `Save & Connect` or just `Save`.

The connection appears in the Servers tree. Click it once to open. The Servers tree expands the parent group and scrolls to the row.

Passwords are stored encrypted with a per-install key. The key file lives next to `connections.json` in your data directory.

### What the certificate is for, and what actually turns TLS on

These are two separate settings and the certificate is the second one, not the
first.

**`sslmode` is what turns encryption on.** It sits on the connection itself:

| `sslmode` | What you get |
| --- | --- |
| `disable` | No encryption |
| `prefer` (default) | Encrypted if the server offers it, plaintext if it does not - silently |
| `require` | Encryption is mandatory, but nothing is verified: any certificate is accepted, including one a man in the middle just generated |
| `verify-ca` | The server's certificate must chain to a root you trust |
| `verify-full` | …and the hostname you dialled must match the certificate |

`require` is where most people stop, and it is the setting that reads as safe
and is not: it encrypts the traffic and then hands it to whoever answered.

**The certificate is the root that `verify-ca` and `verify-full` check
against.** libpq looks for one in `~/.postgresql/root.crt`. If your server's
certificate was issued by a public CA that is already there, you do not need to
upload anything - set `sslmode` and you are done. You upload a certificate when
the issuer is *not* already trusted: your own internal PKI, or a cloud
provider's CA bundle (AWS, Google and Azure each publish one).

So:

1. Upload the PEM under **Settings → Certificates**. It is copied to
   `<DBCHECKS_HOME>/certs/` at mode 0600.
2. Open the connection and set **sslmode** to `verify-ca` or `verify-full`.
3. Pick the uploaded file in **TLS root CA certificate** on that same
   connection.

Uploading without step 2 changes nothing: the file is stored, referenced, and
never consulted, because at `prefer` or `require` there is no verification to
consult it for.

## 2.3 Accepting the licence agreement

The first time DBApguin runs it shows the End User Licence Agreement and does
not open until you accept it. There is no way past it - no close button, no
Esc, no clicking outside - because there is nothing behind it to use yet.

The panel shows the agreement itself, not a summary of it, read from the same
file Help → Licence agreement shows later. It carries a **language picker**,
and links to the privacy notice and the third-party component list beside the
buttons.

**Decline and quit** records nothing and closes the window. Nothing is
installed or registered by declining.

**I accept** records two things in `app_prefs.json`, on your machine only:
which version you accepted and when. Nothing is transmitted anywhere. The
stored version is why you are asked once rather than on every start - and why
you will be asked again if the agreement itself changes, since agreeing to
one version is not agreeing to whatever a later one says.

If you ever want to re-read it: **Help → Licence agreement**.

## 2.4 The startup notice

On first run the app shows a short notice with the logo: what it does, and
that the previews it offers are guidance rather than a safety net. It is
there because this tool sends `DROP` and `TRUNCATE` to production at your
direction, and that deserves saying once before it is first used.

The notice also carries a **language picker**, because the first run is when
the choice matters and Settings is not where anyone looks yet. Changing it
switches the notice you are reading immediately, which is the only way to
confirm you picked the right one. The same setting lives in
**Settings → Language** afterwards.

Tick **Do not show this again** and it stops appearing. To bring it back:
**Settings → Startup notice**.

Turning it off hides the reminder. It does not change the terms it points to - 
those are in Help → Licence agreement.

## 2.5 Terms of use

DBApguin is **free for everyone**, private use and commercial use alike, with
every feature included.

What that means in practice:

- **Every feature is available from the first run**, on as many machines as you
  like: the advisors, workload analysis, AWR snapshots, query history, index
  and bloat analysis, the comparison and migration tools - all of it.
- **Nothing is checked over the network**, so nothing phones home. The app
  talks only to the database servers you configure - and, if you
  install plugins, to whatever those plugins are set up to reach (section 7,
  Plugins).
- **No account, no sign-in, no registration.**

The one thing you are asked to do is accept the licence agreement on first run
(section 2.3).

In short: use it freely, do not sell it, and do not present it as your own work.
Free is not the same as open source: the source code is not published. The full text is at **Help -> Licence agreement**.

DBApguin is created by Darko Lovrenčec.

## 3. Servers sidebar

The left panel has three sections:

### 3.1 Servers (top)

Lists every saved connection, grouped by Group. Toolbar buttons:

- `+ New`: register a connection
- `Edit`: edit the currently active connection (alias, group, color, host, password)
- `Collapse all`: collapse every group
- `Expand all`: expand every group
- `More` (...): open the full connection manager modal (import, export, bulk reorganise)

Per-connection right-click menu:
- Connect / Switch to tab
- Edit
- Refresh connection (re-enter password, reconnect)
- Pin / Unpin tab
- Disconnect
- Open SQL editor / psql / Sessions / Dashboard / Maintenance / Server tabs (only on active conn)
- Cluster checks submenu
- Create database / table / role
- Test connection
- Duplicate
- Delete (toast with Undo for 6 seconds)

Status dots next to each connection:
- Green filled: this is the active tab
- Green hollow: tab open but you are looking at another tab
- Red X: connection lost (server side closed it)
- No dot: not connected

### 3.2 Databases (only visible when connected)

Lists every database on the cluster. Click one to switch the active database. Filter box at the top.

**Right-click the DATABASES heading** for the bar's **Database** menu, entry for entry - connections, background jobs, compare, backup and restore, create, clone, rename, privileges and drop, extensions - followed by `Refresh databases list`. The create, clone, rename and drop entries are greyed out, with the reason, when your role or the provider does not allow them.

**Right-click a database row** for everything scoped to that one: SQL editor,
sessions, dashboard, maintenance, the whole check catalogue, object creation,
backup and restore, clone, rename, privileges and drop.

#### Extensions

`Extensions` opens on one of three lists, and the one you usually want is the
third:

- `Installed extensions…` - what this database has already created
- `Available extensions (not installed)…` - what the **server** ships that
  nobody has run `CREATE EXTENSION` for yet
- `Manage extensions…` - everything, with a filter and a count

Anything shipped by the server appears here whether or not it has been
created. Install it, change its version, move it to another schema or drop it
from the same list. Extensions the server does not have at all cannot appear:
they have to be installed on the machine first, usually by a package.

### 3.2.5 Roles and users

Sits beside Databases rather than under one, because roles are cluster-wide:
they are not owned by a database, and nesting them under one would say
otherwise.

**Right-click the ROLES AND USERS heading**:

- `+ Create role / user…`
- `Roles and users overview` - the full audit
- `Compare roles / users…` across two servers
- `Export ALL roles as .sql…`
- `Refresh roles`

**Click a role** to run a permissions audit on it. **Right-click a role** for
its own actions: privileges, `search_path`, memberships, create-like, copy
privileges, export DDL, drop.

Login users show 👤, group roles 👥.

#### Exporting a role {#role-export}

**Export DDL** (one role) and **Export ALL roles** write a script that
recreates the role elsewhere: `CREATE ROLE` with every attribute (inside a
block that skips a role that already exists), its comment, its memberships,
its `ALTER ROLE ... SET` settings - per database too - and, ticked by
default, **every privilege it holds in the cluster**: on databases,
tablespaces and server parameters (PostgreSQL 15+), and in each database on
schemas, tables and their columns, sequences, functions and procedures,
types and domains, languages, foreign-data wrappers and servers and large
objects, plus the default privileges it gives or is given (`ALTER DEFAULT
PRIVILEGES`). Each database is read separately and its part opens with
`\connect`, so the file is a psql script: `psql -f role.sql` against the
target, as a superuser. A database the export could not read - one the role
you are connected as may not connect to - is named at the end of the file
and in a notification, so a missing part is never silent. **What it owns**
adds `ALTER ... OWNER TO` for its schemas, tables, views, sequences and
routines, for when those objects are already on the target. Password hashes
are an opt-in of their own. Measured: a role with privileges of every kind in
two databases, exported, removed with `DROP OWNED` and `DROP ROLE`, and the
file run with psql - every privilege and every ownership came back as it was.

### 3.3 Schema browser (only visible when connected)

The schema browser is part of the **Servers** tree. Under the connection you are on, **Databases** lists every database on the server with a count. Expand one and it opens into its schemas, each with the number of objects it holds, and each schema into its objects, grouped by kind.

Every schema shows the same 40 kinds: tables, partitioned tables, views, materialized views, foreign tables, sequences, indexes, functions, procedures, aggregates, window functions, triggers, event triggers, rules, RLS policies, types, domains, range types, multirange types, extensions, languages, operators, operator classes, operator families, casts, collations, conversions, text-search configurations, dictionaries, parsers and templates, foreign-data wrappers, foreign servers, user mappings, publications, subscriptions, extended statistics, default privileges, large objects and tablespaces. A kind with nothing in it stays in the list, greyed out, so "there are none" never looks like "not shown". Objects that belong to no schema - event triggers, languages, casts, foreign-data wrappers, foreign servers, user mappings, publications, subscriptions, default privileges set without IN SCHEMA, large objects and tablespaces - are listed under **(cluster)**. The system schemas `pg_catalog` and `information_schema` show their tables, views, indexes, routines and types; the operators, casts and similar objects built into PostgreSQL are left out of the tree.

Relations open the way PostgreSQL holds them. A partitioned table opens into its partitions; a partition that is partitioned itself sits in place with its own partitions under it, at any depth. A table opens into the tables that inherit from it. Each of them is listed once, under its parent, not again among the tables. A partition that lives in another schema than its parent is shown under the parent with its schema, and in its own schema at the top with the parent named. Every table, view and materialized view also opens into its parts - indexes, constraints, triggers, rules, RLS policies, the sequences of its columns, statistics objects and the publications it is in - read when you open it, so a schema of thousands of tables does not load slower for them. What PostgreSQL copied from a partitioned table onto a partition (an index, a key, a row trigger) says *from* the original there, and is not in the schema's lists of indexes and triggers, which show what was declared; such a copy cannot be dropped on its own. A sequence that belongs to a column says so (*owned by* or *identity*).

Objects that belong to an extension are left out of every kind - the extension itself stays under Extensions - and the schema says how many it left out. Click that line to show them, each marked with its extension; click again to hide them.

Click a schema for a list of its tables with live and dead rows, sizes, index count, scans and the last vacuum and analyze; the filter box above that list narrows it by name. Click an object to open it in its own tab (DDL, Relations, Storage, Advisor, Data for a table). Right-click for everything else it can be asked to do - see [Object menus](#object-menus). The search box above the tree finds connections by alias, host, user or database; it does not search inside the tree.

## 4. Work tabs

Each tab carries its own connection, schema position, and SQL editor state. Tabs are persisted across app restarts.

- `+ New connection` (top-left tab bar): opens a new tab with no connection.
- Right-click any tab for: Switch / Refresh connection / Pin / Save as connection (for unsaved tabs) / Disconnect / Close.
- Pinned tabs survive "Close all".

#### Creating a connection

The New connection dialog holds the connection details and, below them, the
name, the group and the colour tag. The group dropdown lists the groups you
already have and offers **Create new group** and **Create sub-group** - a
connection can be filed where it belongs while it is being created, rather than
landing in `default` and having to be moved afterwards.

Three buttons end the dialog and all three do the same thing with the
connection:

| | |
|---|---|
| **Test** | checks the credentials and stays where it is |
| **Connect** | opens it in a work tab without saving it |
| **Save & connect** | writes the saved connection, then opens it |

"Opens it" means the same thing in both cases: a work tab of its own unless the
current one is still blank, tagged with the saved connection so the tab strip
above shows it, the sidebar marks it active, and every tool tab opened
afterwards belongs to that connection. Save & connect used to save the row and
stop there, leaving whichever tab was in front showing a connection it did not
know it had.

When more tabs are open than fit, arrows appear at either end of the strip
and a **slim slider** sits under the tabs. The slider only appears when there
is somewhere to scroll to; drag it, or click the empty track to page. It has
a lane of its own so it never covers a tab or its close button.

#### Role, service and SSH tunnel {#conn-role-service-ssh}

Three settings in the connection dialog that most connections leave empty,
named as pgAdmin names them.

- **Role** - the session runs `SET ROLE` to it right after every connect, on
  every connection the app opens for this server: the pooled ones, a reconnect,
  checks, background collectors, a held SQL transaction. Everything then runs
  as that role, its privileges decide what the app offers, and the header names
  it. The login user must be a member of the role; if the server refuses, the
  connect fails with the role's name and the server's reason, and nothing runs
  as the login user instead. A role picked in the header's role list still
  takes over for as long as it is picked.
- **Service** - a libpq service name. Host, port, database, user and password
  left empty in the dialog come from that service, and anything you do fill in
  overrides it. libpq looks for the service in the file named by
  `PGSERVICEFILE`, else in `.pg_service.conf` in your home directory
  (`%APPDATA%\postgresql\.pg_service.conf` on Windows), then in
  `pg_service.conf` in the directory named by `PGSYSCONFDIR`.
- **Advanced: SSH tunnel** (folded until a connection uses it) - for a database
  reachable only through a jump host. Give the SSH host, port (22 if empty),
  user and, if needed, the key file. The app starts your computer's own `ssh`
  program with a local port forward, so your ssh settings, `known_hosts` and
  `ssh-agent` apply. It logs in with a key only and never asks for or stores an
  SSH password; a key with a passphrase has to be loaded into `ssh-agent`. Host
  in the dialog is then the database host as the jump host sees it, often
  `localhost`. A jump host whose key you have not accepted yet is refused with
  the command to run once in a terminal to check and accept it. The tunnel
  closes with the last tab that uses it, and when the app exits.

## 5. Main views

The top tab strip switches between views. Each view operates against the active connection of the active work tab.

Every drop-down list can be searched. Click it, or just start typing while it has the focus, and a search field opens on top of it: the list narrows with every key, matching any part of an entry, without regard to case or accents ("sifra" finds "šifra"), and every word typed has to appear. Up and Down move, Enter picks, Escape closes and leaves the choice as it was, and Tab takes the highlighted entry and moves on. Lists of only two or three entries open the system's own list, where typing a letter jumps to the first entry that starts with it.

### 5.1 Checks

The default landing view. Top menu bar exposes the check catalogue organised by group:

- Cluster health: db-health, db-locks, db-blocking, db-long-tx, db-idle-in-tx, db-conn-limits, db-vacuum, db-autovacuum, db-checkpoints, db-replication, db-repl-slots, db-logical-replication
- Connections & queries: db-connections, db-users, db-active-queries, db-long-queries, db-stat-statements, db-query-history, db-temp-queries, db-temp-files
- Storage & performance: db-sizes, db-schema-sizes, db-table-sizes, db-index-sizes, db-bloat, db-bloat-indexes, db-bloat-breakdown, db-pgstattuple, db-buffer-cache, db-tablespaces
- Tuning & diagnostics: db-settings, db-cache-hit, db-cache-advisor, db-index-usage, db-index-unused, db-index-advisor, db-duplicate-indexes, db-table-stats, db-replica-identity, db-workload-report, db-ash, db-awr-snapshot, db-trace-session, db-errors
- Security / permissions: db-roles-list, db-role-perms (pick target), db-ownership (pick scope)
- Also reachable: db-sequences, db-seq-usage, db-seq-sync (sequence tools in the schema browser), db-setup-advisor (Help menu)

**Three reports answer "where has the space gone", at three levels.**
`db-sizes` gives every database in the cluster; `db-schema-sizes` breaks the
**connected** database down by schema - total, with data and indexes shown
separately, each schema's share, object counts and its largest table;
`db-table-sizes` goes down to the individual table; `db-index-sizes` goes down
to the individual index. How to read all four, and what each total does and
does not include, is under *Where the space goes* below.

Schema sizes covers only the database you are connected to, and that is a
limit rather than an oversight: `pg_namespace` is per-database, so listing
every database's schemas would need a connection to each one. The total counts
everything in the schema that occupies disk - tables, their TOAST, their
indexes, materialised views and sequences - each byte once.

Each check opens its own result tab. Tabs survive view switches so you can compare several checks side by side. Each tab has two panels:

- Structured: rich tables, advisor cards, drilldowns
- Raw output: plain text terminal log

Many checks include "advisor" sections with severity-tagged findings (OK / INFO / WARN / CRIT) and concrete next-step suggestions.

#### Adding up what a report shows

Every report table sorts by a column and filters by typing. Two more things
answer the question these reports are usually opened for - "is this worth
doing".

**A total under the table.** Any column whose values read as sizes or as plain
numbers is summed, and the sum follows every filter. Narrow a bloat report to
one schema and the total tells you how much space reindexing that schema would
return; narrow it further to one table and you can see whether the schema-wide
job is worth it or whether one table carries almost all of it.

The numbers are read back from what is printed, because for most of these
checks that is all there is - PostgreSQL's own size formatting and nothing
underneath it. The sum is therefore a sum of the values as shown, rounded to
the unit they are printed in: forty indexes rounded to the megabyte can be
tens of megabytes out either way. Close enough to decide with, not a figure to
quote.

Percentages, ratios and averages are never summed - adding them produces a
number that means nothing, and printing it would invite reading it. The column
name is what says so: a bloat report prints `bloat_pct` as a bare `44.9`, with
the sign nowhere in the cell.

The SQL editor's result grid and the schema browser's Data tab total sizes
only. A report knows what its columns are; a `SELECT` you have just typed does
not, and a sum under a column of ids is noise.

**Picking values in a column.** The caret beside each column's filter box
opens the list of values actually present in that column, with a count beside
each. Ticking three schemas whose names have nothing in common is not
something a substring filter can do, and reports about storage are full of it
 - one schema out of twenty, four tables out of nine hundred. The typed filter
and the ticked list both apply, so "tables in these two schemas whose name
contains _log" is one gesture each.

A column with more than two hundred distinct values says so instead of
rendering a thousand checkboxes.

#### Which database a report is about

Every one of these reports is scoped to one database, and the line above the
results says which - the check's name, the database, and the connection it ran
on. It comes from the tab rather than from the live connection, so a tab that
ran against `shop` keeps saying `shop` after the connection is switched
underneath it.

#### One index, and what to do about it

The reports name a bloated or an unused index. Clicking the name opens the
**index advisor** for that one index, and it is also on every index in the
schema browser's index list.

It answers the question the report leaves open. The bloat estimate is the same
arithmetic as the report you clicked from, scoped to the one index so the two
can never disagree. Beside it: how many times it has been scanned since the
statistics were last reset, its size against its table's, and a check against
every other index on the same table - another index on exactly the same
columns with the same method, or one whose leading columns are this index's
whole list, in which case the planner can use the wider one for anything this
one answers.

Constraint indexes are read differently, and this is why "never scanned" is
not by itself a reason to drop anything. An index backing a PRIMARY KEY, a
UNIQUE or an EXCLUDE constraint enforces it whether or not a query ever reads
it, and `DROP INDEX` refuses it outright; the same goes for a table's replica
identity, which logical replication needs. The advisor says so and does not
offer the drop.

What it offers to run is written for one index and no more:

| | |
|---|---|
| `REINDEX INDEX CONCURRENTLY` | builds a replacement beside it and swaps - no long lock, needs room for both copies, cannot run inside a transaction (PostgreSQL 12+) |
| `REINDEX INDEX` | holds an ACCESS EXCLUSIVE lock on the table for the whole rebuild |
| `DROP INDEX CONCURRENTLY` | offered only where the index is droppable |
| the definition | keep it before dropping - that is the only copy |

Each opens in the same dialog every generated statement in this application
opens in: look at it, copy it, save it, or open it in the SQL editor. The two
rebuilds also offer **Run as a background job**, which puts it in Background
jobs with its output streaming, so a rebuild that takes twenty minutes is not
tied to a window staying open.

Before this the only REINDEX the application could issue was per table, per
schema or per database - so a report naming one bloated index could only be
acted on by rebuilding every index on its table.

#### Notable checks {#notable-checks}

**db-health** (recommended first stop): one-screen pulse check across connections, idle-in-tx, long-active-tx, blocking, cache hit, replication lag, WAL archiving, bloat, vacuum pending, sequences, unused indexes. Drill-downs per category surface offending rows.

**db-cache-advisor**: top-line heap/index/toast hit ratios, hot tables/indexes/TOAST with concrete remediation, shared_buffers sizing check vs cluster size, effective_cache_size sanity, pg_buffercache snapshot when the extension is installed.

**db-index-advisor**: seq-scan analysis with per-table candidate column lists. Cross-references pg_stat_statements for hot queries, surfaces leading-column gaps, partial-index opportunities, composite FK coverage. Generates CREATE INDEX statements you can copy.

**db-vacuum**: per-table vacuum/analyze status with autovacuum advisor. Shows live, dead, mod_pct, heat label (very-hot / hot / medium / default), tags (WRAPAROUND risk, autovacuum disabled, insert-only), and a "recommended" reloption set vs "current" for each candidate. Since 1.0.x of October 2026 it also looks the other way: how often each table is vacuumed (runs per day and the gap between them, with the time they take on PostgreSQL 18), a table vacuumed hourly or more often - and whether that is fine (see [when vacuuming often is fine](#vacuum-bloat-and-table-statistics)), a table vacuumed again and again with no effect because something holds a snapshot older than its dead rows (named, with what to end), a table whose every vacuum is aggressive, and - in the *Autovacuum cost and I/O* section - autovacuum's pace settings and, on PostgreSQL 16 and newer, its share of the cluster's reads and writes from `pg_stat_io`. A recommendation never tightens the thresholds of a table already vacuumed hourly. It also says when vacuum cannot get to a table at all: a session holding a lock VACUUM cannot share (SHARE UPDATE EXCLUSIVE or stronger - a long ANALYZE, CREATE INDEX CONCURRENTLY, ALTER TABLE, LOCK TABLE) in a transaction open a minute or more, on a table past its trigger. A plain autovacuum skips such a table and tries again a naptime later, so nothing else said about it happens; the finding names the session (pid, user, application, client, how long its transaction has been open, its statement) and comes first. A writer idle in a READ COMMITTED transaction counts as a snapshot holder too: between statements it has no snapshot, but its transaction ID holds the horizon.

**db-stat-statements**: top 25 queries with per-row advisory flags (plan instability, work_mem spill with concrete bytes-per-call, low cache hit, slow on every call, bad-case path). Top-queries widget lets you copy the full query.

**db-replica-identity**: tables without PRIMARY KEY plus tables with non-default REPLICA IDENTITY. Cross-references pg_publication_tables to distinguish CRITICAL (in a publication, will break on UPDATE/DELETE) vs advisory. Recommends ALTER in priority order: ADD PRIMARY KEY, REPLICA IDENTITY USING INDEX, REPLICA IDENTITY FULL.

**db-errors**: cloud-aware error watcher. Always samples pg_stat_database counters over a 5-second window (rollbacks, deadlocks, conflicts, sessions_fatal/killed/abandoned on PG14+). On self-hosted clusters also tails the last 64 KB of the server log via `pg_current_logfile()` + `pg_read_file()`, parses ERROR/FATAL/PANIC lines. On managed clouds the log tail is skipped with provider-specific guidance (RDS console, log_fdw, Cloud Logging, Azure Server Logs, Supabase / Neon / Heroku log pages).

**db-roles-list**: roles + memberships listing plus a security findings sweep: Superuser + login, BYPASSRLS without superuser, CREATEROLE on PG <16, login without conn limit and no VALID UNTIL, expired login roles, MD5 password hashes, REPLICATION + LOGIN, PUBLIC has CONNECT on databases, PUBLIC has CREATE on schemas, SECURITY DEFINER routines owned by superuser with PUBLIC execute.

**db-role-perms** (pick target role): 19 listing sections (overview, memberships, db/schema/table/column/sequence/function/type/fdw/server/language/default privileges, RLS) with per-row interpretation column ("via PUBLIC", "WITH GRANT OPTION", "no USAGE on schema X - grant ineffective", etc.) and a cross-section findings struct (effective superuser via chain, BYPASSRLS / CREATEROLE / REPLICATION inheritance, table grants without schema USAGE, etc.).

**db-ownership** (pick scope): listings for schemas / tables / sequences / functions / types / FDWs / languages / publications / subscriptions plus findings (app objects owned by superuser, SECURITY DEFINER routines owned by superuser, dangling owner OIDs after a DROP ROLE without REASSIGN, public schema with non-superuser owner, schemas with 3+ distinct object owners).

**db-settings**: curated guide for ~70 tuning-relevant pg_settings parameters with DBA rationale, rule-of-thumb, and status callbacks (warn / danger badges). Editable via the Server view.

**db-setup-advisor**: probes every server-side configuration DBApguin features rely on (extensions, GUCs, role grants), reports OK / PARTIAL / MISSING per feature, and prints the exact SQL or parameter-group line to fix each gap. Provider-aware: on managed clouds the remediation hint says "set in the parameter group" instead of suggesting ALTER SYSTEM. Run this first when setting up DBApguin against a new cluster, and again after any postgresql.conf change to verify.

The list above is what each check produces. Below is how to read the ones
whose output takes interpreting, and the mistakes they exist to catch.

#### The health overview {#the-health-overview}

`db-health` is one screen: a line per category - connections, idle-in-transaction, long transactions, blocking, cache hit, replication lag, WAL archiving, vacuum lag, wasted space, vacuum backlog, sequences near their ceiling, unused indexes, durability, partitions - each green, amber or red. Run it first; it is often the only check you need. A finding that is about a table names it in the **table** column - the worst one when it counts several - and that name is a link: it opens the table in the Schema browser.

Amber means "worth a look", red means "production is affected now". Click a category to open its rows: blocking expands to blocker/waiter pairs, idle-in-transaction to the sessions and how long each has pinned its snapshot. A red line with nothing in the drill-down was tripped by a total rather than one row - a cache hit ratio pulled down across the whole database, say. The trap is reading amber as noise: amber wasted space is a VACUUM you can schedule for tonight; the same table red next month is `pg_repack` on a busy system.

**Vacuum lag and wasted space are two separate rows because they measure two different things.** *VACUUM LAG (dead tuple %)* is `n_dead_tup / (n_live_tup + n_dead_tup)` - the share of a table's rows that are dead. It says autovacuum is behind. It is **not** a space measurement: a table can be 4% dead and still be sitting on a terabyte of free space that was never returned. *WASTED SPACE (est.)* is that terabyte - estimated bytes the table holds but does not use, heap plus TOAST plus every index, the same combined estimate `db-bloat-breakdown` and the Advisor tab use. A table shows up as `4.3% dead tuples but 1.3 TB wasted (39x its live data)` when both are true at once. The row fires on either measure: 30% dead (CRIT at 50%) or, whatever the share, 1,000,000 dead rows in one table (CRIT at 10,000,000) - 29% of a billion-row table is still a vacuum that is far behind.

**WAL ARCHIVING** says whether the archiver is shipping WAL now, from `pg_stat_archiver` and the archive settings - the same reading the header alarm and the archive cards in Key settings make. *Failing* (red) means the last attempt failed and nothing has been archived since: the row names the segment that failed and when, the last one that was archived, and how many segments wait in `pg_wal`. `archive_mode` on with neither `archive_command` nor `archive_library` set is amber - nothing is archived and every finished segment stays in `pg_wal` - and red once 10 GB waits. Both `archive_command` and `archive_library` set is red: from PostgreSQL 16 the archiver refuses to run at all, while its statistics look unchanged. The drill-down lists the command as the server has it and what to do: read its own error in the server log (`archive command failed with exit code`), run it by hand for the segment named, and never delete files from `pg_wal` - once the command works the archiver catches up by itself. Counting the waiting segments needs PostgreSQL 12 and superuser or `pg_monitor`; without them the count is left out.

**DURABILITY** reads the settings that decide whether a committed transaction survives a crash. It is red for `fsync = off` or `full_page_writes = off` - the two settings with which an operating-system crash or a power cut can corrupt the cluster itself, not only lose its last transactions - and for a `synchronous_standby_names` that asks for more synchronous standbys than are connected while commits wait for them (`synchronous_commit` at `on`, `remote_write` or `remote_apply`): every COMMIT then hangs, and the row says how many sessions are waiting right now (wait event `SyncRep`) and for how long. With the server-wide `synchronous_commit` at `local` or `off` the same missing standby is amber: commits do not wait now, but a session that sets it back to `on` does. `synchronous_commit = off` and data checksums turned off are blue (INFO): the first can lose the last transactions already reported as committed (up to three `wal_writer_delay`) but corrupts nothing, a trade often made on purpose; the second means a page the disk damaged is read back without an error - turning checksums on needs the cluster stopped (`pg_checksums --enable`, PostgreSQL 12+). The Key settings cards for the same five settings give the same verdict. On a standby the synchronous-standby part is skipped: it applies to a primary only.

**PARTITIONS** is the runway of the range-partitioned tables in the connected database: amber when a last range ends within 7 days or a RANGE table's DEFAULT partition holds rows, red when the last range has ended - see [Partition runway](#partition-runway).

The **MIGRATIONS** row appears when the database has a migration tool's history table: red for a failed migration, amber or red for a lock its process left behind (see *Migration history*).

#### Connections, sessions and load {#connections-sessions-and-load}

These read `pg_stat_activity` and `pg_stat_statements` from different angles.

`db-connections` is the count by state - active, idle, idle in transaction, and waiting. **Idle in transaction** is the number to watch: those sessions hold a transaction open, pinning the oldest snapshot vacuum can remove and often holding locks, and a rising count is usually an application that forgot to commit or a pool with no idle-in-transaction timeout. `db-users` breaks the same connections down by role and application, which is how you find *which* service is the one leaking them.

`db-conn-limits` is `max_connections` against what is used, with the breakdown by state and the `superuser_reserved_connections` headroom. Only client backends count against the limit - background workers do not - so the number here can be lower than a raw `pg_stat_activity` count. The header's connection alarm counts the same way. **Waiting** is an active statement held up by something other than its client - a lock, an LWLock, I/O; an idle session also shows a wait event (`Client: ClientRead`, waiting for its next command) and is not counted. When it is close to the ceiling the fix is almost never a higher `max_connections` (each slot costs memory); it is a connection pooler, or finding the idle-in-transaction sessions above.

`db-active-queries` is what is executing right now, longest first, with the backend PID and wait event. `db-long-queries` is the same list filtered to statements running over five seconds - the ones worth interrupting or explaining. Both are a snapshot: a query that finished between refreshes is gone.

`db-stat-statements` is the cumulative view - the top statements by total time since the last `pg_stat_statements_reset()`, each normalised so `WHERE id = 1` and `WHERE id = 2` are one row. Per-row advisory flags call out plan instability, `work_mem` spill (with the bytes-per-call), a low cache-hit ratio, and statements slow on every single call rather than on average. Total time is the column to sort by: a query taking 3 ms a million times a day is a bigger cost than one taking 20 seconds twice. The widget copies the full statement text. Read `db-query-history` (section 6.6) for the same data cross-referenced with a clock.

After the top statements comes **Optimistic locking - stale writes**: every UPDATE and DELETE in the view whose WHERE compares a version column with a parameter - Hibernate's `@Version`, Rails' `lock_version`, Entity Framework's `xmin` or `ConcurrencyStamp`, a hand-written `version = $n`, an `updated_at` check - with `stale_%` = 1 - rows / calls: the share of its writes that found the row changed (or deleted) and wrote nothing. The application reloads and retries each of those, a loop the database side never sees otherwise. A statement is amber when at least 5% of at least 100 calls wrote nothing, or when 1,000 calls did whatever the share; the same statement in the top 25 carries the same flag. A statement that writes more rows than it is called (a batch with a version condition) is listed without a share, since rows / calls is then not one row per call.

`db-temp-queries` and `db-temp-files` are about spills to disk. `db-temp-files` is live - files being written right now, grouped by PID, so you catch the query spilling as it happens. `db-temp-queries` is cumulative from `pg_stat_statements` - the chronic spillers, which is a `work_mem` that is too low for that query's sort or hash, or a join order the planner got wrong. A little temp is normal on a reporting query; the same statement spilling gigabytes on every run is the one to fix.

#### Locks, blocking and stuck transactions {#locks-blocking-and-stuck-transactions}

`db-blocking` is the one to open first in a "the app is frozen" incident: it pairs each blocked session with the exact session blocking it (via `pg_blocking_pids()`), so the head of the chain - the transaction everyone else is waiting on - is one row, with its query and how long it has held the lock. Cancel or terminate that one and the queue drains.

`db-locks` is the raw `pg_locks` view - every awaited lock, and every lock the sessions blocking them hold (on PostgreSQL 9.6 and newer; older servers pair table-level waits only). Use it when `db-blocking` is empty but something still feels stuck, or to see *which* lock mode is involved (a `ShareLock` waiting on a `RowExclusiveLock` is a foreign-key insert against a row being updated elsewhere; an `AccessExclusiveLock` request behind a long `SELECT` is a DDL statement or a `VACUUM FULL` that will freeze the table until the read finishes).

`db-long-tx` lists transactions open longer than a minute, whatever they are doing. A long transaction that is *working* is a slow query; one that is **idle in transaction** is an application bug, and it is worse than a slow query because it holds its snapshot and its locks while doing nothing. `db-idle-in-tx` is that second case on its own, with how long each session has been parked mid-transaction. Either way the cost is the same: `VACUUM` cannot clean up any row version newer than the oldest open transaction, cluster-wide, so one forgotten `BEGIN` on a laptop can bloat every table on the server.

The **Sessions** view carries the transaction angle further than the checks. A session's detail panel shows *Snapshot age (backend_xmin)* - how many transactions back its snapshot is pinned - and, when it applies, a red line saying **this session currently defines the cluster xmin horizon**: autovacuum is blocked everywhere until its transaction ends. The Live sessions list tags that same session, and the Blocking view walks the lock-wait graph into a tree so the one transaction holding everyone else up is a single row.

#### Prepared (two-phase) transactions {#prepared-two-phase-transactions}

`db-prepared-xacts` covers the one kind of transaction that has no session at all. `PREPARE TRANSACTION 'gid'` writes the transaction to disk and returns; a later `COMMIT PREPARED 'gid'` or `ROLLBACK PREPARED 'gid'` finishes it. In between it keeps every lock it held and pins the cluster xmin horizon - and nothing in `pg_stat_activity` shows it, because it is not a connection. An external transaction manager (a Java app server's JTA, Windows MSDTC, an XA driver) that prepares a transaction and then crashes or loses track leaves it prepared for ever. That is the classic "the database keeps bloating and every session looks fine" incident.

The check lists each one - `gid`, owner, database, how long ago it was prepared, its transaction's age, and the tables it locks - and flags anything older than ten minutes as a likely orphan. Resolve one connected to **its own database**: `COMMIT PREPARED 'gid'` if the work should land, `ROLLBACK PREPARED 'gid'` if it should not.

**Most servers cannot have prepared transactions at all.** `max_prepared_transactions` defaults to `0`, and the check says so plainly rather than showing an empty table - raise it above zero only if something actually needs `PREPARE TRANSACTION`. When it is enabled, a prepared transaction that is blocking a live session now shows up in the Blocking view as its own node (labelled `prepared 'gid'`, with the two resolving statements to copy), and in `db-locks` its held locks are attributed to the gid instead of vanishing.

#### Migration history {#migration-history}

`db-migrations` (Cluster health menu) reads the history tables the migration tools keep in the connected database - Flyway (`flyway_schema_history`, `schema_version` before Flyway 5), Liquibase (`databasechangelog`, `databasechangeloglock`), Alembic (`alembic_version`), golang-migrate and Rails (`schema_migrations`, told apart by golang-migrate's `dirty` column), Django (`django_migrations`), Prisma (`_prisma_migrations`), knex (`knex_migrations`, `knex_migrations_lock`) and Entity Framework Core (`__EFMigrationsHistory`) - in any schema. For each it says what was applied last, and it flags the states that stop the next deploy, with the row that shows it and how to clear it:

- **CRIT - a failed migration:** Flyway `success = false` (a migration that could not run in a transaction, such as `CREATE INDEX CONCURRENTLY`, failed half-way; Flyway refuses to migrate until `flyway repair`), golang-migrate `dirty = true` (fix the schema by hand, then `migrate force`), Prisma a migration that started and neither finished nor was rolled back, ten minutes or more ago (`prisma migrate resolve`).
- **A lock left behind by a process that died:** Liquibase `databasechangeloglock.locked = true` is a warning once it has been held **5 minutes** and critical at **60 minutes** (a long data migration holds it legitimately, so the row names who took it and when; `liquibase releaseLocks` frees it); knex `is_locked = 1` is a warning (it records no time).
- **INFO:** Flyway migrations applied out of order (`outOfOrder=true` - allowed, but this database ran them in an order a fresh install would not), more than one Alembic head, Liquibase changesets recorded as `FAILED` under `failOnError=false`.

A table this login may not read is named as found and not guessed at. The Health overview carries the worst of these as its **MIGRATIONS** row, and only when the database has such a table.

#### Queue tables and LISTEN/NOTIFY {#queue-tables-and-listen-notify}

`db-queues` (Connections & queries menu) finds the tables used as job queues: the ones the `FOR UPDATE SKIP LOCKED` statements in `pg_stat_statements` name (this database only), and the tables of the libraries that keep their jobs in PostgreSQL - pgmq, PgQ, que, graphile-worker and River. For each: the jobs waiting (only those that are due, not ones scheduled for later), how long the oldest has waited, failed jobs where the library records them, live and dead rows, size, and the last vacuum. A table found only through its statements has no known "waiting" state - many keep finished jobs - so its oldest row is shown and not graded.

- **Dead rows** are walked past by every dequeue, which is why a queue table is the first to suffer when vacuum falls behind or something pins the xmin horizon. Warning at **50%** of the table with at least **10,000** dead rows, or **1,000,000** on their own; critical at **90%** with **100,000**, or **10,000,000**. The text states the share and the count.
- **Waiting:** warning when the oldest due job has waited **5 minutes**, critical at **1 hour**. PgQ consumers are graded the same way on their lag.
- **LISTEN/NOTIFY:** `pg_notification_queue_usage()` (PostgreSQL 9.5+) says how full the one cluster-wide notification queue is. It is trimmed only up to what the slowest listener has read, and a session reads only between transactions, so a listener left idle in a transaction lets it grow until every `NOTIFY` fails at commit. Warning at **10%** of its maximum or **512 MB**, critical at **50%** (where PostgreSQL starts warning in its log) or **4 GB**; the maximum is 8 GB, or `max_notify_queue_pages` from PostgreSQL 17. The sessions whose last statement was `LISTEN` are listed with their state - PostgreSQL has no view of listeners, so that is the closest it gets. The same lines light the header's **LISTEN/NOTIFY queue** lamp, shown on a cluster where somebody listens.

The counts are bounded (a million rows) and each probe has five seconds, so a huge backlog without an index is reported as not measured rather than scanned.

#### The cache and index advisors {#the-cache-and-index-advisors}

`db-cache-hit` is the one number: the fraction of block reads served from `shared_buffers` since the last stats reset, for this database. It is a cumulative average over the whole life of the stats, so a one-off cold report an hour ago still drags it down now - it tells you almost nothing on its own. `db-cache-advisor` is the version worth acting on.

`db-cache-advisor` reports the heap, index and TOAST hit ratios, then the tables and indexes doing the most physical reads, `shared_buffers` against the size of the data, and an `effective_cache_size` sanity check. A hit ratio below ~99% only matters if the misses are concentrated - one cold reporting table read once a day is not worth tuning for. Act on the hot-object list, not the top-line number.

`db-index-advisor` finds tables with a high sequential-scan share and lists candidate columns from the predicates in `pg_stat_statements`, including leading-column gaps in existing indexes and foreign keys with no covering index. It writes the `CREATE INDEX` for each. Two traps: a small table is meant to be seq-scanned, so ignore candidates under a few thousand rows; and every index is a write cost, so add the one the hot query needs, not all of them. To cost a specific index before building it, use the index tryout (section 6.5.5), which plans it with HypoPG.

With AWR snapshots of the database stored, `db-index-advisor` adds three history sections after an **Evidence** block that names the snapshots and the ASH days it used. **Sequential scans across the stored windows**: per table, scans and rows read a day, rows per scan, and the first half of the history against the second; *growing* means the later half is at least twice the earlier and at least 100 a day, or at least 10,000 more a day, and a table is listed when it is growing or its sequential scans read at least 100,000 rows a day. Beside each, the statements that name it and read the most from disk. **Statements reading the most from disk**: `shared_blks_read` from `pg_stat_statements` summed over the windows, per call, with the busiest window and how often ASH caught the statement waiting for I/O (matched by query id from PostgreSQL 14). **No scans over the stored window**: each index's scans summed over the windows (a statistics reset in between starts a new count, nothing is lost), with the window stated. Over at least **7 days** an index with no scans is a *candidate to review*, under that the window is *too short to judge*, and every row carries the caveat that a monthly, quarterly or yearly report may be its only reader. No `DROP` is offered there, and primary keys, unique indexes and foreign-key indexes are never candidates: they are listed apart as kept, with the reason.

#### Index housekeeping {#index-housekeeping}

Three checks find indexes that cost writes and disk while earning nothing back. All three read `pg_stat_user_indexes`, so their scan counts are **since the last statistics reset** - a `pg_stat_statements_reset()` or a `pg_stat_reset()` last week means "never scanned" only covers a week.

`db-index-usage` is the full picture: every index with its scan count, size, and the ratio of scans to table writes. `db-index-unused` is the subset that matters - indexes with zero scans - and it already excludes the ones a zero is normal for: those backing a `PRIMARY KEY`, a `UNIQUE` or an `EXCLUDE` constraint, and a table's replica identity. Those enforce something whether or not a query reads them and `DROP INDEX` refuses them, so a drop list that included them would be dangerous. It also leaves out every index on a **foreign key** column: it reads as never scanned because `DELETE` and `UPDATE` on the parent table use it internally, and without it each of them scans the child table under a lock. Those are listed apart, each with the foreign key it backs, and are never offered for dropping - here, in the health overview, in index bloat and in the table Advisor. What is left is a **candidate to review, not a verdict**: the check writes the `DROP INDEX CONCURRENTLY` for each and says how long its counters cover, because an index used only by a month-end, quarterly or yearly report reads as unused for the whole time in between - confirm over a window that includes one.

`db-duplicate-indexes` finds indexes that repeat each other. **Identical** ones - same table, columns, operator classes, expressions and predicate - are shown as a group, and the check names the one that **stays** and why: the one that enforces or serves something (a primary key, a `UNIQUE` or `EXCLUDE` constraint, a unique index, the replica identity, a foreign key's index), else the most scanned. The others are marked **redundant - identical to** the one that stays. When two of them both enforce something (two `UNIQUE` constraints on the same columns), it says so instead: removing one means changing a constraint. It also lists a plain index whose columns are the **leading columns of a wider one** with the same operator classes - the wider one answers the same lookups, a little bigger to read; unique and constraint indexes are never listed there, and when the narrower one backs a foreign key the wider one still covers it. The check writes no `DROP` - a redundant index costs writes and disk, and the decision is yours. Keep the definition before removing one - it is the only copy.

To act on a single index the reports name - the exact `REINDEX` or `DROP`, its scan history, and a check against every other index on the same table - click its name to open the per-index advisor (see *One index, and what to do about it* above).

#### Where the space goes: the size reports {#where-the-space-goes}

Four reports answer "what is using the disk", each one level finer than the
last. They are pure measurement - no advisor, no recommendation - so the only
way to misread them is to misread what a number includes.

`db-sizes` - **every database in the cluster.** `pg_database_size()` per
database, plus the live connection count (client backends only - autovacuum
workers and background processes carry a `datname` too and counting them makes
an idle database look busy), the cache-hit ratio since the last stats reset,
and cumulative temp-file usage. Template databases are left out. This is the
only report here that sees across databases; the connection you are on does
not matter to its numbers.

`db-schema-sizes` - **the connected database, broken out by schema.** Total
per schema, table data and index bytes shown separately, each schema's percent
of the database, object counts, and the largest single table in each. The
total counts everything in the schema that occupies disk once: tables, their
TOAST, their indexes, materialised views, sequences. It cannot cross into
other databases because `pg_namespace` is per-database.

`db-table-sizes` - **the individual table**, top 25 by total size.
`total` is `pg_total_relation_size()` - heap **plus** its TOAST table and
TOAST index **plus** every index on it **plus** the free-space and
visibility maps. `table_only` is the heap alone; `indexes` is
`pg_indexes_size()`. So `total` minus `table_only` minus `indexes` is the
TOAST and the maps, which for a table with wide `text`/`jsonb` columns can be
most of the size. Partitions are rolled up to the partition root - a
partitioned table shows as one row with a `partitions` count, not as a wall of
child rows with the parent reading zero. `live_rows` and `dead_rows` are the
statistics collector's **estimates**, not a `count(*)`; `stats_age` is how
long ago the last `ANALYZE` refreshed them, and a large `stats_age` means the
row counts are stale in whichever direction the table has moved since.

`db-index-sizes` - **the individual index**, top 50 by index size, each with
its parent table's size beside it so an index that has grown larger than the
data it indexes is obvious at a glance. That ratio is the signal: a btree
index bigger than its table is either indexing very wide keys or is bloated
(see the next section), and either way it is worth a look. A GIN or GiST index
legitimately dwarfing its table is normal - they store far more than one entry
per row.

The trap common to all four: these are **sizes on disk**, which include dead
tuples and free space that a `VACUUM FULL` or `pg_repack` would reclaim. A
table reading 4 GB is not 4 GB of data if half of it is bloat. Cross-read a
surprising size against `db-bloat-breakdown` or `db-pgstattuple` before
concluding you need a bigger disk.

#### Bloat: the estimate and the real number {#bloat-the-estimate-and-the-real-number}

"Bloat" is space a relation occupies on disk beyond what its live rows need -
dead tuples autovacuum has not reclaimed, and pages left partly empty after
deletes and updates. Three reports address it, from fast-and-approximate to
slow-and-exact, and every estimate-based one of them - the per-table
breakdown, the index ranking, the Advisor tab, and the health overview's
*WASTED SPACE* row - is computed by the same code and lands on the same
number for the same relation, because they all call into one shared module
rather than each working it out themselves. **Read the estimates as "worth
investigating", never as a measurement.**

`db-bloat-breakdown` is where to start. Per table, it reads real sizes
(`pg_relation_size`, `pg_indexes_size`, `pg_total_relation_size`,
`pg_total_relation_size` on the TOAST relation) and combines them with an
*expected* ("packed") size computed from `pg_stats` (column count, average
width, null fractions) for the heap, the same per-entry model for every
btree index, and the TOAST table's own dead-tuple ratio - one `est_waste` /
`est_waste_pct` for the **whole table**, heap and TOAST and every index
added together, not the heap alone. That is the fix for a table that used to
show "0% bloat" here while its indexes alone carried hundreds of gigabytes
of estimated waste: the number now includes them. It prints a
`recommendation` column derived from the same totals it shows, so the advice
and the figure can no longer disagree with each other. Read that column as a
**prompt, not an instruction** - it does not know whether you have
`pg_repack` installed. A "VACUUM FULL or pg_repack" line on a 20 MB table is
satisfied by a plain `VACUUM`; the same line on a 200 GB busy table means an
online rebuild and a maintenance plan.

**One component's estimate can be unreliable without blanking the row.** The
packed-size formula for a single component - usually the heap on a big or
long-unanalyzed table - can come out *larger* than that component's real
size: stale statistics, or rows unusually wide or heavily TOASTed for
`pg_stats`' per-column model to fit. That is not "no waste", it is "this
component's estimate does not apply here" - and it does not throw away
what the *other* components (TOAST, each index) still know. `est_waste`
becomes a **lower bound** instead: marked `≥` and naming what is missing,
e.g. `≥ 448 GB (heap unknown)`, rather than a fabricated zero or a blank
"n/a" - the real total is that figure or higher, never less. `est_waste_pct`
carries the same `≥` marker when it applies. `ANALYZE`, or `db-pgstattuple`
below, is the way past it for the specific missing component - the row's
own note names which one and why: never analyzed, statistics stale by N
days with a given share of rows changed since (both point at `ANALYZE`), or
statistics that are current yet still do not fit the table's row shape
(points at `db-pgstattuple` instead, since re-running `ANALYZE` would not
change anything). This same "known components stay visible, missing ones
are named, nothing gets blanked" rule applies everywhere a breakdown sums
several components - the health overview's *WASTED SPACE* row included.

**`est_waste_pct` and `dead_pct` are different numbers on purpose.**
`est_waste_pct` is space: bytes the table (heap, TOAST, indexes) holds but
does not use. `dead_pct` is `n_dead_tup / (n_live_tup + n_dead_tup)` - a
tuple count, and only a measure of how far autovacuum is behind. A table
churned by `UPDATE` can be 4% dead and 90% wasted at once. `idx_to_heap` is
total index bytes over heap bytes, where much above 1 is worth an
unused/duplicate-index audit.

`db-bloat-indexes` answers a different question: not "which table is worst"
but "which single index, anywhere in the cluster, is worth a `REINDEX`
right now" - the same per-index arithmetic as the breakdown's index rows,
ranked cluster-wide instead of grouped by table. It assumes `fillfactor` 90
and covers **btree only** - GIN, GiST, BRIN, hash and expression indexes get
a real size and no estimate, which is the estimator declining rather than a
clean index. **`VACUUM` never compacts an index**; the fix it names is
`REINDEX` (`... CONCURRENTLY` on PostgreSQL 12+, checked against the
connected server so an older one is not told to run a statement it does not
have), one line per index. A table built with `fillfactor` below 100
deliberately, to keep HOT updates cheap, will show that free space as
"bloat" here too - it is the room you asked for, not waste.

`db-pgstattuple` is the real number, for both halves. It reads **every
page** of each table through the `pgstattuple` extension for the true
dead-tuple and free-space percentages, then `pgstatindex()` on that table's
btree indexes for a real leaf-page-density figure per index - no `pg_stats`
model on either side, an actual page scan. It is exact and it is as slow as
that scan implies: the cost is the table's size in I/O, every time. Use it
to confirm a specific table the estimate flagged - or one it flagged as
unreliable - not to survey a database. The size-range parameters are a time
budget: the default skips tables under 1 MB (bloat there is a rounding
error) and over 5 GB (raise it when you specifically want the exact figure
on a big table and can wait). The extension also offers `pgstattuple_approx`,
which skips pages the visibility map marks all-visible and so runs far
faster on a well-vacuumed table at the cost of some precision - worth
knowing about when the full scan is too slow.

**A practical order:** `db-bloat-breakdown` to see the shape of it across
every table, `db-bloat-indexes` for a cluster-wide ranked list of index
suspects, `db-pgstattuple` on the one or two tables you are actually going
to act on (or any row flagged unreliable). Then the per-index advisor
(above) for the exact `REINDEX` or `VACUUM` statement.

#### Shared buffers: what is actually cached {#shared-buffers-what-is-actually-cached}

`db-buffer-cache` opens `pg_buffercache` and shows which relations occupy
PostgreSQL's shared buffer pool right now - the top 30 by buffer count, with
each relation's total size, its share of `shared_buffers`, and how much of the
relation itself is cached.

**The cost is real and worth knowing before you run it on production.** The
`pg_buffercache` view walks the entire buffer pool; on a server with
`shared_buffers` of several gigabytes that is millions of buffers examined
under partition locks, and it shows up as a brief latency bump for everyone
else. Run it when you need it, not on a schedule, and prefer a quiet moment.
On PostgreSQL 16 and later the cheaper `pg_buffercache_summary()` and
`pg_buffercache_usage_counts()` functions give the top-line picture without
the per-buffer walk - use those if all you want is "how full is the cache".

How to read the two percentages:

- **`pct_of_relation`** - how much of this relation is in cache. A small,
  hot lookup table at 100% is exactly right. A 200 GB table at 3% is also
  right - you would not want it evicting everything else. This number is only
  a problem when a table you *expect* to be served from memory is sitting low.
- **`pct_of_buffers`** - this relation's share of the whole pool. It is an
  alarm only when one or two relations dominate it and the tables you care
  about are starved. A healthy pool is usually a long tail.

If the top of the list is TOAST tables and indexes you do not recognise,
that is normal - index pages and TOAST chunks are cached like anything else.
The check is for answering "why is my working set not staying in memory",
and the answer is usually either `shared_buffers` is too small for the working
set or one batch job is repeatedly pulling a large cold table through it.

#### TimescaleDB {#timescaledb}

`db-timescaledb` reads the TimescaleDB extension of the database you are
connected to. It needs TimescaleDB 2 for everything below; with 1.x installed
it says so and the schema browser still shows the hypertables and their
chunks.

**Edition.** The first line names the version and the license setting
(`timescaledb.license`). Under `apache` - the Apache-only build - compression,
continuous aggregates, the retention, compression, refresh and reorder
policies and user-defined jobs are refused by the server itself; hypertables,
chunks, `drop_chunks` and `time_bucket` work. The panel greys those actions
and says why. A newer version installed on the server than in the database is
named with the `ALTER EXTENSION timescaledb UPDATE` that applies it.

**In the schema browser** hypertables have a category of their own, each
saying how it is chunked (time column and chunk interval, and the space
partitions if any), how many chunks it has and how many are compressed; its
row count is the sum of its chunks, because the hypertable's own table is
empty. Opening one shows its size, the compression ratio and its chunks,
newest first (the newest 500; the rest are counted). A chunk opens as the
table it is, and its menu compresses or decompresses it. TimescaleDB's
schemas (`_timescaledb_internal`, `_timescaledb_catalog`, `timescaledb_information`
and the others) and the relations it runs - chunks, compressed data, the
tables and views behind continuous aggregates - are hidden like other
objects of extensions, behind the same "show" switch.

**What the check judges**, with the numbers it judges by:

- **A job that keeps failing** - two failed runs in a row since its last
  success (counted from `timescaledb_information.job_errors`; before 2.9,
  which has no such view, only the last run is known). An alarm.
- **A job that did not run on schedule** - its next start more than 5 minutes
  in the past. The scheduler starts a job within seconds when a background
  worker is free, so minutes late means none is. An alarm.
- **The scheduler not running** for this database while jobs are scheduled -
  usually `timescaledb.max_background_workers` too low (it needs one worker
  per database with TimescaleDB plus one per job running at once) or
  `max_worker_processes` exhausted. An alarm.
- TimescaleDB's own housekeeping jobs (telemetry, job-history retention) are
  reported when they fail, never as an alarm: telemetry fails on any server
  without internet access.
- **Compression behind its policy** - chunks older than the policy's
  `compress_after` that are still uncompressed, each with a Compress button.
- **Retention behind its policy** - chunks older than `drop_after` still there.
- **A continuous aggregate falling behind** - the time from its watermark
  (what is materialized) to now, against what its refresh policy should keep
  it within: `end_offset` plus the schedule. It is flagged past twice that
  and past 15 minutes, so a policy refreshing every minute is not flagged
  for a moment's delay. Without a policy it is said either way, as a warning
  when the aggregate is `materialized_only` (queries then miss everything
  after the watermark).
- **Chunk interval against memory** - Timescale's own rule: the chunks being
  written to, indexes included, of all hypertables together should fit in
  about 25% of main memory, which is the share `timescaledb-tune` gives
  `shared_buffers`. The check measures the newest interval that has ended for
  every hypertable written to in the last 7 days and compares their sum with
  `shared_buffers`; past it, each hypertable gets an interval scaled down to
  fit. The opposite is flagged too: more than 1000 chunks whose interval holds
  under 1% of `shared_buffers` or under 10 MB - every chunk is a table the
  planner considers. A new interval (`set_chunk_time_interval`) applies to
  chunks made from then on.

A warning or an alarm in the list above the panel has the usual "Add to task
list" button, and the alarms light the TimescaleDB lamp in the header (the
same reading Health shows as TIMESCALEDB JOBS).

**Actions.** From the panel: add or remove a compression or retention policy,
compress one chunk, refresh a continuous aggregate for a window, run a job
now, change its schedule, pause and resume it; from a chunk's menu, compress
or decompress it. Each shows its statement first, with the one that undoes
it - removing a policy keeps its setting in the statement that puts it back -
and both go into the audit log. Only the owner of the hypertable, aggregate
or job, or a member of that role, may do these; the panel asks
`pg_has_role(owner, 'USAGE')` and greys what you may not do, and the server
asks again.

**The rest of the app on a TimescaleDB database:**

- Compare compares a hypertable as a table and also by how it is chunked and
  compressed, so a plain table and a hypertable, or two chunk intervals,
  differ. Chunks are not compared: they are named in creation order, and
  `_hyper_1_5_chunk` on one side is a different time range from the same
  name on the other. A different interval gets a `set_chunk_time_interval`;
  turning a table into a hypertable is left as a commented statement, because
  it moves every row.
- Restore wraps a dump with TimescaleDB the way TimescaleDB requires:
  `timescaledb_pre_restore()` before, `timescaledb_post_restore()` after -
  also when the restore fails, so the database is never left with its jobs
  stopped. The extension is created in an empty target first. The target has
  to run the TimescaleDB version the dump came from. Restoring only part of
  such a dump, or with `--clean`, is warned about: a hypertable without its
  catalogue rows and chunks comes back as an empty plain table.
- Changing a hypertable's owner moves its chunks with it (TimescaleDB does
  that). The copy of the definitions taken before a change holds the
  hypertable, not its chunks, and the statement that makes it a hypertable
  again.
- The relation map draws a hypertable as one card with its chunk count and
  links a continuous aggregate to the hypertable it aggregates.
- Other checks that list tables (sizes, bloat, vacuum) name the hypertable or
  continuous aggregate next to each chunk.

#### Vacuum, bloat and table statistics {#vacuum-bloat-and-table-statistics}

`db-vacuum` shows, per table, when it was last vacuumed and analyzed, live and dead tuple counts, the percent modified since the last analyze, and a heat label, with a recommended autovacuum reloption set next to the current one. The tags matter most: **WRAPAROUND risk** is act-now; **autovacuum disabled** on a busy table is usually a forgotten mistake; an **insert-only** table needs `autovacuum_vacuum_insert_threshold`, since the default vacuum threshold never fires for it.

`db-autovacuum` is the live view: which autovacuum workers are running right now and what each is doing. Empty is the normal state. A worker that has been on the same large table for a long time, or `max` workers all busy while the vacuum backlog in `db-vacuum` keeps growing, means autovacuum cannot keep up - raise `autovacuum_max_workers` or `autovacuum_vacuum_cost_limit`, or lower the per-table scale factor on the worst offenders.

`db-bloat-breakdown` separates heap, TOAST and index bloat per table, combines them into one `est_waste` for the whole table, and names the next step - VACUUM, REINDEX, `pg_repack`, or nothing. Its `recommendation` column is a prompt derived from the same totals it shows, not an instruction: *Bloat: the estimate and the real number* (above) covers what it does and does not account for, and how the cluster-wide index ranking (`db-bloat-indexes`) and the exact scan (`db-pgstattuple`) fit around it.

The **health overview**'s *WASTED SPACE (est.)* row uses this same combined heap-plus-TOAST-plus-index estimate, aggregated across the database, and flags a table on **whichever trips first**: an absolute **20 GB** of estimated waste in one table (red at **200 GB**), or **25% or more** of the table wasted once that is at least **1 GB**. The absolute number is there because 40% of a 200 MB table is housekeeping while 6% of a 20 TB table is more than a terabyte of disk and hours added to every restore. A table with one unreliable component (usually heap) is **not** dropped from this row - its other known components still count, as a `≥` lower bound, the same rule `db-bloat-breakdown` follows for the per-table figure; `db-bloat-breakdown` is where to see which component and why. The drill-down lists the worst tables and the per-table Advisor tab breaks each one down by component. `pg_repack` and `VACUUM FULL` return heap/TOAST space to the operating system; a plain `VACUUM` only frees it for reuse inside the same table; an index needs `REINDEX`.

The **Advisor tab** in the schema browser (select a table -> Advisor) is the per-table version of all of this in one place. It shows a **bloat breakdown** row per component - heap, TOAST, each index - each with **on disk** (exact), **useful (est.)** and **waste (est.)**. Hover any column header for what it means. "Useful" for an index is what a fresh `REINDEX` would produce, from `pg_stats` column widths - an estimate, and btree deduplication can make the real minimum smaller still. "Waste" is anchored to the real on-disk size (same arithmetic as `db-bloat-breakdown`), so it stays right even when `pg_class.relpages` is stale, and a component's own row shows `?` instead of a number when its inputs do not fit. The **total** row never does that: it is marked `≥` and names the missing component (with a one-line "why", and `ANALYZE`/Accurate-scan actions where they apply) whenever one component could not be estimated, rather than showing a fabricated zero or a blank figure. The **Analysis & suggestions** block below turns the numbers into conditional advice with ready-to-copy SQL: reusable waste with autovacuum keeping up is left alone; an inflated file gets `pg_repack` or `VACUUM FULL`; **each bloated index gets its own `REINDEX INDEX CONCURRENTLY` line** (plain `REINDEX` on PostgreSQL < 12, flagged), because `VACUUM` never touches an index; TOAST waste is called out for the same reason. Thresholds fire on a share **or** an absolute size, whichever trips first.

The Advisor tab is five sections, in the same cards a check result uses, with a strip under the title that jumps to any of them: **Table** (its numbers), **Vacuum & autovacuum**, **Indexes**, **Maintenance** and **Deep analysis**. The vacuum section starts with how vacuum has been treating this table - how many runs since the statistics were reset and how many per day, the gap between them, on PostgreSQL 18 how long they take per day, the dead-row share, and the oldest snapshot anything in the database holds - and says, in the same words as `db-vacuum`, when a table is vacuumed hourly or more often, when its vacuums achieve nothing because a transaction, a prepared transaction, a replication slot or a standby holds a snapshot older than its dead rows (and what to end), when every vacuum of it is an aggressive one, and when autovacuum runs with no brake. A recommendation never tightens the thresholds of a table that is already vacuumed hourly. Then come the trigger lines - when autovacuum fires today and after the recommendation - the four parameters, and the buttons that apply them. When a lock keeps vacuum away from the table, that finding comes first and names the session that holds it.

**When vacuuming often is fine.** A table vacuumed hourly or more often - at least 24 runs a day *and* at least 50 runs in the statistics window; every quarter hour is "very frequent" - is not called a problem by its count alone. Four more things are weighed, and the finding says which of five verdicts applies:

- **The trigger.** Autovacuum fires when the dead rows pass `threshold + scale_factor × rows` (the table's own setting first, then the server's; from PostgreSQL 18 capped by `autovacuum_vacuum_max_threshold`). The finding says which part decides it - on a small table the fixed threshold (50 + 0.2 × 120 rows is mostly the 50), on a large one the scale factor - and what share of the table the trigger is.
- **The work each run finds.** Rows updated or deleted per run (from PostgreSQL 13 also rows inserted, against the insert trigger). A run *cleans* when that reaches **half the trigger or 10,000 rows**; it is *idle* when it is under **10% of the trigger and under 1,000 rows**. A scheduled manual `VACUUM` is the usual idle case: it runs whether or not anything changed.
- **What one run costs.** On PostgreSQL 18 it is measured per table. Before 18 it is estimated from the brake: the pages a run reads (those the visibility map does not mark all-visible, those the changes touched, and every index when the run vacuums indexes - from PostgreSQL 14 skipped when fewer than 2% of the pages hold dead index entries), each read counted as a disk read at `vacuum_cost_page_miss`, each dirtied page at `vacuum_cost_page_dirty`, at the pace of `cost_limit` units per `cost_delay`. The cost limit is shared between the autovacuum workers running at once, unless the table has its own cost settings, so the estimate divides it by the workers running now. A manual `VACUUM` runs with `vacuum_cost_delay` 0 by default - no brake - and its time cannot be estimated that way. On PostgreSQL 18 the estimate stays as the explanation: the runs take about what the brake allows (brake-bound: a higher cost limit would shorten them), less (the pages came from cache) or more (they wait on disk, locks or CPU). The finding also gives the read and write ceiling the brake implies. *Cheap* is **at most 5 seconds per run and at most 15 minutes a day**.
- **The WAL it writes.** Always an estimate - no PostgreSQL version counts WAL per table: about 100 bytes per dirtied page, plus a full-page image for a page not logged since the last checkpoint. A page the workload changed after the last checkpoint is not imaged again, so a table vacuumed *more* often than checkpoints happen pays far fewer images than its run count suggests; the finding compares the gap between runs with the real checkpoint interval and says which case applies. From PostgreSQL 14 the estimate is capped at the cluster's measured WAL, and set against it. Large is **1 GB a day and 10% of the cluster's WAL** (a warning from **10 GB and 30%**). `wal_compression` shrinks the images; the finding mentions it without modelling it.

The verdicts: **ok** - cheap, cleans, dead rows at most 1.5 × the trigger (or under 1,000), WAL not large: shown as OK, the runs per day in green, nothing to change. **idle** - vacuumed more often than the table changes (a note when the runs are cheap; a warning when they are very frequent and not cheap, or of unknown cost): schedule the manual `VACUUM` less often; for autovacuum's own runs, look at the insert trigger and the transaction-ID age. **costly** - expensive runs on a table of at least 100,000 rows whose trigger is under 1% of it, with the write rate and the threshold and scale factor that would vacuum it about hourly - those become the recommended values that **Apply** and the ALTER statement carry; or runs fine in every way but their WAL. **busy** - the frequency comes from the write rate, not the settings: vacuuming less would let bloat grow, and each run gets cheaper with more HOT updates (fillfactor), fewer indexes or a gentler brake. **unknown** - the cost cannot be told. A recommendation never tightens the thresholds of a frequently vacuumed table, whatever the verdict; a costly one is loosened to the hourly trigger instead, and the reason stands beside the recommended values.

**How "per day" is counted.** Every per-day figure is the runs since the start of the table's statistics window divided by its length - an average over that window - and the card and the finding say where it starts, for example *since 2026-10-03 14:20 CET (25.0 days, this database's statistics reset)*. The start is, in this order: on PostgreSQL 19 the table's own reset time; otherwise this database's last statistics reset; for a database never reset (PostgreSQL 15 and newer leave that time empty) the earliest reset of the server's own statistics; failing all of them, the server start. PostgreSQL moves the database's reset time when the counters of any *one* table are reset (`pg_stat_reset_single_table_counters`), so the other tables' counters can be older than it. A reset also clears a table's last vacuum and analyze times, so a table with a time older than the database's reset was not reset with it: it gets **no rate** instead of a figure inflated by the short window, and the *Autovacuum cost and I/O* section counts such tables in one line. A window under an hour gives a projection, shown uncoloured. A sixth verdict, **past**, is a table frequent on average whose last vacuum was at least ten of its average gaps and three hours ago: the average comes from earlier activity, so nothing is flagged and no recommendation is held back for it; AWR snapshots measure the current rate per window. On PostgreSQL 15 and newer a database created after the server's statistics began is divided by the longer window, so its rate can read low, never high.

Autovacuum visits each database once per `autovacuum_naptime`, so it can vacuum one table at most that often; a table vacuumed on most of those visits is due nearly every time, and the finding says so. When the autovacuum workers are saturated - on PostgreSQL 18 busy **at least 50% of their capacity and 6 worker-hours a day** (a warning from **80% and 12 hours**), measured from the time this database's tables spent in autovacuum and autoanalyze, a lower bound because other databases add to it; on 19 also every worker busy while tables are due - due tables wait their turn, and the advice is more workers or a looser brake, not other thresholds.

Every finding ends with how solid its numbers are: PostgreSQL 18 and newer measure each table's run time; 16 and 17 measure only the cluster's I/O; 14 and 15 estimate everything per table; on 13 the finding is the frequency alone.

Beside the tabs, **Reload** re-fetches the current tab for the selected object without leaving it, and **Export CSV** saves the table on the tab exactly as you have it filtered and sorted.

Right-click a table and choose **Open Advisor** (or click a table name in a check result) and the Advisor opens as its own tab in the strip, labelled `Advisor: schema.table`. Do it for a second table and the two sit side by side - switch between them to compare, without either replacing the other. The bloat-breakdown note, when the heap estimate is `?` for want of column statistics, carries a **Run ANALYZE** button; the Advisor refreshes once it lands.

The Advisor tab's **Maintenance actions** section also shows any VACUUM / ANALYZE / REINDEX / VACUUM FULL running on that table **right now**, read from the server by relation id - so it appears whether the app started it, another tool did, or the window was only just opened. Each shows live progress from `pg_stat_progress_*` and a **Stop** that cancels it on the server; stopping an operation someone else started asks for confirmation first. On PostgreSQL 12+ the safer `REINDEX CONCURRENTLY` (which does not take an `ACCESS EXCLUSIVE` lock) leads; below 12 it is shown disabled with the reason.

**From the stored history (AWR and ASH).** Everything above reads the counters since the last statistics reset: one average over however long that is. When AWR snapshots of the connected database are stored, `db-vacuum` and the Advisor tab read them too: per table, the rows updated, deleted and inserted in each window between two snapshots, how often vacuum ran, and how far the table was from its trigger at every snapshot, with the settings in force then (snapshots taken by an earlier version lack these figures). From the average rate they size a trigger of about **one run an hour**, never looser than 20% of the table. Today's trigger is **tightened** when it is at least **twice** that size, lets at least **10,000** dead rows wait, and the table changes at least 500 rows an hour; it is **loosened** when it is at most half that size on a table vacuumed hourly or more often (at least 10 runs in the history); anything within a factor of two **fits**. From PostgreSQL 13 an insert-mostly table (90% of its writes are inserts) gets the insert pair the same way. These values replace the size heuristic in the recommendation that **Apply** and the ALTER statement carry, and the reason beside them names the rates and the busiest window. A table that stayed past its trigger for at least **an hour over two or more snapshots** is called out (a warning from **six hours**), and then no new thresholds are offered: the trigger is not what decides when it is vacuumed. When ASH samples are stored they say why: an autovacuum worker on the table **waiting for a lock** (at least 10% of its samples and at least 5, or 60 samples on their own), with the session that held it; asleep in its **cost brake** (`VacuumDelay`: at least half its samples and at least 30, or 600 on their own); or **every autovacuum worker busy at once** (at least half the samples with a worker at work and at least 60, or 1,800 on their own). Under an hour of history nothing is sized. An **Evidence** block lists the snapshots and windows used, from when to when, and the ASH days, or says that there is no history and that the advice is what it always was.

`db-table-stats` is sequential vs index scan counts per table, plus `last_seq_scan` / `last_idx_scan` on PostgreSQL 16+. A table with millions of seq scans and a climbing count is being scanned in production now - cross-check it against `db-index-advisor`.

#### Sequences, checkpoints and tablespaces {#sequences-checkpoints-and-tablespaces}

`db-sequences` and `db-seq-usage` list sequences by how close they are to their maximum, with the owning column and every default that references each. An `integer` sequence at 80% is a change to `bigint` you want to plan, not discover when `nextval` throws.

The maximum that counts is the smaller of two: the sequence's own `MAXVALUE`, and the largest value the column it feeds can hold (32,767 for `smallint`, 2,147,483,647 for `integer`). Before PostgreSQL 10 a sequence had no type - every one was `bigint` - so every `serial` made then is an `integer` column fed by a `bigint` sequence, and an upgraded schema keeps that shape. Measured against the sequence alone such a column reads 0% used while it has a few thousand ids left; the next insert past the edge fails with *integer out of range*. `db-sequences`, `db-seq-usage` and the health overview's SEQUENCES row measure against the column, and grade on two things, either one enough:

- **the share of the range used**: amber from 80%, red from 95%;
- **the time left**: the ids left divided by the inserts a day into the column's table (its insert counter over the statistics window - the same window the table Advisor states): amber under 180 days, red under 30. A window shorter than an hour makes the rate a projection: the time is shown and said to be one, and does not change the colour.

Each one running out names the foreign key columns that point at it and are narrower than `bigint`: moving the column to `bigint` moves the problem to them, so they change in the same migration. Once the column itself is `bigint`, a narrower foreign key column is the ceiling: the first id past its maximum cannot be referenced, so the room left is measured against it and it is named. `ALTER COLUMN ... TYPE bigint` rewrites the whole table under an `ACCESS EXCLUSIVE` lock - nothing reads or writes it until it is done; the way without the outage is a new `bigint` column kept in step by a trigger, a backfill in batches, and a swap.

`db-seq-sync` writes a `SELECT setval(...)` for every sequence, setting each to just past the largest value currently in its owning column. Run it after a logical-replication cutover or a `pg_dump --data-only` load, where the rows arrive but the sequences stay at 1 and the next `INSERT` collides with an existing key. It is a generated script like any other - review it, then run it on the target.

`db-checkpoints` shows how checkpoints fire: **timed** (good, driven by `checkpoint_timeout`) versus **requested** (forced by `max_wal_size` pressure), the average gap between them, and write and sync time. A high requested share means `max_wal_size` is too low for the write rate - raising it trades disk space for much less checkpoint I/O.

`db-tablespaces` lists every tablespace with location, owner, size, options (`seq_page_cost`, `random_page_cost`, `effective_io_concurrency` overrides) and the databases using it, plus the contents of each tablespace in the connected database. The per-database contents section is scoped to the current connection for the same reason schema sizes is - the catalogue that maps objects to tablespaces is per-database. Check it before moving a large object, to confirm the target has room and sits on the disk you expect; a `MOVE` writes a full second copy before dropping the first, so the target needs headroom equal to the object's size.

#### Partition runway {#partition-runway}

`db-partition-runway` (Storage) lists every partitioned table in the connected database: its strategy and key, how many partitions, where the last range ends and how long that is from now, the DEFAULT partition and the rows in it, and whether pg_partman manages the table. It is there for the incident that happens at midnight on the 1st: a table partitioned by month whose next partition nobody created. From the moment the last range ends, an INSERT for "now" either fails with *no partition of relation found for row* or, when there is a DEFAULT partition, lands in it - and a DEFAULT partition holding rows of a range blocks creating the partition for that range later (*updated partition constraint for default partition would be violated*) until the rows are moved out by hand.

- A table is **amber** when its last range ends within the number of days the check is given (7 by default, and Health uses 7), and **red** once it has ended. The runway is measured for a RANGE key whose first column is a `date`, `timestamp` or `timestamptz`, also when the key is an expression such as `(created_at::date)`; a numeric key (an id range) has no calendar to measure, and the row says so. A last range that goes up to `MAXVALUE` never runs out.
- **Rows in a DEFAULT partition** are amber for a RANGE table, whatever their number - one row is enough to block the partition that should hold it - and blue for a LIST table, where a catch-all is a common design. The row gives the count and the share of the table.
- **pg_partman**: when the extension is installed in the database, the check reads its configuration - the premake and interval of every table it manages - and what runs its maintenance (the `pg_partman_bgw` worker, a `pg_cron` job). A table pg_partman manages is amber when its runway has dropped below what its premake keeps ahead: a daily table with premake 4 always has three to four days, so the 7-day warning would never be quiet for it. The maintenance is then not running, or - with `infinite_time_partitions = off` - adds future partitions only once new rows have arrived.
- A table whose last range ended long ago and which no longer takes new rows (an archive) is red as well: the catalogue cannot tell an archive from a table whose inserts are failing, and the row says so.

Nothing scans a table: the bounds come from the catalogue and the row counts from the planner's statistics, so they are estimates.

#### Replication health {#replication-health}

`db-replication` is the primary's view of its replicas: each standby's state, and the lag in both bytes and time at the write, flush and replay stages. `replay` lag is what a read replica actually serves behind; a growing `write`/`flush` lag points at the network or the standby's disk, a growing `replay` lag at a long-running query on the standby holding replay off. On a standby with no downstream replicas the check simply says so.

`db-replica-identity` finds tables logical replication will break on: no primary key, or a non-default `REPLICA IDENTITY`. A table with no PK that is **in a publication** is CRITICAL - the next `UPDATE` or `DELETE` fails on the subscriber - while one that is not is only advisory. It writes the `ALTER TABLE` in priority order: primary key, else `REPLICA IDENTITY USING INDEX`, else `FULL`.

`db-repl-slots` lists physical and logical slots with their `active` flag and the WAL each is holding back. **An inactive slot fills the disk** - it retains WAL for ever while nothing consumes it. Drop slots left by a decommissioned replica.

Creating a **logical** slot waits until every transaction that holds a transaction id when it starts has ended - in any database of the cluster, a forgotten prepared transaction included. The Create slot dialog lists those transactions before you create it, and the creation waits at most the time you choose (30 seconds unless you pick another); when that runs out the slot is not created and the answer names what it was still waiting for. A read-only transaction does not hold it up. Without `pg_read_all_stats` other users' sessions are only partly visible, and the dialog says the list may be incomplete.

The header's **REPLICA** chip on a standby reads *caught up* when everything it has received has been replayed, however long ago the primary last committed - its tooltip gives that time as how long the primary has been quiet. A lag in seconds is shown only while there is WAL still to replay; a standby with no WAL receiver (archive recovery) shows no lag figure at all, because it cannot know whether more is coming.

`db-logical-replication` is the subscriber side: `pg_stat_subscription` lag and per-table sync state from `pg_subscription_rel`. A table stuck in `COPY` or `SYNC` usually means the initial copy hit an error - check the subscriber log.

#### The errors watcher {#the-errors-watcher}

`db-errors` samples `pg_stat_database` over a five-second window and reports the per-second rate of rollbacks, deadlocks and conflicts, plus `sessions_fatal` / `killed` / `abandoned` on PostgreSQL 14+. On a self-hosted cluster where `pg_read_file` is allowed it also reads the last 256 KB of the server log - the `jsonlog` file (PostgreSQL 15+) or the `csvlog` file when the server writes one, else the plain log - and reads the `ERROR` / `FATAL` / `PANIC` entries as entries, each with its `DETAIL`, `HINT`, `CONTEXT` and `STATEMENT` lines. On a managed cloud it skips the log and points you at where the provider keeps it.

From the log it reports:

- **Errors by SQLSTATE:** each class with its count, the busiest minute, first and last seen, how many sources and the main one, and what it means for the classes that are about concurrency and retries - `40P01` deadlock, `40001` serialization failure, `23505` unique violation, `55P03` lock not available, `57014` statement cancelled, `42P10` `ON CONFLICT` without a matching unique index, `53300` too many connections, `25P02` statements sent into an aborted transaction.
- **Errors per minute**, per SQLSTATE.
- **Retry storms:** the same SQLSTATE from the same source **10 or more times within one minute** - something is retrying in a loop. An absolute count, not a share: a retry loop is a problem on a quiet server and a busy one alike. The source is the application when the log names it (`jsonlog`, `csvlog`, or `%a` in `log_line_prefix`), else the statement's shape.
- **Deadlocks, both sides:** for every deadlock, each process with its pid, what it waited for, who held it, the statement it was running, and which one PostgreSQL cancelled.

The plain log carries what `log_line_prefix` puts there; the check reads the prefix from the server and finds `%e` (the SQLSTATE), `%a`, `%u` and `%d` wherever they sit. Without `%e` the SQLSTATE is read from the English message text and the check says so: add `%e` (and `%a`) to `log_line_prefix`, or use `jsonlog` or `csvlog`.

Read the rates first: a steady deadlock rate is an application lock-ordering bug; a spike of conflicts on a replica is queries colliding with replay. The log is the detail, the counters are the alarm.

#### The setup and key-settings advisors {#the-setup-and-key-settings-advisors}

`db-setup-advisor` checks every server-side thing DBApguin's checks depend on - the `pg_stat_statements`, `auto_explain`, `pg_buffercache` and `pg_repack` extensions, GUCs like `track_io_timing`, `compute_query_id`, `log_lock_waits`, and the `pg_monitor` / `pg_read_server_files` grants a non-superuser needs - and reports each OK / PARTIAL / MISSING with the exact fix. **Run it first against a new cluster** and again after any config change. On a managed cloud the hint says "set it in the parameter group" instead of `ALTER SYSTEM`.

`db-settings` is a curated guide to ~70 tuning-relevant parameters with a rule of thumb and a warn/danger badge each. It is a reference, not an alarm: a badge means "this value is unusual", not "this is broken". These parameters are editable from the Server view. Five cards are about whether committed data survives a crash and give the same verdict as the DURABILITY row of the health overview: `fsync` and `full_page_writes` off are danger, `synchronous_commit = off` is info (a deliberate trade, not a fault), `data_checksums` (read-only) off is info, and `synchronous_standby_names` shows how many of the synchronous standbys it asks for are connected - danger while commits wait for one that is not.

**Computed for this server** opens Key settings: the fifteen parameters a tuner such as DBtune changes without a restart (`work_mem`, `maintenance_work_mem`, `effective_cache_size`, `random_page_cost`, `seq_page_cost`, `effective_io_concurrency`, `default_statistics_target`, the three parallel-worker limits, `max_wal_size`, `min_wal_size`, `checkpoint_completion_target`, `bgwriter_delay`, `bgwriter_lru_maxpages`), each with a value worked out for this machine, the measurement or formula it rests on, reload or restart, and **Apply**. RAM and CPUs come from `/proc` and the cgroup limits when the server may read files (a superuser or `pg_read_server_files`); a container shows the host's, so enter what the server may use and press **Recompute**. The disk is assumed SSD unless you choose HDD. A measurement decides the level wherever there is one: `work_mem` is only a warning when sorts really spill (at least 100 temp files AND 1 GB since the statistics reset), `max_wal_size` when at least a fifth of the checkpoints AND five of them were forced by WAL volume (or from the WAL rate: three `checkpoint_timeout`s of WAL), the background writer when backends wrote at least a quarter of the dirty buffers AND 10,000 of them themselves. **Apply the N to look at** runs exactly the listed statements after one confirmation, then reads the server again. These are starting points for the machine, not a measurement of your workload: change a few, watch the same figures, keep what helped. The setup advisor window (Help) shows the count with a button to this section.

**With history it reads what the server did, not only what it has.** When ASH or scheduled AWR snapshots have run for this server (from any of its databases), the section opens with **What it is based on**: ASH's days and samples, how many sessions were really active at once (p95 and peak), the share of active time spent waiting on I/O and the busiest hour; AWR's windows, the temp spilled per day, the checkpoints forced, the WAL written in the busiest window against the average, the average read time and the worst cache hit; the statements that spill and how much one call writes; and what runs now. The advice uses them where they are stronger than a rule: the `work_mem` ceiling counts the sessions ASH saw at once (twice the p95 or the peak) instead of every connection - but only lifts `work_mem` as far as spilling statements need, twice what one call wrote to disk; a statement that needs more than the RAM allows is listed with a `SET work_mem` of its own. `max_wal_size` follows the busiest AWR window, the disk class follows the measured read time (`pg_stat_io` with `track_io_timing`), `effective_io_concurrency` and `shared_buffers` (a restart) become warnings when ASH shows a fifth of the active time AND 600 samples waiting on I/O, or AWR a cache hit under 99 % on at least a million reads. Without history it says so and offers **Start ASH** and **Start AWR snapshots** (every 15 minutes); the more of it there is, the closer the advice is to this workload. AWR snapshots now also record `pg_stat_wal` (PostgreSQL 14+) and `pg_stat_io` (16+); older snapshots simply lack them.

#### Ready for PostgreSQL 19? {#ready-for-postgresql-19}

**Tuning & diagnostics -> Ready for PostgreSQL 19?** asks this server about
every item in PostgreSQL 19's list of incompatibilities and answers each one:

- **BLOCKS** - pg_upgrade refuses, or 19 will not start: a database in the
  removed MULE_INTERNAL encoding, a line break in a database, role or
  tablespace name, an index using btree_gist's inet/cidr operator classes,
  RADIUS in pg_hba.conf, escape_string_warning set anywhere.
- **CHANGES** - the upgrade goes through and something behaves differently:
  standard_conforming_strings off, MD5 passwords (every login warns on 19),
  max_locks_per_transaction set by hand (19 needs double for the same
  capacity), JIT in use (off by default on 19), postgres_fdw (read-only
  transactions now pass to the remote side).
- **INFO** - third-party extensions that need a 19 build before pg_upgrade,
  and monitoring tools that read renamed columns.

Several items need superuser; without it they say "could not look" rather
than reporting all clear. The index item reads the database you are
connected to, so run the check in each database. On a server that already
runs 19 the same rows show what still differs from 19's defaults.

#### Oracle-style observability stack

DBApguin ships a set of checks that mirror Oracle's AWR / ASH / SQL Trace tooling for PostgreSQL:

**ASH and AWR each have one tab.** The *Tuning & diagnostics* menu opens *Active session history (ASH)* and *AWR snapshots and reports*, one tab per server. The tab's header shows what the collector is doing (sampling or taking snapshots, since when, how often, what is kept; amber and *paused* while the server is unreachable) with **Start**, **Stop** and the settings behind the gear; the views are the tab's sub-navigation (ASH: Live, Report, Stored days; AWR: Live, Snapshots, Report, Compare, Capture, Trend), and a view's inputs, the report window and database or the two snapshots to compare (four for two periods), sit above the result instead of in a dialog. A date is chosen with the date-and-time picker, whose calendar underlines the days that hold samples. The `action` parameters described below are what the views run; they keep working from scripts and older links and open the same tab. When you connect to a server whose sampler or scheduler is running, its tab opens by itself.

**db-workload-report (AWR-style live capture)** - AWR "For a while", up to an hour, with *also sample the sessions every 5 s* runs it (its result still shows in the AWR tab under *Capture*): a window (10 s to 60 min, default 5 min) and a sample interval. It takes the same snapshot an AWR snapshot takes at the start, samples pg_stat_activity every few seconds in between, takes the second snapshot at the end, and reads the window exactly as an AWR report reads two snapshots: **Window summary** (with the samples, the peak of active and of all sessions, and how many users and applications were seen), **Findings** with their evidence, **Top SQL to look at** with why each one, then **Active sessions over time** (a chart of the samples by wait class), **Wait events**, **Blocking pairs observed**, **Longest open transaction seen**, the Top SQL rankings, the tables by DML, the background and checkpoint activity and the settings for a better analysis. See [Reading an AWR / workload report](#reading-an-awr-report). Use it when the workload is running right now and you want one report of it; the report is kept in [Saved reports](#saved-reports).

**db-awr-snapshot (persistent snapshot store + arbitrary compare + scheduler)**: six actions controlled by the `action` parameter. Three manual, three for the auto-scheduler.

  Manual actions:

  - action=take: capture pg_stat_statements + pg_stat_database + pg_stat_user_tables + pg_stat_user_indexes + the autovacuum settings + bgwriter / checkpointer to a JSON file at `$DBCHECKS_HOME/awr_snapshots/<server-fingerprint>/snap-<id>.json`. Optional comment string is saved with the snapshot. Most of a snapshot is the copy of pg_stat_statements; the per-table and per-index rows the autovacuum and index advice read back add about 320 bytes a table and 160 bytes an index, so a database with a thousand tables adds about half a megabyte to each snapshot. Server fingerprint is `host__port__dbname` so snapshots from different clusters never mix. The column set follows the server version - `pg_stat_database.sessions_*` on PostgreSQL 14+, `pg_stat_checkpointer` on 17+, `pg_stat_bgwriter` before that - so a snapshot is valid from PostgreSQL 13 up, and a compare between two different major versions simply omits the columns one of them does not have.
  - action=list (**Stored snapshots**): every snapshot kept for this server - day, time (UTC), comment, statements, size - with **Compare with the one before** on each row, **Compare the two chosen**, **Export** (the chosen ones, or all, to one zip where the Save dialog says), **Import...**, **Delete** the chosen ones and **Delete all older than N days**. Deleting asks first. It works on the files alone, so the history of a server that is switched off can still be looked at, cleaned up or taken elsewhere.

    **Export and import.** An export is one zip: the files as they are and a small manifest naming ASH or AWR and the server. Import puts them back under that server - connect to it to see their reports - takes only the store's own file names (a zip with anything else in it is refused whole, nothing written) and never overwrites a file that is already there: it says which it kept.
  - action=compare: diff any two snapshots by id and render the same AWR-style sections. The "window" can span hours or days because the data is on disk. Take one snapshot before a deploy / batch / migration, another after, then compare whenever you want.

    **Across a restart.** A clean restart keeps PostgreSQL's counters - it writes them out at shutdown and reads them back - so a comparison across one covers the whole window and only says that the server restarted. A crash, a recovery at start-up or a failover starts them from zero, and so does pg_stat_reset(). Each snapshot records when the server started; where a counter is lower in the newer snapshot, the comparison counts it from zero (the newer value is what happened since the reset) instead of printing a negative number, opens with a finding that says so, and after a crash takes the window from the server's start. The trend does the same for each window. Oracle's AWR has the same limit - its counters start again with the instance and it refuses a report across a restart; what both keep is the history of snapshots.

  - action=compare-periods (the **Compare** view): two periods side by side, the way Oracle's *Compare Periods* report sets them - period 1 from snapshot A1 to B1, period 2 from A2 to B2; the older snapshot of each pair is taken as its start. Each period is computed exactly as compare computes its window, across a restart or a reset too, with the same **Database** choice. Opened with nothing chosen, the view takes the newest three snapshots as two consecutive windows - the third newest to the second newest, then the second newest to the newest - and compares them; with fewer than three it says that a comparison needs three and runs nothing. The periods may overlap or come in either order; the report says so. See [Reading a period comparison](#reading-a-period-comparison).


  Auto-scheduler actions (Oracle's `DBMS_SCHEDULER`-driven auto-AWR analogue):

  - action=auto-start: spawn a background thread that takes a snapshot every `interval_minutes` (default 60, range 5-1440) and auto-deletes snapshots older than `retention_days` (default 30, range 1-365). The thread runs on a dedicated psycopg2 connection so it never starves the user-facing pool. Idempotent (re-running updates the params on the live thread). The very first snapshot fires immediately: `auto-start` waits for its result and reports what actually happened, so a scheduler that cannot take one snapshot says so instead of "started". If that first snapshot fails the thread does not stay up. Once it is running, a server that stops answering does not end it: after three failed snapshots in a row the scheduler pauses, keeps trying to reconnect (every 30 seconds, then less often) and carries on by itself as soon as the server is back; the live panel shows the pause in amber.
  - action=auto-stop: stop the scheduler thread. `auto-stop` reports what it captured while it ran; if it never took a snapshot it says that plainly rather than "files remain on disk". The persisted `enabled` flag in `auto_config.json` flips to false.
  - action=auto-status: leads with the verdict - healthy, paused, failing, aborted, or not running - then the live thread state (interval, retention, count of snapshots taken, last snapshot id and time, total files pruned, last error if any) and the persisted config. After an app restart the scheduler comes back on its own the next time you open that server; auto-start is only needed if you stopped it.

  A running auto-scheduler also shows up in the **Background jobs** list, the same as the ASH sampler, so you can see at a glance whether it is healthy without opening the check.

  **Scheduler, live** (in `auto-status` and after `auto-start`) is the same panel ASH has, refreshed every 5 seconds while the tab is open: the state, the interval and retention, how long it has run, snapshots taken, how long ago the last one was (yellow once it is half an interval late), when the next one is due, what is on disk and from when to when, and what retention has removed. The dialog shows only the fields the chosen action uses.

  Persistence note: the scheduler config is saved next to the snapshot files and survives app restarts. A scheduler that was running when the app closed is not started at boot (nothing is decrypted then); it comes back the moment you open that server in the app again, with the connection you open it with and the saved interval and retention. Until then `auto-status` shows it as enabled but not running. `auto-stop` is what switches it off for good.

**db-ash (Active Session History sampler)**: continuous background sampler. Four actions.

  - action=status: leads with the verdict - running and healthy, paused (the server stopped answering; it keeps reconnecting and resumes by itself), failing, or stopped - then when it started, how many samples + rows so far, the last error if any, and the on-disk inventory.
  - action=start: spawn a daemon thread that polls pg_stat_activity every **interval** seconds (1 by default; 2, 5, 10, 30 or 60) on a dedicated connection (separate from the user-facing pool), filters out idle backends, and appends each non-idle row to a daily JSONL file at `$DBCHECKS_HOME/awr_snapshots/<fingerprint>/ash/ash-YYYY-MM-DD.jsonl`. Idempotent (re-running is a no-op). The per-sample column set follows the server version - `query_id` is PostgreSQL 14+, so on 13 it is simply not recorded and the report notes that once rather than failing.
  - **interval** and **retention days** are chosen when starting and kept per server for the next start. Retention is 30 days unless you choose otherwise, and 0 keeps everything; day files older than the retention are removed when the sampler starts and once an hour while it runs - whole days only, never part of one. A longer interval writes less and misses short waits; one second is what a classic ASH does. The sampler keeps the settings it started with: starting again with other values says so and changes nothing until it is stopped.
  - action=stop: stop the sampler and report what it captured. If nothing was ever written it says so, instead of "JSONL files remain on disk". After three failed samples in a row the sampler pauses rather than stopping: it keeps trying to reconnect (every 30 seconds, then once a minute) and resumes by itself when the server answers again; `status` and the live panel show the pause in amber with the last error. A sampler left running also comes back on its own after an app restart, the next time you open that server; `stop` is what switches it off.
  - action=report: aggregate any date range on the server's own clock into four classic ASH sections: state histogram, wait events top-20, top sessions by sample count, top queries by sample count, blocking pairs observed. **Report from** and **Report to** are chosen with the app's date-and-time picker: quick choices (now, today 00:00, 1 h ago, 24 h ago, yesterday), a calendar and hour and minute fields; Clear leaves a bound open.
  - action=data (**Stored days**): every day file kept for this server with its size, today marked; **Report** on a row opens that day's report, **Report for the chosen days** spans the first to the last chosen. Export, import and delete as for AWR's stored snapshots below.

  A running sampler shows up in the **Background jobs** list with its health, so a sampler that has quietly stopped is visible without opening the check.

  **Sampler, live** (in `status` and after `start`) refreshes every 5 seconds while the tab is open: the state, the interval and retention, how long it has run, samples and rows written, how many sessions were active in the last sample, how long ago that sample was (yellow once it is three intervals late), what is on disk and how many days it spans, and what retention has removed.

  Storage: one row per active session per sample, about 450 bytes with its query text. Measured at one sample a second with two sessions active on average: 2.8 MiB an hour, about 67 MiB a day; at one sample every 5 seconds a fifth of that. The disk grows with how many sessions are active and how long their statements are, not with time alone, which is why retention is 30 days unless you choose otherwise. On the server one sample is one read of pg_stat_activity, about 0.3 ms. JSONL keeps the format trivial to inspect with `jq`.

**db-trace-session (Oracle-style SQL trace)**: per-backend sampler with three target modes and two filters. *Tuning & diagnostics* > *Session trace* opens its tab, one per server, with three views. **Live** is what the running trace is doing now: whom it follows, how far into its window it is, the samples and statements caught so far, each followed session with its state, its wait and its statement, and the waits seen most; it refreshes every two seconds, the tab opens on it while a trace runs for this server, and a trace started from the form runs on it and lands on Report when it ends. **New** is the form, above the result instead of in a dialog: whom to follow (every session of a user, every session whose application_name matches a pattern, or one backend chosen from the live list), the window (30 s, 1, 2, 5, 10 or 30 minutes, or any number of seconds from 10 to 1800; 2 minutes by default), the statement category and the query text filter, then **Start trace**. Before it starts, the trace asks, as ASH and AWR do, about the extensions and settings it reads when one it needs is missing, unless that question was switched off; the New view says in one line how many of them are in place, and **Check settings...** shows the same window at any time, its Start starting the trace. While it runs, the header says whom it traces and how much of the window has passed, and **Stop** ends it earlier: the report covers what was caught until then. The tab then shows **Report**, with the export row (text, CSV, JSON, PDF). The report ends with **Settings for a better analysis**, the table the ASH report ends with (see [Reading an ASH report](#reading-an-ash-report)), led by what decides how much a trace sees: `auto_explain` and its `log_min_duration` and `log_analyze` (the plan that actually ran goes to the server log), `log_min_duration_statement`, `log_lock_waits`, `compute_query_id` and `track_activity_query_size`. **History** lists the traces kept for this server, newest first: when each started, whom it followed, the window and how long it really ran (stopped ones are marked), how many statements it caught and its size. **Open** shows one as its report, read-only and exportable in the same four formats; **Delete** removes it after a question. Traces are kept per server until they are deleted (the newest 500), however many other checks run meanwhile. **Export all**, or the chosen ones, writes them to one zip; **Import...** reads such a zip and never overwrites a trace that is already here; the app data export carries them with the server's AWR snapshots and ASH days. Stop ends the AWR tab's *Capture* early in the same way.

  - target_mode=pid: trace a single backend by pid (chosen from the list of the backends connected right now).
  - target_mode=application_name: trace every backend whose application_name matches a POSIX regex. Useful when the app pool churns pids but app_name stays fixed.
  - target_mode=user: trace every backend of a specific role name. Use to investigate "what is this report user doing right now".
  - duration_seconds: 10 to 1800 seconds (default 120). Sampler runs at 2 Hz for the whole window.
  - category_filter: all / dml / ddl / select / utility. Drops rows whose query text does not match the chosen statement category.
  - query_filter: optional case-insensitive regex applied to query text. Empty = no filter.

  The trace also probes whether `auto_explain` is loaded and reports the exact SET commands to enable plan capture for the target session before the workload runs. Output: query sequence (transitions ordered by time), wait-event histogram for the targeted backends, per-query delta from pg_stat_statements. On PostgreSQL 13 the `query_id` column does not exist yet; the trace runs without it and says so in a note.

### 5.2 SQL {#the-sql-editor}

SQL editor with PostgreSQL syntax highlighting and custom completion. Toolbar:

- Run (`Ctrl/Cmd + Enter`, `F5`): execute everything in the editor
- Statement (`Ctrl/Cmd + Shift + Enter`): execute only the selection, or the statement under the cursor
- Validate (`Ctrl/Cmd + E`): parse + plan without executing
- Explain: EXPLAIN / EXPLAIN ANALYZE / BUFFERS
- Export CSV: re-runs the query on the server so the file holds every row,
  not only the page on screen
- Save as…: writes the result you are looking at as JSON, JSON Lines,
  SQL `INSERT`, a Markdown table, or aligned text
- Clear history

Below the editor, output panel with:
- Result grid (see below)
- Live + stat history (collapsible)
- Per-execution timing + row count

**Saved queries.** **Save query** (Ctrl+S) stores the editor's text under a
name; the **saved** list beside it opens one in a tab. **SQL → Saved
queries…** (or *manage saved queries…* at the end of that list) opens their
own dialog: a list you can search by name, note and SQL, and for the one
chosen its name, a note and the SQL to edit, **Open in the editor**,
**Duplicate** and **Delete**. A change is saved only after **Review and
save** has shown it side by side with what is saved now - before on the
left, after on the right, a line only one side has in red or green, changed
words marked - and you press **Save changes**. Every saved change keeps the
version before it (the last 25): **Earlier versions** compares any of them
with the current one the same way and can put it back into the fields, to
review and save. A tab opened from a saved query remembers it: **Save
query** there shows what changed side by side and offers **Update** or
**Save as new**. Saved queries have no limit; the **history** list beside
them keeps the last 30 queries you ran. Per-conn workspace persistence keeps
your open tabs across restarts.

#### EXPLAIN as a tree {#explain-as-a-tree}

Explain and Explain Analyze open as a **tree** by default: one row per plan node, indented under its parent, each showing the planner's cost estimate, estimated rows and (for Analyze) actual rows, this node's own time and its total time including children, and the number of loops. Two things are called out automatically rather than left for you to spot in a wall of numbers:

- **costliest** marks the single node with the most *self*-time - its own time with every child's time subtracted out. This is deliberately not the same as "the node with the largest total time", which is almost always the outermost node simply because it includes everyone underneath it; self-time finds where the time actually went.
- **worst estimate** marks the node with the largest gap between what the planner expected (`Plan Rows`) and what actually came out (`Actual Rows`), in either direction. A large gap here, especially paired with a Seq Scan the planner thought would return little, is the single most common reason a plan goes wrong - it usually means the table's statistics are stale (`ANALYZE` fixes it) or the WHERE clause has a correlation `pg_stats` cannot see.

Per-column badges: `cost` (planner estimate, arbitrary units, only comparable within the same plan), `rows` (estimated → actual, with an "Nx" multiplier once the miss is 3x or more), `self` and `total` time, and `×N loops` when a node under a Nested Loop ran more than once. A short advisory line appears directly under the node it is about - a large Seq Scan, a Sort that spilled to disk, a Hash Join with more than one batch, a Nested Loop over a large outer side - the same categories the summary badges above the plan already used, now read off the node's own fields instead of guessed from the printed text, and shown where the problem actually is rather than in a separate list underneath.

A Seq Scan is judged by the rows it **reads**, not the rows it returns: under EXPLAIN ANALYZE that is the rows it kept plus the rows its filter removed, across every worker, and without ANALYZE a high scan cost counts too. When the filter throws away nearly everything, the line says how many rows were read and how many kept - the case an index on the filtered expression fixes. Only a scan that reads little is called a small table.

**Text view** switches to a plain, copyable text block - the same information, PostgreSQL-EXPLAIN-shaped, for pasting into a chat or a ticket. It is rendered from the same parsed plan the tree reads (not a second call to the server): EXPLAIN ANALYZE actually executes the query, so asking PostgreSQL for the tree format and the text format separately would mean running the query twice - real cost for a slow query, and for a data-modifying statement (wrapped in a transaction that gets rolled back either way) double the writes performed before the rollback. The trade-off is that this text is a close, readable reconstruction rather than guaranteed to be byte-for-byte what `EXPLAIN (FORMAT TEXT)` would have printed natively - the numbers and structure are exact, real PostgreSQL output is the only thing enforcing them, only the surrounding punctuation is reproduced by hand.

#### The result grid

A `SELECT` fetches the first 500 rows and shows them straight away. Scroll to
the bottom and the next 500 arrive, and so on; the status line reads
`500 of 48,213 rows` so you always know where you are in the result. The
count is worked out once, up front - on a table large enough that counting
would take too long it says so instead of a number, and paging still works.

The rows are kept on the server between pages, on a cursor that is closed the
moment you run something else, switch tabs, or leave it untouched for a few
minutes. The editor does not hold a transaction open behind your back.

Past 5,000 rows in the grid the automatic loading stops and a **Load 500
more** button takes over, so a careless scroll on a huge result cannot fill
the window. For the whole result, `Export CSV` streams every row on the
server without going through the grid at all.

There is no sort or filter on the grid itself. It only ever holds part of the
result, so sorting it would order 500 of N and look like it had ordered
everything. Sort in the query, with `ORDER BY`.

#### The search_path box

Beside the mode picker, `search_path` sets where unqualified names are looked
up for this run only. Leave it empty and the session context from the header
applies.

The caret at the end of the field drops the schemas of the connected
database - only the ones this session may actually enter, since a schema you
have no `USAGE` on is not an answer. Yours come first, `pg_catalog` and
`information_schema` last under their own heading. Clicking a name adds it,
clicking it again takes it out, and everything already in the path is ticked,
so the field's whole state is visible in one place. The menu stays open while
you pick, and the field is still typeable for what a list cannot cover.

This matters more than it looks - 
a misspelt schema in a search_path is not an error. PostgreSQL accepts a path
naming schemas that do not exist and then cannot find your table, which reads
as "the table is gone" rather than "the path is wrong".

#### What "Run" actually runs

The editor never guesses at a statement boundary you cannot see. Two buttons,
two scopes:

| Button | Shortcut | Runs |
|---|---|---|
| `▶ Run` | `Ctrl/Cmd + Enter`, or `F5` | every statement in the tab, in order |
| `▶| Statement` | `Ctrl/Cmd + Shift + Enter` | the selected text, or - with nothing selected - the statement the cursor is inside |

"The statement the cursor is inside" runs from the previous `;` to the next
one. Dollar-quoted bodies (`$$ ... $$`), quoted strings and comments are
excluded from that search, so a function or procedure whose body contains
`BEGIN ... ; ... END` counts as one statement rather than four.

`F5` is a page reload in a browser; DBApguin captures it. On the SQL view it
runs, everywhere else it refreshes the current view. `Ctrl/Cmd + R` stays a
view refresh everywhere, including on the SQL view.

#### Transactions and autocommit

The dropdown next to Run decides what happens around the statements in a run:

- **autocommit** (default) - each statement commits on its own, the way
  `psql` behaves. What you ran is durable the moment it returns, and a
  failure halfway through leaves the earlier statements applied.
- **transaction** - the whole run is wrapped in one `BEGIN ... COMMIT`. If
  any statement fails, everything in that run is rolled back and the database
  is exactly as it was. This is the mode to use for a migration.
- **dry run** - the run is wrapped in `BEGIN` and always ends in `ROLLBACK`.
  You get the row counts, the errors and the timings without leaving a trace.
  Nothing you do in this mode is kept, including the parts that succeeded.
- **run as script** - like autocommit, except a failing statement does not
  end the run. Every statement is tried, each failure is listed against its
  own line with the error, and the ones that worked are committed. This is the
  mode for a file of DDL where part of it is already applied: you want the
  rest to land and a list of what did not. The result is the raw output only,
  as `psql -f` prints it - each statement's tag or rows, its notices, its
  error - with **Copy** beside it; there is no grid.
- **keep open (commit yourself)** - the transaction stays open *between* Runs
  and ends when you press Commit or Rollback, which appear beside the mode
  while one is open. Run something, look at what it did, then decide.

  An open transaction holds its locks until somebody ends it, so the state is
  always on screen - how many statements are in it and how long it has been
  open - and one that has not been touched for ten minutes is rolled back for
  you. A failing statement does **not** end it: PostgreSQL marks the
  transaction aborted and nothing else will run until you roll back, which the
  app says rather than throwing your work away.

#### How long a statement may run

Beside the mode there is a timeout: a tick box and a number of seconds. Every
new SQL tab starts with it **on at 180 seconds**, and the setting belongs to
the tab - one tab holding a migration you are willing to wait an hour for,
another for the queries you are still guessing at.

When it fires, the statement is cancelled **on the server**, not just in the
app. Unticking it lets a statement run until it finishes or you cancel it; the
app asks once, because a statement with no limit holds whatever locks it took
for as long as it runs.

`ADD CONSTRAINT`, `CREATE INDEX` and a large `UPDATE` on a real table all take
longer than the old fixed thirty seconds, which is what this replaces.

#### DDL, exclusive locks and the lock queue {#ddl-exclusive-locks-and-the-lock-queue}

`DROP`, `ALTER TABLE`, `TRUNCATE`, `REINDEX` (without `CONCURRENTLY`), `CLUSTER`
and `VACUUM FULL` take an **AccessExclusiveLock** on the table. The important
thing about that lock is what happens *before* the statement gets it:

> A request for an exclusive lock does not jump the queue. While it waits,
> PostgreSQL parks every new lock request on that table behind it - including
> a plain `SELECT`. So a `DROP` that is only *waiting* already blocks new
> reads and writes on those tables, before it has changed anything.

That is how a `DROP SCHEMA` behind one slow report becomes a site-wide outage.
The editor treats these statements differently from a normal query:

- **A pre-flight before it runs.** When the statement takes an exclusive lock
  and something is already on the target, a dialog lists those sessions -
  who, from where, how long, and whether any are already queued - and
  explains the wait before you commit to it. A session in the first second of
  a statement, in a transaction no older than that, is not listed: on a busy
  table there always is one (the application's short statements, this app's
  own dashboard reading table sizes), and it is gone before the DDL would
  notice. A transaction left open - in another program or in another editor
  tab of this app - is listed however young, and so is a statement that has
  run longer.
- **The lock of every statement, before it runs.** The run confirmation lists each DDL statement with the lock it takes and on what, and says when it writes the table again or reads it whole - asked of the server where the text cannot tell: `ADD COLUMN ... DEFAULT clock_timestamp()` (any volatile default - `random()`, `gen_random_uuid()`, a `serial`) rewrites every row, while a constant or `now()` default does not (PostgreSQL 11+); widening `varchar(n)` to a longer `varchar` or to `text`, or `numeric` to more digits at the same scale, is a catalogue change with the rows and indexes kept; `ADD FOREIGN KEY` without `NOT VALID` reads the whole child table while writes to both tables wait, so add it `NOT VALID` and `VALIDATE CONSTRAINT` it afterwards - which on a foreign key also takes ROW SHARE on the table it references, so only DDL that locks that table exclusively waits; dropping a foreign key takes the referenced table under ACCESS EXCLUSIVE as well, and every read of it waits - so does dropping a column a foreign key is made of, dropping a table that references another (with `CASCADE` also the tables that reference it) and changing a key column's type on either side, and the confirmation names that other table; `VACUUM FULL`, `CLUSTER` and `REPACK` hold ACCESS EXCLUSIVE and write the table again; `REINDEX` without `CONCURRENTLY` stops nearly every query on the table. Compare -> Apply reads the statements the same way.
- **A default lock-wait limit.** Exclusive-lock DDL run from the editor gets a
  **3-second** `lock_timeout`, regardless of the statement-timeout tick box.
  If it cannot get the lock in three seconds it gives up, and nothing has
  queued behind it for more than that. This is a guard, not the
  statement-timeout: it only limits the wait *for the lock*, not the work
  after.
- **Turning the guard off.** The pre-flight has a *"wait as long as it takes"*
  checkbox. Tick it and the DDL waits indefinitely for the lock - do that only
  when you can stop the workload, because until the DDL gets in, traffic on
  those tables is stopped too.

If the guard fires, the message says so - which limit, and that it is the
editor's DDL guard rather than a `statement_timeout` or a server-side
`lock_timeout` - and keeps the last picture of the queue on screen.

#### While it runs

The status line keeps a clock, so a statement taking four minutes never looks
like an app that has stopped answering. Once the statement reaches the server
the app also asks what it is doing, from a second connection:

- **waiting on a lock** - shown ahead of anything else. A statement that is
  waiting is not slow, it is stuck, and the two want different reactions. For a
  lock wait the editor shows the whole queue: how many sessions hold the lock
  ahead of you, how long you have waited, and - the part nothing else shows -
  how many new queries have already piled up behind you. Buttons there open the
  Blocking view or terminate the holders.
- **progress**, for the operations PostgreSQL reports it for - `CREATE INDEX`,
  `VACUUM`, `ANALYZE`, `CLUSTER`, `COPY` - with the phase it is in and a bar.

The bar appears only when the server actually reports a percentage. An
invented one would be worse than none.

**Stop** stands next to Run while something is running and stops it on the
server, which is the only place it can be stopped from: the request that
started it is blocked on the same connection, waiting for the same answer.
Escape does the same. The statement comes back as cancelled on request
(57014), and the session survives - including a transaction you are holding
open.

A cancel is a request, not an order, and a backend deep inside a system call
will not notice one. When it does not land within a few seconds the button
offers **Force stop**, which closes the connection instead. That always works
and costs the session: an open transaction is rolled back with it, so it asks
before doing it.

The run belongs to the tab it was started in. Leave that tab and its close ×
becomes a turning ring - a tick when the statement lands, a cross when it ends
in an error. Come back to the tab and the × returns, because by then the
result is on the screen in front of you. Tabs with nothing running of their
own are left alone.

**read-only** decides whether this tab's statements run in a read-only
session. It is off by default, per tab, and the app does not guess: ticked,
PostgreSQL refuses anything that writes, which is a way to put a migration
script in front of a production server and see where it would stop.

It used to be inferred from the SQL instead, and that inference cannot be made
correctly. The first keyword of a statement does not say whether it writes:
`IMPORT FOREIGN SCHEMA` reads like neither, and `SELECT cron.schedule(...)` -
the way pg_cron's own documentation tells you to schedule a job - writes a row
while looking exactly like a query. Both came back refused with *"cannot
execute ... in a read-only transaction"*, on a primary, about a restriction the
app itself had imposed, while the same statement went straight through in
`psql`.

Some statements refuse to run inside a transaction at all: `CREATE INDEX
CONCURRENTLY`, `REINDEX CONCURRENTLY`, `VACUUM`, `CREATE DATABASE`. Run them
in **autocommit**. In the other two modes the server answers *"cannot run
inside a transaction block"*, and the editor says which selector to change.

#### The output panel

Under the result, the app prints what actually happened - the same things
`psql` prints, and one thing it is easy to miss anywhere else.

For every statement in the run: the command tag (`UPDATE 3`, `CREATE INDEX`,
`SELECT 1`), how many rows it touched, how long it took, and the statement
itself. A five-statement script therefore says which one was slow, rather than
giving one total for all five.

Under that, **messages from the server**: everything `RAISE NOTICE` produced
in a function or a `DO` block, and the running commentary of `VACUUM VERBOSE`.
If you debug PL/pgSQL by raising notices, this is where they appear.

The header carries the time the run started and the total duration, and
**Copy** puts the whole panel on the clipboard for a ticket or a mail.

When a statement fails, the panel is the more useful half of the answer: the
statements listed are the ones that already ran. In autocommit they are
committed and will not be undone by the failure; in transaction mode the
whole run was rolled back and none of them stand.

#### Saving the result in another format

`Export CSV` asks the server to run the query again and stream every row, so
it is the one to use for a whole table. The **save as…** picker beside it
writes the result already on screen, which is the one you are reading:

| Format | What it is for |
|---|---|
| JSON | a developer pasting a fixture into a test |
| JSON Lines | one object per line - `jq`, a log pipeline, a bulk load |
| SQL `INSERT` | loading the rows into another database; the real table name is used when the result came from one table |
| Markdown table | a ticket, a merge request, a wiki page |
| Aligned text | a mail or a terminal, readable without any tool |

Every format is written through the same file dialog as any other export, so
you choose where it goes and the app tells you the path afterwards.

### 5.3 Schema

Detailed view of a single object. Tabs:
- Columns (with default, type, nullable, comment)
- Indexes
- Constraints
- Triggers
- Privileges (editable)
- Foreign-key relations (in / out)
- Statistics
- DDL - for a table or a partitioned table this is the column editor (below);
  for everything else it is the formatted CREATE statement
- Data - the rows, editable in place on a table with a primary key. Two
  modes: **builder** puts the query together from the limit, the columns,
  the filters and ORDER BY, and shows the SQL it will run; **raw SQL** lets
  you type the whole query yourself. In raw SQL mode the SQL box is editable
  and the column picker, the filters and ORDER BY step aside; it starts from
  what the builder had. The query runs read-only and must be one SELECT,
  WITH, TABLE or VALUES statement; Limit still caps the rows fetched, and
  Ctrl+Enter runs it. The grid stays editable only when the query returns
  every column of this table under its own name; otherwise the result is
  read-only. Switching back to the builder finds it as you left it.

The schema browser in the left sidebar drives this view. Click an object there to load it here.

#### 5.3.1 Editing columns {#editing-columns}

Select a table and open the DDL tab. Instead of a read-only CREATE TABLE
dump you get one row per column, each editable in place:

| Field | What the button next to it does |
|---|---|
| Name | `RENAME COLUMN` |
| Type | `SET DATA TYPE`, with an optional `USING` expression for conversions PostgreSQL will not do implicitly |
| NOT NULL | `SET NOT NULL` or `DROP NOT NULL`, depending on the checkbox |
| Default | `SET DEFAULT` from the box, or `DROP DEFAULT` |
| Actions | `DROP COLUMN`, with a CASCADE tickbox beside it |

The Extra column shows what the catalogue knows and you cannot edit here:
PRIMARY KEY, `IDENTITY (ALWAYS)` or `IDENTITY (BY DEFAULT)`, and
`GENERATED STORED` or `GENERATED VIRTUAL`.

**Add column** at the bottom takes a name, a type, an optional NOT NULL, and
one of:
- *no default*
- *DEFAULT* - an expression
- *GENERATED* - an expression plus the storage kind. STORED materialises the
  value on write and needs PostgreSQL 12 or newer; VIRTUAL computes it on
  read and needs 18 or newer. The app always writes the keyword out, because
  PostgreSQL 18 changed what a bare `GENERATED ALWAYS AS (...)` means - it
  used to be an error, it is now VIRTUAL - and the same statement would
  otherwise do different things on different servers.

Every action shows you the exact statement first, with the lock it takes and
what that blocks, and runs nothing until you confirm.

Four of them - `SET DATA TYPE`, `SET NOT NULL`, and `ADD COLUMN` with a
DEFAULT or a GENERATED expression - do work proportional to the size of the
table: the first rewrites every row, the second reads every row to prove the
constraint. Those run as background jobs, so they are not cut off partway
through on a large table and they keep going if you close the panel; watch
them in **Database → Background jobs**. The rest are catalogue updates that
finish immediately and run as ordinary requests.

Views, materialized views and foreign tables are not editable here - their
columns come from their definition, so the tab shows the structure only.

#### 5.3.2 Object menus: what each kind can be asked to do {#object-menus}

Right-click an object in the schema browser. Below the navigation and export
entries, every kind carries the commands PostgreSQL actually has for it -
not a common subset. Two rules make the menu honest on every server the app
supports:

- **Nothing is hidden.** An entry this server is too old for is shown greyed
  out, and its tooltip names the version it needs (`needs PostgreSQL 12 or
  newer`). A shorter menu on an old server would leave you guessing whether
  the option does not exist or the app forgot it.
- **The gate is not only in the menu.** Every version and object-kind rule
  the menu draws is enforced again in the endpoint behind it, so a replayed
  request cannot walk past it.

**Index**

| Entry | Notes |
|---|---|
| REINDEX | Exclusive lock, fast |
| REINDEX CONCURRENTLY | PostgreSQL 12+. No write lock; an interrupted run leaves an INVALID index behind |
| REINDEX with options… | CONCURRENTLY, VERBOSE (9.5+) and TABLESPACE (14+), each offered only where it exists |
| Rename / Move to tablespace | |
| Storage parameters… | Only the ones this access method takes: `fillfactor`, `deduplicate_items` (btree, 13+), `fastupdate` and `gin_pending_list_limit` (gin), `pages_per_range` and `autosummarize` (brin), `buffering` (gist) |
| Set statistics on an expression… | PostgreSQL 11+, and only for an expression column - a plain column is refused on every version |
| Drop | CONCURRENTLY and CASCADE as tickboxes, with the statement shown before it runs |

An index that is INVALID gets an extra top-level entry saying so, with a
one-click rebuild. An index that exists only to enforce a constraint says
which constraint, and refuses to be dropped on its own.

**Table, partitioned table, materialized view**

A **Maintenance** submenu with ANALYZE, VACUUM, VACUUM ANALYZE, VACUUM FULL,
REINDEX TABLE, REINDEX TABLE CONCURRENTLY (12+) and CLUSTER. Each one runs as
a background job: close the panel, come back, find it in
**Database → Background jobs**.

*VACUUM with options…* is the full option list, each row greyed out with the
version it needs where the server is older: `DISABLE_PAGE_SKIPPING` (9.6+),
`SKIP_LOCKED` and `INDEX_CLEANUP` and `TRUNCATE` (12+), `PARALLEL` (13+),
`PROCESS_TOAST` (14+), `PROCESS_MAIN` and `SKIP_DATABASE_STATS` and
`BUFFER_USAGE_LIMIT` (16+). `PROCESS_TOAST`, `PROCESS_MAIN` and `TRUNCATE`
are on in PostgreSQL, so they appear ticked - untick one to turn it off.

*CLUSTER…* lists the table's indexes and marks the one it is currently
clustered on, because `CLUSTER` without `USING` only works when there is one.

**TRUNCATE…** is its own entry with ONLY, RESTART IDENTITY and CASCADE, and
shows the statement before running it. It goes through the job path too:
CASCADE can reach a tree of referencing tables, and the lock it needs may
queue behind other sessions first.

Under **Manage**: *Storage parameters…* (`fillfactor`, `parallel_workers`
(10+), `toast_tuple_target` (11+), `vacuum_index_cleanup` and
`vacuum_truncate` (12+), the `autovacuum_*` block, plus the insert-only pair
from 13+), *Move to tablespace…*, and for an ordinary table *Logged /
unlogged…* (9.5+).

**Materialized view** additionally has REFRESH, REFRESH CONCURRENTLY and
REFRESH … WITH NO DATA. The last one empties the view and makes every query
against it fail until the next full refresh, so it asks separately.

**View** - *View options…*: `check_option` (9.4+), `security_barrier` (9.2+)
and `security_invoker` (15+). The last decides whether a query through the
view is checked against your privileges or the view owner's.

**Sequence** - *Sequence properties…* is the whole `ALTER SEQUENCE`
parameter block in one form: `AS` type (10+), INCREMENT BY, MINVALUE,
MAXVALUE, START WITH, CACHE, CYCLE and OWNED BY. Only the fields you change
end up in the statement, and the preview shows exactly what will run.
*Restart…* sets the next value it hands out.

**Trigger** - a **Firing** submenu with all four states: ENABLE, DISABLE,
ENABLE REPLICA (fires only while `session_replication_role` is `replica`) and
ENABLE ALWAYS (fires regardless, so a replication apply or a restore cannot
silence it). Plus *Rename…*.

**Function / procedure** - the editor's Properties block now also carries
COST, ROWS, PARALLEL (9.6+, greyed out below that) and LEAKPROOF, next to the
rename / schema / owner / search_path controls that were already there.

**Schema** - a **Maintenance** submenu that runs ANALYZE, VACUUM, VACUUM
ANALYZE or REINDEX over every ordinary table and materialized view in the
schema, one after another, continuing past a table it cannot lock. The full
panel, with cluster-wide scope and every option, is still one entry below.

#### A copy of the definitions before the change {#copy-before-the-change}

Moving an object to another schema, dropping objects, changing an owner or a
tablespace first writes the definitions of the objects to the data folder,
as SQL that puts them back: for a table the columns, constraints, indexes,
triggers, policies, owner, privileges, comments and the tablespace it and
its indexes live in; for a routine its definition. Background jobs lists the
copy ("Before: ..."), the toast after the change opens it, and from there it
goes into the SQL editor or to a `.sql` file. The copy comes from
`pg_dump --schema-only` when a pg_dump matching the server is available, and
from the catalogue otherwise. When no copy can be taken, the change is
refused with the reason; you may go on without one, and the audit log records
that you did. PostgreSQL itself rolls a failed change back; this is for the
change that succeeded and should not have.

**A drop, and everything it takes.** The right-click **Drop** on a table,
view, sequence, index, routine, type or schema takes the copy as well as the
multi-object drop does - it used to take none. With **CASCADE**, the copy
holds everything the drop removes, not only what was named. Which objects
those are is PostgreSQL's own answer: the drop is rehearsed in a transaction
that is rolled back, and the catalogue says what went. Measured, a
`DROP TABLE ... CASCADE` takes views over the table in other schemas, a view
over one of those, materialized views, a function whose argument is the
table's row type and the foreign keys of other tables; a `DROP TYPE ...
CASCADE` takes the columns of that type out of other tables, values and all;
a dropped trigger function takes its triggers, a dropped sequence the
defaults that call it. The dialog lists them when you ask (**Work out what the drop
takes**), read from the dependency graph: nothing is rehearsed and nothing
is locked to show the list, also for a schema of hundreds of tables.
The copy names them at the top, exactly as the rehearsal found them. When
the rehearsal cannot run - a table too busy to lock within three seconds, or
event triggers that would fire on it - the copy's list is read from the
dependency graph too and the copy says it is an estimate. What the copy
cannot write is named under *NOT in this copy*.

**What to copy first.** The drop dialog, and the dialog that drops several
selected objects, open without asking the server anything. Under the
statement and its options (**CASCADE**, **IF EXISTS**, the SQL) they ask,
in this order:

1. **Keep a copy before the drop?** **No copy - run only the DROP** is the
   first choice: nothing is saved and nothing is worked out first, only the
   DROP statement runs (for a schema with **CASCADE**, `DROP SCHEMA ...
   CASCADE` on that schema and nothing else). It cannot be undone, except
   from a backup. **Keep a copy first** opens the next question.
2. **What to keep:** **Copy definitions only** (the definitions of
   everything the drop takes; a table comes back empty) or **Copy
   definitions and rows** (the rows of the tables that go are written to
   the data folder too).
3. **Work out what the drop takes** - only when you click it: the objects
   **CASCADE** removes and, for the tables that go, how many there are,
   about how many rows and how much data. A copy needs this before **Drop**
   is enabled; without a copy you can ask for it too, but you do not have
   to.

The sizes are the estimate the server keeps (rows and pages from its
statistics), read without opening a table; a table never vacuumed or
analyzed has no size yet, and the dialog says so. A copy first rehearses the
drop to find exactly what goes, and that holds the drop's locks for a moment
(it waits at most three seconds for them). The dialog remembers your last
choice until the application is closed (it is not saved anywhere): the next
drop starts on it, except that rows over 50 MB are turned into the
definitions only once the analysis shows it, and the dialog says why.
Without a copy, when the drop can take data or other objects (a table, a
schema, anything with **CASCADE**, unless the analysis showed nothing goes),
you type the name of the object (for several objects, how many) before
**Drop** is enabled. A kind no copy can be written for (an extension, a
trigger, a role ...) says so and offers only the analysis.

**While it runs.** The drop is a background job, and the dialog follows it
step by step: working out exactly what the drop removes, copying the
definitions, copying the rows (so many of so many tables, how much is
written), dropping - with the time it has taken so far. Until the DROP
itself starts, **Cancel** reads **Stop**: it stops at the next safe point,
nothing is dropped, and a copy taken up to then is removed with its rows.
Once the DROP runs it cannot be stopped, and the button says so. Closing the
dialog does not stop the job; it stays in Background jobs. Moving and
cloning from the same dialog show the same busy state.

**The rows.** The definitions are not the data. **Copy definitions and
rows** writes the rows of every table the drop takes to the
data folder next to the copy, and the values of a column a cascade takes out
of a table that stays (matched back by its primary key; a table without one
cannot have them back, and the copy says so). To undo a drop: open the copy,
run it in the SQL editor, then **Put the rows back** - one transaction,
parents before children, into tables that must exist again and be empty, so
nothing is loaded twice. The rows go back where the copy was taken and
nowhere else: the copy records the server (its address, libpq service or SSH
tunnel, and the server's own system identifier) and the database, **Put the
rows back** uses an open tab or a saved connection that reaches that server -
never the tab in front when that one is somewhere else - and the server is
asked who it is before a row is written. **Open in SQL editor** opens the copy
only on a tab connected to that database, and says so when there is none. A foreign key of a table that stays is put back
`NOT VALID` when that table has rows pointing at a table that comes back
empty, and checked once the rows are back; materialized views are refreshed
then too. A foreign table's rows live on the other server and are not kept.
With the definitions only, the copy says at the top that it holds the
definitions only.

### 5.4 Sessions

Three modes (toggle at top):

- List: pg_stat_activity sortable table with pid, datname, user, app, client, state, wait, ages, query.
- Live load: progress + ETA per active query. Joins pg_stat_statements history (mean / stddev / blocks-per-call) and pg_stat_progress_* views (VACUUM / ANALYZE / CREATE INDEX / CLUSTER / COPY).

#### "idle in transaction" and what it costs

A session in this state gets one of three readings, and they are measured
rather than assumed:

| what it says | what it means |
|---|---|
| *N sessions waiting on it* | somebody is stuck behind it. This is a blocking problem, and the same sessions appear in Blocking and in the `db-locks` check |
| *holding N locks, nobody waiting* | it took locks its statements needed and will keep them until it commits, but nothing wants them yet |
| *holding no locks anyone can wait for* | a read-only transaction that has been left open. It blocks nobody |

The third is the common one, and it used to read "holding locks" like all the
others. That sent people to Blocking and to the cluster health locks, where
they correctly found nothing - the label was describing the state, not the
locks. Its own virtual transaction id is not counted: every session has one and
nothing can wait for it.

A long-open transaction is still worth closing even in the third case, and the
card says so past five minutes: the oldest open transaction holds back the xmin
horizon, so autovacuum cannot clean any row version newer than it - anywhere in
the database, not only in the tables that transaction touched.
- Dashboard: one-screen snapshot (versions, conn states, wait events, cache hit, top hot tables, checkpoint stats, blocking, replication lag, longest open tx, autovacuum workers).

Per-session right-click: Cancel (cancel current query) or Terminate (kill the backend).

### 5.4.1 What an open window costs the server {#what-an-open-window-costs-the-server}

A connected window asks the server one thing every five seconds: the active sessions, the facts behind the header's alarm lamps, and whether the server is a primary or a standby. The answers about the server that do not change from one tick to the next (which columns and functions it has, where pg_stat_statements lives and whether it is loaded, the worker and archiving settings) are asked once and kept, for ten minutes or for one minute when an extension can change them, and an extension installed, updated, moved or dropped from the app is asked about again at once. The Live load view uses the same five-second answer as the header instead of asking a second time. Measured on PostgreSQL 19, for one window:

| Window | Statements a minute | Server time a minute | Blocks a minute |
|---|---:|---:|---:|
| Idle (a check tab in front) | about 500 | 14 ms | about 2,000 |
| Live load in front | about 470 | 12 ms | about 1,500 |

Most of the statements are the few settings each request makes on its connection (role, search path, time limits), under 0.1 ms each. Nothing is asked while the window is minimised. A check costs what its own queries cost, once, when you run it; the ASH sampler adds one read of pg_stat_activity per sample (see *db-ash* above).

### 5.4.2 The header alarms and their lines {#header-alarms}

The header's alarm lamps are graded from the same five-second answer. Every line is a judgement, so here they are, with the reason (reviewed 27.10.2026 after the spill alarm and the wraparound alarm had both been crying wolf):

- **Connections:** warning at 80% of `max_connections`, critical at 95%.
- **Blocking:** warning when somebody has waited 5 s on a lock, critical at 60 s - a minute behind a lock is not going to let go by itself; fifteen seconds, the old line, was an ordinary migration.
- **Spilling to disk:** the rate is a 60 s average of `pg_stat_database.temp_bytes`, and it has to *stay* up: warning after three minutes above 2 MB/s, critical after five minutes above 20 MB/s. One report that spills once pushed the average over the line and the alarm stood for a minute; it no longer does. While `CREATE INDEX`, `REINDEX`, `CLUSTER` or `VACUUM FULL` run (PostgreSQL 12+ says so), their sorting on disk is `maintenance_work_mem`'s business and is not graded.
- **Transaction id wraparound:** `autovacuum_freeze_max_age` (200 million by default) is where autovacuum starts anti-wraparound vacuums by itself, and busy tables sit near it in normal operation - the old lines, 80% and 90% of it, were 8% of the way to the real stop. The alarm now lights at five times the setting, capped at 1 billion, and goes critical at 7.5 times, capped at 1.5 billion, which is where the Health check says CRIT; PostgreSQL's failsafe vacuum starts at 1.6 billion and the hard stop is near 2.1 billion. The caps hold for a DBA who raised the setting; the multiples let a test cluster reach the lines by lowering it. The text says the age in millions and the way to the stop in percent, and names the oldest relation, the database it is in, every relation past the line, and the snapshot holder vacuum cannot get past. Multixact ids are graded by the same rule against `autovacuum_multixact_freeze_max_age` and lead the text when they are the older of the two.
- **An open transaction:** warning at 5 minutes. Critical at 30 minutes only when it is *idle in transaction* - something nothing will end; an active statement that old is a report or a batch and stays a warning.
- **Prepared transactions:** warning after an hour, critical after a day.
- **A dead replication slot:** warning as soon as it is inactive, critical once it pins 10 GB of WAL.
- **WAL archiving:** warning when the last attempt failed; critical once it has kept failing for five minutes - a failed attempt is retried a minute later. The alarm names the segment that failed and when, the last one archived, the command or library, and how many segments wait in `pg_wal`. `archive_mode` on with no `archive_command` or `archive_library` is a warning, critical once 10 GB of WAL waits; both set is critical (from PostgreSQL 16 the archiver refuses to run).
- **Worker pools:** warning at 80% of a pool, critical at 95%.
- **Replication lag:** warning at 60 s, critical at 300 s.
- **LISTEN/NOTIFY queue:** warning at 10% of the queue's maximum or 512 MB, critical at 50% or 4 GB, whichever comes first. The lamp is shown where somebody listens (or the queue already holds something); it goes to *Queue tables and LISTEN/NOTIFY*, which names the listening sessions.

A critical alarm opens the window that asks for an acknowledgement (recorded in the audit log) and can be put on the task list; a warning is a lamp and a notification. The question before a critical window can be switched off in Settings.

### 5.5 Maintenance

Maintenance operations + autovacuum tuning.

- Pick a schema + table, then run VACUUM / VACUUM (FULL) / VACUUM ANALYZE / REINDEX / CLUSTER / ANALYZE / pg_repack. Live progress polls pg_stat_progress_*.
- Per-table autovacuum advisor: edit reloptions (`autovacuum_vacuum_scale_factor`, `autovacuum_vacuum_threshold`, ...) with a recommended target derived from table heat (very hot / hot / medium / default). On PostgreSQL 13 and newer an insert-only table also gets `autovacuum_vacuum_insert_threshold` and `autovacuum_vacuum_insert_scale_factor` recommended and applied: dead rows never reach its vacuum trigger, so the insert trigger is the only vacuum it gets.

The option list is built from the server's own version. VACUUM's PROCESS_TOAST
arrived in 14 and BUFFER_USAGE_LIMIT in 16; sending one to an older server is a
hard error, not a warning, so anything the server predates is not offered. The
one-click VACUUM / ANALYZE / REINDEX buttons in the table advisor and the vacuum
status advisor draw on the same list, so the same button means the same thing
wherever it is pressed.

There is no log pane here. A maintenance run is a background job like any other:
the panel shows that it is running and links through to **Background jobs**,
where its output is read alongside every other long operation.

**A run that did nothing says so.** PostgreSQL skips a relation the caller may
not maintain and still reports success - the statement returns VACUUM, with only
a warning to say nothing happened. The job ends as *skipped* rather than *done*,
and names what was skipped.

### 5.6 Server

Per-cluster settings + pg_hba editor + extensions + tablespaces + roles.

- Key settings: editable cards for ~70 tuning parameters with rationale, rule-of-thumb, and status badges. Save changes via ALTER SYSTEM (where supported by the provider).
- pg_hba.conf: text editor with backup-before-save. Restore from any backup. Provider must support raw file write.

#### Every ALTER SYSTEM made here can be undone {#settings-undo}

Each change applied from DBApguin - Key settings, the tuning advice, the
settings editor - is recorded with the value it replaced and where that
value came from. The toast that confirms the change carries **Undo**, and
the settings view lists the changes made here, newest first, each with
Undo until it is undone. Undo is the exact statement that returns the
parameter to how it was: `ALTER SYSTEM SET` to the old value when that
value was DBApguin's own line in `postgresql.auto.conf`, `ALTER SYSTEM
RESET` otherwise, so the old source (postgresql.conf, the default) applies
again. The record is per server, in the data folder, and the audit log
keeps the value before and after.

#### pg_hba.conf belongs to one node

`pg_hba.conf` is a file in one server's data directory, and replication does
not carry it: WAL streams data, not configuration. A rule added on the primary
is not on the standby, which still has whatever its base backup contained. The
day that matters is a failover - connections that worked yesterday are refused
by a node that never got the edit, and by then nobody is looking at this
screen.

So the editor says so above the file, and names the nodes wherever the server
will say who they are:

- **standbys streaming from this server**, with their application name and
  address, from `pg_stat_replication`;
- **physical replication slots**, when a standby is configured but currently
  away - a slot with nothing attached is still a node that is expected back;
- **"this node is a standby"**, when you are editing the copy on a replica: the
  primary has its own, and an edit here does not reach it.

When none of that is visible the note is shorter and stays. A role without
`pg_monitor` cannot read `pg_stat_replication`, and a disconnected standby
looks the same from here as one that does not exist. Whatever the topology,
the same change has to be made on every node - including a connection pooler
in front of them, which has its own rules of its own kind.

**Editing the file on a replica works.** It used to fail with *cannot set
transaction read-write mode during recovery*, a message about transaction
modes from an operation that was writing a file. Two things caused it: the
application asked PostgreSQL for a read-write session, which a server in
recovery refuses outright, and the file was written through a large object,
which is a write to the database. Neither is necessary. The session is now
asked for what the server can give, and on a standby the file is written with
`COPY … TO`, which touches nothing in the database. The backup before the
save, the reload afterwards and the parse-error report all work the same.
- Extensions: inventory + per-extension install / update / drop / set schema with version picker.
- Tablespaces: inventory + create / alter / drop.
- Roles: listing with attribute editor. Click a role to drill into the full role audit.

### 5.7 Roles, users and privileges {#roles-users-and-privileges}

#### The roles editor

`Roles and users overview` lists login users and group roles in two editable
tables. Every boolean attribute is a true/false dropdown; `conn_limit` and
`valid_until` are text fields.

**Pick a role, then act.** Click a row to select it, or tick its box;
ctrl-click and the boxes select several at once. One row of buttons sits next
to *+ Create role / user* and acts on whatever is selected - it stays hidden
until something is. With more than one role selected, the actions that make
sense for a set stay enabled (**Privileges**, **Lock**, **Drop**, **Save**)
and the ones that edit a single role's own settings grey out and say so.

**Save** sends only the difference, so a role you looked at and left alone
issues nothing. With several selected it issues one `ALTER ROLE` per role that
you actually changed, and lists them in the confirmation first.

**Edit** opens that role in a tab of its own - its attributes, its privileges
and its memberships in one place, one tab per role, so two roles can be worked
on side by side. The privilege form in that tab is the same form the
**Privileges** button opens as a dialog.

**Privileges** with several roles selected shows the first role's current
privileges and writes the same GRANT / REVOKE set to every selected role; the
banner at the top names them. There is no such thing as the shared current
state of five roles - what is shared is the change - so the baseline shown is
one role's and the dialog says which.

Group roles carry a **`can_login`** column of their own. A group role is
`NOLOGIN` by definition, so the column always starts false - it is there so
you can turn a group role into a login role later without leaving the app.
Login users have a **Lock** button for the other direction, which sets
`NOLOGIN` behind a confirmation.

#### Limits and timeouts for one role

Open a role's attributes and the **Limits and expiry** section holds, beside
the connection limit and the expiry date, the five per-session timeouts:

| setting | stops |
|---|---|
| `statement_timeout` | a single statement running too long |
| `lock_timeout` | waiting for a lock |
| `idle_in_transaction_session_timeout` | a session idle inside an open transaction |
| `idle_session_timeout` | a session idle outside a transaction (PostgreSQL 14+) |
| `transaction_timeout` | a whole transaction, however busy (PostgreSQL 17+) |

Only the ones this server has are offered - the last two arrived in later
versions, and offering a field the server will refuse would be worse than
leaving it out.

**A bare number is milliseconds.** PostgreSQL reads an unqualified value in
the setting's own unit, and for all five of these that unit is milliseconds:
`30` means thirty milliseconds, which is not what anybody typing 30 means.
Write `30s`, `2min` or `500ms` and there is nothing to guess at. An empty
field clears the setting for the scope you are in.

**Otherwise** is the value the role meets when this scope sets nothing, and it
names the layer it came from - the role's own role-wide setting, the
database's, or the cluster's, in the order PostgreSQL applies them. The
cluster figure says whether it comes from a configuration file or is the
built-in default, because those mean different things when you are deciding
whether to set one. It is read without the connected database's own settings
in the way: a database carrying
`ALTER DATABASE … SET idle_in_transaction_session_timeout = '90s'` used to make
this column report 90s as the cluster's, when the cluster imposes nothing.

**Scope** chooses where a change lands. Left at *every database* it is
`ALTER ROLE … SET`; pick a database and it is `ALTER ROLE … IN DATABASE … SET`,
which is the narrower setting and wins there. The list marks the databases
that already carry something for this role, so twenty databases with two
overrides no longer means opening the list twenty times.

The rest of a role's attributes - LOGIN, INHERIT, SUPERUSER, CREATEDB, the
connection limit, the expiry - are **not** per-database and have no scope
selector. They live in a catalogue shared by the whole cluster, so `ALTER ROLE`
has the same effect whichever database you run it from. Privileges on tables,
schemas and functions are the opposite: those live in each database's own
catalogues, and the privilege editor applies them in the database named at the
top of the window.

These are `USERSET` settings: the role can raise or disable them for its own
session with a plain `SET`. They stop an accident, not somebody deliberate.
The one limit a role cannot lift for itself is the connection limit above
them - worth knowing when choosing between the two.

#### What one membership grants

Ticking a role in **Member of** issues `GRANT parent TO this_role`. Beside each
one sit the options that decide what the membership actually gives:

| | |
|---|---|
| **ADMIN** | the member may grant this role on to others, and revoke it. Off by default; without it the membership is a dead end |
| **INHERIT** | the member uses the role's privileges automatically, without `SET ROLE`. PostgreSQL 16+ |
| **SET** | the member may `SET ROLE` to it at all. PostgreSQL 16+, on by default |

The last two arrived in PostgreSQL 16, which split one membership into three
separate answers; on older servers only ADMIN is offered, because only ADMIN
exists. Each box shows what the membership currently carries, read from the
server, so a membership granted elsewhere is displayed as it really is rather
than as a default.

INHERIT off with SET on is the shape worth knowing: the member holds the
privileges only while it has deliberately become the role, which is how you
stop objects being created under the wrong owner by accident. INHERIT on with
SET off is the opposite - the privileges apply always, and becoming the role is
forbidden.

Changing an option on a membership that already exists issues the `GRANT`
again. There is no `ALTER` for one, and the SQL preview shows exactly that.

Every box in this dialog and in Create role explains itself on hover.

#### The privilege editor

Opened from a role's right-click menu, or from `Privileges` in the roles
editor. It applies to one role and one schema at a time; the schema picker is
at the top.

The checkboxes show **what the role already has**, and they have three states
rather than two, because each one covers every object in the schema at once:

| State | Meaning |
|---|---|
| Ticked | every object in the schema has this privilege |
| A dash | some do and some do not - the count is shown beside the label, e.g. `[3/12]` |
| Empty | none do |

A dash is not a decision. Leave it alone and nothing is issued for it. Click
it once and it becomes ticked - a `GRANT … ON ALL …`, which brings the rest
up. Click again and it clears - a `REVOKE`, which takes it from the few that
had it.

The SQL panel at the bottom shows exactly what Apply will run, and reads
"no changes" until you actually change something. Opening the editor and
pressing Apply does nothing.

Greyed-out privileges are ones the **connecting user** cannot grant; the
tooltip says so.

An `Actions` button beside Apply carries everything from the role's
right-click menu - audit, `search_path`, create-like, copy privileges, export
DDL, compare, drop - so you do not have to close the editor to reach them.
Anything that changes the role closes the editor first, so a stale screen
cannot be applied on top of it.

#### A caution about what "has" means

These screens report **effective** privileges: what the role can actually do,
including what it inherits through group membership and what it gets from
being a superuser. A superuser will therefore show everything ticked, whether
or not a single explicit grant exists. To see direct grants only, use the
per-object privileges screen in the Schema view.

#### Export GRANTs

**Export GRANTs** writes privileges as a script to keep in git or to run
somewhere else. It is in three places:

- the privileges window of one object: that object, and the grants on its
  columns (a column grant on a partitioned table too);
- the schema privileges window: the schema and every table, view, sequence,
  column, routine and type in it, with the default privileges set
  `IN SCHEMA` for it;
- the role privileges audit: everything the role was granted, in the
  databases you tick (the connection's database is ticked, *All databases*
  reads every one), and its grants on databases, tablespaces and server
  parameters. With more than one database each part starts with `\connect`,
  so run that script with psql.

The script turns objects created afresh by the same owners into the ones you
exported. An owner's own privileges are therefore not written, and a
privilege that was taken away - `EXECUTE` from `PUBLIC`, an owner's own
`TRUNCATE`, a default privilege removed with `ALTER DEFAULT PRIVILEGES ...
REVOKE` - comes out as a `REVOKE`. What a role reaches through membership in
another role is not in its export; the script names those roles.

### 5.8 Renaming a schema

Right-click a schema in the browser → **Rename schema**.

The rename itself is one `ALTER SCHEMA ... RENAME TO ...`, and PostgreSQL
carries almost everything with it: tables, views, sequences, indexes,
constraints, foreign keys and grants all follow the schema without any
further work. The name is checked before the attempt, so a name that is
already taken is reported as an answer rather than as a failure.

What does **not** follow automatically is any reference written as text
inside a routine - a function body that says `SELECT * FROM old_name.t`, or a
`search_path` set on a function. Those are listed after the rename under
*"routines still reference ..."*, with the routine name and where the
reference was found (body, `search_path`, or both). Nothing is rewritten for
you: a function body is code, and silently editing someone's code is worse
than telling them which four functions to look at.

If the target name already exists, a rename is not what you want - see below.

### 5.9 Merging two schemas

Right-click a schema in the browser → **Merge schema**.

`ALTER SCHEMA ... RENAME` cannot fold one schema into another, because the
target name is already taken. Merging is therefore per-object, and every
object needs its own decision when the same name exists on both sides.
DBApguin works this out before anything happens.

#### The plan

Picking a target and pressing **Analyse** runs a preflight that changes
nothing and classifies every table, view, materialised view and sequence in
the source:

| In the target | What it means | Actions you can pick |
|---|---|---|
| `not there` | only the source has it | moved - there is nothing to decide |
| `same columns` | both have it, column names and types match | **Merge rows**, **Replace**, **Skip** |
| `columns differ` | both have it, the shapes disagree | **Replace**, **Skip** |
| `moves with its table` | a `serial` / identity sequence | none - it follows its table |

Objects that exist only in the *target* are listed underneath and are never
touched.

The action column is per object. **Set all** is a convenience that applies a
choice everywhere it is legal and leaves the rest alone - it will not arm
*Merge rows* on a table whose columns differ. An action the preflight did not
offer is rejected by the server rather than quietly downgraded, so what the
dialog showed you is what runs.

#### What each action does

- **Move** - `ALTER ... SET SCHEMA`. The object itself changes schema. Its
  grants, comments, indexes and constraints go with it, and it is **gone from
  the source**.
- **Copy** - the source keeps everything. The table is recreated in the target
  with `LIKE ... INCLUDING ALL` (defaults, constraints, indexes, comments,
  storage settings) and its rows are inserted; a sequence is recreated at the
  same value. Offered for tables and sequences only: a view's definition names
  the schema it reads from, so a copied view would quietly go on reading the
  source's data, which is worse than not offering the choice.
- **Replace** - the target's object is dropped `CASCADE` and the source's
  takes its place. This destroys the target's data *and* the target's grants
  on that object. It is the only genuinely destructive choice in the dialog,
  so the Merge button turns red and counts what will be dropped.
- **Merge rows** - the target's table stays exactly where it is and the
  source's rows are inserted into it. The source table is not emptied and not
  dropped: its rows are still there afterwards, so the merge can be repeated
  or checked before you clean up.

So the source survives **Copy** and **Merge rows**, and does not survive
**Move** or **Replace**. The plan says which each object will do before you
run it.
- **Skip** - the target's object is left alone.

#### When rows collide

Only relevant to *Merge rows*, and decided by the primary key of the target
table:

- **Keep the target's row** - `ON CONFLICT DO NOTHING`. Rows that are already
  there win; new rows are added.
- **Let the source's row win** - an upsert. Colliding rows are overwritten
  with the source's values.
- **Stop the merge** - a plain `INSERT`. The first collision aborts, and
  because the whole merge is one transaction, nothing at all is applied.

If the target table has **no primary key** there is nothing for a conflict to
match on, and every source row is appended - duplicates included. The plan
says so on the row itself, before you run it, rather than leaving you to find
the doubled rows afterwards.

#### Things worth knowing

- **The whole merge is one transaction.** If anything fails, nothing is
  applied. A merge that stopped halfway would leave two half-populated
  schemas and no record of where it stopped, which is worse than not having
  started.
- **A `serial` or identity sequence moves with its table**, because
  PostgreSQL moves it as part of `ALTER TABLE ... SET SCHEMA`. It is listed
  in the plan so you can see it will move, but it has no action of its own.
  Standalone sequences are ordinary objects and do get a choice.
- **Everything in the schema is moved, not just the tables.** `ALTER ... SET
  SCHEMA` works on every schema-scoped object kind, so functions, procedures,
  aggregates, types (enum, domain, composite, range), collations, conversions,
  operators, operator classes and families, text search configurations,
  dictionaries, parsers and templates, extended statistics and foreign tables
  are moved along with the relations. They are listed in their own section of
  the plan, *Everything else in the source*, and each takes its own action the
  same way a table does.

  For a long time this was not the case: the plan was built from `pg_class`,
  the four relation kinds moved, and the functions and types were abandoned in
  the source without a word.
- **A range type carries its companions.** `ALTER TYPE` moves a range type and
  its array type and nothing more: the multirange PostgreSQL generated with it
  and the constructor functions of both stay behind, and a range type that
  moved while they did not goes on depending on the source schema. They are
  moved in the same statement group, and the plan says how many came along.
- **What can't move says why.** Two cases. An object whose identity is already
  taken in the target - a routine with the same name and argument types, an
  operator with the same operands, an operator class with the same index method
  - can only be skipped: `SET SCHEMA` would fail on the collision, and since
  the merge is one transaction that one failure would undo all of it. Unlike a
  table, these get no **Replace**, because dropping a type in the target
  `CASCADE` would take the target's own columns with it and the dialog cannot
  show you a count of what that destroys; remove the target's object by hand
  first. The other case is a non-relocatable extension, which PostgreSQL will
  not move at all.
- **Anything that stays is named, and so is what still needs it.** If a type
  stays behind because its name was taken, the moved table's column is still
  typed by it. That dependency is by OID and does not break, but the source
  schema cannot be dropped while it holds the type, so both the object and the
  column that needs it are listed. When everything moves, both lists are empty.
- **Drop the source when it is empty** drops it only when the schema really is
  empty. `DROP SCHEMA` is `RESTRICT`: anything still in the source refuses it,
  and since the drop is the last statement in the one transaction, that refusal
  used to roll back every move before it - you waited for a merge that moved
  nothing and were handed a dependency error about objects you never asked to
  move. Now the merge is committed, the source is kept, and the result says
  which objects kept it.
- **Hardcoded references are flagged before you run.** Views, foreign keys
  and CHECK constraints are rebuilt from the catalogue on every read, so they
  follow a moved table for free. A PL/pgSQL function or trigger body is plain
  text resolved by name at call time: a body that says `public.orders` keeps
  pointing at the old schema after the table becomes `archive.orders`, and
  fails at the next call rather than at merge time. The preflight scans
  `pg_proc` and `pg_policies` for exactly this and lists what would break, so
  you fix those bodies first or decide the reference is fine. RLS policies
  are OID-bound like a view and are flagged only when they call a function
  the scan already caught.
- **Foreign keys keep pointing where they pointed.** A moved table whose
  foreign key referenced a table left behind ends up with a cross-schema
  foreign key that still enforces. It is valid, and it is usually a sign that
  the referenced table wanted to move too.
- **Drop the source schema if it ends up empty** does exactly that, and only
  that: if anything was skipped, the schema stays and the dialog tells you
  how many objects are still in it.

### 5.10 Backup and restore

Right-click a database → **Backup** or **Restore**, or a schema →
**Export schema (pg_dump)**.

Both run the real `pg_dump` / `pg_restore` binaries. Their location comes
from Settings → Tools if you set one, otherwise from auto-discovery; a
version mismatch between the tool and the server is reported rather than
guessed at.

#### The destination is checked first

A dump that fails after twenty minutes because the folder was read-only is a
waste of twenty minutes, so the path is checked before `pg_dump` starts:

- a directory given where a file was expected is refused
- a folder you cannot write to is refused, naming the folder
- an existing file is reported rather than silently overwritten

Use **Browse** to pick a folder instead of typing a path. It shows only
directories, marks the ones that are writable, and hides dotfolders. It
browses the **server's** filesystem, not this machine's - pg_dump runs on the
server, so that is the filesystem the path has to exist on. Both the database
backup and the schema export use the same browser.

#### Backing up the whole cluster

Right-click a database → **Backup cluster (pg_dumpall)…**

`pg_dump` works inside one database and structurally cannot reach what lives
outside it. **Roles, tablespaces and per-database settings are cluster-wide**,
so a backup made only with `pg_dump` restores tables that nobody can log in
to own - which is a thing people discover during a restore, not before one.

| Contents | What you get |
|---|---|
| Globals only *(recommended)* | roles and tablespaces - the companion to your per-database dumps |
| Roles only | just `CREATE ROLE` |
| Tablespaces only | just `CREATE TABLESPACE` |
| Everything | globals plus every database in the cluster, in one file |

*Globals only* is the one most people want: keep taking per-database dumps in
the custom format, and take this alongside them. *Everything* produces one
large plain-SQL file and is slower to restore selectively.

`pg_dumpall` has **no format option** - the output is always plain SQL, so it
goes back in with `psql`, not `pg_restore`. The dialog says so after the dump
finishes rather than leaving you to find out.

#### pg_dump options

Open **Options** in the backup dialog. Every control maps onto one `pg_dump`
flag, and the flag is printed on the control so what you picked can be
checked against the PostgreSQL documentation:

- **Format** - custom (`-Fc`, the one `pg_restore` wants), plain SQL, directory or tar
- **Compression** - 0-9; see below
- **Section** - which third of the dump to write; see below
- **Only these schemas / Only tables / Exclude tables / Exclude table data / Exclude schemas** - comma-separated names, see *Names and partitions* below. **Pick…** beside *Only these schemas* lists the database's schemas to tick
- **Parallel jobs** - directory format only. The field is disabled for every other format, with the reason shown next to the options, rather than letting you fill it in and fail after the fact
- `--no-owner`, `--no-privileges`, `--clean`, `--create`, `--inserts`, `--no-comments`, `--no-tablespaces`
- **Advanced** - `--load-via-partition-root`, `--no-publications`, `--no-subscriptions`, `--no-security-labels`, `--enable-row-security`, large objects in or out, a lock wait timeout, *Only extensions* (`--extension`) and *Run as role* (`--role`)

`--no-owner`, `--clean` (with `--if-exists`), `--create` and
`--no-tablespaces` only change a **plain SQL** file. For a custom, tar or
directory archive `pg_dump` ignores them without a word - they are chosen when
the archive is restored - so they are switched off for those formats, with
the reason under the options, and the summary does not promise them.

Which `pg_dump` runs depends on the server (the client is matched to it), and
an option that client does not have is switched off with the version it
needs - `--extension` needs 14, for example. The line under *Advanced* names
the client.

**Names and partitions.** Write names as the catalogue shows them, upper and
lower case included: `Sales.Orders` finds the table the tree shows as
`Sales.Orders`. Put a name in double quotes when it contains a dot or a comma
(`"order.lines"`). Every name is looked up before the dump starts: one that
matches nothing - a typo, a table in another schema - stops it with the name
and, when the case is the only difference, the name that does exist. A name
that matches nothing used to be silently left out of a dump that then looked
complete.

A partitioned table is its partitions: the parent alone holds no rows. So a
table in *Only tables* comes **with its partitions and child tables, at every
level** (the box under the field, ticked by default), and so does a table in
*Exclude tables* or *Exclude table data*. With `pg_dump` 16 or newer that is
`--table-and-children`; an older client gets every partition named one by
one. Once the dump is written, its check confirms that every table asked for,
partitions included, is in the file. What `pg_dump --table` leaves out is said
when the dump starts: what the picked tables use that is not theirs - the
function a trigger calls, a standalone sequence a default draws from, the type
of a column, a table a foreign key points at, an extension - which the
database the dump is restored into needs already, and what uses them (a view,
a foreign key of another table).

Nothing is passed through as free text. An option DBApguin does not know is
not sent, which is what stops a stray `--something` from ending up on the
command line.

**Exporting one schema uses the same options.** Right-click a schema →
*Export schema (pg_dump)* opens the same option set, minus *Exclude schemas*
 - which cannot change anything in a dump already restricted to one schema.
Both dialogs are built from one definition, so an option added to either
appears in both.

One thing the schema export does for you: `pg_dump` matches `--table`
patterns against *schema-qualified* names, so asking for `orders` inside a
single-schema dump matches nothing and the tool stops with *"no matching
tables were found"*. Names typed without a schema are qualified with the
schema being exported, so `orders` means `sales.orders` - and the schema is
quoted, so a mixed-case one is not folded to lower case on the way.

The full destination path is shown under the path field in both dialogs. A
text box shows the end of a long path and hides the beginning, which is the
half naming the machine and the directory.

#### Section: what the three parts are

A dump is written in three passes, and `pg_dump` lets you take one of them on
its own. **all** is the default and is what you want unless you have a
specific reason not to.

| Section | What lands in it |
|---|---|
| `pre-data` | `CREATE SCHEMA`, `CREATE TABLE`, `CREATE VIEW`, `CREATE SEQUENCE`, owners - everything a table needs to exist, but nothing that would slow the load down |
| `data` | the rows themselves, as `COPY` blocks, plus `setval` calls that leave sequences at their current value |
| `post-data` | primary keys, unique and foreign-key constraints, indexes, triggers - everything that checks or indexes the rows |

The split is not arbitrary: constraints and indexes go last **because loading
rows into a table that already has them is far slower**, and every row would
be checked and indexed one at a time. Restoring pre-data, then data, then
post-data builds the indexes once, at the end, over the finished table.

Taking one section alone is for special cases:

- `pre-data` - the empty shape of a database, to compare structures or to
  build a target you will fill some other way
- `data` - refreshing rows into a structure that already exists
- `post-data` - recreating constraints and indexes after a data-only load

Restoring a `data` dump into a database with no tables fails, and so does
`post-data` without the tables to attach to. The sections are pieces of one
dump, not three independent backups.

#### Compression: what the numbers cost

Compression applies to the custom, directory and plain formats. **Tar cannot
be compressed at all** - the field is disabled when you pick it.

Measured on a 120,000-row table of hashes (a realistic mix, not a file of
repeated text):

| Level | Size | Time |
|---|---|---|
| 0 - none | 13.4 MB | 0.3 s |
| 1 | 7.5 MB | 0.8 s |
| 5 | 7.35 MB | 1.5 s |
| 9 - smallest | 7.35 MB | 1.7 s |
| default | 7.35 MB | 1.6 s |

The shape of that table is the whole point: **almost all of the saving is
between 0 and 1.** Level 1 gives 44% off for half a second; going from 1 to 9
buys another 2% and takes twice as long. Level 9 is not "much smaller", it is
"the same size, slower".

So:

- leave it on **default** unless you have a reason - it is already near the
  best size
- pick **0** when the dump is going somewhere that compresses anyway, or onto
  a fast disk where the time matters more than the space
- pick **9** only when the file has to travel over something slow and you do
  not care how long the dump takes

The custom and directory formats compress by default; a plain SQL dump is
uncompressed unless you ask, and asking makes it a gzip file rather than
readable SQL.

#### Importing one schema

Right-click a database → **Import schema (pg_restore)…**. It is the restore
dialog with the schema filter in front: name the schema as it appears in the
dump, and only that schema is restored into this database. Leave it empty and
the whole dump is restored, which is plain Restore.

This works with a whole-database dump too - you do not need a schema-only
dump to pull one schema out of it.

One thing DBApguin does for you here: `pg_restore --schema sales` restores
the objects *inside* `sales`, but `CREATE SCHEMA sales` is not one of them.
Against a database that does not already have the schema, every single object
fails with *"schema sales does not exist"* - and `pg_restore` still exits
reporting only "errors ignored on restore". When the database does not have
the schema, it is therefore restored from the dump first - with its owner,
its grants and its comment, as the dump has them. A schema that exists is
left exactly as it is.

#### Importing under a different schema name

PostgreSQL has no equivalent of Oracle's `REMAP_SCHEMA`. Neither `pg_dump`
nor `pg_restore` can rename a schema on the way in - there is no such option
in either tool.

What it does have is `ALTER SCHEMA … RENAME`, which renames properly:
constraints, sequences, indexes and grants all follow. So **Restore as
schema** in the import dialog restores under the dump's own name and renames
afterwards. Same result, by a route the server actually supports.

If the name you asked for is already taken the rename is *not* forced - two
schemas cannot be folded together by renaming one onto the other. The import
says so and points at **Merge schema** (5.9), which exists for exactly that.

#### What a restore can leave broken

`pg_restore` reports problems and carries on. A schema-scoped import whose
foreign key points at a table in another schema is the common case: the
constraint fails, the table is restored **without it**, and `pg_restore`
exits saying only *"errors ignored on restore"*. Nothing in the database then
records that the constraint should have been there.

There is no list of invalid objects to consult either. **PostgreSQL has no
INVALID state the way Oracle does** - `pg_proc` has no validity column, and a
function referencing a missing table is created without complaint and fails
only when someone calls it. Views cannot go invalid at all, because the
server refuses to drop what a view depends on.

What the catalogue *does* record, and what the import checks for you when it
finishes:

| Reported | Where it comes from |
|---|---|
| constraints added `NOT VALID` | `pg_constraint.convalidated` - existing rows were never checked |
| invalid indexes | `pg_index.indisvalid` - a `CREATE INDEX CONCURRENTLY` that did not finish |
| disabled triggers | `pg_trigger.tgenabled` - left off by an interrupted `--disable-triggers` restore |
| foreign keys pointing out of the schema | the ones a schema-only restore cannot carry |

Each finding comes with the statement that fixes it. When there is nothing to
report it says so - silence there is a result, not a missing answer. The
exact failed statements stay in the job log, reachable from **Background
jobs**.

#### A new database is the default target {#restore-into-a-new-database}

The restore dialog opens on **(a new database)**, named after the one you
opened it from with the date appended; the database is created empty and
the dump restored into it, so nothing that exists is written over - under
any name you give it, from any format. The database you took the dump from
can stay where it is, in the same cluster.

That holds for a plain SQL file taken with `--create` too, which it did not
always. Such a file creates its database and switches into it
(`CREATE DATABASE shop`, `\connect shop`), and psql obeys it rather than the
database it was started in: replayed as it was, it restored nothing into the
new database, and a file taken with `--create --clean` first dropped the
database it came from and built it again from the file - measured, the rows
added after the dump were gone. The dialog now reads the file before the
restore and says when it does this; the restore then runs from a copy with
the `DROP DATABASE`, `CREATE DATABASE` and `\connect` lines left out and the
statements about the database itself (its settings, comment and grants)
pointed at the one you chose. Your file is not changed. A custom, tar or
directory archive never had the problem: `pg_restore` is not given
`--create`, so it writes into the database it is pointed at.

A file of a whole cluster (`pg_dumpall`) holds several databases, roles and
tablespaces and cannot be poured into one database; it is refused with that
reason. Restore it with psql against a cluster that has none of its
databases, or take a dump of the one database you need.

#### Restoring into a database that exists {#restore-into-an-existing-database}

An existing database can still be chosen from the list; the confirmation
then says what it holds - how many tables and views, how big - before
anything is written. Know what happens there, because it is not a
replacement:

- **Without --clean** nothing in the database is replaced. An object that
  exists stays as it is - creating it fails, and the failure is reported -
  and the rows in the dump are **added** to the tables that exist. A table
  with a primary or unique key that the dump's rows collide with loads none
  of them; a table without one gets them a second time. Measured: a table
  without a key went from 3 rows to 5.
- **With --clean** the objects in the dump are dropped first and created
  again, so their current rows are replaced by the dump's. Objects the dump
  does not contain are left alone.

#### Tablespaces on restore {#restore-tablespaces}

A dump remembers the tablespace of each object that was not in its
database's default, and puts it back there. The dialog lists the
tablespaces the file names and which of them the server does not have.
A missing one is not passed over quietly: `pg_restore` would put those
objects in the default tablespace with an error for each, and psql would
stop at the first. The restore is refused with the names instead, and one
click ticks **--no-tablespaces**.

**Tablespace of the new database** creates the new database in the
tablespace you pick. With **--no-tablespaces** every object the dump brings
goes there too - each database in its own tablespace, whatever the source
used. Without it, objects keep the tablespaces the dump names and only the
rest go to the new database's.

It can also be **a new tablespace**: give it a name and a directory on the database server - which has to exist already, be empty and belong to the user PostgreSQL runs as - and the tablespace is created first, then the database in it. Creating a tablespace needs a superuser.

#### Restoring into a different name

`pg_restore` cannot rename a schema on the way in - the names in the dump are
the names you get. DBApguin does not pretend otherwise. Two routes work:

1. **Restore, then rename.** Restore the dump as it is, then right-click the
   schema → **Rename schema** (5.8). Constraints, sequences and grants follow
   the rename by themselves.
2. **Restore into a staging schema, then merge.** When the name you want is
   already in use and holds data you intend to keep, restore into a staging
   schema and use **Merge schema** (5.9) to fold it into the live one, with a
   decision per object and a choice about what happens to colliding rows.

Route 2 is the one to use on a live database: the merge runs in a single
transaction, so a collision you did not expect changes nothing.

#### The format is read from the file, not its name

Leave **Format** on *Detect from the file contents* and DBApguin looks inside
the file rather than at its extension:

| Signature | Format | Restored with |
|---|---|---|
| starts with `PGDMP` | custom | `pg_restore` |
| `ustar` at offset 257 | tar | `pg_restore` |
| a directory | directory | `pg_restore` |
| anything else | plain SQL | `psql` |

This matters because the extension is not evidence. A plain SQL dump saved as
`backup.dump` is common, and it used to be handed to `pg_restore`, which
answered *"input file appears to be a text format dump. Please use psql."* - 
correct, and no help to someone who just wanted their file restored. Both
directions now work: SQL named `.dump`, and a custom archive named `.sql`.

The dropdown still lets you name the format outright when you want to
override the detection.

#### Restore options

Restore has its own set: `--clean`, `--if-exists` (only with `--clean`,
which `pg_restore` insists on), `--no-owner`, `--no-privileges`,
`--single-transaction`, `--exit-on-error`, `--disable-triggers` (data-only
restores), `--data-only`, `--schema-only`, `--no-comments`,
`--no-publications`, `--no-subscriptions`, `--no-security-labels`,
`--enable-row-security`, *Run as role*, parallel jobs, and - to narrow what
comes out of a dump - **Only these schemas**, **Only tables**, **Exclude
schemas** and **Section**.

`--single-transaction` and parallel jobs are mutually exclusive; picking both
is refused rather than passed on.

**A plain SQL file is replayed by `psql` exactly as it was written**: owners,
grants and everything else come with it, and it stops at the first error.
`psql` can run it as one transaction and as another role, and those two are
applied; every other option above needs a custom, tar or directory dump, so
they are switched off when the file is plain SQL, and asking for one is
refused by name rather than ignored.

Things about restoring a *part* of a dump worth knowing, because
`pg_restore` is not symmetric with `pg_dump`:

- **Names are matched exactly as they are in the dump**, upper and lower case
  included; `sales.orders` and `orders` both work, and a name in double quotes
  may contain a dot or a comma. One that matches nothing in the archive is
  refused before anything runs.
- **Only tables restores the whole table.** `pg_restore --table` on its own
  brings the table definition and its rows and nothing else: no primary or
  foreign keys, no indexes, defaults, triggers, owned sequences or
  partitions - a table that looks right in the tree and behaves like none of
  that exists. By default the restore takes what `pg_dump --table` would have
  dumped: the partitions at every level, keys, indexes, defaults, triggers,
  policies, owned and identity sequences with their values, comments and
  grants - and the function a trigger calls or the sequence a default draws
  from when this database lacks them. *the table and its rows only* is still
  there, and says what it leaves out.
- **There is no "exclude tables" on restore.** `pg_restore` has no
  `--exclude-table`. Exclude them when taking the dump, or name the ones you
  want. Asking for it is refused with that explanation instead of an
  "unrecognized option" from the tool.
- **Excluding a schema still creates it, empty.** `--exclude-schema` skips
  the schema's contents, not the `CREATE SCHEMA`. Drop the empty schema
  afterwards if you do not want it.

#### Before and after a restore {#before-and-after-a-restore}

When the file is chosen, the dialog reads what the restore will ask of the
target and says what is missing there:

- **roles** the dump makes owners of objects or grants rights to - with
  `--no-owner` / `--no-privileges` they stop mattering (not for a plain SQL
  file, which carries them regardless);
- **extensions** the dump creates and this server cannot install;
- **schemas** the dump's objects live in but that it does not create - a dump
  taken with *Only tables* has no `CREATE SCHEMA` - offered to be created
  first, empty;
- a different **encoding** (into UTF8 nothing is lost; into anything else a
  character it has no code for stops the load of its table) or **locale**
  (text sorts and indexes in this database's order).

With `--exit-on-error` or `--single-transaction`, and always for a plain SQL
file, a missing role, extension or schema refuses the restore before it
starts: it would stop at the first of them, half done.

When the restore has finished, the target is compared with the dump itself:
the rows of every table (counted in the dump's data, not estimated), each
sequence's value, every owner and every grant the dump would have set. What
differs is listed. `pg_restore`'s exit code only says whether it logged
errors; this says whether the database holds what the dump held.

#### Cloning a schema without its constraints

**Constraints → leave out constraints and indexes**, in Backup and in Export
schema, dumps the tables and the rows and stops there: no primary keys, no
unique or foreign keys, no indexes.

It is one `pg_dump` run - `--section=pre-data --section=data`, which the tool
accepts together. pre-data holds the table definitions, data holds the rows,
and everything this leaves out lives in post-data. Verified against a live
server: the resulting file has no `ADD CONSTRAINT` and no `CREATE INDEX` in it
at all.

Two reasons to want it. A bulk load runs faster without the checks and the
index maintenance, and a replication tool doing its initial sync into the
target usually refuses to start while foreign keys are enforced there. Both end
the same way: the constraint manager writes the script that puts them back, and
you run it when the load or the sync is done.

The tick overrides the **Section** picker rather than combining with it - two
controls for the same flag would otherwise produce a dump neither of them
describes.

#### Timeouts on a long dump or restore

A dump or restore that dies part way through - half an hour in, nothing wrong
with the file - is usually the server ending it, not the tool failing.

Four settings can do that:

- `statement_timeout` - one statement ran too long.
- `lock_timeout` - waiting for a lock took too long.
- `idle_in_transaction_session_timeout` - a transaction sat idle too long.
- `transaction_timeout` (PostgreSQL 17 and later) - a transaction has been open
  too long, idle or not.

`pg_dump`, `pg_restore`, and a dump loaded through `psql`, all clear the first
three for themselves: on a live server their log shows `SET statement_timeout =
0`, `SET lock_timeout = 0` and `SET idle_in_transaction_session_timeout = 0`
before any work starts. They do not clear `transaction_timeout` - and that is
the one that ends long work, because `pg_dump` holds a single REPEATABLE READ
transaction for the whole dump and `pg_restore --single-transaction` holds one
for the whole load. A server with `transaction_timeout = 30min` stops both at
thirty minutes however healthy they are.

A plain `.sql` file that `pg_dump` did not write is the other gap: it carries no
`SET` header, so all four apply to it.

**Server timeouts -> clear them for this run**, in Backup, Export schema and
Restore, clears whichever of them the target has. It goes through `PGOPTIONS`,
which libpq applies at connect and which overrides an `ALTER DATABASE ... SET`
or `ALTER ROLE ... SET`. `transaction_timeout` is sent only to PostgreSQL 17 and
later: an unknown parameter in `PGOPTIONS` is not ignored, the connection is
refused with `FATAL: unrecognized configuration parameter`, and that would turn
"your restore may be cut short" into "your restore cannot start".

The line under the checkbox reports what the target actually has set. Unticking
leaves the server's settings alone.

If a run already died this way, the message names the setting: `canceling
statement due to statement timeout`, `terminating connection due to
idle-in-transaction timeout`, or `canceling statement due to transaction
timeout`.

The line under the checkbox reports not only the value but **where it was set** - 
`statement_timeout = 30min (set on database sales)`, or `(set on role app_user)`.
A value with no source named came from the server's own configuration:
`postgresql.conf`, or the parameter group on Aurora and RDS. That is the
difference between knowing something is cutting the run short and knowing what
to change.

**On Aurora and RDS specifically.** `transaction_timeout` does not exist before
PostgreSQL 17, so on Aurora 15 or 16 a run that dies at a round number of
minutes is one of these instead:

- a `statement_timeout` on the parameter group, the database or the role - the
  line under the checkbox names which;
- **RDS Proxy**, if the connection goes through one. Its `IdleClientTimeout`
  defaults to 1800 seconds, which is exactly thirty minutes, and it counts a
  client that is waiting on a long statement as idle;
- a **NAT gateway or firewall** between you and the server. A long single
  statement - the index build at the end of a restore, a COPY of one huge table
 - sends nothing in either direction while it runs, so the device drops the
  flow and the statement finishes into a socket nobody is holding. AWS NAT
  gateways time an idle flow out after 350 seconds. The app sets TCP keepalives
  on the connection for exactly this reason whenever the host is not a Unix
  socket, which keeps the flow alive without any traffic of its own.

Nothing in the app limits how long a dump or restore may take. Both run as
detached processes with their state on disk, so they survive the app being
closed or restarted, and neither the HTTP request nor the window is waiting on
them.

#### Watching a dump or a restore while it runs

`pg_dump` and `pg_restore` narrate what they are doing, and the dialog shows
all of it as it arrives - from the first line to the last, not the last line
only. Scroll up to read something and it stops following; untick *follow* to
keep it still. **Copy** puts the whole log on the clipboard.

The job runs detached, so closing the dialog does not stop it, and the log is
still there afterwards under *Background jobs*.

#### "errors ignored on restore" is not a failed restore

`pg_restore` exits with an error status when it ignored **any** error, and
prints only a count. The commonest one by far is a `CREATE` meeting an object
the target already has - restoring a schema into a database that already has
that schema fails on `CREATE SCHEMA` and restores everything underneath it
perfectly.

The dialog therefore lists the errors themselves and says which of them are an
object that was already there. When they all are, it says so plainly: nothing
was skipped because of them. A real error is still reported as a real error.

#### A dump from a newer PostgreSQL

A plain `.sql` dump opens with a short run of `SET` statements, and a newer
`pg_dump` writes settings an older server has never heard of - 
`transaction_timeout` arrived in PostgreSQL 17. Replayed into 16, that line
stops the restore on the spot and nothing is restored.

The app reads the header first and refuses before starting, naming the
setting, where the file came from and what this server is. Dumps travel
forwards, not back: take the dump again with a `pg_dump` matching the target,
or restore into a server at least as new as the one it came from.

#### Which client it runs

A `pg_dump`, `pg_restore` or `psql` from a different major version than the
server is not interchangeable, and the mismatch hurts in opposite directions:

- `pg_restore` and `psql` **talk to** the server. They open the session with a
  block of `SET` statements chosen by their own version, so a client newer
  than the server sends settings it has never heard of - `pg_restore` 17 sends
  `transaction_timeout`, which nothing below 17 has, and every restore then
  reports an ignored error and exits 1 on a load that worked.
- `pg_dump` and `pg_dumpall` **read** the server. They query catalogs they
  only know their own version of, so one older than the server refuses to
  start at all: *"aborting because of server version mismatch"*.

So the app asks the server its version and picks a copy that suits it, from
the versioned directories distributions keep side by side
(`/usr/lib/postgresql/*/bin`, `/usr/pgsql-*/bin`, Homebrew, the EDB
installers) as well as the bundled one and whatever is on `$PATH`. For a
restore there is a second constraint - a client older than the `pg_dump` that
wrote the archive cannot read it - and that is not settled from a table: the
candidate is asked with `pg_restore -l`, which reads the index and restores
nothing, and the next major up is tried if it refuses.

Reading an archive back has no server in it at all, only the file, so the
backup verification takes the newest copy installed. That one is a bug fix as
much as a refinement: a perfectly good archive read by a `pg_restore` older
than the `pg_dump` that wrote it answers *"unsupported version in file
header"*, which looks exactly like corruption.

**Settings -> Tools** reports the path each tool resolves to, and lists the
other majors installed beside it, because the path shown there is the fallback
rather than the whole answer. A path you set yourself is absolute and turns
the choosing off entirely - which is the escape hatch when a server needs a
client the app would not have picked.

#### Choosing files

Everywhere the app reads or writes a file - an export, a dump, the file to
restore, importing connections, a certificate - it opens your desktop's own
file dialog, with its places bar, its recent folders and its overwrite prompt.
On a system with no such dialog available, the app falls back to its own
folder browser and nothing is lost.

The dialog is opened where the files are. `pg_dump` runs on the machine the
app runs on and writes where that machine can write, so the chooser browses
that filesystem.

Saving always asks for a file name. The one place that asks for a folder
instead is the destination of a backup, because a folder is what `pg_dump`
writes into.

### 5.10.5 Partitioned tables {#partitioned-tables}

Partitioned parents appear under the **partitioned tables** category in the schema tree. Their leaf partitions are ordinary tables, so a hundred `events_2026_NN` rows would otherwise fill the **tables** category with nothing tying them together - so the tree groups them under their root instead: the parent has an **N partitions · M rows** sub-node that expands to the leaves. When the leaf partitions live in a different schema from their root (a common layout), that schema shows a group of its own labelled with the root's name and `root in <schema>`. An **expand N leaf partitions** line opens every group in the schema at once (the choice is remembered). Nothing is hidden - the count on each category is what you can actually reach by expanding.

**Creating one**: the *+ Table* dialog has a **Partitioning** section - pick RANGE, LIST or HASH and a key (a column, a `(col1, col2)` list, or an expression like `(date_trunc('month', at AT TIME ZONE 'UTC'))`). The parent is created empty; you add partitions to it afterwards. Any `PRIMARY KEY` or `UNIQUE` on a partitioned table must contain every partition-key column - PostgreSQL enforces this and the error is clear if it is missed.

**Managing them**: right-click a partitioned table and choose **Partitions…**. The dialog opens on the strategy and key and lists every partition: its bounds, estimated rows, size, tablespace when it has its own, and a `detach pending` pill if a concurrent detach was interrupted (see below). The default partition is marked and sorted last; a partition that is itself partitioned opens from the same row.

- **Create partition** - a new empty partition attached in one statement. Fill in the bounds the strategy asks for.
- **Attach existing table** - brings in a table you already loaded. The statement note tells you whether PostgreSQL will **full-scan the child** (it shows the row count and size) to validate the bound, or whether it can be metadata-only. It is metadata-only when the child already has a `CHECK` constraint that proves every row fits; if it does not, add a matching `CHECK`, run `ALTER TABLE ... VALIDATE CONSTRAINT` (that takes only `SHARE UPDATE EXCLUSIVE`, not the full lock), then attach.
- **Detach** - the table stops being a partition and keeps its rows. **DETACH CONCURRENTLY** is ticked by default on PostgreSQL 14+: it does not take an `ACCESS EXCLUSIVE` lock on the parent, so queries on the other partitions keep running. It runs outside a transaction and cannot be rolled back, and if it is interrupted the partition is left **detach pending** - the row then shows **Finalize detach**, which completes it. PostgreSQL refuses `DETACH CONCURRENTLY` while the table has a DEFAULT partition, and the checkbox is disabled with that reason; on PostgreSQL 13 and older it is disabled because the feature does not exist, and a plain detach takes the exclusive lock for its (usually brief) duration.
- **Drop** - deletes the partition and every row in it.

Nothing runs on the click. The dialog shows the statement first, with a line saying exactly what it locks, for how long, and what happens if it is interrupted, and runs it only when you press *Run it*.

**Show DDL** for a partitioned parent emits `CREATE TABLE ... PARTITION BY ...`; for a leaf partition it emits `CREATE TABLE ... PARTITION OF <parent> FOR VALUES ...` with no column list and only the constraints and indexes defined on that partition itself.

**Converting between plain and partitioned**: right-click a table and choose **Convert to partitioned (recipe)…**, or right-click a partitioned table and choose **Flatten to a plain table (recipe)…**. Neither option runs anything. The move is a rebuild (a new table, a copy of every row, then a swap of names), so the dialog shows a SQL script to copy, read and run yourself. The script is a single transaction. It holds writes off from its first statement to its COMMIT and compares the row counts before it commits, so if anything fails, the table is left as it was. It carries over:

- columns and defaults, generated columns, and CHECK and NOT NULL constraints;
- identity and serial counters, so the next id continues where the old table stopped;
- keys and indexes under their original names, and triggers;
- the owner, privileges, comments and row-level security policies;
- foreign keys in both directions, views that read the table, and publications.

A conversion has two TODOs to fill in:

- the partition key, which every PRIMARY KEY and UNIQUE constraint must include;
- the partitions. A DEFAULT partition is already in the script, so no row is rejected during the copy.

Anything that cannot come across is listed at the top under *Read first*. Examples are NO INHERIT checks on a partitioned table, EXCLUDE constraints before PostgreSQL 17, materialized views, and functions that use the table's row type. The old table stays as `<name>_old` until you drop it.

### 5.10.6 Cloning a schema

Right-click a schema, then **Transfer → Clone schema…**. It copies the schema
into a new one on the same database: tables, partitioned tables, views,
materialised views, sequences, functions, procedures, aggregates, types (enum,
domain, composite, range), collations, conversions, foreign tables, indexes,
constraints, triggers, rules, row level security and its policies, extended
statistics, and optionally the rows. **Merge schema** moves the same breadth
of objects; the difference is that a merge moves them out of the source and a
clone leaves the source untouched and generates a second copy.

Four kinds cannot be generated and are **named in the result instead of being
passed over in silence**: text search configurations, dictionaries, parsers and
templates, and operators, operator classes and operator families. Each is
declared with a form that carries more than the catalogue hands back in one
piece - a text search configuration is a parser plus a mapping per token type,
an operator class is a list of operators and support functions - and half a
`CREATE` is worse than none. The dialog says `18 of 20 object(s) copied, 2 to
create by hand` and lists exactly which two, rather than reporting twenty.

Foreign keys are rebuilt pointing at the copy rather than back at the
original - a clone whose keys still reference the source is not a copy, it is
a second door into the first schema's data.

With *copy the rows too*, the order matters and is handled for you: the
objects are created without their foreign keys, the rows go in, and the keys
are added afterwards. Copying rows into tables that already enforce their
references fails on whichever table happens to go first. Sequences are moved
to where the originals had got to, so the first insert into the copy does not
collide with a row that came across with the data. An `IDENTITY` column
carries its sequence settings too - `START WITH`, `INCREMENT BY`, `MINVALUE`,
`MAXVALUE`, `CACHE`, `CYCLE` - so the copy is not silently reset to the
defaults.

The rows are taken **exactly as they are**, whatever is in them. Anything that
judges or rewrites a row while it is written waits until the rows are in:

- a constraint the source has `NOT VALID` (a `CHECK`, a foreign key, a domain's
  `CHECK`, PostgreSQL 18's `NOT NULL ... NOT VALID`) is added `NOT VALID`
  after the rows, so rows the source never checked are not checked in the copy
  either, and new rows are;
- triggers and rules are created after the rows, so none of them refuses,
  rewrites or adds to a copied row;
- row level security and its policies are switched on after the rows;
- an index a failed `CREATE INDEX CONCURRENTLY` left invalid is not copied
  (it enforces nothing in the source, and built again on the same rows it
  fails again) - as `pg_dump` leaves it out.

Row level security does not refuse a read, it shortens it. If its policies
apply to the role you are connected as (you are not the table's owner, or the
table has `FORCE ROW LEVEL SECURITY`, and you are neither a superuser nor a role
with `BYPASSRLS`), or you may not read one of its columns, the clone names the
table and the reason **before** anything runs and copies nothing, rather than
copying the rows you happen to see.

**Table by table** shows, for every table, the rows copied, what was added
`NOT VALID` as in the source, what was created after the rows and what was not
copied. If a row cannot be copied after all, the message names the table, the
column, the constraint, the failing row and the rows in the source by their
primary key. **Clone table**, **Copy table** and the *copy* choice of
**Merge schema** take the rows the same way.

**Show the script** prints the whole thing before anything runs. The clone
itself runs as one transaction: it either arrives whole or not at all. It
runs as a server-side job, so a clone of a large schema with its rows - which
is minutes of work - is not lost if you close the dialog: **Background jobs**
shows it while it runs and keeps the full result to reopen afterwards.

When it finishes the dialog lists the checks the copy was verified against
rather than a bare "Done": the new schema is read back before the transaction
commits, and every table, sequence, constraint and row count is compared with
the original.

Cloning onto a schema that already exists is refused. That is a merge, and
[Merging two schemas](#59-merging-two-schemas) is the tool for it.

### 5.11 Changing the owner of everything in a schema

Right-click a schema → **Change owner of all objects…**

A schema restored under the wrong role, or a person leaving, leaves every
object in it owned by someone who should not own it. Doing that one object at
a time is fine for three objects and unusable for three hundred.

Pick the new owner. The dialog lists every object in the schema that has an
owner of its own, grouped the way the schema tree groups them: tables,
partitioned tables with their partitions nested underneath at every level,
views, materialised views, foreign tables, sequences, functions, procedures,
aggregates, types, domains, range types, operators, operator classes and
families, collations, conversions, text search configurations and
dictionaries, statistics objects, and the schema itself, ticked. Under the
list are the exact `ALTER ... OWNER TO` statements for what is ticked; nothing
runs until you press the button. Objects the new owner already owns are shown
and left alone.

Partitions are listed one by one because PostgreSQL gives every partition an
owner of its own: changing the owner of a partitioned table does not change
its partitions. Ticking or unticking a partitioned table does the same to
everything under it.

Foreign servers and publications used by the schema's tables belong to the
whole database, not to the schema. They are listed at the end, unticked; tick
them if they should move too.

Some objects are not changed on their own, and the dialog says which:

- a sequence that belongs to a table column (serial or identity) moves with
  its table, and PostgreSQL refuses to change it by itself. When its table
  already has the new owner and the sequence does not, nothing would move it:
  a serial sequence is then listed with the statements that untie it, hand it
  over and tie it to its column again; an identity sequence cannot be untied,
  so it is shown above the list with the two statements that move it through
  its table, and what they cost;
- objects that belong to an extension keep the extension's owner;
- indexes always have their table's owner.

Applying runs in **one transaction**. A half-reassigned schema is harder to
reason about than one that was never touched, and there is no way to tell
from the outside where it stopped. The schema itself is changed first, so the
new owner can create in it by the time its objects are handed over. The result
shows the statements that give every object its previous owner back, in case
the change has to be undone.

Afterwards the old owner keeps in this schema only what PostgreSQL's own
REASSIGN OWNED would leave too: the default privileges that role set for the
schema.

**One object.** Right-click an object → **Change owner…** (for a database, in
the database menu). The dialog shows the current owner and the exact
statement before anything runs. For a partitioned table it also says how many
partitions it has, counting every level, and who owns them, and offers **Also
change the owner of its N partitions**. The box is off by default, which is
what PostgreSQL does: the table changes owner and every partition keeps its
own. Left off, the result says how many partitions kept their owner.

### 5.12 Comparing schemas across servers

Right-click a schema → **Compare with another schema**. The compare dialog
opens with that schema chosen on side A; choose one on side B too, and it
compares the two schemas. Leave both on *whole database* and it compares the
databases.

Side B has its own **server** picker listing your saved connections, so the
comparison can run between two different clusters - dev against production is
the usual reason. Choosing a server reloads that server's schema list, since
the schemas on one cluster say nothing about the schemas on another.

Comparing `sales` on one cluster with `sales` on another is a normal thing to
want, so the "same schema" guard only applies when both sides are on the same
connection.

The result is the object-level difference plus a **sync script** that would
make B match A. Nothing runs by itself: see [Running it on B](#running-it-on-b).

#### A and B side by side {#a-and-b-side-by-side}

Every row that differs - an object in a schema, a schema's owner and grants,
an extension, a property, a role, a cast, a partitioned table's partitions -
opens in place when you click it (or press its arrow button: Tab reaches it,
Enter or Space opens it). Two panes show A's definition on the left and B's
on the right, with line numbers:

- a line only A has is red, a line only B has is green;
- a line both have but not the same stands across from its counterpart, with
  the words that differ marked;
- a long stretch that did not change folds into *N unchanged lines*; click it
  to see those lines;
- **Copy A** and **Copy B** copy that side's text;
- under the two panes, the statements the sync script writes for that object -
  the same ones **Generate sync script** writes, with the options set now.

On a narrow window the two panes stack, A above B. A definition of thousands
of lines opens at once: only the changes and three lines either side are
drawn until you open a fold.

**Database compare** - the Databases menu, or right-click the database in the
sidebar → Manage → Compare with another database - works the same way for
whole databases, and takes a list of schemas to restrict the comparison. That
matters on a large database: the per-schema object walk is the expensive half,
so comparing one schema out of forty should not cost the same as comparing all
forty.

The database compare also covers what belongs to no schema: publications,
subscriptions, foreign servers, event triggers, and the casts somebody created
with CREATE CAST (the built-in ones and those an extension brings are left
out). A differing cast has a tick box like a schema or a role, and the sync
script drops and creates it again - a cast holds no data and cannot be
altered.

The database and schema lists of both sides can be sorted in their drop-down: A-Z, Z-A, or largest first. Database sizes are exact; schema sizes are an estimate from the planner statistics (the last VACUUM or ANALYZE), which costs nothing and waits on no lock. The entry for the whole database stays at the top in every order.

#### Running it on B {#running-it-on-b}

Every row the script writes statements for has a tick box, ticked; a row it
writes nothing for (a row count, a note) has one that cannot be set. In the database compare a schema's tick ticks its objects. Under
the result, three actions work on the ticked rows' statements, in the order the
script has them:

- **Dry run...** runs them on B in one transaction that is always rolled back.
  Every statement runs under a savepoint of its own and is reported on its own:
  ran, failed with PostgreSQL's error, or skipped. Nothing is kept, but while it
  runs it takes the same locks an apply would.
- **Apply to B...** runs them on B in one transaction: all of them, or on the
  first error none, with the failing statement named and the ones after it
  marked as not run. Each statement waits at most 5 seconds for its lock.
- **Export changeset...** saves them as a Flyway migration
  (`V<timestamp>__<name>.sql`) or a Liquibase changelog (formatted SQL or YAML,
  one changeset per statement, with its id and author), where you choose.

Before a dry run or an apply a review shows, read from B itself:

- every statement with the lock it takes and on which object - ACCESS
  EXCLUSIVE stops every read and write of that table until the transaction
  ends, SHARE lets reads go on - and whether it drops, detaches or rewrites
  something;
- the statements that cannot run inside a transaction block (anything
  CONCURRENTLY, ALTER TYPE ... ADD VALUE before PostgreSQL 12, ALTER SYSTEM):
  they are skipped and listed, to be run on their own;
- the copy: before an apply, the definitions of every object on B the
  statements change or drop are saved as SQL that puts them back; the report
  links to it and Background jobs keeps it. What it has no way to write - a
  role, a cast, a statistics object, a collation, text search objects,
  operators - is named, with the reason. The rows are not in the copy: a table
  or a column the apply drops comes back empty from it;
- what your login may not do on B, asked of B: an object owned by a role you
  are not a member of, a schema you may not create in, a role you cannot hand
  an object to.

When anything is dropped, detached or rewritten, **Apply to B** waits until you
type the name of B's schema (two schemas) or database (two databases).

After an apply the two are compared again at once; the report stays above the
new result, which is what is left. Apply runs only on the B the result was
compared against: point the dialog somewhere else and it asks you to compare
again. Every dry run and apply goes into the audit log with its statements; an
apply that rolled back is recorded as failed, with the statement that failed.

#### Handing the comparison over

**PDF report** turns what is on screen into a document: which server it came
from and when, the counts, the table of what differs, and the sync script. It
is built from the same answer the screen shows, so the two cannot disagree.

The file is written where you choose, by the application rather than by the
browser, and an existing report is never replaced without being asked.

One limit worth knowing: the report uses the fonts every PDF reader already
has, which cover German and Spanish completely but not the Croatian č, ć and
đ. Those are transliterated - "sifra", not a blank - so a name stays readable
even when an accent is lost.

#### What the four statuses mean

The whole result of a comparison is one word per row, so it is worth being
precise. Both dialogs compare **structure, not data** - definitions, properties
and grants. No row is read from either side, unless you ask for the current
state as well (below), and nothing is written to either until you apply
the ticked rows to B.

| | |
|---|---|
| **same** | Present on both sides and identical. A schema whose objects all match is hidden unless you untick *differences only*. |
| **changed** | Present on both sides under the same name, but the two definitions are not the same text: a column added, a type widened, a different function body, a property set to another value. Click the row to see A and B side by side (see [A and B side by side](#a-and-b-side-by-side)). |
| **only in A** | Exists in the source and not in the target. A sync script creates it on B. |
| **only in B** | Exists in the target and not in the source. A sync script **drops** it, because making B match A means removing what A does not have. |

Direction is not symmetric: A is the source, B is the target, and every script
these dialogs produce changes B to look like A.

#### Restricting what is compared

**What to compare** holds the filters, and they apply to the comparison and to
the script it generates alike - a comparison that ignores indexes must not
produce a script that touches them.

- **Object types**: tables, views, materialised views, sequences, functions.
  Turning one off makes the comparison blind to it.
- **Inside a table**: columns, indexes, foreign keys, primary keys, unique
  constraints, check constraints. This is how you get a comparison that looks
  only at foreign keys, or one that ignores index differences - useful when
  two environments are deliberately indexed differently.
- **By name**: an include list and an exclude list, both accepting `*`.
  `audit_*` in the exclude box keeps a compare focused on what you care about.
- Six presets do the common combinations in one click: everything, tables
  only, foreign keys only, sequences only, structure without indexes, views
  and functions.
- **Allow DROP statements** decides whether the sync script may remove what
  exists only in B. Untick it and the script only creates and alters, which
  is the safe default when B is not yours to prune.
- **Move tables to A's tablespace and access method** is off by default:
  moving a table rewrites all of it under an ACCESS EXCLUSIVE lock. Off, the
  difference is still reported and the script says why it leaves it.
- **Count a different column order as a difference** is off by default.
  PostgreSQL cannot move a column, so no statement can make B's columns stand
  in A's order. Off, a table whose columns are the same in another order
  compares as *same* and carries a note - *columns in another order* - which
  opens to both orders, B's as it will be after the script (a column the
  script adds lands at the end). Ticked, the order counts, and the script says
  the table has to be rebuilt by hand to match it.

#### Partitions, inheritance and storage {#partitions-inheritance-and-storage}

A partition is compared with its place in the tree: its parent and its bound
(`FOR VALUES ...`), and a partitioned table with its partition key. The sync
script moves partitions with **ATTACH PARTITION** and **DETACH PARTITION**,
never by dropping and creating them again, because a partition holds rows:

- a bound that changed: detached, then attached again with A's bound;
- a partition A has attached and B keeps as a table of its own: attached;
  the reverse: detached;
- a partition under another parent: detached from B's parent, attached to A's;
- a partition only B has, under a parent both sides have: detached and kept as
  a table of its own. It is dropped afterwards only when **Allow DROP
  statements** is ticked.

Every DETACH runs first, in a section of its own, so a bound written further
down cannot overlap a partition that is still attached. The script's header
lists the locks: DETACH takes ACCESS EXCLUSIVE on the parent and the
partition; ATTACH takes SHARE UPDATE EXCLUSIVE on the parent (ACCESS
EXCLUSIVE before PostgreSQL 12) and reads the partition through to prove its
rows fit the bound. On PostgreSQL 14 and later each DETACH is followed by its
`... CONCURRENTLY` form as a comment: it takes a weaker lock, but it cannot
run inside a transaction block or when the parent has a DEFAULT partition, so
run it on its own if you use it.

A partition key cannot be changed in place. When A and B partition a table
differently the script says so and writes nothing executable for it, and what
A puts under that table waits, as a comment, until the key is settled by hand.

**INHERITS.** A table that inherits from another (classic inheritance, not a
partition) is compared with its parents. The script writes `ALTER TABLE ...
INHERIT` or `NO INHERIT`, and a new child is created with only its own
columns, after its parent.

**Tablespace and access method** are part of a table's definition: a table in
another tablespace, or stored with another access method, compares as
changed. A new table is created where A has it; an existing one is moved only
with the switch above.

**A column whose type changes under views.** PostgreSQL refuses `ALTER COLUMN
... TYPE` while a view, a materialised view or a rule reads the column. The
script drops each of them on B right before the change - the views built on
them too, in any schema, the outermost first - and creates them again right
after, as they are on B: definition and options, a materialised view's
indexes and rows (REFRESH), a view's column defaults and INSTEAD OF triggers,
rules, owner, comments, grants and column grants. Each DROP and CREATE holds
an ACCESS EXCLUSIVE lock on its object until the transaction ends, so queries
on them wait; the review marks each such DROP as one that comes back. What
cannot be put back from here - a trigger whose WHEN reads the column, a
policy, a generated column, a function body - is named in the script as work
to do by hand before it runs.

**A column that is dropped under views.** PostgreSQL refuses `DROP COLUMN`
while a view, a materialised view or a rule reads the column. The script drops
each of them on B right before the column - the views built on them too, the
outermost first - and after it each becomes what the rest of the script makes
of it: one that is only on B stays dropped; one that A defines differently,
without the column, is created as it is in A, with B's owner, grants and
comments (and the column comments, column grants, rules, triggers and indexes
that do not name a lost column), and its own change is not written a second
time further down; any other comes back as it is on B. The review marks each
DROP that comes back. A view that would come back still reading the column - one
in another schema, one the script leaves alone, one built on a view that
changes or goes - cannot come back, and neither can a trigger whose WHEN reads
the column, a policy or a function body: then the script says which one has
to change first and does not drop that column (it writes the DROP as a
comment), because a DROP that fails rolls the whole run back. Change that
object and compare again.

#### Owner and grants {#owner-and-grants}

The **owner** and **grants** switches decide two things together: whether a
difference in who owns an object, or who may use it, counts - and whether the
sync script writes the ALTER ... OWNER TO, GRANT and REVOKE that close it.
Both are on by default; untick one and that kind of difference is neither
reported nor scripted. Grants take in column grants, the grants on an identity
column's sequence, what an owner revoked from itself, and the schema's own
grants and the default privileges set in it, which are shown as one row named
*(this schema)*. A grant is read as PostgreSQL applies it: an object nobody
ever granted on and one whose grants were made and taken back again compare
the same. When the script names
a role that does not exist on the target, its header says which roles.

#### Comparing the current state too {#comparing-the-current-state-too}

A comparison is by definition: two schemas are the same when every object in
them would be created the same way. **Also compare the current state** adds
what a running database has and a definition does not. Every switch is off by
default, because two copies of one schema always differ in state.

| | |
|---|---|
| **sequence current values** | Where each sequence is, and whether it has been called yet. The script sets B's with setval(). |
| **whether a materialised view is populated** | The script runs REFRESH MATERIALIZED VIEW, or REFRESH ... WITH NO DATA when A's is empty. |
| **row counts** | Tables, partitions and materialised views. The estimate comes from the planner's statistics and counts as different when the two are 5% or more apart; **exact row counts** run count(*) on every table, read every row, and compare exactly. |
| **sizes** | Total size, indexes and TOAST included. Different when the two are at least 1 MB apart and either 10% or 64 MB apart. |
| **index validity and usage** | An index left invalid by a CREATE INDEX CONCURRENTLY that failed, and how often each index has been used: different when one side has used it and the other never has, or when the two counts are more than 1000 apart. |

Row counts, sizes and index usage are reported and never scripted - no
statement a sync script could run would make them equal - and the script ends
with the list of objects that differ in them. In the database compare, the
partitions of a partitioned table that differ are named, not only counted.

#### Comparing what a role may do, on two sides {#comparing-what-a-role-may-do-on-two-sides}

**Compare users** answers "can this role do the same thing here as it does
there", and it asks the question in that order: which roles, from where, then
how far down to look.

Roles are cluster-wide in PostgreSQL - one `pg_authid` per cluster - so the
role list is what the two *servers* have, and a role missing on one side is
reported rather than skipped. Privileges are the opposite: they live in each
database's own catalogue, which is why the database pickers matter and why the
comparison is per database. Naming schemas narrows it further; leaving that
empty compares every schema the connection can see.

**PUBLIC** - every role, present and future - has its own row in the roles
list. It has no attributes to compare, only grants: **compare privileges** on
that row shows what is granted to PUBLIC on each side, and the sync script
writes the GRANT ... TO PUBLIC and REVOKE ... FROM PUBLIC that close it.

Seven levels are read, and the first one is the level everything else depends
on:

| | |
|---|---|
| **database** | `CONNECT`, `CREATE`, `TEMPORARY` - whether the role can open the door at all |
| **schema** | `USAGE`, `CREATE` |
| **table** | and views, materialised views, partitioned and foreign tables |
| **column** | a role can hold `SELECT` on three columns and nothing on the table, which every table-level check reads as "no access" |
| **sequence** | `USAGE`, `SELECT`, `UPDATE` |
| **function** | `EXECUTE` |
| **default** | what *new* objects will carry - a role can be correctly granted on everything that exists today and lose access to everything created tomorrow |

Grants are read straight from the catalogue ACLs, not through
`has_table_privilege`. That function answers "can this role do X" including
everything inherited through membership, and inherited privilege cannot be
granted or revoked on this role. Only a direct grant can be acted on, so only
direct grants are compared; membership itself is compared as a role attribute.

**The sync script** makes one side match the other, in whichever direction you
choose. Neither side is automatically the right one - production may be the one
that drifted, or the one that is correct - so both directions are offered and
neither is the default in a way that hides the choice.

Grants the target holds and the source does not are written as comments unless
you tick *also revoke*. Adding a grant is recoverable; taking one away in the
middle of someone's working day is not.

Applying from the database comparison shows the statements first. **Apply**
there used to issue the grants on the click and report a count afterwards;
it now opens the same dialog every generated script gets - copy, save, open in
the SQL editor - with Apply beside them, and runs only what you were shown. If
the privileges change between looking and applying, the apply stops and hands
you the new list rather than running something else.

Nothing is executed. Read the script, save it with **Save script…**, and run it
when you decide to.

### 5.13 Cloning a database

Right-click a database → **Clone database (same server)**.

This is one statement - `CREATE DATABASE "copy" TEMPLATE "original"` - and
PostgreSQL copies the database at the file level rather than dumping and
restoring it. That is why it is fast: a 25 MB database clones in well under a
second, and the time depends on size, not on the number of objects.

The copy is fully independent from the moment it exists. Writing to one does
not affect the other; they share no storage.

#### The cost, and why the dialog insists

A database cannot serve as a `TEMPLATE` while anything else is connected to
it, so before copying, DBApguin runs:

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'source'
```

**Every open connection to the source is killed**, including other people's,
and any transaction in flight is aborted. This is a PostgreSQL requirement,
not a choice DBApguin makes - which is why the dialog asks you to tick a box
rather than just clicking through.

**Your own connection goes with them.** DBApguin's session on the source
database is terminated like any other, and the connection pool quietly opens
a new one for the next request - so the app carries on and nothing looks
wrong. What does not survive is anything attached to the old session: an open
transaction in the SQL editor is rolled back, temporary tables are gone,
session-level `SET`s are back to their defaults, advisory locks are released,
and a query running at that moment is aborted. Commit or finish what you are
doing before you clone.

#### What comes across, and what does not

Everything *inside* the database is copied. The properties attached *to* the
database are not - they belong to the cluster's catalogue rather than to the
files being copied:

| Copied | Not copied |
|---|---|
| tables, rows, indexes, constraints | the database **owner** - the clone belongs to whoever cloned it |
| sequences, at their current values | `GRANT CONNECT` / `TEMPORARY` on the database |
| views, materialized views, functions, types | per-database settings (`ALTER DATABASE … SET work_mem`, `search_path`, …) |
| extensions | the comment on the database |
| table and column privileges | roles and users (they are cluster-wide, and already exist) |
| comments on objects, RLS policies, large objects | |

That right-hand column is the part that surprises people. A production
database cloned for testing arrives owned by you, with no `CONNECT` grants
for the application's role, and running on the server's default `work_mem`
and `search_path` rather than the ones the original had. The data is
identical; the environment around it is not.

To restore those four, on the clone:

```sql
ALTER DATABASE copy OWNER TO original_owner;
GRANT CONNECT, TEMPORARY ON DATABASE copy TO app_role;
ALTER DATABASE copy SET work_mem = '64MB';
COMMENT ON DATABASE copy IS '…';
```

#### When to use it

- before a migration or a risky change, so there is an exact state to return to
- for a test or development copy on the same server
- **not** for moving a database to another server - `TEMPLATE` works only
  within one cluster; use backup and restore for that
- **not** on a database people are using right now - they will be disconnected
- a clone occupies as much disk as the original; nothing is shared

### 5.14 Renaming a database {#renaming-a-database}

Right-click a database -> **Rename database**.

This is one statement, `ALTER DATABASE "old" RENAME TO "new"`, and it changes
**only the catalogue entry**. Nothing is copied or moved, so it is
instantaneous regardless of the database's size. The owner, every `GRANT`, the
comment, and all per-database settings (`ALTER DATABASE ... SET ...`) stay
attached and unchanged - the name is the only thing that is different
afterwards.

#### It cannot run while anyone is connected

PostgreSQL refuses `ALTER DATABASE ... RENAME` while any session is connected
to that database - including DBApguin's own connection pool if you have been
browsing it. Without help, the statement fails with:

```
database "shop" is being accessed by other users
DETAIL: There is 1 other session using the database.
```

So the dialog checks first. If the database is idle, the rename just runs. If
it is not, the dialog shows **how many connections there are and which users
hold them**, and offers to terminate them - the same
`pg_terminate_backend(...)` that Clone uses. This is destructive: an open
transaction on that database is rolled back, temporary tables and
session-level `SET`s are gone, and a query running at that moment is aborted.
You have to tick the box.

DBApguin's own pooled connections are terminated along with the rest; the pool
simply opens fresh ones against the new name on the next request. The
connection performing the rename is on the `postgres` database, never on the
target, so the rename can never sever itself.

#### Your saved connection and open tabs

If you rename the database this window is **currently connected to**, DBApguin
updates the saved connection entry to the new name, switches the active
connection across, and reloads the object tree. Open SQL and schema tabs that
were bound to the old name are marked stale with a badge until you refresh
them, the same as any database switch. A saved connection for a *different*
window that also pointed at the old name is not touched - update it by hand.

#### What it does not fix

The rename is a catalogue change and nothing outside the catalogue knows about
it: an application with the old database name in its connection string, a
`~/.pg_service.conf` entry, a backup script, a `dblink` or `postgres_fdw`
server definition, or SQL that hard-codes the name. `current_database()` in a
function returns the new name from the moment the rename commits. Rename a
production database only when you can update everything that reaches it in the
same maintenance window.

`postgres`, `template0` and `template1` cannot be renamed - the dialog refuses
them - because initdb and a great deal of tooling assume those names exist.

### 5.15 Extensions, and what "move" does

**Database -> Manage extensions** lists what the server has available and what this
database has installed, with the installed version beside the default one so an
`ALTER EXTENSION ... UPDATE` candidate is visible at a glance.

An extension is a bundle: functions, operators, types, sometimes tables, all installed
into one schema. **Move** runs

    ALTER EXTENSION "name" SET SCHEMA "target"

and relocates the whole bundle in one statement.

Why you would:

- **Keep `public` yours.** `CREATE EXTENSION pg_trgm` with no schema drops a few dozen
  functions into the first schema in `search_path`, which is normally `public`, mixed in
  with your own tables.
- **Security.** Anyone who can create objects in a schema can define a function with the
  same name and argument types as an extension function. If that schema comes earlier in
  `search_path`, their version is the one that runs. Extensions in a dedicated `ext`
  schema that only the owner can write to remove that possibility. This is the reason
  the usual recipe is `CREATE SCHEMA ext; CREATE EXTENSION pg_trgm SCHEMA ext;`

What it will not do:

- **Non-relocatable extensions cannot move.** The extension itself declares this
  (`pg_extension.extrelocatable`), usually because its own code refers to its schema by
  name. PostGIS is the well-known case. DBApguin marks those `(non-relocatable)` and
  offers no Move button rather than letting the server refuse.
- **It does not update your code.** Anything that names the old schema explicitly - 
  a view body, a function, an application query - breaks. Unqualified calls keep working
  only if the new schema is in `search_path`:
  `ALTER DATABASE app SET search_path = public, ext;`
- **It is not a version upgrade.** Use Update for that.

Moving requires ownership of the extension, which in practice means superuser. The
statement is fast - it rewrites catalogue entries, not data - but it takes a lock on the
extension's objects, so do it when nothing is running against them.

### 5.16 Task list

An alarm, an advisor recommendation and a health check finding all say "somebody
should do this" and then disappear - the alarm when the condition flickers, the
advisor when the panel closes, the notification when it is dismissed. A cluster
with transaction ids 1.2 billion old and nine tables past the line is a morning's work,
not a toast.

**View -> Task list**, with a count of what is open on the menu itself; the count
turns red when one of them is critical.

Things reach the list from three places:

- **a notification** - every alarm card carries *Add to task list* beside *Open*;
- **the critical alarm window** - each alarm in it has its own button, because
  acknowledging says "I have seen this", which is not the same as "this is
  handled";
- **any graded advisor row** - hover it and a small list icon appears.

You can also simply type one. A typed task is yours, and never merges with
anything or gets marked stale.

Each task records the server, database and object it came from, so one read a
week later still says what it was about. **Passwords are never stored** - the
connection is kept as host, port, database and user, for identification only.

Adding the same finding twice is the normal case, because the alarm is still
firing; the existing task is refreshed rather than duplicated.

**A task whose condition has cleared says so.** When the alarm behind it stops
firing on the server it came from, the task is marked *condition no longer
present* and its edge turns green. It stays in the list - removing somebody's
task because a reading changed would be worse - it just stops claiming to be
current. The claim is deliberately narrow: only for a task raised from an alarm,
on the connection in front of you, matched on host and port.

**Export checklist** writes the list as a plain text checklist with `[ ]` boxes,
the object, the server and your own note - something to print, paste into a
ticket, or carry to a machine that does not have this application on it.

### 5.17 Foreign servers and user mappings {#foreign-servers-and-user-mappings}

**Database -> Manage foreign servers** lists every foreign server in this
database with the wrapper behind it, its owner, its options, and the user
mappings hanging off it.

Creating a foreign server was a wizard; everything after that was the SQL
editor. A remote host moves, a port changes, a stored remote password expires,
a server has to be handed to another owner - all of it ordinary, none of it
reachable from the place that made the server, and the options the wizard had
asked for were not shown back anywhere. This is that half.

#### What a foreign server is made of

Three objects, and all three have to agree before a single row crosses:

| object | what it carries | who it belongs to |
|---|---|---|
| the wrapper | `postgres_fdw`, `mysql_fdw`, `file_fdw`, ... | the extension |
| the server | where the far end is: host, port, dbname | the database |
| the user mapping | who you are over there: remote user and password | one local role |

The mapping is the part people forget. PostgreSQL looks it up **by the role
that actually connected**, so a server with a mapping only for `postgres` is
invisible to everybody else: their queries fail on *"user mapping not found"*
however correct the server is. That is also why a mapping is offered for any
login role on the cluster rather than only for yourself and `PUBLIC` - one
local role to one remote login is the normal shape, and the reason remote
credentials are per role at all.

A mapping **for a NOLOGIN group role** is accepted by the server and then never
used, because such a role never connects. Those roles, and the built-in `pg_*`
ones, are left out of the picker on purpose.

#### Editing a server

**Edit** shows the options the server carries now, each as an editable field,
with room for keys it does not have yet. Only what you change is sent.

**Clearing an option's value removes the key.** PostgreSQL has no empty-string
option, and `OPTIONS (SET k '')` is a different statement from the key being
gone, so the empty field is how you say "drop this".

Which of `ADD`, `SET` and `DROP` a key needs depends on whether the server
already carries it, and that is read from the catalogue in the same transaction
as the `ALTER` rather than guessed from what the screen last showed. A change
of four options comes out as one statement:

    ALTER SERVER "loopback" OPTIONS (SET host '10.0.0.9', SET port '5433',
                                     DROP dbname, ADD connect_timeout '7')

**Version** and **Owner** are separate `ALTER SERVER` statements on purpose.
PostgreSQL accepts them in one, but then a single bad option key takes the
owner change down with it and the error names neither. All of them go in one
transaction, so a half-applied change is not a state you can end up in.

#### Passwords

A mapping's remote password is never shown. PostgreSQL does not hand a stored
password back to a client, and a server option spelled `password` is masked
here for the same reason - including in the SQL echoed back and in the audit
log, which is a file people export and send on. The password field in the
mapping editor is therefore blank and travels only if you type into it.

#### Dropping

A server that still has foreign tables on it refuses a plain `DROP`, so the
dialog offers `CASCADE` explicitly and counts the user mappings that go with
it. Cascading takes every foreign table on that server too - those are local
catalogue entries, so nothing on the far end is touched, but the queries and
views that read through them break.

**Drop user mapping** is the narrower knife: that one role loses its
credentials for the server and its queries fail until a mapping exists again,
while every other role keeps working.

## 6. Common workflows

### 6.1 Performance triage on a slow DB

1. Open the connection.
2. Run `db-health` first - one screen overview.
3. If cache hit is low: `db-cache-advisor` for per-object detail.
4. If seq scans are dominating: `db-index-advisor` for index candidates.
5. If queries are slow: `db-stat-statements` for the top-25 with per-query flags. Sort by total_exec_time or pick the queries with WARN flags.
6. If bloat is suspect: `db-bloat-breakdown` for HEAP+TOAST+index per table. Use `db-pgstattuple` for accurate (slow) scan.

### 6.2 Locking issue

1. `db-health` -> BLOCKING category, drill down for the blocker/waiter pairs.
2. Or open Sessions view -> List, sort by blocking_pids non-empty.
3. Right-click the blocker session and choose Terminate when ready.

### 6.3 Replication lag

1. `db-replication` for replica state + write/flush/replay lag in time and bytes.
2. `db-repl-slots` for WAL retention per slot. Inactive slots holding GBs of WAL are the classic disk-fill cause.
3. For logical: `db-logical-replication` (subscriber side) shows DISABLED / NOT STARTED subs and apply lag >=16 MB.
4. `db-replica-identity` if subscribers complain about missing identity.

### 6.4 Security audit

1. `db-roles-list` for cluster-wide security findings.
2. `db-role-perms` (pick the audited role) for per-target deep dive.
3. `db-ownership` for object ownership findings.
4. Export the audit log from Settings -> View menu -> Audit log.

### 6.5 Schema migration safety

1. Compare two databases via Server view -> Schema compare.
2. Review the generated sync SQL.
3. Apply selected statements.

### 6.5.5 Would this index actually help?

`db-index-advisor` reports which tables are being scanned sequentially. The
next question is always whether an index on *this* column would fix it, and
the only honest way to answer it used to be to build one - a long write lock
for an answer that is often "the planner would not have used it anyway".

`EXPLAIN` cannot answer it either. The planner costs the indexes that exist,
so without the index there is no plan with it, and `EXPLAIN ANALYZE` needs it
to exist for the same reason.

Under the advisor's table there is a panel that answers it without building
anything. Pick the table, write the index you are considering, paste the query
you care about, and press **Compare plans**. It reports:

- whether the planner would use the index at all - the common answer is no,
  and it is worth knowing before you build a 4 GB index;
- the estimated cost with and without it, and the difference;
- roughly how much space the index would occupy.

It deliberately does not report a time. Nothing is executed, so there is no
measurement to give, and presenting an estimate as one would be worse than
giving neither.

This needs the `hypopg` extension, which creates the index in the planner's
head only - nothing is written and no lock is taken, so it is safe on
production. It is available on RDS and Aurora; self-hosted it ships as its own
package. If the extension is not installed the panel says so, and whether this
server could have it.

### 6.6 Oracle-style workload analysis (AWR + ASH + trace) {#oracle-style-workload-analysis-awr-ash-trace}

#### When it runs: the capture plan {#capture-plan}

The AWR, ASH and Session trace tabs start the same way: one **When** row under the header.

- **Snapshot now** (AWR only): one snapshot, with an optional comment shown beside it in every list.
- **For a while**: starts now and stops by itself. Quick lengths sit beside a wheel of days, hours, minutes and seconds, so any length within the tool's limits can be chosen: AWR 10 minutes to 92 days, ASH 1 minute to 31 days, the trace 10 seconds to 30 minutes.
- **Scheduled**: from a date and time to another, chosen with the date-and-time picker on this computer's clock, optionally **every day**. A trace is scheduled with a start and a length, up to a day ahead.
- **Until stopped**: what the scheduler and the sampler did before (not for the trace).

**Every** is how often the tool gathers, with its own quick choices and the same wheel: AWR a snapshot every 1 minute to 1 day, ASH a sample every 1 second to 1 hour, the trace a look every 0.1 second to 1 minute. **Keep** is how many days the stored snapshots or days are kept. AWR's **also sample the sessions every 5 s** starts the ASH sampler for the same window; for a plan of up to an hour it takes the workload report instead (a snapshot at each end with the sessions sampled in between), which used to be the Capture view.

Above the button one sentence says what will happen, how many snapshots or samples that is, what it takes on disk and on the server, and what opens at the end; it warns when a choice is heavy (an AWR snapshot more often than every 5 minutes). While a plan runs, the panel folds into one bar: what runs, since and until when, a progress line, the time left, **Change** and **Stop**; a scheduled plan says when it starts and offers **Start now instead**. When a plan with an end is over, the report of its period opens by itself: for AWR its first snapshot against its last, for ASH the report of its window.

In AWR the **Comment** beside the When row is there for every mode: it goes on the one snapshot of *Snapshot now* and on every snapshot a plan takes. The date-and-time picker of *Scheduled* has quick times ahead (in 1 h, tonight 02:00, tomorrow 08:00, Monday 00:00) and wheels for the hour and the minute; the chevron at the end of the When row folds the panel to one line on a small screen.

**Stop** ends a plan early and works like its own end: what was collected stays, AWR takes one last snapshot at that moment (with the plan's comment), and the report of the period opens - the run's first snapshot against that last one, or for ASH the time it sampled. A scheduled plan that has not started yet is only cancelled.

Plans run while DBApguin is open: nothing is left on the server and nothing runs by itself. A plan in the middle of its window when the app closes carries on for the rest of it the next time the server is opened, a daily plan waits for its next window, and a plan whose window passed while the app was closed is over. A scheduled trace lives only while the app is open.

On a Mac the wheels tick on the trackpad as each value passes, the way a picker on a phone does.

1. **Verify setup once**: run `db-setup-advisor`. If `pg_stat_statements`, `auto_explain`, or `compute_query_id` are MISSING, follow the printed remediation. You only do this once per cluster.

2. **Baseline a slow period**: when the user complains about slowness, run `db-workload-report` for 5 minutes. The Top SQL sections show what was actually consuming time during the window.

3. **Continuous observability**: when you want hours of session-level history (not just one 5-minute snapshot), start the ASH sampler:
   - `db-ash` action=start
   - leave it running in the background (it survives until you click action=stop or close the app)
   - run `db-ash` action=report later to aggregate any UTC date range

4. **Pre / post deploy comparison**: take a baseline snapshot before the deploy, another after the workload settles, then compare them:
   - `db-awr-snapshot` action=take, comment="before deploy v24.07"
   - (deploy)
   - `db-awr-snapshot` action=take, comment="after deploy v24.07"
   - `db-awr-snapshot` action=list (tick both, then **Compare the two chosen**)
   - `db-awr-snapshot` action=compare, snap_id_a=<before>, snap_id_b=<after>

5. **Per-app deep trace**: when one connection-pooled app is slow and you suspect a specific query:
   - `db-trace-session` target_mode=application_name, target_value=`myapp-worker`, duration_seconds=120, category_filter=`select`, query_filter=`reports`
   - In a parallel psql connected as that app's user: `SET auto_explain.log_min_duration = 0; SET auto_explain.log_analyze = on;` then trigger the workload
   - The trace report shows query transitions + wait events for every matching backend; the server log captures the plans

#### Reading an ASH report {#reading-an-ash-report}

`db-ash` action=report turns the samples of the chosen window into six sections. The first two are the report's own reading of the rest; the tables after them are the counts.

**Findings** is what stood out, worst first, each with the statement behind it and the same buttons every statement carries here: Copy, load into the SQL editor, EXPLAIN there, or **Plan** - the plan that actually ran, drawn in place (see [The plan that actually ran](#the-plan-that-actually-ran)). "Show in timeline" lights the session's row up in the next section.

| Finding | Fires when | What to do |
| --- | --- | --- |
| Blocking: session N | one session held others up for 3 samples or more; critical from a minute lost or five sessions held | The text says whether the blocker was idle in a transaction (the application forgot to commit) or running a statement (a long write). "A chain" means the blocker was itself waiting on another. |
| One statement took X% of the active time | a statement has 20% or more of the running-or-waiting samples, and there were other statements to compare it with | The finding says where its time went: mostly locks, mostly disk, or mostly CPU - each points at a different fix. |
| X% of the active time was lock waits | 20% or more of the active samples were `Lock` waits | The blocking pairs section names the sessions. |
| X% waiting on disk | 30% or more were `IO` waits | A working set larger than `shared_buffers`, or one scan flushing the cache; the top queries say which statements. |
| Internal contention (LWLock) | 20% or more were `LWLock` waits | `WALWrite` is the WAL disk, `buffer_content` / `buffer_mapping` are hot pages, `lock_manager` is very many locks per transaction. |
| Session idle in a transaction for up to T | a session sat idle inside a transaction for 5 minutes or more | It keeps its locks and holds back vacuum for the whole server; critical when it also blocked someone. `idle_in_transaction_session_timeout` ends such sessions. |
| Statements were cut | a sampled statement's text reached `track_activity_query_size` | No plan can be made from a cut text; the settings section at the end says what to raise. |

**Timeline** is the window as a grid: one row per session (the busiest forty, the rest summed into one "other" row), one column per slice of time - a second for a short window, a minute or more for a long one, said in the title. Each cell is painted in the colour of what the session was doing most in that slice:

| Colour | Meaning |
| --- | --- |
| green | running on CPU (active, no wait event) |
| red | waiting on a lock held by another session |
| yellow | waiting on disk (`IO` wait events) |
| blue | another wait: `LWLock`, `IPC`, client, timeout |
| purple | idle inside a transaction - not working, but holding its locks |
| empty | not in the samples: idle, or not connected |

A pale cell is one the session was in for only part of the slice. **Click a cell** for that session's statements in that slice, with the wait event, the pids that blocked it (each a button that jumps to their row) and the statement's buttons. **Click a session's label** for everything it did in the window: its colour split and its statements summed. **Click a time** at the top for every session in that column. The pids are from the past: a backend may be gone, or be another session by now, so the timeline offers no cancel or terminate.

**State histogram**, **Wait events** and **Top sessions** are the counts; top sessions now also carry each session's lock, disk and idle-in-transaction shares.

**Top queries** are cards, one per statement, ranked by samples: its share of the active time, a bar with its colour split (where its time went), how many sessions ran it and from which applications, the longest single run seen, its most frequent wait event, and - when `pg_stat_statements` is on the server - its calls, mean and total time since the counters were reset. The buttons are the same as in the findings; a statement cut by `track_activity_query_size` has no Plan button, because the planner would only report the cut.

**Blocking pairs** name both sides: the waiter and the blocker with their applications, what the blocker was doing while it blocked (`idle-txn` is the one to look at), when the pair was first and last seen, the lock the waiter was after, and both statements in full.

**Settings for a better analysis** closes the report (and the status action shows it too, before a single sample is taken): the server settings that decide what the samples can say, each with its current value, the recommended one, whether it takes a reload, a restart or a CREATE EXTENSION, and why. Warnings are settings without which this report is blind to something - `compute_query_id` (no statement identity in the samples), `track_activity_query_size` (cut statements), `pg_stat_statements` (no per-statement counters), `track_io_timing` (disk time invisible), `log_lock_waits` (no record of waits shorter than a sample); the rest are nice to have (`auto_explain`, which is the only way to see the plan that *actually ran*, `log_min_duration_statement`, `log_temp_files`, `track_functions`). **Apply** runs `ALTER SYSTEM SET` and a reload after the usual confirmation; a restart-only setting is marked so and waits for the next restart. Without a superuser the statements are there to copy. It also lists everything the plan that actually ran needs - auto_explain and how it logs, the log collector and destination, the privilege to read the log (a GRANT, applied here when this login may grant it) and pg_store_plans (installing a package is shown as a command to copy) - each with its cost; see [The plan that actually ran](#the-plan-that-actually-ran).

**Before ASH or AWR starts collecting** (ASH start, AWR take and auto-start) the app asks the server for the same list. When something they read is missing, a window comes up first: the extensions (`pg_stat_statements` - created in this database? in `shared_preload_libraries`? - and `auto_explain`), then every setting not in place, each with its current value, the recommended one, whether it takes a reload, a restart or a CREATE EXTENSION, and why. The ones that leave the report blind are ticked; the nice-to-haves are for you to tick. **Enable selected** runs exactly the statements shown under the list (two rows that add to `shared_preload_libraries` are shown as the one statement they end in), reads the server again and says what now waits for a restart - the app never restarts the server. A row it put in place stays in the list, with its result, until **Check again**. **Extensions…** closes the window and opens the extensions manager; **Start anyway** runs with what there is. Each tab asks once; once nothing is missing it no longer comes up by itself, and **Check settings...** beside Start in the ASH and AWR plan shows it at any time, with the nice-to-haves (such as `auto_explain`'s own settings, which exist only after the restart that loads it), its Start starting the plan. Re-running a tab (its circular arrow) reads the tab's view again; it never starts, stops or takes anything. A non-superuser gets the statements to copy. The window can be switched off in Settings (*Check before ASH / AWR start*), or with its own checkbox.

#### Query history search {#query-history-search}

`db-query-history` is a search over `pg_stat_statements`, cross-referenced with the ASH sampler for a clock. Its parameters run BEFORE the check, in the same params dialog every parameterised check uses:

| Field | Does |
| --- | --- |
| Statement contains | Case-insensitive substring match against the normalised statement text (literals stripped - a table or column name is usually the most useful thing to type). Empty matches everything. |
| Run by | Only statements executed by this role. |
| Worst by | Which ranking to sort by - total time (what the database spends its life on), mean time (what is slow per run), worst single run, call count, or rows returned. Total time and mean time answer different questions: a query taking 2 ms a million times outranks one taking a minute twice. |
| Ignore faster than (ms) | Drops rows whose mean time is below this, so a page of fast noise does not bury the one slow shape worth reading. |
| How many | Row cap, 1-200. |
| Include plans | EXPLAIN of the normalised text for the top 5 rows - the plan for the shape, not a record of the plan actually used (see the caveat below). |

**This search runs against the full server-side history every time you open the dialog and click Run check** - not just what happens to be on screen. The result table then gets the same per-column filter box every check result gets (typing in a column header narrows the rows already visible); that filter box only narrows what this search already returned, it does not search the corpus again. To look further back or by a different criterion, change the fields above and run the check again.

Result columns:

| Column | What it is | How to read it |
| --- | --- | --- |
| `user` | Role that ran the statement shape | - |
| `calls` | Executions since the counters were last reset | - |
| `total` / `mean` / `max` | Cumulative / average / worst single execution time | Same reading as `total_ms`/`mean_ms` in the workload report, above. |
| `rows` | Total rows returned or affected across all calls | - |
| `ash_samples` | How many ASH samples caught this statement's `queryid` running | 0 means the ASH sampler was not running while this ran, not that it never ran - `calls` is the count that matters. Needs `db-ash` started first; PG13 does not have `queryid` in ASH at all. |
| `last_seen` | Timestamp of the most recent ASH sample that caught it | Empty for the same reason as `ash_samples` = 0. |
| `query` | The normalised statement text, truncated | Literals are stripped to placeholders (`$1`, `$2`, ...) - a row here covers every execution that differed only in its values. |

Counters reset with `pg_stat_statements_reset()` or a crash; a clean server restart keeps them when `pg_stat_statements.save` is on (PostgreSQL's own default). A statement from before the last reset is simply not there - "nothing matched" can mean that as easily as it can mean the pattern was wrong.

#### Reading an AWR / workload report {#reading-an-awr-report}

Every number in these reports is a **delta over the window** - the count between the two snapshots, not a rate and not a lifetime total. A saved report keeps this glossary attached (the PDF export includes it), so a report reopened weeks later still explains itself.

**Window summary**

| Metric | What it is | How to read it |
| --- | --- | --- |
| `window_seconds` / `window_hours` | Wall-clock time between the two snapshots | Everything else is "per this much time". A 30-second window and a 6-hour window are not comparable. |
| `txn_commit` / `txn_rollback` | Transactions committed / rolled back in the window | Counts, not rates. Divide by `window_seconds` for TPS. |
| `rollback_pct` | `txn_rollback / (txn_commit + txn_rollback)` | Above ~10% with real traffic means the application is throwing errors or hitting deadlocks - look at the errors watcher and the deadlock count. A handful of rollbacks on a quiet window is noise. |
| `cache_hit_pct` | `blks_hit / (blks_hit + blks_read)` - fraction of block reads served from **shared_buffers**, not from disk | This is the PostgreSQL buffer cache, not the OS page cache, so a "low" number can still be fast if the OS cached it. Below ~95% with meaningful traffic usually means `shared_buffers` is too small for the working set, or one big sequential scan flushed the cache during the window. |
| `deadlocks` | Deadlocks detected in the window | Any value above 0 is an application lock-ordering bug. PostgreSQL resolved it by killing one transaction; find the pair in the server log. |
| `temp_files` / `temp_bytes` | Files (and total size) PostgreSQL spilled to disk for sorts and hashes | Non-zero means a query's sort or hash did not fit in `work_mem`. Cross-reference Top SQL by temp writes. |
| `new_sessions` / `sessions_per_s` | Sessions started in the window (`pg_stat_database.sessions`, PostgreSQL 14+), and per second | Many short sessions is connection churn: an application without a connection pool, or a pool that opens and closes connections. Before PostgreSQL 14 the summary says it cannot be measured. |
| `mean_session` | Session time accrued in the window divided by the sessions started | Long-lived (pooled) connections add their time too, so this makes the short sessions look longer than they are: a small number is a strong sign of churn, a large one proves nothing. |

**Top SQL** (four rankings of the same rows: by total time, by call count, by disk reads, by temp writes)

| Column | What it is | How to read it |
| --- | --- | --- |
| `calls` | Executions of this normalised statement in the window | - |
| `total_ms` | Sum of execution time across all `calls` (the `total_exec_time` delta) | This is the ranking that matters most: the query with the highest `total_ms` consumed the most server time in the window, regardless of whether it is individually slow. |
| `mean_ms` | `total_ms / calls` - average time for one execution | **A large `mean_ms`** (say > 100 ms) means each run is slow: a missing index, a bad plan, lock waits, or a genuinely large query - EXPLAIN (ANALYZE, BUFFERS) it. **A small `mean_ms` with a large `total_ms`** means the query is cheap per call but runs so often it dominates - the fix is to call it less (cache the result, batch the work) or shave the per-call cost, not to tune the plan. |
| `rows` | Total rows returned or affected across all calls | `rows / calls` is the average result size; a report query pulling 100k rows per call is a different problem from an OLTP query pulling 3. |
| `shared_read` | 8 KB blocks read **from disk** (not from cache) across all calls | High means this query's data is not in `shared_buffers` - a cold cache, or a working set larger than the cache. `shared_read x 8192` is the byte volume. |
| `temp_written` | 8 KB blocks written to **temp files** across all calls | Non-zero means the sort or hash spilled. Raise `work_mem` for the session running it, or fix the plan so it sorts fewer rows. `temp_written x 8192` is the byte volume. |
| `qid` | The `queryid` (normalised-statement fingerprint) | Same value across snapshots and in `pg_stat_statements`; use it to track one statement over time. |

**Top tables by DML**

| Column | What it is | How to read it |
| --- | --- | --- |
| `ins` / `upd` / `del` | Rows inserted / updated / deleted in the window | Drives autovacuum. A table with heavy `upd` + `del` and no recent autovacuum is a bloat candidate - see [Vacuum, bloat and table statistics](#vacuum-bloat-and-table-statistics). |
| `hot_upd_%` | HOT updates as a fraction of all updates | HOT updates do not touch indexes and do not need an index cleanup. **Low** (< ~50%) on a heavily-updated table means updates change indexed columns, or pages have no free room - lower the table's `fillfactor` so updates stay on-page. |
| `seq_scan` / `idx_scan` | Sequential vs index scans started on the table in the window | A large `seq_scan` delta on a large table is a missing-index signal - confirm with the index advisor. |

**Background / checkpoint activity**

| Column | What it is | How to read it |
| --- | --- | --- |
| `ckpt_timed` / `ckpt_req` | Checkpoints triggered by schedule vs by WAL volume | Mostly `req` means checkpoints are being forced by write volume before `checkpoint_timeout` - raise `max_wal_size` so they spread out. |
| `ckpt_write_ms` / `ckpt_sync_ms` | Time spent writing / fsyncing dirty buffers during checkpoints | A large `sync_ms` means the storage cannot absorb the checkpoint write burst. |
| `buffers_backend` | Dirty buffers that **backends** had to write themselves | Non-trivial values mean the background writer and checkpointer fell behind and query backends paid the write cost inline. |

**Findings** is the report's own reading of the window, worst first, drawn like the ASH findings: each with the statement behind it and its buttons. Beyond the thresholds above (deadlocks, rollback share, cache hit) it names a statement that took 30% or more of all execution time, statements averaging over 500 ms per call, statements that spilled 64 MB or more to temp files (`work_mem`), checkpoints forced by WAL volume more often than by the clock (`max_wal_size`), backends writing a fifth or more of the dirty buffers themselves (`bgwriter_*`), and a heavily updated table with fewer than half its updates HOT (`fillfactor`). Every statement threshold here needs the window to hold at least ten seconds of execution time in total, so a quiet test database does not read like a server in trouble. Four findings are judged on their own volume instead - a pure N+1 workload, a fast lookup run hundreds of thousands of times with nothing slow beside it, never reaches ten seconds and used to read "Nothing stood out in this window":

- **1+N pattern**: one statement (B) runs k times for every run of another (A), in the same database as the same user, with k at least 5 and B run at least 1,000 times. The ratio alone could be chance, so it needs one piece of evidence as well: A returned (within 5%) as many rows as B ran - once per row; or the ratio held within 10% in at least three intervals between snapshots while A's own rate changed by half or more (an interval in which A started or stopped does not count, and under a flat load any two statements keep their ratio, which proves nothing); or B's calls are a whole multiple of A's to within two calls, A having run at least 500 times. A rows match is one number for the whole window, so when A also ran at least three times in each of three intervals between snapshots, B's calls must follow A's rows in each of them as well (within 10%, beside the rows of two of A's calls whose lookups ran after the snapshot); a match the intervals contradict is not evidence. When several statements could be A, the one with the strongest evidence is named: rows that held in every interval, then a steady ratio, then rows the intervals could not measure, then a whole multiple. Statements that read only the server's catalogs and statistics views (pg_stat_statements, pg_stat_activity, pg_class, information_schema and the like) are never A or B: they are a monitor's polls, this report's own counts among them, and their row totals follow the size of the server, not a loop. A stored comparison reads the snapshots taken between the two you picked (up to 24 of them); a workload report counts the statements about a dozen times while it records. Amber when B runs 200 times a second or takes 10% of the execution time, blue below that. Statements run inside a function are left out - they loop in the server, with no round trip to save. The finding says what pg_stat_statements cannot: every call also costs the application a round trip, usually more than the query itself.
- **A sub-millisecond statement run very often**: at least 10,000 calls under 1 ms each AND either 10% of all execution time (the share) or 200 calls a second (the rate), whichever comes first; a statement already named in a 1+N finding is not repeated.
- **Connection churn** (PostgreSQL 14+): amber at 20 new sessions a second, blue at one new session for every two transactions, both with at least 200 new sessions in the window. It names the databases, and adds the sessions the client dropped without closing, that ended in a fatal error or that were terminated.
- **Optimistic locking**: an UPDATE or DELETE whose WHERE compares a version column with a parameter and wrote no row on some of its calls - those lost the race to another writer and were retried. Amber when at least 5% of at least 100 calls, or 1,000 calls, wrote nothing; see [Connections, sessions and load](#connections-sessions-and-load).

**Top SQL to look at** are cards for the statements the findings named, ranked by their share of all execution time, each with the reasons as tags and the same buttons: Copy, load into the SQL editor, EXPLAIN there, or **Plan** - the plan drawn in place. It shows the plan that actually ran, from `auto_explain` in the server log or from `pg_store_plans`, and the plan the server would choose *now* only when neither has it, labelled so - see [The plan that actually ran](#the-plan-that-actually-ran).

**Settings for a better analysis** closes a compare, a take and a live capture: the same table the ASH report ends with - see [Reading an ASH report](#reading-an-ash-report). It is read from the server you are connected to, so each cluster gets its own advice for its own version: `compute_query_id` and `track_wal_io_timing` are offered from PostgreSQL 14, `pg_stat_statements.track_planning` from 13, and nothing is proposed that the server's version does not have.

**What only the workload report has**, from its own samples of pg_stat_activity (the whole server, every few seconds):

- **Active sessions over time** is a chart with one point per sample: all active sessions, and of them how many were running, waiting on a lock, waiting on disk or waiting on something else, beside the sessions idle inside a transaction (they hold their locks). The colours are the ASH timeline's, so a lock is red in both. A long window is drawn as lines without dots; the export writes every sample as a row, and the PDF averages them into at most 60 rows, each with its time span.
- **Wait events** show where sampled backends were waiting: `Lock` (blocked on another transaction), `IO` (waiting on disk), `LWLock` (internal contention), `Client` (waiting for the client - usually just idle connections, not a problem). A window dominated by `Client` waits is a healthy idle system.
- **Blocking pairs observed** lists who waited for whom, in how many samples, with both statements; **Longest open transaction seen** is the oldest transaction any sample caught.
- Their findings join the others: sessions that waited for another session's lock (the statement holding it is shown, the waiting one beside it); a fifth or more of the active time spent waiting on locks AND at least 5 such samples; sessions idle inside a transaction in a fifth or more of the busy samples AND at least 10 of them; a transaction open for a minute or more.

#### Reading a period comparison {#reading-a-period-comparison}

A period comparison (`db-awr-snapshot` action=compare-periods, the AWR tab's **Compare** view) answers "what changed between then and now" rather than "what happened in this window". Every number is still a delta over its own window; the report sets the two windows side by side.

- **Periods compared**: one row per metric - window length, transactions committed and per second, rollback %, cache hit %, blocks read from disk, temp bytes written, deadlocks, checkpoints timed and requested, buffers written by backends, WAL bytes, the execution time of all statements and how many statements ran. **Change** is period 2 minus period 1, drawn with an arrow up or down, and **change %** is relative to period 1 (for a percentage the change is in points). Red is a change for the worse, green one for the better; a metric where a rise is neither (more transactions is more work, not a worse server) is not coloured. When the comparison is about one database, the last rows are the whole server's.
- Counts grow with the window: when the two periods differ in length, compare the per-second rates and the percentages - a finding says so.
- **Findings** name the numbers of both periods: a statement whose time per call rose by half or more and by at least 10 ms, when that cost period 2 at least a second; a statement that did not run in period 1 and took a fifth or more of period 2's execution time (and at least a second); the cache hit down two points or more; temp files at least doubled and 64 MB or more; checkpoints forced by WAL volume more often than by the clock in period 2 only; the rollback share up five points or more with at least 50 transactions; deadlocks that appeared. A top statement that became at least twice as fast per call, saving at least a second, gets a green line. A restart, a reset or an eviction inside either period is said first, with the period it happened in.
- **Statements: period 1 vs period 2** lists the 30 statements whose total time changed the most, each with both periods' calls, total and mean time, rows and disk reads, its status - *both*, *only period 1* (gone) or *only period 2* (new) - and the buttons Top SQL has (copy, the SQL editor, EXPLAIN, **Plan**).
- **Tables by DML: period 1 vs period 2** lists the 20 tables of the connected database whose inserts, updates and deletes changed the most, with the sequential and index scans of each period.

#### The plan that actually ran {#the-plan-that-actually-ran}

**Plan** on a statement in the ASH report, in the AWR and workload reports' findings and **Top SQL to look at**, and in a period comparison shows the plan the statement *actually ran with* whenever the server kept it, and says plainly when it did not.

**Why EXPLAIN now is not enough.** EXPLAIN made afterwards plans the statement again, today: with today's table statistics, today's settings and indexes, and - for a statement with `$1` parameters - a generic plan instead of the one the real values got. The plan that made the statement slow in the window may be a different one, and EXPLAIN cannot tell you so.

**Two sources keep the real plan:**

- **auto_explain** (a contrib module that ships with PostgreSQL) writes the plan of every statement slower than `auto_explain.log_min_duration` to the server log, with the actual row counts when `auto_explain.log_analyze` is on. DBApguin reads it back through the same database connection, only when you open a plan, and only a bounded part of the log: the log files that overlap the report's window, read from their end, at most 32 MB and 8 seconds per look. A logged plan is matched to the statement by its query identifier where the log has one, and otherwise by the statement's text with its constants set aside.
- **pg_store_plans** (third-party, BSD licence) keeps every plan of every statement in shared memory with calls, total time and the first and last call of each plan, under the same query id `pg_stat_statements` uses. Nothing is read from a log.

**What the Plan view shows.** Three buttons on top - **auto_explain (server log)**, **pg_store_plans** and **EXPLAIN now** - each with the number of plans it found; it opens on the first that has one, in that order. A badge says which kind of plan you are looking at: *The plan that ran* in green, or *The plan the server would choose now - not the one that ran* in yellow, with one line on what to enable. When a statement ran with more than one plan in the window (the plan changed), the plans are listed newest first with when each was last logged (on the server's clock, as the report's From and To are), its duration, how many executions were logged with it and their shortest, mean and longest time; click one to draw it. Under the tree: whether the plan was matched by query id or by text, whether it carries actual rows and times, the parameter values when the server logged them, and how much of the log was read. A source with nothing to show says why (not loaded, no log collector, no privilege to read the log, nothing slower than the threshold in the window, a managed platform). EXPLAIN now still works everywhere and is only run when you choose it or when nothing better exists.

**Setting it up, step by step.** *Settings for a better analysis* at the end of every ASH, AWR, workload and trace report, and the window before ASH, AWR or a trace starts, list every piece with its current value, the recommended one and how it is put in place. What DBApguin can do itself has **Apply** (or is ticked in the window) with the usual confirmation and an audit-log entry; what needs the operating system shows the command to copy.

1. **Load auto_explain** - in DBApguin: Apply adds `auto_explain` to `shared_preload_libraries`. In a terminal: restart the server (the commands for apt.postgresql.org, PGDG RPM, `pg_ctl` and a container are shown under the table; a restart ends every session).
2. **Write plans** - in DBApguin: `auto_explain.log_min_duration` (for example `1s`), `auto_explain.log_format = json` (the view draws JSON as a tree), and on PostgreSQL 16+ `auto_explain.log_verbose = on` (the query identifier inside the plan). These take a reload, not a restart.
3. **Write the log where SQL can read it** - in DBApguin: `logging_collector = on` (restart) and `log_destination` with `jsonlog` added on PostgreSQL 15+, `csvlog` before. stderr is read too, as well as its `log_line_prefix` allows.
4. **Statement identity** - in DBApguin: `compute_query_id = on` (PostgreSQL 14+). Optional and recommended: `track_io_timing = on`.
5. **The privilege to read the log** - a superuser reads it as it is. For a monitoring login the least that works is membership in `pg_monitor` (lists the log directory) and `EXECUTE` on `pg_read_file(text, bigint, bigint, boolean)`, which reads only inside the data directory and the log directory. DBApguin shows the two GRANTs and offers Apply when the connected login may grant them (asked of the server, not guessed from a role name); otherwise a superuser runs them. A login that may grant them - a superuser, typically - also gets **Grant to a login...**: it lists the logins that cannot read the log yet (not superusers, which need nothing), shows the two statements for the one you pick and runs them, recorded in the audit log. `pg_read_server_files` or superuser would read every file the server's operating-system user can - far more than a plan needs.
6. **Or pg_store_plans** - in a terminal: install the package (`dnf install pg_store_plans_NN` from the PGDG repository on RHEL, Rocky and Fedora; on Debian and Ubuntu there is no apt package, so build it with `make USE_PGXS=1` and `make USE_PGXS=1 install`). In DBApguin: Apply adds it to `shared_preload_libraries`, a restart loads it, then **Install…** runs `CREATE EXTENSION pg_store_plans` in each database you want to read it in.

**What exists in which version** (checked against the PostgreSQL documentation of each version and against servers 9.4 to 19):

| Name | PostgreSQL | Notes |
| --- | --- | --- |
| `auto_explain` in `shared_preload_libraries` | all | restart to load |
| `auto_explain.log_min_duration`, `log_analyze`, `log_timing`, `log_buffers`, `log_verbose`, `log_format`, `log_triggers`, `log_nested_statements` | all | reload; superuser to change |
| `auto_explain.sample_rate` | 9.6+ | |
| `auto_explain.log_level`, `auto_explain.log_settings` | 12+ | |
| `auto_explain.log_wal` | 13+ | |
| `auto_explain.log_parameter_max_length` | 16+ | parameter values of prepared statements, in full by default (-1) |
| `auto_explain.log_io`, `auto_explain.log_extension_options` | 19 | |
| `compute_query_id` | 14+ | before 14 a plan is matched by its text |
| Query identifier written into the logged plan (`log_verbose` honours `compute_query_id`) | 16+ | on 14 and 15 the identifier is only in the csvlog / jsonlog `query_id` column |
| `logging_collector` | all | restart |
| `log_destination = csvlog` | all | `query_id` column from 14 |
| `log_destination = jsonlog` | 15+ | one entry per line, with `query_id` |
| `pg_current_logfile()`, `pg_ls_logdir()` | 10+ | `pg_ls_logdir()`: superuser and `pg_monitor` |
| `pg_monitor` role | 10+ | |
| `pg_read_file(text, bigint, bigint, boolean)` | 9.5+ | `GRANT EXECUTE` possible from 11; superuser only before |
| `pg_read_server_files` role | 11+ | more than a plan needs |
| `pg_store_plans` | third-party | tested with 1.10 on 17 and 18, where its query id is `pg_stat_statements`' |

Before PostgreSQL 10 there is no `pg_ls_logdir()`, so the log is not read through SQL there; the view says so and offers pg_store_plans or EXPLAIN now. PostgreSQL 13 is the oldest version DBApguin supports in full.

**What it costs** - said again beside each setting where you turn it on:

- **A restart** for `shared_preload_libraries` (auto_explain, pg_store_plans) and for `logging_collector`. DBApguin never restarts a server.
- **`auto_explain.log_analyze` instruments every statement**, logged or not, which can slow a busy server noticeably. Measure the clock first with `pg_test_timing`; `auto_explain.log_timing = off` keeps the actual row counts (what shows a misestimate) and drops most of the cost; `auto_explain.sample_rate` below 1 instruments only a share of the statements, at the price of missing some slow ones.
- **Log volume.** Every plan is many lines; a low `log_min_duration` on a busy server fills the log quickly. Plan the rotation (`log_rotation_age`, `log_rotation_size`) and the disk. jsonlog or csvlog is a second file beside stderr.
- **The log is sensitive.** It holds statement text and, from PostgreSQL 16, parameter values (`auto_explain.log_parameter_max_length = 0` leaves them out). Whoever may read the log reads that.
- **Reading the log needs file-reading privileges.** `EXECUTE` on `pg_read_file` reads any file in the data directory and the log directory past every privilege inside the database; grant it to a monitoring login only.
- **pg_store_plans** takes shared memory (`pg_store_plans.max` plans) and does work on every statement; most managed clouds do not offer it.
- **Managed clouds** (RDS, Aurora, Cloud SQL, Azure and others) do not let SQL read the server log: the log is in the provider's console, and the view says so and falls back to pg_store_plans where offered, then EXPLAIN now.

DBApguin reads the log only through your connection, keeps what it read in memory while the plan is open, and sends nothing anywhere - see the [privacy notice](PRIVACY.md).

#### Scope: your database, with the server beside it {#awr-ash-scope}

ASH and AWR collect for the whole server, as Oracle's do for the instance: the sampler reads every session of every database, and every snapshot holds `pg_stat_statements` and `pg_stat_database` for all of them. Both keep running in the background when you switch to another connection or tab; the Background jobs list shows them.

A report says first where its data comes from (**Collected for**: server and the database the sampler or the snapshots were started from) and then what it is about. By default that is the same database: Top SQL, waits, findings and cards are that database's, and beside them stands the whole server - a second row in the AWR **Window summary**, **Activity by database** for every database, and an **Elsewhere on the server** finding when another database did something worth knowing (deadlocks, 50 or more rollbacks, a fifth of the execution time AND at least 10 s; in ASH a fifth of the active samples AND at least 60 of them, or 10 or more samples waiting on locks). `*` in the **Database** field shows the whole server, and another database's name shows that database with the same overview. The choice is made when reading, not when capturing, so the same snapshots and samples serve all three. **Top tables by DML** is the connected database's (PostgreSQL keeps those counters per database) and appears only when that is the database the report is about. **Plan** asks for the plan in the database the statement ran in. Samples and snapshots from before 22-23 October 2026 record no database per statement; for them the report shows the whole server and says so.

**pg_stat_statements was full and evicted statements** in a compare means AWR's Top SQL is missing whatever those statements cost: `pg_stat_statements` keeps `pg_stat_statements.max` statements and drops the least-used 5% (at least 10) each time it is full. PostgreSQL 14 and later count those rounds; the finding gives them, an estimate of the statements lost and how full it is. Raise `pg_stat_statements.max` (a restart) - the settings table at the end has the row. A statement whose counters went backwards (evicted and back, or a reset) counts from its return.

#### Trend: every snapshot charted over time {#awr-trend}

`db-awr-snapshot` action=trend turns the whole snapshot history for one server into charts, instead of the one-pair-at-a-time view `compare` gives. Needs at least two snapshots - take them manually, or let the auto-scheduler build up history in the background.

Two different kinds of number appear on these charts, and they are drawn differently on purpose:

- **Point-in-time values** - database size, estimated heap bloat - are exactly what was measured at the moment of that snapshot. One point per snapshot, charted as-is.
- **Rates** - cache-hit%, rollback%, top queries by total time - come from `pg_stat_database` and `pg_stat_statements` counters, which only ever count up since the last reset. A single snapshot's raw counter says nothing; what is charted is the **delta between two consecutive snapshots**, turned into a rate for that window. This is the same arithmetic the two-snapshot `compare` uses, just repeated for every consecutive pair in the history instead of one pair the user picked.

**Regressions** are called out above the charts, not left for the reader to spot in a line: a metric is flagged only when it moved by both a relative amount (a percentage) and an absolute amount (bytes, or percentage points) - the same rule every threshold in this app uses, so a small database doubling in size does not out-rank a large one growing by far more in absolute terms, or the other way round. The comparison point is about a week before the most recent snapshot, or the oldest available point when the history is shorter than that.

**Gaps are shown, never smoothed over.** When the interval between two snapshots is far wider than the server's usual interval (2.5x the median, with a floor of 3 hours so two manual snapshots taken an hour apart are never flagged against each other), the chart breaks the line there instead of drawing a straight edge across time nothing was actually captured for. A shaded band marks exactly where the missing stretch is; hovering it says how wide.

The **top-queries** chart tracks the handful of `queryid`s that cost the most total time summed across the whole history, each as its own line of per-window time spent - not the single cumulative total, which only grows and would just be a straight diagonal line saying nothing about *when* the cost was paid.

A **biggest tables** list under the size chart names the largest tables as of the latest snapshot - "database size" alone does not say which table actually grew. It is not tracked per-table over time (a table can drop out of the top 10 and back in as others grow around it), just the current picture, the same as `db-table-sizes`.

#### Saved reports {#saved-reports}

Every workload capture, AWR report (a report of two snapshots, a period comparison or a trend), ASH report and session trace is kept when it finishes. *Tuning & diagnostics* > **Saved reports** lists them from every server in one place, newest first: when, the type, the server, the database it is about (*whole server* for a report over all of them) and what it covers (the window, the two snapshots, the traced user), with its size. The **Saved reports** link in the AWR, ASH and session trace tabs opens the same list filtered to that tool and that server; from the AWR tab's *Capture* result it shows the workload captures.

- **Filters**: type, server, database and a date range (the date picker underlines the days that hold a report). **Clear filters** shows everything again.
- **Open** shows the report as it was saved, read-only: nothing is asked of the database, and the buttons that would change it are not there. Its footer has the export row the report had (text, CSV, JSON, PDF), written with the report's own server and run time, not today's.
- **Export** opens the same four formats in the row, without opening the report.
- **Delete** asks first and says what goes: the report with its sections and its text, and its entry in Background jobs; for a trace also its entry in the trace's History. A copy exported earlier is not affected. The audit log records the deletion.

Reports are kept apart from the other results Background jobs keeps: the newest 200 of them stay however many other checks run (the trace store keeps the newest 500 traces per server). The library stores nothing of its own, so the app data export and import carry it: saved reports travel with the reports item, traces with the server's AWR and ASH history.

### 6.7 Logical subset export (a slice that loads on its own)

A backup copies whole tables; this copies a slice of one. You name the starting
rows with a WHERE clause - one customer, one tenant, last month's orders - and
the export follows the foreign keys out from them: parents, because a row whose
customer is missing will not load, and children, because an order without its
items is not the same order. What comes out is a single `.sql` file, written
parents first, that loads into an empty database on its own without a foreign
key error.

The usual reason to want one is a test database that behaves like production and
fits on a laptop: tenant 42 instead of 400 GB.

1. Schema browser -> right-click a table -> Sub-setting.
2. Type a WHERE clause for the seed table.
3. **Discover closure** walks the foreign keys without materialising any rows and
   reports how many each table would contribute. Run it first: a WHERE clause
   that looks narrow can pull in most of the database through one wide join
   table, and this is where you find that out rather than half way through the
   export.
4. Export writes the rows to a location you pick - INSERT statements, or COPY
   blocks when the load matters more than being able to edit it.

Depth (how many foreign keys the walk follows) and a per-table row cap both stop
the walk. Hitting either is reported rather than silently applied: a truncated
subset that looks complete is worse than one that says it is not.

Rows are copied exactly as they are, **personal data included**. Masking is in
Copy table, below.

### 6.8 Masking personal data in a copy (PII anonymisation)

Personal data - names, e-mail addresses, phone numbers, national ID numbers,
dates of birth - is what stops a production copy being something you can hand to
a developer or a supplier. Schema browser -> right-click a table -> **Copy
table** -> PII anonymization replaces those values while the rows are read: the
column is swapped for an expression inside the SELECT, so the real value never
reaches the destination and is never written to disk. The source table is not
modified. You name the columns yourself - nothing is detected automatically, and
a column you forget is copied in the clear.

| Rule | Writes | What still works |
| --- | --- | --- |
| `NULL` | nothing | Nothing to leak. Fails on a NOT NULL column. |
| `@HASH` | `sha256` of the value (`md5` on PostgreSQL below 11) | Equal values stay equal, so joins and GROUP BY still work. Add a salt (below) so the hash cannot be matched back to production. |
| `@HASH:<salt>` | as `@HASH`, with this per-column salt mixed in | For the rare case where one column needs its own salt on top of the copy-level one. |
| `@SEQ` | `name_1`, `name_2`, ... | Values stay unique. Any match to another table is gone. |
| `@EMAIL` | `user1@example.invalid` | Valid syntax, and `.invalid` can never be delivered to - no test run can mail a customer. |
| `@PHONE` | `0000000000` | The length, so format checks still pass. |
| `@DATE` | the first of the month | Monthly reporting and age brackets. The exact day is gone. |
| `@REDACT` | `REDACTED` | Only that a value was there. |
| `@EXPR:<sql>` | whatever the SQL returns | Your own rule, e.g. `left(name,1) || '***'`. It is not checked - a wrong expression fails the copy. |
| any other text | that text, in every row | A fixed placeholder. The column stops varying at all. |

#### The salt

An unsalted hash is a disguise, not proof. Where the set of possible inputs is
small and known - a national ID, a phone number, a postcode - anyone with the
same hash function can hash every possibility and match the values straight
back. A **salt** is a secret string mixed into the value before it is hashed, so
the hash cannot be reproduced without it. The **Salt for @HASH** control offers:

- **none** - the hash of the value alone (re-identifiable as above).
- **random, this copy only** - a fresh random salt is generated for this copy
  and thrown away afterwards. Equal inputs still hash equal *within* this copy,
  so joins and GROUP BY on masked columns keep working, but the copy cannot be
  matched against production or against a different copy.
- **fixed** - you supply the salt. Use this when you make **incremental** copies
  (a second copy must line up with the first) or when you need to **compare two
  copies** - both need the same salt. Keep it somewhere safe; anyone who has it
  can undo the salt's protection.

The salt is sent to the server as a transaction-local setting
(`set_config('dbapguin.anon_salt', ...)`) and read back by the masking
expression through `current_setting()`. It is **never** written into the
generated SQL, the DBApguin audit log, or the destination table. It does reach
the source server as one `set_config` statement - if that server logs every
statement (`log_statement = all`), the salt appears there once.

`@SEQ`, `@REDACT` or `NULL` are still the right choice when the copy must not be
re-identifiable at all, salt or no salt. And masking works one column at a time:
a row with the name removed but the date of birth and postcode intact often
still points at one person.

A column you do not need at all is better dropped entirely, under Data filters ->
Skip columns.

### 6.9 Constraints - take them off, put them back

Sometimes the constraints have to come off a table and go back on afterwards.
The usual case is a replication or CDC tool doing its initial sync into a
target: it will not start while foreign keys are enforced there. A bulk load is
the other one - it runs faster without the checks and gets them back at the end.

Done by hand this is where data quietly stops being consistent, because three
things are easy to get wrong and PostgreSQL only stops you on one of them:

- `DROP ... CASCADE` removes the foreign keys other tables have pointing at this
  one, and does not record what it removed. Their definitions go with them.
- A definition retyped from memory loses `ON DELETE CASCADE`, or `DEFERRABLE
  INITIALLY DEFERRED`, or a `WHERE` clause on a partial unique index. It looks
  restored. It enforces something else.
- Adding a foreign key back the ordinary way scans the whole table under a lock
  that stops reads and writes.

Schema browser -> right-click a table -> **Constraints - drop and restore**. It
reads every constraint from `pg_constraint` through `pg_get_constraintdef`,
which is PostgreSQL's own rendering of it, and writes two scripts: one that
drops them in an order that works, one that puts them back exactly as they were.
Nothing is executed by the app. You get SQL, and you run it when you decide to.

Three options change the restore script:

- **Add NOT VALID, then validate separately.** `ADD CONSTRAINT ... NOT VALID`
  skips the scan; `VALIDATE CONSTRAINT` does it afterwards under `SHARE UPDATE
  EXCLUSIVE`, which readers and writers run straight through. Measured on a
  200,000-row table: 139 ms of blocking became 1.2 ms, plus a 131 ms pass nobody
  waits on. Until VALIDATE has run the constraint applies to new rows only, and
  `pg_constraint.convalidated` says so.
- **Rebuild primary key and unique indexes online.** `CREATE UNIQUE INDEX
  CONCURRENTLY`, then `ADD CONSTRAINT ... USING INDEX`, which adopts that index
  instead of building a second one - so the result is identical to a plain
  re-add. The index is built under the constraint's own name, from the
  definition the server itself gives (`pg_get_indexdef`) - so a covering key,
  a fillfactor, a tablespace or `NULLS NOT DISTINCT` all come back as they
  were, and a deferrable constraint keeps its deferrability. `CREATE INDEX
  CONCURRENTLY` cannot run inside a transaction block, which is why the restore
  script must not be wrapped in `BEGIN`/`COMMIT`.

  It is also the one statement in the script that can fail and leave something
  behind: a duplicate row or a cancelled run leaves an index that exists and is
  unusable, and nothing in the constraint list explains it. The script drops
  such an index itself before rebuilding, so running it again after a failure
  is enough. Only an index PostgreSQL has marked invalid is dropped - one that
  is in use is never touched.
- **Include the foreign keys pointing at this table.** Off, the drop script
  fails on the primary key: another table's foreign key depends on its index.
  On, those tables briefly stop enforcing their reference to this one - which is
  the trade the situation asks for, stated rather than hidden inside a CASCADE.

Both scripts open with `SET lock_timeout = '3s'`. Every `ALTER TABLE` in them
needs `ACCESS EXCLUSIVE`, and waiting for that lock is what turns an online
rebuild into an outage: the request queues behind whatever is running, and
everything arriving afterwards queues behind the request. Three seconds means
the worst case is a statement to run again rather than a table nobody can
read. Delete the line to wait for as long as it takes. DBApguin's own editor
sets two seconds on every connection regardless, so this matters most when the
file is run months later from `psql`.

Save the restore script before you run the drop script. It is the only copy of
what the table used to enforce.

#### Scope: one table, one schema, or several

The same dialog answers all three. Right-click a **table** for that table;
right-click a **schema** → Transfer → *Constraints - drop and restore* for
every table in it. A migration or a replication cut-over is a question about
the schema, and doing it table by table is how a constraint gets forgotten.

**Which constraints** filters by kind - primary keys, unique, foreign keys,
check, exclude. Ticking only *foreign keys* is the common one: a CDC tool
usually objects to those and to nothing else, and leaving the primary keys in
place means the target keeps its identity and its indexes.

At schema scope the order matters more than it does for one table, and the
script takes care of it: every primary and unique key goes back before any
foreign key that references it, and the drop script does the reverse. Ordered
by table name instead - which is what the first version did - a foreign key on
`items` is added before `orders` has its primary key, and PostgreSQL refuses
it with *there is no unique constraint matching given keys*.

#### Copying constraints onto another schema

Name a **target schema** and the dialog stops writing a drop script and writes
one that puts these constraints on that schema's tables instead. The case it
is for: a copy of the data loaded without constraints - because that is how it
loads fast, or because replication would not start otherwise - and now it
needs the same rules the original has.

Table and column names have to match. Only the constraints travel: no data, no
columns, no indexes beyond the ones a key needs.

The definitions are read from inside the source schema, so a reference to a
table in that schema comes out unqualified - `REFERENCES customers(id)` - and
the script's `SET search_path` points it at the target. A reference into a
*different* schema stays qualified and keeps pointing where it pointed, which
is also right: a foreign key into a shared reference schema should not be
redirected.

Constraint names come across unchanged, because a constraint name is unique
per table rather than per database. If the target already has one of that name
on that table, the statement fails - the safe failure, since the alternative is
replacing something silently.

### 6.10 Similarity search: pg_trgm and pgvector {#similarity-search}

Two extensions answer "find rows like this one", in two different ways:

- **pg_trgm** compares how text *looks*. It cuts text into three-letter pieces
  (trigrams), so a typo or a part of a name still matches: `LIKE '%pad%'`,
  `ILIKE`, the similarity operator `%` and the trigram distance `<->`. It ships
  with PostgreSQL's contrib modules.
- **pgvector** compares what a model says the text *means*. A model turns each
  text into a list of numbers (an embedding, stored in a `vector(n)`,
  `halfvec(n)` or `sparsevec(n)` column); close numbers mean close meaning.
  Queries ask for the nearest rows: `ORDER BY embedding <=> $1 LIMIT 10`.

The two need different indexes, and a btree serves neither: not `ILIKE` or a
pattern that starts with a wildcard, and not a distance. What the app does with
each:

**Database -> Manage extensions** lists both with the installed version
and the newest one the server ships; **update** offers every later version, in
version order (0.10 after 0.9).

**EXPLAIN** reads the indexes a plan uses, not just their names:

- an index scan through **HNSW** or **IVFFlat** is named as an approximate
  nearest-neighbour search, with the operator class it was built for and the
  setting that trades speed for recall (`hnsw.ef_search`, `ivfflat.probes`)
  as the session that ran the plan had it;
- a filter on the same step is flagged: the index hands over its candidates
  first and the filter runs after, so fewer rows than the `LIMIT` can come
  back. From pgvector 0.8.0 the hint offers `hnsw.iterative_scan`;
- a **Sort by a distance over a sequential scan** is an exact search over
  every row. The hint says which of three it is: no approximate index on the
  column; an index built for *another* operator (an index with
  `vector_cosine_ops` serves `<=>` only, so `ORDER BY embedding <-> ...` reads
  the whole table); or a matching index the planner did not use, because it is
  only used for `ORDER BY column <op> value` in ascending order with a `LIMIT`;
- a **trigram index** (GIN `gin_trgm_ops`, GiST `gist_trgm_ops`) is explained
  as pieces plus a recheck, with how many of the rows it found were no match;
- a sequential scan filtered by `LIKE`/`ILIKE` on a column with only a btree
  says the btree cannot serve it and gives the trigram index; `similarity(a,
  b) > x` in a filter is never indexed, and the hint says to write `a % b`.

Nodes a `LIMIT` stopped early (a top-10 search reads 10 rows of an estimated
20,000) are no longer flagged as the worst estimate of the plan.

**Index advisor** (from pg_stat_statements): for `LIKE`, `ILIKE`, `%` and
`similarity()` on a column it proposes a trigram GIN index, never a btree, with
`CREATE EXTENSION pg_trgm` first where the database does not have it yet (the
Run dialog shows both and runs both). For `ORDER BY column <distance> ...` it
proposes an HNSW index with the operator class for the operator the queries
use, read from the server's own catalogue, and says the trade-off: approximate,
it can miss some of the true nearest rows. It never proposes a btree on a
vector column. A column with more dimensions than an index takes (2,000 for
`vector`, 4,000 for `halfvec`) or with none declared gets the explanation
instead of a statement.

**Key settings**, where pgvector is installed in the database: `hnsw.ef_search`,
`hnsw.iterative_scan`, `hnsw.max_scan_tuples`, `ivfflat.probes` and
`ivfflat.iterative_scan` (the iterative ones from pgvector 0.8.0), and
`pg_trgm.similarity_threshold` where pg_trgm is. `maintenance_work_mem` is
compared with the largest HNSW index: pgvector's README says an HNSW build is
significantly faster when the graph fits into it, and that in a container a
parallel build needs `/dev/shm` at least that large. `max_parallel_maintenance_workers`
says it is what pgvector builds its indexes with.

**Sizes.** Schema -> a table -> **Storage** lists each vector column with the
bytes one value takes (pgvector's formula: `vector` 4 x dimensions + 8,
`halfvec` 2 x dimensions + 8), what a sample of stored values takes, and on
PostgreSQL 17 how many of them live in the TOAST table - a `vector(768)` is
about 3 kB, more than a row keeps inline, which is why such a table shows a
small heap and a large TOAST table. Below it, the table's indexes with method,
operator class, options (`m`, `ef_construction`, `lists`) and size. The
**Index sizes** check names the method and operator class of every index and
the TOAST size of its table.

**Create index** offers `hnsw` and `ivfflat` as methods and an operator class
for the columns (`vector_cosine_ops`, `gin_trgm_ops`, ...); ASC/DESC and NULLS
are offered for btree only, the one method that accepts them.

**Copies.** DDL, Compare and Copy keep operator classes and index options. A
schema clone with rows builds the HNSW and IVFFlat indexes after the rows (an
IVFFlat index computes its lists from the rows present when it is built), and
Copy table rebuilds an IVFFlat index once the rows are in. Compare puts a note
above a change of a vector column's dimension: stored vectors are not converted
to another dimension, so the rows have to be embedded again.

## 7. Settings

Top right gear icon opens Settings:

- Theme: Dark / Light
- **Language: English, Hrvatski, Deutsch, Español.** The choice is stored on
  the server side (`app_prefs.json`), so it follows you across restarts
  rather than living in one browser profile. It applies immediately, without
  a restart, including to menus and dialogs already on screen.
- Font size (10-30 px)
- **Startup -> Port:** the port DBApguin's own backend listens on - on this
  computer only, `127.0.0.1` - 3236 unless you change it. Change it when
  another program needs 3236; it takes effect the next time DBApguin starts.
  What the window remembers (theme, favourites, SQL history, open tabs) moves
  with it.
- Audit log: enable / disable (see below)
- Tools: paths to psql / pg_dump / pg_restore / pg_repack (override auto-discovery)
- Certificates: upload PEM-encoded root certs for `sslmode=verify-*`
- Certificates and tool paths as above; **About has moved to Help -> About DBApguin**, alongside the legal documents

### Audit log {#audit-log}

On unless you switch it off - the switch is in the log window itself. Every
statement the SQL editor runs (unless the tab is set read-only) and every
change made through DBApguin's own tools is appended to
`audit/audit-YYYY-MM.jsonl` (one file per month) - the time, the target, your
operating-system user and the database user, the statement with passwords
redacted, how long it took and the outcome. Open it from **View -> Audit
log**: search by date range,
action or a substring of the SQL, export a range as JSONL, or read the raw
files directly with any text editor or `jq`.

**Every record is chained to the one before it.** Each line carries a
`hash` of its own content plus the previous line's `hash` (`prev_hash`), so
changing or deleting a past line changes what the next line's hash should
have been - **Verify chain** in the log viewer recomputes the whole thing
and tells you either "intact" or exactly which file and line is the first
one that no longer matches. A record written before this existed has
neither field; the verifier treats those as pre-chain and reports where the
chain itself begins, rather than failing on them. The chain also survives
the monthly rotation - a new month's first record links back to the
previous file's last one, so splitting the log into files does not split
the chain.

**Read what this proves carefully, because it is easy to overstate.** It
proves nobody made a casual, unnoticed edit to a chain that verifies -
useful on its own, and the reason the button exists. It does **not** prove
nobody with write access to the file tampered with it: hashing needs no
secret, the algorithm is public, and the log sits right next to the
application that can recompute it, so a local administrator (or anyone with
your OS-level file permissions) who edits a record can also recompute every
hash after it, and the chain will verify again as if nothing happened. Real
protection against that needs an anchor outside this machine - a signature
with a key you do not hold, or the hash sent somewhere else the moment it
is written - and this feature does not do that. Where you need a record
that survives a compromised local administrator, use your database
server's own logging (`pgaudit`, `log_statement`) writing to a separate,
access-controlled destination, not this file. See the [licence agreement](EULA.md), section 7 (Help -> Licence agreement).

### App data: export, import and clear {#app-data-backup-and-clear}

Everything DBApguin keeps between runs lives in one directory (its path is shown in both dialogs). Settings -> App data has three buttons.

**Export app data** writes one `.tar.gz` of what you tick: saved connections, preferences, saved queries, the SQL workspace, the task list, saved reports, the stored AWR snapshots, ASH days and session traces of every server, the audit log, installed plugins and their data, TLS certificates. Everything is ticked by default. The data key travels with the connections and the plugin data, because their passwords cannot be read without it.

**Import app data** reads such an archive, lists what it holds beside what is already here, and takes what you tick (again everything by default). It merges rather than replaces: stored history (AWR snapshots, ASH days, session traces, saved reports, job logs, the audit log, plugins) is added and never overwrites a file that is already here; saved connections are merged by id, a name already taken gets a number, and their passwords are re-encrypted for this installation's own key; a single file such as the preferences or the saved queries replaces yours. An archive whose key is missing, or protected by a master password, imports the connections without their passwords, and says so. This is how a DBA takes the collected history and the setup to another computer; there is no shared store between installations.

> The backup **includes `key` and `connections.json`** - the data key and your encrypted connection passwords. That is deliberate: a backup of your setup without them is not one. But it means the archive is sensitive. Save it somewhere private and **never in a folder that syncs to a cloud service** (Dropbox, OneDrive, iCloud, Nextcloud, a synced Documents folder). If a master password is set, the passwords in the archive stay encrypted with it; without one they are only lightly obfuscated.

The backup also carries your **installed plugins** (`plugins/`) and what they keep (`plugin-data/`) - their settings and any passwords they store, encrypted with the same `key` - so restoring it brings the plugins back, working. The folder you write your own plugins in, if you do, is not in it: it is wherever you chose to keep it.

Exporting connections from the connection manager is a different thing: it writes only the connections and their groups, without passwords, settings or plugins.

**Clear app data** shows a checklist with each item's size. Reports and logs - AWR snapshots, job logs, saved check reports, the error log - are ticked by default; preferences, saved queries and the SQL workspace are listed but unticked. Saved connections (`connections.json` + `key`) are in a separate section and require typing the word `DELETE` to confirm, because clearing them loses every saved connection and its password. Installed plugins are listed unticked; their settings and stored passwords (`plugin-data/`) are in the same protected section as the connections, since clearing them loses those passwords. A directory that still holds a running background job is refused; cancel the job first. Every clear is written to the audit log.

### Plugins {#plugins}

**Plugins → Manage plugins…** lists what is installed: switch a plugin on or off, remove it, or install one from a `.dbapg-plugin` file. A plugin adds checks, menu entries and tabs without a new version of DBApguin.

**Anyone can create a plugin** - the application includes a builder for it (switch on *Develop plugins in this application* in the same tab; the *Plugin guide* there explains the rest). A plugin runs with your rights: it can reach every database you connect to and any other service it is set up to use. **Use only plugins from publishers you trust.** The licence agreement (section 15) and the privacy notice say the same in their own terms.

**Info** on each plugin explains it in two parts. First what the plugin says about itself - what it does, what it reads, what it sends where - in your language when it brings one. Then what DBApguin itself reads from the plugin's files, whatever the plugin says: whether it runs Python inside the application or JavaScript in the window, which checks and addresses it adds, what it stores, its folder and every file.

**DBApguin certified** beside a plugin's name means that DBApguin's author signed it and that none of its files has changed since. *Changed since certified* means files differ from the signed ones (Info names them); *signature invalid* means the signature does not hold; *not certified* means nothing vouches for where it comes from - which is what a plugin your team wrote looks like. A plugin that is not certified still installs and runs: the badge tells you where a plugin comes from, it does not decide for you. Installing one that is not certified asks first, and the question shows what it would run and every file it brings.

Removing a plugin deletes its settings and stored passwords with it. A backup of the app data carries the plugins and their settings.

## 8. Toast notifications

Bottom-right corner. Color-coded:
- Blue (info): neutral status
- Green (ok): success
- Red (err): operation failed

Click the toast stack to expand and scroll. Wheel scrolls. Click outside to collapse.

While a window is open, notifications keep clear of its buttons. A new one stands above them, or in the room beside the window, and when neither leaves room it goes straight into the counter, whose number and colour still tell you it came. The counter itself stays above the window's buttons; if not even it fits, it is hidden until the window closes. Everything goes back to the corner when the window closes.

One action offers an Undo: deleting a saved connection, which the toast can put back for about six seconds. A toast cannot undo what reached a database. What the application does instead is show you the statement before it runs, ask once, and record it in the audit log; a setting changed here can be undone from the settings view, and a drop, a move or an owner change leaves a copy - for a drop with the rows, when chosen, and none when you choose to drop without one - that puts the objects back (see *A copy of the definitions before the change*).

### 8.1 While something is running {#while-something-runs}

Anything you asked for that takes longer than a quarter of a second says so, in the same places every time:

- A thin bar slides along the top edge of the window, and a ring turns at the right end of the row of tabs below the menus. Point at the ring, or move to it with Tab, for the list of what is running, the step each one is on and how long it has taken; past three seconds the ring shows the time itself. Where the work can be stopped - a check, a statement in the SQL editor - its line in the list has a Cancel.
- The button you pressed shows a ring in place of its label and does not take a second press until the work has ended. Nothing beside it moves: the button keeps its size, and the bar, the ring and the list take no room from the page.
- A part of the window that is loading says so ("Reading the jobs…", "loading columns…") instead of showing an empty list or values that are about to change, and past three seconds it shows how long it has been waiting. A setting whose saved value is still being read is dimmed and cannot be changed until the value is in.

Exports and reports name each step: building the report, waiting for your choice in the file dialog (if the dialog opened behind the window, that line is the clue), writing the file or the PDF. The busy state ends the moment the work does, whether it worked or failed, and a notification says where the file went or why it did not.

What the application does on its own - the dashboard's refresh, the connection checks, the job watcher, the collectors' polls - never shows here, so a turning ring always means something you asked for is still on its way.

## 9. Keyboard shortcuts

- `F5` (or `Ctrl/Cmd + Enter`): run the whole SQL editor
- `F6` (or `Ctrl/Cmd + Shift + Enter`): run only the selection, or the statement the cursor is in
- `Ctrl/Cmd + S`: save current SQL as snippet
- `Ctrl/Cmd + E`: validate query
- `Ctrl/Cmd + K`: command palette
- `Esc`: close current modal / context menu / clear search
- `Ctrl/Cmd + W`: close current work tab (if not pinned)
- `Shift + Tab`: switch tabs, the way Alt + Tab switches windows. A window of cards opens, one per open tab, each with a picture of the tab's content as you last saw it, and the next tab marked. Keep Shift held and press Tab or the arrow keys to move the mark (Left goes back, Up and Down move a row). Let go of Shift to open the marked tab; Esc closes the window and stays where you were. In a text field or the SQL editor Shift + Tab keeps its usual meaning (previous field, outdent); use `Ctrl + Tab` there, and let go of Ctrl to open the tab.
- `Ctrl + Shift + Tab`: switch connections in the same kind of window: one card per open connection with a picture of what was on screen in it, its server, its state and how many tabs it holds. The connection is switched when you let go of Ctrl, not while you look. On macOS this stays on the Control key, because macOS keeps Cmd + Shift + Tab for itself.
- `F10`, or a tap of `Alt` (Option on a Mac): open the menu bar from the keyboard. Left and Right move between menus, Up and Down through the entries, Right opens a submenu and Left closes it, a letter jumps to the next entry starting with it, Enter runs the entry (a check opens as it does on a click), Esc closes one level at a time. A menu opened with the mouse takes the same keys.

The keys are shown where they are used, named for your system (the Command, Option and Control symbols on a Mac): on the SQL editor's buttons and in their tooltips, in this list (press `?`), and as a short tip after you have done the same thing with the mouse a few times. Each tip appears at most twice; *No more tips* turns them off, and the switch at the bottom of the `?` list turns them back on.

## 10. Provider notes

DBApguin detects the cluster's provider on connect and adapts:

| Provider | What is gated |
|----------|---------------|
| Aurora PostgreSQL | No ALTER SYSTEM, no raw pg_hba write, no superuser-level role creation, no custom tablespaces, no raw file read |
| Amazon RDS PostgreSQL | Same as Aurora plus an "allowlist" check for CREATE EXTENSION |
| Google Cloud SQL | Same as RDS |
| Google AlloyDB | Same as Cloud SQL: settings are the instance's database flags. An AlloyDB Omni server you run yourself has a real superuser and counts as self-hosted |
| Azure Postgres Flexible | Same as RDS |
| Supabase | Same as RDS plus reserved-role gates |
| Neon | Single-DB tenant; no CREATE DATABASE / DROP DATABASE |
| Heroku Postgres | Same as Neon |
| Crunchy Bridge | Most enterprise features available |
| DigitalOcean Managed | Same as RDS |
| Timescale Cloud | Allowlist extensions plus the tsdbadmin role gate |
| Self-hosted PostgreSQL | Everything available |

UI greys out blocked menu items with a tooltip explaining why. When the role of the session is a real superuser, nothing is greyed out or refused: no managed platform gives a customer one, so the server is one where those limits do not apply (for example a self-hosted server that imported a platform's role names). The platform is still named, and its banner says that this session is a superuser. The `db-errors` watcher falls back to counter deltas + provider-specific log access guidance when `pg_read_file` is denied.

#### Backing them up as a .sql file

**Drop script…** and **Restore script…** put the script on screen first, with
the three choices every generated script in this application offers: copy it,
save it, or open it in the SQL editor and run it there. Saving refuses to
overwrite an existing file without asking. The restore file is the backup: it
carries the create statements and the validation step, and it is the thing to
keep before the drop script runs.

Every file opens with a header - the app version, the server and database it
came from, the scope, the time it was taken, and how many constraints it holds.
The password never appears in it. Six months later that header is the
difference between a usable file and an anonymous pile of ALTER TABLE
statements.

For a whole database rather than a schema, `pg_dump --section=post-data` is the
other way round to the same place: it writes the constraints, indexes and
triggers of everything at once. That is Backup → Options → Section → post-data.

### IAM logins and generated names

On a managed cloud the login role usually **is** the IAM principal, so its
name is an email address or a service account:

```
alice@example.com
123456-compute@developer.gserviceaccount.com
```

Those are valid PostgreSQL role names - quoted identifiers may contain `@`,
`.` and `-`, and may begin with a digit. DBApguin accepts them everywhere a
role name is asked for: create, drop, grant, revoke, membership,
`search_path`, ownership and every privilege screen. The SQL it emits quotes
them, so a grant reads:

```
GRANT SELECT ON TABLE "app-schema"."orders" TO "alice@example.com"
```

The same applies to database and schema names. Managed providers generate
names like `b2b-ext-assistant-dev-pg-db` routinely, and hyphens are legal in
a quoted identifier.

What is still refused, deliberately, is a name containing a double quote, a
NUL, whitespace or a control character - none of which a real cloud
principal has, and all of which could end an identifier early.

IAM users have **no password**. When creating one, pick `No password` in the
Create role wizard, then grant the provider's IAM membership (`rds_iam` on
AWS, `cloudsqliamuser` on Cloud SQL) from the Member of list.

## 11. Troubleshooting

### "Password is required"
Saved connection has no password stored. It is tried as it is first: a local server that needs no password (trust, peer, `.pgpass`) simply connects. A server that asks for one opens the Refresh connection dialog; type the password and click Connect. If the tab connects some other way while the dialog is open, the dialog closes by itself.

### "Refusing to write inside <path>"
pg_dump backup destination is inside a read-only location (AppImage mount, app data dir). Pick a path under `~/Documents` or similar.

### "pg_stat_statements must be loaded via shared_preload_libraries"

pg_stat_statements has two halves, and both are required:

1. the library must be listed in `shared_preload_libraries`, and
2. the extension must be created in the database you are querying.

Running `CREATE EXTENSION pg_stat_statements` does only the second. The
extension then exists in `pg_extension`, everything looks installed, and
every read of the view raises this error. Running `CREATE EXTENSION` again
is the natural next thing to try, and it changes nothing.

DBApguin recognises this state and says which half is missing. Any check that
reads query statistics prints the reason and the fix underneath the error,
and **db-setup-advisor** reports it as *"Created in this database but NOT in
shared_preload_libraries"* rather than as a plain "not installed".

The fix, as a superuser:

```
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';
```

then **restart** PostgreSQL. `shared_preload_libraries` is postmaster-only - 
`SELECT pg_reload_conf()` will not pick it up. On a managed cloud, set it in
the parameter group and reboot the instance.

The reverse state - library loaded, extension not created - needs no restart
and no intervention: the checks create the extension themselves and carry on.

Checks affected while it is half-configured: db-stat-statements,
db-index-advisor (its pg_stat_statements heuristic only), db-temp-queries,
db-workload-report, db-awr-snapshot and db-trace-session.

### An index exists but the planner ignores it {#leftover-damage}

Usually this means the index is INVALID, and nothing in a normal listing
says so. `CREATE INDEX CONCURRENTLY` and `REINDEX CONCURRENTLY` build in
phases, and a session that dies partway through - a crash, a kill, a lock
timeout - leaves the half-built index behind. The planner will not use it,
it still takes up space, and `\d` marks it only if you look closely.

**Cluster health → Check for leftover damage...** scans for the states
PostgreSQL records but does not surface:

- invalid indexes, from an interrupted CONCURRENTLY build
- `NOT VALID` constraints, which are not enforced against existing rows -
  a restore that deferred validation leaves these
- disabled triggers, which a `--disable-triggers` restore leaves off if it
  was interrupted

Pick a schema or leave it on *(whole database)*. Each finding comes with the
statement that fixes it, and an invalid index gets a **Fix now** button that
runs `REINDEX INDEX CONCURRENTLY` on it as a background job.

One thing to expect: a failed `REINDEX CONCURRENTLY` leaves its own
half-built copy under a `_ccnew` suffix, which is a separate object from the
original. Repairing the original does not remove it - run the check again
afterwards and fix that one too.

### Window does not appear (Linux)
DBApguin opens a setup dialog with per-distro install commands when WebKit2GTK is missing. Pick your distribution's tab, click "Copy command", paste into a terminal, then relaunch the AppImage. The dialog never falls back to a web browser - DBApguin is standalone-only.

### Window does not appear (Windows)
DBApguin opens a setup dialog with three install paths for Microsoft Edge WebView2 Runtime: the bundled bootstrapper from the app folder, the online installer, and enterprise (GPO/SCCM/Intune) deployment guidance.

### Window hangs on close (Windows)
Should be fixed as of the June 2026 build. The app force-exits 2 seconds after you click the close X. If you still see it hang, please attach the contents of `<DBCHECKS_HOME>\dbchecks_error.log` to a bug report.

### DBApguin does not start: the port is taken
- When another program already uses DBApguin's port, the window stays open and
  says so, with a field to start on another port instead - a free one is
  already filled in. DBApguin keeps that port; change it later in
  **Settings -> Startup**.
- When it says DBApguin is already running, an earlier copy is still there:
  switch to its window, or stop the process it names and choose **Try again**.

### Schema browser is empty
- Are you connected? Check the status dot next to the connection.
- Did you switch databases? Click another database under **Databases** to switch; the schema tree reloads with it.
- Hover the **Databases** header, a database or a schema and click the refresh icon that appears, or use **Schema -> Reload schema tree**.

### Audit log is missing
- It is on unless switched off: check the switch in View -> Audit log
- Or check the `audit_enabled` key in `app_prefs.json` under your data dir
- Files live under `<DBCHECKS_HOME>/audit/audit-YYYY-MM.jsonl`

### App data location
On Linux: `~/.local/share/DBApguin/`. Override with `DBCHECKS_HOME=/path .venv/bin/python launcher.py`.

On Windows: the folder next to `DBApguin.exe` by default.

## 12. Data files reference

| File | What it holds | Mode |
|------|--------------|------|
| `connections.json` | Saved connections with Fernet-encrypted passwords | 0644 |
| `key` | Per-install random key for the password cipher | 0600 |
| `app_prefs.json` | UI preferences (font_size_px, audit_enabled, theme, etc.) | 0644 |
| `certs/` | Uploaded TLS root certs | 0600 each |
| `audit/audit-YYYY-MM.jsonl` | DDL audit log, one file per month | 0600 |
| `dbchecks_error.log` | Traceback log (auto-rotates at 1 MiB, keeps 3 backups) | 0644 |
| `webview/` | pywebview localStorage / cookies | (Linux only) |

Back these up before re-installing. Re-installing DBApguin to the same data dir keeps your connections.

## 12.4 Where the documentation is

Three routes, all from **Help** in the menu bar:

| | |
|---|---|
| **User guide** | this document, bundled with the app - works offline |
| **Documentation (online)** | the website, for anything newer than your build |
| **About DBApguin** | version and build, with both of the above beside it |

The licence panel on first run and the startup notice after it both link to
these, which is where most people look the first time.

Documents render inside the app, in the app's own styling - headings, tables,
and SQL examples in a real code block rather than reflowed prose.

There is a fourth route, and in practice it is the one you will use most: the
small circled **i** next to a dialog's heading. It opens this guide at the
section describing that dialog, so the explanation is one click from the
thing it explains rather than a search away. You will find it on the backup
and restore dialogs, on schema rename and merge, and next to the SQL
editor's transaction mode.

The bundled copy is the one that ships with your build, so it always matches
the version you are running. The online copy may be ahead of it.

## 12.5 Legal, privacy and third-party software

Everything below is reachable from **Help** in the menu bar, and ships with
the application - the file the app displays is the file it was built with.

- **Licence agreement (EULA)** - what you may and may not do with the
  software, the warranty disclaimer, and the liability limit. Section 7 is
  worth reading even if you skip the rest: it is about the fact that this
  tool issues `DROP`, `TRUNCATE` and `pg_terminate_backend` against live
  servers at your direction.
- **Privacy notice** - what is stored on your machine and where. The short
  version: everything is local, there is no telemetry, and the only network
  connections are to your databases and to `127.0.0.1`.
- **Third-party notices** - every open-source component the build links,
  with its licence. Generated from the build itself
  (`tools/legal/gen_third_party.py`) rather than maintained by hand, so it
  cannot describe a dependency set that no longer exists.

The legal documents are written in English and English governs. Translations
of the interface do not extend to them, deliberately: a translated licence
agreement that has not been through a lawyer in that language is worse than
no translation, because it looks authoritative.

## 12.6 Why it looks the same on every machine

DBApguin ships the two fonts it uses - Inter for the interface and
Cascadia Mono for anything monospaced - rather than naming fonts and hoping the
machine has them.

That is not cosmetic. A tool that displays identifiers has to display them
exactly: when the app named a font that was not installed, the system
substituted a different one, and one substitute drew the underscore below the
line box, so `pg_read_all_stats` could read as `pgreadallstats` on one
machine and correctly on another. Shipping the font removes the guesswork.

Both are under the SIL Open Font License 1.1 and travel unmodified, with
their licence texts in `static/fonts/` and in
**Help → Third-party components**.

## 13. Coming from Oracle

If you have run Oracle databases for years, most of what you know transfers. What
follows is the part that does not, and where the equivalent numbers live in this
application.

### 13.1 The map

| Oracle | PostgreSQL | Where in DBApguin |
| --- | --- | --- |
| Instance + database (one per instance, usually) | Cluster + many databases; one server process group serves all of them | Servers sidebar; each database is a node |
| Schema == user | Schema and role are separate objects; one role can own many schemas | Schema browser; Roles view |
| Tablespace (mandatory, sized) | Tablespace (optional, just a directory) | `db-tablespaces` |
| Undo tablespace, rollback segments | No undo: old row versions stay in the table until vacuum removes them | Autovacuum advisor, `db-bloat-breakdown` |
| Redo logs + archive logs | WAL segments + archiving | `db-checkpoints`, `db-settings` (WAL and archiving GUCs) |
| SGA / buffer cache | `shared_buffers` + the OS page cache | `db-settings`, `db-cache-advisor`, `db-buffer-cache` |
| PGA / sort area | `work_mem`, per operation per backend | `db-settings` (the WARN there is usually about this) |
| AWR report | `db-workload-report` over pg_stat_statements deltas | Storage & performance menu |
| AWR snapshot / baseline | `db-awr-snapshot` (take / list / compare) | Storage & performance menu |
| ASH / v$active_session_history | `db-ash` sampler over pg_stat_activity | Connections & queries menu |
| v$session | Sessions view; click a row for the full drill-down | Sessions view |
| v$sql / library cache | `pg_stat_statements`, `db-stat-statements` | Connections & queries menu |
| SQL trace / 10046 | `db-trace-session` plus `auto_explain` on the server | Tuning & diagnostics menu, *Session trace* |
| DBMS_STATS | `ANALYZE`, autovacuum's analyze half | Advisor tab on a table |
| Optimizer hints | No hints. You change statistics, indexes, or the query | `db-index-advisor`, EXPLAIN in the SQL view |
| RMAN | `pg_dump` / `pg_restore` (logical), `pg_basebackup` (physical) | Backup and restore |
| Data Guard | Streaming replication + replication slots | `db-replication`, `db-repl-slots` |
| Materialized view refresh (fast/complete) | `REFRESH MATERIALIZED VIEW [CONCURRENTLY]`, always a full rebuild | Right-click a matview |
| DBA_* / V$ views | `pg_catalog.pg_*` and `pg_stat_*` views | Every check prints the SQL it ran |
| Invalid objects, recompile | No such state: a function is parsed when it runs | the post-import check, run automatically after a restore |
| Public synonyms | `search_path`, or a view in a shared schema | `db-settings` |
| Sequences with NOCACHE / ORDER | Sequences cache per session; gaps are normal and not an error | `db-sequences`, Sequence health check |
| Profiles, resource manager | Not built in. Connection limits per role, plus a pooler | `db-connections` |

### 13.2 The five differences that actually bite

**1. There is no undo - vacuum is the price of MVCC.** An `UPDATE` writes a new row
version and leaves the old one behind, dead but still on the page. Autovacuum removes
dead versions later. Two consequences that have no Oracle counterpart: a table can grow
even when the row count does not (bloat), and a transaction left open for hours stops
vacuum from cleaning *any* table, because the old versions might still be visible to it.
`db-long-tx` and `db-idle-in-tx` are the checks that find those; the Advisor tab on a
table shows exactly how many dead rows it takes before autovacuum will act.

**2. Schemas are not users.** `CREATE USER app` does not create a place for `app` to put
tables. Objects land in the first schema in `search_path` - usually `public`. Grant
`USAGE` on the schema *and* privileges on the objects: the two are separate, and a
missing schema `USAGE` is the most common "permission denied" that looks inexplicable.
The Roles view shows both layers, and the privileges editor writes both.

**3. DDL is transactional.** `BEGIN; ALTER TABLE ...; ROLLBACK;` really does undo the
`ALTER`. Migrations can be all-or-nothing, which is a genuine advantage - but it also
means a migration that holds `ACCESS EXCLUSIVE` on a hot table blocks every reader until
it commits. `db-locks` and the Sessions view show who is waiting behind whom.

**4. No hints, and the statistics are the whole story.** If a plan is wrong, the answer
is `ANALYZE`, a better index, an adjusted `default_statistics_target`, or a rewritten
query. The Advisor tab reports when a table was last analyzed; `db-index-advisor` reports
missing indexes, including the foreign keys with no index behind them - which in
PostgreSQL makes deletes on the parent table scan the child.

**5. Connections are processes, not threads.** Each connection is an OS process with its
own memory. Several hundred idle connections cost real RAM and make every snapshot of
`pg_stat_activity` slower. Where Oracle would use shared servers, PostgreSQL uses an
external pooler (PgBouncer). `db-connections` shows how close you are to `max_connections`
and how many of those are idle.

### 13.3 Things with no Oracle equivalent

- **TOAST.** Values wider than about 2 KB are compressed and stored in a side table.
  A table's own size can look small while its TOAST table holds most of the data;
  `db-bloat-breakdown` splits heap, TOAST and indexes for exactly this reason.
- **Extensions.** Much of what is built into Oracle is an installable module here:
  `pg_stat_statements`, `pgcrypto`, `postgis`. See section 5.14.
- **Transaction ID wraparound.** The 32-bit transaction counter has to be frozen
  periodically or the cluster refuses writes. `db-vacuum` reports the age;
  it is the one autovacuum job you must never disable.
- **`NULL` and the empty string are different.** Oracle treats `''` as `NULL`;
  PostgreSQL does not. Data migrated from Oracle often carries this difference silently.

## 14. PostgreSQL for developers

You do not need to become a DBA to keep a PostgreSQL database healthy. You need one
mental model and five habits.

### 14.1 The model: rows are versions

Every `UPDATE` and `DELETE` leaves the old version of the row in place, marked dead.
Readers never block writers because each transaction sees the versions that were current
when it started. Two things follow, and almost everything in this application is about
one of them:

- **Dead rows accumulate** until autovacuum removes them. Until then they take space and
  slow down scans. That is *bloat*.
- **Vacuum can only remove versions no running transaction can still see.** One
  forgotten open transaction freezes cleanup for the whole database.

`ANALYZE` is the other half: it re-samples the data so the planner knows how many rows a
condition will match. Stale statistics are the usual cause of "this query was fast last
week".

### 14.2 When someone says "the database is slow"

1. **Sessions view.** How many connections, how many are actually running, is anything
   blocked. A blocked query is a lock problem, not a performance problem.
2. **`db-health`.** One screen: connections, cache hit, long transactions, bloat, locks.
3. **`db-stat-statements`.** The 25 queries that consumed the most time. Almost always
   the answer is here - usually one query called far more often than anyone expected.
4. **EXPLAIN the query** in the SQL view. Look for `Seq Scan` on a large table, and for
   an estimated row count that is wildly different from the actual one - that second one
   means the statistics are stale.
5. **`db-index-advisor`.** Missing indexes and unindexed foreign keys.

### 14.3 Five habits

1. **Do not leave transactions open.** An ORM session that opened a transaction and went
   to lunch is the single most damaging thing a developer can do to a PostgreSQL
   database. `db-idle-in-tx` finds them.
2. **Index your foreign keys.** PostgreSQL does not create the index for you, and
   without it every delete or update on the parent scans the child table.
3. **`ANALYZE` after bulk loads.** A million rows inserted with no statistics behind
   them will produce bad plans until autovacuum gets around to it.
4. **Batch, but not too big.** A single transaction deleting ten million rows holds
   locks, generates WAL, and leaves ten million dead versions behind. Loop in batches of
   a few thousand and commit.
5. **Use a connection pool.** Opening a connection per request costs a process fork.
   Read `db-connections` before raising `max_connections`.

### 14.4 The words this app uses

- **Bloat** - space held by dead row versions and never returned to the OS.
- **Dead tuple** - an old row version waiting for vacuum. "Tuple" means "row" here.
- **Autovacuum** - the background process that removes dead tuples and refreshes
  statistics. It triggers per table, on thresholds you can see in the Advisor tab.
- **VACUUM vs VACUUM FULL** - plain `VACUUM` marks space reusable and never blocks
  readers. `VACUUM FULL` rewrites the whole table, returns space to the OS, and holds a
  lock that blocks *everything* on that table for the duration. `REINDEX CONCURRENTLY`
  and `pg_repack` are the online alternatives.
- **WAL** - the write-ahead log. Every change is written there first; replication and
  crash recovery both read it.
- **Checkpoint** - the periodic flush of dirty pages to the data files. Too frequent
  means write spikes; `db-checkpoints` reports the ratio of timed to requested ones.
- **search_path** - the list of schemas searched for an unqualified name. It decides
  where `CREATE TABLE x` lands and which function `f()` resolves to.
- **Role** - the single concept behind both users and groups. A role with `LOGIN` is
  what other databases call a user.
- **Sequence gaps are normal.** Sequences are not transactional: a rolled-back insert
  still consumed its number.

## Reporting a bug

**Settings -> Report a bug** builds the report for you: a bug ID, the version,
the commit, your OS, and the last forty lines of the error log. You fill in
three sections - what you did, what you expected, what happened instead - and
press one of three buttons.

**Open mail app** hands the text to your own mail program, addressed to
**dbapguin@gmail.com**. The application sends nothing itself and never has;
you see the exact text first and you press Send. **Copy report** and **Save to
file** are there for when you would rather attach it or paste it somewhere.

**What is removed before anything leaves your machine**: passwords, usernames,
database names, hostnames, connection URIs, IP addresses, and your home
directory path. The dialog lists what it took out, so you can see it rather
than take it on trust.

Quote the bug ID (`DBAP-261011-K4QP` and so on) in any follow-up - it is how
one report is told from another.

## 15. Getting help

- Internal documentation: this file plus `docs/TECHNICAL.md` and `ARCHITECTURE.md`
- Build provenance: **Help -> About DBApguin** shows version, git commit, build timestamp, the Python it was built with. Include this in bug reports.
- Audit log: `<DBCHECKS_HOME>/audit/audit-YYYY-MM.jsonl` is a JSON Lines file. Open with any text editor or `jq` for analysis.
- Error log: `<DBCHECKS_HOME>/dbchecks_error.log` for the most recent server-side tracebacks. Passwords are redacted before write. It rotates at 1 MiB, keeping three older copies, and each run that logs anything writes a banner first - so the entries after the last `=== DBApguin ... started ...` line are the ones from your current session. Entries above it may be weeks old.

For commercial support contact the vendor with:
1. Build version (from Help -> About DBApguin).
2. Provider key (from connection details).
3. PostgreSQL server version.
4. A copy of the relevant audit log section + error log if available.
