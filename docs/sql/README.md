# The SQL DBApguin runs

Every SQL statement DBApguin can send to a PostgreSQL server, read out of the application's source. Each one shows the file, the function and the line it lives in. Parts written as `{expression}` are filled in when it runs - a quoted name, a branch for the server's version, a limit. The exact text a server received is in its `pg_stat_statements` and log, and in DBApguin's own audit log for what you ran.

Nothing here is secret: a DBA should be able to see what a tool runs on their server. The user guide explains what each check is for and how to read it.

| Part of the app | Statements |
|---|---|
| [Checks](checks.md) | 408 |
| [Checks (catalogue)](checks-catalogue.md) | 18 |
| [Checks (catalogue)](checks-catalogue.md) | 18 |
| [Sessions and live load](sessions-and-live-load.md) | 50 |
| [SQL editor](sql-editor.md) | 60 |
| [Schema browser](schema-browser.md) | 349 |
| [Database tools](database-tools.md) | 527 |
| [The rest of the app](the-rest-of-the-app.md) | 785 |

**2197 statements in all.**

## Licence

These SQL statements are part of DBApguin, (c) Darko Lovrenčec. They are published
under the PostgreSQL Licence:

Permission to use, copy, modify, and distribute this software and its documentation
for any purpose, without fee, and without a written agreement is hereby granted,
provided that the above copyright notice and this paragraph and the following two
paragraphs appear in all copies.

IN NO EVENT SHALL THE AUTHOR BE LIABLE TO ANY PARTY FOR DIRECT, INDIRECT, SPECIAL,
INCIDENTAL, OR CONSEQUENTIAL DAMAGES, INCLUDING LOST PROFITS, ARISING OUT OF THE USE
OF THIS SOFTWARE AND ITS DOCUMENTATION, EVEN IF THE AUTHOR HAS BEEN ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.

THE AUTHOR SPECIFICALLY DISCLAIMS ANY WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE. THE
SOFTWARE PROVIDED HEREUNDER IS ON AN "AS IS" BASIS, AND THE AUTHOR HAS NO OBLIGATIONS
TO PROVIDE MAINTENANCE, SUPPORT, UPDATES, ENHANCEMENTS, OR MODIFICATIONS.

This licence covers the SQL text on these pages only. The DBApguin application itself
is licensed under its own agreement (EULA.md).
