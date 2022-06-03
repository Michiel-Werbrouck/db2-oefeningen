# Oefeningen over indexen

## 1. Eigen databank

> **Ruimtereizen.**
> Welke index kan er best worden toegevoegd aan de tabel bezoeken uit ruimtereizen?
> Maak deze index aan op je lokale databank.
> Op welke andere kolommen staat er een index?

We kijken eerst of er al indexen bestaan voor de tabel bezoeken. Op het eerste zicht blijkt dit niet zo te zijn (indexes tab vouwt niet open op pgadmin).
Dit is echter fout, aangezien postgres (onze dmbs) automatisch bij de creatie van de tabel al een index aanmaakt voor de primary key. Dit betekent dus dat op onze primary key (volgnr + reisnr) eigenlijk al een index bestaat. We kunnen dus gerust zijn dat onze queries al efficiënter gaan zijn door deze index.

We kunnen de indexen van een tabel opzoeken dmv de pg_indexes tabel.

```sql
SELECT tablename, indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'ruimtereizen' and tablename = 'bezoeken'
ORDER BY tablename, indexname;
```

Op welke kolom zouden we dan nog een index kunnen maken? Wel, we kunnen kijken naar de foreign keys van bezoeken. Er is één op reisnr en één op objectnaam.
Een index op reisnr maken zou echter niet veel helpen aangezien deze kolom deel is van de primary key (er bestaat al een index dus die deze kolom bevat).
Wat wel interessant lijkt is een index maken op de FK objectnaam aangezien deze waarschijnlijk bij veel joins zal gebruikt worden.

```sql
CREATE INDEX ON bezoeken(objectnaam);
```

Je kunt ook je eigen naam meegeven voor deze index, maar dit hoeft niet.

```sql
CREATE INDEX bezoeken_objectnaam_idx ON bezoeken(objectnaam);
```

Ten slotte droppen we onze aangemaakte index zo:

```sql
DROP INDEX bezoeken_objectnaam_idx;
```

Indien je geen naam hebt meegegeven voor de index, kun je deze makkelijk vinden door een simpele select te doen in pg_indexes (zie query boven).

> **Eigen tabel.**
> Voeg zelf een tabel naar keuze toe op je lokale databank, met minstens 4
> kolommen en 1 primaire sleutel, optioneel een vreemde sleutel.
> Bestaat er nu ook een index op deze zelf gemaakte tabel?

Deze oefening heb ik eigenlijk al hierboven uitgelegd.  
**Spoiler**: ja er gaat een index aangemaakt woorden op de primary key. Waarom? Omdat postgres één van de velen dmbs is die dit automatisch doet.

### 1.1. Grootte

> Hoe groot is de tabel bezoeken fysisch? Hoe groot is de index die je hebt aangemaakt op bezoeken?  
> Verwijder de index die je hebt aangemaakt en controleer terug de grootte.

**Tabel**

```sql
SELECT pg_size_pretty(pg_table_size('bezoeken'));
--resultaat: 8192 bytes
```

pg_size_pretty maakt de grootte leesbaar voor mensen (tenzij ge een masochist zijt).

**Index**

```sql
SELECT pg_size_pretty(pg_indexes_size('bezoeken'));
--resultaat: 32 kB
--als we onze eigen aangemaakte index verwijderen wordt dit 16 kB.
```

## 2. Geolite

Het geolite schema vindt je terug op 54321 -> gis -> geolite

### 2.1. Inleiding

> Wat is de structuur van de tabellen?

Dit kunnen we makkelijk terugvinden door een ERD te laten genereren van de gis DB. Rechterklik -> Generate ERD.
We kunnen ook gwn naar de kolommen van de tabellen in het geolite schema kijken, maar hoe kun je nee zeggen tegen een mooie ERD?

![Screenshot](Images/erd_geolite.PNG)

> Is er een verband tussen de tabellen?

