# Projet d'analyse des contenus de la plateforme Open Data de Côte d'Ivoire

## Documents clés

* spreadsheet d'analyse: https://docs.google.com/spreadsheets/d/1reb0uhMsQlQCVJsv3MHdi8AOlJ7BhmFQz1Sov9NvIfc/edit?usp=sharing
* document de notes: https://github.com/clombion/open-data-portail-ci/blob/master/notes-data-cote-divoire.md


## Objectifs

1. Récupérer les infos de publication de tous les jeux de données de le plateforme
2. Faire une analyse simple de ces infos afin de faire ressortir des points clés

## TO DO

- [x] récupérer les données via l'API de la plateforme
- [x] Faire l'analyse des données récupérées sous forme de CSV

## Notes

### Bon à savoir: code

* Bien mettre les crochets de l'array autour des éléments du csv dans le code jq, et non autour de toute l'expression
     * oui: jq 'data[] | [.element1, .element2] | @csv'
     * non: jq '[data[] | .element1, .element2] | @csv'
     Le second exemple ne produit pas un CSV correctement formatté

### Bon à savoir: données

* Les données de l'API n'incluent pas le nombre de vues de chaque jeu de données

## Actions

DEFINIR

voir objectifs ci-dessus

FIND

* test de l'API sur data.gouv.ci --> Echec, les données ne sont pas disponible
* Contact du responsable du site, qui indique que le site est sur un serveur temporaire, ce qui rend l'API indisponible. Une alternative est fournie pour récupérer les données via l'API.
* test de l'API avec 

`curl http://data.gouv.ci/opendata/api/datasets`

RECUPERER & VERIFIER

* utilisation de jq pour avoir une vision d'ensemble du jeu de données

`jq '[path(..)|map(if type=="number" then "[]" else tostring end)|join(".")|split(".[]")|join("[]")]|unique|map("."+.)|.[]'`

(enregistrée sous le raccourci clavier jqsummary" pour moi)

ce qui donne:

`"."
".data"
".data[]"
".data[].created_at"
".data[].description"
".data[].id"
".data[].last_modifie"
".data[].organization"
".data[].organization_id"
".data[].slug"
".data[].tags"
".data[].title"`


* j'ai ensuite: 
     1. regardé le premier objet de la liste 
     2. compté le nombre total d'objets pour voir s'ils correspondaient à l'élément id (non, mais c'est courant dans ce genre de système, car des éléments ont pu être supprimés sans que leur id n'ait été réutilisée par la base de donnée),
     3. exporté toute la liste dans un csv

1. curl http://data.gouv.ci/opendata/api/datasets | jq '.data[0]''
2. curl http://data.gouv.ci/opendata/api/datasets | jq '.data | length'
3. curl -s http://data.gouv.ci/opendata/api/datasets | jq -r '.data[] | [.id, .created_at, .description, .last_modifie, .organization, .organization_id, .slug, .tags, .title] | @csv' > data-portail-ci.csv

* Le fichier est ensuite importé sur Google Spreadsheets

NETTOYER

* (toilettage) Rajout des en-têtes de colonnes dans Google Spreadsheets
* (édition) Rajout d'une colonne (annee_mois) permettant de faire un analyse de l'évolution mois par mois en utilisant la formule `LEFT(B2,7)` appliquée à toute la colonne

ANALYSER

Utilisation d'un tableau croisé dynamique pour voir à la fois :

     * les administrations qui ont partagées des données, triées par nombre de jeux de données
     * le nombre de jeux de données partagé par chaque administration au fil du temps

Cela donne:

     * en lignes: `organizations`
     * en colonnes: `annee_mois` 
     * en valeurs: `id (COUNTA)`

On observe qu'une seule administration (Institut National de la Statistique) a fait un vrai travail d'ouverture des données dans le temps, les autres administrations ont sont restées à des actions ponctuelles, probablement motivées par l'organisation d'un atelier de l'open data du CICG.

Evidemment, il n'est pas étonnant que l'institut national de la statistique soit l'agence la plus active, car elle a un important nombre de fichiers prêt à partager, et une forte culture de la donnée. Un retour d'expérience de leur équipe peut cependant être intéressant.

PRESENTER

Un graphique en ligne est utilisé afin de montrer visuellement l'engagement ponctuel ou dans le temps des différentes agences.

     * axe X: `annee_mois`
     * axe Y: nombre de jeux de données (`COUNTA de id`)



