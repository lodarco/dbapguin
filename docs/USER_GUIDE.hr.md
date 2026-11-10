# DBApguin - korisničke upute

**Za DBApguin 1.0.1**

*Ove su upute prevedene s engleskog uz pomoć AI-a. Ako je nešto nejasno ili se razlikuje, mjerodavan je engleski izvornik (Pomoć → Korisničke upute, uz engleski jezik sučelja).*

DBApguin je desktop konzola za PostgreSQL DBA poslove. Ove upute pokrivaju
svaku funkcionalnost iz korisnikove perspektive i isporučuju se s aplikacijom,
pa uvijek opisuju build koji imate pred sobom.

Otvorite ih kad god želite iz **Pomoć → Korisničke upute**.

## 1. Instalacija

### Linux

1. Preuzmite `DBApguin-<verzija>-x86_64.AppImage`.
2. Učinite ga izvršnim: `chmod +x DBApguin-*.AppImage`
3. Dvoklik ili pokretanje iz terminala.

AppImage je samodostatan (~27 MB) i sprema podatke u `~/.local/share/DBApguin/`.

Ako se ugrađeni prozor ne pojavi, AppImage prikaže dijalog s uputama za instalaciju za svaki veći package manager (apt, dnf, pacman, zypper, apk, emerge). Dijalog sam odabere karticu koja odgovara vašoj distribuciji i nudi gumb "Kopiraj naredbu". Za Debian/Ubuntu naredba je:

```
sudo apt-get install python3-gi python3-gi-cairo gir1.2-gtk-3.0 gir1.2-webkit2-4.1
```

DBApguin je samostalna desktop aplikacija: ako WebKit2GTK nije dostupan, NEĆE tiho otvoriti karticu u vašem web pregledniku. Instalirajte paket pa ponovno pokrenite AppImage.

### Windows

1. Raspakirajte DBApguin zip bilo gdje (Desktop, `C:\Tools\` i slično).
2. Otvorite mapu.
3. Dvoklik na `DBApguin.exe`.

Prvo pokretanje može prikazati SmartScreen ("Windows protected your PC"). Kliknite "More info" pa "Run anyway". DBApguin još nije potpisan certifikatom za Windows.

Ako nedostaje Microsoft Edge WebView2 Runtime, aplikacija otvara dijalog s tri načina instalacije: (1) pokretanje priloženog `MicrosoftEdgeWebView2Setup.exe` iz mape aplikacije, (2) preuzimanje online bootstrappera s `developer.microsoft.com/microsoft-edge/webview2/`, ili (3) postavljanje samostalnog instalera kroz GPO / SCCM / Intune u upravljanim okruženjima. Svaki blok ima gumb "Kopiraj" za putanje i poveznice.

Aplikacija se ne instalira u sustav; radi iz svoje mape. Za uklanjanje obrišite mapu.

Dijalog za postavljanje čita jezik iz vaših DBApguin postavki (`app_prefs.json` -> `language`); pri prvom pokretanju slijedi jezik sustava (hrvatski za `hr_*`, inače engleski).

## 1.4 Podržane verzije PostgreSQL-a

DBApguin cilja na **PostgreSQL 13 i noviji**. To je verzija na kojoj se testira i za koju je pisana svaka provjera.

Radi i na starijim poslužiteljima, a pravilo je **napravi što verzija dopušta, ne odbijaj**. Provjera koja može izostaviti stupac kojeg poslužitelj nema, ili preskočiti sekciju kojoj treba nedostupan pogled, upravo to i napravi i ispiše kratku napomenu što je izostavljeno. Samo provjera čija je *cijela* tema značajka koje poslužitelj nema - logička replikacija, koja je stigla cijela u PostgreSQL 10 - prikaže karticu "Treba PostgreSQL N" umjesto da se izvrši. Konkretno, na poslužitelju starom kao **9.4**: lista sesija, live load i prikaz blokiranja rade - prikaz blokiranja se vraća na pg_locks self-join gdje nema `pg_blocking_pids()` (9.6), a lista sesija prikazuje `waiting` zastavicu gdje nema wait eventa (9.6). `Blocking pairs`, `Long transactions`, `Connection limits`, `Autovacuum activity`, `Replication`, `Table sizes`, `Replication slots`, procjene napuhnuća, replica identity i health pregled svi grade upit prilagođen verziji i navedu što je grublje. Deklarativno particioniranje je 10+, pa na starijem poslužitelju stablo sheme prikazuje djecu iz nasljeđivanja kao tablice ispod tablice od koje nasljeđuju, nikad kao particije.

**PostgreSQL 10, 11 i 12** su potpuno povezivi i većina aplikacije radi nepromijenjeno; nekoliko panela nadzorne ploče koji čitaju novije statističke poglede tiho ostaje prazno.

Ispod **PostgreSQL 10** si izvan onoga što je testirano. Spajanje radi, SQL uređivač / preglednik sheme / alati za sigurnosnu kopiju su u redu, i provjere koje se mogu prilagoditi to i čine; ali ovo je best-effort, i obavijest pri spajanju to kaže. Ispod **9.4** su se katalozi razišli dovoljno da DBApguin uopće ne tvrdi da radi.

Nikad ne vidiš sirovo `column "..." does not exist` iz provjere: kad je uzrok to što je poslužitelj stariji nego što provjera očekuje, provjera to kaže riječima.

### PostgreSQL 19

DBApguin je izmjeren i na PostgreSQL 19: svaka provjera radi na njemu, sa i
bez contrib ekstenzija, kao superuser i kao obična uloga. Beta ili release
candidate to kaže pri spajanju, jer se detalji još mogu promijeniti do
konačnog izdanja. Glavna verzija novija od 19 spaja se i radi, uz napomenu
da je novija od svega izmjerenog.

Na 19 aplikacija koristi ono što je izdanje donijelo:

- **REPACK** u panelu Održavanje i u savjetima za bloat. Prepisuje tablicu i
  vraća prostor, a s CONCURRENTLY to radi dok čitanja i pisanja teku dalje.
  Stariji serveri zadržavaju VACUUM FULL i CLUSTER.
- **Serverov vlastiti red autovacuuma** u Vacuum statusu: koje tablice
  autovacuum uzima sljedeće i zašto, uključujući sprječavanje wraparounda.
- **Statistika lockova** pod Locks: čekanja po tipu locka od zadnjeg reseta i
  koliko je puta fast path ostao bez slotova.
- **Kako je vacuum pokrenut i koliko jako radi** (aggressive, failsafe, radi
  sprječavanja wraparounda), u Live loadu i u savjetniku za vacuum.
- **Vrijeme reseta po indeksu** u Nekorištenim indeksima, pa indeks kojemu su
  brojači tek resetirani nije proglašen nekorištenim.
- **Broj izvršenja s generičkim i s prilagođenim planom** po naredbi u Top
  upitima.
- **EXPLAIN ANALYZE s I/O podacima**: zahtjevi koje je skeniranje poslalo,
  njegova čekanja i koliko je unaprijed čitalo.
- **Replikacija**: greške i konflikti po pretplati, koliko je puta logičko
  dekodiranje preraslo svoju memoriju i stanje oporavka replike.
- **Logički slotovi uz wal_level = replica**: 19 sam podiže razinu WAL-a, pa
  dijalog za slotove to dopušta i kaže što to košta.
- **Isključivanje ograničenja**: upravitelj ograničenja može strane ključeve
  (od 18) i provjere (od 19) postaviti na NOT ENFORCED umjesto brisanja.
- **Savjeti za postavke prema zadanim vrijednostima verzije 19**: JIT
  isključen, drukčije dimenzionirana tablica lockova, zapisivanje čekanja na
  lockove uključeno, upozorenja za MD5 prijave, RADIUS uklonjen iz
  pg_hba.conf.

Backup prvo provjerava da pg_dump na ovom računalu nije stariji od servera,
što je uobičajeno stanje računala na dan kad izađe nova glavna verzija; kaže i
koji klijentski paket treba instalirati. Prije nadogradnje pokreni **Spremno za
PostgreSQL 19?** (vidi [provjeru spremnosti](#ready-for-postgresql-19)).

## 1.5 Preduvjeti na poslužitelju



DBApguin radi s običnim PostgreSQL klasterom bez posebnih priprema. Neke naprednije funkcije traže da su na poslužitelju uključene ekstenzije ili GUC parametri. Najbrži način da doznate što nedostaje je ugrađeni savjetnik za postavljanje:

> Gornji meni -> Zdravlje klastera -> "DBApguin savjetnik za postavljanje"

On ispita klaster i po funkciji ispiše stanje (OK / DJELOMIČNO / NEDOSTAJE) te točan SQL ili redak u postgresql.conf kojim uključujete ono što nedostaje. Na upravljanim oblacima (RDS, Aurora, Cloud SQL, Azure, Supabase, Neon) savjet se mijenja u parameter grupu pružatelja, jer je ALTER SYSTEM tamo blokiran.

Brzi pregled, po važnosti:

| Što želite koristiti | Uvjet na poslužitelju | Kako uključiti |
|--|--|--|
| Najskuplji upiti, povijest upita, izvještaji u AWR stilu | ekstenzija pg_stat_statements učitana kroz `shared_preload_libraries` | Dodajte `pg_stat_statements` u `shared_preload_libraries` + restart, pa `CREATE EXTENSION pg_stat_statements` u svakoj bazi za koju želite statistiku |
| Hvatanje planova tijekom praćenja | ekstenzija auto_explain + `log_min_duration` >= 0 | Dodajte `auto_explain` u `shared_preload_libraries` + restart, pa `SET auto_explain.log_min_duration = 500` (ili 0 za sve) |
| Query_id u pg_stat_activity, povezivanje ASH-a s pg_stat_statements | `compute_query_id = on` | `ALTER SYSTEM SET compute_query_id = on; SELECT pg_reload_conf();` (PG14+) |
| Realno I/O vrijeme u pg_stat_statements / EXPLAIN (BUFFERS) | `track_io_timing = on` | `ALTER SYSTEM SET track_io_timing = on; SELECT pg_reload_conf();` |
| Cijeli tekst upita u pg_stat_activity i ASH-u | `track_activity_query_size >= 4096` | `ALTER SYSTEM SET track_activity_query_size = 4096;` pa RESTART |
| Potpuni Top SQL u AWR-u (bez naredbi izbačenih između snimaka) | `pg_stat_statements.max` dovoljno velik za različite naredbe | `ALTER SYSTEM SET pg_stat_statements.max = 10000;` pa RESTART - AWR usporedba kaže kad su naredbe izbačene |
| Statistika poziva funkcija | `track_functions = 'pl'` | `ALTER SYSTEM SET track_functions = 'pl'; SELECT pg_reload_conf();` |
| Snimka predmemorije u db-cache-advisor | ekstenzija pg_buffercache | `CREATE EXTENSION pg_buffercache;` (superuser) |
| Online prepisivanje tablice (Održavanje -> pg_repack) | ekstenzija pg_repack + klijentski program | Instalirajte pg_repack paket na poslužitelju i na računalu s DBApguinom, pa `CREATE EXTENSION pg_repack;` |
| Čitanje kraja server loga u db-errors | `logging_collector = on`, `log_destination` uključuje stderr, dodijeljen `pg_read_server_files` | Samo za vlastite poslužitelje. Upravljani oblaci blokiraju pg_read_file. |
| Detalji čekanja na lockove u server logu | `log_lock_waits = on`, `deadlock_timeout = '1s'` | `ALTER SYSTEM SET log_lock_waits = on;` |
| Log sporih upita | `log_min_duration_statement >= 0` | `ALTER SYSTEM SET log_min_duration_statement = '1s';` (ili `'100ms'` za strože) |
| Revizijski zapis na poslužitelju (uz onaj u aplikaciji) | ekstenzija pgaudit ili `log_statement = 'ddl'` | Dodajte `pgaudit` u `shared_preload_libraries`, restart, `CREATE EXTENSION pgaudit;` |
| Da provjere može pokretati i ne-superuser | Dodijelite roli `pg_monitor` | `GRANT pg_monitor TO <rola>;` (PG10+) |

Ako uključujete samo jednu stvar, neka to bude `pg_stat_statements`. Većina DBApguin funkcija postaje bitno korisnija čim je on učitan.

## 2. Prvo postavljanje

Pri prvom pokretanju lijevo vidite prazno stablo Serveri.

1. Kliknite `+ Nova` u alatnoj traci Servera.
2. Ispunite:
   - Alias (naziv za prikaz)
   - Grupa (zadano "default")
   - Neobavezna boja (pomaže da na prvi pogled razlikujete produkciju od testa)
   - Host, Port, Korisnik, Lozinka, Baza, sslmode
   - Neobavezni sslrootcert (učitava se kroz Postavke -> Certifikati)
3. Kliknite `Spremi i spoji se` ili samo `Spremi`.

Veza se pojavi u stablu Servera. Jedan klik je otvara. Stablo raširi nadređenu grupu i doskrola do retka.

Lozinke se spremaju šifrirane ključem koji je jedinstven za tu instalaciju. Datoteka s ključem stoji uz `connections.json` u vašem podatkovnom direktoriju.

### Čemu služi certifikat, a što zapravo uključuje TLS

To su dvije odvojene postavke, i certifikat je druga, ne prva.

**`sslmode` je ono što uključuje enkripciju.** Stoji na samoj konekciji:

| `sslmode` | Što dobiješ |
| --- | --- |
| `disable` | Bez enkripcije |
| `prefer` (zadano) | Šifrirano ako poslužitelj nudi, inače otvoreno - bez upozorenja |
| `require` | Enkripcija je obavezna, ali se ništa ne provjerava: prihvaća se bilo koji certifikat, uključujući onaj koji je napadač upravo generirao |
| `verify-ca` | Certifikat poslužitelja mora voditi do korijena kojem vjeruješ |
| `verify-full` | …i ime hosta na koje si se spojio mora odgovarati certifikatu |

Većina stane na `require`, a to je upravo postavka koja zvuči sigurno a nije:
šifrira promet i onda ga preda onome tko se javio.

**Certifikat je korijen prema kojem `verify-ca` i `verify-full` provjeravaju.**
libpq ga traži u `~/.postgresql/root.crt`. Ako je certifikat tvog poslužitelja
izdao javni CA koji je ondje već prisutan, ne trebaš ništa uploadati - postaviš
`sslmode` i gotovo. Certifikat dodaješ kad izdavatelj *nije* već povjerljiv:
tvoj interni PKI, ili CA bundle cloud providera (AWS, Google i Azure svaki
objavljuju svoj).

Dakle:

1. Uploadaj PEM pod **Postavke → Certifikati**. Kopira se u
   `<DBCHECKS_HOME>/certs/` u modu 0600.
2. Otvori konekciju i postavi **sslmode** na `verify-ca` ili `verify-full`.
3. Odaberi uploadanu datoteku u **TLS root CA certifikat** na toj istoj
   konekciji.

Upload bez koraka 2 ne mijenja ništa: datoteka je spremljena, referencirana i
nikad konzultirana, jer na `prefer` ili `require` nema provjere koja bi je
konzultirala.

## 2.3 Prihvaćanje licencnog ugovora

Pri prvom pokretanju DBApguin prikazuje licencni ugovor za krajnjeg korisnika i
ne otvara se dok ga ne prihvatite. Nema načina da ga zaobiđete - nema gumba za
zatvaranje, ni Esc, ni klika izvan njega - jer iza njega još nema ničega što
biste koristili.

Panel prikazuje sam ugovor, ne njegov sažetak, čitan iz iste datoteke koju
kasnije prikazuje Pomoć → Licencni ugovor. Ima **birač jezika**, a uz gumbe su
poveznice na izjavu o privatnosti i popis komponenti trećih strana.

**Odbij i izađi** ne bilježi ništa i zatvara prozor. Odbijanjem se ništa ne
instalira niti registrira.

**Prihvaćam** bilježi dvije stvari u `app_prefs.json`, isključivo na vašem
računalu: koju ste verziju prihvatili i kada. Ništa se nikamo ne šalje.
Zapisana verzija je razlog zašto vas se pita jednom, a ne pri svakom
pokretanju - i zašto ćete biti pitani ponovno ako se sam ugovor promijeni, jer
pristanak na jednu verziju nije pristanak na ono što piše u nekoj kasnijoj.

Ako ga ikad želite ponovno pročitati: **Pomoć → Licencni ugovor**.

## 2.4 Napomena pri pokretanju

Pri prvom pokretanju aplikacija prikazuje kratku napomenu uz logo: što radi i
da su pregledi koje nudi smjernica, a ne sigurnosna mreža. Tu je zato što ovaj
alat na vašu naredbu šalje `DROP` i `TRUNCATE` u produkciju, a to zaslužuje da
se kaže jednom prije prve upotrebe.

Napomena nosi i **birač jezika**, jer je prvo pokretanje trenutak kad taj izbor
znači, a u Postavke još nitko ne gleda. Promjena odmah mijenja napomenu koju
čitate, što je jedini način da potvrdite da ste odabrali pravi jezik. Ista
postavka poslije živi u **Postavke → Jezik**.

Označite **Ne prikazuj više** i napomena prestaje izlaziti. Za povratak:
**Postavke → Napomena pri pokretanju**.

Isključivanjem skrivate podsjetnik. Uvjeti na koje upućuje time se ne mijenjaju
 - oni su u Pomoć → Licencni ugovor.

## 2.5 Uvjeti korištenja

DBApguin je **besplatan za sve**, i za privatnu i za poslovnu upotrebu, sa svim
funkcijama.

Što to znači u praksi:

- **Sve je dostupno od prvog pokretanja**, na koliko god strojeva želiš:
  savjetnici, analiza radnog opterećenja, AWR snimke, povijest upita, analiza
  indeksa i napuhnutosti, alati za usporedbu i migraciju - sve.
- **Ništa se ne provjerava preko mreže**, pa se ništa ne javlja kući.
  Aplikacija razgovara samo s bazama koje sam postaviš - i, ako
  instaliraš pluginove, s onim na što su ti pluginovi postavljeni (poglavlje 7,
  Pluginovi).
- **Bez računa, bez prijave, bez registracije.**

Jedino što se od tebe traži je prihvaćanje licencnog ugovora pri prvom
pokretanju (odjeljak 2.3).

Ukratko: koristi slobodno, ne prodavaj i ne predstavljaj kao svoj rad.
Besplatno nije isto što i open source: izvorni kod nije objavljen. Cijeli tekst je pod **Pomoć -> Licencni ugovor**.

DBApguin je napravio Darko Lovrenčec.

## 3. Bočna traka Serveri

Lijevi panel ima tri odjeljka:

### 3.1 Serveri (vrh)

Popisuje svaku spremljenu vezu, grupiranu po grupi. Gumbi u alatnoj traci:

- `+ Nova`: registriraj vezu
- `Uredi`: uredi trenutno aktivnu vezu (alias, grupu, boju, host, lozinku)
- `Skupi sve`: skupi svaku grupu
- `Raširi sve`: raširi svaku grupu
- `Više` (...): otvori potpuni upravitelj vezama (uvoz, izvoz, masovna reorganizacija)

Meni desnog klika po vezi:
- Spoji se / Prebaci na karticu
- Uredi
- Osvježi vezu (ponovni unos lozinke, ponovno spajanje)
- Prikvači / Otkvači karticu
- Odspoji se
- Otvori SQL editor / psql / Sesije / Nadzornu ploču / Održavanje / Poslužitelj (samo na aktivnoj vezi)
- Podmeni provjera klastera
- Stvori bazu / tablicu / rolu
- Testiraj vezu
- Dupliciraj
- Obriši (obavijest s "Poništi" u trajanju 6 sekundi)

Točke stanja uz svaku vezu:
- Puna zelena: ovo je aktivna kartica
- Prazna zelena: kartica je otvorena, ali gledate drugu
- Crveni X: veza je izgubljena (poslužitelj ju je zatvorio)
- Bez točke: nije spojeno

### 3.2 Baze (vidljivo samo kad ste spojeni)

Popisuje svaku bazu na klasteru. Klik prebacuje aktivnu bazu. Polje za filtriranje je na vrhu.

**Desni klik na naslov BAZE** daje izbornik **Database** s trake, stavku po stavku - veze, pozadinske poslove, usporedbu, backup i vraćanje, stvaranje, kloniranje, preimenovanje, ovlasti i brisanje, ekstenzije - a nakon njih `Osvježi popis baza`. Stvaranje, kloniranje, preimenovanje i brisanje su zasivljeni, s razlogom, kad ih tvoja uloga ili pružatelj ne dopušta.

**Desni klik na redak baze** daje sve što se odnosi na tu bazu: SQL editor,
sesije, nadzornu ploču, održavanje, cijeli katalog provjera, stvaranje objekata,
backup i vraćanje, kloniranje, preimenovanje, ovlasti i brisanje.

#### Ekstenzije

`Ekstenzije` se otvara na jednom od tri popisa, a onaj koji obično želite je
treći:

- `Instalirane ekstenzije…` - što je u ovoj bazi već stvoreno
- `Dostupne ekstenzije (nisu instalirane)…` - što **poslužitelj** ima, a za što
  još nitko nije pokrenuo `CREATE EXTENSION`
- `Upravljaj ekstenzijama…` - sve, s filtrom i brojačem

Sve što poslužitelj isporučuje pojavljuje se ovdje, bez obzira je li stvoreno.
Iz istog popisa je instalirate, mijenjate joj verziju, premještate je u drugu
shemu ili je brišete. Ekstenzije koje poslužitelj uopće nema ne mogu se pojaviti:
njih prvo treba instalirati na stroj, obično paketom.

### 3.2.5 Role i korisnici

Stoji uz Baze, a ne unutar jedne, jer role vrijede na razini klastera: ne
pripadaju bazi, a smještanje ispod jedne govorilo bi suprotno.

**Desni klik na naslov ROLE I KORISNICI**:

- `+ Stvori rolu / korisnika…`
- `Pregled rola i korisnika` - puna revizija
- `Usporedi role / korisnike…` između dva poslužitelja
- `Izvezi SVE role kao .sql…`
- `Osvježi role`

**Klik na rolu** pokreće reviziju njenih ovlasti. **Desni klik na rolu** daje
njene radnje: ovlasti, `search_path`, članstva, stvori sličnu, kopiraj ovlasti,
izvezi DDL, obriši.

Korisnici s prijavom prikazani su s 👤, grupne role s 👥.

#### Izvoz role {#role-export}

**Export DDL** (jedna rola) i **Export ALL roles** pišu skriptu koja rolu
stvara negdje drugdje: `CREATE ROLE` sa svim atributima (unutar bloka koji
preskače rolu koja već postoji), njezin komentar, članstva, postavke
`ALTER ROLE ... SET` - i one po bazi - te, označeno po zadanom, **svaku
privilegiju koju ima u klasteru**: na bazama, tablespaceovima i postavkama
poslužitelja (PostgreSQL 15+), a u svakoj bazi na shemama, tablicama i
njihovim stupcima, sekvencama, funkcijama i procedurama, tipovima i
domenama, jezicima, foreign-data wrapperima i poslužiteljima te velikim
objektima, plus default privilegije koje daje ili dobiva (`ALTER DEFAULT
PRIVILEGES`). Svaka se baza čita zasebno i njezin dio počinje s
`\connect`, pa je datoteka psql skripta: `psql -f role.sql` na odredištu,
kao superuser. Baza koju izvoz nije mogao pročitati - ona na koju se rola s
kojom si spojen ne smije spojiti - navedena je na kraju datoteke i u
obavijesti, pa dio koji nedostaje nikad ne prođe tiho. **Ono što posjeduje**
dodaje `ALTER ... OWNER TO` za njezine sheme, tablice, poglede, sekvence i
rutine, za kad su ti objekti već na odredištu. Hashevi lozinki su zasebna
opcija. Izmjereno: rola s privilegijama svih vrsta u dvije baze, izvezena,
uklonjena s `DROP OWNED` i `DROP ROLE`, a datoteka pokrenuta psql-om - svaka
privilegija i svako vlasništvo vratilo se kakvo je bilo.

### 3.3 Pregled sheme (vidljivo samo kad ste spojeni)

Pregled sheme dio je stabla **Serveri**. Ispod veze na kojoj ste, **Baze podataka** popisuju sve baze na poslužitelju, s brojem. Proširite bazu i otvorit će se njezine sheme, svaka s brojem objekata koje sadrži, a svaka shema njezini objekti, grupirani po vrsti.

Svaka shema prikazuje istih 40 vrsta: tablice, particionirane tablice, poglede, materijalizirane poglede, strane tablice, sekvence, indekse, funkcije, procedure, agregate, prozorske funkcije, okidače, okidače događaja, pravila, RLS politike, tipove, domene, tipove raspona, tipove višestrukih raspona, proširenja, jezike, operatore, klase operatora, obitelji operatora, pretvorbe tipova (casts), kolacije, konverzije, konfiguracije, rječnike, parsere i predloške za pretraživanje teksta, omotače stranih podataka, strane poslužitelje, mapiranja korisnika, publikacije, pretplate, proširenu statistiku, zadane ovlasti, velike objekte i tablespaceove. Vrsta bez ijednog objekta ostaje na popisu, zasivljena, pa "nema ih" nikad ne izgleda kao "nije prikazano". Objekti koji ne pripadaju nijednoj shemi - okidači događaja, jezici, pretvorbe tipova, omotači stranih podataka, strani poslužitelji, mapiranja korisnika, publikacije, pretplate, zadane ovlasti bez IN SCHEMA, veliki objekti i tablespaceovi - nalaze se pod **(cluster)**. Sistemske sheme `pg_catalog` i `information_schema` prikazuju svoje tablice, poglede, indekse, rutine i tipove; operatori, pretvorbe tipova i slični objekti ugrađeni u PostgreSQL izostavljeni su iz stabla.

Relacije se otvaraju onako kako ih PostgreSQL drži. Particionirana tablica otvara se u svoje particije; particija koja je i sama particionirana stoji na svom mjestu sa svojim particijama ispod, na bilo kojoj dubini. Tablica se otvara u tablice koje od nje nasljeđuju. Svaka od njih navedena je jednom, ispod roditelja, a ne još jednom među tablicama. Particija koja živi u drugoj shemi od roditelja prikazana je ispod roditelja sa svojom shemom, i u svojoj shemi na vrhu, s imenom roditelja. Svaka tablica, pogled i materijalizirani pogled otvara se i u svoje dijelove - indekse, ograničenja, okidače, pravila, RLS politike, sekvence svojih stupaca, objekte statistike i publikacije u kojima je - koji se čitaju kad je otvorite, pa shema s tisućama tablica zbog njih ne učitava sporije. Ono što je PostgreSQL kopirao s particionirane tablice na particiju (indeks, ključ, okidač retka) tamo kaže *iz* kojeg originala dolazi i nije na popisima indeksa i okidača sheme, koji prikazuju ono što je deklarirano; takva se kopija ne može obrisati sama za sebe. Sekvenca koja pripada stupcu to i kaže (*pripada* ili *identity*).

Objekti koji pripadaju proširenju izostavljeni su iz svake vrste - samo proširenje ostaje pod Proširenjima - a shema kaže koliko ih je izostavila. Klik na taj redak prikazuje ih, svaki označen svojim proširenjem; ponovni klik ih skriva.

Klik na shemu daje popis njezinih tablica sa živim i mrtvim recima, veličinama, brojem indeksa, skeniranjima te zadnjim vacuumom i analyzeom; polje za filtriranje iznad popisa sužava ga po imenu. Klik na objekt otvara ga u vlastitoj kartici (za tablicu DDL, Relacije, Pohrana, Savjetnik, Podaci). Desni klik daje sve ostalo što se s njim može - vidi [Izbornici objekata](#object-menus). Polje za pretraživanje iznad stabla traži veze po aliasu, hostu, korisniku ili bazi; ne pretražuje unutar stabla.

## 4. Radne kartice

Svaka kartica nosi vlastitu vezu, položaj u shemi i stanje SQL editora. Kartice se čuvaju i nakon ponovnog pokretanja aplikacije.

- `+ Nova veza` (traka kartica gore lijevo): otvara novu karticu bez veze.
- Desni klik na karticu: Prebaci / Osvježi vezu / Prikvači / Spremi kao vezu (za nespremljene kartice) / Odspoji / Zatvori.
- Prikvačene kartice preživljavaju "Zatvori sve".

#### Stvaranje veze

Dijalog Nova veza sadrži podatke o vezi, a ispod njih ime, grupu i oznaku
bojom. Padajući izbornik grupa nudi postojeće grupe te **Stvori novu grupu** i
**Stvori podgrupu** - vezu se može odmah smjestiti gdje pripada, umjesto da
padne u `default` pa da je poslije treba premještati.

Dijalog završavaju tri gumba i sva tri s vezom rade isto:

| | |
|---|---|
| **Test** | provjeri podatke i ostani gdje jesi |
| **Spoji se** | otvori vezu u radnoj kartici bez spremanja |
| **Spremi i spoji** | zapiši spremljenu vezu, pa je otvori |

„Otvori" u oba slučaja znači isto: vlastita radna kartica osim ako je trenutna
još prazna, označena tom spremljenom vezom pa je traka kartica iznad prikazuje,
u bočnoj traci je označena kao aktivna, a svaka kartica alata otvorena poslije
pripada toj vezi. „Spremi i spoji" je prije samo zapisao red i stao, ostavljajući
karticu koja je bila u prvom planu da prikazuje vezu za koju nije znala da je
ima.

Kad je otvoreno više kartica nego što stane, na krajevima trake pojavljuju se
strelice, a ispod kartica **tanki klizač**. Klizač se pojavljuje samo kad ima
kamo skrolati; povucite ga ili kliknite prazan dio trake za listanje. Ima
vlastitu traku pa nikad ne prekriva karticu ni njen gumb za zatvaranje.

#### Rola, servis i SSH tunel {#conn-role-service-ssh}

Tri postavke u dijalogu veze koje većina veza ostavlja praznima, nazvane kao u
pgAdminu.

- **Rola** - sesija izvodi `SET ROLE` na nju odmah nakon svakog spajanja, na
  svakoj vezi koju aplikacija otvori za ovaj server: onima iz poola, ponovnom
  spajanju, provjerama, sakupljačima u pozadini, zadržanoj SQL transakciji.
  Sve tada radi kao ta rola, njezine ovlasti određuju što aplikacija nudi, a
  zaglavlje je imenuje. Korisnik za prijavu mora biti član role; ako server
  odbije, spajanje ne uspijeva s imenom role i razlogom servera, i ništa ne radi
  kao korisnik za prijavu umjesto toga. Rola odabrana na popisu rola u
  zaglavlju i dalje preuzima dok je odabrana.
- **Servis** - naziv libpq servisa. Poslužitelj, port, baza, korisnik i lozinka
  ostavljeni prazni u dijalogu dolaze iz tog servisa, a ono što ipak upišete ima
  prednost. libpq traži servis u datoteci zadanoj u `PGSERVICEFILE`, inače u
  `.pg_service.conf` u vašem home direktoriju (na Windowsima
  `%APPDATA%\postgresql\.pg_service.conf`), zatim u `pg_service.conf` u
  direktoriju zadanom u `PGSYSCONFDIR`.
- **Napredno: SSH tunel** (zatvoreno dok ga veza ne koristi) - za bazu dostupnu
  samo preko posredničkog (jump) poslužitelja. Upišite SSH poslužitelj, port (22
  ako je prazno), korisnika i po potrebi datoteku ključa. Aplikacija pokreće
  `ssh` program vašeg računala s lokalnim prosljeđivanjem porta, pa vrijede vaše
  ssh postavke, `known_hosts` i `ssh-agent`. Prijavljuje se samo ključem i nikad
  ne traži niti sprema SSH lozinku; ključ s lozinkom mora biti učitan u
  `ssh-agent`. Poslužitelj u dijalogu je tada poslužitelj baze kako ga vidi jump
  poslužitelj, često `localhost`. Jump poslužitelj čiji ključ još niste
  prihvatili odbija se uz naredbu koju jednom pokrenete u terminalu da ga
  provjerite i prihvatite. Tunel se zatvara sa zadnjom karticom koja ga koristi
  i kad aplikacija završi.

## 5. Glavni prikazi

Gornja traka kartica prebacuje prikaze. Svaki prikaz radi nad aktivnom vezom aktivne radne kartice.

U svakom padajućem popisu može se pretraživati. Kliknite ga ili samo počnite tipkati dok je u fokusu i nad njim se otvara polje za pretragu: popis se sužava sa svakom tipkom, pronalazi bilo koji dio stavke bez obzira na velika i mala slova i dijakritike ("sifra" pronalazi "šifra"), a svaka upisana riječ mora se pojaviti. Strelice gore i dolje pomiču odabir, Enter bira, Escape zatvara i ostavlja odabir kakav je bio, a Tab uzima označenu stavku i ide dalje. Popisi sa samo dvije ili tri stavke otvaraju popis sustava, u kojem upisano slovo skače na prvu stavku koja njime počinje.

### 5.1 Provjere

Zadani prikaz pri ulasku. Gornja traka izlaže katalog provjera složen po grupama:

- Zdravlje klastera: db-health, db-locks, db-blocking, db-long-tx, db-idle-in-tx, db-conn-limits, db-vacuum, db-autovacuum, db-checkpoints, db-replication, db-repl-slots, db-logical-replication
- Konekcije i upiti: db-connections, db-users, db-active-queries, db-long-queries, db-stat-statements, db-query-history, db-temp-queries, db-temp-files
- Pohrana i performanse: db-sizes, db-schema-sizes, db-table-sizes, db-index-sizes, db-bloat, db-bloat-indexes, db-bloat-breakdown, db-pgstattuple, db-buffer-cache, db-tablespaces
- Ugađanje i dijagnostika: db-settings, db-cache-hit, db-cache-advisor, db-index-usage, db-index-unused, db-index-advisor, db-duplicate-indexes, db-table-stats, db-replica-identity, db-workload-report, db-ash, db-awr-snapshot, db-trace-session, db-errors
- Sigurnost / ovlasti: db-roles-list, db-role-perms (odabir cilja), db-ownership (odabir opsega)
- Također dostupno: db-sequences, db-seq-usage, db-seq-sync (alati za sekvence u pregledniku sheme), db-setup-advisor (izbornik Pomoć)

**Tri izvještaja odgovaraju na „gdje je otišao prostor", na tri razine.**
`db-sizes` daje svaku bazu u klasteru; `db-schema-sizes` razlaže **spojenu**
bazu po shemama - ukupno, s odvojeno prikazanim podacima i indeksima, udio
svake sheme, broj objekata i njenu najveću tablicu; `db-table-sizes` ide do
pojedine tablice; `db-index-sizes` do pojedinog indeksa. Kako čitati sva
četiri, i što svaki ukupni iznos uključuje a što ne, opisano je pod *Gdje
odlazi prostor* niže.

Schema sizes pokriva samo bazu na koju si spojen, i to je ograničenje a ne
propust: `pg_namespace` je po bazi, pa bi popis shema svih baza tražio
konekciju na svaku. Ukupno obuhvaća sve što u shemi zauzima disk - tablice,
njihov TOAST, njihove indekse, materialized viewove i sekvence - svaki bajt
jednom.

Svaka provjera otvara vlastitu karticu s rezultatom. Kartice preživljavaju promjenu prikaza pa možete usporediti više provjera jednu uz drugu. Svaka kartica ima dva panela:

- Strukturirano: bogate tablice, kartice savjetnika, detaljni prikazi
- Sirovi ispis: obični tekstualni zapis

Mnoge provjere imaju odjeljke "savjetnika" s nalazima označenima po težini (OK / INFO / UPOZ / KRIT) i konkretnim prijedlozima sljedećeg koraka.

#### Zbrajanje onoga što izvještaj pokazuje

Svaka tablica izvještaja sortira se po stupcu i filtrira tipkanjem. Dvije stvari
odgovaraju na pitanje zbog kojeg se ti izvještaji obično i otvaraju - „isplati
li se".

**Zbroj ispod tablice.** Svaki stupac čije se vrijednosti čitaju kao veličine
ili kao obični brojevi zbraja se, a zbroj prati svaki filtar. Suzi izvještaj o
napuhanosti na jednu shemu i ukupno ti kaže koliko bi prostora vratio reindeks
te sheme; suzi dalje na jednu tablicu i vidiš isplati li se posao nad cijelom
shemom ili gotovo sve nosi jedna tablica.

Brojevi se čitaju natrag iz onoga što je ispisano, jer kod većine tih provjera
to je sve što postoji - PostgreSQLovo vlastito formatiranje veličina i ništa
ispod njega. Zbroj je stoga zbroj vrijednosti kakve su prikazane, zaokružen na
jedinicu u kojoj su ispisane: četrdeset indeksa zaokruženih na megabajt može
odstupati desetak megabajta u bilo koju stranu. Dovoljno točno za odluku, nije
brojka za citiranje.

Postoci, omjeri i prosjeci se nikad ne zbrajaju - njihov zbroj je broj koji ne
znači ništa, a ispis bi pozivao da ga se čita. To govori ime stupca: izvještaj
o napuhanosti ispisuje `bloat_pct` kao goli `44.9`, a znaka u ćeliji nema.

Mreža rezultata u SQL editoru i Data tab u pregledniku sheme zbrajaju samo
veličine. Izvještaj zna što su mu stupci; `SELECT` koji si upravo napisao ne
zna, a zbroj ispod stupca s ID-evima je šum.

**Biranje vrijednosti u stupcu.** Kvačica pokraj filtra svakog stupca otvara
popis vrijednosti koje u tom stupcu doista postoje, sa brojem pojavljivanja uz
svaku. Označiti tri sheme čija imena nemaju ništa zajedničko nije nešto što
filtar po podnizu može, a izvještaji o prostoru su toga puni - jedna shema od
dvadeset, četiri tablice od devetsto. Upisani filtar i označeni popis vrijede
oboje, pa je „tablice u ove dvije sheme kojima ime sadrži _log" jedan potez za
svako.

Stupac s više od dvjesto različitih vrijednosti to kaže umjesto da iscrta tisuću
kvačica.

#### O kojoj je bazi izvještaj

Svaki od ovih izvještaja odnosi se na jednu bazu, a redak iznad rezultata kaže
na koju - ime provjere, baza i veza na kojoj je pokrenuta. Dolazi iz taba, a ne
iz žive veze, pa tab koji je pokrenut nad `shop` i dalje piše `shop` nakon što
se veza ispod njega promijeni.

#### Jedan indeks, i što s njim

Izvještaji imenuju napuhan ili neiskorišten indeks. Klik na ime otvara
**savjetnik za indekse** za taj jedan indeks, a isto stoji na svakom indeksu u
popisu indeksa u pregledniku sheme.

On odgovara na pitanje koje izvještaj ostavlja otvorenim. Procjena napuhanosti
je isti izračun kao u izvještaju iz kojeg si došao, sužen na taj jedan indeks
pa se to dvoje ne može razilaziti. Uz to: koliko je puta skeniran otkad su
statistike zadnji put resetirane, njegova veličina naspram tablice, i provjera
protiv svih ostalih indeksa na istoj tablici - drugi indeks nad točno istim
stupcima istom metodom, ili onaj čiji su vodeći stupci cijeli popis ovoga, u
kojem slučaju planer može koristiti širi za sve na što ovaj odgovara.

Indeksi iza ograničenja čitaju se drukčije, i zato „nikad skeniran" sam po sebi
nije razlog za brisanje. Indeks iza PRIMARY KEY-a, UNIQUE-a ili EXCLUDE-a
provodi ograničenje bez obzira čita li ga ijedan upit, a `DROP INDEX` ga odbija;
isto vrijedi za replica identity tablice, koji logička replikacija treba.
Savjetnik to kaže i ne nudi brisanje.

Ono što nudi za pokretanje pisano je za jedan indeks i ništa više:

| | |
|---|---|
| `REINDEX INDEX CONCURRENTLY` | gradi zamjenu pokraj njega i zamijeni ih - bez dugog lock-a, treba mjesta za obje kopije, ne može unutar transakcije (PostgreSQL 12+) |
| `REINDEX INDEX` | drži ACCESS EXCLUSIVE lock na tablici cijelo vrijeme pregradnje |
| `DROP INDEX CONCURRENTLY` | nudi se samo ondje gdje se indeks smije obrisati |
| definicija | sačuvaj je prije brisanja - to je jedina kopija |

Svako otvara isti prozor u kojem se otvara svaka generirana naredba u ovoj
aplikaciji: pogledaj je, kopiraj, spremi ili otvori u SQL editoru. Dvije
pregradnje nude i **Pokreni kao pozadinski posao**, što ih stavlja u Pozadinske
poslove s ispisom uživo, pa pregradnja od dvadeset minuta nije vezana za
prozor koji mora ostati otvoren.

Prije ovoga jedini REINDEX koji je aplikacija mogla izdati bio je po tablici,
shemi ili bazi - pa se izvještaj koji imenuje jedan napuhan indeks mogao
riješiti samo pregradnjom svih indeksa na njegovoj tablici.

#### Provjere vrijedne spomena {#notable-checks}

**db-health** (preporučena prva postaja): pregled stanja na jednom ekranu - veze, idle-in-tx, duge aktivne transakcije, blokiranja, pogodak predmemorije, kašnjenje replikacije, arhiviranje WAL-a, napuhanost, vacuum na čekanju, sekvence, nekorišteni indeksi. Detaljni prikaz po kategoriji izvlači problematične retke.

**db-cache-advisor**: glavni omjeri pogodaka za heap/indekse/toast, vruće tablice/indekse/TOAST s konkretnim rješenjem, provjera veličine shared_buffers naspram klastera, razumnost effective_cache_size, snimka pg_buffercache kad je ekstenzija instalirana.

**db-index-advisor**: analiza seq scanova s popisom kandidatskih stupaca po tablici. Uspoređuje s pg_stat_statements za vruće upite, izvlači praznine u vodećem stupcu, prilike za parcijalne indekse, pokrivenost složenih FK-ova. Generira CREATE INDEX naredbe koje možete kopirati.

**db-vacuum**: stanje vacuum/analyze po tablici sa savjetnikom za autovacuum. Prikazuje žive i mrtve retke, mod_pct, oznaku vrućine (vrlo vruća / vruća / srednja / zadana), oznake (rizik od WRAPAROUNDA, isključen autovacuum, samo umetanja) te "preporučeni" skup reloptiona naspram "trenutnog" za svakog kandidata. Od 1.0.x iz listopada 2026. gleda i u drugom smjeru: koliko se često svaka tablica vacuumira (pokretanja na dan i razmak među njima, s trajanjem na PostgreSQL-u 18), tablicu koja se vacuumira jednom na sat ili češće - i je li to u redu (vidi [kad je često vakuumiranje u redu](#vacuum-bloat-and-table-statistics)), tablicu koja se vacuumira opet i opet bez učinka jer nešto drži snapshot stariji od njezinih mrtvih redaka (imenovano, s onim što treba završiti), tablicu čiji je svaki vacuum agresivan te - u odjeljku *Autovacuum cost and I/O* - postavke tempa autovacuuma i, na PostgreSQL-u 16 i novijem, njegov udio u čitanjima i pisanjima klastera iz `pg_stat_io`. Preporuka nikad ne pooštrava pragove tablici koja se već vacuumira jednom na sat. Kaže i kad vacuum do tablice uopće ne može doći: sesija drži lock koji VACUUM ne može dijeliti (SHARE UPDATE EXCLUSIVE ili jači - dugi ANALYZE, CREATE INDEX CONCURRENTLY, ALTER TABLE, LOCK TABLE) u transakciji otvorenoj minutu ili dulje, na tablici koja je prešla okidač. Obični autovacuum takvu tablicu preskoči i pokuša opet nakon naptimea, pa se ništa drugo što piše za nju ne događa; nalaz imenuje sesiju (pid, korisnik, aplikacija, klijent, koliko je dugo otvorena njezina transakcija, njezin upit) i stoji prvi. I sesija koja je pisala pa ostala idle u READ COMMITTED transakciji broji se kao držač snapshota: između naredbi nema snapshot, ali njezin ID transakcije drži horizont.

**db-stat-statements**: prvih 25 upita s oznakama po retku (nestabilnost plana, ispis u privremene datoteke uz konkretne bajtove po pozivu, nizak pogodak predmemorije, sporo pri svakom pozivu, put lošeg slučaja). Widget s najskupljim upitima omogućuje kopiranje cijelog upita.

**db-replica-identity**: tablice bez PRIMARY KEY-a i tablice s nezadanim REPLICA IDENTITY. Uspoređuje s pg_publication_tables da razlikuje KRITIČNO (u publikaciji je, past će na UPDATE/DELETE) od savjetodavnog. Preporučuje ALTER po prioritetu: ADD PRIMARY KEY, REPLICA IDENTITY USING INDEX, REPLICA IDENTITY FULL.

**db-errors**: nadzor grešaka svjestan oblaka. Uvijek uzorkuje brojače iz pg_stat_database u prozoru od 5 sekundi (rollbackovi, deadlockovi, konflikti, sessions_fatal/killed/abandoned na PG14+). Na vlastitim klasterima uz to čita zadnjih 64 KB server loga preko `pg_current_logfile()` + `pg_read_file()` i parsira ERROR/FATAL/PANIC retke. Na upravljanim oblacima čitanje loga se preskače uz upute specifične za pružatelja (RDS konzola, log_fdw, Cloud Logging, Azure Server Logs, Supabase / Neon / Heroku stranice s logovima).

**db-roles-list**: popis rola i članstava plus sigurnosni pregled: superuser + prijava, BYPASSRLS bez superusera, CREATEROLE na PG <16, prijava bez ograničenja veza i bez VALID UNTIL, istekle role s prijavom, MD5 hashevi lozinki, REPLICATION + LOGIN, PUBLIC ima CONNECT na bazama, PUBLIC ima CREATE na shemama, SECURITY DEFINER rutine u vlasništvu superusera s pravom izvođenja za PUBLIC.

**db-role-perms** (odabir ciljne role): 19 odjeljaka (pregled, članstva, ovlasti na bazi/shemi/tablici/stupcu/sekvenci/funkciji/tipu/fdw/poslužitelju/jeziku, zadane ovlasti, RLS) sa stupcem tumačenja po retku ("preko PUBLIC", "WITH GRANT OPTION", "nema USAGE na shemi X - dodjela je bez učinka" i slično) te strukturom nalaza kroz sve odjeljke (stvarni superuser preko lanca, nasljeđivanje BYPASSRLS / CREATEROLE / REPLICATION, dodjele na tablici bez USAGE na shemi i slično).

**db-ownership** (odabir opsega): popisi za sheme / tablice / sekvence / funkcije / tipove / FDW-ove / jezike / publikacije / pretplate te nalazi (objekti aplikacije u vlasništvu superusera, SECURITY DEFINER rutine u vlasništvu superusera, viseći OID-ovi vlasnika nakon DROP ROLE bez REASSIGN, shema public s vlasnikom koji nije superuser, sheme s tri ili više različitih vlasnika objekata).

**db-settings**: vodič kroz ~70 parametara iz pg_settings koji su bitni za podešavanje, s DBA obrazloženjem, praktičnim pravilom i oznakama stanja (upozorenje / opasnost). Uređuje se kroz prikaz Poslužitelj.

**db-setup-advisor**: ispituje svaku postavku na strani poslužitelja o kojoj DBApguin funkcije ovise (ekstenzije, GUC-ovi, dodjele rolama), po funkciji javlja OK / DJELOMIČNO / NEDOSTAJE i ispisuje točan SQL ili redak parameter grupe kojim se svaka praznina popunjava. Svjestan je pružatelja: na upravljanim oblacima savjet kaže "postavite u parameter grupi" umjesto da predlaže ALTER SYSTEM. Pokrenite ovo prvo kad postavljate DBApguin na novi klaster, i ponovno nakon svake promjene postgresql.conf radi provjere.

Popis iznad kaže što svaka provjera proizvodi. Ispod je kako čitati one čiji
izlaz treba tumačenja, i pogreške zbog kojih postoje.

#### Pregled zdravlja {#the-health-overview}

`db-health` je jedan ekran: redak po kategoriji - veze, idle-in-transaction, duge transakcije, blokiranja, pogodak keša, kašnjenje replikacije, arhiviranje WAL-a, kašnjenje vacuuma, potrošeni prostor, zaostatak vacuuma, sekvence pri kraju, nekorišteni indeksi, trajnost, particije - svaki zelen, žut ili crven. Pokrenite ga prvog; često je jedina provjera koja vam treba. Nalaz koji se tiče tablice imenuje je u stupcu **table** - najgoru kad ih broji više - i to je ime link: otvara tablicu u pregledniku sheme.

Žuto znači "vrijedi pogledati", crveno znači "produkcija je pogođena sada". Kliknite kategoriju da otvorite njene retke: blokiranje se raširi u parove blokira/čeka, idle-in-transaction u sesije i koliko dugo svaka drži svoj snapshot. Crveni redak bez ičega u dubinskom prikazu okinut je zbroja, a ne pojedinog retka - npr. pogodak keša srušen kroz cijelu bazu. Zamka je čitati žuto kao šum: žuti potrošeni prostor je VACUUM koji možete zakazati za večeras; ista tablica crvena za mjesec dana je `pg_repack` na opterećenom sustavu.

**Kašnjenje vacuuma i potrošeni prostor su dva odvojena retka jer mjere dvije različite stvari.** *VACUUM LAG (dead tuple %)* je `n_dead_tup / (n_live_tup + n_dead_tup)` - udio redaka tablice koji su mrtvi. Govori da autovacuum zaostaje. To **nije** mjera prostora: tablica može biti 4% mrtva i i dalje sjediti na terabajtu slobodnog prostora koji nikad nije vraćen. *WASTED SPACE (est.)* je taj terabajt - procijenjeni bajtovi koje tablica drži ali ne koristi, heap plus TOAST plus svaki indeks, ista kombinirana procjena koju koriste `db-bloat-breakdown` i Advisor tab. Tablica se pojavi kao `4.3% dead tuples but 1.3 TB wasted (39x its live data)` kad je oboje istovremeno istina. Redak se pali na bilo koju od dvije mjere: 30 % mrtvih redaka (CRIT na 50 %) ili, bez obzira na udio, 1.000.000 mrtvih redaka u jednoj tablici (CRIT na 10.000.000) - 29 % tablice od milijardu redaka i dalje je vacuum koji jako kasni.

**WAL ARCHIVING** kaže šalje li arhiver WAL upravo sada, iz `pg_stat_archiver` i postavki arhiviranja - isto čitanje koje rade alarm u zaglavlju i kartice arhiviranja u Key settings. *Pada* (crveno) znači da zadnji pokušaj nije uspio i otad ništa nije arhivirano: redak imenuje segment koji nije uspio i kada, zadnji arhivirani, i koliko segmenata čeka u `pg_wal`. `archive_mode` uključen bez ijednog od `archive_command` i `archive_library` je žuto - ništa se ne arhivira i svaki završeni segment ostaje u `pg_wal` - a crveno kad čeka 10 GB. Postavljeni i `archive_command` i `archive_library` je crveno: od PostgreSQL-a 16 arhiver uopće odbija raditi, dok mu statistika izgleda nepromijenjeno. Detaljni prikaz navodi naredbu onako kako je server ima i što napraviti: pročitati njezinu grešku u logu servera (`archive command failed with exit code`), pokrenuti je ručno za navedeni segment, i nikad ne brisati datoteke iz `pg_wal` - kad naredba proradi, arhiver sam sustigne. Brojanje segmenata koji čekaju treba PostgreSQL 12 i superusera ili `pg_monitor`; bez toga broja nema.

**TRAJNOST** čita postavke o kojima ovisi preživljava li potvrđena transakcija pad. Crvena je za `fsync = off` ili `full_page_writes = off` - dvije postavke uz koje pad operacijskog sustava ili nestanak struje mogu oštetiti sam klaster, a ne samo izgubiti njegove zadnje transakcije - i za `synchronous_standby_names` koji traži više sinkronih standbyja nego što ih je spojeno dok commitovi čekaju na njih (`synchronous_commit` na `on`, `remote_write` ili `remote_apply`): tada svaki COMMIT visi, a redak kaže koliko sesija upravo čeka (wait event `SyncRep`) i koliko dugo. Kad je `synchronous_commit` na razini servera `local` ili `off`, isti nedostajući standby je žut: commitovi sada ne čekaju, ali sesija koja ga vrati na `on` čeka. `synchronous_commit = off` i isključeni kontrolni zbrojevi podataka su plavi (INFO): prvo može izgubiti zadnje transakcije već prijavljene kao potvrđene (do tri `wal_writer_delay`), ali ništa ne oštećuje, zamjena koja se često radi namjerno; drugo znači da se stranica koju je disk oštetio čita bez greške - uključivanje kontrolnih zbrojeva traži zaustavljen klaster (`pg_checksums --enable`, PostgreSQL 12+). Kartice Ključnih postavki za istih pet postavki daju isti zaključak. Na standbyju se dio o sinkronom standbyju preskače: vrijedi samo za primarni.

**PARTICIJE** je zaliha tablica particioniranih po rasponu u spojenoj bazi: žuto kad zadnji raspon završava unutar 7 dana ili DEFAULT particija RANGE tablice drži retke, crveno kad je zadnji raspon završio - vidi [Zaliha particija](#partition-runway).

Red **MIGRACIJE** pojavljuje se kad baza ima tablicu povijesti alata za migracije: crven za neuspjelu migraciju, žut ili crven za lock koji je ostavio njezin proces (vidi *Povijest migracija*).

#### Veze, sesije i opterećenje {#connections-sessions-and-load}

Ove čitaju `pg_stat_activity` i `pg_stat_statements` iz različitih kutova.

`db-connections` je broj po stanju - active, idle, idle in transaction i waiting. **Idle in transaction** je broj koji treba pratiti: te sesije drže transakciju otvorenom, prikvačuju najstariji snapshot koji vacuum smije ukloniti i često drže lockove, a rastući broj je obično aplikacija koja je zaboravila commit-ati ili pool bez idle-in-transaction timeouta. `db-users` razlaže iste veze po roli i aplikaciji, a tako se nađe *koja* usluga ih curi.

`db-conn-limits` je `max_connections` naspram iskorištenog, s razlaganjem po stanju i rezervom `superuser_reserved_connections`. Samo klijentski backendi se broje protiv limita - pozadinski radnici ne - pa broj ovdje može biti niži od sirovog broja iz `pg_stat_activity`. Alarm za veze u zaglavlju broji isto. **Waiting** je aktivna naredba koju zadržava nešto drugo osim njezina klijenta - lock, LWLock, I/O; i neaktivna sesija pokazuje wait event (`Client: ClientRead`, čeka sljedeću naredbu) i ne broji se. Kad je blizu stropa, rješenje gotovo nikad nije viši `max_connections` (svaki slot košta memorije); to je connection pooler, ili nalaženje gornjih idle-in-transaction sesija.

`db-active-queries` je što se izvršava upravo sad, najduže prvo, s PID-om backenda i wait eventom. `db-long-queries` je isti popis filtriran na naredbe koje traju preko pet sekundi - one koje vrijedi prekinuti ili objasniti. Oba su snapshot: upit koji je završio između osvježenja je nestao.

`db-stat-statements` je kumulativni prikaz - top naredbe po ukupnom vremenu od zadnjeg `pg_stat_statements_reset()`, svaka normalizirana tako da su `WHERE id = 1` i `WHERE id = 2` jedan redak. Savjetodavne zastavice po retku ističu nestabilnost plana, izljev `work_mem` (s bajtovima po pozivu), nizak omjer pogodaka keša i naredbe spore na svakom pojedinom pozivu, ne u prosjeku. Ukupno vrijeme je stupac po kojem treba sortirati: upit koji traje 3 ms milijun puta dnevno veći je trošak od onog koji traje 20 sekundi dvaput. Widget kopira puni tekst naredbe. Vidi `db-query-history` (odjeljak 6.6) za iste podatke unakrsno povezane sa satom.

Nakon top naredbi dolazi **Optimističko zaključavanje - zastarjela pisanja**: svaki UPDATE i DELETE u prikazu čiji WHERE uspoređuje stupac verzije s parametrom - Hibernateov `@Version`, Railsov `lock_version`, `xmin` ili `ConcurrencyStamp` Entity Frameworka, ručno pisan `version = $n`, provjera `updated_at` - sa `stale_%` = 1 - rows / calls: udio pisanja koja su našla redak promijenjen (ili obrisan) i ništa nisu zapisala. Aplikacija svako od njih ponovno učitava i pokušava, petlja koju strana baze inače nikad ne vidi. Naredba je žuta kad barem 5% od barem 100 poziva nije ništa zapisalo, ili kad to učini 1.000 poziva bez obzira na udio; ista naredba među top 25 nosi istu oznaku. Naredba koja piše više redaka nego što je pozvana (paket s uvjetom verzije) navodi se bez udjela, jer rows / calls tada nije jedan redak po pozivu.

`db-temp-queries` i `db-temp-files` su o izljevima na disk. `db-temp-files` je uživo - datoteke koje se pišu upravo sad, grupirane po PID-u, pa uhvatite upit koji se izlijeva dok se to događa. `db-temp-queries` je kumulativno iz `pg_stat_statements` - kronični izljevnici, što je `work_mem` premalen za sortiranje ili hash tog upita ili redoslijed spajanja koji je planer pogriješio. Malo temp-a je normalno na izvještajnom upitu; ista naredba koja se izlijeva gigabajtima na svakom pokretanju je ona koju treba popraviti.

#### Lockovi, blokiranje i zaglavljene transakcije {#locks-blocking-and-stuck-transactions}

`db-blocking` je prva za otvoriti u incidentu „aplikacija je smrznuta": uparuje svaku blokiranu sesiju s točnom sesijom koja je blokira (preko `pg_blocking_pids()`), pa je vrh lanca - transakcija na koju svi ostali čekaju - jedan redak, s njenim upitom i koliko dugo drži lock. Prekinite ili terminirajte tu jednu i red se isprazni.

`db-locks` je sirovi pogled `pg_locks` - svaki držani i čekani lock, ne samo oni koji blokiraju. Koristite ga kad je `db-blocking` prazan a nešto se i dalje čini zaglavljeno, ili da vidite *koji* je način locka u pitanju (`ShareLock` koji čeka na `RowExclusiveLock` je insert stranog ključa protiv retka koji se drugdje ažurira; zahtjev za `AccessExclusiveLock` iza dugog `SELECT`-a je DDL naredba ili `VACUUM FULL` koji će smrznuti tablicu dok čitanje ne završi).

`db-long-tx` popisuje transakcije otvorene duže od minute, što god radile. Duga transakcija koja *radi* je spor upit; ona koja je **idle in transaction** je greška aplikacije, i gora je od sporog upita jer drži svoj snapshot i svoje lockove dok ne radi ništa. `db-idle-in-tx` je taj drugi slučaj sam, s time koliko je dugo svaka sesija parkirana usred transakcije. U oba slučaja trošak je isti: `VACUUM` ne može počistiti nijednu verziju retka noviju od najstarije otvorene transakcije, na razini cijelog klastera, pa jedan zaboravljeni `BEGIN` na laptopu može napuhati svaku tablicu na poslužitelju.

Prikaz **Sessions** vodi transakcijski kut dalje od checkova. Panel s detaljima sesije pokazuje *Dob snimke (backend_xmin)* - koliko transakcija unatrag je prikvačena njena snimka - i, kad se odnosi, crveni redak koji kaže da **ova sesija trenutno definira xmin horizont klastera**: autovacuum je blokiran svugdje dok njena transakcija ne završi. Popis Live sessions označava tu istu sesiju, a prikaz Blocking hoda graf čekanja na lockove u stablo pa je jedna transakcija koja drži sve ostale jedan redak.

#### Pripremljene (two-phase) transakcije {#prepared-two-phase-transactions}

`db-prepared-xacts` pokriva jednu vrstu transakcije koja uopće nema sesiju. `PREPARE TRANSACTION 'gid'` zapiše transakciju na disk i vrati se; kasniji `COMMIT PREPARED 'gid'` ili `ROLLBACK PREPARED 'gid'` je dovršava. Između toga drži svaki lock koji je imala i prikvačuje xmin horizont klastera - a ništa u `pg_stat_activity` je ne pokazuje jer nije konekcija. Vanjski transakcijski upravitelj (JTA Java app servera, Windows MSDTC, XA driver) koji pripremi transakciju pa se sruši ili je izgubi ostavlja je pripremljenu zauvijek. To je klasičan incident „baza se stalno napuhava a svaka sesija izgleda uredno".

Provjera popisuje svaku - `gid`, vlasnika, bazu, koliko davno je pripremljena, dob njene transakcije i tablice koje zaključava - i označava sve starije od deset minuta kao vjerojatnu napuštenu. Riješite je spojeni na **njezinu bazu**: `COMMIT PREPARED 'gid'` ako posao treba proći, `ROLLBACK PREPARED 'gid'` ako ne treba.

**Većina poslužitelja uopće ne može imati pripremljene transakcije.** `max_prepared_transactions` je po zadanom `0`, i provjera to jasno kaže umjesto da pokaže praznu tablicu - podignite iznad nule samo ako nešto stvarno treba `PREPARE TRANSACTION`. Kad je uključeno, pripremljena transakcija koja sada blokira živu sesiju pojavljuje se u prikazu Blocking kao vlastiti čvor (označen `prepared 'gid'`, s dvije naredbe za rješavanje za kopiranje), a u `db-locks` njeni držani lockovi pripišu se gid-u umjesto da nestanu.

#### Povijest migracija {#migration-history}

`db-migrations` (izbornik Zdravlje klastera) čita tablice povijesti koje alati za migracije drže u spojenoj bazi - Flyway (`flyway_schema_history`, prije Flywaya 5 `schema_version`), Liquibase (`databasechangelog`, `databasechangeloglock`), Alembic (`alembic_version`), golang-migrate i Rails (`schema_migrations`, razlikuje ih stupac `dirty` golang-migratea), Django (`django_migrations`), Prisma (`_prisma_migrations`), knex (`knex_migrations`, `knex_migrations_lock`) i Entity Framework Core (`__EFMigrationsHistory`) - u bilo kojoj shemi. Za svaki kaže što je zadnje primijenjeno i označi stanja koja zaustavljaju sljedeći deploy, s redom koji to pokazuje i kako ga očistiti:

- **CRIT - neuspjela migracija:** Flyway `success = false` (migracija koja se nije mogla izvršiti u transakciji, npr. `CREATE INDEX CONCURRENTLY`, pala je na pola; Flyway odbija migrirati do `flyway repair`), golang-migrate `dirty = true` (ručno popravi shemu, zatim `migrate force`), Prisma migracija koja je počela a nije ni završila ni bila vraćena, prije deset ili više minuta (`prisma migrate resolve`).
- **Lock koji je ostavio proces koji je pao:** Liquibase `databasechangeloglock.locked = true` je upozorenje kad se drži **5 minuta** i kritično na **60 minuta** (duga migracija podataka drži ga s razlogom, pa red imenuje tko ga je uzeo i kad; `liquibase releaseLocks` ga otpušta); knex `is_locked = 1` je upozorenje (ne bilježi vrijeme).
- **INFO:** Flyway migracije primijenjene izvan redoslijeda (`outOfOrder=true` - dopušteno, ali ova baza ih je izvršila redom kojim nova instalacija ne bi), više od jednog Alembic heada, Liquibase changesetovi zabilježeni kao `FAILED` uz `failOnError=false`.

Tablica koju ovaj login ne smije čitati navodi se kao pronađena i ništa se ne nagađa. Pregled zdravlja nosi najgore od ovoga kao red **MIGRACIJE**, i to samo kad baza ima takvu tablicu.

#### Tablice redova i LISTEN/NOTIFY {#queue-tables-and-listen-notify}

`db-queues` (izbornik Konekcije i upiti) nalazi tablice koje služe kao redovi poslova: one koje navode naredbe `FOR UPDATE SKIP LOCKED` u `pg_stat_statements` (samo ova baza) i tablice biblioteka koje drže poslove u PostgreSQL-u - pgmq, PgQ, que, graphile-worker i River. Za svaku: poslovi koji čekaju (samo oni kojima je došlo vrijeme, ne zakazani za kasnije), koliko je najstariji čekao, neuspjeli poslovi gdje ih biblioteka bilježi, živi i mrtvi redovi, veličina i zadnji vacuum. Tablica pronađena samo preko svojih naredbi nema poznato stanje "čeka" - mnoge čuvaju gotove poslove - pa se njezin najstariji red pokazuje, ali ne ocjenjuje.

- **Mrtve redove** preskače svako preuzimanje posla, zato tablica reda prva strada kad vacuum zaostane ili nešto prikvači xmin horizont. Upozorenje na **50%** tablice uz barem **10.000** mrtvih redova, ili na **1.000.000** samih; kritično na **90%** uz **100.000**, ili na **10.000.000**. Tekst navodi i udio i broj.
- **Čekanje:** upozorenje kad je najstariji dospjeli posao čekao **5 minuta**, kritično na **1 sat**. PgQ potrošači ocjenjuju se jednako po svom zaostatku.
- **LISTEN/NOTIFY:** `pg_notification_queue_usage()` (PostgreSQL 9.5+) kaže koliko je pun jedini red obavijesti na klasteru. Skraćuje se samo do onoga što je najsporiji slušatelj pročitao, a sesija čita samo između transakcija, pa slušatelj ostavljen idle u transakciji pušta red da raste dok svaki `NOTIFY` ne padne pri commitu. Upozorenje na **10%** maksimuma ili **512 MB**, kritično na **50%** (gdje PostgreSQL počne upozoravati u logu) ili **4 GB**; maksimum je 8 GB, ili `max_notify_queue_pages` od PostgreSQL 17. Navode se sesije čija je zadnja naredba bila `LISTEN`, s njihovim stanjem - PostgreSQL nema pogled na slušatelje, pa je to najbliže što se može. Iste granice pale lampicu **LISTEN/NOTIFY red** u zaglavlju, prikazanu na klasteru na kojem netko sluša.

Brojanja su ograničena (milijun redova) i svaka provjera ima pet sekundi, pa se golemi zaostatak bez indeksa javlja kao neizmjeren umjesto da se čita cijeli.

#### Savjetnici za keš i indekse {#the-cache-and-index-advisors}

`db-cache-hit` je jedan broj: udio čitanja blokova posluženih iz `shared_buffers` od zadnjeg reseta statistike, za ovu bazu. Kumulativni je prosjek kroz cijeli život statistike, pa jednokratni hladni izvještaj od prije sat vremena i dalje ga povlači dolje sada - sam po sebi vam ne govori gotovo ništa. `db-cache-advisor` je verzija po kojoj vrijedi djelovati.

`db-cache-advisor` javlja omjere pogodaka za heap, indekse i TOAST, pa tablice i indekse koji rade najviše fizičkih čitanja, `shared_buffers` naspram veličine podataka i provjeru zdravog razuma za `effective_cache_size`. Omjer ispod ~99% važan je samo ako su promašaji koncentrirani - jedna hladna izvještajna tablica čitana jednom dnevno ne vrijedi ugađanja. Djelujte po popisu vrućih objekata, ne po gornjem broju.

`db-index-advisor` nalazi tablice s visokim udjelom sekvencijalnih pretraga i popisuje kandidate stupaca iz predikata u `pg_stat_statements`, uključujući praznine vodećeg stupca u postojećim indeksima i strane ključeve bez pokrivnog indeksa. Za svaki napiše `CREATE INDEX`. Dvije zamke: mala tablica se i treba sekvencijalno pretraživati, pa ignorirajte kandidate ispod nekoliko tisuća redaka; i svaki indeks je trošak pri pisanju, pa dodajte onaj koji vrući upit treba, ne sve. Da procijenite određeni indeks prije gradnje, koristite probu indeksa (odjeljak 6.5.5), koja ga isplanira s HypoPG-om.

Kad su spremljene AWR snimke baze, `db-index-advisor` dodaje tri odjeljka iz povijesti nakon bloka **Dokazi** koji navodi korištene snimke i ASH dane. **Sekvencijalna skeniranja kroz spremljene prozore**: po tablici skeniranja i pročitani redci dnevno, redci po skeniranju, i prva polovica povijesti naspram druge; *raste* znači da je druga polovica barem dvostruko veća od prve i barem 100 dnevno, ili barem 10.000 više dnevno, a tablica je navedena kad raste ili kad njena sekvencijalna skeniranja čitaju barem 100.000 redaka dnevno. Uz svaku, naredbe koje je spominju i najviše čitaju s diska. **Naredbe koje najviše čitaju s diska**: `shared_blks_read` iz `pg_stat_statements` zbrojen kroz prozore, po pozivu, s najprometnijim prozorom i koliko je često ASH uhvatio naredbu kako čeka I/O (povezano po query id od PostgreSQL 14). **Bez skeniranja kroz spremljeni prozor**: skeniranja svakog indeksa zbrojena kroz prozore (resetiranje statistike između započinje novo brojanje, ništa se ne gubi), uz navedeni prozor. Kroz barem **7 dana** indeks bez skeniranja je *kandidat za pregled*, ispod toga prozor je *prekratak za procjenu*, a svaki redak nosi napomenu da mjesečni, kvartalni ili godišnji izvještaj može biti njegov jedini čitatelj. Tu se ne nudi `DROP`, a primarni ključevi, jedinstveni indeksi i indeksi stranih ključeva nikad nisu kandidati: navedeni su zasebno kao zadržani, s razlogom.

#### Održavanje indeksa {#index-housekeeping}

Tri provjere nalaze indekse koji koštaju pisanja i disk a ne zarađuju ništa natrag. Sve tri čitaju `pg_stat_user_indexes`, pa su njihovi brojevi pretraga **od zadnjeg reseta statistike** - `pg_stat_statements_reset()` ili `pg_stat_reset()` od prošlog tjedna znači da „nikad pretraženo" pokriva samo tjedan.

`db-index-usage` je puna slika: svaki indeks s brojem pretraga, veličinom i omjerom pretraga naspram pisanja u tablicu. `db-index-unused` je podskup koji je bitan - indeksi s nula pretraga - i već isključuje one za koje je nula normalna: one iza `PRIMARY KEY`, `UNIQUE` ili `EXCLUDE` ograničenja, i replica identity tablice. Oni nešto provode bez obzira čita li ih ijedan upit i `DROP INDEX` ih odbija, pa bi popis za brisanje koji bi ih uključivao bio opasan. Izostavlja i svaki indeks na stupcu **stranog ključa**: izgleda kao da nikad nije skeniran jer ga `DELETE` i `UPDATE` na roditeljskoj tablici koriste interno, a bez njega svaki od njih skenira podređenu tablicu pod zaključavanjem. Oni se navode zasebno, svaki sa stranim ključem koji podupire, i nikad se ne nude za brisanje - ni ovdje, ni u pregledu zdravlja, ni kod napuhanosti indeksa, ni u Advisoru tablice. Ono što ostaje **kandidat je za provjeru, a ne presuda**: provjera piše `DROP INDEX CONCURRENTLY` za svaki i kaže koliko dugo brojači pokrivaju, jer indeks koji koristi samo mjesečno, kvartalno ili godišnje izvješće izgleda nekorišteno cijelo vrijeme između - potvrdite kroz razdoblje koje ga uključuje.

`db-duplicate-indexes` nalazi indekse koji se međusobno ponavljaju. **Identični** - ista tablica, stupci, klase operatora, izrazi i predikat - prikazuju se kao grupa, a provjera imenuje onaj koji **ostaje** i zašto: onaj koji nešto provodi ili poslužuje (primarni ključ, `UNIQUE` ili `EXCLUDE` ograničenje, jedinstveni indeks, replica identity, indeks stranog ključa), inače najviše skeniran. Ostali su označeni kao **redundantni - identični** onome koji ostaje. Kad dva od njih oba nešto provode (dva `UNIQUE` ograničenja na istim stupcima), to kaže umjesto toga: uklanjanje jednog znači mijenjanje ograničenja. Navodi i obični indeks čiji su stupci **početni stupci šireg** s istim klasama operatora - širi odgovara na iste pretrage, malo veći za čitanje; jedinstveni indeksi i indeksi ograničenja tu se nikad ne navode, a kad uži podupire strani ključ, širi ga i dalje pokriva. Provjera ne piše `DROP` - redundantan indeks košta pisanja i disk, a odluka je tvoja. Prije uklanjanja sačuvaj definiciju - to je jedina kopija.

Da djelujete na jednom indeksu koji izvještaji imenuju - točan `REINDEX` ili `DROP`, njegova povijest pretraga i provjera naspram svakog drugog indeksa na istoj tablici - kliknite njegovo ime da otvorite savjetnik po indeksu (vidi *Jedan indeks, i što s njim* gore).

#### Gdje odlazi prostor: izvještaji o veličinama {#where-the-space-goes}

Četiri izvještaja odgovaraju na „što troši disk", svaki jednu razinu finiji od
prethodnog. Čista su mjera - bez savjetnika, bez preporuke - pa je jedini
način da ih se pogrešno pročita taj da se pogrešno shvati što neki broj
uključuje.

`db-sizes` - **svaka baza u klasteru.** `pg_database_size()` po bazi, plus broj
živih konekcija (samo klijentski backendi - autovacuum radnici i pozadinski
procesi također nose `datname` pa bi njihovo brojanje učinilo praznu bazu
zaposlenom), omjer pogodaka keša od zadnjeg reseta statistike i kumulativna
upotreba privremenih datoteka. Template baze su izostavljene. Ovo je jedini
izvještaj ovdje koji vidi preko baza; konekcija na kojoj ste ne utječe na
njegove brojeve.

`db-schema-sizes` - **spojena baza, razložena po shemi.** Ukupno po shemi,
bajtovi podataka tablica i indeksa prikazani odvojeno, postotak baze za svaku
shemu, broj objekata i najveća pojedina tablica u svakoj. Ukupno broji jednom
sve što u shemi zauzima disk: tablice, njihov TOAST, njihove indekse,
materialized viewove, sekvence. Ne može prijeći u druge baze jer je
`pg_namespace` po bazi.

`db-table-sizes` - **pojedina tablica**, prvih 25 po ukupnoj veličini. `total`
je `pg_total_relation_size()` - heap **plus** njegova TOAST tablica i TOAST
indeks **plus** svaki indeks na njoj **plus** mape slobodnog prostora i
vidljivosti. `table_only` je sam heap; `indexes` je `pg_indexes_size()`. Dakle
`total` minus `table_only` minus `indexes` je TOAST i mape, što za tablicu sa
širokim `text`/`jsonb` stupcima može biti većina veličine. Particije se rolaju
na korijen particije - particionirana tablica pokazuje se kao jedan redak s
brojem `partitions`, ne kao zid dječjih redaka s roditeljem na nuli.
`live_rows` i `dead_rows` su **procjene** kolektora statistike, ne `count(*)`;
`stats_age` je koliko je davno zadnji `ANALYZE` osvježio te brojeve, a velik
`stats_age` znači da su brojevi redaka zastarjeli u smjeru u kojem se tablica
mijenjala.

`db-index-sizes` - **pojedini indeks**, prvih 50 po veličini indeksa, svaki s
veličinom matične tablice pored sebe pa je indeks koji je narastao veći od
podataka koje indeksira očit na prvi pogled. Taj omjer je signal: btree indeks
veći od svoje tablice ili indeksira jako široke ključeve ili je napuhan (vidi
sljedeći odjeljak), i vrijedi ga pogledati u oba slučaja. GIN ili GiST indeks
koji legitimno zasjeni svoju tablicu je normalan - pohranjuju daleko više od
jednog zapisa po retku.

Zamka zajednička svima četvorici: ovo su **veličine na disku**, koje uključuju
mrtve n-torke i slobodan prostor koji bi `VACUUM FULL` ili `pg_repack`
vratio. Tablica koja pokazuje 4 GB nije 4 GB podataka ako je pola napuhanost.
Iznenađujuću veličinu usporedite s `db-bloat-breakdown` ili `db-pgstattuple`
prije nego zaključite da vam treba veći disk.

#### Napuhanost: procjena i pravi broj {#bloat-the-estimate-and-the-real-number}

„Napuhanost" je prostor koji relacija zauzima na disku iznad onoga što njeni
živi retci trebaju - mrtve n-torke koje autovacuum nije vratio i stranice
ostavljene dijelom prazne nakon brisanja i ažuriranja. Tri izvještaja se time
bave, od brzog-i-približnog do sporog-i-točnog, i svaki od njih koji se
temelji na procjeni - raščlamba po tablici, rang-lista indeksa, Advisor tab i
redak *WASTED SPACE* pregleda zdravlja - računa isti kod i za istu relaciju
dolazi do iste brojke, jer svi pozivaju jedan zajednički modul umjesto da ga
svaki računa zasebno. **Procjene čitajte kao „vrijedi istražiti", nikad kao
mjeru.**

`db-bloat-breakdown` je mjesto od kojeg treba krenuti. Po tablici čita
stvarne veličine (`pg_relation_size`, `pg_indexes_size`,
`pg_total_relation_size`, i `pg_total_relation_size` nad TOAST relacijom) i
kombinira ih s *očekivanom* („zapakiranom") veličinom izračunatom iz
`pg_stats` (broj stupaca, prosječna širina, udjeli null-ova) za heap, istim
modelom po stavci za svaki btree indeks, i vlastitim omjerom mrtvih n-torki
TOAST tablice - jedan `est_waste` / `est_waste_pct` za **cijelu tablicu**,
heap i TOAST i svaki indeks zbrojeni zajedno, ne samo heap. To je popravak
za tablicu koja je ovdje nekad pokazivala „0% napuhanosti" dok su samo njeni
indeksi nosili stotine gigabajta procijenjenog otpada: brojka ih sad
uključuje. Ispisuje stupac `recommendation` izveden iz istih zbrojeva koje
prikazuje, tako da se savjet i brojka više ne mogu ne slagati. Taj stupac
čitajte kao **poticaj, ne uputu** - ne zna imate li instaliran `pg_repack`.
Redak „VACUUM FULL or pg_repack" na tablici od 20 MB zadovoljava obični
`VACUUM`; isti redak na zauzetoj tablici od 200 GB znači online pregradnju i
plan održavanja.

**Kad je procjena jedne komponente nepouzdana, to ne isprazni cijeli redak.**
Formula zapakirane veličine jedne komponente - obično heapa velike ili dugo
neanalizirane tablice - može ispasti *veća* od stvarne veličine te
komponente: zastarjela statistika, ili retci neobično široki ili jako
toastani da bi im odgovarao model `pg_stats` po stupcu. To ne znači „nema
otpada", nego „procjena ove komponente ovdje ne vrijedi" - a ono što *druge*
komponente (TOAST, svaki indeks) i dalje znaju se time ne odbacuje.
`est_waste` tada postaje **donja granica**: označena s `≥` i uz naznaku što
nedostaje, npr. `≥ 448 GB (heap unknown)`, umjesto izmišljene nule ili
praznog „n/a" - stvarni zbroj je ta brojka ili veći, nikad manji.
`est_waste_pct` nosi istu `≥` oznaku kad se primjenjuje. `ANALYZE`, ili
`db-pgstattuple` niže, put je preko toga konkretno za komponentu koja
nedostaje - napomena samog retka navodi koju i zašto: nikad analizirana,
statistika zastarjela N dana uz udio redaka promijenjenih otad (oboje vodi
na `ANALYZE`), ili aktualna statistika koja ipak ne odgovara obliku
tablice (vodi na `db-pgstattuple` umjesto toga, jer ponovni `ANALYZE` ništa
ne bi promijenio). Isto ovo pravilo - poznate komponente ostaju vidljive,
one koje nedostaju su imenovane, ništa se ne prazni - vrijedi svugdje gdje
raščlamba zbraja više komponenti, uključujući redak *WASTED SPACE* pregleda
zdravlja.

**`est_waste_pct` i `dead_pct` su namjerno različiti brojevi.**
`est_waste_pct` je prostor: bajtovi koje tablica (heap, TOAST, indeksi) drži
ali ne koristi. `dead_pct` je `n_dead_tup / (n_live_tup + n_dead_tup)` - broj
n-torki, i samo mjera koliko autovacuum zaostaje. Tablica koju `UPDATE` jako
melje može istovremeno biti 4% mrtva i 90% potrošena. `idx_to_heap` je
ukupni bajtovi indeksa kroz bajtove heapa, gdje mnogo iznad 1 vrijedi
reviziju nekorištenih/dupliciranih indeksa.

`db-bloat-indexes` odgovara na drugo pitanje: ne „koja je tablica najgora"
nego „koji jedan indeks, bilo gdje u clusteru, sada vrijedi `REINDEX`" -
ista aritmetika po indeksu kao u redcima indeksa raščlambe, ali rangirana
kroz cijeli cluster umjesto grupirana po tablici. Pretpostavlja `fillfactor`
90 i pokriva **samo btree** - GIN, GiST, BRIN, hash i izrazni indeksi dobiju
stvarnu veličinu i nikakvu procjenu, što je procjenitelj koji odustaje, ne
čist indeks. **`VACUUM` nikad ne zbija indeks**; rješenje koje imenuje je
`REINDEX` (`... CONCURRENTLY` na PostgreSQL 12+, provjereno prema spojenom
serveru da starijem ne bude rečeno da izvrši naredbu koju nema), jedan redak
po indeksu. Tablica namjerno građena s `fillfactor` ispod 100, da HOT
ažuriranja ostanu jeftina, ovdje će taj slobodan prostor također pokazati kao
„napuhanost" - to je prostor koji ste tražili, ne otpad.

`db-pgstattuple` je pravi broj, za obje polovice. Čita **svaku stranicu**
svake tablice kroz ekstenziju `pgstattuple` za istinske postotke mrtvih
n-torki i slobodnog prostora, a zatim `pgstatindex()` na btree indeksima te
tablice za pravu brojku gustoće listova po indeksu - ni na jednoj strani
model `pg_stats`, nego stvaran pregled stranica. Točan je i spor koliko taj
pregled implicira: trošak je veličina tablice u I/O-u, svaki put. Koristite
ga da potvrdite određenu tablicu koju je procjena označila - ili onu koju je
označila kao nepouzdanu - ne da pregledate bazu. Parametri raspona veličine
su proračun vremena: zadano preskače tablice ispod 1 MB (napuhanost tamo je
pogreška zaokruživanja) i iznad 5 GB (povećajte kad baš želite točan broj na
velikoj tablici i možete čekati). Ekstenzija nudi i `pgstattuple_approx`,
koja preskače stranice koje mapa vidljivosti označava kao potpuno vidljive
pa radi daleko brže na dobro vacuumiranoj tablici uz nešto manje preciznosti
- vrijedi znati za nju kad je potpuna pretraga prespora.

**Praktičan redoslijed:** `db-bloat-breakdown` da vidite oblik toga kroz sve
tablice, `db-bloat-indexes` za rangiran popis osumnjičenika kroz cijeli
cluster, `db-pgstattuple` na jednoj ili dvije tablice na kojima ćete stvarno
djelovati (ili bilo koji redak označen kao nepouzdan). Pa savjetnik po
indeksu (gore) za točnu naredbu `REINDEX` ili `VACUUM`.

#### Shared buffers: što je stvarno u kešu {#shared-buffers-what-is-actually-cached}

`db-buffer-cache` otvara `pg_buffercache` i pokazuje koje relacije upravo sad
zauzimaju PostgreSQL-ov shared buffer pool - prvih 30 po broju buffera, sa
ukupnom veličinom svake relacije, njenim udjelom u `shared_buffers` i koliko
je same relacije u kešu.

**Trošak je stvaran i vrijedi ga znati prije nego to pokrenete na
produkciji.** Pogled `pg_buffercache` prolazi cijeli buffer pool; na
poslužitelju sa `shared_buffers` od nekoliko gigabajta to je milijuni buffera
pregledanih pod partition lockovima, i pokaže se kao kratak skok latencije za
sve ostale. Pokrenite ga kad vam treba, ne po rasporedu, i radije u mirnom
trenutku. Na PostgreSQL-u 16 i novijem jeftinije funkcije
`pg_buffercache_summary()` i `pg_buffercache_usage_counts()` daju gornju sliku
bez prolaska po svakom bufferu - koristite njih ako želite samo „koliko je keš
pun".

Kako čitati dva postotka:

- **`pct_of_relation`** - koliko je ove relacije u kešu. Mala, vruća lookup
  tablica na 100% je točno to. Tablica od 200 GB na 3% je također točno - ne
  biste htjeli da izbacuje sve ostalo. Ovaj broj je problem samo kad tablica
  za koju *očekujete* da se poslužuje iz memorije stoji nisko.
- **`pct_of_buffers`** - udio ove relacije u cijelom poolu. Alarm je samo kad
  jedna ili dvije relacije dominiraju njime a tablice do kojih vam je stalo su
  izgladnjele. Zdrav pool je obično dugi rep.

Ako je vrh popisa TOAST tablice i indeksi koje ne prepoznajete, to je
normalno - stranice indeksa i TOAST komadi keširaju se kao i sve drugo.
Provjera je za odgovor na „zašto mi radni skup ne ostaje u memoriji", a
odgovor je obično ili da je `shared_buffers` premalen za radni skup ili da
jedan batch posao opetovano provlači veliku hladnu tablicu kroz njega.

#### TimescaleDB {#timescaledb}

`db-timescaledb` čita proširenje TimescaleDB baze na koju ste spojeni. Za sve
niže treba TimescaleDB 2; s instaliranim 1.x to kaže, a preglednik sheme i
dalje prikazuje hipertablice i njihove chunkove.

**Izdanje.** Prvi redak navodi verziju i postavku licence
(`timescaledb.license`). Pod `apache` - izdanjem samo s Apache licencom -
poslužitelj sam odbija kompresiju, kontinuirane agregate, politike
zadržavanja, kompresije, osvježavanja i preslagivanja te korisničke poslove;
hipertablice, chunkovi, `drop_chunks` i `time_bucket` rade. Ploča te radnje
prikazuje zasivljene i kaže zašto. Ako je na poslužitelju instalirana novija
verzija nego u bazi, navodi se s naredbom `ALTER EXTENSION timescaledb UPDATE`
koja je primjenjuje.

**U pregledniku sheme** hipertablice imaju vlastitu kategoriju; svaka kaže
kako je podijeljena na chunkove (vremenski stupac i interval chunka te
prostorne particije ako ih ima), koliko chunkova ima i koliko ih je
komprimirano; broj redaka je zbroj njezinih chunkova, jer je vlastita tablica
hipertablice prazna. Otvorena prikazuje veličinu, omjer kompresije i
chunkove, najnovije prve (najnovijih 500; ostali se prebroje). Chunk se
otvara kao tablica koja jest, a njegov izbornik ga komprimira ili
dekomprimira. Sheme TimescaleDB-a (`_timescaledb_internal`,
`_timescaledb_catalog`, `timescaledb_information` i ostale) i relacije koje
on vodi - chunkovi, komprimirani podaci, tablice i pogledi iza kontinuiranih
agregata - skrivene su kao i drugi objekti proširenja, iza istog prekidača
"prikaži".

**Što provjera procjenjuje**, s brojevima po kojima procjenjuje:

- **Posao koji stalno pada** - dva neuspjela izvođenja zaredom od zadnjeg
  uspjeha (broji se iz `timescaledb_information.job_errors`; prije 2.9, koji
  nema taj pogled, poznato je samo zadnje izvođenje). Alarm.
- **Posao koji se nije izveo po rasporedu** - njegov sljedeći početak je više
  od 5 minuta u prošlosti. Planer pokreće posao unutar nekoliko sekundi kad je
  pozadinski radnik slobodan, pa minute kašnjenja znače da nijedan nije.
  Alarm.
- **Planer ne radi** za ovu bazu dok su poslovi zakazani - obično je
  `timescaledb.max_background_workers` premalen (treba jednog radnika po bazi
  s TimescaleDB-om plus jednog po poslu koji radi istodobno) ili je
  `max_worker_processes` potrošen. Alarm.
- Vlastiti pomoćni poslovi TimescaleDB-a (telemetrija, zadržavanje povijesti
  poslova) prijavljuju se kad padnu, nikad kao alarm: telemetrija pada na
  svakom poslužitelju bez pristupa internetu.
- **Kompresija zaostaje za politikom** - chunkovi stariji od `compress_after`
  politike koji su još nekomprimirani, svaki s gumbom Komprimiraj.
- **Zadržavanje zaostaje za politikom** - chunkovi stariji od `drop_after` još
  su tu.
- **Kontinuirani agregat zaostaje** - vrijeme od njegove vodene oznake
  (watermark, ono što je materijalizirano) do sada, u odnosu na ono u čemu ga
  politika osvježavanja treba držati: `end_offset` plus raspored. Označava se
  iznad dvostrukog toga i iznad 15 minuta, pa politika koja osvježava svake
  minute nije označena zbog trenutnog kašnjenja. Bez politike to se svakako
  kaže, kao upozorenje kad je agregat `materialized_only` (upiti tada ne vide
  ništa iza vodene oznake).
- **Interval chunka u odnosu na memoriju** - Timescaleovo pravilo: chunkovi u
  koje se upisuje, s indeksima, svih hipertablica zajedno trebaju stati u
  otprilike 25% glavne memorije, što je udio koji `timescaledb-tune` daje
  `shared_buffers`. Provjera mjeri najnoviji završeni interval svake
  hipertablice u koju se upisivalo u zadnjih 7 dana i njihov zbroj uspoređuje
  sa `shared_buffers`; iznad toga svaka hipertablica dobiva interval smanjen
  da stane. Označava se i suprotno: više od 1000 chunkova čiji interval drži
  manje od 1% `shared_buffers` ili manje od 10 MB - svaki chunk je tablica
  koju planer razmatra. Novi interval (`set_chunk_time_interval`) vrijedi za
  chunkove napravljene od tada.

Upozorenje ili alarm na popisu iznad ploče ima uobičajeni gumb "Dodaj na
popis zadataka", a alarmi pale lampicu TimescaleDB u zaglavlju (isto
očitanje koje Health prikazuje kao TIMESCALEDB POSLOVI).

**Radnje.** S ploče: dodati ili ukloniti politiku kompresije ili
zadržavanja, komprimirati jedan chunk, osvježiti kontinuirani agregat za
prozor, pokrenuti posao sada, promijeniti mu raspored, pauzirati ga i
nastaviti; iz izbornika chunka: komprimirati ga ili dekomprimirati. Svaka
najprije prikazuje svoju naredbu i onu koja je poništava - uklanjanje
politike čuva njezinu postavku u naredbi koja je vraća - i obje idu u
revizijski zapisnik. To smije samo vlasnik hipertablice, agregata ili posla,
ili član te uloge; ploča pita `pg_has_role(owner, 'USAGE')` i zasivljuje ono
što ne smijete, a poslužitelj pita ponovno.

**Ostatak aplikacije na bazi s TimescaleDB-om:**

- Usporedba uspoređuje hipertablicu kao tablicu, ali i po tome kako je
  podijeljena na chunkove i komprimirana, pa se obična tablica i
  hipertablica, ili dva intervala chunka, razlikuju. Chunkovi se ne
  uspoređuju: imenovani su redom nastanka, i `_hyper_1_5_chunk` na jednoj
  strani drugi je vremenski raspon od istog imena na drugoj. Drugačiji
  interval dobiva `set_chunk_time_interval`; pretvaranje tablice u
  hipertablicu ostaje kao zakomentirana naredba, jer premješta svaki redak.
- Vraćanje dump s TimescaleDB-om obavija onako kako TimescaleDB traži:
  `timescaledb_pre_restore()` prije, `timescaledb_post_restore()` poslije -
  i kad vraćanje ne uspije, pa baza nikad ne ostaje sa zaustavljenim
  poslovima. U praznom cilju proširenje se najprije stvori. Cilj mora imati
  verziju TimescaleDB-a iz koje je dump nastao. Na vraćanje samo dijela
  takvog dumpa ili s `--clean` upozorava se: hipertablica bez svojih redaka
  kataloga i chunkova vraća se kao prazna obična tablica.
- Promjena vlasnika hipertablice premješta i njezine chunkove (to radi
  TimescaleDB). Kopija definicija uzeta prije promjene sadrži hipertablicu,
  ne njezine chunkove, i naredbu koja je ponovno pretvara u hipertablicu.
- Karta relacija crta hipertablicu kao jednu karticu s brojem chunkova i
  povezuje kontinuirani agregat s hipertablicom koju agregira.
- Ostale provjere koje navode tablice (veličine, bloat, vacuum) uz svaki
  chunk navode hipertablicu ili kontinuirani agregat.

#### Vacuum, napuhanost i statistika tablica {#vacuum-bloat-and-table-statistics}

`db-vacuum` po tablici pokazuje kad je zadnji put vacuumirana i analizirana, broj živih i mrtvih n-torki, postotak promijenjen od zadnje analize i oznaku vrućine, uz preporučeni skup autovacuum reloptiona pored trenutnog. Oznake su najvažnije: **rizik od WRAPAROUND-a** je za odmah; **autovacuum isključen** na opterećenoj tablici obično je zaboravljena greška; **insert-only** tablici treba `autovacuum_vacuum_insert_threshold`, jer zadani vacuum prag za nju nikad ne okine.

`db-autovacuum` je prikaz uživo: koji autovacuum radnici rade upravo sad i što svaki radi. Prazno je normalno stanje. Radnik koji je dugo na istoj velikoj tablici, ili `max` radnika svi zauzeti dok zaostatak vacuuma u `db-vacuum` i dalje raste, znači da autovacuum ne stiže - povećajte `autovacuum_max_workers` ili `autovacuum_vacuum_cost_limit`, ili smanjite scale factor po tablici na najgorim slučajevima.

`db-bloat-breakdown` razdvaja napuhanost heapa, TOAST-a i indeksa po tablici, zbraja ih u jedan `est_waste` za cijelu tablicu i imenuje sljedeći korak - VACUUM, REINDEX, `pg_repack` ili ništa. Njegov stupac `recommendation` je poticaj izveden iz istih zbrojeva koje prikazuje, ne uputa: *Napuhanost: procjena i pravi broj* (gore) pokriva što uzima u obzir a što ne, i kako se rang-lista indeksa kroz cijeli cluster (`db-bloat-indexes`) i točna pretraga (`db-pgstattuple`) slažu oko toga.

Redak *WASTED SPACE (est.)* u **pregledu zdravlja** koristi istu kombiniranu procjenu heap-plus-TOAST-plus-indeksi, zbrojenu kroz bazu, i označi tablicu prema **onome što prvo okine**: apsolutnih **20 GB** procijenjenog otpada u jednoj tablici (crveno na **200 GB**), ili **25% ili više** tablice potrošeno kad je to barem **1 GB**. Apsolutna brojka postoji jer je 40% tablice od 200 MB kućanski posao, dok je 6% tablice od 20 TB više od terabajta diska i sati dodani svakom vraćanju iz kopije. Tablica s jednom nepouzdanom komponentom (obično heapom) se **ne** izostavlja iz ovog retka - njene ostale poznate komponente i dalje se broje, kao donja granica označena s `≥`, po istom pravilu kao redak po tablici u `db-bloat-breakdown`; tamo piše koja komponenta nedostaje i zašto. Dubinski prikaz izlista najgore tablice, a Advisor tab po tablici razloži svaku po komponenti. `pg_repack` i `VACUUM FULL` vraćaju heap/TOAST prostor operativnom sustavu; obični `VACUUM` ga samo oslobodi za ponovnu upotrebu unutar iste tablice; indeksu treba `REINDEX`.

**Advisor tab** u pregledniku sheme (odaberi tablicu -> Advisor) je verzija svega ovoga po tablici na jednom mjestu. Prikazuje **raščlambu napuhnuća** po komponenti - heap, TOAST, svaki indeks - s tri broja za svaku: što je na disku (točno), koliko je od toga procijenjeni **korisni podatak**, a koliko procijenjeni **otpad**. Veličina indeksa nije otpad; otpad indeksa je prostor koji svježi `REINDEX` ne bi trebao. Velik TOAST sam po sebi nije problem - otpad unutar njega jest. „Otpad" je usidren na stvarnu veličinu na disku (ista aritmetika kao `db-bloat-breakdown`), pa ostaje točan i kad je `pg_class.relpages` zastario, a redak jedne komponente umjesto broja pokazuje `?` kad njeni ulazni podaci ne pristaju. Redak **ukupno** to nikad ne radi: označen je s `≥` i navodi komponentu koja nedostaje (uz kratak razlog i `ANALYZE`/Točan scan radnje gdje se primjenjuju) čim jedna komponenta nije mogla biti procijenjena, umjesto da pokaže izmišljenu nulu ili praznu brojku. Procjene pretpostavljaju fillfactor, a TOAST brojka je samo omjer mrtvih tuplova, pa pokreni *Točan scan (pgstattuple)* za stvarne brojke prije nego djeluješ na nečemu velikom. Blok **Analiza i prijedlozi** ispod pretvara brojke u uvjetne savjete s gotovim SQL-om za kopiranje: otpad koji je reupotrebljiv i autovacuum stiže se ne dira; napuhana datoteka dobije `pg_repack` ili `VACUUM FULL`; autovacuum koji ne stiže cross-linka per-tablicu postavke koje je Autovacuum advisor izračunao iz brojki te tablice; napuhani indeksi dobiju per-indeks `REINDEX INDEX CONCURRENTLY` (ili plain `REINDEX` na PostgreSQL < 12, s oznakom); otpad u TOAST-u se posebno naznači jer ga plain `VACUUM` ne kompaktira.

Tab Advisor ima pet odjeljaka, u istim karticama kakve koristi rezultat provjere, s trakom ispod naslova koja skače na bilo koji od njih: **Tablica** (njezini brojevi), **Vacuum i autovacuum**, **Indeksi**, **Održavanje** i **Dubinska analiza**. Odjeljak o vacuumu počinje time kako vacuum postupa s ovom tablicom - koliko je pokretanja bilo od resetiranja statistike i koliko ih je na dan, razmak među njima, na PostgreSQL-u 18 koliko traju po danu, udio mrtvih redaka i najstariji snapshot koji išta u bazi drži - i kaže, istim riječima kao `db-vacuum`, kad se tablica vacuumira jednom na sat ili češće, kad njezini vacuumi ne postižu ništa jer transakcija, pripremljena transakcija, replikacijski slot ili standby drže snapshot stariji od njezinih mrtvih redaka (i što treba završiti), kad je svaki njezin vacuum agresivan i kad autovacuum radi bez kočnice. Preporuka nikad ne pooštrava pragove tablici koja se već vacuumira jednom na sat. Zatim slijede linije okidača - kad se autovacuum pokreće danas i nakon preporuke - četiri parametra i gumbi koji ih primjenjuju. Kad lock drži vacuum podalje od tablice, taj nalaz stoji prvi i imenuje sesiju koja ga drži.

**Kad je često vakuumiranje u redu.** Tablica koja se vacuumira jednom na sat ili češće - najmanje 24 pokretanja na dan *i* najmanje 50 pokretanja u prozoru statistike; svakih 15 minuta je "vrlo često" - ne proglašava se problemom samo zbog broja. Važu se još četiri stvari, a nalaz kaže koja od pet presuda vrijedi:

- **Okidač.** Autovacuum se pokreće kad mrtvi redovi prijeđu `threshold + scale_factor × redovi` (prvo postavka same tablice, zatim poslužitelja; od PostgreSQL-a 18 ograničeno s `autovacuum_vacuum_max_threshold`). Nalaz kaže koji dio odlučuje - na maloj tablici fiksni prag (50 + 0.2 × 120 redaka je uglavnom onih 50), na velikoj faktor skaliranja - i koliki je udio tablice okidač.
- **Posao koji svako pokretanje nađe.** Redovi ažurirani ili obrisani po pokretanju (od PostgreSQL-a 13 i umetnuti redovi, prema okidaču za umetanja). Pokretanje *čisti* kad to dosegne **pola okidača ili 10.000 redaka**; *prazno* je kad je ispod **10% okidača i ispod 1.000 redaka**. Zakazani ručni `VACUUM` je uobičajeni prazni slučaj: pokreće se bez obzira je li se išta promijenilo.
- **Koliko jedno pokretanje košta.** Na PostgreSQL-u 18 mjeri se po tablici. Prije 18 procjenjuje se iz kočnice: stranice koje pokretanje pročita (one koje mapa vidljivosti ne označava kao all-visible, one koje su promjene dotaknule i svi indeksi kad pokretanje vacuumira indekse - od PostgreSQL-a 14 preskočeno kad manje od 2% stranica drži mrtve unose indeksa), svako čitanje kao čitanje s diska po `vacuum_cost_page_miss`, svaka uprljana stranica po `vacuum_cost_page_dirty`, tempom od `cost_limit` jedinica po `cost_delay`. Cost limit dijele autovacuum workeri koji rade istodobno, osim ako tablica nema vlastite postavke troška, pa ga procjena dijeli brojem workera koji sada rade. Ručni `VACUUM` po zadanom radi s `vacuum_cost_delay` 0 - bez kočnice - i njegovo se trajanje tako ne može procijeniti. Na PostgreSQL-u 18 procjena ostaje kao objašnjenje: pokretanja traju otprilike koliko kočnica dopušta (ograničena kočnicom: viši cost limit bi ih skratio), kraće (stranice su došle iz cachea) ili dulje (čekaju disk, lockove ili CPU). Nalaz daje i gornju granicu čitanja i pisanja koju kočnica određuje. *Jeftino* je **najviše 5 sekundi po pokretanju i najviše 15 minuta na dan**.
- **WAL koji zapiše.** Uvijek procjena - nijedna verzija PostgreSQL-a ne broji WAL po tablici: oko 100 bajtova po uprljanoj stranici, plus slika cijele stranice za stranicu koja nije zapisana od zadnjeg checkpointa. Stranica koju je radno opterećenje promijenilo nakon zadnjeg checkpointa ne snima se ponovno, pa tablica koja se vacuumira *češće* nego što se događaju checkpointi plaća daleko manje slika nego što broj pokretanja sugerira; nalaz uspoređuje razmak između pokretanja sa stvarnim intervalom checkpointa i kaže koji slučaj vrijedi. Od PostgreSQL-a 14 procjena je ograničena izmjerenim WAL-om klastera i stavljena u odnos s njim. Velik je **1 GB na dan i 10% WAL-a klastera** (upozorenje od **10 GB i 30%**). `wal_compression` smanjuje slike; nalaz ga spominje, ali ga ne modelira.

Presude: **ok** - jeftino, čisti, mrtvi redovi najviše 1,5 × okidač (ili ispod 1.000), WAL nije velik: prikazano kao OK, pokretanja na dan zeleno, ništa ne treba mijenjati. **idle** - vacuumira se češće nego što se tablica mijenja (napomena kad su pokretanja jeftina; upozorenje kad su vrlo česta i nisu jeftina, ili im je trošak nepoznat): zakažite ručni `VACUUM` rjeđe; za vlastita pokretanja autovacuuma pogledajte okidač za umetanja i starost ID-a transakcije. **costly** - skupa pokretanja na tablici od najmanje 100.000 redaka čiji je okidač ispod 1% nje, s brzinom pisanja te pragom i faktorom skaliranja koji bi je vacuumirali otprilike jednom na sat - te vrijednosti postaju preporučene vrijednosti koje nose **Apply** i naredba ALTER; ili pokretanja u svemu dobra osim WAL-a. **busy** - učestalost dolazi od brzine pisanja, ne od postavki: rjeđe vakuumiranje pustilo bi da napuhanost raste, a svako pokretanje postaje jeftinije s više HOT ažuriranja (fillfactor), manje indeksa ili blažom kočnicom. **unknown** - trošak se ne može reći. Preporuka nikad ne pooštrava pragove često vakuumirane tablice, kakva god bila presuda; skupa se umjesto toga popušta na okidač od jednom na sat, a razlog stoji uz preporučene vrijednosti.

**Kako se računa „na dan“.** Svaka dnevna brojka je broj pokretanja od početka prozora statistike tablice podijeljen s trajanjem prozora - prosjek preko tog prozora - a kartica i nalaz kažu gdje prozor počinje, na primjer *since 2026-10-03 14:20 CET (25.0 days, this database's statistics reset)*. Početak je, ovim redom: na PostgreSQL-u 19 vrijeme reseta same tablice; inače zadnji reset statistike ove baze; za bazu koja nikad nije resetirana (PostgreSQL 15 i noviji ostavljaju to vrijeme prazno) najraniji reset statistike samog servera; ako ničega od toga nema, pokretanje servera. PostgreSQL pomakne vrijeme reseta baze i kad se resetiraju brojači samo *jedne* tablice (`pg_stat_reset_single_table_counters`), pa brojači ostalih tablica mogu biti stariji od njega. Reset briše i vremena zadnjeg vacuuma i analyzea tablice, pa tablica s vremenom starijim od reseta baze nije resetirana s njom: dobiva **bez stope** umjesto brojke napuhane kratkim prozorom, a odjeljak *Autovacuum cost and I/O* broji takve tablice u jednom retku. Prozor kraći od sat vremena daje projekciju, prikazanu bez boje. Šesta presuda, **past**, je tablica u prosjeku česta čiji je zadnji vacuum bio prije barem deset njezinih prosječnih razmaka i tri sata: prosjek dolazi od ranije aktivnosti, pa se ništa ne označava i nijedna preporuka se zbog toga ne zadržava; trenutnu stopu po prozoru mjere AWR snimke. Na PostgreSQL-u 15 i novijem baza stvorena nakon početka statistike servera dijeli se duljim prozorom, pa njezina stopa može ispasti niska, nikad visoka.

Autovacuum posjećuje svaku bazu jednom po `autovacuum_naptime`, pa jednu tablicu može vakuumirati najviše toliko često; tablica vakuumirana pri većini tih posjeta dospijeva gotovo svaki put, i nalaz to kaže. Kad su autovacuum workeri zasićeni - na PostgreSQL-u 18 zauzeti **najmanje 50% kapaciteta i 6 worker-sati na dan** (upozorenje od **80% i 12 sati**), izmjereno iz vremena koje su tablice ove baze provele u autovacuumu i autoanalyzeu, donja granica jer druge baze pridodaju svoje; na 19 i svi workeri zauzeti dok tablice čekaju - dospjele tablice čekaju red, a savjet je više workera ili blaža kočnica, ne drugi pragovi.

Svaki nalaz završava time koliko su mu brojke čvrste: PostgreSQL 18 i noviji mjere trajanje pokretanja svake tablice; 16 i 17 mjere samo I/O klastera; 14 i 15 sve procjenjuju po tablici; na 13 nalaz je samo učestalost.

Sekcija **Maintenance actions** u Advisor tabu također prikazuje bilo koji VACUUM / ANALYZE / REINDEX / VACUUM FULL koji se **upravo sada** izvodi na toj tablici, čitano sa servera po relation id-u - pa se pojavi bez obzira je li ga pokrenula aplikacija, drugi alat, ili je prozor tek otvoren. Svaki prikazuje živi progres iz `pg_stat_progress_*` i **Stop** koji ga otkazuje na serveru; zaustavljanje tuđe operacije prvo traži potvrdu. Na PostgreSQL 12+ sigurniji `REINDEX CONCURRENTLY` (koji ne uzima `ACCESS EXCLUSIVE` lock) je prvi; ispod 12 je prikazan onemogućen uz razlog.

**Iz spremljene povijesti (AWR i ASH).** Sve gore čita brojače od zadnjeg resetiranja statistike: jedan prosjek kroz koliko god to dugo bilo. Kad su spremljene AWR snimke povezane baze, `db-vacuum` i kartica Advisor čitaju i njih: po tablici, retke ažurirane, obrisane i umetnute u svakom prozoru između dvije snimke, koliko je često vacuum radio i koliko je tablica bila daleko od svog okidača u svakoj snimci, s postavkama koje su tada vrijedile (snimke ranije verzije nemaju te podatke). Iz prosječne brzine određuju okidač za otprilike **jedno pokretanje na sat**, nikad blaži od 20% tablice. Današnji okidač se **pooštrava** kad je barem **dvostruko** veći, pušta barem **10.000** mrtvih redaka da čekaju, a tablica se mijenja barem 500 redaka na sat; **ublažava** se kad je najviše upola manji na tablici koja se vakuumira svaki sat ili češće (barem 10 pokretanja u povijesti); sve unutar faktora dva **odgovara**. Od PostgreSQL 13 tablica s pretežno umetanjima (90% pisanja su umetanja) na isti način dobiva insert par. Te vrijednosti zamjenjuju heuristiku po veličini u preporuci koju nose **Apply** i ALTER naredba, a razlog pored njih navodi brzine i najprometniji prozor. Tablica koja je ostala preko okidača barem **sat vremena kroz dvije ili više snimki** posebno se ističe (upozorenje od **šest sati**), i tada se ne nude novi pragovi: okidač nije ono što određuje kad se vakuumira. Kad su spremljeni ASH uzorci, oni kažu zašto: autovacuum worker na tablici **čeka lock** (barem 10% njegovih uzoraka i barem 5, ili 60 uzoraka sami za sebe), uz sesiju koja ga je držala; spava u svojoj **kočnici troška** (`VacuumDelay`: barem polovica uzoraka i barem 30, ili 600 sami za sebe); ili su **svi autovacuum workeri zauzeti istodobno** (barem polovica uzoraka s workerom na poslu i barem 60, ili 1.800 sami za sebe). S manje od sat povijesti ništa se ne određuje. Blok **Dokazi** navodi korištene snimke i prozore, od kad do kad, i ASH dane, ili kaže da povijesti nema i da je savjet onakav kakav je uvijek bio.

`db-table-stats` je broj sekvencijalnih naspram indeksnih pretraga po tablici, plus `last_seq_scan` / `last_idx_scan` na PostgreSQL 16+. Tablica s milijunima sekvencijalnih pretraga i rastućim brojem sekvencijalno se pretražuje u produkciji sada - usporedite je s `db-index-advisor`.

#### Sekvence, checkpointi i tablespaceovi {#sequences-checkpoints-and-tablespaces}

`db-sequences` i `db-seq-usage` popisuju sekvence po tome koliko su blizu maksimumu, s vlasničkim stupcem i svakim defaultom koji ih referencira. `integer` sekvenca na 80% je prelazak na `bigint` koji želite isplanirati, a ne otkriti kad `nextval` pukne.

Maksimum koji se računa manji je od dva: vlastiti `MAXVALUE` sekvence i najveća vrijednost koju stupac koji puni može primiti (32.767 za `smallint`, 2.147.483.647 za `integer`). Prije PostgreSQL-a 10 sekvenca nije imala tip - svaka je bila `bigint` - pa je svaki tada napravljen `serial` stupac `integer` koji puni `bigint` sekvenca, a nadograđena shema taj oblik zadržava. Mjeren samo prema sekvenci, takav stupac pokazuje 0% iskorištenosti dok mu je ostalo nekoliko tisuća ID-jeva; sljedeći unos preko ruba pada s *integer out of range*. `db-sequences`, `db-seq-usage` i redak SEQUENCES u pregledu zdravlja mjere prema stupcu i ocjenjuju po dvije stvari, dovoljna je bilo koja:

- **udio iskorištenog raspona**: žuto od 80%, crveno od 95%;
- **preostalo vrijeme**: preostali ID-jevi podijeljeni s brojem unosa dnevno u tablicu stupca (njezin brojač unosa kroz prozor statistike - isti prozor koji navodi Advisor tablice): žuto ispod 180 dana, crveno ispod 30. Prozor kraći od sat vremena čini ritam procjenom: vrijeme se prikaže, kaže se da je procjena, i ne mijenja boju.

Svaka sekvenca koja se troši navodi stupce stranih ključeva koji pokazuju na nju i uži su od `bigint`: prebacivanje stupca na `bigint` prebacuje problem na njih, pa se mijenjaju u istoj migraciji. Kad je sam stupac već `bigint`, uži stupac stranog ključa postaje strop: na prvi ID preko njegova maksimuma ne može se pokazati, pa se preostali prostor mjeri prema njemu i on se navodi. `ALTER COLUMN ... TYPE bigint` prepisuje cijelu tablicu pod `ACCESS EXCLUSIVE` lockom - nitko je ne čita ni ne piše dok ne završi; put bez prekida je novi `bigint` stupac koji trigger drži usklađenim, punjenje u serijama i zamjena.

`db-seq-sync` piše `SELECT setval(...)` za svaku sekvencu, postavljajući svaku tik iznad najveće vrijednosti trenutno u njenom vlasničkom stupcu. Pokrenite je nakon prebacivanja logičke replikacije ili `pg_dump --data-only` punjenja, gdje retci stignu ali sekvence ostanu na 1 pa se sljedeći `INSERT` sudari s postojećim ključem. Generirana je skripta kao i svaka druga - pregledajte je, pa je pokrenite na cilju.

`db-checkpoints` pokazuje kako checkpointi okidaju: **tempirani** (dobro, vođeni s `checkpoint_timeout`) naspram **traženih** (nametnutih pritiskom `max_wal_size`), prosječan razmak među njima te vrijeme pisanja i sinkronizacije. Visok udio traženih znači da je `max_wal_size` premalen za brzinu pisanja - povećanje mijenja prostor na disku za puno manje checkpoint I/O-a.

`db-tablespaces` popisuje svaki tablespace s lokacijom, vlasnikom, veličinom, opcijama (override-i za `seq_page_cost`, `random_page_cost`, `effective_io_concurrency`) i bazama koje ga koriste, plus sadržaj svakog tablespacea u spojenoj bazi. Odjeljak sa sadržajem po bazi vezan je uz trenutnu konekciju iz istog razloga kao i schema sizes - katalog koji preslikava objekte na tablespaceove je po bazi. Provjerite ga prije premještanja velikog objekta, da potvrdite da odredište ima mjesta i stoji na disku koji očekujete; `MOVE` zapiše punu drugu kopiju prije nego odbaci prvu, pa odredištu treba slobodnog mjesta jednako veličini objekta.

#### Zaliha particija {#partition-runway}

`db-partition-runway` (Pohrana) navodi svaku particioniranu tablicu u spojenoj bazi: strategiju i ključ, broj particija, gdje završava zadnji raspon i koliko je to od sada, DEFAULT particiju i retke u njoj, te upravlja li tablicom pg_partman. Postoji zbog incidenta koji se događa u ponoć prvog u mjesecu: tablica particionirana po mjesecu čiju sljedeću particiju nitko nije stvorio. Od trenutka kad zadnji raspon završi, INSERT za "sada" ili pada s *no partition of relation found for row* ili, kad postoji DEFAULT particija, završava u njoj - a DEFAULT particija koja drži retke nekog raspona kasnije sprječava stvaranje particije za taj raspon (*updated partition constraint for default partition would be violated*) dok se reci ručno ne premjeste.

- Tablica je **žuta** kad joj zadnji raspon završava unutar broja dana zadanog provjeri (7 prema zadanom, i Zdravlje koristi 7), a **crvena** kad je završio. Zaliha se mjeri za RANGE ključ čiji je prvi stupac `date`, `timestamp` ili `timestamptz`, i kad je ključ izraz poput `(created_at::date)`; brojčani ključ (raspon id-ova) nema kalendar za mjerenje, i redak to kaže. Zadnji raspon do `MAXVALUE` nikad ne istječe.
- **Reci u DEFAULT particiji** su žuti za RANGE tablicu, bez obzira na broj - jedan redak dovoljan je da spriječi particiju koja bi ga trebala držati - i plavi za LIST tablicu, gdje je particija za sve ostalo uobičajen dizajn. Redak daje broj i udio u tablici.
- **pg_partman**: kad je ekstenzija instalirana u bazi, provjera čita njenu konfiguraciju - premake i interval svake tablice kojom upravlja - i što pokreće njeno održavanje (worker `pg_partman_bgw`, posao `pg_cron`). Tablica kojom upravlja pg_partman žuta je kad joj zaliha padne ispod onoga što njen premake drži unaprijed: dnevna tablica s premake 4 uvijek ima tri do četiri dana, pa upozorenje od 7 dana za nju nikad ne bi utihnulo. Održavanje tada ne radi, ili - uz `infinite_time_partitions = off` - dodaje buduće particije tek kad stignu novi reci.
- Tablica čiji je zadnji raspon završio davno i koja više ne prima nove retke (arhiva) također je crvena: katalog ne razlikuje arhivu od tablice čija umetanja ne uspijevaju, i redak to kaže.

Ništa ne skenira tablicu: granice dolaze iz kataloga, a broj redaka iz statistike planera, pa su to procjene.

#### Zdravlje replikacije {#replication-health}

`db-replication` je pogled primarnog čvora na njegove replike: stanje svakog standbyja i kašnjenje u bajtovima i vremenu na fazama write, flush i replay. `replay` kašnjenje je ono za koliko read replika stvarno zaostaje; rastuće `write`/`flush` kašnjenje upire na mrežu ili disk standbyja, rastuće `replay` kašnjenje na dugotrajan upit na standbyju koji drži replay. Na standbyju bez vlastitih replika provjera to jednostavno kaže.

`db-replica-identity` nalazi tablice na kojima će logička replikacija puknuti: bez primarnog ključa ili s ne-zadanim `REPLICA IDENTITY`. Tablica bez PK-a koja je **u publikaciji** je KRITIČNA - sljedeći `UPDATE` ili `DELETE` pukne na pretplatniku - dok je ona koja nije samo savjetodavna. Napiše `ALTER TABLE` po prioritetu: primarni ključ, inače `REPLICA IDENTITY USING INDEX`, inače `FULL`.

`db-repl-slots` popisuje fizičke i logičke slotove s njihovom `active` zastavicom i količinom WAL-a koji svaki zadržava. **Neaktivan slot puni disk** - zauvijek zadržava WAL dok ga ništa ne troši. Obrišite slotove koje je ostavila povučena replika.

Stvaranje **logičkog** slota čeka da završi svaka transakcija koja ima transaction id u trenutku kad krene - u bilo kojoj bazi klastera, uključujući zaboravljenu pripremljenu transakciju. Dijalog Create slot navodi te transakcije prije nego što ga stvorite, a stvaranje čeka najviše koliko odaberete (30 sekundi ako ne odaberete drugo); kad to istekne, slot nije stvoren i odgovor imenuje što je još čekao. Transakcija koja samo čita ga ne zadržava. Bez `pg_read_all_stats` sesije drugih korisnika vide se samo djelomično, pa dijalog kaže da popis možda nije potpun.

Čip **REPLICA** u zaglavlju na standbyju kaže *sustigao* kad je sve što je primio i reproducirao, bez obzira na to koliko je davno primarni zadnji put potvrdio transakciju - tooltip to vrijeme daje kao koliko je primarni miran. Kašnjenje u sekundama prikazuje se samo dok ima WAL-a za reproducirati; standby bez WAL receivera (oporavak iz arhive) nema broj kašnjenja, jer ne može znati dolazi li još.

`db-logical-replication` je pretplatnička strana: kašnjenje iz `pg_stat_subscription` i sinkronizacijsko stanje po tablici iz `pg_subscription_rel`. Tablica zaglavljena u `COPY` ili `SYNC` obično znači da je početno kopiranje naišlo na grešku - pogledajte log pretplatnika.

#### Nadzornik grešaka {#the-errors-watcher}

`db-errors` uzorkuje `pg_stat_database` kroz prozor od pet sekundi i javlja stopu po sekundi za rollbackove, deadlockove i konflikte, plus `sessions_fatal` / `killed` / `abandoned` na PostgreSQL 14+. Na self-hosted klasteru gdje je `pg_read_file` dopušten čita i zadnjih 256 KB server loga - datoteku `jsonlog` (PostgreSQL 15+) ili `csvlog` kad ih server piše, inače običan log - i čita unose `ERROR` / `FATAL` / `PANIC` kao unose, svaki s pripadnim redovima `DETAIL`, `HINT`, `CONTEXT` i `STATEMENT`. Na managed cloudu preskače log i upućuje kamo ga provider sprema.

Iz loga javlja:

- **Greške po SQLSTATE-u:** svaka klasa s brojem, najzaposlenijom minutom, prvim i zadnjim pojavljivanjem, koliko izvora i glavnim izvorom, te što znači za klase koje se tiču istodobnosti i ponavljanja - `40P01` deadlock, `40001` serialization failure, `23505` unique violation, `55P03` lock not available, `57014` otkazana naredba, `42P10` `ON CONFLICT` bez odgovarajućeg unique indeksa, `53300` previše konekcija, `25P02` naredbe poslane u prekinutu transakciju.
- **Greške po minuti**, po SQLSTATE-u.
- **Oluje ponavljanja:** isti SQLSTATE iz istog izvora **10 ili više puta unutar jedne minute** - nešto ponavlja u petlji. Apsolutan broj, ne udio: petlja ponavljanja je problem i na mirnom i na zaposlenom serveru. Izvor je aplikacija kad je log imenuje (`jsonlog`, `csvlog` ili `%a` u `log_line_prefix`), inače oblik naredbe.
- **Deadlockovi, obje strane:** za svaki deadlock svaki proces s pidom, na što je čekao, tko ga je držao, naredbom koju je izvršavao i koji je PostgreSQL otkazao.

Običan log nosi ono što `log_line_prefix` stavi; provjera čita prefiks sa servera i nalazi `%e` (SQLSTATE), `%a`, `%u` i `%d` gdje god stoje. Bez `%e` SQLSTATE se čita iz engleskog teksta poruke i provjera to kaže: dodaj `%e` (i `%a`) u `log_line_prefix`, ili koristi `jsonlog` ili `csvlog`.

Prvo pročitaj stope: stalna stopa deadlockova je bug redoslijeda zaključavanja u aplikaciji; skok konflikata na replici su upiti koji se sudaraju s replayem. Log su detalji, brojači su alarm.

#### Savjetnici za postavljanje i ključne postavke {#the-setup-and-key-settings-advisors}

`db-setup-advisor` provjerava sve na strani poslužitelja o čemu DBApguin provjere ovise - ekstenzije `pg_stat_statements`, `auto_explain`, `pg_buffercache` i `pg_repack`, GUC-ove poput `track_io_timing`, `compute_query_id`, `log_lock_waits` te dodjele `pg_monitor` / `pg_read_server_files` koje ne-superuseru trebaju - i svaku javlja OK / DJELOMIČNO / NEDOSTAJE s točnim rješenjem. **Pokrenite ga prvog na novom klasteru** i ponovno nakon svake promjene konfiguracije. Na upravljanom oblaku savjet kaže "postavite u parameter grupi" umjesto `ALTER SYSTEM`.

`db-settings` je uređen vodič kroz ~70 parametara bitnih za ugađanje, s pravilom palca i oznakom upozorenja/opasnosti za svaki. To je referenca, ne alarm: oznaka znači "ova vrijednost je neobična", ne "ovo je pokvareno". Ti se parametri uređuju iz prikaza Poslužitelj. Pet kartica govori o tome preživljavaju li potvrđeni podaci pad i daje isti zaključak kao redak TRAJNOST u pregledu zdravlja: `fsync` i `full_page_writes` isključeni su opasnost, `synchronous_commit = off` je info (svjesna zamjena, ne kvar), isključen `data_checksums` (samo za čitanje) je info, a `synchronous_standby_names` pokazuje koliko je od sinkronih standbyja koje traži spojeno - opasnost dok commitovi čekaju na onaj koji nije.

**Izračunato za ovaj poslužitelj** otvara Key settings: petnaest parametara koje alat za ugađanje poput DBtunea mijenja bez restarta (`work_mem`, `maintenance_work_mem`, `effective_cache_size`, `random_page_cost`, `seq_page_cost`, `effective_io_concurrency`, `default_statistics_target`, tri ograničenja paralelnih workera, `max_wal_size`, `min_wal_size`, `checkpoint_completion_target`, `bgwriter_delay`, `bgwriter_lru_maxpages`), svaki s vrijednošću izračunatom za ovaj stroj, mjerenjem ili formulom na kojoj počiva, reloadom ili restartom, i **Primijeni**. RAM i CPU-i dolaze iz `/proc` i cgroup ograničenja kad poslužitelj smije čitati datoteke (superuser ili `pg_read_server_files`); kontejner prikazuje one od hosta, pa upiši koliko poslužitelj smije koristiti i pritisni **Izračunaj ponovno**. Disk se pretpostavlja kao SSD dok ne odabereš HDD. Mjerenje odlučuje o razini gdje god ga ima: `work_mem` je upozorenje samo kad se sortovi stvarno prelijevaju (najmanje 100 privremenih datoteka I 1 GB od resetiranja statistike), `max_wal_size` kad je barem petina checkpointa I njih pet iznuđeno količinom WAL-a (ili iz brzine WAL-a: tri `checkpoint_timeouta` WAL-a), background writer kad su backendovi sami zapisali barem četvrtinu prljavih buffera I njih 10.000. **Primijeni N za pogledati** nakon jedne potvrde izvršava točno navedene naredbe, a zatim ponovno čita poslužitelj. Ovo su polazne vrijednosti za stroj, ne mjerenje tvog opterećenja: promijeni nekoliko, prati iste brojke, zadrži ono što je pomoglo. Prozor DBApguin setup advisor (Pomoć) prikazuje broj s gumbom do ovog odjeljka.

**S poviješću čita što je poslužitelj radio, a ne samo što ima.** Kad su za ovaj poslužitelj radili ASH ili zakazane AWR snimke (iz bilo koje njegove baze), odjeljak počinje s **Na čemu se temelji**: ASH-ovi dani i uzorci, koliko je sesija stvarno bilo aktivno istovremeno (p95 i vrh), udio aktivnog vremena u čekanju na I/O i najprometniji sat; AWR-ovi prozori, preliveno po danu, iznuđeni checkpointi, WAL u najprometnijem prozoru naspram prosjeka, prosječno vrijeme čitanja i najgori pogodak cachea; naredbe koje prelijevaju i koliko zapiše jedan poziv; i što radi sada. Savjet ih koristi gdje su jači od pravila: gornja granica za `work_mem` broji sesije koje je ASH vidio istovremeno (dvostruki p95 ili vrh) umjesto svih veza - ali `work_mem` podiže samo koliko trebaju naredbe koje prelijevaju, dvostruko od onoga što je jedan poziv zapisao na disk; naredba kojoj treba više nego što RAM dopušta navodi se s vlastitim `SET work_mem`. `max_wal_size` prati najprometniji AWR prozor, vrsta diska prati izmjereno vrijeme čitanja (`pg_stat_io` uz `track_io_timing`), `effective_io_concurrency` i `shared_buffers` (restart) postaju upozorenja kad ASH pokaže petinu aktivnog vremena I 600 uzoraka u čekanju na I/O, ili AWR pogodak cachea ispod 99 % na barem milijun čitanja. Bez povijesti to kaže i nudi **Pokreni ASH** i **Pokreni AWR snimke** (svakih 15 minuta); što je više ima, savjet je bliži ovom opterećenju. AWR snimke sada bilježe i `pg_stat_wal` (PostgreSQL 14+) i `pg_stat_io` (16+); starijim snimkama to jednostavno nedostaje.

#### Spremno za PostgreSQL 19? {#ready-for-postgresql-19}

**Podešavanje i dijagnostika -> Spremno za PostgreSQL 19?** pita ovaj server o
svakoj stavci s popisa nekompatibilnosti PostgreSQL-a 19 i odgovara na svaku:

- **BLOCKS** - pg_upgrade odbija ili se 19 ne pokreće: baza u uklonjenom
  kodiranju MULE_INTERNAL, prijelom reda u imenu baze, uloge ili tablespacea,
  indeks s btree_gist inet/cidr klasama operatora, RADIUS u pg_hba.conf,
  escape_string_warning postavljen bilo gdje.
- **CHANGES** - nadogradnja prolazi, ali se nešto ponaša drukčije:
  standard_conforming_strings isključen, MD5 lozinke (na 19 svaka prijava
  upozorava), ručno postavljen max_locks_per_transaction (19 treba dvostruko
  za isti kapacitet), JIT u upotrebi (na 19 po defaultu isključen),
  postgres_fdw (read-only transakcije sada se prenose na udaljenu stranu).
- **INFO** - ekstenzije trećih strana koje trebaju build za 19 prije
  pg_upgrade, i alati za nadzor koji čitaju preimenovane stupce.

Nekoliko stavki treba superusera; bez njega kažu "nije moguće provjeriti"
umjesto da javljaju da je sve u redu. Stavka o indeksima čita bazu na koju si
spojen, pa provjeru pokreni u svakoj bazi. Na serveru koji već radi na 19 isti
redovi pokazuju što se još razlikuje od zadanih vrijednosti verzije 19.

#### Opservabilnost u Oracle stilu

DBApguin donosi skup provjera koje za PostgreSQL rade ono što u Oracleu rade AWR / ASH / SQL Trace:

**ASH i AWR imaju po jednu karticu.** Izbornik *Podešavanje i dijagnostika* otvara *Povijest aktivnih sesija (ASH)* i *AWR snimke i izvještaji*, po jednu karticu za svaki poslužitelj. Zaglavlje kartice pokazuje što prikupljanje radi (uzorkuje ili snima, od kada, koliko često, što se čuva; žuto i *pauzirano* dok je poslužitelj nedostupan) s gumbima **Pokreni**, **Zaustavi** i postavkama iza zupčanika; pogledi su podnavigacija kartice (ASH: Uživo, Izvještaj, Pohranjeni dani; AWR: Uživo, Snimke, Izvještaj, Usporedba, Snimanje, Trend), a unosi pogleda, prozor i baza izvještaja ili dvije snimke za usporedbu (četiri za dva razdoblja), stoje iznad rezultata umjesto u dijalogu. Datum se bira biračem datuma i vremena, čiji kalendar podcrtava dane s uzorcima. Parametri `action` opisani niže su ono što pogledi pokreću; i dalje rade iz skripti i starijih poveznica i otvaraju istu karticu. Kad se spojite na poslužitelj čije uzorkovanje ili raspored radi, njegova se kartica otvori sama.

**db-workload-report (snimanje uživo u AWR stilu)** - pokreće ga AWR "Neko vrijeme", do sat vremena, uz *uzorkuj i sesije svakih 5 s* (rezultat se i dalje prikazuje u AWR kartici pod *Snimanje*): prozor (10 s do 60 min, zadano 5 min) i interval uzorkovanja. Na početku uzme istu snimku koju uzima AWR snimka, između uzorkuje pg_stat_activity svakih nekoliko sekundi, na kraju uzme drugu snimku i čita prozor točno kako AWR izvještaj čita dvije snimke: **Sažetak prozora** (s uzorcima, vrhom aktivnih i svih sesija te koliko je korisnika i aplikacija viđeno), **Nalazi** s dokazima, **Glavni SQL za pogledati** s razlogom za svaki, zatim **Aktivne sesije kroz vrijeme** (grafikon uzoraka po klasi čekanja), **Događaji čekanja**, **Uočeni parovi blokiranja**, **Najdulja viđena otvorena transakcija**, poretci glavnog SQL-a, tablice po DML-u, pozadinska aktivnost i checkpointi te postavke za bolju analizu. Vidi [Kako čitati AWR / workload izvještaj](#reading-an-awr-report). Koristite kad opterećenje traje upravo sada i želite jedan izvještaj o njemu; izvještaj se čuva u [Spremljenim izvještajima](#saved-reports).

**db-awr-snapshot (trajno spremište snimaka + usporedba bilo koje dvije + raspoređivač)**: šest radnji kroz parametar `action`. Tri ručne, tri za automatski raspoređivač.

  Ručne radnje:

  - action=take: snima pg_stat_statements + pg_stat_database + pg_stat_user_tables + pg_stat_user_indexes + postavke autovacuuma + bgwriter / checkpointer u JSON datoteku na `$DBCHECKS_HOME/awr_snapshots/<otisak-poslužitelja>/snap-<id>.json`. Uz snimku se sprema i neobavezni komentar. Većina snimke je kopija pg_stat_statements; retci po tablici i po indeksu koje čitaju savjeti za autovacuum i indekse dodaju oko 320 bajtova po tablici i 160 bajtova po indeksu, pa baza s tisuću tablica svakoj snimci dodaje oko pola megabajta. Otisak poslužitelja je `host__port__baza`, pa se snimke različitih klastera nikad ne miješaju. Skup stupaca prati verziju poslužitelja - `pg_stat_database.sessions_*` na PostgreSQL 14+, `pg_stat_checkpointer` na 17+, `pg_stat_bgwriter` prije toga - pa je snimka valjana od PostgreSQL 13 naviše, a usporedba dviju različitih glavnih verzija jednostavno izostavi stupce koje jedna od njih nema.
  - action=list (**Spremljene snimke**): svaka snimka čuvana za ovaj poslužitelj - dan, vrijeme (UTC), komentar, naredbe, veličina - s **Usporedi s prethodnom** u svakom retku, **Usporedi dvije odabrane**, **Izvezi** (odabrane ili sve, u jedan zip tamo gdje kaže dijalog za spremanje), **Uvezi...**, **Obriši** odabrane i **Obriši sve starije od N dana**. Brisanje prvo pita. Radi samo s datotekama, pa se povijest poslužitelja koji je ugašen i dalje može pregledati, počistiti ili prenijeti drugamo.

    **Izvoz i uvoz.** Izvoz je jedan zip: datoteke kakve jesu i mali manifest koji navodi ASH ili AWR i poslužitelj. Uvoz ih vraća pod taj poslužitelj - spojite se na njega da vidite njihove izvještaje -, prima samo vlastita imena datoteka pohrane (zip s bilo čim drugim odbija se cijeli, ništa se ne zapisuje) i nikad ne prepisuje datoteku koja već postoji: kaže koje je zadržao.
  - action=compare: uspoređuje bilo koje dvije snimke po id-u i prikazuje iste odjeljke u AWR stilu. "Prozor" može biti dug satima ili danima jer su podaci na disku. Snimite jednu prije deploya / batcha / migracije, drugu poslije, pa uspoređujte kad god želite.

    **Preko ponovnog pokretanja.** Čisto ponovno pokretanje zadržava PostgreSQL-ove brojače - zapisuje ih pri gašenju i čita pri pokretanju -, pa usporedba preko njega pokriva cijeli prozor i samo kaže da je poslužitelj ponovno pokrenut. Pad, oporavak pri pokretanju ili failover vraćaju ih na nulu, kao i pg_stat_reset(). Svaka snimka bilježi kad je poslužitelj pokrenut; gdje je brojač u novijoj snimci manji, usporedba ga broji od nule (novija vrijednost je ono što se dogodilo od reseta) umjesto da ispiše negativan broj, počinje nalazom koji to kaže i nakon pada uzima prozor od pokretanja poslužitelja. Trend radi isto za svaki prozor. Oracleov AWR ima isto ograničenje - brojači mu kreću ispočetka s instancom i odbija izvještaj preko ponovnog pokretanja; ono što oba čuvaju je niz snimki.

  - action=compare-periods (pogled **Usporedba**): dva razdoblja jedno uz drugo, kako ih postavlja Oracleov izvještaj *Compare Periods* - razdoblje 1 od snimke A1 do B1, razdoblje 2 od A2 do B2; starija snimka svakog para uzima se kao njegov početak. Svako razdoblje računa se točno kao što compare računa svoj prozor, i preko ponovnog pokretanja ili resetiranja, s istim izborom **Baza**. Otvoren bez izbora, pogled uzima tri najnovije snimke kao dva uzastopna prozora - treću najnoviju do druge najnovije, pa drugu najnoviju do najnovije - i uspoređuje ih; s manje od tri kaže da usporedba treba tri i ništa ne pokreće. Razdoblja se smiju preklapati i doći bilo kojim redom; izvještaj to kaže. Vidi [Kako čitati usporedbu razdoblja](#reading-a-period-comparison).


  Radnje automatskog raspoređivača (pandan Oracleovom auto-AWR-u vođenom `DBMS_SCHEDULER`-om):

  - action=auto-start: pokreće pozadinsku dretvu koja uzima snimku svakih `interval_minutes` (zadano 60, raspon 5-1440) i sama briše snimke starije od `retention_days` (zadano 30, raspon 1-365). Dretva radi na vlastitoj psycopg2 vezi pa nikad ne izgladnjuje korisnički pool. Idempotentna je (ponovno pokretanje ažurira parametre žive dretve). Prva snimka nastaje odmah: `auto-start` čeka njezin rezultat i javi što se stvarno dogodilo, pa raspoređivač koji ne može uzeti ni jednu snimku to kaže umjesto "pokrenuto". Ako ta prva snimka padne, dretva ne ostaje živa. Jednom kad radi, poslužitelj koji prestane odgovarati ne gasi je: nakon tri neuspjele snimke zaredom raspoređivač se pauzira, pokušava se ponovno spojiti (svakih 30 sekundi, zatim rjeđe) i sam nastavlja čim se poslužitelj vrati; živi panel prikazuje pauzu narančasto.
  - action=auto-stop: zaustavlja dretvu raspoređivača. `auto-stop` javlja što je uhvatila dok je radila; ako nikad nije uzela snimku, to kaže jasno umjesto "datoteke ostaju na disku". Spremljena zastavica `enabled` u `auto_config.json` prelazi u false.
  - action=auto-status: prvo javlja presudu - zdravo, pauzirano, pada, prekinuto ili ne radi - pa stanje žive dretve (interval, čuvanje, broj uzetih snimaka, id i vrijeme zadnje snimke, ukupno obrisanih datoteka, zadnja greška ako je bilo) i spremljenu konfiguraciju. Nakon ponovnog pokretanja aplikacije raspoređivač se sam vraća kad sljedeći put otvorite taj poslužitelj; auto-start treba samo ako ste ga zaustavili.

  Aktivni automatski raspoređivač vidi se i u popisu **Background jobs**, isto kao ASH uzorkivač, pa na prvi pogled vidite je li zdrav bez otvaranja provjere.

  **Raspoređivač, uživo** (u `auto-status` i nakon `auto-start`) isti je panel kao kod ASH-a i osvježava se svakih 5 sekundi dok je kartica otvorena: stanje, interval i zadržavanje, koliko dugo radi, snimljene snimke, koliko je davno bila zadnja (žuto kad kasni pola intervala), kad je sljedeća, što je na disku i od kada do kada, te što je zadržavanje obrisalo. Dijalog prikazuje samo polja koja odabrana radnja koristi.

  Napomena o trajnosti: konfiguracija raspoređivača sprema se uz datoteke snimaka i preživljava ponovno pokretanje aplikacije. Raspoređivač koji je radio kad se aplikacija zatvorila ne pokreće se pri startu (tada se ništa ne dešifrira); vraća se onog trena kad ponovno otvorite taj poslužitelj u aplikaciji, s vezom kojom ga otvorite i spremljenim intervalom i zadržavanjem. Do tada `auto-status` pokazuje da je uključen, ali ne radi. `auto-stop` ga gasi za stalno.

**db-ash (uzorkivač povijesti aktivnih sesija)**: neprekidni pozadinski uzorkivač. Četiri radnje.

  - action=status: prvo javlja presudu - radi i zdrav je, pauziran (server je prestao odgovarati; stalno se ponovno spaja i sam nastavlja), pada, ili je zaustavljen - pa kad je pokrenut, koliko je uzoraka i redaka dosad, zadnju grešku ako je bilo, i što je na disku.
  - action=start: pokreće daemon dretvu koja na vlastitoj vezi (odvojenoj od korisničkog poola) ispituje pg_stat_activity svakih **interval** sekundi (zadano 1; 2, 5, 10, 30 ili 60), izbacuje besposlene backendove i svaki neaktivni redak dopisuje u dnevnu JSONL datoteku na `$DBCHECKS_HOME/awr_snapshots/<otisak>/ash/ash-YYYY-MM-DD.jsonl`. Idempotentna je (ponovno pokretanje ne radi ništa). Skup stupaca po uzorku prati verziju poslužitelja - `query_id` je PostgreSQL 14+, pa se na 13 jednostavno ne bilježi, a izvještaj to jednom napomene umjesto da padne.
  - **interval** i **retention days** biraju se pri pokretanju i pamte po poslužitelju za sljedeće pokretanje. Zadržavanje je 30 dana ako ne odabereš drugačije, a 0 čuva sve; dnevne datoteke starije od zadržavanja brišu se kad se uzorkivač pokrene i jednom na sat dok radi - samo cijeli dani, nikad dio dana. Dulji interval piše manje i propušta kratka čekanja; jedna sekunda je ono što radi klasični ASH. Uzorkivač zadržava postavke s kojima je pokrenut: ponovno pokretanje s drugim vrijednostima to kaže i ne mijenja ništa dok se ne zaustavi.
  - action=stop: zaustavlja uzorkivač i javlja što je uhvatio. Ako ništa nije zapisano, to kaže, umjesto "JSONL datoteke ostaju na disku". Nakon tri neuspjela uzorka zaredom uzorkivač se pauzira umjesto da stane: pokušava se ponovno spojiti (svakih 30 sekundi, zatim jednom u minuti) i sam nastavlja kad poslužitelj opet odgovori; `status` i živi panel prikazuju pauzu narančasto uz zadnju grešku. Uzorkivač ostavljen da radi vraća se sam i nakon ponovnog pokretanja aplikacije, kad sljedeći put otvorite taj poslužitelj; `stop` ga gasi.
  - action=report: agregira bilo koji raspon datuma po vlastitom satu poslužitelja u četiri klasična ASH odjeljka: histogram stanja, prvih 20 događaja čekanja, najzastupljenije sesije po broju uzoraka, najzastupljeniji upiti po broju uzoraka, zabilježeni parovi blokiranja. **Izvještaj od** i **Izvještaj do** biraju se aplikacijskim odabirom datuma i vremena: brzi odabiri (sada, danas 00:00, prije 1 h, prije 24 h, jučer), kalendar te polja za sat i minutu; Očisti ostavlja granicu otvorenom.
  - action=data (**Spremljeni dani**): svaka dnevna datoteka čuvana za ovaj poslužitelj s veličinom, današnja označena; **Izvještaj** u retku otvara izvještaj za taj dan, **Izvještaj za odabrane dane** obuhvaća od prvog do zadnjeg odabranog. Izvoz, uvoz i brisanje kao kod spremljenih AWR snimki niže.

  Aktivni uzorkivač vidi se u popisu **Background jobs** s vlastitim stanjem, pa je uzorkivač koji je tiho stao vidljiv bez otvaranja provjere.

  **Uzorkivač, uživo** (u `status` i nakon `start`) osvježava se svakih 5 sekundi dok je kartica otvorena: stanje, interval i zadržavanje, koliko dugo radi, uzorci i zapisani redci, koliko je sesija bilo aktivno u zadnjem uzorku, koliko je davno bio (žuto kad kasni tri intervala), što je na disku i koliko dana obuhvaća, te što je zadržavanje obrisalo.

  Prostor: jedan redak po aktivnoj sesiji i uzorku, oko 450 bajtova s tekstom upita. Izmjereno uz jedan uzorak u sekundi i u prosjeku dvije aktivne sesije: 2,8 MiB na sat, oko 67 MiB dnevno; uz uzorak svakih 5 sekundi petina toga. Disk raste s brojem aktivnih sesija i duljinom njihovih naredbi, ne samo s vremenom, zato je zadržavanje 30 dana ako ne odabereš drugačije. Na poslužitelju je jedan uzorak jedno čitanje pg_stat_activity, oko 0,3 ms. JSONL format je namjerno trivijalan za pregled s `jq`.

**db-trace-session (praćenje u Oracle stilu)**: uzorkivač po backendu s tri načina ciljanja i dva filtra. *Podešavanje i dijagnostika* > *Praćenje sesije* otvara njegovu karticu, po jednu za svaki poslužitelj, s tri pogleda. **Uživo** pokazuje što trace koji radi upravo radi: koga prati, koliko je prozora prošlo, uzorke i dosad uhvaćene izjave, svaku praćenu sesiju s njezinim stanjem, čekanjem i izjavom, te najčešća čekanja; osvježava se svake dvije sekunde, kartica se otvara na njemu dok za ovaj poslužitelj radi trace, a trace pokrenut iz obrasca radi na njemu i završava na Izvještaju. **Novo** je obrazac, iznad rezultata umjesto u dijalogu: koga pratiti (svaku sesiju jednog korisnika, svaku sesiju čiji application_name odgovara uzorku ili jedan backend odabran s popisa trenutno spojenih), prozor (30 s, 1, 2, 5, 10 ili 30 minuta, ili bilo koji broj sekundi od 10 do 1800; po defaultu 2 minute), vrstu naredbi i filtar teksta upita, pa **Pokreni praćenje**. Prije pokretanja praćenje, kao i ASH i AWR, pita za ekstenzije i postavke koje čita kad nedostaje neka koja mu treba, osim ako je to pitanje isključeno; pogled Novo u jednom retku kaže koliko ih je na mjestu, a **Provjeri postavke...** u svakom trenutku otvara isti prozor, čiji Pokreni pokreće praćenje. Dok praćenje traje, zaglavlje kaže koga prati i koliko je prozora prošlo, a **Zaustavi** ga završava ranije: izvještaj pokriva ono što je uhvaćeno do tada. Kartica zatim pokazuje **Izvještaj**, s redom za izvoz (tekst, CSV, JSON, PDF). Izvještaj završava s **Postavke za bolju analizu**, tablicom kojom završava ASH izvještaj (vidi [Kako čitati ASH izvještaj](#reading-an-ash-report)), a na vrhu su postavke koje odlučuju koliko praćenje vidi: `auto_explain` i njegovi `log_min_duration` i `log_analyze` (plan koji se stvarno izvršio ide u log poslužitelja), `log_min_duration_statement`, `log_lock_waits`, `compute_query_id` i `track_activity_query_size`. **Povijest** navodi praćenja spremljena za ovaj poslužitelj, najnovija prva: kada je koje počelo, koga je pratilo, prozor i koliko je stvarno trajalo (zaustavljena su označena), koliko je naredbi uhvatilo i koliko je veliko. **Otvori** prikazuje jedno kao njegov izvještaj, samo za čitanje i izvozivo u ista četiri formata; **Obriši** ga uklanja nakon pitanja. Praćenja se čuvaju po poslužitelju dok ih ne obrišete (najnovijih 500), koliko god drugih provjera u međuvremenu radilo. **Izvezi sve**, ili odabrana, zapisuje ih u jedan zip; **Uvezi...** čita takav zip i nikad ne prepisuje praćenje koje je već ovdje; izvoz podataka aplikacije nosi ih zajedno s AWR snimkama i ASH danima poslužitelja. Zaustavi na isti način ranije završava i *Snimanje* u AWR kartici.

  - target_mode=pid: prati jedan backend po pid-u (odabranom s popisa backenda spojenih upravo sada).
  - target_mode=application_name: prati svaki backend čiji application_name odgovara POSIX regexu. Korisno kad pool mijenja pid-ove, a ime aplikacije ostaje isto.
  - target_mode=user: prati svaki backend određene role. Koristite za pitanje "što ovaj izvještajni korisnik radi upravo sada".
  - duration_seconds: 10 do 1800 sekundi (zadano 120). Uzorkivač radi na 2 Hz cijeli prozor.
  - category_filter: sve / dml / ddl / select / utility. Izbacuje retke čiji tekst upita ne odgovara odabranoj kategoriji naredbi.
  - query_filter: neobavezni regex nad tekstom upita, neosjetljiv na velika i mala slova. Prazno = bez filtra.

  Praćenje usput ispituje je li `auto_explain` učitan i javlja točne SET naredbe kojima se hvatanje planova uključuje za ciljanu sesiju prije nego opterećenje krene. Ispis: slijed upita (prijelazi poredani po vremenu), histogram događaja čekanja za ciljane backendove, razlika po upitu iz pg_stat_statements. Na PostgreSQL 13 stupac `query_id` još ne postoji; praćenje radi bez njega i to napomene.

### 5.2 SQL {#the-sql-editor}

SQL editor s isticanjem PostgreSQL sintakse i vlastitim dovršavanjem. Alatna traka:

- Pokreni (`Ctrl/Cmd + Enter`, `F5`): izvedi sve u editoru
- Naredba (`Ctrl/Cmd + Shift + Enter`): izvedi samo odabrano ili naredbu pod kursorom
- Provjeri (`Ctrl/Cmd + E`): parsiraj i isplaniraj bez izvođenja
- Explain: EXPLAIN / EXPLAIN ANALYZE / BUFFERS
- Izvoz CSV: ponovo pokreće upit na poslužitelju, pa datoteka sadrži sve retke,
  ne samo stranicu koja je na ekranu
- Spremi kao…: zapisuje rezultat koji gledate kao JSON, JSON Lines,
  SQL `INSERT`, Markdown tablicu ili poravnati tekst
- Očisti povijest

Ispod editora je panel s ispisom:
- Rešetka rezultata (vidi niže)
- Povijest izvođenja i statistike (može se skupiti)
- Vrijeme i broj redaka po izvođenju

**Spremljeni upiti.** **Save query** (Ctrl+S) sprema tekst editora pod
imenom; popis **saved** pokraj njega otvara jedan u kartici. **SQL →
Spremljeni upiti…** (ili *manage saved queries…* na kraju tog popisa)
otvara njihov vlastiti dijalog: popis koji se pretražuje po imenu, bilješci
i SQL-u, a za odabrani njegovo ime, bilješka i SQL za uređivanje, **Otvori u
editoru**, **Dupliciraj** i **Obriši**. Promjena se sprema tek nakon što ju
je **Pregledaj i spremi** pokazao jednu uz drugu sa spremljenom - prije
lijevo, poslije desno, redak koji ima samo jedna strana crveno ili zeleno,
promijenjene riječi označene - i pritisneš **Spremi promjene**. Svaka
spremljena promjena čuva verziju prije sebe (zadnjih 25): **Ranije verzije**
uspoređuje bilo koju s trenutnom na isti način i može je vratiti u polja,
za pregled i spremanje. Kartica otvorena iz spremljenog upita to pamti:
**Save query** tamo pokaže što se promijenilo jedno uz drugo i nudi
**Ažuriraj** ili **Spremi kao novi**. Spremljeni upiti nemaju ograničenje;
popis **history** pokraj njih čuva zadnjih 30 pokrenutih upita. Radni
prostor po vezi čuva otvorene kartice između pokretanja.

#### EXPLAIN kao stablo {#explain-as-a-tree}

Explain i Explain Analyze otvaraju se kao **stablo** po defaultu: jedan red po čvoru plana, uvučen pod svojim roditeljem, svaki prikazuje procjenu troška planera, procijenjene retke i (za Analyze) stvarne retke, vlastito vrijeme ovog čvora i njegovo ukupno vrijeme uključujući djecu, te broj petlji. Dvije stvari se automatski ističu umjesto da ih sami tražite u zidu brojki:

- **najskuplji** označava jedini čvor s najviše *vlastitog* vremena - njegovo vlastito vrijeme s oduzetim vremenom svakog djeteta. To je namjerno različito od "čvora s najvećim ukupnim vremenom", što je gotovo uvijek najvanjskiji čvor jednostavno zato što uključuje sve ispod sebe; vlastito vrijeme pronalazi gdje je vrijeme stvarno otišlo.
- **najgora procjena** označava čvor s najvećim razmakom između onoga što je planer očekivao (`Plan Rows`) i onoga što je stvarno izašlo (`Actual Rows`), u bilo kojem smjeru. Velik razmak ovdje, pogotovo uparen sa Seq Scan-om za koji je planer mislio da će vratiti malo, najčešći je razlog zašto plan krene po zlu - obično znači da je statistika tablice zastarjela (`ANALYZE` to popravlja) ili WHERE klauzula ima korelaciju koju `pg_stats` ne vidi.

Značke po stupcu: `cost` (procjena planera, proizvoljne jedinice, usporediva samo unutar istog plana), `rows` (procijenjeno → stvarno, s "Nx" množiteljem kad je promašaj 3x ili više), `self` i `total` vrijeme, i `×N loops` kad je čvor pod Nested Loopom izvršen više puta. Kratka savjetodavna linija pojavljuje se izravno ispod čvora na koji se odnosi - velik Seq Scan, Sort koji se prelio na disk, Hash Join s više od jedne serije, Nested Loop nad velikom vanjskom stranom - iste kategorije koje su već koristile značke sažetka iznad plana, sada očitane iz vlastitih polja čvora umjesto pogođene iz ispisanog teksta, i prikazane tamo gdje je problem stvarno, umjesto u odvojenom popisu ispod.

Seq Scan se procjenjuje po retcima koje **pročita**, a ne po onima koje vrati: uz EXPLAIN ANALYZE to su zadržani retci plus oni koje je filtar uklonio, preko svih radnika, a bez ANALYZE računa se i visok trošak skeniranja. Kad filtar odbaci gotovo sve, redak kaže koliko je redaka pročitano i koliko zadržano - slučaj koji rješava indeks na filtriranom izrazu. Malom tablicom zove se samo skeniranje koje malo čita.

**Tekstualni prikaz** prebacuje na obični, kopirljivi tekstualni blok - iste informacije, u obliku PostgreSQL EXPLAIN-a, za lijepljenje u chat ili tiket. Renderira se iz istog parsiranog plana koji čita stablo (ne drugi poziv na server): EXPLAIN ANALYZE stvarno izvršava upit, pa bi traženje od PostgreSQL-a stabla i teksta zasebno značilo pokretanje upita dvaput - stvaran trošak za spor upit, a za naredbu koja mijenja podatke (umotanu u transakciju koja se ionako vraća) dvostruko pisanje prije rollbacka. Kompromis je da je ovaj tekst blizak, čitljiv rekonstrukt, a ne zajamčeno bajt-za-bajt ono što bi `EXPLAIN (FORMAT TEXT)` ispisao izvorno - brojevi i struktura su točni, stvarni PostgreSQL izlaz je jedino što to jamči, samo je okolna interpunkcija reproducirana ručno.

#### Rešetka rezultata

`SELECT` dohvati prvih 500 redaka i odmah ih pokaže. Doscrollate li do dna,
stigne sljedećih 500, i tako dalje; statusni redak kaže `500 od 48.213 redaka`
pa uvijek znate gdje ste u rezultatu. Broj se izračuna jednom, unaprijed - na
tablici dovoljno velikoj da bi brojanje predugo trajalo to kaže umjesto broja,
a straničenje i dalje radi.

Retci se između stranica čuvaju na poslužitelju, na kursoru koji se zatvori čim
pokrenete nešto drugo, promijenite karticu ili ga ostavite netaknutog nekoliko
minuta. Editor iza vaših leđa ne drži otvorenu transakciju.

Nakon 5.000 redaka u rešetki automatsko učitavanje stane i preuzme ga gumb
**Učitaj još 500**, pa nemaran scroll nad ogromnim rezultatom ne može zatrpati
prozor. Za cijeli rezultat `Izvoz CSV` streama svaki redak na poslužitelju,
bez rešetke.

Na samoj rešetki nema sortiranja ni filtera. U njoj je uvijek samo dio
rezultata, pa bi sortiranje poredalo 500 od N i izgledalo kao da je poredalo
sve. Sortirajte u upitu, s `ORDER BY`.

#### Polje search_path

Uz izbornik modova, `search_path` određuje gdje se traže nekvalificirana imena,
samo za to izvođenje. Ostaviš li ga praznim, vrijedi kontekst sesije iz
zaglavlja.

Kvačica na kraju polja spušta sheme spojene baze - samo one u koje ova sesija
doista smije ući, jer shema na kojoj nemaš `USAGE` nije odgovor. Tvoje idu
prve, `pg_catalog` i `information_schema` na kraj, pod svojim naslovom. Klik na
ime dodaje shemu, ponovni klik je miče, a sve što je već u putu ima kvačicu - 
pa je cijelo stanje polja vidljivo na jednom mjestu. Izbornik ostaje otvoren
dok biraš, a polje se i dalje može tipkati za ono što popis ne pokriva.

To je važnije nego što izgleda - kriva shema u search_pathu nije greška. PostgreSQL prihvaća put koji
imenuje sheme kojih nema i onda ne nalazi tvoju tablicu, što izgleda kao „nema
više tablice" umjesto „put je kriv".

#### Što "Pokreni" zapravo pokreće

Editor nikad ne nagađa granicu naredbe koju ne vidite. Dva gumba, dva opsega:

| Gumb | Prečac | Pokreće |
|---|---|---|
| `▶ Pokreni` | `Ctrl/Cmd + Enter` ili `F5` | svaku naredbu u kartici, redom |
| `▶| Naredba` | `Ctrl/Cmd + Shift + Enter` | odabrani tekst, a bez odabira - naredbu u kojoj je kursor |

"Naredba u kojoj je kursor" ide od prethodnog `;` do sljedećeg. Tijela u dolar
navodnicima (`$$ ... $$`), tekst u navodnicima i komentari izuzeti su iz te
pretrage, pa funkcija ili procedura čije tijelo sadrži `BEGIN ... ; ... END`
broji kao jedna naredba, a ne kao četiri.

`F5` je u pregledniku ponovno učitavanje stranice; DBApguin ga presreće. U SQL
prikazu pokreće upit, drugdje osvježava trenutni prikaz. `Ctrl/Cmd + R` ostaje
osvježavanje prikaza svugdje, pa i u SQL prikazu.

#### Transakcije i autocommit {#transactions-and-autocommit}

Padajući izbornik uz gumb Pokreni odlučuje što se događa oko naredbi u jednom izvođenju:

- **autocommit** (zadano) - svaka naredba potvrđuje se sama, kao u `psql`-u.
  Ono što ste pokrenuli trajno je čim se vrati, a pad na pola puta ostavlja
  ranije naredbe primijenjenima.
- **transakcija** - cijelo izvođenje omotano je u jedan `BEGIN ... COMMIT`. Ako
  ijedna naredba padne, sve iz tog izvođenja se poništava i baza je točno kakva
  je bila. Ovo je način za migraciju.
- **probno izvođenje** - izvođenje je omotano u `BEGIN` i uvijek završava s
  `ROLLBACK`. Dobijete broj redaka, greške i vremena bez ijednog traga. Ništa
  što ovdje napravite ne ostaje, ni ono što je uspjelo.

**read-only** odlučuje izvode li se naredbe ovog taba u read-only sesiji. Po
defaultu je iskljucen, vrijedi za taj tab, i aplikacija ne pogađa: kad je
uključen, PostgreSQL odbija sve što piše, što je način da migracijsku skriptu
stavite pred produkcijski server i vidite gdje bi stala.

Prije se to izvodilo iz samog SQL-a, a taj se zaključak ne može ispravno
napraviti. Prva ključna riječ naredbe ne govori piše li ona: `IMPORT FOREIGN
SCHEMA` ne izgleda ni kao jedno ni kao drugo, a `SELECT cron.schedule(...)` -
upravo onako kako pg_cron u svojoj dokumentaciji kaže da se job zakazuje -
upisuje red a izgleda točno kao upit. Oboje se vraćalo odbijeno uz *"cannot
execute ... in a read-only transaction"*, na primariju, zbog ograničenja koje
je sama aplikacija nametnula, dok je ista naredba kroz `psql` prolazila bez
problema.

Neke naredbe uopće ne mogu unutar transakcije: `CREATE INDEX CONCURRENTLY`,
`REINDEX CONCURRENTLY`, `VACUUM`, `CREATE DATABASE`. Njih pokrenite u
**autocommit** modu. U druga dva poslužitelj odgovara *"cannot run inside a
transaction block"*, a editor kaže koji izbornik promijeniti.

- **pokreni kao skriptu** - kao autocommit, samo što naredba koja padne ne
  prekida izvođenje. Svaka se pokuša, svaki neuspjeh se ispiše uz svoj redni
  broj i poruku, a one koje su prošle ostaju potvrđene. To je mod za datoteku
  DDL-a od koje je dio već primijenjen: želiš da ostatak uđe i popis onoga
  što nije. Rezultat je samo sirovi ispis, kakav daje `psql -f` - oznaka ili
  redovi svake naredbe, njezine obavijesti, njezina greška - s **Copy** uz
  njega; tablice nema.
- **drži otvoreno (sam potvrdiš)** - transakcija ostaje otvorena **između**
  pokretanja i završava kad pritisneš Commit ili Rollback, koji se pojave uz
  izbornik dok je otvorena. Pokreni nešto, pogledaj što je napravilo, pa
  odluči.

  Otvorena transakcija drži svoja zaključavanja dok je netko ne zatvori, pa je
  stanje uvijek na ekranu - koliko je naredbi u njoj i koliko je dugo otvorena
 - a ona koju nitko nije dirao deset minuta poništi se umjesto tebe. Naredba
  koja padne **ne** prekida je: PostgreSQL transakciju označi abortiranom i
  ništa drugo neće proći dok ne poništiš, što aplikacija i kaže umjesto da ti
  odbaci posao.

#### Koliko naredba smije trajati

Uz izbornik modova stoji timeout: kvačica i broj sekundi. Svaki novi SQL tab
kreće s **uključenih 180 sekundi**, a postavka pripada tabu - jedan tab drži
migraciju koju si spreman čekati sat vremena, drugi upite koje još pogađaš.

Kad se aktivira, naredba se otkazuje **na poslužitelju**, ne samo u aplikaciji.
Ako isključiš kvačicu, naredba radi dok ne završi ili je ne prekineš; app pita
jednom, jer naredba bez ograničenja drži zaključavanja koliko god traje.

`ADD CONSTRAINT`, `CREATE INDEX` i veliki `UPDATE` na stvarnoj tablici svi
traju dulje od nekadašnjih fiksnih trideset sekundi, koje ovo zamjenjuje.

#### DDL, ekskluzivni lockovi i red čekanja na lock {#ddl-exclusive-locks-and-the-lock-queue}

`DROP`, `ALTER TABLE`, `TRUNCATE`, `REINDEX` (bez `CONCURRENTLY`), `CLUSTER` i
`VACUUM FULL` uzimaju **AccessExclusiveLock** na tablici. Bitno je što se
događa *prije* nego naredba dobije taj lock:

> Zahtjev za ekskluzivnim lockom ne preskače red. Dok čeka, PostgreSQL stavlja
> svaki novi zahtjev za lockom na toj tablici iza njega - uključujući običan
> `SELECT`. Zato `DROP` koji samo *čeka* već blokira nova čitanja i pisanja na
> tim tablicama, prije nego išta promijeni.

Tako `DROP SCHEMA` iza jednog sporog izvještaja postane ispad na razini cijele
baze. Editor takve naredbe tretira drukčije od običnog upita:

- **Provjera prije izvršavanja.** Kad naredba uzima ekskluzivni lock, a na
  cilju već nešto radi, dijalog izlista te sesije - tko, odakle, koliko dugo i
  je li koja već u redu čekanja - i objasni čekanje prije nego se odlučiš.
  Sesija u prvoj sekundi naredbe, u transakciji koja nije starija od toga, nije
  na popisu: na prometnoj tablici takva uvijek postoji (kratke naredbe
  aplikacije, nadzorna ploča ove aplikacije dok čita veličine tablica) i
  nestane prije nego bi je DDL primijetio. Transakcija ostavljena otvorenom - u
  drugom programu ili u drugom tabu editora ove aplikacije - na popisu je ma
  koliko bila mlada, kao i naredba koja traje dulje.
- **Lock svake naredbe, prije nego se izvrši.** Potvrda pokretanja navodi svaku DDL naredbu s lockom koji uzima i na čemu, i kaže kad ponovno piše tablicu ili je čita cijelu - pitajući server gdje tekst ne može reći: `ADD COLUMN ... DEFAULT clock_timestamp()` (svaki volatilni default - `random()`, `gen_random_uuid()`, `serial`) prepisuje svaki red, dok konstantan default ili `now()` ne (PostgreSQL 11+); proširenje `varchar(n)` na dulji `varchar` ili na `text`, ili `numeric` na više znamenki uz isti broj decimala, promjena je samo u katalogu, redovi i indeksi ostaju; `ADD FOREIGN KEY` bez `NOT VALID` čita cijelu child tablicu dok pisanja u obje tablice čekaju, pa ga dodaj s `NOT VALID` i poslije `VALIDATE CONSTRAINT` - koji na stranom ključu uzima i ROW SHARE na tablici na koju ključ pokazuje, pa čeka samo DDL koji tu tablicu zaključava isključivo; brisanje stranog ključa i tu tablicu zaključava s ACCESS EXCLUSIVE, pa čeka svako njeno čitanje - isto čini brisanje stupca od kojeg je strani ključ sastavljen, brisanje tablice koja pokazuje na drugu (s `CASCADE` i tablica koje pokazuju na nju) i promjena tipa stupca ključa na bilo kojoj strani, a potvrda imenuje tu drugu tablicu; `VACUUM FULL`, `CLUSTER` i `REPACK` drže ACCESS EXCLUSIVE i ponovno pišu tablicu; `REINDEX` bez `CONCURRENTLY` zaustavlja gotovo svaki upit na tablici. Usporedba -> Primijeni čita naredbe na isti način.
- **Zadano ograničenje čekanja na lock.** Ekskluzivni DDL pokrenut iz editora
  dobije **3-sekundni** `lock_timeout`, bez obzira na kvačicu za timeout
  naredbe. Ako ne dobije lock za tri sekunde, odustaje i ništa nije čekalo iza
  njega dulje od toga. To je zaštita, ne timeout naredbe: ograničava samo
  čekanje *na lock*, ne posao nakon toga.
- **Isključivanje zaštite.** Provjera ima kvačicu *"čekaj koliko treba"*.
  Označi je i DDL čeka lock neograničeno - učini to samo ako možeš zaustaviti
  promet, jer dok DDL ne uđe, promet na tim tablicama je također zaustavljen.

Ako zaštita okine, poruka to kaže - koje ograničenje, i da je riječ o
editorovoj DDL zaštiti, a ne o `statement_timeout` ili `lock_timeout` sa
strane poslužitelja - i zadrži zadnju sliku reda čekanja na ekranu.

#### Dok radi

Statusna linija drži sat, pa naredba koja traje četiri minute nikad ne izgleda
kao aplikacija koja je prestala odgovarati. Čim naredba dođe na poslužitelj,
app s druge konekcije pita i što radi:

- **čeka na zaključavanje** - prikazuje se ispred svega ostalog. Naredba koja
  čeka nije spora nego zaglavljena, a to dvoje traži različitu reakciju. Za
  čekanje na lock editor prikaže cijeli red: koliko sesija drži lock ispred
  tebe, koliko dugo čekaš i - dio koji nigdje drugdje nema - koliko se novih
  upita već nakupilo iza tebe. Gumbi tamo otvaraju prikaz blokiranja ili
  prekidaju držatelje.
- **napredak**, za operacije za koje ga PostgreSQL javlja - `CREATE INDEX`,
  `VACUUM`, `ANALYZE`, `CLUSTER`, `COPY` - s fazom u kojoj je i trakom.

Traka se pojavljuje samo kad poslužitelj stvarno javlja postotak. Izmišljen bi
bio gori od nikakvog.

**Zaustavi** stoji uz Pokreni dok se nešto izvodi i zaustavlja to na
poslužitelju, jer se odande jedino i može: zahtjev koji je naredbu pokrenuo
blokiran je na istoj konekciji i čeka isti odgovor. Esc radi isto. Naredba se
vraća kao prekinuta na zahtjev (57014), a sesija preživi - uključujući
transakciju koju držiš otvorenom.

Prekid je molba, ne naredba, i backend duboko u sistemskom pozivu neće ga
primijetiti. Ako ne uhvati u par sekundi, gumb nudi **Prisilno**, što umjesto
toga zatvara konekciju. To uvijek uspije i stoji te sesije: otvorena
transakcija poništava se s njom, pa prije toga pita.

Izvođenje pripada tabu s kojeg je pokrenuto. Odeš li s tog taba, njegov × za
zatvaranje postaje prsten koji se vrti - kvačica kad naredba završi, križić
kad završi greškom. Vratiš li se na tab, × se vraća, jer je do tada rezultat
već pred tobom. Tabovi na kojima se ništa ne izvodi ostaju netaknuti.

#### Panel s izlazom

Ispod rezultata aplikacija ispisuje što se stvarno dogodilo - isto što ispisuje
`psql`, i jedno što je drugdje lako promašiti.

Za svaku naredbu u pokretanju: oznaka naredbe (`UPDATE 3`, `CREATE INDEX`,
`SELECT 1`), koliko je redaka dirala, koliko je trajala i sama naredba. Skripta
od pet naredbi tako kaže koja je bila spora, umjesto jednog zbroja za svih pet.

Ispod toga **poruke poslužitelja**: sve što je `RAISE NOTICE` proizveo u
funkciji ili `DO` bloku, i tijek `VACUUM VERBOSE`. Ako PL/pgSQL debugirate
podizanjem obavijesti, ovdje se pojavljuju.

Zaglavlje nosi vrijeme početka i ukupno trajanje, a **Copy** stavlja cijeli
panel na međuspremnik, za tiket ili mail.

Kad naredba padne, panel je korisnija polovica odgovora: naredbe koje su
navedene su one koje su već prošle. U autocommitu su potvrđene i neuspjeh ih
ne poništava; u transakcijskom modu cijelo pokretanje je poništeno i nijedna
ne stoji.

#### Spremanje rezultata u drugom formatu

`Izvoz CSV` traži od poslužitelja da ponovo pokrene upit i pošalje sve retke,
pa je to izbor za cijelu tablicu. Izbornik **spremi kao…** pokraj njega
zapisuje rezultat koji je već na ekranu, dakle onaj koji čitate:

| Format | Čemu služi |
|---|---|
| JSON | programeru koji lijepi podatke u test |
| JSON Lines | jedan objekt po retku - `jq`, log cjevovod, masovni uvoz |
| SQL `INSERT` | učitavanje redaka u drugu bazu; koristi se stvarno ime tablice kad rezultat dolazi iz jedne |
| Markdown tablica | tiket, merge request, wiki stranica |
| Poravnati tekst | mail ili terminal, čitljivo bez ikakvog alata |

Svaki format ide kroz isti dijalog za datoteke kao i ostali izvozi, pa birate
gdje ide, a aplikacija poslije javi punu putanju.

### 5.3 Shema

Detaljan prikaz jednog objekta. Kartice:
- Stupci (sa zadanom vrijednošću, tipom, dopuštenim NULL-om, komentarom)
- Indeksi
- Ograničenja
- Okidači
- Ovlasti (uredive)
- Veze stranim ključem (dolazne / odlazne)
- Statistika
- DDL - za tablicu ili particioniranu tablicu ovo je uređivač stupaca (niže);
  za sve ostalo je formatirana CREATE naredba
- Podaci - redovi, uredivi na mjestu u tablici s primarnim ključem. Dva
  načina: **builder** slaže upit od limita, stupaca, filtera i ORDER BY i
  prikazuje SQL koji će izvršiti; **raw SQL** ti daje da cijeli upit upišeš
  sam. U načinu raw SQL polje SQL je uredivo, a izbor stupaca, filteri i
  ORDER BY se sklanjaju; polje počinje od onoga što je builder imao. Upit se
  izvršava samo za čitanje i mora biti jedna naredba SELECT, WITH, TABLE ili
  VALUES; Limit i dalje ograničava broj dohvaćenih redova, a Ctrl+Enter ga
  pokreće. Tablica rezultata ostaje uređiva samo kad upit vraća svaki stupac
  ove tablice pod njegovim imenom; inače je rezultat samo za čitanje.
  Povratak na builder zatiče ga kako si ga ostavio.

Ovaj prikaz vodi pregled sheme u lijevoj traci. Klik na objekt tamo učitava ga ovdje.

#### 5.3.1 Uređivanje stupaca {#editing-columns}

Odaberite tablicu i otvorite karticu DDL. Umjesto CREATE TABLE teksta koji se
samo čita, dobivate jedan redak po stupcu, svaki uredivo na mjestu:

| Polje | Što radi gumb pokraj njega |
|---|---|
| Naziv | `RENAME COLUMN` |
| Tip | `SET DATA TYPE`, uz neobavezni `USING` izraz za pretvorbe koje PostgreSQL neće napraviti sam |
| NOT NULL | `SET NOT NULL` ili `DROP NOT NULL`, ovisno o kvačici |
| Zadana vrijednost | `SET DEFAULT` iz polja, ili `DROP DEFAULT` |
| Radnje | `DROP COLUMN`, s kvačicom za CASCADE pokraj |

Stupac Dodatno prikazuje ono što katalog zna, a ovdje se ne uređuje:
PRIMARY KEY, `IDENTITY (ALWAYS)` ili `IDENTITY (BY DEFAULT)`, te
`GENERATED STORED` ili `GENERATED VIRTUAL`.

**Dodaj stupac** na dnu traži naziv, tip, neobavezni NOT NULL i jedno od:
- *bez zadane vrijednosti*
- *DEFAULT* - izraz
- *GENERATED* - izraz plus vrsta pohrane. STORED materijalizira vrijednost pri
  pisanju i treba PostgreSQL 12 ili noviji; VIRTUAL je računa pri čitanju i
  treba 18 ili noviji. Aplikacija uvijek ispisuje ključnu riječ jer je
  PostgreSQL 18 promijenio što znači goli `GENERATED ALWAYS AS (...)` - prije
  je to bila greška, sada je VIRTUAL - pa bi ista naredba inače radila
  različite stvari na različitim poslužiteljima.

Svaka radnja prvo pokaže točnu naredbu, s ključem koji uzima i time što taj
ključ blokira, i ne izvodi ništa dok ne potvrdite.

Četiri od njih - `SET DATA TYPE`, `SET NOT NULL` i `ADD COLUMN` sa zadanom ili
GENERATED vrijednošću - rade posao razmjeran veličini tablice: prva prepisuje
svaki redak, druga čita svaki redak da dokaže ograničenje. One se izvode kao
pozadinski poslovi, pa se ne prekidaju na pola na velikoj tablici i nastavljaju
ako zatvorite panel; pratite ih u **Baza → Pozadinski zadaci**. Ostale su
katalogške izmjene koje završe odmah i idu kao obični zahtjev.

Pogledi, materijalizirani pogledi i strane tablice ovdje se ne uređuju - njihovi
stupci dolaze iz definicije, pa kartica prikazuje samo strukturu.

#### 5.3.2 Izbornici objekata: što se od koje vrste može tražiti {#object-menus}

Desni klik na objekt u pregledniku sheme. Ispod navigacijskih i izvoznih
stavki svaka vrsta nosi naredbe koje PostgreSQL za nju stvarno ima - ne
zajednički podskup. Dva pravila drže izbornik iskrenim na svakoj verziji
koju aplikacija podržava:

- **Ništa se ne skriva.** Stavka za koju je ovaj poslužitelj prestar
  prikazuje se zasivljeno, a tooltip imenuje verziju koja joj treba
  (`treba PostgreSQL 12 ili noviji`). Kraći izbornik na starom poslužitelju
  ostavio bi vas u nedoumici postoji li opcija uopće ili ju je aplikacija
  zaboravila.
- **Ograničenje nije samo u izborniku.** Svako versijsko pravilo i pravilo
  vrste objekta koje izbornik crta ponovno se provodi u endpointu iza njega,
  pa ga ponovljeni zahtjev ne može zaobići.

**Indeks**

| Stavka | Napomene |
|---|---|
| REINDEX | Isključivi ključ, brzo |
| REINDEX CONCURRENTLY | PostgreSQL 12+. Bez ključa na pisanje; prekinuto izvođenje ostavlja NEVAŽEĆI indeks |
| REINDEX s opcijama… | CONCURRENTLY, VERBOSE (9.5+) i TABLESPACE (14+), svako ponuđeno samo gdje postoji |
| Preimenuj / Premjesti u tablespace | |
| Parametri pohrane… | Samo oni koje ova pristupna metoda prima: `fillfactor`, `deduplicate_items` (btree, 13+), `fastupdate` i `gin_pending_list_limit` (gin), `pages_per_range` i `autosummarize` (brin), `buffering` (gist) |
| Postavi statistiku na izraz… | PostgreSQL 11+, i samo za izraz-stupac - običan stupac odbija se na svakoj verziji |
| Obriši | CONCURRENTLY i CASCADE kao kvačice, uz prikaz naredbe prije izvođenja |

Indeks koji je NEVAŽEĆI dobiva dodatnu stavku na vrhu koja to kaže, s
popravkom u jednom kliku. Indeks koji postoji samo da provodi ograničenje
kaže koje, i odbija se obrisati zasebno.

**Tablica, particionirana tablica, materijalizirani pogled**

Podizbornik **Održavanje** s ANALYZE, VACUUM, VACUUM ANALYZE, VACUUM FULL,
REINDEX TABLE, REINDEX TABLE CONCURRENTLY (12+) i CLUSTER. Svako se izvodi
kao pozadinski posao: zatvorite ploču, vratite se, nađite ga u
**Baza → Pozadinski poslovi**.

*VACUUM s opcijama…* je puni popis opcija, svaki redak zasivljen s verzijom
koja mu treba ako je poslužitelj stariji: `DISABLE_PAGE_SKIPPING` (9.6+),
`SKIP_LOCKED`, `INDEX_CLEANUP` i `TRUNCATE` (12+), `PARALLEL` (13+),
`PROCESS_TOAST` (14+), `PROCESS_MAIN`, `SKIP_DATABASE_STATS` i
`BUFFER_USAGE_LIMIT` (16+). `PROCESS_TOAST`, `PROCESS_MAIN` i `TRUNCATE` u
PostgreSQL-u su uključeni, pa su prikazani označeni - odznačite da ih
isključite.

*CLUSTER…* nabraja indekse tablice i označava onaj po kojem je trenutno
složena, jer `CLUSTER` bez `USING` radi samo kad takav postoji.

**TRUNCATE…** je zasebna stavka s ONLY, RESTART IDENTITY i CASCADE, i
prikazuje naredbu prije izvođenja. I ona ide kroz pozadinski posao: CASCADE
može zahvatiti stablo tablica koje se pozivaju na ovu, a ključ koji joj treba
može prvo čekati u redu iza drugih sesija.

Pod **Upravljanje**: *Parametri pohrane…* (`fillfactor`, `parallel_workers`
(10+), `toast_tuple_target` (11+), `vacuum_index_cleanup` i
`vacuum_truncate` (12+), blok `autovacuum_*`, plus par za samo-umetanje od
13+), *Premjesti u tablespace…*, a za običnu tablicu *Zapisivana /
nezapisivana…* (9.5+).

**Materijalizirani pogled** dodatno ima REFRESH, REFRESH CONCURRENTLY i
REFRESH … WITH NO DATA. Zadnje prazni pogled i čini da svaki upit nad njim
javlja grešku do sljedećeg punog osvježavanja, pa pita zasebno.

**Pogled** - *Opcije pogleda…*: `check_option` (9.4+), `security_barrier`
(9.2+) i `security_invoker` (15+). Zadnja odlučuje provjerava li se upit kroz
pogled prema vašim ovlastima ili ovlastima vlasnika pogleda.

**Sekvenca** - *Svojstva sekvence…* je cijeli blok parametara
`ALTER SEQUENCE` u jednoj formi: tip `AS` (10+), INCREMENT BY, MINVALUE,
MAXVALUE, START WITH, CACHE, CYCLE i OWNED BY. U naredbu ulaze samo polja
koja ste promijenili, a pregled pokazuje točno što će se izvesti.
*Ponovno pokreni…* postavlja sljedeću vrijednost koju sekvenca izdaje.

**Okidač** - podizbornik **Okidanje** sa sva četiri stanja: ENABLE, DISABLE,
ENABLE REPLICA (okida se samo dok je `session_replication_role` jednak
`replica`) i ENABLE ALWAYS (okida se bez obzira na to, pa ga primjena
replikacije ili vraćanje iz kopije ne mogu utišati). Plus *Preimenuj…*.

**Funkcija / procedura** - blok Svojstva u uređivaču sada nosi i COST, ROWS,
PARALLEL (9.6+, ispod toga zasivljeno) i LEAKPROOF, uz kontrole za
preimenovanje / shemu / vlasnika / search_path koje su već postojale.

**Shema** - podizbornik **Održavanje** koji izvodi ANALYZE, VACUUM, VACUUM
ANALYZE ili REINDEX nad svakom običnom tablicom i materijaliziranim pogledom
u shemi, jedno za drugim, nastavljajući i preko tablice koju ne može
zaključati. Puna ploča, s dosegom na razini klastera i svim opcijama, i dalje
je jedna stavka niže.

#### Kopija definicija prije promjene {#copy-before-the-change}

Premještanje objekta u drugu shemu, brisanje objekata, promjena vlasnika ili
tablespacea najprije zapisuje definicije objekata u mapu s podacima, kao SQL
koji ih vraća: za tablicu stupce, ograničenja, indekse, okidače, politike,
vlasnika, prava, komentare i tablespace u kojem su ona i njezini indeksi; za
rutinu njezinu definiciju. Pozadinski poslovi navode kopiju ("Before: ..."),
obavijest nakon promjene je otvara, a odande ide u SQL uređivač ili u `.sql`
datoteku. Kopija dolazi iz `pg_dump --schema-only` kad je dostupan pg_dump
koji odgovara poslužitelju, inače iz kataloga. Kad se kopija ne može
napraviti, promjena se odbija s razlogom; možeš nastaviti bez nje, a dnevnik
radnji bilježi da jesi. Neuspjelu promjenu PostgreSQL sam vraća; ovo je za
promjenu koja je uspjela, a nije trebala.

**Brisanje i sve što ono odnosi.** Desni klik **Drop** na tablici, pogledu,
sekvenci, indeksu, rutini, tipu ili shemi pravi kopiju jednako kao brisanje
više objekata - prije je nije pravio. S **CASCADE** kopija sadrži sve što
brisanje uklanja, ne samo ono što je imenovano. Koji su to objekti, kaže sam
PostgreSQL: brisanje se isproba u transakciji koja se vrati, a katalog kaže
što je nestalo. Izmjereno, `DROP TABLE ... CASCADE` odnosi poglede nad
tablicom u drugim shemama, pogled nad jednim od njih, materijalizirane
poglede, funkciju kojoj je argument tip retka tablice i strane ključeve
drugih tablica; `DROP TYPE ... CASCADE` vadi stupce tog tipa iz drugih
tablica, zajedno s vrijednostima; obrisana funkcija okidača odnosi svoje
okidače, obrisana sekvenca defaulte koji je pozivaju. Dijalog ih navodi kad to
zatražiš (**Izračunaj što drop uzima**), pročitane iz grafa ovisnosti: za
popis se ništa ne isprobava i ništa ne zaključava, i za shemu sa stotinama
tablica. Kopija ih imenuje na vrhu, točno kako ih je proba našla. Kad se
proba ne može izvesti - tablica prezauzeta da se zaključa u tri sekunde ili
event triggeri koji bi se na njoj okinuli - i popis kopije čita se iz grafa
ovisnosti, a kopija kaže da je procjena. Ono što kopija ne može zapisati
navedeno je pod *NOT in this copy*.

**Što kopirati prije.** Dijalog za brisanje, i dijalog koji briše više
odabranih objekata, otvaraju se bez ijednog upita poslužitelju. Ispod
naredbe i njezinih opcija (**CASCADE**, **IF EXISTS**, SQL) pitaju, ovim
redom:

1. **Spremiti kopiju prije dropa?** Prvi izbor je **Bez kopije - izvrši samo
   DROP**: ništa se ne sprema i ništa se prije ne računa, izvrši se samo
   naredba DROP (za shemu s **CASCADE**, `DROP SCHEMA ... CASCADE` na toj
   shemi i ništa drugo). Ne može se poništiti, osim iz sigurnosne kopije.
   **Prvo spremi kopiju** otvara sljedeće pitanje.
2. **Što spremiti:** **Kopiraj samo definicije** (definicije svega što
   brisanje odnosi; tablica se vraća prazna) ili **Kopiraj definicije i
   redove** (i redovi tablica koje odlaze zapisuju se u mapu s podacima).
3. **Izračunaj što drop uzima** - tek kad to klikneš: objekti koje
   **CASCADE** uklanja i, za tablice koje odlaze, koliko ih je, otprilike
   koliko redova i koliko podataka. Kopija to treba prije nego što se
   **Ukloni** omogući; bez kopije to također možeš zatražiti, ali ne moraš.

Veličine su procjena koju poslužitelj vodi (redovi i stranice iz njegove
statistike), pročitana bez otvaranja ijedne tablice; tablica koja nikad nije
vacuumirana ni analizirana još nema veličinu, i dijalog to kaže. Kopija
najprije isproba brisanje da točno utvrdi što odlazi, a to na trenutak drži
zaključavanja brisanja (čeka ih najviše tri sekunde). Dijalog pamti tvoj
zadnji izbor dok se aplikacija ne zatvori (nigdje se ne sprema): sljedeće
brisanje počinje od njega, samo što redovi iznad 50 MB prelaze u samo
definicije čim analiza to pokaže, i dijalog kaže zašto. Bez kopije, kad
brisanje može odnijeti podatke ili druge objekte (tablica, shema, bilo što s
**CASCADE**, osim ako je analiza pokazala da ništa ne odlazi), upisuješ ime
objekta (za više objekata, koliko ih je) prije nego što se **Ukloni**
omogući. Vrsta za koju se kopija ne može napisati (ekstenzija, okidač,
uloga ...) to kaže i nudi samo analizu.

**Dok radi.** Brisanje je pozadinski posao, a dijalog ga prati korak po
korak: utvrđivanje što točno brisanje uklanja, kopiranje definicija,
kopiranje redova (toliko od toliko tablica, koliko je zapisano), uklanjanje -
s vremenom koje je dosad prošlo. Dok sam DROP ne počne, **Odustani** glasi
**Zaustavi**: staje na sljedećoj sigurnoj točki, ništa se ne uklanja, a
kopija napravljena dotad briše se zajedno s redovima. Kad DROP krene, više
se ne može zaustaviti, i gumb to kaže. Zatvaranje dijaloga ne zaustavlja
posao; ostaje u Pozadinskim poslovima. Premještanje i kloniranje iz istog
dijaloga pokazuju isto stanje rada.

**Redovi.** Definicije nisu podaci. **Kopiraj definicije i redove**
zapisuje redove svake tablice koju brisanje odnosi u mapu s podacima uz
kopiju, kao i vrijednosti stupca koji cascade vadi iz tablice koja ostaje
(vraćaju se po primarnom ključu; tablica bez njega ne može ih dobiti natrag,
i kopija to kaže). Za poništavanje brisanja: otvori kopiju, pokreni je u
SQL uređivaču, zatim **Vrati redove** - jedna transakcija, roditelji prije
djece, u tablice koje moraju ponovno postojati i biti prazne, pa se ništa ne
učita dvaput. Redovi se vraćaju tamo gdje je kopija uzeta i nigdje drugdje:
kopija pamti poslužitelj (adresu, libpq servis ili SSH tunel, i vlastiti
identifikator sustava poslužitelja) i bazu, **Vrati redove** koristi otvorenu
karticu ili spremljenu vezu koja dolazi do tog poslužitelja - nikad karticu
sprijeda kad je ona negdje drugdje - a poslužitelj se pita tko je prije nego
što se zapiše ijedan red. **Otvori u SQL uređivaču** otvara kopiju samo na
kartici spojenoj na tu bazu, i kaže kad takve nema. Strani ključ tablice koja ostaje vraća se kao `NOT VALID` kad
ta tablica ima redove koji pokazuju na tablicu koja se vraća prazna, a
provjerava se kad se redovi vrate; tada se osvježavaju i materijalizirani
pogledi. Redovi strane tablice žive na drugom poslužitelju i ne spremaju se.
Sa samo definicijama kopija na vrhu kaže da sadrži samo definicije.

### 5.4 Sesije

Tri načina (prekidač na vrhu):

- Popis: sortirana tablica iz pg_stat_activity s pid-om, bazom, korisnikom, aplikacijom, klijentom, stanjem, čekanjem, trajanjima i upitom.
- Živo opterećenje: napredak i procjena završetka po aktivnom upitu. Spaja povijest iz pg_stat_statements (prosjek / standardna devijacija / blokovi po pozivu) i pg_stat_progress_* poglede (VACUUM / ANALYZE / CREATE INDEX / CLUSTER / COPY).

#### „idle in transaction" i što košta

Sesija u tom stanju dobiva jedno od tri očitanja, a sva su izmjerena, ne
pretpostavljena:

| što piše | što znači |
|---|---|
| *N sesija čeka na nju* | netko je zaglavio iza nje. To je problem blokiranja i te iste sesije vidiš u Blocking pogledu i u `db-locks` provjeri |
| *drži N lockova, nitko ne čeka* | uzela je lockove koje su njezine naredbe trebale i držat će ih do commita, ali ih zasad nitko ne traži |
| *ne drži nijedan lock na koji se može čekati* | read-only transakcija ostavljena otvorenom. Ne blokira nikoga |

Treće je najčešće, a prije je pisalo „holding locks" kao i sve ostalo. To je
ljude slalo u Blocking i u lockove u zdravlju clustera, gdje s pravom nisu
našli ništa - natpis je opisivao stanje, ne lockove. Vlastiti virtualni id
transakcije se ne broji: ima ga svaka sesija i ništa ga ne može čekati.

Dugo otvorenu transakciju ipak vrijedi zatvoriti i u trećem slučaju, i kartica
to kaže nakon pet minuta: najstarija otvorena transakcija drži xmin horizont,
pa autovacuum ne može počistiti nijednu verziju retka noviju od nje - nigdje u
bazi, ne samo u tablicama koje je ta transakcija dirala.
- Nadzorna ploča: snimka na jednom ekranu (verzije, stanja veza, događaji čekanja, pogodak predmemorije, najopterećenije tablice, statistika checkpointa, blokiranja, kašnjenje replikacije, najduža otvorena transakcija, autovacuum radnici).

Desni klik na sesiju: Prekini (prekid trenutnog upita) ili Ubij (gašenje backenda).

### 5.4.1 Koliko otvoreni prozor košta poslužitelj {#what-an-open-window-costs-the-server}

Spojeni prozor svakih pet sekundi pita poslužitelj jednu stvar: aktivne sesije, podatke iza alarmnih lampica u zaglavlju i je li poslužitelj primarni ili standby. Ono što se od jednog otkucaja do sljedećeg ne mijenja (koje stupce i funkcije ima, gdje je pg_stat_statements i je li učitan, postavke workera i arhiviranja) pita se jednom i pamti, deset minuta, ili jednu minutu gdje to ekstenzija može promijeniti; ekstenzija instalirana, ažurirana, premještena ili uklonjena iz aplikacije odmah se ponovno pita. Prikaz Live load koristi isti petosekundni odgovor kao zaglavlje umjesto da pita drugi put. Izmjereno na PostgreSQL 19, za jedan prozor:

| Prozor | Naredbi u minuti | Vrijeme poslužitelja u minuti | Blokova u minuti |
|---|---:|---:|---:|
| U mirovanju (tab provjere naprijed) | oko 500 | 14 ms | oko 2.000 |
| Live load naprijed | oko 470 | 12 ms | oko 1.500 |

Većina naredbi su nekoliko postavki koje svaki zahtjev postavlja na svojoj vezi (uloga, search path, vremenska ograničenja), svaka ispod 0,1 ms. Dok je prozor minimiziran ništa se ne pita. Provjera košta koliko koštaju njezini upiti, jednom, kad je pokreneš; ASH uzorkivač dodaje jedno čitanje pg_stat_activity po uzorku (vidi *db-ash* gore).

### 5.4.2 Alarmi u zaglavlju i njihove granice {#header-alarms}

Lampice alarma u zaglavlju ocjenjuju se iz istog odgovora svakih pet sekundi. Svaka granica je procjena, pa su ovdje navedene s razlogom (pregledano 27.10.2026., nakon što su alarm za prelijevanje i alarm za wraparound oba dizali lažnu uzbunu):

- **Konekcije:** upozorenje na 80% `max_connections`, kritično na 95%.
- **Blokiranje:** upozorenje kad netko čeka na zaključavanje 5 s, kritično na 60 s - minuta iza zaključavanja neće proći sama od sebe; petnaest sekundi, stara granica, bila je obična migracija.
- **Prelijevanje na disk:** brzina je prosjek 60 s iz `pg_stat_database.temp_bytes` i mora *ostati* visoka: upozorenje nakon tri minute iznad 2 MB/s, kritično nakon pet minuta iznad 20 MB/s. Jedan izvještaj koji jednom prelije gurnuo je prosjek preko granice i alarm je stajao minutu; to više ne radi. Dok rade `CREATE INDEX`, `REINDEX`, `CLUSTER` ili `VACUUM FULL` (PostgreSQL 12+ to kaže), njihovo sortiranje na disku je stvar `maintenance_work_mem` i ne ocjenjuje se.
- **Wraparound ID-eva transakcija:** `autovacuum_freeze_max_age` (zadano 200 milijuna) je mjesto gdje autovacuum sam započinje vacuum protiv wraparounda, a zauzete tablice su u normalnom radu blizu toga - stare granice, 80% i 90% toga, bile su 8% puta do pravog zastoja. Alarm se sada pali na peterostrukoj postavci, najviše na 1 milijardi, a kritičan postaje na 7,5-strukoj, najviše na 1,5 milijardi, gdje i Health check kaže CRIT; PostgreSQL-ov failsafe vacuum kreće na 1,6 milijardi, a zastoj je blizu 2,1 milijarde. Gornje granice vrijede za DBA-a koji je podigao postavku; višekratnici omogućuju testnom klasteru da dosegne granice snižavanjem postavke. Tekst navodi starost u milijunima i put do zastoja u postocima te imenuje najstariju relaciju, bazu u kojoj je, svaku relaciju preko granice i držatelja snapshota kojeg vacuum ne može preskočiti. Multixact ID-evi ocjenjuju se istim pravilom prema `autovacuum_multixact_freeze_max_age` i vode tekst kad su stariji od ova dva.
- **Otvorena transakcija:** upozorenje na 5 minuta. Kritično na 30 minuta samo kad je *idle in transaction* - nešto što ništa neće završiti; aktivna naredba te starosti je izvještaj ili batch i ostaje upozorenje.
- **Pripremljene transakcije:** upozorenje nakon sat vremena, kritično nakon dana.
- **Mrtvi slot replikacije:** upozorenje čim je neaktivan, kritično kad drži 10 GB WAL-a.
- **Arhiviranje WAL-a:** upozorenje kad zadnji pokušaj ne uspije; kritično kad neuspjesi traju pet minuta - neuspjeli pokušaj ponavlja se nakon minute. Alarm imenuje segment koji nije uspio i kada, zadnji arhivirani, naredbu ili biblioteku, i koliko segmenata čeka u `pg_wal`. `archive_mode` uključen bez `archive_command` i `archive_library` je upozorenje, kritično kad čeka 10 GB WAL-a; oba postavljena je kritično (od PostgreSQL-a 16 arhiver odbija raditi).
- **Bazeni radnika:** upozorenje na 80% bazena, kritično na 95%.
- **Kašnjenje replikacije:** upozorenje na 60 s, kritično na 300 s.
- **LISTEN/NOTIFY red:** upozorenje na 10% maksimuma reda ili 512 MB, kritično na 50% ili 4 GB, što prije dođe. Lampica se prikazuje gdje netko sluša (ili red već nešto drži); vodi na *Tablice redova i LISTEN/NOTIFY*, koja imenuje sesije koje slušaju.

Kritični alarm otvara prozor koji traži potvrdu (zapisanu u revizijski log) i može se staviti na popis zadataka; upozorenje je lampica i obavijest. Pitanje prije kritičnog prozora može se isključiti u Postavkama.

### 5.5 Održavanje

Operacije održavanja i podešavanje autovacuuma.

- Odaberite shemu i tablicu pa pokrenite VACUUM / VACUUM (FULL) / VACUUM ANALYZE / REINDEX / CLUSTER / ANALYZE / pg_repack. Napredak uživo dolazi iz pg_stat_progress_*.
- Savjetnik za autovacuum po tablici: uređujete reloptione (`autovacuum_vacuum_scale_factor`, `autovacuum_vacuum_threshold`, …) uz preporučenu vrijednost izvedenu iz vrućine tablice (vrlo vruća / vruća / srednja / zadana). Na PostgreSQL-u 13 i novijem insert-only tablica dobiva i preporučene te primijenjene `autovacuum_vacuum_insert_threshold` i `autovacuum_vacuum_insert_scale_factor`: mrtvi redci nikad ne dosegnu njezin vacuum okidač, pa je insert okidač jedini vacuum koji dobiva.

Popis opcija gradi se prema verziji samog servera. VACUUM-ov PROCESS_TOAST stigao
je u 14, a BUFFER_USAGE_LIMIT u 16; slanje takve opcije starijem serveru je tvrda
greska, ne upozorenje, pa se ono sto server ne poznaje uopce ne nudi. Gumbi za
VACUUM / ANALYZE / REINDEX u table advisoru i u vacuum status advisoru koriste
isti popis, pa isti gumb znaci isto gdje god ga pritisnete.

Ovdje nema log prozora. Maintenance je background posao kao i svaki drugi: panel
pokazuje da nesto radi i vodi u **Background jobs**, gdje se izlaz cita uz sve
ostale duge operacije.

**Izvrsavanje koje nije nista napravilo to i kaze.** PostgreSQL preskoci relaciju
koju pozivatelj ne smije odrzavati i svejedno javi uspjeh - naredba vrati VACUUM,
uz samo upozorenje da se nista nije dogodilo. Posao zavrsava kao *skipped* a ne
*done*, i imenuje sto je preskoceno.

### 5.6 Poslužitelj

Postavke klastera + uređivač pg_hba + ekstenzije + tablespaceovi + role.

- Ključne postavke: uredive kartice za ~70 parametara podešavanja s obrazloženjem, praktičnim pravilom i oznakama stanja. Promjene se spremaju kroz ALTER SYSTEM (gdje ga pružatelj podržava).
- pg_hba.conf: tekstualni uređivač koji prije spremanja radi kopiju. Vraćanje s bilo koje kopije. Pružatelj mora podržavati izravno pisanje datoteka.

#### Svaki ALTER SYSTEM napravljen ovdje može se poništiti {#settings-undo}

Svaka promjena primijenjena iz DBApguina - ključne postavke, savjeti za
podešavanje, uređivač postavki - zapisuje se s vrijednošću koju je
zamijenila i s time odakle je ta vrijednost došla. Obavijest koja potvrđuje
promjenu nosi **Poništi**, a prikaz postavki navodi promjene napravljene
ovdje, najnovije prve, svaku s Poništi dok se ne poništi. Poništi je
točno ona naredba koja parametar vraća kakav je bio: `ALTER SYSTEM SET` na
staru vrijednost kad je ta vrijednost bila DBApguinov vlastiti redak u
`postgresql.auto.conf`, inače `ALTER SYSTEM RESET`, pa opet vrijedi stari
izvor (postgresql.conf, zadana vrijednost). Zapis je po poslužitelju, u
mapi s podacima, a dnevnik radnji čuva vrijednost prije i poslije.

#### pg_hba.conf pripada jednom nodeu

`pg_hba.conf` je datoteka u data direktoriju jednog servera, a replikacija je
ne prenosi: WAL nosi podatke, ne konfiguraciju. Pravilo dodano na primaryju
nije na standbyju, koji i dalje ima ono što je zateklo u base backupu. Dan kad
to postane bitno je failover - veze koje su jučer radile odbija node koji
izmjenu nikad nije dobio, a tada nitko ne gleda u ovaj ekran.

Zato uređivač to piše iznad datoteke i imenuje nodeove gdje god server kaže tko
su:

- **standby nodeovi koji čitaju s ovog servera**, s imenom aplikacije i
  adresom, iz `pg_stat_replication`;
- **fizički replikacijski slotovi**, kad je standby konfiguriran ali trenutno
  odsutan - slot bez ičega na sebi i dalje je node koji se očekuje natrag;
- **„ovaj node je standby"**, kad uređuješ kopiju na replici: primary ima svoju
  i izmjena odavde do njega ne dolazi.

Kad se ništa od toga ne vidi, poruka je kraća ali ostaje. Rola bez `pg_monitor`
ne može čitati `pg_stat_replication`, a odspojen standby odavde izgleda isto
kao onaj koji ne postoji. Kakva god bila topologija, istu promjenu treba
napraviti na svakom nodeu - uključujući i connection pooler ispred njih, koji
ima svoja pravila svoje vrste.

**Uređivanje datoteke na replici radi.** Prije je padalo uz *cannot set
transaction read-write mode during recovery* - poruka o transakcijskim
načinima, iz operacije koja je pisala datoteku. Dva su uzroka: aplikacija je
tražila read-write sesiju, koju server u recoveryju odbija, a datoteka se
pisala preko large objecta, što je pisanje u bazu. Ni jedno nije potrebno.
Sesija se sada traži onakva kakvu server može dati, a na standbyju se datoteka
piše s `COPY … TO`, što ne dira ništa u bazi. Sigurnosna kopija prije
spremanja, reload nakon toga i prijava grešaka u parsiranju rade jednako.
- Ekstenzije: popis te instalacija / nadogradnja / brisanje / promjena sheme po ekstenziji, uz birač verzije.
- Tablespaceovi: popis te stvaranje / izmjena / brisanje.
- Role: popis s uređivačem atributa. Klik na rolu otvara punu reviziju te role.

### 5.7 Role, korisnici i ovlasti {#roles-users-and-privileges}

#### Uređivač rola

`Pregled rola i korisnika` popisuje korisnike s prijavom i grupne role u dvije
uredive tablice. Svaki logički atribut je padajući izbornik true/false;
`conn_limit` i `valid_until` su tekstualna polja.

**Prvo odaberite rolu, onda radnju.** Klik na red odabire tu rolu, a isto radi
i kvačica; ctrl-klik i kvačice odabiru više njih. Jedan red gumba stoji uz
*+ Create role / user* i radi nad onim što je odabrano - skriven je dok ništa
nije odabrano. Kad je odabrano više rola, ostaju aktivne radnje koje imaju
smisla nad skupom (**Privileges**, **Lock**, **Drop**, **Save**), a one koje
mijenjaju postavke pojedine role zasive i kažu zašto.

**Save** šalje samo razliku, pa rola koju ste pogledali i ostavili na miru ne
ispaljuje ništa. Kad je odabrano više njih, izdaje po jedan `ALTER ROLE` za
svaku koju ste zaista promijenili, a prije toga ih popiše u potvrdi.

**Edit** otvara tu rolu u vlastitom tabu - atributi, privilegije i članstva na
jednom mjestu, jedan tab po roli, pa se dvije role mogu uređivati jedna uz
drugu. Obrazac za privilegije u tom tabu isti je obrazac koji gumb
**Privileges** otvara kao dijalog.

**Privileges** s više odabranih rola prikazuje trenutne privilegije prve role i
upisuje isti GRANT / REVOKE set na svaku odabranu; natpis na vrhu ih imenuje.
Ne postoji zajedničko trenutno stanje pet rola - zajednička je promjena - pa je
prikazana podloga jedne role i dijalog kaže koje.

Grupne role imaju vlastiti stupac **`can_login`**. Grupna rola je po definiciji
`NOLOGIN`, pa stupac uvijek počinje kao false - tu je da grupnu rolu kasnije
možete pretvoriti u rolu s prijavom bez izlaska iz aplikacije. Korisnici s
prijavom imaju gumb **Zaključaj** za suprotan smjer, koji uz potvrdu postavlja
`NOLOGIN`.

#### Ograničenja i timeouti za jednu rolu

Otvoriš atribute role i u odjeljku **Ograničenja i istek** stoje, uz
ograničenje broja konekcija i datum isteka, pet timeouta po sesiji:

| postavka | zaustavlja |
|---|---|
| `statement_timeout` | jednu naredbu koja predugo traje |
| `lock_timeout` | čekanje na zaključavanje |
| `idle_in_transaction_session_timeout` | sesiju koja miruje unutar otvorene transakcije |
| `idle_session_timeout` | sesiju koja miruje izvan transakcije (PostgreSQL 14+) |
| `transaction_timeout` | cijelu transakciju, koliko god bila zaposlena (PostgreSQL 17+) |

Nudi se samo ono što ovaj poslužitelj ima - zadnja dva stigla su u kasnijim
verzijama, a ponuditi polje koje će poslužitelj odbiti gore je nego ga
izostaviti.

**Goli broj su milisekunde.** PostgreSQL čita vrijednost bez jedinice u
vlastitoj jedinici postavke, a za svih pet to su milisekunde: `30` znači
trideset milisekundi, što nije ono što itko misli kad napiše 30. Napiši `30s`,
`2min` ili `500ms` i nema se što nagađati. Prazno polje briše postavku za
doseg u kojem se nalaziš.

**Inače** je vrijednost na koju rola naiđe kad ovaj doseg ne postavlja ništa, i
imenuje sloj iz kojeg dolazi - vlastita postavka role za sve baze, postavka
baze, ili klastera, redom kojim ih PostgreSQL primjenjuje. Kod klastera piše
dolazi li iz konfiguracijske datoteke ili je ugrađena zadana vrijednost, jer to
znači različite stvari kad odlučuješ hoćeš li je uopće postaviti. Čita se bez
utjecaja postavki spojene baze: baza koja nosi
`ALTER DATABASE … SET idle_in_transaction_session_timeout = '90s'` znala je
navesti ovaj stupac da 90s prikaže kao vrijednost klastera, a klaster ne
nameće ništa.

**Doseg** bira gdje promjena slijeće. Ostane li na *svaka baza*, to je
`ALTER ROLE … SET`; odabereš li bazu, to je `ALTER ROLE … IN DATABASE … SET`,
što je uža postavka i ondje pobjeđuje. Popis označava baze koje za ovu rolu već
nešto nose, pa dvadeset baza s dva override-a više ne znači dvadeset otvaranja
popisa.

Ostali atributi role - LOGIN, INHERIT, SUPERUSER, CREATEDB, ograničenje veza,
istek - **nisu** po bazi i nemaju izbornik dosega. Žive u katalogu zajedničkom
cijelom klasteru, pa `ALTER ROLE` ima isti učinak iz koje god baze ga pokreneš.
Privilegije na tablicama, shemama i funkcijama su obrnute: one žive u
katalozima svake pojedine baze, a uređivač privilegija primjenjuje ih u bazi
navedenoj na vrhu prozora.

Ovo su `USERSET` postavke: rola ih za vlastitu sesiju može podići ili
isključiti običnim `SET`-om. Zaustavljaju nesreću, ne nekoga s namjerom. Jedino
ograničenje koje rola sebi ne može ukloniti jest ograničenje broja konekcija
iznad njih - vrijedi znati kad biraš između to dvoje.

#### Što jedno članstvo daje

Kvačica na roli u **Član role** izdaje `GRANT parent TO ova_rola`. Uz svaku
stoje opcije koje odlučuju što članstvo zapravo daje:

| | |
|---|---|
| **ADMIN** | član smije ovu rolu dodijeliti dalje drugima i oduzeti je. Po defaultu isključeno; bez toga je članstvo slijepa ulica |
| **INHERIT** | član automatski koristi privilegije role, bez `SET ROLE`. PostgreSQL 16+ |
| **SET** | član uopće smije napraviti `SET ROLE` na nju. PostgreSQL 16+, po defaultu uključeno |

Zadnje dvije stigle su u PostgreSQLu 16, koji je jedno članstvo razdvojio na tri
odvojena odgovora; na starijim serverima nudi se samo ADMIN, jer samo ADMIN i
postoji. Svaka kućica prikazuje ono što članstvo trenutno nosi, pročitano sa
servera, pa se članstvo dodijeljeno negdje drugdje vidi kakvo doista jest, a ne
kao default.

INHERIT isključen uz SET uključen je oblik koji vrijedi znati: član drži
privilegije samo dok je namjerno postao rola, čime se sprječava da objekti
slučajno nastanu pod krivim vlasnikom. INHERIT uključen uz SET isključen je
obrnuto - privilegije vrijede uvijek, a postati rola je zabranjeno.

Promjena opcije na članstvu koje već postoji ponovno izdaje `GRANT`. Za njega
nema `ALTER`, i SQL pregled pokazuje točno to.

Svaka kućica u ovom dijalogu i u Stvori rolu objašnjava se na hover.

#### Uređivač ovlasti

Otvara se iz menija desnog klika na rolu ili preko `Ovlasti` u uređivaču rola.
Odnosi se na jednu rolu i jednu shemu odjednom; birač sheme je na vrhu.

Kvačice prikazuju **što rola već ima**, i imaju tri stanja umjesto dva, jer
svaka pokriva sve objekte u shemi odjednom:

| Stanje | Značenje |
|---|---|
| Označeno | svaki objekt u shemi ima tu ovlast |
| Crtica | neki imaju, neki ne - broj je uz oznaku, npr. `[3/12]` |
| Prazno | nijedan nema |

Crtica nije odluka. Ostavite je na miru i ništa se za nju ne šalje. Kliknete
jednom i postaje označena - `GRANT … ON ALL …`, koji podiže ostatak. Kliknete
opet i briše se - `REVOKE`, koji je oduzima onima koji su je imali.

SQL panel na dnu pokazuje točno što će Primijeni pokrenuti i piše "nema
promjena" dok stvarno nešto ne promijenite. Otvaranje uređivača i pritisak na
Primijeni ne radi ništa.

Zasivljene ovlasti su one koje **korisnik s kojim ste spojeni** ne može
dodijeliti; tooltip to i kaže.

Gumb `Radnje` uz Primijeni nosi sve iz menija desnog klika na rolu - revizija,
`search_path`, stvori sličnu, kopiraj ovlasti, izvezi DDL, usporedi, obriši - 
pa uređivač ne morate zatvarati da biste do njih došli. Sve što mijenja rolu
prvo zatvara uređivač, da se ustajali ekran ne bi primijenio preko toga.

#### Oprez oko toga što znači "ima"

Ovi ekrani prikazuju **stvarne** ovlasti: što rola doista može, uključujući ono
što nasljeđuje kroz članstvo u grupi i ono što dobiva time što je superuser.
Superuser će zato imati sve označeno, bez obzira postoji li ijedna izričita
dodjela. Za samo izravne dodjele koristite ekran ovlasti po objektu u prikazu
Shema.

#### Izvoz GRANT-ova

**Izvezi GRANT-ove** zapisuje ovlasti kao skriptu koju možete čuvati u gitu ili
pokrenuti negdje drugdje. Nalazi se na tri mjesta:

- u prozoru ovlasti jednog objekta: taj objekt i ovlasti nad njegovim
  stupcima (i ovlast nad stupcem particionirane tablice);
- u prozoru ovlasti sheme: shema i svaka tablica, pogled, sekvenca, stupac,
  rutina i tip u njoj, sa zadanim ovlastima postavljenima `IN SCHEMA` za nju;
- u auditu ovlasti role: sve što je roli dodijeljeno, u bazama koje
  označite (baza na koju ste spojeni je označena, *Sve baze* čita svaku), te
  njezine ovlasti nad bazama, tablespaceovima i parametrima poslužitelja. S
  više od jedne baze svaki dio počinje s `\connect`, pa takvu skriptu
  pokrenite psql-om.

Skripta od objekata koje su isti vlasnici ponovno stvorili radi one koje ste
izvezli. Zato vlastite ovlasti vlasnika nisu zapisane, a ovlast koja je
oduzeta - `EXECUTE` od `PUBLIC`, vlasnikov vlastiti `TRUNCATE`, zadana ovlast
uklonjena s `ALTER DEFAULT PRIVILEGES ... REVOKE` - izlazi kao `REVOKE`. Ono
što rola dobiva članstvom u drugoj roli nije u njezinom izvozu; skripta te
role imenuje.

### 5.8 Preimenovanje sheme {#renaming-a-schema}

Desni klik na shemu u pregledu → **Preimenuj shemu**.

Samo preimenovanje je jedan `ALTER SCHEMA ... RENAME TO ...`, a PostgreSQL uz
njega nosi gotovo sve: tablice, poglede, sekvence, indekse, ograničenja, strane
ključeve i dodjele - sve slijedi shemu bez dodatnog posla. Ime se provjerava
prije pokušaja, pa se već zauzeto ime javlja kao odgovor, a ne kao greška.

Ono što **ne** ide automatski jest svaka referenca zapisana kao tekst unutar
rutine - tijelo funkcije koje kaže `SELECT * FROM staro_ime.t` ili `search_path`
postavljen na funkciju. Njih ispisuje popis nakon preimenovanja pod
*"rutine i dalje referenciraju ..."*, s imenom rutine i mjestom gdje je
referenca nađena (tijelo, `search_path` ili oboje). Ništa se ne prepisuje umjesto
vas: tijelo funkcije je kod, a tiho uređivanje tuđeg koda gore je nego reći im
koje četiri funkcije trebaju pogledati.

Ako ciljno ime već postoji, preimenovanje nije ono što želite - vidi niže.

### 5.9 Spajanje dviju shema {#merging-two-schemas}

Desni klik na shemu u pregledu → **Spoji shemu**.

`ALTER SCHEMA ... RENAME` ne može uklopiti jednu shemu u drugu, jer je ciljno
ime već zauzeto. Spajanje je zato po objektu, a svaki objekt treba vlastitu
odluku kad isto ime postoji s obje strane. DBApguin to razriješi prije nego se
išta dogodi.

#### Plan

Odabir cilja i pritisak na **Analiziraj** pokreće provjeru koja ništa ne mijenja
i razvrstava svaku tablicu, pogled, materijalizirani pogled i sekvencu u izvoru:

| U cilju | Što znači | Radnje koje možete odabrati |
|---|---|---|
| `nema ga` | ima ga samo izvor | premješta se - nema se što odlučiti |
| `isti stupci` | oboje ga imaju, imena i tipovi stupaca se poklapaju | **Spoji retke**, **Zamijeni**, **Preskoči** |
| `stupci se razlikuju` | oboje ga imaju, oblici se ne slažu | **Zamijeni**, **Preskoči** |
| `ide sa svojom tablicom` | `serial` / identity sekvenca | nijedna - slijedi svoju tablicu |

Objekti koji postoje samo u *cilju* popisani su ispod i nikad se ne diraju.

Stupac radnje je po objektu. **Postavi sve** je pomoć koja primjenjuje izbor
svugdje gdje je dopušten, a ostalo ostavlja na miru - neće naoružati *Spoji
retke* na tablici kojoj se stupci razlikuju. Radnju koju provjera nije ponudila
poslužitelj odbija umjesto da je tiho spusti, pa ono što je dijalog pokazao jest
ono što se izvodi.

#### Što koja radnja radi

- **Premjesti** - `ALTER ... SET SCHEMA`. Sam objekt mijenja shemu. Dodjele,
  komentari, indeksi i ograničenja idu s njim, a **iz izvora nestaje**.
- **Kopiraj** - izvor zadržava sve. Tablica se u cilju ponovno stvara s
  `LIKE ... INCLUDING ALL` (zadane vrijednosti, ograničenja, indeksi, komentari,
  postavke pohrane) i njeni se retci umeću; sekvenca se ponovno stvara na istoj
  vrijednosti. Nudi se samo za tablice i sekvence: definicija pogleda imenuje
  shemu iz koje čita, pa bi kopirani pogled tiho i dalje čitao podatke izvora,
  što je gore nego ne ponuditi taj izbor.
- **Zamijeni** - objekt u cilju briše se s `CASCADE`, a njegovo mjesto zauzima
  onaj iz izvora. To uništava podatke cilja *i* dodjele cilja nad tim objektom.
  To je jedini doista razoran izbor u dijalogu, pa gumb Spoji postaje crven i
  broji što će biti obrisano.
- **Spoji retke** - tablica u cilju ostaje točno gdje jest, a retci iz izvora se
  u nju umeću. Izvorna tablica se ne prazni i ne briše: njeni retci su i dalje
  tamo, pa se spajanje može ponoviti ili provjeriti prije čišćenja.

Dakle izvor preživljava **Kopiraj** i **Spoji retke**, a ne preživljava
**Premjesti** ni **Zamijeni**. Plan za svaki objekt kaže što će biti prije nego
pokrenete.
- **Preskoči** - objekt u cilju ostaje na miru.

#### Kad se retci sudare

Odnosi se samo na *Spoji retke*, a odlučuje primarni ključ ciljne tablice:

- **Zadrži redak iz cilja** - `ON CONFLICT DO NOTHING`. Pobjeđuju retci koji su
  već tamo; novi se dodaju.
- **Neka pobijedi redak iz izvora** - upsert. Sudareni retci prepisuju se
  vrijednostima iz izvora.
- **Prekini spajanje** - običan `INSERT`. Prvi sudar prekida posao, a kako je
  cijelo spajanje jedna transakcija, ne primjenjuje se baš ništa.

Ako ciljna tablica **nema primarni ključ**, nema po čemu prepoznati sudar, pa se
svaki redak iz izvora dodaje - uključujući duplikate. Plan to kaže na samom
retku, prije pokretanja, umjesto da udvostručene retke otkrijete poslije.

#### Što je dobro znati

- **Cijelo spajanje je jedna transakcija.** Ako išta padne, ništa se ne
  primjenjuje. Spajanje prekinuto na pola ostavilo bi dvije napola popunjene
  sheme i nikakav zapis o tome gdje je stalo, što je gore nego da nije ni počelo.
- **`serial` ili identity sekvenca ide sa svojom tablicom**, jer je PostgreSQL
  premješta kao dio `ALTER TABLE ... SET SCHEMA`. U planu je navedena da vidite
  da će se premjestiti, ali nema vlastitu radnju. Samostalne sekvence su obični
  objekti i dobivaju izbor.
- **Premješta se sve u shemi, ne samo tablice.** `ALTER ... SET SCHEMA` radi na
  svakoj vrsti objekta vezanoj na shemu, pa se funkcije, procedure, agregati,
  tipovi (enum, domena, kompozit, range), kolacije, konverzije, operatori,
  operatorske klase i familije, text search konfiguracije, rječnici, parseri i
  predlošci, proširene statistike i strane tablice premještaju zajedno s
  relacijama. Nabrojani su u vlastitom dijelu plana, *Sve ostalo u izvoru*, i
  svaki prima vlastitu akciju isto kao tablica.

  Dugo nije bilo tako: plan se gradio iz `pg_class`, četiri vrste relacija su
  se premjestile, a funkcije i tipovi su ostajali u izvoru bez ijedne riječi.
- **Range tip nosi svoje pratioce.** `ALTER TYPE` premjesti range tip i njegov
  array tip i ništa više: multirange koji je PostgreSQL stvorio uz njega i
  konstruktorske funkcije obojega ostaju, a range tip koji je otišao bez njih
  i dalje ovisi o izvornoj shemi. Premještaju se u istoj skupini naredbi, a
  plan kaže koliko ih je pošlo.
- **Što se ne može premjestiti, kaže zašto.** Dva slučaja. Objekt čije je ime
  već zauzeto u odredištu - rutina istog imena i istih tipova argumenata,
  operator s istim operandima, operatorska klasa s istom index metodom - može
  se samo preskočiti: `SET SCHEMA` bi pao na koliziji, a kako je spajanje jedna
  transakcija, taj jedan pad bi poništio sve. Za razliku od tablice, ovdje nema
  **Replace**, jer `DROP TYPE ... CASCADE` u odredištu odnosi i njegove vlastite
  stupce, a dijalog vam ne može pokazati koliko to uništava; najprije rukom
  uklonite objekt iz odredišta. Drugi slučaj je nepremjestiva ekstenzija, koju
  PostgreSQL odbija premjestiti.
- **Sve što ostane je imenovano, kao i ono što ga i dalje treba.** Ako tip
  ostane jer mu je ime bilo zauzeto, stupac premještene tablice je i dalje njime
  tipiziran. Ta ovisnost ide preko OID-a i ne puca, no izvorna shema se ne može
  obrisati dok drži taj tip, pa se nabraja i objekt i stupac koji ga treba. Kad
  se sve premjesti, oba popisa su prazna.
- **Obriši izvor kad je prazan** briše ga samo kad je shema stvarno prazna.
  `DROP SCHEMA` je `RESTRICT`: bilo što u izvoru ga odbija, a kako je brisanje
  zadnja naredba u jednoj transakciji, to je odbijanje prije rollbackalo svako
  premještanje prije sebe - čekali ste spajanje koje nije premjestilo ništa i
  dobili grešku o ovisnostima objekata koje nikad niste tražili premjestiti.
  Sada se spajanje commita, izvor se zadrži, a rezultat kaže koji su ga objekti
  zadržali.
- **Hardkodirane reference se označe prije pokretanja.** Viewovi, strani
  ključevi i CHECK ograničenja grade se iz kataloga pri svakom čitanju pa
  besplatno slijede premještenu tablicu. Tijelo PL/pgSQL funkcije ili okidača
  je običan tekst koji se razrješava po imenu u trenutku poziva: tijelo koje
  kaže `public.narudzbe` i dalje pokazuje na staru shemu nakon što tablica
  postane `arhiva.narudzbe`, i pukne pri sljedećem pozivu, a ne pri spajanju.
  Predprovjera baš to skenira u `pg_proc` i `pg_policies` i popisuje što bi
  puklo, pa ta tijela prvo popravite ili zaključite da je referenca u redu.
  RLS politike su vezane na OID kao view i označe se samo kad zovu funkciju
  koju je skeniranje već uhvatilo.
- **Strani ključevi i dalje pokazuju kamo su pokazivali.** Premještena tablica
  čiji je strani ključ pokazivao na tablicu koja je ostala završi s
  međushemskim stranim ključem koji i dalje vrijedi. To je ispravno i obično je
  znak da je i referencirana tablica htjela ići.
- **Obriši izvornu shemu ako ostane prazna** radi točno to i samo to: ako je išta
  preskočeno, shema ostaje, a dijalog vam kaže koliko je objekata još u njoj.

### 5.10 Backup i vraćanje {#backup-and-restore}

Desni klik na bazu → **Backup** ili **Vrati**, ili na shemu →
**Izvezi shemu (pg_dump)**.

Oboje pokreće prave `pg_dump` / `pg_restore` programe. Njihova lokacija dolazi
iz Postavke → Alati ako ste je zadali, inače iz automatskog pronalaska;
nepodudarnost verzije alata i poslužitelja se javlja, a ne nagađa.

#### Odredište se provjerava prvo

Dump koji padne nakon dvadeset minuta jer je mapa bila samo za čitanje je
bacanje dvadeset minuta, pa se putanja provjerava prije nego `pg_dump` krene:

- direktorij zadan tamo gdje se očekivala datoteka se odbija
- mapa u koju ne možete pisati se odbija, uz njeno ime
- postojeća datoteka se javlja, a ne tiho prepisuje

Gumbom **Pregledaj** birate mapu umjesto da tipkate putanju. Prikazuje samo
direktorije, označava one u koje se može pisati i skriva skrivene mape.
Pregledava datotečni sustav **poslužitelja**, ne ovog računala - pg_dump radi na
poslužitelju, pa putanja mora postojati tamo. Isti preglednik koriste i backup
baze i izvoz sheme.

#### Backup cijelog klastera {#backing-up-the-whole-cluster}

Desni klik na bazu → **Backup klastera (pg_dumpall)…**

`pg_dump` radi unutar jedne baze i strukturno ne može doseći ono što živi izvan
nje. **Role, tablespaceovi i postavke po bazi vrijede na razini klastera**, pa
backup napravljen samo s `pg_dump` vraća tablice koje nema tko posjedovati - 
a to se otkriva tijekom vraćanja, ne prije njega.

| Sadržaj | Što dobivate |
|---|---|
| Samo globalni objekti *(preporučeno)* | role i tablespaceove - pratitelj vaših dumpova po bazi |
| Samo role | samo `CREATE ROLE` |
| Samo tablespaceovi | samo `CREATE TABLESPACE` |
| Sve | globalne objekte plus svaku bazu u klasteru, u jednoj datoteci |

*Samo globalni objekti* je ono što većina želi: nastavite raditi dumpove po bazi
u custom formatu, a ovo radite uz njih. *Sve* daje jednu veliku plain-SQL
datoteku i sporije se selektivno vraća.

`pg_dumpall` **nema izbor formata** - ispis je uvijek plain SQL, pa se vraća
`psql`-om, ne `pg_restore`-om. Dijalog to kaže nakon dumpa, a ne prepušta vam da
sami otkrijete.

#### Opcije pg_dumpa {#pg-dump-options}

Otvorite **Opcije** u dijalogu backupa. Svaka kontrola odgovara jednoj `pg_dump`
zastavici, a zastavica je ispisana na kontroli, pa svoj izbor možete provjeriti
u PostgreSQL dokumentaciji:

- **Format** - custom (`-Fc`, onaj koji `pg_restore` želi), plain SQL, direktorij ili tar
- **Kompresija** - 0-9; vidi niže
- **Odjeljak** - koju trećinu dumpa zapisati; vidi niže
- **Samo ove sheme / Samo tablice / Isključi tablice / Bez podataka tablica / Isključi sheme** - imena odvojena zarezom, vidi *Imena i particije* niže. **Odaberi…** uz *Samo ove sheme* nudi sheme baze za označavanje
- **Paralelnih poslova** - samo za format direktorija. Za sve druge formate polje je onemogućeno, uz razlog uz opcije, umjesto da ga ispunite pa naknadno padne
- `--no-owner`, `--no-privileges`, `--clean`, `--create`, `--inserts`, `--no-comments`, `--no-tablespaces`
- **Napredno** - `--load-via-partition-root`, `--no-publications`, `--no-subscriptions`, `--no-security-labels`, `--enable-row-security`, veliki objekti uključeni ili izostavljeni, vrijeme čekanja na zaključavanje, *Samo ekstenzije* (`--extension`) i *Pokreni kao rola* (`--role`)

`--no-owner`, `--clean` (s `--if-exists`), `--create` i `--no-tablespaces`
mijenjaju samo **običnu SQL** datoteku. Za custom, tar ili directory arhivu
`pg_dump` ih bez riječi zanemaruje - biraju se pri vraćanju arhive - pa su za te
formate isključeni, s razlogom ispod opcija, a sažetak ih ne obećava.

Koji `pg_dump` se pokreće ovisi o serveru (klijent mu se prilagođava), a opcija
koju taj klijent nema isključena je uz verziju koja joj treba - `--extension`
traži 14, na primjer. Redak ispod *Napredno* imenuje klijenta.

**Imena i particije.** Imena pišite onako kako ih katalog prikazuje, s velikim i
malim slovima: `Sales.Orders` pronalazi tablicu koju stablo prikazuje kao
`Sales.Orders`. Ime koje sadrži točku ili zarez stavite u dvostruke navodnike
(`"order.lines"`). Svako se ime traži prije početka dumpa: ono koje ništa ne
pogađa - tipfeler, tablica u drugoj shemi - zaustavlja ga uz to ime i, kad se
razlikuju samo velika i mala slova, uz ime koje postoji. Ime koje ništa ne pogađa
prije se tiho izostavljalo iz dumpa koji je onda izgledao potpun.

Particionirana tablica su njezine particije: sam roditelj nema redaka. Zato
tablica u *Samo tablice* dolazi **sa svojim particijama i tablicama-djecom, na
svim razinama** (kućica ispod polja, zadano označena), a isto vrijedi za
tablicu u *Isključi tablice* ili *Bez podataka tablica*. S `pg_dumpom` 16 ili
novijim to je `--table-and-children`; stariji klijent dobiva svaku particiju
imenovanu jednu po jednu. Kad je dump zapisan, njegova provjera potvrđuje da je
svaka tražena tablica, s particijama, u datoteci. Što `pg_dump --table`
izostavlja kaže se kad dump krene: što odabrane tablice koriste, a nije
njihovo - funkcija koju poziva okidač, samostalna sekvenca iz koje vuče zadana
vrijednost, tip stupca, tablica na koju pokazuje strani ključ, ekstenzija -
što baza u koju se dump vraća mora već imati, i što njih koristi (pogled,
strani ključ druge tablice).

Ništa se ne prosljeđuje kao slobodan tekst. Opcija koju DBApguin ne poznaje se ne
šalje, i to je ono što sprječava da zalutali `--nešto` završi u naredbenom retku.

**Izvoz jedne sheme koristi iste opcije.** Desni klik na shemu →
*Izvezi shemu (pg_dump)* otvara isti skup opcija, bez *Isključi sheme* - koje u
dumpu već ograničenom na jednu shemu ionako ništa ne mijenja. Oba su dijaloga
građena iz jedne definicije, pa se opcija dodana bilo kojem pojavljuje u oba.

Jedno što izvoz sheme radi umjesto vas: `pg_dump` uspoređuje `--table` uzorke s
imenima *kvalificiranima shemom*, pa traženje `orders` u dumpu jedne sheme ne
pogađa ništa i alat stane uz *"nije pronađena nijedna tablica"*. Imena upisana
bez sheme kvalificiraju se shemom koja se izvozi, pa `orders` znači `sales.orders` -
a shema je u navodnicima, pa se ona s velikim slovima usput ne pretvara u mala.

Puna odredišna putanja prikazana je ispod polja u oba dijaloga. Tekstualno polje
pokazuje kraj duge putanje i skriva početak, a to je upravo polovica koja imenuje
stroj i direktorij.

#### Odjeljak: koja su tri dijela {#section-what-the-three-parts-are}

Dump se zapisuje u tri prolaza, a `pg_dump` dopušta da uzmete samo jedan.
**sve** je zadano i ono je što želite osim ako imate konkretan razlog da ne.

| Odjeljak | Što u njega ide |
|---|---|
| `pre-data` | `CREATE SCHEMA`, `CREATE TABLE`, `CREATE VIEW`, `CREATE SEQUENCE`, vlasnici - sve što tablici treba da postoji, ali ništa što bi usporilo punjenje |
| `data` | sami retci, kao `COPY` blokovi, plus `setval` pozivi koji sekvence ostavljaju na trenutnoj vrijednosti |
| `post-data` | primarni ključevi, unique i strani ključevi, indeksi, okidači - sve što provjerava ili indeksira retke |

Podjela nije proizvoljna: ograničenja i indeksi idu zadnji **jer je punjenje
redaka u tablicu koja ih već ima daleko sporije**, budući da bi se svaki redak
provjeravao i indeksirao pojedinačno. Vraćanje pre-data, pa data, pa post-data
gradi indekse jednom, na kraju, nad gotovom tablicom.

Uzimanje jednog odjeljka je za posebne slučajeve:

- `pre-data` - prazan oblik baze, za usporedbu struktura ili za cilj koji ćete
  puniti nekim drugim putem
- `data` - osvježavanje redaka u strukturu koja već postoji
- `post-data` - ponovno stvaranje ograničenja i indeksa nakon punjenja samo podacima

Vraćanje `data` dumpa u bazu bez tablica pada, kao i `post-data` bez tablica na
koje bi se objesio. Odjeljci su dijelovi jednog dumpa, ne tri neovisna backupa.

#### Kompresija: što koji broj košta {#compression-what-the-numbers-cost}

Kompresija vrijedi za custom, direktorij i plain format. **Tar se uopće ne može
komprimirati** - polje je onemogućeno kad ga odaberete.

Mjereno na tablici od 120.000 redaka hasheva (realna mješavina, ne datoteka
ponovljenog teksta):

| Razina | Veličina | Vrijeme |
|---|---|---|
| 0 - bez | 13,4 MB | 0,3 s |
| 1 | 7,5 MB | 0,8 s |
| 5 | 7,35 MB | 1,5 s |
| 9 - najmanje | 7,35 MB | 1,7 s |
| zadano | 7,35 MB | 1,6 s |

Oblik te tablice je cijela poanta: **gotovo sva ušteda je između 0 i 1.** Razina
1 daje 44% manje za pola sekunde; put od 1 do 9 kupuje još 2% i traje dvostruko
dulje. Razina 9 nije "puno manje", nego "isto, ali sporije".

Dakle:

- ostavite **zadano** ako nemate razlog - već je blizu najbolje veličine
- odaberite **0** kad dump ide negdje gdje se ionako komprimira, ili na brz disk
  gdje vrijeme znači više od prostora
- odaberite **9** samo kad datoteka mora putovati nečim sporim, a svejedno vam je
  koliko dump traje

Custom i direktorij format komprimiraju po zadanom; plain SQL dump je
nekomprimiran osim ako ne tražite drukčije, a tražiti znači dobiti gzip datoteku
umjesto čitljivog SQL-a.

#### Uvoz jedne sheme {#importing-one-schema}

Desni klik na bazu → **Uvezi shemu (pg_restore)…**. To je dijalog vraćanja s
filtrom sheme sprijeda: imenujte shemu onako kako se zove u dumpu i samo se ona
vraća u ovu bazu. Ostavite prazno i vraća se cijeli dump, a to je obično Vraćanje.

Radi i s dumpom cijele baze - ne treba vam dump samo jedne sheme da biste iz njega
izvukli jednu shemu.

Jedno što DBApguin ovdje radi umjesto vas: `pg_restore --schema sales` vraća
objekte *unutar* `sales`, ali `CREATE SCHEMA sales` nije jedan od njih. U bazi
koja tu shemu još nema, svaki pojedini objekt padne uz *"schema sales does not
exist"* - a `pg_restore` svejedno izađe javljajući samo "errors ignored on
restore". Kad baza tu shemu nema, ona se zato prvo vraća iz dumpa - s vlasnikom,
pravima i komentarom kakve ima u dumpu. Shema koja postoji ostaje točno kakva
jest.

#### Uvoz pod drugim imenom sheme

PostgreSQL nema ekvivalent Oracleovog `REMAP_SCHEMA`. Ni `pg_dump` ni
`pg_restore` ne mogu preimenovati shemu pri uvozu - takve opcije nema ni u
jednom od njih.

Ono što ima jest `ALTER SCHEMA … RENAME`, koji preimenuje kako treba:
ograničenja, sekvence, indeksi i dodjele slijede. Zato **Vrati kao shemu** u
dijalogu uvoza vraća pod imenom iz dumpa pa preimenuje nakon toga. Isti rezultat,
putem koji poslužitelj doista podržava.

Ako je traženo ime već zauzeto, preimenovanje se *ne* forsira - dvije sheme se ne
mogu uklopiti tako da se jedna preimenuje preko druge. Uvoz to kaže i upućuje na
**Spoji shemu** (5.9), koja postoji upravo za to.

#### Što vraćanje može ostaviti pokvarenim

`pg_restore` javlja probleme i nastavlja dalje. Uvoz ograničen na shemu čiji
strani ključ pokazuje na tablicu u drugoj shemi je uobičajen slučaj: ograničenje
padne, tablica se vrati **bez njega**, a `pg_restore` izađe govoreći samo
*"errors ignored on restore"*. Nakon toga ništa u bazi ne bilježi da je to
ograničenje trebalo postojati.

Nema ni popisa neispravnih objekata koji bi se dao pogledati. **PostgreSQL nema
stanje INVALID kakvo ima Oracle** - `pg_proc` nema stupac valjanosti, a funkcija
koja referencira nepostojeću tablicu stvori se bez prigovora i padne tek kad je
netko pozove. Pogledi uopće ne mogu postati neispravni, jer poslužitelj odbija
obrisati ono o čemu pogled ovisi.

Ono što katalog *bilježi*, i što uvoz provjeri za vas kad završi:

| Prijavljeno | Odakle dolazi |
|---|---|
| ograničenja dodana kao `NOT VALID` | `pg_constraint.convalidated` - postojeći retci nikad nisu provjereni |
| neispravni indeksi | `pg_index.indisvalid` - `CREATE INDEX CONCURRENTLY` koji nije završio |
| isključeni okidači | `pg_trigger.tgenabled` - ostali isključeni nakon prekinutog vraćanja s `--disable-triggers` |
| strani ključevi koji pokazuju izvan sheme | oni koje vraćanje samo jedne sheme ne može ponijeti |

Uz svaki nalaz ide naredba koja ga popravlja. Kad nema što prijaviti, to i kaže - 
tišina je ovdje rezultat, a ne izostali odgovor. Točne naredbe koje su pale ostaju
u zapisu posla, dostupnom kroz **Poslove u pozadini**.

#### Nova baza je zadani cilj {#restore-into-a-new-database}

Dijalog za vraćanje otvara se na **(nova baza)**, nazvanoj po onoj iz koje
si ga otvorio s dodanim datumom; baza se stvara prazna i dump se vraća u
nju, pa se ništa postojeće ne prepisuje - pod bilo kojim imenom koje joj
daš, iz bilo kojeg formata. Baza iz koje je dump napravljen može ostati gdje
jest, u istom klasteru.

To vrijedi i za običnu SQL datoteku napravljenu s `--create`, što nije uvijek
bilo tako. Takva datoteka stvara svoju bazu i prebacuje se u nju
(`CREATE DATABASE shop`, `\connect shop`), a psql sluša nju, ne bazu u kojoj
je pokrenut: pokrenuta kakva jest nije vratila ništa u novu bazu, a datoteka
napravljena s `--create --clean` najprije je obrisala bazu iz koje je došla
i ponovno je izgradila iz datoteke - izmjereno, redovi dodani nakon dumpa su
nestali. Dijalog sada čita datoteku prije vraćanja i kaže kad to radi;
vraćanje se tada pokreće iz kopije bez linija `DROP DATABASE`,
`CREATE DATABASE` i `\connect`, a naredbe o samoj bazi (njezine postavke,
komentar i prava) usmjerene su na bazu koju si odabrao. Tvoja datoteka se ne
mijenja. Custom, tar i directory arhiva nikad nisu imale taj problem:
`pg_restore` ne dobiva `--create`, pa piše u bazu na koju je usmjeren.

Datoteka cijelog klastera (`pg_dumpall`) sadrži više baza, uloge i
tablespaceove i ne može se uliti u jednu bazu; odbija se s tim razlogom.
Vrati je psql-om na klaster koji nema nijednu od njezinih baza ili napravi
dump samo baze koja ti treba.

#### Vraćanje u bazu koja postoji {#restore-into-an-existing-database}

Postojeću bazu i dalje možeš odabrati s popisa; potvrda tada kaže što
sadrži - koliko tablica i pogleda, koliko je velika - prije nego što se išta
zapiše. Znaj što se tamo događa, jer to nije zamjena:

- **Bez --clean** ništa se u bazi ne zamjenjuje. Objekt koji postoji ostaje
  kakav jest - njegovo stvaranje ne uspije i to se prijavi - a redovi iz
  dumpa **dodaju se** u postojeće tablice. Tablica s primarnim ili
  jedinstvenim ključem s kojim se redovi iz dumpa sudare ne učita nijedan od
  njih; tablica bez ključa dobije ih još jednom. Izmjereno: tablica bez
  ključa narasla je s 3 na 5 redova.
- **S --clean** se objekti iz dumpa najprije brišu i ponovno stvaraju, pa
  njihove sadašnje redove zamijene redovi iz dumpa. Objekti kojih u dumpu
  nema ostaju netaknuti.

#### Tablespaceovi pri vraćanju {#restore-tablespaces}

Dump pamti tablespace svakog objekta koji nije bio u zadanom tablespaceu
svoje baze i vraća ga tamo. Dijalog navodi tablespaceove koje datoteka
imenuje i koje od njih poslužitelj nema. Onaj koji nedostaje ne preskače se
potiho: `pg_restore` bi te objekte stavio u zadani tablespace uz grešku za
svaki, a psql bi stao na prvom. Vraćanje se umjesto toga odbija s imenima, a
jedan klik uključuje **--no-tablespaces**.

**Tablespace nove baze** stvara novu bazu u tablespaceu koji odabereš. S
**--no-tablespaces** u njega ide i svaki objekt iz dumpa - svaka baza u svom
tablespaceu, što god je izvor koristio. Bez toga objekti zadržavaju
tablespaceove iz dumpa, a u tablespace nove baze ide samo ostatak.

Može biti i **novi tablespace**: daj mu ime i direktorij na poslužitelju baze - koji mora već postojati, biti prazan i pripadati korisniku pod kojim PostgreSQL radi - pa se najprije stvara tablespace, a onda baza u njemu. Za stvaranje tablespacea treba superuser.

#### Vraćanje pod drugim imenom {#restoring-into-a-different-name}

`pg_restore` ne može preimenovati shemu pri uvozu - imena iz dumpa su imena koja
dobivate. DBApguin se ne pravi da je drukčije. Rade dva puta:

1. **Vratite pa preimenujte.** Vratite dump kakav jest, pa desni klik na shemu →
   **Preimenuj shemu** (5.8). Ograničenja, sekvence i dodjele slijede
   preimenovanje sami.
2. **Vratite u pomoćnu shemu pa spojite.** Kad je ime koje želite već zauzeto i
   drži podatke koje namjeravate zadržati, vratite u pomoćnu shemu pa je s
   **Spoji shemu** (5.9) uklopite u onu živu, uz odluku po objektu i izbor što se
   događa sa sudarenim retcima.

Put 2 je onaj za živu bazu: spajanje ide u jednoj transakciji, pa sudar koji niste
očekivali ne mijenja ništa.

#### Format se čita iz datoteke, ne iz imena {#the-format-is-read-from-the-file-not-its-name}

Ostavite **Format** na *Prepoznaj iz sadržaja datoteke* i DBApguin gleda u
datoteku, a ne u njenu ekstenziju:

| Potpis | Format | Vraća se s |
|---|---|---|
| počinje s `PGDMP` | custom | `pg_restore` |
| `ustar` na odmaku 257 | tar | `pg_restore` |
| direktorij | direktorij | `pg_restore` |
| sve ostalo | plain SQL | `psql` |

Ovo je važno jer ekstenzija nije dokaz. Plain SQL dump spremljen kao
`backup.dump` je čest slučaj, i prije se predavao `pg_restore`-u, koji je
odgovarao *"input file appears to be a text format dump. Please use psql."* - 
točno, i bez ikakve pomoći nekome tko je samo htio vratiti svoju datoteku. Sad
rade oba smjera: SQL nazvan `.dump` i custom arhiva nazvana `.sql`.

Padajući izbornik i dalje dopušta da format zadate izrijekom kad želite zaobići
prepoznavanje.

#### Opcije vraćanja {#restore-options}

Vraćanje ima vlastiti skup: `--clean`, `--if-exists` (samo uz `--clean`, na
čemu `pg_restore` inzistira), `--no-owner`, `--no-privileges`,
`--single-transaction`, `--exit-on-error`, `--disable-triggers` (za vraćanje samo
podataka), `--data-only`, `--schema-only`, `--no-comments`, `--no-publications`,
`--no-subscriptions`, `--no-security-labels`, `--enable-row-security`, *Pokreni
kao rola*, paralelni poslovi, i - za sužavanje onoga što izlazi iz dumpa - **Samo
ove sheme**, **Samo tablice**, **Isključi sheme** i **Odjeljak**.

`--single-transaction` i paralelni poslovi se međusobno isključuju; odabir oboje
se odbija umjesto da se proslijedi.

**Običnu SQL datoteku `psql` izvodi točno onako kako je zapisana**: vlasnici,
prava i sve ostalo dolaze s njom, a staje na prvoj grešci. `psql` je može izvesti
kao jednu transakciju i kao drugu rolu, i te dvije stvari se primjenjuju; svaka
druga gornja opcija traži custom, tar ili directory dump, pa su za običnu SQL
datoteku isključene, a traženje neke od njih odbija se po imenu umjesto da se
zanemari.

Stvari o vraćanju *dijela* dumpa koje vrijedi znati, jer `pg_restore` nije
simetričan s `pg_dumpom`:

- **Imena se uspoređuju točno onako kako su u dumpu**, s velikim i malim
  slovima; rade i `sales.orders` i `orders`, a ime u dvostrukim navodnicima smije
  sadržavati točku ili zarez. Ime koje u arhivi ništa ne pogađa odbija se prije
  nego išta krene.
- **Samo tablice vraća cijelu tablicu.** Sam `pg_restore --table` donosi
  definiciju tablice i redke i ništa drugo: bez primarnih i stranih ključeva,
  indeksa, zadanih vrijednosti, okidača, vlastitih sekvenci i particija - tablica
  u stablu izgleda ispravno, a ponaša se kao da ništa od toga ne postoji. Zadano
  vraćanje uzima ono što bi `pg_dump --table` dumpao: particije na svim
  razinama, ključeve, indekse, zadane vrijednosti, okidače, politike, vlastite i
  identity sekvence s vrijednostima, komentare i prava - te funkciju koju okidač
  poziva ili sekvencu iz koje zadana vrijednost uzima, kad ih ova baza nema.
  *samo tablica i njezini redci* i dalje postoji i kaže što izostavlja.
- **Pri vraćanju nema "isključi tablice".** `pg_restore` nema `--exclude-table`.
  Isključite ih pri dumpu ili imenujte one koje želite. Traženje toga odbija se
  uz to objašnjenje, umjesto da alat vrati "unrecognized option".
- **Isključivanje sheme je svejedno stvara, praznu.** `--exclude-schema`
  preskače sadržaj sheme, ne `CREATE SCHEMA`. Praznu shemu obrišite poslije ako
  je ne želite.

#### Prije i poslije vraćanja {#before-and-after-a-restore}

Kad je datoteka odabrana, dijalog pročita što će vraćanje tražiti od cilja i kaže
što tamo nedostaje:

- **role** koje dump postavlja za vlasnike objekata ili im dodjeljuje prava - uz
  `--no-owner` / `--no-privileges` prestaju biti važne (ne za običnu SQL
  datoteku, koja ih nosi svejedno);
- **ekstenzije** koje dump stvara, a ovaj ih server ne može instalirati;
- **sheme** u kojima žive objekti dumpa, a koje ih on ne stvara - dump
  napravljen sa *Samo tablice* nema `CREATE SCHEMA` - uz ponudu da se najprije
  stvore, prazne;
- drugačiji **encoding** (u UTF8 se ništa ne gubi; u bilo što drugo znak za koji
  nema koda zaustavlja učitavanje njegove tablice) ili **locale** (tekst se
  sortira i indeksira redom ove baze).

Uz `--exit-on-error` ili `--single-transaction`, a uvijek za običnu SQL
datoteku, rola, ekstenzija ili shema koja nedostaje odbija vraćanje prije
početka: stalo bi na prvoj od njih, napola gotovo.

Kad vraćanje završi, cilj se uspoređuje sa samim dumpom: redci svake tablice
(prebrojani u podacima dumpa, ne procijenjeni), vrijednost svake sekvence, svaki
vlasnik i svako pravo koje bi dump postavio. Što se razlikuje, navede se. Izlazni
kod `pg_restorea` kaže samo je li zapisao greške; ovo kaže drži li baza ono što
je dump držao.

#### Kloniranje sheme bez constrainta {#cloning-a-schema-without-its-constraints}

**Constrainti → izostavi constrainte i indekse**, u Backupu i u Izvozu sheme,
dumpa tablice i retke i tu stane: bez primarnih ključeva, bez jedinstvenih i
stranih ključeva, bez indeksa.

To je jedno pokretanje `pg_dumpa` - `--section=pre-data --section=data`, što
alat prihvaća zajedno. pre-data drži definicije tablica, data drži retke, a sve
što ovo izostavlja živi u post-data. Provjereno na živom serveru: nastala
datoteka nema nijedan `ADD CONSTRAINT` ni `CREATE INDEX`.

Dva razloga da to poželite. Masovni uvoz ide brže bez provjera i održavanja
indeksa, a alat za replikaciju koji radi početnu sinkronizaciju u odredište
obično neće krenuti dok su ondje strani ključevi u primjeni. Oboje završava
isto: constraints manager napiše skriptu koja ih vraća, a vi je pokrenete kad
uvoz ili sinkronizacija završi.

Kvačica nadjačava izbornik **Section** umjesto da se s njim kombinira - dvije
kontrole za istu zastavicu inače bi dale dump koji nijedna od njih ne opisuje.

#### Timeouti kod dugog dumpa ili vraćanja {#timeouts-on-a-long-dump-or-restore}

Dump ili vraćanje koje umre na pola - pola sata unutra, a s datotekom je sve u
redu - obično je server koji ga prekida, a ne alat koji je pao.

To mogu učiniti četiri postavke:

- `statement_timeout` - jedna naredba je predugo trajala.
- `lock_timeout` - čekanje na zaključavanje je predugo trajalo.
- `idle_in_transaction_session_timeout` - transakcija je predugo stajala prazna.
- `transaction_timeout` (PostgreSQL 17 i noviji) - transakcija je predugo
  otvorena, radila ili ne.

`pg_dump`, `pg_restore` i dump učitan kroz `psql` prve tri sami maknu: na živom
serveru njihov log pokazuje `SET statement_timeout = 0`, `SET lock_timeout = 0`
i `SET idle_in_transaction_session_timeout = 0` prije ikakvog posla.
`transaction_timeout` ne miču - a upravo on prekida dugi posao, jer `pg_dump`
drži jednu REPEATABLE READ transakciju za cijeli dump, a `pg_restore
--single-transaction` jednu za cijelo učitavanje. Server s `transaction_timeout
= 30min` zaustavi oboje na trideset minuta koliko god zdravo radili.

Obična `.sql` datoteka koju nije napisao `pg_dump` je druga rupa: nema `SET`
zaglavlje, pa na nju vrijede sve četiri.

**Timeouti na serveru -> makni ih za ovo pokretanje**, u Backupu, Izvozu sheme i
Vraćanju, miče one koje odredište ima. Ide kroz `PGOPTIONS`, koje libpq
primjenjuje pri spajanju i koje nadjačava `ALTER DATABASE ... SET` ili `ALTER
ROLE ... SET`. `transaction_timeout` se šalje samo PostgreSQL-u 17 i novijem:
nepoznat parametar u `PGOPTIONS` se ne ignorira, spajanje se odbija s `FATAL:
unrecognized configuration parameter`, a to bi "vraćanje bi moglo biti
prekinuto" pretvorilo u "vraćanje ne može ni krenuti".

Redak ispod kućice javlja što odredište stvarno ima postavljeno. Ako je
odznačite, serverske postavke ostaju kakve jesu.

Ako je pokretanje već umrlo na taj način, poruka imenuje postavku: `canceling
statement due to statement timeout`, `terminating connection due to
idle-in-transaction timeout` ili `canceling statement due to transaction
timeout`.

Redak ispod kućice javlja ne samo vrijednost nego i **gdje je postavljena** - 
`statement_timeout = 30min (postavljeno na database sales)`, ili
`(postavljeno na role app_user)`. Vrijednost bez navedenog izvora došla je iz
konfiguracije samog servera: `postgresql.conf`, ili parameter grupa na Aurori i
RDS-u. To je razlika između toga da znate da vam nešto skraćuje posao i toga da
znate što promijeniti.

**Konkretno na Aurori i RDS-u.** `transaction_timeout` ne postoji prije
PostgreSQL-a 17, pa je na Aurori 15 ili 16 posao koji umre na okruglom broju
minuta jedno od ovoga:

- `statement_timeout` na parameter grupi, bazi ili roli - redak ispod kućice
  kaže na kojoj;
- **RDS Proxy**, ako veza ide kroz njega. Njegov `IdleClientTimeout` je po
  defaultu 1800 sekundi, što je točno trideset minuta, a klijenta koji čeka
  dugu naredbu broji kao neaktivnog;
- **NAT gateway ili vatrozid** između vas i servera. Jedna duga naredba - 
  gradnja indeksa na kraju vraćanja, COPY jedne ogromne tablice - dok traje ne
  šalje ništa ni u jednom smjeru, pa uređaj prekine tok, a naredba završi u
  utičnicu koju više nitko ne drži. AWS NAT gateway neaktivan tok gasi nakon
  350 sekundi. Aplikacija upravo zato postavlja TCP keepaliveove na vezu kad
  god host nije Unix socket, čime tok ostaje živ bez ikakvog vlastitog prometa.

Ništa u aplikaciji ne ograničava koliko dump ili vraćanje smije trajati.
Oboje se vrti kao odvojeni proces sa stanjem na disku, pa preživi zatvaranje ili
ponovno pokretanje aplikacije, i ni HTTP zahtjev ni prozor ih ne čekaju.

#### Praćenje dumpa ili restorea dok traje

`pg_dump` i `pg_restore` pripovijedaju što rade, a dijalog to prikazuje kako
stiže - od prvog retka do zadnjeg, ne samo zadnji redak. Kad odskrolate gore
da nešto pročitate, prestaje pratiti kraj; isključite *follow* da miruje.
**Copy** stavlja cijeli zapis na međuspremnik.

Posao radi odvojeno, pa ga zatvaranje dijaloga ne prekida, a zapis ostaje
dostupan pod *Background jobs*.

#### "errors ignored on restore" nije neuspio restore

`pg_restore` izlazi s greškom čim je **išta** ignorirao, a ispiše samo broj.
Daleko najčešća je `CREATE` nad objektom koji odredište već ima - restore
sheme u bazu koja tu shemu već ima pukne na `CREATE SCHEMA` i uredno vrati
sve ispod njega.

Zato dijalog izlistava same greške i kaže koje su od njih objekt koji je već
postojao. Kad su sve takve, kaže to izravno: ništa zbog njih nije preskočeno.
Prava greška i dalje se prijavljuje kao prava greška.

#### Dump iz novijeg PostgreSQL-a

Obični `.sql` dump počinje nizom `SET` naredbi, a noviji `pg_dump` piše
postavke koje stariji poslužitelj nikad nije čuo - `transaction_timeout`
stigao je s PostgreSQL-om 17. Vraćen u 16, taj redak zaustavi restore na
mjestu i ništa se ne vrati.

Aplikacija prvo pročita zaglavlje i odbije prije početka, imenujući postavku,
odakle datoteka dolazi i što je ovaj poslužitelj. Dumpovi putuju naprijed, ne
natrag: napravite dump s `pg_dump`-om koji odgovara odredištu, ili vratite u
poslužitelj barem toliko nov koliko je onaj s kojeg je došao.

#### Koji klijent se pokreće

`pg_dump`, `pg_restore` ili `psql` druge major verzije od servera nisu
zamjenjivi, a neslaganje boli u suprotnim smjerovima:

- `pg_restore` i `psql` **govore** serveru. Sesiju otvaraju blokom `SET`
  naredbi koje odlučuje njihova vlastita verzija, pa klijent noviji od servera
  šalje postavke o kojima server nikad nije slušao - `pg_restore` 17 šalje
  `transaction_timeout`, kojeg ništa ispod 17 nema, i onda svaki restore
  prijavi ignoriranu grešku i izađe s 1 na poslu koji je prošao.
- `pg_dump` i `pg_dumpall` **čitaju** server. Pitaju kataloge koje poznaju
  samo u svojoj verziji, pa se stariji od servera ne pokreće uopće:
  *"aborting because of server version mismatch"*.

Zato aplikacija pita server za verziju i bira kopiju koja mu odgovara, iz
verzioniranih direktorija koje distribucije drže jedan uz drugi
(`/usr/lib/postgresql/*/bin`, `/usr/pgsql-*/bin`, Homebrew, EDB instalateri),
kao i iz priložene kopije i onoga što je na `$PATH`. Za restore postoji i
drugo ograničenje - klijent stariji od `pg_dump`-a koji je napisao arhivu ne
može je pročitati - i to se ne rješava tablicom: kandidata se pita s
`pg_restore -l`, koji pročita indeks i ne obnovi ništa, a ako odbije, pokuša se
sljedeći major iznad.

Čitanje arhive natrag nema servera u sebi, samo datoteku, pa provjera backupa
uzima najnoviju instaliranu kopiju. To je koliko ispravak buga toliko i
profinjenje: zdravu arhivu koju čita `pg_restore` stariji od `pg_dump`-a koji
ju je napisao vraća *"unsupported version in file header"*, što izgleda točno
kao korupcija.

**Settings -> Tools** prikazuje putanju na koju se svaki alat razrješava i
popisuje ostale instalirane majore uz nju, jer prikazana putanja je zapravo
rezerva, a ne cijeli odgovor. Putanja koju sami postavite je apsolutna i
posve isključuje biranje - što je izlaz kad server traži klijenta kojeg
aplikacija ne bi odabrala.

#### Odabir datoteka

Svugdje gdje aplikacija čita ili piše datoteku - izvoz, dump, datoteka za
restore, uvoz konekcija, certifikat - otvara se dijalog vašeg sustava, s
njegovim mjestima, nedavnim mapama i pitanjem o prepisivanju. Na sustavu bez
takvog dijaloga aplikacija se vraća na vlastiti preglednik mapa i ništa nije
izgubljeno.

Dijalog se otvara ondje gdje su datoteke. `pg_dump` se izvršava na računalu
na kojem radi aplikacija i piše kamo to računalo može pisati, pa birač
pregledava taj datotečni sustav.

Spremanje uvijek pita za ime datoteke. Jedino mjesto koje umjesto toga pita za
mapu jest odredište sigurnosne kopije, jer je mapa ono u što `pg_dump` piše.

### 5.10.5 Particionirane tablice {#partitioned-tables}

Particionirani roditelji su u kategoriji **partitioned tables** u stablu sheme. Njihove leaf particije su obične tablice, pa bi inače zatrpale kategoriju **tables** - sto `events_2026_NN` redaka bez ičega što ih povezuje - zato ih stablo po defaultu sklanja. Redak `tables (12) · show 8 leaf partitions` ih vraća po shemi (izbor se pamti), a svaka prikazana nosi oznaku `part of <roditelj>`.

**Stvaranje**: dijalog *+ Table* ima sekciju **Partitioning** - odaberi RANGE, LIST ili HASH i ključ (stupac, lista `(col1, col2)`, ili izraz poput `(date_trunc('month', at AT TIME ZONE 'UTC'))`). Roditelj se stvara prazan; particije mu dodaješ naknadno. Svaki `PRIMARY KEY` ili `UNIQUE` na particioniranoj tablici mora sadržavati svaki stupac ključa particioniranja - PostgreSQL to provodi i greška je jasna ako se propusti.

**Upravljanje**: desni klik na particioniranu tablicu pa **Partitions…**. Dijalog otvara po strategiji i ključu i izlistava svaku particiju: granice, procjenu redaka, veličinu, tablespace kad ga ima svoj, i oznaku `detach pending` ako je concurrent detach bio prekinut (vidi dolje). Default particija je označena i ide zadnja; particija koja je i sama particionirana otvara se iz istog retka.

- **Create partition** - nova prazna particija zakačena u jednoj naredbi. Popuni granice koje strategija traži.
- **Attach existing table** - uvodi tablicu koju si već napunio. Napomena uz naredbu kaže hoće li PostgreSQL **puno skenirati dijete** (prikazuje broj redaka i veličinu) radi provjere granice, ili može biti samo metapodatak. Samo je metapodatak kad dijete već ima `CHECK` ograničenje koje dokazuje da svaki redak stane; ako nema, dodaj poklapajući `CHECK`, pokreni `ALTER TABLE ... VALIDATE CONSTRAINT` (to uzima samo `SHARE UPDATE EXCLUSIVE`, ne puni lock), pa pridruži.
- **Detach** - tablica prestaje biti particija i zadržava retke. **DETACH CONCURRENTLY** je označen po defaultu na PostgreSQL 14+: ne uzima `ACCESS EXCLUSIVE` lock na roditelju, pa upiti na ostalim particijama nastavljaju raditi. Izvodi se izvan transakcije i ne može se poništiti, a ako se prekine particija ostaje **detach pending** - redak tada pokazuje **Finalize detach** koji ga dovršava. PostgreSQL odbija `DETACH CONCURRENTLY` dok tablica ima DEFAULT particiju, i kvačica je onemogućena uz taj razlog; na PostgreSQL 13 i starijem je onemogućena jer značajka ne postoji, a običan detach drži isključivi lock tijekom svog (obično kratkog) trajanja.
- **Drop** - briše particiju i svaki redak u njoj.

Ništa se ne izvršava na klik. Dijalog prvo pokaže naredbu, uz redak koji kaže točno što zaključava, koliko dugo i što se događa ako se prekine, i izvrši je tek kad pritisneš *Run it*.

**Show DDL** za particioniranog roditelja emitira `CREATE TABLE ... PARTITION BY ...`; za leaf particiju emitira `CREATE TABLE ... PARTITION OF <roditelj> FOR VALUES ...` bez liste stupaca i samo s ograničenjima i indeksima definiranima na toj particiji.

**Pretvaranje između obične i particionirane tablice**: desni klik na tablicu pa **Convert to partitioned (recipe)…**, ili desni klik na particioniranu tablicu pa **Flatten to a plain table (recipe)…**. Nijedna opcija ništa ne pokreće. Pretvaranje je ponovna izgradnja (nova tablica, kopija svih redova, pa zamjena imena), pa dijalog prikazuje SQL skriptu koju kopiraš, pregledaš i pokrećeš sam. Skripta je jedna transakcija. Pisanje u tablicu čeka od prve naredbe do COMMIT-a, a prije COMMIT-a skripta uspoređuje broj redova, pa ako išta padne, tablica ostaje kakva je bila. Prenosi:

- stupce i zadane vrijednosti, generirane stupce te CHECK i NOT NULL ograničenja;
- brojače identity i serial stupaca, pa sljedeći id nastavlja gdje je stara tablica stala;
- ključeve i indekse pod izvornim imenima te okidače;
- vlasnika, ovlasti, komentare i politike row-level securityja;
- strane ključeve u oba smjera, poglede koji čitaju tablicu i publikacije.

Kod pretvaranja popunjavaš dva TODO-a:

- ključ particioniranja, koji mora biti dio svakog PRIMARY KEY i UNIQUE ograničenja;
- particije. DEFAULT particija je već u skripti, pa se tijekom kopiranja nijedan red ne odbija.

Ono što se ne može prenijeti navedeno je na vrhu pod *Read first*. Primjeri su NO INHERIT checkovi na particioniranoj tablici, EXCLUDE ograničenja prije PostgreSQL-a 17, materijalizirani pogledi i funkcije koje koriste redni tip tablice. Stara tablica ostaje kao `<ime>_old` dok je ne obrišeš.

### 5.10.6 Kloniranje sheme

Desni klik na shemu, pa **Transfer → Clone schema…**. Kopira shemu u novu na
istoj bazi: tablice, particionirane tablice, viewove, materijalizirane viewove,
sekvence, funkcije, procedure, agregate, tipove (enum, domena, kompozit, range),
kolacije, konverzije, strane tablice, indekse, ograničenja, okidače, pravila,
row level security s politikama, proširene statistike i po izboru retke.
**Merge schema** premješta istu širinu objekata; razlika je u tome da spajanje
objekte odnosi iz izvora, a klon izvor ostavlja nedirnut i stvara drugu kopiju.

Četiri vrste se ne mogu generirati i **imenuju se u rezultatu umjesto da budu
preskočene u tišini**: text search konfiguracije, rječnici, parseri i predlošci,
te operatori, operatorske klase i familije. Svaka se deklarira oblikom koji nosi
više nego što katalog vrati u jednom komadu - text search konfiguracija je parser
plus mapiranje po tipu tokena, operatorska klasa je popis operatora i pomoćnih
funkcija - a pola `CREATE`-a gore je od nijednog. Dijalog kaže `18 of 20
object(s) copied, 2 to create by hand` i navede koja su to dva, umjesto da
prijavi dvadeset.

Strani ključevi se grade tako da pokazuju na kopiju, a ne natrag na original - 
klon čiji ključevi i dalje referenciraju izvor nije kopija, nego druga vrata
u podatke prve sheme.

Uz *copy the rows too* redoslijed je bitan i riješen je za vas: objekti se
stvore bez stranih ključeva, retci uđu, pa se ključevi dodaju. Kopiranje
redaka u tablice koje već provode svoje reference pukne na onoj koja slučajno
ide prva. Sekvence se pomiču tamo gdje su originali stali, pa prvi insert u
kopiju ne udari u redak koji je došao s podacima. `IDENTITY` stupac nosi i
svoje postavke sekvence - `START WITH`, `INCREMENT BY`, `MINVALUE`,
`MAXVALUE`, `CACHE`, `CYCLE` - pa se kopija ne vraća tiho na zadane.

Redci se preuzimaju **točno onakvi kakvi jesu**, što god u njima bilo. Sve što
ocjenjuje ili mijenja redak dok se upisuje čeka da redci budu unutra:

- ograničenje koje izvor ima kao `NOT VALID` (`CHECK`, strani ključ, `CHECK`
  domene, `NOT NULL ... NOT VALID` iz PostgreSQL-a 18) dodaje se kao
  `NOT VALID` nakon redaka, pa se redci koje izvor nikad nije provjerio ne
  provjeravaju ni u kopiji, a novi redci da;
- okidači i pravila stvaraju se nakon redaka, pa nijedan ne odbija, ne mijenja
  i ne dodaje kopirane retke;
- sigurnost na razini redaka i njezine politike uključuju se nakon redaka;
- indeks koji je neuspjeli `CREATE INDEX CONCURRENTLY` ostavio neispravnim ne
  kopira se (u izvoru ništa ne osigurava, a ponovno izgrađen na istim redcima
  ponovno pada) - kao što ga ni `pg_dump` ne prenosi.

Sigurnost na razini redaka ne odbija čitanje, nego ga skraćuje. Ako se njezine
politike odnose na ulogu kojom ste spojeni (niste vlasnik tablice ili tablica
ima `FORCE ROW LEVEL SECURITY`, a niste ni superkorisnik ni uloga s
`BYPASSRLS`) ili ne smijete čitati neki njezin stupac, kloniranje **prije**
pokretanja navodi tablicu i razlog i ne kopira ništa, umjesto da kopira samo
retke koje vidite.

**Tablica po tablica** za svaku tablicu pokazuje kopirane retke, što je dodano
kao `NOT VALID` kao u izvoru, što je stvoreno nakon redaka i što nije
kopirano. Ako se redak ipak ne može kopirati, poruka navodi tablicu, stupac,
ograničenje, redak koji ne prolazi i retke u izvoru po primarnom ključu.
**Kloniraj tablicu**, **Kopiraj tablicu** i izbor *kopiraj* u **Spoji shemu**
preuzimaju retke na isti način.

**Show the script** ispiše cijelu skriptu prije nego išta krene. Sam klon
radi kao jedna transakcija: ili stigne cijel ili nikako. Izvodi se kao
poslužiteljski posao, pa se klon velike sheme s retcima - a to su minute
posla - ne gubi zatvorite li dijalog: **Background jobs** ga prikazuje dok
radi i čuva cijeli rezultat za kasnije.

Kad završi, dijalog nabroji provjere kroz koje je kopija prošla, umjesto
golog "Gotovo": nova shema se pročita natrag prije nego se transakcija
potvrdi, i svaka tablica, sekvenca, ograničenje i broj redaka usporedi se s
originalom.

Kloniranje u shemu koja već postoji se odbija. To je spajanje, a za to služi
[Spajanje dviju shema](#59-merging-two-schemas).

### 5.11 Promjena vlasnika svega u shemi {#changing-the-owner-of-everything-in-a-schema}

Desni klik na shemu → **Promijeni vlasnika svih objekata…**

Shema vraćena pod pogrešnom rolom, ili odlazak zaposlenika, ostavlja svaki
objekt u njoj u vlasništvu nekoga tko ga ne bi trebao posjedovati. Raditi to
objekt po objekt je u redu za tri objekta i neupotrebljivo za tristo.

Odaberite novog vlasnika. Dijalog popisuje svaki objekt u shemi koji ima
vlastitog vlasnika, grupiran kao u stablu sheme: tablice, particionirane tablice
s particijama ugniježđenima ispod na svim razinama, poglede, materijalizirane
poglede, strane tablice, sekvence, funkcije, procedure, agregate, tipove,
domene, tipove raspona, operatore, klase i obitelji operatora, kolacije,
konverzije, konfiguracije i rječnike za pretraživanje teksta, objekte
statistike, te samu shemu, označenu. Ispod popisa su točne naredbe
`ALTER ... OWNER TO` za ono što je označeno; ništa se ne izvršava dok ne
pritisnete gumb. Objekti koje novi vlasnik već posjeduje prikazuju se i ostaju
netaknuti.

Particije se navode jedna po jedna jer PostgreSQL svakoj particiji daje
vlastitog vlasnika: promjena vlasnika particionirane tablice ne mijenja njezine
particije. Označavanje ili odznačavanje particionirane tablice čini isto sa
svime ispod nje.

Strani poslužitelji i publikacije koje koriste tablice sheme pripadaju cijeloj
bazi, ne shemi. Navedeni su na kraju, neoznačeni; označite ih ako i oni trebaju
prijeći.

Neki se objekti ne mijenjaju zasebno, i dijalog kaže koji:

- sekvenca koja pripada stupcu tablice (serial ili identity) seli se sa svojom
  tablicom, a PostgreSQL odbija promijeniti je zasebno. Kad njezina tablica
  već ima novog vlasnika, a sekvenca nema, ništa je ne bi preselilo: serial
  sekvenca tada je na popisu s naredbama koje je odvežu, predaju i ponovno
  vežu uz isti stupac; identity sekvenca ne može se odvezati, pa je prikazana
  iznad popisa s dvije naredbe koje je sele preko njezine tablice i s onim što
  one stoje;
- objekti koji pripadaju ekstenziji zadržavaju vlasnika ekstenzije;
- indeksi uvijek imaju vlasnika svoje tablice.

Primjena ide u **jednoj transakciji**. Napola prepisana shema teža je za
razumijevanje od one koju nitko nije dirao, a izvana se ne može vidjeti gdje je
stalo. Sama shema mijenja se prva, kako bi novi vlasnik mogao stvarati u njoj
kad mu se predaju njezini objekti. Rezultat prikazuje naredbe koje svakom
objektu vraćaju prethodnog vlasnika, za slučaj da promjenu treba poništiti.

Nakon toga stari vlasnik u ovoj shemi zadržava samo ono što bi ostavio i
PostgreSQL-ov REASSIGN OWNED: zadane privilegije koje je ta rola postavila za
shemu.

**Jedan objekt.** Desni klik na objekt → **Promijeni vlasnika…** (za bazu u
izborniku baze). Dijalog prikazuje trenutnog vlasnika i točnu naredbu prije
nego išta krene. Za particioniranu tablicu kaže i koliko particija ima, na
svim razinama, i tko ih posjeduje, te nudi **Promijeni vlasnika i njezinih
particija (N)**. Kvačica je isključena kao i u PostgreSQL-u: tablica mijenja
vlasnika, a svaka particija zadržava svog. Ako ostane isključena, rezultat kaže
koliko je particija zadržalo vlasnika.

### 5.12 Usporedba shema između poslužitelja {#comparing-schemas-across-servers}

Desni klik na shemu → **Usporedi s drugom shemom**. Otvara se dijalog
usporedbe s tom shemom odabranom na strani A; odaberite shemu i na strani B i
usporedit će te dvije sheme. Ostavite obje na *cijela baza* i usporedit će baze.

Strana B ima vlastiti birač **poslužitelja** s vašim spremljenim vezama, pa
usporedba može ići između dva različita klastera - razvojni protiv produkcije je
uobičajen razlog. Odabir poslužitelja ponovno učitava popis shema tog
poslužitelja, jer sheme na jednom klasteru ne govore ništa o shemama na drugom.

Usporedba `sales` na jednom klasteru sa `sales` na drugom je posve normalna
želja, pa zaštita "ista shema" vrijedi samo kad su obje strane na istoj vezi.

Rezultat je razlika na razini objekata plus **sync skripta** koja bi B izjednačila
s A. Ništa se ne izvršava samo od sebe: vidi [Izvršavanje na B](#running-it-on-b).

#### A i B jedno uz drugo {#a-and-b-side-by-side}

Svaki redak koji se razlikuje - objekt u shemi, vlasnik i prava sheme,
ekstenzija, svojstvo, rola, pretvorba tipa (cast), particije particionirane
tablice - otvara se na mjestu kad ga kliknete (ili pritisnete njegovu strelicu:
Tab dolazi do nje, Enter ili razmaknica je otvara). Dva okna pokazuju definiciju
iz A lijevo i iz B desno, s brojevima redaka:

- redak koji ima samo A je crven, redak koji ima samo B je zelen;
- redak koji imaju obje strane, ali nije isti, stoji nasuprot svom paru, s
  označenim riječima koje se razlikuju;
- dulji dio koji se nije promijenio sklapa se u *N nepromijenjenih redaka*;
  kliknite ga da vidite te retke;
- **Kopiraj A** i **Kopiraj B** kopiraju tekst te strane;
- ispod dva okna su naredbe koje skripta za usklađivanje piše za taj objekt -
  iste koje piše **Generiraj sync skriptu**, s trenutno postavljenim opcijama.

U uskom prozoru okna se slažu jedno ispod drugog, A iznad B. Definicija od više
tisuća redaka otvara se odmah: crtaju se samo promjene i po tri retka oko njih,
dok ne otvorite sklopljeni dio.

**Usporedba baza** - meni Baza podataka, ili desni klik na bazu u bočnoj traci →
Upravljanje → Usporedi s drugom bazom - radi isto za cijele baze i prima popis
shema kojima se usporedba ograničava. To je bitno na velikoj bazi: obilazak
objekata po shemi je skupa polovica, pa usporedba jedne sheme od četrdeset ne bi
smjela koštati kao usporedba svih četrdeset.

Usporedba baza pokriva i ono što ne pripada nijednoj shemi: publikacije,
pretplate, strane poslužitelje, event triggere i pretvorbe tipova koje je netko
napravio s CREATE CAST (ugrađene i one koje donosi ekstenzija su izostavljene).
Pretvorba koja se razlikuje ima kvačicu kao shema ili rola, a skripta za
usklađivanje je briše i stvara ponovno - pretvorba ne drži podatke i ne može se
mijenjati.

Popisi baza i shema na obje strane mogu se u svom padajućem popisu poredati: A-Ž, Ž-A ili najveće prvo. Veličine baza su točne; veličine shema procjena su iz statistike planera (zadnji VACUUM ili ANALYZE), koja ništa ne košta i ne čeka ni na jedno zaključavanje. Stavka za cijelu bazu ostaje na vrhu u svakom poretku.

#### Izvršavanje na B {#running-it-on-b}

Svaki redak za koji skripta piše naredbe ima kvačicu, označenu; redak za koji
ne piše ništa (broj redaka, napomena) ima kvačicu koja se ne može označiti. U usporedbi baza kvačica sheme označava njezine objekte. Ispod
rezultata tri radnje rade s naredbama označenih redaka, redom kojim ih skripta
ima:

- **Probni rad...** izvršava ih na B u jednoj transakciji koja se uvijek
  poništava. Svaka naredba radi pod svojom točkom spremanja i javlja se
  zasebno: izvršena, neuspjela s porukom PostgreSQL-a ili preskočena. Ništa se
  ne zadržava, ali dok traje uzima ista zaključavanja kao primjena.
- **Primijeni na B...** izvršava ih na B u jednoj transakciji: sve ili, kod
  prve greške, nijednu, s imenovanom naredbom koja nije uspjela i onima nakon
  nje označenima kao neizvršene. Svaka naredba čeka svoje zaključavanje najviše
  5 sekundi.
- **Izvezi changeset...** sprema ih kao Flyway migraciju
  (`V<vrijeme>__<ime>.sql`) ili Liquibase changelog (formatirani SQL ili YAML,
  jedan changeset po naredbi, s id-jem i autorom), gdje odabereš.

Prije probnog rada ili primjene pregled pokazuje, pročitano sa samog B:

- svaku naredbu sa zaključavanjem koje uzima i na kojem objektu - ACCESS
  EXCLUSIVE zaustavlja svako čitanje i pisanje te tablice do kraja
  transakcije, SHARE pušta čitanja - i briše li, odvaja ili prepisuje nešto;
- naredbe koje se ne mogu izvršiti unutar bloka transakcije (sve s
  CONCURRENTLY, ALTER TYPE ... ADD VALUE prije PostgreSQL 12, ALTER SYSTEM):
  preskaču se i navode, da ih izvršiš zasebno;
- kopiju: prije primjene definicije svakog objekta na B koji naredbe mijenjaju
  ili brišu spremaju se kao SQL koji ih vraća; izvještaj vodi do nje, a
  Poslovi u pozadini je čuvaju. Ono što ne zna zapisati - ulogu, cast, objekt
  statistike, kolaciju, objekte pretraživanja teksta, operatore - navodi s
  razlogom. Redovi nisu u kopiji: tablica ili stupac koji primjena briše iz nje
  se vraća prazan;
- što tvoja prijava ne smije na B, pitano B-a: objekt čiji je vlasnik uloga
  kojoj nisi član, shema u kojoj ne smiješ stvarati, uloga kojoj ne možeš
  predati objekt.

Kad se nešto briše, odvaja ili prepisuje, **Primijeni na B** čeka da upišeš
ime sheme B (dvije sheme) ili baze B (dvije baze).

Nakon primjene to dvoje se odmah ponovno uspoređuje; izvještaj ostaje iznad
novog rezultata, a on je ono što je preostalo. Primjena se izvršava samo na B s
kojim je rezultat uspoređen: usmjeriš li dijalog drugamo, traži da ponovno
usporediš. Svaki probni rad i primjena ulaze u audit log sa svojim naredbama;
primjena koja je poništena zapisuje se kao neuspjela, s naredbom koja nije
uspjela.

#### Predaja usporedbe dalje

**PDF izvještaj** pretvara ono što je na ekranu u dokument: s kojeg je
poslužitelja i kada, brojevi, tablica onoga što se razlikuje i skripta za
usklađivanje. Gradi se iz istog odgovora koji ekran prikazuje, pa se to dvoje
ne može razići.

Datoteka se zapisuje ondje gdje odabereš, i to iz aplikacije a ne iz
preglednika, a postojeći izvještaj nikad se ne prepisuje bez pitanja.

Jedno ograničenje vrijedi znati: izvještaj koristi fontove koje ima svaki PDF
čitač, a oni pokrivaju njemački i španjolski u cijelosti, ali ne i hrvatske č,
ć i đ. Ta se slova preslovljavaju - „sifra", a ne prazno mjesto - pa je ime
čitljivo i kad kvačica otpadne.

#### Što znače četiri statusa {#what-the-four-statuses-mean}

Cijeli rezultat usporedbe je jedna riječ po retku, pa vrijedi biti precizan. Oba
dijaloga uspoređuju **strukturu, ne podatke** - definicije, svojstva i ovlasti.
Nijedan redak se ne čita ni s jedne strane, osim ako zatražite i trenutno stanje
(niže), i ni na jednu se ništa ne piše dok označene retke ne primijeniš na B.

| | |
|---|---|
| **same** | Postoji s obje strane i identično je. Shema kojoj se svi objekti podudaraju skrivena je dok ne odznačite *differences only*. |
| **changed** | Postoji s obje strane pod istim imenom, ali dvije definicije nisu isti tekst: dodan stupac, prošireni tip, drugačije tijelo funkcije, svojstvo postavljeno na drugu vrijednost. Kliknite redak da vidite A i B jedno uz drugo (vidi [A i B jedno uz drugo](#a-and-b-side-by-side)). |
| **only in A** | Postoji u izvoru, a ne u odredištu. Sync skripta ga stvara na B. |
| **only in B** | Postoji u odredištu, a ne u izvoru. Sync skripta ga **miče**, jer izjednačiti B s A znači ukloniti ono što A nema. |

Smjer nije simetričan: A je izvor, B je odredište, i svaka skripta iz ovih
dijaloga mijenja B da izgleda kao A.

#### Ograničavanje onoga što se uspoređuje {#restricting-what-is-compared}

**Što uspoređivati** drži filtere, a oni jednako vrijede za usporedbu i za
skriptu koju ona generira - usporedba koja ignorira indekse ne smije proizvesti
skriptu koja ih dira.

- **Vrste objekata**: tablice, pogledi, materijalizirani pogledi, sekvence,
  funkcije. Isključivanje jedne čini usporedbu slijepom za nju.
- **Unutar tablice**: stupci, indeksi, strani ključevi, primarni ključevi,
  jedinstveni constrainti, check constrainti. Tako dobijete usporedbu koja gleda
  samo strane ključeve, ili onu koja ignorira razlike u indeksima - korisno kad
  su dva okruženja namjerno različito indeksirana.
- **Po imenu**: popis za uključivanje i popis za izuzimanje, oba primaju `*`.
  `audit_*` u polju za izuzimanje drži usporedbu na onome što vas zanima.
- Šest presetova radi uobičajene kombinacije jednim klikom: sve, samo tablice,
  samo strani ključevi, samo sekvence, struktura bez indeksa, pogledi i funkcije.
- **Dopusti DROP naredbe** odlučuje smije li sync skripta ukloniti ono što
  postoji samo u B. Odznačite i skripta samo stvara i mijenja, što je sigurna
  postavka kad B nije vaše da ga potkresujete.
- **Premjesti tablice u tablespace i access method iz A** je isključeno dok ga
  ne uključite: premještanje tablice prepisuje cijelu tablicu pod ACCESS
  EXCLUSIVE zaključavanjem. Isključeno, razlika se i dalje prikazuje, a skripta
  kaže zašto je ostavlja.
- **Računaj drugačiji redoslijed stupaca kao razliku** je isključeno.
  PostgreSQL ne može premjestiti stupac pa nijedna naredba ne može poredati
  stupce u B kao u A. Isključeno, tablica s istim stupcima drugim redom
  uspoređuje se kao *same* i nosi napomenu - *stupci drugim redom* - koja
  prikazuje oba redoslijeda, onaj u B kakav će biti nakon skripte (stupac koji
  skripta dodaje završava na kraju). Uključeno, redoslijed se računa, a skripta
  kaže da tablicu treba ručno izgraditi iznova da se poklopi.

#### Particije, nasljeđivanje i pohrana {#partitions-inheritance-and-storage}

Particija se uspoređuje sa svojim mjestom u stablu: roditeljem i granicom
(`FOR VALUES ...`), a particionirana tablica sa svojim ključem particioniranja.
Skripta za usklađivanje premješta particije s **ATTACH PARTITION** i **DETACH
PARTITION**, nikad brisanjem i ponovnim stvaranjem, jer particija drži retke:

- promijenjena granica: odvaja se pa ponovno pripaja s granicom iz A;
- particija koju A ima pripojenu, a B je drži kao zasebnu tablicu: pripaja se;
  obrnuto: odvaja se;
- particija pod drugim roditeljem: odvaja se od roditelja u B, pripaja roditelju
  iz A;
- particija koju ima samo B, pod roditeljem kojeg imaju obje strane: odvaja se i
  ostaje kao zasebna tablica. Briše se poslije samo kad je uključeno **Dopusti
  DROP naredbe**.

Svaki DETACH ide prvi, u svom odjeljku, pa granica napisana niže ne može se
preklopiti s particijom koja je još pripojena. Zaglavlje skripte navodi
zaključavanja: DETACH uzima ACCESS EXCLUSIVE na roditelju i particiji; ATTACH
uzima SHARE UPDATE EXCLUSIVE na roditelju (ACCESS EXCLUSIVE prije PostgreSQL-a
12) i čita particiju da dokaže da njeni retci stanu u granicu. Na PostgreSQL-u
14 i novijem iza svakog DETACH stoji njegov oblik `... CONCURRENTLY` kao
komentar: uzima slabije zaključavanje, ali ne može se pokrenuti unutar
transakcijskog bloka ni kad roditelj ima DEFAULT particiju, pa ga pokrenite
zasebno ako ga koristite.

Ključ particioniranja ne može se promijeniti na mjestu. Kad A i B tablicu
particioniraju različito, skripta to kaže i za nju ne piše ništa izvršno, a ono
što A stavlja pod tu tablicu čeka, kao komentar, dok se ključ ne riješi ručno.

**INHERITS.** Tablica koja nasljeđuje drugu (klasično nasljeđivanje, ne
particija) uspoređuje se sa svojim roditeljima. Skripta piše `ALTER TABLE ...
INHERIT` ili `NO INHERIT`, a novo dijete stvara se samo sa svojim stupcima,
nakon svog roditelja.

**Tablespace i access method** dio su definicije tablice: tablica u drugom
tablespaceu, ili pohranjena drugim access methodom, uspoređuje se kao changed.
Nova tablica stvara se ondje gdje je ima A; postojeća se premješta samo s
prekidačem iznad.

**Stupac kojem se mijenja tip ispod pogleda.** PostgreSQL odbija `ALTER COLUMN
... TYPE` dok stupac čita pogled, materijalizirani pogled ili pravilo. Skripta
svaki od njih na B briše neposredno prije promjene - i poglede izgrađene na
njima, u bilo kojoj shemi, najvanjskiji prvi - i stvara ih ponovno odmah nakon
nje, kakvi su na B: definicija i opcije, indeksi i reci materijaliziranog
pogleda (REFRESH), zadane vrijednosti stupaca i INSTEAD OF okidači pogleda,
pravila, vlasnik, komentari, prava i prava na stupcima. Svaki DROP i CREATE
drži ACCESS EXCLUSIVE zaključavanje na svom objektu do kraja transakcije, pa
upiti na njima čekaju; pregled prije izvršavanja označava svaki takav DROP kao
onaj koji se vraća. Ono što se odavde ne može vratiti - okidač čiji WHEN čita
stupac, politika, generirani stupac, tijelo funkcije - skripta navodi kao
ručni posao prije nego se pokrene.

**Stupac koji se briše ispod pogleda.** PostgreSQL odbija `DROP COLUMN` dok
stupac čita pogled, materijalizirani pogled ili pravilo. Skripta svaki od njih
na B briše neposredno prije stupca - i poglede izgrađene na njima, najvanjskiji
prvi - a nakon njega svaki postaje ono što od njega napravi ostatak skripte:
onaj koji postoji samo na B ostaje obrisan; onaj koji A definira drukčije, bez
tog stupca, stvara se kakav je u A, s vlasnikom, pravima i komentarima s B (i
komentarima stupaca, pravima na stupcima, pravilima, okidačima i indeksima koji
ne spominju izgubljeni stupac), a njegova vlastita promjena ne piše se niže još
jednom; svaki drugi vraća se kakav je na B. Pregled označava svaki DROP koji se
vraća. Pogled koji bi se vratio i dalje čitajući stupac - u drugoj shemi, onaj
koji skripta ne dira, onaj izgrađen na pogledu koji se mijenja ili nestaje - ne
može se vratiti, kao ni okidač čiji WHEN čita stupac, politika ili tijelo
funkcije: tada skripta kaže što se prvo mora promijeniti i taj stupac ne briše
(DROP piše kao komentar), jer DROP koji ne uspije poništava cijelo izvršavanje.
Promijenite taj objekt i usporedite ponovno.

#### Vlasnik i prava {#owner-and-grants}

Prekidači **vlasnik** i **prava** odlučuju dvije stvari zajedno: računa li se
razlika u tome tko je vlasnik objekta ili tko ga smije koristiti - i piše li
skripta za usklađivanje ALTER ... OWNER TO, GRANT i REVOKE koji je zatvaraju.
Oba su uključena; isključite jedan i ta se vrsta razlike niti prikazuje niti
upisuje u skriptu. Prava uključuju prava na stupce, prava na sekvencu identity
stupca, ono što je vlasnik oduzeo sam sebi te prava same sheme i zadana prava
postavljena u njoj, prikazana kao jedan redak imena *(this schema)*. Pravo se
čita onako kako ga PostgreSQL primjenjuje: objekt na kojem nitko nikad nije
dodijelio prava i onaj na kojem su prava dodijeljena pa opet oduzeta
uspoređuju se jednako. Kad skripta imenuje rolu koja na cilju ne postoji,
zaglavlje skripte kaže koje su to role.

#### Usporedba i trenutnog stanja {#comparing-the-current-state-too}

Usporedba je po definiciji: dvije su sheme iste kad bi se svaki objekt u njima
stvorio na isti način. **Usporedi i trenutno stanje** dodaje ono što baza koja
radi ima, a definicija nema. Svaki je prekidač isključen dok ga ne uključite, jer
se dvije kopije iste sheme u stanju uvijek razlikuju.

| | |
|---|---|
| **trenutne vrijednosti sekvenci** | Gdje je svaka sekvenca i je li već pozvana. Skripta ih na B postavlja sa setval(). |
| **je li materijalizirani pogled napunjen** | Skripta pokreće REFRESH MATERIALIZED VIEW, ili REFRESH ... WITH NO DATA kad je onaj u A prazan. |
| **broj redaka** | Tablice, particije i materijalizirani pogledi. Procjena dolazi iz statistike plannera i računa se kao razlika kad su dva broja 5% ili više udaljena; **točan broj redaka** pokreće count(*) na svakoj tablici, čita svaki redak i uspoređuje točno. |
| **veličine** | Ukupna veličina, s indeksima i TOAST-om. Razlikuju se kad su barem 1 MB udaljene i to ili 10% ili 64 MB. |
| **valjanost i korištenje indeksa** | Indeks koji je ostao nevaljan nakon neuspjelog CREATE INDEX CONCURRENTLY, i koliko je puta svaki indeks korišten: razlika je kad ga je jedna strana koristila, a druga nikad, ili kad su dva broja udaljena više od 1000. |

Broj redaka, veličine i korištenje indeksa samo se prikazuju i nikad ne ulaze u
skriptu - nijedna naredba skripte ne bi ih izjednačila - a skripta završava
popisom objekata koji se u njima razlikuju. U usporedbi baza particije
particionirane tablice koje se razlikuju navedene su imenom, ne samo brojem.

#### Usporedba onoga što rola smije, na dvije strane {#comparing-what-a-role-may-do-on-two-sides}

**Usporedi korisnike** odgovara na pitanje "može li ova rola ovdje isto što i
ondje", i pita tim redom: koje role, odakle, pa koliko duboko gledati.

Role su u PostgreSQL-u na razini clustera - jedan `pg_authid` po clusteru - pa
je popis rola ono što dva *servera* imaju, a rola koje na jednoj strani nema se
javlja, ne preskače. Ovlasti su obrnute: žive u katalogu svake baze, i zato su
izbornici baze bitni i zato je usporedba po bazi. Imenovanje shema sužava
dalje; ako je ostavite prazno, uspoređuju se sve sheme koje veza vidi.

**PUBLIC** - svaka rola, sadašnja i buduća - ima svoj redak u popisu rola. Nema
atribute za usporedbu, samo prava: **compare privileges** na tom retku pokazuje
što je dodijeljeno PUBLIC-u na svakoj strani, a skripta za usklađivanje piše
GRANT ... TO PUBLIC i REVOKE ... FROM PUBLIC koji to zatvaraju.

Čita se sedam razina, a prva je ona o kojoj sve ostalo ovisi:

| | |
|---|---|
| **database** | `CONNECT`, `CREATE`, `TEMPORARY` - može li rola uopće otvoriti vrata |
| **schema** | `USAGE`, `CREATE` |
| **table** | te pogledi, materijalizirani pogledi, particionirane i strane tablice |
| **column** | rola može imati `SELECT` na tri stupca i ništa na tablici, što svaka provjera na razini tablice čita kao "nema pristupa" |
| **sequence** | `USAGE`, `SELECT`, `UPDATE` |
| **function** | `EXECUTE` |
| **default** | što će nositi *novi* objekti - rola može biti ispravno ovlaštena na svemu što danas postoji i izgubiti pristup svemu što nastane sutra |

Ovlasti se čitaju izravno iz ACL-ova u katalogu, ne kroz `has_table_privilege`.
Ta funkcija odgovara na "može li rola X" uključujući sve naslijeđeno kroz
članstvo, a naslijeđena ovlast se ne može dodijeliti ni oduzeti na toj roli.
Djelovati se može samo na izravnu ovlast, pa se samo one i uspoređuju; samo
članstvo se uspoređuje kao atribut role.

**Sync skripta** izjednačava jednu stranu s drugom, u smjeru koji odaberete.
Nijedna strana nije automatski ispravna - produkcija može biti ona koja je
odlutala, ili ona koja je točna - pa se nude oba smjera i nijedan nije zadan
tako da sakrije izbor.

Ovlasti koje ima odredište a nema izvor pišu se kao komentari dok ne označite
*također oduzmi*. Dodati ovlast je popravljivo; oduzeti je nekome usred radnog
dana nije.

Primjena iz usporedbe baza prvo pokazuje naredbe. **Primijeni** je ondje prije
izvodila dodjele na klik i tek onda javljala broj; sada otvara isti prozor koji
dobiva svaka generirana skripta - kopiraj, spremi, otvori u SQL editoru - s
Primijeni pokraj njih, i izvodi samo ono što ti je pokazano. Promijene li se
ovlasti između gledanja i primjene, primjena stane i preda ti novi popis umjesto
da izvede nešto drugo.

Ništa se ne izvršava. Pročitajte skriptu, spremite je s **Spremi skriptu…** i
pokrenite kad odlučite.

### 5.13 Kloniranje baze {#cloning-a-database}

Desni klik na bazu → **Kloniraj bazu (isti poslužitelj)**.

To je jedna naredba - `CREATE DATABASE "kopija" TEMPLATE "izvornik"` - a
PostgreSQL bazu kopira na razini datoteka, umjesto da je dumpa i vraća. Zato je
brzo: baza od 25 MB klonira se u dobrano manje od sekunde, a vrijeme ovisi o
veličini, ne o broju objekata.

Kopija je potpuno neovisna od trenutka kad postoji. Pisanje u jednu ne utječe na
drugu; ne dijele prostor.

#### Cijena, i zašto dijalog inzistira

Baza ne može služiti kao `TEMPLATE` dok je na nju išta spojeno, pa prije
kopiranja DBApguin pokreće:

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'izvor'
```

**Svaka otvorena veza prema izvoru se ubija**, uključujući tuđe, a svaka
transakcija u tijeku se prekida. To je zahtjev PostgreSQL-a, ne odluka
DBApguina - i zato dijalog traži da označite kvačicu, a ne da samo prokliktate.

**S njima ide i vaša veza.** DBApguinova sesija na izvornoj bazi prekida se kao
i svaka druga, a pool tiho otvara novu za sljedeći zahtjev - pa aplikacija radi
dalje i ništa ne izgleda krivo. Ono što ne preživi jest sve vezano uz staru
sesiju: otvorena transakcija u SQL editoru se poništava, privremene tablice
nestaju, `SET` na razini sesije vraća se na zadano, savjetodavni lockovi se
otpuštaju, a upit koji je u tom trenutku tekao prekida se. Potvrdite ili
dovršite ono što radite prije kloniranja.

#### Što prelazi, a što ne

Sve *unutar* baze se kopira. Svojstva vezana *uz* bazu ne - ona pripadaju
katalogu klastera, a ne datotekama koje se kopiraju:

| Kopira se | Ne kopira se |
|---|---|
| tablice, retci, indeksi, ograničenja | **vlasnik** baze - klon pripada onome tko ga je klonirao |
| sekvence, na trenutnim vrijednostima | `GRANT CONNECT` / `TEMPORARY` nad bazom |
| pogledi, materijalizirani pogledi, funkcije, tipovi | postavke po bazi (`ALTER DATABASE … SET work_mem`, `search_path`, …) |
| ekstenzije | komentar na bazi |
| ovlasti nad tablicama i stupcima | role i korisnici (vrijede na razini klastera i već postoje) |
| komentari na objektima, RLS politike, veliki objekti | |

Desni stupac je ono što ljude iznenadi. Produkcijska baza klonirana za testiranje
stigne u vašem vlasništvu, bez `CONNECT` dodjela za rolu aplikacije, i radi na
poslužiteljevim zadanim `work_mem` i `search_path` umjesto na onima koje je imao
izvornik. Podaci su identični; okolina oko njih nije.

Za vraćanje te četiri stvari, nad klonom:

```sql
ALTER DATABASE kopija OWNER TO izvorni_vlasnik;
GRANT CONNECT, TEMPORARY ON DATABASE kopija TO rola_aplikacije;
ALTER DATABASE kopija SET work_mem = '64MB';
COMMENT ON DATABASE kopija IS '…';
```

#### Kad ga koristiti

- prije migracije ili rizične promjene, da postoji točno stanje kojem se možete vratiti
- za testnu ili razvojnu kopiju na istom poslužitelju
- **ne** za premještanje baze na drugi poslužitelj - `TEMPLATE` radi samo unutar
  jednog klastera; za to koristite backup i vraćanje
- **ne** na bazi koju ljudi upravo koriste - bit će odspojeni
- klon zauzima jednako diska kao izvornik; ništa se ne dijeli

### 5.14 Preimenovanje baze {#renaming-a-database}

Desni klik na bazu -> **Rename database**.

To je jedna naredba, `ALTER DATABASE "staro" RENAME TO "novo"`, i mijenja
**samo unos u katalogu**. Ništa se ne kopira ni premješta, pa je trenutna bez
obzira na veličinu baze. Vlasnik, svaki `GRANT`, komentar i sve postavke po
bazi (`ALTER DATABASE ... SET ...`) ostaju vezane i nepromijenjene - ime je
jedino što je poslije drugačije.

#### Ne može raditi dok je itko spojen

PostgreSQL odbija `ALTER DATABASE ... RENAME` dok je bilo koja sesija spojena
na tu bazu - uključujući vlastiti connection pool DBApguina ako ste je
pregledavali. Bez pomoći naredba pukne s:

```
database "shop" is being accessed by other users
DETAIL: There is 1 other session using the database.
```

Zato dijalog prvo provjeri. Ako je baza u mirovanju, rename samo prođe. Ako
nije, dijalog pokaže **koliko je konekcija i koji ih korisnici drže** i ponudi
da ih prekine - isti `pg_terminate_backend(...)` koji koristi Clone. To je
destruktivno: otvorena transakcija na toj bazi se poništava, privremene
tablice i `SET` na razini sesije nestaju, a upit koji se tada izvršava se
prekida. Morate potvrditi kvačicom.

Vlastite pool konekcije DBApguina prekidaju se zajedno s ostalima; pool
jednostavno otvori nove prema novom imenu na sljedeći zahtjev. Konekcija koja
izvodi rename je na bazi `postgres`, nikad na meti, pa rename ne može sam sebi
presjeći granu.

#### Vaša spremljena konekcija i otvoreni tabovi

Ako preimenujete bazu na koju je **ovaj prozor trenutno spojen**, DBApguin
ažurira spremljeni unos konekcije na novo ime, prebaci aktivnu konekciju i
ponovno učita stablo objekata. Otvoreni SQL i schema tabovi vezani uz staro
ime označe se zastarjelima sa značkom dok ih ne osvježite, isto kao pri svakoj
promjeni baze. Spremljena konekcija za *drugi* prozor koja je također
pokazivala na staro ime ne dira se - ažurirajte je ručno.

#### Što ne popravlja

Rename je promjena kataloga i ništa izvan kataloga za nju ne zna: aplikacija
sa starim imenom baze u connection stringu, unos u `~/.pg_service.conf`,
backup skripta, definicija `dblink` ili `postgres_fdw` servera, ili SQL koji
hardkodira ime. `current_database()` u funkciji vraća novo ime od trenutka
kad se rename commita. Preimenujte produkcijsku bazu samo kad možete ažurirati
sve što je doseže u istom prozoru za održavanje.

`postgres`, `template0` i `template1` ne mogu se preimenovati - dijalog ih
odbija - jer initdb i mnogo alata pretpostavljaju da ta imena postoje.

### 5.15 Ekstenzije i što radi "premjesti" {#extensions-and-what-move-does}

**Baza podataka -> Upravljaj ekstenzijama** popisuje što poslužitelj ima na
raspolaganju i što je u ovoj bazi instalirano, uz instaliranu verziju pokraj
zadane, pa se kandidat za `ALTER EXTENSION ... UPDATE` vidi na prvi pogled.

Ekstenzija je paket: funkcije, operatori, tipovi, ponekad tablice, sve
instalirano u jednu shemu. **Premjesti** pokreće

    ALTER EXTENSION "ime" SET SCHEMA "cilj"

i cijeli paket premješta jednom naredbom.

Zašto biste:

- **Da `public` ostane vaš.** `CREATE EXTENSION pg_trgm` bez sheme ubaci nekoliko
  desetaka funkcija u prvu shemu iz `search_path`, obično `public`, izmiješane

  s vašim tablicama.
- **Sigurnost.** Tko god može stvarati objekte u shemi, može definirati funkciju
  istog imena i istih tipova argumenata kao funkcija ekstenzije. Ako ta shema
  dolazi ranije u `search_path`, izvodi se njihova verzija. Ekstenzije u
  namjenskoj shemi `ext` u koju piše samo vlasnik tu mogućnost uklanjaju. Zato
  uobičajeni recept glasi `CREATE SCHEMA ext; CREATE EXTENSION pg_trgm SCHEMA ext;`

Što neće napraviti:

- **Ekstenzije koje se ne mogu premjestiti ne premještaju se.** Ekstenzija to
  sama deklarira (`pg_extension.extrelocatable`), obično zato što njezin kod
  imenuje vlastitu shemu. PostGIS je poznat primjer. DBApguin takve označava s
  `(non-relocatable)` i ne nudi gumb Premjesti, umjesto da pusti poslužitelj da
  odbije.
- **Ne mijenja vaš kod.** Sve što izrijekom imenuje staru shemu - tijelo pogleda,
  funkcija, upit aplikacije - prestaje raditi. Nekvalificirani pozivi rade dalje
  samo ako je nova shema u `search_path`:
  `ALTER DATABASE app SET search_path = public, ext;`
- **Nije nadogradnja verzije.** Za to koristite Ažuriraj.

Premještanje traži vlasništvo nad ekstenzijom, što u praksi znači superusera.
Naredba je brza - prepisuje unose u katalogu, ne podatke - ali uzima lock na
objekte ekstenzije, pa je radite kad nad njima ništa ne radi.

### 5.16 Lista zadataka

Alarm, preporuka advisora i nalaz provjere svi kazu "netko bi ovo trebao
napraviti" pa nestanu - alarm kad uvjet zatreperi, advisor kad se panel zatvori,
notifikacija kad ju maknete. Klaster s ID-evima transakcija starim 1,2 milijarde i devet tablica preko
granice je posao za jedno jutro, ne obavijest koja nestane.

**View -> Lista zadataka**, s brojem otvorenih na samom meniju; broj postaje
crven kad je neki od njih kritican.

Stvari dolaze na listu s tri mjesta:

- **iz notifikacije** - svaka alarm kartica ima *Dodaj u listu zadataka* uz
  *Otvori*;
- **iz prozora kriticnog alarma** - svaki alarm ondje ima svoj gumb, jer
  potvrdivanje znaci "vidio sam ovo", a to nije isto sto i "ovo je rijeseno";
- **iz bilo kojeg ocijenjenog advisor retka** - predite misem i pojavi se mala
  ikona liste.

Zadatak mozete i jednostavno upisati. Upisani zadatak je vas, i nikad se ni s cim
ne spaja niti dobiva oznaku da je zastario.

Svaki zadatak biljezi server, bazu i objekt iz kojeg je dosao, pa i tjedan dana
kasnije kaze o cemu se radilo. **Lozinke se nikad ne spremaju** - konekcija se
cuva kao host, port, baza i korisnik, samo za prepoznavanje.

Dodavanje istog nalaza dvaput je normalno, jer alarm jos gori; postojeci zadatak
se osvjezi umjesto da se udvostruci.

**Zadatak cijeg uvjeta vise nema to i kaze.** Kad alarm iza njega prestane gorjeti
na serveru s kojeg je dosao, zadatak dobiva oznaku *uvjet vise ne postoji* i rub
mu postaje zelen. Ostaje na listi - brisati tudi zadatak jer se ocitanje
promijenilo bilo bi gore - samo prestaje tvrditi da je aktualan. Tvrdnja je
namjerno uska: samo za zadatak nastao iz alarma, na konekciji pred vama, uz
poklapanje hosta i porta.

**Izvezi checklistu** zapisuje listu kao obican tekst s `[ ]` kvadraticima,
objektom, serverom i vasom biljeskom - nesto sto se moze isprintati, zalijepiti u
ticket ili odnijeti na racunalo koje nema ovu aplikaciju.

### 5.17 Foreign serveri i user mappinzi {#foreign-servers-and-user-mappings}

**Database -> Manage foreign servers** popisuje svaki foreign server u ovoj
bazi, wrapper ispod njega, ownera, njegove opcije i user mappinge koji vise o
njemu.

Kreiranje foreign servera bio je wizard; sve nakon toga bio je SQL editor.
Remote host se preseli, port se promijeni, zapisana remote lozinka istekne,
server treba predati drugom owneru - sve obično, ništa od toga dohvatljivo s
mjesta koje je server i napravilo, a opcije koje je wizard tražio nisu se nigdje
prikazivale natrag. Ovo je ta druga polovica.

#### Od čega se foreign server sastoji

Tri objekta, i sva tri se moraju poklopiti prije nego ijedan red prijeđe:

| objekt | što nosi | čiji je |
|---|---|---|
| wrapper | `postgres_fdw`, `mysql_fdw`, `file_fdw`, ... | ekstenzije |
| server | gdje je druga strana: host, port, dbname | baze |
| user mapping | ko si tamo: remote user i lozinka | jedne lokalne role |

Mapping je dio koji se zaboravlja. PostgreSQL ga traži **po roli koja se
stvarno spojila**, pa je server s mappingom samo za `postgres` nevidljiv svima
ostalima: njihovi upiti padaju na *"user mapping not found"* kako god server
bio ispravan. Zato se mapping nudi za bilo koju login rolu na clusteru, a ne
samo za sebe i `PUBLIC` - jedna lokalna rola na jedan remote login je normalan
oblik, i razlog zašto su remote kredencijali uopće po roli.

Mapping **za NOLOGIN grupnu rolu** server prihvaća i nikad ne koristi, jer se
takva rola nikad ne spaja. Te role, i ugrađene `pg_*`, namjerno nisu u listi.

#### Uređivanje servera

**Edit** prikazuje opcije koje server trenutno nosi, svaku kao uredivo polje,
uz mjesta za ključeve koje još nema. Šalje se samo ono što promijenite.

**Prazna vrijednost opcije briše ključ.** PostgreSQL ne poznaje opciju s praznim
stringom, a `OPTIONS (SET k '')` je druga naredba od toga da ključa nema, pa je
prazno polje način da kažete "izbriši ovo".

Treba li ključu `ADD`, `SET` ili `DROP` ovisi o tome nosi li ga server već, a to
se čita iz kataloga u istoj transakciji kao i `ALTER`, ne nagađa iz onoga što je
ekran zadnje pokazao. Promjena četiri opcije izađe kao jedna naredba:

    ALTER SERVER "loopback" OPTIONS (SET host '10.0.0.9', SET port '5433',
                                     DROP dbname, ADD connect_timeout '7')

**Version** i **Owner** su namjerno odvojene `ALTER SERVER` naredbe. PostgreSQL
ih prihvaća u jednoj, ali onda jedan loš ključ opcije sruši i promjenu ownera, a
greška ne imenuje ni jedno ni drugo. Sve idu u jednoj transakciji, pa polovično
primijenjena promjena nije stanje u kojem možete završiti.

#### Lozinke

Remote lozinka mappinga se nikad ne prikazuje. PostgreSQL klijentu ne vraća
zapisanu lozinku, a opcija servera nazvana `password` je ovdje maskirana iz istog
razloga - uključujući u SQL-u koji se vraća i u audit logu, a to je datoteka koju
ljudi izvoze i šalju dalje. Polje za lozinku u uređivaču mappinga zato je prazno
i putuje samo ako nešto u njega upišete.

#### Brisanje

Server koji još ima foreign tablice odbija obični `DROP`, pa dijalog izričito
nudi `CASCADE` i prebroji user mappinge koji odlaze s njim. Cascade odnosi i
svaku foreign tablicu na tom serveru - to su lokalni katalozi, pa se na drugoj
strani ništa ne dira, ali upiti i viewovi koji čitaju kroz njih se lome.

**Drop user mapping** je uži rez: ta rola ostaje bez kredencijala za server i
njeni upiti padaju dok mapping ponovno ne postoji, a svaka druga rola nastavlja
raditi.

## 6. Uobičajeni tijekovi rada

### 6.1 Trijaža performansi na sporoj bazi

1. Otvorite vezu.
2. Prvo pokrenite `db-health` - pregled na jednom ekranu.
3. Ako je pogodak predmemorije nizak: `db-cache-advisor` za detalje po objektu.
4. Ako prevladavaju seq scanovi: `db-index-advisor` za kandidate za indeks.
5. Ako su upiti spori: `db-stat-statements` za prvih 25 s oznakama po upitu. Sortirajte po total_exec_time ili uzmite upite s oznakom UPOZ.
6. Ako sumnjate na napuhanost: `db-bloat-breakdown` za HEAP+TOAST+indekse po tablici. Za točan (spor) pregled koristite `db-pgstattuple`.

### 6.2 Problem sa zaključavanjem

1. `db-health` -> kategorija BLOCKING, pa u detalje za parove blokator/čekatelj.
2. Ili otvorite prikaz Sesije -> Popis i sortirajte po nepraznom blocking_pids.
3. Desni klik na sesiju blokatora pa Ubij kad ste spremni.

### 6.3 Kašnjenje replikacije

1. `db-replication` za stanje replike i kašnjenje pisanja/flusha/replaya u vremenu i bajtovima.
2. `db-repl-slots` za zadržavanje WAL-a po slotu. Neaktivni slotovi koji drže gigabajte WAL-a klasičan su uzrok punjenja diska.
3. Za logičku: `db-logical-replication` (strana pretplatnika) prikazuje ISKLJUČENE / NEPOKRENUTE pretplate i kašnjenje primjene >=16 MB.
4. `db-replica-identity` ako se pretplatnici žale na nedostajući identitet.

### 6.4 Sigurnosna revizija

1. `db-roles-list` za sigurnosne nalaze na razini klastera.
2. `db-role-perms` (odaberite rolu koju revidirate) za detaljan uvid po cilju.
3. `db-ownership` za nalaze o vlasništvu nad objektima.
4. Izvezite revizijski zapis iz Postavke -> meni Prikaz -> Revizijski zapis.

### 6.5 Sigurna migracija sheme

1. Usporedite dvije baze kroz prikaz Poslužitelj -> Usporedba shema.
2. Pregledajte generirani sync SQL.
3. Primijenite odabrane naredbe.

### 6.5.5 Bi li ovaj indeks stvarno pomogao?

`db-index-advisor` javlja koje se tablice čitaju sekvencijalno. Sljedeće je
pitanje uvijek bi li indeks na *ovom* stupcu to riješio, a jedini pošten
način da se odgovori bio je sagraditi ga - dugo zaključavanje pisanja za
odgovor koji je često "planer ga ionako ne bi upotrijebio".

Ni `EXPLAIN` na to ne odgovara. Planer košta indekse koji postoje, pa bez
indeksa nema plana s njim, a `EXPLAIN ANALYZE` ga iz istog razloga treba
stvarnog.

Ispod advisorove tablice je panel koji odgovara bez gradnje. Odaberite
tablicu, napišite indeks koji razmatrate, zalijepite upit koji vas zanima i
pritisnite **Compare plans**. Javlja:

- bi li planer indeks uopće upotrijebio - čest odgovor je ne, i to vrijedi
  znati prije nego sagradite indeks od 4 GB;
- procijenjeni trošak sa i bez njega, i razliku;
- otprilike koliko bi prostora indeks zauzeo.

Namjerno ne javlja vrijeme. Ništa se ne izvršava, pa mjerenja nema, a
prikazati procjenu kao mjerenje bilo bi gore nego ne dati ništa.

Za ovo treba ekstenzija `hypopg`, koja indeks stvara samo u planerovoj glavi - 
ništa se ne zapisuje i nikakvo zaključavanje se ne uzima, pa je sigurno i na
produkciji. Dostupna je na RDS-u i Aurori; kod samostalnog postava dolazi kao
zaseban paket. Ako ekstenzija nije instalirana, panel to kaže, i kaže može li
je ovaj poslužitelj dobiti.

### 6.6 Analiza opterećenja u Oracle stilu (AWR + ASH + praćenje) {#oracle-style-workload-analysis-awr-ash-trace}

#### Kada radi: plan snimanja {#capture-plan}

Tabovi AWR, ASH i Praćenje sesije pokreću se jednako: jedan redak **Kada** ispod zaglavlja.

- **Snimka sada** (samo AWR): jedna snimka, s neobaveznim komentarom koji stoji uz nju u svakom popisu.
- **Neko vrijeme**: počinje sada i sam se zaustavlja. Uz brze duljine stoji kotačić dana, sati, minuta i sekundi, pa se može odabrati bilo koja duljina u granicama alata: AWR od 10 minuta do 92 dana, ASH od 1 minute do 31 dana, praćenje od 10 sekundi do 30 minuta.
- **Zakazano**: od jednog datuma i vremena do drugog, odabrano biračem datuma i vremena po satu ovog računala, po želji **svaki dan**. Praćenje se zakazuje početkom i duljinom, najviše dan unaprijed.
- **Do zaustavljanja**: ono što su raspoređivač i uzorkivač radili i prije (ne za praćenje).

**Svakih** je koliko često alat skuplja, s vlastitim brzim izborima i istim kotačićem: AWR snimka svakih 1 minutu do 1 dan, ASH uzorak svakih 1 sekundu do 1 sat, praćenje pogled svakih 0,1 sekundu do 1 minutu. **Čuvaj** je koliko se dana spremljene snimke ili dani čuvaju. Kod AWR-a **uzorkuj i sesije svakih 5 s** pokreće ASH uzorkivač za isti prozor; za plan do jednog sata umjesto toga daje izvještaj opterećenja (snimka na svakom kraju i sesije uzorkovane između), nekadašnji prikaz Capture.

Iznad gumba jedna rečenica kaže što će se dogoditi, koliko je to snimki ili uzoraka, koliko zauzima na disku i košta na poslužitelju te što se otvara na kraju; upozorava kad je izbor težak (AWR snimka češće nego svakih 5 minuta). Dok plan radi, panel se sklapa u jednu traku: što radi, od i do kada, traka napretka, preostalo vrijeme, **Promijeni** i **Stop**; zakazani plan kaže kada počinje i nudi **Pokreni odmah**. Kad plan s krajem završi, izvještaj njegova razdoblja otvara se sam: za AWR prva snimka prema posljednjoj, za ASH izvještaj njegova prozora.

Kod AWR-a **Komentar** uz redak Kada postoji za svaki način: ide na jedinu snimku *Snimka sada* i na svaku snimku plana. Birač datuma i vremena kod *Zakazano* ima brza vremena unaprijed (za 1 h, noćas 02:00, sutra 08:00, ponedjeljak 00:00) i kotačiće za sat i minutu; strelica na kraju retka Kada sklapa panel u jedan red na malim ekranima.

**Stop** ranije završava plan i radi kao njegov vlastiti kraj: prikupljeno ostaje, AWR u tom trenutku snima posljednju snimku (s komentarom plana) i otvara se izvještaj razdoblja - prva snimka pokretanja prema toj posljednjoj, odnosno za ASH vrijeme koje je uzorkovao. Zakazani plan koji još nije počeo samo se otkazuje.

Planovi rade dok je DBApguin otvoren: na poslužitelju ništa ne ostaje i ništa ne radi samo. Plan usred svog prozora kad se aplikacija zatvori nastavlja za ostatak kad se poslužitelj sljedeći put otvori, dnevni čeka svoj sljedeći prozor, a onaj čiji je prozor prošao dok je aplikacija bila zatvorena je gotov. Zakazano praćenje živi samo dok je aplikacija otvorena.

Na Macu trackpad otkucava dok kotačići prolaze vrijednostima, kao birač na telefonu.

1. **Provjerite postavke jednom**: pokrenite `db-setup-advisor`. Ako `pg_stat_statements`, `auto_explain` ili `compute_query_id` javljaju NEDOSTAJE, slijedite ispisano rješenje. To radite jednom po klasteru.

2. **Snimite sporo razdoblje**: kad se korisnik žali na sporost, pokrenite `db-workload-report` na 5 minuta. Odjeljci s najskupljim SQL-om pokazuju što je stvarno trošilo vrijeme u tom prozoru.

3. **Trajna opservabilnost**: kad želite satima povijesti na razini sesija (a ne jednu petominutnu snimku), pokrenite ASH uzorkivač:
   - `db-ash` action=start
   - pustite ga da radi u pozadini (traje dok ne kliknete action=stop ili ne zatvorite aplikaciju)
   - kasnije pokrenite `db-ash` action=report za agregaciju bilo kojeg UTC raspona datuma

4. **Usporedba prije i poslije deploya**: uzmite snimku prije deploya, drugu kad se opterećenje smiri, pa ih usporedite:
   - `db-awr-snapshot` action=take, comment="prije deploya v24.07"
   - (deploy)
   - `db-awr-snapshot` action=take, comment="poslije deploya v24.07"
   - `db-awr-snapshot` action=list (označite obje, pa **Usporedi dvije odabrane**)
   - `db-awr-snapshot` action=compare, snap_id_a=<prije>, snap_id_b=<poslije>

5. **Detaljno praćenje jedne aplikacije**: kad je jedna aplikacija s poolom spora, a sumnjate na određeni upit:
   - `db-trace-session` target_mode=application_name, target_value=`myapp-worker`, duration_seconds=120, category_filter=`select`, query_filter=`reports`
   - U usporednom psql-u spojenom kao taj korisnik: `SET auto_explain.log_min_duration = 0; SET auto_explain.log_analyze = on;` pa pokrenite opterećenje
   - Izvještaj praćenja prikazuje prijelaze upita i događaje čekanja za svaki podudarni backend; planove hvata server log

#### Kako čitati ASH izvještaj {#reading-an-ash-report}

`db-ash` action=report pretvara uzorke odabranog prozora u šest odjeljaka. Prva dva su vlastito čitanje izvještaja; tablice iza njih su brojevi.

**Nalazi** su ono što se isticalo, najgore prvo, svaki s naredbom iza sebe i istim gumbima koje svaka naredba ovdje nosi: kopiraj, učitaj u SQL uređivač, EXPLAIN tamo, ili **Plan** - plan koji se stvarno izvršio, nacrtan na mjestu (vidi [Plan koji se stvarno izvršio](#the-plan-that-actually-ran)). "Prikaži u vremenskoj traci" osvjetljava redak te sesije u sljedećem odjeljku.

| Nalaz | Okida se kad | Što učiniti |
| --- | --- | --- |
| Blokiranje: sesija N | jedna sesija zadržala je druge 3 uzorka ili više; kritično od izgubljene minute ili pet zadržanih sesija | Tekst kaže je li blokator bio neaktivan u transakciji (aplikacija je zaboravila commit) ili je izvršavao naredbu (dugo pisanje). "Lanac" znači da je blokator i sam čekao na drugoga. |
| Jedna naredba uzela je X% aktivnog vremena | naredba ima 20% ili više uzoraka koji se izvršavaju ili čekaju, a bilo je drugih naredbi za usporedbu | Nalaz kaže kamo je otišlo njezino vrijeme: uglavnom brave, uglavnom disk ili uglavnom CPU - svako upućuje na drugi popravak. |
| X% aktivnog vremena bila su čekanja na brave | 20% ili više aktivnih uzoraka bila su `Lock` čekanja | Odjeljak parova blokiranja imenuje sesije. |
| X% čekanje na disk | 30% ili više bila su `IO` čekanja | Radni skup veći od `shared_buffers` ili jedno skeniranje koje prazni cache; top upiti kažu koje naredbe. |
| Unutarnja borba (LWLock) | 20% ili više bila su `LWLock` čekanja | `WALWrite` je WAL disk, `buffer_content` / `buffer_mapping` su vruće stranice, `lock_manager` je vrlo mnogo brava po transakciji. |
| Sesija neaktivna u transakciji do T | sesija je sjedila neaktivna unutar transakcije 5 minuta ili dulje | Drži svoje brave i koči vacuum za cijeli poslužitelj; kritično kad je usput i nekoga blokirala. `idle_in_transaction_session_timeout` takve sesije prekida. |
| Naredbe su odrezane | tekst uzorkovane naredbe dosegnuo je `track_activity_query_size` | Iz odrezanog teksta ne može se napraviti plan; odjeljak s postavkama na kraju kaže što povećati. |

**Vremenska traka** je prozor kao mreža: jedan redak po sesiji (četrdeset najzaposlenijih, ostale zbrojene u jedan redak "ostale"), jedan stupac po odsječku vremena - sekunda za kratak prozor, minuta ili više za dug, rečeno u naslovu. Svaka ćelija obojena je bojom onoga što je sesija najviše radila u tom odsječku:

| Boja | Značenje |
| --- | --- |
| zelena | izvršava se na CPU-u (aktivna, bez događaja čekanja) |
| crvena | čeka na bravu koju drži druga sesija |
| žuta | čeka na disk (`IO` događaji čekanja) |
| plava | drugo čekanje: `LWLock`, `IPC`, klijent, timeout |
| ljubičasta | neaktivna unutar transakcije - ne radi, ali drži svoje brave |
| prazna | nije u uzorcima: neaktivna ili nije spojena |

Blijeda ćelija je ona u kojoj je sesija bila samo dio odsječka. **Klikni ćeliju** za naredbe te sesije u tom odsječku, s događajem čekanja, pidovima koji su je blokirali (svaki je gumb koji skače na njihov redak) i gumbima naredbe. **Klikni oznaku sesije** za sve što je radila u prozoru: njezinu podjelu po bojama i zbrojene naredbe. **Klikni vrijeme** na vrhu za svaku sesiju u tom stupcu. Pidovi su iz prošlosti: backend je možda nestao ili je sada druga sesija, pa vremenska traka ne nudi prekid ni terminaciju.

**Histogram stanja**, **Događaji čekanja** i **Top sesije** su brojevi; top sesije sada nose i udjele brava, diska i neaktivnosti u transakciji svake sesije.

**Top upiti** su kartice, jedna po naredbi, poredane po uzorcima: njezin udio u aktivnom vremenu, traka s podjelom po bojama (kamo je otišlo njezino vrijeme), koliko ju je sesija izvršavalo i iz kojih aplikacija, najdulje pojedinačno izvršavanje, najčešći događaj čekanja i - kad je `pg_stat_statements` na poslužitelju - njezini pozivi, prosječno i ukupno vrijeme od resetiranja brojača. Gumbi su isti kao u nalazima; naredba odrezana na `track_activity_query_size` nema gumb Plan jer bi planer samo prijavio rez.

**Parovi blokiranja** imenuju obje strane: čekatelja i blokatora s njihovim aplikacijama, što je blokator radio dok je blokirao (`idle-txn` je onaj za pogledati), kad je par prvi i zadnji put viđen, bravu koju je čekatelj tražio i obje naredbe u cijelosti.

**Postavke za bolju analizu** zatvaraju izvještaj (a prikazuje ih i akcija status, prije ijednog uzorka): postavke poslužitelja koje odlučuju što uzorci mogu reći, svaka s trenutnom vrijednošću, preporučenom, treba li reload, restart ili CREATE EXTENSION, i zašto. Upozorenja su postavke bez kojih je ovaj izvještaj za nešto slijep - `compute_query_id` (bez identiteta naredbe u uzorcima), `track_activity_query_size` (odrezane naredbe), `pg_stat_statements` (bez brojača po naredbi), `track_io_timing` (vrijeme diska nevidljivo), `log_lock_waits` (bez zapisa o čekanjima kraćim od uzorka); ostale su dobrodošle (`auto_explain`, jedini način da se vidi plan koji se *stvarno izvršio*, `log_min_duration_statement`, `log_temp_files`, `track_functions`). **Primijeni** izvršava `ALTER SYSTEM SET` i reload nakon uobičajene potvrde; postavka koja traži restart tako je označena i čeka sljedeće ponovno pokretanje. Bez superusera naredbe su tu za kopiranje. Navode i sve što treba planu koji se stvarno izvršio - auto_explain i kako zapisuje, log collector i odredište loga, ovlast za čitanje loga (GRANT, primijenjen ovdje kad ga ova prijava smije dodijeliti) i pg_store_plans (instalacija paketa prikazana je kao naredba za kopiranje) - svako sa svojom cijenom; vidi [Plan koji se stvarno izvršio](#the-plan-that-actually-ran).

**Prije nego ASH ili AWR počne prikupljati** (ASH start, AWR take i auto-start) aplikacija pita poslužitelj za isti popis. Kad nešto što čitaju nedostaje, prvo se otvori prozor: ekstenzije (`pg_stat_statements` - kreirana u ovoj bazi? u `shared_preload_libraries`? - i `auto_explain`), zatim svaka postavka koja nije na mjestu, s trenutnom vrijednošću, preporučenom, traži li reload, restart ili CREATE EXTENSION, i zašto. One bez kojih je izvještaj slijep su označene; one koje je dobro imati označavate sami. **Uključi odabrano** izvršava točno naredbe prikazane ispod popisa (dva retka koja dodaju u `shared_preload_libraries` prikazana su kao jedna naredba u kojoj završe), ponovno čita poslužitelj i kaže što sada čeka restart - aplikacija nikad ne restartira poslužitelj. Redak koji je klik postavio ostaje u popisu, s rezultatom, do **Provjeri ponovno**. **Ekstenzije…** zatvara prozor i otvara upravitelja ekstenzija; **Ipak pokreni** radi s onim što postoji. Svaka kartica pita jednom; kad ništa ne nedostaje, prozor se više ne otvara sam, a **Provjeri postavke...** pokraj Starta u ASH i AWR planu prikazuje ga u svakom trenutku, s onim što je dobro imati (poput vlastitih postavki `auto_explain`, koje postoje tek nakon restarta koji ga učita), a njegov Start pokreće plan. Ponovno pokretanje kartice (njezina kružna strelica) ponovno čita prikaz kartice; nikad ništa ne pokreće, ne zaustavlja i ne snima. Korisnik koji nije superuser dobiva naredbe za kopiranje. Prozor se isključuje u Postavkama (*Provjera prije pokretanja ASH / AWR*) ili vlastitom kvačicom u prozoru.

#### Pretraga povijesti upita {#query-history-search}

`db-query-history` je pretraga nad `pg_stat_statements`, unakrsno provjerena s ASH uzorkovačem radi vremenske oznake. Njeni parametri postavljaju se PRIJE checka, u istom dijalogu parametara koji koristi svaki parametrizirani check:

| Polje | Radi |
| --- | --- |
| Statement contains | Pretraga podniza teksta naredbe bez razlike velikih/malih slova (literali su uklonjeni - naziv tablice ili stupca obično je najkorisnije upisati). Prazno polje odgovara svemu. |
| Run by | Samo naredbe koje je izvršila ova rola. |
| Worst by | Po čemu sortirati - ukupno vrijeme (na što baza troši život), prosječno vrijeme (što je sporo po izvršenju), najgore pojedinačno izvršenje, broj poziva, ili vraćeni redovi. Ukupno i prosječno vrijeme odgovaraju na različita pitanja: upit koji traje 2 ms milijun puta nadmašuje onaj koji traje minutu dvaput. |
| Ignore faster than (ms) | Izbacuje retke čije je prosječno vrijeme ispod ovoga, da stranica brzog šuma ne zatrpa onaj jedan spori oblik vrijedan čitanja. |
| How many | Ograničenje broja redaka, 1-200. |
| Include plans | EXPLAIN normaliziranog teksta za prvih 5 redaka - plan za oblik, ne zapis stvarno korištenog plana (vidi napomenu ispod). |

**Ova pretraga prolazi kroz cijelu povijest na serveru svaki put kad otvorite dijalog i kliknete Run check** - ne samo ono što je trenutno na ekranu. Tablica rezultata potom dobiva isti filter po stupcu koji dobiva svaki rezultat checka (upisivanje u zaglavlje stupca sužava već prikazane retke); taj filter samo sužava ono što je ova pretraga već vratila, ne pretražuje ponovno cijeli korpus. Da pretražite dalje unatrag ili po drugom kriteriju, promijenite polja iznad i pokrenite check ponovno.

Stupci rezultata:

| Stupac | Što je | Kako čitati |
| --- | --- | --- |
| `user` | Rola koja je izvršila oblik naredbe | - |
| `calls` | Broj izvršenja od zadnjeg resetiranja brojača | - |
| `total` / `mean` / `max` | Kumulativno / prosječno / najgore pojedinačno vrijeme izvršenja | Isto čitanje kao `total_ms`/`mean_ms` u workload izvještaju, iznad. |
| `rows` | Ukupno vraćenih ili pogođenih redaka kroz sva izvršenja | - |
| `ash_samples` | Koliko ASH uzoraka je uhvatilo `queryid` ove naredbe u izvršenju | 0 znači da ASH uzorkovač nije radio dok se ovo izvršavalo, ne da se nikad nije izvršilo - `calls` je brojka koja stvarno vrijedi. Treba prvo pokrenuti `db-ash`; PG13 uopće nema `queryid` u ASH-u. |
| `last_seen` | Vremenska oznaka zadnjeg ASH uzorka koji ga je uhvatio | Prazno iz istog razloga kao `ash_samples` = 0. |
| `query` | Normalizirani tekst naredbe, skraćen | Literali su uklonjeni u zamjenske oznake (`$1`, `$2`, ...) - jedan redak pokriva svako izvršenje koje se razlikovalo samo po vrijednostima. |

Brojači se resetiraju s `pg_stat_statements_reset()` ili padom servera; čist restart servera ih zadržava kad je `pg_stat_statements.save` uključen (defaultna postavka PostgreSQL-a). Naredba od prije zadnjeg resetiranja jednostavno nije tamo - "ništa nije pronađeno" može značiti i to, jednako lako kao da je uzorak bio pogrešan.

#### Kako čitati AWR / workload izvještaj {#reading-an-awr-report}

Svaki broj u ovim izvještajima je **delta kroz prozor** - vrijednost između dva snimka, ne stopa i ne zbroj od početka. Spremljeni izvještaj nosi ovaj rječnik sa sobom (PDF izvoz ga uključuje), pa se izvještaj otvoren tjednima kasnije i dalje sam objašnjava.

**Sažetak prozora**

| Metrika | Što je | Kako čitati |
| --- | --- | --- |
| `window_seconds` / `window_hours` | Stvarno vrijeme između dva snimka | Sve ostalo je "po ovoliko vremena". Prozor od 30 sekundi i prozor od 6 sati nisu usporedivi. |
| `txn_commit` / `txn_rollback` | Transakcije potvrđene / povučene u prozoru | Brojevi, ne stope. Podijelite s `window_seconds` za TPS. |
| `rollback_pct` | `txn_rollback / (txn_commit + txn_rollback)` | Iznad ~10% uz stvaran promet znači da aplikacija baca greške ili udara u deadlockove - pogledajte errors watcher i broj deadlockova. Nekoliko rollbackova u mirnom prozoru je šum. |
| `cache_hit_pct` | `blks_hit / (blks_hit + blks_read)` - udio čitanja blokova posluženih iz **shared_buffers**, ne s diska | Ovo je PostgreSQL buffer cache, ne OS page cache, pa "niska" brojka i dalje može biti brza ako je OS keširao. Ispod ~95% uz značajan promet obično znači da je `shared_buffers` premalen za radni skup, ili je jedan veliki sekvencijalni scan isprao cache tijekom prozora. |
| `deadlocks` | Deadlockovi otkriveni u prozoru | Svaka vrijednost iznad 0 je bug u redoslijedu zaključavanja u aplikaciji. PostgreSQL ga je razriješio ubijanjem jedne transakcije; par nađite u server logu. |
| `temp_files` / `temp_bytes` | Datoteke (i ukupna veličina) koje je PostgreSQL prelio na disk za sortiranja i hasheve | Vrijednost različita od nule znači da sortiranje ili hash upita nije stalo u `work_mem`. Usporedite s Top SQL po temp zapisima. |
| `new_sessions` / `sessions_per_s` | Sesije započete u prozoru (`pg_stat_database.sessions`, PostgreSQL 14+), i po sekundi | Mnogo kratkih sesija je izmjena veza: aplikacija bez poola veza, ili pool koji otvara i zatvara veze. Prije PostgreSQL 14 sažetak kaže da se to ne može izmjeriti. |
| `mean_session` | Vrijeme sesija nakupljeno u prozoru podijeljeno sa započetim sesijama | Dugovječne veze (iz poola) dodaju i svoje vrijeme, pa kratke sesije izgledaju dulje nego što jesu: mali broj jak je znak izmjene veza, velik ne dokazuje ništa. |

**Top SQL** (četiri rangiranja istih redaka: po ukupnom vremenu, po broju poziva, po čitanjima s diska, po temp zapisima)

| Stupac | Što je | Kako čitati |
| --- | --- | --- |
| `calls` | Izvršavanja ove normalizirane naredbe u prozoru | - |
| `total_ms` | Zbroj vremena izvršavanja kroz sve `calls` (delta `total_exec_time`) | Ovo je rangiranje koje najviše znači: upit s najvišim `total_ms` potrošio je najviše serverskog vremena u prozoru, bez obzira je li pojedinačno spor. |
| `mean_ms` | `total_ms / calls` - prosječno vrijeme jednog izvršavanja | **Velik `mean_ms`** (recimo > 100 ms) znači da je svako pokretanje sporo: nedostaje indeks, loš plan, čekanje na lock, ili genuinno velik upit - napravite EXPLAIN (ANALYZE, BUFFERS). **Malen `mean_ms` uz velik `total_ms`** znači da je upit jeftin po pozivu ali se pokreće toliko često da dominira - rješenje je zvati ga rjeđe (keširati rezultat, grupirati posao) ili smanjiti cijenu po pozivu, ne ugađati plan. |
| `rows` | Ukupno redaka vraćenih ili pogođenih kroz sve pozive | `rows / calls` je prosječna veličina rezultata; izvještajni upit koji vuče 100k redaka po pozivu je drugačiji problem od OLTP upita koji vuče 3. |
| `shared_read` | 8 KB blokova pročitanih **s diska** (ne iz cachea) kroz sve pozive | Visoko znači da podaci ovog upita nisu u `shared_buffers` - hladan cache, ili radni skup veći od cachea. `shared_read x 8192` je volumen u bajtovima. |
| `temp_written` | 8 KB blokova zapisanih u **temp datoteke** kroz sve pozive | Vrijednost različita od nule znači da se sortiranje ili hash prelilo. Podignite `work_mem` za sesiju koja ga pokreće, ili popravite plan da sortira manje redaka. `temp_written x 8192` je volumen u bajtovima. |
| `qid` | `queryid` (otisak normalizirane naredbe) | Ista vrijednost kroz snimke i u `pg_stat_statements`; koristite je za praćenje jedne naredbe kroz vrijeme. |

**Top tablice po DML-u**

| Stupac | Što je | Kako čitati |
| --- | --- | --- |
| `ins` / `upd` / `del` | Redaka umetnuto / ažurirano / obrisano u prozoru | Pokreće autovacuum. Tablica s puno `upd` + `del` i bez nedavnog autovacuuma je kandidat za bloat - vidi [Vacuum, bloat i statistika tablica](#vacuum-bloat-and-table-statistics). |
| `hot_upd_%` | HOT ažuriranja kao udio svih ažuriranja | HOT ažuriranja ne diraju indekse i ne trebaju čišćenje indeksa. **Nisko** (< ~50%) na jako ažuriranoj tablici znači da ažuriranja mijenjaju indeksirane stupce, ili stranice nemaju slobodnog mjesta - snizite `fillfactor` tablice da ažuriranja ostanu na stranici. |
| `seq_scan` / `idx_scan` | Sekvencijalni vs indeksni scanovi započeti na tablici u prozoru | Velika `seq_scan` delta na velikoj tablici je signal za nedostajući indeks - potvrdite s index advisorom. |

**Pozadinska / checkpoint aktivnost**

| Stupac | Što je | Kako čitati |
| --- | --- | --- |
| `ckpt_timed` / `ckpt_req` | Checkpointi okinuti rasporedom vs volumenom WAL-a | Većinom `req` znači da se checkpointi forsiraju volumenom pisanja prije `checkpoint_timeout` - podignite `max_wal_size` da se rašire. |
| `ckpt_write_ms` / `ckpt_sync_ms` | Vrijeme provedeno pišući / fsync-ajući prljave buffere tijekom checkpointa | Velik `sync_ms` znači da storage ne može apsorbirati navalu pisanja checkpointa. |
| `buffers_backend` | Prljavi bufferi koje su **backendi** morali sami zapisati | Netrivijalne vrijednosti znače da su background writer i checkpointer zaostali pa su backendi upita platili cijenu pisanja inline. |

**Nalazi** su vlastito čitanje prozora, najgore prvo, nacrtano kao nalazi ASH-a: svaki s naredbom iza sebe i njezinim gumbima. Osim gornjih pragova (deadlockovi, udio poništavanja, pogodak cachea) imenuju naredbu koja je uzela 30% ili više ukupnog vremena izvršavanja, naredbe s prosjekom preko 500 ms po pozivu, naredbe koje su prelile 64 MB ili više u temp datoteke (`work_mem`), checkpointe izazvane količinom WAL-a češće nego satom (`max_wal_size`), backende koji sami zapisuju petinu prljavih buffera ili više (`bgwriter_*`) i jako ažuriranu tablicu s manje od pola HOT ažuriranja (`fillfactor`). Svaki prag za naredbe ovdje traži da prozor ima barem deset sekundi ukupnog vremena izvršavanja, pa tiha testna baza ne izgleda kao poslužitelj u nevolji. Četiri nalaza sude se umjesto toga po vlastitom obujmu - čisti N+1 teret, brzo dohvaćanje izvršeno stotine tisuća puta bez ičeg sporog pored, nikad ne dosegne deset sekundi i prije je glasio "Ništa se nije isticalo u ovom prozoru":

- **Uzorak 1+N**: jedna naredba (B) izvršava se k puta za svako izvršavanje druge (A), u istoj bazi kao isti korisnik, uz k barem 5 i B izvršen barem 1.000 puta. Sam omjer mogao bi biti slučajan, pa treba i jedan dokaz: A je vratio (unutar 5%) onoliko redaka koliko se puta izvršio B - jednom po retku; ili se omjer držao unutar 10% u barem tri intervala između snimki dok se vlastita brzina A mijenjala za pola ili više (interval u kojem je A počeo ili stao ne broji se, a pod ravnim opterećenjem svake dvije naredbe drže svoj omjer, što ništa ne dokazuje); ili su pozivi B cijeli višekratnik poziva A do na dva poziva, uz A izvršen barem 500 puta. Pohranjena usporedba čita snimke snimljene između dviju koje ste odabrali (do 24); workload izvještaj broji naredbe desetak puta dok snima. Žuto kad se B izvršava 200 puta u sekundi ili uzme 10% vremena izvršavanja, plavo ispod toga. Naredbe izvršene unutar funkcije izostavljaju se - vrte se u serveru, bez povratnog putovanja koje bi se uštedjelo. Nalaz kaže ono što pg_stat_statements ne može: svaki poziv aplikaciju košta i povratno putovanje, obično više od samog upita.
- **Naredba ispod milisekunde izvršavana vrlo često**: barem 10.000 poziva ispod 1 ms svaki I ili 10% ukupnog vremena izvršavanja (udio) ili 200 poziva u sekundi (brzina), što prije dođe; naredba već imenovana u nalazu 1+N ne ponavlja se.
- **Izmjena veza** (PostgreSQL 14+): žuto od 20 novih sesija u sekundi, plavo od jedne nove sesije na svake dvije transakcije, oboje uz barem 200 novih sesija u prozoru. Imenuje baze i dodaje sesije koje je klijent prekinuo bez zatvaranja, koje su završile fatalnom greškom ili su prekinute.
- **Optimističko zaključavanje**: UPDATE ili DELETE čiji WHERE uspoređuje stupac verzije s parametrom i koji u nekim pozivima nije zapisao nijedan redak - ti su izgubili utrku s drugim piscem i ponovljeni su. Žuto kad barem 5% od barem 100 poziva, ili 1.000 poziva, nije ništa zapisalo; vidi [Veze, sesije i opterećenje](#connections-sessions-and-load).

**Top SQL za pogledati** su kartice naredbi koje su nalazi imenovali, poredane po udjelu u ukupnom vremenu izvršavanja, svaka s razlozima kao oznakama i istim gumbima: kopiraj, učitaj u SQL uređivač, EXPLAIN tamo, ili **Plan** - plan nacrtan na mjestu. Prikazuje plan koji se stvarno izvršio, iz `auto_explain` u logu poslužitelja ili iz `pg_store_plans`, a plan koji bi poslužitelj odabrao *sada* samo kad ga nijedan nema, i tako označen - vidi [Plan koji se stvarno izvršio](#the-plan-that-actually-ran).

**Postavke za bolju analizu** zatvaraju usporedbu, snimanje i snimanje uživo: ista tablica kojom završava ASH izvještaj - vidi [Kako čitati ASH izvještaj](#reading-an-ash-report). Čita se s poslužitelja na koji si spojen, pa svaki klaster dobiva vlastite prijedloge za svoju verziju: `compute_query_id` i `track_wal_io_timing` nude se od PostgreSQL 14, `pg_stat_statements.track_planning` od 13, i ne predlaže se ništa što verzija poslužitelja nema.

**Ono što ima samo workload izvještaj**, iz vlastitih uzoraka pg_stat_activity (cijeli poslužitelj, svakih nekoliko sekundi):

- **Aktivne sesije kroz vrijeme** je grafikon s jednom točkom po uzorku: sve aktivne sesije i od njih koliko ih je radilo, čekalo zaključavanje, čekalo disk ili čekalo nešto drugo, uz sesije neaktivne u transakciji (drže svoja zaključavanja). Boje su one s ASH vremenske crte, pa je zaključavanje crveno u oba. Dugi prozor crta se kao linije bez točaka; izvoz zapisuje svaki uzorak kao redak, a PDF ih uprosječuje na najviše 60 redaka, svaki sa svojim vremenskim rasponom.
- **Događaji čekanja** pokazuju gdje su uzorkovani backendi čekali: `Lock` (blokiran drugom transakcijom), `IO` (čeka disk), `LWLock` (interno natjecanje), `Client` (čeka klijenta - obično samo besposlene konekcije, nije problem). Prozor kojim dominira `Client` čekanje je zdrav besposlen sustav.
- **Uočeni parovi blokiranja** navodi tko je čekao koga, u koliko uzoraka, s obje naredbe; **Najdulja viđena otvorena transakcija** je najstarija transakcija koju je uhvatio neki uzorak.
- Njihovi nalazi pridružuju se ostalima: sesije koje su čekale zaključavanje druge sesije (prikazana je naredba koja ga je držala, ona koja je čekala pokraj nje); petina ili više aktivnog vremena u čekanju zaključavanja I barem 5 takvih uzoraka; sesije neaktivne u transakciji u petini ili više zauzetih uzoraka I barem 10 njih; transakcija otvorena minutu ili dulje.

#### Kako čitati usporedbu razdoblja {#reading-a-period-comparison}

Usporedba razdoblja (`db-awr-snapshot` action=compare-periods, pogled **Usporedba** u AWR kartici) odgovara na "što se promijenilo između tada i sada", a ne na "što se dogodilo u ovom prozoru". Svaki broj i dalje je razlika unutar vlastitog prozora; izvještaj postavlja dva prozora jedan uz drugi.

- **Usporedba razdoblja**: jedan redak po metrici - duljina prozora, potvrđene transakcije i po sekundi, postotak rollbackova, pogodak u cache, blokovi pročitani s diska, zapisano u privremene datoteke, deadlockovi, checkpointi po rasporedu i zatraženi, bufferi koje su zapisali backendi, WAL bajtovi, vrijeme izvođenja svih naredbi i koliko je naredbi radilo. **Promjena** je razdoblje 2 minus razdoblje 1, sa strelicom gore ili dolje, a **promjena %** je u odnosu na razdoblje 1 (za postotak je promjena u postotnim bodovima). Crveno je promjena na gore, zeleno na bolje; metrika kod koje rast nije ni jedno ni drugo (više transakcija je više posla, ne lošiji poslužitelj) nije obojena. Kad je usporedba za jednu bazu, zadnji redci su cijelog poslužitelja.
- Brojevi rastu s prozorom: kad su razdoblja različite duljine, uspoređujte stope po sekundi i postotke - nalaz to kaže.
- **Nalazi** navode brojeve oba razdoblja: naredba čije je vrijeme po pozivu poraslo za pola ili više i barem 10 ms, kad je to razdoblje 2 stajalo barem sekundu; naredba koja nije radila u razdoblju 1 a uzela je petinu ili više vremena izvođenja razdoblja 2 (i barem sekundu); pogodak u cache pao za dva boda ili više; privremene datoteke barem udvostručene i 64 MB ili više; checkpointi izazvani količinom WAL-a češće nego satom samo u razdoblju 2; udio rollbackova porastao za pet bodova ili više uz barem 50 transakcija; deadlockovi koji su se pojavili. Glavna naredba koja je postala barem dvostruko brža po pozivu i tako uštedjela barem sekundu dobiva zeleni redak. Ponovno pokretanje, resetiranje ili izbacivanje unutar bilo kojeg razdoblja kaže se prvo, uz razdoblje u kojem se dogodilo.
- **Naredbe: razdoblje 1 prema razdoblju 2** navodi 30 naredbi čije se ukupno vrijeme najviše promijenilo, svaku s pozivima, ukupnim i prosječnim vremenom, redcima i čitanjima s diska oba razdoblja, njezin status - *oba*, *samo razdoblje 1* (nestala) ili *samo razdoblje 2* (nova) - i gumbe koje ima Top SQL (kopiranje, SQL uređivač, EXPLAIN, **Plan**).
- **Tablice po DML-u: razdoblje 1 prema razdoblju 2** navodi 20 tablica spojene baze čija su se umetanja, izmjene i brisanja najviše promijenila, sa sekvencijalnim i indeksnim skeniranjima svakog razdoblja.

#### Plan koji se stvarno izvršio {#the-plan-that-actually-ran}

**Plan** na naredbi u ASH izvještaju, u nalazima i karticama **Top SQL za pogledati** AWR izvještaja i izvještaja o opterećenju te u usporedbi razdoblja prikazuje plan s kojim se naredba *stvarno izvršila* kad god ga je poslužitelj sačuvao, i jasno kaže kad nije.

**Zašto EXPLAIN sada nije dovoljan.** EXPLAIN napravljen naknadno ponovno planira naredbu, danas: s današnjom statistikom tablica, današnjim postavkama i indeksima, a za naredbu s parametrima `$1` - generički plan umjesto onoga koji su dobile stvarne vrijednosti. Plan koji je naredbu u prozoru učinio sporom može biti drugi, a EXPLAIN to ne može reći.

**Dva izvora čuvaju stvarni plan:**

- **auto_explain** (contrib modul koji dolazi s PostgreSQL-om) zapisuje u log poslužitelja plan svake naredbe sporije od `auto_explain.log_min_duration`, sa stvarnim brojem redaka kad je `auto_explain.log_analyze` uključen. DBApguin ga čita natrag kroz istu vezu s bazom, samo kad otvorite plan, i samo ograničeni dio loga: log datoteke koje se preklapaju s prozorom izvještaja, čitane od kraja, najviše 32 MB i 8 sekundi po pogledu. Zapisani plan povezuje se s naredbom po identifikatoru upita gdje ga log ima, a inače po tekstu naredbe bez njezinih konstanti.
- **pg_store_plans** (treće strane, BSD licenca) čuva svaki plan svake naredbe u dijeljenoj memoriji s pozivima, ukupnim vremenom te prvim i zadnjim pozivom svakog plana, pod istim query id koji koristi `pg_stat_statements`. Ništa se ne čita iz loga.

**Što prikaz plana pokazuje.** Tri gumba na vrhu - **auto_explain (log poslužitelja)**, **pg_store_plans** i **EXPLAIN sada** - svaki s brojem planova koje je našao; otvara se na prvom koji ga ima, tim redom. Oznaka kaže kakav plan gledate: *Plan koji se izvršio* zeleno, ili *Plan koji bi poslužitelj sada odabrao - ne onaj koji se izvršio* žuto, s jednim retkom o tome što uključiti. Kad se naredba u prozoru izvršila s više planova (plan se mijenjao), planovi su navedeni od najnovijeg, svaki s time kad je zadnji put zapisan (po satu poslužitelja, kao Od i Do izvještaja), trajanjem, koliko je izvršavanja s njim zapisano te njihovim najkraćim, prosječnim i najdužim vremenom; klik na jedan ga crta. Ispod stabla: je li plan povezan po query id ili po tekstu, nosi li stvarne retke i vremena, vrijednosti parametara kad ih je poslužitelj zapisao i koliko je loga pročitano. Izvor bez rezultata kaže zašto (nije učitan, nema log collectora, nema ovlasti za čitanje loga, ništa sporije od praga u prozoru, upravljana platforma). EXPLAIN sada i dalje radi svugdje i pokreće se samo kad ga odaberete ili kad ništa bolje ne postoji.

**Postavljanje, korak po korak.** *Postavke za bolju analizu* na kraju svakog ASH, AWR, izvještaja o opterećenju i tracea te prozor prije pokretanja ASH-a, AWR-a ili tracea navode svaki dio s trenutnom vrijednošću, preporučenom i načinom postavljanja. Što DBApguin može sam ima **Primijeni** (ili se označava u prozoru) uz uobičajenu potvrdu i zapis u revizijskom logu; što traži operacijski sustav prikazuje naredbu za kopiranje.

1. **Učitajte auto_explain** - u DBApguinu: Primijeni dodaje `auto_explain` u `shared_preload_libraries`. U terminalu: restartajte poslužitelj (naredbe za apt.postgresql.org, PGDG RPM, `pg_ctl` i kontejner prikazane su ispod tablice; restart prekida sve sesije).
2. **Zapisujte planove** - u DBApguinu: `auto_explain.log_min_duration` (na primjer `1s`), `auto_explain.log_format = json` (prikaz crta JSON kao stablo), a na PostgreSQL 16+ `auto_explain.log_verbose = on` (identifikator upita unutar plana). Ovo traži reload, ne restart.
3. **Zapisujte log tamo gdje ga SQL može čitati** - u DBApguinu: `logging_collector = on` (restart) i `log_destination` s dodanim `jsonlog` na PostgreSQL 15+, a `csvlog` prije. stderr se također čita, onoliko dobro koliko njegov `log_line_prefix` dopušta.
4. **Identitet naredbe** - u DBApguinu: `compute_query_id = on` (PostgreSQL 14+). Neobavezno i preporučeno: `track_io_timing = on`.
5. **Ovlast za čitanje loga** - superuser ga čita kakav jest. Za prijavu za nadzor najmanje što radi je članstvo u `pg_monitor` (popis direktorija loga) i `EXECUTE` na `pg_read_file(text, bigint, bigint, boolean)`, koji čita samo unutar direktorija podataka i direktorija loga. DBApguin prikazuje ta dva GRANT-a i nudi Primijeni kad ih spojena prijava smije dodijeliti (pita se poslužitelj, ne pogađa po imenu uloge); inače ih izvršava superuser. Prijava koja ih smije dodijeliti - obično superuser - dobiva i **Dodijeli prijavi...**: navodi prijave koje još ne mogu čitati log (ne superusere, njima ne treba ništa), prikazuje dvije naredbe za odabranu i izvršava ih, uz zapis u revizijskom logu. `pg_read_server_files` ili superuser čitali bi svaku datoteku koju može OS korisnik poslužitelja - daleko više nego što plan treba.
6. **Ili pg_store_plans** - u terminalu: instalirajte paket (`dnf install pg_store_plans_NN` iz PGDG repozitorija na RHEL-u, Rockyju i Fedori; za Debian i Ubuntu nema apt paketa, pa ga izgradite s `make USE_PGXS=1` i `make USE_PGXS=1 install`). U DBApguinu: Primijeni ga dodaje u `shared_preload_libraries`, restart ga učitava, a zatim **Instaliraj…** izvršava `CREATE EXTENSION pg_store_plans` u svakoj bazi u kojoj ga želite čitati.

**Što postoji u kojoj verziji** (provjereno u dokumentaciji PostgreSQL-a za svaku verziju i na poslužiteljima od 9.4 do 19):

| Naziv | PostgreSQL | Napomene |
| --- | --- | --- |
| `auto_explain` u `shared_preload_libraries` | sve | restart za učitavanje |
| `auto_explain.log_min_duration`, `log_analyze`, `log_timing`, `log_buffers`, `log_verbose`, `log_format`, `log_triggers`, `log_nested_statements` | sve | reload; mijenja ih superuser |
| `auto_explain.sample_rate` | 9.6+ | |
| `auto_explain.log_level`, `auto_explain.log_settings` | 12+ | |
| `auto_explain.log_wal` | 13+ | |
| `auto_explain.log_parameter_max_length` | 16+ | vrijednosti parametara pripremljenih naredbi, zadano u cijelosti (-1) |
| `auto_explain.log_io`, `auto_explain.log_extension_options` | 19 | |
| `compute_query_id` | 14+ | prije 14 plan se povezuje po tekstu |
| Identifikator upita upisan u zapisani plan (`log_verbose` poštuje `compute_query_id`) | 16+ | na 14 i 15 identifikator je samo u stupcu `query_id` csvloga / jsonloga |
| `logging_collector` | sve | restart |
| `log_destination = csvlog` | sve | stupac `query_id` od 14 |
| `log_destination = jsonlog` | 15+ | jedan unos po retku, s `query_id` |
| `pg_current_logfile()`, `pg_ls_logdir()` | 10+ | `pg_ls_logdir()`: superuser i `pg_monitor` |
| uloga `pg_monitor` | 10+ | |
| `pg_read_file(text, bigint, bigint, boolean)` | 9.5+ | `GRANT EXECUTE` moguć od 11; prije samo superuser |
| uloga `pg_read_server_files` | 11+ | više nego što plan treba |
| `pg_store_plans` | treće strane | testirano s 1.10 na 17 i 18, gdje je njegov query id onaj iz `pg_stat_statements` |

Prije PostgreSQL 10 nema `pg_ls_logdir()`, pa se log tamo ne čita kroz SQL; prikaz to kaže i nudi pg_store_plans ili EXPLAIN sada. PostgreSQL 13 je najstarija verzija koju DBApguin u potpunosti podržava.

**Što to košta** - ponovljeno uz svaku postavku na mjestu gdje je uključujete:

- **Restart** za `shared_preload_libraries` (auto_explain, pg_store_plans) i za `logging_collector`. DBApguin nikad ne restarta poslužitelj.
- **`auto_explain.log_analyze` instrumentira svaku naredbu**, zapisala se ili ne, što može primjetno usporiti opterećen poslužitelj. Prvo izmjerite sat s `pg_test_timing`; `auto_explain.log_timing = off` zadržava stvarni broj redaka (koji otkriva pogrešnu procjenu) i uklanja većinu cijene; `auto_explain.sample_rate` ispod 1 instrumentira samo dio naredbi, uz cijenu da se neke spore ne zapišu.
- **Obujam loga.** Svaki plan ima mnogo redaka; nizak `log_min_duration` na opterećenom poslužitelju brzo puni log. Isplanirajte rotaciju (`log_rotation_age`, `log_rotation_size`) i disk. jsonlog ili csvlog je druga datoteka uz stderr.
- **Log je osjetljiv.** Sadrži tekst naredbi i, od PostgreSQL 16, vrijednosti parametara (`auto_explain.log_parameter_max_length = 0` ih izostavlja). Tko smije čitati log, čita i to.
- **Čitanje loga traži ovlasti za čitanje datoteka.** `EXECUTE` na `pg_read_file` čita svaku datoteku u direktoriju podataka i direktoriju loga mimo svih ovlasti unutar baze; dodijelite ga samo prijavi za nadzor.
- **pg_store_plans** zauzima dijeljenu memoriju (`pg_store_plans.max` planova) i radi na svakoj naredbi; većina upravljanih oblaka ga ne nudi.
- **Upravljani oblaci** (RDS, Aurora, Cloud SQL, Azure i drugi) ne dopuštaju SQL-u čitanje loga poslužitelja: log je u konzoli pružatelja usluge, a prikaz to kaže i prelazi na pg_store_plans gdje je ponuđen, zatim na EXPLAIN sada.

DBApguin čita log samo kroz vašu vezu, pročitano drži u memoriji dok je plan otvoren i ništa nikamo ne šalje - vidi [obavijest o privatnosti](PRIVACY.md).

#### Opseg: vaša baza, uz pregled poslužitelja {#awr-ash-scope}

ASH i AWR prikupljaju za cijeli poslužitelj, kao Oracleovi za instancu: uzorkivač čita svaku sesiju svake baze, a svaka snimka sadrži `pg_stat_statements` i `pg_stat_database` za sve baze. Oboje radi u pozadini i kad prijeđete na drugu vezu ili karticu; popis Background jobs ih prikazuje.

Izvještaj prvo kaže odakle su podaci (**Collected for**: poslužitelj i baza iz koje su uzorkivač ili snimke pokrenuti), a zatim o čemu govori. Zadano je to ista baza: Top SQL, čekanja, nalazi i kartice su te baze, a uz njih stoji cijeli poslužitelj - drugi red u AWR-ovom **Window summary**, **Activity by database** za svaku bazu i nalaz **Elsewhere on the server** kad je druga baza napravila nešto vrijedno spomena (deadlockovi, 50 ili više rollbackova, petina vremena izvršavanja I barem 10 s; u ASH-u petina aktivnih uzoraka I barem 60 njih, ili 10 i više uzoraka koji čekaju zaključavanja). `*` u polju **Database** prikazuje cijeli poslužitelj, a ime druge baze tu bazu uz isti pregled. Izbor se radi pri čitanju, ne pri prikupljanju, pa iste snimke i uzorci služe za sve troje. **Top tables by DML** je iz baze na koju ste spojeni (PostgreSQL te brojače vodi po bazi) i pojavljuje se samo kad je izvještaj o toj bazi. **Plan** traži plan u bazi u kojoj se naredba izvršavala. Uzorci i snimke od prije 22.-23. listopada 2026. ne bilježe bazu po naredbi; za njih izvještaj prikazuje cijeli poslužitelj i to kaže.

**pg_stat_statements was full and evicted statements** u usporedbi znači da AWR-ovom Top SQL-u nedostaje ono što su te naredbe koštale: `pg_stat_statements` čuva `pg_stat_statements.max` naredbi i svaki put kad se napuni izbaci najmanje korištenih 5% (najmanje 10). PostgreSQL 14 i noviji broje ta čišćenja; nalaz ih navodi, uz procjenu izgubljenih naredbi i popunjenost. Povećajte `pg_stat_statements.max` (restart) - tablica postavki na kraju ima taj red. Naredba kojoj su brojači pali (izbačena pa vraćena, ili reset) broji se od povratka.

#### Trend: svaka snimka nacrtana kroz vrijeme {#awr-trend}

`db-awr-snapshot` action=trend pretvara cijelu povijest snimaka za jedan server u grafove, umjesto prikaza jednog para odjednom koji daje `compare`. Treba barem dvije snimke - uzmite ih ručno, ili pustite auto-scheduler da izgradi povijest u pozadini.

Na ovim grafovima pojavljuju se dvije različite vrste brojki, i namjerno su nacrtane drugačije:

- **Vrijednosti u trenutku snimanja** - veličina baze, procijenjeni heap bloat - točno su ono što je izmjereno u trenutku te snimke. Jedna točka po snimci, nacrtana kakva jest.
- **Stope** - cache-hit%, rollback%, top upiti po ukupnom vremenu - dolaze iz brojača `pg_stat_database` i `pg_stat_statements`, koji samo rastu od zadnjeg resetiranja. Sirovi brojač jedne snimke sam po sebi ne govori ništa; ono što se crta je **razlika između dvije uzastopne snimke**, pretvorena u stopu za taj prozor. Ista je to računica koju koristi `compare` za dvije snimke, samo ponovljena za svaki uzastopni par u povijesti umjesto jednog para koji je korisnik odabrao.

**Regresije** su istaknute iznad grafova, ne prepuštene čitatelju da ih primijeti u liniji: metrika je označena samo kad se pomaknula i za relativni iznos (postotak) i za apsolutni iznos (bajtovi, ili postotne točke) - isto pravilo koje koristi svaki prag u ovoj aplikaciji, pa mala baza koja udvostruči veličinu ne nadmašuje veliku koja naraste za puno više u apsolutnim brojkama, i obrnuto. Točka usporedbe je otprilike tjedan dana prije zadnje snimke, ili najstarija dostupna točka kad je povijest kraća od toga.

**Rupe se prikazuju, nikad se ne izglađuju.** Kad je razmak između dvije snimke puno širi od uobičajenog razmaka za taj server (2.5x medijan, s podom od 3 sata da se dvije ručne snimke uzete sat vremena razmaknute nikad ne označe jedna protiv druge), graf prekida liniju umjesto da crta ravan rub kroz vrijeme za koje ništa nije stvarno snimljeno. Osjenčana traka označava točno gdje je nedostajući razmak; lebdenje mišem preko nje kaže koliko je širok.

Graf **top upita** prati šačicu `queryid`-ova koji su koštali najviše ukupnog vremena zbrojeno kroz cijelu povijest, svaki kao vlastita linija potrošenog vremena po prozoru - ne jedinstveni kumulativni total, koji samo raste i bio bi tek ravna dijagonalna linija koja ne govori ništa o tome *kad* je cijena plaćena.

Popis **najveće tablice** ispod grafa veličine imenuje najveće tablice prema zadnjoj snimci - "veličina baze" sama po sebi ne govori koja je tablica zapravo narasla. Ne prati se po tablici kroz vrijeme (tablica može ispasti iz top 10 pa se vratiti kako druge rastu oko nje), samo trenutna slika, isto kao `db-table-sizes`.

#### Spremljeni izvještaji {#saved-reports}

Svako snimanje opterećenja, AWR izvještaj (izvještaj dviju snimki, usporedba razdoblja ili trend), ASH izvještaj i praćenje sesije čuva se kad završi. *Podešavanje i dijagnostika* > **Spremljeni izvještaji** navodi ih sa svih poslužitelja na jednom mjestu, najnovije prvo: kada, vrsta, poslužitelj, baza o kojoj je riječ (*cijeli poslužitelj* za izvještaj o svima) i što pokriva (prozor, dvije snimke, praćeni korisnik), s veličinom. Poveznica **Spremljeni izvještaji** u karticama AWR, ASH i praćenja sesije otvara isti popis filtriran na taj alat i taj poslužitelj; iz rezultata *Snimanja* u AWR kartici prikazuje snimanja opterećenja.

- **Filtri**: vrsta, poslužitelj, baza i raspon datuma (birač datuma podcrtava dane u kojima postoji izvještaj). **Očisti filtre** opet prikazuje sve.
- **Otvori** prikazuje izvještaj kakav je spremljen, samo za čitanje: baza se ništa ne pita, a gumba koji bi je mijenjali nema. U podnožju je redak za izvoz koji je izvještaj imao (tekst, CSV, JSON, PDF), zapisan s poslužiteljem i vremenom izvođenja samog izvještaja, ne današnjim.
- **Izvezi** otvara ista četiri formata u retku, bez otvaranja izvještaja.
- **Obriši** prvo pita i kaže što odlazi: izvještaj s odjeljcima i tekstom te njegov unos u Poslovima u pozadini; za praćenje i njegov unos u Povijesti praćenja. Ranije izvezena kopija nije pogođena. Revizijski zapis bilježi brisanje.

Izvještaji se čuvaju odvojeno od ostalih rezultata koje čuvaju Poslovi u pozadini: ostaje najnovijih 200, koliko god drugih provjera radilo (spremište praćenja čuva najnovijih 500 praćenja po poslužitelju). Knjižnica ne sprema ništa svoje, pa je izvoz i uvoz podataka aplikacije nose sa sobom: spremljeni izvještaji putuju sa stavkom izvještaja, praćenja s AWR i ASH poviješću poslužitelja.

### 6.7 Logički izvoz podskupa (kriška koja se sama učita) {#logical-subset-export-a-slice-that-loads-on-its-own}

Backup kopira cijele tablice; ovo kopira krišku jedne. WHERE uvjetom imenujete
polazne retke - jedan kupac, jedan tenant, prošlomjesečne narudžbe - a izvoz
prati strane ključeve iz njih: roditelje, jer se redak kojem nedostaje kupac
neće učitati, i djecu, jer narudžba bez stavki nije ista narudžba. Van izlazi
jedna `.sql` datoteka, pisana roditelji-prvo, koja se sama učita u praznu bazu
bez ijedne greške na stranom ključu.

Uobičajen razlog da je poželite: testna baza koja se ponaša kao produkcija i
stane na laptop - tenant 42 umjesto 400 GB.

1. Pregled sheme -> desni klik na tablicu -> Podskup.
2. Upišite WHERE za polaznu tablicu.
3. **Otkrivanje zatvaranja** prolazi strane ključeve bez materijalizacije ijednog
   retka i javlja koliko bi ih koja tablica dala. Pokrenite ga prvo: WHERE koji
   izgleda usko zna kroz jednu široku spojnu tablicu povući skoro cijelu bazu, a
   to je bolje saznati ovdje nego na pola izvoza.
4. Izvoz zapisuje retke na lokaciju koju odaberete - INSERT naredbe, ili COPY
   blokove kad je brzina učitavanja važnija od mogućnosti uređivanja.

Dubina (koliko stranih ključeva obilazak slijedi) i ograničenje redaka po
tablici oboje zaustavljaju obilazak. Doseg bilo kojeg od njih se javlja umjesto
da se tiho primijeni: krnji podskup koji izgleda potpuno gori je od onog koji
kaže da nije.

Retci se kopiraju točno kakvi jesu, **uključujući osobne podatke**. Maskiranje
je u Copy table, niže.

### 6.8 Maskiranje osobnih podataka u kopiji (PII anonimizacija) {#masking-personal-data-in-a-copy-pii-anonymisation}

Osobni podaci - imena, e-mail adrese, brojevi telefona, OIB-ovi, datumi rođenja
- su ono zbog čega kopija produkcije nije nešto što možete dati developeru ili
dobavljaču. Pregled sheme -> desni klik na tablicu -> **Copy table** -> PII
anonymization zamjenjuje te vrijednosti dok se retci čitaju: stupac se u SELECT-u
zamijeni izrazom, pa prava vrijednost nikad ne stigne na odredište niti se
zapiše na disk. Izvorna tablica se ne mijenja. Stupce imenujete sami - ništa se
ne prepoznaje automatski, a stupac koji zaboravite prekopira se u čistom obliku.

| Pravilo | Zapisuje | Što i dalje radi |
| --- | --- | --- |
| `NULL` | ništa | Nema što procuriti. Puca na NOT NULL stupcu. |
| `@HASH` | `sha256` vrijednosti (`md5` na PostgreSQL ispod 11) | Jednake vrijednosti ostaju jednake, pa joinovi i GROUP BY i dalje rade. Dodaj salt (ispod) da se hash ne može povezati s produkcijom. |
| `@HASH:<salt>` | kao `@HASH`, s ovim saltom po stupcu | Za rijedak slučaj kad jedan stupac treba svoj salt uz onaj na razini kopije. |
| `@SEQ` | `ime_1`, `ime_2`, ... | Vrijednosti ostaju jedinstvene. Poveznica s drugom tablicom nestaje. |
| `@EMAIL` | `user1@example.invalid` | Ispravna sintaksa, a na `.invalid` se ne može isporučiti - nijedan test ne može poslati mail kupcu. |
| `@PHONE` | `0000000000` | Duljina, pa provjere formata i dalje prolaze. |
| `@DATE` | prvi dan u mjesecu | Mjesečni izvještaji i dobne skupine. Točan dan nestaje. |
| `@REDACT` | `REDACTED` | Samo to da je vrijednost postojala. |
| `@EXPR:<sql>` | što god SQL vrati | Vlastito pravilo, npr. `left(ime,1) || '***'`. Ne provjerava se - pogrešan izraz sruši kopiranje. |
| bilo koji drugi tekst | taj tekst, u svakom retku | Fiksni ispunjivač. Stupac prestaje varirati. |

#### Salt

Nesoljeni hash je maska, ne dokaz. Kad je skup mogućih ulaza malen i poznat -
OIB, broj telefona, poštanski broj - tko god ima istu hash funkciju može
hashirati svaku mogućnost i vratiti vrijednosti natrag. **Salt** je tajni niz
umiješan u vrijednost prije hashiranja, pa se hash bez njega ne može
reproducirati. Kontrola **Salt za @HASH** nudi:

- **bez** - hash same vrijednosti (deanonimizabilno kao gore).
- **slučajan, samo ova kopija** - svjež slučajan salt generira se za ovu kopiju
  i baca nakon toga. Jednaki ulazi i dalje hashiraju jednako *unutar* ove
  kopije, pa joinovi i GROUP BY na maskiranim stupcima i dalje rade, ali se
  kopija ne može povezati s produkcijom ni s drugom kopijom.
- **fiksni** - sami zadate salt. Koristite ovo kod **inkrementalnih** kopija
  (druga kopija se mora poklapati s prvom) ili kad trebate **usporediti dvije
  kopije** - obje trebaju isti salt. Čuvajte ga sigurno; tko god ga ima može
  poništiti zaštitu salta.

Salt se šalje serveru kao postavka transakcije
(`set_config('dbapguin.anon_salt', ...)`) i čita natrag preko
`current_setting()` iz izraza za maskiranje. **Nikad** se ne upisuje u
generirani SQL, DBApguin audit log ni odredišnu tablicu. Do izvornog servera
stigne kao jedna `set_config` naredba - ako taj server bilježi svaku naredbu
(`log_statement = all`), salt se tamo pojavi jednom.

`@SEQ`, `@REDACT` ili `NULL` i dalje su pravi izbor kad kopija uopće ne smije
biti ponovno identifikabilna, sa saltom ili bez njega. I maskiranje radi
stupac po stupac: redak bez imena, ali s netaknutim datumom rođenja i
poštanskim brojem, često i dalje pokazuje na jednu osobu.

Stupac koji vam uopće ne treba bolje je posve izbaciti, pod Data filters ->
Skip columns.

### 6.9 Constrainti - makni ih i vrati ih {#constraints-take-them-off-put-them-back}

Ponekad constrainti moraju s tablice pa se poslije vratiti. Uobičajen slučaj je
alat za replikaciju ili CDC koji radi početnu sinkronizaciju u odredište: neće
krenuti dok su ondje strani ključevi u primjeni. Masovni uvoz je drugi - brži je
bez provjera, pa ih na kraju vrati.

Ručno, ovo je mjesto gdje podaci tiho prestanu biti konzistentni, jer se tri
stvari lako pogriješe, a PostgreSQL vas zaustavi samo na jednoj:

- `DROP ... CASCADE` makne strane ključeve koje druge tablice imaju prema ovoj i
  ne zabilježi što je maknuo. Njihove definicije odu s njima.
- Definicija prepisana po sjećanju izgubi `ON DELETE CASCADE`, ili `DEFERRABLE
  INITIALLY DEFERRED`, ili `WHERE` na parcijalnom jedinstvenom indeksu. Izgleda
  vraćeno. Provodi nešto drugo.
- Vraćanje stranog ključa na uobičajen način skenira cijelu tablicu pod
  zaključavanjem koje zaustavlja čitanja i pisanja.

Pregled sheme -> desni klik na tablicu -> **Constrainti - makni i vrati**. Čita
svaki constraint iz `pg_constraint` kroz `pg_get_constraintdef`, što je vlastiti
PostgreSQL-ov ispis, i piše dvije skripte: jednu koja ih miče redoslijedom koji
radi, drugu koja ih vraća točno onakve kakvi su bili. Aplikacija ne izvršava
ništa. Dobijete SQL i pokrenete ga kad odlučite.

Tri opcije mijenjaju skriptu za vraćanje:

- **Dodaj NOT VALID, pa validiraj odvojeno.** `ADD CONSTRAINT ... NOT VALID`
  preskače skeniranje; `VALIDATE CONSTRAINT` ga obavi poslije pod `SHARE UPDATE
  EXCLUSIVE`, kroz koji čitatelji i pisci prolaze bez zastoja. Mjereno na
  tablici od 200.000 redaka: 139 ms blokade postalo je 1,2 ms plus prolaz od
  131 ms koji nitko ne čeka. Dok VALIDATE nije prošao, constraint vrijedi samo
  za nove retke, i `pg_constraint.convalidated` to kaže.
- **Gradi indekse primarnog i jedinstvenog ključa bez blokade.** `CREATE UNIQUE
  INDEX CONCURRENTLY`, pa `ADD CONSTRAINT ... USING INDEX`, koji taj indeks
  posvoji umjesto da gradi drugi - pa je rezultat identičan običnom ponovnom
  dodavanju. Indeks se gradi pod vlastitim imenom constrainta, iz definicije
  koju daje sam poslužitelj (`pg_get_indexdef`) - pa se pokrivajući ključ,
  fillfactor, tablespace ili `NULLS NOT DISTINCT` vrate onakvi kakvi su bili, a
  odgodivi constraint zadrži odgodivost. `CREATE INDEX
  CONCURRENTLY` ne može unutar transakcijskog bloka, zbog čega skripta za
  vraćanje ne smije biti umotana u `BEGIN`/`COMMIT`.

  To je ujedno jedina naredba u skripti koja može pasti i nešto ostaviti za
  sobom: duplikat u podacima ili prekinuto izvođenje ostave indeks koji postoji
  i neupotrebljiv je, a u popisu constrainta nema ničega što bi to objasnilo.
  Skripta takav indeks sama makne prije gradnje, pa je nakon neuspjeha dovoljno
  pokrenuti je ponovno. Briše se samo indeks koji je PostgreSQL označio
  nevaljanim - onaj koji je u upotrebi ne dira se.
- **Uključi strane ključeve koji pokazuju na ovu tablicu.** Isključeno, skripta
  za micanje pukne na primarnom ključu: strani ključ druge tablice ovisi o
  njegovom indeksu. Uključeno, te tablice nakratko prestanu provoditi svoju
  referencu na ovu - što je nagodba koju situacija traži, rečena naglas umjesto
  skrivena u CASCADE-u.

Obje skripte počinju s `SET lock_timeout = '3s'`. Svaki `ALTER TABLE` u njima
treba `ACCESS EXCLUSIVE`, a čekanje na to zaključavanje pretvara online
pregradnju u zastoj: zahtjev se poreda iza onoga što se već izvodi, a sve što
poslije stigne poreda se iza zahtjeva. Tri sekunde znače da je najgori ishod
naredba koju treba ponoviti, a ne tablica koju nitko ne može čitati. Obrišite
redak ako želite čekati koliko god treba. DBApguinov editor ionako postavlja
dvije sekunde na svaku konekciju, pa je ovo najvažnije kad se datoteka mjesecima
kasnije pokrene iz `psql`-a.

Spremite skriptu za vraćanje prije nego pokrenete onu za micanje. Ona je jedina
kopija onoga što je tablica prije provodila.

#### Opseg: jedna tablica, jedna shema ili više njih {#scope-one-table-one-schema-or-several}

Isti dijalog odgovara na sva tri. Desni klik na **tablicu** za tu tablicu;
desni klik na **shemu** → Prijenos → *Constrainti - makni i vrati* za svaku
tablicu u njoj. Migracija ili prelazak na replikaciju je pitanje o shemi, a
raditi to tablicu po tablicu je način na koji se constraint zaboravi.

**Koji constrainti** filtrira po vrsti - primarni ključevi, jedinstveni, strani
ključevi, check, exclude. Označiti samo *strane ključeve* je najčešći slučaj:
CDC alat obično prigovara samo njima, a ostavljeni primarni ključevi znače da
odredište zadržava identitet i svoje indekse.

Na razini sheme redoslijed je važniji nego za jednu tablicu, i skripta se za to
brine: svaki primarni i jedinstveni ključ vraća se prije bilo kojeg stranog
ključa koji ga referencira, a skripta za micanje radi obrnuto. Poredano po imenu
tablice - kako je prva verzija radila - strani ključ na `items` dodaje se prije
nego `orders` ima primarni ključ, i PostgreSQL to odbije s *there is no unique
constraint matching given keys*.

#### Kopiranje constrainta na drugu shemu {#copying-constraints-onto-another-schema}

Imenujte **ciljnu shemu** i dijalog prestaje pisati skriptu za micanje pa piše
onu koja ove constrainte stavlja na tablice te sheme. Slučaj za koji služi:
kopija podataka učitana bez constrainta - jer se tako učitava brzo, ili jer
replikacija inače ne bi krenula - kojoj sad trebaju ista pravila kakva ima
original.

Imena tablica i stupaca moraju se podudarati. Putuju samo constrainti: bez
podataka, bez stupaca, bez indeksa osim onih koje ključ treba.

Definicije se čitaju iznutra izvorne sheme, pa referenca na tablicu u toj shemi
izlazi nekvalificirana - `REFERENCES customers(id)` - a `SET search_path` u
skripti je usmjerava na cilj. Referenca u *drugu* shemu ostaje kvalificirana i
i dalje pokazuje kamo je pokazivala, što je također ispravno: strani ključ u
zajedničku referentnu shemu ne bi se smio preusmjeriti.

Imena constrainta prelaze nepromijenjena, jer je ime constrainta jedinstveno po
tablici, a ne po bazi. Ako cilj već ima constraint tog imena na toj tablici,
naredba pukne - sigurniji ishod, jer je alternativa tiha zamjena.

### 6.10 Pretraga po sličnosti: pg_trgm i pgvector {#similarity-search}

Dvije ekstenzije odgovaraju na "nađi retke slične ovome", na dva različita načina:

- **pg_trgm** uspoređuje kako tekst *izgleda*. Reže tekst na komade od tri
  slova (trigrame), pa i tipfeler ili dio imena i dalje pogađa: `LIKE '%pad%'`,
  `ILIKE`, operator sličnosti `%` i trigramska udaljenost `<->`. Dolazi s
  contrib modulima PostgreSQL-a.
- **pgvector** uspoređuje što model kaže da tekst *znači*. Model svaki tekst
  pretvara u niz brojeva (embedding, spremljen u stupac `vector(n)`,
  `halfvec(n)` ili `sparsevec(n)`); bliski brojevi znače blisko značenje.
  Upiti traže najbliže retke: `ORDER BY embedding <=> $1 LIMIT 10`.

Trebaju različite indekse, a btree ne služi nijednu: ni `ILIKE` ni uzorak koji
počinje zamjenskim znakom, ni udaljenost. Što aplikacija radi sa svakom:

**Baza podataka -> Upravljaj ekstenzijama** prikazuje obje s instaliranom
verzijom i najnovijom koju poslužitelj nudi; **update** nudi svaku kasniju
verziju, redom verzija (0.10 iza 0.9).

**EXPLAIN** čita indekse koje plan koristi, ne samo njihova imena:

- skeniranje indeksa kroz **HNSW** ili **IVFFlat** imenuje se kao približno
  traženje najbližih susjeda, s klasom operatora za koju je izgrađen i
  postavkom koja mijenja brzinu za odziv (`hnsw.ef_search`, `ivfflat.probes`)
  kakvu je imala sesija koja je izvela plan;
- filtar na istom koraku se označava: indeks prvo preda svoje kandidate, a
  filtar radi nakon toga, pa se može vratiti manje redaka od `LIMIT`-a. Od
  pgvectora 0.8.0 savjet nudi `hnsw.iterative_scan`;
- **Sort po udaljenosti nad sekvencijalnim skeniranjem** je točna pretraga
  svih redaka. Savjet kaže koji je od tri slučaja: nema približnog indeksa na
  stupcu; indeks je izgrađen za *drugi* operator (indeks s `vector_cosine_ops`
  služi samo `<=>`, pa `ORDER BY embedding <-> ...` čita cijelu tablicu); ili
  odgovarajući indeks planer nije iskoristio, jer se koristi samo za
  `ORDER BY stupac <op> vrijednost` uzlazno uz `LIMIT`;
- **trigramski indeks** (GIN `gin_trgm_ops`, GiST `gist_trgm_ops`) objašnjava
  se kao komadi plus ponovna provjera, s brojem pronađenih redaka koji nisu bili
  pogodak;
- sekvencijalno skeniranje filtrirano s `LIKE`/`ILIKE` na stupcu koji ima samo
  btree kaže da ga btree ne može poslužiti i daje trigramski indeks;
  `similarity(a, b) > x` u filtru nikad nije indeksiran, a savjet kaže da se
  napiše `a % b`.

Čvorovi koje je `LIMIT` rano zaustavio (top-10 pretraga čita 10 od procijenjenih
20.000 redaka) više se ne označavaju kao najgora procjena plana.

**Savjetnik za indekse** (iz pg_stat_statements): za `LIKE`, `ILIKE`, `%` i
`similarity()` na stupcu predlaže trigramski GIN indeks, nikad btree, s
`CREATE EXTENSION pg_trgm` ispred kad ga baza još nema (dijalog Run prikazuje
i izvodi oboje). Za `ORDER BY stupac <udaljenost> ...` predlaže HNSW indeks s
klasom operatora za operator koji upiti koriste, pročitanom iz kataloga samog
poslužitelja, i kaže cijenu: približno, može propustiti neke od stvarno
najbližih redaka. Nikad ne predlaže btree na vektorskom stupcu. Stupac s više
dimenzija nego što indeks prima (2.000 za `vector`, 4.000 za `halfvec`) ili bez
deklarirane dimenzije dobiva objašnjenje umjesto naredbe.

**Ključne postavke**, gdje je pgvector instaliran u bazi: `hnsw.ef_search`,
`hnsw.iterative_scan`, `hnsw.max_scan_tuples`, `ivfflat.probes` i
`ivfflat.iterative_scan` (iterativne od pgvectora 0.8.0), te
`pg_trgm.similarity_threshold` gdje je pg_trgm. `maintenance_work_mem`
uspoređuje se s najvećim HNSW indeksom: README pgvectora kaže da je izgradnja
HNSW-a znatno brža kad graf stane u nju, i da u kontejneru paralelna izgradnja
treba `/dev/shm` barem toliko velik. `max_parallel_maintenance_workers` kaže da
s njima pgvector gradi svoje indekse.

**Veličine.** Shema -> tablica -> **Pohrana** prikazuje svaki vektorski stupac
s bajtovima koje zauzima jedna vrijednost (formula pgvectora: `vector`
4 x dimenzije + 8, `halfvec` 2 x dimenzije + 8), koliko zauzima uzorak
spremljenih vrijednosti i, na PostgreSQL-u 17, koliko ih živi u TOAST tablici -
`vector(768)` ima oko 3 kB, više nego što redak drži u sebi, i zato takva
tablica ima malu heap i veliku TOAST tablicu. Ispod su indeksi tablice s
metodom, klasom operatora, opcijama (`m`, `ef_construction`, `lists`) i
veličinom. Provjera **Veličine indeksa** imenuje metodu i klasu operatora
svakog indeksa i TOAST veličinu njegove tablice.

**Stvori indeks** nudi `hnsw` i `ivfflat` kao metode i klasu operatora za
stupce (`vector_cosine_ops`, `gin_trgm_ops`, ...); ASC/DESC i NULLS nude se samo
za btree, jedinu metodu koja ih prihvaća.

**Kopije.** DDL, usporedba i kopiranje čuvaju klase operatora i opcije indeksa.
Kloniranje sheme s retcima gradi HNSW i IVFFlat indekse nakon redaka (IVFFlat
indeks računa svoje liste iz redaka prisutnih u trenutku izgradnje), a Kopiraj
tablicu ponovno gradi IVFFlat indeks kad su retci unutra. Usporedba stavlja
napomenu iznad promjene dimenzije vektorskog stupca: spremljeni vektori ne
pretvaraju se u drugu dimenziju, pa retke treba ponovno pretvoriti u embeddinge.

## 7. Postavke

Ikona zupčanika gore desno otvara Postavke:

- Tema: tamna / svijetla
- **Jezik: English, Hrvatski, Deutsch, Español.** Izbor se sprema na strani
  poslužitelja (`app_prefs.json`), pa vas prati kroz ponovna pokretanja umjesto
  da živi u jednom profilu preglednika. Primjenjuje se odmah, bez ponovnog
  pokretanja, uključujući na menije i dijaloge koji su već na ekranu.
- Veličina fonta (10-30 px)
- **Pokretanje -> Port:** port na kojem sluša DBApguinov vlastiti backend -
  samo na ovom računalu, `127.0.0.1` - 3236 ako ga ne promijenite. Promijenite
  ga kad drugi program treba 3236; primjenjuje se pri sljedećem pokretanju
  DBApguina. Ono što prozor pamti (tema, favoriti, povijest SQL-a, otvorene
  kartice) seli se s njim.
- Revizijski zapis: uključi / isključi (vidi niže)
- Alati: putanje do psql / pg_dump / pg_restore / pg_repack (nadjačavaju automatski pronalazak)
- Certifikati: učitavanje korijenskih certifikata u PEM obliku za `sslmode=verify-*`
- Certifikati i putanje alata kao gore; **O aplikaciji se preselilo u Pomoć -> O DBApguinu**, uz pravne dokumente

### Revizijski zapis {#audit-log}

Uključen je dok ga ne isključite - prekidač je u samom prozoru zapisa. Svaka
naredba koju izvrši SQL editor (osim kad je tab postavljen samo za čitanje) i
svaka promjena napravljena DBApguinovim alatima dodaje se u
`audit/audit-YYYY-MM.jsonl` (jedna datoteka po mjesecu) - vrijeme, cilj, vaš
korisnik operacijskog sustava i korisnik baze, naredba s prikrivenim lozinkama,
trajanje i ishod. Otvorite ga iz **Prikaz -> Zapisnik revizije**: pretražite po rasponu datuma,
akciji ili podnizu SQL-a, izvezite raspon kao JSONL, ili čitajte sirove
datoteke izravno bilo kojim uređivačem teksta ili `jq`-om.

**Svaki je zapis ulančan s prethodnim.** Svaki redak nosi `hash` vlastitog
sadržaja plus `hash` prethodnog retka (`prev_hash`), pa izmjena ili brisanje
prošlog retka mijenja ono što je hash sljedećeg retka trebao biti -
**Provjeri lanac** u pregledniku zapisa ponovno izračunava cijeli lanac i
javlja ili "netaknut" ili točno koja je datoteka i redak prvi koji se više ne
podudara. Zapis napisan prije nego što je ovo postojalo nema nijedno od ta
dva polja; provjera takve zapise tretira kao one prije lanca umjesto da na
njima padne, i javlja gdje sam lanac počinje. Lanac također preživljava
mjesečnu rotaciju - prvi zapis novog mjeseca vezuje se natrag na posljednji
zapis prethodne datoteke, pa dijeljenje zapisa u datoteke ne dijeli lanac.

**Pažljivo pročitajte što ovo dokazuje, jer je lako precijeniti.** Dokazuje
da nitko nije napravio usputnu, neprimijećenu izmjenu lanca koji se
provjerava - korisno samo po sebi, i razlog zašto gumb postoji. **Ne**
dokazuje da nitko s pravom pisanja u datoteku nije petljao po njoj: hashiranje
ne treba nikakvu tajnu, algoritam je javan, a zapis stoji odmah pored
aplikacije koja ga može ponovno izračunati - pa tko god može urediti
`audit-2026-05.jsonl` može i ponovno izračunati svaki hash nakon svoje
izmjene, i lanac će se ponovno provjeriti kao da se ništa nije dogodilo.
Prava zaštita od toga treba sidro izvan ovog stroja - potpis ključem koji ne
posjedujete, ili hash poslan negdje drugdje u trenutku pisanja - a ova
značajka to ne radi. Gdje vam treba zapis koji preživi kompromitiranog
lokalnog administratora, koristite vlastito bilježenje vašeg poslužitelja
baze podataka (`pgaudit`, `log_statement`) koje piše u odvojeno, pristupom
ograničeno odredište, ne ovu datoteku. Vidi [licencni ugovor](EULA.md), odjeljak 7 (Pomoć -> Licencni ugovor).

### Podaci aplikacije: izvoz, uvoz i čišćenje {#app-data-backup-and-clear}

Sve što DBApguin čuva između pokretanja nalazi se u jednom direktoriju (putanja se prikazuje u oba dijaloga). Postavke -> Podaci aplikacije ima tri gumba.

**Izvezi podatke aplikacije** zapisuje jedan `.tar.gz` onoga što označiš: spremljene konekcije, postavke, spremljene upite, SQL workspace, popis zadataka, spremljene izvještaje, pohranjene AWR snimke, ASH dane i praćenja sesija svakog poslužitelja, revizijski zapisnik, instalirane pluginove i njihove podatke, TLS certifikate. Po defaultu je sve označeno. Podatkovni ključ ide s konekcijama i podacima pluginova, jer se njihove lozinke bez njega ne mogu pročitati.

**Uvezi podatke aplikacije** čita takvu arhivu, pokazuje što sadrži uz ono što je već ovdje i preuzima što označiš (opet sve po defaultu). Spaja, ne zamjenjuje: spremljena povijest (AWR snimke, ASH dani, praćenja sesija, spremljeni izvještaji, logovi poslova, revizijski zapisnik, pluginovi) se dodaje i nikad ne prepisuje datoteku koja već postoji; spremljene konekcije se spajaju po id-u, zauzeto ime dobiva broj, a lozinke se ponovno enkriptiraju ključem ove instalacije; pojedinačna datoteka poput postavki ili spremljenih upita zamjenjuje tvoju. Arhiva bez ključa, ili s ključem zaštićenim master lozinkom, uvozi konekcije bez lozinki i to kaže. Tako DBA prenosi prikupljenu povijest i postavke na drugo računalo; zajedničkog spremišta između instalacija nema.

> Kopija **uključuje `key` i `connections.json`** - podatkovni ključ i tvoje enkriptirane lozinke konekcija. To je namjerno: kopija tvojih postavki bez toga nije kopija. Ali znači da je arhiva osjetljiva. Spremi je na privatno mjesto i **nikad u mapu koja se sinkronizira s uslugom u oblaku** (Dropbox, OneDrive, iCloud, Nextcloud, sinkronizirana mapa Dokumenti). Ako je postavljena master lozinka, lozinke u arhivi ostaju njome enkriptirane; bez nje su tek lagano zamaskirane.

Kopija nosi i **instalirane pluginove** (`plugins/`) i ono što oni čuvaju (`plugin-data/`) - njihove postavke i lozinke koje spremaju, enkriptirane istim `key` - pa vraćanje kopije vraća i pluginove, ispravne. Mapa u kojoj pišeš vlastite pluginove, ako ih pišeš, nije u njoj: nalazi se gdje god si je odabrao.

Izvoz veza iz upravitelja veza je nešto drugo: zapisuje samo veze i njihove grupe, bez lozinki, postavki i pluginova.

**Čišćenje podataka aplikacije** prikazuje popis s veličinom svake stavke. Izvještaji i logovi - AWR snimke, logovi poslova, spremljeni izvještaji provjera, error log - označeni su po defaultu; postavke, spremljeni upiti i SQL workspace su navedeni ali neoznačeni. Spremljene konekcije (`connections.json` + `key`) su u zasebnoj sekciji i traže upis riječi `DELETE` za potvrdu, jer njihovo brisanje gubi svaku spremljenu konekciju i njezinu lozinku. Instalirani pluginovi su navedeni neoznačeni; njihove postavke i spremljene lozinke (`plugin-data/`) su u istoj zaštićenoj sekciji kao konekcije, jer njihovo brisanje gubi te lozinke. Direktorij koji još drži pokrenuti pozadinski posao se odbija; prvo otkaži posao. Svako čišćenje se zapisuje u revizijski zapisnik.

### Pluginovi {#plugins}

**Pluginovi → Upravljanje pluginovima…** prikazuje što je instalirano: plugin možeš uključiti ili isključiti, ukloniti, ili instalirati novi iz datoteke `.dbapg-plugin`. Plugin dodaje provjere, stavke izbornika i tabove bez nove verzije DBApguina.

**Plugin može napraviti bilo tko** - aplikacija za to ima i alat (u istom tabu uključi *Razvijaj pluginove u ovoj aplikaciji*; *Vodič za pluginove* ondje objašnjava ostalo). Plugin radi s tvojim pravima: može pristupiti svakoj bazi na koju se spojiš i svakoj drugoj usluzi za koju je postavljen. **Koristi samo pluginove izdavača kojima vjeruješ.** Licencni ugovor (odjeljak 15) i izjava o privatnosti kažu isto svojim riječima.

**O pluginu** uz svaki plugin objašnjava ga u dva dijela. Prvo što plugin kaže o sebi - što radi, što čita, što šalje i kamo - na tvom jeziku kad ga donosi. Zatim što DBApguin sam pročita iz datoteka plugina, bez obzira na to što plugin kaže: izvršava li Python unutar aplikacije ili JavaScript u prozoru, koje provjere i adrese dodaje, što sprema, njegovu mapu i svaku datoteku.

**DBApguin certificiran** uz ime plugina znači da ga je potpisao autor DBApguina i da se nijedna njegova datoteka otad nije promijenila. *Promijenjen nakon certificiranja* znači da se datoteke razlikuju od potpisanih (O pluginu ih imenuje); *potpis nije valjan* znači da potpis ne vrijedi; *nije certificiran* znači da ništa ne jamči odakle dolazi - tako izgleda i plugin koji je napisao tvoj tim. Plugin koji nije certificiran i dalje se instalira i radi: oznaka ti kaže odakle plugin dolazi, ne odlučuje umjesto tebe. Instalacija plugina koji nije certificiran prvo pita, a pitanje pokazuje što bi izvršavao i svaku datoteku koju donosi.

Uklanjanje plugina briše i njegove postavke i spremljene lozinke. Sigurnosna kopija podataka aplikacije nosi pluginove i njihove postavke.

## 8. Obavijesti

Donji desni kut. Označene bojom:
- Plava (info): neutralno stanje
- Zelena (u redu): uspjeh
- Crvena (greška): operacija nije uspjela

Klik na snop obavijesti ga širi i omogućuje skrolanje. Kotačić skrola. Klik izvan ga skuplja.

Dok je otvoren prozor, obavijesti ne prekrivaju njegove gumbe. Nova obavijest stane iznad njih ili u prostor pokraj prozora, a kad nigdje nema mjesta, odmah ide u brojač, čiji broj i boja i dalje govore da je stigla. I brojač stoji iznad gumba prozora; ako ni on ne stane, skriven je dok se prozor ne zatvori. Kad se prozor zatvori, sve se vraća u kut.

Jedna radnja nudi Poništi: brisanje spremljene konekcije, koju obavijest može vratiti otprilike šest sekundi. Obavijest ne može poništiti ono što je stiglo do baze. Ono što aplikacija radi umjesto toga jest da ti prije izvršenja pokaže naredbu, pita jednom i zapiše je u audit log; postavka promijenjena ovdje može se poništiti iz prikaza postavki, a brisanje, premještanje ili promjena vlasnika ostavlja kopiju - za brisanje s redovima, kad su odabrani, a nikakvu kad odabereš brisanje bez kopije - koja vraća objekte (vidi *Kopija definicija prije promjene*).

### 8.1 Dok nešto radi {#while-something-runs}

Sve što si zatražio, a traje dulje od četvrt sekunde, to i kaže, svaki put na istim mjestima:

- Tanka traka klizi uz gornji rub prozora, a na desnom kraju reda kartica ispod izbornika vrti se kružić. Pokaži na kružić, ili dođi do njega tipkom Tab, i vidjet ćeš popis onoga što radi, korak na kojem je svaka stavka i koliko već traje; nakon tri sekunde kružić sam pokazuje vrijeme. Gdje se posao može zaustaviti - provjera, naredba u SQL editoru - njegov redak u popisu ima Odustani.
- Gumb koji si pritisnuo umjesto natpisa pokazuje kružić i ne prima drugi pritisak dok posao ne završi. Ništa se pokraj njega ne pomiče: gumb zadržava veličinu, a traka, kružić i popis ne uzimaju mjesto na stranici.
- Dio prozora koji se učitava to i kaže ("Čitam poslove…", "učitavam stupce…") umjesto da pokazuje prazan popis ili vrijednosti koje će se upravo promijeniti, a nakon tri sekunde pokazuje i koliko već čeka. Postavka čija se spremljena vrijednost još čita zatamnjena je i ne može se mijenjati dok vrijednost ne stigne.

Izvozi i izvještaji imenuju svaki korak: sastavljam izvještaj, čekam tvoj odabir u dijalogu za datoteke (ako se dijalog otvorio iza prozora, taj redak je trag), zapisujem datoteku ili PDF. Stanje rada završava čim završi i posao, uspio ili ne, a obavijest kaže kamo je datoteka otišla ili zašto nije.

Ono što aplikacija radi sama od sebe - osvježavanje nadzorne ploče, provjere konekcija, praćenje poslova, upiti kolektora - ovdje se nikad ne pokazuje, pa kružić koji se vrti uvijek znači da je nešto što si zatražio još na putu.

## 9. Prečaci na tipkovnici

- `F5` (ili `Ctrl/Cmd + Enter`): pokreni upit u SQL editoru
- `F6` (ili `Ctrl/Cmd + Shift + Enter`): izvrši samo označeno ili naredbu u kojoj je kursor
- `Ctrl/Cmd + S`: spremi trenutni SQL kao isječak
- `Ctrl/Cmd + E`: provjeri upit
- `Ctrl/Cmd + K`: paleta naredbi
- `Esc`: zatvori trenutni dijalog / kontekstni meni / očisti pretragu
- `Ctrl/Cmd + W`: zatvori trenutnu radnu karticu (ako nije prikvačena)
- `Shift + Tab`: prebacivanje tabova, kao što Alt + Tab prebacuje prozore. Otvori se prozor s karticama, po jedna za svaki otvoreni tab, svaka sa slikom sadržaja taba kakav si zadnji put vidio, a sljedeći tab je označen. Drži Shift i Tabom ili strelicama pomiči oznaku (lijevo ide natrag, gore i dolje skaču za red). Pusti Shift da otvoriš označeni tab; Esc zatvara prozor i ostaješ gdje si bio. U tekstualnom polju ili SQL editoru Shift + Tab zadržava uobičajeno značenje (prethodno polje, smanjenje uvlake); tamo koristi `Ctrl + Tab` i pusti Ctrl da otvoriš tab.
- `Ctrl + Shift + Tab`: prebacivanje konekcija u istom takvom prozoru: kartica za svaku otvorenu konekciju sa slikom onoga što je u njoj bilo na ekranu, poslužiteljem, stanjem i brojem tabova. Konekcija se prebaci kad pustiš Ctrl, ne dok gledaš. Na macOS-u to ostaje na tipki Control jer macOS Cmd + Shift + Tab drži za sebe.
- `F10` ili kratki pritisak na `Alt` (Option na Macu): otvaranje trake izbornika tipkovnicom. Lijevo i desno mijenjaju izbornik, gore i dolje stavke, desno otvara podizbornik a lijevo ga zatvara, slovo skače na sljedeću stavku koja njime počinje, Enter pokreće stavku (provjera se otvara kao na klik), Esc zatvara razinu po razinu. Izbornik otvoren mišem prihvaća iste tipke.

Tipke se prikazuju ondje gdje se koriste, s nazivima za tvoj sustav (na Macu simboli za Command, Option i Control): na gumbima SQL editora i u njihovim opisima, u ovom popisu (tipka `?`) te kao kratki savjet nakon što isto nekoliko puta napraviš mišem. Svaki se savjet pojavljuje najviše dvaput; *Bez savjeta* ih isključuje, a prekidač na dnu popisa `?` ponovno uključuje.

## 10. Napomene po pružatelju

DBApguin pri spajanju prepozna pružatelja klastera i prilagodi se:

| Pružatelj | Što je ograničeno |
|----------|---------------|
| Aurora PostgreSQL | Nema ALTER SYSTEM, nema izravnog pisanja pg_hba, nema stvaranja rola na razini superusera, nema vlastitih tablespaceova, nema izravnog čitanja datoteka |
| Amazon RDS PostgreSQL | Isto kao Aurora, plus provjera dopuštenog popisa za CREATE EXTENSION |
| Google Cloud SQL | Isto kao RDS |
| Google AlloyDB | Isto kao Cloud SQL: postavke su database flagovi instance. AlloyDB Omni koji vodite sami ima pravog superusera i vrijedi kao vlastiti PostgreSQL |
| Azure Postgres Flexible | Isto kao RDS |
| Supabase | Isto kao RDS, plus ograničenja rezerviranih rola |
| Neon | Najam s jednom bazom; nema CREATE DATABASE / DROP DATABASE |
| Heroku Postgres | Isto kao Neon |
| Crunchy Bridge | Većina enterprise funkcija je dostupna |
| DigitalOcean Managed | Isto kao RDS |
| Timescale Cloud | Dopušteni popis ekstenzija plus ograničenje role tsdbadmin |
| Vlastiti PostgreSQL | Sve je dostupno |

Sučelje zasivljuje blokirane stavke menija uz tooltip koji objašnjava zašto. Kad je rola sesije pravi superuser, ništa nije zasivljeno ni odbijeno: nijedna upravljana platforma ga ne daje korisniku, pa na tom serveru ta ograničenja ne vrijede (na primjer vlastiti server koji je uvezao imena rola neke platforme). Platforma se i dalje imenuje, a njezin natpis kaže da je ova sesija superuser. Nadzor `db-errors` se, kad je `pg_read_file` zabranjen, vraća na razlike brojača i upute za pristup logovima specifične za pružatelja.

#### Backup u .sql datoteku {#backing-them-up-as-a-sql-file}

**Skripta za micanje…** i **Skripta za vraćanje…** prvo prikažu skriptu na
ekranu, s tri izbora koje u ovoj aplikaciji ima svaka generirana skripta:
kopiraj je, spremi je ili je otvori u SQL editoru i ondje pokreni. Spremanje
odbija prepisati postojeću datoteku bez pitanja. Datoteka za vraćanje je
backup: nosi naredbe za stvaranje i korak validacije, i nju treba sačuvati
prije nego skripta za micanje krene.

Svaka datoteka počinje zaglavljem - verzija aplikacije, server i baza odakle
dolazi, opseg, vrijeme kad je uzeta i koliko constrainta sadrži. Lozinka se u
njoj ne pojavljuje nikad. Za šest mjeseci to zaglavlje je razlika između
upotrebljive datoteke i anonimne hrpe ALTER TABLE naredbi.

Za cijelu bazu umjesto sheme, `pg_dump --section=post-data` je drugi put do
istog mjesta: zapiše constrainte, indekse i okidače svega odjednom. To je
Backup → Opcije → Section → post-data.

### IAM prijave i generirana imena

Na upravljanom oblaku rola za prijavu obično **jest** IAM principal, pa je njeno
ime adresa e-pošte ili servisni račun:

```
alice@example.com
123456-compute@developer.gserviceaccount.com
```

To su valjana imena PostgreSQL rola - identifikatori u navodnicima smiju
sadržavati `@`, `.` i `-` i smiju početi znamenkom. DBApguin ih prihvaća svugdje
gdje se traži ime role: stvaranje, brisanje, dodjela, oduzimanje, članstvo,
`search_path`, vlasništvo i svaki ekran s ovlastima. SQL koji ispisuje stavlja ih
u navodnike, pa dodjela izgleda ovako:

```
GRANT SELECT ON TABLE "app-schema"."orders" TO "alice@example.com"
```

Isto vrijedi za imena baza i shema. Upravljani pružatelji rutinski generiraju
imena poput `b2b-ext-assistant-dev-pg-db`, a crtice su u identifikatoru pod
navodnicima dopuštene.

Ono što se i dalje odbija, s namjerom, jest ime koje sadrži dvostruki navodnik,
NUL, prazninu ili kontrolni znak - ništa od toga stvarni principal u oblaku
nema, a sve bi moglo prerano zatvoriti identifikator.

IAM korisnici **nemaju lozinku**. Kad ga stvarate, u čarobnjaku za rolu odaberite
`Bez lozinke`, pa iz popisa Član od dodijelite IAM članstvo pružatelja (`rds_iam`
na AWS-u, `cloudsqliamuser` na Cloud SQL-u).

## 11. Rješavanje problema

### "Lozinka je obavezna"
Spremljena veza nema pohranjenu lozinku. Najprije se pokuša bez nje: lokalni server kojem lozinka ne treba (trust, peer, `.pgpass`) jednostavno se spoji. Server koji je traži otvara dijalog za osvježavanje veze; upišite lozinku i kliknite Spoji se. Ako se kartica dok je dijalog otvoren spoji na neki drugi način, dijalog se sam zatvori.

### "Odbijam pisati unutar <putanja>"
Odredište pg_dump backupa je na mjestu samo za čitanje (AppImage mount, direktorij podataka aplikacije). Odaberite putanju ispod `~/Dokumenti` ili sličnu.

### "pg_stat_statements mora biti učitan kroz shared_preload_libraries"

pg_stat_statements ima dvije polovice i obje su nužne:

1. biblioteka mora biti navedena u `shared_preload_libraries`, i
2. ekstenzija mora biti stvorena u bazi koju ispitujete.

`CREATE EXTENSION pg_stat_statements` radi samo drugo. Ekstenzija onda postoji u
`pg_extension`, sve izgleda instalirano, a svako čitanje pogleda javlja ovu
grešku. Ponovno pokretanje `CREATE EXTENSION` je prirodna sljedeća pomisao, i ne
mijenja ništa.

DBApguin prepoznaje to stanje i kaže koja polovica nedostaje. Svaka provjera koja
čita statistiku upita ispod greške ispisuje razlog i rješenje, a
**db-setup-advisor** to javlja kao *"Stvoreno u ovoj bazi, ali NIJE u
shared_preload_libraries"*, a ne kao obično "nije instalirano".

Rješenje, kao superuser:

```
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';
```

pa **restartajte** PostgreSQL. `shared_preload_libraries` se čita samo pri
pokretanju postmastera - `SELECT pg_reload_conf()` ga neće pokupiti. Na
upravljanom oblaku postavite ga u parameter grupi i restartajte instancu.

Obrnuto stanje - biblioteka učitana, ekstenzija nije stvorena - ne traži ni
restart ni intervenciju: provjere same stvore ekstenziju i nastave.

Provjere pogođene dok je konfiguracija napola: db-stat-statements,
db-index-advisor (samo njegova pg_stat_statements heuristika), db-temp-queries,
db-workload-report, db-awr-snapshot i db-trace-session.

### Indeks postoji, a planer ga ignorira {#leftover-damage}

Obično to znači da je indeks NEVAŽEĆI, a u običnom popisu to ništa ne kaže.
`CREATE INDEX CONCURRENTLY` i `REINDEX CONCURRENTLY` grade u fazama, i sesija
koja pogine na pola - rušenje, prekid, istek ključa - ostavlja napola izgrađen
indeks. Planer ga neće koristiti, i dalje zauzima prostor, a `\d` ga označi
samo ako dobro pogledate.

**Cluster health → Provjeri ostatke oštećenja...** traži stanja koja
PostgreSQL bilježi, ali ih ne ističe:

- nevažeće indekse, od prekinute CONCURRENTLY izgradnje
- `NOT VALID` ograničenja, koja se ne provode nad postojećim redcima - vraćanje
  iz backupa koje je odgodilo provjeru ostavlja ovakva
- onemogućene okidače, koje `--disable-triggers` vraćanje ostavi isključenima
  ako je prekinuto

Odaberite shemu ili ostavite *(cijela baza podataka)*. Uz svaki nalaz dolazi
naredba koja ga popravlja, a nevažeći indeks dobiva gumb **Popravi sada** koji
nad njim pokreće `REINDEX INDEX CONCURRENTLY` kao pozadinski posao.

Jedno očekujte: neuspjeli `REINDEX CONCURRENTLY` ostavlja vlastitu napola
izgrađenu kopiju sa sufiksom `_ccnew`, koja je zaseban objekt od izvornog.
Popravak izvornog ne uklanja ju - pokrenite provjeru ponovno i popravite i nju.

### Prozor se ne pojavljuje (Linux)
DBApguin otvara dijalog s naredbama za instalaciju po distribuciji kad nedostaje WebKit2GTK. Odaberite karticu svoje distribucije, kliknite "Kopiraj naredbu", zalijepite u terminal pa ponovno pokrenite AppImage. Dijalog nikad ne pribjegava web pregledniku - DBApguin radi isključivo samostalno.

### Prozor se ne pojavljuje (Windows)
DBApguin otvara dijalog s tri načina instalacije Microsoft Edge WebView2 Runtimea: priloženi bootstrapper iz mape aplikacije, online instaler i upute za postavljanje u poduzeću (GPO/SCCM/Intune).

### Prozor se zaglavi pri zatvaranju (Windows)
Trebalo bi biti riješeno od builda iz lipnja 2026. Aplikacija se prisilno gasi 2 sekunde nakon klika na X. Ako se i dalje zaglavljuje, priložite sadržaj `<DBCHECKS_HOME>\dbchecks_error.log` uz prijavu greške.

### DBApguin se ne pokreće: port je zauzet
- Kad drugi program već koristi DBApguinov port, prozor ostaje otvoren i kaže
  to, s poljem za pokretanje na drugom portu - slobodan je već upisan. DBApguin
  zadržava taj port; kasnije ga promijenite u **Postavke -> Pokretanje**.
- Kad kaže da DBApguin već radi, ranija kopija je još tu: prebacite se na
  njezin prozor ili zaustavite proces koji navodi i odaberite **Pokušaj ponovno**.

### Pregled sheme je prazan
- Jeste li spojeni? Provjerite točku stanja uz vezu.
- Jeste li promijenili bazu? Kliknite drugu bazu pod **Baze podataka** da se prebacite; stablo sheme ponovno se učitava s njom.
- Zadržite pokazivač na zaglavlju **Baze podataka**, na bazi ili na shemi i kliknite ikonu osvježavanja koja se pojavi, ili upotrijebite **Shema -> Ponovno učitaj stablo shema**.

### Nema revizijskog zapisa
- Uključen je dok se ne isključi: provjerite prekidač u Prikaz -> Zapisnik revizije
- Ili provjerite ključ `audit_enabled` u `app_prefs.json` u svom direktoriju podataka
- Datoteke su u `<DBCHECKS_HOME>/audit/audit-YYYY-MM.jsonl`

### Gdje su podaci aplikacije
Na Linuxu: `~/.local/share/DBApguin/`. Nadjačajte s `DBCHECKS_HOME=/putanja .venv/bin/python launcher.py`.

Na Windowsima: po zadanom mapa uz `DBApguin.exe`.

## 12. Pregled podatkovnih datoteka

| Datoteka | Što drži | Prava |
|------|--------------|------|
| `connections.json` | Spremljene veze s lozinkama šifriranima Fernetom | 0644 |
| `key` | Slučajan ključ za šifru lozinki, jedinstven po instalaciji | 0600 |
| `app_prefs.json` | Postavke sučelja (font_size_px, audit_enabled, theme i ostalo) | 0644 |
| `certs/` | Učitani TLS korijenski certifikati | svaki 0600 |
| `audit/audit-YYYY-MM.jsonl` | Revizijski zapis DDL-a, jedna datoteka po mjesecu | 0600 |
| `dbchecks_error.log` | Zapis tragova grešaka (rotira na 1 MiB, čuva 3 kopije) | 0644 |
| `webview/` | pywebview localStorage / kolačići | (samo Linux) |

Napravite kopiju ovoga prije ponovne instalacije. Ponovna instalacija DBApguina u isti direktorij podataka čuva vaše veze.

## 12.4 Gdje je dokumentacija

Tri puta, svi iz **Pomoć** u traci menija:

| | |
|---|---|
| **Korisničke upute** | ovaj dokument, isporučen s aplikacijom - radi bez interneta |
| **Dokumentacija (online)** | web stranica, za sve novije od vašeg builda |
| **O DBApguinu** | verzija i build, s oba prethodna uz sebe |

Panel licence pri prvom pokretanju i napomena nakon njega vode na ovo, a tamo
većina ljudi prvi put i pogleda.

Dokumenti se prikazuju unutar aplikacije, u njenom vlastitom stilu - naslovi,
tablice i SQL primjeri u pravom bloku koda, a ne kao prelomljen tekst.

Postoji i četvrti put, i njega ćete u praksi koristiti najviše: malo zaokruženo
**i** uz naslov dijaloga. Otvara ove upute na poglavlju koje opisuje taj dijalog,
pa je objašnjenje jedan klik od stvari koju objašnjava, a ne pretraga daleko.
Naći ćete ga na dijalozima backupa i vraćanja, na preimenovanju i spajanju shema
te uz način transakcije u SQL editoru.

Priložena kopija je ona koja dolazi s vašim buildom, pa uvijek odgovara verziji
koju koristite. Online kopija može biti ispred nje.

## 12.5 Pravno, privatnost i softver trećih strana

Sve niže dostupno je iz **Pomoć** u traci menija i isporučuje se s aplikacijom - 
datoteka koju aplikacija prikazuje je datoteka s kojom je izgrađena.

- **Licencni ugovor (EULA)** - što smijete, a što ne smijete raditi sa
  softverom, odricanje od jamstva i ograničenje odgovornosti. Odjeljak 7 vrijedi
  pročitati i ako preskočite ostalo: govori o tome da ovaj alat na vašu naredbu
  šalje `DROP`, `TRUNCATE` i `pg_terminate_backend` na žive poslužitelje.
- **Izjava o privatnosti** - što se sprema na vaše računalo i gdje. Kratka
  verzija: sve je lokalno, nema telemetrije, a jedine mrežne veze su prema vašim
  bazama i prema `127.0.0.1`.
- **Napomene o trećim stranama** - svaka open-source komponenta koju build
  povezuje, s njenom licencom. Generirano iz samog builda
  (`tools/legal/gen_third_party.py`), a ne održavano ručno, pa ne može opisivati
  skup ovisnosti koji više ne postoji.

Pravni dokumenti pisani su na engleskom i engleski je mjerodavan. Prijevodi
sučelja na njih se namjerno ne protežu: prevedeni licencni ugovor koji nije
prošao pravnika u tom jeziku gori je od nikakvog prijevoda, jer izgleda
mjerodavno.

## 12.6 Zašto svugdje izgleda isto

DBApguin isporučuje dva fonta koja koristi - Inter za sučelje i Cascadia Mono za
sve monospace - umjesto da ih imenuje i nada se da ih stroj ima.

To nije kozmetika. Alat koji prikazuje identifikatore mora ih prikazati točno:
kad je aplikacija imenovala font koji nije bio instaliran, sustav je podmetnuo
drugi, a jedna zamjena crtala je donju crtu ispod okvira retka, pa se
`pg_read_all_stats` na jednom stroju čitalo kao `pgreadallstats`, a na drugom
ispravno. Isporuka fonta uklanja nagađanje.

Oba su pod SIL Open Font License 1.1 i putuju neizmijenjena, s tekstovima licenci
u `static/fonts/` i u **Pomoć → Komponente trećih strana**.

## 13. Dolazite s Oraclea {#coming-from-oracle}

Ako godinama vodite Oracle baze, većina onoga što znate prenosi se. Ovdje je dio
koji se ne prenosi i gdje u ovoj aplikaciji žive brojke koje su vam poznate.

### 13.1 Preslikavanje

| Oracle | PostgreSQL | Gdje u DBApguinu |
| --- | --- | --- |
| Instanca + baza (obično jedna po instanci) | Klaster + više baza; jedna skupina procesa poslužuje sve | Bočna traka Serveri; svaka baza je čvor |
| Shema == korisnik | Shema i rola su odvojeni objekti; jedna rola može posjedovati više shema | Pregled sheme; prikaz Role |
| Tablespace (obavezan, dimenzioniran) | Tablespace (neobavezan, samo direktorij) | `db-tablespaces` |
| Undo tablespace, rollback segmenti | Nema undo: stare verzije redaka ostaju u tablici dok ih vacuum ne makne | Savjetnik za autovacuum, `db-bloat-breakdown` |
| Redo i arhivski logovi | WAL segmenti + arhiviranje | `db-checkpoints`, `db-settings` (WAL i arhivski GUC-ovi) |
| SGA / buffer cache | `shared_buffers` + predmemorija stranica u OS-u | `db-settings`, `db-cache-advisor`, `db-buffer-cache` |
| PGA / prostor za sortiranje | `work_mem`, po operaciji po backendu | `db-settings` (UPOZ tamo obično je o tome) |
| AWR izvještaj | `db-workload-report` nad razlikama iz pg_stat_statements | Meni Pohrana i performanse |
| AWR snimka / baseline | `db-awr-snapshot` (take / list / compare) | Meni Pohrana i performanse |
| ASH / v$active_session_history | `db-ash` uzorkivač nad pg_stat_activity | Meni Konekcije i upiti |
| v$session | Prikaz Sesije; klik na redak daje pun uvid | Prikaz Sesije |
| v$sql / library cache | `pg_stat_statements`, `db-stat-statements` | Meni Konekcije i upiti |
| SQL trace / 10046 | `db-trace-session` uz `auto_explain` na poslužitelju | Meni Podešavanje i dijagnostika, *Praćenje sesije* |
| DBMS_STATS | `ANALYZE`, analitička polovica autovacuuma | Kartica Savjetnik na tablici |
| Hintovi optimizatoru | Nema hintova. Mijenjate statistiku, indekse ili upit | `db-index-advisor`, EXPLAIN u SQL prikazu |
| RMAN | `pg_dump` / `pg_restore` (logički), `pg_basebackup` (fizički) | Backup i vraćanje |
| Data Guard | Streaming replikacija + replikacijski slotovi | `db-replication`, `db-repl-slots` |
| Osvježavanje materijaliziranog pogleda (fast/complete) | `REFRESH MATERIALIZED VIEW [CONCURRENTLY]`, uvijek potpuna izgradnja | Desni klik na matview |
| DBA_* / V$ pogledi | `pg_catalog.pg_*` i `pg_stat_*` pogledi | Svaka provjera ispisuje SQL koji je pokrenula |
| Neispravni objekti, recompile | Takvog stanja nema: funkcija se parsira kad se izvodi | provjera nakon uvoza, pokreće se automatski nakon restorea |
| Javni sinonimi | `search_path` ili pogled u zajedničkoj shemi | `db-settings` |
| Sekvence s NOCACHE / ORDER | Sekvence keširaju po sesiji; praznine su normalne i nisu greška | `db-sequences`, provjera zdravlja sekvenci |
| Profili, resource manager | Nije ugrađeno. Ograničenja veza po roli, plus pooler | `db-connections` |

### 13.2 Pet razlika koje stvarno bole

**1. Nema undo-a - vacuum je cijena MVCC-a.** `UPDATE` zapisuje novu verziju retka
i staru ostavlja iza sebe, mrtvu ali i dalje na stranici. Autovacuum mrtve verzije
briše kasnije. Dvije posljedice bez Oracle pandana: tablica može rasti i kad broj
redaka ne raste (napuhanost), a transakcija ostavljena otvorenom satima sprječava
vacuum da očisti *ijednu* tablicu, jer bi stare verzije njoj još mogle biti
vidljive. `db-long-tx` i `db-idle-in-tx` su provjere koje ih nalaze; kartica
Savjetnik na tablici pokazuje točno koliko mrtvih redaka treba da autovacuum
krene.

**2. Sheme nisu korisnici.** `CREATE USER app` ne stvara mjesto gdje će `app`
odlagati tablice. Objekti završe u prvoj shemi iz `search_path` - obično
`public`. Dodijelite `USAGE` na shemi *i* ovlasti nad objektima: to su dvije
odvojene stvari, a nedostajući `USAGE` na shemi najčešći je "permission denied"
koji izgleda neobjašnjivo. Prikaz Role pokazuje oba sloja, a uređivač ovlasti
piše oba.

**3. DDL je transakcijski.** `BEGIN; ALTER TABLE ...; ROLLBACK;` doista poništava
`ALTER`. Migracije mogu biti sve-ili-ništa, što je stvarna prednost - ali znači i
da migracija koja drži `ACCESS EXCLUSIVE` na prometnoj tablici blokira svakog
čitatelja dok se ne potvrdi. `db-locks` i prikaz Sesije pokazuju tko čeka iza koga.

**4. Nema hintova, a statistika je cijela priča.** Ako je plan pogrešan, odgovor je
`ANALYZE`, bolji indeks, prilagođen `default_statistics_target` ili prepisan upit.
Kartica Savjetnik javlja kad je tablica zadnji put analizirana; `db-index-advisor`
javlja indekse koji nedostaju, uključujući strane ključeve bez indeksa - a to u
PostgreSQL-u znači da brisanje u roditeljskoj tablici skenira dijete.

**5. Veze su procesi, ne dretve.** Svaka veza je proces operacijskog sustava s
vlastitom memorijom. Nekoliko stotina besposlenih veza troši stvarni RAM i
usporava svaku snimku `pg_stat_activity`. Gdje bi Oracle koristio shared servere,
PostgreSQL koristi vanjski pooler (PgBouncer). `db-connections` pokazuje koliko
ste blizu `max_connections` i koliko ih je besposleno.

### 13.3 Stvari bez Oracle pandana

- **TOAST.** Vrijednosti šire od otprilike 2 KB komprimiraju se i spremaju u
  pomoćnu tablicu. Vlastita veličina tablice može izgledati malena dok njena TOAST
  tablica drži većinu podataka; `db-bloat-breakdown` upravo zato razdvaja heap,
  TOAST i indekse.
- **Ekstenzije.** Mnogo toga što je u Oracleu ugrađeno ovdje je modul koji se
  instalira: `pg_stat_statements`, `pgcrypto`, `postgis`. Vidi poglavlje 5.14.
- **Prelijevanje broja transakcije.** 32-bitni brojač transakcija mora se
  povremeno zamrznuti ili klaster odbija pisanje. `db-vacuum` javlja
  starost; to je jedini autovacuum posao koji nikad ne smijete isključiti.
- **`NULL` i prazan string nisu isto.** Oracle `''` tretira kao `NULL`;
  PostgreSQL ne. Podaci migrirani s Oraclea tu razliku često nose tiho.

## 14. PostgreSQL za developere {#postgresql-for-developers}

Ne morate postati DBA da bi PostgreSQL baza bila zdrava. Treba vam jedan mentalni
model i pet navika.

### 14.1 Model: retci su verzije

Svaki `UPDATE` i `DELETE` ostavlja staru verziju retka na mjestu, označenu mrtvom.

Čitatelji nikad ne blokiraju pisce, jer svaka transakcija vidi verzije koje su
bile aktualne kad je počela. Iz toga slijede dvije stvari, a gotovo sve u ovoj
aplikaciji tiče se jedne od njih:

- **Mrtvi retci se gomilaju** dok ih autovacuum ne makne. Do tada zauzimaju
  prostor i usporavaju čitanja. To je *napuhanost*.
- **Vacuum može maknuti samo verzije koje nijedna aktivna transakcija više ne
  može vidjeti.** Jedna zaboravljena otvorena transakcija zamrzava čišćenje cijele
  baze.

`ANALYZE` je druga polovica: ponovno uzorkuje podatke da planer zna koliko će
redaka uvjet pogoditi. Zastarjela statistika uobičajen je uzrok rečenice "ovaj
upit je prošli tjedan bio brz".

### 14.2 Kad netko kaže "baza je spora"

1. **Prikaz Sesije.** Koliko veza, koliko ih doista radi, je li išta blokirano.
   Blokiran upit je problem zaključavanja, ne performansi.
2. **`db-health`.** Jedan ekran: veze, pogodak predmemorije, duge transakcije,
   napuhanost, lockovi.
3. **`db-stat-statements`.** 25 upita koji su potrošili najviše vremena. Odgovor je
   gotovo uvijek ovdje - obično jedan upit pozvan daleko češće nego što je itko
   očekivao.
4. **Napravite EXPLAIN upita** u SQL prikazu. Tražite `Seq Scan` na velikoj tablici
   i procjenu broja redaka koja se jako razlikuje od stvarne - ovo drugo znači da
   je statistika zastarjela.
5. **`db-index-advisor`.** Indeksi koji nedostaju i neindeksirani strani ključevi.

### 14.3 Pet navika

1. **Ne ostavljajte transakcije otvorenima.** ORM sesija koja je otvorila
   transakciju pa otišla na ručak najštetnija je pojedinačna stvar koju developer
   može učiniti PostgreSQL bazi. `db-idle-in-tx` ih nalazi.
2. **Indeksirajte strane ključeve.** PostgreSQL indeks ne stvara umjesto vas, a
   bez njega svako brisanje ili ažuriranje roditelja skenira tablicu djeteta.
3. **`ANALYZE` nakon masovnih punjenja.** Milijun redaka umetnut bez statistike iza
   sebe davat će loše planove dok autovacuum ne stigne.
4. **Radite u serijama, ali ne prevelikima.** Jedna transakcija koja briše deset
   milijuna redaka drži lockove, generira WAL i ostavlja deset milijuna mrtvih
   verzija. Vrtite u serijama od nekoliko tisuća i potvrđujte.
5. **Koristite connection pool.** Otvaranje veze po zahtjevu košta fork procesa.
   Pročitajte `db-connections` prije nego podignete `max_connections`.

### 14.4 Riječi koje ova aplikacija koristi {#the-words-this-app-uses}

- **Napuhanost (bloat)** - prostor koji drže mrtve verzije redaka i koji se nikad
  ne vraća operacijskom sustavu.
- **Mrtvi tuple** - stara verzija retka koja čeka vacuum. "Tuple" ovdje znači
  "redak".
- **Autovacuum** - pozadinski proces koji miče mrtve tuple i osvježava statistiku.
  Okida se po tablici, na pragovima koje vidite u kartici Savjetnik.
- **VACUUM naspram VACUUM FULL** - obični `VACUUM` označava prostor kao ponovno
  upotrebljiv i nikad ne blokira čitatelje. `VACUUM FULL` prepisuje cijelu tablicu,
  vraća prostor operacijskom sustavu i drži lock koji blokira *sve* nad tom
  tablicom dok traje. `REINDEX CONCURRENTLY` i `pg_repack` su online alternative.
- **WAL** - write-ahead log. Svaka promjena prvo se upisuje tamo; iz njega čitaju i
  replikacija i oporavak nakon pada.
- **Checkpoint** - povremeno spuštanje prljavih stranica u podatkovne datoteke.
  Prečest znači skokove u pisanju; `db-checkpoints` javlja omjer vremenskih i
  traženih.
- **search_path** - popis shema u kojima se traži nekvalificirano ime. On odlučuje
  gdje završi `CREATE TABLE x` i koja se funkcija `f()` razriješi.
- **Rola** - jedinstven pojam iza korisnika i grupa. Rola s `LOGIN` je ono što
  druge baze zovu korisnikom.
- **Praznine u sekvencama su normalne.** Sekvence nisu transakcijske: poništeni
  insert je svoj broj ipak potrošio.

## Prijava greške

**Postavke -> Prijavi grešku** sastavi izvještaj za tebe: ID greške, verziju,
commit, tvoj OS i zadnjih četrdeset redaka zapisa grešaka. Ti ispuniš tri
odjeljka - što si radio, što si očekivao, što se umjesto toga dogodilo - i
pritisneš jedan od tri gumba.

**Otvori mail** preda tekst tvom mail programu, adresiran na
**dbapguin@gmail.com**. Aplikacija sama ne šalje ništa i nikad nije; prvo vidiš
točan tekst, a Pošalji pritišćeš ti. **Kopiraj izvještaj** i **Spremi u
datoteku** su tu ako ga radije želiš priložiti ili zalijepiti drugdje.

**Što se uklanja prije nego išta napusti tvoje računalo**: lozinke, korisnička
imena, imena baza, imena hostova, connection URI-ji, IP adrese i putanja tvoje
kućne mape. Dijalog popiše što je maknuo, da to vidiš umjesto da vjeruješ na
riječ.

Navedi ID greške (`DBAP-261011-K4QP` i slično) u svakoj daljnjoj poruci - po
njemu se jedan izvještaj razlikuje od drugog.

## 15. Kako doći do pomoći

- Interna dokumentacija: ova datoteka plus `docs/TECHNICAL.md` i `ARCHITECTURE.md`
- Podrijetlo builda: **Pomoć -> O DBApguinu** prikazuje verziju, git commit, vrijeme builda i Python s kojim je građen. Priložite to uz prijavu greške.
- Revizijski zapis: `<DBCHECKS_HOME>/audit/audit-YYYY-MM.jsonl` je JSON Lines datoteka. Otvorite je bilo kojim uređivačem teksta ili s `jq` za analizu.
- Zapis grešaka: `<DBCHECKS_HOME>/dbchecks_error.log` za najnovije tragove grešaka sa strane poslužitelja. Lozinke se uklanjaju prije zapisa. Rotira na 1 MiB i čuva tri starije kopije, a svako pokretanje koje išta zapiše prvo ispiše zaglavlje - pa su zapisi nakon zadnjeg retka `=== DBApguin ... started ...` oni iz vaše trenutne sesije. Oni iznad mogu biti stari tjednima.

Za komercijalnu podršku javite se proizvođaču uz:
1. Verziju builda (iz Pomoć -> O DBApguinu).
2. Ključ pružatelja (iz detalja veze).
3. Verziju PostgreSQL poslužitelja.
4. Kopiju relevantnog dijela revizijskog zapisa i zapisa grešaka ako ih imate.
