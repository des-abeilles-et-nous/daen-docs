# Gestion des environnements ![GDE](https://img.shields.io/badge/Status-In_Progress-orange?style=plastic)

L'objectif de ce document est de présenter l'ensemble des pré-requis et opérations nécessaires au déploiement
d'un nouvel environnement logiciel (production ou dev/test) pour l'application `daenscout`

## Environnements

### Typologie et usages des environnements

- LOCAL : environnement de développement propre à l'utilisateur, ne sera pas décrit dans ce document
- DEV : environnement de développement, utilisé par les équipes de dev pour faire les tests d'intégration et les tests usines en phase de développement de l'application
- STAGING : préproduction réalisation des tests utilisateur et/ou les pilotes
- LIVE/PROD : environnement cible utilisé par les utilisateurs en conditions réelles,

### Sous-systèmes concernés

- Firebase
- Facebook (Auth)
- Google (Auth)
- Google Maps
- Sentry

### Mapping environnements / sous-systèmes

| `<env>` | local files | Git           | Firebase    | Google | Facebook              | sentry           |
| ------- | ----------- | ------------- | ----------- | ------ | --------------------- | ---------------- |
| dev     | env/dev/\*  | dev\*         | BSC-DEV     | ---    | DA&N (dev)            | #daen/daen-scout |
| live    | env/live/\* | dev\*, master | DSC-LIVE-EU | ---    | da&n - observ'acteurs | #daen/daen-scout |

## Mise en oeuvre d'un environnement

### Firebase

#### Création du projet

Une fois le projet `<project_name>` crée et configuré sur Firebase (modop à rédiger), aller sur les paramètres du projet et ajouter une app android avec les paramètres suivants :

- Android package name : le nom du package défini dans `app.json`
- appnickname : reprendre le `slug` utilisé dans le `app.json` (facultatif)

Récupérer les informations de configuration depuis la console Firebase qui doivent ressembler à qq chose comme ça :

```JS
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "cwcsqdqsxsxqsxq",
  authDomain: "nomduprojet.firebaseapp.com",
  databaseURL: "https://nomduprojet.firebaseio.com",
  projectId: "nomduprojet",
  storageBucket: "nomduprojet.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:123456789123456789",
  measurementId: "G-123456789"
};
```

#### Règles d'accès

Mettre à jour les règles d'accès aux documents de la base Firestore utilisée par l'app. Le fichier de règles doit être stocké dans le repository `fb-daen-workers`

#### Cloud Functions

Configurer si nécessaire les Cloud Functions (actuellement dans le repository `fb-daen-workers`) et les charger dans le projet

```sh
firebase init
firebase use <nom de la base>
firebase deploy
```

### Facebook (Auth)

#### Application Facebook

créer une application Facebook (depuis votre compte facebook developper). Cette application est définie par les paramètres suivants qui doivent être réintégrés dans la config de build (native) via la lib https://github.com/thebergamo/react-native-fbsdk-next dont la config est définie dans l'`app.json` ou l'`app.config.js` à partir des variables de l'environnement cible

```js
{
  "expo": {
    "plugins": [
      [
        "react-native-fbsdk-next",
        {
          "appID": process.env.FACEBOOK_APPID,
          "clientToken": process.env.FACEBOOK_CLIENT_TOKEN,
          "displayName": process.env.FACEBOOK_DISPLAYNAME,
          "scheme": process.env.FACEBOOK_SCHEME,
          "advertiserIDCollectionEnabled": false,
          "autoLogAppEventsEnabled": false,
          "isAutoInitEnabled": true,
          "iosUserTrackingPermission": "This identifier will be used to deliver personalized ads to you."
        }
      ]
    ]
  }
}
```

| envar                 | local .env | eas.json | app.json | expo servers |
| --------------------- | ---------- | -------- | -------- | ------------ |
| FACEBOOK_APPID        | X          |          |          |              |
| FACEBOOK_DISPLAYNAME  | X          |          |          |              |
| FACEBOOK_SCHEME       | X          |          |          |              |
| FACEBOOK_CLIENT_TOKEN | X          |          |          |              |

Lui ajouter un service Facebook login, configurer le tout et récupérer les infos de config
Passer, si nécessaire, l'app Facebook en mode `live` (après les tests de préfèrence)

#### ajout Auth à Firebase

Depuis la console Firebase, ajouter le fournisseur d'authentification Facebook et paramétrer avec les infos de l'app nouvellement créée

#### ajout signature APK

