## SPARQL queries

---
### Prefix of the ontology
```txt
PREFIX : <http://www.semanticweb.org/aless/ontologies/2026/4/transactionrisk#>
```

---
### Retrieve all transactions

Returns all individuals that are instances of `Transaction` or any subclass of `Transaction`.

```txt 
SELECT ?x 
WHERE {
  ?x rdf:type :Transaction .
}

```

---

### Transactions issued to merchants based in South Africa

Returns all transactions issued to merchants located in South Africa.

```txt
SELECT ?x 
WHERE {
  ?x :issuedTo ?y .
  ?y :basedIn :South_Africa.
}
```

---

### Transactions made with European cards

Returns transactions made with cards belonging to banks located in Europe.

```txt
SELECT ?x 
WHERE {
  ?x :madeWith ?y .
  ?y :belongTo ?w .
  ?w :basedIn ?z .
  ?z :isPartOfContinent :Europe
}
```

---

### Transactions issued to Italian merchants with merchant risk

Returns all transactions issued to merchants located in Italy together with their merchant risk score.

```txt
SELECT ?x ?mr
WHERE {
  ?x :issuedTo ?y .
  ?y :basedIn :Italy.
  ?y :merchantRisk ?mr
}
```
---
### Transactions that don't violate the time conflict rule

Set time risk equals to `0` to all the transaction that don't violate the time conflict rule

```txt
INSERT {
  ?t2 :timeRisk 0 .
}
WHERE {
  ?t2 rdf:type/rdfs:subClassOf* :Transaction .
  
  FILTER NOT EXISTS {
    ?t1 rdf:type/rdfs:subClassOf* :Transaction .
    FILTER(?t1 != ?t2) 
    
    ?t1 :madeWith ?card .
    ?t2 :madeWith ?card .
    
    ?t1 :issuedTo ?m1 .
    ?t2 :issuedTo ?m2 .
    
    ?m1 :store_type ?st1 .
    ?m2 :store_type ?st2 .
    FILTER(STR(?st1) = STR(?st2))
    FILTER(STR(?st1) = "physical")
    
    ?m1 :basedIn ?c1 .
    ?m2 :basedIn ?c2 .
    
    ?c1 :name ?n1 .
    ?c2 :name ?n2 .
    FILTER(STR(?n1) != STR(?n2))
    
    ?t1 :timestamp ?time1 .
    ?t2 :timestamp ?time2 .
    
    BIND((?time2 - ?time1) AS ?diff)
    FILTER(?diff >= 0 && ?diff <= 300) 
  }
}
```

---

### Retrieve fraudulent transactions

Returns transactions classified as `DeclinedTransaction`.

```txt
SELECT ?transaction ?type
WHERE {
  VALUES ?type { :DeclinedTransaction }
  ?transaction rdf:type/rdfs:subClassOf* ?type .
}
```

---


### Insert BTransaction classification

Creates a new classification by assigning `BTransaction` to all transactions that are not already classified as `ATransaction`.

```txt
INSERT {
    ?transaction rdf:type :BTransaction .
}
WHERE {
    ?transaction rdf:type :Transaction .
    FILTER NOT EXISTS {
        ?transaction rdf:type/rdfs:subClassOf* :ATransaction .
    }
}
```

---

### Retrieve ATransaction and BTransaction instances

Returns all transactions classified as either `ATransaction` or `BTransaction`.

```txt
SELECT ?transaction ?type
WHERE {
  VALUES ?type { :ATransaction :BTransaction }
  ?transaction rdf:type/rdfs:subClassOf* ?type .
}
```