Er is een verband, maar deze wordt niet afgedwongen (locid is geen constraint in de blocks tabel).  
Het verband zit dus in het feit dat deze 2 tabellen een locid kolom hebben.

> Op welke velden bestaat er een index

Op de blocks tabel is er een index op ip_range en in de location tabel een index op locid (logisch want dat is daar de PK).

### 2.2. Index

**Vergelijk de snelheden van**:

> Toon de plaats informatie voor locid 244

```sql
SELECT * FROM geolite.location WHERE locid = 244;
```

Dit duurde ongeveer 1 seconde, redelijk snel dus.

> Toon alle rijen met 'Cape Town'

```sql
SELECT * FROM geolite.location WHERE city = 'Cape Town';
```

Duurde iets langer, maar niet héél lang.

> Wat is caching?

Standaard gezien zijn indexen redelijk goed, maar waarom was het verschil van locid (wat een index heeft) tov city (geen index) redelijk klein?  
Dit komt door caching. Je hebt verschillende soorten caches, de cache die hier een rol speelt is de cache van de DB.  
Als de data die opgevraagd worden al opgevraagd is geweest door iemand anders, dan wordt dit resultaat gecached.  
Dan is het logisch dat je sneller output krijgt van je query ondanks dat er redelijk wat rijen zijn.

> Hoeveel rijen bevatten de tabellen?

```sql
SELECT COUNT(*) FROM geolite.location;
```

Bijna een miljoen rijen! Wajoh! 890521 om exact te zijn.

### 2.3. Opzoeking

Er zijn voor deze tabellen enkele nieuwe operatoren toegevoegd, waarvan twee  
uit de verzamelingenleer; >>= wat wil zeggen omvat of <<= wat wil zeggen bevat.

Bijvoorbeeld:

```sql
WHERE iprange >>= '208.118.235.174';
```

> Wat is een schema?

Even googlen!

_In een databaseschema wordt de structuur van een databasesysteem beschreven in een formele taal die door het databasemanagementsysteem wordt ondersteund.  
Deze structuur verwijst naar de organisatie van de data nodig om een blauwdruk te maken van hoe een database wordt geconstrueerd._

> Kan je meerdere schema’s in je zoek pad opgeven?

Yes, ezpez.

```sql
set search_path to geolite, cell_tower;
```

> Waar staat waarschijnlijk de server projektwerk.ucll.be?

Even een kijkje nemen!

```sql
select * from geolite.location
inner join geolite.blocks using (locid)
where country = 'BE' and city = 'Leuven' and postalcode = '3001'
```

locid = 775468  
location = (4.7009,50.8796)

### 2.4. Botnet

Er is een aanval geweest van een klein botnet, dit zijn de ipadressen van waarop
de aanval werd uitgevoerd. Kan de vermoedelijke lokatie achterhaald worden?

> 62.235.77.23  
> 94.225.38.78  
> 94.225.101.3  
> 62.235.77.56  
> 94.225.101.99  
> 94.225.101.127

Als ik de >>= of <<= operatoren wil gebruiken krijg ik een error, maar normaal moet deze query werken.

```sql
select location from geolite.location
inner join geolite.blocks using (locid)
where iprange >>= '94.225.101.127' or iprange >>= '62.235.77.56'
```

Dit is echter een schatting. Ik weet niet of deze query gaat werken.

# Oefeningen over optimalisatie

## 2. Ruimtereizen

### 2.1.

> Maak een lijst met alle hemelobjecten waar ons reisbureau nog niet op bezoek geweest is of  
> gepasseerd is en die een diameter hebben van meer dan 100.000 km.  
> Sorteer de lijst aflopend volgens de grootte van de diamete

```sql
select h.objectnaam, diameter
from hemelobjecten h
left join bezoeken b on h.objectnaam = b.objectnaam
where b.objectnaam is null and diameter > 100000
order by diameter desc
```

### 2.2.

