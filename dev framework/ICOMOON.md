# ICOMOON GUIDE

## Produire des icones SVG propres pour utilisation avec ICOMOON

> _Note: Procédure manuelle, il y a des alternatives CLI/BATCH_

### Obligatoire

- Load svg original sous inskape
- Select all
- Object > Ungroup
- Path > Stroke to Path
- Path > Union
- Path > Combine
- File > Cleanup document
- Save svg (plain ou inkskape, peu importe.

    **Nomenclature des fichiers**

  - N'utiliser que des minuscules
        Donner des noms anglais (ou au moins ne pas mélanger les langues) car en général l'anglais est plus concis.
  - Donner les noms les plus courts possibles. Inutile de préfixer en daen ou icon, puisque dans notre cas on créera un composant DaenIcon qui porte déjà ces notions et qu'on invoquera ainsi `<DaenIcon name="home"/>`. On comprend bien que c'est une icone spécifique à DAEN :smirk:
  - Dans le cas où il faut plusieur mots, utiliser le - (dash) comme séparateur, et ordonner par groupement d'icone Ex: 'chevron-left' et non 'left-chevron'
  - _Je sais, ça parait rigide comme ça, mais ça paie sur le long terme._

### Optionnel

Traiter les avec [SVGOMG](https://jakearchibald.github.io/svgomg/)

Options par défaut + Remove Style Elements + Remove Script Elements

## Produire une font avec [ICOMOON](https://icomoon.io/app/#/select)

> _Note: Procédure manuelle, il y a des alternatives CLI/BATCH, des extensions chrome, etc_

### Pour créer un nouveau set dans IcoMoon

- Main Hamburger (Left) > New Empty Set
- Set Hamburger ( Right) > Properties > Set Metadata et donner au moins un nom au set (Ex: DaenIcons)

### Pour éditer un set dans IcoMoon

- Importer via top menue > Import Icons le fichier selection.json récupéré lors du téléchargment de la police (voir plus bas)

### Pour ajouter des SVG dans un set IcoMoon

- Ajouter les svg au set via top menu > import icons ou drag'n drop dans le set
- Pour homogénéiser en monochrome Set Hamburger ( Right) > Properties > Remove all colors

### Pour Créer une police à partir d'un set IcoMoon, et la télécharger

- Sélectionner les icones à inclure (individuellement ou via Set Hamburger ( Right) > Select all
- Cliquer sur Generate Font
- Configurer la sortie via le menu Préférences ou l'engrenage à droite du bouton download font et
  - Donner un nom à la police (Ex: 'DaenIcons')
  - Choisir un prefixe (Ex: 'daen-icon-'). C'est uniquement pour une utilisation HTML, où l'utilisation du nom court seul dans une classe d'élément n'est pas assez descriptif en dehors de tout contexte si la font family est perdue au fond d'un CSS.
- Cliquer sur le bouton Download

### Pour utiliser la police dans Expo

- Copier le fichier /fonts/DaenIcons.ttf du zip vers l'endroit où sont les assets fonts (en général quelque part dans assets/fonts :smirk:)
- Copier le fichier selection.json quelque part où s'est cohérent, le plus propre étant avec la police, dans un sous repertoire de assets/fonts (donc assets/fonts/DaenIcon)\*\*
- Préloader la police de la même façon que les autres polices
- Ecrire le [code pour utiliser la police](https://docs.expo.io/versions/latest/guides/icons/)

## Exemple de composant

```javascript
import * as React from 'react'
import { createIconSetFromIcoMoon } from '@expo/vector-icons'
import icoMoonConfig from 'fonts/DaenIcons/selection.json'

// On utilise ici @expo/vector-icons, https://docs.expo.io/versions/latest/guides/icons/
// qui est un wrapper de react-native-vector-icons
// cf: https://github.com/oblador/react-native-vector-icons/blob/master/README.md
//const expoAssetId = require("fonts/DaenIcons/daen-icons.ttf");
//
// On peut l'invoquer de plusieurs façons:
//
// 1) En effectuant le require ici
//  const expoAssetId = require("assets/fonts/DaenIcons.ttf");
//  export default createIconSetFromIcoMoon(icoMoonConfig, 'DaenIcons', expoAssetId);
//
//  Ce qui est équivalent à faire:
//  export default createIconSetFromIcoMoon(icoMoonConfig, 'DaenIcons', 'fonts/DaenIcons/DaenIcons.ttf');
//
// 2) Mais comme on a préloadé la police, on peut se contenter de la font family
//  export default createIconSetFromIcoMoon(icoMoonConfig, 'DaenIcons');
//
// 3) Et si on a une bonne nomenclature dans le fichier icomoon selection.json parce qu'on a bien suivi
// ce guide, on peut faire encore plus court:

export default createIconSetFromIcoMoon(icoMoonConfig)
```

### Notes

- On peut aussi créer des projets, qui sont sauvés localement sur le PC tant qu'on ne nettoie pas le cache, ou dans le cloud pour les utilisateurs premium.
