# Javazone-cheat sheet

# jz_nta

## Fundamentals

Store any kind of data using the following graph concepts:

* **Node**: Graph data records
* **Relationship**: Connect nodes (has direction and a type)
* **Property**: Stores data in key-value pair in nodes and relationships
* **Label**: Groups nodes and relationships (optional)

---
# Datamodell
## Person

```
Person(
foedselsdato:	date
foedselsnummer:	String
kjoenn	{MANN|KVINNE}
navn	String
personstatus	{BOSATT|MIDLERTIDIG|DOED|UTFLYTTET}
postnummer: String

)

FAMILIERELASJON {
ergjeldende: boolean
gyldighetsdato: date
rolle: {BARN|EKTEFELLE_ELLER_PARTNER|MOR|FAR|MEDMOR}
}

SIVILSTAND {
ergjeldende: boolean
gyldighetsdato: date
myndighet: String
sivilstand: {UGIFT|GIFT|SEPARERT|SKILT|UOPPGITT}
}

FORELDREANSVAR {
ergjeldende: boolean
gyldighetsdato: date
ansvarstype: {FELLES|MOR|FAR|MEDMOR|UKJENT}
}
```

# Spørringer


### Hent samtlige personer (Match node)

```cypher
MATCH (p:Person)
WHERE p.navn = "Effektiv Mann"
RETURN p;
```

### Hent person med spesifikt fødselsnummer
```cypher
match(n:Person{foedselsnummer:'07841497982'}) return n
```

### Hent person med navn

```cypher
MATCH (p:Person)
WHERE p.navn = "Effektiv Mann"
RETURN p;
```
eller
```cypher
MATCH (p:Person {navn= 'Effektiv Mann'})
RETURN p;
```

### Hent person med flere enn 10 familierelasjoner

```cypher
MATCH (p:Person)-[r:FAMILIERELASJON]-(:Person)
WITH p, count(r) as relasjoner
WHERE relasjoner > 10
RETURN p,relasjoner
LIMIT 1
```

### Hent person som har vært gift mer enn en gang (må være bosatt). 

```cypher
MATCH (p:Person {personstatus: 'BOSATT'})-[r:SIVILSTAND]-(:Person {personstatus: 'BOSATT'})
WITH p, count(r) as relasjoner
WHERE relasjoner > 2
RETURN p, relasjoner
LIMIT 1
```
#### Bytt ut person 2 med DOED for å finne enke/enkemann

```cypher
MATCH (p:Person {personstatus: 'BOSATT'})-[r:SIVILSTAND]-(:Person {personstatus: 'DOED'})
WITH p, count(r) as relasjoner
WHERE relasjoner > 2
RETURN p, relasjoner
LIMIT 1
```
### Antall personer i relasjonen mellom `03912849065` og `03912949833`
```cypher
MATCH (p:Person {foedselsnummer: '03912849065'}),
(q:Person {foedselsnummer: '03912949833'}),
r = shortestPath((p)-[*]-(q))
WHERE length(r) > 1
RETURN r
```

### Finn personer med inkonsistente relasjoner

```cypher
MATCH (p:Person)
where size((p)<-[:FAMILIERELASJON]-(:Person)) <> size((p)-[:FAMILIERELASJON]->(:Person))
return p limit 5
```

```
MATCH (p:Person {personstatus: 'BOSATT'})-[r:FORELDREANSVAR {ergjeldende: TRUE}]-(:Person {personstatus: 'BOSATT'})
WITH p, count(r) as relasjoner
WHERE relasjoner > 1
RETURN p, relasjoner
LIMIT 1
```

# Import av data

## Persondata
```
LOAD CSV WITH HEADERS FROM "file:///personer.csv" AS row
create(p:Person {foedselsnummer: row.fnummer, personstatus:row.personstatus,kjoenn:row.kjoenn,foedselsdato:date(row.foedselsdato),sivilstand:row.sivilstand,navn:row.fornavn + " " + row.etternavn, postnummer:row.postnummer})
```

## Familierelasjoner
```
LOAD CSV WITH HEADERS FROM "file:///relasjoner.csv" AS row
match (p:Person {foedselsnummer: row.fnummer1}),(q:Person {foedselsnummer: row.fnummer2})
create (p)-[:FAMILIERELASJON {rolle: row.fnummer1_rolle,ergjeldende:toBoolean(row.ergjeldende),gyldighetstidspunkt:date(row.gyldighetsdato)}]->(q)
```

## Foreldreansvar
```
LOAD CSV WITH HEADERS FROM "file:///foreldreansvar.csv" AS row
match (p:Person {foedselsnummer: row.ansvarlig}),(q:Person {foedselsnummer: row.ansvarssubjekt})
create (p)-[:FORELDREANSVAR {ansvarstype: row.ansvarstype,ergjeldende:toBoolean(row.ergjeldende),gyldighetstidspunkt:date(row.gyldighetsdato)}]->(q)
```

## Sivilstand
```
LOAD CSV WITH HEADERS FROM "file:///sivilstand.csv" AS row
match (p:Person {foedselsnummer: row.hovedperson}),(q:Person {foedselsnummer: row.ektefelleEllerPartner})
create (p)-[:SIVILSTAND {sivilstand: row.sivilstand,ergjeldende:toBoolean(row.erGjeldende),sivilstandsdato:date(row.sivilstandsdato),myndighet:row.myndighet}]->(q)
```
## Avvik
```
create(p:Person {foedselsnummer:'01818254321',foedselsdato:date('1982-01-01'), personstatus:'BOSATT', navn:'Levende Ektefelle', kjoenn:'KVINNE',postnummer:'0772'})

create(p:Person {foedselsnummer:'01918054391',foedselsdato:date('1980-11-01'), personstatus:'DOED', navn:'Død Ektefelle', kjoenn:'MANN',postnummer:'0772'})

match(d:Person{foedselsnummer:'01918054391'}),(l:Person{foedselsnummer: '01818254321'})
create (d)-[:SIVILSTAND{sivilstand: 'GIFT', ergjeldende: TRUE, gyldighetstidspunkt:date('2010-01-01')}]->(l)

match(d:Person{foedselsnummer:'01818254321'}),(l:Person{foedselsnummer: '01918054391'})
create (d)-[:SIVILSTAND{sivilstand: 'GIFT', ergjeldende: TRUE, gyldighetstidspunkt:date('2010-01-01')}]->(l)
```

## Finne dette
```
MATCH (p:Person {personstatus: 'BOSATT'})-[r1:SIVILSTAND{sivilstand:'GIFT', ergjeldende: TRUE}]-(e:Person {personstatus: 'DOED'})
, (p)-[:FAMILIERELASJON{rolle:'EKTEFELLE', ergjeldende: TRUE}]-(e)
WITH p
RETURN p
```