> Maak een overzicht waarbij je voor de Maan en voor Mars aangeeft hoeveel ruimtereizen  
> één of meer keer de betreffende bestemming bezocht hebben (d.w.z. erop geland zijn).

```sql
select objectnaam, count(case verblijfsduur when 0 then null else 1 end)
from bezoeken
where objectnaam = 'Maan' or objectnaam = 'Mars'
group by objectnaam
```

zo kan ook

```sql
SELECT objectnaam AS bestemming, COUNT(*) AS aantal_reizen
FROM bezoeken
WHERE objectnaam in ('Mars', 'Maan') and verblijfsduur > 0
group by objectnaam
```

### 2.3.

> Schrijf de instructies die de nuttige indexen op een tabel die je in de probeer databank kan aanmaken.

Geen flauw idee wat Bertels wilt.

### 2.4.

Deze oefening bestaat niet, Bertels skipt gwn naar 5.

### 2.5.

> Ga na op welke velden er een index is aangemaakt bij de spelers_l of spelers_xl of spelers_xxl tabel.

Er is een index gemaakt op de primaire key (spelersnr).

```sql
SELECT tablename, indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'tennis' and tablename = 'spelers_l'
ORDER BY tablename, indexname;
```

> Hoeveel gegevens zitten er in de door jouw gekozen tabel?

8730 gegevens

```sql
SELECT count(*) from spelers_l
```

> Maak een heel eenvoudige querie waarbij je zoekt op een geïndexeerd veld.

```sql
SELECT naam from spelers_l where spelersnr = 6
```

> Maak ook een heel eenvoudige querie waarbij je zoekt op een niet-geïndexeeerd veld. Is er verschil?

```sql
SELECT naam from spelers_l where bondsnr = '93'
```

96ms (niet-geïndexeerd) vs 92ms (geïndexeerd). Eigenlijk amper een verschil dus.

### 2.6.

> Bereken de kostprijs om al de gegevens van de spelers, spelers_l en spelers_xl en spelers_xxl tabel te tonen.

```sql
explain select * from spelers
```

Voer deze query ook uit voor l, xl en xxl!

**Kostprijzen**

-   Spelers: 1.14
-   Spelers_l: 281.30
-   Spelers_xl: 3223.00
-   Spelers_xxl: 32223.00

> Wat is verband met de vorige oefening

Het verband zit in de kostprijs van elke tabel. Bij de vorige oefening keken we zuiver naar één tabel zijn totale uitvoeringstijd (= planning + uitvoering) en verschil tov index en geen index.
Hier kijken we naar de exponentiële groei in kostprijs naarmaate de tabel meer gegevens heeft.

### 2.7.

```sql
select reisnr, count(*) from deelnames
group by reisnr
order by 2 desc
```

### 2.8.

```sql
select r.reisnr,
	(select count(distinct klanten.klantnr) from klanten
	inner join deelnames on (deelnames.klantnr = klanten.klantnr)
	where deelnames.reisnr = r.reisnr)
from reizen r
```

of

```sql
select reisnr, count(distinct klantnr)
from deelnames right join reizen using (reisnr)
group by reisnr
order by 2 desc
```

# Oefeningen over limiting result sets

## Chinook Schema

> Geef voor elk album de 2 langste nummers (tracks). (Dus alle albums moeten getoond worden).  
> SToon de naam van het album, het nummer en de tijd.  
> Schrijf deze query zo performant mogelijk.

```sql
select r1."Title", hs."Name", hs."Milliseconds"
from "Album" r1 left join lateral
    (select *
    from "Track" b2
    natural inner join "Album"
    where r1."AlbumId" = b2."AlbumId"
    order by "Milliseconds" desc
    fetch first 2 row only) hs on true;
```

of

```sql
select "Name", "Title", "Milliseconds"
        from "Track" as t inner join "Album" using ("AlbumId")
        where 2 > (select count ("TrackId")
        from "Track"
        where "Track"."Milliseconds" > t."Milliseconds" and t."AlbumId" = "Track"."AlbumId")
```

