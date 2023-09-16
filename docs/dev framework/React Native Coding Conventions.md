## CODE
### Conventions de nommage
- **camelCase** pour les fonctions et méthodes (ex: *getCustomerName*, *isNameValid*, etc)
- Les méthodes à usage strictement interne à la classe (private) sont préfixées par un underscore (ex: *_doSomethingInternal*, etc)
- **PascalCase** pour les classes (ex: *MyPrettyComponent*, *BigFatButton*, etc)
- **UPPER_CASE_SNAKE_CASE** pour les constantes (ex: *SERVER_URL*, *PRIMARY_LOG_DIRECTORY*, etc)

- Voir aussi: https://github.com/airbnb/javascript

### Formatage
- Formatage plugin VSCode **Prettier** options par défaut, idéalement automatisé pré-commit, l'idée étant d'éviter les delta diff causés par du pur formattage.

### Data Binding
- Jamais pour les fonctions, utiliser systématiquement la notation fléchée (qui évite le switch de contexte)

## STRUCTURE PROJET
### Convention de nommage
- **camelCase** pour les fichiers qui ne sont pas des composants, i.e. n'exportent pas une classe par défaut mais des fonctions
- **PascalCase** pour les fichiers qui implémentent un ou des composants (exception: index.js) et pour les répertoires contenant des composants autoporteurs ou groupant des composants autoporteurs 
- **lowercase** et un seul mot pour les répertoires et sous répertoires globaux (ex: library/components, features/screens, assets/fonts, etc
- **camelCase** pour les autres répertoires

