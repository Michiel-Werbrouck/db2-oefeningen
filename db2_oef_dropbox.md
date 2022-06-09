# Extra 1

## 1.

> Hoeveel sets zijn er in totaal gewonnen, hoeveel sets werden in totaal verloren?
>
> Wat is het uiteindelijke saldo?

```sql
SELECT SUM(gewonnen) AS totaal_gewonnen, SUM(verloren) AS totaal_verloren, (SUM(gewonnen) - SUM(verloren)) AS saldo FROM wedstrijden
```

## 2.

> Geef een totaal overzicht van alle spelers, hun boetes en de functies die ooit vervuld hebben.
>
> Elke speler moet getoond worden, als ie een eventuele boete heeft gekregen en/of een eventuele functie als bestuurslid heeft gehad dan moet dit ook getoond worden.
>
> Toon de volledige naam, het bedrag en datum van de eventuele boete en de eventuele bestuursfuncties.
>
> Sorteer van voor naar achter.

```sql
SELECT naam, voorletters, functie, bedrag, datum FROM spelers
FULL OUTER JOIN boetes USING(spelersnr)
FULL OUTER JOIN bestuursleden USING(spelersnr)

ORDER BY 1,2,3,4,5 ASC
```


## 3.

> Geef een overzicht van de spelers die een boete hebben gekregen, indien deze boete meer van 90 euro is toon je informatie “pijnlijk” anders “te doen”.
>
> Toon de volledige naam en de categorie van de bijhorende boete.
>
> Sorteer van voor naar achter.

```sql
SELECT naam, voorletters, CASE WHEN
	bedrag > 90 THEN 'pijnlijk'
	ELSE 'te doen' END AS comment
FROM spelers INNER JOIN boetes
USING(spelersnr)
ORDER BY 1,2,3 asc
```

# Subq 1

## 1.

> Geef voor elke speler die een wedstrijd heeft gespeeld het spelersnr en het totaal aantal boetes. 
>
> Spelers die een wedstrijd gespeeld hebben, maar geen boetes hebben, moeten ook getoond worden.
>
> Sorteer op het aantal boetes en op spelersnr.

```sql
SELECT DISTINCT(spelersnr), aantalboetes FROM wedstrijden
LEFT OUTER JOIN (SELECT spelersnr, COUNT(betalingsnr) AS aantalboetes FROM boetes GROUP BY spelersnr)
boetes USING(spelersnr)
ORDER BY 2,1
```

## 2.

> Geef voor alle spelers die geen penningmeester zijn of zijn geweest alle gewonnen wedstrijden, gesorteerd op wedstrijdnummer.

```sql
SELECT spelersnr, wedstrijdnr
FROM wedstrijden
WHERE spelersnr NOT IN (SELECT spelersnr
FROM bestuursleden
WHERE functie = 'Penningmeester') AND gewonnen > verloren
ORDER BY wedstrijdnr
```

## 3. 

>Geef van elke speler het spelersnr, de naam en het verschil tussen zijn of haar jaar van toetreding en het gemiddeld jaar van toetreding van de spelers die in dezelfde plaats wonen. 
>
>Sorteer op spelersnr. 
>
> Toon 3 getallen na de komma, zet het verschil om naar numeric type met precisie van 5 en een schaal van 3.

```sql
SELECT spelersnr, naam, voorletters, CAST(jaartoe - gem AS numeric(5,3)) 
FROM spelers
INNER JOIN 
(SELECT AVG(jaartoe) as gem, plaats FROM spelers GROUP BY plaats) 
AS "t" USING(plaats)
ORDER BY spelersnr
```

## 4.

> Geef van alle spelers het verschil tussen het jaar van toetreding en het geboortejaar, maar geef alleen die spelers waarvan dat verschil groter is dan 20. 
>
> Sorteer deze gegevens beginnend bij de eerste kolom, eindigend bij de laatste kolom.

```sql
SELECT spelersnr, naam, voorletters, toetredingsleeftijd
FROM spelers 
INNER JOIN (SELECT (jaartoe - EXTRACT(YEAR FROM geb_datum)) AS toetredingsleeftijd, spelersnr FROM spelers) AS t USING(spelersnr)
WHERE toetredingsleeftijd > 20
ORDER BY 1,2,3,4
```

## 5.

> Geef voor elke aanvoerder het spelersnr, de naam en het aantal boetes dat voor hem of haar betaald is en het aantal teams dat hij of zij aanvoert. 
>
> Aanvoerders zonder boetes mogen niet getoond worden. 
>
> Sorteer, beginnend bij de eerste kolom, eindigend bij de laatste kolom.

```sql
SELECT spelersnr, naam, voorletters, aantalboetes, aantalteams
FROM spelers
INNER JOIN (SELECT COUNT(*) AS aantalboetes, spelersnr FROM boetes GROUP BY spelersnr) AS b USING(spelersnr)
INNER JOIN (SELECT COUNT(*) AS aantalteams, spelersnr FROM teams GROUP BY spelersnr) AS "t" USING (spelersnr)
GROUP BY 1,2,3,4,5
```

## 6.

> Geef van elke speler het spelersnr, de naam en het verschil tussen zijn of haar jaar van toetreding en het gemiddeld jaar van toetreding van de spelers die in dezelfde plaats wonen. 
>
> Sorteer op spelersnr. 
>
> Zet het berekende verschil om naar datatype numeric met precisie 5 en schaal 3.

```sql
SELECT spelersnr, naam, voorletters, CAST(jaartoe - gem AS numeric(5,3))
FROM spelers
INNER JOIN (SELECT AVG(jaartoe) AS gem, plaats FROM spelers GROUP BY plaats) AS s USING(plaats)
ORDER BY spelersnr
```

## 7.

> Geef een lijst van alle spelers (spelersnr en woonplaats) die met minstens twee in dezelfde plaats wonen. 
>
> Sorteer aflopend op woonplaats, daarna op spelersnr.

```sql
SELECT spelersnr, plaats
FROM spelers
WHERE plaats in (SELECT plaats
	FROM spelers
	GROUP BY plaats
	HAVING count(plaats) >= 2)
ORDER BY plaats DESC, spelersnr ASC
```

## 8. 

> Geef de spelersgegevens van de speler(s) met het hoogste bedrag (voor één boete, niet het totaalbedrag).
>
> Als twee spelers een even hoge boete gehad hebben, moeten beide spelers getoond worden (LIMIT is dus geen optie).
>
> Sorteer alfabetisch op naam en voorletters.

```sql
SELECT spelersnr, voorletters, naam
FROM spelers
WHERE spelersnr IN (SELECT spelersnr
FROM boetes
WHERE bedrag = (SELECT max(bedrag) FROM boetes))
ORDER BY naam, voorletters
```

## 9.

> We willen een statistiek van hoeveel wedstrijden de spelers winnen.
>
> Geef een lijst met aantal gewonnen wedstrijden en het aantal spelers dat dit aantal wedstrijden gewonnen heeft.
>
> Dus bv als er vier spelers zijn die elk drie wedstrijden hebben gewonnen, dan is de output: aantal_gewonnen: 3, aantal_spelers: 4.
>
> Dit graag voor alle aantallen gewonnen wedstrijden en alle spelers.
>
> Sorteer op aantal gewonnen wedstrijden.

```sql
SELECT aantal_gewonnen, COUNT(*) AS aantal_spelers
FROM (SELECT COUNT(wedstrijdnr) AS aantal_gewonnen FROM wedstrijden WHERE gewonnen > verloren GROUP BY spelersnr) AS gewonnen
GROUP BY aantal_gewonnen
ORDER BY aantal_gewonnen
```

# Herh 1.

## 1.

> Geef voor alle huidige bestuurleden hun functie en de lijst van boetes die voor hen werd betaald.
>
> Omdat je dit wil vergelijken met de boetebedragen die betaald werden voor leden die niet in het bestuur zitten, wil je deze boetebedragen ook opnemen in de tweede kolom van je resultaat.
>
> Sorteer je antwoord eerst op functie en daarna op het boetebedrag.

