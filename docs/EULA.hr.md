# DBApguin - Licencni ugovor s krajnjim korisnikom

**Verzija 1.0.0 - 2. studenoga 2026.**

---

Ovo je izvorni tekst Ugovora. Engleski tekst je prijevod radi lakšeg
razumijevanja; u slučaju neslaganja mjerodavan je ovaj hrvatski tekst
(odjeljak 18). Softver prikazuje oba teksta (Pomoć, Ugovor o licenci).

Verzija ovog Ugovora je verzija DBApguina u kojoj je zadnji put izmijenjen.

**DBApguin je besplatan za sve**, za
privatnu i za komercijalnu upotrebu jednako. Nema licencnog ključa, aktivacije,
probnog razdoblja ni uskraćenih funkcija.

Ovaj je Ugovor kratak jer se ništa ne kupuje: nema kupnje, broja računala ni
trajanja licence koje bi trebalo urediti.

## 1. Ugovor

Ovaj Licencni ugovor s krajnjim korisnikom ("Ugovor") sklapaju vi ("Korisnik",
bilo fizička ili pravna osoba) i Darko Lovrenčec, autor DBApguina ("Autor").

Softver prikazuje ovaj Ugovor pri prvom pokretanju i ne otvara se dok ga ne
prihvatite. Prihvaćanjem na tom mjestu, ili na drugi način instaliranjem ili
korištenjem DBApguina ("Softver"), prihvaćate ovaj Ugovor. Ako ga ne
prihvaćate, odbijte ga na tom mjestu i ne koristite Softver.

Vaše se prihvaćanje bilježi lokalno - prihvaćena verzija i datum. Ništa od toga
nikamo se ne šalje. Odjeljak 16 uređuje što se događa kada kasnija verzija
izmijeni ovaj Ugovor.

## 2. Što je predmet licence

Softver se **licencira, a ne prodaje**, i licencira se **bez naknade**. Autor
zadržava sva prava, vlasništvo i interese u Softveru, uključujući njegov
izvorni kod, dizajn i dokumentaciju.

Besplatno nije isto što i javno dobro, i nije licenca otvorenog koda. Softver
smijete koristiti bez plaćanja. Ne smijete ga prodavati, licencirati pod drugim
uvjetima niti ga predstavljati kao svoje djelo. Odjeljak 4 je potpuni popis
tih ograničenja.

DBApguin je objavljen kako bi ljudima pomogao u svakodnevnom radu s bazama
podataka. Sve što nudi - provjere, analize, preporuke, upozorenja, potvrde,
procjene i kopije napravljene prije promjene - nudi se prema najboljem
nastojanju, a ne kao jamstvo (odjeljak 10).

Nema kupnje, pa nema ni računa, preprodavača, povrata novca ni pretplate koju
bi trebalo otkazati.

Ovaj Ugovor obuhvaća DBApguin 1.0.x: svako izdanje čiji broj verzije
počinje s 1.0.

## 3. Davanje licence

Autor vam daje neisključivu, svjetsku, besplatnu i trajnu licencu za
instaliranje i korištenje Softvera:

- na **koliko god računala želite**, osobno ili unutar svoje organizacije;
- u **bilo koju svrhu**, uključujući komercijalni i produkcijski rad;
- **bez ograničenja** broja spremljenih konekcija, otvorenih kartica, baza
  podataka, korisnika ili funkcija.

Softver smijete dalje distribuirati **neizmijenjen i potpun**, zajedno s ovim
Ugovorom, pod uvjetom da za to ništa ne naplaćujete i da ga ne predstavljate
kao svoj. Zrcaljenje službenog preuzimanja je dobrodošlo.

## 4. Što ne smijete

- Prodavati Softver, naplaćivati pristup njemu ili ga uključiti u plaćeni
  proizvod ili plaćenu uslugu bez pisanog dopuštenja Autora.
- Licencirati ga pod drugim uvjetima, davati podlicence ili ga distribuirati
  pod drugačijim uvjetima.