Une fois l'application déployée sur le Play Store **il est nécessaire de renseigner la clef de l'APK signée par Google (en plus de celle de l'apk générée par Expo)**. Pour ce faire, aller sur la console Google Play Store [Key Management Google](https://play.google.com/console/u/4/developers/5360587266525515968/app/4975039708111061731/keymanagement) et récupérer l'empreinte SHA-1 du certificat de signature de l'application, la convertir en base64 avec [base64 converter](http://tomeko.net/online_tools/hex_to_base64.php) ou un autre outil et la réinsérer dans la config de l'app sur la console Facebook.

![warning Facebook naming](https://img.shields.io/badge/WARNING-naming-critical) **NE PAS METTRE DE CARACTÈRES SPÉCIAUX DANS LES PARAMÉTRES du `app.json`**, ca fait méchamment planter le build expo (c'est du vécu 😕)

### onfiguration Google (Auth)

cf [doc expo FCM](https://docs.expo.io/push-notifications/using-fcm/). En résumé il faut :

- ajouter une app Android (et iOS si déploiement en mode standalone sur iOS) à Firebase
- récupérer le fichier `google-services.json` et le référencer dans le `app.json` sous la clef
- ajouter le fingerprint des applications et ajouter les signatures SHA-1 des apk/aab autorisés (cf. procédure facebook, `expo fetch:android:hashes` en standalone ou via la console Google en mode 'store')

```js
{
  ...
  "android": {
    "googleServicesFile": "./google-services.json",
    ...
  }
}
```

- aller sur la console Google Cloud Platform pour et configurer/vérifier que les API Credentials sont OK au niveau Key et restrictions d'usage :
  - pour les apps mobiles API Firebase Installation, Firebase Cloud Messaging API et Google Maps (bien penser à activer l'API Maps SDK for Android et/ou iOS au préalable)
  - pour les apps web API Firebase Installation et Identity Toolkit

### Step 4 : configuration Google Maps

à rédiger

### Step 5 : configuration Sentry

Sentry est configuré selon les docs expo et sentry. Les valeurs de référence sont dispo sur la console Sentry (`Settings...`) et la config est définie dans l'`app.json` ou l'`app.config.js` à partir des variables de l'environnement cible

```js
extra: {
 sentry_dsn: process.env.SENTRY_DSN,
},
plugins: ['sentry-expo'],
hooks: {
postPublish: [{
   file: 'sentry-expo/upload-sourcemaps',
  config: {
   organization: process.env.SENTRY_ORG,
   project: process.env.SENTRY_PROJECT,
   authToken: process.env.SENTRY_AUTH_TOKEN,
     },
  },
  ],
},
```

| envar          | local .env | eas.json | app.json | expo servers |
| -------------- | ---------- | -------- | -------- | ------------ |
| SENTRY_DSN     | X          |          |          |              |
| SENTRY_ORG     | X          |          |          |              |
| SENTRY_PROJECT | X          |          |          |              |
| SENTRY_AUTH    | X          |          |          |              |

### step 6 : configuration EAS/Google/Apple ![configuration EGA](https://img.shields.io/badge/Status-TO_BE_VALIDATED-orange?style=plastic)

Pas grand chose à faire, c'est expo/EAS qui se charge de tout automatiser au moment du build. Il suffit de renseigner les champs demandés.

## Gestion multienvironnements

La configuration de l'intégration est nécessaire afin de permettre à l'application d'accéder aux différents fournisseurs de services. Ces éléments de configuration sont habituellement transmis à l'application par les fichiers `app.json` et `app.config.js`. L'utilisation de `app.config.js` permet notamment la configuration dynamique de ces paramètres afin d'apporter de la souplesse dans la gestion d'environnements multiples.

### Définitions

- `config`: la configuration applicative (les `app.json`,`app.config.js` et `.env`)
- `target` : la cible d'exécution qui permet de définir quelles variables d'environnement sont appelées lors du build. Dans l'ordre : `$DAEN_TARGET<nom de la variable> > <nom de la variable>`, i.e. si une variable d'environnement existe préfixée par `$DAEN_TARGET` alors elle est lue en lieu et place de la variable d'environnement non préfixée.

### Fichiers de configuration

- `app.json` : paramètres globaux de l'application expo (nom de l'app, du package, etc.) invariants quelque soit l'environnement,
- `app.config.js` : commandes de post traitement de app.json avant passage des paramètres à l'app
- `.env` : variables d'environnements (clefs d'API, URL, etc...)
- `.env.dist` : fichier modèle de `.env`
- `buildconfig/<target>/google-services.json` : paramètres pour l'authentification Google sur android
- `buildconfig/<target>/GoogleService-info.plist` : paramètres pour l'authentification Google sur iOS

### Configuration de l'environnement : fichier `.env`

Le paramétrage de l'environnement en cours se fait, pour le cas le plus simple, en copiant le fichier `.env.dist` en un fichier `.env`.

## Selection la cible de build

La `config` applicative est générée dynamiquement grâce aux variables d'environnement sélectionnées en fonction de la `target` ou une valeur par défaut, i.e. `process.env.$DAEN_TARGET_VARIABLE ?? process.env.VARIABLE`.

Les valeurs par défaut sont définies dans le `.env` et le pendant avec une `target` non `dev` sont déclarées comme secrets sur les serveurs expo en charge du build.

`eas.json` permet de configurer :

- les variables d'environnement par défaut (i.e. les mêmes que celle du `.env.dist`),
- `DAEN_TARGET` pour la sélection des API
- la nature et l'usage de l'artefact en sortie de build :
  - `internal` pour un déploiement sur des terminaux connus (développeurs, utilisateurs interne)
  - `store` pour le déploiement d'une app signée et validée par l'éditeur depuis un store. A noter que dans le cas android, l'artefact peut être soit un `aab`, soit un `apk`, auquel cas l'installation peut être locale.

Les différents `profile` d'`eas.json` permettent de configurer les cibles en fonction des besoins des équipes dev et produit.