```sql
SELECT functie, bedrag
FROM bestuursleden FULL OUTER JOIN boetes USING(spelersnr)
WHERE eind_datum IS NULL
ORDER BY functie, bedrag
```

## 2.

> Geef voor elke mannelijke speler wiens naam minstens 2 keer de letter 'e' bevat zijn functie die hij op dit moment uitoefent, als die er op dit moment één heeft.
>
> Sorteer op naam en functie.

```sql
SELECT naam, geslacht, functie FROM bestuursleden
FULL JOIN spelers ON bestuursleden.spelersnr = spelers.spelersnr AND eind_datum IS null
WHERE geslacht = 'M' AND naam LIKE '%e%e%'
ORDER BY naam, functie
```

## 3.

> Geef een aflopend gesorteerde lijst van de nummers van alle spelers waarvoor nog geen boete werd betaald en die nog nooit in het bestuur van de tennisvereniging hebben gezeten.

```sql
SELECT spelers.spelersnr FROM bestuursleden
FULL OUTER JOIN spelers USING(spelersnr)
LEFT JOIN boetes USING(spelersnr)
WHERE bedrag IS null AND bestuursleden.spelersnr IS null
ORDER BY spelers.spelersnr DESC
```

## 4.

> Geef de nummers van alle spelers (ook al hebben ze geen wedstrijd gespeeld), een lijst van de nummers van alle wedstrijden die ze gewonnen hebben, alsook het verschil in sets waarmee ze gewonnen hebben.
>
> Toon je resultaten gesorteerd volgens dit verschil van groot naar klein en oplopend op spelersnr.

```sql
SELECT spelers.spelersnr, wedstrijdnr, (gewonnen - verloren) AS verschil FROM spelers
FULL OUTER JOIN wedstrijden on spelers.spelersnr = wedstrijden.spelersnr AND (gewonnen - verloren ) > 0 
WHERE (gewonnen - verloren ) > 0 or wedstrijdnr is null
ORDER BY verschil DESC, spelersnr ASC
```

## 5.

> Geef van elke speler (die wedstrijden gespeeld heeft en boetes gekregen heeft) wonend in Rijswijk het spelersnr, de naam, de lijst met boetebedragen en de lijst met teams waarvoor hij/zij een wedstrijd gespeeld heeft.
>
> Geef het resultaat volgens oplopend spelersnr en boetebedrag.

```sql
SELECT spelersnr, naam, bedrag, teamnr FROM spelers
LEFT JOIN boetes USING(spelersnr)
INNER JOIN wedstrijden USING(spelersnr)
WHERE plaats LIKE '%Rijswijk%' 
ORDER BY spelersnr, bedrag
```

## 6.

> Geef het spelersnummer en bondsnummer van alle spelers die jonger zijn dan de speler met bondsnummer 8467.
>
> Gebruik een INNER JOIN
>
> Sorteer op 1,2

```sql
SELECT DISTINCT s2.spelersnr, s2.bondsnr
FROM spelers s1 INNER JOIN spelers s2 ON (s1.bondsnr = '8467')
WHERE s1.geb_datum < s2.geb_datum
ORDER BY 1, 2
```

## 7.

> Geef een lijst met het spelersnummer, de naam van de speler, de datum van de boete en het bedrag van de boete van al de spelers die een boete gekregen hebben met een bedrag groter dan 45,50 euro en in Rijswijk wonen.
> 
> Geef expliciet aan welke join je gebruikt.

```sql
SELECT spelersnr, naam, datum, bedrag FROM spelers 
INNER JOIN boetes USING(spelersnr)
WHERE bedrag > 45.50 AND plaats = 'Rijswijk'
```

## 8.

> Geef voor alle vrouwelijke spelers die in Den Haag, Zoetermeer of Leiden wonen
het spelersnummer, hun woonplaats en een lijst van de teams waarvoor ze ooit gespeeld hebben, als ze ooit een wedstrijd gespeeld hebben. 
>
> Sorteer volgens 1,2,3

```sql
SELECT spelers.spelersnr, plaats, teamnr FROM spelers
FULL OUTER JOIN wedstrijden USING(spelersnr)
FULL OUTER JOIN teams USING(teamnr)
WHERE geslacht = 'V' AND plaats LIKE '%Den Haag%' OR plaats LIKE '%Leiden%' OR plaats LIKE '%Zoetermeer%'
ORDER BY 1,2,3
```

## 9.

> Geef voor de actieve bestuursleden zonder boete hun laatste gespeelde wedstrijd (die met het hoogste wedstrijdnummer).
>
> Sorteer aflopend op spelersnr.

```sql
SELECT spelersnr, MAX(wedstrijdnr) AS laatstewedstrijd  FROM wedstrijden
FULL OUTER JOIN boetes USING(spelersnr)
INNER JOIN bestuursleden USING(spelersnr)
WHERE eind_datum IS null AND bedrag IS null
GROUP BY spelersnr
ORDER BY spelersnr DESC
```

## 10.

> Geef per team de leeftijd van de aanvoerder (tip: postgresql heeft een AGE() functie) en het aantal verschillende spelers dat voor dit team gespeeld heeft.
>
> Alleen teams waarvoor wedstrijden zijn gespeeld en die een aanvoerder hebben, moeten vermeld worden.
>
> Sorteer op leeftijd en daarna op aantal verschillende spelers en daarna op teamnr.

```sql
SELECT DISTINCT w2.teamnr, extract(year FROM age(geb_datum)) || ' jaar' AS leeftijd ,count(DISTINCT w2.spelersnr) AS aantalspelers
FROM teams INNER JOIN wedstrijden w2 ON teams.teamnr = w2.teamnr INNER JOIN spelers s2 ON teams.spelersnr = s2.spelersnr
GROUP BY w2.teamnr, s2.geb_datum
ORDER BY leeftijd, aantalspelers, w2.teamnr
```

# Herh 2

## 1.

> Welke reizigers hebben al meer dan 1 reis ondernomen waarvoor ze meer dan 2,5 miljoen euro moesten betalen?
>
> Sorteer op naam

```sql
SELECT naam, COUNT(reizen.reisnr) AS aantal_reizen FROM reizen
INNER JOIN deelnames USING(reisnr)
INNER JOIN klanten USING(klantnr)
WHERE prijs > 2.5
GROUP BY naam
HAVING COUNT(reizen.reisnr) > 1
ORDER BY naam
```

## 2.

> Op welke planeten verblijft men gemiddeld langer dan 2 dagen?
>
> Sorteer op objectnaam.

```sql
SELECT objectnaam, AVG(verblijfsduur) FROM bezoeken
INNER JOIN hemelobjecten USING(objectnaam)
WHERE satellietvan = 'Zon'
GROUP BY objectnaam
HAVING avg(verblijfsduur) > 2 
ORDER BY objectnaam
```

## 3.

> Maak een lijst met een overzicht van de reizen met deelnemers. Geef per reis het aantal deelnemers van elke reis. 
>
> Orden de lijst dalend op basis van het aantal deelnemers, als er eenzelfde aantal deelnemers is, moeten deze stijgend geordend worden volgens reisnummer.

```sql
SELECT reisnr, COUNT(klantnr) AS deelnemers FROM deelnames
GROUP BY reisnr
ORDER BY deelnemers DESC , reisnr ASC
```

## 4.

> Geef een lijst van alle reizen met tenminste 1 bezoek of passage aan de Maan of aan Mars.
>
> De lijst moet stijgend gesorteerd zijn op basis van het vertrekdatum.

```sql
SELECT reisnr, vertrekdatum FROM bezoeken
INNER JOIN reizen USING(reisnr)
WHERE objectnaam LIKE '%Maan%' OR objectnaam LIKE '%Mars%'
GROUP BY reisnr, vertrekdatum
HAVING COUNT(reisnr) > 0
ORDER BY vertrekdatum ASC
```

## 5.

> Maak een lijst met een overzicht van de reizen en het aantal deelnemers van elke reis.
>
> Orden de lijst dalend op basis van het aantal deelnemers, als er eenzelfde aantal deelnemers is, moeten deze stijgend geordend worden volgens reisnummer.
>
> Zorg dat reizen zonder deelnames ook in het resultaat staan.