- Uklanjati, mijenjati ili prikrivati oznake autorstva, autorskog prava ili
  licence, ili predstavljati Softver kao djelo bilo koga osim Autora.
- Distribuirati izmijenjenu verziju pod imenom DBApguin, ili na način koji
  sugerira da ju je Autor izradio ili odobrio.
- Dekompilirati, rastavljati ili na drugi način provoditi obrnuti inženjering
  Softvera kako biste izradili konkurentski proizvod, uklonili njegovo
  autorstvo ili distribuirali ono što ste time dobili.

Posljednja točka odnosi se na zaobilaženje licence, a ne na razumijevanje
Softvera. Kada vam članak 6. Direktive 2009/24/EZ ili odgovarajuće prisilno
pravo daje pravo na dekompiliranje radi **interoperabilnosti** s drugim
programom, to pravo vrijedi i ništa ga ovdje ne ograničava.

Korištenje Softvera u plaćenom radu koji obavljate za klijenta uobičajena je
komercijalna upotreba i izričito je dopušteno. Naplaćivanje samog DBApguina
klijentu nije.

## 5. Vaši podaci i vaše baze podataka

Softver radi u potpunosti na vašem računalu i spaja se samo na poslužitelje
baza podataka koje sami podesite, te na adrese do kojih su podešeni dodaci koje
instalirate (odjeljak 15). **Ne šalje ništa Autoru** - nema telemetrije,
izvještavanja o korištenju ni provjere licence preko mreže. Što Softver sprema
lokalno opisuje [obavijest o privatnosti](PRIVACY.md).

Vaše baze podataka, vaše sheme i rezultati vaših upita pripadaju vama. Autor ih
ne prima i nema nikakav pristup njima.

## 6. Softver izvršava naredbe nad vašim bazama podataka

Ovaj dio Ugovora vrijedi pročitati dvaput.

DBApguin je alat za administraciju baza podataka. Izvršava SQL - uključujući
DDL kao što su CREATE, ALTER, DROP, REINDEX i VACUUM, te operacije održavanja
koje mogu zaključati tablice ili dugo trajati - nad poslužiteljima na koje ga
usmjerite. Nastoji vam pokazati što će izvršiti i tražiti potvrdu za
destruktivne operacije, ali **možda neće prepoznati svaku naredbu koja nešto
mijenja, zaključava ili uklanja**: njegova upozorenja i potvrde pomoć su koja se
nudi prema najboljem nastojanju, a ne jamstvo. **Vi odlučujete što se izvršava i
nad kojim poslužiteljem**.

Vi ste odgovorni za:

- odabir prave konekcije i znanje je li riječ o produkciji;
- postojanje **ispravne, provjerene sigurnosne kopije** prije nego što
  promijenite bilo što što si ne možete priuštiti izgubiti;
- posljedice naredbi koje potvrdite, uključujući gubitak podataka, prekid
  rada, sukobe zaključavanja i učinke na replikaciju.

Softver je alat u vašim rukama, a ne zaštita od vaših namjera. Ništa u njemu ne
uklanja potrebu da znate što će naredba napraviti prije nego što je izvršite.

## 7. Revizijski zapisnik

Softver vodi revizijski zapisnik osim ako ga ne isključite (Prikaz, Zapisnik
revizije). U lokalnu datoteku dodaje redak za svaku naredbu izvršenu iz SQL
editora - osim ako je kartica postavljena samo za čitanje - i za svaku promjenu
napravljenu vlastitim alatima Softvera: vrijeme, cilj, vaše korisničko ime u
operativnom sustavu i korisnika baze podataka, naredbu sa skrivenim lozinkama
te ishod. [Obavijest o privatnosti](PRIVACY.md) navodi gdje se datoteka nalazi
i što još bilježi.

Postoji kako bi pitanje "tko je ovo promijenio i kada" imalo odgovor koji
možete potražiti. **To je pomoć vama, a ne certificirani zapis za
usklađenost**: riječ je o lokalnoj datoteci; njezini su zapisi povezani
hashom, što otkriva usputnu izmjenu ili izbrisan redak, ali svatko tko može
pristupiti datoteci može prepisati i nju i lanac; i bilježi samo ono što je
napravio sam Softver.

