# Examen 2

Note de cours examen 2 méga donnée

## Table des matière


<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>



## Match de données
```sql
MATCH (e:Employee)-[:SOLD]->(o:Order)-[c:CONTAINS]->(p:Product) RETURN e.firstName, e.lastName, o.orderID, o.orderDate, SUM(c.unitPrice * c.quantity) ORDER BY SUM(c.unitPrice * c.quantity) DESC
```

## Suppression de données
```sql
MATCH (n:Employee) WHERE n.employeeID = '21032190' DETACH DELETE n
```

## Ajout de données
```sql
MATCH (e:Employee) WHERE e.employeeID = "21032190"
MATCH (p1:Product) WHERE p1.productID = "20"
MATCH (p2:Product) WHERE p2.productID = "18"
MATCH (p3:Product) WHERE p3.productID = "24"
MATCH (p4:Product) WHERE p4.productID = "22"
MATCH (p5:Product) WHERE p5.productID = "19"
CREATE (e)-[:SOLD]->(o1:Order {orderID: '99999999', orderDate: '1997-12-01', shipName: "TEST"})
CREATE (e)-[:SOLD]->(o2:Order {orderID: '11111111', orderDate: '1997-12-01', shipName: "TEST2"})
CREATE (o1)-[:CONTAINS {quantity: 15, unitPrice: p1.unitPrice}]->(p1)
CREATE (o1)-[:CONTAINS {quantity: 20, unitPrice: p2.unitPrice}]->(p2)
CREATE (o1)-[:CONTAINS {quantity: 15, unitPrice: p3.unitPrice}]->(p3)
CREATE (o2)-[:CONTAINS {quantity: 20, unitPrice: p4.unitPrice}]->(p4)
CREATE (o2)-[:CONTAINS {quantity: 23, unitPrice: p5.unitPrice}]->(p5)
RETURN e

```

## Type de retour

### Tableau de données
```sql
MATCH (e:Employee)-[:SOLD]->(o:Order)-[c:CONTAINS]->(p:Product) RETURN e.firstName, e.lastName
```

### Sous forme graph
```sql
MATCH (e:Employee)-[:SOLD]->(o:Order)-[c:CONTAINS]->(p:Product) RETURN e
```


### Tous retourner
```sql
MATCH (e:Employee)-[:SOLD]->(o:Order)-[c:CONTAINS]->(p:Product) RETURN *
```

## Astuce

### Afficher la structure
```sql
CALL db.schema.vizualisation()
```

### Jouer avec les dates
```sql
// Exercice 11
// Liste des visites ayant eu lieu a une en mai 2020
MATCH (v:Visit) where date({year: 2020, month: 5}) < date(v.starttime) < date({year: 2020, month: 6}) return v;

// Exercice 12
// durée moyenne des visites par visites
MATCH (n:Visit)--(p:Place) RETURN p.name, avg(n.duration.hours)
```