```sql
SELECT reisnr, COUNT(klantnr) AS deelnemers FROM deelnames
FULL OUTER JOIN reizen USING(reisnr)
GROUP BY reisnr
ORDER BY reisnr ASC
```

## 6.

> Welke reizen hebben exact drie verschillende hemelobjecten als reisdoel?
>
> Sorteer op reisnr.

```sql
SELECT reisnr FROM bezoeken
INNER JOIN hemelobjecten USING(objectnaam)
GROUP BY reisnr
HAVING COUNT(DISTINCT(objectnaam)) = 3
```

## 7.

> Welke planeten met meer dan 7 manen worden bezocht (met of zonder verblijf)?
>
> Sorteer aflopend op basis van het aantal manen.
>
> Let erop dat je planeten die meerdere keren bezocht worden, niet dubbel telt.

```sql
SELECT p.objectnaam
FROM hemelobjecten AS p
INNER JOIN hemelobjecten AS m ON m.satellietvan = p.objectnaam
INNER JOIN bezoeken AS b ON b.objectnaam = p.objectnaam
INNER JOIN hemelobjecten AS zon ON zon.objectnaam = p.satellietvan AND zon.satellietvan IS NULL
GROUP BY p.objectnaam
HAVING COUNT(distinct m.objectnaam) > 7
ORDER BY COUNT(distinct m.objectnaam) DESC
```

## 8.

> Geef een lijst met alle planeten en per planeet zijn satellieten.
>
> Sorteer op planeet en daarna op satelliet.

```sql
SELECT p.objectnaam AS planeet, m.objectnaam AS maan
FROM hemelobjecten AS p
LEFT OUTER JOIN hemelobjecten AS m ON m.satellietvan = p.objectnaam
WHERE p.satellietvan = 'Zon'
ORDER BY planeet, maan
```

## 9.

> Geef de leeftijd van de jongste klant op moment van vertrek.

```sql
SELECT EXTRACT(year FROM min(age(r.vertrekdatum, k.geboortedatum))) AS jongsteleeftijd
FROM deelnames AS d
INNER JOIN reizen AS r USING (reisnr)
INNER JOIN klanten AS k USING (klantnr)
```

## 10.

> Geef voor elk hemelobject de minimale en maximale gemiddelde afstand tot zijn zon (de centrale ster in een sterrenstELSEl) als je weet dat de kolom 'afstand' de gemiddelde afstand bevat tot het hemelobject waarrond ze draaien.
>
> Met de grootte van het hemelobject hoef je geen rekening te houden.
>
> Sorteer op minimale afstand en op objectnaam.
>
> Tip: bekijk de functie COALESCE om met null-waardes rekening te houden.
>
> Opm: berekende afstanden dienen altijd positief te zijn, je kan eventueel abs() functie gebruiken.

```sql
SELECT h1.objectnaam, COALESCE(h1.afstand,0) + COALESCE(h2.afstand, 0)
AS maximale_afstand, ABS(COALESCE(h2.afstand, 0) - COALESCE(h1.afstand, 0))
AS minimale_afstand
FROM hemelobjecten h1 LEFT JOIN hemelobjecten h2
ON h2.objectnaam = h1.satellietvan
ORDER BY minimale_afstand, h1.objectnaam
```

# Subq 2

## 1.

> Geef het totaal aantal boetes, het totale boetebedrag, het minimum en het maximum boetebedrag dat door onze club betaald werd. 
>
> Let er hierbij op dat er gehele getallen worden getoond (rond af indien nodig).
>
> Sorteer van voor naar achter, oplopend.
>
> Deze opgave behoeft geen subquery.

_No clue waarom ze hier die dan bijgezet hebben als het hoofdstuk letterlijk over fucking subquery's is but anyway hier is het antwoord cuties_

```sql
SELECT 
COUNT(betalingsnr) AS aantal_boetes, 
ROUND(SUM(bedrag),0) AS totaal_bedrag, 
ROUND(MIN(bedrag),0) AS minimum, 
ROUND(MAX(bedrag),0) AS maximum
FROM boetes
```

## 2.

> Geef voor elke aanvoerder het spelersnr, de naam en het aantal boetes dat voor hem of haar betaald is en het aantal teams dat hij of zij aanvoert.
>
> Toon enkel aanvoerders die boetes gekregen hebben.
>
> Sorteer van voor naar achter, oplopend.

```sql
SELECT spelersnr, naam, voorletters, aantalboetes, aantalteams
FROM spelers 
INNER JOIN 
    (SELECT spelersnr, COUNT(betalingsnr) AS aantalboetes FROM boetes GROUP BY spelersnr) AS boetes USING(spelersnr)
INNER JOIN
    (SELECT spelersnr, COUNT(teamnr) AS aantalteams FROM teams GROUP BY spelersnr) AS teams USING(spelersnr)
ORDER BY 4,3,2,1 ASC
```

## 3.

> Geef een lijst met het spelersnummer en de naam van de spelers die in Rijswijk wonen en die in 1980 een boete gekregen hebben van 25 euro (meerdere voorwaarden dus).
>
> Gebruik hiervoor geen exists operator maar wel zijn tegenhanger die meestal bij niet-gecorreleerde subquery's wordt gebruikt. 
>
> Sorteer van voor naar achter, oplopend.

```sql
SELECT spelersnr, naam
FROM spelers
WHERE plaats='Rijswijk' AND spelersnr IN
(SELECT spelersnr FROM boetes WHERE EXTRACT(YEAR FROM datum) = 1980 AND bedrag = 25)
```

## 4.

> Geef alle spelers voor wie meer boetes zijn betaald dan dat ze wedstrijden hebben gespeeld.
>
> Zorg dat spelers zonder wedstrijd ook getoond worden.
>
> Sorteer van voor naar achter, oplopend.

```sql
SELECT naam,  voorletters, geb_datum FROM spelers
WHERE (SELECT COUNT(betalingsnr) FROM boetes WHERE boetes.spelersnr = spelers.spelersnr)
	  > (SELECT COUNT(wedstrijdnr) FROM wedstrijden WHERE wedstrijden.spelersnr = spelers.spelersnr)
ORDER BY 1,2,3 ASC
```

## 5. 

> Geef alle spelers die geen wedstrijd voor team 1 heeft gespeeld. 
>
> Sorteer op naam, daarna op nr.

```sql
SELECT spelersnr, naam
FROM spelers
WHERE spelersnr NOT IN (SELECT spelersnr FROM wedstrijden WHERE teamnr = 1)
ORDER BY naam, spelersnr
```

## 6.

> Geef voor elke speler die ooit een boete heeft betaald, de hoogste boete weer en hoelang het geleden is dat deze boete werd betaald.
>
> Sorteer van groot naar klein op bedrag en daarna omgekeerd op “leeftijd..” van de boete.

```sql
SELECT spelersnr, bedrag, AGE(datum) FROM boetes AS b
WHERE bedrag IN (SELECT MAX(bedrag) FROM boetes WHERE boetes.spelersnr = b.spelersnr)
ORDER BY bedrag DESC, datum DESC
```

## 7.

> Welke spelers hebben voor alle teams gespeeld uit de teamstabel ?
>
> (= voor welke speler bestaat er geen enkel team waar de betreffende speler nooit voor gespeeld heeft).
>
> Sorteer op spelers nummer
>
> Gebruik de exists operator.

```sql
SELECT spelersnr
FROM spelers
WHERE NOT EXISTS
(SELECT * FROM teams WHERE NOT EXISTS
    (SELECT * FROM wedstrijden WHERE teams.teamnr = wedstrijden.teamnr AND spelers.spelersnr = wedstrijden.spelersnr))
ORDER BY spelersnr
```

## 8.

> Maak een lijst met de spelers (naam van de speler, voorletter en woonplaats) die ooit gespeeld hebben voor een team dat nu in de tweede divisie speelt en waarvoor geen enkele boete betaald werd voor 1 januari 1981.
>
> Sorteer van voor naar achter, oplopend.
>
> Zorg dat er geen dubbels worden getoond.