Deze query was zo kut, holy shit.

# Oefeningen op Procedurele SQL

Tips van meester Mikkel: denk eerst na of het gaat over een functie of een procedure!

returned het iets? Dan is het een functie!
Voert het een mutatie uit op bv tabel(len)? Dan is het een procedure!

Alright, with that said... let's go.

## 1.

> Schrijf een functie waarbij je gegevens uit een transactie tabel verwijdert die ouder zijn dan  
> 10 dagen (of een andere “archiverings” functie op 1 van jou eigen tabellen).

Ik denk dat dit een procedure moet zijn omdat je gegevens gaat verwijderen.

```sql
create or replace procedure delete_items_older_than_10days()
$$
    begin
        delete from messages
        where 10 < (select current_date-datum);
    end
$$ language 'plpgsql'

call delete_items_older_than_10days()
```

_Credits go to a certain dog_

## 2.

> Schrijf een functie die drie getallen optelt en teruggeeft.

```sql
CREATE OR REPLACE FUNCTION increment(num1 INT, num2 INT, num3 INT) RETURNS INT AS
$$
  BEGIN
  RETURN num1 + num2 + num3;
  END;
$$ LANGUAGE 'plpgsql';
```

## 3.

> Geef 3 mogelijke procedurale talen op postgresql en vind een overeenkomst met een ander  
> software pakket (dbsoftware) op minstens 1 van die procedurale talen.

Ik ga er gwn 4 geven die in de basis distributie van postgres zitten:

-   PL/pgSQL, PL/Tcl, PL/Perl en PL/Python

Het 2e deel van deze vraag is nog een mysterie. Ik vind niet echt direct een onvereenkomst.

## 4.

> Zou het mogelijk zijn een aggregatie functie te schrijven? Hoe zou je dit dan moeten doen  
> en is dit sql standaard?

Louche vraag. Bedoeld hij nu een aggregatie functie zelf? Onze eigen custom sum() ofzo? Of bedoeld hij dat we aggregaties kunnen gebruiken
in een functie?

```sql
CREATE OR REPLACE FUNCTION get_objectnaam_meeste_bezoeken()
        returns table (label text, cnt bigint) as
        $$
            select objectnaam, count(*) as aantal_bezoeken
            from bezoeken
            group by objectnaam
            having count(*) = (SELECT MAX(y.num)
            FROM (SELECT COUNT(*) AS num, objectnaam
                FROM bezoeken
                group by objectnaam) y )
        $$ LANGUAGE sql;
```

## 5.

> Schrijf een functie die nuttig is voor jouw persoonlijke databank

Meneer was weer te lui zeker. Dan ben ik ook lui, skip!

## 6.

> Schrijf een functie met als input parameter een tabel, die als output een overzicht geeft van  
> alle kolommen op de volgende wijze:  
> table A (x text, y int, z char(4)) - - tabel A met 3 kolommen als input  
> output: drie lijnen met een opsomming van de kolommen en hun datatypes.  
> x_value text;  
> y_value int;  
> z_value char;  
> -- tip: check table information about pg_attribute

```sql
CREATE OR REPLACE FUNCTION public.kolom_overzicht ("tabel" TEXT)
        RETURNS TABLE ("kolom_naam" TEXT, "kolom_datatype" TEXT) AS
        $$
        BEGIN
        RETURN QUERY SELECT attname::text as kolom_naam,
        pg_catalog.format_type(a.atttypid, a.atttypmod)::text as kolom_datatype
        FROM pg_attribute A
        WHERE attrelid = "tabel"::regclass AND attnum > 0 AND NOT attisdropped;
        END;
        $$
        LANGUAGE 'plpgsql';

--En zo gebruik je het
SELECT * FROM public.kolom_overzicht ('countries');
```

## 7.

