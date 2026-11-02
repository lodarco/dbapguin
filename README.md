<p align="center">
  <img src="docs/logo.png" alt="" width="104">
</p>

<h1 align="center">DBApguin</h1>

<p align="center">
  A desktop PostgreSQL DBA console.<br>
  Free for everyone - no licence key, no trial, no account, nothing held back.
</p>

<p align="center">
  <a href="https://github.com/lodarco/dbapguin/releases/latest"><b>Download</b></a> ·
  <a href="https://lodarco.github.io/dbapguin/">Website</a> ·
  <a href="docs/USER_GUIDE.en.md">User guide</a> ·
  <a href="https://github.com/lodarco/dbapguin/issues">Report a problem</a>
</p>

---

## What it is

One window for the work a PostgreSQL DBA actually does: look at what the
server is doing right now, find out why it is slow, and move schemas and data
around without writing the same `pg_dump` incantation for the hundredth time.

It is a desktop application, not a web service. It runs on your machine and
talks only to the database servers you point it at. There is no cloud, no
telemetry and no account.

## What is inside

**Watching a live server.** Sessions and the load they are making, blocking
pairs and what is waiting on what, long transactions, idle-in-transaction,
connection limits, temporary-file spills, checkpoints, replication lag and
replication slots.

**Finding out why it is slow.** Index advisor, cache advisor, autovacuum
advisor, setup advisor, bloat estimates for tables and indexes, duplicate and
unused indexes, query history, workload reports and AWR-style snapshots you
can compare against each other.

**Moving things.** Clone a table or a whole schema, merge one schema into
another, copy a table across databases or across servers, compare two schemas
or two databases and get a reviewable script, back up and restore.

**Everyday work.** A SQL editor with EXPLAIN, the object tree, privileges and
roles, column and sequence editors, maintenance operations with the options
the connected server actually supports.

**Extending it.** Plugins add checks, menu entries and tabs without a new
version of DBApguin, and the app includes a builder for writing your own.
Anyone can make one and nothing checks who did, so install only plugins from
publishers you trust.

Every destructive action shows you the statement before it runs.

## PostgreSQL versions

Tested against **PostgreSQL 13 and newer**.

It connects to older servers too, and the rule is *do what the version allows,
do not refuse*: a check that can leave out a column the server lacks does
that and says so in a short note, rather than failing. It is exercised against
real 9.4, 9.6, 10, 11, 12, 13, 14 and 18 servers on every release round. Below
9.4 the catalogues have diverged too far and the app does not claim to work.

PostgreSQL 19 is measured too - on 19rc1, the release candidate, since 19.0 is
not out yet: every check runs on it, and
**Tuning & diagnostics → Ready for PostgreSQL 19?** goes through 19's list of
incompatibilities against your server before you upgrade.

## Install

| | |
|---|---|
| **Linux** | `DBApguin-x86_64.AppImage` - `chmod +x`, run it. Or `dbapguin_<version>_amd64.deb` for a menu entry. |
| **Windows** | `DBApguin-<version>-setup.exe`, or the portable zip if you would rather not install. |
| **macOS** | `DBApguin-<version>-macOS-x86_64.dmg`, drag to Applications. Intel build. |

Everything is on the [releases page](https://github.com/lodarco/dbapguin/releases/latest),
with a `SHA256SUMS` beside it:

```
sha256sum -c SHA256SUMS
```

Neither the Windows nor the macOS build is code-signed. Windows SmartScreen
will warn on first run ("More info" then "Run anyway"). On macOS, open it the
first time from System Settings → Privacy & Security → "Open Anyway" (older
macOS: right-click then Open). If macOS says the app "is damaged and can't be
opened", run once in Terminal:
`xattr -dr com.apple.quarantine /Applications/DBApguin.app`

## Documentation

The full guide ships inside the app - **Help → User guide** - so it always
describes the build you are running. The same text, rendered here:

[English](docs/USER_GUIDE.en.md) ·
[Hrvatski](docs/USER_GUIDE.hr.md) ·
[Deutsch](docs/USER_GUIDE.de.md) ·
[Español](docs/USER_GUIDE.es.md)

## Getting in touch

Bugs and questions: [open an issue](https://github.com/lodarco/dbapguin/issues),
or write to **dbapguin@gmail.com**. The app can assemble a report for you -
**Settings → Report a bug** fills in the version, the commit and the last lines of
the error log, redacts passwords, hostnames and IP addresses, shows you the
exact text, and leaves sending it to you.

## Terms

DBApguin 1.0.x is free for private and commercial use alike - no licence key,
no activation, no trial that runs out, and no feature held back. Use it
freely, do not sell it, and do not present it as your own work. The full text:
[licence agreement](docs/EULA.hr.md) (written in Croatian - the Croatian text governs) ·
[English translation](docs/EULA.md) ·
[privacy notice](docs/PRIVACY.md) ·
[third-party components](docs/THIRD_PARTY.md)

Nothing is sent anywhere. The app has no telemetry, no update check and no
network traffic beyond the database connections you configure. The one
exception is a plugin you choose to install: its traffic is its own.

## Source, and the SQL it runs

This repository carries the releases and the documentation. The application's
source is not published.

**The SQL it sends to your servers is.** Every statement DBApguin can run is
listed in [docs/sql](docs/sql/README.md), with where it lives in the app, read
out of the source by a script rather than written by hand. It is published under
the PostgreSQL Licence, so you can read it, check it and reuse it. None of it is
secret anyway: it is in your server's `pg_stat_statements` and log the moment it
runs, and the [user guide](docs/USER_GUIDE.en.md) explains what each check is
for.

## How DBApguin is made

DBApguin started as a personal project: a tool to make everyday database work
easier, for experienced DBAs and just as much for people who know less about
databases. It is free of charge.

It is written with the help of AI coding assistants. Every change is designed,
reviewed and tested by me - automated tests against PostgreSQL 9.4 to 19 (19rc1),
and by hand in the app itself. Testing goes on with every release, on a
best-effort basis: DBApguin is provided as is, without warranty. Bug reports
are welcome.

## Credits

DBApguin is built on open-source components, each listed with its licence in
the [third-party components](docs/THIRD_PARTY.md), with the full licence texts
shipped in the app. Where a query follows someone else's method - the bloat
estimate follows check_postgres - that is credited there too.

## Supporting it

DBApguin is written in the evenings and at weekends, on hardware and test
databases paid for privately. It is free and fully unlocked for everyone.

---

<p align="center"><sub>Created by Darko Lovrenčec</sub></p>