```sql
SELECT naam, voorletters, plaats
FROM spelers
WHERE spelersnr IN 
    (SELECT w.spelersnr FROM wedstrijden w INNER JOIN teams USING(teamnr) WHERE divisie = 'tweede')
AND spelersnr IN 
(SELECT spelersnr FROM boetes RIGHT OUTER JOIN spelers USING(spelersnr) WHERE datum > '1981-01-01'::DATE OR datum IS NULL)
ORDER BY 1,2,3
```

## 9.

> Geef de twee laagste bondnrs terug.
>
> (tip: dwz er zijn dus minder dan 2 bondsnr die kleiner zijn).
>
> Sorteer op bondnr.
>
> Zonder het gebruik van LIMIT.

```sql
SELECT bondsnr
FROM spelers
WHERE 2  > (SELECT count(*)
	   FROM spelers s2
	   WHERE spelers.bondsnr > s2.bondsnr)
	AND bondsnr IS NOT NULL
ORDER BY bondsnr
```

# Joins 1

## 1.

> Maak een lijst met alle spelers die ooit een boete gekregen hebben die hoger is dan 50 euro.
>
> Geen dubbels.
>
> Sorteer van voor naar achter.

```sql
SELECT DISTINCT naam, voorletters, plaats
FROM spelers inner join boetes b on spelers.spelersnr = b.spelersnr
WHERE bedrag > 50
order by naam
```

## 2.

> Geef van elke boete het betalingsnr, het boetebedrag en het percentage dat het bedrag uitmaakt van de som van alle bedragen.
> 
> Sorteer deze data op het betalingsnr.
>
> Zorg dat er maar twee getallen na de komma getoond worden (rond af).
>
> Sorteer van voor naar achter.

```sql
SELECT betalingsnr, bedrag, round((bedrag/(SELECT sum(bedrag) FROM boetes)*100),2)
FROM boetes
GROUP BY betalingsnr
ORDER BY betalingsnr
```

## 3. 

> Geef chronologisch de spelersnummers van de bestuursleden die voorzitter zijn of geweest zijn (chronologisch op begindatum van het voorzitterschap) met vermelding van deze begindatum, alsook hun naam en huidig adres.
>
> Als het vollegie adres niet gekend is dan moet “adres ongekend” weergegeven worden. 
>
> Sorteer van voor naar achter.

```sql
SELECT begin_datum, naam, CASE
  WHEN straat IS NULL OR huisnr IS NULL OR postcode IS NULL OR plaats IS NULL
    THEN 'adres ongekend'
  ELSE straat || ' ' || huisnr || ' ' || plaats || ' ' || postcode
END AS adres
FROM bestuursleden INNER JOIN spelers s ON bestuursleden.spelersnr = s.spelersnr
WHERE functie = 'Voorzitter'
ORDER BY begin_datum
```

## 4.

> Geef alle wedstrijden van het team waarvan speler 6 aanvoerder is.
>
> Sorteer

```sql
SELECT wedstrijdnr FROM spelers
INNER JOIN wedstrijden USING(spelersnr)
INNER JOIN teams USING(teamnr) 
WHERE teams.spelersnr = 6
ORDER BY 1 ASC;
```

## 5. 

> Geef alle spelers die geen enkele wedstrijd voor team 1 hebben gespeeld. 
>
> Sorteer op naam, daarna op spelersnr.

```sql
SELECT s.spelersnr, naam
FROM spelers s 
WHERE spelersnr not in (SELECT spelersnr FROM wedstrijden WHERE teamnr = 1)
ORDER BY 2,1
```

## 6.

> Maak een lijst met de spelers (naam van de speler, voorletter en woonplaats) die ooit gespeeld hebben voor een team dat nu in de tweede divisie speelt en waarvoor geen enkele boete betaald werd voor 1 januari 1981. 
>
> Geen dubbels, sorteer van voor naar achter.

```sql
SELECT naam, voorletters, plaats
FROM spelers
WHERE spelersnr IN 
    (SELECT w.spelersnr FROM wedstrijden w INNER JOIN teams USING(teamnr) WHERE divisie = 'tweede')

AND spelersnr IN 
    (SELECT spelersnr FROM boetes RIGHT OUTER JOIN spelers USING(spelersnr) WHERE datum > '1981-01-01'::DATE OR datum IS NULL)

ORDER BY 1,2,3
```

# Set Oper

## 1.

> Geef een overzicht van alle spelers, gevolgd door alle bestuursleden, gesorteerd op jaar van toetreding of beginjaar van hun functie en vervolgens op spelersnr.
>
> Geen dubbels tonen.

```sql
SELECT spelersnr AS veld1, naam AS veld2, jaartoe AS veld3
FROM spelers
UNION
SELECT spelersnr, functie, EXTRACT(YEAR FROM DATE(begin_datum))
FROM bestuursleden
ORDER BY veld3, veld1
```

## 2.

> Geef een lijst met alle spelersnrs, naam en het aantalwedstrijden ze gespeeld hebben en op een nieuwe lijn het aantal bestuursfuncties die ze hebben/hadden.
>
> Spelers die zowel wedstrijden gespeeld hebben als bestuurslid zijn, komen dus twee keer voor in het resultaat.
>
> Sorteer op spelersnr en aantal. 
>
> Geen dubbels tonen.

```sql
SELECT spelersnr AS nr, naam, aantal
FROM spelers INNER JOIN (
  SELECT spelersnr, count(wedstrijdnr) AS aantal
  FROM wedstrijden
  GROUP BY spelersnr
  ) t USING (spelersnr)
UNION
SELECT s.spelersnr, naam, count(*)
FROM bestuursleden INNER JOIN spelers s ON bestuursleden.spelersnr = s.spelersnr
GROUP BY s.spelersnr
ORDER BY nr, aantal
```

## 3.

> Geef een overzicht van het aantal records in de vijf tabellen uit de database tennis.
> 
> Je mag hiervoor elke tabel manueel tellen.
>
> Sorteer op label.

```sql
SELECT 'bestuursleden' AS label,count(*) AS aantal FROM bestuursleden
UNION ALL
SELECT 'boetes' AS label, count(*) AS aantal FROM boetes
UNION ALL
SELECT 'spelers' AS label, count(*) AS aantal FROM spelers
UNION ALL
SELECT 'teams' AS label, count(*) AS aantal FROM teams
UNION ALL
SELECT 'wedstrijden' AS label, count(*) AS aantal FROM wedstrijden
```

## 4. 

> Geef de spelersnummers die minstens één keer bestuurslid zijn geweest.
>
> Gebruik hiervoor geen `JOIN, DISTINCT, GROUP BY, IN, ANY, ALL of EXISTS.`
>
> Sorteer op spelersnr.

```sql
SELECT spelers.spelersnr
FROM spelers, bestuursleden
WHERE spelers.spelersnr = bestuursleden.spelersnr
INTERSECT
SELECT spelers.spelersnr
FROM spelers
ORDER BY spelersnr
```

## 5.

> Geef de spelersnummers die geen wedstrijd gespeeld hebben.
>
> Gebruik hiervoor geen `JOIN, DISTINCT, GROUP BY, IN, ANY, ALL of EXISTS.`
>
> Sorteer op spelersnr.

```sql
SELECT spelers.spelersnr
FROM spelers
EXCEPT
SELECT spelers.spelersnr
FROM spelers, wedstrijden
WHERE spelers.spelersnr = wedstrijden.spelersnr
ORDER BY spelersnr
```

## 6.

> Geef voor de spelers die bestuurslid en/of teamkapitein zijn hun naam en een oplijsting van hun functienamen (huidig of verleden) en hun divisies waarvoor ze kapitein zijn.
>
> Sorteer op spelersnaam en naam. 
>
> Gebruik geen OUTER JOIN of WHERE.

```sql
SELECT naam AS spelersnaam, functie AS naam
FROM spelers s INNER JOIN bestuursleden b USING (spelersnr)
UNION
SELECT naam, divisie
FROM spelers INNER JOIN teams t ON spelers.spelersnr = t.spelersnr
ORDER BY spelersnaam, naam
```