Ne oslanjajte se na nj kao na forenzički dokaz, kao na jedini zapis za
regulatornu ili ugovornu obvezu, ni kao na dokaz onoga što je napravio netko
drugi. Kada vam treba mjerodavan zapis, koristite vlastito bilježenje
poslužitelja baze podataka i kontrole svoje organizacije.

## 8. Ažuriranja i promjene Softvera

Autor može izdavati ažuriranja - ispravke, poboljšanja i nove funkcije - i ne
obvezuje se izdavati ih po ikakvom rasporedu, niti uopće. Ažuriranje je dio
Softvera i na njega se primjenjuje ovaj Ugovor. Kada ažuriranje dolazi s
izmijenjenom verzijom ovog Ugovora, primjenjuje se odjeljak 16 i Softver traži
vaš pristanak prije nego što ažuriranje stupi na snagu.

Softver je u aktivnom razvoju, a ne gotov isporučeni proizvod. Između verzija
Autor može dodavati, mijenjati, napuštati ili uklanjati funkcije, mijenjati
komponente trećih strana koje koristi (odjeljak 13), te u bilo kojem trenutku
prestati razvijati, podržavati ili distribuirati Softver ili bilo koji njegov
dio. Ništa od toga nije povreda ovog Ugovora.

Autor ne zadire u verziju koju već koristite kako bi iz nje uklonio neku
funkciju, a Softver nema daljinsko gašenje. Primjerak koji ste instalirali
nastavlja raditi pod svojim uvjetima, što god Autor kasnije učinio. Prestanak
razvoja zaustavlja nove verzije i novu podršku; ne dira primjerak koji je već
na vašem računalu.

## 9. Vaša obveza obeštećenja Autora

Odjeljak 11 uređuje odgovornost između vas i Autora. Ovaj odjeljak odnosi se na
treću stranu.

Branit ćete i obeštetiti Autora te ga oslobađate od svakog zahtjeva, potraživanja,
gubitka, odgovornosti, štete, troška ili izdatka (uključujući razumne troškove
pravnog zastupanja) koji podnese treća strana, a proizlazi iz:

1. vašeg korištenja Softvera, uključujući svaku naredbu koju je on po vašem
   nalogu izvršio nad bazom podataka (odjeljak 6);
2. vaše povrede ovog Ugovora;
3. vaše povrede ili protupravnog prisvajanja prava treće strane; ili
4. podataka s kojima ste uz pomoć Softvera postupali, koje ste otkrili,
   izmijenili ili uništili.

Autor će vas bez odgode obavijestiti o svakom takvom zahtjevu, prepustiti vam
vođenje obrane i nagodbe - osim što nagodba kojom se priznaje krivnja Autora ili
mu se nameće bilo kakva obveza zahtijeva prethodni pisani pristanak Autora - i
razumno surađivati s vama o vašem trošku.

Ovaj se odjeljak ne primjenjuje na zahtjev u mjeri u kojoj je uzrokovan
Autorovom vlastitom povredom ovog Ugovora. Ako ste potrošač, ovaj vas odjeljak
obvezuje samo u mjeri u kojoj je takvo obeštećenje provedivo prema potrošaču
po prisilnom pravu mjesta u kojem živite.

## 10. Bez jamstva

**Softver se isporučuje "takav kakav jest", bez ikakvog jamstva**, izričitog
ili prešutnog, uključujući, ali ne ograničavajući se na, prešutna jamstva
prikladnosti za prodaju, prikladnosti za određenu svrhu i nepovređivanja prava.

Autor ne jamči da je Softver bez nedostataka, da će raditi bez prekida, da su
njegove analize i preporuke točne za vaše opterećenje, niti da je prikladan za
bilo koju određenu bazu podataka, verziju ili konfiguraciju.

