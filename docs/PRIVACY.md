# DBApguin - Privacy Notice

**Version 1.0.0 - 2 November 2026**

> The statements in this notice were verified against the source code rather
> than written from intention. Where a claim is checkable, the place to check
> it is named.

---

## The short version

DBApguin sends nothing to the Author. There is no telemetry, no analytics and
no crash reporting. There is no licence check either - the product is free for
everyone, so there is no key to validate and nothing to validate it against,
and DBApguin does not read your computer's identifier. Everything it stores
stays on your machine, in two folders you can open and read (listed below).

The only network connections it makes are the ones you ask for: to your
PostgreSQL servers, and to `127.0.0.1` to talk to its own backend. A connection
you give an SSH tunnel also reaches the SSH server you name there: DBApguin
starts your computer's own `ssh` program for it, which logs in with your keys
and your ssh settings, and forwards a port on `127.0.0.1` only. The one thing
you can set up that makes others is a plugin you install (see Plugins below).

## What runs where

DBApguin is a desktop application in two parts: a window (the Tauri shell on
Windows and Linux, a small Python launcher on macOS) and a local backend that
speaks to PostgreSQL. The backend listens on `127.0.0.1` - loopback only,
not reachable from your network - on port 3236, or on the port you chose in
Settings, under Startup.

Before starting the backend, the window makes one HTTP request to
`/api/app/version` on that address to detect whether a copy is already
running, so a second launch says so instead of starting a competing
backend. It is a hand-written TCP request in the Rust shell
(`shell/src/backend.rs`, `probe_version`), deliberately not an HTTP library;
on macOS the launcher makes the same request. It never leaves your machine.

The window is drawn by the operating system's own web engine: Microsoft Edge
WebView2 on Windows, WebKitGTK on Linux, WebKit on macOS. DBApguin switches off
WebView2's SmartScreen check, so the addresses the window shows are not sent to
Microsoft to be looked up. The engine belongs to the operating system and is
updated with it; what it does on its own is governed by its vendor and your
system settings, not by DBApguin.

The Windows installer makes no network connection either. If WebView2 is
missing it says so and asks whether to continue, rather than downloading it.

## What is stored on your machine

Two folders. The first is DBApguin's data directory:

