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
  - [Modification de documents](#modification-de-documents)
  - [Suppression de documents](#suppression-de-documents)

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
db.profs.insert({"nom": "Baumann", "prenom": "Maxime", "statut": "Vacataire"})
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
db.profs.insert({"nom": "Baumann", "prenom": "Maxime", "statut": "Vacataire"})
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

TODO

### Modification de documents

TODO

### Suppression de documents

TODO