Ono što ovaj Ugovor, korisničke upute i Softver kažu o tome što Softver radi -
da upozorava prije destruktivne naredbe, da traži potvrdu prije nego što je
izvrši, da kopija napravljena prije promjene može vratiti stanje, da je
procjena, nalaz ili preporuka točna - opisuje ono za što je Softver zamišljen,
prema najboljem nastojanju Autora. To nije obećanje da će tako uvijek raditi, na
svakom poslužitelju, verziji i opterećenju. Provjerite ono što je važno prije
nego što se na to oslonite.

Kada vam prisilno potrošačko pravo zemlje u kojoj imate prebivalište daje prava
koja se ne mogu isključiti, ovaj odjeljak ta prava ne dira.

## 11. Ograničenje odgovornosti

U najvećoj mjeri koju zakon dopušta, Autor **nije odgovoran** za neizravnu,
slučajnu, posebnu, posljedičnu ili kaznenu štetu, ni za gubitak podataka,
gubitak dobiti, gubitak prihoda, prekid poslovanja ili trošak zamjenskog
softvera ili usluga, koji proizlaze iz Softvera ili ovog Ugovora ili su s njima
povezani, bez obzira na uzrok i na pravnu osnovu odgovornosti.

Budući da se Softver isporučuje besplatno, za njega nije plaćen nikakav iznos,
pa je ukupna odgovornost Autora po ovom Ugovoru **ograničena na nulu** - osim
kada se odgovornost ne može zakonito ograničiti, osobito za smrt ili tjelesnu
ozljedu prouzročenu nepažnjom, za prijevaru i za sve drugo što prisilno pravo
izuzima iz dosega ugovora.

## 12. Trajanje i prestanak

Ova licenca stupa na snagu kada prihvatite ovaj Ugovor i traje neograničeno.

Prestaje automatski ako povrijedite odjeljak 4. Po prestanku morate prestati
koristiti Softver i ukloniti ga sa svojih računala. Odjeljci 5, 6, 7, 9, 10,
11, 17 i 18 ostaju na snazi i nakon prestanka.

Prestanak razvoja uređen je odjeljkom 8: on ne dovodi do prestanka vaše
licence za primjerak koji već imate.

## 13. Komponente trećih strana

Softver sadrži komponente otvorenog koda trećih strana, svaku pod vlastitom
licencom. Te licence uređuju te komponente i navedene su u
[obavijestima o komponentama trećih strana](THIRD_PARTY.md) (Pomoć, Komponente
trećih strana), uz potpune tekstove licenci. Ništa u ovom
Ugovoru ne ograničava prava koja vam te licence daju.

## 14. Usluge trećih strana

Softver ne uključuje vlastite mrežne usluge trećih strana: nema sustav
korisničkih računa, komponentu u oblaku, pružatelja analitike ili telemetrije
ni poslužitelj za licenciranje. Spaja se samo na poslužitelje baza podataka
koje podesite i na `127.0.0.1` za vlastiti pozadinski dio, kako opisuje
[obavijest o privatnosti](PRIVACY.md). Jedina iznimka je ona koju stvorite
sami: dodatak koji instalirate može se spajati na druge usluge koje mu
podesite - to je djelovanje dodatka i uređeno je odjeljkom 15.

Softver sadrži poveznice na dokumentaciju trećih strana - primjerice na popis
parametara pružatelja usluge smještaja baza podataka. Otvaranjem poveznice ta
se stranica otvara u vašem pregledniku, a ono što se ondje događa odnos je
između vas i te treće strane, prema njezinim uvjetima, a ne prema ovom Ugovoru.

Kada podesite bazu podataka smještenu u oblaku, vaš odnos s tim pružateljem
isključivo je vaš i na njega se primjenjuju njegovi uvjeti. Autor nije stranka
tog odnosa.

## 15. Dodaci

Softver se može proširiti dodacima (pluginima) - kodom u Pythonu i
JavaScriptu koji radi unutar njega, instaliranim kroz Pluginovi, Upravljanje
pluginovima, ili napisanim alatom za izradu dodataka koji Softver sadrži.

