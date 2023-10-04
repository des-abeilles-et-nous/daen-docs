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

## Object Storage Management systems

### Firebase realtime database - `rtdb:/`

`rtdb` storage is one unique JSON tree which is organized as follow:

```JSON
{
  "buffer" : {        //queue of worker to be launched instantaneously
    "$buffered_uuid": <action>
  },
  "tasks" : {         //queue of worker to be launched on schedule, scanned every minute
    "$task_uuid" : <action>
  },
  "logs" : {          //queue execution residus for postmortem analysis or worker relaunch
    "$status": {
      "$task_uuid" : <action>
    }
  },
  "feedbacks" : {     //map of user feedbacks awaiting processing
    "$feedback_uuid": <feedback>
  },
  "subs" :{           //map of all users' subscriptions to event feeders
    "$subs_uuid" : <subs>
  },
}

```

## Firestore - `fs:{}`

This storage is built as a collections of JSON documents or subcollection. Each node of the collectionhierarchy is searchable and can be indexed to improved search capabilities and performances. We structures daen collections as follows:

```JSON
"__fs__": {
  "tiles_view": {   //Collection of tiles with viewable POIs params
    "$tileid": <tile>
  },
  "POIs" : {        //Collection of active, i.e. not archived, POIs
    "$poiUid": <POI>,
  },
  "POIs_attic" : {  //Collection of archived POIs
    "$poiUid": <POI>,
  },
  "sequences" : {   //Last auto numbered value given for each value category
    "$seq_name": {"count":<number>, "prefix":<string>},
  },
  "poi_tile" : {    //Old tiled view tile model, kept for upward compatibility (deprecated)
    "$tileid": <deprecated>
  },
  "tile_ids" : {    //(deprecated) workaround to keep track of tiles with viewable POIs
    "$tileid": <deprecated>
  },
  "users": {
    "$uuid":  {
      (to be described)
    },
  },
}
```

## Objects meta types

Since most of platform systems are `JSON` friendly, including the data storage, data structure will be presented in this format.

### POI - `fs:{POIs}/`

```JSON
"POI": {
    "uid": <string>       //unique POI id
    "code": <string>,     //category/type/specy coding key format
    "opacity": <number>,  //(deprecated) moved to <POIview>
    "lat": <number>,      //latitude of POI report
    "lon": <number>,      //longitude of POI report
    "alt": <number>,      //altitude of POI report
    "desc": <string>,     //additional information on POI
    "plus": <number>,     //num of "also seen" feedbacks received
    "notseen": <number>,  //num of "not seen" feedbacks received
    "tup": <number>,      //num of "thumbup" feedbacks received
    "created_t": <number>,      //creation timestamp
    "creator_id": <string>,     //POI creator user unique id (uuid)
    "creator_pseudo": <string>, //creator pseudo (autoupdated from creator_id)
    "status": <"handled"|"disbelieved"|"locked"|"archived">, //visible status
    "m_t": <number>,      //last modified timestamp
    "views": [<string>],  //array of views POI appears in (autoupdated)
}
```

### Map tile content - `fs:{tiled_views}/`

```JSON
"<Tile>":{
  "_clotho": <bool>,    //(internal) true if next tile refresh task is scheduled
  "a": <integer>,       //size in degree of arc of a square tile vertex
  "id": <string>,       //tile unique id
  "poi_lists":{
    <string:category>:[<POIview>]
  },
  "updated_t": <integer>
}
```

### POI viewable attributes - `fs:{poi_tiles}/tiled_views/{category}[]`

```JSON
"<POIview>": {      //See <POI> for attributes explanation
  "uid": <string>,
  "code": <string>,
  "opacity": <number>,
  "lat": <number>,
  "lon": <number>,
  "status": <"handled"|"disbelieved"|"locked"|"archived">,
}
```

### User profile - `fs:{users}`

```JSON
"<User>": {
  alerts:{
      "blooming":{
      "status": (string),
      "subsid": <string>,
    },
  hornet:{
    "status": (string),
    "subsid": <string>,
    },
  nest:{
    "status": (string),
    "subsid": <string>,
    },
  },
  "bkpr":<(string)>,
  "created_t": <(timestamp)>,
  "display_name": <(string)>,
  "email":<(string)>,
  "isBeekeeper": <(Boolean)>,
  "isHunter":<(Boolean)>,
  "modified_t":<(timestamp)>,
  "num_id":<number>,
  "picture":<string>,
  "pseudo": <(string)>,
  "public_id":<(string)>,
  "refLocation":{
  "isoCountryCode":<string>,
  "latitude": <number>,
  "longitude": <number>,
  "postalCode": <string>,},
  "uid": <string>,
  "..."
}
```

### User feedback - `rtdb:/feedbacks/`

```JSON
"<feedback>": {
  "at" : <number>,      //timestamp of feedback action
  "type" : "PLUS|HANDLED|NOTSEEN|THUMBUP",    //feedback type
  "owner" : <string>,   //feeback author uuid
  "poi" : <string>,     //POI uid which is receiving the feeback
  "pos" : [ 48.8350309, 2.3290567 ],  //[lat,lon] of author when sending feedback
  "status" : "new",     //for future use must be set to "new"
}
```

### Action item - `rtdb:/(buffer|tasks)/`

```JSON
"<action>": {
  "at" :  <number>,               //timestamp to schedule action launch, 0 is instant launch
  "worker" : <string:worker>,     //code name of the worker to launch
  "options" : {<string:option>:*}, //options object specific to worker
  "owner" : <string>,             //action owner uuid,
  "status" : "new|running|complete|error|archived",   //action status according to platform scheduling
}

```

### Alert feed subscription item - `rtdb:/subs/`

```JSON
"subs" : {
  "uuid": <string>,             //user uid owing the subscription
  "feeds": [<string>],          //array of feed uid which user listens to
  "status": "active|new|notified|read|archived", //subscription status
  "trigger": <string>,          //POI event type
  "mode": "all|firstof|...",    //type of triggering mode
  "opts": <string>,             //option of the trigger
  "filter": <number>,           //param value used to fast filter event of the feeder
  "fopts": {<string:option>:*}, //options object specific to filter of this feeder
  "logs": [<event>],            //array of POI events which had triggered the subscription
  "name": <string>,             //code for alert naming label
}
```
