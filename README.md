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
- [TP Blanc](#tp-blanc)
  - [Correction](#correction)
- [Tableaux et sous-documents](#tableaux-et-sous-documents)
  - [Opérations avec les tableaux](#op%C3%A9rations-avec-les-tableaux)
    - [Tableaux en mode insertion](#tableaux-en-mode-insertion)
    - [Lecture de données dans un tableau](#lecture-de-donn%C3%A9es-dans-un-tableau)
      - [`$all`](#all)
      - [`$elemMatch`](#elemmatch)
      - [`$size`](#size)
    - [Opérateurs de projection](#op%C3%A9rateurs-de-projection)
      - [`$` (Projection)](#-projection)
      - [`$elemMatch` (Projection)](#elemmatch-projection)
      - [`$slice` (Projection)](#slice-projection)
    - [Opérateurs de mise à jour de données dans un tableau](#op%C3%A9rateurs-de-mise-%C3%A0-jour-de-donn%C3%A9es-dans-un-tableau)
      - [`$`, `$[]` et `$[<identifier>]`](#--et-identifier)
      - [`$addToSet`](#addtoset)
      - [`$pop`](#pop)
      - [`$pull`](#pull)
      - [`$push`](#push)
      - [`$pullAll`](#pullall)
    - [Modifiers pour l'update](#modifiers-pour-lupdate)
      - [`$each`](#each)
      - [`$position`](#position)
      - [`$slice` (Update)](#slice-update)
      - [`$sort`](#sort)
  - [Sous-documents et tableaux de sous-documents](#sous-documents-et-tableaux-de-sous-documents)
  - [Correction](#correction-1)
- [Agrégats](#agr%C3%A9gats)
  - [Map-Reduce](#map-reduce)
  - [Pipeline d'aggrégation](#pipeline-daggr%C3%A9gation)
- [Recherches textuelles et par Regex](#recherches-textuelles-et-par-regex)
- [Recherche Géospatiale](#recherche-g%C3%A9ospatiale)
- [Introduction à la mise en place de cluster mongodb](#introduction-%C3%A0-la-mise-en-place-de-cluster-mongodb)
- [Pour aller plus loins](#pour-aller-plus-loins)

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

## TP Blanc

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
> Question 4 : Récupérez les personnes de plus de 27 ans et ayant au moins 3 enfants. Passez les deux premiers résultats.
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

### Correction

```js
// Question 1 : Créez une collection `people` dans une base de données `analytics`.
use analytics
db.createCollection('people')
// Question 2 : À l'aide d'une commande d'insertion, insérez les données suivantes :
db.people.insertMany([
  { technicalId: 1, firstname: 'Clément', lastname: 'Charpentier', email: 'clement.charpentier@gmail.com', age: 25, children: 2 },
  { technicalId: 2, firstname: 'Lauriane', lastname: 'Rodriguez', email: 'lauriane.rodriguez@gmail.com', age: 32, children: 3 },
  { technicalId: 3, firstname: 'Amélie', lastname: 'Martin', email: 'amelie.martin@gmail.com', age: 31, children: 1 },
  { technicalId: 4, firstname: 'Lola', lastname: 'Brunet', email: 'lola.brunet@gmail.com', age: 27, children: 2 },
  { technicalId: 5, firstname: 'Timéo', lastname: 'Blanchard', email: 'timeo.blanchard@gmail.com', age: 43, children: 3 },
  { technicalId: 6, firstname: 'Corentin', lastname: 'Deschamps', email: 'corentin.deschamps@gmail.com', age: 15, children: 0 },
  { technicalId: 7, firstname: 'Hugo', lastname: 'Bernard', email: 'hugo.bernard@gmail.com', age: 19, children: 0 },
  { technicalId: 8, firstname: 'Davy', lastname: 'Roy', email: 'davy.roy@gmail.com', age: 23, children: 0 },
  { technicalId: 9, firstname: 'Bruno', lastname: 'Robin', email: 'bruno.robin@gmail.com', age: 35, children: 2 },
  { technicalId: 10, firstname: 'Yasmine', lastname: 'Maillard', email: 'yasmine.maillard@gmail.com', age: 47, children: 3 }
])
// Question 3 : À l'aide d'une méthode de mise à jour de données, insérez les données suivantes :
db.people.updateMany({ technicalId: 11 }, { $set: { firstname: 'Zacharis', lastname: 'Dufour', email: 'zacharis.dufour@gmail.com', age: 53, children: 3 } }, { upsert: true })
db.people.updateMany({ technicalId: 12 }, { $set: { firstname: 'Renaud', lastname: 'Marechal', email: 'renaud.marechal@gmail.com', age: 48, children: 2 } }, { upsert: true })
db.people.updateMany({ technicalId: 13 }, { $set: { firstname: 'Lilou', lastname: 'Germain', email: 'lilou.germain@gmail.com', age: 29, children: 1 } }, { upsert: true })
db.people.updateMany({ technicalId: 14 }, { $set: { firstname: 'Maryam', lastname: 'Louis', email: 'maryam.louis@gmail.com', age: 36, children: 2 } }, { upsert: true })
db.people.updateMany({ technicalId: 15 }, { $set: { firstname: 'Amandine', lastname: 'Adam', email: 'amandine.adam@gmail.com', age: 21, children: 0 } }, { upsert: true })
db.people.updateMany({ technicalId: 16 }, { $set: { firstname: 'Lana', lastname: 'Daniel', email: 'lana.daniel@gmail.com', age: 27, children: 2 } }, { upsert: true })
db.people.updateMany({ technicalId: 17 }, { $set: { firstname: 'Clotilde', lastname: 'Vincent', email: 'clotilde.vincent@gmail.com', age: 18, children: 0 } }, { upsert: true })
db.people.updateMany({ technicalId: 18 }, { $set: { firstname: 'Benjamin', lastname: 'Morin', email: 'benjamin.morin@gmail.com', age: 36, children: 1 } }, { upsert: true })
db.people.updateMany({ technicalId: 19 }, { $set: { firstname: 'Kilian', lastname: 'Le gall', email: 'kilian.le-gall@gmail.com', age: 17, children: 0 } }, { upsert: true })
db.people.updateMany({ technicalId: 20 }, { $set: { firstname: 'Noë', lastname: 'Germain', email: 'noe.germain@gmail.com', age: 27, children: 0 } }, { upsert: true })
// Question 4 : Récupérez les personnes de plus de 27 ans et ayant au moins 3 enfants. Passez les deux premiers résultats.
db.people.find({ age: { $gt: 27 }, children: { $gte: 3 } })
// Question 5 : Renommez le champ `technicalId` en `id`.
db.people.updateMany({}, { $rename: { 'technicalId' : 'id' } })
// Question 6 : Ajoutez un an à toutes les personnes de moins de 20 ans.
db.people.updateMany({ age: { $lt: 20 } }, { $inc: { age: 1 } })
// Question 7 : Récupérez les prénoms des 5 personnes ayant le plus d'enfants.
db.people.find({}, { _id: 0, firstname: 1 }).sort({ children: -1 }).limit(5)
// Question 8 : Ajoutez un champ `cars` avec une valeur par défaut de `2` aux personnes de plus de 30 ans.
db.people.updateMany({ age: { $gt: 30 } }, { $set: { cars: 2 } })
// Question 9 : Sur les documents avec un id technique inférieure ou égal à 10, ajoutez un champ `lastUpdate` avec pour valeur le timestamp courant.
db.people.updateMany({ id: { $lte: 10 } }, { $currentDate: { lastUpdate: { $type: 'timestamp' } } })
// Question 10 : Supprimez les personnes ayant entre 25 et 35 ans n'ayant qu'un seul enfant, ainsi que les personnes de plus de 40 ans avec 2 enfants.
db.people.deleteMany({ $or: [{ age: { $gt: 25, $lt: 35 }, children: 1 }, { age: { $gt: 40 }, children: 2 }] })
// Question 11 : Récupérez les 5 personnes les plus agées, par nombre d'enfants décroissant. Rendez le résultat plus lisible.
db.people.find().sort({ age: -1, children: -1}).limit(5).pretty()
// Question 12 : Supprimez le champ `email` des personnes entre 30 et 40 ans.
db.people.updateMany({ age: { $gt: 30, $lt: 40 } }, { $unset: { email: '' } })
// Question 13 : Récupérez 5 personnes ayant un email en triant par email de façon décroissante et nom de famille croissant, à condition que les personnes aient entre 15 et 20 ans ou 25 et 30 ans.
db.people.find({ email: { $exists: true }, $or: [{ age: { $gt: 15, $lt: 20 } }, { age: { $gt: 25, $lt: 30 } }] }).sort({ email: 1, lastname: -1 })
// Question 14 : Récupérez les personnes ayant un email et des voitures et trier par nombre d'enfant décroissant et âge croissant.
db.people.find({ email: { $exists: true }, cars: { $exists: true } }).sort({ children: -1, age: 1 })
// Question 15 : Supprimer les personnes ayant deux enfants.
db.people.deleteMany({ children: 2 })
```

## Tableaux et sous-documents

Jusqu'à présent, nous avons utilisé des objets mongo assez simple, avec une seule dimension dans les champs. Nous allons à présent voir la capacité de mongo à gérer des tableaux ainsi que des sous-documents.

### Opérations avec les tableaux

Mongo offre la possibilité de traiter les tableaux comme n'importe quel type avec des opérateurs dédiés aux opérations CRUD. Nous pouvons donc définir des champs de type tableaux dans nos différents documents. 

#### Tableaux en mode insertion

Les tableaux se notent entre crochets (`[]`), et peuvent être de tout types.

Si nous reprennons notre exemple avec les salles de l'IUT, nous pouvons faire la requête suivante pour ajouter une salle `TP4` avec quelques `postes` :

```js
db.salles.insert({ nom: 'TP4', places: 26, postes: ['TP4-01', 'TP4-02', 'TP4-03', 'TP4-04'] })
// Retourne : WriteResult({ "nInserted" : 1 })
db.salles.find({ nom: 'TP4' }).pretty()
// Retourne :
// {
//         "_id" : ObjectId("5bddd862d43bed336676b6bf"),
//         "nom" : "TP4",
//         "places" : 26,
//         "postes" : [
//                 "TP4-01",
//                 "TP4-02",
//                 "TP4-03",
//                 "TP4-04"
//         ]
// }
```

#### Lecture de données dans un tableau

Afin de récupérer finement des données dans un tableau, mongo nous met à disposition de nouveaux opérateurs.

| Opérateur                                                                                           | Action                                                                                                    |
|-----------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| [`$all`](https://docs.mongodb.com/manual/reference/operator/query/all/#op._S_all)                   | Match les documents dont les tableaux contenant toutes les valeurs spécifiées                             |
| [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#op._S_elemMatch) | Match les documents dont les tableaux pour lesquels au moins un élément remplit les critères de sélection |
| [`$size`](https://docs.mongodb.com/manual/reference/operator/query/size/#op._S_size)                | Match les documents dont les tableaux pour lesquels la longueur correspond à la valeur entrée             |

Ces opérateurs ne font pas tout, il y a également un nouvel élément syntaxique : la notation `.`. Le `.` permet d'accéder à des sous-élements d'un tableau. Nous l'illustrerons plus tard quand nous travaillerons sur les sous-documents, qui utilisent la même syntaxe.

##### `$all`

L'opérateur `$all` permet de récupérer les document dont le champ de tableau désigné contient toutes les valeurs spécifiées. Si il en manque une seule, le document n'est pas pris, si elles sont toutes présentes et d'autres valeurs avec, le document est pris.

Syntaxe : `{ <field>: { $all: [ <value1> , <value2> ... ] } }`

On peut reproduire son comportement à l'aide de l'opérateur `$and` étudié précédement.

```js
db.salles.find({ postes: { $all: ['TP4-01', 'TP4-03'] } })
db.salles.find({ $and: [{ postes: 'TP4-01' }, { postes: 'TP4-03' }] })
// Ces deux commandes sont équivalentes
```

##### `$elemMatch`

L'opérateur `$elemMatch`permet de récupérer les documents dont le champ de tableau désigné match les critères de sélection (query).

Syntaxe : `{ <field>: { $elemMatch: { <query1>, <query2>, ... } } }`

Exemple :

```js
db.test.insert({ titre: 'Suite quelconque', valeurs: [2, 4, 6, 8, 10, 12, 14] })
db.test.insert({ titre: 'Autre suite', valeurs: [20, 40, 60, 80, 100, 120, 140] })
db.test.find({ valeurs: { $elemMatch: { $gte: 10, $lt: 20  } } })
// Retourne le premier document : le second ne contient pas de valeur supérieures ou égales à 10 et inférieures à 20.
db.test.find({ valeurs: { $elemMatch: { $gt: 10, $lte: 20  } } })
// Retourne les deux documents, ils conetienent biens des valeurs plus grandes que 10 et inférieures ou égales à 20 dans le tableau valeurs.
```

##### `$size`

L'opérateur `$size` retourne les documents dont le champ donné est de la taille spécifiée.

Syntaxe : `{ <field>: { $size: <fieldSize> } }`

#### Opérateurs de projection

Mongo propose différents opérateurs de projection, nous allons nous intéressé à seulement trois d'entre eux qui nous seront utile pour les traitements sur les tableaux.

| Opérateur                                                                                                  | Action                                                                                              |
|------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| [`$`](https://docs.mongodb.com/manual/reference/operator/projection/positional/#proj._S_)                  | Permet de projeter le premier élément de tableau remplissant la condition de recherche              |
| [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/projection/elemMatch/#proj._S_elemMatch) | Permet de projeter le premier élément de tableau remplissant la condition de recherche en paramètre |
| [`$slice`](https://docs.mongodb.com/manual/reference/operator/projection/slice/#proj._S_slice)             | Permet de projeter une partie du tableau                                                            |

##### `$` (Projection)

Permet de projeter le premier élément de tableau remplissant la condition de recherche.

Syntaxes :
```js
db.collection.find({ <array>: <value>, ... }, { "<array>.$": 1 })
db.collection.find({ <array.field>: <value>, ... }, { "<array>.$": 1 })
```

##### `$elemMatch` (Projection)

Permet de projeter le premier élément de tableau remplissant la condition de recherche en paramètre.

Syntaxe : `db.collection.find({ <field>: <value> }, { <field>: { $elemMatch: <query> } })`

##### `$slice` (Projection)

Permet de projeter une partie du tableau.

Syntaxe : `db.collection.find({ <field>: <value> }, { <array>: { $slice: <count> } })`

Remarque : Si le `<count>` est positif, la projection affichera ce nombre d'éléments en partant du début du tableau, si il est négatif, il le fera en partant de la fin du tableau.

#### Opérateurs de mise à jour de données dans un tableau

Pour mettre à jour les tableaux, mongo nous offre de nouveau plusieurs opérateurs.

| Opérateur                                                                                                                    | Action                                                                                                                            |
|------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| [`$`](https://docs.mongodb.com/manual/reference/operator/update/positional/#up._S_)                                          | Agit comme un placeholder pour mettre à jour le premier élément du tableau satisfaisant la condition                              |
| [`$[]`](https://docs.mongodb.com/manual/reference/operator/update/positional-all/#up._S_[])                                  | Agit comme un placeholder pour mettre à jour tous les éléments du tableau satisfaisant la condition                               |
| [`$[<identifier>]`](https://docs.mongodb.com/manual/reference/operator/update/positional-filtered/#up._S_[%3Cidentifier%3E]) | Agit comme un placeholder pour mettre à jour tous les éléments de tableaux satisfaisant la clause `arrayFilters` pour le document |
| [`$addToSet`](https://docs.mongodb.com/manual/reference/operator/update/addToSet/#up._S_addToSet)                            | Ajoute un élément au tableau si il n'existe pas déjà                                                                              |
| [`$pop`](https://docs.mongodb.com/manual/reference/operator/update/pop/#up._S_pop)                                           | Retire le premier ou le dernier élément du tableau                                                                                |
| [`$pull`](https://docs.mongodb.com/manual/reference/operator/update/pull/#up._S_pull)                                        | Retire tous les éléments du tableau satisfaisant la condition                                                                     |
| [`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push)                                        | Ajoute un élément dans un tableau                                                                                                 |
| [`$pullAll`](https://docs.mongodb.com/manual/reference/operator/update/pullAll/#up._S_pullAll)                               | Retire toutes les valeurs du tableau spécifié si elles sont présente dans la liste donnée en paramètre                            |

##### `$`, `$[]` et `$[<identifier>]`

Pour ces trois opérateurs, nous allons directement nous reporter à la documentation de mongo :
- [Documentation : `$`](https://docs.mongodb.com/manual/reference/operator/update/positional/#up._S_)
- [Documentation : `$[]`](https://docs.mongodb.com/manual/reference/operator/update/positional-all/#up._S_[])
- [Documentation : `$[<identifier>]`](https://docs.mongodb.com/manual/reference/operator/update/positional-filtered/#up._S_[%3Cidentifier%3E])

##### `$addToSet`

L'opérateur `$addToSet` ajoute l'élément passé en paramètre dans le tableau, à condition qu'il ne soit pas déjà présent. Si l'élément en paramètre est un tableau, par défaut, il sera ajouté en tant que sous-tableau de notre tableau.

Syntaxe : `{ $addToSet: { <field1>: <value1>, ... } }`

##### `$pop`

L'opérateur `$pop` retire le premier ou le dernier élément du tableau spécifié. Si le champ n'est pas un tableau, la commande échouée, si le tableau est vide suite à l'opération, il reste en tant que tableau vide.

Syntaxe : `{ $pop: { <field>: <-1 | 1>, ... } }`

Si on passe `-1`, le premier élément est retiré, si on passe `1`, le dernier est retiré.

##### `$pull`

L'opérateur `$pull` supprime toutes les occurences des éléments correspondant aux critères de sélection dans un tableau.

Syntaxe : `{ $pull: { <field1>: <value|condition>, <field2>: <value|condition>, ... } }`

Notes :
- Si la valeur a supprimer est un tableau, il faut que le critère de sélection soit exactement le même tableau, incluant l'ordre des éléments.
- Si la valeur est un document, le document devra comporter les mêmes champs, peut importe leur ordre.

##### `$push`

L'opérateur `$push` permet d'ajouter l'élément en paramètre dans un tableau. Si l'élément en paramètre est un tableau, par défaut, il est inséré en tant que sous-tableau.

Syntaxe : `{ $push: { <field1>: <value1>, ... } }`

##### `$pullAll`

L'opérateur `$pullAll` permet de supprimer toutes les valeurs listées.

Syntaxe : `{ $pullAll: { <field1>: [ <value1>, <value2> ... ], ... } }`

#### Modifiers pour l'update

En plus des opérateurs, nous avons également différents modifiers qui vont permettre de changer le comportement des opérateurs, pricipalement de l'opérateur `$push`.

| Opérateur                                                                                         | Action                                                                                        |
|---------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| [`$each`](https://docs.mongodb.com/manual/reference/operator/update/each/#up._S_each)             | Modifie les opérateurs `$push` et `$addToSet` pour permettre l'insertion de plusieurs valeurs |
| [`$position`](https://docs.mongodb.com/manual/reference/operator/update/position/#up._S_position) | Modifie l'opérateur `$push`pour permettre l'insertion à une position précise du tableau       |
| [`$slice`](https://docs.mongodb.com/manual/reference/operator/update/slice/#up._S_slice)          | Modifie l'opérateur `$push`pour permettre de limiter la taille finale du tableau              |
| [`$sort`](https://docs.mongodb.com/manual/reference/operator/update/sort/#up._S_sort)             | Modifie l'opérateur `$push`pour trier les éléments du tableau à l'update                      |

##### `$each`

Modifie les opérateurs `$push` et `$addToSet` pour permettre l'insertion de plusieurs valeurs.

Syntaxes :
- `{ $addToSet: { <field>: { $each: [ <value1>, <value2> ... ] } } }`
- `{ $push: { <field>: { $each: [ <value1>, <value2> ... ] } } }`

##### `$position`

Modifie l'opérateur `$push`pour permettre l'insertion à une position précise du tableau, avec un index basé sur 0. Si on prend des valeurs négatives, il faut compter en positions avant la fin du tableau. Par exemple, `-2` signifie "deux cases avant la dernière case du tableau".

Syntaxe :

```js
{
  $push: {
    <field>: {
       $each: [ <value1>, <value2>, ... ],
       $position: <num>
    }
  }
}
```

Remarque : Par défaut, `$push` ajoute les éléments en fin de tableau, c'est pour celà qu'il n'est pas possible de faire cette action avec ce modifier. Le modifier `$position` requiert la présence du modifier `$each`.

##### `$slice` (Update)

Modifie l'opérateur `$push`pour permettre de limiter la taille finale du tableau. Si on passe on valeur positive, mongo gardera ce nombre d'éléments du tableau en partant du début. Avec une valeur négative, on gardera les derniers éléments du tableau.

Syntaxe :

```js
{
  $push: {
     <field>: {
       $each: [ <value1>, <value2>, ... ],
       $slice: <num>
     }
  }
}
```

Remarque : Le modifier `$slice` requiert la présence du modifier `$each`.

##### `$sort`

Modifie l'opérateur `$push`pour trier les éléments du tableau à l'update

Syntaxe :

```js
{
  $push: {
     <field>: {
       $each: [ <value1>, <value2>, ... ],
       $sort: <sort specification>
     }
  }
}
```

Remarques :
- Pour `<sort specification>` :
  - Pour faire du tri sur les éléments, quelque soient leur type, il faut passer `1` ou `-1` pour faire des tris croissant ou décroissant.
  - Si les éléments sont des documents, pour tirer sur un champ en particulier, la syntaxe de `<sort specification>` est la suivante : `{ <field>: 1 }` ou `{ <field>: -1 }`
- Le modifier `$sort` requiert la présence du modifier `$each`.
- Dans certaines versions de mongo, le modifier `$slice` est requis. Ce n'est pas notre cas.

### Sous-documents et tableaux de sous-documents

À l'aide de la notation `.` dans les noms de champs, nous allons pouvoir effectuer toutes sortes de requêtes sur des sous-documents et des tableaux de sous-documents.

Tout ce que nous avons vu jusque ici reste entièrement valable pour ces deux éléments.

> À vous de jouer !
>
> Dans cet exercice, nous allons entièrement remodelé notre base iut. Dans un premier temps, supprimez entièrement la base iut (via la commande ou l'[`IHM`](http://localhost:8081)), nous allons ensuite la recréer au propre.
>
> Afin de garder notre base relativement simple, nous allons faire les suppositions suivantes :
> - Il n'y a que trois `salles` TP de cinq `postes` chacunes
> - Il n'y a que quatre `enseignants`
> - Il n'y a que deux `groupes` de dix `étudiants` chacuns, en `binômes`
> - Il n'y a que deux créneaux de cours (matin et après-midi) par jour
> - La semaine de cours dure du mardi au jeudi
>
> Pour cet exercice, nous ne modéliserons que la collection `salles`, et le nécéssaire des sous-documents.
>
> Les `salles` sont caractérisées par les éléments suivants :
> - Un nom
> - Des `cours`
> - Des `postes`
>
> Les `postes` sont caractérisés par les éléments suivants :
> - Un nom, sous la forme `<nom de la salle TP>-<identifiant à un chiffre>`
> - Des `OS` (simples chaînes de caractères)
> - Une `salle`
>
> Les `enseignants` sont caractérisés par les éléments suivants :
> - Un prénom
> - Un nom
> - Des `matières` (simples chaînes de caractères)
> - Des `cours`
>
> Les `groupes` sont caractérisés par les éléments suivants :
> - Un nom
> - Des `étudiants`
> - Une promotion (nombre entier)
>
> Les `étudiants` sont caractérisés par les éléments suivants :
> - Un prénom
> - Un nom
> - Un `groupe`
> - Des `cours`
> - Un `binôme`
>
> Les `binômes` sont caractérisés par les éléments suivants :
> - Un nom
> - Des `étudiants`
>
> Les `cours` sont caractérisés par les éléments suivants :
> - Un `enseignant`
> - Un `groupe`
> - Une `matière`
> - Une `salle`
> - Un jour (simple chaîne de caractères)
> - Une heure de début (nombre entier)
> - Une heure de fin (nombre entier)
>
> Question 1 : Créez trois salles, respectivement nommées TP01, TP02 et TP03.
>
> Question 2 : Ajoutez des postes dans les salles de sorte que :
> - La TP01 comporte des postes sous Windows 7 et Ubuntu 14
> - La TP02 comporte des postes sous Windows 7 et Centos 7
> - La TP03 comporte des postes sous Windows 10 et Ubuntu 14
>
> Question 3 : Placez un cours de `programmation réseau` en TP01 le mercredi de 9h à 13h avec `Denis Desroches` et le groupe 2, composé des étudiants suivants :
> - Alice Duffet
> - Michèle Roy
> - Alexis Boivin
> - Romain Fluet
> - Vincent Fugère
> - Coralie Briard
>
> Question 4 : Un étudiant rejoint la promo tardivement, ajoutez le au groupe 2 en triant les étudiants par nom de famille croissant et prénom décroissant : Mathias Fluet.
>
> Question 5 : Les horraires du cours changent, il a maintenant lieu de 9h à 12h.
>
> Question 6 : Récupérez les postes sous Ubuntu 14, en en prenant 3 à partir du second
>
> Question 7 : Récupérez les salles ayant des postes sous Windows 7
>
> Question 8 : Le poste 3 de la salle 1 tombe en panne, retirez-le
>
> Question 9 : Ajoutez les 4 étudiants suivants dans le groupe 2, en veillant à la capacité maximale de la classe :
> - Emmeline Faure
> - Martin Guimond
> - Eugène Quiron
> - Gauthier Guibord
>
> Question 10 : Immaginez maintenant que l'on gère l'emploi du temps d'une année de cette façon, quel(s) problème(s) y voyez-vous ?

### Correction

```js
// Question 1
// On se place sur la base
use iut
// On créée notre collection
db.createCollection('salles')
// On insère les TP01 et TP02
db.salles.insertMany([
    { nom: 'TP01' },
    { nom: 'TP02' }
])
// On insère la TP03 avec un champ poste initialisé à tableau vide, ceci afin de vous montrer comment travailler avec quelques opérateurs particuliers
db.salles.insert({ nom: 'TP03', postes: [] })
// Question 2
// On update notre TP01
db.salles.update({ nom: 'TP01' }, {
    $set: { postes: [
        {
            nom: 'TP01-01',
            OS: [
                'Windows 7',
                'Ubutnu 14'
            ]
        },
        {
            nom: 'TP01-02',
            OS: [
                'Windows 7',
                'Ubutnu 14'
            ]
        },
        {
            nom: 'TP01-03',
            OS: [
                'Windows 7',
                'Ubutnu 14'
            ]
        },
        {
            nom: 'TP01-04',
            OS: [
                'Windows 7',
                'Ubutnu 14'
            ]
        },
        {
            nom: 'TP01-05',
            OS: [
                'Windows 7',
                'Ubutnu 14'
            ]
        }
    ]}
})
// Puis on update notre TP02
db.salles.update({ nom: 'TP02' }, {
    $set: { postes: [
        {
            nom: 'TP02-01',
            OS: [
                'Windows 7',
                'Centos 7'
            ]
        },
        {
            nom: 'TP02-02',
            OS: [
                'Windows 7',
                'Centos 7'
            ]
        },
        {
            nom: 'TP02-03',
            OS: [
                'Windows 7',
                'Centos 7'
            ]
        },
        {
            nom: 'TP02-04',
            OS: [
                'Windows 7',
                'Centos 7'
            ]
        },
        {
            nom: 'TP02-05',
            OS: [
                'Windows 7',
                'Centos 7'
            ]
        }
    ]}
})
// On va update notre TP03 en deux temps : on va d'abord ajouter les postes,
// mais avec un OS initialisé sur un tableau vide, sauf pour le premier poste
// Notez que l'ordre des poste n'est pas correct, mais que l'on effectue un tri
// et que l'on limite la taille du tableau
db.salles.update({ nom: 'TP03' }, {
    $push: {
        postes: {
            $each: [
                {
                    nom: 'TP03-01',
                    OS: [
                        'Windows 10',
                        'Ubuntu 14'
                    ]
                },
                {
                    nom: 'TP03-02',
                    OS: []
                },
                {
                    nom: 'TP03-06',
                    OS: []
                },
                {
                    nom: 'TP03-07',
                    OS: []
                },
                {
                    nom: 'TP03-03',
                    OS: []
                },
                {
                    nom: 'TP03-04',
                    OS: []
                },
                {
                    nom: 'TP03-05',
                    OS: []
                },
                {
                    nom: 'TP03-08',
                    OS: []
                },
                {
                    nom: 'TP03-09',
                    OS: []
                },
                {
                    nom: 'TP03-10',
                    OS: []
                }
            ],
            $slice: 5,
            $sort: {
                nom: 1
            }
        }
    }
})
// Une fois nos postes ajoutés, on va utiliser l'opérateur `$[]` afin de placer les OS dans les postes
// Notez l'utilisation de `$addToSet` pour éviter d'insérer les valeurs si elles sont déjà présentes
db.salles.update({ nom: 'TP03' }, {
    $addToSet: {
        'postes.$[].OS': {
            $each: [
                'Windows 10',
                'Ubuntu 14'
            ]
        }
    }
})
// On aurait pu procéder comme ceci :
// db.salles.update({ nom: 'TP03' }, {
//     $addToSet: {
//         'postes.$[poste].OS': {
//             $each: [
//                 'Windows 10',
//                 'Ubuntu 14'
//             ]
//         }
//     }
// }, {
//     arrayFilters: [
//         {
//             'poste.nom': { $ne: 'TP03-01' }
//         }
//     ]
// })
// Question 3
db.salles.update({ nom: 'TP01' }, {
    $set: {
        cours: [
            {
                enseignant: {
                    prenom: 'Denis',
                    nom: 'Desroches'
                },
                groupe: 2,
                matiere: 'Programmation réseau',
                jour: 'mercredi',
                debut: 9,
                fin: 13,
                etudiants: [
                    {
                        prenom: 'Alice',
                        nom: 'Duffet'
                    },
                    {
                        prenom: 'Michèle',
                        nom: 'Roy'
                    },
                    {
                        prenom: 'Alexis',
                        nom: 'Boivin'
                    },
                    {
                        prenom: 'Romain',
                        nom: 'Fluet'
                    },
                    {
                        prenom: 'Vincent',
                        nom: 'Fugère'
                    },
                    {
                        prenom: 'Coralie',
                        nom: 'Briard'
                    }
                ]
            }
        ]
    }
})
```

## Agrégats

MongoDB nous met à disposition un pipeline d'aggrégation très complet. Il va nous permettre de faire de nombreuses tâches de recoupage de données, de filtrage, de calculs et de jointures.

### Map-Reduce

La première façon d'aggréger des données avec MongoDB est l'utilisation de Map-Reduce.

Cette commande va nous permettre d'envoyer des fonctions JavaScript au moteur de MongoDB afin de faire des traitements sur les données.

![Exemple de Map Reduce](https://docs.mongodb.com/manual/_images/map-reduce.bakedsvg.svg)

Bien que très avantageux niveau syntaxique pour un développeur, cette façon de faire reste assez lente sur de très grands volumes de données. C'est pour celà que Mongo a implémenter sa propre pipeline d'aggrégation.

### Pipeline d'aggrégation

Pour les aggrégats, nous allons utiliser la commande `aggregate()` sur nos collections, ou directement sur notre base.

En mongo, un aggrégat est divisé en différentes phases, appelées `stages`, qui consistent chacune en un opération d'aggrégation de données. Il existe actuellement 23 stages différents pour effecuter ces opérations. La liste complète des stages est disponible dans [la documentation](https://docs.mongodb.com/manual/meta/aggregation-quick-reference/).

![Exemple de pipeline d'aggrégation](https://docs.mongodb.com/manual/_images/aggregation-pipeline.bakedsvg.svg)

En plus de ces stages, nous avons à disposition un peu plus de 200 opérateurs, détaillés dans [la documentation](https://docs.mongodb.com/manual/reference/operator/aggregation/). Une partie d'entre eux possède plusieurs définitions selon le type d'objets sur lequel effectuer des traitements, d'autres sont des opérateurs très similaires à tout ceux que nous avons pu voir jusqu'à présent, et une grande partie consistent en des fonctions mathématiques ou de traitements de tableaux.

Dans le cadre de ce cours, nous n'allons par voir tous ces stages et opérateurs de façon exhaustive, cependant, nous allons nous familiariser avec les opérateurs suivants :
- [`$addFields`](https://docs.mongodb.com/manual/reference/operator/aggregation/addFields/#pipe._S_addFields)
- [`$bucket`](https://docs.mongodb.com/manual/reference/operator/aggregation/bucket/#pipe._S_bucket)
- [`$count`](https://docs.mongodb.com/manual/reference/operator/aggregation/count/#pipe._S_count)
- [`$facet`](https://docs.mongodb.com/manual/reference/operator/aggregation/facet/#pipe._S_facet)
- [`$limit`](https://docs.mongodb.com/manual/reference/operator/aggregation/limit/#pipe._S_limit)
- [`$lookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup)
- [`$match`](https://docs.mongodb.com/manual/reference/operator/aggregation/match/#pipe._S_match)
- [`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out)
- [`$project`](https://docs.mongodb.com/manual/reference/operator/aggregation/project/#pipe._S_project)
- [`$skip`](https://docs.mongodb.com/manual/reference/operator/aggregation/skip/#pipe._S_skip)
- [`$sort`](https://docs.mongodb.com/manual/reference/operator/aggregation/sort/#pipe._S_sort)
- [`$sortByCount`](https://docs.mongodb.com/manual/reference/operator/aggregation/sortByCount/#pipe._S_sortByCount)
- [`$unwind`](https://docs.mongodb.com/manual/reference/operator/aggregation/unwind/#pipe._S_unwind)

## Recherches textuelles et par Regex



## Recherche Géospatiale



## Introduction à la mise en place de cluster mongodb



## Pour aller plus loins


