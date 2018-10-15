NoSQL Course MongoDB
====================

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Introduction](#introduction)
- [Présentation de MongoDB](#pr%C3%A9sentation-de-mongodb)
- [Lancement de Docker](#lancement-de-docker)
- [Passons à Mongo](#passons-%C3%A0-mongo)
- [Création d'une base de données](#cr%C3%A9ation-dune-base-de-donn%C3%A9es)
- [Introduction aux collections](#introduction-aux-collections)
  - [Création d'une collection](#cr%C3%A9ation-dune-collection)
  - [Suppression d'une collection](#suppression-dune-collection)
- [Types de données dans MongoDB](#types-de-donn%C3%A9es-dans-mongodb)
- [Opérations CRUD sur les collections](#op%C3%A9rations-crud-sur-les-collections)
  - [Insertion de documents](#insertion-de-documents)
  - [Récupération de documents](#r%C3%A9cup%C3%A9ration-de-documents)
    - [Projections](#projections)
    - [Requêtes avec critères](#requ%C3%AAtes-avec-crit%C3%A8res)
    - [Itération sur le curseur](#it%C3%A9ration-sur-le-curseur)
    - [Modification du comportement du curseur](#modification-du-comportement-du-curseur)
  - [Modification de documents](#modification-de-documents)
    - [Clause query](#clause-query)
    - [Clause update](#clause-update)
      - [`$currentDate`](#currentdate)
      - [`$inc`](#inc)
      - [`$min` et `$max`](#min-et-max)
      - [`$mul`](#mul)
      - [`$rename`](#rename)
      - [`$set` et `$setOnInsert`](#set-et-setoninsert)
      - [`$unset`](#unset)
    - [Clause options](#clause-options)
  - [Suppression de documents](#suppression-de-documents)
- [Tableaux et sous-documents](#tableaux-et-sous-documents)
  - [Opérations avec les tableaux](#op%C3%A9rations-avec-les-tableaux)
  - [Intérêt des sous-documents](#int%C3%A9r%C3%AAt-des-sous-documents)
- [Agrégats](#agr%C3%A9gats)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Introduction
------------

Dans ce TP, nous allons nous familiariser avec l'utilisation de MongoDB, toujours avec Docker comme support pour faire fonctionner notre palteforme.

Présentation de MongoDB
-----------------------

MongoDB est un SGBD NoSQL orienté documents. Contrairement à Redis, il ne permet pas de stockage sous la forme clé-valeur, ici, nous parlerons de collections. Une collection est un regroupement de données. On pourrait assimiler les collections à des tables en SQL si l'on mettait de côté un peu de rigueur.

MongoDB est écrit en C++. Il a été conçu par des anciens de chez DoubleClick, une entreprise de publicité en ligne rachetée par Google, pour un besoin spécifique : la montée en charge.

Lancement de Docker
-------------------

Pour ce nouveau projet, nous allons de nouveau utiliser Docker avec l'utilitaire docker-compose. Notre infrastructure comporte trois conteneurs :
- Un conteneur mongodb en mode base de données,
- Un conteneur mongodb qui nous servira à avoir la ligne de commande,
- Un conteneur Mongo Express, un utilitaire en NodeJS + Express pour visualiser notre base comme on le ferait sur MySQL avec un phpMyAdmin.

Afin de démarrer notre infrastructure, comme pour le TP précédent, nous allons éxécuter la commande suivante :

```
docker-compose up -d
```

Une fois l'infrastructure démarrée, vous pouvez vous pouvez ouvrir la ligne de commande avec la commande suivante :

```
docker-compose run mgcli
```

En fin de séance, n'oubliez pas d'éteindre l'infrastructure avec la commande appropriée :

```
docker-compose down
```

Attention : Cette fois-ci, notre docker est configuré pour faire persister les données ! Si vous corrompez votre base, éteignez votre environement docker, et supprimez le contenu du répertoire `mongo/data`. De même, si vous changez de poste, pensez à copier votre répertoire `mongo/data`.

Passons à Mongo
---------------

Mongo va nous permettre de stocker des données avec une représentation en documents au sein de collections, qui sont stockées dans des bases de données. Ainsi, un serveur mongo va posséder une à plusieurs bases, qui vont posséder une à plusieurs colelctions, qui elles-même vont posséder un ou plusieurs documents. On pourrait définir les équivalences suivantes :

| SGBD Relationnel       | MongoDB                |
|------------------------|------------------------|
| Base de données        | Base de données        |
| Table                  | Collection             |
| Enregistrement (Ligne) | Document               |
| Colone                 | Champ                  |

À tout moment, si vous avez besoin d'afficher l'aide de MongoDB (la liste des commandes disponibles), vous pouvez, dans un shell mongo, éxécuter la commande suivante :

```javascript
db.help();
```

Vous aurez peut-être remarqué que la syntaxe de la ligne de commande est identique à JavaScript. Nous allons en effet manipuler notre base à travers des commandes ressemblant à du JavaScript, et elle nous répondra avec des retours ressemblants à du JSON.

## Création d'une base de données

Contrairement aux SGBD relationnels que vous avez étudié précédement, ils n'y a pas de commande de création de base de données à proprement parler. Pour créer une base, vous devez vous positionner sur celle-ci et y créer au moins un document.

```shell
> use mabase;
switched to db mabase
> db;
mabase
> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
```

Dans l'exemple au dessus, nous avons créé une base `mabase`, nous sommes bien psitionné dessus comme l'indique la seconde commande, mais pourtant, lorsque l'on liste les bases disponibles, elle n'y figure pas.

## Introduction aux collections

Les collections vont faire office de tables dans MongoDB. Contrairement aux tables SQL, les collections n'ont pas de déclaration. Un document peut parfaitement n'avoir aucune colone de commune avec les autres documents de sa collection.

### Création d'une collection

Nous allons créer une collection vide afin d'y stocker des documents ultérieurement. Avant celà, il est important de se positionner sur la base `iut` afin d'y créer notre collection.

```js
use iut;
```

À ce stade, notre base n'est pas encore créée, car elle ne contient pas de collection. Contrairement aux bases de données, qui doivent avoir du contenu pour exister, on peut créer une collection sans qu'elle ne contienne de documents. Nous allons commencer par créer une collection `salles` vide dans notre base `iut`.

```js
db.createCollection('salles');
// Retourne : { "ok" : 1 }
```

Si nous éxécutons maintenant la commande `show dbs`, nous verrons que notre base `iut` est désormais créée. Nous avons également notre collection `salles`de créée.

Il n'est pas nécéssaire de crééer une collection de cette façon, on peut la créer implicitement en y insérant des données :

```js
use iut
// Retourne : switched to db iut
show collections
// Retourne :
// salles
db.profs.insert({nom: "Baumann", prenom: "Maxime", statut: "Vacataire"})
// Retourne : WriteResult({ "nInserted" : 1 })
show collections
// Retourne :
// profs
// salles
```

Ici, en ajoutant un document dans la collection `profs` alors que celle-ci n'existe pas déclenche la création de celle-ci avant d'y insérer le document.

### Suppression d'une collection

Pour supprimer une collection, il faut utiliser la commande `.drop()` précédée par le nom de la collection. Par exemple, pour supprimer notre collection `profs` : 

```js
show collections
// Retourne :
// profs
// salles
db.profs.drop()
// Retourne : true
show collections
// Retourne : 
// salles
```

## Types de données dans MongoDB

Avant de commencer à créer de documents, nous allons jeter un oeil aux différents types de données que MongoDB met à notre disposition pour structurer nos documents :
- Des châines de caractères, elles doivent être valide en UTF-8,
- Des entiers, sur 32 ou 64 bits (`int` ou `long`),
- Des booléens,
- Des nombres décimaux 64 ou 128 bits (`double` ou `decimal`),
- Des tableaux,
- Des objets,
- Des dates,
- Des timestamps,
- Du binaire,
- Du JavaScript,
- Des expressions régulières,
- Des valeurs nulles (`null`).

Ceci implique que nos documents peuvent contenir des sous-documents, des tableaux, etc, et ça implique aussi de repenser la structuration de nos données par rapport à un SGBD relationnel.

## Opérations CRUD sur les collections

Mongo nous permet de faire des opérations CRUD (Create, Read, Update, Delete). Nous allons faires quelques opérations CRUD sur notre collection `salles` de notre base `iut`.

### Insertion de documents

Afin d'insérer un document dans une collection, il faut appeler la commande `insert()` sur cette dernière. Ainsi, pour insérer un document dans une collection `profs`, nous ferions ainsi :

```js
db.profs.insert({nom: "Baumann", prenom: "Maxime", statut: "Vacataire"})
```

On peut faire de l'insertion de multiples documents à l'aide de `insertMany()` ainsi que d'un seul document avec `insertOne()`. `insert()` sert à faire de l'insertion peut importe le nombre (1 ou plus). Dans les cas d'insertion multiple, il faut systématiquement passer un tableau de document comme premier paramètre.

> À vous de jouer !
>
> Dans la collection `salles`, insérez les données suivantes :
>
> | nom | postes | places |
> |-----|-------:|-------:|
> | TP1 | 15     | 28     |
> | TP2 | 16     | 32     |
> | TP3 | 15     | 26     |
> | TD1 | 0      | 30     |
> | TD2 | 0      | 28     |
> | TD3 | 0      | 26     |

### Récupération de documents

Afin de récupérer des documents, nous allons utiliser la commande `find()` sur notre collection. Par exemple, pour récupérer les documents de la collection `salles` que nous venons de créer, il faudrait faire `db.salles.find()`. Notez qu'il existe également une commande `findOne()` qui retourne un seul document correspondant aux critères choisis.

#### Projections

Comme avec un SGBD relationnel, nous pouvons faire une projection des champs qui nous intéresse. Par exemple, si nous souhaitons récupérer uniquement le champ `nom`, il faut passer la projection en second paramètre : `db.salles.find({},{_id: 0, nom: 1})`.

Dans ce dernier exemple, `{_id: 0}` indique que nous ne souhaitons pas récupérer l'id de notre élément dans la projection.

Avec Mongo, les projections supportent quelques opérateurs, majoritairement dédiés aux `array`. Nous les étudierons ultérieurement.

#### Requêtes avec critères

Nous avons la possibilité de faire des requêtes avec des critères, sur un champ, un champ de sous-document (nous verrons celà plus tard), sur des plages de valeurs, etc.

La première chose que nous allons voir sont les critères sur l'égalité. Pour sélectionner un document avec un champ dont la valeur est égale à une valeur, il faut passer cette égalité comme premier paramètre de la commande `find()`.

Par exemple, si nous voulons récupérer les salles qui possèdent `0` `postes` : `db.salles.find({postes:0})`.

On va également pouvoir faire des conditions sur des valeurs plus petites ou plus grandes. Par exemple, si je veux récupérer les salles avec plus de 10 places, mais avec moins que 30 places : `db.salles.find({places: {$gt: 10, $lt: 30}})`.

Il existe d'autres opérateurs similaires à `$gt` ou `$lt`, nous les expérimenterons au fur et à mesure. En voici la liste complète :

| Opérateur                                                                         | Description                                                       |
|-----------------------------------------------------------------------------------|-------------------------------------------------------------------|
| [`$eq`](https://docs.mongodb.com/manual/reference/operator/query/eq/#op._S_eq)    | Match les valeurs égales à une valeur spécifiée                   |
| [`$gt`](https://docs.mongodb.com/manual/reference/operator/query/gt/#op._S_gt)    | Match les valeurs strictement supéreieures à une valeur spécifiée |
| [`$gte`](https://docs.mongodb.com/manual/reference/operator/query/gte/#op._S_gte) | Match les valeurs supérieures ou égales à une valeur spécifiée    |
| [`$in`](https://docs.mongodb.com/manual/reference/operator/query/in/#op._S_in)    | Match les valeurs présentes dans un ensemble                      |
| [`$lt`](https://docs.mongodb.com/manual/reference/operator/query/lt/#op._S_lt)    | Match les valeurs strictement inférieures à une valeur spécifiée  |
| [`$lte`](https://docs.mongodb.com/manual/reference/operator/query/lte/#op._S_lte) | Match les valeurs inférieures ou égales à une valeur spécifiée    |
| [`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne)    | Match les valeurs différentes à la valeur sépcifiée               |
| [`$nin`](https://docs.mongodb.com/manual/reference/operator/query/nin/#op._S_nin) | Match les valeurs absentes d'un ensemble                          |

On peut également utiliser des opérateurs logiques pour composer des requêtes plus complexes. Par défaut, lorsqu'on ne précise pas d'opérateur logique, il s'agit d'un opérateur `$and` implicite. Avant d'aller plus loin, voici la liste complète des opérateurs logiques :

| Opérateur                                                                         | Syntaxe                                                                        |
|-----------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| [`$and`](https://docs.mongodb.com/manual/reference/operator/query/and/#op._S_and) | `{ $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] }` |
| [`$not`](https://docs.mongodb.com/manual/reference/operator/query/not/#op._S_not) | `{ field: { $not: { <operator-expression> } } }`                               |
| [`$nor`](https://docs.mongodb.com/manual/reference/operator/query/nor/#op._S_nor) | `{ $nor: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }`  |
| [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#op._S_or)    | `{ $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }`   |

Il existe également des opérateurs pour vérifier l'existence ou le type de certains champs au sein de documents.

| Opérateur                                                                                  | Syntaxe                                                                                             |
|--------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| [`$exists`](https://docs.mongodb.com/manual/reference/operator/query/exists/#op._S_exists) | `{ field: { $exists: <boolean> } }`                                                                 |
| [`$type`](https://docs.mongodb.com/manual/reference/operator/query/type/#op._S_type)       | `{ field: { $type: <BSON type> } }` ou `{ field: { $type: [ <BSON type1> , <BSON type2>, ... ] } }` |

Il existe encore d'autres types d'opérateurs, mais nous les étudierons plus tard.

> À vous de jouer !
>
> À l'aide des opérateurs vus précédement, écrivez les requêtes suivantes :
> - Récupérez les salles contenant 15 postes ou moins,
> - Récupérez les salles sans postes et avec 28 places ou plus,
> - Récupérez les salles sans postes et avec 26 places, ou avec 15 postes et 28 places ou moins,
> - Récupérez les salles n'ayant pas moins de 15 postes ou ayant 15 postes ou moins et 28 places.

#### Itération sur le curseur

Lors d'une requête de sélection sur une collection, on peut stocker le curseur du résultat dans une variable JavaScript.

```javascript
var resultat = db.salles.find()
var document = resultat.hasNext() ? resultat.next() : null
if (document) { printjson(document.nom) }
// Retourne "TP1"
```

On peut ainsi programmer quelques éléments sur le curseur en JavaScript directement depuis la cli.

#### Modification du comportement du curseur

Par défaut, le curseur nous renvoie tout les éléments correspondants à nos critères sans garantie sur l'ordre en sortie et sur un format en une ligne par document. On peut modifier ce comportement à l'aide de différentes commandes.

| Commande                                                                                 | Description                                                    |
|------------------------------------------------------------------------------------------|----------------------------------------------------------------|
| [`pretty()`](https://docs.mongodb.com/manual/reference/method/cursor.pretty/)            | Permet d'afficher le résultat du cuseur de façon plus lisible  |
| [`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort)    | Permet de trier la sortie du curseur sur un ou plusieur champs |
| [`skip()`](https://docs.mongodb.com/manual/reference/method/cursor.skip/#cursor.skip)    | Permet de passer un nombre de résultats défini                 |
| [`limit()`](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit) | Permet de limiter le nombre de résultats                       |

### Modification de documents

Pour modifier des documents, Mongo nous met à disposition la commande `update()`, avec deux variantes, `updateOne()` et `updateMany()`. Ces trois commandes partagent la même syntaxe et fonctionnent avec les mêmes principes. En premier paramètre, on va avoir la requête permettant de sélectionner le ou les éléments à mettre à jour, en second paramètre, la modification à apporter, et en dernier paramètre, des options.

```js
db.collection.update(<query>, <update>, <options>)
```

#### Clause query

La partie `<query>` fonctionne avec les mêmes opérateurs que pour les requêtes de sélection de données. Vous pouvez vous baser sur cette partie du cours pour composer vos différents filtres afin de sélectionner les documents à mettre à jour.

#### Clause update

La partie `<update>`, elle, dispose de ses propores opérateurs. Ceux-ci décrivent des opérations atomiques. Pour le moment, nous ne verrons que les opérateurs de mise à jour simples.

| Opérateur                                                                                                  | Action                                                                           |
|------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| [`$currentDate`](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate) | Met à jour un champ avec la date courante                                        |
| [`$inc`](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc)                         | Incrémente un champ avec la valeur donnée                                        |
| [`$min`](https://docs.mongodb.com/manual/reference/operator/update/min/#up._S_min)                         | Met à jour un champ si la nouvelle valeur est plus petite que la valeur courante |
| [`$max`](https://docs.mongodb.com/manual/reference/operator/update/max/#up._S_max)                         | Met à jour un champ si la nouvelle valeur est plus grande que la valeur courante |
| [`$mul`](https://docs.mongodb.com/manual/reference/operator/update/mul/#up._S_mul)                         | Multiplie la valeur d'un champ avec la valeur donnée                             |
| [`$rename`](https://docs.mongodb.com/manual/reference/operator/update/rename/#up._S_rename)                | Renome un champ dans un document                                                 |
| [`$set`](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set)                         | Remplace la valeur d'un champ                                                    |
| [`$setOnInsert`](https://docs.mongodb.com/manual/reference/operator/update/setOnInsert/#up._S_setOnInsert) | Remplace la valeur d'un champ dans le cas d'une création de document             |
| [`$unset`](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset)                   | Supprime un champ d'un document                                                  |

##### `$currentDate`

L'opérateur `$currentDate` permet de mettre à jour un champ en lui donnant la valeur de la date courante.

Syntaxe : `{ $currentDate: { <field>: <typeSpecification>, ... } }`.

`<typeSpecification>` peut prendre deux valeurs :
- `true`, qui stockera la date courante avec un type `Date`
- Un document, soit `{ $type: "timestamp" }` soit `{ $type: "date" }` qui définissent explicitement le type. Ce document est sensible à la casse.

##### `$inc`

L'opérateur `$inc` permet d'incrémenter la valeur d'un champ avec un certain pas.

Syntaxe : `{ $inc: { <field>: <amount>, ... } }`

##### `$min` et `$max`

Les opérateurs `$min` et `$max` permettent de mettre à jour la valeur d'un champ seulement si elle est respectivement plus petite ou plus grande que la valeur du champ avant mise à jour. Si le champ n'existe pas, il est créé et prend la valeur indiquée.

Syntaxe : `{ $min: { <field>: <value>, ... } }` et `{ $max: { <field>: <value>, ... } }`

##### `$mul`

L'opérateur `$mul` multiplie le contenu d'un champ par la valeur indiquée. Si le champ n'existe pas, il est créé et prend la valeur `0`.

Syntaxe : `{ $mul: { <field>: <number>, ... } }`

##### `$rename`

L'opérateur `$rename` permet de renommer un champ. Si le champ à renommer n'existe pas, il ne se passe rien. Si le champ destination existe déjà, il est supprimé et remplacé.

Syntaxe : `{$rename: { <field>: <newName>, ... } }`

##### `$set` et `$setOnInsert`

Les opérateurs `$set` et `$setonInsert` permettent d'assigner une valeur à un champ. Si la champ n'existe pas, il est créé avec la valeur assignée. Dans le cas de `$setOnInsert`, si aucun nouveau document n'est créé, il ne fait rien.

Syntaxe : `{ $set: { <field>: <value>, ... } }` et `{ $setOnInsert: { <field>: <value>, ... } }`

##### `$unset`

L'opérateur `$unset` permet de supprimer un champ d'un document. Il doit prendre une valeur, mais peut importe la valeur transmise, il supprimera le champ. On met la valeur `""` par convention.

Syntaxe : `{ $unset: { <field>: "", ... } }`

#### Clause options

Nous ne regarderons qu'une seule option pour le moment, l'option `upsert`. Cette option, quand renseignée à `true`, permet de créer un document correspondant à la valeur du paramètre `<update>` si aucun document ne remplissait les exigences du paramètre `<query>`.

```js
db.salles.updateOne({nom: "TD3"}, {$set: {places: 28}})
```

### Suppression de documents

Enfin, nous pouvons supprimer des documents de MongoDB à l'aides des commandes `deleteOne()` et `deleteMany()`. Elles prennent en premier paramètre une `<query>`, de la même façon que pour les requêtes de récupération et de mise à jour/

```js
db.salles.updateMany({nom: "TD4"}, {$set: {nom: "TD4", places: 28, postes: 12}}, {upsert: true})
// Retourne :
//{
//        "acknowledged" : true,
//        "matchedCount" : 0,
//        "modifiedCount" : 0,
//        "upsertedId" : ObjectId("5bc3740142b6e9f33bce2178")
//}
db.salles.deleteOne({nom: "TD4"})
```
> À vous de jouer !
>
> Pour ce second exercice type TP noté, vous avez le droit à la documentation de MongoDB et de me demander de l'aide. Le projet sera à rendre au plus tard le dimanche 4 novembre à 12h à mon adresse mail.
>
> Le rendu sera sous forme de document texte contenant la suite des commandes que vous aurez éxécutés. Pour chaque question, vous ferez un encart rappelant le numéro de la question comme sur l'exemple suivant : `Question 1.`.
>
> Question 1 : Créez une collection `people` dans une base de données `analytics`.
>
> Question 2 : À l'aide d'une commande d'insertion, insérez les données suivantes :
>
> | technicalId | firstname | lastname    | email                         | age | children | 
> |-------------|-----------|-------------|-------------------------------|-----|----------|
> | 1           | Clément   | Charpentier | clement.charpentier@gmail.com | 25  | 2        |
> | 2           | Lauriane  | Rodriguez   | lauriane.rodriguez@gmail.com  | 32  | 3        |
> | 3           | Amélie    | Martin      | amelie.martin@gmail.com       | 31  | 1        |
> | 4           | Lola      | Brunet      | lola.brunet@gmail.com         | 27  | 2        |
> | 5           | Timéo     | Blanchard   | timeo.blanchard@gmail.com     | 43  | 3        |
> | 6           | Corentin  | Deschamps   | corentin.deschamps@gmail.com  | 15  | 0        |
> | 7           | Hugo      | Bernard     | hugo.bernard@gmail.com        | 19  | 0        |
> | 8           | Davy      | Roy         | davy.roy@gmail.com            | 23  | 0        |
> | 9           | Bruno     | Robin       | bruno.robin@gmail.com         | 35  | 2        |
> | 10          | Yasmine   | Maillard    | yasmine.maillard@gmail.com    | 47  | 3        |
>
> Question 3 : À l'aide d'une méthode de mise à jour de données, insérez les données suivantes :
>
> | technicalId | firstname | lastname    | email                         | age | children |
> |-------------|-----------|-------------|-------------------------------|-----|----------|
> | 11          | Zacharis  | Dufour      | zacharis.dufour@gmail.com     | 53  | 3        |
> | 12          | Renaud    | Marechal    | renaud.marechal@gmail.com     | 48  | 2        |
> | 13          | Lilou     | Germain     | lilou.germain@gmail.com       | 29  | 1        |
> | 14          | Maryam    | Louis       | maryam.louis@gmail.com        | 36  | 2        |
> | 15          | Amandine  | Adam        | amandine.adam@gmail.com       | 21  | 0        |
> | 16          | Lana      | Daniel      | lana.daniel@gmail.com         | 27  | 2        |
> | 17          | Clotilde  | Vincent     | clotilde.vincent@gmail.com    | 18  | 0        |
> | 18          | Benjamin  | Morin       | benjamin.morin@gmail.com      | 36  | 1        |
> | 19          | Kilian    | Le gall     | kilian.le-gall@gmail.com      | 17  | 0        |
> | 20          | Noë       | Germain     | noe.germain@gmail.com         | 27  | 0        |
>
> Question 4 : Récupérez personnes de plus de 27 ans et ayant au moins 3 enfants. Passez les deux premiers résultats.
>
> Question 5 : Renommez le champ `technicalId` en `id`.
>
> Question 6 : Ajoutez un an à toutes les personnes de moins de 20 ans.
>
> Question 7 : Récupérez les prénoms des 5 personnes ayant le plus d'enfants.
>
> Question 8 : Ajoutez un champ `cars` avec une valeur par défaut de `2` aux personnes de plus de 30 ans.
>
> Question 9 : Sur les documents avec un id technique inférieure ou égal à 10, ajoutez un champ `lastUpdate` avec pour valeur le timestamp courant.
>
> Question 10 : Supprimez les personnes ayant entre 25 et 35 ans n'ayant qu'un seul enfant, ainsi que les personnes de plus de 40 ans avec 2 enfants.
>
> Question 11 : Récupérez les 5 personnes les plus agées, par nombre d'enfants décroissant. Rendez le résultat plus lisible.
>
> Question 12 : Supprimez le champ `email` des personnes entre 30 et 40 ans.
>
> Question 13 : Récupérez 5 personnes ayant un email en triant par email de façon décroissante et nom de famille croissant, à condition que les personnes aient entre 15 et 20 ans ou 25 et 30 ans.
>
> Question 14 : Récupérez les personnes ayant un email et des voitures et trier par nombre d'enfant décroissant et âge croissant.
>
> Question 15 : Supprimer les personnes ayant deux enfants.

## Tableaux et sous-documents


### Opérations avec les tableaux


### Intérêt des sous-documents


## Agrégats