> Schrijf een functie met dezelfde functionaliteit als een bestaande php functie uit je  
> persoonlijk project.

Huh, php? Nou meid daar doen we niet aan mee. Oké dan, laten we de php abs functie namaken.

```php
//abs(-2) geefts obvs 2
abs(number);
```

Oké, nu met een procedurele functie!

```sql
CREATE OR REPLACE FUNCTION mijn_abs(num INT)
        RETURNS INT AS
        $$
        BEGIN
        IF num < 0 THEN
        RETURN -num;
        END IF;

        RETURN num;
        END;
        $$
        LANGUAGE 'plpgsql';

--even testen
select mijn_abs(-2);
--res: 2
```

## 8.

> Probeer 1 van de volgende aan te maken (je mag kiezen welke, maar je mag het nog niet  
> gedaan hebben eg: table) : AGGREGATE, DATABASE, GROUP, OPERATOR,  
> SEQUENCE, TRIGGER, USER, CAST, DOMAIN, INDEX, RULE,  
> TABLE, TYPE, VIEW, CONVERSION, FUNCTION, LANGUAGE , SCHEMA,  
> TEMP, UNIQUE  
> Controleer nu of deze functie of een SQL functie is en niet postgresql specifiek.

Wait... Bertels what have you been smoking? Nee echt, ik weet niet wat hij verwacht.  
Ik dacht dat deze oefenreeks over procedurele functies ging...

# Oefeningen op triggers

Before we go: met deze query kun je alle triggers oplijsten.

```sql
SELECT event_object_table AS table_name, trigger_name
FROM information_schema.triggers
GROUP BY table_name, trigger_name
ORDER BY table_name, trigger_name
```

Let's goooo.

## 1.

> Schrijf een functie waarbij je zinnige uitvoer probeert terug te geven naar gebruikers wanneer ze gegevens verkeerd/vergeten in voeren in een zelf gekozen tabel en zorg ervoor dat dit vanzelf gecontroleerd wordt.

```sql
CREATE OR REPLACE FUNCTION check_reservering() RETURNS TRIGGER AS
$$
BEGIN
  IF( NEW.datum_aankomst > NEW.datum_vertrek ) THEN
    RAISE EXCEPTION 'Aankomstdatum moet voor vertrekdatum liggen';
  END IF;

  RETURN NEW;
END;
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER trigger_check_reservering
  BEFORE INSERT OR UPDATE
  ON reservering
  FOR EACH ROW
  EXECUTE PROCEDURE check_reservering();
```

## 2.

> Zorg ervoor dat er een tabel met een simpele statistiek van een tabel van jouw project, namelijk het aantal inserts dat er reeds is op uitgevoerd.

**bzzz... error occured when trying to translate bertelsiaans to human language... calculating possible solution...**

Ik had dus voor IPMajor een user tabel. Deze query toont het aantal rij inserts op de user tabel.

```sql
select n_tup_ins
from pg_stat_all_tables
where schemaname = 'public' and relname = 'users'
```

## 3.

> In welke situaties kan je een trigger schrijven?  
> Bv bij een select die door een gebruiker word uitgevoerd?

Bij elk soort database event kun je een trigger maken.
In welke situaties? Bv als je bij een insert wilt nakijken of
de parameters valide zijn. Bij een update kun je hetzelfde doen.

