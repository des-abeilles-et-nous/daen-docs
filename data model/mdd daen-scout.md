# Modèles de données daen-scout

L'application daen-scout repose sur des environnements de stockage et d'exécution de traitements dans le cloud. A date c'est la solution Firebase de Google qui est utilisée mais potentiellement n'importe quel environnement NoSQL (pour pouvoir héberger nos données) pourrait convenir.

Les données sont stockées dans deux types de bases de données différentes :

- une base de données NOSQL 'temps réel' (firebase realtime database => **rtdb**) pour la gestion des interactions avec les utilisateurs
- une base de données NOSQL 'orientée documents' (firestore => **fs**) pour le stockage des données persistantes liées aux Points d'Intérêts

L'intéret de ce double stockage est de pouvoir profiter du meilleur des deux mondes tout en minimisant les couts opérationnels. Toutes les données auraient pu être stockées dans la même base de données mais, au prix d'un léger accroissement de la complexité de BUILD, ce double stockage s'avère plus performant en RUN :

- **rtdb** est un arbre `JSON` qui peut être requêté au niveau de chacun de ses noeuds et dont la tarification est liée au volume de données transférée. Cette structure favorise les requêtages fréquents de données de faible volumétrie (par exemple des retours utilisateurs)
- **fs** est une collection de documents `JSON` qui sont requêtés soit par document soit par collection/sous-collection de documents et dont la tarification est liée au nombre de lecture/écriture. Cette stucture est donc optimisée pour le stockage de gros volumes peu (500 écritures/secondes ca reste très relatif comme peu 😉) sollicités.

Le modèle de données **`daen-scout`** tire partie de cette dualité pour optimiser les coûts et les performances de l'application.

## Objets métier

l'app **`daen-scout`** manipule différents type d'objets persistant en base. L'une des forces des bases NOSQL est leur capacité à stocker _tel quel_ des structures JSON / objets Javascript. Les types d'objets métiers gérés par firebase sont donc les mêmes que ceux utilisés par l'application, à savoir :

```JS
"POI": {
    "uid": <string>,
    "code": <string>,
    "opacity": <number>,
    "lat": <number>,
    "lon": <number>,
    "alt": <number>,
    "desc": <string>,
    "plus": <number>,
    "notseen": <number>,
    "tup": <number>,
    "created_t": <number>,
    "creator_id": <string>,
    "creator_pseudo": <string>,
    "status": "handled|disbelieved|locked|archived",
    "updated_t": <number>,
}

"POIview": {
    "uid": <string>,
    "code": <string>,
    "opacity": <number>,
    "lat": <number>,
    "lon": <number>,
    "status": "handled|disbelieved|locked|archived",
  }

"feedback": {
  "at" : 1622556370380,
  "type" : "PLUS|HANDLED|THUMBUP",
  "owner" : "...",
  "poi" : "...",
  "pos" : [ 48.8350309, 2.3290567 ],
  "status" : "new",
}

"buffer": {
  "at" : 1623599131240,
  "worker" : "...",
  "options" : {
    "opt1" : "..."
  },
  "owner" : "...",
  "status" : "new|running|complete|error|archived",
}
```

## Modèle de stockage firebase realtime database

l'arbre de stockage est structuré comme suit :

```JSON
{
  "buffer" : {
    $buffered_uuid : {
      "at" : 1623599131240,
      "worker" : "...",
      "options" : {
        "opt1" : "..."
      },
      "owner" : "...",
      "status" : "new|running|complete|error|archived",
    }
  },
  "feedbacks" : {
    $feedback_uuid : {
      "at" : 1622556370380,
      "type" : "PLUS|HANDLED|THUMBUP",
      "owner" : "...",
      "poi" : "...",
      "pos" : [ 48.8350309, 2.3290567 ],
      "status" : "new",
    },
  },
  "logs" : {
    $status: {
      $task_uuid : {
      "at" : 1623162848446,
      "status" : "new|running|complete|error|archived",
      "worker" : "..."
      },
    }
  },
  "subs" :{
    $subs_uuid : {
      "userid": "userId",
      "feedid": "feedId",
      "status": "new|notified|read|archived",
      "at": 1623162848446,
    }
  },
  "tasks" : {
    $task_uuid : {
      "at" : 1623261317191,
      "status" : "new|running|complete|error|archived",
      "worker" : "cleaner",
      "options" : {
        "opt1" : "..."
      },
    }
  }
}

```

(to be continued)

## Modèle de données firestore

Cette base est organisée sous forme de collections de documents, chaque document étant un arbre JSON qui peut, outre ses propres champs, aussi inclure une ou plusieurs sous-collections de documents.

```JSON
{
  "__collections__": {
    "poi_tile" : {
      "tileid" : {
        "a" : 1,
        "poi_lists" : {
          "HORNET" : [{<POIview>}],
          "BLOOMING" : [{<POIview>}],
          "SWARM" : [{<POIview>}],
          "NEST" : [{<POIview>}],
        },
        "updated_t": 1623599131929,
        "_clotho" : true | false
      },
    },
    "POIs" : {
      "poiUid" : {
        <POI>,
        "logs" : [{"at": <number>, "detail": <string>, "type": <string>, "who": <string>}]
        },
    },
    "sequences" : {

    },
    "tile_ids" : {

    },
    "users" : {

    },
  }
}
```

(to be continued)