## 7.

> Geef een lijst van alle spelers die bestuurslid geweest zijn (of nu nog zijn) en/of een boete hebben gehad hun aantal boetes en hun laatst begonnen bestuursfunctie.
>
> Zorg dat spelers die boetes gehad hebben én bestuurder zijn (geweest) twee keer voorkomen in de kolom 'data' twee verschillende waardes.
>
> Sorteer op naam, voorletters en data.
>
> Gebruik de to_char functie voor het formaat van de geboortedatum (bv 12/12/1900).

```sql
SELECT naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') AS geboortedatum, 
'Boetes: ' || COUNT(betalingsnr) AS DATA
FROM spelers INNER JOIN boetes b USING(spelersnr)
GROUP BY 1,2,3
UNION
SELECT naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY'), functie
FROM spelers INNER JOIN bestuursleden b USING (spelersnr)
WHERE begin_datum = (SELECT max(begin_datum) FROM bestuursleden WHERE b.spelersnr = spelersnr)
ORDER BY naam, voorletters, data
```

## 8.

> (Een variant op een vorige opgave)
>
> We zijn wat betreft boetes alleen geïnteresseerd als de speler geboren is voor 1963.
>
> Verder uitleg:
>
> Geef een lijst van ALLE spelers die bestuurslid geweest zijn en/of een boete hebben gehad, (alleen als ze geboren zijn voor 1963) en hun laatst begonnen bestuursfunctie. 
>
> Zorg ervoor dat spelers die een bestuursfunctie hebben/hadden, geboren zijn voor 1963, maar geen boete hebben gekregen, toch in het resultaat voorkomen met een extra lijn 'Geen boetes'.
>
> Spelers die geen bestuurslid zijn geweest, maar een boete hebben gehad, komen één keer voor in het resultaat met hun aantal boetes.
>
> Spelers die geen bestuurslid zijn geweest, maar een boete hebben gehad, komen één keer voor in het resultaat met hun aantal boetes.
>
> Spelers die geen bestuurslid zijn geweest en geboren zijn na 1962, moeten ook één keer in het resultaat voorkomen met als data 'Gewone speler' (het kan zelfs zijn dat geen boete gekregen hebben, alle spelers moeten sowieso getoond worden).
>
> Sorteer op naam, voorletters en data (let op de hoofdletters in het veld data).
>
> (ps: in deze oefening zit bijna alles wat je de voorbije twee jaar gezien hebt van SQL).

_holy fuck wat een clusterfuck van een oefening, idealiter zou ge dit beter in 2 queries doen, maarja wie ben ik nu om daar over te oordelen?_

```sql
SELECT naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') AS geboortedatum, 
CASE 
    WHEN begin_datum IS NULL THEN 'Gewone speler' 
    ELSE functie 
END AS data

FROM spelers LEFT OUTER JOIN bestuursleden b USING(spelersnr)
WHERE begin_datum = (SELECT max(begin_datum) FROM bestuursleden WHERE b.spelersnr = bestuursleden.spelersnr)
   OR (begin_datum IS NULL AND extract(year FROM geb_datum) > '1962')
UNION


SELECT naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') AS geboortedatum, 
CASE 
    WHEN betalingsnr IS NULL THEN 'Geen boetes' 
    ELSE 'Boetes: ' || count(betalingsnr) 
END

FROM spelers 
LEFT OUTER JOIN boetes USING (spelersnr)
WHERE extract(year FROM geb_datum) < '1963'
GROUP BY betalingsnr, spelersnr
ORDER BY naam, voorletters, data
```

_Ik heb die een beetje opgebroken, anders was deze query te onoverzichtelijk_

# Extra 2

## 1. 

> Geef van elke speler het spelersnr, de naam en het verschil tussen zijn of haar jaar van toetreding en het gemiddeld jaar van toetreding van de spelers die in dezelfde plaats wonen.
>
> Sorteer van voor naar achter. 
> 
> Toon 3 getallen na de komma, maximaal 2 voor de komma
>
> Gebruik een cast functie.

```sql
SELECT spelersnr, naam, voorletters, jaartoe - ROUND(
    (SELECT AVG(jaartoe) FROM spelers s2 WHERE s.plaats = s2.plaats),3) AS numeric
FROM spelers s
ORDER BY 1,2,3,4
```

## 2. 

> Toon alle mogelijke combinaties van de letters 'x' en 'y'. 
> 
> Tip zie handboek, voorbeeld met cijfers.
>
> (Wat is een cartesisch product?). 
>
> Sorteer.

_Ik snap ni echt wa hij hier mee wilt zeggen, ma oke_

```sql
SELECT CONCAT(t.column1, tt.column1) AS "?column?"
FROM 
(
    VALUES('x'), ('y')
) AS t 

CROSS JOIN 

(
    VALUES('x'), ('y')
) AS tt
```

## 3.

> Geef de wedstrijden die door spelers zijn gespeeld die in Leuven, Rotterdam of Leiden wonen.
>
> Sorteer van voor naar achter.
>
> Geef alle (*) informatie van wedstrijden.

```sql
SELECT w.*
FROM spelers INNER JOIN wedstrijden w ON spelers.spelersnr = w.spelersnr
WHERE plaats LIKE 'Rotterdam' OR plaats LIKE 'Leuven' OR plaats LIKE 'Leiden'
```


# GROUP BY

## 1.

> Geef voor elke geboortejaar van de klanten, het aantal klanten, het kleinste klantennummer en het grootste klantennummer
>
> Geef ook het totaal aantal klanten en het kleinste en grootste klantennummer.
>
> Sorteer van voor naar achter.

```sql
SELECT EXTRACT(YEAR FROM geboortedatum), COUNT(*), MIN(klanten.klantnr), MAX(klanten.klantnr)
FROM klanten
GROUP BY (EXTRACT(YEAR FROM geboortedatum))
UNION
(SELECT null , COUNT(*), MIN(klantnr), MAX(klantnr) FROM klanten)
ORDER BY 1
```

## 2.

> We willen telkens het aantal reizen en de totale prijs van de volgende situaties. 
>
> Voor elke maand van vertrek, voor elk tiental van de reisduur, voor de combinatie van maand van vertrek en tiental van de reisduur en het het totale plaatje.
>
> Sorteer van voor naar achter.

```sql
SELECT EXTRAXT(MONTH FROM vertrekdatum) AS date_part, ROUND((reisduur/10), 0), COUNT(vertrekdatum), SUM(prijs)
FROM reizen
GROUP BY ROLLUP(date_part, ROUND((reisduur/10), 0))
UNION ALL
SELECT null, ROUND((reisduur/10),0), COUNT(vertrekdatum), SUM(prijs)
FROM reizen
GROUP BY ROUND((reisduur/10),0)
ORDER BY 1,2,3,4
```

## 3.

> Geef voor de hemelobjecten een diameter groter dan 1000 het aantal hemelobjecten en de gemiddelde diameter per groep die aan het volgende voldoet.
>
> We zien per object zien hoeveel satellieten hieraan voldoen, daarnaast in combinatie met hetzelfde object en afstand per 100tal.
>
> Alsook een algemeen overzicht.
>
> Sorteer van voor naar achter.

_Ge kunt dit relatief kort doen door rollup te gebruikenn ik geef u de keuze hier xxx_

```sql
SELECT objectnaam, ROUND(afstand/100),satellietvan, COUNT(*) AS count, AVG(diameter)
FROM hemelobjecten
WHERE diameter > 1000
GROUP BY objectnaam
UNION

SELECT null AS objectnaam, null AS COUNT,satellietvan, COUNT(*) AS count, AVG(diameter)
FROM hemelobjecten
WHERE diameter > 1000
GROUP BY satellietvan
UNION

SELECT null AS objectnaam, null as COUNT,null AS satellietvan, COUNT(*) AS count, AVG(diameter)
FROM hemelobjecten
WHERE diameter > 1000
ORDER BY 3,1,4

--- MET ROLLUP

SELECT objectnaam, ROUND(afstand/100), satellietvan, COUNT(*) AS count, AVG(diameter)
FROM hemelobjecten
WHERE diameter > 1000
GROUP BY ROLLUP ((satellietvan),(objectnaam,afstand))
ORDER BY 3,1,4
```

