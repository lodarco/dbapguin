# DBApguin - Benutzerhandbuch

**Für DBApguin 1.0.0**

*Dieses Handbuch wurde mit Hilfe von KI aus dem Englischen übersetzt. Wo etwas unklar ist oder abweicht, gilt das englische Original (Hilfe → Benutzerhandbuch mit Englisch als Oberflächensprache).*

DBApguin ist eine Desktop-Konsole für PostgreSQL-DBA-Arbeit. Dieses Handbuch
beschreibt jede Funktion aus Sicht der Anwenderin und wird mit der Anwendung
ausgeliefert, beschreibt also immer genau den Build, den Sie vor sich haben.

Jederzeit zu öffnen über **Hilfe → Benutzerhandbuch**.

## 1. Installation

### Linux

1. Laden Sie `DBApguin-<version>-x86_64.AppImage` herunter.
2. Ausführbar machen: `chmod +x DBApguin-*.AppImage`
3. Doppelklick oder Start aus einem Terminal.

Das AppImage ist eigenständig (~27 MB) und legt seine Daten unter `~/.local/share/DBApguin/` ab.

Erscheint das eingebettete Fenster nicht, zeigt das AppImage einen Dialog mit Installationsanweisungen für jeden gängigen Paketmanager (apt, dnf, pacman, zypper, apk, emerge). Der Dialog wählt selbst den Reiter Ihrer Distribution und bietet „Befehl kopieren" mit einem Klick. Für Debian/Ubuntu lautet der Befehl:

```
sudo apt-get install python3-gi python3-gi-cairo gir1.2-gtk-3.0 gir1.2-webkit2-4.1
```

DBApguin ist eine eigenständige Desktop-Anwendung: Fehlt WebKit2GTK, öffnet sie NICHT stillschweigend einen Tab im Webbrowser. Installieren Sie das Paket und starten Sie das AppImage erneut.

### Windows

1. Entpacken Sie das DBApguin-Zip an einen beliebigen Ort (Desktop, `C:\Tools\` und so weiter).
2. Öffnen Sie den Ordner.
3. Doppelklick auf `DBApguin.exe`.

Beim ersten Start kann SmartScreen erscheinen („Windows protected your PC"). Klicken Sie „More info" und dann „Run anyway". DBApguin ist für Windows noch nicht signiert.

Fehlt die Microsoft Edge WebView2 Runtime, öffnet die Anwendung einen Dialog mit drei Installationswegen: (1) den mitgelieferten `MicrosoftEdgeWebView2Setup.exe` aus dem Anwendungsordner starten, (2) den Online-Bootstrapper von `developer.microsoft.com/microsoft-edge/webview2/` laden oder (3) den eigenständigen Installer über GPO / SCCM / Intune verteilen. Jeder Block hat eine Schaltfläche „Kopieren" für Pfade und Links.

Die Anwendung installiert sich nicht ins System; sie läuft aus ihrem Ordner. Zum Entfernen löschen Sie den Ordner.

Der Setup-Dialog liest seine Sprache aus Ihren DBApguin-Einstellungen (`app_prefs.json` -> `language`); beim ersten Start folgt er der Systemsprache (Kroatisch bei `hr_*`, sonst Englisch).

## 1.4 Unterstützte PostgreSQL-Versionen

DBApguin ist auf **PostgreSQL 13 und neuer** ausgerichtet. Gegen diese Version wird getestet, und für sie ist jede Prüfung geschrieben.

Es läuft auch gegen ältere Server, und die Regel lautet: **tun, was die Version erlaubt, nicht verweigern**. Eine Prüfung, die eine dem Server fehlende Spalte weglassen oder einen Abschnitt überspringen kann, der eine nicht vorhandene Sicht braucht, tut genau das und schreibt einen kurzen Hinweis, was ausgelassen wurde. Nur eine Prüfung, deren *ganzes* Thema eine Funktion ist, die es nicht gibt - logische Replikation, die in PostgreSQL 10 komplett kam -, zeigt eine Karte "Braucht PostgreSQL N", statt zu laufen. Konkret, auf einem Server so alt wie **9.4**: die Sitzungsliste, Live Load und die Blockierungsansicht funktionieren alle - die Blockierungsansicht fällt auf einen pg_locks-Self-Join zurück, wo `pg_blocking_pids()` (9.6) fehlt, und die Sitzungsliste zeigt das `waiting`-Flag, wo Wait-Events (9.6) fehlen. `Blocking pairs`, `Long transactions`, `Connection limits`, `Autovacuum activity`, `Replication`, `Table sizes`, `Replication slots`, die Bloat-Schätzungen, Replica Identity und die Health-Übersicht bauen alle eine versionsgerechte Abfrage und weisen darauf hin, was gröber ist. Deklarative Partitionierung ist 10+, also zeigt der Schema-Baum auf einem älteren Server Vererbungskinder als Tabellen unter der Tabelle, von der sie erben, nie als Partitionen.

**PostgreSQL 10, 11 und 12** sind voll verbindbar und das meiste der App funktioniert unverändert; ein paar Dashboard-Panels, die neuere Statistik-Sichten lesen, bleiben still leer.

Unter **PostgreSQL 10** sind Sie jenseits des Getesteten. Verbinden geht, SQL-Editor / Schema-Browser / Backup-Werkzeuge sind in Ordnung, und die Prüfungen, die sich anpassen können, tun das; aber das ist Best-Effort, und der Hinweis beim Verbinden sagt das. Unter **9.4** haben sich die Kataloge weit genug entfernt, dass DBApguin gar nicht behauptet zu funktionieren.

Ein rohes `column "..." does not exist` aus einer Prüfung sehen Sie nie: liegt die Ursache darin, dass der Server älter ist als die Prüfung erwartet, sagt die Prüfung das in Worten.

### PostgreSQL 19

DBApguin ist auch gegen PostgreSQL 19 gemessen: jede Prüfung läuft darauf,
mit und ohne die Contrib-Erweiterungen, als Superuser und als normale Rolle.
Eine Beta oder ein Release Candidate sagt das beim Verbinden, weil sich
Details bis zur endgültigen Version noch ändern können. Eine Hauptversion
neuer als 19 verbindet sich und läuft, mit dem Hinweis, dass sie neuer ist
als alles Gemessene.

Auf 19 nutzt die Anwendung, was die Version hinzugefügt hat:

- **REPACK** im Wartungsbereich und in den Bloat-Empfehlungen. Es schreibt
  eine Tabelle neu und gibt den Platz zurück, mit CONCURRENTLY, während
  Lesen und Schreiben weiterlaufen. Ältere Server behalten VACUUM FULL und
  CLUSTER.
- **Die Autovacuum-Warteschlange des Servers** im Vacuum-Status: welche
  Tabellen Autovacuum als Nächstes nimmt und warum, auch zur
  Wraparound-Vermeidung.
- **Sperrstatistik** unter Locks: Wartevorgänge je Sperrtyp seit dem letzten
  Reset und wie oft die Fast-Path-Slots erschöpft waren.
- **Wie ein Vacuum gestartet wurde und wie hart es arbeitet** (aggressive,
  failsafe, zur Wraparound-Vermeidung), in der Live-Last und im
  Vacuum-Advisor.
- **Reset-Zeiten je Index** bei Ungenutzten Indizes, damit ein gerade
  zurückgesetzter Index nicht ungenutzt heißt.
- **Anzahl der Ausführungen mit generischem und individuellem Plan** je
  Anweisung in Top-Abfragen.
- **EXPLAIN ANALYZE mit I/O**: die Anfragen eines Scans, seine Wartezeiten
  und wie weit er vorausgelesen hat.
- **Replikation**: Fehler und Konflikte je Abonnement, wie oft logisches
  Dekodieren seinen Speicher überschritt, und der Recovery-Zustand eines
  Standbys.
- **Logische Slots bei wal_level = replica**: 19 hebt die WAL-Stufe selbst
  an, daher erlaubt der Slot-Dialog sie und sagt, was das kostet.
- **Constraints abschalten**: der Constraint-Manager kann Fremdschlüssel (ab
  18) und Checks (ab 19) auf NOT ENFORCED setzen, statt sie zu löschen.
- **Einstellungsempfehlungen nach den Standards von 19**: JIT aus, anders
  bemessene Sperrtabelle, Protokollierung von Sperrwartezeiten an, Warnungen
  bei MD5-Anmeldungen, RADIUS aus pg_hba.conf entfernt.

Eine Sicherung prüft zuerst, dass das pg_dump auf diesem Rechner nicht älter
ist als der Server - der übliche Zustand eines Rechners am Tag, an dem eine
neue Hauptversion erscheint - und nennt das zu installierende Client-Paket.
Vor dem Upgrade **Bereit für PostgreSQL 19?** ausführen (siehe
[die Bereitschaftsprüfung](#ready-for-postgresql-19)).

## 1.5 Voraussetzungen auf dem Server



DBApguin arbeitet mit einem gewöhnlichen PostgreSQL-Cluster ohne besondere Vorbereitung. Einige fortgeschrittene Funktionen setzen Erweiterungen oder GUCs auf dem Server voraus. Am schnellsten finden Sie über den eingebauten Setup-Berater heraus, was fehlt:

> Oberes Menü -> Cluster-Zustand -> „DBApguin Setup-Berater"

Er prüft den Cluster und gibt je Funktion einen Status aus (OK / TEILWEISE / FEHLT) sowie genau das SQL oder die Zeile in postgresql.conf, mit der Sie das Fehlende aktivieren. In Managed Clouds (RDS, Aurora, Cloud SQL, Azure, Supabase, Neon) verweist der Hinweis auf die Parametergruppe des Anbieters, weil ALTER SYSTEM dort gesperrt ist.

Kurzübersicht, nach Wirkung sortiert:

| Was Sie nutzen möchten | Voraussetzung auf dem Server | Wie aktivieren |
|--|--|--|
| Teuerste Abfragen, Abfragehistorie, AWR-artige Berichte | Erweiterung pg_stat_statements über `shared_preload_libraries` geladen | `pg_stat_statements` zu `shared_preload_libraries` hinzufügen + Neustart, danach `CREATE EXTENSION pg_stat_statements` in jeder gewünschten Datenbank |
| Planerfassung beim Tracing | Erweiterung auto_explain + `log_min_duration` >= 0 | `auto_explain` zu `shared_preload_libraries` hinzufügen + Neustart, dann `SET auto_explain.log_min_duration = 500` (oder 0 für alles) |
| Query_id in pg_stat_activity, ASH-Korrelation mit pg_stat_statements | `compute_query_id = on` | `ALTER SYSTEM SET compute_query_id = on; SELECT pg_reload_conf();` (PG14+) |
| Realistische I/O-Zeit in pg_stat_statements / EXPLAIN (BUFFERS) | `track_io_timing = on` | `ALTER SYSTEM SET track_io_timing = on; SELECT pg_reload_conf();` |
| Vollständiger Abfragetext in pg_stat_activity und ASH | `track_activity_query_size >= 4096` | `ALTER SYSTEM SET track_activity_query_size = 4096;` danach NEUSTART |
| Vollständiges Top SQL in AWR (keine zwischen Snapshots verdrängten Anweisungen) | `pg_stat_statements.max` groß genug für die verschiedenen Anweisungen | `ALTER SYSTEM SET pg_stat_statements.max = 10000;` dann NEUSTART - der AWR-Vergleich sagt, wann Anweisungen verdrängt wurden |
| Statistik zu Funktionsaufrufen | `track_functions = 'pl'` | `ALTER SYSTEM SET track_functions = 'pl'; SELECT pg_reload_conf();` |
| Puffer-Cache-Snapshot in db-cache-advisor | Erweiterung pg_buffercache | `CREATE EXTENSION pg_buffercache;` (Superuser) |
| Online-Neuschreiben von Tabellen (Wartung -> pg_repack) | Erweiterung pg_repack + Client-Programm | pg_repack-Paket auf dem Server und auf dem DBApguin-Rechner installieren, dann `CREATE EXTENSION pg_repack;` |
| Server-Log-Ende in db-errors | `logging_collector = on`, `log_destination` enthält stderr, `pg_read_server_files` gewährt | Nur selbst betriebene Server. Managed Clouds sperren pg_read_file. |
| Details zu Sperrwartezeiten im Server-Log | `log_lock_waits = on`, `deadlock_timeout = '1s'` | `ALTER SYSTEM SET log_lock_waits = on;` |
| Log langsamer Abfragen | `log_min_duration_statement >= 0` | `ALTER SYSTEM SET log_min_duration_statement = '1s';` (oder `'100ms'` strenger) |
| Server-seitiges Audit-Log (ergänzend zum Client-Audit) | Erweiterung pgaudit oder `log_statement = 'ddl'` | `pgaudit` zu `shared_preload_libraries` hinzufügen, Neustart, `CREATE EXTENSION pgaudit;` |
| Prüfungen auch für Nicht-Superuser | Rolle `pg_monitor` gewähren | `GRANT pg_monitor TO <Rolle>;` (PG10+) |

Wenn Sie nur eines aktivieren, aktivieren Sie `pg_stat_statements`. Die meisten Funktionen von DBApguin werden deutlich nützlicher, sobald er geladen ist.

## 2. Erste Einrichtung

Beim ersten Start sehen Sie links eine leere Server-Leiste.

1. Klicken Sie `+ Neu` in der Server-Werkzeugleiste.
2. Füllen Sie aus:
   - Alias (Anzeigename)
   - Gruppe (Standard „default")
   - Optionale Farbmarkierung (hilft, Produktion und Test auf einen Blick zu unterscheiden)
   - Host, Port, Benutzer, Passwort, Datenbank, sslmode
   - Optionales sslrootcert (über Einstellungen -> Zertifikate hochladen)
3. Klicken Sie `Speichern und verbinden` oder nur `Speichern`.

Die Verbindung erscheint im Server-Baum. Ein Klick öffnet sie. Der Baum klappt die übergeordnete Gruppe auf und scrollt zur Zeile.

Passwörter werden mit einem Schlüssel verschlüsselt gespeichert, der für jede Installation eigen ist. Die Schlüsseldatei liegt neben `connections.json` in Ihrem Datenverzeichnis.

### Wofür das Zertifikat da ist, und was TLS tatsächlich einschaltet

Das sind zwei getrennte Einstellungen, und das Zertifikat ist die zweite, nicht
die erste.

**`sslmode` schaltet die Verschlüsselung ein.** Es sitzt an der Verbindung
selbst:

| `sslmode` | Was Sie bekommen |
| --- | --- |
| `disable` | Keine Verschlüsselung |
| `prefer` (Standard) | Verschlüsselt, wenn der Server es anbietet, sonst Klartext - stillschweigend |
| `require` | Verschlüsselung ist Pflicht, aber nichts wird geprüft: jedes Zertifikat wird akzeptiert, auch eines, das ein Angreifer gerade erzeugt hat |
| `verify-ca` | Das Serverzertifikat muss zu einer Wurzel führen, der Sie vertrauen |
| `verify-full` | …und der angewählte Hostname muss zum Zertifikat passen |

Bei `require` hören die meisten auf, und genau diese Einstellung wirkt sicher
und ist es nicht: sie verschlüsselt den Verkehr und übergibt ihn dann dem, der
geantwortet hat.

**Das Zertifikat ist die Wurzel, gegen die `verify-ca` und `verify-full`
prüfen.** libpq sucht eine in `~/.postgresql/root.crt`. Wurde das Zertifikat
Ihres Servers von einer öffentlichen CA ausgestellt, die dort bereits liegt,
müssen Sie nichts hochladen - `sslmode` setzen genügt. Ein Zertifikat laden Sie
hoch, wenn der Aussteller *nicht* schon vertraut wird: Ihre eigene interne PKI,
oder das CA-Bundle eines Cloud-Anbieters (AWS, Google und Azure veröffentlichen
je eines).

Also:

1. Das PEM unter **Einstellungen → Zertifikate** hochladen. Es wird nach
   `<DBCHECKS_HOME>/certs/` mit Modus 0600 kopiert.
2. Die Verbindung öffnen und **sslmode** auf `verify-ca` oder `verify-full`
   setzen.
3. Die hochgeladene Datei bei **TLS-Wurzelzertifikat** an derselben Verbindung
   auswählen.

Hochladen ohne Schritt 2 ändert nichts: die Datei ist gespeichert, referenziert
und wird nie herangezogen, denn bei `prefer` oder `require` gibt es keine
Prüfung, die sie heranziehen würde.

## 2.3 Zustimmung zum Lizenzvertrag

Beim ersten Start zeigt DBApguin den Endbenutzer-Lizenzvertrag und öffnet sich
erst, wenn Sie ihm zustimmen. Es gibt keinen Weg daran vorbei - keine
Schaltfläche zum Schließen, kein Esc, kein Klick daneben - weil dahinter noch
nichts liegt, was man benutzen könnte.

Das Fenster zeigt den Vertrag selbst, nicht seine Zusammenfassung, gelesen aus
derselben Datei, die später Hilfe → Lizenzvertrag anzeigt. Es hat eine
**Sprachauswahl**, und neben den Schaltflächen stehen Links auf die
Datenschutzhinweise und die Liste der Komponenten Dritter.

**Ablehnen und beenden** hält nichts fest und schließt das Fenster. Durch
Ablehnen wird nichts installiert oder registriert.

**Ich stimme zu** hält zwei Dinge in `app_prefs.json` fest, ausschließlich auf
Ihrem Rechner: welcher Version Sie zugestimmt haben und wann. Es wird nichts
übertragen. Die gespeicherte Version ist der Grund, warum Sie einmal gefragt
werden statt bei jedem Start - und warum erneut gefragt wird, wenn sich der
Vertrag selbst ändert, denn die Zustimmung zu einer Version ist keine Zustimmung
zu dem, was eine spätere sagt.

Zum Nachlesen jederzeit: **Hilfe → Lizenzvertrag**.

## 2.4 Der Hinweis beim Start

Beim ersten Lauf zeigt die Anwendung einen kurzen Hinweis mit dem Logo: was sie
tut, und dass ihre Vorschauen eine Orientierung sind und kein Sicherheitsnetz.
Er steht dort, weil dieses Werkzeug auf Ihre Anweisung hin `DROP` und `TRUNCATE`
an die Produktion schickt, und das verdient es, vor der ersten Nutzung einmal
ausgesprochen zu werden.

Der Hinweis trägt außerdem eine **Sprachauswahl**, denn der erste Start ist der
Moment, in dem diese Wahl zählt, und in die Einstellungen schaut da noch
niemand. Eine Änderung wechselt sofort den Hinweis, den Sie gerade lesen - die
einzige Art, sich der richtigen Wahl zu vergewissern. Dieselbe Einstellung liegt
danach unter **Einstellungen → Sprache**.

Setzen Sie **Nicht mehr anzeigen** und der Hinweis bleibt weg. Zurückholen:
**Einstellungen → Starthinweis**.

Ihn abzuschalten verbirgt die Erinnerung. An den Bedingungen, auf die er
verweist, ändert das nichts - die stehen unter Hilfe → Lizenzvertrag.

## 2.5 Nutzungsbedingungen

DBApguin ist **für alle kostenlos**, für private wie für kommerzielle Nutzung,
mit allen Funktionen.

Was das in der Praxis bedeutet:

- **Alle Funktionen stehen vom ersten Start an bereit**, auf beliebig vielen
  Rechnern: die Berater, die Lastanalyse, AWR-Snapshots, der Abfrageverlauf,
  die Index- und Bloat-Analyse, die Vergleichs- und Migrationswerkzeuge - alles.
- **Über das Netz wird nichts geprüft**, also telefoniert nichts nach Hause.
  Die Anwendung spricht ausschließlich mit den Datenbankservern, die Sie einrichten -
  und, wenn Sie Plugins installieren, mit dem, wofür diese Plugins eingerichtet
  sind (Kapitel 7, Plugins).
- **Kein Konto, keine Anmeldung, keine Registrierung.**

Das Einzige, worum Sie gebeten werden, ist die Lizenzvereinbarung beim ersten
Start zu akzeptieren (Abschnitt 2.3).

Kurz gesagt: frei nutzen, nicht verkaufen, nicht als eigenes Werk ausgeben.
Kostenlos ist nicht dasselbe wie Open Source: der Quellcode wird nicht
veröffentlicht. Der vollständige Text steht unter
**Hilfe -> Lizenzvereinbarung**.

DBApguin wurde von Darko Lovrenčec erstellt.

## 3. Server-Leiste

Das linke Feld hat drei Abschnitte:

### 3.1 Server (oben)

Listet jede gespeicherte Verbindung, nach Gruppe geordnet. Schaltflächen der Werkzeugleiste:

- `+ Neu`: Verbindung anlegen
- `Bearbeiten`: die aktive Verbindung bearbeiten (Alias, Gruppe, Farbe, Host, Passwort)
- `Alle einklappen`: jede Gruppe einklappen
- `Alle aufklappen`: jede Gruppe aufklappen
- `Mehr` (...): die vollständige Verbindungsverwaltung öffnen (Import, Export, Massenumbau)

Kontextmenü je Verbindung:
- Verbinden / Zum Tab wechseln
- Bearbeiten
- Verbindung auffrischen (Passwort erneut eingeben, neu verbinden)
- Tab anheften / lösen
- Trennen
- SQL-Editor / psql / Sitzungen / Dashboard / Wartung / Server öffnen (nur bei aktiver Verbindung)
- Untermenü Cluster-Prüfungen
- Datenbank / Tabelle / Rolle anlegen
- Verbindung testen
- Duplizieren
- Löschen (Meldung mit „Rückgängig" für 6 Sekunden)

Statuspunkte neben jeder Verbindung:
- Grün gefüllt: das ist der aktive Tab
- Grün hohl: Tab offen, Sie sehen aber einen anderen
- Rotes X: Verbindung verloren (der Server hat sie geschlossen)
- Kein Punkt: nicht verbunden

### 3.2 Datenbanken (nur sichtbar, wenn verbunden)

Listet jede Datenbank des Clusters. Ein Klick wechselt die aktive Datenbank. Filterfeld oben.

**Rechtsklick auf die Überschrift DATENBANKEN** zeigt das **Database**-Menü der Leiste, Eintrag für Eintrag - Verbindungen, Hintergrundjobs, Vergleich, Backup und Zurückspielen, Anlegen, Klonen, Umbenennen, Rechte und Löschen, Erweiterungen - danach `Datenbankliste auffrischen`. Anlegen, Klonen, Umbenennen und Löschen sind ausgegraut, mit Begründung, wenn Ihre Rolle oder der Anbieter sie nicht erlaubt.

**Rechtsklick auf eine Datenbankzeile** für alles, was diese eine betrifft:
SQL-Editor, Sitzungen, Dashboard, Wartung, den ganzen Prüfkatalog, Objekte
anlegen, Backup und Zurückspielen, Klonen, Umbenennen, Rechte und Löschen.

#### Erweiterungen

`Erweiterungen` öffnet eine von drei Listen, und die dritte ist meist die
gesuchte:

- `Installierte Erweiterungen…` - was diese Datenbank bereits angelegt hat
- `Verfügbare Erweiterungen (nicht installiert)…` - was der **Server**
  mitbringt und wofür noch niemand `CREATE EXTENSION` ausgeführt hat
- `Erweiterungen verwalten…` - alles, mit Filter und Zähler

Alles, was der Server mitbringt, erscheint hier, ob angelegt oder nicht. Aus
derselben Liste installieren Sie es, ändern die Version, verschieben es in ein
anderes Schema oder entfernen es. Erweiterungen, die der Server gar nicht hat,
können nicht erscheinen: sie müssen zuerst auf dem Rechner installiert werden,
üblicherweise als Paket.

### 3.2.5 Rollen und Benutzer

Steht neben Datenbanken statt unter einer, denn Rollen gelten clusterweit: sie
gehören keiner Datenbank, und sie unter eine zu hängen würde etwas anderes
behaupten.

**Rechtsklick auf die Überschrift ROLLEN UND BENUTZER**:

- `+ Rolle / Benutzer anlegen…`
- `Übersicht über Rollen und Benutzer` - die vollständige Prüfung
- `Rollen / Benutzer vergleichen…` zwischen zwei Servern
- `ALLE Rollen als .sql exportieren…`
- `Rollen auffrischen`

**Klick auf eine Rolle** startet eine Rechteprüfung für sie. **Rechtsklick auf
eine Rolle** für ihre eigenen Aktionen: Rechte, `search_path`, Mitgliedschaften,
Ähnliche anlegen, Rechte kopieren, DDL exportieren, Löschen.

Login-Benutzer zeigen 👤, Gruppenrollen 👥.

#### Eine Rolle exportieren {#role-export}

**Export DDL** (eine Rolle) und **Export ALL roles** schreiben ein Skript,
das die Rolle anderswo neu anlegt: `CREATE ROLE` mit jedem Attribut (in
einem Block, der eine schon vorhandene Rolle überspringt), ihr Kommentar,
ihre Mitgliedschaften, ihre Einstellungen `ALTER ROLE ... SET` - auch pro
Datenbank - und, standardmäßig angehakt, **jedes Recht, das sie im Cluster
hält**: auf Datenbanken, Tablespaces und Serverparametern (PostgreSQL 15+)
und in jeder Datenbank auf Schemas, Tabellen und deren Spalten, Sequenzen,
Funktionen und Prozeduren, Typen und Domänen, Sprachen, Foreign-Data-Wrappern
und Servern sowie Large Objects, dazu die Standardrechte, die sie vergibt
oder erhält (`ALTER DEFAULT PRIVILEGES`). Jede Datenbank wird einzeln gelesen,
und ihr Teil beginnt mit `\connect`, die Datei ist also ein psql-Skript:
`psql -f role.sql` gegen das Ziel, als Superuser. Eine Datenbank, die der
Export nicht lesen konnte - eine, mit der sich die verbundene Rolle nicht
verbinden darf -, steht am Ende der Datei und in einer Meldung, ein fehlender
Teil bleibt also nie stumm. **Was ihr gehört** ergänzt `ALTER ... OWNER TO`
für ihre Schemas, Tabellen, Sichten, Sequenzen und Routinen, wenn diese
Objekte schon am Ziel sind. Passwort-Hashes sind eine eigene Option.
Gemessen: eine Rolle mit Rechten jeder Art in zwei Datenbanken, exportiert,
mit `DROP OWNED` und `DROP ROLE` entfernt und die Datei mit psql ausgeführt -
jedes Recht und jeder Besitz kam zurück wie zuvor.

### 3.3 Schema-Browser (nur sichtbar, wenn verbunden)

Der Schema-Browser ist Teil des Baums **Server**. Unter der Verbindung, auf der Sie sind, listet **Datenbanken** jede Datenbank des Servers mit einer Anzahl. Klappen Sie eine auf, öffnet sie sich in ihre Schemas, jedes mit der Zahl der Objekte, die es enthält, und jedes Schema in seine Objekte, nach Art gruppiert.

Jedes Schema zeigt dieselben 40 Arten: Tabellen, partitionierte Tabellen, Views, materialisierte Views, Fremdtabellen, Sequenzen, Indizes, Funktionen, Prozeduren, Aggregate, Fensterfunktionen, Trigger, Ereignistrigger, Regeln, RLS-Policies, Typen, Domains, Bereichstypen, Multirange-Typen, Erweiterungen, Sprachen, Operatoren, Operatorklassen, Operatorfamilien, Casts, Kollationen, Konvertierungen, Textsuche-Konfigurationen, -Wörterbücher, -Parser und -Vorlagen, Foreign-Data-Wrapper, Fremdserver, Benutzerzuordnungen, Publikationen, Subskriptionen, erweiterte Statistiken, Standardrechte, Large Objects und Tablespaces. Eine Art ohne Inhalt bleibt in der Liste, ausgegraut, damit "es gibt keine" nie wie "nicht angezeigt" aussieht. Objekte, die zu keinem Schema gehören - Ereignistrigger, Sprachen, Casts, Foreign-Data-Wrapper, Fremdserver, Benutzerzuordnungen, Publikationen, Subskriptionen, Standardrechte ohne IN SCHEMA, Large Objects und Tablespaces - stehen unter **(cluster)**. Die Systemschemas `pg_catalog` und `information_schema` zeigen ihre Tabellen, Views, Indizes, Routinen und Typen; die in PostgreSQL eingebauten Operatoren, Casts und ähnlichen Objekte bleiben aus dem Baum heraus.

Relationen öffnen sich so, wie PostgreSQL sie hält. Eine partitionierte Tabelle öffnet sich in ihre Partitionen; eine Partition, die selbst partitioniert ist, steht an ihrem Platz mit ihren eigenen Partitionen darunter, in jeder Tiefe. Eine Tabelle öffnet sich in die Tabellen, die von ihr erben. Jede davon steht einmal, unter ihrem Elternteil, nicht noch einmal bei den Tabellen. Eine Partition, die in einem anderen Schema liegt als ihr Elternteil, steht unter dem Elternteil mit ihrem Schema und in ihrem eigenen Schema oben, mit dem Elternteil genannt. Jede Tabelle, View und materialisierte View öffnet sich außerdem in ihre Teile - Indizes, Constraints, Trigger, Regeln, RLS-Policies, die Sequenzen ihrer Spalten, Statistikobjekte und die Publikationen, in denen sie ist -, gelesen erst beim Öffnen, sodass ein Schema mit Tausenden Tabellen dadurch nicht langsamer lädt. Was PostgreSQL von einer partitionierten Tabelle auf eine Partition kopiert hat (ein Index, ein Schlüssel, ein Zeilentrigger), sagt dort *von* dem Original und steht nicht in den Listen der Indizes und Trigger des Schemas, die zeigen, was deklariert wurde; eine solche Kopie lässt sich nicht für sich löschen. Eine Sequenz, die zu einer Spalte gehört, sagt das (*gehört zu* oder *Identity*).

Objekte, die zu einer Erweiterung gehören, bleiben aus jeder Art heraus - die Erweiterung selbst steht weiter unter Erweiterungen -, und das Schema sagt, wie viele es ausgelassen hat. Ein Klick auf diese Zeile zeigt sie, jedes mit seiner Erweiterung markiert; ein weiterer Klick blendet sie wieder aus.

Ein Klick auf ein Schema zeigt seine Tabellen mit lebenden und toten Zeilen, Größen, Indexanzahl, Scans sowie dem letzten Vacuum und Analyze; das Filterfeld über dieser Liste grenzt sie nach Namen ein. Ein Klick auf ein Objekt öffnet es in einem eigenen Tab (bei einer Tabelle DDL, Beziehungen, Speicher, Berater, Daten). Rechtsklick für alles Weitere - siehe [Objektmenüs](#object-menus). Das Suchfeld über dem Baum findet Verbindungen nach Alias, Host, Benutzer oder Datenbank; innerhalb des Baums sucht es nicht.

## 4. Arbeits-Tabs

Jeder Tab trägt seine eigene Verbindung, seine Position im Schema und den Zustand des SQL-Editors. Tabs überdauern Neustarts der Anwendung.

- `+ Neue Verbindung` (Tableiste oben links): öffnet einen Tab ohne Verbindung.
- Rechtsklick auf einen Tab: Wechseln / Verbindung auffrischen / Anheften / Als Verbindung speichern (für ungespeicherte Tabs) / Trennen / Schließen.
- Angeheftete Tabs überstehen „Alle schließen".

#### Eine Verbindung anlegen

Der Dialog Neue Verbindung enthält die Verbindungsdaten und darunter den Namen,
die Gruppe und die Farbmarkierung. Das Gruppen-Dropdown listet die vorhandenen
Gruppen und bietet **Neue Gruppe anlegen** und **Untergruppe anlegen** - eine
Verbindung lässt sich also gleich dort ablegen, wo sie hingehört, statt in
`default` zu landen und danach verschoben werden zu müssen.

Drei Schaltflächen beenden den Dialog, und alle drei tun mit der Verbindung
dasselbe:

| | |
|---|---|
| **Test** | prüft die Zugangsdaten und bleibt stehen |
| **Verbinden** | öffnet sie in einem Arbeits-Tab, ohne sie zu speichern |
| **Speichern & verbinden** | schreibt die gespeicherte Verbindung und öffnet sie dann |

„Öffnet sie" bedeutet in beiden Fällen dasselbe: ein eigener Arbeits-Tab, sofern
der aktuelle nicht noch leer ist, mit der gespeicherten Verbindung markiert, so
dass die Tab-Leiste oben sie zeigt, in der Seitenleiste als aktiv markiert, und
jeder danach geöffnete Werkzeug-Tab gehört zu dieser Verbindung. „Speichern &
verbinden" hat früher nur die Zeile geschrieben und dort aufgehört und ließ den
gerade vordersten Tab eine Verbindung zeigen, von der er nichts wusste.

Sind mehr Tabs offen als Platz haben, erscheinen an beiden Enden Pfeile und
unter den Tabs ein **schmaler Schieber**. Der Schieber erscheint nur, wenn es
etwas zu scrollen gibt; ziehen Sie ihn oder klicken Sie in die leere Bahn, um zu
blättern. Er hat eine eigene Spur und verdeckt daher nie einen Tab oder dessen
Schließen-Schaltfläche.

#### Rolle, Dienst und SSH-Tunnel {#conn-role-service-ssh}

Drei Einstellungen im Verbindungsdialog, die die meisten Verbindungen leer
lassen, benannt wie in pgAdmin.

- **Rolle** - die Sitzung führt direkt nach jedem Verbinden `SET ROLE` auf sie
  aus, auf jeder Verbindung, die die App zu diesem Server öffnet: den Verbindungen
  aus dem Pool, einem erneuten Verbinden, Prüfungen, Hintergrund-Sammlern, einer
  gehaltenen SQL-Transaktion. Alles läuft dann als diese Rolle, ihre Rechte
  bestimmen, was die App anbietet, und die Kopfzeile nennt sie. Der
  Anmeldebenutzer muss Mitglied der Rolle sein; lehnt der Server ab, schlägt das
  Verbinden mit dem Namen der Rolle und dem Grund des Servers fehl, und nichts
  läuft stattdessen als Anmeldebenutzer. Eine in der Rollenliste der Kopfzeile
  gewählte Rolle übernimmt weiterhin, solange sie gewählt ist.
- **Dienst** - ein libpq-Dienstname. Host, Port, Datenbank, Benutzer und
  Passwort, die im Dialog leer bleiben, kommen aus diesem Dienst, und was Sie
  doch ausfüllen, hat Vorrang. libpq sucht den Dienst in der Datei aus
  `PGSERVICEFILE`, sonst in `.pg_service.conf` im Home-Verzeichnis (unter Windows
  `%APPDATA%\postgresql\.pg_service.conf`), dann in `pg_service.conf` im
  Verzeichnis aus `PGSYSCONFDIR`.
- **Erweitert: SSH-Tunnel** (eingeklappt, bis eine Verbindung ihn nutzt) - für
  eine Datenbank, die nur über einen Jump-Host erreichbar ist. Geben Sie SSH-Host,
  Port (22, wenn leer), Benutzer und bei Bedarf die Schlüsseldatei an. Die App
  startet das `ssh`-Programm Ihres Computers mit einer lokalen Portweiterleitung,
  daher gelten Ihre ssh-Einstellungen, `known_hosts` und `ssh-agent`. Sie meldet
  sich nur per Schlüssel an und fragt nie nach einem SSH-Passwort oder speichert
  eines; ein Schlüssel mit Passphrase muss in `ssh-agent` geladen sein. Host im
  Dialog ist dann der Datenbank-Host aus Sicht des Jump-Hosts, oft `localhost`.
  Ein Jump-Host, dessen Schlüssel Sie noch nicht akzeptiert haben, wird mit dem
  Befehl abgelehnt, den Sie einmal im Terminal ausführen, um ihn zu prüfen und zu
  akzeptieren. Der Tunnel schließt mit dem letzten Tab, der ihn nutzt, und beim
  Beenden der App.

## 5. Hauptansichten

Die obere Tableiste wechselt die Ansicht. Jede Ansicht arbeitet gegen die aktive Verbindung des aktiven Arbeits-Tabs.

In jeder Auswahlliste lässt sich suchen. Klicken Sie sie an oder tippen Sie einfach los, solange sie den Fokus hat, und über ihr öffnet sich ein Suchfeld: die Liste wird mit jeder Taste kürzer, gefunden wird jeder Teil eines Eintrags, ohne Rücksicht auf Groß- und Kleinschreibung oder Akzente ("sifra" findet "šifra"), und jedes getippte Wort muss vorkommen. Pfeil hoch und runter bewegen, Enter wählt, Escape schließt und lässt die Auswahl, wie sie war, und Tab übernimmt den markierten Eintrag und geht weiter. Listen mit nur zwei oder drei Einträgen öffnen die Liste des Systems, in der ein getippter Buchstabe zum ersten Eintrag springt, der mit ihm beginnt.

### 5.1 Prüfungen

Die Startansicht. Die obere Menüleiste zeigt den Prüfkatalog nach Gruppen:

- Cluster-Zustand: db-health, db-locks, db-blocking, db-long-tx, db-idle-in-tx, db-conn-limits, db-vacuum, db-autovacuum, db-checkpoints, db-replication, db-repl-slots, db-logical-replication
- Verbindungen und Abfragen: db-connections, db-users, db-active-queries, db-long-queries, db-stat-statements, db-query-history, db-temp-queries, db-temp-files
- Speicher und Leistung: db-sizes, db-schema-sizes, db-table-sizes, db-index-sizes, db-bloat, db-bloat-indexes, db-bloat-breakdown, db-pgstattuple, db-buffer-cache, db-tablespaces
- Tuning und Diagnose: db-settings, db-cache-hit, db-cache-advisor, db-index-usage, db-index-unused, db-index-advisor, db-duplicate-indexes, db-table-stats, db-replica-identity, db-workload-report, db-ash, db-awr-snapshot, db-trace-session, db-errors
- Sicherheit / Berechtigungen: db-roles-list, db-role-perms (Ziel wählen), db-ownership (Bereich wählen)
- Ebenfalls erreichbar: db-sequences, db-seq-usage, db-seq-sync (Sequenz-Werkzeuge im Schema-Browser), db-setup-advisor (Menü Hilfe)

**Drei Berichte beantworten „wo ist der Platz geblieben", auf drei Ebenen.**
`db-sizes` liefert jede Datenbank im Cluster; `db-schema-sizes` schlüsselt die
**verbundene** Datenbank nach Schemas auf - Gesamtgröße, Daten und Indizes
getrennt ausgewiesen, der Anteil jedes Schemas, die Objektzahl und seine
größte Tabelle; `db-table-sizes` geht bis zur einzelnen Tabelle;
`db-index-sizes` bis zum einzelnen Index. Wie alle vier zu lesen sind und was
jede Gesamtsumme einschließt und was nicht, steht unter *Wohin der Platz
geht* weiter unten.

Schema-Größen deckt nur die Datenbank ab, mit der Sie verbunden sind, und das
ist eine Grenze, kein Versäumnis: `pg_namespace` gilt je Datenbank, eine Liste
über alle hinweg bräuchte eine Verbindung zu jeder einzelnen. Die Gesamtgröße
umfasst alles, was im Schema Platz belegt - Tabellen, ihr TOAST, ihre Indizes,
materialisierte Sichten und Sequenzen - jedes Byte einmal.

Jede Prüfung öffnet ihren eigenen Ergebnis-Tab. Tabs überstehen den Wechsel der Ansicht, sodass Sie mehrere Prüfungen nebeneinander vergleichen können. Jeder Tab hat zwei Bereiche:

- Strukturiert: reiche Tabellen, Beraterkarten, Detailansichten
- Rohausgabe: einfacher Textbericht

Viele Prüfungen enthalten „Berater"-Abschnitte mit nach Schwere markierten Befunden (OK / INFO / WARN / KRIT) und konkreten Vorschlägen für den nächsten Schritt.

#### Zusammenzählen, was ein Bericht zeigt

Jede Berichtstabelle sortiert nach Spalte und filtert durch Tippen. Zwei
weitere Dinge beantworten die Frage, für die diese Berichte meist geöffnet
werden - „lohnt sich das".

**Eine Summe unter der Tabelle.** Jede Spalte, deren Werte sich als Größen oder
als reine Zahlen lesen, wird summiert, und die Summe folgt jedem Filter. Grenzt
man einen Bloat-Bericht auf ein Schema ein, sagt die Summe, wie viel Platz ein
Reindex dieses Schemas zurückgäbe; grenzt man weiter auf eine Tabelle ein,
sieht man, ob sich die Arbeit am ganzen Schema lohnt oder ob fast alles an
einer Tabelle hängt.

Die Zahlen werden aus dem Gedruckten zurückgelesen, denn bei den meisten dieser
Prüfungen ist das alles, was es gibt - PostgreSQLs eigene Größenformatierung
und nichts darunter. Die Summe ist deshalb eine Summe der angezeigten Werte,
gerundet auf die gedruckte Einheit: vierzig auf Megabyte gerundete Indizes
können zweistellig danebenliegen. Genau genug zum Entscheiden, keine Zahl zum
Zitieren.

Prozentsätze, Verhältnisse und Mittelwerte werden nie summiert - ihre Summe ist
eine Zahl ohne Bedeutung, und sie zu drucken lüde dazu ein, sie zu lesen. Der
Spaltenname sagt es: ein Bloat-Bericht druckt `bloat_pct` als bloße `44,9`, das
Zeichen steht nirgends in der Zelle.

Das Ergebnisraster des SQL-Editors und der Daten-Tab des Schema-Browsers
summieren nur Größen. Ein Bericht weiß, was seine Spalten sind; ein gerade
getipptes `SELECT` weiß es nicht, und eine Summe unter einer Spalte von IDs ist
Rauschen.

**Werte in einer Spalte wählen.** Der Pfeil neben dem Filterfeld jeder Spalte
öffnet die Liste der Werte, die in dieser Spalte tatsächlich vorkommen, mit
ihrer Anzahl. Drei Schemas anzuhaken, deren Namen nichts gemeinsam haben, kann
ein Teilstring-Filter nicht, und Speicherberichte sind voll davon - ein Schema
von zwanzig, vier Tabellen von neunhundert. Der getippte Filter und die
angehakte Liste gelten beide.

Eine Spalte mit mehr als zweihundert verschiedenen Werten sagt das, statt
tausend Kästchen zu zeichnen.

#### Um welche Datenbank es in einem Bericht geht

Jeder dieser Berichte gilt für eine Datenbank, und die Zeile über den
Ergebnissen sagt, für welche - der Name der Prüfung, die Datenbank und die
Verbindung, auf der sie lief. Sie kommt aus dem Tab, nicht aus der lebenden
Verbindung: ein Tab, der gegen `shop` lief, sagt weiterhin `shop`, auch wenn
die Verbindung darunter gewechselt wird.

#### Ein Index, und was damit zu tun ist

Die Berichte nennen einen aufgeblähten oder ungenutzten Index. Ein Klick auf
den Namen öffnet den **Index-Berater** für genau diesen Index; ebenso auf jedem
Index in der Indexliste des Schema-Browsers.

Er beantwortet die Frage, die der Bericht offenlässt. Die Bloat-Schätzung ist
dieselbe Rechnung wie im Bericht, aus dem der Klick kam, auf den einen Index
verengt - die beiden können sich also nicht widersprechen. Daneben: wie oft er
seit dem letzten Zurücksetzen der Statistik gescannt wurde, seine Größe im
Verhältnis zur Tabelle, und ein Abgleich mit allen anderen Indizes derselben
Tabelle - ein zweiter Index auf genau denselben Spalten mit derselben Methode,
oder einer, dessen führende Spalten die ganze Spaltenliste dieses Index sind;
dann kann der Planer den breiteren für alles verwenden, was dieser beantwortet.

Indizes hinter Bedingungen liest man anders, und darum ist „nie gescannt" für
sich noch kein Grund zum Löschen. Ein Index hinter einem PRIMARY KEY, einem
UNIQUE oder einem EXCLUDE erzwingt die Bedingung, ob eine Abfrage ihn je liest
oder nicht, und `DROP INDEX` verweigert ihn; dasselbe gilt für die Replica
Identity einer Tabelle, die die logische Replikation braucht. Der Berater sagt
das und bietet das Löschen nicht an.

Was er zur Ausführung anbietet, ist für einen Index geschrieben und für keinen
weiteren:

| | |
|---|---|
| `REINDEX INDEX CONCURRENTLY` | baut daneben einen Ersatz und tauscht - keine lange Sperre, braucht Platz für beide Kopien, läuft nicht in einer Transaktion (PostgreSQL 12+) |
| `REINDEX INDEX` | hält während des gesamten Neuaufbaus eine ACCESS EXCLUSIVE-Sperre auf der Tabelle |
| `DROP INDEX CONCURRENTLY` | nur dort angeboten, wo der Index löschbar ist |
| die Definition | vor dem Löschen aufheben - das ist die einzige Kopie |

Jedes öffnet denselben Dialog, in dem jede erzeugte Anweisung dieser Anwendung
öffnet: ansehen, kopieren, speichern oder im SQL-Editor öffnen. Die beiden
Neuaufbauten bieten zusätzlich **Als Hintergrundauftrag ausführen**, was sie mit
laufender Ausgabe in die Hintergrundaufträge stellt - ein Neuaufbau von zwanzig
Minuten hängt dann nicht an einem offen bleibenden Fenster.

Vorher war der einzige REINDEX, den die Anwendung ausgeben konnte, pro Tabelle,
pro Schema oder pro Datenbank - ein Bericht, der einen aufgeblähten Index
nannte, ließ sich also nur behandeln, indem man alle Indizes seiner Tabelle neu
baute.

#### Erwähnenswerte Prüfungen {#notable-checks}

**db-health** (empfohlene erste Station): Zustandsprüfung auf einem Bildschirm über Verbindungen, idle-in-tx, lange aktive Transaktionen, Blockaden, Cache-Trefferrate, Replikationsrückstand, WAL-Archivierung, Bloat, ausstehendes Vacuum, Sequenzen, ungenutzte Indizes. Detailansichten je Kategorie holen die auffälligen Zeilen hervor.

**db-cache-advisor**: Trefferraten für Heap/Index/TOAST, heiße Tabellen/Indizes/TOAST mit konkreter Abhilfe, Prüfung der shared_buffers-Größe gegen die Clustergröße, Plausibilität von effective_cache_size, pg_buffercache-Snapshot, wenn die Erweiterung installiert ist.

**db-index-advisor**: Seq-Scan-Analyse mit Kandidatenspalten je Tabelle. Gleicht mit pg_stat_statements für heiße Abfragen ab, zeigt Lücken bei führenden Spalten, Gelegenheiten für Teilindizes und die Abdeckung zusammengesetzter FKs. Erzeugt CREATE-INDEX-Anweisungen zum Kopieren.

**db-vacuum**: Vacuum-/Analyze-Zustand je Tabelle mit Autovacuum-Berater. Zeigt lebende und tote Zeilen, mod_pct, Hitze-Einstufung (sehr heiß / heiß / mittel / Standard), Markierungen (WRAPAROUND-Risiko, autovacuum deaktiviert, nur Inserts) sowie die „empfohlenen" gegenüber den „aktuellen" reloptions je Kandidat. Seit 1.0.x vom Oktober 2026 schaut er auch in die andere Richtung: wie oft jede Tabelle gesäubert wird (Läufe pro Tag und der Abstand dazwischen, mit der Dauer auf PostgreSQL 18), eine stündlich oder öfter gesäuberte Tabelle - und ob das in Ordnung ist (siehe [wann häufiges Vacuum in Ordnung ist](#vacuum-bloat-and-table-statistics)), eine wieder und wieder ohne Wirkung gesäuberte, weil etwas einen Snapshot hält, der älter ist als ihre toten Zeilen (benannt, mit dem, was zu beenden ist), eine Tabelle, deren jedes Vacuum aggressiv ist, und - im Abschnitt *Autovacuum cost and I/O* - die Tempo-Einstellungen von Autovacuum und auf PostgreSQL 16 und neuer seinen Anteil an den Lese- und Schreibzugriffen des Clusters aus `pg_stat_io`. Eine Empfehlung verschärft nie die Schwellen einer schon stündlich gesäuberten Tabelle. Er sagt auch, wann Vacuum an eine Tabelle gar nicht herankommt: eine Sitzung hält eine Sperre, die VACUUM nicht teilen kann (SHARE UPDATE EXCLUSIVE oder stärker - ein langes ANALYZE, CREATE INDEX CONCURRENTLY, ALTER TABLE, LOCK TABLE), in einer seit mindestens einer Minute offenen Transaktion, auf einer Tabelle über ihrem Auslöser. Ein normales Autovacuum überspringt eine solche Tabelle und versucht es eine Naptime später erneut, also geschieht nichts, was sonst über sie gesagt wird; der Befund nennt die Sitzung (PID, Benutzer, Anwendung, Client, wie lange ihre Transaktion offen ist, ihre Anweisung) und steht an erster Stelle. Auch ein Schreiber, der untätig in einer READ-COMMITTED-Transaktion steht, zählt als Snapshot-Halter: zwischen den Anweisungen hat er keinen Snapshot, aber seine Transaktions-ID hält den Horizont.

**db-stat-statements**: die 25 teuersten Abfragen mit Hinweisen je Zeile (Planinstabilität, work_mem-Auslagerung mit konkreten Bytes pro Aufruf, niedrige Cache-Trefferrate, bei jedem Aufruf langsam, Pfad des schlechten Falls). Das Widget mit den teuersten Abfragen erlaubt es, den vollständigen Text zu kopieren.

**db-replica-identity**: Tabellen ohne PRIMARY KEY und Tabellen mit abweichender REPLICA IDENTITY. Gleicht mit pg_publication_tables ab, um KRITISCH (in einer Publikation, bricht bei UPDATE/DELETE) von rein hinweisend zu trennen. Empfiehlt ALTER in dieser Reihenfolge: ADD PRIMARY KEY, REPLICA IDENTITY USING INDEX, REPLICA IDENTITY FULL.

**db-errors**: Cloud-bewusste Fehlerüberwachung. Nimmt immer Zähler aus pg_stat_database über ein Fenster von 5 Sekunden (Rollbacks, Deadlocks, Konflikte, sessions_fatal/killed/abandoned ab PG14). Auf selbst betriebenen Clustern liest sie zusätzlich die letzten 64 KB des Server-Logs über `pg_current_logfile()` + `pg_read_file()` und wertet ERROR/FATAL/PANIC-Zeilen aus. In Managed Clouds entfällt das Log-Ende, mit anbieterspezifischen Hinweisen (RDS-Konsole, log_fdw, Cloud Logging, Azure Server Logs, Supabase / Neon / Heroku).

**db-roles-list**: Rollen und Mitgliedschaften plus ein Sicherheitsdurchgang: Superuser + Login, BYPASSRLS ohne Superuser, CREATEROLE auf PG <16, Login ohne Verbindungsgrenze und ohne VALID UNTIL, abgelaufene Login-Rollen, MD5-Passwort-Hashes, REPLICATION + LOGIN, PUBLIC hat CONNECT auf Datenbanken, PUBLIC hat CREATE auf Schemas, SECURITY-DEFINER-Routinen im Besitz eines Superusers mit Ausführungsrecht für PUBLIC.

**db-role-perms** (Zielrolle wählen): 19 Abschnitte (Übersicht, Mitgliedschaften, Rechte auf Datenbank/Schema/Tabelle/Spalte/Sequenz/Funktion/Typ/FDW/Server/Sprache, Standardrechte, RLS) mit einer Deutungsspalte je Zeile („über PUBLIC", „WITH GRANT OPTION", „kein USAGE auf Schema X - Gewährung wirkungslos" und so weiter) sowie einem abschnittsübergreifenden Befundblock (faktischer Superuser über eine Kette, Vererbung von BYPASSRLS / CREATEROLE / REPLICATION, Tabellenrechte ohne Schema-USAGE und mehr).

**db-ownership** (Bereich wählen): Listen für Schemas / Tabellen / Sequenzen / Funktionen / Typen / FDWs / Sprachen / Publikationen / Abonnements plus Befunde (Anwendungsobjekte im Besitz eines Superusers, SECURITY-DEFINER-Routinen im Besitz eines Superusers, verwaiste Eigentümer-OIDs nach DROP ROLE ohne REASSIGN, public-Schema mit Nicht-Superuser als Eigentümer, Schemas mit drei oder mehr verschiedenen Objekteigentümern).

**db-settings**: kuratierter Leitfaden zu rund 70 einstellungsrelevanten Parametern aus pg_settings mit DBA-Begründung, Faustregel und Statusabzeichen (Warnung / Gefahr). Bearbeitbar über die Server-Ansicht.

**db-setup-advisor**: prüft jede serverseitige Einstellung, auf die DBApguin-Funktionen bauen (Erweiterungen, GUCs, Rollenrechte), meldet je Funktion OK / TEILWEISE / FEHLT und druckt das genaue SQL oder die Zeile der Parametergruppe, die die Lücke schließt. Anbieterbewusst: in Managed Clouds heißt der Hinweis „in der Parametergruppe setzen" statt ALTER SYSTEM vorzuschlagen. Führen Sie ihn zuerst aus, wenn Sie DBApguin gegen einen neuen Cluster einrichten, und nach jeder Änderung an postgresql.conf zur Kontrolle erneut.

Die Liste oben sagt, was jede Prüfung erzeugt. Unten steht, wie man die liest,
deren Ausgabe Deutung braucht, und welche Fehler sie fangen sollen.

#### Der Gesundheitsüberblick {#the-health-overview}

`db-health` ist ein Bildschirm: eine Zeile je Kategorie - Verbindungen, idle-in-transaction, lange Transaktionen, Blockaden, Cache-Trefferquote, Replikationsverzögerung, WAL-Archivierung, Vacuum-Verzug, verschwendeter Speicher, Vacuum-Rückstand, Sequenzen nahe ihrer Grenze, ungenutzte Indexe, Dauerhaftigkeit, Partitionen - jede grün, gelb oder rot. Führen Sie sie zuerst aus; oft ist sie die einzige Prüfung, die Sie brauchen. Ein Befund, der eine Tabelle betrifft, nennt sie in der Spalte **table** - die schlimmste, wenn er mehrere zählt - und dieser Name ist ein Link: er öffnet die Tabelle im Schema-Browser.

Gelb heißt „einen Blick wert", Rot heißt „die Produktion ist jetzt betroffen". Klicken Sie eine Kategorie an, um ihre Zeilen zu öffnen: Blockaden entfalten sich zu Blocker/Warter-Paaren, idle-in-transaction zu den Sitzungen und wie lange jede ihren Snapshot hält. Eine rote Zeile ohne Inhalt im Aufriss wurde von einer Summe ausgelöst, nicht von einer einzelnen Zeile - etwa eine über die ganze Datenbank gedrückte Cache-Trefferquote. Die Falle ist, Gelb als Rauschen zu lesen: gelber verschwendeter Speicher ist ein VACUUM, das Sie für heute Nacht planen können; dieselbe Tabelle nächsten Monat rot ist `pg_repack` auf einem ausgelasteten System.

**Vacuum-Verzug und verschwendeter Speicher sind zwei getrennte Zeilen, weil sie zwei verschiedene Dinge messen.** *VACUUM LAG (dead tuple %)* ist `n_dead_tup / (n_live_tup + n_dead_tup)` - der Anteil toter Zeilen einer Tabelle. Er sagt, dass Autovacuum im Rückstand ist. Er ist **keine** Speichermessung: eine Tabelle kann zu 4% tot sein und trotzdem auf einem Terabyte nie zurückgegebenen freien Speichers sitzen. *WASTED SPACE (est.)* ist dieses Terabyte - geschätzte Bytes, die die Tabelle hält, aber nicht nutzt, Heap plus TOAST plus jeden Index, dieselbe kombinierte Schätzung, die `db-bloat-breakdown` und der Advisor-Tab verwenden. Eine Tabelle erscheint als `4.3% dead tuples but 1.3 TB wasted (39x its live data)`, wenn beides zugleich zutrifft. Die Zeile schlägt bei einem von beiden Maßen an: 30 % tote Zeilen (CRIT ab 50 %) oder, unabhängig vom Anteil, 1.000.000 tote Zeilen in einer Tabelle (CRIT ab 10.000.000) - 29 % einer Tabelle mit einer Milliarde Zeilen sind immer noch ein weit zurückliegendes Vacuum.

**WAL ARCHIVING** sagt, ob der Archiver gerade WAL ausliefert, aus `pg_stat_archiver` und den Archiv-Einstellungen - dieselbe Lesung wie der Alarm im Kopf und die Archiv-Karten in Key settings. *Schlägt fehl* (rot) heißt: der letzte Versuch scheiterte und seitdem wurde nichts archiviert; die Zeile nennt das gescheiterte Segment und wann, das zuletzt archivierte und wie viele Segmente in `pg_wal` warten. `archive_mode` an, aber weder `archive_command` noch `archive_library` gesetzt, ist gelb - nichts wird archiviert und jedes fertige Segment bleibt in `pg_wal` - und rot, sobald 10 GB warten. `archive_command` und `archive_library` beide gesetzt ist rot: ab PostgreSQL 16 verweigert der Archiver die Arbeit ganz, während seine Statistik unverändert aussieht. Die Detailansicht zeigt den Befehl so, wie der Server ihn hat, und was zu tun ist: seinen eigenen Fehler im Serverlog lesen (`archive command failed with exit code`), ihn für das genannte Segment von Hand ausführen und nie Dateien aus `pg_wal` löschen - sobald der Befehl funktioniert, holt der Archiver von selbst auf. Das Zählen der wartenden Segmente braucht PostgreSQL 12 und Superuser oder `pg_monitor`; ohne sie fehlt die Zahl.

**DAUERHAFTIGKEIT** liest die Einstellungen, von denen abhängt, ob eine bestätigte Transaktion einen Absturz übersteht. Rot ist sie bei `fsync = off` oder `full_page_writes = off` - den zwei Einstellungen, mit denen ein Absturz des Betriebssystems oder ein Stromausfall den Cluster selbst beschädigen kann, nicht nur seine letzten Transaktionen verliert - und bei einem `synchronous_standby_names`, das mehr synchrone Standbys verlangt, als verbunden sind, während Commits auf sie warten (`synchronous_commit` auf `on`, `remote_write` oder `remote_apply`): dann hängt jedes COMMIT, und die Zeile sagt, wie viele Sitzungen gerade warten (Wait-Event `SyncRep`) und wie lange. Steht `synchronous_commit` serverweit auf `local` oder `off`, ist derselbe fehlende Standby gelb: Commits warten jetzt nicht, aber eine Sitzung, die es wieder auf `on` setzt, wartet. `synchronous_commit = off` und ausgeschaltete Datenprüfsummen sind blau (INFO): das erste kann die letzten bereits als bestätigt gemeldeten Transaktionen verlieren (bis zu drei `wal_writer_delay`), beschädigt aber nichts, ein oft bewusst gewählter Tausch; das zweite heißt, dass eine von der Platte beschädigte Seite ohne Fehler gelesen wird - Prüfsummen einzuschalten verlangt einen gestoppten Cluster (`pg_checksums --enable`, PostgreSQL 12+). Die Key-Settings-Karten derselben fünf Einstellungen kommen zum selben Urteil. Auf einem Standby entfällt der Teil zum synchronen Standby: er gilt nur für einen Primary.

**PARTITIONEN** ist der Vorlauf der nach Bereich partitionierten Tabellen in der verbundenen Datenbank: gelb, wenn ein letzter Bereich innerhalb von 7 Tagen endet oder die DEFAULT-Partition einer RANGE-Tabelle Zeilen enthält, rot, wenn der letzte Bereich geendet hat - siehe [Partitionsvorlauf](#partition-runway).

Die Zeile **MIGRATIONEN** erscheint, wenn die Datenbank die Historientabelle eines Migrationswerkzeugs hat: rot für eine gescheiterte Migration, gelb oder rot für eine Sperre, die ihr Prozess zurückließ (siehe *Migrationshistorie*).

#### Verbindungen, Sitzungen und Last {#connections-sessions-and-load}

Diese lesen `pg_stat_activity` und `pg_stat_statements` aus verschiedenen Blickwinkeln.

`db-connections` ist die Zählung nach Zustand - active, idle, idle in transaction und waiting. **Idle in transaction** ist die Zahl, die man beobachtet: diese Sitzungen halten eine Transaktion offen, pinnen den ältesten Snapshot, den Vacuum entfernen darf, und halten oft Sperren, und eine steigende Zahl ist meist eine Anwendung, die den Commit vergessen hat, oder ein Pool ohne idle-in-transaction-Timeout. `db-users` schlüsselt dieselben Verbindungen nach Rolle und Anwendung auf - so findet man, *welcher* Dienst sie leckt.

`db-conn-limits` ist `max_connections` gegen das Genutzte, mit der Aufschlüsselung nach Zustand und dem Puffer aus `superuser_reserved_connections`. Nur Client-Backends zählen gegen das Limit - Hintergrund-Worker nicht -, also kann die Zahl hier niedriger sein als eine rohe `pg_stat_activity`-Zählung. Der Verbindungsalarm im Kopf zählt genauso. **Waiting** ist eine aktive Anweisung, die von etwas anderem als ihrem Client aufgehalten wird - einer Sperre, einem LWLock, I/O; auch eine untätige Sitzung zeigt ein Wait-Event (`Client: ClientRead`, sie wartet auf ihren nächsten Befehl) und wird nicht gezählt. Nahe der Grenze ist die Lösung fast nie ein höheres `max_connections` (jeder Slot kostet Speicher); es ist ein Connection-Pooler oder das Finden der idle-in-transaction-Sitzungen oben.

`db-active-queries` ist, was gerade jetzt läuft, die längsten zuerst, mit Backend-PID und Wait-Event. `db-long-queries` ist dieselbe Liste, gefiltert auf Anweisungen, die über fünf Sekunden laufen - die, die es lohnt zu unterbrechen oder zu erklären. Beides ist eine Momentaufnahme: eine Abfrage, die zwischen zwei Aktualisierungen fertig wurde, ist weg.

`db-stat-statements` ist die kumulative Sicht - die Top-Anweisungen nach Gesamtzeit seit dem letzten `pg_stat_statements_reset()`, jede so normalisiert, dass `WHERE id = 1` und `WHERE id = 2` eine Zeile sind. Hinweis-Flags je Zeile markieren Planinstabilität, `work_mem`-Überlauf (mit den Bytes pro Aufruf), eine niedrige Cache-Trefferquote und Anweisungen, die bei jedem einzelnen Aufruf langsam sind, nicht im Mittel. Gesamtzeit ist die Spalte, nach der man sortiert: eine Abfrage, die eine Million Mal am Tag 3 ms dauert, ist ein größerer Kostenpunkt als eine, die zweimal 20 Sekunden dauert. Das Widget kopiert den vollen Anweisungstext. `db-query-history` (Abschnitt 6.6) liefert dieselben Daten mit einer Uhr verknüpft.

Nach den Top-Anweisungen folgt **Optimistisches Sperren - veraltete Schreibvorgänge**: jedes UPDATE und DELETE der Sicht, dessen WHERE eine Versionsspalte mit einem Parameter vergleicht - Hibernates `@Version`, Rails' `lock_version`, `xmin` oder `ConcurrencyStamp` von Entity Framework, ein handgeschriebenes `version = $n`, eine Prüfung auf `updated_at` - mit `stale_%` = 1 - rows / calls: dem Anteil der Schreibvorgänge, die die Zeile geändert (oder gelöscht) vorfanden und nichts schrieben. Die Anwendung lädt jeden davon neu und versucht es wieder, eine Schleife, die die Datenbankseite sonst nie sieht. Eine Anweisung ist gelb, wenn mindestens 5% von mindestens 100 Aufrufen nichts schrieben, oder wenn 1.000 Aufrufe es taten, gleich welcher Anteil; dieselbe Anweisung unter den Top 25 trägt dasselbe Kennzeichen. Eine Anweisung, die mehr Zeilen schreibt, als sie aufgerufen wird (ein Stapel mit Versionsbedingung), steht ohne Anteil in der Liste, weil rows / calls dann nicht eine Zeile je Aufruf ist.

`db-temp-queries` und `db-temp-files` geht es um Überläufe auf die Platte. `db-temp-files` ist live - Dateien, die gerade jetzt geschrieben werden, nach PID gruppiert, sodass man die überlaufende Abfrage im Moment erwischt. `db-temp-queries` ist kumulativ aus `pg_stat_statements` - die chronischen Überläufer, also ein `work_mem`, das für den Sort oder Hash dieser Abfrage zu klein ist, oder eine Join-Reihenfolge, die der Planer falsch getroffen hat. Ein wenig Temp ist bei einer Berichtsabfrage normal; dieselbe Anweisung, die bei jedem Lauf Gigabyte überläuft, ist die zu behebende.

#### Sperren, Blockaden und feststeckende Transaktionen {#locks-blocking-and-stuck-transactions}

`db-blocking` ist die erste, die man in einem „die App hängt"-Vorfall öffnet: sie paart jede blockierte Sitzung mit genau der Sitzung, die sie blockiert (über `pg_blocking_pids()`), sodass der Kopf der Kette - die Transaktion, auf die alle anderen warten - eine Zeile ist, mit ihrer Abfrage und wie lange sie die Sperre hält. Diese eine abbrechen oder beenden, und die Warteschlange leert sich.

`db-locks` ist die rohe `pg_locks`-Sicht - jede gehaltene und erwartete Sperre, nicht nur die blockierenden. Nutzen Sie sie, wenn `db-blocking` leer ist und trotzdem etwas festzuhängen scheint, oder um zu sehen, *welcher* Sperrmodus im Spiel ist (ein `ShareLock`, der auf einen `RowExclusiveLock` wartet, ist ein Fremdschlüssel-Insert gegen eine anderswo aktualisierte Zeile; eine `AccessExclusiveLock`-Anforderung hinter einem langen `SELECT` ist eine DDL-Anweisung oder ein `VACUUM FULL`, das die Tabelle einfriert, bis das Lesen fertig ist).

`db-long-tx` listet Transaktionen, die länger als eine Minute offen sind, egal was sie tun. Eine lange Transaktion, die *arbeitet*, ist eine langsame Abfrage; eine, die **idle in transaction** ist, ist ein Anwendungsfehler und schlimmer als eine langsame Abfrage, weil sie ihren Snapshot und ihre Sperren hält, während sie nichts tut. `db-idle-in-tx` ist genau dieser zweite Fall für sich, mit der Dauer, die jede Sitzung mitten in der Transaktion geparkt ist. So oder so sind die Kosten dieselben: `VACUUM` kann keine Zeilenversion aufräumen, die neuer ist als die älteste offene Transaktion, clusterweit - also kann ein vergessenes `BEGIN` auf einem Laptop jede Tabelle auf dem Server aufblähen.

Die **Sessions**-Ansicht führt den Transaktionsblick weiter als die Prüfungen. Das Detailfeld einer Sitzung zeigt *Snapshot-Alter (backend_xmin)* - wie viele Transaktionen zurück ihr Snapshot fixiert ist - und, wo es zutrifft, eine rote Zeile: **diese Sitzung definiert derzeit den Cluster-xmin-Horizont**; Autovacuum ist überall blockiert, bis ihre Transaktion endet. Die Live-Sitzungsliste markiert dieselbe Sitzung, und die Blocking-Ansicht baut den Sperr-Warte-Graphen zu einem Baum, sodass die eine Transaktion, die alle anderen aufhält, eine einzige Zeile ist.

#### Vorbereitete (zweiphasige) Transaktionen {#prepared-two-phase-transactions}

`db-prepared-xacts` deckt die eine Art Transaktion ab, die überhaupt keine Sitzung hat. `PREPARE TRANSACTION 'gid'` schreibt die Transaktion auf die Platte und kehrt zurück; ein späteres `COMMIT PREPARED 'gid'` oder `ROLLBACK PREPARED 'gid'` schließt sie ab. Dazwischen behält sie jede Sperre, die sie hielt, und fixiert den Cluster-xmin-Horizont - und nichts in `pg_stat_activity` zeigt sie, weil sie keine Verbindung ist. Ein externer Transaktionsmanager (die JTA eines Java-App-Servers, Windows MSDTC, ein XA-Treiber), der eine Transaktion vorbereitet und dann abstürzt oder den Überblick verliert, lässt sie für immer vorbereitet. Das ist der klassische Vorfall „die Datenbank bläht sich ständig auf und jede Sitzung sieht in Ordnung aus".

Die Prüfung listet jede - `gid`, Eigentümer, Datenbank, wie lange her sie vorbereitet wurde, das Alter ihrer Transaktion und die Tabellen, die sie sperrt - und markiert alles älter als zehn Minuten als wahrscheinlich verwaist. Lösen Sie eine, verbunden mit **ihrer eigenen Datenbank**: `COMMIT PREPARED 'gid'`, wenn die Arbeit landen soll, `ROLLBACK PREPARED 'gid'`, wenn nicht.

**Die meisten Server können überhaupt keine vorbereiteten Transaktionen haben.** `max_prepared_transactions` ist standardmäßig `0`, und die Prüfung sagt das klar, statt eine leere Tabelle zu zeigen - nur über null anheben, wenn etwas tatsächlich `PREPARE TRANSACTION` braucht. Wenn es aktiviert ist, erscheint eine vorbereitete Transaktion, die gerade eine lebende Sitzung blockiert, in der Blocking-Ansicht als eigener Knoten (beschriftet `prepared 'gid'`, mit den beiden auflösenden Anweisungen zum Kopieren), und in `db-locks` werden ihre gehaltenen Sperren der gid zugeordnet, statt zu verschwinden.

#### Migrationshistorie {#migration-history}

`db-migrations` (Menü Cluster-Gesundheit) liest die Historientabellen, die die Migrationswerkzeuge in der verbundenen Datenbank führen - Flyway (`flyway_schema_history`, vor Flyway 5 `schema_version`), Liquibase (`databasechangelog`, `databasechangeloglock`), Alembic (`alembic_version`), golang-migrate und Rails (`schema_migrations`, unterschieden an der Spalte `dirty` von golang-migrate), Django (`django_migrations`), Prisma (`_prisma_migrations`), knex (`knex_migrations`, `knex_migrations_lock`) und Entity Framework Core (`__EFMigrationsHistory`) - in jedem Schema. Für jedes sagt sie, was zuletzt angewendet wurde, und markiert die Zustände, die den nächsten Deploy stoppen, mit der Zeile, die es zeigt, und wie man es bereinigt:

- **CRIT - eine gescheiterte Migration:** Flyway `success = false` (eine Migration, die nicht in einer Transaktion laufen konnte, etwa `CREATE INDEX CONCURRENTLY`, ist mittendrin gescheitert; Flyway verweigert jede Migration bis `flyway repair`), golang-migrate `dirty = true` (Schema von Hand reparieren, dann `migrate force`), Prisma eine Migration, die begonnen und weder beendet noch zurückgerollt wurde, vor zehn Minuten oder mehr (`prisma migrate resolve`).
- **Eine Sperre, die ein gestorbener Prozess zurückließ:** Liquibase `databasechangeloglock.locked = true` ist eine Warnung, sobald sie **5 Minuten** gehalten wird, und kritisch bei **60 Minuten** (eine lange Datenmigration hält sie zu Recht, darum nennt die Zeile, wer sie wann nahm; `liquibase releaseLocks` gibt sie frei); knex `is_locked = 1` ist eine Warnung (es speichert keine Zeit).
- **INFO:** Flyway-Migrationen außer der Reihe angewendet (`outOfOrder=true` - erlaubt, aber diese Datenbank lief sie in einer Reihenfolge, die eine frische Installation nicht hätte), mehr als ein Alembic-Head, Liquibase-Changesets mit `FAILED` unter `failOnError=false`.

Eine Tabelle, die dieses Login nicht lesen darf, wird als gefunden genannt und nicht erraten. Der Gesundheitsüberblick führt das Schlimmste davon als Zeile **MIGRATIONEN**, und nur, wenn die Datenbank eine solche Tabelle hat.

#### Queue-Tabellen und LISTEN/NOTIFY {#queue-tables-and-listen-notify}

`db-queues` (Menü Verbindungen und Abfragen) findet die als Job-Queues genutzten Tabellen: die, die die Anweisungen mit `FOR UPDATE SKIP LOCKED` in `pg_stat_statements` nennen (nur diese Datenbank), und die Tabellen der Bibliotheken, die ihre Jobs in PostgreSQL halten - pgmq, PgQ, que, graphile-worker und River. Für jede: wartende Jobs (nur fällige, nicht für später geplante), wie lange der älteste gewartet hat, gescheiterte Jobs, wo die Bibliothek sie festhält, lebende und tote Zeilen, Größe und das letzte Vacuum. Eine nur über ihre Anweisungen gefundene Tabelle hat keinen bekannten Zustand "wartend" - viele behalten erledigte Jobs -, also wird ihre älteste Zeile gezeigt und nicht bewertet.

- **Tote Zeilen** muss jedes Abholen überspringen, darum leidet eine Queue-Tabelle als erste, wenn Vacuum zurückfällt oder etwas den xmin-Horizont festhält. Warnung bei **50%** der Tabelle mit mindestens **10.000** toten Zeilen, oder bei **1.000.000** allein; kritisch bei **90%** mit **100.000**, oder bei **10.000.000**. Der Text nennt Anteil und Anzahl.
- **Warten:** Warnung, wenn der älteste fällige Job **5 Minuten** gewartet hat, kritisch bei **1 Stunde**. PgQ-Consumer werden ebenso nach ihrem Rückstand bewertet.
- **LISTEN/NOTIFY:** `pg_notification_queue_usage()` (PostgreSQL 9.5+) sagt, wie voll die eine clusterweite Benachrichtigungs-Warteschlange ist. Sie wird nur bis zu dem gekürzt, was der langsamste Listener gelesen hat, und eine Sitzung liest nur zwischen Transaktionen, also lässt ein Listener, der untätig in einer Transaktion steht, sie wachsen, bis jedes `NOTIFY` beim Commit scheitert. Warnung bei **10%** ihres Maximums oder **512 MB**, kritisch bei **50%** (ab da warnt PostgreSQL in seinem Log) oder **4 GB**; das Maximum ist 8 GB, oder ab PostgreSQL 17 `max_notify_queue_pages`. Die Sitzungen, deren letzte Anweisung `LISTEN` war, werden mit ihrem Zustand gelistet - PostgreSQL hat keine Sicht auf Listener, näher kommt man nicht heran. Dieselben Linien schalten die Lampe **LISTEN/NOTIFY-Warteschlange** in der Kopfzeile, die auf einem Cluster erscheint, auf dem jemand lauscht.

Die Zählungen sind begrenzt (eine Million Zeilen) und jede Abfrage hat fünf Sekunden, also wird ein riesiger Rückstand ohne Index als nicht gemessen gemeldet statt gelesen.

#### Die Cache- und Index-Advisor {#the-cache-and-index-advisors}

`db-cache-hit` ist die eine Zahl: der Anteil der Blocklesevorgänge, die seit dem letzten Statistik-Reset aus `shared_buffers` bedient wurden, für diese Datenbank. Es ist ein kumulativer Durchschnitt über die ganze Lebensdauer der Statistik, also drückt ein einmaliger kalter Bericht von vor einer Stunde ihn immer noch nach unten - für sich genommen sagt sie fast nichts. `db-cache-advisor` ist die Version, nach der zu handeln sich lohnt.

`db-cache-advisor` meldet die Trefferquoten für Heap, Index und TOAST, dann die Tabellen und Indexe mit den meisten physischen Lesevorgängen, `shared_buffers` gegen die Datengröße und eine Plausibilitätsprüfung für `effective_cache_size`. Eine Quote unter ~99% zählt nur, wenn die Fehlgriffe konzentriert sind - eine kalte Berichtstabelle, einmal am Tag gelesen, ist kein Grund zum Tunen. Handeln Sie nach der Liste der heißen Objekte, nicht nach der Kopfzahl.

`db-index-advisor` findet Tabellen mit hohem Anteil sequenzieller Scans und listet Kandidatenspalten aus den Prädikaten in `pg_stat_statements`, samt Lücken bei der führenden Spalte in vorhandenen Indexen und Fremdschlüsseln ohne deckenden Index. Für jeden schreibt er das `CREATE INDEX`. Zwei Fallen: eine kleine Tabelle soll sequenziell gescannt werden, ignorieren Sie also Kandidaten unter ein paar tausend Zeilen; und jeder Index kostet beim Schreiben, fügen Sie also den hinzu, den die heiße Abfrage braucht, nicht alle. Um einen bestimmten Index vor dem Bau zu bewerten, nutzen Sie den Index-Test (Abschnitt 6.5.5), der ihn mit HypoPG plant.

Sind AWR-Snapshots der Datenbank gespeichert, fügt `db-index-advisor` drei Verlaufsabschnitte hinzu, nach einem Block **Belege**, der die verwendeten Snapshots und ASH-Tage nennt. **Sequenzielle Scans über die gespeicherten Fenster**: pro Tabelle Scans und gelesene Zeilen pro Tag, Zeilen pro Scan und die erste Hälfte des Verlaufs gegen die zweite; *wachsend* heißt, die zweite Hälfte ist mindestens doppelt so hoch wie die erste und mindestens 100 pro Tag, oder mindestens 10.000 mehr pro Tag, und eine Tabelle wird gelistet, wenn sie wächst oder ihre sequenziellen Scans mindestens 100.000 Zeilen pro Tag lesen. Daneben die Anweisungen, die sie nennen und am meisten von der Platte lesen. **Anweisungen, die am meisten von der Platte lesen**: `shared_blks_read` aus `pg_stat_statements`, summiert über die Fenster, pro Aufruf, mit dem geschäftigsten Fenster und wie oft ASH die Anweisung beim Warten auf I/O erwischte (ab PostgreSQL 14 per Query-ID zugeordnet). **Keine Scans über das gespeicherte Fenster**: die Scans jedes Index, summiert über die Fenster (ein Zurücksetzen der Statistik dazwischen beginnt eine neue Zählung, nichts geht verloren), mit genanntem Fenster. Über mindestens **7 Tage** ist ein Index ohne Scans ein *Kandidat zur Prüfung*, darunter ist das Fenster *zu kurz für ein Urteil*, und jede Zeile trägt den Hinweis, dass ein Monats-, Quartals- oder Jahresbericht sein einziger Leser sein kann. Dort wird kein `DROP` angeboten, und Primärschlüssel, eindeutige Indizes und Fremdschlüssel-Indizes sind nie Kandidaten: sie werden getrennt als behalten gelistet, mit dem Grund.

#### Index-Pflege {#index-housekeeping}

Drei Prüfungen finden Indexe, die Schreibvorgänge und Platz kosten und nichts zurückbringen. Alle drei lesen `pg_stat_user_indexes`, ihre Scan-Zähler sind also **seit dem letzten Statistik-Reset** - ein `pg_stat_statements_reset()` oder ein `pg_stat_reset()` letzte Woche heißt, „nie gescannt" deckt nur eine Woche ab.

`db-index-usage` ist das ganze Bild: jeder Index mit seiner Scan-Zahl, Größe und dem Verhältnis von Scans zu Tabellen-Schreibvorgängen. `db-index-unused` ist die Teilmenge, die zählt - Indexe mit null Scans -, und schließt die schon aus, für die eine Null normal ist: die hinter einem `PRIMARY KEY`, einem `UNIQUE`- oder `EXCLUDE`-Constraint und der Replica Identity einer Tabelle. Die erzwingen etwas, ob eine Abfrage sie liest oder nicht, und `DROP INDEX` verweigert sie, eine Drop-Liste mit ihnen wäre also gefährlich. Sie lässt auch jeden Index auf einer **Fremdschlüssel**-Spalte aus: er liest sich als nie gescannt, weil `DELETE` und `UPDATE` auf der Elterntabelle ihn intern nutzen, und ohne ihn scannt jede davon die Kindtabelle unter einer Sperre. Diese werden gesondert aufgeführt, jeweils mit dem Fremdschlüssel, den sie stützen, und nie zum Löschen angeboten - weder hier noch im Gesundheitsüberblick, beim Index-Bloat oder im Tabellen-Advisor. Was übrig bleibt, ist ein **Kandidat zur Prüfung, kein Urteil**: die Prüfung schreibt das `DROP INDEX CONCURRENTLY` für jeden und sagt, wie lange ihre Zähler reichen, denn ein Index, den nur ein Monats-, Quartals- oder Jahresbericht nutzt, liest sich die ganze Zeit dazwischen als ungenutzt - bestätigen Sie es über einen Zeitraum, der einen solchen einschließt.

`db-duplicate-indexes` findet Indizes, die einander wiederholen. **Identische** - gleiche Tabelle, Spalten, Operatorklassen, Ausdrücke und Prädikat - erscheinen als Gruppe, und die Prüfung nennt den, der **bleibt**, und warum: den, der etwas erzwingt oder bedient (ein Primärschlüssel, ein `UNIQUE`- oder `EXCLUDE`-Constraint, ein eindeutiger Index, die Replica Identity, der Index eines Fremdschlüssels), sonst den meistgescannten. Die anderen sind als **redundant - identisch mit** dem bleibenden markiert. Wenn zwei davon beide etwas erzwingen (zwei `UNIQUE`-Constraints auf denselben Spalten), sagt sie das stattdessen: einen zu entfernen heißt, einen Constraint zu ändern. Sie listet auch einen einfachen Index, dessen Spalten die **führenden Spalten eines breiteren** mit denselben Operatorklassen sind - der breitere beantwortet dieselben Suchen, etwas größer zu lesen; eindeutige und Constraint-Indizes erscheinen dort nie, und wenn der schmalere einen Fremdschlüssel stützt, deckt ihn der breitere weiter ab. Die Prüfung schreibt kein `DROP` - ein redundanter Index kostet Schreibvorgänge und Platte, und die Entscheidung liegt bei Ihnen. Bewahren Sie die Definition auf, bevor Sie einen entfernen - sie ist die einzige Kopie.

Um an einem einzelnen von den Berichten benannten Index zu handeln - das exakte `REINDEX` oder `DROP`, seine Scan-Historie und ein Abgleich mit jedem anderen Index derselben Tabelle -, klicken Sie seinen Namen an, um den Per-Index-Advisor zu öffnen (siehe *Ein Index, und was damit zu tun ist* oben).

#### Wohin der Platz geht: die Größenberichte {#where-the-space-goes}

Vier Berichte beantworten „was belegt die Platte", jeder eine Ebene feiner als
der vorige. Sie sind reine Messung - kein Advisor, keine Empfehlung -, also
kann man sie nur falsch lesen, indem man missversteht, was eine Zahl
einschließt.

`db-sizes` - **jede Datenbank im Cluster.** `pg_database_size()` je Datenbank,
dazu die Zahl der lebenden Verbindungen (nur Client-Backends - Autovacuum-
Worker und Hintergrundprozesse tragen ebenfalls einen `datname`, und sie
mitzuzählen ließe eine untätige Datenbank beschäftigt wirken), die
Cache-Trefferquote seit dem letzten Statistik-Reset und die kumulierte
Nutzung temporärer Dateien. Template-Datenbanken bleiben außen vor. Das ist
der einzige Bericht hier, der über Datenbanken hinweg sieht; die Verbindung,
auf der Sie sind, spielt für seine Zahlen keine Rolle.

`db-schema-sizes` - **die verbundene Datenbank, nach Schema aufgeschlüsselt.**
Gesamt je Schema, Tabellendaten- und Index-Bytes getrennt ausgewiesen, der
Prozentanteil jedes Schemas an der Datenbank, die Objektzahlen und die größte
einzelne Tabelle je Schema. Die Gesamtsumme zählt einmal alles, was im Schema
Platz belegt: Tabellen, ihr TOAST, ihre Indizes, materialisierte Sichten,
Sequenzen. Sie kann nicht in andere Datenbanken übergreifen, weil
`pg_namespace` je Datenbank gilt.

`db-table-sizes` - **die einzelne Tabelle**, Top 25 nach Gesamtgröße. `total`
ist `pg_total_relation_size()` - Heap **plus** seine TOAST-Tabelle und deren
TOAST-Index **plus** jeder Index darauf **plus** die Free-Space- und
Visibility-Maps. `table_only` ist der Heap allein; `indexes` ist
`pg_indexes_size()`. Also ist `total` minus `table_only` minus `indexes` das
TOAST und die Maps, was bei einer Tabelle mit breiten `text`/`jsonb`-Spalten
der größte Teil der Größe sein kann. Partitionen werden auf die
Partitionswurzel aufgerollt - eine partitionierte Tabelle erscheint als eine
Zeile mit einer `partitions`-Zahl, nicht als Wand aus Kindzeilen mit dem
Elternteil bei null. `live_rows` und `dead_rows` sind **Schätzungen** des
Statistiksammlers, kein `count(*)`; `stats_age` ist, wie lange der letzte
`ANALYZE` her ist, und ein großer `stats_age` bedeutet, dass die Zeilenzahlen
in der Richtung veraltet sind, in die sich die Tabelle bewegt hat.

`db-index-sizes` - **der einzelne Index**, Top 50 nach Indexgröße, jeder mit
der Größe seiner Elterntabelle daneben, sodass ein Index, der größer geworden
ist als die von ihm indizierten Daten, sofort auffällt. Dieses Verhältnis ist
das Signal: ein Btree-Index, der größer ist als seine Tabelle, indiziert
entweder sehr breite Schlüssel oder ist aufgebläht (siehe nächster Abschnitt),
und in beiden Fällen lohnt ein Blick. Ein GIN- oder GiST-Index, der seine
Tabelle legitim in den Schatten stellt, ist normal - sie speichern weit mehr
als einen Eintrag je Zeile.

Die Falle, die allen vieren gemein ist: das sind **Größen auf der Platte**,
die tote Tupel und freien Platz einschließen, den ein `VACUUM FULL` oder
`pg_repack` zurückholen würde. Eine Tabelle, die 4 GB anzeigt, ist keine
4 GB Daten, wenn die Hälfte davon Bloat ist. Gleichen Sie eine überraschende
Größe mit `db-bloat-breakdown` oder `db-pgstattuple` ab, bevor Sie auf eine
größere Platte schließen.

#### Bloat: die Schätzung und die echte Zahl {#bloat-the-estimate-and-the-real-number}

„Bloat" ist Platz, den eine Relation auf der Platte über das hinaus belegt,
was ihre lebenden Zeilen brauchen - tote Tupel, die Autovacuum nicht
zurückgeholt hat, und Seiten, die nach Löschungen und Updates halb leer
zurückblieben. Drei Berichte behandeln ihn, von schnell-und-ungefähr bis
langsam-und-exakt, und jeder schätzungsbasierte davon - die
Per-Tabelle-Aufschlüsselung, die Index-Rangliste, der Advisor-Tab und die
Zeile *WASTED SPACE* des Gesundheitsüberblicks - wird vom selben Code
berechnet und landet für dieselbe Relation auf derselben Zahl, weil sie alle
ein gemeinsames Modul aufrufen, statt es jeweils selbst zu berechnen. **Lesen
Sie die Schätzungen als „einen Blick wert", nie als Messung.**

`db-bloat-breakdown` ist der richtige Ausgangspunkt. Er liest je Tabelle
echte Größen (`pg_relation_size`, `pg_indexes_size`, `pg_total_relation_size`,
dazu `pg_total_relation_size` der TOAST-Relation) und kombiniert sie mit
einer *erwarteten* („gepackten") Größe, die aus `pg_stats` (Spaltenzahl,
mittlere Breite, Null-Anteile) für den Heap berechnet wird, demselben
Pro-Eintrag-Modell für jeden Btree-Index, und dem eigenen
Tot-Tupel-Verhältnis der TOAST-Tabelle - ein einziges `est_waste` /
`est_waste_pct` für die **gesamte Tabelle**, Heap und TOAST und jeder Index
zusammengezählt, nicht der Heap allein. Das behebt den Fall, dass eine
Tabelle hier früher „0% Bloat" zeigte, während allein ihre Indizes Hunderte
Gigabyte geschätzter Verschwendung trugen: Die Zahl schließt sie jetzt ein.
Er gibt eine Spalte `recommendation` aus, die aus denselben Summen abgeleitet
ist, die er zeigt, sodass Ratschlag und Zahl einander nicht mehr
widersprechen können. Lesen Sie diese Spalte als **Anstoß, nicht Anweisung**
- sie weiß nicht, ob Sie `pg_repack` installiert haben. Eine Zeile „VACUUM
FULL or pg_repack" auf einer 20-MB-Tabelle erledigt ein einfaches `VACUUM`;
dieselbe Zeile auf einer belasteten 200-GB-Tabelle bedeutet einen
Online-Rebuild und einen Wartungsplan.

**Wenn die Schätzung einer Komponente unzuverlässig ist, wird deswegen nicht
die ganze Zeile geleert.** Die gepackte Größe einer einzelnen Komponente -
meist der Heap einer großen oder lange nicht analysierten Tabelle - kann
*größer* herauskommen als die echte Größe dieser Komponente: veraltete
Statistiken, oder Zeilen, die ungewöhnlich breit oder stark getoastet sind,
als dass das Pro-Spalte-Modell von `pg_stats` noch passt. Das heißt nicht
„keine Verschwendung", sondern „die Schätzung dieser Komponente greift hier
nicht" - und was die *anderen* Komponenten (TOAST, jeder Index) weiterhin
wissen, wird deswegen nicht verworfen. `est_waste` wird stattdessen zu einer
**Untergrenze**: mit `≥` markiert und mit Angabe, was fehlt, z. B.
`≥ 448 GB (heap unknown)`, statt einer erfundenen Null oder eines leeren
„n/a" - die wahre Summe liegt bei diesem Wert oder höher, nie darunter.
`est_waste_pct` trägt dieselbe `≥`-Markierung, wenn sie zutrifft. `ANALYZE`,
oder `db-pgstattuple` weiter unten, führt speziell an der fehlenden
Komponente vorbei - der Hinweis der Zeile selbst nennt welche und warum: nie
analysiert, Statistik seit N Tagen veraltet mit einem Anteil geänderter
Zeilen seither (beides führt zu `ANALYZE`), oder aktuelle Statistik, die
trotzdem nicht zur Form der Tabelle passt (führt stattdessen zu
`db-pgstattuple`, da ein erneutes `ANALYZE` nichts ändern würde). Diese
Regel - bekannte Komponenten bleiben sichtbar, fehlende werden benannt,
nichts wird geleert - gilt überall, wo eine Aufschlüsselung mehrere
Komponenten summiert, auch bei der Zeile *WASTED SPACE* der
Gesundheitsübersicht.

**`est_waste_pct` und `dead_pct` sind absichtlich unterschiedliche Zahlen.**
`est_waste_pct` ist Platz: Bytes, die die Tabelle (Heap, TOAST, Indizes)
hält, aber nicht nutzt. `dead_pct` ist `n_dead_tup / (n_live_tup +
n_dead_tup)` - eine Tupel-Zählung, und nur ein Maß dafür, wie weit Autovacuum
im Rückstand ist. Eine Tabelle, die stark per `UPDATE` durchgewalkt wird,
kann gleichzeitig 4% tot und 90% verschwendet sein. `idx_to_heap` ist
Gesamt-Index-Bytes über Heap-Bytes, wo deutlich über 1 eine Prüfung auf
ungenutzte/doppelte Indizes wert ist.

`db-bloat-indexes` beantwortet eine andere Frage: nicht „welche Tabelle ist
am schlimmsten", sondern „welcher einzelne Index, wo auch immer im Cluster,
ist gerade ein `REINDEX` wert" - dieselbe Pro-Index-Arithmetik wie die
Index-Zeilen der Aufschlüsselung, aber clusterweit gereiht statt nach
Tabelle gruppiert. Er nimmt `fillfactor` 90 an und deckt **nur Btree** ab -
GIN-, GiST-, BRIN-, Hash- und Ausdrucks-Indizes bekommen eine echte Größe und
keine Schätzung, was der Schätzer ist, der aussteigt, kein sauberer Index.
**`VACUUM` verdichtet einen Index nie**; die genannte Abhilfe ist `REINDEX`
(`... CONCURRENTLY` ab PostgreSQL 12, geprüft gegen den verbundenen Server,
damit ein älterer nicht zu einer Anweisung geraten wird, die er nicht hat),
eine Zeile pro Index. Eine Tabelle, die absichtlich mit `fillfactor` unter
100 gebaut wurde, um HOT-Updates billig zu halten, zeigt diesen freien Platz
hier ebenfalls als „Bloat" - es ist der Platz, den Sie angefordert haben,
keine Verschwendung.

`db-pgstattuple` ist die echte Zahl, für beide Hälften. Er liest **jede
Seite** jeder Tabelle über die Erweiterung `pgstattuple` für die wahren
Prozentsätze toter Tupel und freien Platzes, dann `pgstatindex()` auf den
Btree-Indizes dieser Tabelle für eine echte Blattseiten-Dichte je Index - auf
keiner Seite ein `pg_stats`-Modell, sondern ein echter Seitenscan. Er ist
exakt und so langsam, wie dieser Scan es nahelegt: die Kosten sind die
Tabellengröße in I/O, jedes Mal. Nutzen Sie ihn, um eine bestimmte von der
Schätzung markierte Tabelle zu bestätigen - oder eine, die sie als
unzuverlässig markiert hat -, nicht um eine Datenbank zu überblicken. Die
Größenbereichs-Parameter sind ein Zeitbudget: die Voreinstellung überspringt
Tabellen unter 1 MB (Bloat dort ist ein Rundungsfehler) und über 5 GB
(erhöhen Sie das, wenn Sie gezielt die exakte Zahl auf einer großen Tabelle
wollen und warten können). Die Erweiterung bietet auch `pgstattuple_approx`,
das Seiten überspringt, die die Visibility-Map als vollständig sichtbar
markiert, und so auf einer gut vacuumten Tabelle weit schneller läuft, zum
Preis etwas geringerer Präzision - gut zu wissen, wenn der volle Scan zu
langsam ist.

**Eine praktische Reihenfolge:** `db-bloat-breakdown`, um die Form über alle
Tabellen zu sehen, `db-bloat-indexes` für eine clusterweit gereihte Liste von
Index-Verdächtigen, `db-pgstattuple` auf den ein oder zwei Tabellen, an denen
Sie wirklich handeln werden (oder jeder als unzuverlässig markierten Zeile).
Dann der Per-Index-Advisor (oben) für das exakte `REINDEX`- oder
`VACUUM`-Statement.

#### Shared Buffers: was wirklich im Cache ist {#shared-buffers-what-is-actually-cached}

`db-buffer-cache` öffnet `pg_buffercache` und zeigt, welche Relationen den
Shared-Buffer-Pool von PostgreSQL gerade jetzt belegen - die Top 30 nach
Bufferzahl, mit der Gesamtgröße jeder Relation, ihrem Anteil an
`shared_buffers` und wie viel von der Relation selbst im Cache ist.

**Die Kosten sind real und wert, sie zu kennen, bevor Sie das in Produktion
ausführen.** Die Sicht `pg_buffercache` durchläuft den gesamten Buffer-Pool;
auf einem Server mit `shared_buffers` von mehreren Gigabyte sind das Millionen
Buffer, geprüft unter Partition-Locks, und es zeigt sich als kurzer
Latenzanstieg für alle anderen. Führen Sie sie aus, wenn Sie sie brauchen,
nicht nach Zeitplan, und bevorzugt in einem ruhigen Moment. Ab PostgreSQL 16
liefern die billigeren Funktionen `pg_buffercache_summary()` und
`pg_buffercache_usage_counts()` das Gesamtbild ohne den Buffer-für-Buffer-
Durchlauf - nutzen Sie die, wenn Sie nur „wie voll ist der Cache" wollen.

Wie die zwei Prozentsätze zu lesen sind:

- **`pct_of_relation`** - wie viel von dieser Relation im Cache ist. Eine
  kleine, heiße Lookup-Tabelle bei 100% ist genau richtig. Eine 200-GB-
  Tabelle bei 3% ist ebenfalls richtig - Sie wollten nicht, dass sie alles
  andere verdrängt. Diese Zahl ist nur ein Problem, wenn eine Tabelle, von
  der Sie *erwarten*, dass sie aus dem Speicher bedient wird, niedrig liegt.
- **`pct_of_buffers`** - der Anteil dieser Relation am ganzen Pool. Ein Alarm
  nur, wenn eine oder zwei Relationen ihn dominieren und die Tabellen, die
  Ihnen wichtig sind, ausgehungert werden. Ein gesunder Pool ist meist ein
  langer Schwanz.

Wenn oben in der Liste TOAST-Tabellen und Indizes stehen, die Sie nicht
erkennen, ist das normal - Index-Seiten und TOAST-Chunks werden wie alles
andere gecacht. Die Prüfung beantwortet „warum bleibt mein Working Set nicht
im Speicher", und die Antwort ist meist entweder, dass `shared_buffers` zu
klein für den Working Set ist oder dass ein Batch-Job wiederholt eine große
kalte Tabelle hindurchzieht.

#### TimescaleDB {#timescaledb}

`db-timescaledb` liest die Erweiterung TimescaleDB der Datenbank, mit der Sie
verbunden sind. Für alles Folgende braucht es TimescaleDB 2; mit 1.x sagt es
das, und der Schema-Browser zeigt weiterhin die Hypertables und ihre Chunks.

**Edition.** Die erste Zeile nennt die Version und die Lizenzeinstellung
(`timescaledb.license`). Unter `apache` - der reinen Apache-Edition - lehnt
der Server selbst Kompression, Continuous Aggregates, die Aufbewahrungs-,
Kompressions-, Aktualisierungs- und Reorder-Richtlinien und
benutzerdefinierte Jobs ab; Hypertables, Chunks, `drop_chunks` und
`time_bucket` funktionieren. Das Panel graut diese Aktionen aus und sagt
warum. Ist auf dem Server eine neuere Version installiert als in der
Datenbank, wird sie mit dem `ALTER EXTENSION timescaledb UPDATE` genannt, das
sie anwendet.

**Im Schema-Browser** haben Hypertables eine eigene Kategorie; jede sagt, wie
sie in Chunks geteilt ist (Zeitspalte und Chunk-Intervall, dazu die
Raumpartitionen, falls vorhanden), wie viele Chunks sie hat und wie viele
komprimiert sind; ihre Zeilenzahl ist die Summe ihrer Chunks, weil die
eigene Tabelle der Hypertable leer ist. Geöffnet zeigt sie ihre Größe, das
Kompressionsverhältnis und ihre Chunks, die neuesten zuerst (die neuesten
500; der Rest wird gezählt). Ein Chunk öffnet sich als die Tabelle, die er
ist, und sein Menü komprimiert oder dekomprimiert ihn. Die Schemas von
TimescaleDB (`_timescaledb_internal`, `_timescaledb_catalog`,
`timescaledb_information` und die anderen) und die Relationen, die es
betreibt - Chunks, komprimierte Daten, die Tabellen und Sichten hinter
Continuous Aggregates - sind wie andere Objekte von Erweiterungen
ausgeblendet, hinter demselben Schalter "anzeigen".

**Was die Prüfung beurteilt**, mit den Zahlen, nach denen sie urteilt:

- **Ein Job, der immer wieder fehlschlägt** - zwei fehlgeschlagene Läufe
  nacheinander seit seinem letzten Erfolg (gezählt aus
  `timescaledb_information.job_errors`; vor 2.9, das diese Sicht nicht hat,
  ist nur der letzte Lauf bekannt). Ein Alarm.
- **Ein Job, der nicht nach Zeitplan lief** - sein nächster Start liegt mehr
  als 5 Minuten in der Vergangenheit. Der Scheduler startet einen Job binnen
  Sekunden, wenn ein Hintergrundprozess frei ist, Minuten Verspätung heißen
  also, dass keiner frei ist. Ein Alarm.
- **Der Scheduler läuft nicht** für diese Datenbank, während Jobs geplant
  sind - meist ist `timescaledb.max_background_workers` zu niedrig (es braucht
  einen Prozess pro Datenbank mit TimescaleDB plus einen pro gleichzeitig
  laufendem Job) oder `max_worker_processes` ist erschöpft. Ein Alarm.
- TimescaleDBs eigene Wartungsjobs (Telemetrie, Aufbewahrung der
  Job-Historie) werden gemeldet, wenn sie fehlschlagen, nie als Alarm: die
  Telemetrie schlägt auf jedem Server ohne Internetzugang fehl.
- **Kompression hinter ihrer Richtlinie** - Chunks älter als das
  `compress_after` der Richtlinie, die noch unkomprimiert sind, jeder mit
  einer Schaltfläche zum Komprimieren.
- **Aufbewahrung hinter ihrer Richtlinie** - Chunks älter als `drop_after`
  sind noch da.
- **Ein Continuous Aggregate fällt zurück** - die Zeit von seinem Watermark
  (was materialisiert ist) bis jetzt, gemessen an dem, worin seine
  Aktualisierungsrichtlinie es halten sollte: `end_offset` plus Zeitplan.
  Markiert wird ab dem Doppelten davon und ab 15 Minuten, sodass eine
  Richtlinie, die jede Minute aktualisiert, nicht für einen kurzen Verzug
  markiert wird. Ohne Richtlinie wird es in jedem Fall gesagt, als Warnung,
  wenn das Aggregat `materialized_only` ist (Abfragen sehen dann nichts nach
  dem Watermark).
- **Chunk-Intervall gegen Speicher** - die Regel von Timescale: die Chunks,
  in die geschrieben wird, samt Indizes, aller Hypertables zusammen sollten
  in etwa 25% des Hauptspeichers passen, den Anteil, den `timescaledb-tune`
  `shared_buffers` gibt. Die Prüfung misst das neueste abgeschlossene
  Intervall jeder Hypertable, in die in den letzten 7 Tagen geschrieben
  wurde, und vergleicht ihre Summe mit `shared_buffers`; darüber bekommt jede
  Hypertable ein Intervall, das passend verkleinert ist. Auch das Gegenteil
  wird markiert: mehr als 1000 Chunks, deren Intervall unter 1% von
  `shared_buffers` oder unter 10 MB hält - jeder Chunk ist eine Tabelle, die
  der Planer betrachtet. Ein neues Intervall (`set_chunk_time_interval`) gilt
  für Chunks, die ab dann entstehen.

Eine Warnung oder ein Alarm in der Liste über dem Panel hat die übliche
Schaltfläche "Zur Aufgabenliste hinzufügen", und die Alarme schalten die
TimescaleDB-Lampe in der Kopfzeile ein (dieselbe Lesung, die Health als
TIMESCALEDB-JOBS zeigt).

**Aktionen.** Im Panel: eine Kompressions- oder Aufbewahrungsrichtlinie
hinzufügen oder entfernen, einen Chunk komprimieren, ein Continuous Aggregate
für ein Fenster aktualisieren, einen Job jetzt ausführen, seinen Zeitplan
ändern, ihn pausieren und fortsetzen; im Menü eines Chunks: ihn komprimieren
oder dekomprimieren. Jede zeigt zuerst ihre Anweisung und die, die sie
rückgängig macht - das Entfernen einer Richtlinie behält ihre Einstellung in
der Anweisung, die sie wiederherstellt -, und beide gehen ins Audit-Log. Das
darf nur der Eigentümer der Hypertable, des Aggregats oder des Jobs oder ein
Mitglied dieser Rolle; das Panel fragt `pg_has_role(owner, 'USAGE')` und
graut aus, was Sie nicht dürfen, und der Server fragt noch einmal.

**Der Rest der Anwendung auf einer Datenbank mit TimescaleDB:**

- Der Vergleich vergleicht eine Hypertable als Tabelle und zusätzlich danach,
  wie sie in Chunks geteilt und komprimiert ist, sodass eine normale Tabelle
  und eine Hypertable oder zwei Chunk-Intervalle sich unterscheiden. Chunks
  werden nicht verglichen: sie sind in der Reihenfolge ihres Entstehens
  benannt, und `_hyper_1_5_chunk` auf der einen Seite ist ein anderer
  Zeitraum als derselbe Name auf der anderen. Ein anderes Intervall bekommt
  ein `set_chunk_time_interval`; eine Tabelle in eine Hypertable zu
  verwandeln bleibt eine auskommentierte Anweisung, weil das jede Zeile
  verschiebt.
- Die Wiederherstellung umgibt einen Dump mit TimescaleDB so, wie TimescaleDB
  es verlangt: `timescaledb_pre_restore()` davor, `timescaledb_post_restore()`
  danach - auch wenn die Wiederherstellung fehlschlägt, sodass die Datenbank
  nie mit angehaltenen Jobs zurückbleibt. In einem leeren Ziel wird die
  Erweiterung zuerst angelegt. Das Ziel muss die TimescaleDB-Version haben,
  aus der der Dump stammt. Vor dem Wiederherstellen nur eines Teils eines
  solchen Dumps oder mit `--clean` wird gewarnt: eine Hypertable ohne ihre
  Katalogzeilen und Chunks kommt als leere normale Tabelle zurück.
- Den Eigentümer einer Hypertable zu ändern nimmt ihre Chunks mit (das tut
  TimescaleDB). Die Kopie der Definitionen vor einer Änderung enthält die
  Hypertable, nicht ihre Chunks, und die Anweisung, die sie wieder zur
  Hypertable macht.
- Die Beziehungskarte zeichnet eine Hypertable als eine Karte mit ihrer
  Chunk-Zahl und verbindet ein Continuous Aggregate mit der Hypertable, die
  es aggregiert.
- Andere Prüfungen, die Tabellen auflisten (Größen, Bloat, Vacuum), nennen
  neben jedem Chunk die Hypertable oder das Continuous Aggregate.

#### Vacuum, Bloat und Tabellenstatistik {#vacuum-bloat-and-table-statistics}

`db-vacuum` zeigt je Tabelle, wann sie zuletzt vacuumt und analysiert wurde, die Anzahl lebender und toter Tupel, den seit der letzten Analyse geänderten Prozentsatz und ein Hitze-Label, mit einem empfohlenen Satz Autovacuum-Reloptions neben dem aktuellen. Die Tags zählen am meisten: **WRAPAROUND-Risiko** ist sofort; **Autovacuum deaktiviert** auf einer belasteten Tabelle ist meist ein vergessener Fehler; eine **insert-only**-Tabelle braucht `autovacuum_vacuum_insert_threshold`, da die Standard-Vacuum-Schwelle für sie nie auslöst.

`db-autovacuum` ist die Live-Sicht: welche Autovacuum-Worker gerade jetzt laufen und was jeder tut. Leer ist der Normalzustand. Ein Worker, der lange an derselben großen Tabelle ist, oder `max` Worker alle beschäftigt, während der Vacuum-Rückstand in `db-vacuum` weiter wächst, heißt, Autovacuum kommt nicht hinterher - erhöhen Sie `autovacuum_max_workers` oder `autovacuum_vacuum_cost_limit`, oder senken Sie den Per-Tabelle-Scale-Factor bei den schlimmsten Fällen.

`db-bloat-breakdown` trennt Heap-, TOAST- und Index-Bloat je Tabelle, fasst sie zu einem `est_waste` für die gesamte Tabelle zusammen und benennt den nächsten Schritt - VACUUM, REINDEX, `pg_repack` oder nichts. Seine Spalte `recommendation` ist ein aus denselben Summen abgeleiteter Anstoß, die sie zeigt, keine Anweisung: *Bloat: die Schätzung und die echte Zahl* (oben) behandelt, was sie berücksichtigt und was nicht, und wie sich die clusterweite Index-Rangliste (`db-bloat-indexes`) und der exakte Scan (`db-pgstattuple`) darum herum einordnen.

Die Zeile *WASTED SPACE (est.)* im **Gesundheitsüberblick** verwendet dieselbe kombinierte Heap-plus-TOAST-plus-Index-Schätzung, über die Datenbank aggregiert, und markiert eine Tabelle, **je nachdem, was zuerst auslöst**: absolute **20 GB** geschätzter Verschwendung in einer Tabelle (rot bei **200 GB**), oder **25% oder mehr** der Tabelle verschwendet, sobald das mindestens **1 GB** ist. Die absolute Zahl gibt es, weil 40% einer 200-MB-Tabelle Hausarbeit sind, während 6% einer 20-TB-Tabelle mehr als ein Terabyte Platte und Stunden zusätzlich bei jeder Wiederherstellung bedeuten. Eine Tabelle mit einer unzuverlässigen Komponente (meist Heap) wird aus dieser Zeile **nicht** entfernt - ihre übrigen bekannten Komponenten zählen weiterhin, als `≥`-Untergrenze, nach derselben Regel wie die Zeile pro Tabelle in `db-bloat-breakdown`; dort steht auch, welche Komponente fehlt und warum. Der Aufriss listet die schlimmsten Tabellen, und der Per-Tabelle-Advisor-Tab schlüsselt jede nach Komponente auf. `pg_repack` und `VACUUM FULL` geben Heap-/TOAST-Platz an das Betriebssystem zurück; ein einfaches `VACUUM` gibt ihn nur zur Wiederverwendung innerhalb derselben Tabelle frei; ein Index braucht `REINDEX`.

Der **Advisor-Tab** im Schema-Browser (Tabelle wählen -> Advisor) ist die Per-Tabelle-Version von all dem an einem Ort. Er zeigt eine **Bloat-Aufschlüsselung** je Komponente - Heap, TOAST, jeder Index - mit je drei Zahlen: was auf der Platte liegt (exakt), wie viel davon geschätzte **nützliche Daten** sind und wie viel geschätzte **Verschwendung**. Index-*Größe* ist keine Verschwendung; Index-Verschwendung ist der Platz, den ein frisches `REINDEX` nicht bräuchte. Großes TOAST ist für sich kein Problem - Verschwendung darin schon. Die „Verschwendung" ist an der echten Größe auf der Platte verankert (dieselbe Arithmetik wie `db-bloat-breakdown`), sodass sie auch bei veraltetem `pg_class.relpages` stimmt, und die Zeile einer einzelnen Komponente zeigt `?` statt einer Zahl, wenn ihre Eingaben nicht passen. Die Zeile **gesamt** tut das nie: Sie wird mit `≥` markiert und nennt die fehlende Komponente (mit einer kurzen Begründung und `ANALYZE`-/Genauer-Scan-Aktionen, wo passend), sobald eine Komponente nicht geschätzt werden konnte, statt eine erfundene Null oder eine leere Zahl zu zeigen. Die Schätzungen nehmen einen Fillfactor an und die TOAST-Zahl ist nur ein Dead-Tuple-Verhältnis, also führen Sie *Genauer Scan (pgstattuple)* für die echten Zahlen aus, bevor Sie bei etwas Großem handeln. Der Block **Analyse & Vorschläge** darunter macht aus den Zahlen bedingte Ratschläge mit fertig kopierbarem SQL: wiederverwendbare Verschwendung, bei der Autovacuum mitkommt, bleibt in Ruhe; eine aufgeblähte Datei bekommt `pg_repack` oder `VACUUM FULL`; wenn Autovacuum zurückfällt, wird auf die Per-Tabelle-Einstellungen verlinkt, die der Autovacuum-Advisor aus den Zahlen dieser Tabelle berechnet hat; aufgeblähte Indexe bekommen ein Per-Index-`REINDEX INDEX CONCURRENTLY` (oder einfaches `REINDEX` auf PostgreSQL < 12, markiert); TOAST-Verschwendung wird gesondert genannt, weil ein einfaches `VACUUM` sie nicht kompaktiert.

Der Advisor-Tab besteht aus fünf Abschnitten, in denselben Karten wie ein Prüfergebnis, mit einer Leiste unter dem Titel, die zu jedem springt: **Tabelle** (ihre Zahlen), **Vacuum & Autovacuum**, **Indizes**, **Wartung** und **Tiefenanalyse**. Der Vacuum-Abschnitt beginnt damit, wie Vacuum diese Tabelle behandelt hat - wie viele Läufe seit dem Zurücksetzen der Statistik und wie viele pro Tag, der Abstand zwischen ihnen, auf PostgreSQL 18 wie lange sie pro Tag dauern, der Anteil toter Zeilen und der älteste Snapshot, den irgendetwas in der Datenbank hält - und sagt mit denselben Worten wie `db-vacuum`, wann eine Tabelle stündlich oder öfter gesäubert wird, wann ihre Vacuums nichts bewirken, weil eine Transaktion, eine vorbereitete Transaktion, ein Replikationsslot oder ein Standby einen Snapshot hält, der älter ist als ihre toten Zeilen (und was zu beenden ist), wann jedes Vacuum ein aggressives ist und wann Autovacuum ohne Bremse läuft. Eine Empfehlung verschärft nie die Schwellen einer Tabelle, die schon stündlich gesäubert wird. Dann folgen die Auslöserzeilen - wann Autovacuum heute und nach der Empfehlung anspringt -, die vier Parameter und die Schaltflächen, die sie anwenden. Hält eine Sperre Vacuum von der Tabelle fern, steht dieser Befund zuerst und nennt die Sitzung, die sie hält.

**Wann häufiges Vacuum in Ordnung ist.** Eine Tabelle, die stündlich oder öfter gesäubert wird - mindestens 24 Läufe am Tag *und* mindestens 50 Läufe im Statistikfenster; viertelstündlich ist „sehr häufig" -, wird nicht allein wegen der Zahl als Problem gemeldet. Vier weitere Dinge werden gewogen, und der Befund sagt, welches von fünf Urteilen gilt:

- **Der Auslöser.** Autovacuum startet, wenn die toten Zeilen `threshold + scale_factor × Zeilen` überschreiten (zuerst die Einstellung der Tabelle, dann die des Servers; ab PostgreSQL 18 begrenzt durch `autovacuum_vacuum_max_threshold`). Der Befund sagt, welcher Teil entscheidet - bei einer kleinen Tabelle die feste Schwelle (50 + 0.2 × 120 Zeilen sind vor allem die 50), bei einer großen der Skalierungsfaktor - und welchen Anteil der Tabelle der Auslöser ausmacht.
- **Die Arbeit, die jeder Lauf vorfindet.** Pro Lauf aktualisierte oder gelöschte Zeilen (ab PostgreSQL 13 auch eingefügte, gegen den Insert-Auslöser). Ein Lauf *räumt auf*, wenn das **die Hälfte des Auslösers oder 10.000 Zeilen** erreicht; er ist *leer*, wenn es **unter 10 % des Auslösers und unter 1.000 Zeilen** liegt. Ein geplantes manuelles `VACUUM` ist der übliche leere Fall: Es läuft, ob sich etwas geändert hat oder nicht.
- **Was ein Lauf kostet.** Auf PostgreSQL 18 wird es je Tabelle gemessen. Vor 18 wird es aus der Bremse geschätzt: die Seiten, die ein Lauf liest (die von der Visibility Map nicht als all-visible markierten, die von den Änderungen berührten und jeder Index, wenn der Lauf Indizes säubert - ab PostgreSQL 14 übersprungen, wenn weniger als 2 % der Seiten tote Indexeinträge haben), jeder Lesezugriff als Plattenzugriff zu `vacuum_cost_page_miss`, jede veränderte Seite zu `vacuum_cost_page_dirty`, im Tempo von `cost_limit` Einheiten pro `cost_delay`. Das Kostenlimit teilen sich die gleichzeitig laufenden Autovacuum-Worker, außer die Tabelle hat eigene Kosteneinstellungen; die Schätzung teilt es deshalb durch die jetzt laufenden Worker. Ein manuelles `VACUUM` läuft standardmäßig mit `vacuum_cost_delay` 0 - ohne Bremse -, seine Dauer lässt sich so nicht schätzen. Auf PostgreSQL 18 bleibt die Schätzung als Erklärung: Die Läufe dauern etwa so lange, wie die Bremse erlaubt (bremsgebunden: ein höheres Kostenlimit würde sie verkürzen), kürzer (die Seiten kamen aus dem Cache) oder länger (sie warten auf Platte, Sperren oder CPU). Der Befund nennt auch die Lese- und Schreibobergrenze, die die Bremse ergibt. *Günstig* heißt **höchstens 5 Sekunden pro Lauf und höchstens 15 Minuten am Tag**.
- **Der WAL, den er schreibt.** Immer eine Schätzung - keine PostgreSQL-Version zählt WAL pro Tabelle: etwa 100 Bytes pro veränderter Seite, dazu ein Full-Page-Image für eine Seite, die seit dem letzten Checkpoint nicht protokolliert wurde. Eine Seite, die die Last nach dem letzten Checkpoint geändert hat, wird nicht erneut abgebildet; eine Tabelle, die *öfter* gesäubert wird, als Checkpoints stattfinden, zahlt deshalb weit weniger Images, als ihre Laufzahl vermuten lässt. Der Befund vergleicht den Abstand der Läufe mit dem tatsächlichen Checkpoint-Intervall und sagt, welcher Fall gilt. Ab PostgreSQL 14 wird die Schätzung auf den gemessenen WAL des Clusters begrenzt und ins Verhältnis dazu gesetzt. Groß ist **1 GB am Tag und 10 % des Cluster-WAL** (eine Warnung ab **10 GB und 30 %**). `wal_compression` verkleinert die Images; der Befund erwähnt es, ohne es zu modellieren.

Die Urteile: **ok** - günstig, räumt auf, tote Zeilen höchstens 1,5 × Auslöser (oder unter 1.000), WAL nicht groß: als OK angezeigt, Läufe pro Tag grün, nichts zu ändern. **idle** - öfter gesäubert, als sich die Tabelle ändert (ein Hinweis, wenn die Läufe günstig sind; eine Warnung, wenn sie sehr häufig und nicht günstig sind oder ihre Kosten unbekannt): das manuelle `VACUUM` seltener planen; bei Autovacuums eigenen Läufen den Insert-Auslöser und das Transaktions-ID-Alter prüfen. **costly** - teure Läufe auf einer Tabelle mit mindestens 100.000 Zeilen, deren Auslöser unter 1 % von ihr liegt, mit der Schreibrate sowie Schwelle und Skalierungsfaktor, die sie etwa stündlich säubern würden - diese werden die empfohlenen Werte, die **Apply** und die ALTER-Anweisung tragen; oder Läufe, die in allem gut sind außer ihrem WAL. **busy** - die Häufigkeit kommt von der Schreibrate, nicht von den Einstellungen: seltener säubern ließe den Bloat wachsen, und jeder Lauf wird günstiger mit mehr HOT-Updates (fillfactor), weniger Indizes oder einer sanfteren Bremse. **unknown** - die Kosten lassen sich nicht sagen. Eine Empfehlung verschärft nie die Schwellen einer häufig gesäuberten Tabelle, welches Urteil auch gilt; eine teure wird stattdessen auf den stündlichen Auslöser gelockert, und der Grund steht neben den empfohlenen Werten.

**Wie „pro Tag“ gezählt wird.** Jede Tageszahl sind die Läufe seit Beginn des Statistikfensters der Tabelle, geteilt durch seine Länge - ein Durchschnitt über dieses Fenster -, und Karte und Befund sagen, wo es beginnt, zum Beispiel *since 2026-10-03 14:20 CET (25.0 days, this database's statistics reset)*. Der Beginn ist, in dieser Reihenfolge: auf PostgreSQL 19 der eigene Rücksetzzeitpunkt der Tabelle; sonst das letzte Zurücksetzen der Statistik dieser Datenbank; für eine nie zurückgesetzte Datenbank (PostgreSQL 15 und neuer lassen diesen Zeitpunkt leer) das früheste Zurücksetzen der Statistik des Servers selbst; fehlt all das, der Serverstart. PostgreSQL verschiebt den Rücksetzzeitpunkt der Datenbank auch, wenn die Zähler nur *einer* Tabelle zurückgesetzt werden (`pg_stat_reset_single_table_counters`), daher können die Zähler der anderen Tabellen älter sein. Ein Zurücksetzen löscht auch die Zeitpunkte des letzten Vacuum und Analyze einer Tabelle; eine Tabelle mit einem älteren Zeitpunkt als dem der Datenbank wurde also nicht mit zurückgesetzt: Sie bekommt **keine Rate** statt einer durch das kurze Fenster aufgeblähten Zahl, und der Abschnitt *Autovacuum cost and I/O* zählt solche Tabellen in einer Zeile. Ein Fenster unter einer Stunde ergibt eine Hochrechnung, ohne Farbe gezeigt. Ein sechstes Urteil, **past**, ist eine im Durchschnitt häufige Tabelle, deren letztes Vacuum mindestens zehn ihrer durchschnittlichen Abstände und drei Stunden zurückliegt: Der Durchschnitt stammt aus früherer Aktivität, also wird nichts markiert und keine Empfehlung deswegen zurückgehalten; AWR-Snapshots messen die aktuelle Rate pro Fenster. Auf PostgreSQL 15 und neuer wird eine Datenbank, die nach dem Beginn der Serverstatistik angelegt wurde, durch das längere Fenster geteilt, ihre Rate kann also niedrig ausfallen, nie hoch.

Autovacuum besucht jede Datenbank einmal pro `autovacuum_naptime` und kann eine Tabelle daher höchstens so oft säubern; eine Tabelle, die bei den meisten dieser Besuche gesäubert wird, ist fast jedes Mal fällig, und der Befund sagt das. Sind die Autovacuum-Worker ausgelastet - auf PostgreSQL 18 **mindestens 50 % ihrer Kapazität und 6 Worker-Stunden am Tag** belegt (eine Warnung ab **80 % und 12 Stunden**), gemessen an der Zeit, die die Tabellen dieser Datenbank in Autovacuum und Autoanalyze verbracht haben, eine Untergrenze, weil andere Datenbanken hinzukommen; auf 19 auch alle Worker belegt, während Tabellen fällig sind -, warten fällige Tabellen, bis sie an der Reihe sind, und der Rat lautet mehr Worker oder eine lockerere Bremse, nicht andere Schwellen.

Jeder Befund endet damit, wie belastbar seine Zahlen sind: PostgreSQL 18 und neuer messen die Laufzeit jeder Tabelle; 16 und 17 messen nur die I/O des Clusters; 14 und 15 schätzen alles je Tabelle; auf 13 bleibt der Befund bei der Häufigkeit allein.

Der Abschnitt **Maintenance actions** im Advisor-Tab zeigt außerdem jedes VACUUM / ANALYZE / REINDEX / VACUUM FULL, das **gerade jetzt** auf dieser Tabelle läuft, vom Server per Relation-ID gelesen - es erscheint also, ob die App es gestartet hat, ein anderes Werkzeug oder das Fenster gerade erst geöffnet wurde. Jedes zeigt Live-Fortschritt aus `pg_stat_progress_*` und ein **Stop**, das es auf dem Server abbricht; das Stoppen einer von jemand anderem gestarteten Operation fragt zuerst nach Bestätigung. Auf PostgreSQL 12+ steht das sicherere `REINDEX CONCURRENTLY` (das keinen `ACCESS EXCLUSIVE`-Lock nimmt) voran; unter 12 wird es deaktiviert mit Begründung gezeigt.

**Aus dem gespeicherten Verlauf (AWR und ASH).** Alles oben liest die Zähler seit dem letzten Zurücksetzen der Statistik: ein Durchschnitt über diese ganze Zeit. Sind AWR-Snapshots der verbundenen Datenbank gespeichert, lesen `db-vacuum` und der Advisor-Tab auch sie: pro Tabelle die in jedem Fenster zwischen zwei Snapshots geänderten, gelöschten und eingefügten Zeilen, wie oft Vacuum lief, und wie weit die Tabelle bei jedem Snapshot von ihrem Auslöser entfernt war, mit den damals geltenden Einstellungen (Snapshots einer früheren Version haben diese Zahlen nicht). Aus der durchschnittlichen Rate bemessen sie einen Auslöser für etwa **einen Lauf pro Stunde**, nie lockerer als 20% der Tabelle. Der heutige Auslöser wird **enger**, wenn er mindestens **doppelt** so groß ist, mindestens **10.000** tote Zeilen warten lässt und sich die Tabelle um mindestens 500 Zeilen pro Stunde ändert; er wird **lockerer**, wenn er höchstens halb so groß ist und die Tabelle stündlich oder öfter gevacuumt wird (mindestens 10 Läufe im Verlauf); alles innerhalb eines Faktors zwei **passt**. Ab PostgreSQL 13 bekommt eine Tabelle mit überwiegend Einfügungen (90% ihrer Schreibvorgänge) das Insert-Paar auf dieselbe Weise. Diese Werte ersetzen die Größenheuristik in der Empfehlung, die **Apply** und die ALTER-Anweisung tragen, und die Begründung daneben nennt die Raten und das geschäftigste Fenster. Eine Tabelle, die mindestens **eine Stunde über zwei oder mehr Snapshots** über ihrem Auslöser blieb, wird hervorgehoben (eine Warnung ab **sechs Stunden**), und dann werden keine neuen Schwellwerte angeboten: der Auslöser entscheidet nicht, wann sie gevacuumt wird. Sind ASH-Stichproben gespeichert, sagen sie warum: ein Autovacuum-Worker an der Tabelle **wartet auf eine Sperre** (mindestens 10% seiner Stichproben und mindestens 5, oder 60 Stichproben für sich allein), mit der Sitzung, die sie hielt; er schläft in seiner **Kostenbremse** (`VacuumDelay`: mindestens die Hälfte seiner Stichproben und mindestens 30, oder 600 für sich allein); oder **alle Autovacuum-Worker sind gleichzeitig beschäftigt** (mindestens die Hälfte der Stichproben mit einem Worker am Werk und mindestens 60, oder 1.800 für sich allein). Unter einer Stunde Verlauf wird nichts bemessen. Ein Block **Belege** nennt die verwendeten Snapshots und Fenster, von wann bis wann, und die ASH-Tage, oder sagt, dass es keinen Verlauf gibt und der Rat so ist wie immer.

`db-table-stats` sind die Zähler für sequenzielle vs. Index-Scans je Tabelle, dazu `last_seq_scan` / `last_idx_scan` ab PostgreSQL 16. Eine Tabelle mit Millionen sequenzieller Scans und steigendem Zähler wird gerade in Produktion gescannt - gleichen Sie sie mit `db-index-advisor` ab.

#### Sequenzen, Checkpoints und Tablespaces {#sequences-checkpoints-and-tablespaces}

`db-sequences` und `db-seq-usage` listen Sequenzen danach, wie nah sie ihrem Maximum sind, mit der besitzenden Spalte und jedem Default, der auf sie verweist. Eine `integer`-Sequenz bei 80% ist eine Umstellung auf `bigint`, die Sie planen wollen, nicht entdecken, wenn `nextval` scheitert.

Es zählt das kleinere von zwei Maxima: der eigene `MAXVALUE` der Sequenz und der größte Wert, den die gespeiste Spalte aufnehmen kann (32.767 für `smallint`, 2.147.483.647 für `integer`). Vor PostgreSQL 10 hatte eine Sequenz keinen Typ - jede war `bigint` -, also ist jede damals angelegte `serial`-Spalte eine `integer`-Spalte, gespeist von einer `bigint`-Sequenz, und ein aktualisiertes Schema behält diese Form. Nur an der Sequenz gemessen zeigt eine solche Spalte 0% Verbrauch, während ihr noch ein paar tausend IDs bleiben; die nächste Einfügung über die Kante scheitert mit *integer out of range*. `db-sequences`, `db-seq-usage` und die Zeile SEQUENCES der Gesundheitsübersicht messen an der Spalte und bewerten nach zwei Dingen, eines genügt:

- **Anteil des verbrauchten Bereichs**: gelb ab 80%, rot ab 95%;
- **verbleibende Zeit**: die verbleibenden IDs geteilt durch die Einfügungen pro Tag in die Tabelle der Spalte (ihr Einfügezähler über das Statistikfenster - dasselbe Fenster, das der Tabellen-Advisor nennt): gelb unter 180 Tagen, rot unter 30. Ein Fenster unter einer Stunde macht die Rate zur Hochrechnung: die Zeit wird gezeigt, als solche bezeichnet und ändert die Farbe nicht.

Jede auslaufende Sequenz nennt die Fremdschlüsselspalten, die auf ihre Spalte zeigen und schmaler als `bigint` sind: die Umstellung der Spalte auf `bigint` verlagert das Problem zu ihnen, also ändern sie sich in derselben Migration. Ist die Spalte selbst schon `bigint`, ist eine schmalere Fremdschlüsselspalte die Obergrenze: auf die erste ID über ihrem Maximum kann nicht verwiesen werden, also wird der verbleibende Platz an ihr gemessen und sie wird genannt. `ALTER COLUMN ... TYPE bigint` schreibt die ganze Tabelle unter einer `ACCESS EXCLUSIVE`-Sperre neu - niemand liest oder schreibt sie, bis es fertig ist; der Weg ohne Ausfall ist eine neue `bigint`-Spalte, per Trigger synchron gehalten, ein Auffüllen in Paketen und ein Tausch.

`db-seq-sync` schreibt ein `SELECT setval(...)` für jede Sequenz und setzt jede knapp über den größten Wert, der aktuell in ihrer besitzenden Spalte steht. Führen Sie es nach einem Logical-Replication-Cutover oder einem `pg_dump --data-only`-Load aus, wo die Zeilen ankommen, die Sequenzen aber bei 1 bleiben und das nächste `INSERT` mit einem vorhandenen Schlüssel kollidiert. Es ist ein generiertes Skript wie jedes andere - prüfen, dann auf dem Ziel ausführen.

`db-checkpoints` zeigt, wie Checkpoints feuern: **zeitgesteuert** (gut, durch `checkpoint_timeout`) gegen **angefordert** (durch `max_wal_size`-Druck erzwungen), den mittleren Abstand und die Schreib- und Sync-Zeit. Ein hoher Anteil angeforderter bedeutet, `max_wal_size` ist zu klein für die Schreibrate - es zu erhöhen tauscht Plattenplatz gegen viel weniger Checkpoint-I/O.

`db-tablespaces` listet jeden Tablespace mit Ort, Besitzer, Größe, Optionen (Overrides für `seq_page_cost`, `random_page_cost`, `effective_io_concurrency`) und den nutzenden Datenbanken, dazu den Inhalt jedes Tablespace in der verbundenen Datenbank. Der Abschnitt mit dem Inhalt je Datenbank ist an die aktuelle Verbindung gebunden, aus demselben Grund wie die Schema-Größen - der Katalog, der Objekte auf Tablespaces abbildet, gilt je Datenbank. Prüfen Sie ihn, bevor Sie ein großes Objekt verschieben, um sicherzugehen, dass das Ziel Platz hat und auf der erwarteten Platte liegt; ein `MOVE` schreibt eine vollständige zweite Kopie, bevor es die erste verwirft, das Ziel braucht also freien Platz in Höhe der Objektgröße.

#### Partitionsvorlauf {#partition-runway}

`db-partition-runway` (Speicher) listet jede partitionierte Tabelle der verbundenen Datenbank: Strategie und Schlüssel, Zahl der Partitionen, wo der letzte Bereich endet und wie weit das von jetzt ist, die DEFAULT-Partition und die Zeilen darin, und ob pg_partman die Tabelle verwaltet. Sie gibt es für den Vorfall, der um Mitternacht am Ersten passiert: eine nach Monat partitionierte Tabelle, deren nächste Partition niemand angelegt hat. Ab dem Moment, in dem der letzte Bereich endet, scheitert ein INSERT für "jetzt" mit *no partition of relation found for row* oder landet, wenn es eine DEFAULT-Partition gibt, in ihr - und eine DEFAULT-Partition mit Zeilen eines Bereichs verhindert später das Anlegen der Partition für diesen Bereich (*updated partition constraint for default partition would be violated*), bis die Zeilen von Hand herausgeholt sind.

- Eine Tabelle ist **gelb**, wenn ihr letzter Bereich innerhalb der Tage endet, die der Prüfung mitgegeben werden (standardmäßig 7, und Health nimmt 7), und **rot**, sobald er geendet hat. Gemessen wird für einen RANGE-Schlüssel, dessen erste Spalte ein `date`, `timestamp` oder `timestamptz` ist, auch wenn der Schlüssel ein Ausdruck wie `(created_at::date)` ist; ein numerischer Schlüssel (ein ID-Bereich) hat keinen Kalender zum Messen, und die Zeile sagt das. Ein letzter Bereich bis `MAXVALUE` läuft nie aus.
- **Zeilen in einer DEFAULT-Partition** sind bei einer RANGE-Tabelle gelb, gleich wie viele - eine Zeile genügt, um die Partition zu verhindern, die sie aufnehmen sollte - und bei einer LIST-Tabelle blau, wo ein Auffangbecken ein übliches Design ist. Die Zeile nennt die Zahl und den Anteil an der Tabelle.
- **pg_partman**: ist die Erweiterung in der Datenbank installiert, liest die Prüfung ihre Konfiguration - premake und Intervall jeder verwalteten Tabelle - und was ihre Wartung ausführt (der Worker `pg_partman_bgw`, ein `pg_cron`-Job). Eine von pg_partman verwaltete Tabelle ist gelb, wenn ihr Vorlauf unter das gefallen ist, was ihr premake vorhält: eine tägliche Tabelle mit premake 4 hat immer drei bis vier Tage, die 7-Tage-Warnung wäre für sie also nie still. Die Wartung läuft dann nicht, oder legt - mit `infinite_time_partitions = off` - künftige Partitionen erst an, wenn neue Zeilen angekommen sind.
- Eine Tabelle, deren letzter Bereich längst endete und die keine neuen Zeilen mehr aufnimmt (ein Archiv), ist ebenfalls rot: der Katalog unterscheidet ein Archiv nicht von einer Tabelle, deren Einfügungen scheitern, und die Zeile sagt das.

Nichts liest eine Tabelle: die Grenzen kommen aus dem Katalog und die Zeilenzahlen aus der Statistik des Planers, sie sind also Schätzungen.

#### Replikationsgesundheit {#replication-health}

`db-replication` ist die Sicht des Primärservers auf seine Replikate: der Zustand jedes Standbys und die Verzögerung in Bytes und Zeit in den Phasen write, flush und replay. Die `replay`-Verzögerung ist, wie weit ein Read-Replikat tatsächlich hinterherhinkt; eine wachsende `write`/`flush`-Verzögerung deutet auf das Netz oder die Platte des Standbys, eine wachsende `replay`-Verzögerung auf eine langlaufende Abfrage auf dem Standby, die den Replay aufhält. Auf einem Standby ohne eigene nachgelagerte Replikate sagt die Prüfung das einfach.

`db-replica-identity` findet Tabellen, an denen die logische Replikation bricht: kein Primärschlüssel oder eine nicht-standardmäßige `REPLICA IDENTITY`. Eine Tabelle ohne PK, die **in einer Publikation** ist, ist KRITISCH - das nächste `UPDATE` oder `DELETE` scheitert beim Subscriber - während eine, die es nicht ist, nur ein Hinweis ist. Sie schreibt das `ALTER TABLE` nach Priorität: Primärschlüssel, sonst `REPLICA IDENTITY USING INDEX`, sonst `FULL`.

`db-repl-slots` listet physische und logische Slots mit ihrem `active`-Flag und dem WAL, das jeder zurückhält. **Ein inaktiver Slot füllt die Platte** - er hält WAL für immer, während nichts es verbraucht. Löschen Sie Slots, die eine stillgelegte Replik hinterlassen hat.

Das Anlegen eines **logischen** Slots wartet, bis jede Transaktion beendet ist, die beim Start eine Transaktions-ID hält - in jeder Datenbank des Clusters, eine vergessene vorbereitete Transaktion eingeschlossen. Der Dialog Create slot listet diese Transaktionen auf, bevor Sie ihn anlegen, und das Anlegen wartet höchstens so lange, wie Sie wählen (30 Sekunden, wenn Sie nichts anderes wählen); läuft das ab, wird der Slot nicht angelegt und die Antwort nennt, worauf er noch wartete. Eine nur lesende Transaktion hält ihn nicht auf. Ohne `pg_read_all_stats` sind die Sitzungen anderer Benutzer nur teilweise sichtbar, und der Dialog sagt, dass die Liste unvollständig sein kann.

Der **REPLICA**-Chip im Kopf eines Standbys zeigt *aufgeholt*, wenn alles Empfangene eingespielt ist, egal wie lange der Primary zuletzt etwas bestätigt hat - der Tooltip nennt diese Zeit als die, die der Primary ruhig war. Ein Rückstand in Sekunden erscheint nur, solange noch WAL einzuspielen ist; ein Standby ohne WAL-Receiver (Wiederherstellung aus dem Archiv) zeigt gar keine Zahl, weil er nicht wissen kann, ob noch etwas kommt.

`db-logical-replication` ist die Subscriber-Seite: Verzögerung aus `pg_stat_subscription` und der Sync-Status je Tabelle aus `pg_subscription_rel`. Eine Tabelle, die in `COPY` oder `SYNC` festhängt, bedeutet meist, dass die Erstkopie auf einen Fehler stieß - prüfen Sie das Subscriber-Log.

#### Der Fehler-Wächter {#the-errors-watcher}

`db-errors` tastet `pg_stat_database` über ein Fünf-Sekunden-Fenster ab und meldet die Rate pro Sekunde für Rollbacks, Deadlocks und Konflikte, dazu `sessions_fatal` / `killed` / `abandoned` ab PostgreSQL 14. Auf einem selbst betriebenen Cluster, auf dem `pg_read_file` erlaubt ist, liest er außerdem die letzten 256 KB des Serverlogs - die `jsonlog`-Datei (PostgreSQL 15+) oder die `csvlog`-Datei, wenn der Server eine schreibt, sonst das einfache Log - und liest die Einträge `ERROR` / `FATAL` / `PANIC` als Einträge, jeden mit seinen Zeilen `DETAIL`, `HINT`, `CONTEXT` und `STATEMENT`. In einer Managed Cloud überspringt er das Log und sagt, wo der Anbieter es aufbewahrt.

Aus dem Log meldet er:

- **Fehler nach SQLSTATE:** jede Klasse mit Anzahl, der geschäftigsten Minute, erstem und letztem Auftreten, wie vielen Quellen und der wichtigsten, und was sie bedeutet, für die Klassen rund um Nebenläufigkeit und Wiederholungen - `40P01` Deadlock, `40001` Serialisierungsfehler, `23505` Unique-Verletzung, `55P03` Sperre nicht verfügbar, `57014` Anweisung abgebrochen, `42P10` `ON CONFLICT` ohne passenden Unique-Index, `53300` zu viele Verbindungen, `25P02` Anweisungen in eine abgebrochene Transaktion.
- **Fehler pro Minute**, je SQLSTATE.
- **Wiederholungsstürme:** derselbe SQLSTATE aus derselben Quelle **10-mal oder öfter innerhalb einer Minute** - etwas wiederholt in einer Schleife. Eine absolute Zahl, kein Anteil: eine Wiederholungsschleife ist auf einem ruhigen wie auf einem vollen Server ein Problem. Die Quelle ist die Anwendung, wenn das Log sie nennt (`jsonlog`, `csvlog` oder `%a` in `log_line_prefix`), sonst die Form der Anweisung.
- **Deadlocks, beide Seiten:** für jeden Deadlock jeder Prozess mit seiner PID, worauf er wartete, wer es hielt, der Anweisung, die er ausführte, und welchen PostgreSQL abbrach.

Das einfache Log trägt, was `log_line_prefix` hineinschreibt; die Prüfung liest das Präfix vom Server und findet `%e` (den SQLSTATE), `%a`, `%u` und `%d`, wo immer sie stehen. Ohne `%e` wird der SQLSTATE aus dem englischen Meldungstext gelesen, und die Prüfung sagt es: `%e` (und `%a`) in `log_line_prefix` aufnehmen, oder `jsonlog` bzw. `csvlog` nutzen.

Zuerst die Raten lesen: eine stetige Deadlock-Rate ist ein Fehler in der Sperrreihenfolge der Anwendung; ein Sprung bei Konflikten auf einem Replikat sind Abfragen, die mit dem Replay kollidieren. Das Log ist das Detail, die Zähler sind der Alarm.

#### Die Setup- und Key-Settings-Advisor {#the-setup-and-key-settings-advisors}

`db-setup-advisor` prüft alles Serverseitige, von dem DBApguins Prüfungen abhängen - die Erweiterungen `pg_stat_statements`, `auto_explain`, `pg_buffercache` und `pg_repack`, GUCs wie `track_io_timing`, `compute_query_id`, `log_lock_waits` sowie die Rechte `pg_monitor` / `pg_read_server_files`, die ein Nicht-Superuser braucht - und meldet jedes OK / TEILWEISE / FEHLT mit der genauen Abhilfe. **Führen Sie ihn zuerst gegen einen neuen Cluster aus** und nach jeder Konfigurationsänderung erneut. In einer Managed Cloud heißt der Hinweis „in der Parametergruppe setzen" statt `ALTER SYSTEM`.

`db-settings` ist ein kuratierter Führer durch ~70 tuning-relevante Parameter mit Faustregel und Warn-/Gefahr-Badge je. Es ist eine Referenz, kein Alarm: ein Badge heißt „dieser Wert ist ungewöhnlich", nicht „das ist kaputt". Diese Parameter sind aus der Server-Ansicht editierbar. Fünf Karten betreffen die Frage, ob bestätigte Daten einen Absturz überstehen, und kommen zum selben Urteil wie die Zeile DAUERHAFTIGKEIT im Gesundheitsüberblick: `fsync` und `full_page_writes` aus sind Gefahr, `synchronous_commit = off` ist Info (ein bewusster Tausch, kein Fehler), ausgeschaltetes `data_checksums` (nur lesbar) ist Info, und `synchronous_standby_names` zeigt, wie viele der verlangten synchronen Standbys verbunden sind - Gefahr, solange Commits auf einen warten, der es nicht ist.

**Für diesen Server berechnet** öffnet Key settings: die fünfzehn Parameter, die ein Tuner wie DBtune ohne Neustart ändert (`work_mem`, `maintenance_work_mem`, `effective_cache_size`, `random_page_cost`, `seq_page_cost`, `effective_io_concurrency`, `default_statistics_target`, die drei Grenzen für parallele Worker, `max_wal_size`, `min_wal_size`, `checkpoint_completion_target`, `bgwriter_delay`, `bgwriter_lru_maxpages`), jeweils mit einem für diese Maschine berechneten Wert, der Messung oder Formel dahinter, Reload oder Neustart, und **Anwenden**. RAM und CPUs kommen aus `/proc` und den cgroup-Grenzen, wenn der Server Dateien lesen darf (ein Superuser oder `pg_read_server_files`); ein Container zeigt die des Hosts, also eintragen, was der Server nutzen darf, und **Neu berechnen** drücken. Die Platte gilt als SSD, solange Sie nicht HDD wählen. Eine Messung entscheidet die Stufe, wo es eine gibt: `work_mem` ist nur eine Warnung, wenn Sorts wirklich auslagern (mindestens 100 temporäre Dateien UND 1 GB seit dem Zurücksetzen der Statistik), `max_wal_size`, wenn mindestens ein Fünftel der Checkpoints UND fünf davon durch die WAL-Menge erzwungen wurden (oder aus der WAL-Rate: drei `checkpoint_timeout` WAL), der Background Writer, wenn Backends mindestens ein Viertel der schmutzigen Puffer UND 10.000 davon selbst schrieben. **Die N zu prüfenden anwenden** führt nach einer Bestätigung genau die aufgeführten Anweisungen aus und liest den Server dann erneut. Das sind Startwerte für die Maschine, keine Messung Ihrer Last: einige ändern, dieselben Zahlen beobachten, behalten, was geholfen hat. Das Fenster DBApguin setup advisor (Hilfe) zeigt die Anzahl mit einer Schaltfläche zu diesem Abschnitt.

**Mit Historie liest es, was der Server getan hat, nicht nur, was er hat.** Wenn ASH oder geplante AWR-Snapshots für diesen Server liefen (aus einer beliebigen seiner Datenbanken), beginnt der Abschnitt mit **Worauf es beruht**: ASHs Tage und Samples, wie viele Sitzungen wirklich gleichzeitig aktiv waren (p95 und Spitze), der Anteil der aktiven Zeit mit Warten auf I/O und die geschäftigste Stunde; AWRs Fenster, das pro Tag Ausgelagerte, die erzwungenen Checkpoints, das WAL im geschäftigsten Fenster gegenüber dem Schnitt, die mittlere Lesezeit und die schlechteste Cache-Trefferquote; die auslagernden Anweisungen und wie viel ein Aufruf schreibt; und was jetzt läuft. Die Empfehlung nutzt sie, wo sie stärker sind als eine Regel: die Obergrenze für `work_mem` zählt die Sitzungen, die ASH gleichzeitig sah (das Doppelte des p95 oder die Spitze), statt aller Verbindungen - hebt `work_mem` aber nur so weit, wie auslagernde Anweisungen brauchen, das Doppelte dessen, was ein Aufruf auf die Platte schrieb; eine Anweisung, die mehr braucht, als der RAM erlaubt, erscheint mit einem eigenen `SET work_mem`. `max_wal_size` folgt dem geschäftigsten AWR-Fenster, die Plattenart der gemessenen Lesezeit (`pg_stat_io` mit `track_io_timing`), `effective_io_concurrency` und `shared_buffers` (Neustart) werden Warnungen, wenn ASH ein Fünftel der aktiven Zeit UND 600 Samples mit Warten auf I/O zeigt oder AWR eine Trefferquote unter 99 % bei mindestens einer Million Lesevorgängen. Ohne Historie sagt es das und bietet **ASH starten** und **AWR-Snapshots starten** (alle 15 Minuten); je mehr davon da ist, desto näher ist die Empfehlung an dieser Last. AWR-Snapshots erfassen jetzt auch `pg_stat_wal` (PostgreSQL 14+) und `pg_stat_io` (16+); älteren Snapshots fehlt das einfach.

#### Bereit für PostgreSQL 19? {#ready-for-postgresql-19}

**Optimierung und Diagnose -> Bereit für PostgreSQL 19?** fragt diesen Server nach
jedem Punkt aus der Inkompatibilitätsliste von PostgreSQL 19 und beantwortet
jeden:

- **BLOCKS** - pg_upgrade lehnt ab oder 19 startet nicht: eine Datenbank in
  der entfernten Kodierung MULE_INTERNAL, ein Zeilenumbruch im Namen einer
  Datenbank, Rolle oder eines Tablespaces, ein Index mit den
  inet/cidr-Operatorklassen von btree_gist, RADIUS in pg_hba.conf,
  escape_string_warning irgendwo gesetzt.
- **CHANGES** - das Upgrade geht durch, aber etwas verhält sich anders:
  standard_conforming_strings aus, MD5-Passwörter (auf 19 warnt jede
  Anmeldung), max_locks_per_transaction von Hand gesetzt (19 braucht für
  dieselbe Kapazität das Doppelte), JIT in Gebrauch (auf 19 standardmäßig
  aus), postgres_fdw (Nur-Lese-Transaktionen gehen jetzt an die Gegenseite).
- **INFO** - Erweiterungen von Drittanbietern, die vor pg_upgrade einen Build
  für 19 brauchen, und Überwachungswerkzeuge, die umbenannte Spalten lesen.

Mehrere Punkte brauchen Superuser; ohne ihn melden sie „nicht prüfbar" statt
Entwarnung. Der Index-Punkt liest die Datenbank, mit der Sie verbunden sind,
daher die Prüfung in jeder Datenbank ausführen. Auf einem Server, der bereits
mit 19 läuft, zeigen dieselben Zeilen, was noch von den Standards von 19
abweicht.

#### Observability im Oracle-Stil

DBApguin bringt Prüfungen mit, die für PostgreSQL das leisten, was in Oracle AWR / ASH / SQL Trace leisten:

**ASH und AWR haben je einen Tab.** Das Menü *Optimierung und Diagnose* öffnet *Active Session History (ASH)* und *AWR-Snapshots und Berichte*, je einen Tab pro Server. Die Kopfzeile des Tabs zeigt, was der Sammler tut (sammelt oder nimmt Snapshots auf, seit wann, wie oft, was behalten wird; gelb und *pausiert*, solange der Server nicht erreichbar ist) mit **Starten**, **Stoppen** und den Einstellungen hinter dem Zahnrad; die Ansichten sind die Unternavigation des Tabs (ASH: Live, Bericht, Gespeicherte Tage; AWR: Live, Snapshots, Bericht, Vergleich, Aufzeichnung, Trend), und die Eingaben einer Ansicht, das Berichtsfenster und die Datenbank oder die zwei zu vergleichenden Snapshots (vier für zwei Zeiträume), stehen über dem Ergebnis statt in einem Dialog. Ein Datum wird mit der Datum-und-Uhrzeit-Auswahl gewählt, deren Kalender die Tage mit Samples unterstreicht. Die unten beschriebenen `action`-Parameter sind das, was die Ansichten ausführen; aus Skripten und älteren Links funktionieren sie weiter und öffnen denselben Tab. Beim Verbinden mit einem Server, dessen Sampler oder Zeitplan läuft, öffnet sich dessen Tab von selbst.

**db-workload-report (Live-Aufzeichnung im AWR-Stil)** - AWR "Für eine Weile" bis zu einer Stunde mit *auch die Sitzungen alle 5 s abtasten* führt ihn aus (sein Ergebnis erscheint im AWR-Tab weiter unter *Aufzeichnung*): ein Fenster (10 s bis 60 min, Standard 5 min) und ein Abtastintervall. Er nimmt am Anfang denselben Snapshot, den ein AWR-Snapshot nimmt, tastet dazwischen alle paar Sekunden pg_stat_activity ab, nimmt am Ende den zweiten Snapshot und liest das Fenster genau so, wie ein AWR-Bericht zwei Snapshots liest: **Fensterzusammenfassung** (mit den Stichproben, der Spitze aktiver und aller Sitzungen und wie viele Benutzer und Anwendungen gesehen wurden), **Befunde** mit ihren Belegen, **Top-SQL zum Ansehen** mit dem Grund für jedes, dann **Aktive Sitzungen im Zeitverlauf** (ein Diagramm der Stichproben nach Wartklasse), **Wait-Events**, **Beobachtete blockierende Paare**, **Längste gesehene offene Transaktion**, die Top-SQL-Ranglisten, die Tabellen nach DML, die Hintergrund- und Checkpoint-Aktivität und die Einstellungen für eine bessere Analyse. Siehe [Einen AWR- / Workload-Bericht lesen](#reading-an-awr-report). Nutzen Sie ihn, wenn die Last gerade jetzt läuft und Sie einen Bericht darüber wollen; der Bericht wird unter [Gespeicherte Berichte](#saved-reports) aufbewahrt.

**db-awr-snapshot (dauerhafter Snapshot-Speicher + beliebiger Vergleich + Scheduler)**: sechs Aktionen über den Parameter `action`. Drei manuelle, drei für den Scheduler.

  Manuelle Aktionen:

  - action=take: erfasst pg_stat_statements + pg_stat_database + pg_stat_user_tables + pg_stat_user_indexes + die Autovacuum-Einstellungen + bgwriter / checkpointer in eine JSON-Datei unter `$DBCHECKS_HOME/awr_snapshots/<Server-Fingerabdruck>/snap-<id>.json`. Ein optionaler Kommentar wird mitgespeichert. Den größten Teil eines Snapshots macht die Kopie von pg_stat_statements aus; die Zeilen pro Tabelle und pro Index, die der Autovacuum- und der Index-Rat zurücklesen, kosten etwa 320 Bytes je Tabelle und 160 Bytes je Index, eine Datenbank mit tausend Tabellen macht jeden Snapshot also etwa ein halbes Megabyte größer. Der Fingerabdruck ist `host__port__dbname`, sodass sich Snapshots verschiedener Cluster nie mischen. Der Spaltensatz folgt der Serverversion - `pg_stat_database.sessions_*` ab PostgreSQL 14, `pg_stat_checkpointer` ab 17, davor `pg_stat_bgwriter` - ein Snapshot ist also ab PostgreSQL 13 gültig, und ein Vergleich zweier verschiedener Hauptversionen lässt die Spalten schlicht weg, die einer von beiden fehlen.
  - action=list (**Gespeicherte Snapshots**): jeder für diesen Server aufbewahrte Snapshot - Tag, Zeit (UTC), Kommentar, Anweisungen, Größe - mit **Mit dem vorherigen vergleichen** in jeder Zeile, **Die zwei gewählten vergleichen**, **Exportieren** (die gewählten oder alle, in ein Zip, wohin der Speichern-Dialog sagt), **Importieren...**, **Löschen** der gewählten und **Alles löschen, was älter ist als N Tage**. Löschen fragt vorher. Es arbeitet nur mit den Dateien, sodass die Historie eines abgeschalteten Servers trotzdem angesehen, aufgeräumt oder mitgenommen werden kann.

    **Export und Import.** Ein Export ist ein Zip: die Dateien, wie sie sind, und ein kleines Manifest, das ASH oder AWR und den Server nennt. Der Import legt sie wieder unter diesem Server ab - verbinden Sie sich mit ihm, um ihre Berichte zu sehen -, nimmt nur die eigenen Dateinamen des Speichers (ein Zip mit etwas anderem darin wird ganz abgelehnt, nichts geschrieben) und überschreibt nie eine Datei, die schon da ist: er sagt, welche er behalten hat.
  - action=compare: vergleicht zwei beliebige Snapshots nach ID und zeigt dieselben Abschnitte im AWR-Stil. Das „Fenster" darf Stunden oder Tage umfassen, weil die Daten auf der Platte liegen. Einen Snapshot vor einem Deploy / Batch / einer Migration nehmen, einen danach, und dann vergleichen, wann immer Sie wollen.

    **Über einen Neustart hinweg.** Ein sauberer Neustart behält die Zähler von PostgreSQL - sie werden beim Herunterfahren geschrieben und beim Start wieder gelesen -, also deckt ein Vergleich darüber das ganze Fenster ab und sagt nur, dass der Server neu gestartet wurde. Ein Absturz, eine Wiederherstellung beim Start oder ein Failover setzt sie auf null, ebenso pg_stat_reset(). Jeder Snapshot hält fest, wann der Server gestartet wurde; wo ein Zähler im neueren Snapshot kleiner ist, zählt der Vergleich ihn ab null (der neuere Wert ist das, was seit dem Reset geschah), statt eine negative Zahl auszugeben, beginnt mit einem Befund, der das sagt, und nimmt nach einem Absturz das Fenster ab dem Serverstart. Der Trend macht dasselbe für jedes Fenster. Oracles AWR hat dieselbe Grenze - seine Zähler beginnen mit der Instanz neu, und einen Bericht über einen Neustart lehnt es ab; was beide behalten, ist die Reihe der Snapshots.

  - action=compare-periods (die Ansicht **Vergleich**): zwei Zeiträume nebeneinander, so wie Oracles Bericht *Compare Periods* sie setzt - Zeitraum 1 von Snapshot A1 bis B1, Zeitraum 2 von A2 bis B2; der ältere Snapshot jedes Paares gilt als sein Anfang. Jeder Zeitraum wird genau so berechnet, wie compare sein Fenster berechnet, auch über einen Neustart oder ein Zurücksetzen hinweg, mit derselben Wahl der **Datenbank**. Ohne Auswahl geöffnet, nimmt die Ansicht die drei neuesten Snapshots als zwei aufeinanderfolgende Fenster - den drittneuesten bis zum zweitneuesten, dann den zweitneuesten bis zum neuesten - und vergleicht sie; mit weniger als drei sagt sie, dass ein Vergleich drei braucht, und führt nichts aus. Die Zeiträume dürfen sich überlappen und in beliebiger Reihenfolge kommen; der Bericht sagt es. Siehe [Einen Zeitraumvergleich lesen](#reading-a-period-comparison).


  Aktionen des Schedulers (das Gegenstück zum von `DBMS_SCHEDULER` getriebenen Auto-AWR):

  - action=auto-start: startet einen Hintergrund-Thread, der alle `interval_minutes` (Standard 60, Bereich 5-1440) einen Snapshot nimmt und Snapshots älter als `retention_days` (Standard 30, Bereich 1-365) selbst löscht. Der Thread läuft auf einer eigenen psycopg2-Verbindung und hungert den Benutzer-Pool nie aus. Idempotent (erneutes Ausführen aktualisiert die Parameter des laufenden Threads). Der erste Snapshot entsteht sofort: `auto-start` wartet auf sein Ergebnis und meldet, was wirklich geschah - ein Scheduler, der nicht einen einzigen Snapshot nehmen kann, sagt das statt „gestartet". Schlägt dieser erste Snapshot fehl, bleibt der Thread nicht stehen. Im Betrieb beendet ihn ein Server, der nicht mehr antwortet, nicht: nach drei fehlgeschlagenen Snapshots in Folge pausiert der Scheduler, versucht weiter, sich neu zu verbinden (alle 30 Sekunden, dann seltener), und macht von selbst weiter, sobald der Server zurück ist; das Live-Panel zeigt die Pause in Gelb.
  - action=auto-stop: stoppt den Scheduler-Thread. `auto-stop` meldet, was er im Betrieb erfasst hat; hat er nie einen Snapshot genommen, sagt er das klar statt „Dateien bleiben liegen". Das gespeicherte Flag `enabled` in `auto_config.json` wird false.
  - action=auto-status: nennt zuerst das Urteil - gesund, pausiert, fehlerhaft, abgebrochen oder nicht laufend - dann den Zustand des laufenden Threads (Intervall, Aufbewahrung, Anzahl der Snapshots, ID und Zeit des letzten, insgesamt gelöschte Dateien, letzter Fehler falls vorhanden) und die gespeicherte Konfiguration. Nach einem Neustart der Anwendung kommt der Scheduler von selbst zurück, sobald Sie diesen Server das nächste Mal öffnen; auto-start braucht es nur, wenn Sie ihn gestoppt haben.

  Ein laufender Auto-Scheduler erscheint auch in der Liste **Background jobs**, wie der ASH-Sampler, sodass Sie auf einen Blick sehen, ob er gesund ist, ohne die Prüfung zu öffnen.

  **Scheduler, in Echtzeit** (in `auto-status` und nach `auto-start`) ist dasselbe Panel wie bei ASH und aktualisiert sich alle 5 Sekunden, solange der Tab offen ist: der Zustand, Intervall und Aufbewahrung, wie lange er läuft, erstellte Snapshots, wie lange der letzte her ist (gelb, sobald er ein halbes Intervall zu spät ist), wann der nächste fällig ist, was auf der Platte liegt und von wann bis wann, und was die Aufbewahrung entfernt hat. Der Dialog zeigt nur die Felder, die die gewählte Aktion verwendet.

  Zur Dauerhaftigkeit: die Scheduler-Konfiguration wird neben den Snapshot-Dateien gespeichert und überdauert Neustarts der Anwendung. Ein Scheduler, der beim Schließen der Anwendung lief, startet nicht beim Programmstart (dann wird nichts entschlüsselt); er kommt in dem Moment zurück, in dem Sie diesen Server in der App wieder öffnen, mit der Verbindung, mit der Sie ihn öffnen, und dem gespeicherten Intervall und der Aufbewahrung. Bis dahin zeigt `auto-status` ihn als aktiviert, aber nicht laufend. `auto-stop` schaltet ihn endgültig ab.

**db-ash (Sampler für die Historie aktiver Sitzungen)**: durchgehender Hintergrund-Sampler. Vier Aktionen.

  - action=status: nennt zuerst das Urteil - läuft und gesund, pausiert (der Server antwortet nicht mehr; er verbindet sich immer wieder neu und macht von selbst weiter), fehlerhaft oder gestoppt - dann seit wann, wie viele Stichproben und Zeilen bisher, den letzten Fehler falls vorhanden und den Bestand auf der Platte.
  - action=start: startet einen Daemon-Thread, der pg_stat_activity alle **interval** Sekunden (Standard 1; 2, 5, 10, 30 oder 60) auf einer eigenen Verbindung (getrennt vom Benutzer-Pool) abfragt, untätige Backends aussortiert und jede aktive Zeile an eine Tagesdatei unter `$DBCHECKS_HOME/awr_snapshots/<Fingerabdruck>/ash/ash-YYYY-MM-DD.jsonl` anhängt. Idempotent (erneutes Ausführen tut nichts). Der Spaltensatz je Stichprobe folgt der Serverversion - `query_id` gibt es ab PostgreSQL 14, auf 13 wird es also einfach nicht erfasst, und der Bericht merkt das einmal an, statt zu scheitern.
  - **interval** und **retention days** werden beim Start gewählt und je Server für den nächsten Start gemerkt. Die Retention beträgt 30 Tage, sofern nicht anders gewählt, und 0 behält alles; Tagesdateien, die älter als die Retention sind, werden beim Start des Samplers und einmal pro Stunde während er läuft entfernt - nur ganze Tage, nie ein Teil davon. Ein längeres Intervall schreibt weniger und verpasst kurze Wartezeiten; eine Sekunde ist, was ein klassisches ASH tut. Der Sampler behält die Einstellungen, mit denen er gestartet wurde: ein erneuter Start mit anderen Werten sagt das und ändert nichts, bis er gestoppt ist.
  - action=stop: stoppt den Sampler und meldet, was er erfasst hat. Wurde nie etwas geschrieben, sagt er das, statt „die JSONL-Dateien bleiben liegen". Nach drei fehlgeschlagenen Stichproben in Folge pausiert der Sampler, statt zu stoppen: er versucht weiter, sich neu zu verbinden (alle 30 Sekunden, dann einmal pro Minute), und macht von selbst weiter, wenn der Server wieder antwortet; `status` und das Live-Panel zeigen die Pause in Gelb mit dem letzten Fehler. Ein laufend gelassener Sampler kommt auch nach einem Neustart der Anwendung von selbst zurück, sobald Sie diesen Server das nächste Mal öffnen; `stop` schaltet ihn ab.
  - action=report: fasst einen beliebigen Datumsbereich auf der eigenen Uhr des Servers in vier klassische ASH-Abschnitte: Zustandshistogramm, Top-20-Wait-Events, häufigste Sitzungen nach Stichprobenzahl, häufigste Abfragen nach Stichprobenzahl, beobachtete Blockierpaare. **Bericht von** und **Bericht bis** werden mit der Datum-und-Uhrzeit-Auswahl der App gewählt: Schnellwahl (jetzt, heute 00:00, vor 1 h, vor 24 h, gestern), Kalender sowie Felder für Stunde und Minute; Leeren lässt eine Grenze offen.
  - action=data (**Gespeicherte Tage**): jede für diesen Server aufbewahrte Tagesdatei mit ihrer Größe, heute markiert; **Bericht** in einer Zeile öffnet den Bericht dieses Tages, **Bericht für die gewählten Tage** reicht vom ersten bis zum letzten gewählten. Export, Import und Löschen wie bei den gespeicherten AWR-Snapshots unten.

  Ein laufender Sampler erscheint in der Liste **Background jobs** mit seinem Zustand, sodass ein still stehen gebliebener Sampler sichtbar ist, ohne die Prüfung zu öffnen.

  **Sampler, in Echtzeit** (in `status` und nach `start`) aktualisiert sich alle 5 Sekunden, solange der Tab offen ist: der Zustand, Intervall und Aufbewahrung, wie lange er läuft, Stichproben und geschriebene Zeilen, wie viele Sitzungen in der letzten Stichprobe aktiv waren, wie lange diese her ist (gelb, sobald sie drei Intervalle zu spät ist), was auf der Platte liegt und wie viele Tage es umfasst, und was die Aufbewahrung entfernt hat.

  Platzbedarf: eine Zeile je aktiver Sitzung und Probe, etwa 450 Byte mit ihrem Abfragetext. Gemessen bei einer Probe pro Sekunde mit durchschnittlich zwei aktiven Sitzungen: 2,8 MiB pro Stunde, etwa 67 MiB pro Tag; bei einer Probe alle 5 Sekunden ein Fünftel davon. Der Platz wächst mit der Zahl aktiver Sitzungen und der Länge ihrer Anweisungen, nicht mit der Zeit allein, deshalb beträgt die Retention 30 Tage, sofern nicht anders gewählt. Auf dem Server ist eine Probe ein Lesen von pg_stat_activity, etwa 0,3 ms. JSONL hält das Format trivial für `jq`.

**db-trace-session (Tracing im Oracle-Stil)**: Sampler je Backend mit drei Zielmodi und zwei Filtern. *Optimierung und Diagnose* > *Sitzungs-Trace* öffnet seinen Tab, einen pro Server, mit drei Ansichten. **Live** zeigt, was der laufende Trace gerade tut: wem er folgt, wie weit er in seinem Fenster ist, die bisherigen Samples und erfassten Anweisungen, jede verfolgte Sitzung mit Zustand, Wartegrund und Anweisung, und die häufigsten Wartegründe; es aktualisiert sich alle zwei Sekunden, der Tab öffnet sich darauf, solange für diesen Server ein Trace läuft, und ein aus dem Formular gestarteter Trace läuft darauf und endet im Bericht. **Neu** ist das Formular, über dem Ergebnis statt in einem Dialog: wem gefolgt wird (jeder Sitzung eines Benutzers, jeder Sitzung, deren application_name zu einem Muster passt, oder einem Backend aus der Liste der gerade verbundenen), das Fenster (30 s, 1, 2, 5, 10 oder 30 Minuten, oder eine beliebige Zahl von Sekunden von 10 bis 1800; standardmäßig 2 Minuten), die Anweisungsart und der Filter auf den Abfragetext, dann **Trace starten**. Vor dem Start fragt der Trace, wie ASH und AWR, nach den Erweiterungen und Einstellungen, die er liest, wenn eine fehlt, die er braucht, sofern diese Frage nicht abgeschaltet ist; die Ansicht Neu sagt in einer Zeile, wie viele davon vorhanden sind, und **Einstellungen prüfen...** zeigt dasselbe Fenster jederzeit, dessen Starten den Trace startet. Während er läuft, sagt der Kopf, wem er folgt und wie viel des Fensters vergangen ist, und **Stopp** beendet ihn früher: der Bericht umfasst, was bis dahin erfasst wurde. Der Tab zeigt dann **Bericht**, mit der Exportzeile (Text, CSV, JSON, PDF). Der Bericht endet mit **Einstellungen für eine bessere Analyse**, der Tabelle, mit der der ASH-Bericht endet (siehe [Einen ASH-Bericht lesen](#reading-an-ash-report)), angeführt von dem, was entscheidet, wie viel ein Trace sieht: `auto_explain` mit `log_min_duration` und `log_analyze` (der tatsächlich genutzte Plan geht ins Serverlog), `log_min_duration_statement`, `log_lock_waits`, `compute_query_id` und `track_activity_query_size`. **Verlauf** listet die für diesen Server gespeicherten Traces, die neuesten zuerst: wann jeder begann, wem er folgte, das Fenster und wie lange er wirklich lief (gestoppte sind markiert), wie viele Anweisungen er erfasste und wie groß er ist. **Öffnen** zeigt einen als seinen Bericht, nur lesend und in denselben vier Formaten exportierbar; **Löschen** entfernt ihn nach einer Rückfrage. Traces werden je Server aufbewahrt, bis sie gelöscht werden (die neuesten 500), wie viele andere Prüfungen in der Zwischenzeit auch laufen. **Alles exportieren**, oder die gewählten, schreibt sie in ein Zip; **Importieren...** liest ein solches Zip und überschreibt nie einen Trace, der schon hier ist; der Export der App-Daten nimmt sie mit den AWR-Snapshots und ASH-Tagen des Servers mit. Stopp beendet die *Aufzeichnung* im AWR-Tab auf dieselbe Weise früher.

  - target_mode=pid: verfolgt ein einzelnes Backend über die PID (aus der Liste der gerade verbundenen Backends gewählt).
  - target_mode=application_name: verfolgt jedes Backend, dessen application_name einem POSIX-Regex entspricht. Nützlich, wenn der Pool die PIDs wechselt, der Anwendungsname aber gleich bleibt.
  - target_mode=user: verfolgt jedes Backend einer bestimmten Rolle. Für die Frage „was macht dieser Berichtsbenutzer gerade".
  - duration_seconds: 10 bis 1800 Sekunden (Standard 120). Der Sampler läuft das ganze Fenster mit 2 Hz.
  - category_filter: alle / dml / ddl / select / utility. Verwirft Zeilen, deren Abfragetext nicht zur gewählten Kategorie passt.
  - query_filter: optionaler Regex über den Abfragetext, Groß- und Kleinschreibung egal. Leer = kein Filter.

  Das Tracing prüft nebenbei, ob `auto_explain` geladen ist, und nennt die genauen SET-Befehle, mit denen die Planerfassung für die Zielsitzung vor Beginn der Last eingeschaltet wird. Ausgabe: Abfolge der Abfragen (Übergänge nach Zeit), Histogramm der Wait-Events für die Zielbackends, Delta je Abfrage aus pg_stat_statements. Auf PostgreSQL 13 gibt es die Spalte `query_id` noch nicht; das Tracing läuft ohne sie und merkt das an.

### 5.2 SQL {#the-sql-editor}

SQL-Editor mit PostgreSQL-Syntaxhervorhebung und eigener Vervollständigung. Werkzeugleiste:

- Ausführen (`Strg/Cmd + Enter`, `F5`): alles im Editor ausführen
- Anweisung (`Strg/Cmd + Umschalt + Enter`): nur die Auswahl oder die Anweisung unter dem Cursor ausführen
- Prüfen (`Strg/Cmd + E`): parsen und planen, ohne auszuführen
- Explain: EXPLAIN / EXPLAIN ANALYZE / BUFFERS
- Export CSV: führt die Abfrage auf dem Server erneut aus, damit die Datei alle
  Zeilen enthält und nicht nur die angezeigte Seite
- Speichern als…: schreibt das angezeigte Ergebnis als JSON, JSON Lines,
  SQL-`INSERT`, Markdown-Tabelle oder ausgerichteten Text
- Verlauf leeren

Unter dem Editor liegt der Ausgabebereich mit:
- Ergebnisraster (siehe unten)
- Live- und Statistikverlauf (einklappbar)
- Zeit und Zeilenzahl je Ausführung

**Gespeicherte Abfragen.** **Save query** (Strg+S) speichert den Text des
Editors unter einem Namen; die Liste **saved** daneben öffnet eine in einem
Tab. **SQL → Gespeicherte Abfragen…** (oder *manage saved queries…* am Ende
dieser Liste) öffnet ihren eigenen Dialog: eine Liste, durchsuchbar nach
Name, Notiz und SQL, und für die gewählte ihr Name, eine Notiz und das SQL
zum Bearbeiten, **Im Editor öffnen**, **Duplizieren** und **Löschen**. Eine
Änderung wird erst gespeichert, nachdem **Prüfen und speichern** sie neben
dem Gespeicherten gezeigt hat - vorher links, nachher rechts, eine Zeile,
die nur eine Seite hat, rot oder grün, geänderte Wörter markiert - und Sie
**Änderungen speichern** drücken. Jede gespeicherte Änderung behält die
Version davor (die letzten 25): **Frühere Versionen** vergleicht jede davon
genauso mit der aktuellen und kann sie in die Felder zurücksetzen, zum
Prüfen und Speichern. Ein Tab, der aus einer gespeicherten Abfrage geöffnet
wurde, merkt sich das: **Save query** zeigt dort die Änderung nebeneinander
und bietet **Aktualisieren** oder **Als neu speichern**. Gespeicherte
Abfragen haben keine Grenze; die Liste **history** daneben behält die
letzten 30 ausgeführten Abfragen. Der Arbeitsbereich je Verbindung behält
Ihre offenen Tabs über Neustarts hinweg.

#### EXPLAIN als Baum {#explain-as-a-tree}

Explain und Explain Analyze öffnen standardmäßig als **Baum**: eine Zeile pro Planknoten, unter seinem Elternteil eingerückt, jede zeigt die Kostenschätzung des Planers, geschätzte Zeilen und (bei Analyze) tatsächliche Zeilen, die Eigenzeit dieses Knotens und seine Gesamtzeit einschließlich Kindern, sowie die Anzahl der Durchläufe. Zwei Dinge werden automatisch hervorgehoben, statt sie in einer Wand von Zahlen selbst suchen zu müssen:

- **teuerster** markiert den einzigen Knoten mit der meisten *Eigen*-Zeit - seine eigene Zeit abzüglich der Zeit jedes Kindes. Das ist absichtlich nicht dasselbe wie "der Knoten mit der größten Gesamtzeit", was fast immer der äußerste Knoten ist, einfach weil er alle darunter einschließt; die Eigenzeit findet, wohin die Zeit tatsächlich ging.
- **schlechteste Schätzung** markiert den Knoten mit der größten Kluft zwischen dem, was der Planer erwartet hat (`Plan Rows`), und dem, was tatsächlich herauskam (`Actual Rows`), in beide Richtungen. Eine große Kluft hier, besonders gepaart mit einem Seq Scan, von dem der Planer dachte, er würde wenig zurückgeben, ist der häufigste Grund, warum ein Plan schiefgeht - meist bedeutet es, dass die Statistiken der Tabelle veraltet sind (`ANALYZE` behebt das) oder die WHERE-Klausel eine Korrelation hat, die `pg_stats` nicht sehen kann.

Badges pro Spalte: `cost` (Schätzung des Planers, willkürliche Einheiten, nur innerhalb desselben Plans vergleichbar), `rows` (geschätzt → tatsächlich, mit einem "Nx"-Multiplikator, sobald die Abweichung 3x oder mehr beträgt), `self`- und `total`-Zeit, und `×N loops`, wenn ein Knoten unter einem Nested Loop mehr als einmal lief. Ein kurzer Hinweis erscheint direkt unter dem Knoten, um den es geht - ein großer Seq Scan, ein Sort, das auf die Festplatte übergelaufen ist, ein Hash Join mit mehr als einem Batch, ein Nested Loop über eine große äußere Seite - dieselben Kategorien, die die Zusammenfassungs-Badges oben bereits verwendet haben, jetzt aus den eigenen Feldern des Knotens gelesen statt aus dem gedruckten Text erraten, und dort gezeigt, wo das Problem tatsächlich liegt, statt in einer separaten Liste darunter.

Ein Seq Scan wird nach den Zeilen beurteilt, die er **liest**, nicht nach denen, die er zurückgibt: unter EXPLAIN ANALYZE sind das die behaltenen Zeilen plus die vom Filter entfernten, über alle Worker, und ohne ANALYZE zählen auch hohe Scan-Kosten. Wirft der Filter fast alles weg, nennt die Zeile, wie viele Zeilen gelesen und wie viele behalten wurden - der Fall, den ein Index auf dem gefilterten Ausdruck löst. Kleine Tabelle heißt nur ein Scan, der wenig liest.

**Textansicht** wechselt zu einem einfachen, kopierbaren Textblock - dieselben Informationen, im Format von PostgreSQL EXPLAIN, zum Einfügen in einen Chat oder ein Ticket. Er wird aus demselben geparsten Plan gerendert, den der Baum liest (kein zweiter Aufruf an den Server): EXPLAIN ANALYZE führt die Abfrage tatsächlich aus, sodass PostgreSQL separat um das Baum- und das Textformat zu bitten bedeuten würde, die Abfrage zweimal laufen zu lassen - echte Kosten für eine langsame Abfrage, und bei einer datenverändernden Anweisung (in eine Transaktion eingepackt, die ohnehin zurückgerollt wird) doppelt so viele Schreibvorgänge vor dem Rollback. Der Kompromiss ist, dass dieser Text eine enge, lesbare Rekonstruktion ist, nicht garantiert byte-genau das, was `EXPLAIN (FORMAT TEXT)` nativ ausgegeben hätte - die Zahlen und die Struktur sind exakt, die echte PostgreSQL-Ausgabe ist das Einzige, was das erzwingt, nur die umgebende Zeichensetzung wird von Hand nachgebildet.

#### Das Ergebnisraster

Ein `SELECT` holt die ersten 500 Zeilen und zeigt sie sofort. Scrollen Sie
ans Ende, kommen die nächsten 500, und so weiter; die Statuszeile sagt
`500 von 48.213 Zeilen`, damit Sie immer wissen, wo Sie im Ergebnis stehen.
Die Zahl wird einmal vorab ermittelt - bei einer Tabelle, die so groß ist,
dass das Zählen zu lange dauern würde, sagt sie das statt einer Zahl, und das
Blättern funktioniert weiter.

Die Zeilen bleiben zwischen den Seiten auf dem Server, auf einem Cursor, der
geschlossen wird, sobald Sie etwas anderes ausführen, den Tab wechseln oder
ihn ein paar Minuten unberührt lassen. Der Editor hält hinter Ihrem Rücken
keine Transaktion offen.

Ab 5.000 Zeilen im Raster stoppt das automatische Laden, und ein Knopf
**500 weitere laden** übernimmt, damit ein achtloser Scroll auf einem riesigen
Ergebnis das Fenster nicht füllt. Für das ganze Ergebnis streamt `Export CSV`
jede Zeile auf dem Server, ganz ohne das Raster.

Am Raster selbst gibt es kein Sortieren und kein Filtern. Es hält immer nur
einen Teil des Ergebnisses, ein Sortieren würde also 500 von N ordnen und so
aussehen, als hätte es alles geordnet. Sortieren Sie in der Abfrage, mit
`ORDER BY`.

#### Das search_path-Feld

Neben der Modusauswahl legt `search_path` fest, wo unqualifizierte Namen für
diesen Lauf gesucht werden. Bleibt es leer, gilt der Sitzungskontext aus der
Kopfzeile.

Der Pfeil am Ende des Feldes klappt die Schemas der verbundenen Datenbank auf
 - nur die, die diese Sitzung auch betreten darf, denn ein Schema ohne `USAGE`
ist keine Antwort. Ihre eigenen zuerst, `pg_catalog` und `information_schema`
zuletzt unter eigener Überschrift. Ein Klick auf einen Namen fügt ihn hinzu,
ein zweiter nimmt ihn wieder heraus, und alles, was schon im Pfad steht, ist
abgehakt - der ganze Zustand des Feldes an einer Stelle. Das Menü bleibt beim
Auswählen offen, und das Feld lässt sich weiterhin tippen.

Das zählt mehr, als es aussieht - ein falsch geschriebenes Schema im
search_path ist kein Fehler. PostgreSQL akzeptiert einen Pfad, der nicht
vorhandene Schemas nennt, und findet Ihre Tabelle dann nicht, was sich liest
wie „die Tabelle ist weg" statt „der Pfad ist falsch".

#### Was „Ausführen" tatsächlich ausführt

Der Editor rät nie an einer Anweisungsgrenze, die Sie nicht sehen. Zwei Schaltflächen, zwei Umfänge:

| Schaltfläche | Tastenkürzel | Führt aus |
|---|---|---|
| `▶ Ausführen` | `Strg/Cmd + Enter` oder `F5` | jede Anweisung im Tab, der Reihe nach |
| `▶| Anweisung` | `Strg/Cmd + Umschalt + Enter` | den markierten Text, oder - ohne Markierung - die Anweisung, in der der Cursor steht |

„Die Anweisung, in der der Cursor steht" reicht vom vorherigen `;` bis zum
nächsten. In Dollar-Zeichen eingefasste Rümpfe (`$$ ... $$`), Zeichenketten und
Kommentare sind von dieser Suche ausgenommen, sodass eine Funktion, deren Rumpf
`BEGIN ... ; ... END` enthält, als eine Anweisung zählt und nicht als vier.

`F5` lädt im Browser die Seite neu; DBApguin fängt es ab. In der SQL-Ansicht
führt es aus, sonst aktualisiert es die aktuelle Ansicht. `Strg/Cmd + R` bleibt
überall die Aktualisierung, auch in der SQL-Ansicht.

#### Transaktionen und Autocommit {#transactions-and-autocommit}

Die Auswahl neben „Ausführen" entscheidet, was um die Anweisungen eines Laufs herum geschieht:

- **autocommit** (Standard) - jede Anweisung committet für sich, so wie `psql`
  es tut. Was Sie ausgeführt haben, ist mit der Rückkehr dauerhaft, und ein
  Fehler auf halbem Weg lässt die vorherigen Anweisungen angewendet.
- **Transaktion** - der ganze Lauf steckt in einem `BEGIN ... COMMIT`. Scheitert
  eine Anweisung, wird alles zurückgerollt und die Datenbank ist genau wie
  zuvor. Das ist der Modus für eine Migration.
- **Probelauf** - der Lauf steckt in `BEGIN` und endet immer mit `ROLLBACK`. Sie
  bekommen Zeilenzahlen, Fehler und Zeiten, ohne eine Spur zu hinterlassen.
  Nichts davon bleibt, auch nicht das Geglückte.

**read-only** entscheidet, ob die Anweisungen dieses Tabs in einer
Nur-Lese-Sitzung laufen. Standardmäßig aus, pro Tab, und die Anwendung rät
nicht: eingeschaltet lehnt PostgreSQL alles Schreibende ab - so lässt sich ein
Migrationsskript vor einen Produktionsserver stellen und sehen, wo es stehen
bliebe.

Früher wurde das aus dem SQL selbst geschlossen, und dieser Schluss lässt sich
nicht richtig ziehen. Das erste Schlüsselwort sagt nicht, ob eine Anweisung
schreibt: `IMPORT FOREIGN SCHEMA` sieht nach keinem von beidem aus, und
`SELECT cron.schedule(...)` - genau so, wie pg_cron in seiner eigenen
Dokumentation das Einplanen eines Jobs beschreibt - schreibt eine Zeile und
sieht dabei wie eine Abfrage aus. Beides kam mit *„cannot execute ... in a
read-only transaction"* zurück, auf einem Primary, wegen einer Einschränkung,
die die Anwendung selbst gesetzt hatte, während dieselbe Anweisung in `psql`
ohne Weiteres durchging. Es läuft so, wie es geschrieben ist.

Manche Anweisungen laufen überhaupt nicht in einer Transaktion: `CREATE INDEX
CONCURRENTLY`, `REINDEX CONCURRENTLY`, `VACUUM`, `CREATE DATABASE`. Führen Sie
diese im Modus **autocommit** aus. In den beiden anderen antwortet der Server
*„cannot run inside a transaction block"*, und der Editor nennt die
Auswahlliste, die zu ändern ist.

- **als Skript ausführen** - wie Autocommit, nur dass eine gescheiterte
  Anweisung den Lauf nicht beendet. Jede wird versucht, jeder Fehler wird mit
  seiner Nummer und seiner Meldung ausgegeben, und was durchgelaufen ist,
  bleibt committet. Das ist der Modus für eine DDL-Datei, von der ein Teil
  schon angewandt ist: Sie wollen den Rest drin haben und eine Liste dessen,
  was nicht ging. Das Ergebnis ist nur die rohe Ausgabe, wie `psql -f` sie
  druckt - Tag oder Zeilen jeder Anweisung, ihre Hinweise, ihr Fehler - mit
  **Copy** daneben; eine Tabelle gibt es nicht.
- **offen halten (Sie schließen selbst)** - die Transaktion bleibt **zwischen**
  den Läufen offen und endet, wenn Sie Commit oder Rollback drücken; beide
  erscheinen neben der Auswahl, solange sie offen ist. Etwas ausführen,
  ansehen, was es getan hat, und dann entscheiden.

  Eine offene Transaktion hält ihre Sperren, bis jemand sie schließt, deshalb
  steht ihr Zustand immer auf dem Schirm - wie viele Anweisungen darin sind
  und wie lange sie schon offen ist - und eine, die zehn Minuten niemand
  angefasst hat, wird für Sie zurückgerollt. Eine gescheiterte Anweisung
  beendet sie **nicht**: PostgreSQL markiert die Transaktion als abgebrochen,
  und nichts anderes geht mehr durch, bis Sie zurückrollen. Genau das sagt die
  Anwendung dann, statt Ihre Arbeit wegzuwerfen.

#### Wie lange eine Anweisung laufen darf

Neben der Modusauswahl steht ein Timeout: ein Häkchen und eine Sekundenzahl.
Jeder neue SQL-Tab startet mit **eingeschalteten 180 Sekunden**, und die
Einstellung gehört dem Tab - der eine hält eine Migration, auf die Sie eine
Stunde warten wollen, der andere Abfragen, an denen Sie noch feilen.

Greift es, wird die Anweisung **auf dem Server** abgebrochen, nicht nur in der
Anwendung. Nehmen Sie das Häkchen weg, läuft sie, bis sie fertig ist oder Sie
sie stoppen; die Anwendung fragt einmal nach, denn eine Anweisung ohne Grenze
hält ihre Sperren, solange sie läuft.

`ADD CONSTRAINT`, `CREATE INDEX` und ein großes `UPDATE` auf einer echten
Tabelle dauern alle länger als die früheren festen dreißig Sekunden, die dies
ersetzt.

#### DDL, exklusive Sperren und die Sperr-Warteschlange {#ddl-exclusive-locks-and-the-lock-queue}

`DROP`, `ALTER TABLE`, `TRUNCATE`, `REINDEX` (ohne `CONCURRENTLY`), `CLUSTER`
und `VACUUM FULL` nehmen eine **AccessExclusiveLock** auf der Tabelle. Wichtig
ist, was passiert, *bevor* die Anweisung sie bekommt:

> Eine Anforderung für eine exklusive Sperre überholt die Warteschlange nicht.
> Während sie wartet, stellt PostgreSQL jede neue Sperranforderung auf dieser
> Tabelle hinter sie - auch ein einfaches `SELECT`. Ein `DROP`, das nur
> *wartet*, blockiert also bereits neue Lese- und Schreibzugriffe auf diesen
> Tabellen, bevor es etwas geändert hat.

So wird aus einem `DROP SCHEMA` hinter einem langsamen Report ein
clusterweiter Ausfall. Der Editor behandelt diese Anweisungen anders als eine
normale Abfrage:

- **Eine Vorabprüfung, bevor sie läuft.** Wenn die Anweisung eine exklusive
  Sperre nimmt und auf dem Ziel schon etwas aktiv ist, listet ein Dialog diese
  Sitzungen auf - wer, woher, wie lange und ob eine schon in der Warteschlange
  steht - und erklärt die Wartezeit, bevor Sie sich festlegen. Eine Sitzung in
  der ersten Sekunde einer Anweisung, in einer nicht älteren Transaktion, steht
  nicht darin: Auf einer viel genutzten Tabelle gibt es immer eine (die kurzen
  Anweisungen der Anwendung, das Dashboard dieser App beim Lesen der
  Tabellengrößen), und sie ist vorbei, bevor die DDL es merken würde. Eine offen
  gelassene Transaktion - in einem anderen Programm oder in einem anderen
  Editor-Tab dieser App - steht darin, wie jung sie auch ist, ebenso eine
  Anweisung, die schon länger läuft.
- **Die Sperre jeder Anweisung, bevor sie läuft.** Die Bestätigung vor dem Ausführen listet jede DDL-Anweisung mit der Sperre, die sie nimmt, und worauf, und sagt, wenn sie die Tabelle neu schreibt oder ganz liest - beim Server erfragt, wo der Text es nicht verrät: `ADD COLUMN ... DEFAULT clock_timestamp()` (jeder volatile Default - `random()`, `gen_random_uuid()`, ein `serial`) schreibt jede Zeile neu, ein konstanter Default oder `now()` nicht (PostgreSQL 11+); `varchar(n)` auf ein längeres `varchar` oder auf `text` zu erweitern, oder `numeric` auf mehr Stellen bei gleicher Nachkommastellenzahl, ist eine Katalogänderung, Zeilen und Indizes bleiben; `ADD FOREIGN KEY` ohne `NOT VALID` liest die ganze Kindtabelle, während Schreibzugriffe auf beide Tabellen warten, also mit `NOT VALID` anlegen und danach `VALIDATE CONSTRAINT` - das bei einem Fremdschlüssel auch ROW SHARE auf die referenzierte Tabelle nimmt, sodass nur DDL wartet, die diese Tabelle exklusiv sperrt; das Löschen eines Fremdschlüssels nimmt auch die referenzierte Tabelle unter ACCESS EXCLUSIVE, und jedes Lesen von ihr wartet - ebenso das Löschen einer Spalte, aus der ein Fremdschlüssel besteht, das Löschen einer Tabelle, die eine andere referenziert (mit `CASCADE` auch der Tabellen, die sie referenzieren), und die Typänderung einer Schlüsselspalte auf beiden Seiten, und die Bestätigung nennt diese andere Tabelle; `VACUUM FULL`, `CLUSTER` und `REPACK` halten ACCESS EXCLUSIVE und schreiben die Tabelle neu; `REINDEX` ohne `CONCURRENTLY` hält fast jede Abfrage auf der Tabelle an. Vergleich -> Anwenden liest die Anweisungen genauso.
- **Ein voreingestelltes Sperrwarte-Limit.** Exklusiv sperrendes DDL aus dem
  Editor bekommt einen **3-Sekunden**-`lock_timeout`, unabhängig vom
  Anweisungs-Timeout-Kästchen. Bekommt es die Sperre nicht in drei Sekunden,
  gibt es auf, und nichts hat länger als das dahinter gewartet. Das ist ein
  Schutz, nicht das Anweisungs-Timeout: es begrenzt nur die Wartezeit *auf die
  Sperre*, nicht die Arbeit danach.
- **Den Schutz abschalten.** Die Vorabprüfung hat ein Kästchen *"So lange
  warten wie nötig"*. Ankreuzen, und das DDL wartet unbegrenzt auf die Sperre -
  tun Sie das nur, wenn Sie die Last stoppen können, denn bis das DDL
  durchkommt, ist auch der Verkehr auf diesen Tabellen gestoppt.

Wenn der Schutz auslöst, sagt die Meldung das - welches Limit, und dass es der
DDL-Schutz des Editors ist und nicht ein `statement_timeout` oder ein
serverseitiges `lock_timeout` - und behält das letzte Bild der Warteschlange
auf dem Bildschirm.

#### Während sie läuft

Die Statuszeile führt eine Uhr, damit eine Anweisung, die vier Minuten
braucht, nie wie eine Anwendung aussieht, die nicht mehr antwortet. Sobald die
Anweisung auf dem Server ist, fragt die Anwendung über eine zweite Verbindung
auch, was sie tut:

- **wartet auf eine Sperre** - steht vor allem anderen. Eine wartende
  Anweisung ist nicht langsam, sie steckt fest, und beides verlangt eine
  andere Reaktion. Bei einer Sperrwartezeit zeigt der Editor die ganze
  Warteschlange: wie viele Sitzungen die Sperre vor Ihnen halten, wie lange
  Sie gewartet haben und - der Teil, den nichts sonst zeigt - wie viele neue
  Abfragen sich schon hinter Ihnen aufgestaut haben. Schaltflächen dort öffnen
  die Blockierungsansicht oder beenden die Sperrhalter.
- **Fortschritt**, für die Operationen, für die PostgreSQL ihn meldet - 
  `CREATE INDEX`, `VACUUM`, `ANALYZE`, `CLUSTER`, `COPY` - mit der Phase und
  einem Balken.

Der Balken erscheint nur, wenn der Server wirklich einen Prozentsatz meldet.
Ein erfundener wäre schlechter als keiner.

**Stopp** steht neben Ausführen, solange etwas läuft, und stoppt es auf dem
Server - dem einzigen Ort, von dem aus das geht: die Anfrage, die es gestartet
hat, hängt auf derselben Verbindung und wartet auf dieselbe Antwort. Escape
tut dasselbe. Die Anweisung kommt als auf Anforderung abgebrochen zurück
(57014), und die Sitzung überlebt - samt einer Transaktion, die Sie offen
halten.

Ein Abbruch ist eine Bitte, kein Befehl, und ein Backend tief in einem
Systemaufruf bemerkt ihn nicht. Landet er nicht in ein paar Sekunden, bietet
die Schaltfläche **Erzwingen** an, was stattdessen die Verbindung schließt.
Das gelingt immer und kostet die Sitzung: eine offene Transaktion wird damit
zurückgerollt, deshalb wird vorher gefragt.

Der Lauf gehört dem Tab, in dem er gestartet wurde. Verlassen Sie diesen Tab,
wird sein Schließkreuz zu einem drehenden Ring - ein Haken, wenn die Anweisung
ankommt, ein Kreuz, wenn sie mit einem Fehler endet. Kommen Sie zurück, ist
das Kreuz wieder da, denn dann steht das Ergebnis vor Ihnen. Tabs, in denen
nichts von ihnen läuft, bleiben unangetastet.

#### Das Ausgabefeld

Unter dem Ergebnis zeigt die Anwendung, was tatsächlich passiert ist - dasselbe,
was `psql` ausgibt, und eines, das man anderswo leicht übersieht.

Für jede Anweisung des Laufs: das Kommando-Tag (`UPDATE 3`, `CREATE INDEX`,
`SELECT 1`), wie viele Zeilen sie berührt hat, wie lange sie gedauert hat und
die Anweisung selbst. Ein Skript aus fünf Anweisungen sagt so, welche langsam
war, statt einer Summe für alle fünf.

Darunter die **Meldungen des Servers**: alles, was `RAISE NOTICE` in einer
Funktion oder einem `DO`-Block erzeugt hat, und der laufende Kommentar von
`VACUUM VERBOSE`. Wer PL/pgSQL mit Notices debuggt, findet sie hier.

Die Kopfzeile trägt die Startzeit und die Gesamtdauer, und **Copy** legt das
ganze Feld in die Zwischenablage - für ein Ticket oder eine Mail.

Scheitert eine Anweisung, ist dieses Feld die nützlichere Hälfte der Antwort:
die aufgeführten Anweisungen sind die, die bereits gelaufen sind. Im
Autocommit sind sie festgeschrieben und werden vom Fehler nicht rückgängig
gemacht; im Transaktionsmodus wurde der ganze Lauf zurückgerollt.

#### Das Ergebnis in einem anderen Format speichern

`Export CSV` lässt den Server die Abfrage erneut ausführen und jede Zeile
streamen - das ist die Wahl für eine ganze Tabelle. Die Auswahl **speichern
als…** daneben schreibt das bereits angezeigte Ergebnis, also das, was Sie
lesen:

| Format | Wofür |
|---|---|
| JSON | eine Entwicklerin, die Testdaten einfügt |
| JSON Lines | ein Objekt pro Zeile - `jq`, eine Log-Pipeline, ein Massenimport |
| SQL-`INSERT` | die Zeilen in eine andere Datenbank laden; stammt das Ergebnis aus einer Tabelle, wird deren echter Name verwendet |
| Markdown-Tabelle | ein Ticket, ein Merge Request, eine Wiki-Seite |
| Ausgerichteter Text | eine Mail oder ein Terminal, ohne Werkzeug lesbar |

Jedes Format läuft über denselben Dateidialog wie jeder andere Export: Sie
wählen den Ort, und die Anwendung nennt danach den vollen Pfad.

### 5.3 Schema

Detailansicht eines einzelnen Objekts. Reiter:
- Spalten (mit Standardwert, Typ, NULL-Zulässigkeit, Kommentar)
- Indizes
- Constraints
- Trigger
- Rechte (bearbeitbar)
- Fremdschlüssel-Beziehungen (ein- und ausgehend)
- Statistik
- DDL - bei einer Tabelle oder partitionierten Tabelle ist das der Spalten-Editor
  (unten); bei allem anderen die formatierte CREATE-Anweisung
- Daten - die Zeilen, direkt bearbeitbar bei einer Tabelle mit
  Primärschlüssel. Zwei Modi: **builder** setzt die Abfrage aus Limit,
  Spalten, Filtern und ORDER BY zusammen und zeigt das SQL, das ausgeführt
  wird; **raw SQL** lässt die ganze Abfrage selbst schreiben. Im Modus raw
  SQL ist das SQL-Feld bearbeitbar, Spaltenauswahl, Filter und ORDER BY
  treten zurück; das Feld beginnt mit dem, was der Builder hatte. Die
  Abfrage läuft nur lesend und muss eine SELECT-, WITH-, TABLE- oder
  VALUES-Anweisung sein; Limit begrenzt weiterhin die abgerufenen Zeilen,
  Strg+Enter führt sie aus. Das Raster bleibt nur bearbeitbar, wenn die
  Abfrage jede Spalte dieser Tabelle unter ihrem eigenen Namen liefert;
  sonst ist das Ergebnis nur lesbar. Zurück im Builder ist alles, wie es war.

Der Schema-Browser links steuert diese Ansicht. Ein Klick auf ein Objekt dort lädt es hier.

#### 5.3.1 Spalten bearbeiten {#editing-columns}

Wählen Sie eine Tabelle und öffnen Sie den Reiter DDL. Statt eines
schreibgeschützten CREATE TABLE erhalten Sie eine Zeile je Spalte, jede direkt
bearbeitbar:

| Feld | Was die Schaltfläche daneben tut |
|---|---|
| Name | `RENAME COLUMN` |
| Typ | `SET DATA TYPE`, mit optionalem `USING`-Ausdruck für Umwandlungen, die PostgreSQL nicht von selbst vornimmt |
| NOT NULL | `SET NOT NULL` oder `DROP NOT NULL`, je nach Häkchen |
| Standardwert | `SET DEFAULT` aus dem Feld, oder `DROP DEFAULT` |
| Aktionen | `DROP COLUMN`, mit CASCADE-Häkchen daneben |

Die Spalte Extra zeigt, was der Katalog weiß und was hier nicht bearbeitet
wird: PRIMARY KEY, `IDENTITY (ALWAYS)` oder `IDENTITY (BY DEFAULT)` sowie
`GENERATED STORED` oder `GENERATED VIRTUAL`.

**Spalte hinzufügen** unten nimmt einen Namen, einen Typ, ein optionales
NOT NULL und eines von:
- *kein Standardwert*
- *DEFAULT* - ein Ausdruck
- *GENERATED* - ein Ausdruck plus die Speicherart. STORED materialisiert den
  Wert beim Schreiben und braucht PostgreSQL 12 oder neuer; VIRTUAL berechnet
  ihn beim Lesen und braucht 18 oder neuer. Die App schreibt das Schlüsselwort
  immer aus, denn PostgreSQL 18 hat die Bedeutung eines nackten
  `GENERATED ALWAYS AS (...)` geändert - früher ein Fehler, jetzt VIRTUAL -
  und dieselbe Anweisung täte sonst auf verschiedenen Servern Verschiedenes.

Jede Aktion zeigt zuerst die genaue Anweisung, mit der Sperre, die sie nimmt,
und was diese blockiert, und führt nichts aus, bevor Sie bestätigen.

Vier davon - `SET DATA TYPE`, `SET NOT NULL` und `ADD COLUMN` mit einem
Standardwert oder einem GENERATED-Ausdruck - leisten Arbeit proportional zur
Größe der Tabelle: die erste schreibt jede Zeile neu, die zweite liest jede
Zeile, um das Constraint zu belegen. Diese laufen als Hintergrundaufgaben,
werden also auf einer großen Tabelle nicht mittendrin abgebrochen und laufen
weiter, wenn Sie das Panel schließen; zu sehen unter
**Datenbank → Hintergrundaufgaben**. Der Rest sind Katalogänderungen, die
sofort fertig sind und als gewöhnliche Anfrage laufen.

Sichten, materialisierte Sichten und Fremdtabellen sind hier nicht bearbeitbar -
ihre Spalten stammen aus ihrer Definition, der Reiter zeigt nur die Struktur.

#### 5.3.2 Objektmenüs: was jede Art kann {#object-menus}

Rechtsklick auf ein Objekt im Schema-Browser. Unter den Navigations- und
Export-Einträgen trägt jede Art die Befehle, die PostgreSQL für sie
tatsächlich hat - nicht eine gemeinsame Teilmenge. Zwei Regeln halten das
Menü auf jedem unterstützten Server ehrlich:

- **Nichts wird versteckt.** Ein Eintrag, für den dieser Server zu alt ist,
  erscheint ausgegraut, und sein Tooltip nennt die benötigte Version
  (`benötigt PostgreSQL 12 oder neuer`). Ein kürzeres Menü auf einem alten
  Server ließe offen, ob es die Option nicht gibt oder die App sie vergessen
  hat.
- **Die Sperre liegt nicht nur im Menü.** Jede Versions- und Objektart-Regel,
  die das Menü zeichnet, wird im dahinterliegenden Endpunkt erneut
  durchgesetzt, sodass eine wiederholte Anfrage nicht daran vorbeikommt.

**Index**

| Eintrag | Hinweise |
|---|---|
| REINDEX | Exklusive Sperre, schnell |
| REINDEX CONCURRENTLY | PostgreSQL 12+. Keine Schreibsperre; ein abgebrochener Lauf hinterlässt einen UNGÜLTIGEN Index |
| REINDEX mit Optionen… | CONCURRENTLY, VERBOSE (9.5+) und TABLESPACE (14+), jeweils nur wo vorhanden |
| Umbenennen / In Tablespace verschieben | |
| Speicherparameter… | Nur die, die diese Zugriffsmethode annimmt: `fillfactor`, `deduplicate_items` (btree, 13+), `fastupdate` und `gin_pending_list_limit` (gin), `pages_per_range` und `autosummarize` (brin), `buffering` (gist) |
| Statistik für einen Ausdruck setzen… | PostgreSQL 11+, und nur für eine Ausdrucksspalte - eine gewöhnliche Spalte wird auf jeder Version abgelehnt |
| Löschen | CONCURRENTLY und CASCADE als Kästchen, mit der Anweisung vorab |

Ein UNGÜLTIGER Index bekommt einen zusätzlichen Eintrag ganz oben, der das
sagt, mit Neuaufbau per Klick. Ein Index, der nur ein Constraint erzwingt,
nennt welches und lässt sich nicht für sich allein löschen.

**Tabelle, partitionierte Tabelle, materialisierte Sicht**

Ein Untermenü **Wartung** mit ANALYZE, VACUUM, VACUUM ANALYZE, VACUUM FULL,
REINDEX TABLE, REINDEX TABLE CONCURRENTLY (12+) und CLUSTER. Jedes läuft als
Hintergrundaufgabe: Panel schließen, zurückkommen, unter
**Datenbank → Hintergrundaufgaben** wiederfinden.

*VACUUM mit Optionen…* ist die vollständige Optionsliste, jede Zeile mit der
benötigten Version ausgegraut, wenn der Server älter ist:
`DISABLE_PAGE_SKIPPING` (9.6+), `SKIP_LOCKED`, `INDEX_CLEANUP` und
`TRUNCATE` (12+), `PARALLEL` (13+), `PROCESS_TOAST` (14+), `PROCESS_MAIN`,
`SKIP_DATABASE_STATS` und `BUFFER_USAGE_LIMIT` (16+). `PROCESS_TOAST`,
`PROCESS_MAIN` und `TRUNCATE` sind in PostgreSQL an und deshalb angehakt -
zum Abschalten abwählen.

*CLUSTER…* listet die Indizes der Tabelle und markiert den, nach dem sie
derzeit geclustert ist, denn `CLUSTER` ohne `USING` funktioniert nur dann.

**TRUNCATE…** ist ein eigener Eintrag mit ONLY, RESTART IDENTITY und
CASCADE und zeigt die Anweisung vorab. Auch sie läuft als Aufgabe: CASCADE
kann einen ganzen Baum referenzierender Tabellen erreichen, und die nötige
Sperre wartet unter Umständen erst hinter anderen Sitzungen.

Unter **Verwalten**: *Speicherparameter…* (`fillfactor`, `parallel_workers`
(10+), `toast_tuple_target` (11+), `vacuum_index_cleanup` und
`vacuum_truncate` (12+), der `autovacuum_*`-Block sowie das Insert-Paar ab
13), *In Tablespace verschieben…* und für eine gewöhnliche Tabelle
*Geloggt / ungeloggt…* (9.5+).

**Materialisierte Sicht** hat zusätzlich REFRESH, REFRESH CONCURRENTLY und
REFRESH … WITH NO DATA. Letzteres leert die Sicht und lässt jede Abfrage
darauf bis zur nächsten vollen Aktualisierung fehlschlagen, fragt also
separat nach.

**Sicht** - *Sicht-Optionen…*: `check_option` (9.4+), `security_barrier`
(9.2+) und `security_invoker` (15+). Letztere entscheidet, ob eine Abfrage
über die Sicht gegen Ihre Rechte oder die des Sicht-Eigentümers geprüft wird.

**Sequenz** - *Sequenz-Eigenschaften…* ist der ganze
`ALTER SEQUENCE`-Parameterblock in einem Formular: Typ `AS` (10+),
INCREMENT BY, MINVALUE, MAXVALUE, START WITH, CACHE, CYCLE und OWNED BY. Nur
geänderte Felder landen in der Anweisung, und die Vorschau zeigt genau, was
laufen wird. *Neu starten…* setzt den nächsten ausgegebenen Wert.

**Trigger** - ein Untermenü **Auslösen** mit allen vier Zuständen: ENABLE,
DISABLE, ENABLE REPLICA (löst nur aus, solange `session_replication_role`
gleich `replica` ist) und ENABLE ALWAYS (löst unabhängig davon aus, sodass
ein Replikations-Apply oder ein Restore ihn nicht stummschalten kann). Dazu
*Umbenennen…*.

**Funktion / Prozedur** - der Eigenschaften-Block im Editor trägt jetzt auch
COST, ROWS, PARALLEL (9.6+, darunter ausgegraut) und LEAKPROOF, neben den
bereits vorhandenen Feldern für Umbenennen / Schema / Eigentümer /
search_path.

**Schema** - ein Untermenü **Wartung**, das ANALYZE, VACUUM, VACUUM ANALYZE
oder REINDEX über jede gewöhnliche Tabelle und materialisierte Sicht im
Schema laufen lässt, eine nach der anderen, und über eine nicht sperrbare
Tabelle hinweg weitermacht. Das vollständige Panel mit clusterweitem Umfang
und allen Optionen steht weiterhin einen Eintrag darunter.

#### Eine Kopie der Definitionen vor der Änderung {#copy-before-the-change}

Ein Objekt in ein anderes Schema verschieben, Objekte löschen, Eigentümer
oder Tablespace ändern schreibt zuerst die Definitionen der Objekte in den
Datenordner, als SQL, das sie zurückbringt: für eine Tabelle die Spalten,
Constraints, Indizes, Trigger, Policies, Eigentümer, Rechte, Kommentare und
den Tablespace, in dem sie und ihre Indizes liegen; für eine Routine ihre
Definition. Hintergrundjobs listet die Kopie ("Before: ..."), die Meldung
nach der Änderung öffnet sie, und von dort geht sie in den SQL-Editor oder in
eine `.sql`-Datei. Die Kopie stammt aus `pg_dump --schema-only`, wenn ein zum
Server passendes pg_dump verfügbar ist, sonst aus dem Katalog. Lässt sich
keine Kopie erstellen, wird die Änderung mit Begründung abgelehnt; Sie können
ohne fortfahren, und das Audit-Log hält fest, dass Sie es getan haben. Eine
fehlgeschlagene Änderung rollt PostgreSQL selbst zurück; dies ist für die
Änderung, die gelungen ist und es nicht hätte sollen.

**Ein Löschen und alles, was es mitnimmt.** Der Rechtsklick **Drop** auf
einer Tabelle, Sicht, Sequenz, einem Index, einer Routine, einem Typ oder
Schema erstellt die Kopie ebenso wie das Löschen mehrerer Objekte - früher
erstellte er keine. Mit **CASCADE** enthält die Kopie alles, was das Löschen
entfernt, nicht nur das Genannte. Welche Objekte das sind, sagt PostgreSQL
selbst: das Löschen wird in einer Transaktion geprobt, die zurückgerollt
wird, und der Katalog sagt, was verschwunden ist. Gemessen nimmt ein
`DROP TABLE ... CASCADE` Sichten auf die Tabelle in anderen Schemas mit, eine
Sicht auf eine davon, materialisierte Sichten, eine Funktion, deren Argument
der Zeilentyp der Tabelle ist, und die Fremdschlüssel anderer Tabellen; ein
`DROP TYPE ... CASCADE` nimmt die Spalten dieses Typs aus anderen Tabellen,
samt Werten; eine gelöschte Triggerfunktion nimmt ihre Trigger mit, eine
gelöschte Sequenz die Defaults, die sie aufrufen. Der Dialog listet sie, wenn
Sie es anfordern (**Ermitteln, was der Drop mitnimmt**), aus dem
Abhängigkeitsgraphen gelesen: für die Liste wird nichts geprobt und nichts
gesperrt, auch für ein Schema mit Hunderten von Tabellen. Die Kopie nennt sie oben, genau so, wie
die Probe sie gefunden hat. Kann die Probe nicht laufen - eine Tabelle, die
sich nicht binnen drei Sekunden sperren lässt, oder Event-Trigger, die dabei
feuern würden - wird auch die Liste der Kopie aus dem Abhängigkeitsgraphen
gelesen, und die Kopie sagt, dass sie eine Schätzung ist. Was die Kopie
nicht schreiben kann, steht unter *NOT in this copy*.

**Was zuerst kopiert wird.** Der Lösch-Dialog und der Dialog, der mehrere
ausgewählte Objekte löscht, öffnen sich, ohne den Server etwas zu fragen.
Unter der Anweisung und ihren Optionen (**CASCADE**, **IF EXISTS**, das SQL)
fragen sie, in dieser Reihenfolge:

1. **Vor dem Drop eine Kopie behalten?** Die erste Wahl ist **Keine Kopie -
   nur das DROP ausführen**: nichts wird gesichert und nichts vorher
   ermittelt, nur die DROP-Anweisung läuft (für ein Schema mit **CASCADE**
   `DROP SCHEMA ... CASCADE` auf diesem Schema und sonst nichts). Sie lässt
   sich nicht rückgängig machen, außer aus einer Sicherung. **Zuerst eine
   Kopie behalten** öffnet die nächste Frage.
2. **Was behalten:** **Nur die Definitionen kopieren** (die Definitionen von
   allem, was das Löschen mitnimmt; eine Tabelle kommt leer zurück) oder
   **Definitionen und Zeilen kopieren** (auch die Zeilen der Tabellen, die
   gehen, werden in den Datenordner geschrieben).
3. **Ermitteln, was der Drop mitnimmt** - erst, wenn Sie es anklicken: die
   Objekte, die **CASCADE** entfernt, und für die Tabellen, die gehen, wie
   viele es sind, etwa wie viele Zeilen und wie viele Daten. Eine Kopie
   braucht das, bevor **Löschen** freigegeben wird; ohne Kopie können Sie es
   auch anfordern, müssen aber nicht.

Die Größen sind die Schätzung, die der Server führt (Zeilen und Seiten aus
seiner Statistik), gelesen, ohne eine Tabelle zu öffnen; eine Tabelle, die
nie gevacuumt oder analysiert wurde, hat noch keine Größe, und der Dialog
sagt das. Eine Kopie probt das Löschen zuerst, um genau zu ermitteln, was
geht, und das hält die Sperren des Löschens einen Moment lang (es wartet
höchstens drei Sekunden darauf). Der Dialog merkt sich Ihre letzte Wahl, bis
die Anwendung geschlossen wird (sie wird nirgends gespeichert): das nächste
Löschen beginnt damit, nur werden Zeilen über 50 MB zu den Definitionen
allein, sobald die Analyse das zeigt, und der Dialog sagt warum. Ohne Kopie
geben Sie, wenn das Löschen Daten oder andere Objekte mitnehmen kann (eine
Tabelle, ein Schema, alles mit **CASCADE**, außer die Analyse hat gezeigt,
dass nichts geht), den Namen des Objekts ein (bei mehreren Objekten ihre
Anzahl), bevor **Löschen** freigegeben wird. Eine Art, für die keine Kopie
geschrieben werden kann (eine Erweiterung, ein Trigger, eine Rolle ...),
sagt das und bietet nur die Analyse an.

**Während es läuft.** Das Löschen ist ein Hintergrundauftrag, und der Dialog
folgt ihm Schritt für Schritt: ermitteln, was das Löschen genau entfernt,
die Definitionen kopieren, die Zeilen kopieren (so viele von so vielen
Tabellen, wie viel geschrieben ist), löschen - mit der bisher vergangenen
Zeit. Bis das DROP selbst beginnt, heißt **Abbrechen** **Stopp**: es hält am
nächsten sicheren Punkt an, nichts wird gelöscht, und eine bis dahin
erstellte Kopie wird samt ihren Zeilen entfernt. Läuft das DROP, lässt es
sich nicht mehr anhalten, und der Knopf sagt es. Den Dialog zu schließen
hält den Auftrag nicht an; er bleibt in den Hintergrundaufträgen. Verschieben
und Klonen aus demselben Dialog zeigen denselben Arbeitszustand.

**Die Zeilen.** Die Definitionen sind nicht die Daten. **Definitionen und
Zeilen kopieren** schreibt die Zeilen jeder Tabelle, die
das Löschen mitnimmt, neben die Kopie in den Datenordner, und die Werte einer
Spalte, die ein Cascade aus einer bleibenden Tabelle entfernt (über den
Primärschlüssel zugeordnet; eine Tabelle ohne ihn kann sie nicht
zurückbekommen, und die Kopie sagt es). Um ein Löschen rückgängig zu machen:
die Kopie öffnen, im SQL-Editor ausführen, dann **Zeilen zurückspielen** -
eine Transaktion, Eltern vor Kindern, in Tabellen, die wieder existieren und
leer sein müssen, damit nichts doppelt geladen wird. Die Zeilen gehen dorthin
zurück, wo die Kopie erstellt wurde, und nirgendwo sonst: die Kopie hält den
Server fest (seine Adresse, den libpq-Dienst oder SSH-Tunnel und die eigene
Systemkennung des Servers) und die Datenbank, **Zeilen zurückspielen** nimmt
einen offenen Tab oder eine gespeicherte Verbindung, die diesen Server
erreicht - nie den vorderen Tab, wenn der woanders ist -, und der Server wird
gefragt, wer er ist, bevor eine Zeile geschrieben wird. **Im SQL-Editor öffnen**
öffnet die Kopie nur in einem Tab, der mit dieser Datenbank verbunden ist, und
sagt es, wenn es keinen gibt. Ein Fremdschlüssel
einer bleibenden Tabelle kommt `NOT VALID` zurück, wenn diese Tabelle Zeilen
hat, die auf eine leer zurückkommende Tabelle zeigen, und wird geprüft,
sobald die Zeilen zurück sind; dann werden auch materialisierte Sichten
aufgefrischt. Die Zeilen einer Fremdtabelle liegen auf dem anderen Server
und werden nicht gesichert. Mit den Definitionen allein sagt die Kopie oben,
dass sie nur die Definitionen enthält.

### 5.4 Sitzungen

Drei Modi (Umschalter oben):

- Liste: sortierbare Tabelle aus pg_stat_activity mit PID, Datenbank, Benutzer, Anwendung, Client, Zustand, Wartegrund, Dauer und Abfrage.
- Live-Last: Fortschritt und Restzeit je aktiver Abfrage. Verbindet die Historie aus pg_stat_statements (Mittelwert / Standardabweichung / Blöcke pro Aufruf) mit den pg_stat_progress_*-Views (VACUUM / ANALYZE / CREATE INDEX / CLUSTER / COPY).

#### „idle in transaction" und was es kostet

Eine Sitzung in diesem Zustand bekommt eine von drei Lesarten, und alle sind
gemessen statt angenommen:

| was dasteht | was es bedeutet |
|---|---|
| *N Sitzungen warten darauf* | jemand steckt dahinter fest. Das ist ein Blockadeproblem, und dieselben Sitzungen erscheinen in Blocking und in der Prüfung `db-locks` |
| *hält N Sperren, niemand wartet* | sie hat die Sperren genommen, die ihre Anweisungen brauchten, und hält sie bis zum Commit - nur will sie gerade niemand |
| *hält keine Sperre, auf die jemand warten könnte* | eine offen gelassene Nur-Lese-Transaktion. Sie blockiert niemanden |

Der dritte Fall ist der häufige, und er las sich früher wie alle anderen:
„holding locks". Das schickte die Leute in Blocking und in die Sperren der
Cluster-Gesundheit, wo sie zu Recht nichts fanden - die Beschriftung beschrieb
den Zustand, nicht die Sperren. Die eigene virtuelle Transaktions-ID zählt
nicht mit: die hat jede Sitzung, und niemand kann auf sie warten.

Eine lange offene Transaktion sollte man auch im dritten Fall schließen, und
die Karte sagt das nach fünf Minuten: die älteste offene Transaktion hält den
xmin-Horizont, autovacuum kann also keine Zeilenversion aufräumen, die jünger
ist als sie - nirgends in der Datenbank, nicht nur in den Tabellen, die diese
Transaktion angefasst hat.
- Dashboard: Momentaufnahme auf einem Bildschirm (Versionen, Verbindungszustände, Wait-Events, Cache-Trefferrate, heißeste Tabellen, Checkpoint-Statistik, Blockaden, Replikationsrückstand, längste offene Transaktion, Autovacuum-Worker).

Rechtsklick auf eine Sitzung: Abbrechen (die laufende Abfrage) oder Beenden (das Backend).

### 5.4.1 Was ein offenes Fenster den Server kostet {#what-an-open-window-costs-the-server}

Ein verbundenes Fenster fragt den Server alle fünf Sekunden eine Sache: die aktiven Sitzungen, die Fakten hinter den Alarmlampen der Kopfzeile und ob der Server Primary oder Standby ist. Was sich von einem Tick zum nächsten nicht ändert (welche Spalten und Funktionen er hat, wo pg_stat_statements liegt und ob es geladen ist, die Worker- und Archivierungseinstellungen), wird einmal gefragt und behalten, zehn Minuten lang oder eine Minute, wo eine Erweiterung es ändern kann; eine in der App installierte, aktualisierte, verschobene oder entfernte Erweiterung wird sofort neu erfragt. Die Ansicht Live load nutzt dieselbe Fünf-Sekunden-Antwort wie die Kopfzeile, statt ein zweites Mal zu fragen. Gemessen auf PostgreSQL 19, für ein Fenster:

| Fenster | Anweisungen pro Minute | Serverzeit pro Minute | Blöcke pro Minute |
|---|---:|---:|---:|
| Untätig (ein Prüfungs-Tab vorne) | etwa 500 | 14 ms | etwa 2.000 |
| Live load vorne | etwa 470 | 12 ms | etwa 1.500 |

Die meisten Anweisungen sind die wenigen Einstellungen, die jede Anfrage auf ihrer Verbindung setzt (Rolle, Suchpfad, Zeitlimits), jeweils unter 0,1 ms. Bei minimiertem Fenster wird nichts gefragt. Eine Prüfung kostet, was ihre eigenen Abfragen kosten, einmal, wenn Sie sie ausführen; der ASH-Sampler fügt je Probe ein Lesen von pg_stat_activity hinzu (siehe *db-ash* oben).

### 5.4.2 Die Alarme der Kopfzeile und ihre Linien {#header-alarms}

Die Alarmlampen der Kopfzeile werden aus derselben Antwort alle fünf Sekunden bewertet. Jede Linie ist ein Urteil, darum stehen sie hier mit ihrem Grund (geprüft am 27.10.2026, nachdem der Auslagerungs- und der Wraparound-Alarm beide blinden Alarm geschlagen hatten):

- **Verbindungen:** Warnung bei 80% von `max_connections`, kritisch bei 95%.
- **Blockierung:** Warnung, wenn jemand 5 s auf eine Sperre wartet, kritisch bei 60 s - eine Minute hinter einer Sperre löst sich nicht von selbst; fünfzehn Sekunden, die alte Linie, waren eine gewöhnliche Migration.
- **Auslagern auf die Platte:** die Rate ist ein 60-s-Mittel von `pg_stat_database.temp_bytes` und muss *oben bleiben*: Warnung nach drei Minuten über 2 MB/s, kritisch nach fünf Minuten über 20 MB/s. Ein Bericht, der einmal auslagert, schob das Mittel über die Linie, und der Alarm stand eine Minute; das tut er nicht mehr. Während `CREATE INDEX`, `REINDEX`, `CLUSTER` oder `VACUUM FULL` laufen (PostgreSQL 12+ sagt es), ist ihr Sortieren auf der Platte Sache von `maintenance_work_mem` und wird nicht bewertet.
- **Transaktions-ID-Wraparound:** `autovacuum_freeze_max_age` (standardmäßig 200 Millionen) ist der Punkt, an dem Autovacuum von selbst Anti-Wraparound-Vacuums beginnt, und beschäftigte Tabellen liegen im Normalbetrieb nahe daran - die alten Linien, 80% und 90% davon, lagen bei 8% des Wegs zum echten Stillstand. Der Alarm leuchtet nun beim Fünffachen der Einstellung, höchstens bei 1 Milliarde, und wird kritisch beim 7,5-Fachen, höchstens bei 1,5 Milliarden, wo auch der Health-Check CRIT sagt; PostgreSQLs Failsafe-Vacuum beginnt bei 1,6 Milliarden, und der Stillstand liegt nahe 2,1 Milliarden. Die Obergrenzen gelten für einen DBA, der die Einstellung erhöht hat; die Vielfachen lassen einen Testcluster die Linien durch Senken erreichen. Der Text nennt das Alter in Millionen und den Weg zum Stillstand in Prozent sowie die älteste Relation, ihre Datenbank, jede Relation über der Linie und den Snapshot-Halter, an dem Vacuum nicht vorbeikommt. Multixact-IDs werden nach derselben Regel gegen `autovacuum_multixact_freeze_max_age` bewertet und führen den Text an, wenn sie die älteren sind.
- **Eine offene Transaktion:** Warnung bei 5 Minuten. Kritisch bei 30 Minuten nur, wenn sie *idle in transaction* ist - etwas, das nichts beenden wird; eine so alte aktive Anweisung ist ein Bericht oder ein Batch und bleibt eine Warnung.
- **Vorbereitete Transaktionen:** Warnung nach einer Stunde, kritisch nach einem Tag.
- **Ein toter Replikationsslot:** Warnung, sobald er inaktiv ist, kritisch, sobald er 10 GB WAL festhält.
- **WAL-Archivierung:** Warnung, wenn der letzte Versuch fehlschlug; kritisch, sobald es fünf Minuten lang fehlschlägt - ein fehlgeschlagener Versuch wird eine Minute später wiederholt. Der Alarm nennt das gescheiterte Segment und wann, das zuletzt archivierte, den Befehl oder die Bibliothek und wie viele Segmente in `pg_wal` warten. `archive_mode` an ohne `archive_command` und `archive_library` ist eine Warnung, kritisch, sobald 10 GB WAL warten; beide gesetzt ist kritisch (ab PostgreSQL 16 verweigert der Archiver die Arbeit).
- **Worker-Pools:** Warnung bei 80% eines Pools, kritisch bei 95%.
- **Replikationsverzögerung:** Warnung bei 60 s, kritisch bei 300 s.
- **LISTEN/NOTIFY-Warteschlange:** Warnung bei 10% ihres Maximums oder 512 MB, kritisch bei 50% oder 4 GB, je nachdem, was zuerst kommt. Die Lampe erscheint, wo jemand lauscht (oder die Warteschlange schon etwas hält); sie führt zu *Queue-Tabellen und LISTEN/NOTIFY*, das die lauschenden Sitzungen nennt.

Ein kritischer Alarm öffnet das Fenster, das eine Bestätigung verlangt (im Audit-Log festgehalten) und auf die Aufgabenliste gesetzt werden kann; eine Warnung ist eine Lampe und eine Benachrichtigung. Die Frage vor einem kritischen Fenster lässt sich in den Einstellungen abschalten.

### 5.5 Wartung

Wartungsarbeiten und Autovacuum-Einstellung.

- Schema und Tabelle wählen, dann VACUUM / VACUUM (FULL) / VACUUM ANALYZE / REINDEX / CLUSTER / ANALYZE / pg_repack ausführen. Der Fortschritt kommt live aus pg_stat_progress_*.
- Autovacuum-Berater je Tabelle: reloptions bearbeiten (`autovacuum_vacuum_scale_factor`, `autovacuum_vacuum_threshold`, …) mit einem empfohlenen Wert aus der Hitze der Tabelle (sehr heiß / heiß / mittel / Standard). Auf PostgreSQL 13 und neuer bekommt eine Insert-only-Tabelle zusätzlich `autovacuum_vacuum_insert_threshold` und `autovacuum_vacuum_insert_scale_factor` empfohlen und gesetzt: Tote Zeilen erreichen ihren Vacuum-Auslöser nie, daher ist der Insert-Auslöser das einzige Vacuum, das sie bekommt.

Die Optionsliste richtet sich nach der Version des Servers selbst. PROCESS_TOAST
kam in 14, BUFFER_USAGE_LIMIT in 16; eine davon an einen aelteren Server zu
schicken ist ein harter Fehler und keine Warnung, also wird nichts angeboten,
was der Server nicht kennt. Die VACUUM- / ANALYZE- / REINDEX-Schaltflaechen im
Table-Advisor und im Vacuum-Status-Advisor nutzen dieselbe Liste.

Hier gibt es kein Log-Fenster. Ein Maintenance-Lauf ist ein Hintergrundjob wie
jeder andere: das Panel zeigt, dass etwas laeuft, und verweist auf
**Background jobs**, wo die Ausgabe neben allen anderen langen Operationen steht.

**Ein Lauf, der nichts getan hat, sagt das.** PostgreSQL ueberspringt eine
Relation, die der Aufrufer nicht pflegen darf, und meldet trotzdem Erfolg. Der
Job endet als *skipped* statt *done* und nennt, was uebersprungen wurde.

### 5.6 Server

Cluster-Einstellungen + pg_hba-Editor + Erweiterungen + Tablespaces + Rollen.

- Wichtige Einstellungen: bearbeitbare Karten für rund 70 Parameter mit Begründung, Faustregel und Statusabzeichen. Änderungen werden über ALTER SYSTEM gespeichert (wo der Anbieter es zulässt).
- pg_hba.conf: Texteditor, der vor dem Speichern sichert. Wiederherstellung aus jeder Sicherung. Der Anbieter muss direktes Schreiben von Dateien erlauben.

#### Jedes hier gemachte ALTER SYSTEM lässt sich rückgängig machen {#settings-undo}

Jede aus DBApguin angewandte Änderung - Schlüsseleinstellungen, die
Tuning-Hinweise, der Einstellungseditor - wird mit dem ersetzten Wert und
dessen Herkunft aufgezeichnet. Die Meldung, die die Änderung bestätigt,
trägt **Rückgängig**, und die Einstellungsansicht listet die hier gemachten
Änderungen, neueste zuerst, jede mit Rückgängig, bis sie zurückgenommen
ist. Rückgängig ist genau die Anweisung, die den Parameter zurückbringt:
`ALTER SYSTEM SET` auf den alten Wert, wenn dieser DBApguins eigene Zeile in
`postgresql.auto.conf` war, sonst `ALTER SYSTEM RESET`, sodass die alte
Quelle (postgresql.conf, der Standard) wieder gilt. Die Aufzeichnung ist je
Server im Datenordner, und das Audit-Protokoll hält den Wert vorher und
nachher fest.

#### pg_hba.conf gehört einem Knoten

`pg_hba.conf` ist eine Datei im Datenverzeichnis eines Servers, und die
Replikation trägt sie nicht: WAL überträgt Daten, keine Konfiguration. Eine auf
dem Primary hinzugefügte Regel steht nicht auf dem Standby, der weiterhin hat,
was sein Base-Backup enthielt. Der Tag, an dem das zählt, ist ein Failover - 
Verbindungen, die gestern funktionierten, weist ein Knoten ab, der die Änderung
nie bekam, und dann schaut niemand mehr auf diesen Bildschirm.

Der Editor sagt es deshalb über der Datei und nennt die Knoten, wo der Server
sagt, wer sie sind:

- **Standbys, die von diesem Server streamen**, mit Anwendungsname und
  Adresse, aus `pg_stat_replication`;
- **physische Replikations-Slots**, wenn ein Standby eingerichtet, aber gerade
  abwesend ist - ein Slot ohne Gegenstelle ist immer noch ein Knoten, den man
  zurückerwartet;
- **„dieser Knoten ist ein Standby"**, wenn Sie die Kopie auf einer Replik
  bearbeiten: der Primary hat seine eigene, und eine Änderung hier erreicht ihn
  nicht.

Ist nichts davon sichtbar, wird der Hinweis kürzer, bleibt aber. Eine Rolle
ohne `pg_monitor` kann `pg_stat_replication` nicht lesen, und ein getrennter
Standby sieht von hier aus wie einer, den es nicht gibt. Wie die Topologie auch
aussieht: dieselbe Änderung muss auf jedem Knoten gemacht werden - auch auf
einem Connection-Pooler davor, der eigene Regeln eigener Art hat.

**Das Bearbeiten der Datei auf einer Replik funktioniert.** Früher scheiterte
es an *cannot set transaction read-write mode during recovery* - eine Meldung
über Transaktionsmodi aus einem Vorgang, der eine Datei schrieb. Zwei Ursachen:
die Anwendung forderte eine Read-Write-Sitzung an, die ein Server in der
Wiederherstellung rundweg ablehnt, und die Datei wurde über ein Large Object
geschrieben, was ein Schreibzugriff auf die Datenbank ist. Beides ist unnötig.
Die Sitzung wird jetzt so angefordert, wie der Server sie geben kann, und auf
einem Standby wird die Datei mit `COPY … TO` geschrieben, das die Datenbank
nicht berührt. Sicherung vor dem Speichern, Reload danach und der Bericht über
Parse-Fehler arbeiten unverändert.
- Erweiterungen: Bestand sowie Installieren / Aktualisieren / Entfernen / Schema wechseln je Erweiterung, mit Versionsauswahl.
- Tablespaces: Bestand sowie Anlegen / Ändern / Löschen.
- Rollen: Liste mit Attributeditor. Ein Klick auf eine Rolle öffnet die vollständige Prüfung.

### 5.7 Rollen, Benutzer und Rechte {#roles-users-and-privileges}

#### Der Rolleneditor

`Übersicht über Rollen und Benutzer` listet Login-Benutzer und Gruppenrollen in
zwei bearbeitbaren Tabellen. Jedes boolesche Attribut ist eine true/false-Auswahl;
`conn_limit` und `valid_until` sind Textfelder.

**Erst die Rolle wählen, dann handeln.** Ein Klick auf die Zeile wählt sie aus,
das Kästchen ebenso; Ctrl-Klick und die Kästchen wählen mehrere. Eine Reihe von
Schaltflächen steht neben *+ Create role / user* und wirkt auf die Auswahl - sie
bleibt verborgen, solange nichts gewählt ist. Bei mehreren Rollen bleiben die
Aktionen aktiv, die für eine Menge sinnvoll sind (**Privileges**, **Lock**,
**Drop**, **Save**); die, die die Einstellungen einer einzelnen Rolle ändern,
werden ausgegraut und sagen warum.

**Speichern** sendet nur die Differenz, eine bloß angesehene Rolle löst also
nichts aus. Bei mehreren Ausgewählten je eine `ALTER ROLE` Anweisung pro
tatsächlich geänderter Rolle, zuvor in der Bestätigung aufgelistet.

**Edit** öffnet die Rolle in einem eigenen Tab - Attribute, Rechte und
Mitgliedschaften an einem Ort, ein Tab pro Rolle, sodass zwei Rollen
nebeneinander bearbeitet werden können. Das Rechteformular in diesem Tab ist
dasselbe, das die Schaltfläche **Privileges** als Dialog öffnet.

**Privileges** zeigt bei mehreren ausgewählten Rollen die aktuellen Rechte der
ersten und schreibt denselben GRANT / REVOKE Satz auf jede ausgewählte Rolle;
der Hinweis oben nennt sie. Es gibt keinen gemeinsamen aktuellen Zustand von
fünf Rollen - gemeinsam ist die Änderung - also ist die gezeigte Grundlage die
einer Rolle, und der Dialog sagt welche.

Gruppenrollen haben eine eigene Spalte **`can_login`**. Eine Gruppenrolle ist per
Definition `NOLOGIN`, die Spalte beginnt daher immer mit false - sie ist da,
damit Sie eine Gruppenrolle später zu einer Login-Rolle machen können, ohne die
Anwendung zu verlassen. Login-Benutzer haben für die Gegenrichtung eine
Schaltfläche **Sperren**, die nach Bestätigung `NOLOGIN` setzt.

#### Grenzen und Timeouts für eine Rolle

Öffnen Sie die Attribute einer Rolle, so enthält der Abschnitt **Grenzen und
Ablauf** neben dem Verbindungslimit und dem Ablaufdatum die fünf Timeouts pro
Sitzung:

| Einstellung | stoppt |
|---|---|
| `statement_timeout` | eine einzelne Anweisung, die zu lange läuft |
| `lock_timeout` | das Warten auf eine Sperre |
| `idle_in_transaction_session_timeout` | eine Sitzung, die in einer offenen Transaktion untätig ist |
| `idle_session_timeout` | eine Sitzung, die außerhalb einer Transaktion untätig ist (PostgreSQL 14+) |
| `transaction_timeout` | eine ganze Transaktion, wie beschäftigt auch immer (PostgreSQL 17+) |

Angeboten wird nur, was dieser Server hat - die letzten beiden kamen in
späteren Versionen, und ein Feld anzubieten, das der Server ablehnt, wäre
schlimmer, als es wegzulassen.

**Eine nackte Zahl sind Millisekunden.** PostgreSQL liest einen Wert ohne
Einheit in der Einheit der Einstellung selbst, und das sind bei allen fünf
Millisekunden: `30` bedeutet dreißig Millisekunden, was niemand meint, der 30
tippt. Schreiben Sie `30s`, `2min` oder `500ms`, dann bleibt nichts zu raten.
Ein leeres Feld löscht die Einstellung für den gewählten Bereich.

**Sonst** ist der Wert, auf den die Rolle trifft, wenn dieser Bereich nichts
setzt, und er nennt die Ebene, aus der er kommt - die rollenweite Einstellung
der Rolle, die der Datenbank oder die des Clusters, in der Reihenfolge, in der
PostgreSQL sie anwendet. Beim Cluster steht dabei, ob der Wert aus einer
Konfigurationsdatei stammt oder die eingebaute Vorgabe ist, denn das bedeutet
Verschiedenes, wenn man entscheidet, ob man selbst einen setzt. Gelesen wird er
ohne die Einstellungen der verbundenen Datenbank: eine Datenbank mit
`ALTER DATABASE … SET idle_in_transaction_session_timeout = '90s'` ließ diese
Spalte 90s als Wert des Clusters melden - der Cluster gibt dort nichts vor.

**Bereich** wählt, wo eine Änderung landet. Bleibt es bei *jede Datenbank*, ist
es `ALTER ROLE … SET`; wählen Sie eine Datenbank, ist es
`ALTER ROLE … IN DATABASE … SET` - die engere Einstellung, und dort gewinnt
sie. Die Liste markiert die Datenbanken, die für diese Rolle bereits etwas
tragen, damit zwanzig Datenbanken mit zwei Überschreibungen nicht zwanzigmal
Aufklappen bedeuten.

Die übrigen Attribute einer Rolle - LOGIN, INHERIT, SUPERUSER, CREATEDB, das
Verbindungslimit, der Ablauf - sind **nicht** pro Datenbank und haben keine
Bereichsauswahl. Sie liegen in einem Katalog, den der ganze Cluster teilt, also
wirkt `ALTER ROLE` aus jeder Datenbank gleich. Privilegien auf Tabellen,
Schemas und Funktionen sind das Gegenteil: die liegen in den Katalogen jeder
einzelnen Datenbank, und der Privilegien-Editor wendet sie in der Datenbank an,
die oben im Fenster steht.

Das sind `USERSET`-Einstellungen: Die Rolle kann sie für ihre eigene Sitzung
mit einem einfachen `SET` anheben oder abschalten. Sie stoppen ein Versehen,
nicht jemanden mit Absicht. Die einzige Grenze, die eine Rolle sich selbst
nicht nehmen kann, ist das Verbindungslimit darüber - gut zu wissen, wenn Sie
zwischen beiden wählen.

#### Was eine Mitgliedschaft gewährt

Eine Rolle unter **Mitglied von** anzuhaken gibt `GRANT parent TO diese_rolle`
aus. Daneben stehen die Optionen, die entscheiden, was die Mitgliedschaft
tatsächlich hergibt:

| | |
|---|---|
| **ADMIN** | das Mitglied darf diese Rolle weitergeben und wieder entziehen. Standardmäßig aus; ohne sie ist die Mitgliedschaft eine Sackgasse |
| **INHERIT** | das Mitglied nutzt die Privilegien der Rolle automatisch, ohne `SET ROLE`. PostgreSQL 16+ |
| **SET** | das Mitglied darf überhaupt `SET ROLE` darauf ausführen. PostgreSQL 16+, standardmäßig an |

Die letzten beiden kamen mit PostgreSQL 16, das eine Mitgliedschaft in drei
getrennte Antworten aufgeteilt hat; auf älteren Servern wird nur ADMIN
angeboten, weil es nur ADMIN gibt. Jedes Kästchen zeigt, was die Mitgliedschaft
derzeit trägt, vom Server gelesen - eine anderswo vergebene Mitgliedschaft
erscheint also so, wie sie wirklich ist, und nicht als Vorgabe.

INHERIT aus bei SET an ist die Form, die man kennen sollte: das Mitglied hält
die Privilegien nur, solange es bewusst die Rolle angenommen hat - so verhindert
man, dass Objekte versehentlich unter dem falschen Eigentümer entstehen. INHERIT
an bei SET aus ist das Gegenteil: die Privilegien gelten immer, und die Rolle zu
werden ist untersagt.

Eine Option an einer bestehenden Mitgliedschaft zu ändern gibt erneut `GRANT`
aus. Ein `ALTER` dafür gibt es nicht, und die SQL-Vorschau zeigt genau das.

Jedes Kästchen in diesem Dialog und in Rolle anlegen erklärt sich beim
Überfahren.

#### Der Rechteeditor

Erreichbar über das Kontextmenü einer Rolle oder über `Rechte` im Rolleneditor.
Er gilt für eine Rolle und ein Schema zugleich; die Schemaauswahl steht oben.

Die Kästchen zeigen, **was die Rolle bereits hat**, und haben drei Zustände statt
zwei, weil jedes alle Objekte des Schemas auf einmal abdeckt:

| Zustand | Bedeutung |
|---|---|
| Angehakt | jedes Objekt im Schema hat dieses Recht |
| Ein Strich | manche ja, manche nein - die Zahl steht daneben, z. B. `[3/12]` |
| Leer | keines hat es |

Ein Strich ist keine Entscheidung. Lassen Sie ihn stehen und es wird dafür nichts
gesendet. Ein Klick macht daraus ein Häkchen - ein `GRANT … ON ALL …`, das den
Rest nachzieht. Noch ein Klick leert es - ein `REVOKE`, das es den wenigen nimmt,
die es hatten.

Das SQL-Feld unten zeigt genau, was „Anwenden" ausführen wird, und sagt „keine
Änderungen", bis Sie wirklich etwas ändern. Den Editor zu öffnen und Anwenden zu
drücken tut nichts.

Ausgegraute Rechte sind solche, die der **verbundene Benutzer** nicht gewähren
kann; der Tooltip sagt das.

Eine Schaltfläche `Aktionen` neben Anwenden trägt alles aus dem Kontextmenü der
Rolle - Prüfung, `search_path`, Ähnliche anlegen, Rechte kopieren, DDL
exportieren, Vergleich, Löschen - , damit Sie den Editor dafür nicht schließen
müssen. Alles, was die Rolle verändert, schließt den Editor zuerst, damit kein
veralteter Stand darüber angewendet wird.

#### Eine Warnung dazu, was „hat" bedeutet

Diese Ansichten zeigen **effektive** Rechte: was die Rolle tatsächlich darf,
einschließlich dessen, was sie über Gruppenmitgliedschaft erbt und was sie als
Superuser bekommt. Ein Superuser zeigt daher alles angehakt, ob eine einzige
ausdrückliche Gewährung existiert oder nicht. Für ausschließlich direkte
Gewährungen nutzen Sie die Rechteansicht je Objekt in der Schema-Ansicht.

#### GRANTs exportieren

**GRANTs exportieren** schreibt Rechte als Skript, das Sie in git ablegen oder
anderswo ausführen können. Es gibt es an drei Stellen:

- im Rechtefenster eines Objekts: dieses Objekt und die Rechte auf seinen
  Spalten (auch ein Spaltenrecht auf einer partitionierten Tabelle);
- im Rechtefenster eines Schemas: das Schema und jede Tabelle, Sicht,
  Sequenz, Spalte, Routine und jeder Typ darin, mit den für es gesetzten
  Standardrechten `IN SCHEMA`;
- im Rechteaudit einer Rolle: alles, was der Rolle gewährt wurde, in den
  Datenbanken, die Sie ankreuzen (die Datenbank der Verbindung ist
  angekreuzt, *Alle Datenbanken* liest jede), dazu ihre Rechte auf
  Datenbanken, Tablespaces und Serverparameter. Mit mehr als einer Datenbank
  beginnt jeder Teil mit `\connect`; führen Sie das Skript dann mit psql aus.

Das Skript macht aus Objekten, die dieselben Eigentümer neu angelegt haben,
die exportierten. Die eigenen Rechte eines Eigentümers werden deshalb nicht
geschrieben, und ein entzogenes Recht - `EXECUTE` für `PUBLIC`, das eigene
`TRUNCATE` eines Eigentümers, ein mit `ALTER DEFAULT PRIVILEGES ... REVOKE`
entferntes Standardrecht - kommt als `REVOKE` heraus. Was eine Rolle über die
Mitgliedschaft in einer anderen Rolle erreicht, steht nicht in ihrem Export;
das Skript nennt diese Rollen.

### 5.8 Ein Schema umbenennen {#renaming-a-schema}

Rechtsklick auf ein Schema im Browser → **Schema umbenennen**.

Das Umbenennen selbst ist ein `ALTER SCHEMA ... RENAME TO ...`, und PostgreSQL
nimmt fast alles mit: Tabellen, Views, Sequenzen, Indizes, Constraints,
Fremdschlüssel und Rechte folgen dem Schema ohne weiteres Zutun. Der Name wird
vor dem Versuch geprüft, ein bereits vergebener Name wird also als Antwort
gemeldet und nicht als Fehler.

Was **nicht** von allein folgt, ist jede als Text geschriebene Referenz in einer
Routine - ein Funktionsrumpf mit `SELECT * FROM alter_name.t` oder ein auf eine
Funktion gesetzter `search_path`. Diese listet die Anwendung nach dem Umbenennen
unter *„Routinen referenzieren weiterhin ..."* auf, mit Namen der Routine und
Fundort (Rumpf, `search_path` oder beides). Nichts wird für Sie umgeschrieben:
ein Funktionsrumpf ist Code, und fremden Code stillschweigend zu ändern ist
schlimmer, als zu sagen, welche vier Funktionen anzusehen sind.

Existiert der Zielname bereits, ist Umbenennen nicht das Richtige - siehe unten.

### 5.9 Zwei Schemas zusammenführen {#merging-two-schemas}

Rechtsklick auf ein Schema im Browser → **Schema zusammenführen**.

`ALTER SCHEMA ... RENAME` kann ein Schema nicht in ein anderes falten, weil der
Zielname schon vergeben ist. Zusammenführen geschieht daher je Objekt, und jedes
Objekt braucht eine eigene Entscheidung, wenn derselbe Name auf beiden Seiten
existiert. DBApguin klärt das, bevor irgendetwas geschieht.

#### Der Plan

Ein Ziel wählen und **Analysieren** drücken startet eine Vorabprüfung, die nichts
ändert und jede Tabelle, View, materialisierte View und Sequenz der Quelle
einordnet:

| Im Ziel | Was es bedeutet | Wählbare Aktionen |
|---|---|---|
| `nicht vorhanden` | nur die Quelle hat es | wird verschoben - nichts zu entscheiden |
| `gleiche Spalten` | beide haben es, Spaltennamen und Typen passen | **Zeilen zusammenführen**, **Ersetzen**, **Überspringen** |
| `Spalten weichen ab` | beide haben es, die Formen passen nicht | **Ersetzen**, **Überspringen** |
| `geht mit seiner Tabelle` | eine `serial`- oder Identity-Sequenz | keine - sie folgt ihrer Tabelle |

Objekte, die es nur im *Ziel* gibt, stehen darunter und werden nie angefasst.

Die Aktionsspalte gilt je Objekt. **Alle setzen** ist eine Hilfe, die eine Wahl
überall dort anwendet, wo sie zulässig ist, und den Rest in Ruhe lässt - sie
schaltet *Zeilen zusammenführen* nicht für eine Tabelle mit abweichenden Spalten
scharf. Eine Aktion, die die Vorabprüfung nicht angeboten hat, weist der Server
zurück, statt sie stillschweigend abzuschwächen; was der Dialog zeigte, ist also
das, was läuft.

#### Was die Aktionen tun

- **Verschieben** - `ALTER ... SET SCHEMA`. Das Objekt selbst wechselt das
  Schema. Rechte, Kommentare, Indizes und Constraints gehen mit, und **aus der
  Quelle ist es weg**.
- **Kopieren** - die Quelle behält alles. Die Tabelle wird im Ziel mit
  `LIKE ... INCLUDING ALL` neu angelegt (Standardwerte, Constraints, Indizes,
  Kommentare, Speicheroptionen) und ihre Zeilen eingefügt; eine Sequenz entsteht
  mit demselben Wert neu. Nur für Tabellen und Sequenzen angeboten: die
  Definition einer View nennt das Schema, aus dem sie liest, eine kopierte View
  läse also still weiter die Daten der Quelle - schlimmer, als die Wahl nicht
  anzubieten.
- **Ersetzen** - das Objekt im Ziel wird mit `CASCADE` gelöscht, das der Quelle
  nimmt seinen Platz. Das zerstört die Daten des Ziels *und* dessen Rechte an
  diesem Objekt. Es ist die einzige wirklich zerstörerische Wahl im Dialog,
  darum wird die Schaltfläche rot und zählt, was gelöscht wird.
- **Zeilen zusammenführen** - die Zieltabelle bleibt genau, wo sie ist, und die
  Zeilen der Quelle werden hineingefügt. Die Quelltabelle wird weder geleert noch
  gelöscht: ihre Zeilen sind danach noch da, das Zusammenführen lässt sich also
  wiederholen oder prüfen, bevor Sie aufräumen.

Die Quelle überlebt also **Kopieren** und **Zeilen zusammenführen**, nicht aber
**Verschieben** oder **Ersetzen**. Der Plan sagt je Objekt vorher, was geschieht.
- **Überspringen** - das Objekt im Ziel bleibt unangetastet.

#### Wenn Zeilen kollidieren

Betrifft nur *Zeilen zusammenführen* und entscheidet sich am Primärschlüssel der
Zieltabelle:

- **Zeile des Ziels behalten** - `ON CONFLICT DO NOTHING`. Vorhandene Zeilen
  gewinnen; neue kommen hinzu.
- **Zeile der Quelle gewinnen lassen** - ein Upsert. Kollidierende Zeilen werden
  mit den Werten der Quelle überschrieben.
- **Zusammenführen abbrechen** - ein einfaches `INSERT`. Die erste Kollision
  bricht ab, und da das Ganze eine Transaktion ist, wird überhaupt nichts
  angewendet.

Hat die Zieltabelle **keinen Primärschlüssel**, gibt es nichts, woran eine
Kollision erkennbar wäre, und jede Quellzeile wird angehängt - Dubletten
eingeschlossen. Der Plan sagt das an der Zeile selbst, vor dem Lauf, statt Sie
die doppelten Zeilen hinterher finden zu lassen.

#### Was man wissen sollte

- **Das ganze Zusammenführen ist eine Transaktion.** Scheitert etwas, wird nichts
  angewendet. Ein auf halbem Weg abgebrochener Lauf hinterließe zwei halb
  gefüllte Schemas und keinen Vermerk darüber, wo er stehen blieb - schlimmer,
  als gar nicht begonnen zu haben.
- **Eine `serial`- oder Identity-Sequenz geht mit ihrer Tabelle**, weil
  PostgreSQL sie als Teil von `ALTER TABLE ... SET SCHEMA` verschiebt. Sie steht
  im Plan, damit Sie es sehen, hat aber keine eigene Aktion. Eigenständige
  Sequenzen sind gewöhnliche Objekte und bekommen eine Wahl.
- **Funktionen, Prozeduren und Typen werden nicht verschoben.** Sie bleiben im
  Quellschema. Zusammenführen betrifft Daten und die Objekte, die sie halten; der
  Rumpf einer Routine kann jedes Schema beim Namen nennen, und sie zu verschieben
  ist eine Entscheidung, die nur Sie treffen können.
- **Fest verdrahtete Referenzen werden vor dem Lauf markiert.** Views,
  Fremdschlüssel und CHECK-Constraints werden bei jedem Lesen aus dem Katalog
  aufgebaut und folgen einer verschobenen Tabelle daher gratis. Der Rumpf
  einer PL/pgSQL-Funktion oder eines Triggers ist reiner Text, der beim
  Aufruf über den Namen aufgelöst wird: ein Rumpf mit `public.orders` zeigt
  weiter aufs alte Schema, nachdem die Tabelle zu `archive.orders` geworden
  ist, und scheitert beim nächsten Aufruf statt beim Zusammenführen. Der
  Vorabcheck durchsucht genau dafür `pg_proc` und `pg_policies` und listet,
  was brechen würde - Sie reparieren diese Rümpfe zuerst oder entscheiden,
  dass die Referenz in Ordnung ist. RLS-Policies sind wie ein View an die OID
  gebunden und werden nur markiert, wenn sie eine bereits erfasste Funktion
  aufrufen.
- **Fremdschlüssel zeigen weiter dorthin, wohin sie zeigten.** Eine verschobene
  Tabelle, deren Fremdschlüssel auf eine zurückgebliebene Tabelle zeigte, endet
  mit einem schemaübergreifenden Fremdschlüssel, der weiterhin greift. Das ist
  gültig und meist ein Zeichen dafür, dass die referenzierte Tabelle mitgehen
  wollte.
- **Quellschema löschen, wenn es leer bleibt** tut genau das und nur das: wurde
  etwas übersprungen, bleibt das Schema, und der Dialog sagt, wie viele Objekte
  noch darin sind.

### 5.10 Backup und Zurückspielen {#backup-and-restore}

Rechtsklick auf eine Datenbank → **Backup** oder **Zurückspielen**, oder auf ein
Schema → **Schema exportieren (pg_dump)**.

Beides ruft die echten Programme `pg_dump` / `pg_restore` auf. Ihr Ort kommt aus
Einstellungen → Werkzeuge, sofern gesetzt, sonst aus der automatischen Suche;
eine Versionsabweichung zwischen Werkzeug und Server wird gemeldet statt geraten.

#### Das Ziel wird zuerst geprüft

Ein Dump, der nach zwanzig Minuten scheitert, weil der Ordner schreibgeschützt
war, sind zwanzig verlorene Minuten. Der Pfad wird deshalb geprüft, bevor
`pg_dump` startet:

- ein Verzeichnis, wo eine Datei erwartet wurde, wird abgelehnt
- ein Ordner ohne Schreibrecht wird abgelehnt, unter Nennung des Ordners
- eine vorhandene Datei wird gemeldet statt stillschweigend überschrieben

Mit **Durchsuchen** wählen Sie einen Ordner, statt einen Pfad zu tippen. Es
zeigt nur Verzeichnisse, markiert die beschreibbaren und blendet versteckte
Ordner aus. Es durchsucht das Dateisystem des **Servers**, nicht dieses Rechners
 - pg_dump läuft auf dem Server, dort muss der Pfad existieren. Datenbank-Backup
und Schema-Export nutzen denselben Browser.

#### Den ganzen Cluster sichern {#backing-up-the-whole-cluster}

Rechtsklick auf eine Datenbank → **Cluster sichern (pg_dumpall)…**

`pg_dump` arbeitet innerhalb einer Datenbank und kann strukturell nicht
erreichen, was außerhalb liegt. **Rollen, Tablespaces und Einstellungen je
Datenbank gelten clusterweit**, ein Backup nur mit `pg_dump` stellt also Tabellen
wieder her, die niemand besitzen kann - was man beim Zurückspielen entdeckt,
nicht davor.

| Inhalt | Was Sie bekommen |
|---|---|
| Nur globale Objekte *(empfohlen)* | Rollen und Tablespaces - die Ergänzung zu Ihren Dumps je Datenbank |
| Nur Rollen | nur `CREATE ROLE` |
| Nur Tablespaces | nur `CREATE TABLESPACE` |
| Alles | globale Objekte plus jede Datenbank des Clusters, in einer Datei |

*Nur globale Objekte* ist meist das Gewünschte: weiterhin Dumps je Datenbank im
Custom-Format nehmen und dieses daneben. *Alles* ergibt eine große
Plain-SQL-Datei und ist selektiv langsamer zurückzuspielen.

`pg_dumpall` hat **keine Formatwahl** - die Ausgabe ist immer Plain SQL und geht
mit `psql` zurück, nicht mit `pg_restore`. Der Dialog sagt das nach dem Dump,
statt es Sie herausfinden zu lassen.

#### pg_dump-Optionen {#pg-dump-options}

Öffnen Sie **Optionen** im Backup-Dialog. Jede Bedienelement entspricht einer
`pg_dump`-Option, und die Option steht am Bedienelement, damit sich Ihre Wahl in
der PostgreSQL-Dokumentation nachschlagen lässt:

- **Format** - custom (`-Fc`, das, was `pg_restore` will), Plain SQL, Verzeichnis oder tar
- **Kompression** - 0-9; siehe unten
- **Abschnitt** - welches Drittel des Dumps geschrieben wird; siehe unten
- **Nur diese Schemas / Nur Tabellen / Tabellen ausschließen / Ohne Tabellendaten / Schemas ausschließen** - Namen durch Komma getrennt, siehe *Namen und Partitionen* unten. **Auswählen…** neben *Nur diese Schemas* listet die Schemas der Datenbank zum Ankreuzen
- **Parallele Jobs** - nur beim Verzeichnisformat. Für jedes andere Format ist das Feld gesperrt, mit der Begründung daneben, statt Sie es ausfüllen und danach scheitern zu lassen
- `--no-owner`, `--no-privileges`, `--clean`, `--create`, `--inserts`, `--no-comments`, `--no-tablespaces`
- **Erweitert** - `--load-via-partition-root`, `--no-publications`, `--no-subscriptions`, `--no-security-labels`, `--enable-row-security`, Large Objects ein- oder ausschließen, eine Sperrwartezeit, *Nur Erweiterungen* (`--extension`) und *Als Rolle ausführen* (`--role`)

`--no-owner`, `--clean` (mit `--if-exists`), `--create` und `--no-tablespaces`
ändern nur eine **Plain-SQL**-Datei. Bei einem Custom-, Tar- oder
Directory-Archiv ignoriert `pg_dump` sie kommentarlos - sie werden beim
Zurückspielen des Archivs gewählt -, deshalb sind sie für diese Formate
gesperrt, mit der Begründung unter den Optionen, und die Zusammenfassung
verspricht sie nicht.

Welches `pg_dump` läuft, hängt vom Server ab (der Client wird passend gewählt),
und eine Option, die dieser Client nicht hat, ist mit der nötigen Version
gesperrt - `--extension` braucht zum Beispiel 14. Die Zeile unter *Erweitert*
nennt den Client.

**Namen und Partitionen.** Schreiben Sie Namen so, wie der Katalog sie zeigt,
mit Groß- und Kleinschreibung: `Sales.Orders` findet die Tabelle, die der Baum
als `Sales.Orders` zeigt. Ein Name mit Punkt oder Komma gehört in doppelte
Anführungszeichen (`"order.lines"`). Jeder Name wird vor dem Start des Dumps
nachgeschlagen: einer, der nichts trifft - ein Tippfehler, eine Tabelle in einem
anderen Schema -, stoppt ihn mit diesem Namen und, wenn sich nur die
Schreibweise unterscheidet, mit dem Namen, den es gibt. Früher fiel ein solcher
Name stillschweigend aus einem Dump heraus, der dann vollständig aussah.

Eine partitionierte Tabelle besteht aus ihren Partitionen: der Elternteil allein
hat keine Zeilen. Eine Tabelle in *Nur Tabellen* kommt deshalb **mit ihren
Partitionen und Kindtabellen, auf jeder Ebene** (das Kästchen unter dem Feld,
standardmäßig gesetzt), und das gilt ebenso für eine Tabelle in *Tabellen
ausschließen* oder *Ohne Tabellendaten*. Mit `pg_dump` 16 oder neuer ist das
`--table-and-children`; ein älterer Client bekommt jede Partition einzeln
genannt. Ist der Dump geschrieben, bestätigt seine Prüfung, dass jede gewünschte
Tabelle samt Partitionen in der Datei steht. Was `pg_dump --table` auslässt,
wird beim Start des Dumps gesagt: was die gewählten Tabellen verwenden, ohne
dass es ihnen gehört - die Funktion eines Triggers, eine eigenständige Sequenz
hinter einem Default, der Typ einer Spalte, eine Tabelle, auf die ein
Fremdschlüssel zeigt, eine Extension -, was die Zieldatenbank des Restores
schon haben muss, und was sie verwendet (eine Sicht, der Fremdschlüssel einer
anderen Tabelle).

Nichts wird als freier Text durchgereicht. Eine Option, die DBApguin nicht kennt,
wird nicht gesendet, und genau das verhindert, dass ein verirrtes `--irgendwas`
in der Befehlszeile landet.

**Der Export eines Schemas nutzt dieselben Optionen.** Rechtsklick auf ein Schema
→ *Schema exportieren (pg_dump)* öffnet denselben Satz, ohne *Schemas
ausschließen* - was an einem bereits auf ein Schema beschränkten Dump ohnehin
nichts ändert. Beide Dialoge entstehen aus einer Definition, eine hinzugefügte
Option erscheint also in beiden.

Eines nimmt Ihnen der Schema-Export ab: `pg_dump` vergleicht `--table`-Muster mit
*schemaqualifizierten* Namen, `orders` in einem Dump eines einzelnen Schemas
trifft also nichts, und das Werkzeug hält mit *„keine passenden Tabellen
gefunden"* an. Ohne Schema getippte Namen werden mit dem exportierten Schema
qualifiziert, `orders` heißt also `sales.orders` - und das Schema steht in
Anführungszeichen, damit eines mit Großbuchstaben unterwegs nicht klein wird.

Der vollständige Zielpfad steht in beiden Dialogen unter dem Pfadfeld. Ein
Textfeld zeigt das Ende eines langen Pfades und verbirgt den Anfang - genau die
Hälfte, die Rechner und Verzeichnis nennt.

#### Abschnitt: die drei Teile {#section-what-the-three-parts-are}

Ein Dump wird in drei Durchgängen geschrieben, und `pg_dump` lässt Sie einen
davon allein nehmen. **alle** ist der Standard und das Richtige, sofern Sie
keinen bestimmten Grund haben.

| Abschnitt | Was hineinkommt |
|---|---|
| `pre-data` | `CREATE SCHEMA`, `CREATE TABLE`, `CREATE VIEW`, `CREATE SEQUENCE`, Eigentümer - alles, was eine Tabelle zum Dasein braucht, aber nichts, was das Laden bremst |
| `data` | die Zeilen selbst, als `COPY`-Blöcke, plus `setval`-Aufrufe, die Sequenzen auf ihrem Stand lassen |
| `post-data` | Primärschlüssel, Unique- und Fremdschlüssel-Constraints, Indizes, Trigger - alles, was Zeilen prüft oder indiziert |

Die Aufteilung ist nicht willkürlich: Constraints und Indizes kommen zuletzt,
**weil das Laden in eine Tabelle, die sie schon hat, weit langsamer ist** und
jede Zeile einzeln geprüft und indiziert würde. Erst pre-data, dann data, dann
post-data baut die Indizes einmal, am Ende, über die fertige Tabelle.

Einen Abschnitt allein zu nehmen ist etwas für Sonderfälle:

- `pre-data` - die leere Form einer Datenbank, um Strukturen zu vergleichen oder
  ein Ziel zu bauen, das Sie anders füllen
- `data` - Zeilen in eine bestehende Struktur nachziehen
- `post-data` - Constraints und Indizes nach einem reinen Datenladen neu anlegen

Einen `data`-Dump in eine Datenbank ohne Tabellen zurückzuspielen scheitert,
ebenso `post-data` ohne die Tabellen, an denen es hängen soll. Die Abschnitte
sind Teile eines Dumps, nicht drei unabhängige Backups.

#### Kompression: was die Zahlen kosten {#compression-what-the-numbers-cost}

Kompression gilt für Custom-, Verzeichnis- und Plain-Format. **Tar lässt sich gar
nicht komprimieren** - das Feld ist dann gesperrt.

Gemessen an einer Tabelle mit 120.000 Zeilen Hashes (eine realistische Mischung,
keine Datei aus wiederholtem Text):

| Stufe | Größe | Zeit |
|---|---|---|
| 0 - keine | 13,4 MB | 0,3 s |
| 1 | 7,5 MB | 0,8 s |
| 5 | 7,35 MB | 1,5 s |
| 9 - kleinste | 7,35 MB | 1,7 s |
| Standard | 7,35 MB | 1,6 s |

Die Form dieser Tabelle ist der ganze Punkt: **fast die gesamte Ersparnis liegt
zwischen 0 und 1.** Stufe 1 bringt 44 % weniger für eine halbe Sekunde; der Weg
von 1 auf 9 kauft weitere 2 % und dauert doppelt so lang. Stufe 9 ist nicht „viel
kleiner", sondern „gleich groß, langsamer".

Also:

- lassen Sie es auf **Standard**, wenn Sie keinen Grund haben - das ist schon
  nahe an der besten Größe
- wählen Sie **0**, wenn der Dump irgendwohin geht, wo ohnehin komprimiert wird,
  oder auf eine schnelle Platte, wo Zeit mehr zählt als Platz
- wählen Sie **9** nur, wenn die Datei über etwas Langsames muss und Ihnen die
  Dauer des Dumps gleich ist

Custom- und Verzeichnisformat komprimieren standardmäßig; ein Plain-SQL-Dump ist
unkomprimiert, sofern Sie nichts anderes verlangen - und verlangen heißt, eine
gzip-Datei statt lesbaren SQLs zu bekommen.

#### Ein Schema importieren {#importing-one-schema}

Rechtsklick auf eine Datenbank → **Schema importieren (pg_restore)…**. Das ist
der Zurückspielen-Dialog mit dem Schemafilter davor: nennen Sie das Schema so,
wie es im Dump heißt, und nur dieses wird in diese Datenbank gespielt. Leer
lassen heißt den ganzen Dump zurückspielen, also gewöhnliches Zurückspielen.

Das geht auch mit dem Dump einer ganzen Datenbank - Sie brauchen keinen Dump nur
eines Schemas, um eines herauszuholen.

Eines nimmt DBApguin Ihnen hier ab: `pg_restore --schema sales` spielt die
Objekte *innerhalb* von `sales` zurück, aber `CREATE SCHEMA sales` gehört nicht
dazu. In einer Datenbank ohne dieses Schema scheitert jedes einzelne Objekt mit
*„schema sales does not exist"* - und `pg_restore` endet dennoch mit „errors
ignored on restore". Fehlt das Schema in der Datenbank, wird es deshalb zuerst
aus dem Dump zurückgespielt - mit Eigentümer, Rechten und Kommentar, wie der Dump
sie hat. Ein vorhandenes Schema bleibt genau so, wie es ist.

#### Unter einem anderen Schemanamen importieren

PostgreSQL hat kein Gegenstück zu Oracles `REMAP_SCHEMA`. Weder `pg_dump` noch
`pg_restore` können ein Schema beim Einspielen umbenennen - die Option gibt es in
keinem der beiden.

Was es gibt, ist `ALTER SCHEMA … RENAME`, und das benennt richtig um:
Constraints, Sequenzen, Indizes und Rechte folgen. **Als Schema zurückspielen**
im Importdialog spielt daher unter dem Namen des Dumps zurück und benennt danach
um. Gleiches Ergebnis, auf einem Weg, den der Server wirklich unterstützt.

Ist der gewünschte Name schon vergeben, wird *nicht* erzwungen umbenannt - zwei
Schemas lassen sich nicht dadurch falten, dass man eines über das andere
umbenennt. Der Import sagt das und verweist auf **Schema zusammenführen** (5.9),
das genau dafür da ist.

#### Was ein Zurückspielen kaputt lassen kann

`pg_restore` meldet Probleme und macht weiter. Der häufige Fall ist ein auf ein
Schema begrenzter Import, dessen Fremdschlüssel auf eine Tabelle in einem anderen
Schema zeigt: das Constraint scheitert, die Tabelle wird **ohne es**
zurückgespielt, und `pg_restore` endet nur mit *„errors ignored on restore"*.
Danach hält nichts in der Datenbank fest, dass das Constraint hätte da sein
sollen.

Es gibt auch keine Liste ungültiger Objekte zum Nachsehen. **PostgreSQL kennt
keinen INVALID-Zustand wie Oracle** - `pg_proc` hat keine Gültigkeitsspalte, und
eine Funktion, die eine fehlende Tabelle referenziert, wird ohne Widerspruch
angelegt und scheitert erst beim Aufruf. Views können gar nicht ungültig werden,
weil der Server sich weigert zu löschen, wovon eine View abhängt.

Was der Katalog *doch* festhält, und was der Import zum Schluss für Sie prüft:

| Gemeldet | Woher es kommt |
|---|---|
| als `NOT VALID` hinzugefügte Constraints | `pg_constraint.convalidated` - vorhandene Zeilen wurden nie geprüft |
| ungültige Indizes | `pg_index.indisvalid` - ein nicht beendetes `CREATE INDEX CONCURRENTLY` |
| deaktivierte Trigger | `pg_trigger.tgenabled` - nach einem abgebrochenen `--disable-triggers`-Lauf aus geblieben |
| Fremdschlüssel, die aus dem Schema zeigen | die, die ein schemabegrenztes Zurückspielen nicht mitnehmen kann |

Zu jedem Befund gehört die Anweisung, die ihn behebt. Gibt es nichts zu melden,
sagt die Anwendung das - Stille ist hier ein Ergebnis und keine ausgebliebene
Antwort. Die genauen gescheiterten Anweisungen bleiben im Job-Protokoll,
erreichbar über **Hintergrundaufträge**.

#### Eine neue Datenbank ist das Standardziel {#restore-into-a-new-database}

Der Wiederherstellungsdialog öffnet mit **(eine neue Datenbank)**, benannt
nach der, aus der er geöffnet wurde, mit angehängtem Datum; die Datenbank
wird leer angelegt und der Dump hineingespielt, sodass nichts Bestehendes
überschrieben wird - unter jedem Namen, den Sie ihr geben, aus jedem Format.
Die Datenbank, aus der der Dump stammt, kann bleiben, wo sie ist, im selben
Cluster.

Das gilt auch für eine SQL-Textdatei, die mit `--create` erstellt wurde, was
nicht immer so war. Eine solche Datei legt ihre Datenbank an und wechselt in
sie (`CREATE DATABASE shop`, `\connect shop`), und psql folgt ihr, nicht der
Datenbank, in der es gestartet wurde: unverändert eingespielt kam nichts in
die neue Datenbank, und eine mit `--create --clean` erstellte Datei löschte
zuerst die Datenbank, aus der sie kam, und baute sie aus der Datei neu auf -
gemessen: die nach dem Dump hinzugekommenen Zeilen waren weg. Der Dialog
liest die Datei jetzt vor der Wiederherstellung und sagt, wenn sie das tut;
die Wiederherstellung läuft dann aus einer Kopie ohne die Zeilen
`DROP DATABASE`, `CREATE DATABASE` und `\connect`, und die Anweisungen zur
Datenbank selbst (ihre Einstellungen, ihr Kommentar und ihre Rechte) zeigen
auf die gewählte. Ihre Datei wird nicht verändert. Ein Custom-, Tar- oder
Verzeichnisarchiv hatte das Problem nie: `pg_restore` bekommt kein
`--create` und schreibt in die Datenbank, auf die es zeigt.

Eine Datei eines ganzen Clusters (`pg_dumpall`) enthält mehrere Datenbanken,
Rollen und Tablespaces und passt nicht in eine Datenbank; sie wird mit
dieser Begründung abgelehnt. Spielen Sie sie mit psql in einen Cluster ein,
der keine ihrer Datenbanken hat, oder erstellen Sie einen Dump der einen
Datenbank, die Sie brauchen.

#### In eine bestehende Datenbank zurückspielen {#restore-into-an-existing-database}

Eine bestehende Datenbank lässt sich weiterhin aus der Liste wählen; die
Bestätigung sagt dann, was sie enthält - wie viele Tabellen und Sichten, wie
groß - bevor etwas geschrieben wird. Was dort passiert, ist kein Ersetzen:

- **Ohne --clean** wird in der Datenbank nichts ersetzt. Ein vorhandenes
  Objekt bleibt, wie es ist - sein Anlegen schlägt fehl, und das wird
  gemeldet - und die Zeilen des Dumps werden zu den vorhandenen Tabellen
  **hinzugefügt**. Eine Tabelle mit einem Primär- oder eindeutigen
  Schlüssel, mit dem die Zeilen des Dumps kollidieren, lädt keine davon;
  eine Tabelle ohne Schlüssel bekommt sie ein zweites Mal. Gemessen: eine
  Tabelle ohne Schlüssel ging von 3 auf 5 Zeilen.
- **Mit --clean** werden die Objekte des Dumps zuerst gelöscht und neu
  angelegt, ihre jetzigen Zeilen also durch die des Dumps ersetzt. Objekte,
  die der Dump nicht enthält, bleiben unberührt.

#### Tablespaces beim Zurückspielen {#restore-tablespaces}

Ein Dump merkt sich den Tablespace jedes Objekts, das nicht im Standard
seiner Datenbank lag, und legt es wieder dorthin. Der Dialog nennt die
Tablespaces, die die Datei verwendet, und welche davon dem Server fehlen.
Ein fehlender wird nicht stillschweigend übergangen: `pg_restore` würde
diese Objekte mit einem Fehler pro Objekt in den Standard-Tablespace legen,
und psql würde beim ersten anhalten. Stattdessen wird die Wiederherstellung
mit den Namen abgelehnt, und ein Klick setzt **--no-tablespaces**.

**Tablespace der neuen Datenbank** legt die neue Datenbank im gewählten
Tablespace an. Mit **--no-tablespaces** kommt auch jedes Objekt des Dumps
dorthin - jede Datenbank in ihrem eigenen Tablespace, egal was die Quelle
verwendet hat. Ohne behalten die Objekte die Tablespaces aus dem Dump, und
nur der Rest kommt in den der neuen Datenbank.

Es kann auch **ein neuer Tablespace** sein: geben Sie ihm einen Namen und ein Verzeichnis auf dem Datenbankserver - das schon existieren, leer sein und dem Benutzer gehören muss, unter dem PostgreSQL läuft - dann wird zuerst der Tablespace angelegt und darin die Datenbank. Einen Tablespace anlegen darf nur ein Superuser.

#### Unter einem anderen Namen zurückspielen {#restoring-into-a-different-name}

`pg_restore` kann ein Schema beim Einspielen nicht umbenennen - die Namen im Dump
sind die Namen, die Sie bekommen. DBApguin tut nicht so, als wäre es anders. Zwei
Wege funktionieren:

1. **Zurückspielen, dann umbenennen.** Den Dump so einspielen, wie er ist, dann
   Rechtsklick auf das Schema → **Schema umbenennen** (5.8). Constraints,
   Sequenzen und Rechte folgen von selbst.
2. **In ein Zwischenschema spielen, dann zusammenführen.** Ist der gewünschte
   Name bereits belegt und enthält Daten, die bleiben sollen, spielen Sie in ein
   Zwischenschema und falten es mit **Schema zusammenführen** (5.9) in das
   produktive, mit einer Entscheidung je Objekt und einer Wahl für kollidierende
   Zeilen.

Weg 2 ist der für eine laufende Datenbank: das Zusammenführen läuft in einer
Transaktion, eine unerwartete Kollision ändert also nichts.

#### Das Format wird aus der Datei gelesen, nicht aus dem Namen {#the-format-is-read-from-the-file-not-its-name}

Lassen Sie **Format** auf *Aus dem Dateiinhalt erkennen*, und DBApguin schaut in
die Datei statt auf ihre Endung:

| Signatur | Format | Zurückgespielt mit |
|---|---|---|
| beginnt mit `PGDMP` | custom | `pg_restore` |
| `ustar` an Position 257 | tar | `pg_restore` |
| ein Verzeichnis | Verzeichnis | `pg_restore` |
| alles andere | Plain SQL | `psql` |

Das zählt, weil die Endung kein Beweis ist. Ein Plain-SQL-Dump als `backup.dump`
gespeichert ist verbreitet, und er landete früher bei `pg_restore`, das
antwortete: *„input file appears to be a text format dump. Please use psql."* - 
richtig, und keine Hilfe für jemanden, der nur seine Datei zurückspielen wollte.
Beide Richtungen funktionieren jetzt: SQL mit der Endung `.dump` und ein
Custom-Archiv mit `.sql`.

Die Auswahl erlaubt weiterhin, das Format ausdrücklich zu nennen, wenn Sie die
Erkennung übergehen wollen.

#### Optionen beim Zurückspielen {#restore-options}

Zurückspielen hat einen eigenen Satz: `--clean`, `--if-exists` (nur mit
`--clean`, darauf besteht `pg_restore`), `--no-owner`, `--no-privileges`,
`--single-transaction`, `--exit-on-error`, `--disable-triggers` (bei reinen
Datenläufen), `--data-only`, `--schema-only`, `--no-comments`,
`--no-publications`, `--no-subscriptions`, `--no-security-labels`,
`--enable-row-security`, *Als Rolle ausführen*, parallele Jobs, und - um
einzugrenzen, was aus einem Dump kommt - **Nur diese Schemas**, **Nur
Tabellen**, **Schemas ausschließen** und **Abschnitt**.

`--single-transaction` und parallele Jobs schließen einander aus; beides zu
wählen wird abgelehnt statt weitergereicht.

**Eine Plain-SQL-Datei spielt `psql` genau so ab, wie sie geschrieben wurde**:
Eigentümer, Rechte und alles andere kommen mit, und beim ersten Fehler ist
Schluss. `psql` kann sie als eine Transaktion und als andere Rolle ausführen,
diese beiden werden angewandt; jede andere Option oben braucht einen Custom-,
Tar- oder Directory-Dump, deshalb sind sie bei einer Plain-SQL-Datei gesperrt,
und wer eine davon verlangt, bekommt eine Ablehnung mit ihrem Namen statt eines
stillen Ignorierens.

Dinge über das Zurückspielen eines *Teils*, die wissenswert sind, weil
`pg_restore` nicht symmetrisch zu `pg_dump` ist:

- **Namen werden genau so verglichen, wie sie im Dump stehen**, mit Groß- und
  Kleinschreibung; `sales.orders` und `orders` gehen beide, und ein Name in
  doppelten Anführungszeichen darf Punkt oder Komma enthalten. Einer, der im
  Archiv nichts trifft, wird abgelehnt, bevor etwas läuft.
- **Nur Tabellen spielt die ganze Tabelle zurück.** `pg_restore --table` allein
  bringt die Tabellendefinition und ihre Zeilen und sonst nichts: keine Primär-
  oder Fremdschlüssel, Indizes, Defaults, Trigger, eigenen Sequenzen oder
  Partitionen - eine Tabelle, die im Baum richtig aussieht und sich verhält, als
  gäbe es das alles nicht. Standardmäßig nimmt das Zurückspielen, was
  `pg_dump --table` gesichert hätte: die Partitionen auf jeder Ebene, Schlüssel,
  Indizes, Defaults, Trigger, Policies, eigene und Identity-Sequenzen mit ihren
  Werten, Kommentare und Rechte - und die Funktion, die ein Trigger aufruft, oder
  die Sequenz eines Defaults, wenn diese Datenbank sie nicht hat. *nur die
  Tabelle und ihre Zeilen* gibt es weiterhin, und es sagt, was fehlt.
- **Beim Zurückspielen gibt es kein „Tabellen ausschließen".** `pg_restore` hat
  kein `--exclude-table`. Schließen Sie sie beim Dump aus oder nennen Sie die
  gewünschten. Die Anfrage wird mit dieser Erklärung abgelehnt statt mit einem
  „unrecognized option" des Werkzeugs.
- **Ein ausgeschlossenes Schema wird trotzdem angelegt, leer.**
  `--exclude-schema` überspringt den Inhalt, nicht das `CREATE SCHEMA`. Löschen
  Sie das leere Schema danach, wenn Sie es nicht wollen.

#### Vor und nach dem Zurückspielen {#before-and-after-a-restore}

Sobald die Datei gewählt ist, liest der Dialog, was das Zurückspielen vom Ziel
verlangen wird, und sagt, was dort fehlt:

- **Rollen**, die der Dump zu Eigentümern von Objekten macht oder denen er Rechte
  gibt - mit `--no-owner` / `--no-privileges` spielen sie keine Rolle mehr (nicht
  bei einer Plain-SQL-Datei, die sie trotzdem mitbringt);
- **Erweiterungen**, die der Dump anlegt und dieser Server nicht installieren
  kann;
- **Schemas**, in denen Objekte des Dumps liegen, die er aber nicht anlegt - ein
  mit *Nur Tabellen* erstellter Dump hat kein `CREATE SCHEMA` -, mit dem Angebot,
  sie zuerst leer anzulegen;
- eine andere **Kodierung** (nach UTF8 geht nichts verloren; in jede andere
  stoppt ein Zeichen ohne Code das Laden seiner Tabelle) oder ein anderes
  **Locale** (Text wird in der Reihenfolge dieser Datenbank sortiert und
  indiziert).

Mit `--exit-on-error` oder `--single-transaction`, und bei einer Plain-SQL-Datei
immer, lehnt eine fehlende Rolle, Erweiterung oder ein fehlendes Schema das
Zurückspielen vor dem Start ab: es würde beim ersten davon halb fertig anhalten.

Ist das Zurückspielen fertig, wird das Ziel mit dem Dump selbst verglichen: die
Zeilen jeder Tabelle (in den Daten des Dumps gezählt, nicht geschätzt), der Wert
jeder Sequenz, jeder Eigentümer und jedes Recht, das der Dump gesetzt hätte. Was
abweicht, wird aufgelistet. Der Exit-Code von `pg_restore` sagt nur, ob Fehler
protokolliert wurden; dies sagt, ob die Datenbank enthält, was der Dump enthielt.

#### Ein Schema ohne seine Constraints klonen {#cloning-a-schema-without-its-constraints}

**Constraints → Constraints und Indizes weglassen**, in Backup und im
Schema-Export, schreibt die Tabellen und die Zeilen und hört dort auf: keine
Primärschlüssel, keine Unique- oder Fremdschlüssel, keine Indizes.

Es ist ein `pg_dump`-Lauf - `--section=pre-data --section=data`, was das
Werkzeug zusammen akzeptiert. pre-data enthält die Tabellendefinitionen, data
die Zeilen, und alles, was hier wegfällt, liegt in post-data. An einem
laufenden Server geprüft: die Datei enthält kein einziges `ADD CONSTRAINT` und
kein `CREATE INDEX`.

Zwei Gründe dafür. Ein Massenimport läuft ohne Prüfungen und Indexpflege
schneller, und ein Replikationswerkzeug bei der Erstsynchronisation ins Ziel
startet meist nicht, solange dort Fremdschlüssel durchgesetzt werden. Beides
endet gleich: der Constraint-Manager schreibt das Skript, das sie zurückbringt,
und Sie führen es aus, wenn der Import oder die Synchronisation fertig ist.

Der Haken übersteuert die Auswahl **Section**, statt sich mit ihr zu
kombinieren - zwei Bedienelemente für dieselbe Option ergäben sonst einen Dump,
den keines von beiden beschreibt.

#### Timeouts bei einem langen Dump oder einer langen Wiederherstellung {#timeouts-on-a-long-dump-or-restore}

Ein Dump oder eine Wiederherstellung, die mittendrin stirbt - nach einer halben
Stunde, ohne dass an der Datei etwas falsch wäre - wird meist vom Server
beendet, statt dass das Werkzeug scheitert.

Vier Einstellungen können das:

- `statement_timeout` - eine Anweisung lief zu lange.
- `lock_timeout` - das Warten auf eine Sperre dauerte zu lange.
- `idle_in_transaction_session_timeout` - eine Transaktion lag zu lange untätig.
- `transaction_timeout` (PostgreSQL 17 und neuer) - eine Transaktion ist zu
  lange offen, ob untätig oder nicht.

`pg_dump`, `pg_restore` und ein über `psql` geladener Dump heben die ersten drei
selbst auf: auf einem laufenden Server zeigt dessen Log `SET statement_timeout =
0`, `SET lock_timeout = 0` und `SET idle_in_transaction_session_timeout = 0`,
bevor irgendeine Arbeit beginnt. `transaction_timeout` heben sie nicht auf - und
genau das beendet lange Arbeit, denn `pg_dump` hält für den ganzen Dump eine
einzige REPEATABLE-READ-Transaktion und `pg_restore --single-transaction` eine
für das ganze Laden. Ein Server mit `transaction_timeout = 30min` stoppt beide
nach dreißig Minuten, so gesund sie auch laufen.

Eine einfache `.sql`-Datei, die kein `pg_dump` geschrieben hat, ist die zweite
Lücke: sie trägt keinen `SET`-Kopf, also gelten alle vier für sie.

**Server-Timeouts -> für diesen Lauf aufheben**, in Backup, Schema-Export und
Wiederherstellung, hebt auf, was das Ziel gesetzt hat. Das läuft über
`PGOPTIONS`, das libpq beim Verbinden anwendet und das ein `ALTER DATABASE ...
SET` oder `ALTER ROLE ... SET` übersteuert. `transaction_timeout` wird nur an
PostgreSQL 17 und neuer geschickt: ein unbekannter Parameter in `PGOPTIONS` wird
nicht ignoriert, die Verbindung wird mit `FATAL: unrecognized configuration
parameter` abgelehnt, und das würde aus "Ihre Wiederherstellung könnte
abgeschnitten werden" ein "Ihre Wiederherstellung kann nicht starten" machen.

Die Zeile unter dem Kästchen meldet, was das Ziel tatsächlich gesetzt hat. Ohne
Haken bleiben die Server-Einstellungen unangetastet.

Ist ein Lauf bereits so gestorben, benennt die Meldung die Einstellung:
`canceling statement due to statement timeout`, `terminating connection due to
idle-in-transaction timeout` oder `canceling statement due to transaction
timeout`.

Die Zeile unter dem Kästchen meldet nicht nur den Wert, sondern auch **wo er
gesetzt wurde** - `statement_timeout = 30min (gesetzt auf database sales)` oder
`(gesetzt auf role app_user)`. Ein Wert ohne genannte Quelle stammt aus der
Serverkonfiguration selbst: `postgresql.conf`, oder die Parametergruppe bei
Aurora und RDS. Das ist der Unterschied zwischen "etwas kürzt den Lauf ab" und
"das hier muss geändert werden".

**Speziell bei Aurora und RDS.** `transaction_timeout` gibt es vor PostgreSQL 17
nicht, auf Aurora 15 oder 16 ist ein Lauf, der bei einer runden Minutenzahl
stirbt, also eines davon:

- ein `statement_timeout` an der Parametergruppe, der Datenbank oder der Rolle - 
  die Zeile unter dem Kästchen nennt welche;
- **RDS Proxy**, falls die Verbindung über einen läuft. Sein
  `IdleClientTimeout` steht standardmäßig auf 1800 Sekunden, also genau dreißig
  Minuten, und ein Client, der auf eine lange Anweisung wartet, zählt für ihn
  als untätig;
- ein **NAT-Gateway oder eine Firewall** dazwischen. Eine lange einzelne
  Anweisung - der Indexaufbau am Ende einer Wiederherstellung, ein COPY einer
  riesigen Tabelle - sendet währenddessen in keine Richtung etwas, das Gerät
  verwirft den Fluss, und die Anweisung endet in einem Socket, den niemand mehr
  hält. AWS-NAT-Gateways verwerfen einen untätigen Fluss nach 350 Sekunden.
  Genau deshalb setzt die Anwendung TCP-Keepalives auf die Verbindung, sobald
  der Host kein Unix-Socket ist; damit bleibt der Fluss ohne eigenen Verkehr am
  Leben.

Nichts in der Anwendung begrenzt, wie lange ein Dump oder eine
Wiederherstellung dauern darf. Beide laufen als abgekoppelte Prozesse mit ihrem
Zustand auf Platte, überstehen also das Schließen oder den Neustart der
Anwendung, und weder die HTTP-Anfrage noch das Fenster warten auf sie.

#### Einen Dump oder Restore beim Laufen zusehen

`pg_dump` und `pg_restore` erzählen, was sie tun, und der Dialog zeigt alles
davon, sobald es kommt - von der ersten Zeile bis zur letzten, nicht nur die
letzte. Scrollen Sie hoch, um etwas zu lesen, hört er auf zu folgen; *follow*
abwählen hält ihn still. **Copy** legt das ganze Protokoll in die Zwischenablage.

Der Job läuft abgekoppelt: Den Dialog zu schließen stoppt ihn nicht, und das
Protokoll bleibt danach unter *Background jobs* erhalten.

#### „errors ignored on restore" ist kein gescheiterter Restore

`pg_restore` endet mit Fehlerstatus, sobald es **irgendeinen** Fehler
übergangen hat, und gibt nur eine Anzahl aus. Der mit Abstand häufigste ist ein
`CREATE` auf ein Objekt, das das Ziel schon hat - ein Schema in eine Datenbank
zurückzuspielen, die dieses Schema bereits hat, scheitert an `CREATE SCHEMA`
und stellt alles darunter einwandfrei wieder her.

Der Dialog listet deshalb die Fehler selbst auf und sagt, welche davon ein
bereits vorhandenes Objekt sind. Sind es alle, sagt er das deutlich: nichts
wurde deswegen ausgelassen. Ein echter Fehler bleibt ein echter Fehler.

#### Ein Dump aus einem neueren PostgreSQL

Ein einfacher `.sql`-Dump beginnt mit einer kurzen Folge von `SET`-Anweisungen,
und ein neueres `pg_dump` schreibt Einstellungen, die ein älterer Server nie
gehört hat - `transaction_timeout` kam mit PostgreSQL 17. In 16 eingespielt
stoppt diese Zeile den Restore sofort, und nichts wird wiederhergestellt.

Die Anwendung liest zuerst den Kopf und lehnt vor dem Start ab - mit dem Namen
der Einstellung, der Herkunft der Datei und der Version dieses Servers. Dumps
reisen vorwärts, nicht rückwärts: Nehmen Sie den Dump mit einem `pg_dump` auf,
das zum Ziel passt, oder spielen Sie ihn in einen mindestens so neuen Server
ein.

#### Welcher Client ausgeführt wird

Ein `pg_dump`, `pg_restore` oder `psql` einer anderen Hauptversion als der
Server ist nicht austauschbar, und die Abweichung schmerzt in
entgegengesetzten Richtungen:

- `pg_restore` und `psql` **sprechen mit** dem Server. Sie öffnen die Sitzung
  mit einem Block von `SET` Anweisungen, den ihre eigene Version bestimmt, also
  sendet ein Client, der neuer ist als der Server, Einstellungen, die dieser
  nie gehört hat - `pg_restore` 17 sendet `transaction_timeout`, das es unter
  17 nicht gibt, und jeder Restore meldet danach einen ignorierten Fehler und
  endet mit 1 auf einem Lauf, der funktioniert hat.
- `pg_dump` und `pg_dumpall` **lesen** den Server. Sie fragen Kataloge ab, die
  sie nur in ihrer eigenen Version kennen, also startet ein älterer als der
  Server überhaupt nicht: *"aborting because of server version mismatch"*.

Daher fragt die Anwendung den Server nach seiner Version und wählt eine
passende Kopie - aus den versionierten Verzeichnissen, die Distributionen
nebeneinander halten (`/usr/lib/postgresql/*/bin`, `/usr/pgsql-*/bin`,
Homebrew, die EDB Installer), ebenso aus der mitgelieferten Kopie und dem, was
auf `$PATH` liegt. Für einen Restore gilt eine zweite Bedingung - ein Client,
der älter ist als das `pg_dump`, das das Archiv schrieb, kann es nicht lesen -
und die wird nicht aus einer Tabelle beantwortet: der Kandidat wird mit
`pg_restore -l` gefragt, das den Index liest und nichts wiederherstellt, und
bei einer Absage wird die nächste Hauptversion darüber versucht.

Ein Archiv zurückzulesen hat überhaupt keinen Server darin, nur die Datei,
also nimmt die Backup-Prüfung die neueste installierte Kopie. Das ist so
sehr Fehlerbehebung wie Feinheit: ein einwandfreies Archiv, gelesen von einem
`pg_restore`, das älter ist als das `pg_dump`, das es schrieb, antwortet
*"unsupported version in file header"* - und das sieht genau wie
Beschädigung aus.

**Settings -> Tools** zeigt den Pfad, auf den jedes Werkzeug auflöst, und
listet die anderen installierten Hauptversionen daneben, denn der dort
gezeigte Pfad ist der Rückfall und nicht die ganze Antwort. Ein selbst
gesetzter Pfad ist absolut und schaltet die Auswahl vollständig ab - der
Notausgang, wenn ein Server einen Client braucht, den die Anwendung nicht
gewählt hätte.

#### Dateien auswählen

Überall, wo die Anwendung eine Datei liest oder schreibt - ein Export, ein
Dump, die einzuspielende Datei, der Import von Verbindungen, ein Zertifikat - 
öffnet sich der Dateidialog Ihres Systems, mit seinen Orten, seinen zuletzt
verwendeten Ordnern und seiner Überschreiben-Rückfrage. Auf einem System ohne
einen solchen Dialog fällt die Anwendung auf ihren eigenen Ordner-Browser
zurück, und nichts geht verloren.

Der Dialog wird dort geöffnet, wo die Dateien liegen. `pg_dump` läuft auf dem
Rechner, auf dem die Anwendung läuft, und schreibt dorthin, wo dieser Rechner
schreiben darf - also durchsucht der Dialog dieses Dateisystem.

Speichern fragt immer nach einem Dateinamen. Die einzige Stelle, die
stattdessen nach einem Ordner fragt, ist das Ziel einer Sicherung - ein Ordner
ist das, worin `pg_dump` schreibt.

### 5.10.5 Partitionierte Tabellen {#partitioned-tables}

Partitionierte Elterntabellen erscheinen unter der Kategorie **partitioned tables** im Schema-Baum. Ihre Blatt-Partitionen sind gewöhnliche Tabellen und würden sonst die Kategorie **tables** füllen - hundert `events_2026_NN`-Einträge ohne Zusammenhang -, also blendet der Baum sie standardmäßig aus. Eine Zeile `tables (12) · show 8 leaf partitions` bringt sie pro Schema zurück (die Wahl wird gemerkt), und jede angezeigte trägt eine `part of <Elternteil>`-Markierung.

**Erstellen**: der *+ Table*-Dialog hat einen Abschnitt **Partitioning** - RANGE, LIST oder HASH und einen Schlüssel wählen (eine Spalte, eine `(col1, col2)`-Liste oder ein Ausdruck wie `(date_trunc('month', at AT TIME ZONE 'UTC'))`). Der Elternteil wird leer erstellt; Partitionen fügen Sie danach hinzu. Jeder `PRIMARY KEY` oder `UNIQUE` auf einer partitionierten Tabelle muss jede Partitionsschlüssel-Spalte enthalten - PostgreSQL erzwingt das und die Fehlermeldung ist klar.

**Verwalten**: Rechtsklick auf eine partitionierte Tabelle, dann **Partitions…**. Der Dialog öffnet mit Strategie und Schlüssel und listet jede Partition: Grenzen, geschätzte Zeilen, Größe, eigenen Tablespace falls vorhanden, und ein `detach pending`-Label, falls ein nebenläufiger Detach unterbrochen wurde (siehe unten). Die Default-Partition ist markiert und zuletzt sortiert; eine selbst partitionierte Partition öffnet aus derselben Zeile.

- **Create partition** - eine neue leere Partition, in einer Anweisung angehängt. Die von der Strategie verlangten Grenzen ausfüllen.
- **Attach existing table** - bringt eine bereits geladene Tabelle herein. Die Anweisungsnotiz sagt, ob PostgreSQL das **Kind vollständig scannt** (zeigt Zeilenzahl und Größe) zur Grenzprüfung, oder ob es nur Metadaten sein kann. Nur Metadaten, wenn das Kind bereits einen `CHECK`-Constraint hat, der beweist, dass jede Zeile passt; sonst einen passenden `CHECK` hinzufügen, `ALTER TABLE ... VALIDATE CONSTRAINT` ausführen (nur `SHARE UPDATE EXCLUSIVE`, nicht der volle Lock), dann anhängen.
- **Detach** - die Tabelle ist keine Partition mehr und behält ihre Zeilen. **DETACH CONCURRENTLY** ist auf PostgreSQL 14+ standardmäßig angehakt: es nimmt keinen `ACCESS EXCLUSIVE`-Lock auf dem Elternteil, Abfragen auf den anderen Partitionen laufen weiter. Es läuft außerhalb einer Transaktion und kann nicht zurückgerollt werden; bei Abbruch bleibt die Partition **detach pending** - die Zeile zeigt dann **Finalize detach**, das es abschließt. PostgreSQL verweigert `DETACH CONCURRENTLY`, solange die Tabelle eine DEFAULT-Partition hat, und die Checkbox ist mit diesem Grund deaktiviert; auf PostgreSQL 13 und älter ist sie deaktiviert, weil es die Funktion nicht gibt, und ein einfacher Detach hält den exklusiven Lock für seine (meist kurze) Dauer.
- **Drop** - löscht die Partition und jede Zeile darin.

Auf den Klick passiert nichts. Der Dialog zeigt zuerst die Anweisung mit einer Zeile, die genau sagt, was sie sperrt, wie lange und was bei Abbruch passiert, und führt sie erst bei *Run it* aus.

**Show DDL** für einen partitionierten Elternteil gibt `CREATE TABLE ... PARTITION BY ...` aus; für eine Blatt-Partition `CREATE TABLE ... PARTITION OF <Elternteil> FOR VALUES ...` ohne Spaltenliste und nur mit den auf dieser Partition selbst definierten Constraints und Indexen.

**Umwandeln zwischen normaler und partitionierter Tabelle**: Rechtsklick auf eine Tabelle und **Convert to partitioned (recipe)…** wählen, oder Rechtsklick auf eine partitionierte Tabelle und **Flatten to a plain table (recipe)…** wählen. Keine der beiden Optionen führt etwas aus. Die Umwandlung ist ein Neuaufbau (eine neue Tabelle, eine Kopie aller Zeilen, dann ein Namenstausch), deshalb zeigt der Dialog ein SQL-Skript, das Sie kopieren, lesen und selbst ausführen. Das Skript ist eine einzige Transaktion. Schreibzugriffe warten von der ersten Anweisung bis zum COMMIT, und vor dem COMMIT vergleicht das Skript die Zeilenzahlen. Schlägt irgendetwas fehl, bleibt die Tabelle also, wie sie war. Übernommen werden:

- Spalten und Standardwerte, generierte Spalten sowie CHECK- und NOT-NULL-Constraints;
- Identity- und Serial-Zähler, sodass die nächste ID dort weitermacht, wo die alte Tabelle aufgehört hat;
- Schlüssel und Indizes unter ihren ursprünglichen Namen sowie Trigger;
- Eigentümer, Rechte, Kommentare und Row-Level-Security-Richtlinien;
- Fremdschlüssel in beide Richtungen, Views, die die Tabelle lesen, und Publikationen.

Bei einer Umwandlung füllen Sie zwei TODOs aus:

- den Partitionsschlüssel, den jeder PRIMARY KEY und jeder UNIQUE-Constraint enthalten muss;
- die Partitionen. Eine DEFAULT-Partition steht schon im Skript, damit beim Kopieren keine Zeile abgewiesen wird.

Was nicht übernommen werden kann, steht oben unter *Read first*. Beispiele sind NO-INHERIT-Checks auf einer partitionierten Tabelle, EXCLUDE-Constraints vor PostgreSQL 17, materialisierte Views und Funktionen, die den Zeilentyp der Tabelle verwenden. Die alte Tabelle bleibt als `<name>_old` erhalten, bis Sie sie löschen.

### 5.10.6 Ein Schema klonen

Rechtsklick auf ein Schema, dann **Transfer → Clone schema…**. Es kopiert jedes
Objekt in ein neues Schema derselben Datenbank: Tabellen, Views, Sequenzen,
Funktionen, Indizes und Constraints, auf Wunsch mit den Zeilen.

Fremdschlüssel werden so aufgebaut, dass sie auf die Kopie zeigen und nicht
zurück auf das Original - ein Klon, dessen Schlüssel weiter die Quelle
referenzieren, ist keine Kopie, sondern eine zweite Tür zu den Daten des ersten
Schemas.

Mit *copy the rows too* zählt die Reihenfolge, und sie ist Ihnen abgenommen:
Die Objekte entstehen ohne Fremdschlüssel, dann kommen die Zeilen, dann die
Schlüssel. Zeilen in Tabellen zu kopieren, die ihre Referenzen bereits
durchsetzen, scheitert an derjenigen, die zufällig zuerst drankommt. Sequenzen
werden dorthin gesetzt, wo die Originale standen, damit der erste Insert in die
Kopie nicht mit einer mitgekommenen Zeile kollidiert. Eine `IDENTITY`-Spalte
nimmt auch ihre Sequenzeinstellungen mit - `START WITH`, `INCREMENT BY`,
`MINVALUE`, `MAXVALUE`, `CACHE`, `CYCLE` - die Kopie fällt also nicht stumm
auf die Standardwerte zurück.

Die Zeilen werden **genau so übernommen, wie sie sind**, was immer in ihnen
steht. Alles, was eine Zeile beim Schreiben prüft oder umschreibt, wartet, bis
die Zeilen drin sind:

- ein Constraint, den die Quelle `NOT VALID` hat (ein `CHECK`, ein
  Fremdschlüssel, ein `CHECK` einer Domain, `NOT NULL ... NOT VALID` aus
  PostgreSQL 18), wird nach den Zeilen `NOT VALID` angelegt: Zeilen, die die
  Quelle nie geprüft hat, werden auch in der Kopie nicht geprüft, neue Zeilen
  schon;
- Trigger und Regeln entstehen nach den Zeilen, keiner davon lehnt eine
  kopierte Zeile ab, schreibt sie um oder fügt Zeilen hinzu;
- Row Level Security und ihre Policies werden nach den Zeilen eingeschaltet;
- ein Index, den ein gescheitertes `CREATE INDEX CONCURRENTLY` ungültig
  hinterlassen hat, wird nicht kopiert (er erzwingt in der Quelle nichts und
  scheitert auf denselben Zeilen erneut) - so wie `pg_dump` ihn auslässt.

Row Level Security lehnt ein Lesen nicht ab, sie kürzt es. Gelten ihre Policies
für die Rolle, mit der Sie verbunden sind (Sie sind nicht Eigentümer der
Tabelle oder die Tabelle hat `FORCE ROW LEVEL SECURITY`, und Sie sind weder
Superuser noch eine Rolle mit `BYPASSRLS`), oder dürfen Sie eine ihrer Spalten
nicht lesen, nennt das Klonen Tabelle und Grund **bevor** etwas läuft und
kopiert nichts, statt nur die Zeilen zu kopieren, die Sie sehen.

**Tabelle für Tabelle** zeigt für jede Tabelle die kopierten Zeilen, was wie in
der Quelle `NOT VALID` angelegt wurde, was nach den Zeilen entstand und was
nicht kopiert wurde. Lässt sich eine Zeile doch nicht kopieren, nennt die
Meldung Tabelle, Spalte, Constraint, die fehlerhafte Zeile und die Zeilen der
Quelle nach ihrem Primärschlüssel. **Tabelle klonen**, **Tabelle kopieren** und
die Wahl *kopieren* in **Schema zusammenführen** übernehmen die Zeilen genauso.

**Show the script** druckt das Ganze, bevor irgendetwas läuft. Der Klon selbst
läuft als eine Transaktion: entweder ganz oder gar nicht. Er läuft als
serverseitiger Job, ein Klon eines großen Schemas mit seinen Zeilen - Minuten
Arbeit - geht also nicht verloren, wenn Sie den Dialog schließen:
**Background jobs** zeigt ihn während des Laufs und behält das ganze Ergebnis
zum späteren Öffnen.

Am Ende listet der Dialog die Prüfungen auf, gegen die die Kopie verifiziert
wurde, statt eines bloßen „Fertig": das neue Schema wird zurückgelesen, bevor
die Transaktion committet, und jede Tabelle, Sequenz, jedes Constraint und
jede Zeilenzahl wird mit dem Original verglichen.

Das Klonen auf ein bereits vorhandenes Schema wird abgelehnt. Das ist ein
Zusammenführen, und dafür gibt es
[Zwei Schemas zusammenführen](#59-merging-two-schemas).

### 5.11 Den Eigentümer von allem in einem Schema ändern {#changing-the-owner-of-everything-in-a-schema}

Rechtsklick auf ein Schema → **Eigentümer aller Objekte ändern…**

Ein unter der falschen Rolle zurückgespieltes Schema oder ein ausscheidender
Kollege hinterlässt jedes Objekt darin im Besitz von jemandem, der es nicht
besitzen sollte. Das Objekt für Objekt zu tun ist bei drei Objekten in Ordnung
und bei dreihundert unbrauchbar.

Wählen Sie den neuen Eigentümer. Der Dialog listet jedes Objekt im Schema mit
eigenem Eigentümer, gruppiert wie im Schemabaum: Tabellen, partitionierte
Tabellen mit ihren Partitionen auf jeder Ebene darunter, Views, materialisierte
Views, Fremdtabellen, Sequenzen, Funktionen, Prozeduren, Aggregate, Typen,
Domänen, Bereichstypen, Operatoren, Operatorklassen und -familien, Kollationen,
Konvertierungen, Textsuche-Konfigurationen und -Wörterbücher,
Statistikobjekte, und das Schema selbst, ausgewählt. Unter der Liste stehen die
genauen `ALTER ... OWNER TO`-Anweisungen für das Ausgewählte; nichts läuft, bevor
Sie die Schaltfläche drücken. Objekte, die dem neuen Eigentümer schon gehören,
werden angezeigt und nicht angefasst.

Partitionen stehen einzeln in der Liste, weil PostgreSQL jeder Partition einen
eigenen Eigentümer gibt: Den Eigentümer einer partitionierten Tabelle zu ändern
ändert ihre Partitionen nicht. Eine partitionierte Tabelle aus- oder abzuwählen
tut dasselbe mit allem darunter.

Fremdserver und Publikationen, die die Tabellen des Schemas verwenden, gehören
der ganzen Datenbank, nicht dem Schema. Sie stehen am Ende, nicht ausgewählt;
wählen Sie sie aus, wenn sie mitwechseln sollen.

Manche Objekte werden nicht einzeln geändert, und der Dialog sagt, welche:

- eine Sequenz, die zu einer Tabellenspalte gehört (serial oder identity),
  wechselt mit ihrer Tabelle, und PostgreSQL lehnt es ab, sie allein zu ändern.
  Hat ihre Tabelle schon den neuen Eigentümer und sie selbst nicht, würde sie
  nichts verschieben: Eine serial-Sequenz steht dann mit den Anweisungen in der
  Liste, die sie lösen, übergeben und wieder an ihre Spalte binden; eine
  Identity-Sequenz lässt sich nicht lösen, sie wird über der Liste mit den zwei
  Anweisungen gezeigt, die sie über ihre Tabelle verschieben, und mit dem, was
  das kostet;
- Objekte, die zu einer Erweiterung gehören, behalten den Eigentümer der
  Erweiterung;
- Indizes haben immer den Eigentümer ihrer Tabelle.

Die Anwendung läuft in **einer Transaktion**. Ein halb umgeschriebenes Schema ist
schwerer zu durchschauen als eines, das niemand angefasst hat, und von außen ist
nicht zu sehen, wo es stehen blieb. Das Schema selbst wird zuerst geändert,
damit der neue Eigentümer darin anlegen darf, wenn ihm die Objekte übergeben
werden. Das Ergebnis zeigt die Anweisungen, die jedem Objekt seinen vorherigen
Eigentümer zurückgeben, falls die Änderung rückgängig gemacht werden muss.

Danach behält der alte Eigentümer in diesem Schema nur, was auch PostgreSQLs
eigenes REASSIGN OWNED übrig ließe: die Standardrechte, die diese Rolle für das
Schema gesetzt hat.

**Ein Objekt.** Rechtsklick auf ein Objekt → **Eigentümer ändern…** (für eine
Datenbank im Datenbankmenü). Der Dialog zeigt den aktuellen Eigentümer und die
genaue Anweisung, bevor etwas läuft. Bei einer partitionierten Tabelle sagt er
auch, wie viele Partitionen sie hat, alle Ebenen gezählt, und wem sie gehören,
und bietet **Auch den Eigentümer ihrer N Partitionen ändern** an. Das Kästchen
ist standardmäßig aus, wie in PostgreSQL: Die Tabelle wechselt den Eigentümer,
jede Partition behält ihren. Bleibt es aus, sagt das Ergebnis, wie viele
Partitionen ihren Eigentümer behalten haben.

### 5.12 Schemas zwischen Servern vergleichen {#comparing-schemas-across-servers}

Rechtsklick auf ein Schema → **Mit einem anderen Schema vergleichen**. Der
Vergleichsdialog öffnet sich mit diesem Schema auf Seite A; wählen Sie auch auf
Seite B eines, dann vergleicht er die beiden Schemas. Bleiben beide auf *ganze
Datenbank*, vergleicht er die Datenbanken.

Seite B hat eine eigene **Serverauswahl** mit Ihren gespeicherten Verbindungen,
der Vergleich kann also zwischen zwei Clustern laufen - Entwicklung gegen
Produktion ist der übliche Grund. Die Wahl eines Servers lädt dessen Schemaliste
neu, denn die Schemas eines Clusters sagen nichts über die eines anderen.

`sales` auf einem Cluster mit `sales` auf einem anderen zu vergleichen ist ein
ganz normaler Wunsch, die Sperre „gleiches Schema" greift daher nur, wenn beide
Seiten auf derselben Verbindung liegen.

Das Ergebnis ist der Unterschied auf Objektebene plus ein **Sync-Skript**, das B
an A angleichen würde. Nichts läuft von selbst: siehe
[Auf B ausführen](#running-it-on-b).

#### A und B nebeneinander {#a-and-b-side-by-side}

Jede abweichende Zeile - ein Objekt in einem Schema, Besitzer und Rechte eines
Schemas, eine Erweiterung, eine Eigenschaft, eine Rolle, eine Typumwandlung
(Cast), die Partitionen einer partitionierten Tabelle - öffnet sich an Ort und
Stelle, wenn Sie sie anklicken (oder ihren Pfeil drücken: Tab erreicht ihn,
Enter oder Leertaste öffnet). Zwei Bereiche zeigen die Definition aus A links
und aus B rechts, mit Zeilennummern:

- eine Zeile, die nur A hat, ist rot, eine Zeile, die nur B hat, grün;
- eine Zeile, die beide haben, aber nicht gleich, steht ihrem Gegenstück
  gegenüber, die abweichenden Wörter markiert;
- ein längerer unveränderter Abschnitt faltet sich zu *N unveränderte Zeilen*;
  ein Klick zeigt diese Zeilen;
- **A kopieren** und **B kopieren** kopieren den Text dieser Seite;
- unter den beiden Bereichen stehen die Anweisungen, die das Sync-Skript für
  dieses Objekt schreibt - dieselben, die **Sync-Skript erzeugen** schreibt,
  mit den jetzt gesetzten Optionen.

In einem schmalen Fenster stehen die Bereiche untereinander, A über B. Eine
Definition mit Tausenden Zeilen öffnet sich sofort: gezeichnet werden nur die
Änderungen und je drei Zeilen darum, bis Sie einen gefalteten Abschnitt öffnen.

**Datenbankvergleich** - Menü Datenbank, oder Rechtsklick auf die Datenbank in
der Seitenleiste → Verwalten → Mit einer anderen Datenbank vergleichen - 
arbeitet für ganze Datenbanken genauso und nimmt eine Liste von Schemas
entgegen, um den Vergleich einzugrenzen. Das zählt bei einer großen Datenbank:
der Objektdurchlauf je Schema ist die teure Hälfte, ein Schema von vierzig zu
vergleichen sollte also nicht so viel kosten wie alle vierzig.

Der Datenbankvergleich erfasst auch, was zu keinem Schema gehört:
Publikationen, Subskriptionen, Fremdserver, Event-Trigger und die
Typumwandlungen, die jemand mit CREATE CAST angelegt hat (die eingebauten und
die einer Erweiterung bleiben außen vor). Ein abweichender Cast hat ein
Kästchen wie ein Schema oder eine Rolle, und das Sync-Skript löscht ihn und legt
ihn neu an - ein Cast hält keine Daten und lässt sich nicht ändern.

Die Datenbank- und Schemalisten beider Seiten lassen sich in ihrer Auswahlliste sortieren: A-Z, Z-A oder größte zuerst. Datenbankgrößen sind exakt; Schemagrößen sind eine Schätzung aus der Planer-Statistik (letztes VACUUM oder ANALYZE), die nichts kostet und auf keine Sperre wartet. Der Eintrag für die ganze Datenbank bleibt in jeder Reihenfolge oben.

#### Auf B ausführen {#running-it-on-b}

Jede Zeile, für die das Skript Anweisungen schreibt, hat ein Häkchen, gesetzt;
eine Zeile, für die es nichts schreibt (eine Zeilenzahl, ein Hinweis), hat
eines, das sich nicht setzen lässt. Im Datenbankvergleich hakt
das Häkchen eines Schemas seine Objekte an. Unter dem Ergebnis arbeiten drei
Aktionen mit den Anweisungen der angehakten Zeilen, in der Reihenfolge des
Skripts:

- **Probelauf...** führt sie auf B in einer Transaktion aus, die immer
  zurückgerollt wird. Jede Anweisung läuft unter einem eigenen Savepoint und
  wird einzeln gemeldet: ausgeführt, fehlgeschlagen mit der Meldung von
  PostgreSQL, oder übersprungen. Nichts bleibt erhalten, aber solange er läuft,
  nimmt er dieselben Sperren wie das Anwenden.
- **Auf B anwenden...** führt sie auf B in einer Transaktion aus: alle oder,
  beim ersten Fehler, keine, mit der fehlgeschlagenen Anweisung benannt und den
  folgenden als nicht ausgeführt markiert. Jede Anweisung wartet höchstens 5
  Sekunden auf ihre Sperre.
- **Changeset exportieren...** speichert sie als Flyway-Migration
  (`V<Zeitstempel>__<Name>.sql`) oder als Liquibase-Changelog (formatiertes SQL
  oder YAML, ein Changeset je Anweisung, mit ID und Autor), wo Sie wählen.

Vor einem Probelauf oder dem Anwenden zeigt eine Prüfung, von B selbst gelesen:

- jede Anweisung mit der Sperre, die sie nimmt, und auf welchem Objekt - ACCESS
  EXCLUSIVE hält jedes Lesen und Schreiben dieser Tabelle bis zum Ende der
  Transaktion an, SHARE lässt Lesen weiterlaufen - und ob sie etwas löscht,
  löst oder neu schreibt;
- die Anweisungen, die nicht in einem Transaktionsblock laufen können (alles
  mit CONCURRENTLY, ALTER TYPE ... ADD VALUE vor PostgreSQL 12, ALTER SYSTEM):
  sie werden übersprungen und aufgeführt, um sie einzeln auszuführen;
- die Kopie: vor dem Anwenden werden die Definitionen jedes Objekts auf B, das
  die Anweisungen ändern oder löschen, als SQL gespeichert, das sie
  wiederherstellt; der Bericht verweist darauf, Hintergrundaufgaben bewahrt sie.
  Was sie nicht schreiben kann - eine Rolle, einen Cast, ein Statistikobjekt,
  eine Sortierfolge, Textsuchobjekte, Operatoren - wird mit dem Grund genannt.
  Die Zeilen sind nicht in der Kopie: eine Tabelle oder Spalte, die das
  Anwenden löscht, kommt daraus leer zurück;
- was Ihre Anmeldung auf B nicht darf, bei B erfragt: ein Objekt, das einer
  Rolle gehört, deren Mitglied Sie nicht sind, ein Schema, in dem Sie nichts
  anlegen dürfen, eine Rolle, der Sie kein Objekt übergeben können.

Wenn etwas gelöscht, gelöst oder neu geschrieben wird, wartet **Auf B
anwenden**, bis Sie den Namen des Schemas (zwei Schemas) oder der Datenbank
(zwei Datenbanken) von B eingeben.

Nach dem Anwenden wird sofort erneut verglichen; der Bericht bleibt über dem
neuen Ergebnis, und das ist, was übrig ist. Angewendet wird nur auf das B, mit
dem das Ergebnis verglichen wurde: zeigt der Dialog woandershin, verlangt er
einen neuen Vergleich. Jeder Probelauf und jedes Anwenden kommt mit seinen
Anweisungen ins Audit-Log; ein zurückgerolltes Anwenden wird als
fehlgeschlagen verzeichnet, mit der Anweisung, die fehlschlug.

#### Den Vergleich weitergeben

**PDF-Bericht** macht aus dem, was auf dem Schirm steht, ein Dokument: von
welchem Server und wann, die Zahlen, die Tabelle der Unterschiede und das
Abgleichsskript. Er wird aus derselben Antwort gebaut, die der Schirm zeigt - 
beide können also nicht auseinanderlaufen.

Die Datei wird dort geschrieben, wo Sie es wählen, von der Anwendung und nicht
vom Browser, und ein vorhandener Bericht wird nie ungefragt ersetzt.

Eine Grenze ist erwähnenswert: der Bericht nutzt die Schriften, die jeder
PDF-Betrachter ohnehin hat. Sie decken Deutsch und Spanisch vollständig ab,
nicht aber das kroatische č, ć und đ. Diese werden transliteriert - „sifra“
statt einer Lücke - damit ein Name auch ohne seinen Akzent lesbar bleibt.

#### Was die vier Status bedeuten {#what-the-four-statuses-mean}

Das gesamte Ergebnis eines Vergleichs ist ein Wort je Zeile, also lohnt sich
Genauigkeit. Beide Dialoge vergleichen **Struktur, nicht Daten** - Definitionen,
Eigenschaften und Rechte. Es wird auf keiner Seite eine Zeile gelesen, außer Sie
verlangen auch den aktuellen Zustand (unten), und auf keiner etwas geschrieben,
bis Sie die angehakten Zeilen auf B anwenden.

| | |
|---|---|
| **same** | Auf beiden Seiten vorhanden und identisch. Ein Schema, dessen Objekte alle übereinstimmen, bleibt ausgeblendet, solange *differences only* angehakt ist. |
| **changed** | Auf beiden Seiten unter demselben Namen vorhanden, aber die beiden Definitionen sind nicht derselbe Text: eine Spalte hinzugekommen, ein Typ erweitert, ein anderer Funktionsrumpf, eine Eigenschaft anders gesetzt. Ein Klick auf die Zeile zeigt A und B nebeneinander (siehe [A und B nebeneinander](#a-and-b-side-by-side)). |
| **only in A** | Existiert in der Quelle und nicht im Ziel. Ein Sync-Skript legt es auf B an. |
| **only in B** | Existiert im Ziel und nicht in der Quelle. Ein Sync-Skript **entfernt** es, denn B an A anzugleichen heißt zu löschen, was A nicht hat. |

Die Richtung ist nicht symmetrisch: A ist die Quelle, B das Ziel, und jedes
Skript aus diesen Dialogen verändert B, damit es wie A aussieht.

#### Den Vergleich eingrenzen {#restricting-what-is-compared}

**Was verglichen wird** enthält die Filter, und sie gelten für den Vergleich und
für das erzeugte Skript gleichermaßen - ein Vergleich, der Indizes ignoriert,
darf kein Skript erzeugen, das sie anfasst.

- **Objekttypen**: Tabellen, Views, materialisierte Views, Sequenzen,
  Funktionen. Einen abzuschalten macht den Vergleich blind dafür.
- **Innerhalb einer Tabelle**: Spalten, Indizes, Fremdschlüssel,
  Primärschlüssel, Unique-Constraints, Check-Constraints. So bekommt man einen
  Vergleich, der nur Fremdschlüssel ansieht, oder einen, der Indexunterschiede
  ignoriert - nützlich, wenn zwei Umgebungen bewusst unterschiedlich indiziert
  sind.
- **Nach Name**: eine Einschluss- und eine Ausschlussliste, beide mit `*`.
  `audit_*` im Ausschlussfeld hält den Vergleich bei dem, was zählt.
- Sechs Voreinstellungen erledigen die üblichen Kombinationen mit einem Klick:
  alles, nur Tabellen, nur Fremdschlüssel, nur Sequenzen, Struktur ohne Indizes,
  Views und Funktionen.
- **DROP-Anweisungen erlauben** entscheidet, ob das Sync-Skript entfernen darf,
  was es nur in B gibt. Ohne Haken erzeugt und ändert das Skript nur - die
  sichere Einstellung, wenn B nicht Ihnen gehört.
- **Tabellen in Tablespace und Access Method von A verschieben** ist
  ausgeschaltet, bis Sie es einschalten: eine Tabelle zu verschieben schreibt
  sie ganz neu, unter einer ACCESS-EXCLUSIVE-Sperre. Ausgeschaltet wird der
  Unterschied trotzdem gemeldet, und das Skript sagt, warum es ihn lässt.
- **Eine andere Spaltenreihenfolge als Unterschied zählen** ist ausgeschaltet.
  PostgreSQL kann keine Spalte verschieben, also kann keine Anweisung die
  Spalten von B in die Reihenfolge von A bringen. Ausgeschaltet vergleicht eine
  Tabelle mit denselben Spalten in anderer Reihenfolge als *same* und trägt
  einen Hinweis - *Spalten in anderer Reihenfolge* -, der beide Reihenfolgen
  zeigt, die von B so, wie sie nach dem Skript sein wird (eine Spalte, die das
  Skript hinzufügt, landet am Ende). Eingeschaltet zählt die Reihenfolge, und
  das Skript sagt, dass die Tabelle dafür von Hand neu aufgebaut werden muss.

#### Partitionen, Vererbung und Speicherung {#partitions-inheritance-and-storage}

Eine Partition wird mit ihrem Platz im Baum verglichen: Elterntabelle und
Grenze (`FOR VALUES ...`), eine partitionierte Tabelle mit ihrem
Partitionsschlüssel. Das Sync-Skript verschiebt Partitionen mit **ATTACH
PARTITION** und **DETACH PARTITION**, nie durch Löschen und Neuanlegen, denn
eine Partition hält Zeilen:

- eine geänderte Grenze: abgehängt, dann mit der Grenze aus A wieder
  angehängt;
- eine Partition, die A angehängt hat und B als eigene Tabelle führt:
  angehängt; umgekehrt: abgehängt;
- eine Partition unter einer anderen Elterntabelle: von der in B abgehängt, an
  die aus A angehängt;
- eine Partition, die nur B hat, unter einer Elterntabelle, die beide Seiten
  haben: abgehängt und als eigene Tabelle behalten. Gelöscht wird sie danach
  nur, wenn **DROP-Anweisungen erlauben** angehakt ist.

Jedes DETACH läuft zuerst, in einem eigenen Abschnitt, damit sich eine weiter
unten geschriebene Grenze nicht mit einer noch angehängten Partition
überschneidet. Der Kopf des Skripts nennt die Sperren: DETACH nimmt ACCESS
EXCLUSIVE auf Elterntabelle und Partition; ATTACH nimmt SHARE UPDATE EXCLUSIVE
auf der Elterntabelle (ACCESS EXCLUSIVE vor PostgreSQL 12) und liest die
Partition, um zu beweisen, dass ihre Zeilen in die Grenze passen. Ab
PostgreSQL 14 folgt jedem DETACH seine Form `... CONCURRENTLY` als Kommentar:
sie nimmt eine schwächere Sperre, kann aber nicht in einem Transaktionsblock
laufen und nicht, wenn die Elterntabelle eine DEFAULT-Partition hat - führen
Sie sie also einzeln aus, wenn Sie sie nutzen.

Ein Partitionsschlüssel lässt sich nicht an Ort und Stelle ändern. Wenn A und B
eine Tabelle unterschiedlich partitionieren, sagt das Skript das und schreibt
nichts Ausführbares dafür; was A unter diese Tabelle hängt, wartet als
Kommentar, bis der Schlüssel von Hand geklärt ist.

**INHERITS.** Eine Tabelle, die von einer anderen erbt (klassische Vererbung,
keine Partition), wird mit ihren Elterntabellen verglichen. Das Skript schreibt
`ALTER TABLE ... INHERIT` oder `NO INHERIT`, und ein neues Kind wird nur mit
seinen eigenen Spalten angelegt, nach seiner Elterntabelle.

**Tablespace und Access Method** gehören zur Definition einer Tabelle: eine
Tabelle in einem anderen Tablespace oder mit einer anderen Access Method
gespeichert vergleicht als changed. Eine neue Tabelle wird dort angelegt, wo A
sie hat; eine vorhandene wird nur mit dem Schalter oben verschoben.

**Eine Spalte, deren Typ sich unter Views ändert.** PostgreSQL verweigert
`ALTER COLUMN ... TYPE`, solange eine View, eine materialisierte View oder eine
Regel die Spalte liest. Das Skript löscht jede davon auf B direkt vor der
Änderung - auch die Views, die auf ihnen aufbauen, in jedem Schema, die
äußerste zuerst - und legt sie direkt danach wieder an, so wie sie auf B sind:
Definition und Optionen, die Indizes und Zeilen einer materialisierten View
(REFRESH), die Spalten-Defaults und INSTEAD-OF-Trigger einer View, Regeln,
Besitzer, Kommentare, Rechte und Spaltenrechte. Jedes DROP und CREATE hält eine
ACCESS-EXCLUSIVE-Sperre auf seinem Objekt bis zum Ende der Transaktion, Abfragen
darauf warten also; die Prüfung vor dem Ausführen kennzeichnet jedes solche
DROP als eines, das zurückkommt. Was sich von hier aus nicht wiederherstellen
lässt - ein Trigger, dessen WHEN die Spalte liest, eine Policy, eine
generierte Spalte, ein Funktionsrumpf - nennt das Skript als Arbeit von Hand,
bevor es läuft.

**Eine Spalte, die unter Views gelöscht wird.** PostgreSQL verweigert
`DROP COLUMN`, solange eine View, eine materialisierte View oder eine Regel die
Spalte liest. Das Skript löscht jede davon auf B direkt vor der Spalte - auch
die Views, die auf ihnen aufbauen, die äußerste zuerst - und danach wird jede
zu dem, was der Rest des Skripts aus ihr macht: eine, die es nur auf B gibt,
bleibt gelöscht; eine, die A anders definiert, ohne die Spalte, wird so
angelegt, wie sie in A ist, mit Besitzer, Rechten und Kommentaren von B (und
den Spaltenkommentaren, Spaltenrechten, Regeln, Triggern und Indizes, die keine
verlorene Spalte nennen), und ihre eigene Änderung wird weiter unten nicht ein
zweites Mal geschrieben; jede andere kommt so zurück, wie sie auf B ist. Die
Prüfung vor dem Ausführen kennzeichnet jedes DROP, das zurückkommt. Eine View,
die zurückkäme und die Spalte noch läse - eine in einem anderen Schema, eine,
die das Skript nicht anfasst, eine auf einer View, die sich ändert oder
wegfällt - kann nicht zurückkommen, ebenso wenig ein Trigger, dessen WHEN die
Spalte liest, eine Policy oder ein Funktionsrumpf: dann sagt das Skript, was
sich zuerst ändern muss, und löscht diese Spalte nicht (es schreibt das DROP
als Kommentar), denn ein DROP, das scheitert, rollt den ganzen Lauf zurück.
Ändern Sie dieses Objekt und vergleichen Sie erneut.

#### Besitzer und Rechte {#owner-and-grants}

Die Schalter **Besitzer** und **Rechte** entscheiden zweierlei zugleich: ob ein
Unterschied darin zählt, wem ein Objekt gehört oder wer es benutzen darf - und
ob das Sync-Skript die ALTER ... OWNER TO, GRANT und REVOKE schreibt, die ihn
schließen. Beide sind standardmäßig an; ist einer aus, wird diese Art von
Unterschied weder gemeldet noch geskriptet. Zu den Rechten gehören
Spaltenrechte, die Rechte auf die Sequenz einer Identity-Spalte, was ein
Besitzer sich selbst entzogen hat, sowie die Rechte des Schemas selbst und die
darin gesetzten Standardrechte, die als eine Zeile namens *(this schema)*
erscheinen. Ein Recht wird so gelesen, wie PostgreSQL es anwendet: ein Objekt,
auf das nie jemand Rechte vergeben hat, und eines, dessen Rechte vergeben und
wieder entzogen wurden, vergleichen gleich. Nennt das Skript eine Rolle, die es auf
dem Ziel nicht gibt, sagt sein Kopf, welche.

#### Auch den aktuellen Zustand vergleichen {#comparing-the-current-state-too}

Verglichen wird nach Definition: Zwei Schemas sind gleich, wenn jedes Objekt
darin auf dieselbe Weise angelegt würde. **Auch den aktuellen Zustand
vergleichen** nimmt hinzu, was eine laufende Datenbank hat und eine Definition
nicht. Jeder Schalter ist standardmäßig aus, weil sich zwei Kopien desselben
Schemas im Zustand immer unterscheiden.

| | |
|---|---|
| **aktuelle Sequenzwerte** | Wo jede Sequenz steht und ob sie schon abgerufen wurde. Das Skript setzt die von B mit setval(). |
| **ob eine materialisierte Sicht befüllt ist** | Das Skript führt REFRESH MATERIALIZED VIEW aus, oder REFRESH ... WITH NO DATA, wenn die in A leer ist. |
| **Zeilenzahlen** | Tabellen, Partitionen und materialisierte Sichten. Die Schätzung stammt aus der Planer-Statistik und gilt als verschieden, wenn die beiden 5% oder mehr auseinanderliegen; **exakte Zeilenzahlen** führen count(*) auf jeder Tabelle aus, lesen jede Zeile und vergleichen genau. |
| **Größen** | Gesamtgröße samt Indizes und TOAST. Verschieden, wenn die beiden mindestens 1 MB und dabei 10% oder 64 MB auseinanderliegen. |
| **Gültigkeit und Nutzung der Indizes** | Ein Index, den ein gescheitertes CREATE INDEX CONCURRENTLY ungültig hinterlassen hat, und wie oft jeder Index benutzt wurde: verschieden, wenn eine Seite ihn benutzt hat und die andere nie, oder wenn die beiden Zahlen mehr als 1000 auseinanderliegen. |

Zeilenzahlen, Größen und Indexnutzung werden nur gemeldet und nie geskriptet -
keine Anweisung eines Sync-Skripts würde sie angleichen - und das Skript endet
mit der Liste der Objekte, die sich darin unterscheiden. Im Datenbankvergleich
werden die abweichenden Partitionen einer partitionierten Tabelle mit Namen
genannt, nicht nur gezählt.

#### Vergleichen, was eine Rolle auf beiden Seiten darf {#comparing-what-a-role-may-do-on-two-sides}

**Benutzer vergleichen** beantwortet "kann diese Rolle hier dasselbe wie dort",
und fragt in dieser Reihenfolge: welche Rollen, von wo, dann wie tief.

Rollen gelten in PostgreSQL clusterweit - ein `pg_authid` je Cluster - die
Rollenliste ist also das, was die beiden *Server* haben, und eine Rolle, die auf
einer Seite fehlt, wird gemeldet statt übersprungen. Rechte sind das Gegenteil:
sie liegen im Katalog jeder Datenbank, deshalb zählen die Datenbank-Auswahlen
und deshalb ist der Vergleich je Datenbank. Schemas zu nennen grenzt weiter
ein; leer gelassen werden alle Schemas verglichen, die die Verbindung sieht.

**PUBLIC** - jede Rolle, jetzt und künftig - hat eine eigene Zeile in der
Rollenliste. Sie hat keine Attribute zum Vergleichen, nur Rechte: **compare
privileges** in dieser Zeile zeigt, was PUBLIC auf jeder Seite gewährt ist, und
das Sync-Skript schreibt die GRANT ... TO PUBLIC und REVOKE ... FROM PUBLIC, die
das schließen.

Sieben Ebenen werden gelesen, und die erste ist die, von der alle anderen
abhängen:

| | |
|---|---|
| **database** | `CONNECT`, `CREATE`, `TEMPORARY` - ob die Rolle überhaupt hereinkommt |
| **schema** | `USAGE`, `CREATE` |
| **table** | sowie Views, materialisierte Views, partitionierte und Fremdtabellen |
| **column** | eine Rolle kann `SELECT` auf drei Spalten und nichts auf der Tabelle haben, was jede Prüfung auf Tabellenebene als "kein Zugriff" liest |
| **sequence** | `USAGE`, `SELECT`, `UPDATE` |
| **function** | `EXECUTE` |
| **default** | was *neue* Objekte tragen werden - eine Rolle kann auf allem Bestehenden korrekt berechtigt sein und den Zugriff auf alles Morgige verlieren |

Die Rechte werden direkt aus den Katalog-ACLs gelesen, nicht über
`has_table_privilege`. Diese Funktion beantwortet "darf diese Rolle X"
einschließlich allem über Mitgliedschaft Geerbten, und geerbtes Recht lässt sich
auf dieser Rolle weder gewähren noch entziehen. Handeln kann man nur bei einem
direkten Recht, also werden nur diese verglichen; die Mitgliedschaft selbst wird
als Rollenattribut verglichen.

**Das Sync-Skript** gleicht eine Seite an die andere an, in der Richtung, die
Sie wählen. Keine Seite ist automatisch die richtige - die Produktion kann die
sein, die abgedriftet ist, oder die korrekte - also werden beide Richtungen
angeboten, und keine ist so voreingestellt, dass sie die Wahl verbirgt.

Rechte, die das Ziel hat und die Quelle nicht, werden als Kommentare
geschrieben, bis Sie *auch entziehen* anhaken. Ein Recht zu geben ist
umkehrbar; es jemandem mitten im Arbeitstag zu nehmen nicht.

Das Anwenden aus dem Datenbankvergleich zeigt die Anweisungen zuerst.
**Anwenden** erteilte die Rechte dort früher mit dem Klick und meldete danach
eine Zahl; jetzt öffnet es denselben Dialog wie jedes erzeugte Skript - 
kopieren, speichern, im SQL-Editor öffnen - mit Anwenden daneben, und führt nur
aus, was Sie gesehen haben. Ändern sich die Rechte zwischen Ansehen und
Anwenden, hält das Anwenden an und reicht Ihnen die neue Liste, statt etwas
anderes auszuführen.

Es wird nichts ausgeführt. Lesen Sie das Skript, speichern Sie es mit **Skript
speichern…** und starten Sie es, wenn Sie es entscheiden.

### 5.13 Eine Datenbank klonen {#cloning-a-database}

Rechtsklick auf eine Datenbank → **Datenbank klonen (gleicher Server)**.

Das ist eine Anweisung - `CREATE DATABASE "kopie" TEMPLATE "original"` - und
PostgreSQL kopiert die Datenbank auf Dateiebene, statt sie zu dumpen und
zurückzuspielen. Darum ist es schnell: eine 25-MB-Datenbank ist in deutlich unter
einer Sekunde geklont, und die Zeit hängt an der Größe, nicht an der Zahl der
Objekte.

Die Kopie ist vom ersten Moment an vollständig unabhängig. Schreiben in die eine
berührt die andere nicht; sie teilen sich keinen Speicher.

#### Der Preis, und warum der Dialog darauf besteht

Eine Datenbank kann nicht als `TEMPLATE` dienen, solange etwas mit ihr verbunden
ist. Vor dem Kopieren führt DBApguin daher aus:

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'quelle'
```

**Jede offene Verbindung zur Quelle wird beendet**, auch die anderer Leute, und
jede laufende Transaktion abgebrochen. Das ist eine Vorgabe von PostgreSQL und
keine Entscheidung von DBApguin - deshalb verlangt der Dialog ein Häkchen statt
bloßes Durchklicken.

**Ihre eigene Verbindung geht mit.** Die Sitzung von DBApguin auf der
Quelldatenbank wird wie jede andere beendet, und der Verbindungspool öffnet still
eine neue für die nächste Anfrage - die Anwendung läuft also weiter und nichts
sieht falsch aus. Was nicht überlebt, ist alles, was an der alten Sitzung hing:
eine offene Transaktion im SQL-Editor wird zurückgerollt, temporäre Tabellen sind
weg, `SET` auf Sitzungsebene steht wieder auf Standard, Advisory Locks sind
freigegeben, und eine gerade laufende Abfrage bricht ab. Committen oder beenden
Sie, woran Sie arbeiten, bevor Sie klonen.

#### Was mitkommt und was nicht

Alles *innerhalb* der Datenbank wird kopiert. Die Eigenschaften *an* der
Datenbank nicht - sie gehören zum Katalog des Clusters und nicht zu den Dateien,
die kopiert werden:

| Kopiert | Nicht kopiert |
|---|---|
| Tabellen, Zeilen, Indizes, Constraints | der **Eigentümer** der Datenbank - der Klon gehört dem, der geklont hat |
| Sequenzen, auf ihrem aktuellen Stand | `GRANT CONNECT` / `TEMPORARY` auf der Datenbank |
| Views, materialisierte Views, Funktionen, Typen | Einstellungen je Datenbank (`ALTER DATABASE … SET work_mem`, `search_path`, …) |
| Erweiterungen | der Kommentar an der Datenbank |
| Tabellen- und Spaltenrechte | Rollen und Benutzer (sie gelten clusterweit und existieren bereits) |
| Kommentare an Objekten, RLS-Policies, Large Objects | |

Die rechte Spalte ist der Teil, der überrascht. Eine für Tests geklonte
Produktionsdatenbank gehört Ihnen, hat keine `CONNECT`-Rechte für die Rolle der
Anwendung und läuft mit den Standardwerten des Servers für `work_mem` und
`search_path` statt mit denen des Originals. Die Daten sind identisch; das Umfeld
nicht.

Um diese vier wiederherzustellen, auf dem Klon:

```sql
ALTER DATABASE kopie OWNER TO ursprünglicher_eigentümer;
GRANT CONNECT, TEMPORARY ON DATABASE kopie TO anwendungsrolle;
ALTER DATABASE kopie SET work_mem = '64MB';
COMMENT ON DATABASE kopie IS '…';
```

#### Wann man es nutzt

- vor einer Migration oder riskanten Änderung, damit es einen exakten Stand zum Zurückkehren gibt
- für eine Test- oder Entwicklungskopie auf demselben Server
- **nicht** zum Verschieben auf einen anderen Server - `TEMPLATE` wirkt nur
  innerhalb eines Clusters; dafür Backup und Zurückspielen
- **nicht** an einer Datenbank, die Leute gerade nutzen - sie werden getrennt
- ein Klon belegt so viel Platte wie das Original; nichts wird geteilt

### 5.14 Eine Datenbank umbenennen {#renaming-a-database}

Rechtsklick auf eine Datenbank -> **Rename database**.

Das ist eine Anweisung, `ALTER DATABASE "alt" RENAME TO "neu"`, und sie ändert
**nur den Katalogeintrag**. Es wird nichts kopiert oder verschoben, also ist
sie unabhängig von der Größe der Datenbank sofort fertig. Der Eigentümer, jedes
`GRANT`, der Kommentar und alle Per-Datenbank-Einstellungen
(`ALTER DATABASE ... SET ...`) bleiben angehängt und unverändert - der Name ist
danach das Einzige, was anders ist.

#### Sie kann nicht laufen, solange jemand verbunden ist

PostgreSQL verweigert `ALTER DATABASE ... RENAME`, solange irgendeine Sitzung
mit dieser Datenbank verbunden ist - einschließlich DBApguins eigenem
Verbindungspool, wenn Sie darin gestöbert haben. Ohne Hilfe scheitert die
Anweisung mit:

```
database "shop" is being accessed by other users
DETAIL: There is 1 other session using the database.
```

Also prüft der Dialog zuerst. Ist die Datenbank untätig, läuft das Umbenennen
einfach. Ist sie es nicht, zeigt der Dialog, **wie viele Verbindungen es gibt
und welche Benutzer sie halten**, und bietet an, sie zu beenden - dasselbe
`pg_terminate_backend(...)`, das Clone verwendet. Das ist destruktiv: eine
offene Transaktion auf dieser Datenbank wird zurückgerollt, temporäre Tabellen
und sitzungsweite `SET`s sind weg, und eine gerade laufende Abfrage wird
abgebrochen. Sie müssen das Kästchen ankreuzen.

DBApguins eigene gepoolte Verbindungen werden mit den anderen beendet; der Pool
öffnet beim nächsten Request einfach frische gegen den neuen Namen. Die
Verbindung, die das Umbenennen ausführt, ist auf der Datenbank `postgres`, nie
auf dem Ziel, also kann sich das Umbenennen nie selbst abschneiden.

#### Ihre gespeicherte Verbindung und offene Tabs

Wenn Sie die Datenbank umbenennen, mit der dieses Fenster **gerade verbunden**
ist, aktualisiert DBApguin den gespeicherten Verbindungseintrag auf den neuen
Namen, schaltet die aktive Verbindung um und lädt den Objektbaum neu. Offene
SQL- und Schema-Tabs, die an den alten Namen gebunden waren, werden mit einem
Abzeichen als veraltet markiert, bis Sie sie aktualisieren - genau wie bei
jedem Datenbankwechsel. Eine gespeicherte Verbindung für ein *anderes* Fenster,
die ebenfalls auf den alten Namen zeigte, wird nicht angefasst - passen Sie
sie von Hand an.

#### Was es nicht behebt

Das Umbenennen ist eine Katalogänderung, und nichts außerhalb des Katalogs
weiß davon: eine Anwendung mit dem alten Datenbanknamen im Connection-String,
ein `~/.pg_service.conf`-Eintrag, ein Backup-Skript, eine `dblink`- oder
`postgres_fdw`-Serverdefinition oder SQL, das den Namen fest verdrahtet.
`current_database()` in einer Funktion gibt ab dem Commit des Umbenennens den
neuen Namen zurück. Benennen Sie eine Produktionsdatenbank nur um, wenn Sie
alles, was sie erreicht, im selben Wartungsfenster anpassen können.

`postgres`, `template0` und `template1` können nicht umbenannt werden - der
Dialog verweigert sie - weil initdb und viele Werkzeuge annehmen, dass es diese
Namen gibt.

### 5.15 Erweiterungen, und was „verschieben" tut {#extensions-and-what-move-does}

**Datenbank -> Erweiterungen verwalten** listet, was der Server bereithält und
was diese Datenbank installiert hat, mit der installierten Version neben der
Standardversion, sodass ein Kandidat für `ALTER EXTENSION ... UPDATE` sofort
auffällt.

Eine Erweiterung ist ein Bündel: Funktionen, Operatoren, Typen, manchmal
Tabellen, alle in ein Schema installiert. **Verschieben** führt

    ALTER EXTENSION "name" SET SCHEMA "ziel"

aus und versetzt das ganze Bündel mit einer Anweisung.

Warum man das täte:

- **Damit `public` Ihnen gehört.** `CREATE EXTENSION pg_trgm` ohne Schema legt ein
  paar Dutzend Funktionen in das erste Schema des `search_path`, üblicherweise
  `public`, zwischen Ihre eigenen Tabellen.
- **Sicherheit.** Wer in einem Schema Objekte anlegen darf, kann eine Funktion mit
  demselben Namen und denselben Argumenttypen definieren wie eine Funktion der
  Erweiterung. Kommt dieses Schema früher im `search_path`, läuft ihre Version.
  Erweiterungen in einem eigenen Schema `ext`, in das nur der Eigentümer
  schreiben darf, nehmen diese Möglichkeit weg. Deshalb lautet das übliche
  Rezept `CREATE SCHEMA ext; CREATE EXTENSION pg_trgm SCHEMA ext;`

Was es nicht tut:

- **Nicht verschiebbare Erweiterungen lassen sich nicht verschieben.** Die
  Erweiterung erklärt das selbst (`pg_extension.extrelocatable`), meist weil ihr
  eigener Code ihr Schema beim Namen nennt. PostGIS ist der bekannte Fall.
  DBApguin markiert solche als `(non-relocatable)` und bietet keine
  Verschieben-Schaltfläche an, statt den Server ablehnen zu lassen.
- **Es ändert Ihren Code nicht.** Alles, was das alte Schema ausdrücklich nennt - 
  ein View-Rumpf, eine Funktion, eine Abfrage der Anwendung - bricht.
  Unqualifizierte Aufrufe laufen nur weiter, wenn das neue Schema im
  `search_path` steht: `ALTER DATABASE app SET search_path = public, ext;`
- **Es ist kein Versionswechsel.** Dafür gibt es Aktualisieren.

Verschieben setzt das Eigentum an der Erweiterung voraus, praktisch also
Superuser. Die Anweisung ist schnell - sie schreibt Katalogeinträge um, keine
Daten - , nimmt aber eine Sperre auf die Objekte der Erweiterung; tun Sie es
also, wenn nichts dagegen läuft.

### 5.16 Aufgabenliste

Ein Alarm, eine Advisor-Empfehlung und ein Check-Befund sagen alle "das sollte
jemand tun" und verschwinden dann. Ein Cluster mit 1,2 Milliarden alten Transaktions-IDs und neun
Tabellen ueber der Schwelle ist die Arbeit eines Vormittags, kein Toast.

**View -> Aufgabenliste**, mit der Anzahl der offenen Aufgaben am Menue selbst;
die Zahl wird rot, wenn eine davon kritisch ist.

Aufgaben kommen von drei Stellen in die Liste: aus einer **Benachrichtigung**,
aus dem **Fenster fuer kritische Alarme** (Bestaetigen heisst "gesehen", nicht
"erledigt") und aus **jeder bewerteten Advisor-Zeile**. Eine eigene Aufgabe kann
man auch einfach eintippen; die verschmilzt mit nichts und wird nie als veraltet
markiert.

Jede Aufgabe merkt sich Server, Datenbank und Objekt. **Passwoerter werden nie
gespeichert** - die Verbindung wird nur zur Identifikation als Host, Port,
Datenbank und Benutzer abgelegt.

**Eine Aufgabe, deren Zustand nicht mehr besteht, sagt das.** Sie bleibt in der
Liste und hoert nur auf, aktuell zu sein. **Checkliste exportieren** schreibt die
Liste als reinen Text mit `[ ]`-Kaestchen, Objekt, Server und eigener Notiz.

### 5.17 Foreign Server und User Mappings {#foreign-servers-and-user-mappings}

**Database -> Manage foreign servers** listet jeden Foreign Server dieser
Datenbank mit dem dahinterliegenden Wrapper, seinem Eigentümer, seinen Optionen
und den daran hängenden User Mappings.

Einen Foreign Server anzulegen war ein Assistent; alles danach war der
SQL-Editor. Ein Remote-Host zieht um, ein Port ändert sich, ein gespeichertes
Remote-Passwort läuft ab, ein Server muss einem anderen Eigentümer übergeben
werden - alles alltäglich, nichts davon von der Stelle erreichbar, die den
Server erzeugt hat, und die Optionen, nach denen der Assistent gefragt hatte,
wurden nirgends zurückgezeigt. Das ist diese zweite Hälfte.

#### Woraus ein Foreign Server besteht

Drei Objekte, und alle drei müssen zusammenpassen, bevor eine einzige Zeile
hinübergeht:

| Objekt | was es trägt | wem es gehört |
|---|---|---|
| der Wrapper | `postgres_fdw`, `mysql_fdw`, `file_fdw`, ... | der Erweiterung |
| der Server | wo die Gegenseite liegt: Host, Port, dbname | der Datenbank |
| das User Mapping | wer Sie dort sind: Remote-Benutzer und Passwort | einer lokalen Rolle |

Das Mapping ist der Teil, den man vergisst. PostgreSQL sucht es **anhand der
Rolle, die sich tatsächlich verbunden hat**, also ist ein Server mit einem
Mapping nur für `postgres` für alle anderen unsichtbar: ihre Abfragen scheitern
an *"user mapping not found"*, wie richtig der Server auch eingestellt ist.
Darum wird ein Mapping für jede Login-Rolle des Clusters angeboten und nicht nur
für Sie selbst und `PUBLIC` - eine lokale Rolle auf einen Remote-Login ist die
normale Form und der Grund, warum Remote-Zugangsdaten überhaupt pro Rolle sind.

Ein Mapping **für eine NOLOGIN-Gruppenrolle** nimmt der Server an und benutzt es
dann nie, weil eine solche Rolle sich nie verbindet. Diese Rollen und die
eingebauten `pg_*` fehlen in der Auswahl mit Absicht.

#### Einen Server bearbeiten

**Edit** zeigt die Optionen, die der Server jetzt trägt, jede als bearbeitbares
Feld, mit Platz für Schlüssel, die er noch nicht hat. Gesendet wird nur, was Sie
ändern.

**Den Wert einer Option zu leeren entfernt den Schlüssel.** PostgreSQL kennt
keine Option mit leerer Zeichenkette, und `OPTIONS (SET k '')` ist eine andere
Anweisung als ein fehlender Schlüssel; das leere Feld ist also die Art zu sagen
"entferne das".

Ob ein Schlüssel `ADD`, `SET` oder `DROP` braucht, hängt davon ab, ob der Server
ihn schon trägt, und das wird in derselben Transaktion wie das `ALTER` aus dem
Katalog gelesen, nicht aus dem geraten, was der Bildschirm zuletzt zeigte. Eine
Änderung von vier Optionen kommt als eine Anweisung heraus:

    ALTER SERVER "loopback" OPTIONS (SET host '10.0.0.9', SET port '5433',
                                     DROP dbname, ADD connect_timeout '7')

**Version** und **Owner** sind absichtlich getrennte `ALTER SERVER` Anweisungen.
PostgreSQL nimmt sie in einer an, aber dann reißt ein einziger falscher
Optionsschlüssel die Eigentümeränderung mit, und der Fehler benennt keines von
beidem. Alle laufen in einer Transaktion, ein halb angewandter Stand ist also
kein Zustand, in dem Sie landen können.

#### Passwörter

Das Remote-Passwort eines Mappings wird nie gezeigt. PostgreSQL gibt einem
Client kein gespeichertes Passwort zurück, und eine Serveroption namens
`password` ist hier aus demselben Grund maskiert - auch im zurückgegebenen SQL
und im Audit-Log, einer Datei, die Leute exportieren und weitergeben. Das
Passwortfeld im Mapping-Editor ist daher leer und wird nur gesendet, wenn Sie
etwas eintippen.

#### Löschen

Ein Server, der noch Foreign Tables trägt, verweigert ein einfaches `DROP`, also
bietet der Dialog `CASCADE` ausdrücklich an und zählt die User Mappings, die
mitgehen. CASCADE nimmt auch jede Foreign Table dieses Servers mit - das sind
lokale Katalogeinträge, auf der Gegenseite wird nichts angefasst, aber die
Abfragen und Views, die darüber lesen, brechen.

**Drop user mapping** ist das schmalere Messer: diese eine Rolle verliert ihre
Zugangsdaten für den Server und ihre Abfragen scheitern, bis wieder ein Mapping
existiert, während jede andere Rolle weiterarbeitet.

## 6. Übliche Arbeitsabläufe

### 6.1 Leistungstriage auf einer langsamen Datenbank

1. Verbindung öffnen.
2. Zuerst `db-health` ausführen - der Überblick auf einem Bildschirm.
3. Ist die Cache-Trefferrate niedrig: `db-cache-advisor` für Details je Objekt.
4. Überwiegen Seq Scans: `db-index-advisor` für Indexkandidaten.
5. Sind Abfragen langsam: `db-stat-statements` für die Top 25 mit Hinweisen je Abfrage. Nach total_exec_time sortieren oder die mit WARN nehmen.
6. Steht Bloat im Verdacht: `db-bloat-breakdown` für HEAP+TOAST+Indizes je Tabelle. Für eine genaue (langsame) Messung `db-pgstattuple`.

### 6.2 Sperrproblem

1. `db-health` -> Kategorie BLOCKING, dann in die Details für die Paare aus Blocker und Wartendem.
2. Oder Sitzungsansicht -> Liste, nach nicht leerem blocking_pids sortieren.
3. Rechtsklick auf die blockierende Sitzung und Beenden, wenn Sie so weit sind.

### 6.3 Replikationsrückstand

1. `db-replication` für den Zustand der Replik und den Rückstand bei Schreiben/Flush/Replay in Zeit und Bytes.
2. `db-repl-slots` für die WAL-Aufbewahrung je Slot. Inaktive Slots, die Gigabytes an WAL halten, sind die klassische Ursache voller Platten.
3. Für logische Replikation: `db-logical-replication` (Subscriber-Seite) zeigt DEAKTIVIERTE / NICHT GESTARTETE Abonnements und Anwendungsrückstand >=16 MB.
4. `db-replica-identity`, wenn Subscriber über fehlende Identität klagen.

### 6.4 Sicherheitsprüfung

1. `db-roles-list` für clusterweite Sicherheitsbefunde.
2. `db-role-perms` (geprüfte Rolle wählen) für die Tiefe je Ziel.
3. `db-ownership` für Befunde zum Objekteigentum.
4. Das Audit-Log über Einstellungen -> Menü Ansicht -> Audit-Log exportieren.

### 6.5 Sichere Schemamigration

1. Zwei Datenbanken über Server-Ansicht -> Schemavergleich vergleichen.
2. Das erzeugte Sync-SQL prüfen.
3. Ausgewählte Anweisungen anwenden.

### 6.5.5 Würde dieser Index wirklich helfen?

`db-index-advisor` meldet, welche Tabellen sequenziell gelesen werden. Die
nächste Frage ist immer, ob ein Index auf *dieser* Spalte das behebt, und der
einzige ehrliche Weg zur Antwort war bisher, einen zu bauen - eine lange
Schreibsperre für eine Antwort, die oft „der Planer hätte ihn ohnehin nicht
benutzt" lautet.

`EXPLAIN` beantwortet sie auch nicht. Der Planer bewertet die Indizes, die es
gibt; ohne den Index gibt es keinen Plan mit ihm, und `EXPLAIN ANALYZE`
braucht ihn aus demselben Grund wirklich.

Unter der Tabelle des Advisors gibt es ein Feld, das die Frage ohne Bauen
beantwortet. Tabelle wählen, den erwogenen Index schreiben, die betreffende
Abfrage einfügen und **Compare plans** drücken. Es meldet:

- ob der Planer den Index überhaupt benutzen würde - die häufige Antwort ist
  nein, und das ist es wert, vor einem 4-GB-Index zu wissen;
- die geschätzten Kosten mit und ohne ihn, und den Unterschied;
- ungefähr, wie viel Platz der Index belegen würde.

Eine Zeit meldet es bewusst nicht. Es wird nichts ausgeführt, also gibt es
keine Messung, und eine Schätzung als Messung auszugeben wäre schlimmer, als
gar nichts zu geben.

Dafür wird die Erweiterung `hypopg` gebraucht, die den Index nur im Kopf des
Planers anlegt - nichts wird geschrieben, keine Sperre genommen, also ist es
auch auf Produktion unbedenklich. Auf RDS und Aurora ist sie verfügbar,
selbst gehostet kommt sie als eigenes Paket. Fehlt die Erweiterung, sagt das
Feld es - und ob dieser Server sie haben könnte.

### 6.6 Lastanalyse im Oracle-Stil (AWR + ASH + Trace) {#oracle-style-workload-analysis-awr-ash-trace}

#### Wann es läuft: der Erfassungsplan {#capture-plan}

Die Tabs AWR, ASH und Sitzungs-Trace starten gleich: eine Zeile **Wann** unter der Kopfzeile.

- **Snapshot jetzt** (nur AWR): ein Snapshot, mit einem optionalen Kommentar, der in jeder Liste daneben steht.
- **Für eine Weile**: beginnt jetzt und hält von selbst an. Neben schnellen Längen steht ein Rad aus Tagen, Stunden, Minuten und Sekunden, sodass jede Länge in den Grenzen des Werkzeugs wählbar ist: AWR 10 Minuten bis 92 Tage, ASH 1 Minute bis 31 Tage, der Trace 10 Sekunden bis 30 Minuten.
- **Geplant**: von einem Datum und einer Uhrzeit bis zu einer anderen, gewählt mit der Datums- und Zeitauswahl nach der Uhr dieses Computers, auf Wunsch **jeden Tag**. Ein Trace wird mit einem Beginn und einer Länge geplant, bis zu einen Tag im Voraus.
- **Bis zum Stopp**: was Planer und Sampler bisher taten (nicht für den Trace).

**Alle** sagt, wie oft das Werkzeug sammelt, mit eigenen schnellen Werten und demselben Rad: AWR ein Snapshot alle 1 Minute bis 1 Tag, ASH eine Probe alle 1 Sekunde bis 1 Stunde, der Trace ein Blick alle 0,1 Sekunden bis 1 Minute. **Behalten** sagt, wie viele Tage die gespeicherten Snapshots oder Tage bleiben. Bei AWR startet **auch die Sitzungen alle 5 s abtasten** den ASH-Sampler für dasselbe Fenster; bei einem Plan bis zu einer Stunde gibt es stattdessen den Workload-Bericht (ein Snapshot an jedem Ende, dazwischen die Sitzungen abgetastet), früher die Ansicht Capture.

Über der Schaltfläche sagt ein Satz, was geschehen wird, wie viele Snapshots oder Proben das sind, was es auf der Platte und auf dem Server kostet und was am Ende aufgeht; er warnt bei einer schweren Wahl (ein AWR-Snapshot häufiger als alle 5 Minuten). Während ein Plan läuft, schrumpft das Feld zu einer Leiste: was läuft, seit und bis wann, ein Fortschrittsbalken, die Restzeit, **Ändern** und **Stopp**; ein geplanter Plan sagt, wann er beginnt, und bietet **Stattdessen jetzt starten**. Ist ein Plan mit Ende vorbei, öffnet sich der Bericht seines Zeitraums von selbst: bei AWR sein erster gegen seinen letzten Snapshot, bei ASH der Bericht seines Fensters.

Bei AWR steht der **Kommentar** neben der Zeile Wann für jeden Modus: er kommt an den einen Snapshot von *Snapshot jetzt* und an jeden Snapshot eines Plans. Die Datums- und Zeitauswahl von *Geplant* hat schnelle Zeiten voraus (in 1 h, heute Nacht 02:00, morgen 08:00, Montag 00:00) und Räder für Stunde und Minute; der Pfeil am Ende der Zeile Wann klappt das Feld auf kleinen Bildschirmen auf eine Zeile ein.

**Stopp** beendet einen Plan früher und wirkt wie sein eigenes Ende: das Gesammelte bleibt, AWR nimmt in dem Moment einen letzten Snapshot (mit dem Kommentar des Plans), und der Bericht des Zeitraums öffnet sich - der erste Snapshot des Laufs gegen diesen letzten, bei ASH die abgetastete Zeit. Ein geplanter Plan, der noch nicht begonnen hat, wird nur abgebrochen.

Pläne laufen, solange DBApguin geöffnet ist: auf dem Server bleibt nichts zurück und nichts läuft von selbst. Ein Plan mitten in seinem Fenster läuft beim nächsten Öffnen des Servers für den Rest weiter, ein täglicher wartet auf sein nächstes Fenster, und einer, dessen Fenster bei geschlossener App verstrichen ist, ist vorbei. Ein geplanter Trace lebt nur, solange die App geöffnet ist.

Auf einem Mac tickt das Trackpad, während die Räder Werte passieren, wie bei einer Auswahl auf dem Telefon.

1. **Einrichtung einmal prüfen**: `db-setup-advisor` ausführen. Melden `pg_stat_statements`, `auto_explain` oder `compute_query_id` FEHLT, folgen Sie der ausgegebenen Abhilfe. Das tun Sie einmal je Cluster.

2. **Eine langsame Phase aufnehmen**: klagt jemand über Langsamkeit, `db-workload-report` über 5 Minuten laufen lassen. Die Top-SQL-Abschnitte zeigen, was im Fenster wirklich Zeit verbraucht hat.

3. **Dauerhafte Observability**: wenn Sie Stunden an Sitzungshistorie wollen (nicht nur eine Fünf-Minuten-Aufnahme), starten Sie den ASH-Sampler:
   - `db-ash` action=start
   - im Hintergrund laufen lassen (er läuft, bis Sie action=stop wählen oder die Anwendung schließen)
   - später `db-ash` action=report für einen beliebigen UTC-Datumsbereich

4. **Vergleich vor und nach einem Deploy**: einen Snapshot vor dem Deploy nehmen, einen weiteren, wenn sich die Last beruhigt hat, dann vergleichen:
   - `db-awr-snapshot` action=take, comment="vor Deploy v24.07"
   - (Deploy)
   - `db-awr-snapshot` action=take, comment="nach Deploy v24.07"
   - `db-awr-snapshot` action=list (beide ankreuzen, dann **Die zwei gewählten vergleichen**)
   - `db-awr-snapshot` action=compare, snap_id_a=<vorher>, snap_id_b=<nachher>

5. **Tiefes Tracing einer Anwendung**: ist eine Anwendung mit Verbindungspool langsam und Sie verdächtigen eine bestimmte Abfrage:
   - `db-trace-session` target_mode=application_name, target_value=`myapp-worker`, duration_seconds=120, category_filter=`select`, query_filter=`reports`
   - In einer parallelen psql-Sitzung als dieser Benutzer: `SET auto_explain.log_min_duration = 0; SET auto_explain.log_analyze = on;` dann die Last auslösen
   - Der Trace-Bericht zeigt Abfrageübergänge und Wait-Events für jedes passende Backend; die Pläne fängt das Server-Log

#### Einen ASH-Bericht lesen {#reading-an-ash-report}

`db-ash` action=report macht aus den Stichproben des gewählten Fensters sechs Abschnitte. Die ersten zwei sind die eigene Lesart des Berichts; die Tabellen danach sind die Zahlen.

**Befunde** ist, was auffiel, das Schlimmste zuerst, jeder mit der Anweisung dahinter und denselben Knöpfen, die jede Anweisung hier trägt: Kopieren, in den SQL-Editor laden, dort EXPLAIN, oder **Plan** - der tatsächlich gelaufene Plan, an Ort und Stelle gezeichnet (siehe [Der tatsächlich gelaufene Plan](#the-plan-that-actually-ran)). "In der Zeitleiste zeigen" hebt die Zeile der Sitzung im nächsten Abschnitt hervor.

| Befund | Löst aus, wenn | Was tun |
| --- | --- | --- |
| Blockierung: Sitzung N | eine Sitzung andere 3 Stichproben oder länger aufhielt; kritisch ab einer verlorenen Minute oder fünf aufgehaltenen Sitzungen | Der Text sagt, ob der Blockierer untätig in einer Transaktion war (die Anwendung vergaß den Commit) oder eine Anweisung ausführte (ein langes Schreiben). "Eine Kette" heißt, der Blockierer wartete selbst auf einen anderen. |
| Eine Anweisung nahm X% der aktiven Zeit | eine Anweisung hat 20% oder mehr der laufenden oder wartenden Stichproben, und es gab andere Anweisungen zum Vergleich | Der Befund sagt, wohin ihre Zeit ging: meist Sperren, meist Platte oder meist CPU - jedes zeigt auf eine andere Lösung. |
| X% der aktiven Zeit waren Sperrwartezeiten | 20% oder mehr der aktiven Stichproben waren `Lock`-Wartezeiten | Der Abschnitt mit den Blockierungspaaren nennt die Sitzungen. |
| X% Warten auf die Platte | 30% oder mehr waren `IO`-Wartezeiten | Ein Arbeitsset größer als `shared_buffers`, oder ein Scan, der den Cache leert; die Top-Abfragen sagen, welche Anweisungen. |
| Interne Konkurrenz (LWLock) | 20% oder mehr waren `LWLock`-Wartezeiten | `WALWrite` ist die WAL-Platte, `buffer_content` / `buffer_mapping` sind heiße Seiten, `lock_manager` sind sehr viele Sperren je Transaktion. |
| Sitzung bis zu T untätig in einer Transaktion | eine Sitzung saß 5 Minuten oder länger untätig in einer Transaktion | Sie hält ihre Sperren und bremst Vacuum für den ganzen Server; kritisch, wenn sie dabei auch jemanden blockierte. `idle_in_transaction_session_timeout` beendet solche Sitzungen. |
| Anweisungen wurden abgeschnitten | der Text einer abgetasteten Anweisung erreichte `track_activity_query_size` | Aus einem abgeschnittenen Text lässt sich kein Plan machen; der Abschnitt mit den Einstellungen am Ende sagt, was zu erhöhen ist. |

**Zeitleiste** ist das Fenster als Raster: eine Zeile je Sitzung (die vierzig geschäftigsten, der Rest in einer Zeile "weitere" summiert), eine Spalte je Zeitscheibe - eine Sekunde bei einem kurzen Fenster, eine Minute oder mehr bei einem langen, im Titel genannt. Jede Zelle trägt die Farbe dessen, was die Sitzung in dieser Scheibe am meisten tat:

| Farbe | Bedeutung |
| --- | --- |
| grün | läuft auf der CPU (aktiv, kein Warteereignis) |
| rot | wartet auf eine Sperre, die eine andere Sitzung hält |
| gelb | wartet auf die Platte (`IO`-Warteereignisse) |
| blau | anderes Warten: `LWLock`, `IPC`, Client, Timeout |
| violett | untätig in einer Transaktion - arbeitet nicht, hält aber ihre Sperren |
| leer | nicht in den Stichproben: untätig oder nicht verbunden |

Eine blasse Zelle ist eine, in der die Sitzung nur einen Teil der Scheibe war. **Eine Zelle anklicken** zeigt die Anweisungen dieser Sitzung in dieser Scheibe, mit dem Warteereignis, den pids, die sie blockierten (jede ein Knopf, der zu ihrer Zeile springt) und den Knöpfen der Anweisung. **Das Label einer Sitzung anklicken** zeigt alles, was sie im Fenster tat: ihre Farbaufteilung und ihre Anweisungen summiert. **Eine Zeit oben anklicken** zeigt jede Sitzung in dieser Spalte. Die pids sind aus der Vergangenheit: ein Backend ist vielleicht weg oder inzwischen eine andere Sitzung, daher bietet die Zeitleiste kein Abbrechen oder Beenden.

**Zustandshistogramm**, **Warteereignisse** und **Top-Sitzungen** sind die Zahlen; die Top-Sitzungen tragen jetzt auch je Sitzung ihre Anteile an Sperren, Platte und Untätigkeit in einer Transaktion.

**Top-Abfragen** sind Karten, eine je Anweisung, nach Stichproben geordnet: ihr Anteil an der aktiven Zeit, ein Balken mit ihrer Farbaufteilung (wohin ihre Zeit ging), wie viele Sitzungen sie ausführten und aus welchen Anwendungen, der längste einzelne Lauf, ihr häufigstes Warteereignis und - wenn `pg_stat_statements` auf dem Server ist - ihre Aufrufe, mittlere und gesamte Zeit seit dem Zurücksetzen der Zähler. Die Knöpfe sind dieselben wie bei den Befunden; eine bei `track_activity_query_size` abgeschnittene Anweisung hat keinen Plan-Knopf, weil der Planer nur den Schnitt melden würde.

**Blockierungspaare** nennen beide Seiten: den Wartenden und den Blockierer mit ihren Anwendungen, was der Blockierer tat, während er blockierte (`idle-txn` ist der, den man ansehen sollte), wann das Paar zuerst und zuletzt gesehen wurde, die Sperre, auf die der Wartende aus war, und beide Anweisungen vollständig.

**Einstellungen für eine bessere Analyse** schließen den Bericht ab (und die Status-Aktion zeigt sie ebenfalls, bevor eine einzige Stichprobe genommen ist): die Servereinstellungen, die entscheiden, was die Stichproben sagen können, jede mit ihrem aktuellen Wert, dem empfohlenen, ob sie ein Neuladen, einen Neustart oder ein CREATE EXTENSION braucht, und warum. Warnungen sind Einstellungen, ohne die dieser Bericht für etwas blind ist - `compute_query_id` (keine Anweisungsidentität in den Stichproben), `track_activity_query_size` (abgeschnittene Anweisungen), `pg_stat_statements` (keine Zähler je Anweisung), `track_io_timing` (Plattenzeit unsichtbar), `log_lock_waits` (kein Eintrag für Wartezeiten kürzer als eine Stichprobe); der Rest ist nützlich (`auto_explain`, der einzige Weg, den Plan zu sehen, der *tatsächlich lief*, `log_min_duration_statement`, `log_temp_files`, `track_functions`). **Anwenden** führt `ALTER SYSTEM SET` und ein Neuladen nach der üblichen Bestätigung aus; eine Einstellung, die einen Neustart braucht, ist so markiert und wartet auf den nächsten. Ohne Superuser stehen die Anweisungen zum Kopieren da. Sie listen auch alles, was der tatsächlich gelaufene Plan braucht - auto_explain und wie es schreibt, Log-Collector und Logziel, das Recht, das Log zu lesen (ein GRANT, hier angewendet, wenn diese Anmeldung es erteilen darf) und pg_store_plans (die Paketinstallation wird als Befehl zum Kopieren gezeigt) - jedes mit seinen Kosten; siehe [Der tatsächlich gelaufene Plan](#the-plan-that-actually-ran).

**Bevor ASH oder AWR zu erfassen beginnt** (ASH start, AWR take und auto-start), fragt die App den Server nach derselben Liste. Fehlt etwas, das sie lesen, erscheint zuerst ein Fenster: die Erweiterungen (`pg_stat_statements` - in dieser Datenbank angelegt? in `shared_preload_libraries`? - und `auto_explain`), dann jede Einstellung, die nicht passt, jeweils mit aktuellem Wert, empfohlenem Wert, ob sie ein Reload, einen Neustart oder CREATE EXTENSION braucht, und warum. Was den Bericht blind macht, ist angehakt; das Wünschenswerte haken Sie selbst an. **Ausgewählte aktivieren** führt genau die unter der Liste gezeigten Anweisungen aus (zwei Zeilen, die zu `shared_preload_libraries` hinzufügen, erscheinen als die eine Anweisung, in der sie enden), liest den Server erneut und sagt, was jetzt auf einen Neustart wartet - die App startet den Server nie neu. Eine Zeile, die der Klick eingerichtet hat, bleibt mit ihrem Ergebnis in der Liste, bis **Erneut prüfen**. **Erweiterungen…** schließt das Fenster und öffnet die Erweiterungsverwaltung; **Trotzdem starten** läuft mit dem, was da ist. Jeder Tab fragt einmal; fehlt nichts mehr, erscheint das Fenster nicht mehr von selbst, und **Einstellungen prüfen...** neben Start im ASH- und AWR-Plan zeigt es jederzeit, mit dem Wünschenswerten (etwa den eigenen Einstellungen von `auto_explain`, die es erst nach dem Neustart gibt, der es lädt); sein Start startet den Plan. Ein Tab neu ausführen (sein Kreispfeil) liest die Ansicht des Tabs neu; es startet, stoppt oder erstellt nie etwas. Ohne Superuser gibt es die Anweisungen zum Kopieren. Das Fenster lässt sich in den Einstellungen (*Vor dem Start von ASH / AWR prüfen*) oder mit seinem eigenen Kontrollkästchen abschalten.

#### Abfrageverlauf-Suche {#query-history-search}

`db-query-history` ist eine Suche über `pg_stat_statements`, abgeglichen mit dem ASH-Sampler für eine Zeitangabe. Ihre Parameter laufen VOR dem Check, im selben Parameterdialog, den jeder parametrisierte Check verwendet:

| Feld | Macht |
| --- | --- |
| Statement contains | Groß-/Kleinschreibung-unabhängiger Teilstring-Abgleich gegen den normalisierten Anweisungstext (Literale entfernt - ein Tabellen- oder Spaltenname ist meist am nützlichsten einzugeben). Leer entspricht allem. |
| Run by | Nur Anweisungen, die von dieser Rolle ausgeführt wurden. |
| Worst by | Nach welcher Rangfolge sortiert wird - Gesamtzeit (womit die Datenbank ihr Leben verbringt), mittlere Zeit (was pro Lauf langsam ist), schlechtester Einzellauf, Aufrufanzahl oder zurückgegebene Zeilen. Gesamtzeit und mittlere Zeit beantworten unterschiedliche Fragen: eine Abfrage, die 2 ms eine Million Mal läuft, übertrifft eine, die eine Minute zweimal läuft. |
| Ignore faster than (ms) | Verwirft Zeilen, deren mittlere Zeit darunter liegt, damit eine Seite schnellen Rauschens nicht die eine langsame Form begräbt, die es wert ist, gelesen zu werden. |
| How many | Zeilenobergrenze, 1-200. |
| Include plans | EXPLAIN des normalisierten Texts für die Top 5 Zeilen - der Plan für die Form, kein Datensatz des tatsächlich verwendeten Plans (siehe Vorbehalt unten). |

**Diese Suche läuft jedes Mal, wenn Sie den Dialog öffnen und auf Run check klicken, gegen den vollständigen serverseitigen Verlauf** - nicht nur das, was gerade auf dem Bildschirm steht. Die Ergebnistabelle erhält danach dasselbe Spaltenfilterfeld, das jedes Check-Ergebnis erhält (Eingabe in einer Spaltenüberschrift grenzt die bereits angezeigten Zeilen ein); dieses Filterfeld grenzt nur ein, was diese Suche bereits zurückgegeben hat, es durchsucht nicht erneut den gesamten Bestand. Um weiter zurückzusuchen oder nach einem anderen Kriterium, ändern Sie die Felder oben und führen den Check erneut aus.

Ergebnisspalten:

| Spalte | Was es ist | Wie zu lesen |
| --- | --- | --- |
| `user` | Rolle, die die Anweisungsform ausgeführt hat | - |
| `calls` | Ausführungen seit dem letzten Zurücksetzen der Zähler | - |
| `total` / `mean` / `max` | Kumulative / durchschnittliche / schlechteste Einzelausführungszeit | Gleiche Lesart wie `total_ms`/`mean_ms` im Workload-Bericht oben. |
| `rows` | Insgesamt zurückgegebene oder betroffene Zeilen über alle Aufrufe | - |
| `ash_samples` | Wie viele ASH-Samples die `queryid` dieser Anweisung laufend erfasst haben | 0 bedeutet, der ASH-Sampler lief nicht, während dies lief, nicht dass es nie lief - `calls` ist die Zahl, die wirklich zählt. Braucht zuerst gestartetes `db-ash`; PG13 hat gar keine `queryid` in ASH. |
| `last_seen` | Zeitstempel des jüngsten ASH-Samples, das es erfasst hat | Leer aus demselben Grund wie `ash_samples` = 0. |
| `query` | Der normalisierte Anweisungstext, gekürzt | Literale sind zu Platzhaltern entfernt (`$1`, `$2`, ...) - eine Zeile deckt jede Ausführung ab, die sich nur in ihren Werten unterschied. |

Zähler setzen sich zurück mit `pg_stat_statements_reset()` oder einem Absturz; ein sauberer Server-Neustart behält sie, wenn `pg_stat_statements.save` an ist (PostgreSQLs eigener Standard). Eine Anweisung von vor dem letzten Reset ist einfach nicht da - "nichts gefunden" kann das ebenso gut bedeuten wie ein falsches Muster.

#### Einen AWR- / Workload-Bericht lesen {#reading-an-awr-report}

Jede Zahl in diesen Berichten ist ein **Delta über das Fenster** - der Wert zwischen den beiden Snapshots, keine Rate und keine Gesamtsumme seit dem Start. Ein gespeicherter Bericht behält dieses Glossar bei sich (der PDF-Export enthält es), sodass ein Wochen später wieder geöffneter Bericht sich weiterhin selbst erklärt.

**Fensterzusammenfassung**

| Kennzahl | Was es ist | Wie zu lesen |
| --- | --- | --- |
| `window_seconds` / `window_hours` | Reale Zeit zwischen den beiden Snapshots | Alles andere gilt "pro so viel Zeit". Ein 30-Sekunden-Fenster und ein 6-Stunden-Fenster sind nicht vergleichbar. |
| `txn_commit` / `txn_rollback` | Im Fenster committete / zurückgerollte Transaktionen | Zählwerte, keine Raten. Durch `window_seconds` teilen für TPS. |
| `rollback_pct` | `txn_rollback / (txn_commit + txn_rollback)` | Über ~10% bei echtem Verkehr bedeutet, dass die Anwendung Fehler wirft oder in Deadlocks läuft - sehen Sie sich den Errors-Watcher und die Deadlock-Zahl an. Ein paar Rollbacks in einem ruhigen Fenster sind Rauschen. |
| `cache_hit_pct` | `blks_hit / (blks_hit + blks_read)` - Anteil der Blocklesevorgänge aus den **shared_buffers**, nicht von der Platte | Das ist der PostgreSQL-Puffercache, nicht der OS-Seitencache, also kann eine "niedrige" Zahl trotzdem schnell sein, wenn das OS gecacht hat. Unter ~95% bei nennenswertem Verkehr heißt meist, dass `shared_buffers` für das Working Set zu klein ist, oder ein großer Sequential Scan den Cache im Fenster geleert hat. |
| `deadlocks` | Im Fenster erkannte Deadlocks | Jeder Wert über 0 ist ein Fehler in der Sperrreihenfolge der Anwendung. PostgreSQL hat ihn durch Abbruch einer Transaktion aufgelöst; das Paar steht im Server-Log. |
| `temp_files` / `temp_bytes` | Dateien (und Gesamtgröße), die PostgreSQL für Sortierungen und Hashes auf die Platte ausgelagert hat | Ungleich null heißt, dass die Sortierung oder der Hash einer Abfrage nicht in `work_mem` passte. Mit Top SQL nach Temp-Schreibvorgängen abgleichen. |
| `new_sessions` / `sessions_per_s` | Im Fenster begonnene Sitzungen (`pg_stat_database.sessions`, PostgreSQL 14+), und pro Sekunde | Viele kurze Sitzungen sind Verbindungswechsel: eine Anwendung ohne Verbindungspool, oder ein Pool, der Verbindungen öffnet und schließt. Vor PostgreSQL 14 sagt die Zusammenfassung, dass es sich nicht messen lässt. |
| `mean_session` | Im Fenster aufgelaufene Sitzungszeit geteilt durch die begonnenen Sitzungen | Langlebige (gepoolte) Verbindungen tragen ihre Zeit ebenfalls bei, daher wirken die kurzen Sitzungen länger, als sie sind: eine kleine Zahl ist ein starkes Zeichen für Verbindungswechsel, eine große beweist nichts. |

**Top SQL** (vier Rangfolgen derselben Zeilen: nach Gesamtzeit, nach Aufrufzahl, nach Plattenlesevorgängen, nach Temp-Schreibvorgängen)

| Spalte | Was es ist | Wie zu lesen |
| --- | --- | --- |
| `calls` | Ausführungen dieser normalisierten Anweisung im Fenster | - |
| `total_ms` | Summe der Ausführungszeit über alle `calls` (das Delta von `total_exec_time`) | Das ist die wichtigste Rangfolge: die Abfrage mit dem höchsten `total_ms` hat die meiste Serverzeit im Fenster verbraucht, unabhängig davon, ob sie einzeln langsam ist. |
| `mean_ms` | `total_ms / calls` - Durchschnittszeit einer Ausführung | **Ein großer `mean_ms`** (etwa > 100 ms) heißt, jeder Lauf ist langsam: ein fehlender Index, ein schlechter Plan, Lock-Waits oder eine wirklich große Abfrage - EXPLAIN (ANALYZE, BUFFERS). **Ein kleiner `mean_ms` bei großem `total_ms`** heißt, die Abfrage ist pro Aufruf billig, läuft aber so oft, dass sie dominiert - die Lösung ist, sie seltener aufzurufen (Ergebnis cachen, Arbeit bündeln) oder die Kosten pro Aufruf zu senken, nicht den Plan zu tunen. |
| `rows` | Insgesamt zurückgegebene oder betroffene Zeilen über alle Aufrufe | `rows / calls` ist die durchschnittliche Ergebnisgröße; eine Bericht-Abfrage, die 100k Zeilen pro Aufruf zieht, ist ein anderes Problem als eine OLTP-Abfrage, die 3 zieht. |
| `shared_read` | 8-KB-Blöcke, die über alle Aufrufe **von der Platte** gelesen wurden (nicht aus dem Cache) | Hoch heißt, die Daten dieser Abfrage sind nicht in `shared_buffers` - kalter Cache oder ein Working Set größer als der Cache. `shared_read x 8192` ist das Byte-Volumen. |
| `temp_written` | 8-KB-Blöcke, die über alle Aufrufe in **Temp-Dateien** geschrieben wurden | Ungleich null heißt, die Sortierung oder der Hash ist ausgelagert. Erhöhen Sie `work_mem` für die ausführende Sitzung oder korrigieren Sie den Plan, damit weniger Zeilen sortiert werden. `temp_written x 8192` ist das Byte-Volumen. |
| `qid` | Die `queryid` (Fingerabdruck der normalisierten Anweisung) | Gleicher Wert über Snapshots und in `pg_stat_statements`; damit eine Anweisung über die Zeit verfolgen. |

**Top-Tabellen nach DML**

| Spalte | Was es ist | Wie zu lesen |
| --- | --- | --- |
| `ins` / `upd` / `del` | Im Fenster eingefügte / aktualisierte / gelöschte Zeilen | Treibt Autovacuum. Eine Tabelle mit vielen `upd` + `del` und ohne kürzliches Autovacuum ist ein Bloat-Kandidat - siehe [Vacuum, Bloat und Tabellenstatistiken](#vacuum-bloat-and-table-statistics). |
| `hot_upd_%` | HOT-Updates als Anteil aller Updates | HOT-Updates berühren keine Indexe und brauchen keine Index-Bereinigung. **Niedrig** (< ~50%) bei einer stark aktualisierten Tabelle heißt, Updates ändern indizierte Spalten, oder die Seiten haben keinen Platz - senken Sie den `fillfactor` der Tabelle, damit Updates auf der Seite bleiben. |
| `seq_scan` / `idx_scan` | Im Fenster gestartete sequentielle vs. Index-Scans auf der Tabelle | Ein großes `seq_scan`-Delta auf einer großen Tabelle ist ein Signal für einen fehlenden Index - mit dem Index-Advisor bestätigen. |

**Hintergrund- / Checkpoint-Aktivität**

| Spalte | Was es ist | Wie zu lesen |
| --- | --- | --- |
| `ckpt_timed` / `ckpt_req` | Durch Zeitplan vs. durch WAL-Volumen ausgelöste Checkpoints | Überwiegend `req` heißt, Checkpoints werden vor `checkpoint_timeout` durch Schreibvolumen erzwungen - erhöhen Sie `max_wal_size`, damit sie sich verteilen. |
| `ckpt_write_ms` / `ckpt_sync_ms` | Zeit für das Schreiben / fsync von Dirty Buffers während Checkpoints | Ein großer `sync_ms` heißt, der Speicher kann den Schreib-Burst des Checkpoints nicht aufnehmen. |
| `buffers_backend` | Dirty Buffers, die **Backends** selbst schreiben mussten | Nichttriviale Werte heißen, Background Writer und Checkpointer sind zurückgefallen und die Abfrage-Backends haben die Schreibkosten inline bezahlt. |

**Befunde** ist die eigene Lesart des Fensters, das Schlimmste zuerst, gezeichnet wie die ASH-Befunde: jeder mit der Anweisung dahinter und ihren Knöpfen. Über die obigen Schwellen hinaus (Deadlocks, Rollback-Anteil, Cache-Treffer) nennt er eine Anweisung, die 30% oder mehr der gesamten Ausführungszeit nahm, Anweisungen mit im Mittel über 500 ms je Aufruf, Anweisungen, die 64 MB oder mehr in Temp-Dateien auslagerten (`work_mem`), Checkpoints, die öfter durch WAL-Volumen als durch die Uhr erzwungen wurden (`max_wal_size`), Backends, die ein Fünftel der schmutzigen Puffer oder mehr selbst schrieben (`bgwriter_*`), und eine stark aktualisierte Tabelle mit weniger als der Hälfte HOT-Updates (`fillfactor`). Jede Anweisungsschwelle hier verlangt, dass das Fenster insgesamt mindestens zehn Sekunden Ausführungszeit enthält, damit eine ruhige Testdatenbank nicht wie ein Server in Not aussieht. Vier Befunde werden stattdessen nach ihrem eigenen Volumen beurteilt - eine reine N+1-Last, ein schneller Lookup hunderttausendfach ausgeführt ohne etwas Langsames daneben, erreicht nie zehn Sekunden und hieß früher "In diesem Fenster fiel nichts auf":

- **1+N-Muster**: eine Anweisung (B) läuft k Mal für jeden Lauf einer anderen (A), in derselben Datenbank als derselbe Benutzer, mit k mindestens 5 und B mindestens 1.000 Mal ausgeführt. Das Verhältnis allein könnte Zufall sein, daher braucht es zusätzlich einen Beleg: A lieferte (auf 5% genau) so viele Zeilen, wie B lief - einmal pro Zeile; oder das Verhältnis blieb in mindestens drei Intervallen zwischen Snapshots innerhalb von 10%, während sich die eigene Rate von A um die Hälfte oder mehr änderte (ein Intervall, in dem A begann oder aufhörte, zählt nicht, und unter gleichmäßiger Last behalten zwei beliebige Anweisungen ihr Verhältnis, was nichts beweist); oder die Aufrufe von B sind auf zwei Aufrufe genau ein ganzes Vielfaches der von A, bei mindestens 500 Läufen von A. Ein gespeicherter Vergleich liest die Snapshots, die zwischen den beiden gewählten genommen wurden (bis zu 24); ein Workload-Bericht zählt die Anweisungen etwa ein Dutzend Mal, während er aufzeichnet. Gelb, wenn B 200 Mal pro Sekunde läuft oder 10% der Ausführungszeit nimmt, darunter blau. Anweisungen, die in einer Funktion laufen, bleiben außen vor - sie kreisen im Server, ohne Roundtrip, der sich sparen ließe. Der Befund sagt, was pg_stat_statements nicht kann: jeder Aufruf kostet die Anwendung zusätzlich einen Roundtrip, meist mehr als die Abfrage selbst.
- **Eine Anweisung unter einer Millisekunde, sehr oft ausgeführt**: mindestens 10.000 Aufrufe unter je 1 ms UND entweder 10% der gesamten Ausführungszeit (der Anteil) oder 200 Aufrufe pro Sekunde (die Rate), was zuerst eintritt; eine Anweisung, die schon in einem 1+N-Befund steht, wird nicht wiederholt.
- **Verbindungswechsel** (PostgreSQL 14+): gelb ab 20 neuen Sitzungen pro Sekunde, blau ab einer neuen Sitzung auf je zwei Transaktionen, beides mit mindestens 200 neuen Sitzungen im Fenster. Er nennt die Datenbanken und fügt die Sitzungen hinzu, die der Client ohne Schließen abbrach, die mit einem fatalen Fehler endeten oder beendet wurden.
- **Optimistisches Sperren**: ein UPDATE oder DELETE, dessen WHERE eine Versionsspalte mit einem Parameter vergleicht und das bei einigen Aufrufen keine Zeile schrieb - diese verloren das Rennen gegen einen anderen Schreiber und wurden wiederholt. Gelb, wenn mindestens 5% von mindestens 100 Aufrufen, oder 1.000 Aufrufe, nichts schrieben; siehe [Verbindungen, Sitzungen und Last](#connections-sessions-and-load).

**Top SQL zum Ansehen** sind Karten für die Anweisungen, die die Befunde nannten, geordnet nach ihrem Anteil an der gesamten Ausführungszeit, jede mit den Gründen als Marken und denselben Knöpfen: Kopieren, in den SQL-Editor laden, dort EXPLAIN, oder **Plan** - der Plan an Ort und Stelle gezeichnet. Er zeigt den tatsächlich gelaufenen Plan, aus `auto_explain` im Serverlog oder aus `pg_store_plans`, und den Plan, den der Server *jetzt* wählen würde, nur wenn keiner von beiden ihn hat, so gekennzeichnet - siehe [Der tatsächlich gelaufene Plan](#the-plan-that-actually-ran).

**Einstellungen für eine bessere Analyse** schließen einen Vergleich, eine Aufnahme und eine Live-Aufzeichnung ab: dieselbe Tabelle, mit der der ASH-Bericht endet - siehe [Einen ASH-Bericht lesen](#reading-an-ash-report). Sie wird vom verbundenen Server gelesen, also bekommt jeder Cluster eigene Vorschläge für seine Version: `compute_query_id` und `track_wal_io_timing` ab PostgreSQL 14, `pg_stat_statements.track_planning` ab 13, und es wird nichts vorgeschlagen, was die Version des Servers nicht hat.

**Was nur der Workload-Bericht hat**, aus seinen eigenen Stichproben von pg_stat_activity (der ganze Server, alle paar Sekunden):

- **Aktive Sitzungen im Zeitverlauf** ist ein Diagramm mit einem Punkt pro Stichprobe: alle aktiven Sitzungen und davon wie viele liefen, auf eine Sperre, auf die Platte oder auf etwas anderes warteten, neben den Sitzungen, die untätig in einer Transaktion saßen (sie halten ihre Sperren). Die Farben sind die der ASH-Zeitleiste, eine Sperre ist also in beiden rot. Ein langes Fenster wird als Linien ohne Punkte gezeichnet; der Export schreibt jede Stichprobe als Zeile, und das PDF mittelt sie auf höchstens 60 Zeilen, jede mit ihrer Zeitspanne.
- **Wait-Events** zeigen, wo abgetastete Backends gewartet haben: `Lock` (durch eine andere Transaktion blockiert), `IO` (wartet auf Platte), `LWLock` (interne Contention), `Client` (wartet auf den Client - meist nur inaktive Verbindungen, kein Problem). Ein von `Client`-Waits dominiertes Fenster ist ein gesundes inaktives System.
- **Beobachtete blockierende Paare** listet, wer auf wen wartete, in wie vielen Stichproben, mit beiden Anweisungen; **Längste gesehene offene Transaktion** ist die älteste Transaktion, die eine Stichprobe erfasst hat.
- Ihre Befunde stehen bei den anderen: Sitzungen, die auf die Sperre einer anderen Sitzung warteten (die Anweisung, die sie hielt, wird gezeigt, die wartende daneben); ein Fünftel oder mehr der aktiven Zeit mit Warten auf Sperren UND mindestens 5 solche Stichproben; Sitzungen untätig in einer Transaktion in einem Fünftel oder mehr der belegten Stichproben UND mindestens 10 davon; eine Transaktion, die eine Minute oder länger offen war.

#### Einen Zeitraumvergleich lesen {#reading-a-period-comparison}

Ein Zeitraumvergleich (`db-awr-snapshot` action=compare-periods, die Ansicht **Vergleich** im AWR-Tab) beantwortet „was hat sich zwischen damals und jetzt geändert" statt „was ist in diesem Fenster passiert". Jede Zahl ist weiterhin eine Differenz über ihr eigenes Fenster; der Bericht stellt die zwei Fenster nebeneinander.

- **Zeiträume im Vergleich**: eine Zeile pro Kennzahl - Fensterlänge, bestätigte Transaktionen insgesamt und pro Sekunde, Rollback-Anteil, Cache-Trefferquote, von der Platte gelesene Blöcke, in temporäre Dateien geschriebene Bytes, Deadlocks, zeitgesteuerte und angeforderte Checkpoints, von Backends geschriebene Puffer, WAL-Bytes, die Ausführungszeit aller Anweisungen und wie viele Anweisungen liefen. **Änderung** ist Zeitraum 2 minus Zeitraum 1, mit einem Pfeil nach oben oder unten, und **Änderung %** ist relativ zu Zeitraum 1 (bei einem Prozentwert ist die Änderung in Prozentpunkten). Rot ist eine Änderung zum Schlechteren, Grün eine zum Besseren; eine Kennzahl, bei der ein Anstieg keines von beiden ist (mehr Transaktionen sind mehr Arbeit, kein schlechterer Server), wird nicht eingefärbt. Gilt der Vergleich einer Datenbank, sind die letzten Zeilen die des ganzen Servers.
- Zählwerte wachsen mit dem Fenster: sind die Zeiträume unterschiedlich lang, vergleichen Sie die Raten pro Sekunde und die Prozente - ein Befund sagt es.
- **Befunde** nennen die Zahlen beider Zeiträume: eine Anweisung, deren Zeit pro Aufruf um die Hälfte oder mehr und um mindestens 10 ms stieg, wenn das Zeitraum 2 mindestens eine Sekunde kostete; eine Anweisung, die in Zeitraum 1 nicht lief und ein Fünftel oder mehr der Ausführungszeit von Zeitraum 2 brauchte (und mindestens eine Sekunde); eine um zwei Punkte oder mehr gesunkene Cache-Trefferquote; mindestens verdoppelte temporäre Dateien von 64 MB oder mehr; Checkpoints, die nur in Zeitraum 2 öfter durch WAL-Volumen als durch die Uhr ausgelöst wurden; ein um fünf Punkte oder mehr gestiegener Rollback-Anteil bei mindestens 50 Transaktionen; neu aufgetretene Deadlocks. Eine wichtige Anweisung, die pro Aufruf mindestens doppelt so schnell wurde und so mindestens eine Sekunde sparte, bekommt eine grüne Zeile. Ein Neustart, ein Zurücksetzen oder eine Verdrängung in einem der Zeiträume wird zuerst genannt, mit dem Zeitraum, in dem es geschah.
- **Anweisungen: Zeitraum 1 gegen Zeitraum 2** listet die 30 Anweisungen, deren Gesamtzeit sich am stärksten änderte, jede mit Aufrufen, Gesamt- und Durchschnittszeit, Zeilen und Plattenlesungen beider Zeiträume, ihrem Status - *beide*, *nur Zeitraum 1* (weg) oder *nur Zeitraum 2* (neu) - und den Schaltflächen, die Top SQL hat (Kopieren, SQL-Editor, EXPLAIN, **Plan**).
- **Tabellen nach DML: Zeitraum 1 gegen Zeitraum 2** listet die 20 Tabellen der verbundenen Datenbank, deren Einfügungen, Änderungen und Löschungen sich am stärksten änderten, mit den sequentiellen und den Index-Scans jedes Zeitraums.

#### Der tatsächlich gelaufene Plan {#the-plan-that-actually-ran}

**Plan** an einer Anweisung im ASH-Bericht, in den Befunden und Karten **Top SQL zum Ansehen** der AWR- und Workload-Berichte und in einem Zeitraumvergleich zeigt den Plan, mit dem die Anweisung *tatsächlich gelaufen ist*, wann immer der Server ihn behalten hat, und sagt deutlich, wenn nicht.

**Warum EXPLAIN jetzt nicht genügt.** Ein nachträgliches EXPLAIN plant die Anweisung neu, heute: mit den heutigen Tabellenstatistiken, den heutigen Einstellungen und Indizes und - bei einer Anweisung mit `$1`-Parametern - einem generischen Plan statt dem, den die echten Werte bekamen. Der Plan, der die Anweisung im Fenster langsam machte, kann ein anderer sein, und EXPLAIN kann das nicht sagen.

**Zwei Quellen behalten den echten Plan:**

- **auto_explain** (ein contrib-Modul, das mit PostgreSQL kommt) schreibt den Plan jeder Anweisung, die langsamer als `auto_explain.log_min_duration` ist, ins Serverlog, mit den tatsächlichen Zeilenzahlen, wenn `auto_explain.log_analyze` an ist. DBApguin liest ihn über dieselbe Datenbankverbindung zurück, nur wenn Sie einen Plan öffnen, und nur einen begrenzten Teil des Logs: die Logdateien, die sich mit dem Fenster des Berichts überschneiden, vom Ende her gelesen, höchstens 32 MB und 8 Sekunden je Blick. Ein geschriebener Plan wird der Anweisung über ihren Query Identifier zugeordnet, wo das Log einen hat, sonst über den Text der Anweisung ohne ihre Konstanten.
- **pg_store_plans** (Drittanbieter, BSD-Lizenz) behält jeden Plan jeder Anweisung im Shared Memory, mit Aufrufen, Gesamtzeit und erstem und letztem Aufruf jedes Plans, unter derselben Query-ID, die `pg_stat_statements` verwendet. Aus einem Log wird nichts gelesen.

**Was die Planansicht zeigt.** Oben drei Knöpfe - **auto_explain (Serverlog)**, **pg_store_plans** und **EXPLAIN jetzt** - jeder mit der Zahl der gefundenen Pläne; sie öffnet sich auf dem ersten, der einen hat, in dieser Reihenfolge. Eine Marke sagt, welche Art Plan Sie sehen: *Der gelaufene Plan* in Grün, oder *Der Plan, den der Server jetzt wählen würde - nicht der, der gelaufen ist* in Gelb, mit einer Zeile dazu, was einzuschalten ist. Lief eine Anweisung im Fenster mit mehr als einem Plan (der Plan hat sich geändert), werden die Pläne vom neuesten an aufgelistet, jeder damit, wann er zuletzt geschrieben wurde (nach der Uhr des Servers, wie Von und Bis des Berichts), seiner Dauer, wie viele Ausführungen mit ihm geschrieben wurden und ihrer kürzesten, mittleren und längsten Zeit; ein Klick zeichnet ihn. Unter dem Baum: ob der Plan über die Query-ID oder über den Text zugeordnet wurde, ob er tatsächliche Zeilen und Zeiten trägt, die Parameterwerte, wenn der Server sie geschrieben hat, und wie viel vom Log gelesen wurde. Eine Quelle ohne Ergebnis sagt, warum (nicht geladen, kein Log-Collector, kein Recht, das Log zu lesen, nichts langsamer als die Schwelle im Fenster, eine verwaltete Plattform). EXPLAIN jetzt funktioniert weiterhin überall und läuft nur, wenn Sie es wählen oder nichts Besseres da ist.

**Einrichten, Schritt für Schritt.** *Einstellungen für eine bessere Analyse* am Ende jedes ASH-, AWR-, Workload- und Trace-Berichts und das Fenster vor dem Start von ASH, AWR oder einem Trace listen jedes Teil mit seinem aktuellen Wert, dem empfohlenen und der Art, wie es gesetzt wird. Was DBApguin selbst kann, hat **Anwenden** (oder wird im Fenster angehakt), mit der üblichen Bestätigung und einem Eintrag im Audit-Log; was das Betriebssystem braucht, zeigt den Befehl zum Kopieren.

1. **auto_explain laden** - in DBApguin: Anwenden fügt `auto_explain` zu `shared_preload_libraries` hinzu. In einem Terminal: den Server neu starten (die Befehle für apt.postgresql.org, PGDG-RPM, `pg_ctl` und einen Container stehen unter der Tabelle; ein Neustart beendet jede Sitzung).
2. **Pläne schreiben** - in DBApguin: `auto_explain.log_min_duration` (zum Beispiel `1s`), `auto_explain.log_format = json` (die Ansicht zeichnet JSON als Baum) und ab PostgreSQL 16 `auto_explain.log_verbose = on` (der Query Identifier im Plan). Das braucht ein Neuladen, keinen Neustart.
3. **Das Log dorthin schreiben, wo SQL es lesen kann** - in DBApguin: `logging_collector = on` (Neustart) und `log_destination` mit hinzugefügtem `jsonlog` ab PostgreSQL 15, davor `csvlog`. stderr wird ebenfalls gelesen, so gut sein `log_line_prefix` es zulässt.
4. **Identität der Anweisung** - in DBApguin: `compute_query_id = on` (PostgreSQL 14+). Optional und empfohlen: `track_io_timing = on`.
5. **Das Recht, das Log zu lesen** - ein Superuser liest es, wie es ist. Für eine Überwachungsanmeldung ist das Geringste, das funktioniert, die Mitgliedschaft in `pg_monitor` (listet das Logverzeichnis) und `EXECUTE` auf `pg_read_file(text, bigint, bigint, boolean)`, das nur im Datenverzeichnis und im Logverzeichnis liest. DBApguin zeigt die beiden GRANTs und bietet Anwenden an, wenn die verbundene Anmeldung sie erteilen darf (beim Server erfragt, nicht aus einem Rollennamen geraten); sonst führt sie ein Superuser aus. Eine Anmeldung, die sie erteilen darf - meist ein Superuser -, bekommt außerdem **Einem Login erteilen...**: es listet die Anmeldungen, die das Log noch nicht lesen können (keine Superuser, die brauchen nichts), zeigt die beiden Anweisungen für die gewählte und führt sie aus, im Audit-Log festgehalten. `pg_read_server_files` oder Superuser würden jede Datei lesen, die der Betriebssystembenutzer des Servers lesen kann - weit mehr, als ein Plan braucht.
6. **Oder pg_store_plans** - in einem Terminal: das Paket installieren (`dnf install pg_store_plans_NN` aus dem PGDG-Repository auf RHEL, Rocky und Fedora; für Debian und Ubuntu gibt es kein apt-Paket, also mit `make USE_PGXS=1` und `make USE_PGXS=1 install` bauen). In DBApguin: Anwenden fügt es zu `shared_preload_libraries` hinzu, ein Neustart lädt es, dann führt **Installieren…** `CREATE EXTENSION pg_store_plans` in jeder Datenbank aus, in der es gelesen werden soll.

**Was es in welcher Version gibt** (geprüft an der PostgreSQL-Dokumentation jeder Version und an Servern von 9.4 bis 19):

| Name | PostgreSQL | Hinweise |
| --- | --- | --- |
| `auto_explain` in `shared_preload_libraries` | alle | Neustart zum Laden |
| `auto_explain.log_min_duration`, `log_analyze`, `log_timing`, `log_buffers`, `log_verbose`, `log_format`, `log_triggers`, `log_nested_statements` | alle | Neuladen; ändert ein Superuser |
| `auto_explain.sample_rate` | 9.6+ | |
| `auto_explain.log_level`, `auto_explain.log_settings` | 12+ | |
| `auto_explain.log_wal` | 13+ | |
| `auto_explain.log_parameter_max_length` | 16+ | Parameterwerte vorbereiteter Anweisungen, standardmäßig vollständig (-1) |
| `auto_explain.log_io`, `auto_explain.log_extension_options` | 19 | |
| `compute_query_id` | 14+ | vor 14 wird ein Plan über seinen Text zugeordnet |
| Query Identifier im geschriebenen Plan (`log_verbose` beachtet `compute_query_id`) | 16+ | auf 14 und 15 steht der Identifier nur in der Spalte `query_id` von csvlog / jsonlog |
| `logging_collector` | alle | Neustart |
| `log_destination = csvlog` | alle | Spalte `query_id` ab 14 |
| `log_destination = jsonlog` | 15+ | ein Eintrag pro Zeile, mit `query_id` |
| `pg_current_logfile()`, `pg_ls_logdir()` | 10+ | `pg_ls_logdir()`: Superuser und `pg_monitor` |
| Rolle `pg_monitor` | 10+ | |
| `pg_read_file(text, bigint, bigint, boolean)` | 9.5+ | `GRANT EXECUTE` ab 11 möglich; davor nur Superuser |
| Rolle `pg_read_server_files` | 11+ | mehr, als ein Plan braucht |
| `pg_store_plans` | Drittanbieter | getestet mit 1.10 auf 17 und 18, wo seine Query-ID die von `pg_stat_statements` ist |

Vor PostgreSQL 10 gibt es kein `pg_ls_logdir()`, dort wird das Log also nicht über SQL gelesen; die Ansicht sagt das und bietet pg_store_plans oder EXPLAIN jetzt an. PostgreSQL 13 ist die älteste Version, die DBApguin vollständig unterstützt.

**Was es kostet** - noch einmal neben jeder Einstellung gesagt, dort, wo Sie sie einschalten:

- **Ein Neustart** für `shared_preload_libraries` (auto_explain, pg_store_plans) und für `logging_collector`. DBApguin startet nie einen Server neu.
- **`auto_explain.log_analyze` instrumentiert jede Anweisung**, ob geschrieben oder nicht, was einen ausgelasteten Server spürbar bremsen kann. Die Uhr zuerst mit `pg_test_timing` messen; `auto_explain.log_timing = off` behält die tatsächlichen Zeilenzahlen (an denen eine Fehlschätzung zu erkennen ist) und spart den Großteil der Kosten; `auto_explain.sample_rate` unter 1 instrumentiert nur einen Teil der Anweisungen, um den Preis, manche langsame zu verpassen.
- **Logvolumen.** Jeder Plan hat viele Zeilen; ein niedriges `log_min_duration` füllt auf einem ausgelasteten Server das Log schnell. Rotation (`log_rotation_age`, `log_rotation_size`) und Platte einplanen. jsonlog oder csvlog ist eine zweite Datei neben stderr.
- **Das Log ist vertraulich.** Es enthält Anweisungstext und ab PostgreSQL 16 Parameterwerte (`auto_explain.log_parameter_max_length = 0` lässt sie weg). Wer das Log lesen darf, liest auch das.
- **Das Log zu lesen braucht Rechte zum Dateilesen.** `EXECUTE` auf `pg_read_file` liest jede Datei im Datenverzeichnis und im Logverzeichnis an allen Rechten innerhalb der Datenbank vorbei; nur einer Überwachungsanmeldung erteilen.
- **pg_store_plans** belegt Shared Memory (`pg_store_plans.max` Pläne) und arbeitet bei jeder Anweisung; die meisten verwalteten Clouds bieten es nicht an.
- **Verwaltete Clouds** (RDS, Aurora, Cloud SQL, Azure und andere) lassen SQL das Serverlog nicht lesen: Das Log liegt in der Konsole des Anbieters, die Ansicht sagt das und weicht auf pg_store_plans aus, wo angeboten, dann auf EXPLAIN jetzt.

DBApguin liest das Log nur über Ihre Verbindung, behält das Gelesene im Speicher, solange der Plan offen ist, und schickt nichts irgendwohin - siehe den [Datenschutzhinweis](PRIVACY.md).

#### Umfang: Ihre Datenbank, mit dem Server daneben {#awr-ash-scope}

ASH und AWR erfassen den ganzen Server, wie die von Oracle die Instanz: der Sampler liest jede Sitzung jeder Datenbank, und jeder Snapshot enthält `pg_stat_statements` und `pg_stat_database` für alle. Beide laufen im Hintergrund weiter, wenn Sie zu einer anderen Verbindung oder einem anderen Tab wechseln; die Liste Background jobs zeigt sie.

Ein Bericht sagt zuerst, woher seine Daten stammen (**Collected for**: Server und die Datenbank, aus der Sampler oder Snapshots gestartet wurden), und dann, worum es geht. Standardmäßig ist das dieselbe Datenbank: Top SQL, Wartezeiten, Befunde und Karten gehören zu ihr, und daneben steht der ganze Server - eine zweite Zeile in der AWR-**Window summary**, **Activity by database** für jede Datenbank und ein Befund **Elsewhere on the server**, wenn eine andere Datenbank etwas Erwähnenswertes getan hat (Deadlocks, 50 oder mehr Rollbacks, ein Fünftel der Ausführungszeit UND mindestens 10 s; bei ASH ein Fünftel der aktiven Samples UND mindestens 60 davon, oder 10 oder mehr Samples mit Warten auf Sperren). `*` im Feld **Database** zeigt den ganzen Server, der Name einer anderen Datenbank diese mit demselben Überblick. Die Wahl fällt beim Lesen, nicht beim Erfassen, daher dienen dieselben Snapshots und Samples allen dreien. **Top tables by DML** stammt aus der verbundenen Datenbank (PostgreSQL führt diese Zähler je Datenbank) und erscheint nur, wenn der Bericht diese Datenbank betrifft. **Plan** fragt den Plan in der Datenbank an, in der die Anweisung lief. Samples und Snapshots von vor dem 22.-23. Oktober 2026 erfassen keine Datenbank je Anweisung; für sie zeigt der Bericht den ganzen Server und sagt es.

**pg_stat_statements was full and evicted statements** in einem Vergleich heißt, dass in AWRs Top SQL fehlt, was diese Anweisungen gekostet haben: `pg_stat_statements` behält `pg_stat_statements.max` Anweisungen und verwirft jedes Mal, wenn es voll ist, die am wenigsten genutzten 5% (mindestens 10). Ab PostgreSQL 14 werden diese Runden gezählt; der Befund nennt sie, eine Schätzung der verlorenen Anweisungen und den Füllstand. Erhöhen Sie `pg_stat_statements.max` (Neustart) - die Einstellungstabelle am Ende hat die Zeile. Eine Anweisung, deren Zähler zurückgingen (verdrängt und zurück, oder ein Reset), zählt ab ihrer Rückkehr.

#### Trend: jede Momentaufnahme über die Zeit gezeichnet {#awr-trend}

`db-awr-snapshot` action=trend verwandelt den gesamten Snapshot-Verlauf für einen Server in Diagramme, statt der Ein-Paar-auf-einmal-Ansicht, die `compare` liefert. Braucht mindestens zwei Momentaufnahmen - nehmen Sie sie manuell, oder lassen Sie den Auto-Scheduler im Hintergrund Verlauf aufbauen.

Zwei verschiedene Arten von Zahlen erscheinen in diesen Diagrammen, und sie werden absichtlich unterschiedlich gezeichnet:

- **Momentaufnahme-Werte** - Datenbankgröße, geschätzter Heap-Bloat - sind genau das, was im Moment dieser Momentaufnahme gemessen wurde. Ein Punkt pro Momentaufnahme, so gezeichnet wie erfasst.
- **Raten** - Cache-Hit%, Rollback%, Top-Abfragen nach Gesamtzeit - stammen aus `pg_stat_database`- und `pg_stat_statements`-Zählern, die seit dem letzten Reset nur hochzählen. Der rohe Zähler einer einzelnen Momentaufnahme sagt allein nichts aus; gezeichnet wird die **Differenz zwischen zwei aufeinanderfolgenden Momentaufnahmen**, umgerechnet in eine Rate für dieses Fenster. Das ist dieselbe Rechnung, die das Zwei-Snapshot-`compare` verwendet, nur wiederholt für jedes aufeinanderfolgende Paar im Verlauf statt für ein vom Benutzer gewähltes Paar.

**Regressionen** werden oberhalb der Diagramme hervorgehoben, statt dem Leser überlassen, sie in einer Linie zu bemerken: eine Metrik wird nur markiert, wenn sie sich sowohl um einen relativen Betrag (einen Prozentsatz) als auch einen absoluten Betrag (Bytes oder Prozentpunkte) bewegt hat - dieselbe Regel, die jeder Schwellenwert in dieser App verwendet, sodass eine kleine Datenbank, die sich verdoppelt, eine große, die um weit mehr in absoluten Zahlen wächst, nicht überbietet, und umgekehrt. Der Vergleichspunkt liegt etwa eine Woche vor der jüngsten Momentaufnahme, oder der älteste verfügbare Punkt, wenn der Verlauf kürzer ist als das.

**Lücken werden gezeigt, nie geglättet.** Wenn das Intervall zwischen zwei Momentaufnahmen weit größer ist als das übliche Intervall des Servers (2,5x der Median, mit einer Untergrenze von 3 Stunden, damit zwei manuelle Momentaufnahmen im Abstand von einer Stunde nie gegeneinander markiert werden), unterbricht das Diagramm die Linie dort, statt eine gerade Kante über eine Zeit zu zeichnen, für die tatsächlich nichts erfasst wurde. Ein schattiertes Band markiert genau, wo die fehlende Spanne liegt; Hovern darüber sagt, wie breit sie ist.

Das **Top-Abfragen**-Diagramm verfolgt die Handvoll `queryid`s, die über den gesamten Verlauf summiert die meiste Gesamtzeit gekostet haben, jede als eigene Linie der pro Fenster aufgewendeten Zeit - nicht die einzelne kumulative Summe, die nur wächst und wäre nur eine gerade Diagonale, die nichts darüber aussagt, *wann* die Kosten bezahlt wurden.

Eine Liste der **größten Tabellen** unter dem Größendiagramm nennt die größten Tabellen zum Zeitpunkt der letzten Momentaufnahme - "Datenbankgröße" allein sagt nicht, welche Tabelle tatsächlich gewachsen ist. Sie wird nicht pro Tabelle über die Zeit verfolgt (eine Tabelle kann aus den Top 10 herausfallen und zurückkehren, während andere um sie herum wachsen), nur das aktuelle Bild, genau wie `db-table-sizes`.

#### Gespeicherte Berichte {#saved-reports}

Jede Workload-Aufzeichnung, jeder AWR-Bericht (ein Bericht über zwei Snapshots, ein Zeitraumvergleich oder ein Trend), jeder ASH-Bericht und jedes Sitzungs-Trace wird aufbewahrt, wenn es fertig ist. *Optimierung und Diagnose* > **Gespeicherte Berichte** listet sie von allen Servern an einer Stelle, die neuesten zuerst: wann, der Typ, der Server, die Datenbank, um die es geht (*ganzer Server* für einen Bericht über alle), und was er abdeckt (das Fenster, die zwei Snapshots, der verfolgte Benutzer), mit seiner Größe. Der Link **Gespeicherte Berichte** in den Tabs AWR, ASH und Sitzungs-Trace öffnet dieselbe Liste, gefiltert auf dieses Werkzeug und diesen Server; vom Ergebnis der *Aufzeichnung* im AWR-Tab zeigt er die Workload-Aufzeichnungen.

- **Filter**: Typ, Server, Datenbank und ein Datumsbereich (die Datumsauswahl unterstreicht die Tage, die einen Bericht enthalten). **Filter zurücksetzen** zeigt wieder alles.
- **Öffnen** zeigt den Bericht so, wie er gespeichert wurde, nur lesend: die Datenbank wird nicht gefragt, und die Schaltflächen, die sie ändern würden, fehlen. Seine Fußzeile hat die Exportzeile, die der Bericht hatte (Text, CSV, JSON, PDF), geschrieben mit dem Server und der Laufzeit des Berichts, nicht mit den heutigen.
- **Exportieren** öffnet dieselben vier Formate in der Zeile, ohne den Bericht zu öffnen.
- **Löschen** fragt zuerst und sagt, was entfernt wird: der Bericht mit seinen Abschnitten und seinem Text und sein Eintrag in den Hintergrundaufgaben; bei einem Trace auch sein Eintrag im Verlauf des Trace. Eine früher exportierte Kopie ist nicht betroffen. Das Audit-Protokoll hält das Löschen fest.

Berichte werden getrennt von den anderen Ergebnissen der Hintergrundaufgaben aufbewahrt: die neuesten 200 bleiben, wie viele andere Prüfungen auch laufen (der Trace-Speicher behält die neuesten 500 Traces pro Server). Die Bibliothek speichert nichts Eigenes, daher tragen Export und Import der App-Daten sie mit: gespeicherte Berichte reisen mit dem Eintrag für Berichte, Traces mit der AWR- und ASH-Historie des Servers.

### 6.7 Logischer Teilmengen-Export (eine Scheibe, die für sich lädt) {#logical-subset-export-a-slice-that-loads-on-its-own}

Ein Backup kopiert ganze Tabellen; dies kopiert eine Scheibe davon. Sie
benennen die Ausgangszeilen mit einer WHERE-Bedingung - ein Kunde, ein Mandant,
die Aufträge des letzten Monats - und der Export folgt den Fremdschlüsseln von
dort aus: Eltern, weil eine Zeile ohne ihren Kunden nicht lädt, und Kinder,
weil ein Auftrag ohne Positionen nicht derselbe Auftrag ist. Heraus kommt eine
einzelne `.sql`-Datei, Eltern zuerst geschrieben, die für sich allein in eine
leere Datenbank lädt, ohne einen einzigen Fremdschlüsselfehler.

Der übliche Grund dafür: eine Testdatenbank, die sich wie die Produktion
verhält und auf einen Laptop passt - Mandant 42 statt 400 GB.

1. Schema-Browser -> Rechtsklick auf eine Tabelle -> Teilmenge.
2. Eine WHERE-Bedingung für die Ausgangstabelle eingeben.
3. **Hülle ermitteln** geht die Fremdschlüssel durch, ohne eine einzige Zeile zu
   materialisieren, und meldet, wie viele jede Tabelle beitragen würde. Zuerst
   ausführen: eine WHERE-Bedingung, die eng aussieht, kann über eine breite
   Verbindungstabelle fast die ganze Datenbank hereinziehen, und das erfährt man
   besser hier als auf halbem Weg durch den Export.
4. Der Export schreibt die Zeilen an einen Ort Ihrer Wahl - INSERT-Anweisungen,
   oder COPY-Blöcke, wenn die Ladezeit wichtiger ist als die Bearbeitbarkeit.

Tiefe (wie vielen Fremdschlüsseln der Lauf folgt) und eine Zeilenobergrenze je
Tabelle halten den Lauf beide an. Wird eine davon erreicht, wird das gemeldet
statt stillschweigend angewandt: eine abgeschnittene Teilmenge, die vollständig
aussieht, ist schlimmer als eine, die sagt, dass sie es nicht ist.

Die Zeilen werden genau so kopiert, wie sie sind, **personenbezogene Daten
eingeschlossen**. Die Maskierung sitzt in Copy table, siehe unten.

### 6.8 Personenbezogene Daten in einer Kopie maskieren (PII-Anonymisierung) {#masking-personal-data-in-a-copy-pii-anonymisation}

Personenbezogene Daten - Namen, E-Mail-Adressen, Telefonnummern,
Ausweisnummern, Geburtsdaten - sind der Grund, warum man eine Produktionskopie
nicht einfach einem Entwickler oder einem Dienstleister geben kann.
Schema-Browser -> Rechtsklick auf eine Tabelle -> **Copy table** -> PII
anonymization ersetzt diese Werte schon beim Lesen: die Spalte wird im SELECT
durch einen Ausdruck ersetzt, der echte Wert erreicht das Ziel also nie und wird
nie auf Platte geschrieben. Die Quelltabelle wird nicht verändert. Die Spalten
benennen Sie selbst - nichts wird automatisch erkannt, und eine vergessene
Spalte wird im Klartext kopiert.

| Regel | Schreibt | Was weiterhin funktioniert |
| --- | --- | --- |
| `NULL` | nichts | Nichts, was auslaufen könnte. Scheitert an einer NOT-NULL-Spalte. |
| `@HASH` | `sha256` des Werts (`md5` vor PostgreSQL 11) | Gleiche Werte bleiben gleich, Joins und GROUP BY funktionieren weiter. Fügen Sie einen Salt (unten) hinzu, damit der Hash nicht auf die Produktion zurückgeführt werden kann. |
| `@HASH:<salt>` | wie `@HASH`, mit diesem Salt pro Spalte | Für den seltenen Fall, dass eine Spalte einen eigenen Salt zusätzlich zum Kopie-weiten braucht. |
| `@SEQ` | `name_1`, `name_2`, ... | Die Werte bleiben eindeutig. Jede Übereinstimmung mit einer anderen Tabelle ist weg. |
| `@EMAIL` | `user1@example.invalid` | Gültige Syntax, und an `.invalid` kann nichts zugestellt werden - kein Testlauf schreibt einem Kunden. |
| `@PHONE` | `0000000000` | Die Länge, Formatprüfungen laufen also weiter durch. |
| `@DATE` | der Monatserste | Monatsauswertungen und Altersgruppen. Der genaue Tag ist weg. |
| `@REDACT` | `REDACTED` | Nur, dass überhaupt ein Wert da war. |
| `@EXPR:<sql>` | was das SQL zurückgibt | Ihre eigene Regel, z. B. `left(name,1) || '***'`. Sie wird nicht geprüft - ein falscher Ausdruck lässt die Kopie scheitern. |
| beliebiger anderer Text | dieser Text, in jeder Zeile | Ein fester Platzhalter. Die Spalte variiert überhaupt nicht mehr. |

#### Der Salt

Ein ungesalzener Hash ist eine Verkleidung, kein Beweis. Wenn die Menge der
möglichen Eingaben klein und bekannt ist - Ausweisnummer, Telefonnummer,
Postleitzahl -, kann jeder mit derselben Hash-Funktion jede Möglichkeit hashen
und die Werte direkt zurückführen. Ein **Salt** ist eine geheime Zeichenkette,
die vor dem Hashen in den Wert gemischt wird, sodass der Hash ohne ihn nicht
reproduziert werden kann. Die Kontrolle **Salt für @HASH** bietet:

- **keiner** - der Hash des Werts allein (re-identifizierbar wie oben).
- **zufällig, nur diese Kopie** - ein frischer Zufalls-Salt wird für diese
  Kopie erzeugt und danach verworfen. Gleiche Eingaben hashen weiterhin gleich
  *innerhalb* dieser Kopie, Joins und GROUP BY auf maskierten Spalten
  funktionieren weiter, aber die Kopie lässt sich nicht mit der Produktion
  oder einer anderen Kopie abgleichen.
- **fest** - Sie geben den Salt vor. Nutzen Sie dies bei **inkrementellen**
  Kopien (eine zweite Kopie muss zur ersten passen) oder wenn Sie **zwei
  Kopien vergleichen** müssen - beide brauchen denselben Salt. Bewahren Sie
  ihn sicher auf; wer ihn hat, kann den Schutz des Salts aufheben.

Der Salt wird als transaktionslokale Einstellung an den Server gesendet
(`set_config('dbapguin.anon_salt', ...)`) und von der Maskierungs-Expression
über `current_setting()` zurückgelesen. Er wird **nie** in das generierte
SQL, das DBApguin-Audit-Log oder die Zieltabelle geschrieben. Er erreicht den
Quellserver als ein `set_config`-Aufruf - protokolliert dieser Server jede
Anweisung (`log_statement = all`), erscheint der Salt dort einmal.

`@SEQ`, `@REDACT` oder `NULL` sind weiterhin die richtige Wahl, wenn die Kopie
überhaupt nicht re-identifizierbar sein darf, mit oder ohne Salt. Und
Maskierung arbeitet Spalte für Spalte: eine Zeile ohne Namen, aber mit
unverändertem Geburtsdatum und Postleitzahl, zeigt oft immer noch auf genau
eine Person.

Eine Spalte, die Sie gar nicht brauchen, lassen Sie besser ganz weg - unter
Data filters -> Skip columns.

### 6.9 Constraints - entfernen und zurückstellen {#constraints-take-them-off-put-them-back}

Manchmal müssen die Constraints von einer Tabelle herunter und danach wieder
darauf. Der übliche Fall ist ein Replikations- oder CDC-Werkzeug bei der
Erstsynchronisation ins Ziel: es startet nicht, solange dort Fremdschlüssel
durchgesetzt werden. Ein Massenimport ist der andere - ohne die Prüfungen läuft
er schneller und bekommt sie am Ende zurück.

Von Hand ist das die Stelle, an der Daten still aufhören, konsistent zu sein,
denn drei Dinge lassen sich leicht falsch machen, und PostgreSQL hält Sie nur
bei einem davon auf:

- `DROP ... CASCADE` entfernt die Fremdschlüssel, die andere Tabellen auf diese
  richten, und hält nicht fest, was es entfernt hat. Ihre Definitionen gehen
  mit.
- Eine aus dem Gedächtnis getippte Definition verliert `ON DELETE CASCADE`, oder
  `DEFERRABLE INITIALLY DEFERRED`, oder ein `WHERE` an einem partiellen
  Unique-Index. Sie sieht wiederhergestellt aus. Sie setzt etwas anderes durch.
- Einen Fremdschlüssel auf gewöhnlichem Weg zurückzustellen scannt die ganze
  Tabelle unter einer Sperre, die Lesen und Schreiben anhält.

Schema-Browser -> Rechtsklick auf eine Tabelle -> **Constraints - entfernen und
wiederherstellen**. Es liest jeden Constraint aus `pg_constraint` über
`pg_get_constraintdef`, PostgreSQLs eigener Darstellung, und schreibt zwei
Skripte: eines, das sie in einer funktionierenden Reihenfolge entfernt, und
eines, das sie exakt so zurückstellt, wie sie waren. Die Anwendung führt nichts
aus. Sie bekommen SQL und starten es, wenn Sie es entscheiden.

Drei Optionen verändern das Wiederherstellungsskript:

- **Mit NOT VALID hinzufügen, dann separat validieren.** `ADD CONSTRAINT ... NOT
  VALID` überspringt den Scan; `VALIDATE CONSTRAINT` holt ihn danach unter
  `SHARE UPDATE EXCLUSIVE` nach, durch das Leser und Schreiber glatt
  hindurchlaufen. Gemessen an einer Tabelle mit 200.000 Zeilen: aus 139 ms
  Blockade wurden 1,2 ms plus ein 131-ms-Durchlauf, auf den niemand wartet. Bis
  VALIDATE gelaufen ist, gilt der Constraint nur für neue Zeilen, und
  `pg_constraint.convalidated` sagt das auch.
- **Primärschlüssel- und Unique-Indizes online neu bauen.** `CREATE UNIQUE INDEX
  CONCURRENTLY`, dann `ADD CONSTRAINT ... USING INDEX`, das diesen Index
  übernimmt, statt einen zweiten zu bauen - das Ergebnis ist also identisch mit
  einem einfachen Neu-Hinzufügen. Der Index wird unter dem eigenen Namen des
  Constraints gebaut, aus der Definition, die der Server selbst liefert
  (`pg_get_indexdef`) - ein abdeckender Schlüssel, ein Fillfactor, ein
  Tablespace oder `NULLS NOT DISTINCT` kommen also unverändert zurück, und ein
  aufschiebbarer Constraint behält seine Aufschiebbarkeit. `CREATE INDEX
  CONCURRENTLY` kann nicht in einem
  Transaktionsblock laufen, weshalb das Wiederherstellungsskript nicht in
  `BEGIN`/`COMMIT` gepackt werden darf.

  Es ist zugleich die einzige Anweisung im Skript, die scheitern und etwas
  zurücklassen kann: eine doppelte Zeile oder ein abgebrochener Lauf hinterlässt
  einen Index, den es gibt und der unbrauchbar ist, und nichts in der
  Constraint-Liste erklärt ihn. Das Skript entfernt einen solchen Index vor dem
  Neubau selbst, ein erneuter Lauf nach einem Fehlschlag genügt also. Entfernt
  wird nur ein Index, den PostgreSQL als ungültig markiert hat - einer, der in
  Benutzung ist, wird nie angerührt.
- **Die Fremdschlüssel einbeziehen, die auf diese Tabelle zeigen.** Aus scheitert
  das Entfernungsskript am Primärschlüssel: der Fremdschlüssel einer anderen
  Tabelle hängt an dessen Index. An setzen jene Tabellen ihre Referenz auf diese
  kurz nicht durch - der Handel, den die Situation verlangt, ausgesprochen statt
  in einem CASCADE versteckt.

Beide Skripte beginnen mit `SET lock_timeout = '3s'`. Jedes `ALTER TABLE` darin
braucht `ACCESS EXCLUSIVE`, und das Warten auf diese Sperre ist es, was aus
einem Online-Neubau einen Ausfall macht: die Anforderung reiht sich hinter dem
ein, was gerade läuft, und alles, was danach kommt, reiht sich hinter der
Anforderung ein. Drei Sekunden bedeuten, dass der schlimmste Fall eine erneut
auszuführende Anweisung ist und nicht eine Tabelle, die niemand lesen kann.
Löschen Sie die Zeile, um beliebig lange zu warten. DBApguins eigener Editor
setzt ohnehin zwei Sekunden auf jede Verbindung, das hier zählt also vor allem,
wenn die Datei Monate später aus `psql` läuft.

Speichern Sie das Wiederherstellungsskript, bevor Sie das Entfernungsskript
starten. Es ist die einzige Kopie dessen, was die Tabelle vorher durchgesetzt
hat.

#### Umfang: eine Tabelle, ein Schema oder mehrere {#scope-one-table-one-schema-or-several}

Derselbe Dialog beantwortet alle drei. Rechtsklick auf eine **Tabelle** für
diese Tabelle; Rechtsklick auf ein **Schema** → Übertragen → *Constraints - 
entfernen und wiederherstellen* für jede Tabelle darin. Eine Migration oder
eine Replikationsumstellung ist eine Frage an das Schema, und sie Tabelle für
Tabelle zu erledigen ist die Art, wie ein Constraint vergessen wird.

**Welche Constraints** filtert nach Art - Primärschlüssel, Unique,
Fremdschlüssel, Check, Exclude. Nur *Fremdschlüssel* anzuhaken ist der übliche
Fall: ein CDC-Werkzeug stört sich meist nur an ihnen, und stehen gelassene
Primärschlüssel bedeuten, dass das Ziel seine Identität und seine Indizes
behält.

Auf Schemaebene zählt die Reihenfolge mehr als bei einer Tabelle, und das
Skript kümmert sich darum: jeder Primär- und Unique-Schlüssel kommt zurück,
bevor irgendein Fremdschlüssel ihn referenziert, und das Entfernungsskript
macht es umgekehrt. Nach Tabellennamen sortiert - wie es die erste Fassung tat
 - wird ein Fremdschlüssel auf `items` hinzugefügt, bevor `orders` seinen
Primärschlüssel hat, und PostgreSQL lehnt das mit *there is no unique
constraint matching given keys* ab.

#### Constraints auf ein anderes Schema kopieren {#copying-constraints-onto-another-schema}

Nennen Sie ein **Zielschema**, und der Dialog schreibt kein Entfernungsskript
mehr, sondern eines, das diese Constraints auf die Tabellen jenes Schemas
setzt. Der Fall dafür: eine Kopie der Daten, die ohne Constraints geladen wurde
 - weil sie so schnell lädt, oder weil die Replikation sonst nicht angelaufen
wäre - und die jetzt dieselben Regeln braucht wie das Original.

Tabellen- und Spaltennamen müssen übereinstimmen. Nur die Constraints reisen
mit: keine Daten, keine Spalten, keine Indizes außer denen, die ein Schlüssel
braucht.

Die Definitionen werden aus dem Quellschema heraus gelesen, ein Verweis auf
eine Tabelle in diesem Schema kommt also unqualifiziert heraus - 
`REFERENCES customers(id)` - und das `SET search_path` des Skripts richtet ihn
aufs Ziel. Ein Verweis in ein *anderes* Schema bleibt qualifiziert und zeigt
weiter dorthin, wohin er zeigte; auch das ist richtig, denn ein Fremdschlüssel
in ein gemeinsames Referenzschema soll nicht umgelenkt werden.

Constraint-Namen kommen unverändert mit, denn ein Constraint-Name ist je
Tabelle eindeutig, nicht je Datenbank. Hat das Ziel bereits einen dieses Namens
auf dieser Tabelle, schlägt die Anweisung fehl - das sichere Scheitern, denn
die Alternative wäre, etwas stillschweigend zu ersetzen.

### 6.10 Ähnlichkeitssuche: pg_trgm und pgvector {#similarity-search}

Zwei Erweiterungen beantworten „finde Zeilen wie diese", auf zwei verschiedene
Arten:

- **pg_trgm** vergleicht, wie Text *aussieht*. Es zerlegt Text in Stücke aus
  drei Buchstaben (Trigramme), sodass auch ein Tippfehler oder ein Teil eines
  Namens noch trifft: `LIKE '%pad%'`, `ILIKE`, der Ähnlichkeitsoperator `%` und
  der Trigramm-Abstand `<->`. Es kommt mit den contrib-Modulen von PostgreSQL.
- **pgvector** vergleicht, was ein Modell über die *Bedeutung* sagt. Ein Modell
  macht aus jedem Text eine Liste von Zahlen (ein Embedding, gespeichert in
  einer Spalte `vector(n)`, `halfvec(n)` oder `sparsevec(n)`); nahe Zahlen
  heißen nahe Bedeutung. Abfragen fragen nach den nächsten Zeilen:
  `ORDER BY embedding <=> $1 LIMIT 10`.

Beide brauchen verschiedene Indizes, und ein btree bedient keine von beiden:
weder `ILIKE` noch ein Muster, das mit einem Platzhalter beginnt, noch einen
Abstand. Was die Anwendung mit jeder tut:

**Datenbank -> Erweiterungen verwalten** zeigt beide mit der installierten
Version und der neuesten, die der Server mitbringt; **update** bietet jede
spätere Version an, in Versionsreihenfolge (0.10 nach 0.9).

**EXPLAIN** liest die Indizes, die ein Plan benutzt, nicht nur ihre Namen:

- ein Index-Scan über **HNSW** oder **IVFFlat** wird als näherungsweise
  Nächste-Nachbarn-Suche benannt, mit der Operatorklasse, für die er gebaut
  ist, und der Einstellung, die Geschwindigkeit gegen Trefferquote tauscht
  (`hnsw.ef_search`, `ivfflat.probes`), wie die Sitzung sie hatte, die den
  Plan ausführte;
- ein Filter im selben Schritt wird markiert: der Index liefert zuerst seine
  Kandidaten und der Filter läuft danach, also können weniger Zeilen als das
  `LIMIT` zurückkommen. Ab pgvector 0.8.0 bietet der Hinweis
  `hnsw.iterative_scan` an;
- ein **Sort nach einem Abstand über einem sequenziellen Scan** ist eine
  exakte Suche über alle Zeilen. Der Hinweis sagt, welcher von drei Fällen es
  ist: kein näherungsweiser Index auf der Spalte; ein Index für einen *anderen*
  Operator (ein Index mit `vector_cosine_ops` bedient nur `<=>`, also liest
  `ORDER BY embedding <-> ...` die ganze Tabelle); oder ein passender Index,
  den der Planer nicht benutzt hat, weil er nur für
  `ORDER BY Spalte <op> Wert` aufsteigend mit einem `LIMIT` benutzt wird;
- ein **Trigramm-Index** (GIN `gin_trgm_ops`, GiST `gist_trgm_ops`) wird als
  Stücke plus Nachprüfung erklärt, mit der Zahl der gefundenen Zeilen, die
  kein Treffer waren;
- ein sequenzieller Scan mit einem Filter `LIKE`/`ILIKE` auf einer Spalte, die
  nur einen btree hat, sagt, dass der btree ihn nicht bedienen kann, und nennt
  den Trigramm-Index; `similarity(a, b) > x` in einem Filter wird nie
  indiziert, und der Hinweis sagt, `a % b` zu schreiben.

Knoten, die ein `LIMIT` früh angehalten hat (eine Top-10-Suche liest 10 von
geschätzten 20.000 Zeilen), werden nicht mehr als schlechteste Schätzung des
Plans markiert.

**Index-Berater** (aus pg_stat_statements): für `LIKE`, `ILIKE`, `%` und
`similarity()` auf einer Spalte schlägt er einen Trigramm-GIN-Index vor, nie
einen btree, mit `CREATE EXTENSION pg_trgm` davor, wo die Datenbank es noch
nicht hat (der Run-Dialog zeigt beides und führt beides aus). Für
`ORDER BY Spalte <Abstand> ...` schlägt er einen HNSW-Index mit der
Operatorklasse für den Operator vor, den die Abfragen benutzen, gelesen aus
dem Katalog des Servers selbst, und nennt den Preis: näherungsweise, er kann
einige der tatsächlich nächsten Zeilen verfehlen. Einen btree auf einer
Vektorspalte schlägt er nie vor. Eine Spalte mit mehr Dimensionen, als ein
Index annimmt (2.000 für `vector`, 4.000 für `halfvec`), oder ohne deklarierte
Dimension bekommt die Erklärung statt einer Anweisung.

**Wichtige Einstellungen**, wo pgvector in der Datenbank installiert ist:
`hnsw.ef_search`, `hnsw.iterative_scan`, `hnsw.max_scan_tuples`,
`ivfflat.probes` und `ivfflat.iterative_scan` (die iterativen ab pgvector
0.8.0), und `pg_trgm.similarity_threshold`, wo pg_trgm ist.
`maintenance_work_mem` wird mit dem größten HNSW-Index verglichen: das README
von pgvector sagt, dass ein HNSW-Bau deutlich schneller ist, wenn der Graph
hineinpasst, und dass ein paralleler Bau in einem Container `/dev/shm`
mindestens so groß braucht. `max_parallel_maintenance_workers` sagt, dass
pgvector damit seine Indizes baut.

**Größen.** Schema -> eine Tabelle -> **Speicher** zeigt jede Vektorspalte mit
den Bytes, die ein Wert belegt (Formel von pgvector: `vector`
4 x Dimensionen + 8, `halfvec` 2 x Dimensionen + 8), was eine Stichprobe
gespeicherter Werte belegt und, auf PostgreSQL 17, wie viele davon in der
TOAST-Tabelle liegen - ein `vector(768)` hat etwa 3 kB, mehr als eine Zeile in
sich behält, deshalb zeigt so eine Tabelle einen kleinen Heap und eine große
TOAST-Tabelle. Darunter die Indizes der Tabelle mit Methode, Operatorklasse,
Optionen (`m`, `ef_construction`, `lists`) und Größe. Die Prüfung
**Indexgrößen** nennt Methode und Operatorklasse jedes Index und die
TOAST-Größe seiner Tabelle.

**Index erstellen** bietet `hnsw` und `ivfflat` als Methoden und eine
Operatorklasse für die Spalten (`vector_cosine_ops`, `gin_trgm_ops`, ...); ASC/DESC
und NULLS gibt es nur für btree, die einzige Methode, die sie annimmt.

**Kopien.** DDL, Vergleich und Kopieren behalten Operatorklassen und
Index-Optionen. Ein Schema-Klon mit Zeilen baut die HNSW- und IVFFlat-Indizes
nach den Zeilen (ein IVFFlat-Index berechnet seine Listen aus den Zeilen, die
beim Bau vorhanden sind), und Tabelle kopieren baut einen IVFFlat-Index neu,
sobald die Zeilen drin sind. Der Vergleich setzt eine Notiz über eine Änderung
der Dimension einer Vektorspalte: gespeicherte Vektoren werden nicht in eine
andere Dimension umgewandelt, also müssen die Zeilen neu eingebettet werden.

## 7. Einstellungen

Das Zahnrad oben rechts öffnet die Einstellungen:

- Design: Dunkel / Hell
- **Sprache: English, Hrvatski, Deutsch, Español.** Die Wahl wird serverseitig
  gespeichert (`app_prefs.json`) und folgt Ihnen daher über Neustarts hinweg,
  statt in einem Browserprofil zu leben. Sie greift sofort, ohne Neustart, auch
  für bereits geöffnete Menüs und Dialoge.
- Schriftgröße (10-30 px)
- **Start -> Port:** der Port, auf dem das eigene Backend von DBApguin lauscht -
  nur auf diesem Computer, `127.0.0.1` - 3236, solange Sie ihn nicht ändern.
  Ändern Sie ihn, wenn ein anderes Programm 3236 braucht; er gilt ab dem
  nächsten Start. Was das Fenster sich merkt (Design, Favoriten, SQL-Verlauf,
  offene Tabs), zieht mit um.
- Audit-Log: ein / aus (siehe unten)
- Werkzeuge: Pfade zu psql / pg_dump / pg_restore / pg_repack (überschreiben die automatische Suche)
- Zertifikate: PEM-kodierte Wurzelzertifikate für `sslmode=verify-*` hochladen
- Zertifikate und Werkzeugpfade wie oben; **Über ist nach Hilfe -> Über DBApguin gewandert**, zu den rechtlichen Dokumenten

### Audit-Log {#audit-log}

Eingeschaltet, bis Sie es ausschalten - der Schalter sitzt im Log-Fenster
selbst. Jede Anweisung, die der SQL-Editor ausführt (außer der Tab ist auf
schreibgeschützt gestellt), und jede Änderung über DBApguins eigene Werkzeuge
wird an `audit/audit-YYYY-MM.jsonl` angehängt (eine Datei pro Monat) -
Zeitpunkt, Ziel, Ihr Betriebssystem-Benutzer und der Datenbankbenutzer, die
Anweisung mit geschwärzten Passwörtern, die Dauer und das Ergebnis. Öffnen Sie
es über **Ansicht -> Audit-Protokoll**: Suche nach Datumsbereich, Aktion oder einem Teilstring der SQL,
Export eines Zeitraums als JSONL, oder die Rohdateien direkt mit einem
Texteditor oder `jq` lesen.

**Jeder Eintrag ist mit dem davor verkettet.** Jede Zeile trägt einen `hash`
ihres eigenen Inhalts plus den `hash` der vorherigen Zeile (`prev_hash`), so
dass eine Änderung oder das Löschen einer vergangenen Zeile ändert, was der
Hash der nächsten Zeile hätte sein müssen - **Kette prüfen** im Log-Viewer
berechnet die gesamte Kette neu und meldet entweder "unversehrt" oder genau,
welche Datei und Zeile als erste nicht mehr übereinstimmt. Ein vor dieser
Funktion geschriebener Eintrag trägt keines der beiden Felder; der Prüfer
behandelt solche Einträge als vor-Ketten-Ära, statt bei ihnen fehlzuschlagen,
und meldet, wo die Kette selbst beginnt. Die Kette übersteht auch die
monatliche Rotation - der erste Eintrag eines neuen Monats verweist zurück auf
den letzten Eintrag der vorherigen Datei, sodass die Aufteilung des Logs in
Dateien die Kette nicht aufteilt.

**Lesen Sie genau, was dies beweist, denn es ist leicht überzuschätzt.** Es
beweist, dass niemand eine beiläufige, unbemerkte Änderung an einer Kette
vorgenommen hat, die sich verifiziert - nützlich für sich genommen, und der
Grund, warum es die Schaltfläche gibt. Es beweist **nicht**, dass niemand mit
Schreibzugriff auf die Datei manipuliert hat: Hashing braucht kein Geheimnis,
der Algorithmus ist öffentlich, und das Log liegt direkt neben der Anwendung,
die es neu berechnen kann - wer also `audit-2026-05.jsonl` bearbeiten kann,
kann auch jeden Hash danach neu berechnen, und die Kette verifiziert sich
erneut, als wäre nichts geschehen. Echter Schutz dagegen braucht einen Anker
außerhalb dieser Maschine - eine Signatur mit einem Schlüssel, den Sie nicht
besitzen, oder den Hash, der im Moment des Schreibens anderswohin gesendet
wird - und diese Funktion tut das nicht. Wo Sie einen Datensatz brauchen, der
einen kompromittierten lokalen Administrator übersteht, nutzen Sie das
serverseitige Logging Ihrer Datenbank (`pgaudit`, `log_statement`), das in ein
separates, zugriffsgeschütztes Ziel schreibt, nicht diese Datei. Siehe die [Lizenzvereinbarung](EULA.md), Abschnitt 7 (Hilfe -> Lizenzvereinbarung).

### App-Daten: exportieren, importieren und löschen {#app-data-backup-and-clear}

Alles, was DBApguin zwischen den Starts behält, liegt in einem Verzeichnis (der Pfad steht in beiden Dialogen). Einstellungen -> App-Daten hat drei Schaltflächen.

**App-Daten exportieren** schreibt eine `.tar.gz` dessen, was Sie anhaken: gespeicherte Verbindungen, Einstellungen, gespeicherte Abfragen, den SQL-Arbeitsbereich, die Aufgabenliste, gespeicherte Berichte, die gespeicherten AWR-Snapshots, ASH-Tage und Sitzungs-Traces jedes Servers, das Audit-Log, installierte Plugins und ihre Daten, TLS-Zertifikate. Standardmäßig ist alles angehakt. Der Datenschlüssel geht mit den Verbindungen und den Plugin-Daten mit, weil deren Passwörter ohne ihn nicht lesbar sind.

**App-Daten importieren** liest ein solches Archiv, zeigt, was es enthält, neben dem, was schon hier ist, und übernimmt, was Sie anhaken (wieder standardmäßig alles). Es führt zusammen statt zu ersetzen: gespeicherte Historie (AWR-Snapshots, ASH-Tage, Sitzungs-Traces, gespeicherte Berichte, Job-Logs, das Audit-Log, Plugins) wird ergänzt und überschreibt nie eine vorhandene Datei; gespeicherte Verbindungen werden nach ID zusammengeführt, ein vergebener Name bekommt eine Nummer, und ihre Passwörter werden mit dem Schlüssel dieser Installation neu verschlüsselt; eine einzelne Datei wie die Einstellungen oder die gespeicherten Abfragen ersetzt Ihre. Ein Archiv ohne Schlüssel oder mit einem durch Master-Passwort geschützten importiert die Verbindungen ohne Passwörter und sagt das. So nimmt ein DBA die gesammelte Historie und die Einrichtung auf einen anderen Rechner mit; einen gemeinsamen Speicher zwischen Installationen gibt es nicht.

> Die Sicherung **enthält `key` und `connections.json`** - den Datenschlüssel und Ihre verschlüsselten Verbindungspasswörter. Das ist Absicht: eine Sicherung Ihrer Einrichtung ohne diese ist keine. Aber es bedeutet, dass das Archiv sensibel ist. Bewahren Sie es privat auf und **niemals in einem Ordner, der mit einem Cloud-Dienst synchronisiert** (Dropbox, OneDrive, iCloud, Nextcloud, ein synchronisierter Dokumente-Ordner). Wenn ein Master-Passwort gesetzt ist, bleiben die Passwörter im Archiv damit verschlüsselt; ohne eines sind sie nur leicht verschleiert.

Die Sicherung enthält auch Ihre **installierten Plugins** (`plugins/`) und was sie aufbewahren (`plugin-data/`) - ihre Einstellungen und gespeicherten Passwörter, verschlüsselt mit demselben `key` -, sodass die Wiederherstellung die Plugins funktionsfähig zurückbringt. Der Ordner, in dem Sie eigene Plugins schreiben, ist nicht enthalten: er liegt dort, wo Sie ihn gewählt haben.

Der Verbindungsexport aus der Verbindungsverwaltung ist etwas anderes: er schreibt nur die Verbindungen und ihre Gruppen, ohne Passwörter, Einstellungen und Plugins.

**App-Daten löschen** zeigt eine Checkliste mit der Größe jedes Elements. Berichte und Logs - AWR-Snapshots, Job-Logs, gespeicherte Prüfberichte, das Fehlerprotokoll - sind standardmäßig angehakt; Einstellungen, gespeicherte Abfragen und der SQL-Arbeitsbereich sind aufgeführt, aber nicht angehakt. Gespeicherte Verbindungen (`connections.json` + `key`) stehen in einem eigenen Abschnitt und erfordern die Eingabe des Wortes `DELETE` zur Bestätigung, weil ihr Löschen jede gespeicherte Verbindung und ihr Passwort verliert. Installierte Plugins sind aufgeführt, aber nicht angehakt; ihre Einstellungen und gespeicherten Passwörter (`plugin-data/`) stehen im selben geschützten Abschnitt wie die Verbindungen, weil ihr Löschen diese Passwörter verliert. Ein Verzeichnis, das noch einen laufenden Hintergrundjob hält, wird abgelehnt; brechen Sie den Job zuerst ab. Jedes Löschen wird ins Audit-Log geschrieben.

### Plugins {#plugins}

**Plugins → Plugins verwalten…** zeigt, was installiert ist: Plugins ein- und ausschalten, entfernen oder aus einer `.dbapg-plugin`-Datei installieren. Ein Plugin fügt Prüfungen, Menüeinträge und Tabs hinzu, ohne neue Version von DBApguin.

**Jeder kann ein Plugin erstellen** - die Anwendung bringt dafür ein Werkzeug mit (im selben Tab *Plugins in dieser Anwendung entwickeln* einschalten; der *Plugin-Leitfaden* dort erklärt den Rest). Ein Plugin läuft mit Ihren Rechten: es erreicht jede Datenbank, mit der Sie sich verbinden, und jeden anderen Dienst, für den es eingerichtet ist. **Verwenden Sie nur Plugins von Herausgebern, denen Sie vertrauen.** Die Lizenzvereinbarung (Abschnitt 15) und die Datenschutzhinweise sagen dasselbe in ihren Worten.

**Informationen** an jedem Plugin erklärt es in zwei Teilen. Zuerst, was das Plugin über sich sagt - was es tut, was es liest, was es wohin sendet - in Ihrer Sprache, wenn es eine mitbringt. Dann, was DBApguin selbst aus den Dateien des Plugins liest, unabhängig davon, was das Plugin sagt: ob es Python in der Anwendung oder JavaScript im Fenster ausführt, welche Prüfungen und Adressen es hinzufügt, was es speichert, seinen Ordner und jede Datei.

**DBApguin-zertifiziert** neben dem Namen eines Plugins heißt, dass der Autor von DBApguin es signiert hat und sich seitdem keine seiner Dateien geändert hat. *Seit der Zertifizierung geändert* heißt, dass Dateien von den signierten abweichen (Informationen nennt sie); *Signatur ungültig* heißt, dass die Signatur nicht gilt; *nicht zertifiziert* heißt, dass nichts für die Herkunft bürgt - so sieht auch ein Plugin aus, das Ihr Team geschrieben hat. Ein nicht zertifiziertes Plugin wird trotzdem installiert und läuft: Die Kennzeichnung sagt Ihnen, woher ein Plugin kommt, sie entscheidet nicht für Sie. Vor der Installation eines nicht zertifizierten Plugins wird gefragt, und die Frage zeigt, was es ausführen würde und jede Datei, die es mitbringt.

Das Entfernen eines Plugins löscht seine Einstellungen und gespeicherten Passwörter mit. Eine Sicherung der App-Daten enthält die Plugins und ihre Einstellungen.

## 8. Meldungen

Rechte untere Ecke. Farblich unterschieden:
- Blau (Info): neutraler Status
- Grün (ok): Erfolg
- Rot (Fehler): der Vorgang ist gescheitert

Ein Klick auf den Stapel klappt ihn auf und macht ihn scrollbar. Das Mausrad scrollt. Ein Klick daneben klappt ihn zu.

Solange ein Fenster offen ist, halten Meldungen seine Schaltflächen frei. Eine neue Meldung steht über ihnen oder im Platz neben dem Fenster; wo beides keinen Platz lässt, geht sie sofort in den Zähler, dessen Zahl und Farbe trotzdem zeigen, dass sie gekommen ist. Auch der Zähler steht über den Schaltflächen des Fensters; passt nicht einmal er, ist er ausgeblendet, bis das Fenster geschlossen wird. Danach kehrt alles in die Ecke zurück.

Eine Aktion bietet „Rückgängig": das Löschen einer gespeicherten Verbindung, die die Meldung etwa sechs Sekunden lang zurückholen kann. Eine Meldung kann nicht rückgängig machen, was eine Datenbank erreicht hat; eine hier geänderte Einstellung lässt sich in der Einstellungsansicht zurücknehmen, und ein Löschen, Verschieben oder Eigentümerwechsel hinterlässt eine Kopie - beim Löschen mit den Zeilen, wenn gewählt, und keine, wenn Sie ohne Kopie löschen -, die die Objekte zurückbringt (siehe *Eine Kopie der Definitionen vor der Änderung*). Außerdem zeigt die Anwendung die Anweisung vor der Ausführung, fragt einmal nach und schreibt sie ins Audit-Log.

### 8.1 Während etwas läuft {#while-something-runs}

Alles, was Sie angestoßen haben und länger als eine Viertelsekunde dauert, sagt das, jedes Mal an denselben Stellen:

- Ein schmaler Balken gleitet am oberen Fensterrand entlang, und am rechten Ende der Tab-Leiste unter den Menüs dreht sich ein Ring. Zeigen Sie auf den Ring oder gehen Sie mit Tab dorthin, um die Liste dessen zu sehen, was läuft, bei welchem Schritt es ist und wie lange es schon dauert; nach drei Sekunden zeigt der Ring die Zeit selbst. Wo sich die Arbeit anhalten lässt - eine Prüfung, eine Anweisung im SQL-Editor - hat ihre Zeile in der Liste ein Abbrechen.
- Die Schaltfläche, die Sie gedrückt haben, zeigt statt ihrer Beschriftung einen Ring und nimmt keinen zweiten Druck an, bis die Arbeit beendet ist. Daneben verschiebt sich nichts: Die Schaltfläche behält ihre Größe, und Balken, Ring und Liste nehmen der Seite keinen Platz weg.
- Ein Teil des Fensters, der lädt, sagt das ("Aufträge werden gelesen…", "Spalten werden geladen…"), statt eine leere Liste oder Werte zu zeigen, die sich gleich ändern, und nach drei Sekunden zeigt er, wie lange er schon wartet. Eine Einstellung, deren gespeicherter Wert noch gelesen wird, ist abgeblendet und lässt sich erst ändern, wenn der Wert da ist.

Exporte und Berichte nennen jeden Schritt: Bericht wird erstellt, wartet auf Ihre Auswahl im Dateidialog (öffnet sich der Dialog hinter dem Fenster, ist diese Zeile der Hinweis), Datei bzw. PDF wird geschrieben. Der Arbeitszustand endet in dem Moment, in dem die Arbeit endet, ob sie gelungen ist oder nicht, und eine Meldung sagt, wohin die Datei gegangen ist oder warum nicht.

Was die Anwendung von sich aus tut - die Aktualisierung des Dashboards, die Verbindungsprüfungen, die Auftragsüberwachung, die Abfragen der Sammler - erscheint hier nie, sodass ein drehender Ring immer bedeutet, dass etwas, das Sie angestoßen haben, noch unterwegs ist.

## 9. Tastenkürzel

- `F5` (oder `Strg/Cmd + Enter`): Abfrage im SQL-Editor ausführen
- `F6` (oder `Strg/Cmd + Umschalt + Eingabe`): nur die Markierung oder die Anweisung unter dem Cursor ausführen
- `Strg/Cmd + S`: aktuelles SQL als Schnipsel speichern
- `Strg/Cmd + E`: Abfrage prüfen
- `Strg/Cmd + K`: Befehlspalette
- `Esc`: aktuellen Dialog / das Kontextmenü schließen, Suche leeren
- `Strg/Cmd + W`: aktuellen Arbeits-Tab schließen (sofern nicht angeheftet)
- `Umschalt + Tab`: Tabs wechseln, so wie Alt + Tab Fenster wechselt. Ein Fenster mit Karten öffnet sich, eine je offenem Tab, jede mit einem Bild des Tab-Inhalts, wie Sie ihn zuletzt gesehen haben, und der nächste Tab ist markiert. Umschalt gedrückt halten und mit Tab oder den Pfeiltasten die Markierung bewegen (Links geht zurück, Hoch und Runter springen eine Reihe). Umschalt loslassen öffnet den markierten Tab; Esc schließt das Fenster und bleibt, wo Sie waren. In einem Textfeld oder im SQL-Editor behält Umschalt + Tab seine übliche Bedeutung (vorheriges Feld, Einzug verringern); dort `Strg + Tab` verwenden und Strg loslassen, um den Tab zu öffnen.
- `Strg + Umschalt + Tab`: Verbindungen im gleichen Fenster wechseln: eine Karte je offener Verbindung mit einem Bild dessen, was darin zu sehen war, ihrem Server, ihrem Zustand und der Zahl ihrer Tabs. Gewechselt wird beim Loslassen von Strg, nicht schon beim Ansehen. Unter macOS bleibt das auf der Control-Taste, weil macOS Cmd + Umschalt + Tab für sich behält.
- `F10` oder ein kurzes Tippen auf `Alt` (Option auf dem Mac): die Menüleiste mit der Tastatur öffnen. Links und Rechts wechseln das Menü, Hoch und Runter die Einträge, Rechts öffnet ein Untermenü und Links schließt es, ein Buchstabe springt zum nächsten Eintrag mit diesem Anfang, Eingabe führt den Eintrag aus (eine Prüfung öffnet sich wie beim Klick), Esc schließt eine Ebene nach der anderen. Ein mit der Maus geöffnetes Menü reagiert auf dieselben Tasten.

Die Tasten stehen dort, wo sie gebraucht werden, benannt für Ihr System (auf dem Mac die Symbole für Befehl, Wahl und Control): an den Schaltflächen des SQL-Editors und in deren Tooltips, in dieser Liste (Taste `?`) und als kurzer Tipp, nachdem Sie dasselbe einige Male mit der Maus getan haben. Jeder Tipp erscheint höchstens zweimal; *Keine Tipps mehr* schaltet sie ab, der Schalter unten in der `?`-Liste wieder ein.

## 10. Hinweise je Anbieter

DBApguin erkennt beim Verbinden den Anbieter des Clusters und passt sich an:

| Anbieter | Was gesperrt ist |
|----------|---------------|
| Aurora PostgreSQL | Kein ALTER SYSTEM, kein direktes Schreiben von pg_hba, keine Rollen auf Superuser-Ebene, keine eigenen Tablespaces, kein direktes Lesen von Dateien |
| Amazon RDS PostgreSQL | Wie Aurora, dazu eine Positivliste für CREATE EXTENSION |
| Google Cloud SQL | Wie RDS |
| Google AlloyDB | Wie Cloud SQL: Einstellungen sind die Datenbank-Flags der Instanz. Ein selbst betriebener AlloyDB-Omni-Server hat einen echten Superuser und gilt als selbst betrieben |
| Azure Postgres Flexible | Wie RDS |
| Supabase | Wie RDS, dazu Sperren für reservierte Rollen |
| Neon | Mandant mit einer Datenbank; kein CREATE DATABASE / DROP DATABASE |
| Heroku Postgres | Wie Neon |
| Crunchy Bridge | Die meisten Enterprise-Funktionen verfügbar |
| DigitalOcean Managed | Wie RDS |
| Timescale Cloud | Positivliste für Erweiterungen plus Sperre der Rolle tsdbadmin |
| Selbst betriebenes PostgreSQL | Alles verfügbar |

Die Oberfläche graut gesperrte Menüpunkte aus, mit einem Tooltip, der den Grund nennt. Ist die Rolle der Sitzung ein echter Superuser, wird nichts ausgegraut oder verweigert: Keine verwaltete Plattform gibt einem Kunden einen, also gelten diese Grenzen auf diesem Server nicht (zum Beispiel ein selbst betriebener Server, der die Rollennamen einer Plattform importiert hat). Die Plattform wird weiter genannt, und ihr Hinweisbalken sagt, dass diese Sitzung ein Superuser ist. Die Überwachung `db-errors` fällt auf Zählerdifferenzen und anbieterspezifische Hinweise zum Log-Zugriff zurück, wenn `pg_read_file` verweigert wird.

#### Als .sql-Datei sichern {#backing-them-up-as-a-sql-file}

**Entfernungsskript…** und **Wiederherstellungsskript…** zeigen das Skript
zuerst auf dem Schirm, mit den drei Möglichkeiten, die jedes erzeugte Skript in
dieser Anwendung bietet: kopieren, speichern oder im SQL-Editor öffnen und dort
ausführen. Das Speichern überschreibt eine vorhandene Datei nicht ohne
Rückfrage. Die Wiederherstellungsdatei ist die Sicherung: sie enthält die
Anlege-Anweisungen und den Validierungsschritt, und sie ist das, was man behält,
bevor das Entfernungsskript läuft.

Jede Datei beginnt mit einem Kopf - Anwendungsversion, Server und Datenbank,
Umfang, Zeitpunkt der Erstellung und Anzahl der Constraints. Das Passwort steht
nie darin. Ein halbes Jahr später ist dieser Kopf der Unterschied zwischen einer
brauchbaren Datei und einem anonymen Stapel ALTER-TABLE-Anweisungen.

Für eine ganze Datenbank statt eines Schemas führt `pg_dump
--section=post-data` andersherum zum selben Ziel: es schreibt Constraints,
Indizes und Trigger von allem auf einmal. Das ist Backup → Optionen → Section →
post-data.

### IAM-Logins und generierte Namen

In einer Managed Cloud **ist** die Login-Rolle meist das IAM-Principal, ihr Name
also eine E-Mail-Adresse oder ein Dienstkonto:

```
alice@example.com
123456-compute@developer.gserviceaccount.com
```

Das sind gültige PostgreSQL-Rollennamen - Bezeichner in Anführungszeichen dürfen
`@`, `.` und `-` enthalten und mit einer Ziffer beginnen. DBApguin akzeptiert sie
überall, wo ein Rollenname verlangt wird: Anlegen, Löschen, Gewähren, Entziehen,
Mitgliedschaft, `search_path`, Eigentum und jede Rechteansicht. Das erzeugte SQL
setzt sie in Anführungszeichen, eine Gewährung liest sich also so:

```
GRANT SELECT ON TABLE "app-schema"."orders" TO "alice@example.com"
```

Dasselbe gilt für Datenbank- und Schemanamen. Managed-Anbieter erzeugen
routinemäßig Namen wie `b2b-ext-assistant-dev-pg-db`, und Bindestriche sind in
einem Bezeichner mit Anführungszeichen erlaubt.

Was weiterhin und mit Absicht abgelehnt wird, ist ein Name mit doppeltem
Anführungszeichen, NUL, Leerraum oder Steuerzeichen - nichts davon hat ein
echtes Cloud-Principal, und jedes davon könnte einen Bezeichner vorzeitig
beenden.

IAM-Benutzer haben **kein Passwort**. Wählen Sie beim Anlegen `Kein Passwort` im
Assistenten und gewähren Sie dann aus der Liste „Mitglied von" die
IAM-Mitgliedschaft des Anbieters (`rds_iam` bei AWS, `cloudsqliamuser` bei
Cloud SQL).

## 11. Fehlerbehebung

### „Passwort erforderlich"
Die gespeicherte Verbindung hat kein Passwort hinterlegt. Sie wird zuerst so versucht: ein lokaler Server, der kein Passwort braucht (trust, peer, `.pgpass`), verbindet einfach. Ein Server, der eines verlangt, öffnet den Dialog zum Auffrischen; Passwort eingeben und Verbinden klicken. Verbindet sich der Tab auf anderem Weg, während der Dialog offen ist, schließt er sich von selbst.

### „Schreiben innerhalb <Pfad> wird verweigert"
Das Ziel des pg_dump-Backups liegt an einem schreibgeschützten Ort (AppImage-Mount, Datenverzeichnis der Anwendung). Wählen Sie einen Pfad unter `~/Dokumente` oder ähnlich.

### „pg_stat_statements muss über shared_preload_libraries geladen werden"

pg_stat_statements hat zwei Hälften, und beide sind nötig:

1. die Bibliothek muss in `shared_preload_libraries` stehen, und
2. die Erweiterung muss in der abgefragten Datenbank angelegt sein.

`CREATE EXTENSION pg_stat_statements` erledigt nur das Zweite. Die Erweiterung
steht dann in `pg_extension`, alles sieht installiert aus, und jeder Zugriff auf
die View wirft diesen Fehler. `CREATE EXTENSION` erneut auszuführen ist der
naheliegende nächste Versuch und ändert nichts.

DBApguin erkennt diesen Zustand und sagt, welche Hälfte fehlt. Jede Prüfung, die
Abfragestatistik liest, druckt Grund und Abhilfe unter den Fehler, und
**db-setup-advisor** meldet es als *„In dieser Datenbank angelegt, aber NICHT in
shared_preload_libraries"* statt schlicht als „nicht installiert".

Die Abhilfe, als Superuser:

```
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';
```

danach PostgreSQL **neu starten**. `shared_preload_libraries` wird nur beim Start
des Postmasters gelesen - `SELECT pg_reload_conf()` greift nicht. In einer
Managed Cloud in der Parametergruppe setzen und die Instanz neu starten.

Der umgekehrte Zustand - Bibliothek geladen, Erweiterung nicht angelegt - 
braucht weder Neustart noch Eingriff: die Prüfungen legen die Erweiterung selbst
an und machen weiter.

Betroffene Prüfungen, solange es halb eingerichtet ist: db-stat-statements,
db-index-advisor (nur dessen pg_stat_statements-Heuristik), db-temp-queries,
db-workload-report, db-awr-snapshot und db-trace-session.

### Ein Index existiert, aber der Planer ignoriert ihn {#leftover-damage}

Meist heißt das: der Index ist UNGÜLTIG, und keine normale Auflistung sagt es.
`CREATE INDEX CONCURRENTLY` und `REINDEX CONCURRENTLY` bauen in Phasen, und
eine Sitzung, die mittendrin stirbt - Absturz, Abschuss, Sperren-Timeout -
lässt den halb gebauten Index zurück. Der Planer benutzt ihn nicht, Platz
belegt er trotzdem, und `\d` kennzeichnet ihn nur, wenn man genau hinsieht.

**Cluster health → Auf Reste einer Beschädigung prüfen...** sucht nach den
Zuständen, die PostgreSQL festhält, aber nicht zeigt:

- ungültige Indizes aus einem abgebrochenen CONCURRENTLY-Build
- `NOT VALID`-Constraints, die gegen vorhandene Zeilen nicht durchgesetzt
  werden - eine Wiederherstellung, die die Prüfung aufgeschoben hat, lässt
  solche zurück
- deaktivierte Trigger, die eine `--disable-triggers`-Wiederherstellung
  ausgeschaltet lässt, wenn sie unterbrochen wurde

Wählen Sie ein Schema oder lassen Sie *(gesamte Datenbank)* stehen. Zu jedem
Fund gehört die Anweisung, die ihn behebt, und ein ungültiger Index bekommt
eine Schaltfläche **Jetzt beheben**, die `REINDEX INDEX CONCURRENTLY` darauf
als Hintergrundaufgabe ausführt.

Eines ist zu erwarten: ein fehlgeschlagenes `REINDEX CONCURRENTLY` lässt seine
eigene halb gebaute Kopie mit dem Suffix `_ccnew` zurück, ein eigenes Objekt
neben dem ursprünglichen. Das Reparieren des Originals entfernt sie nicht -
führen Sie die Prüfung danach erneut aus und beheben Sie auch diese.

### Fenster erscheint nicht (Linux)
DBApguin öffnet einen Dialog mit Installationsbefehlen je Distribution, wenn WebKit2GTK fehlt. Wählen Sie den Reiter Ihrer Distribution, klicken Sie „Befehl kopieren", fügen Sie ihn im Terminal ein und starten Sie das AppImage neu. Der Dialog weicht nie auf einen Webbrowser aus - DBApguin läuft ausschließlich eigenständig.

### Fenster erscheint nicht (Windows)
DBApguin öffnet einen Dialog mit drei Installationswegen für die Microsoft Edge WebView2 Runtime: den mitgelieferten Bootstrapper aus dem Anwendungsordner, den Online-Installer und Hinweise zur Verteilung im Unternehmen (GPO/SCCM/Intune).

### Fenster hängt beim Schließen (Windows)
Sollte seit dem Build vom Juni 2026 behoben sein. Die Anwendung beendet sich 2 Sekunden nach dem Klick auf das X erzwungen. Hängt sie weiterhin, legen Sie einer Fehlermeldung bitte den Inhalt von `<DBCHECKS_HOME>\dbchecks_error.log` bei.

### DBApguin startet nicht: der Port ist belegt
- Wenn ein anderes Programm den Port von DBApguin schon verwendet, bleibt das
  Fenster offen und sagt es, mit einem Feld, um auf einem anderen Port zu
  starten - ein freier ist schon eingetragen. DBApguin behält diesen Port;
  ändern Sie ihn später unter **Einstellungen -> Start**.
- Wenn es meldet, dass DBApguin bereits läuft, ist eine frühere Kopie noch da:
  wechseln Sie zu ihrem Fenster, oder beenden Sie den genannten Prozess und
  wählen Sie **Erneut versuchen**.

### Der Schema-Browser ist leer
- Sind Sie verbunden? Prüfen Sie den Statuspunkt neben der Verbindung.
- Haben Sie die Datenbank gewechselt? Klicken Sie unter **Datenbanken** auf eine andere Datenbank; der Schemabaum lädt mit ihr neu.
- Fahren Sie mit der Maus über die Kopfzeile **Datenbanken**, eine Datenbank oder ein Schema und klicken Sie auf das erscheinende Auffrischen-Symbol, oder nutzen Sie **Schema -> Schemabaum neu laden**.

### Kein Audit-Log
- Es ist eingeschaltet, bis es ausgeschaltet wird: Schalter unter Ansicht -> Audit-Protokoll prüfen
- Oder den Schlüssel `audit_enabled` in `app_prefs.json` in Ihrem Datenverzeichnis prüfen
- Die Dateien liegen unter `<DBCHECKS_HOME>/audit/audit-YYYY-MM.jsonl`

### Wo die Anwendungsdaten liegen
Unter Linux: `~/.local/share/DBApguin/`. Überschreiben mit `DBCHECKS_HOME=/pfad .venv/bin/python launcher.py`.

Unter Windows: standardmäßig der Ordner neben `DBApguin.exe`.

## 12. Übersicht der Datendateien

| Datei | Inhalt | Rechte |
|------|--------------|------|
| `connections.json` | Gespeicherte Verbindungen mit Fernet-verschlüsselten Passwörtern | 0644 |
| `key` | Zufälliger Schlüssel für die Passwort-Chiffre, je Installation eigen | 0600 |
| `app_prefs.json` | Oberflächeneinstellungen (font_size_px, audit_enabled, theme und weitere) | 0644 |
| `certs/` | Hochgeladene TLS-Wurzelzertifikate | je 0600 |
| `audit/audit-YYYY-MM.jsonl` | DDL-Audit-Log, eine Datei je Monat | 0600 |
| `dbchecks_error.log` | Protokoll der Tracebacks (rotiert bei 1 MiB, hält 3 Sicherungen) | 0644 |
| `webview/` | pywebview localStorage / Cookies | (nur Linux) |

Sichern Sie dies vor einer Neuinstallation. Eine Neuinstallation in dasselbe Datenverzeichnis behält Ihre Verbindungen.

## 12.4 Wo die Dokumentation liegt

Drei Wege, alle über **Hilfe** in der Menüleiste:

| | |
|---|---|
| **Benutzerhandbuch** | dieses Dokument, mit der Anwendung ausgeliefert - funktioniert offline |
| **Dokumentation (online)** | die Website, für alles Neuere als Ihr Build |
| **Über DBApguin** | Version und Build, mit beidem daneben |

Das Lizenzfenster beim ersten Start und der Hinweis danach verlinken darauf - 
dort schauen die meisten beim ersten Mal.

Dokumente werden in der Anwendung im eigenen Stil dargestellt - Überschriften,
Tabellen und SQL-Beispiele in einem echten Codeblock statt als umbrochener Text.

Es gibt einen vierten Weg, und in der Praxis nutzen Sie ihn am häufigsten: das
kleine eingekreiste **i** neben der Überschrift eines Dialogs. Es öffnet dieses
Handbuch genau bei dem Abschnitt, der diesen Dialog beschreibt - die Erklärung
ist einen Klick von der Sache entfernt, die sie erklärt, statt eine Suche weit.
Sie finden es bei Backup und Zurückspielen, beim Umbenennen und Zusammenführen
von Schemas und neben dem Transaktionsmodus des SQL-Editors.

Die mitgelieferte Fassung gehört zu Ihrem Build und passt daher immer zur
laufenden Version. Die Online-Fassung kann ihr voraus sein.

## 12.5 Recht, Datenschutz und Software Dritter

Alles Folgende ist über **Hilfe** in der Menüleiste erreichbar und wird mit der
Anwendung ausgeliefert - die angezeigte Datei ist die, mit der gebaut wurde.

- **Lizenzvertrag (EULA)** - was Sie mit der Software dürfen und was nicht, der
  Gewährleistungsausschluss und die Haftungsgrenze. Abschnitt 7 lohnt sich auch,
  wenn Sie den Rest überspringen: er handelt davon, dass dieses Werkzeug auf Ihre
  Anweisung hin `DROP`, `TRUNCATE` und `pg_terminate_backend` an laufende Server
  schickt.
- **Datenschutzhinweise** - was auf Ihrem Rechner gespeichert wird und wo. Kurz:
  alles ist lokal, es gibt keine Telemetrie, und die einzigen Netzwerkverbindungen
  gehen zu Ihren Datenbanken und zu `127.0.0.1`.
- **Hinweise zu Dritten** - jede Open-Source-Komponente, die der Build einbindet,
  mit ihrer Lizenz. Aus dem Build selbst erzeugt
  (`tools/legal/gen_third_party.py`) statt von Hand gepflegt, sie kann also
  keinen Abhängigkeitsstand beschreiben, den es nicht mehr gibt.

Die rechtlichen Dokumente sind auf Englisch verfasst, und Englisch ist maßgeblich.
Übersetzungen der Oberfläche erstrecken sich absichtlich nicht auf sie: ein
übersetzter Lizenzvertrag, der in dieser Sprache keinen Juristen gesehen hat, ist
schlechter als keine Übersetzung, weil er verbindlich aussieht.

## 12.6 Warum es auf jedem Rechner gleich aussieht

DBApguin liefert die zwei Schriften mit, die es verwendet - Inter für die
Oberfläche und Cascadia Mono für alles Dicktengleiche - , statt Schriften zu
benennen und zu hoffen, dass der Rechner sie hat.

Das ist keine Kosmetik. Ein Werkzeug, das Bezeichner anzeigt, muss sie genau
anzeigen: als die Anwendung eine nicht installierte Schrift nannte, ersetzte das
System sie durch eine andere, und ein Ersatz zeichnete den Unterstrich unterhalb
der Zeilenbox - `pg_read_all_stats` konnte auf einem Rechner als
`pgreadallstats` gelesen werden und auf einem anderen richtig. Die Schrift
mitzuliefern nimmt das Raten heraus.

Beide stehen unter der SIL Open Font License 1.1 und reisen unverändert mit,
ihre Lizenztexte liegen in `static/fonts/` und unter
**Hilfe → Komponenten Dritter**.

## 13. Umstieg von Oracle {#coming-from-oracle}

Wer jahrelang Oracle-Datenbanken betrieben hat, kann das meiste übertragen. Hier
steht der Teil, der sich nicht überträgt, und wo die vertrauten Zahlen in dieser
Anwendung liegen.

### 13.1 Die Zuordnung

| Oracle | PostgreSQL | Wo in DBApguin |
| --- | --- | --- |
| Instanz + Datenbank (meist eine je Instanz) | Cluster + viele Datenbanken; eine Prozessgruppe bedient alle | Server-Leiste; jede Datenbank ist ein Knoten |
| Schema == Benutzer | Schema und Rolle sind getrennte Objekte; eine Rolle kann viele Schemas besitzen | Schema-Browser; Ansicht Rollen |
| Tablespace (verpflichtend, dimensioniert) | Tablespace (optional, nur ein Verzeichnis) | `db-tablespaces` |
| Undo-Tablespace, Rollback-Segmente | Kein Undo: alte Zeilenversionen bleiben in der Tabelle, bis Vacuum sie entfernt | Autovacuum-Berater, `db-bloat-breakdown` |
| Redo- und Archive-Logs | WAL-Segmente + Archivierung | `db-checkpoints`, `db-settings` (WAL- und Archivierungs-GUCs) |
| SGA / Buffer Cache | `shared_buffers` + der Seitencache des Betriebssystems | `db-settings`, `db-cache-advisor`, `db-buffer-cache` |
| PGA / Sortierbereich | `work_mem`, je Operation je Backend | `db-settings` (die Warnung dort betrifft meist das) |
| AWR-Bericht | `db-workload-report` über Deltas aus pg_stat_statements | Menü Speicher und Leistung |
| AWR-Snapshot / Baseline | `db-awr-snapshot` (take / list / compare) | Menü Speicher und Leistung |
| ASH / v$active_session_history | `db-ash`-Sampler über pg_stat_activity | Menü Verbindungen und Abfragen |
| v$session | Ansicht Sitzungen; ein Klick auf eine Zeile öffnet die Tiefe | Ansicht Sitzungen |
| v$sql / Library Cache | `pg_stat_statements`, `db-stat-statements` | Menü Verbindungen und Abfragen |
| SQL Trace / 10046 | `db-trace-session` plus `auto_explain` auf dem Server | Menü Optimierung und Diagnose, *Sitzungs-Trace* |
| DBMS_STATS | `ANALYZE`, die Analyse-Hälfte des Autovacuums | Reiter Berater an einer Tabelle |
| Optimizer-Hints | Keine Hints. Man ändert Statistiken, Indizes oder die Abfrage | `db-index-advisor`, EXPLAIN in der SQL-Ansicht |
| RMAN | `pg_dump` / `pg_restore` (logisch), `pg_basebackup` (physisch) | Backup und Zurückspielen |
| Data Guard | Streaming-Replikation + Replikations-Slots | `db-replication`, `db-repl-slots` |
| Refresh materialisierter Views (fast/complete) | `REFRESH MATERIALIZED VIEW [CONCURRENTLY]`, immer ein vollständiger Neuaufbau | Rechtsklick auf eine MatView |
| DBA_* / V$-Views | `pg_catalog.pg_*` und `pg_stat_*`-Views | Jede Prüfung druckt das ausgeführte SQL |
| Ungültige Objekte, Recompile | Diesen Zustand gibt es nicht: eine Funktion wird beim Ausführen geparst | die Prüfung nach dem Import, die nach einem Restore automatisch läuft |
| Öffentliche Synonyme | `search_path` oder eine View in einem gemeinsamen Schema | `db-settings` |
| Sequenzen mit NOCACHE / ORDER | Sequenzen cachen je Sitzung; Lücken sind normal und kein Fehler | `db-sequences`, Sequenz-Gesundheitsprüfung |
| Profile, Resource Manager | Nicht eingebaut. Verbindungsgrenzen je Rolle, dazu ein Pooler | `db-connections` |

### 13.2 Die fünf Unterschiede, die wirklich weh tun

**1. Es gibt kein Undo - Vacuum ist der Preis für MVCC.** Ein `UPDATE` schreibt
eine neue Zeilenversion und lässt die alte zurück, tot, aber weiter auf der
Seite. Autovacuum entfernt tote Versionen später. Zwei Folgen ohne Oracle-Pendant:
eine Tabelle kann wachsen, auch wenn die Zeilenzahl es nicht tut (Bloat), und eine
stundenlang offene Transaktion hindert Vacuum daran, *irgendeine* Tabelle zu
säubern, weil die alten Versionen für sie noch sichtbar sein könnten. `db-long-tx`
und `db-idle-in-tx` finden solche; der Reiter Berater an einer Tabelle zeigt
genau, wie viele tote Zeilen es braucht, bis Autovacuum handelt.

**2. Schemas sind keine Benutzer.** `CREATE USER app` schafft für `app` keinen Ort
für Tabellen. Objekte landen im ersten Schema des `search_path` - üblicherweise
`public`. Gewähren Sie `USAGE` auf dem Schema *und* Rechte an den Objekten: das
sind zwei Dinge, und ein fehlendes `USAGE` ist das häufigste „permission denied",
das unerklärlich aussieht. Die Rollenansicht zeigt beide Ebenen, und der
Rechteeditor schreibt beide.

**3. DDL ist transaktional.** `BEGIN; ALTER TABLE ...; ROLLBACK;` macht das
`ALTER` tatsächlich rückgängig. Migrationen können alles-oder-nichts sein, ein
echter Vorteil - aber es heißt auch, dass eine Migration mit `ACCESS EXCLUSIVE`
auf einer heißen Tabelle jeden Leser blockiert, bis sie committet. `db-locks` und
die Sitzungsansicht zeigen, wer hinter wem wartet.

**4. Keine Hints, und die Statistik ist die ganze Geschichte.** Ist ein Plan
falsch, lautet die Antwort `ANALYZE`, ein besserer Index, ein angepasstes
`default_statistics_target` oder eine umgeschriebene Abfrage. Der Reiter Berater
meldet, wann eine Tabelle zuletzt analysiert wurde; `db-index-advisor` meldet
fehlende Indizes, auch die Fremdschlüssel ohne Index dahinter - was in PostgreSQL
dazu führt, dass Löschen in der Elterntabelle das Kind scannt.

**5. Verbindungen sind Prozesse, keine Threads.** Jede Verbindung ist ein Prozess
mit eigenem Speicher. Mehrere hundert untätige Verbindungen kosten echten
Arbeitsspeicher und verlangsamen jede Momentaufnahme von `pg_stat_activity`. Wo
Oracle Shared Server nutzt, nutzt PostgreSQL einen externen Pooler (PgBouncer).
`db-connections` zeigt, wie nah Sie an `max_connections` sind und wie viele davon
untätig sind.

### 13.3 Dinge ohne Oracle-Pendant

- **TOAST.** Werte über etwa 2 KB werden komprimiert und in einer Nebentabelle
  abgelegt. Die eigene Größe einer Tabelle kann klein wirken, während ihre
  TOAST-Tabelle die meisten Daten hält; `db-bloat-breakdown` trennt Heap, TOAST
  und Indizes genau deshalb.
- **Erweiterungen.** Vieles, was in Oracle eingebaut ist, ist hier ein
  installierbares Modul: `pg_stat_statements`, `pgcrypto`, `postgis`. Siehe
  Abschnitt 5.14.
- **Überlauf der Transaktions-ID.** Der 32-Bit-Transaktionszähler muss regelmäßig
  eingefroren werden, sonst verweigert der Cluster Schreibzugriffe.
  `db-vacuum` meldet das Alter; es ist die eine Autovacuum-Aufgabe, die
  man nie abschalten darf.
- **`NULL` und der leere String sind verschieden.** Oracle behandelt `''` wie
  `NULL`; PostgreSQL nicht. Aus Oracle migrierte Daten tragen diesen Unterschied
  oft stillschweigend mit.

## 14. PostgreSQL für Entwickler {#postgresql-for-developers}

Sie müssen kein DBA werden, um eine PostgreSQL-Datenbank gesund zu halten. Sie
brauchen ein Denkmodell und fünf Gewohnheiten.

### 14.1 Das Modell: Zeilen sind Versionen

Jedes `UPDATE` und `DELETE` lässt die alte Version der Zeile an Ort und Stelle,
als tot markiert. Leser blockieren nie Schreiber, weil jede Transaktion die
Versionen sieht, die bei ihrem Beginn aktuell waren. Daraus folgen zwei Dinge,
und fast alles in dieser Anwendung dreht sich um eines davon:

- **Tote Zeilen sammeln sich**, bis Autovacuum sie entfernt. Bis dahin belegen sie
  Platz und bremsen Scans. Das ist *Bloat*.
- **Vacuum kann nur Versionen entfernen, die keine laufende Transaktion mehr sehen
  kann.** Eine vergessene offene Transaktion friert die Aufräumarbeit der ganzen
  Datenbank ein.

`ANALYZE` ist die andere Hälfte: es tastet die Daten neu ab, damit der Planer
weiß, wie viele Zeilen eine Bedingung trifft. Veraltete Statistiken sind die
übliche Ursache für „diese Abfrage war letzte Woche schnell".

### 14.2 Wenn jemand sagt „die Datenbank ist langsam"

1. **Ansicht Sitzungen.** Wie viele Verbindungen, wie viele laufen wirklich, ist
   etwas blockiert. Eine blockierte Abfrage ist ein Sperrproblem, kein
   Leistungsproblem.
2. **`db-health`.** Ein Bildschirm: Verbindungen, Cache-Trefferrate, lange
   Transaktionen, Bloat, Sperren.
3. **`db-stat-statements`.** Die 25 Abfragen mit der meisten Zeit. Fast immer
   steht die Antwort hier - meist eine Abfrage, die weit öfter aufgerufen wird,
   als irgendwer erwartet hat.
4. **EXPLAIN der Abfrage** in der SQL-Ansicht. Achten Sie auf `Seq Scan` an einer
   großen Tabelle und auf eine geschätzte Zeilenzahl, die stark von der
   tatsächlichen abweicht - Letzteres heißt, die Statistik ist veraltet.
5. **`db-index-advisor`.** Fehlende Indizes und nicht indizierte Fremdschlüssel.

### 14.3 Fünf Gewohnheiten

1. **Lassen Sie Transaktionen nicht offen.** Eine ORM-Sitzung, die eine
   Transaktion geöffnet hat und in die Mittagspause ging, ist das Schädlichste,
   was eine Entwicklerin einer PostgreSQL-Datenbank antun kann. `db-idle-in-tx`
   findet sie.
2. **Indizieren Sie Ihre Fremdschlüssel.** PostgreSQL legt den Index nicht für
   Sie an, und ohne ihn scannt jedes Löschen oder Ändern am Elternteil die
   Kindtabelle.
3. **`ANALYZE` nach Massenladungen.** Eine Million eingefügter Zeilen ohne
   Statistik dahinter liefert schlechte Pläne, bis Autovacuum dazu kommt.
4. **In Stapeln arbeiten, aber nicht zu groß.** Eine Transaktion, die zehn
   Millionen Zeilen löscht, hält Sperren, erzeugt WAL und hinterlässt zehn
   Millionen tote Versionen. Schleifen Sie in Stapeln von einigen Tausend und
   committen Sie.
5. **Nutzen Sie einen Verbindungspool.** Eine Verbindung je Anfrage kostet einen
   Prozess-Fork. Lesen Sie `db-connections`, bevor Sie `max_connections` erhöhen.

### 14.4 Die Wörter, die diese Anwendung verwendet {#the-words-this-app-uses}

- **Bloat** - Platz, den tote Zeilenversionen halten und der nie an das
  Betriebssystem zurückgeht.
- **Totes Tupel** - eine alte Zeilenversion, die auf Vacuum wartet. „Tupel" heißt
  hier „Zeile".
- **Autovacuum** - der Hintergrundprozess, der tote Tupel entfernt und Statistiken
  auffrischt. Er löst je Tabelle aus, an Schwellen, die im Reiter Berater stehen.
- **VACUUM gegenüber VACUUM FULL** - einfaches `VACUUM` markiert Platz als
  wiederverwendbar und blockiert nie Leser. `VACUUM FULL` schreibt die ganze
  Tabelle neu, gibt Platz ans Betriebssystem zurück und hält eine Sperre, die
  *alles* an dieser Tabelle blockiert. `REINDEX CONCURRENTLY` und `pg_repack`
  sind die Online-Alternativen.
- **WAL** - das Write-Ahead-Log. Jede Änderung wird zuerst dort geschrieben;
  Replikation und Wiederherstellung lesen daraus.
- **Checkpoint** - das regelmäßige Schreiben schmutziger Seiten in die
  Datendateien. Zu häufig heißt Schreibspitzen; `db-checkpoints` meldet das
  Verhältnis zeitgesteuerter zu angeforderten.
- **search_path** - die Liste der Schemas, in denen ein unqualifizierter Name
  gesucht wird. Sie entscheidet, wo `CREATE TABLE x` landet und welche Funktion
  `f()` trifft.
- **Rolle** - der eine Begriff hinter Benutzern und Gruppen. Eine Rolle mit
  `LOGIN` ist das, was andere Datenbanken Benutzer nennen.
- **Lücken in Sequenzen sind normal.** Sequenzen sind nicht transaktional: ein
  zurückgerollter Insert hat seine Nummer trotzdem verbraucht.

## Fehler melden

**Einstellungen -> Fehler melden** stellt den Bericht für Sie zusammen: eine
Fehler-ID, die Version, den Commit, Ihr Betriebssystem und die letzten vierzig
Zeilen des Fehlerprotokolls. Sie füllen drei Abschnitte aus - was Sie getan
haben, was Sie erwartet haben, was stattdessen geschah - und drücken einen von
drei Knöpfen.

**Mailprogramm öffnen** übergibt den Text Ihrem eigenen Mailprogramm, adressiert
an **dbapguin@gmail.com**. Die Anwendung selbst sendet nichts und hat das nie
getan; Sie sehen zuerst den genauen Text und drücken selbst auf Senden.
**Bericht kopieren** und **In Datei speichern** sind da, falls Sie ihn lieber
anhängen oder woanders einfügen möchten.

**Was entfernt wird, bevor etwas Ihren Rechner verlässt**: Passwörter,
Benutzernamen, Datenbanknamen, Hostnamen, Verbindungs-URIs, IP-Adressen und der
Pfad Ihres Benutzerverzeichnisses. Der Dialog listet auf, was entfernt wurde -
Sie sehen es, statt es glauben zu müssen.

Nennen Sie die Fehler-ID (`DBAP-261011-K4QP` und ähnlich) bei jeder Rückfrage;
daran wird ein Bericht vom anderen unterschieden.

## 15. Wie Sie Hilfe bekommen

- Interne Dokumentation: diese Datei plus `docs/TECHNICAL.md` und `ARCHITECTURE.md`
- Herkunft des Builds: **Hilfe -> Über DBApguin** zeigt Version, Git-Commit, Zeitpunkt des Builds und das verwendete Python. Legen Sie das Fehlermeldungen bei.
- Audit-Log: `<DBCHECKS_HOME>/audit/audit-YYYY-MM.jsonl` ist eine JSON-Lines-Datei. Mit jedem Texteditor oder mit `jq` zur Auswertung zu öffnen.
- Fehlerprotokoll: `<DBCHECKS_HOME>/dbchecks_error.log` für die jüngsten serverseitigen Tracebacks. Passwörter werden vor dem Schreiben entfernt. Es rotiert bei 1 MiB und hält drei ältere Kopien, und jeder Lauf, der etwas protokolliert, schreibt zuerst eine Kopfzeile - die Einträge nach der letzten Zeile `=== DBApguin ... started ...` gehören also zu Ihrer laufenden Sitzung. Die darüber können Wochen alt sein.

Für kommerziellen Support wenden Sie sich an den Hersteller mit:
1. Version des Builds (aus Hilfe -> Über DBApguin).
2. Anbieterschlüssel (aus den Verbindungsdetails).
3. Version des PostgreSQL-Servers.
4. Einer Kopie des betreffenden Abschnitts aus Audit- und Fehlerprotokoll, falls vorhanden.