Meer info: [postgresql_triggers](https://www.tutorialspoint.com/postgresql/postgresql_triggers.htm)

## 4.

> Schrijf een nuttige trigger op je eigen databank.

Ik heb al constraints voor rating & views, maar ik kijk eigenlijk nooit na of een titel of
beschrijving enkel uit nummers bestaat. Dus dit is een mooie oplossing daarvoor.

```sql
CREATE OR REPLACE FUNCTION check_novel() RETURNS TRIGGER AS
$$
BEGIN
  IF( NEW.title ~ '^[0-9\.]+$') THEN
    RAISE EXCEPTION 'Titel mag niet enkel uit nummers bestaan';
  END IF;

  IF( NEW.description ~ '^[0-9\.]+$') THEN
    RAISE EXCEPTION 'Beschrijving mag niet enkel uit nummers bestaan';
  END IF;

  IF (NEW.rating < 0) THEN
    RAISE EXCEPTION 'Rating mag niet kleiner dan nul zijn';
  END IF;

  IF (NEW.views < 0) THEN
    RAISE EXCEPTION 'Views mag niet kleiner dan nul zijn';
  END IF;

  RETURN NEW;
END;
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER trigger_check_novel
  BEFORE INSERT OR UPDATE
  ON novels
  FOR EACH ROW
  EXECUTE PROCEDURE check_novel();
```

## 5.

> Create trigger that deletes 10 oldest fines when new fine will make total  
> number of fines raise above 50

```sql
CREATE OR REPLACE FUNCTION delete_tien_oudste() RETURNS TRIGGER AS
$$
BEGIN
    DELETE FROM tabel1
    WHERE
        (SELECT COUNT(*) FROM tabel1) > 50
    AND
        id IN (
        SELECT id
        FROM tabel1
        ORDER BY id ASC
        FETCH FIRST 10 ROWS ONLY
    );
    RETURN NULL;
END;
$$
language 'plpgsql';

CREATE TRIGGER delete_oudsten
    AFTER INSERT
    ON tabel1
    FOR EACH ROW
    EXECUTE PROCEDURE delete_tien_oudste();
```

## 6.

> Create trigger that prevents inserts of fines with an amount above 200 euros

```sql
CREATE OR REPLACE FUNCTION check_fine() RETURNS TRIGGER AS
$$
BEGIN
  IF( NEW.bedrag > 200) THEN
    RAISE EXCEPTION 'Titel mag niet enkel uit nummers bestaan';
  END IF;

  RETURN NEW;
END;
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER trigger_fine
  BEFORE INSERT
  ON boetes
  FOR EACH ROW
  EXECUTE PROCEDURE check_fine();
```

## 7.

> Create trigger that returns actual total amount and number of fines whenever a new fine is inserted or an existing one is updated.

```sql
DROP TRIGGER IF EXISTS OEF_3 ON boetes;
DROP FUNCTION IF EXISTS ReturnInfoOnChange();

set client_min_messages TO notice;

CREATE OR REPLACE FUNCTION ReturnInfoOnChange() RETURNS trigger AS
$$
DECLARE
	total_amount FLOAT;
	nfines INTEGER;
BEGIN
	total_amount := (SELECT sum(bedrag) FROM boetes);
	nfines := (SELECT count(betalingsnr) FROM boetes);
	RAISE NOTICE 'Total amount of fines: % and total number %', total_amount, nfines;
	return new;
END
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER OEF_3 BEFORE INSERT OR UPDATE ON boetes
FOR EACH ROW
EXECUTE PROCEDURE ReturnInfoOnChange();

INSERT INTO boetes VALUES (1110,2,'2017-11-06', 100);
UPDATE boetes SET bedrag=230 WHERE betalingsnr = 1110;
INSERT INTO boetes VALUES (1210,2,'2017-11-06', 200);
UPDATE boetes SET bedrag=300 WHERE betalingsnr = 1210;
```

# Oefeningen op Views, Stored Procedures en Triggers

## 1.

> Zoals je misschien herinnerd, kan het tellen van al de rijen soms langs duren. Dus het gebruik van de count() functie. Maak op je lokale databank een tabel met minstens 1 miljoen rijen.

```sql
CREATE TABLE items AS
  SELECT
    (random()*1000000)::integer AS n,
    md5(random()::text) AS s
  FROM
    generate_series(1,1000000);
```

Doe daarna een count() op de tabel.

```sql
SELECT COUNT(*) FROM items;
```

## 2.

> Maak een view waarin de rijen van deze tabel telt. Hoe kan je ervoor zorgen dat deze view niet steeds opnieuw moet worden uitgevoerd, ie de onderliggende code.
> Maak zo een view aan.

```sql
CREATE MATERIALIZED VIEW counter
        as
        select count(*)
        from items
```

## 3.

> Schrijf een trigger die bij een toevoeging of verwijdering van een rij in de tabel een teller in teltabel aanpast. Het doel van deze teltabel is het aantal rijen van een andere tabel bij te houden

```sql
CREATE OR REPLACE FUNCTION increment_count() RETURNS TRIGGER AS
$$
BEGIN
  UPDATE count
  SET count = count + 1
  WHERE table_name = 'items';
  RETURN NEW;
END;
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER trigger_replace BEFORE INSERT OR DELETE ON boetes
FOR EACH ROW EXECUTE PROCEDURE increment_count();
```

## 4.

> Welk van de 3 bovenstaande methode is het meest performant?
>
> -   rechstreeks tellen
> -   view (optie 1 en 2)
> -   teltabel (via trigger)

Ik denk een trigger, want dan tel je op per insert of delete en wordt dat bijgehouden (in bv een teltabel).

# Oefeningen op Vensters

_Dit zijn oefeningen van SQLDropbox die gemaakt zijn tijdens de les of die week_

## 1.

> Geef per reis incrementeel de totale verblijfsduur volgens de volgorde waarin de hemelobjecten bezocht worden.
> Geef ook de totale verblijfsduur van alle reizen om alles in perspectief te zettten.
> Sorteer op reisnr, volgnr, objectnaam, verblijfsduur, de volgende kolommen.

```sql
SELECT r.reisnr, b.volgnr, b.objectnaam, b.verblijfsduur, SUM(b.verblijfsduur)
OVER
(
        PARTITION BY r.reisnr
        ORDER BY b.volgnr ROWS BETWEEN UNBOUNDED
        PRECEDING AND current ROW
)
AS inc_duur, (SELECT SUM(bezoeken.verblijfsduur) FROM bezoeken) AS tot_duur
FROM reizen r
LEFT OUTER JOIN bezoeken b USING(reisnr)
ORDER BY 1,2,3,4,5,6
```

## 2.

> Hoe lang was het geleden dat er nog een reis vertrokken was?
> Geef daarnaast de totale reisduur per jaar incrementeel in de tijd (hier genaamd jaar_duur).
> Sorteer op reisnr en de andere kolommen.

```sql
SELECT reisnr, LAG(reisnr) OVER w1 AS vorig_reisnr, vertrekdatum, vertrekdatum - LAG(vertrekdatum) OVER w1 AS tussen_tijd,
reisduur, EXTRACT(YEAR FROM vertrekdatum) AS jaar, SUM(reisduur) OVER w2 AS jaar_duur
FROM reizen
WINDOW w1 AS (ORDER BY vertrekdatum), w2 AS (PARTITION BY EXTRACT(YEAR FROM vertrekdatum) ORDER BY vertrekdatum)
ORDER BY 1
```

# Oefeningen op CTEs

## 1.

> Kijk naar de structuur van de tabel hemelobjecten in het schema public en naar dezelfde tabel in  
> het schema ruimtereizen. Wat merk je op?

Er zijn 2 FKs die verwijzen naar dezelfde kolom hemelobjecten(objectnaam) in het public schema.
Je kunt dus best één van de 2 verwijderen door een alter table query uit te voeren.

## 2.

> Geef de gepaste code waarmee je de tabel hemelobjecten in ruimtereizen 'beter' kan maken.

```sql
ALTER TABLE hemelobjecten
 ADD CONSTRAINT hemelobjecten_hier_fk
 FOREIGN KEY (satellietvan) REFERENCES hemelobjecten(objectnaam)
 NOT VALID;

ALTER TABLE hemelobjecten
 VALIDATE CONSTRAINT hemelobjecten_hier_fk;
```

## 3.

> Er zijn twee (of meer..) manieren om deze tabel 'beter' te maken, met hetzelfde eindresultaat.  
> Welke van deze twee manieren zou jij kiezen en waarom?

Je hebt de optie hierboven, voordeel: sneller, nadeel: pas consistent (ref integriteit) voor de reeds aanwezige
data na validatie.
--1. NOT VALID en daarna VALIDATE (supra)
--2. of samen in 1 statement? (infra)
--En de optie hieronder, nadeel: trager, voordeel: huidige integriteit wordt direct gecontroleerd met betrekking
tot deze constraint

```sql
ALTER TABLE hemelobjecten
 ADD CONSTRAINT hemelobjecten_hier_fk
 FOREIGN KEY (satellietvan) REFERENCES hemelobjecten(objectnaam);
```

## 4.

### 1 - 3.

Zijn useless deel oefeningen, skip.

### 4.

> Schrijf een querie die alle satellieten van een zelf gekozen hemelobject toont. (bv de zon  
> indien aanwezig in de tabel). Zorg ervoor dat de hele hierarchie wordt getoond. (cf CTEs)

```sql
WITH RECURSIVE satellieten AS(
 SELECT objectnaam, satellietvan
 FROM hemelobjecten
 WHERE satellietvan = 'Zon'
 UNION ALL
 SELECT h.objectnaam, h.satellietvan
 FROM hemelobjecten h INNER JOIN satellieten s
 ON (h.satellietvan = s.objectnaam)
)
SELECT *
FROM satellieten
ORDER BY 1,2;
```

### 5.

> Toon de boomstructuur voor elk hemelobject uit de vorige querie.

```sql
WITH RECURSIVE satelliet_van(objectnaam, satellietvan, boom) AS (
SELECT objectnaam, satellietvan, cast(satellietvan as text)
FROM hemelobjecten
WHERE satellietvan = 'Zon'
 UNION ALL
SELECT h.objectnaam, h.satellietvan, s.boom || '-' || h.satellietvan
FROM hemelobjecten h, satelliet_van s
WHERE h.satellietvan = s.objectnaam
)
SELECT *
FROM satelliet_van;
```

### 6 - 7.

Zijn bullshit oefeningen. Enkel de constraint vraag uit 7 is nuttig.

> Begrijp je wat onderstaande code betekent?  
> CONSTRAINT bezoeken_objectnaam_fkey FOREIGN KEY (objectnaam)  
> REFERENCES ruimtereizen.hemelobjecten (objectnaam) MATCH SIMPLE  
> ON UPDATE NO ACTION ON DELETE NO ACTION

Met deze code maak je een FK aan op bezoeken zijn objectnaam kolom aan die een referentie is naar
de pk van hemelobjecten objectnaam kolom.

Bij MATCH SIMPLE vond ik het volgende online:

> MATCH SIMPLE allows any of the foreign key columns to be null; if any of them are null, the row is not required to have a match in the referenced table.

ON UPDATE: doe niks wnr de overéénkomende row in hemelobjecten geupdate wordt.

ON DELETE: doe niks wnr de overéénkomende row in hemelobjecten verwijderd wordt.

### 8.

Zo kun je recursief door hemelobjecten en hun satellieten gaan zonder lus!

```sql
WITH RECURSIVE satelliet_van(objectnaam, satellietvan, pad, lus) AS (
SELECT objectnaam, satellietvan, ARRAY[satellietvan] as pad, false
FROM hemelobjecten
WHERE satellietvan = 'Zon'
 UNION ALL
SELECT h.objectnaam, h.satellietvan, CAST(s.pad || ARRAY[h.satellietvan] as varchar(10)[]) as pad, h.satellietvan = ANY(pad)
FROM hemelobjecten h, satelliet_van s
WHERE h.satellietvan = s.objectnaam
AND NOT lus
)
SELECT *
FROM satelliet_van;
```
