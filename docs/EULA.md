# DBApguin - End User Licence Agreement

**Version 1.0.0 - 2 November 2026**

---

This English text is a translation of the Croatian original, made with the help
of AI and provided for convenience. Where the two differ, the Croatian text
governs (section 18).

The version of this Agreement is the version of DBApguin in which it last
changed.

**DBApguin is free of charge for everyone**, for private use and for commercial use alike. There is no licence
key, no activation, no trial period, and no feature held back.

This Agreement is short because nothing is bought: there is no purchase, no
machine count and no licence term to settle.

## 1. Agreement

This End User Licence Agreement ("Agreement") is between you (the "Licensee",
whether an individual or a legal entity) and Darko Lovrenčec, the author of
DBApguin (the "Author").

The Software presents this Agreement when it first runs and does not open until
you accept it. Accepting it there, or otherwise installing or using DBApguin
(the "Software"), means you accept this Agreement. If you do not accept it,
decline at that prompt and do not use the Software.

Your acceptance is recorded locally - the version accepted and the date.
Nothing about it is transmitted anywhere. Section 16 covers what happens when a
later version revises this Agreement.

## 2. What is licensed

The Software is **licensed, not sold**, and it is licensed **at no cost**. The
Author retains all right, title and interest in the Software, including its
source code, design and documentation.

Free of charge is not the same as public domain, and it is not an open-source
licence. You may use the Software without paying for it. You may not sell it,
relicense it, or present it as your own work. Section 4 is the whole of that
list.

DBApguin is published to help people in their everyday database work. Everything
it offers - its checks, analyses, recommendations, warnings, confirmations,
estimates and copies taken before a change - is offered on a best-effort basis,
not as a guarantee (section 10).

There is no purchase, so there is no invoice, no reseller, no refund, and no
subscription to cancel.

This Agreement covers DBApguin 1.0.x: every release whose version number
begins with 1.0.

## 3. Licence grant

The Author grants you a non-exclusive, worldwide, royalty-free, perpetual
licence to install and use the Software:

- on **as many machines as you like**, personally or within your organisation;
- for **any purpose**, including commercial and production work;
- with **no limit** on saved connections, open tabs, databases, users or
  features.

You may redistribute the Software **unmodified and complete**, with this
Agreement included, provided you charge nothing for it and do not present it as
your own. Mirroring the official download is welcome.

## 4. What you may not do

- Sell the Software, charge for access to it, or include it in a paid product
  or paid service offering without the Author's written permission.
- Relicense it, sublicense it, or distribute it under different terms.
- Remove, alter or obscure the authorship, copyright or licence notices, or
  present the Software as created by anyone other than the Author.
- Distribute a modified build as DBApguin, or in a way that suggests the Author
  produced or endorsed it.
- Decompile, disassemble or otherwise reverse engineer the Software in order to
  produce a competing product, to strip its authorship, or to redistribute what
  you recover.

That last point is about circumventing the licence, not about understanding the
Software. Where Directive 2009/24/EC Article 6 or equivalent mandatory law
gives you the right to decompile for **interoperability** with another program,
that right stands and nothing here limits it.

Using the Software in the course of paid work you perform for a client is
ordinary commercial use and is expressly permitted. Charging a client for
DBApguin itself is not.

## 5. Your data and your databases

The Software runs entirely on your machine and connects only to the database
servers you configure, and to the addresses that plugins you install are set
up to reach (section 15). **It transmits nothing to the Author** - there is no
telemetry, no usage reporting, and no licence check over the network. See
[the privacy notice](PRIVACY.md) for what it stores locally.

Your databases, your schemas and your query results are yours. The Author
neither receives nor has any access to them.

## 6. The Software executes statements against your databases

This is the part of this Agreement worth reading twice.

DBApguin is a database administration tool. It issues SQL - including DDL such
as CREATE, ALTER, DROP, REINDEX and VACUUM, and maintenance operations that can
lock tables or run for a long time - against the servers you point it at. It
tries to show you what it is about to run and to ask you to confirm destructive
operations, but **it may not recognise every statement that changes, locks or
removes something**: its warnings and confirmations are an aid offered on a
best-effort basis, not a guarantee. **You decide what runs, and against which
server**.

You are responsible for:

- choosing the right connection, and knowing whether it is production;
- having a **working, tested backup** before you change anything you cannot
  afford to lose;
- the consequences of statements you confirm, including data loss, downtime,
  lock contention and replication effects.

The Software is a tool in your hands, not a safeguard against your intentions.
Nothing in it removes the need to know what a statement will do before you run
it.

## 7. The audit log

The Software keeps an audit log unless you switch it off (View, Audit log). It
appends a line to a local file for each statement run from the SQL editor -
unless the tab is set read-only - and for each change made through the
Software's own tools: the time, the target, your operating-system user name and
the database user, the statement with passwords redacted, and the outcome.
[The privacy notice](PRIVACY.md) says where the file lives and what else it
records.

It exists so that "who changed this, and when" has an answer you can look up.
**It is a convenience for you, not a certified compliance record**: it is a
local file; its records are chained by a hash, which shows a casual edit or a
deleted line, but anyone who can reach the file can rewrite it and the chain
with it; and it records only what the Software itself did.

Do not rely on it as forensic evidence, as the sole record for a regulatory or
contractual obligation, or as proof of what someone else did. Where you need an
authoritative record, use your database server's own logging and your
organisation's controls.

## 8. Updates and changes to the Software

The Author may release updates - corrections, improvements and new features -
and does not undertake to release them on any schedule, or at all. An update is
part of the Software and is governed by this Agreement. Where an update ships
with a revised version of this Agreement, section 16 applies and the Software
asks for your consent before the update takes effect.

The Software is under active development, not a fixed deliverable. Between
versions the Author may add, change, deprecate or remove features, change which
third-party components it uses (section 13), and may at any time stop
developing, supporting or distributing the Software or any part of it. None of
this is a breach of this Agreement.

The Author does not reach into a version you are already running to remove a
feature from it, and the Software carries no remote kill switch. A copy you
have installed keeps working on its own terms whatever the Author does next.
Discontinuation stops new versions and new support; it does not reach a copy
already on your machine.

## 9. Your indemnity to the Author

Section 11 is about responsibility as between you and the Author. This section
is about a third party.

You will defend, indemnify and hold the Author harmless against any claim,
demand, loss, liability, damage, cost or expense (including reasonable legal
fees) that a third party brings and that arises out of:

1. your use of the Software, including any statement it issued at your
   direction against a database (section 6);
2. your breach of this Agreement;
3. your infringement or misappropriation of a third party's rights; or
4. data you handled, exposed, altered or destroyed using the Software.

The Author will notify you promptly of any such claim, let you control its
defence and settlement - except that a settlement which admits fault by the
Author or imposes any obligation on the Author needs the Author's prior written
consent - and give you reasonable cooperation at your expense.

This section does not apply to a claim to the extent it is caused by the
Author's own breach of this Agreement. If you are a consumer, this section
binds you only so far as such an indemnity is enforceable against a consumer
under the mandatory law where you live.

## 10. No warranty

**The Software is provided "as is", without warranty of any kind**, express or
implied, including but not limited to the implied warranties of
merchantability, fitness for a particular purpose, and non-infringement.

The Author does not warrant that the Software is free of defects, that it will
run without interruption, that its analyses and recommendations are correct for
your workload, or that it is suitable for any particular database, version or
configuration.

What this Agreement, the user guide and the Software say about what the
Software does - that it warns before a destructive statement, that it asks
before it runs one, that a copy taken before a change can put things back, that
an estimate, a finding or a recommendation is right - describes what it is
designed to do, to the best of the Author's effort. It is not a promise that it
always does so, on every server, version and workload. Check what matters
before you rely on it.

Where mandatory consumer law in your country of residence grants you rights
that cannot be excluded, those rights are unaffected by this section.

## 11. Limitation of liability

To the maximum extent permitted by law, the Author is **not liable** for any
indirect, incidental, special, consequential or punitive damages, nor for any
loss of data, loss of profit, loss of revenue, business interruption, or cost
of substitute software or services, arising out of or related to the Software or
this Agreement, however caused and on any theory of liability.

Because the Software is supplied free of charge, no sum has been paid for it,
and the Author's total aggregate liability under this Agreement is therefore
**limited to zero** - except where liability cannot lawfully be limited,
notably for death or personal injury caused by negligence, for fraud, and for
anything else mandatory law places outside the reach of an agreement.

