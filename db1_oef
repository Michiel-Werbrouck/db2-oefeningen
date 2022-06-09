# db1_oef

taken from [this repo](https://github.com/Kwinnieprince/queries_databases)

extra 1
=======

opgave
------

Hoeveel sets zijn er in totaal gewonnen, hoeveel sets werden in totaal
verloren en welk is het uiteindelijke saldo? \#\# oplossing

    select sum(gewonnen) as totaal_gewonnen, sum(verloren) as totaal_verloren, (sum(gewonnen) - sum(verloren)) as saldo
    from wedstrijden

opgave
------

Geef een totaal overzicht van alle spelers, hun boetes en de functies
die ooit vervuld hebben. Elke speler moet getoond worden, als ie een
eventuele boete heeft gekregen en/of een eventuele functie als
bestuurslid heeft gehad dan moet dit ook getoond worden. Toon de
volledige naam, het bedrag en datum van de eventuele boete en de
eventuele bestuursfuncties. Sorteer van voor naar achter. \#\# oplossing

    select naam, voorletters, functie, bedrag, datum
    from boetes right outer join spelers using(spelersnr) left outer join bestuursleden using(spelersnr)
    order by 1,2,3,4,5

opgave
------

Geef een overzicht van de spelers die een boete hebben gekregen, indien
deze boete meer van 90 euro is toon je informatie “pijnlijk” anders “te
doen”. Toen de volledige naam en de categorie van de bijhorende boete.
Sorteer van voor naar achter. \#\# oplossing

    select naam, voorletters, case when bedrag > 90 then 'pijnlijk' else 'te doen' end as comment
    from boetes inner join spelers s on boetes.spelersnr = s.spelersnr
    order by 1,2,3

------------------------------------------------------------------------

extra 2
=======

opgave
------

Geef van elke speler het spelersnr, de naam en het verschil tussen zijn
of haar jaar van toetreding en het gemiddeld jaar van toetreding van de
spelers die in dezelfde plaats wonen. Sorteer van voor naar achter. Toon
3 getallen na de komma, maximaal 2 voor de komma; gebruik een cast
functie. \#\# oplossing

    select spelersnr, naam, voorletters, jaartoe - round((select avg(jaartoe) from spelers s2 where s.plaats = s2.plaats),3) as numeric
    from spelers s
    order by 1,2,3,4

opgave
------

Toon alle mogelijke combinaties van de letters ‘x’ en ‘y’. Tip zie
handboek, voorbeeld met cijfers. (Wat is een cartesisch product?).
Sorteer \#\# oplossing

    select concat(t.column1, tt.column1) as "?column?"
    from (values('x'), ('y')) as t cross join  (values('x'), ('y')) as tt

opgave
------

Geef de wedstrijden die door spelers zijn gespeeld die in Leuven,
Rotterdam of Leiden wonen. Sorteer van voor naar achter. Geef alle (\*)
informatie van wedstrijden. \#\# oplossing

    select w.*
    from spelers inner join wedstrijden w on spelers.spelersnr = w.spelersnr
    where plaats like 'Rotterdam' or plaats like 'Leuven' or plaats like 'Leiden'

------------------------------------------------------------------------

group by
========

opgave
------

Geef voor elke geboortejaar van de klanten, het aantal klanten, het
kleinste klantennummer en het grootste klantennummer. Geef ook het
totaal aantal klanten en het kleinste en grootste klantennummer. Sorteer
van voor naar achter. \#\# oplossing

    select extract(year from geboortedatum), count(*), min(klanten.klantnr), max(klanten.klantnr)
    from klanten
    group by (extract(year from geboortedatum))
    UNION
    (select null , count(*), min(klantnr), max(klantnr) from klanten )
    order by 1

opgave
------

We willen telkens het aantal reizen en de totale prijs van de volgende
situaties. Voor elke maand van vertrek, voor elk tiental van de
reisduur, voor de combinatie van maand van vertrek en tiental van de
reisduur en het het totale plaatje. Sorteer van voor naar achter. \#\#
oplossing

    select extract(month from vertrekdatum) as date_part, round((reisduur/10), 0), count(vertrekdatum), sum(prijs)
    from reizen
    group by rollup(date_part, round((reisduur/10), 0))
    union all
    select null, round((reisduur/10),0), count(vertrekdatum), sum(prijs)
    from reizen
    group by round((reisduur/10),0)
    order by 1,2,3,4

opgave
------

Geef voor de hemelobjecten een diameter groter dan 1000 het aantal
hemelobjecten en de gemiddelde diameter per groep die aan het volgende
voldoet. We zien per object zien hoeveel satellieten hieraan voldoen,
daarnaast in combinatie met hetzelfde object en afstand per 100tal.
Alsook een algemeen overzicht. Sorteer van voor naar achter. \#\#
oplossing

    select objectnaam, round(afstand/100),satellietvan, count(*) as count, avg(diameter)
    from hemelobjecten
    where diameter>1000
    group by objectnaam
    union
    select null as objectnaam, null as count,satellietvan, count(*) as count, avg(diameter)
    from hemelobjecten
    where diameter>1000
    group by satellietvan
    union
    select null as objectnaam, null as count,null as satellietvan, count(*) as count, avg(diameter)
    from hemelobjecten
    where diameter>1000
    order by 3,1,4

    --met rollup
    select objectnaam, round(afstand/100), satellietvan, count(*) as count, avg(diameter)
    from hemelobjecten
    where diameter>1000
    group by rollup ((satellietvan),(objectnaam,afstand))
    order by 3,1,4

------------------------------------------------------------------------

Herhaling 1
===========

opgave
------

Geef voor elke wedstrijd het wedstrijdnummer en de volledige naam van de
aanvoerder van het team waarvoor de wedstrijd werd gespeeld. Sorteer je
resultaat volgens het wedstrijdnummer in oplopende volgorde. \#\#\#
oplossing

    select wedstrijdnr, naam, voorletters
    from wedstrijden inner join teams using(teamnr) left outer join spelers on(teams.spelersnr = spelers.spelersnr)

opgave
------

Geef voor alle huidige bestuurleden hun functie en de lijst van boetes
die voor hen werd betaald. Omdat je dit wil vergelijken met de
boetebedragen die betaald werden voor leden die niet in het bestuur
zitten, wil je deze boetebedragen ook opnemen in de tweede kolom van je
resultaat. Sorteer je antwoord eerst op functie en daarna op het
boetebedrag. \#\#\# oplossing

    SELECT  functie, bedrag
    FROM    bestuursleden FULL OUTER JOIN boetes
    USING   (spelersnr)
    WHERE   eind_datum is null
    ORDER BY functie, bedrag

opgave
------

Geef per team de verloren wedstrijden. Zorg dat teams zonder verloren
wedstrijden ook in de output verschijnen. Duid per wedstrijd aan of het
om een actief bestuurslid gaat. Sorteer op divisie en wedstrijdnummer.
\#\#\# oplossing

    select w.teamnr, divisie, wedstrijdnr, w.spelersnr, case when functie is null then '-' else 'actief' end as bestuurslid
    from (teams t left outer join wedstrijden w on t.teamnr = w.teamnr and verloren > gewonnen) left outer join bestuursleden b on w.spelersnr = b.spelersnr and eind_datum is null
    order by divisie, wedstrijdnr

opgave
------

Geef een alfabetisch gesorteerde lijst van de namen van alle leden van
de tennisvereniging die nog geen wedstrijden gespeeld hebben \#\#\#
oplossing

    select naam
    from spelers left outer join wedstrijden w2 on spelers.spelersnr = w2.spelersnr
    where w2.spelersnr is null
    order by naam;

opgave
------

Geef het spelersnummer en bondsnummer van alle spelers die jonger zijn
dan de speler met bondsnummer 8467. gebruik een INNER JOIN. Sorteer 1,2
\#\#\# oplossing

    select distinct s2.spelersnr, s2.bondsnr
    from spelers s1 inner join spelers s2 on (s1.bondsnr = '8467')
    where s1.geb_datum < s2.geb_datum
    order by 1, 2

opgave
------

Geef een lijst met het spelersnummer, de naam van de speler, de datum
van de boete en het bedrag van de boete van al de spelers die een boete
gekregen hebben met een bedrag groter dan 45,50 euro en in Rijswijk
wonen. Geef expliciet aan welke join je gebruikt. \#\#\# oplossing

    select b.spelersnr, naam, datum, bedrag
    from spelers inner join boetes b on spelers.spelersnr = b.spelersnr
    where plaats ilike 'rijswijk' and bedrag > 45.50

opgave
------

Maak een lijst van alle mannelijke aanvoerders van een team en hun
gespeelde wedstrijden. Hierbij toon je voor deze spelers het
spelersnummer en de volledige naam, voor het team de divisie en voor de
wedstrijd het wedstrijdnummer. Sorteer aflopend op het wedstrijdnummer.
\#\#\# oplossing

    select spelersnr, naam, voorletters, divisie, wedstrijdnr
    from spelers inner join teams using(spelersnr) inner join wedstrijden w2 using(spelersnr)
    where spelers.geslacht = 'M'
    order by wedstrijdnr desc

opgave
------

Geef voor de actieve bestuursleden zonder boete hun laatste gespeelde
wedstrijd (die met het hoogste wedstrijdnummer). Sorteer aflopend op
spelersnr. \#\#\# oplossing

    select bestuursleden.spelersnr, max(wedstrijdnr) as laatstewedstrijd
    from bestuursleden left outer join boetes on bestuursleden.spelersnr = boetes.spelersnr and bestuursleden.eind_datum is null
    inner join wedstrijden on bestuursleden.spelersnr = wedstrijden.spelersnr
    where bedrag is null and eind_datum is null
    group by bestuursleden.spelersnr
    order by bestuursleden.spelersnr desc

opgave
------

Geef per team de leeftijd van de aanvoerder (tip: postgresql heeft een
AGE() functie) en het aantal verschillende spelers dat voor dit team
gespeeld heeft. Alleen teams waarvoor wedstrijden zijn gespeeld en die
een aanvoerder hebben, moeten vermeld worden. Sorteer op leeftijd en
daarna op aantal verschillende spelers en daarna op teamnr. \#\#\#
oplossing

    select distinct w2.teamnr, extract(year from age(geb_datum)) || ' jaar' as leeftijd ,count(distinct w2.spelersnr) as aantalspelers
    from teams inner join wedstrijden w2 on teams.teamnr = w2.teamnr inner join spelers s2 on teams.spelersnr = s2.spelersnr
    group by w2.teamnr, s2.geb_datum
    order by leeftijd, aantalspelers, w2.teamnr

opgave
------

Geef per team het hoogste wedstrijdnummer van een wedstrijd, gespeeld
door een bestuurslid (actief en niet meer actief) die geen boete heeft
gekregen. Sorteer op teamnr. \#\#\# oplossing

    select teamnr, max(wedstrijdnr) as laatstewedstrijd
    from wedstrijden inner join bestuursleden b1 on b1.spelersnr = wedstrijden.spelersnr left outer join boetes on b1.spelersnr = boetes.spelersnr
    where bedrag is null
    group by wedstrijden.teamnr
    order by teamnr

------------------------------------------------------------------------

herhaling 2
===========

opgave
------

Welke reizigers hebben al meer dan 1 reis ondernomen waarvoor ze meer
dan 2,5 miljoen euro moesten betalen? Sorteer op naam \#\# oplossing

    select naam, count(d2.reisnr) as aantal_reizen
    from reizen inner join deelnames d2 on reizen.reisnr = d2.reisnr inner join klanten k on d2.klantnr = k.klantnr
    where prijs > 2.5
    group by naam
    having count(d2.reisnr) > 1
    order by naam

opgave
------

Op welke planeten verblijft men gemiddeld langer dan 2 dagen? Sorteer op
objectnaam. \#\# oplossing

    SELECT h.objectnaam, AVG(b.verblijfsduur)
    FROM bezoeken AS b
    INNER JOIN hemelobjecten AS h USING (objectnaam)
    WHERE h.satellietvan = 'Zon'
    GROUP BY h.objectnaam
    HAVING AVG(b.verblijfsduur) > 2
    ORDER BY h.objectnaam

opgave
------

Geef een lijst van alle reizen met tenminste 1 bezoek of passage aan de
Maan of aan Mars. De lijst moet stijgend gesorteerd zijn op basis van
het vertrekdatum. \#\# oplossing

    select distinct b.reisnr, vertrekdatum
    from reizen inner join bezoeken b on reizen.reisnr = b.reisnr
    where objectnaam like 'Maan' or objectnaam like 'Mars'
    order by vertrekdatum asc

opgave
------

Welke reizen hebben exact drie verschillende hemelobjecten als reisdoel?
Sorteer op reisnr. \#\# oplossing

    select reisnr
    from bezoeken
    group by reisnr
    HAVING count(distinct objectnaam) = 3

opgave
------

Welke klanten (klantnaam, geboortedatum) zijn op een ruimtereis
vertrokken in het jaar dat ze 45 jaar geworden zijn? Het resultaat moet
stijgend gesorteerd worden op de geboortedatum. \#\# oplossing

    SELECT k.naam AS klantnaam, k.geboortedatum
    FROM deelnames d
    INNER JOIN klanten k USING (klantnr)
    INNER JOIN reizen r USING (reisnr)
    WHERE EXTRACT(year FROM r.vertrekdatum) - EXTRACT(year FROM k.geboortedatum) = 45
    GROUP BY k.naam, k.geboortedatum
    ORDER BY geboortedatum

opgave
------

Geef de leeftijd van de jongste klant op moment van vertrek. \#\#
oplossing

    SELECT EXTRACT(year FROM min(age(r.vertrekdatum, k.geboortedatum))) AS jongsteleeftijd
    FROM deelnames AS d
    INNER JOIN reizen AS r USING (reisnr)
    INNER JOIN klanten AS k USING (klantnr)

opgave
------

Geef de diameter van de grootste, niet bezochte maan (satelliet van een
planeet) \#\# oplossing

    select max(diameter) as grootstemaan
    from hemelobjecten left outer join bezoeken b on hemelobjecten.objectnaam = b.objectnaam
    where satellietvan not ilike 'zon' and reisnr is null

opgave
------

Geef de naam, diameter en het langste verblijf op van alle hemelobjecten
met minder dan 5 hemelobjecten die rond dit object draaien. Sorteer op
objectnaam. \#\# oplossing

    select planeet.objectnaam, planeet.diameter, max(verblijfsduur) as maximale_verblijf
    from (hemelobjecten planeet left outer join hemelobjecten maan on planeet.objectnaam = maan.satellietvan) inner join bezoeken b on b.objectnaam = planeet.objectnaam
    group by planeet.objectnaam
    having count(distinct maan.objectnaam) < 5

opgave
------

Geef per klant het totaal aantal reizen waaraan deze klant zal deelnemen
en het langste bezoek dat deze klant zal maken aan een hemelobject, over
alle reizen heen. Klanten zonder reizen of zonder bezoeken moeten ook
voorkomen in het overzicht. Sorteer op klantnr. \#\# oplossing

    select klantnr, count(distinct d.reisnr) as aantal, max(verblijfsduur) as langstebezoek
    from klanten left outer join deelnames d using(klantnr) left outer join reizen r on d.reisnr = r.reisnr left outer join bezoeken b on r.reisnr = b.reisnr
    group by klantnr
    order by klantnr

opgave
------

Geef voor elk hemelobject de minimale en maximale gemiddelde afstand tot
zijn zon (de centrale ster in een sterrenstelsel) als je weet dat de
kolom ‘afstand’ de gemiddelde afstand bevat tot het hemelobject waarrond
ze draaien. Met de grootte van het hemelobject hoef je geen rekening te
houden. Sorteer op minimale afstand en op objectnaam. Tip: bekijk de
functie COALESCE om met null-waardes rekening te houden. \#\# oplossing

    SELECT satelliet.objectnaam,
    COALESCE(planeet.afstand + satelliet.afstand, satelliet.afstand, 0) AS maximale_afstand,
    COALESCE(planeet.afstand - satelliet.afstand, satelliet.afstand, 0) AS minimale_afstand
    FROM hemelobjecten AS satelliet
    LEFT OUTER JOIN hemelobjecten AS planeet ON planeet.objectnaam = satelliet.satellietvan
    ORDER BY minimale_afstand, satelliet.objectnaam
    --klopt niet altijd volgens sqldropbox

------------------------------------------------------------------------

joins 1
=======

opgave
------

Maak een lijst met alle spelers die ooit een boete gekregen hebben die
hoger is dan 50 euro. Geen dubbels. Sorteer van voor naar achter. \#\#
oplossing

    select distinct naam, voorletters, plaats
    from spelers inner join boetes b on spelers.spelersnr = b.spelersnr
    where bedrag > 50
    order by naam

opgave
------

Geef van elke boete het betalingsnr, het boetebedrag en het percentage
dat het bedrag uitmaakt van de som van alle bedragen. Sorteer deze data
op het betalingsnr. Zorg dat er maar twee getallen na de komma getoond
worden (rond af). Sorteer van voor naar achter. \#\# oplossing

    select betalingsnr, bedrag, round((bedrag/(select sum(bedrag) from boetes)*100),2)
    from boetes
    group by betalingsnr
    order by betalingsnr

opgave
------

Geef chronologisch de spelersnummers van de bestuursleden die voorzitter
zijn of geweest zijn (chronologisch op begindatum van het
voorzitterschap) met vermelding van deze begindatum, alsook hun naam en
huidig adres. Als het vollegie adres niet gekend is dan moet “adres
ongekend” weergegeven worden. Sorteer van voor naar achter. \#\#
oplossing

    select begin_datum, naam, case
      when straat is null or huisnr is null or postcode is null or plaats is null
        then 'adres ongekend'
      else straat || ' ' || huisnr || ' ' || plaats || ' ' || postcode
    end as adres
    from bestuursleden inner join spelers s on bestuursleden.spelersnr = s.spelersnr
    where functie = 'Voorzitter'
    order by begin_datum

opgave
------

Geef alle wedstrijden van het team waarvan speler 6 aanvoerder is.
Sorteer \#\# oplossing

    select wedstrijdnr
    from wedstrijden inner join teams using (teamnr)
    where teams.spelersnr = '6'
    order by wedstrijdnr

opgave
------

Geef alle spelers die geen enkele wedstrijd voor team 1 hebben gespeeld.
Sorteer op naam, daarna op spelersnr. \#\# oplossing

    select s.spelersnr, naam
    from spelers s 
    where spelersnr not in (select spelersnr from wedstrijden where teamnr = 1)
    order by naam, spelersnr

opgave
------

Maak een lijst met de spelers (naam van de speler, voorletter en
woonplaats) die ooit gespeeld hebben voor een team dat nu in de tweede
divisie speelt en waarvoor geen enkele boete betaald werd voor 1 januari
1981. Geen dubbels, sorteer van voor naar achter. \#\# oplossing

    select naam, voorletters, plaats
    from spelers
    where spelersnr in (select w.spelersnr from wedstrijden w inner join teams using(teamnr) where divisie = 'tweede')
    and spelersnr in (select spelersnr from boetes right outer join spelers using(spelersnr) where datum > '1981-01-01'::date or datum is null)
    order by 1,2,3

------------------------------------------------------------------------

set oper
========

opgave
------

Geef een overzicht van alle spelers, gevolgd door alle bestuursleden,
gesorteerd op jaar van toetreding of beginjaar van hun functie en
vervolgens op spelersnr. Geen dubbels tonen. \#\# oplossing

    select spelersnr as veld1, naam as veld2, jaartoe as veld3
    from spelers
    union
    select spelersnr, functie, extract(YEAR from date(begin_datum))
    from bestuursleden
    order by veld3, veld1

opgave
------

Geef een lijst met alle spelersnrs, naam en het aantalwedstrijden ze
gespeeld hebben en op een nieuwe lijn het aantal bestuursfuncties die ze
hebben/hadden. Spelers die zowel wedstrijden gespeeld hebben als
bestuurslid zijn, komen dus twee keer voor in het resultaat. Sorteer op
spelersnr en aantal. Geen dubbels tonen. \#\# oplossing

    select spelersnr as nr, naam, aantal
    from spelers inner join (
      select spelersnr, count(wedstrijdnr) as aantal
      from wedstrijden
      group by spelersnr
      ) t using(spelersnr)
    union
    select s.spelersnr, naam, count(*)
    from bestuursleden inner join spelers s on bestuursleden.spelersnr = s.spelersnr
    group by s.spelersnr
    order by nr, aantal

opgave
------

Geef een overzicht van het aantal records in de vijf tabellen uit de
database tennis. Je mag hiervoor elke tabel manueel tellen. Sorteer op
label. \#\# oplossing

    select 'bestuursleden' as label,count(*) as aantal from bestuursleden
    union all
    select 'boetes' as label, count(*) as aantal from boetes
    union all
    select 'spelers' as label, count(*) as aantal from spelers
    union all
    select 'teams' as label, count(*) as aantal from teams
    union all
    select 'wedstrijden' as label, count(*) as aantal from wedstrijden
    --Dit is de juiste oplossing hierboven maar onder vind ik zelf een betere, performantere en makkelijk bruikbaar op andere tabellen
    SELECT relname as label,n_live_tup as aantal
    FROM pg_stat_user_tables
    where schemaname = 'tennis' and relname not ilike '%_%l'
    ORDER BY label;
    --sql dropbox rekent deze niet juist maar het geeft wel de juiste uitkomst

opgave
------

Geef een overzicht van de boetebedragen, aantal gewonnen en verloren
sets en aantal verschillende functies. Bekijk de output voor de manier
hoe het getoond moet worden. Sorteer van links naar rechts. Tip: Het
lijkt onlogisch, maar zelfs NULL krijgt een datatype en kan niet
impliciet wijzigen van datatype. \#\# oplossing

    select sum(bedrag)::text as boetebedrag, null as aantalgewonnen, null as aantalverloren, null as aantalfuncties
    from boetes
    union
    select null, sum(gewonnen)::text, null, null
    from wedstrijden
    union
    select null, null, sum(verloren)::text, null
    from wedstrijden
    union
    select null, null, null, count(distinct functie)::text
    from bestuursleden
    order by boetebedrag, aantalgewonnen, aantalverloren, aantalfuncties

opgave
------

Geef de spelersnummers die minstens één keer bestuurslid zijn geweest.
Gebruik hiervoor geen JOIN, DISTINCT, GROUP BY, IN, ANY, ALL of EXISTS.
Sorteer op spelersnr \#\# oplossing

    select spelers.spelersnr
    from spelers, bestuursleden
    where spelers.spelersnr = bestuursleden.spelersnr
    intersect
    select spelers.spelersnr
    from spelers
    order by spelersnr

opgave
------

Geef de spelersnummers die geen wedstrijd gespeeld hebben. Gebruik
hiervoor geen JOIN, DISTINCT, GROUP BY, IN, ANY, ALL of EXISTS. Sorteer
op spelersnr \#\# oplossing

    select spelers.spelersnr
    from spelers
    except
    select spelers.spelersnr
    from spelers, wedstrijden
    where spelers.spelersnr = wedstrijden.spelersnr
    order by spelersnr

opgave
------

Geef voor de spelers die bestuurslid en/of teamkapitein zijn hun naam en
een oplijsting van hun functienamen (huidig of verleden) en hun divisies
waarvoor ze kapitein zijn. Sorteer op spelersnaam en naam. Gebruik geen
OUTER JOIN of WHERE. \#\# oplossing

    select naam as spelersnaam, functie as naam
    from spelers s inner join bestuursleden b using (spelersnr)
    union
    select naam, divisie
    from spelers inner join teams t on spelers.spelersnr = t.spelersnr
    order by spelersnaam, naam

opgave
------

Geef een lijst van alle spelers die bestuurslid geweest zijn (of nu nog
zijn) en/of een boete hebben gehad hun aantal boetes en hun laatst
begonnen bestuursfunctie. Zorg dat spelers die boetes gehad hebben én
bestuurder zijn (geweest) twee keer voorkomen in de kolom ‘data’ twee
verschillende waardes. Sorteer op naam, voorletters en data. Gebruik de
to\_char functie voor het formaat van de geboortedatum (bv 12/12/1900).
\#\# oplossing

    select naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') as geboortedatum, 'Boetes: ' || count(betalingsnr) as data
    from spelers inner join boetes b using(spelersnr)
    group by spelersnr
    union
    select naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY'), functie
    from spelers inner join bestuursleden b using (spelersnr)
    where begin_datum = (select max(begin_datum) from bestuursleden where b.spelersnr = spelersnr)
    order by naam, voorletters, data

opgave
------

(Een variant op een vorige opgave) We zijn wat betreft boetes alleen
geïnteresseerd als de speler geboren is voor 1963.

Verder uitleg: Geef een lijst van ALLE spelers die bestuurslid geweest
zijn en/of een boete hebben gehad, (alleen als ze geboren zijn voor
1963) en hun laatst begonnen bestuursfunctie. Zorg ervoor dat spelers
die een bestuursfunctie hebben/hadden, geboren zijn voor 1963, maar geen
boete hebben gekregen, toch in het resultaat voorkomen met een extra
lijn ‘Geen boetes’. Spelers die geen bestuurslid zijn geweest, maar een
boete hebben gehad, komen één keer voor in het resultaat met hun aantal
boetes. Spelers die geen bestuurslid zijn geweest en geboren zijn na
1962, moeten ook één keer in het resultaat voorkomen met als data
‘Gewone speler’ (het kan zelfs zijn dat geen boete gekregen hebben, alle
spelers moeten sowieso getoond worden). Sorteer op naam, voorletters en
data (let op de hoofdletters in het veld data).

(ps: in deze oefening zit bijna alles wat je de voorbije twee jaar
gezien hebt van SQL) \#\# oplossing

    select naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') as geboortedatum, case when begin_datum is null then 'Gewone speler' else functie end as data
    from spelers left outer join bestuursleden b using(spelersnr)
    where begin_datum = (select max(begin_datum) from bestuursleden where b.spelersnr = bestuursleden.spelersnr)
       or (begin_datum is null and extract(year from geb_datum) > '1962')
    union
    select naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') as geboortedatum, case when betalingsnr is null then 'Geen boetes' else 'Boetes: ' || count(betalingsnr) end
    from spelers left outer join boetes using (spelersnr)
    where extract(year from geb_datum) < '1963'
    group by betalingsnr, spelersnr
    order by naam, voorletters, data

------------------------------------------------------------------------

Subq 1
======

opgave
------

Geef voor elke aanvoerder het spelersnr, de naam en het aantal boetes
dat voor hem of haar betaald is en het aantal teams dat hij of zij
aanvoert. Aanvoerders zonder boetes mogen niet getoond worden. Sorteer,
beginnend bij de eerste kolom, eindigend bij de laatste kolom. \#\#
oplossing

    select spelersnr, naam, voorletters, aantalboetes, aantalteams
    from spelers
           inner join (select count(*) as aantalboetes, spelersnr from boetes group by spelersnr) as t using (spelersnr)
           inner join (select count(*) as aantalteams, spelersnr from teams group by spelersnr) as u using(spelersnr)
    order by 1,2,3,4

opgave
------

Geef van elke speler het spelersnr, de naam en het verschil tussen zijn
of haar jaar van toetreding en het gemiddeld jaar van toetreding van de
spelers die in dezelfde plaats wonen. Sorteer op spelersnr. Zet het
berekende verschil om naar het datatype numeric met precisie 5 en schaal
3. \#\# oplossing

    select spelersnr, naam, voorletters, cast(jaartoe - gem as numeric(5,3))
    from spelers inner join (select avg(jaartoe) as gem, plaats from spelers group by plaats) as t using(plaats)
    order by spelersnr

opgave
------

Geef de spelersgegevens van de speler(s) met het hoogste bedrag (voor
één boete, niet het totaalbedrag). Als twee spelers een even hoge boete
gehad hebben, moeten beide spelers getoond worden (LIMIT is dus geen
optie). Sorteer alfabetisch op naam en voorletters. \#\# oplossing

    select spelersnr, voorletters, naam
    from spelers
    where spelersnr in (select spelersnr from boetes where bedrag = (select max(bedrag) from boetes))
    order by naam, voorletters

opgave
------

We willen een statistiek van hoeveel wedstrijden de spelers winnen. Geef
een lijst met aantal gewonnen wedstrijden en het aantal spelers dat dit
aantal wedstrijden gewonnen heeft. Dus bv als er vier spelers zijn die
elk drie wedstrijden hebben gewonnen, dan is de output:
aantal\_gewonnen: 3, aantal\_spelers: 4. Dit graag voor alle aantallen
gewonnen wedstrijden en alle spelers. Sorteer op aantal gewonnen
wedstrijden. \#\# oplossing

    select aantal_gewonnen, count(*) as aantal_spelers
    from (select count(wedstrijdnr) as aantal_gewonnen from  wedstrijden where gewonnen > verloren group by spelersnr) as gewonnen
    group by aantal_gewonnen
    order by aantal_gewonnen

opgave
------

Geef voor elke speler die een wedstrijd heeft gespeeld het spelersnr en
het totaal aantal boetes. Spelers die een wedstrijd gespeeld hebben,
maar geen boetes hebben, moeten ook getoond worden. Sorteer op het
aantal boetes en op spelersnr; \#\# oplossing

    select distinct spelersnr, aantalboetes
    from wedstrijden left outer join (
      select spelersnr, count(betalingsnr) as aantalboetes
      from boetes
      group by spelersnr) boetes using(spelersnr)
    order by aantalboetes, spelersnr

opgave
------

Geef voor alle spelers die geen penningmeester zijn of zijn geweest alle
gewonnen wedstrijden, gesorteerd op wedstrijdnummer. \#\# oplossing

    SELECT spelersnr, wedstrijdnr
    FROM wedstrijden
    WHERE spelersnr NOT IN (
        SELECT spelersnr
        FROM bestuursleden
        WHERE functie = 'Penningmeester')
        AND gewonnen > verloren
    ORDER BY wedstrijdnr

opgave
------

Je kan per speler berekenen hoeveel boetes die speler heeft gehad en wat
het totaalbedrag per speler is. Pas nu deze querie aan zodat per
verschillend aantal boetes wordt getoond hoe vaak dit aantal boetes
voorkwam.Sorteer eerst op de eerste kolom en daarna op de tweede kolom.
\#\# oplossing

    select aantalboetes as a, count(aantalboetes)
    from (select count(*) as aantalboetes from boetes
      group by spelersnr) as t
    group by aantalboetes
    order by 1,2

opgave
------

Geef van alle spelers het verschil tussen het jaar van toetreding en het
geboortejaar, maar geef alleen die spelers waarvan dat verschil groter
is dan 20. Sorteer deze gegevens beginnend bij de eerste kolom,
eindigend bij de laatste kolom. \#\# oplossing

    select spelersnr, naam, voorletters, toetredingsleeftijd
    from spelers inner join (select (jaartoe - extract(year from geb_datum)) as toetredingsleeftijd, spelersnr
      from spelers ) as t using(spelersnr)
    where toetredingsleeftijd > 20
    order by 1,2,3,4

opgave
------

Geef een lijst van alle spelers (spelersnr en woonplaats) die met
minstens twee in dezelfde plaats wonen. Sorteer aflopend op woonplaats,
daarna op spelersnr. \#\# oplossing

    SELECT spelersnr, plaats
    FROM spelers
    WHERE plaats in (SELECT plaats
        FROM spelers
        GROUP BY plaats
        HAVING count(plaats) >= 2)
    ORDER BY plaats DESC, spelersnr ASC

------------------------------------------------------------------------

subq 2
======

opgave
------

Geef het totaal aantal boetes, het totale boetebedrag, het minimum en
het maximum boetebedrag dat door onze club betaald werd. Let er hierbij
op dat er gehele getallen worden getoond (rond af indien nodig). Sorteer
van voor naar achter, oplopend. \#\# oplossing

    select round(count(*), 0) as aantal_boetes, round(sum(bedrag), 0) as totaal_bedrag, round(min(bedrag), 0) as minimum, round(max(bedrag)) as maximum
    from boetes
    order by 1,2,3,4

opgave
------

Geef voor elke aanvoerder het spelersnr, de naam en het aantal boetes
dat voor hem of haar betaald is en het aantal teams dat hij of zij
aanvoert. Toon enkel aanvoerders die boetes gekregen hebben. Sorteer van
voor naar achter, oplopend. \#\# oplossing

    select spelersnr, naam, voorletters, aantalboetes, aantalteams
    from spelers inner join (select spelersnr, count(betalingsnr) as aantalboetes from boetes group by spelersnr) as boetes using(spelersnr)
    inner join (select spelersnr, count(teamnr) as aantalteams from teams group by spelersnr) as teams using (spelersnr)
    order by 1,2,3,4,5

opgave
------

Geef een lijst met het spelersnummer en de naam van de spelers die in
Rijswijk wonen en die in 1980 een boete gekregen hebben van 25 euro
(meerdere voorwaarden dus). Gebruik hiervoor geen exists operator maar
wel zijn tegenhanger die meestal bij niet-gecorreleerde subquery’s wordt
gebruikt. Sorteer van voor naar achter, oplopend. \#\# oplossing

    select spelersnr, naam
    from spelers
    where plaats = 'Rijswijk' and spelersnr in (
      select spelersnr from boetes where extract(year from datum) = 1980 and bedrag  = 25
      )

opgave
------

Geef alle spelers voor wie meer boetes zijn betaald dan dat ze
wedstrijden hebben gespeeld. Zorg dat spelers zonder wedstrijd ook
getoond worden. Sorteer van voor naar achter, oplopend. \#\# oplossing

    select naam, voorletters, geb_datum
    from spelers
    where (select count(betalingsnr) from boetes where boetes.spelersnr = spelers.spelersnr)
            >
          (select count(wedstrijdnr) from wedstrijden where spelers.spelersnr = wedstrijden.spelersnr)
    order by 1,2,3

opgave
------

Geef alle spelers die geen wedstrijd voor team 1 heeft gespeeld. Sorteer
op naam, daarna op nr. \#\# oplossing

    select spelersnr, naam
    from spelers
    where spelersnr not in (select spelersnr from wedstrijden where teamnr = 1)
    order by naam, spelersnr

opgave
------

Geef voor elke speler die ooit een boete heeft betaald, de hoogste boete
weer en hoelang het geleden is dat deze boete werd betaald. Sorteer van
groot naar klein op bedrag en daarna omgekeerd op “leeftijd..” van de
boete. \#\# oplossing

    select spelersnr, bedrag, age(datum)
    from boetes
    where bedrag in (select max(bedrag) from boetes b where b.spelersnr = boetes.spelersnr )
    order by  bedrag desc , 3

opgave
------

Welke spelers hebben voor alle teams gespeeld uit de teamstabel ? (=
voor welke speler bestaat er geen enkel team waar de betreffende speler
nooit voor gespeeld heeft). Sorteer op spelers nummer. Gebruik de exists
operator. \#\# oplossing

    select spelersnr
    from spelers
    where not exists(
        select * from teams where not exists(
            select * from wedstrijden where teams.teamnr = wedstrijden.teamnr and spelers.spelersnr = wedstrijden.spelersnr
          )
      )
    order by spelersnr

opgave
------

Maak een lijst met de spelers (naam van de speler, voorletter en
woonplaats) die ooit gespeeld hebben voor een team dat nu in de tweede
divisie speelt en waarvoor geen enkele boete betaald werd voor 1 januari
1981. Sorteer van voor naar achter, oplopend. Zorg dat er geen dubbels
worden getoond. \#\# oplossing

    select naam, voorletters, plaats
    from spelers
    where spelersnr in (select w.spelersnr from wedstrijden w inner join teams using(teamnr) where divisie = 'tweede')
    and spelersnr in (select spelersnr from boetes right outer join spelers using(spelersnr) where datum > '1981-01-01'::date or datum is null)
    order by 1,2,3

opgave
------

Geef de twee laagste bondnrs terug. (tip: dwz er zijn dus minder dan 2
bondsnr die kleiner zijn). Sorteer op bondnr. Zonder het gebruik van
LIMIT. \#\# oplossing

    select bondsnr
    from spelers
    where 2 > (select count(*) from spelers s2 where spelers.bondsnr > s2.bondsnr) and bondsnr is not null
    order by bondsnr

------------------------------------------------------------------------
