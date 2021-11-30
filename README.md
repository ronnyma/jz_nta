# Javazone-cheat sheet

# jz_nta

## Fundamentals

Store any kind of data using the following graph concepts:

* **Node**: Graph data records
* **Relationship**: Connect nodes (has direction and a type)
* **Property**: Stores data in key-value pair in nodes and relationships
* **Label**: Groups nodes and relationships (optional)

---

## Browser editor

### CLI

Examples: `:help` `:clear`

---

# Cypher

## Match

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