## 12. Term and termination

This licence takes effect when you accept this Agreement and continues
indefinitely.

It terminates automatically if you breach Section 4. On termination you must
stop using the Software and remove it from your machines.
Sections 5, 6, 7, 9, 10, 11, 17 and 18 survive termination.

Discontinuation is covered by section 8: it does not terminate your licence
to a copy you already hold.

## 13. Third-party components

The Software includes third-party open-source components, each under its own
licence. Those licences govern those components and are reproduced in
[the third-party notices](THIRD_PARTY.md) (Help,
Third-party components), with the full licence texts beside them. Nothing in this Agreement
restricts rights granted to you by those licences.

## 14. Third-party services

The Software integrates no third-party online services of its own: it has no
account system, no cloud component, no analytics or telemetry provider, and
no licensing server. It connects only to the database servers you configure
and to `127.0.0.1` for its own backend, as [the privacy notice](PRIVACY.md)
describes. The one exception is one you create yourself: a plugin you install
may connect to other services you set it up to use - that is the plugin's
doing and is covered by section 15.

The Software does contain links to third parties' documentation - a database
hosting provider's parameter reference, for example. Following one opens that
page in your own browser, and what happens there is between you and that third
party, under their terms, not this Agreement.

Where you configure a cloud-hosted database, your relationship with that
provider is yours alone and their terms apply to it. The Author is not a party
to it.

## 15. Plugins

The Software can be extended with plugins - Python and JavaScript that run
inside it, installed through Plugins, Manage plugins, or written with the
plugin builder the Software includes.

**Anyone can create a plugin**, and nothing checks who did. The Author
therefore recommends using **only plugins from publishers you trust**.

- **A plugin is not part of the Software.** A plugin written by someone else
  is that person's work and comes on that person's terms. The Author does not
  review, sign, endorse or support it. Plugins the Author publishes are offered
  on the terms of this Agreement.
- **A plugin runs with your rights.** It runs inside the Software as you, can
  reach every database you connect to, and can connect to any other address it
  is set up to reach. Sections 10 and 11 apply to whatever a plugin does as
  they apply to the Software, and the Author is not responsible for a plugin
  written by anyone else.
- **A service a plugin reaches is that service's.** When a plugin sends
  something to a service outside the Software - a monitoring server, a web
  site, a language model - you choose to send it, and the service's terms
  govern what it receives and does with it. The Author is not a party to those
  terms and receives nothing. What a plugin brings back from such a service,
  an analysis or a suggested statement among it, is information to check, not
  an instruction.
- **Writing plugins is permitted use.** Using the plugin interface the
  Software documents - the plugin guide, the plugin builder and the functions a
  plugin is given - is use of the Software, not modification or reverse
  engineering of it under section 4.
- **Your plugins are yours.** You own the plugins you write and may share or
  distribute them on terms of your choosing, provided a plugin contains no part
  of the Software itself and is not presented as made or endorsed by the
  Author.

## 16. Amendments

A later version of the Software may carry a revised Agreement. When that
happens, the Software asks you to accept the new version before it opens, and
tells you which version it is asking about. Continuing to use an **existing**
installation never silently changes the terms you accepted.

If you do not accept a revised Agreement, you may keep using the version you
already accepted, under the terms you accepted. You are not obliged to upgrade.

## 17. Governing law

This Agreement is governed by the laws of the Republic of Croatia, without
regard to conflict-of-law rules. The courts of Croatia have exclusive
jurisdiction, except where mandatory law gives you the right to bring
proceedings in the courts of your own country of residence.

## 18. Language

This Agreement is written in Croatian. The English text and any other
translation are provided for convenience only; in the event of any
inconsistency, **the Croatian text governs**. The Software shows both, in
Help, Licence agreement.

## 19. Entire agreement

This Agreement is the entire agreement between you and the Author concerning
the Software, and supersedes any earlier version of this Agreement. If any
provision is held unenforceable, the rest remains in force.

## 20. Contact

DBApguin is created by **Darko Lovrenčec**. Questions about the Software, bug
reports and this Agreement go to the Author at **dbapguin@gmail.com**.

There is nothing to buy, so there is no billing contact.