# Subq 1 Verbreding

## 1.

> Geef een lijst van alle hemelobjecten die meer keer bezocht gaan worden dan Jupiter (onafhankelijk van het aantal deelnames).
>
> Sorteer op objectnaam, diameter.

```sql
SELECT b.objectnaam, diameter FROM hemelobjecten
INNER JOIN bezoeken AS b USING(objectnaam)
GROUP BY 1,2
HAVING COUNT(b.objectnaam) > (SELECT COUNT(*) FROM bezoeken WHERE bezoeken.objectnaam LIKE 'Jupiter')
ORDER BY b.objectnaam
```

## 2.

> Geef het hemellichaam dat het laatst bezocht is.
>
> Gebruik hiervoor de laatste vertrekdatum van de reis en laatste volgnummer van bezoek. Tip: gebruik hiervoor een rij-subquery.
>
> Gebruik geen limit of top.

```sql
SELECT objectnaam FROM bezoeken
WHERE (reisnr, volgnr) IN
(
	SELECT reisnr, MAX(volgnr) FROM bezoeken 
		WHERE reisnr = 
 			(SELECT reisnr FROM reizen 
		 		WHERE vertrekdatum =
		 	(SELECT MAX(vertrekdatum) AS vertrekdatum FROM reizen))
 	GROUP BY 1
)
```

## 3.

> Geef het reisnr, de prijs en vertrekdatum van de reis met de hoogste gemiddelde verblijfsduur op een hemelobject (=som van de verblijfsduur / aantal bezoeken per reis).

```sql
SELECT reisnr ,prijs ,vertrekdatum
	FROM ( SELECT reisnr ,SUM (verblijfsduur) / COUNT (volgnr) AS gem_duur FROM bezoeken GROUP BY 1 ) AS gemiddelde_duur 
	INNER JOIN reizen USING (reisnr) 
	WHERE gem_duur = ( 
		SELECT MAX (gem_duur) 
		FROM ( SELECT reisnr ,SUM (verblijfsduur) / COUNT (volgnr) AS gem_duur 
			  FROM bezoeken GROUP BY reisnr ) AS gemiddelde_duur )
```

## 4.

> Geef de planeet (draait dus rond de zon) met de meeste satellieten.
>
> Sorteer op objectnaam.

```sql
SELECT objectnaam 
FROM (
    SELECT h.satellietvan AS objectnaam, COUNT(h.objectnaam) AS aantal_manen
    FROM hemelobjecten AS h
    WHERE h.satellietvan <> 'Zon'
    GROUP BY h.satellietvan
        ) AS aantal_manen
WHERE aantal_manen = (
    SELECT MAX(aantal_manen) 
    FROM    (
        SELECT h.satellietvan AS objectnaam, COUNT(h.objectnaam) AS aantal_manen
        FROM hemelobjecten AS h
        WHERE h.satellietvan <> 'Zon'
        GROUP BY h.satellietvan
        ) AS aantal_manen
    )
ORDER BY objectnaam
```

## 5. 

```sql
SELECT objectnaam, aantalkleiner FROM 
(SELECT h1.objectnaam, COUNT(h2.objectnaam) AS aantalkleiner
	FROM hemelobjecten AS h1
	INNER JOIN hemelobjecten AS h2 ON h1.diameter > h2.diameter GROUP BY 1) AS h
	WHERE aantalkleiner = 1
```

# Subq 2 verbreding

## 1.

> Geef de diameter van het grootste hemellichaam dat bezocht is op de vroegste reis waar klantnr 126 niet op meegegaan is.

```sql
SELECT MAX(diameter) AS grootste FROM hemelobjecten AS h1 
INNER JOIN bezoeken USING(objectnaam)
INNER JOIN reizen USING(reisnr)
WHERE vertrekdatum = 
(
	SELECT MIN(vertrekdatum) FROM reizen
	WHERE reisnr NOT IN (
		SELECT klantnr FROM klanten WHERE klantnr = 126
	)
)
```

## 2.

> Geef de deelnemers waarbij hun aantal reizen die ze ondernemen groter is dan alle hemelobjecten (die niet beginnen met de letter 'M') hun aantal keren dat ze bezocht zijn.
>
> Of anders geformuleerd:
>
> Geef de deelnemers met meer deelnames dan het grootste aantal bezoeken aan een hemelobject dat niet met de letter 'M' begint (:deze deelnemer meer deelnames heeft dan de "grootste" .. = deze deelnemer heeft meer deelnames dan "alle" ..)
>
> Sorteer op klantnr.

```sql
SELECT klantnr, vnaam, naam, COUNT(reisnr) aantaldeelnames
FROM klanten INNER JOIN deelnames USING(klantnr)
GROUP BY klantnr, vnaam, naam
HAVING COUNT(reisnr) > ALL (
        SELECT COUNT(*)
        FROM bezoeken
        WHERE NOT EXISTS (
                SELECT objectnaam
                FROM hemelobjecten
                WHERE objectnaam LIKE 'M%'
                AND bezoeken.objectnaam = hemelobjecten.objectnaam
        )
        GROUP BY objectnaam
)
```

## 3.

> Geef alle niet-bezochte hemelobjecten, buiten het grootste hemellichaam.
>
> Sorteer op diameter en objectnaam.

```sql
SELECT objectnaam, afstand, diameter
FROM hemelobjecten
WHERE NOT EXISTS (
        SELECT reisnr
        FROM bezoeken b
        WHERE b.objectnaam = hemelobjecten.objectnaam
)
AND diameter < ANY (
        SELECT max(diameter)
        FROM hemelobjecten
)
ORDER BY diameter, objectnaam
```

## 4. 

> Maak een lijst van klanten die meer dan 2 keer een reis gemaakt hebben waarbij er geen bezoek was aan Jupiter.

```sql
SELECT klantnr, naam || ' ' || vnaam as klantnaam, COUNT(reisnr) aantalreizen
FROM deelnames INNER JOIN klanten USING(klantnr)
WHERE NOT EXISTS (
        SELECT reisnr
        FROM bezoeken
        WHERE objectnaam = 'Jupiter'
        AND deelnames.reisnr = bezoeken.reisnr
)
GROUP BY klantnr, klantnaam
HAVING COUNT(reisnr) > 2
```

## 5.

> Geef de klantnr voor de klant met het meeste bezoeken aan de maan. Geef ook het aantal bezoeken.
>
> Gebruik geen limit of top.

```sql
SELECT klantnr, COUNT(aantalbezoeken) AS count
FROM deelnames JOIN (
        SELECT reisnr, volgnr
        FROM bezoeken
        WHERE objectnaam = 'Maan'
        GROUP BY reisnr, volgnr) AS aantalbezoeken USING (reisnr)
GROUP BY klantnr
HAVING COUNT(reisnr) > 3
```

# Optim

_Little sidenote here: ik heb al de oefeningen wel juist, maar die cost is altijd wel redelijk hoog, moest ge dus een betere oplossing vinden, vergeet zeker dan geen pull request te doen!_

## 1.

> Geef een lijst met het spelersnummer en de naam van de speler die in Rijswijk wonen en die in 1980 een boete gekregen hebben van 25 euro. 
>
> Sorteer van voor naar achter.
>
> Probeer gelijk of beter te doen dan "(cost=2.37..2.37 rows=1 width=68)".

```sql
SELECT spelersnr, naam FROM spelers
WHERE spelersnr IN (SELECT spelersnr FROM boetes WHERE bedrag = 25 AND EXTRACT(YEAR FROM datum) = '1980')
AND plaats LIKE 'Rijswijk'
```


## 2.