- **Windows** - `%APPDATA%\DBApguin\`
- **Linux** - `~/.local/share/DBApguin/`, or the directory named by
  `DBCHECKS_HOME`
- **macOS** - `~/Library/Application Support/DBApguin/`

| File | Contents |
|---|---|
| `connections.json` | Saved servers: alias, host, port, database, user, group, colour, and the encrypted password; when you set them, the role, the libpq service name, and the SSH tunnel's host, port, user and key file location (the location only, never the key) |
| `key` | The data key the saved passwords are encrypted with (see Passwords) |
| `certs/` | Root certificates you uploaded for verified TLS connections |
| `app_prefs.json` | Interface preferences: theme, font size, language, external tool paths, and the port the app runs on |
| `window_state.json` | A copy of the window's own storage (below), so that moving the app to another port keeps it |
| `saved_queries.json` | Queries you chose to save, under the names you gave them |
| `sql_workspace.json` | Open SQL tabs, so a restart does not lose your drafts |
| `plugins/` | The plugins you installed - their code, as the plugin file delivered it |
| `plugin-data/` | What each plugin keeps: its settings, and any password it stores, encrypted like connection passwords |
| `audit/audit-YYYY-MM.jsonl` | The audit log (see below) - on unless you switch it off |
| `awr_snapshots/` | Per server: the AWR snapshots you took - statistics and the text of the statements in them - and, while the ASH sampler runs, the sessions it saw at the interval you chose (every second unless you chose otherwise): user, application, client address and the statement each was running. Kept until the retention you set removes them or you delete them under ASH's Stored days or AWR's Stored snapshots |
| `job_results/` | The last 60 reports of the longer analyses (AWR, workload report, session trace and the like), kept so they can be opened again. They contain the statement texts they report on. Also the copies of definitions taken before a change you make through the schema browser (move, drop, owner, tablespace): the SQL that creates those objects - columns, constraints, indexes, policies, privileges, comments, and everything a CASCADE drop takes with them |
| `preimage_rows/` | Only when you tick *Keep the rows too* on a drop: the rows of the tables that drop takes, and the values of a column it takes from a table that stays, as COPY text - your data, as it was. Removed with the copy they belong to, or by deleting the folder |
| `settings_changes/` | Per server: every ALTER SYSTEM made from the application, with the value it replaced and where that came from, so each can be undone. Parameter names and values, nothing else |
| `jobs/` | State of background jobs - backups, restores, long checks. Restoring a plain SQL file that was taken with `--create`, or with --no-tablespaces ticked, makes a copy of it in `jobs/restore-input/` with the lines that switch to another database (or name a tablespace) changed, so the restore stays where you chose; the copy holds what the file holds, rows included, and is removed two days after the restore |
| `todo.json` | The task list |
| `support.json` | How many times the application has started, which decides when the "Support my work" note appears |
| `dbchecks_error.log` | Errors the backend ran into, with passwords removed; rotated at 1 MB, three older copies kept |

The second is the window's own storage, kept by the web engine:

- **Windows** - `%LOCALAPPDATA%\com.dbapguin.desktop\`
- **Linux** - `~/.local/share/com.dbapguin.desktop/`
- **macOS** - `~/.local/share/DBApguin/webview/`

It holds what the interface remembers between runs: the SQL editor's history
of the statements you ran, the open tabs and their layout, favourites and
interface settings, and the engine's own cache. That storage belongs to the
address the window opened, port included, so the window also keeps a copy of
it - not the cache - in `window_state.json` in the data directory, and a window
opened on a new port takes the copy in once.

Each start, the application also makes a random key that only its own window
is given, and the window keeps it as a cookie, so that the interface is served
to that window and to nothing else on the computer. The key is new every time
the application starts and says nothing about you or the computer.

You can delete any of these. The application recreates what it needs.

## Passwords

Saved connection passwords are encrypted with a data key held in your data
directory, using AES via Fernet (`cryptography`). If you set a master
password, that data key is itself wrapped with a key derived from it, so the
stored passwords cannot be read or used to connect until you unlock them.

Two honest limits:

- Without a master password, the data key sits beside the encrypted
  passwords. That protects against casual reading and a stolen backup file,
  not against someone who already has your user account.
- The Author cannot recover a forgotten master password. There is no
  backdoor and no reset - the wrapped key simply cannot be unwrapped.

## The audit log

On unless you switch it off (View → Audit log). Every statement run from the
SQL editor - unless the tab is set read-only - and every change made through
the application's own tools (structure, roles and privileges, maintenance,
replication, server settings, exports, cancelled queries) is appended to a
monthly JSON Lines file in your data directory: the time, the build, the
action, the connection target (host, port, database), your operating-system
user name and the database user, the statement with passwords removed, how
long it took and whether it worked.

It exists so that "who changed this column at 03:00" has an answer - which
needs it to have been on before 03:00. It is a local file, it is never
transmitted anywhere, and you can delete it. Because it keeps the statements
you run, it keeps what they contain: a value typed into an UPDATE is in the
log.

Each record is chained to the one before it by a hash, and Verify chain in the
log window says whether the chain is intact. That catches a casual edit or a
deleted line; it does not stop someone with write access to the file, who can
recompute the whole chain. It records only what this application did, so it
is a convenience, not an authoritative compliance record - [the licence
agreement](EULA.md) sets out what it is and is not fit for.

## Plugins

A plugin is program code that you choose to install. **Anyone can create
one**, and DBApguin does not check who did - so use only plugins from
publishers you trust. Once installed, a plugin runs inside DBApguin with your
rights: it can read the databases you connect to and can connect to other
services it is set up to use, sending them whatever it is written to send.
That traffic is the plugin's, not DBApguin's, and its publisher's privacy terms
are the ones that apply to it.

A plugin's settings and stored passwords stay in `plugin-data/` on your
machine. Removing a plugin deletes them with it. If you write plugins
yourself, the folder you develop them in is wherever you chose; DBApguin
neither copies it nor includes it in a backup.

What a plugin does with your data is the plugin's to say: its Info, in the
Plugins tab, says what it reads and where it sends it. A plugin certified by
the Author can come with terms of its own, which then apply to that plugin in
addition to this notice.

## There is no licence key

DBApguin is free for everyone. There is no key to enter, no activation, and no
machine identifier to send anyone - DBApguin does not read your computer's
identifier at all.

## Query results and database contents

Your data never leaves the path between your machine and your database
server. The rows a query returns are held in memory to display them and
written to disk only when you export them - to CSV, to a `.sql` file, or to a
backup you named. Exporting ASH or AWR data writes a zip, to the place you
choose, with the same statement texts and session details the files above hold. What does reach the disk on its own, all on your machine
and all listed above: the statements you run (the editor's history, the open
SQL tabs, the audit log), the reports of the longer analyses with the
statement texts in them, and the AWR snapshots and ASH samples you take.

## The server log

When you open **Plan** on a statement in the ASH, AWR or workload report,
DBApguin looks for the plan the statement actually ran with. One place that
plan can be is your PostgreSQL server's own log, where auto_explain writes it.
DBApguin then reads that log **through the database connection you already
have** (`pg_ls_logdir()` and `pg_read_file()`), only while you have the plan
open, and only a bounded part of it: the log files that cover the report's
time window, read from the end, at most 32 MB and 8 seconds per look
(`src/checks/actual_plans.py`, `read_log_plans`). The Error watcher check
reads the last 64 KB of the current log the same way.

What is read is the server's log, so it can contain the text of statements
and the values of their parameters. It is used to find and draw the plan,
held in memory for as long as the plan is on screen, and not written to disk
and not sent anywhere: nothing leaves your computer. Whether DBApguin can read
the log at all is decided by your server - it needs a role that may list the
log directory and read files there - and on most managed cloud services it
cannot.

## Analytics

There are none. No usage counters, no feature telemetry, no error reporting
to the Author. If something breaks, the Author learns about it when you say
so.

## Your rights

The Author holds **no personal data about you at all**. There is no purchase,
no account and no registration, so there is nothing to request access to,
correct, export or erase.

Everything the application stores is on your own machine, in files listed
above. You can read them, copy them or delete them without asking anyone.

## Changes

If this notice changes materially, the new version ships with the release
that changes the behaviour, and the version line at the top changes with it.

## Contact

DBApguin is created by Darko Lovrenčec. There is nothing to buy, so there is no
billing contact - questions about this notice go to the Author at
**dbapguin@gmail.com**.