**Dodatak može izraditi bilo tko**, i ništa ne provjerava tko ga je izradio.
Autor stoga preporučuje korištenje **samo dodataka izdavača kojima vjerujete**.

- **Dodatak nije dio Softvera.** Dodatak koji je napisao netko drugi djelo je
  te osobe i dolazi pod uvjetima te osobe. Autor ga ne pregledava, ne potpisuje,
  ne odobrava i ne podržava. Dodaci koje objavljuje Autor nude se pod uvjetima
  ovog Ugovora.
- **Dodatak radi s vašim pravima.** Radi unutar Softvera kao vi, može doći do
  svake baze podataka na koju se spojite i može se spojiti na bilo koju drugu
  adresu do koje je podešen. Odjeljci 10 i 11 primjenjuju se na sve što dodatak
  radi kao i na Softver, a Autor nije odgovoran za dodatak koji je napisao
  netko drugi.
- **Usluga do koje dodatak dolazi pripada toj usluzi.** Kada dodatak nešto
  pošalje usluzi izvan Softvera - poslužitelju za nadzor, web stranici,
  jezičnom modelu - vi odlučujete to poslati, a uvjeti te usluge uređuju što
  ona prima i što s tim radi. Autor nije stranka tih uvjeta i ništa ne prima.
  Ono što dodatak vrati od takve usluge, uključujući analizu ili predloženu
  naredbu, informacija je koju treba provjeriti, a ne uputa.
- **Pisanje dodataka je dopuštena upotreba.** Korištenje sučelja za dodatke
  koje Softver dokumentira - vodiča za dodatke, alata za izradu dodataka i
  funkcija koje dodatak dobiva - korištenje je Softvera, a ne njegova izmjena
  ili obrnuti inženjering u smislu odjeljka 4.
- **Vaši su dodaci vaši.** Dodaci koje napišete pripadaju vama i smijete ih
  dijeliti ili distribuirati pod uvjetima koje sami odaberete, pod uvjetom da
  dodatak ne sadrži nijedan dio samog Softvera i da se ne predstavlja kao djelo
  Autora ili kao nešto što je Autor odobrio.

## 16. Izmjene

Kasnija verzija Softvera može donijeti izmijenjeni Ugovor. Kada se to dogodi,
Softver traži da prihvatite novu verziju prije nego što se otvori i navodi o
kojoj je verziji riječ. Nastavak korištenja **postojeće** instalacije nikada
prešutno ne mijenja uvjete koje ste prihvatili.

Ako ne prihvatite izmijenjeni Ugovor, možete nastaviti koristiti verziju koju
ste već prihvatili, pod uvjetima koje ste prihvatili. Niste obvezni nadograditi
Softver.

## 17. Mjerodavno pravo

Na ovaj Ugovor primjenjuje se pravo Republike Hrvatske, bez primjene pravila o
sukobu zakona. Za sporove su isključivo nadležni sudovi u Republici Hrvatskoj,
osim kada vam prisilno pravo daje pravo pokrenuti postupak pred sudovima zemlje
u kojoj imate prebivalište.

## 18. Jezik

Ovaj Ugovor sastavljen je na hrvatskom jeziku. Engleski i drugi prijevodi
služe samo radi lakšeg razumijevanja; u slučaju neslaganja **mjerodavan je
hrvatski tekst**.

## 19. Cjelovitost ugovora

Ovaj Ugovor čini cjelokupni sporazum između vas i Autora o Softveru i zamjenjuje
svaku raniju verziju ovog Ugovora. Ako se neka odredba utvrdi neprovedivom,
ostale ostaju na snazi.

## 20. Kontakt

Autor DBApguina je **Darko Lovrenčec**. Pitanja o Softveru, prijave pogrešaka i
pitanja o ovom Ugovoru upućuju se Autoru na **dbapguin@gmail.com**.

Ništa se ne kupuje, pa nema ni kontakta za naplatu.