> Geef de naam en het spelersnummer van de spelers die ooit penningmeester geweest zijn van de club, die bovendien ooit een boete betaald hebben van meer dan 75 euro, en die ooit een wedstrijd gewonnen hebben met meer dan 2 sets verschil. 
>
> Sorteer van voor naar achter. 
>
> Probeer gelijk of beter te doen dan "Unique (cost=100.38..100.54 rows=21 width=68)".

```sql
SELECT naam, spelersnr FROM spelers
INNER JOIN boetes USING(spelersnr)
INNER JOIN bestuursleden USING(spelersnr)
WHERE bedrag > 75 AND EXTRACT(YEAR FROM datum) = 1980 AND functie = 'Penningmeester'
ORDER BY 1,2
```

## 3.

> Geef van elke speler het spelersnr, de naam en het verschil tussen zijn of haar jaar van toetreding en het gemiddeld jaar van toetreding. Sorteer van voor naar achter. 
>
> Probeer gelijk of beter te doen dan "Sort (cost=33.16..33.66 rows=200 width=86)"

```sql
SELECT spelersnr, naam, voorletters, 
(jaartoe - (SELECT avg(jaartoe) FROM spelers)) AS verschil
FROM spelers
ORDER BY 1,2,3,4
```

## 4.

> Je kan per speler berekenen hoeveel boetes die speler heeft gehad en wat het totaalbedrag per speler is. Pas nu deze querie aan zodat per verschillend aantal boetes wordt getoond hoe vaak dit aantal boetes voorkwam. Sorteer van voor naar achter. 
>
> Probeer gelijk of beter te doen dan "Sort (cost=46.39..46.89 rows=200 width=8)".

```sql
SELECT b1.aantalboetes AS a, COUNT(spelersnr)
FROM (SELECT spelersnr, COUNT(*) AS aantalboetes FROM boetes GROUP BY spelersnr) AS b1
GROUP BY b1.aantalboetes
ORDER BY 1,2
```

## 5.

> Geef van alle spelers het verschil tussen het jaar van toetreding en het geboortejaar, maar geef alleen die spelers waarvan dat verschil groter is dan 20.
>
> Sorteer van voor naar achter. 
>
> Probeer zo goed of beter te doen dan "Sort (cost=17.20..17.37 rows=67 width=90)"

```sql
SELECT spelersnr, naam, voorletters, toetredingsleeftijd
FROM (SELECT spelersnr, naam, voorletters, jaartoe - EXTRACT(YEAR FROM geb_datum) AS toetredingsleeftijd FROM spelers) 
AS s WHERE toetredingsleeftijd > 20
ORDER BY 1,2,3,4
```

## 6.

> Geef alle spelers die alfabetisch (dus naam en voorletters, in deze volgorde) voor speler 8 staan. 
>
> Sorteer van voor naar achter.
>
> Probeer zo goed of beter te doen dan "Sort (cost=24.31..24.47 rows=67 width=88)"

```sql
SELECT spelersnr, naam, voorletters, geb_datum
FROM spelers
WHERE naam < (SELECT naam FROM spelers WHERE spelersnr = 8)
ORDER BY 1,2,3,4
```

# Joins Extra

## 1.

> Geef de volledige frequentietabel voor de diameters van de hemelobjecten (frequentie: hoeveel ojecten zijn er met de gegeven diameter, cumulatieve Frequentie, relatieve frequentie, Relatieve cumulatieve frequentie).
>
> Let op de datatypes en de precisie, gebruik CAST, rond niet af. NUMERIC(5,2)) 
> 
> Sorteer op diameter en verder op de volgende kolommen.

_Holy fuck, what is this shit_

```sql
SELECT DISTINCT k.klantnr, k.naam || ' ' || k.vnaam AS naam, 
	(SELECT sum(reizen.prijs) FROM klanten
	LEFT OUTER JOIN deelnames ON (klanten.klantnr = deelnames.klantnr)
	INNER JOIN reizen ON (reizen.reisnr = deelnames.reisnr)
	WHERE k.klantnr = klanten.klantnr) AS tot_bedrag,
	
	sum(h.afstand) AS tot_afstand,
	
	CASE WHEN (SELECT sum(reizen.prijs) FROM klanten
                LEFT OUTER JOIN deelnames ON (klanten.klantnr = deelnames.klantnr)
				INNER JOIN reizen ON (reizen.reisnr = deelnames.reisnr)
				WHERE k.klantnr = klanten.klantnr) = 0
				
				OR
				
				sum(h.afstand) = 0
		THEN 'veel geld voor niks of niet op reis geweest'
		ELSE
				
		((SELECT sum(reizen.prijs) FROM klanten
		LEFT OUTER JOIN deelnames ON (klanten.klantnr = deelnames.klantnr)
		INNER JOIN reizen ON (reizen.reisnr = deelnames.reisnr)
		WHERE k.klantnr = klanten.klantnr) / sum(h.afstand))::varchar 
		END
		
		AS prijs_per_kilometer,
	
	(SELECT max(reizen.vertrekdatum) FROM reizen
	 INNER JOIN deelnames ON (deelnames.reisnr = reizen.reisnr)
	 WHERE deelnames.klantnr = k.klantnr) AS laatste_reis_datum
	
FROM klanten k
LEFT OUTER JOIN deelnames d ON (k.klantnr = d.klantnr)
LEFT OUTER JOIN reizen r ON (r.reisnr = d.reisnr)
LEFT OUTER JOIN bezoeken b ON (b.reisnr = r.reisnr)
LEFT OUTER JOIN hemelobjecten h ON (h.objectnaam = b.objectnaam)

GROUP BY 1,2, 3
ORDER BY 1,2,3,4,5
```

## 2.

> Geef de volledige frequentietabel voor de diameters van de hemelobjecten (frequentie: hoeveel ojecten zijn er met de gegeven diameter, cumulatieve Frequentie, relatieve frequentie, Relatieve cumulatieve frequentie).
>
> Let op de datatypes en de precisie, gebruik CAST, rond niet af. 
>
> Sorteer op diameter.

```sql
SELECT diameter, 
       COUNT(diameter) AS f, SUM(COUNT(diameter)) OVER w1 AS cf,
       to_char(float8 (COUNT(diameter)*100::float / (SELECT COUNT(diameter) FROM hemelobjecten)), 'FM99.00') AS rf,
       to_char( float8 (SUM(COUNT(diameter)) OVER(ORDER BY diameter)*100::float / (SELECT COUNT(diameter)  FROM hemelobjecten)),
       'FM990.90')  AS crf
FROM hemelobjecten
GROUP BY diameter
WINDOW w1 AS ( ORDER BY diameter)
ORDER BY 1
```

## 3. 

> Geef voor elke reis het aantal klanten waarvan de naam niet met een 'G' begint en waarvan de periode van de geboortedatum van de klant tot de vertrekdatum van de reis overlapt met de huidige datum en 50 jaar verder (gebruik hiervoor de gepaste operator: OVERLAPS). 
>
> Indien er op de reis hemelobjecten worden bezocht waarvan de tweede letter van het hemelobject voorkomt in de naam van het hemelobject waarvan dit bezocht hemelobject een satelliet is, dan wordt deze reis genegeerd. 
>
> Sorteer op reisnr.

_Thanks to Kiana_

```sql
SELECT r.reisnr, COUNT(klantnr)
FROM deelnames d INNER JOIN klanten USING(klantnr) RIGHT OUTER JOIN reizen r ON d.reisnr = r.reisnr
AND naam NOT LIKE 'G%' AND (geboortedatum, vertrekdatum) OVERLAPS (NOW(), INTERVAL '50' YEAR)
WHERE r.reisnr NOT IN (
    SELECT reisnr
    FROM hemelobjecten NATURAL INNER JOIN bezoeken
    WHERE satellietvan LIKE '%' || SUBSTR(objectnaam, 2, 1) || '%'
)
GROUP BY r.reisnr
ORDER BY 1, 2
```

# Vensters

## 1.

