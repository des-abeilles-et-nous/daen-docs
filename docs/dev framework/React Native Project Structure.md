# Recommanded structure for a React Native project.

This structure uses a mix of _features grouping_ and _type grouping_ in order to improve both maintainability and modularity while preserving folders navigation agility.
This is not a fixed structure with hard rules or fixed names, but rather a set of guidelines that can be overidden for practical reasons, when needed.
In particular, folders' depth and folders' granularity are essentially driven by the number of files as well as the future expected complexity of your application.

---

## Rules

**Rule 1**: `Common, shared or global files are grouped by type (e.g images, fonts, components, styles, etc) and/or domains (e.g. apis, networking, database, etc)`

**Rule 2**: `Otherwise, files, even heterogeneous, are grouped by feature, with just as many nested levels as needed (i.e. bundle files within folders only when it makes sense)`

**Rule 3**: `You may use directory names or index.js for your imports. It's a question of personal preference`

---

## Explanations

-   **app** : This is the root directory for our app. Most of the time, _you can skip it and use instead the react native project directory_, but this additional level may prove useful when adding tests in the project
-   **./assets** : This is where all the static resources like images, videos, etc will go in. Subdirectories like fonts, icons, images, videos, etc can be used when there are many files
-   **./config** : This is where your global configurations for the app will go in. For example, your environment specific config for dev and prod, etc. Not mandatory if you have only one config.js file that can rest directly in the root directory, but useful if you have multiple configuration files. Configuration files tighly linked to one component (for instance selection.json for Icomoon icons) should be located in this component directory
-   **./features** : The main concept of this project structure. This directory will hold all the smart components. Smart components are those components which contain business logic and states in them. Usually, in classical RN project organisation, this will be a "screens" or "pages" directory. And it's ok, as long as your app is simple.
-   **./common** : It is where we place functions, utilities and components that are used by many features. Things like atomic components, wrappers, quick fixes functions, networking stuff,etc... not belonging to a specific feature. For instance:
    -   **./common/styles** : This is where your global styles, themes, and mixins will go.
    -   **./common/apis** : This is where all the functional interfaces to services (rest or other) will go. They shall be grouped by type and/or destination (e.g. backend/firebase, backend/amplify, weather/openweather, map/mapbox, etc).
    -   **./common/components** : This folder is for reusable (shared) or global components. According to React naming conventions, the component's first letter should be uppercase. These components are "dumb" as they will only do layouting and won't contain any states or business logic. All the data to these components will be passed in as props.
    -   **./common/utils** : This is where all the utility functions such as math, log, networking, etc will go. If there is a lot of files, you can group them by domain using subdirectories (e.g. api, log, math, etc)

<!--
- **app/routes** : This is where we will keep all our app's routing logic. This will contain the map between the pages(smart components) and the routes.
- **app/redux** : This will contain all our redux state management files like actions, reducers, store config, thunks etc.
-->

## Exemple

```
ROOT
├───assets
│   ├───fonts
│   │   ├───DaenIcons
│   │   └───QuickSand
│   ├───images
│   │   ├───backgrounds
│   │   └───faces
│   └───sounds
├───common
│   ├───apis
│   │   └───backend
│   │       ├───firebase
│   │       └───amplify
│   ├───components
│   │   └───DaenComponents
│   │       ├───DaenButton
│   │       ├───DaenIcon
│   │       └───DaenImagePicker
│   ├───themes
│   └───utils
│        ├───firebase
│        └───amplify
├───config
└───screens
    ├───screen1
    │   ├───index.js
    │   ├───style.js
    │   └───utils.js
    ├───screen2.js
    └───screen3
        ├───screen3.js
        ├───style.js
        └───utils.js
```

Now, you may want to know how to manage imports paths with such a structure? No worries, it's explained [here](React%20Native%20Import%20Paths.md)

_Notes:_

-   *https://www.freecodecamp.org/news/how-to-structure-your-project-and-manage-static-resources-in-react-native-6f4cfc947d92/*
-   *https://medium.com/@alexmngn/how-to-better-organize-your-react-applications-2fd3ea1920f1*
-   *https://alligator.io/react/index-js-public-interfaces/*
-   *https://jaysoo.ca/2016/02/28/organizing-redux-application/*
