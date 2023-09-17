# daen Platform Data Models

## Overview

daen Platform Data Models are mainly inspired by underlying NoSQL data storage components. Hence the platform manages two storage assets:

- **rtdb**: Firebase realtime database. A NoSQL, `JSON` storage on Google Cloud Platform. Each node of the JSON tree is requestable, search is limited to shallow parameters match requests, database best used for frequent read with low sized payload.
- **fs**: Firestore is a JSON documents collections database. Here documents are organised in _collections_, i.e. set of `JSON` documents. Search are done at collection (or subcollection) level based on index built upon document paramaters. Write capabilities are limited with a maximum of 5 write requests/s on a document as this database is optimised for document retrieval rather than manipulation.

daen platform Data Model is designed to take the best profit of these two assets:

- **rtbd** is used to handle frequently changing and transitive information linked to platform activities
- **fs** is dedicated to Business Objects storage

## Notation

We use a URL like notation to define storage location and the type of stored data at this location.

```text
<fs|rtbd>:(optional){collection<type>}/path/param
```

## Objects meta types

Since most of platform systems are `JSON` friendly, including the data storage, data structure will be presented in this format.

### POI - `fs:{POIs}/`

```JSON
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
    "status": <"handled"|"disbelieved"|"locked"|"archived">,
    "updated_t": <number>,
}
```

### Map tile content - `fs:{poi_tiles}/`

```JSON
"tile":{
  "_clotho": <bool>,
  "a": <integer>,
  "id": <string>,
  "poi_lists":{
    <string:typeofPOI>:[<POIview>]
  },
  "updated_t": <integer>
}

<POIview>: {
  "uid": <string>,
  "code": <string>,
  "opacity": <number>,
  "lat": <number>,
  "lon": <number>,
  "status": <"handled"|"disbelieved"|"locked"|"archived">,
}
```

### User feedback - `rtdb:/feedbacks/`

```JSON
"feedback": {
"at" : 1622556370380,
"type" : "PLUS|HANDLED|THUMBUP",
"owner" : "...",
"poi" : "...",
"pos" : [ 48.8350309, 2.3290567 ],
"status" : "new",
}
```

### Action item - `rtdb:/(buffer|tasks)/`

```JSON
"action": {
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
        ...<POI>,
        "logs" : [{"at": <number>,
          "detail": <string>,
          "type": <string>,
          "who": <string>}]
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