> Geef per reis incrementeel de totale verblijfsduur volgens de volgorde waarin de hemelobjecten bezocht worden. 
>
> Geef ook de totale verblijfsduur van alle reizen om alles in perspectief te zettten.
>
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
> 
> Geef daarnaast de totale reisduur per jaar incrementeel in de tijd (hier genaamd jaar_duur).
>
> (Dus elk jaar begint aan een nieuwe som)
>
> Sorteer op reisnr en de andere kolommen.

```sql
SELECT reisnr, LAG(reisnr) OVER w1 AS vorig_reisnr, vertrekdatum, vertrekdatum - LAG(vertrekdatum) OVER w1 
AS tussen_tijd,
reisduur, EXTRACT(YEAR FROM vertrekdatum) AS jaar, SUM(reisduur) OVER w2 AS jaar_duur
FROM reizen
WINDOW w1 AS 
    (ORDER BY vertrekdatum), w2 AS (PARTITION BY EXTRACT(YEAR FROM vertrekdatum) ORDER BY vertrekdatum)
ORDER BY 1
```

# Extra 4

## 1.

> Maak een overzicht waarbij je voor de Maan en voor Mars aangeeft hoeveel ruimtereizen één of meer keer de betreffende bestemming bezocht hebben (d.w.z. erop geland zijn).
>
> Sorteer van voor naar achter.

```sql
SELECT b.objectnaam, COUNT(DISTINCT(reisnr)) FROM bezoeken AS b 
INNER JOIN hemelobjecten AS h USING(objectnaam)
WHERE h.objectnaam = 'Mars' OR h.objectnaam = 'Maan'
GROUP BY 1
```

## 2.

> Maak een lijst van de mensen die Mars wel bezocht hebben maar Io nog niet. 
>
> Sorteer van voor naar achter.

_Ik weet niet of dit de efficienste manier is om dit te doen_

```sql
SELECT klantnr, naam FROM klanten AS klant
WHERE klant.klantnr NOT IN 

(
	SELECT klanten.klantnr FROM klanten 
	INNER JOIN deelnames d ON (klanten.klantnr = d.klantnr)
	INNER JOIN reizen r ON (d.reisnr = r.reisnr)
	INNER JOIN bezoeken b ON (r.reisnr = b.reisnr)
	INNER JOIN hemelobjecten h ON (h.objectnaam = b.objectnaam) 
	WHERE h.objectnaam LIKE '%Io%'
)
AND klant.klantnr IN 
(
	SELECT klanten.klantnr
	FROM klanten
	INNER JOIN deelnames d ON (klanten.klantnr = d.klantnr)
	INNER JOIN reizen r ON (d.reisnr = r.reisnr)
	INNER JOIN bezoeken b ON (r.reisnr = b.reisnr)
	INNER JOIN hemelobjecten h ON (h.objectnaam = b.objectnaam)
	WHERE h.objectnaam = 'Mars'
)
```

## 3.

> Maak een overzicht waarbij je voor de Maan en voor Mars aangeeft hoeveel verschillende ruimtereizen er geweest zijn(d.w.z. erop geland zijn is voldoende). 
>
> En enkel indien er meer dan 1 reis geweest is
>
> Sorteer van voor naar achter

```sql
SELECT b.objectnaam, COUNT(DISTINCT(reisnr)) FROM bezoeken AS b 
INNER JOIN hemelobjecten AS h USING(objectnaam)
WHERE h.objectnaam = 'Mars' OR h.objectnaam = 'Maan'
GROUP BY 1
```

## 4. 

> Maak een lijst met die mensen die meer dan 2 maal een reis ondernomen hebben waarin men geen enkele satelliet van Jupiter bezoekt !.
>
> Sorteer van voor naar achter.

```sql
SELECT d.klantnr ,k.naam || k.vnaam  AS "volledige naam", count(d.reisnr) 
AS "aantal ondernomen reizen"
FROM deelnames d 

INNER JOIN klanten k ON (d.klantnr = k.klantnr)
WHERE d.reisnr 
NOT IN 
(
    SELECT r.reisnr FROM reizen r 
	INNER JOIN bezoeken b ON(r.reisnr = b.reisnr)
	
	WHERE objectnaam IN  
	(
		SELECT objectnaam
		FROM hemelobjecten
		WHERE satellietvan = 'Jupiter'
	)
)
GROUP BY d.klantnr, k.naam, k.vnaam
HAVING count(d.reisnr)>2
```

# Venster XML 

## 1. 

> Geef alle klanten waarbij de voorlaatste letter van de naam 1 van de letters uit het woord 'azerty' is.
>
> Gebruik geen OR operator, maar een andere ISO sql operator voor het vergelijken van patronen, sorteer van voor naar achter.

```sql
SELECT klantnr, naam, vnaam, geboortedatum
FROM klanten
WHERE naam SIMILAR TO '%(a|z|e|r|t|y)_'
ORDER BY 1,2,3,4
```

## 2.

> Geef voor elke een klant een overzicht aan uitgaven. Hoe? Geef voor elke klant een cumulatie overzicht van prijs van de reizen waar hij aan deelgenomen heeft. 
>
> De volgorde, waarin er wordt cumulatief wordt opgeteld, wordt bepaald door de vertrekdatum van de reis.
>
> Sorteer van voor naar achter.


```sql
SELECT k.klantnr, r.reisnr, sum(r.prijs)
OVER(PARTITION BY
k.klantnr
	 ORDER BY r.vertrekdatum ROWS BETWEEN UNBOUNDED
PRECEDING AND current ROW
	) 
FROM klanten k
LEFT OUTER JOIN deelnames d ON(d.klantnr = k.klantnr)
INNER JOIN reizen r ON(r.reisnr = d.reisnr)

ORDER BY 1,2,3
```

## 3.

> Geef enkel de 2 voorlaatste reizen terug.
>
> De positie van reizen wordt bepaalt door het reisnummer.
>
> Ter vergelijking, als je de getallen 1 t/m 10 neemt,
dan is dit 8 en 9. 
>
> Sorteer van voor naar achter.
>
> Gebruik enkel ISO sql.

```sql
SELECT *
FROM (SELECT reisnr, vertrekdatum, reisduur, prijs
	FROM reizen
	ORDER BY reisnr DESC
	offset 1 ROW
	FETCH FIRST 2 ROWS only ) t
ORDER BY reisnr
```

# JSON

## 1.

> Gebruik JSON instructies.
>
> Genereer startende van een integer array de volgende output: [[1,5],[99,100]]

```sql
SELECT array_to_json('{{1,5},{99,100}}'::int[])
```

## 2. 

> Gebruik JSON instructies.
>
> Selecteer in onderstaande json string '{"a":[1,2,3],"b":[4,5,6]}' het tweede object

```sql
SELECT '{"a": [1,2,3], "b": [4,5,6]}'::json ->'b'
```

## 3.

> Gebruik JSON instructies. 
>
> Expandeer het buitenste JSON object uit de string '{"a":"foo", "b":"bar"}'.

```sql
SELECT * FROM json_each('{"a": "foo", "b": "bar"}')
```

## 4.

> Gebruik JSON instructies.
>
> Selecteer in onderstaande json string '{"1":[1,2,3],"2":[4,5,6]}' het tweede object

```sql
SELECT '{"1":[1,2,3],"2":[4,5,6]}'::json->'2'
```

## 5.

> Gebruik JSON instructies. 
>
> Selecteer in onderstaande json string '[{"1":"2"},{"4":"5"}]' het eerste object

```sql
SELECT '[{"1":"2"},{"4":"5"}]'::json -> 0
```

## 6.

> Gebruik JSON instructies.
>
> Selecteer in onderstaande json string '[{"1":"2"},{"4":"5"}]' het tweede object

    
```sql
SELECT '[{"1":"2"},{"4":"5"}]'::json -> 1
```

## 7.

> Gebruik JSON instructies.
>
> Selecteer uit onderstaande json string '{"a": {"b":{"c": "foo"}, "c":{"5":"6"}}}' het element "6"

```sql
SELECT '{"a": {"b":{"c": "foo"}, "c":{"5":"6"}}}'::json->'a'->'c'->'5'
```


![](https://nils-braun.github.io/assets/images/2020-11-14-dask-sql/sql-meme.jpg)