---
marp: true
size: 16:9
theme: default
paginate: false
backgroundImage: url('img/background/fond-blanc.avif')
header: "BUT SD - Marvel App"
footer: "Alexandre GIRARD - Conseiller en Nouvelles Technologies - alexandre.girard@maif.fr"
---

<!-- Add this CSS in your Markdown file to style the two-column layout -->
<style>
  .two-columns {
    display: flex;
  }
  .column {
    flex: 1;
    padding: 0 20px;
  }

  span {
    color: grey;
  }
</style>

![bg opacity:.4 ](./img/background/marvelcomics.jpg)

# Marvel - Version 1.0.0

---

# git

Faite le nécessaire pour travailler sur une nouvelle fonctionnalité `Détail d'un personnage`

---

# Api - characters

Généralement, une application web moderne communique avec d'autres services pour récupérer des données ou effectuer des actions spécifiques (authentification, paiement, etc.) via des API.

La communication avec ces services se fait généralement via des requêtes HTTP (GET, POST, PUT, DELETE) et des réponses JSON.

L'appel à une API peut être complexe et nécessite de gérer des erreurs, des temps de réponse, des données manquantes, etc. 

Cela se fait généralement dans un module dédié, qui masque la complexité de l'appel à l'API et expose des fonctions plus simples à utiliser.

---

# Api - characters (suite)

Pour simplifier notre travail, nous allons utiliser un fichier JSON qui contient les données des personnages, mais nous pourrions très bien utiliser une API REST. 

L'implémentation étant "mockée", nous pourrons facilement la remplacer par de vrais appels d'API.

---

# Api - characters (suite)

Créez un fichier `characters-api.js` dans le dossier `src/api` qui expose deux fonctions:

- `getCharacters` qui retourne la liste des personnages
- `getCharacterById` qui retourne un personnage en fonction de son id

en récupérant les données du fichier `characters.json`

---

<!-- Remove header and footer for subsequent slides -->
<!-- _header: "" -->
<!-- _footer: "" -->

# CharactersPage

Modifier le fichier `CharactersPage.jsx` pour utiliser la fonction `getCharacters`

```javascript
import React from 'react';
import { CharactersList } from "../components/CharactersList";
import { NumberOfCharacters } from "../components/NumberOfCharacters";
import { getCharacters } from '../api/characters-api';

// import characters from '../data/characters.json';

const CharactersPage = () => {
    // change the title of the page
    document.title = "Marvel App";

    const characters = getCharacters();

    return (
        <>
            <h2>Marvel Characters</h2>
            <CharactersList characters={characters} />
            <br />
            <NumberOfCharacters characters={characters} />
        </>
    );
};

export default CharactersPage;
```

---

# CharactersPage (suite)

Cette façon de faire fonctionne, mais elle n'est pas optimale. Elle mélange la récupération des données et l'affichage des composants.

`react_router` nous permet de faire mieux en utilisant des `hooks` pour récupérer les données avant d'afficher le composant et ainsi de séparer récupération des données et affichage.

Adapter le code pour utiliser `react_router` et les `hooks` comme dans l'exemple du [guide](https://but-sd.github.io/guide-react/react-router/#loader), grâce aux concepts de `loader`et `useLoaderData`. On appelera directement la fonction `getCharacters` dans le `loader` (pas de fonction `fetch`).

---

# CharacterDetailPage

Ajouter la route permettant d'afficher les détails d'un personnage en fonction de son id, en se basant sur l'exemple du [guide](https://but-sd.github.io/guide-react/react-router/#route-parametree) pour récupérer l'id du personnage puis les détails du personnage grâce à la fonction `getCharacterById`.

Note: Le paramètre `loader` peut récupérer les paramètres de l'URL, voir [ici](https://reactrouter.com/en/main/route/loader).

---

# CharacterDetailPage (suite)

Créez un composant `CharacterDetailPage` qui affiche la page de détail d'un personnage, qui elle-même utilise un composant `CharacterDetail` pour afficher les détails du personnage.

Une nouvelle version du fichier `characters.json` est disponible pour ajouter des détails sur les personnages (description, image, etc.) [characters.json](https://but-sd.github.io/prez/characters-full.json)

Attention, l'api peut ne pas renvoyer l'image du personnage, il faut donc vérifier que l'image n'est pas nulle avant de l'afficher.

---

# CharacterDetailPage (suite)

Le rendu de la page doit ressembler à ceci:

<img src="./img/marvel-app/marvel-app-1.0.0-1.png" alt="Character Detail" style="width: 40%;">

---

# CharactersList

Modifier le composant `CharactersList` pour ajouter un lien vers la page de détail d'un personnage en fonction de son id.

Attention de bien utiliser le bon type de lien pour ne pas recharger la page.

Ajouter ce bout de css pour améliorer l'affichage:

```css
li a {
    color: #333;
    text-decoration: none;
}
```

---

# Sécurisation du projet

Afin de sécuriser le projet, nous allons mettre en place des contrôles afin de valider les modifications appportées au code source des branches `main` et `develop`:
  - contrôle que le code build correctement (**C**ontinuous **I**ntegration)
  - contrôle que les modifications sont validées par un autre développeur avant d'être intégrées au projet (revue de code)

---

# Github Actions - **C**ontinuous **I**ntegration/**C**ontinuous **D**eployment)

github permet d'automatiser certaines tâches grâce aux __workflows__ et de réagir à certains événements comme un __push__ ou une __pull request__. Les executions du workflow sont visibles dans l'onglet __Actions__ du repository.

Nous verrons un peu plus tard ce qu'est une __pull request__.

---

<!-- Remove header and footer for subsequent slides -->
<!-- _header: "" -->
<!-- _footer: "" -->

# Github Actions - CI/CD (suite)

Créez un fichier `.github/workflows/build.yml` qui contient le workflow suivant:

```yaml
name: Build

on:
  push:
  pull_request:

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: 20
        cache: 'npm'
    - run: npm ci
    - run: npm run build

```

---

# Github Actions - CI/CD (suite)

Ce workflow permet de vérifier que le code build correctement. Il est déclenché à chaque push et à chaque pull request quelque soit la branche. Il est composé d'un seul job build qui s'exécute sur une machine virtuelle ubuntu-latest.

Les étapes du job sont les suivantes:
  - récupération du code source avec `actions/checkout`
  - installation de node avec `actions/setup-node`
  - installation des dépendances avec `npm ci`
  - build du projet avec `npm run build`

Une fois le workflow créé, nous pouvons le tester en modifiant le code source et en poussant les modifications sur GitHub.

---

# Github Actions - CI/CD (suite)

Le résultat du workflow est visible sur la page des **actions** du projet sur GitHub.

Si le workflow échoue, il est possible de voir les logs d'exécution du workflow en cliquant sur le nom du workflow dans la page des actions du projet sur GitHub. Cela permet de voir les erreurs et de les corriger ensuite en local avant de pousser les modifications sur GitHub.

---

# Protection des branches

Nous allons protéger les branches **main** et **develop** :
  - afin d'éviter de les modifier directement et de forcer l'utilisation des pull requests
  - afin de mettre en oeuvre la revue de code
  - afin de vérifier que le code poussé sur ces branches build correctement avant d'être intégré

---

# Protection des branches (suite)

Une bonne pratique est de protéger la branche **main** pour éviter de la modifier directement et de forcer l'utilisation des pull requests. Il sera alors impossible de pousser (via un git push) directement sur la branche **main**.

Il peut être intéressant de protéger également la branche **develop** pour les mêmes raisons, mais cela dépend de la politique de développement de l'équipe.

---

# Protection des branches (suite)

Pour protéger une branche, il faut se rendre dans les paramètres du repository, puis dans l'onglet **Branches**.

A minima, il est intéressant de cocher les options suivantes:
  - Require pull request before merging: pour forcer l'utilisation des pull requests
  - Require status checks to pass before merging: pour vérifier que le code passe les étapes de CI/CD avant d'être intégré
  - Require branches to be up to date before merging: pour vérifier que la branche est à jour avant d'être intégrée

---

# Pull Request

La **Pull Request** est une fonctionnalité de GitHub qui permet de proposer la validation de modifications du code source d'un projet.

La **pull request** va permettre de :
  - discuter des modifications proposées avant de les intégrer au projet,
  - de mettre en oeuvre la revue de code, pratique qui consiste à faire relire son code par un autre développeur. Cela permet de s'assurer de la qualité du code et de partager les connaissances entre les développeurs. La revue de code est une pratique très répandue dans les entreprises.
  - de vérifier que le code répond aux critères de qualité définis par l'équipe (tests, documentation, etc.)
  - de vérifier que le code passe les étapes de CI/CD (build, tests, qualité du code, etc.)

---

# Pull Request (suite)

Pour créer une **pull request**, il faut se rendre sur la page du repository sur GitHub, puis cliquer sur le bouton **New pull request**.

Il est possible de choisir la branche source et la branche cible de la **pull request**. La branche source est la branche qui contient les modifications à intégrer, la branche cible est la branche sur laquelle les modifications doivent être intégrées.

Nous pouvons mettre en oeuvre une pull request pour intégrer les modifications de la branche `feature/character-detail` sur la branche `develop`.

---

# Pull Request (suite)

La **pull request** est un outil très puissant pour collaborer sur un projet et pour s'assurer de la qualité du code.

Il est possible de commenter les modifications, de demander des modifications, de valider les modifications, de demander une revue de code à un **reviewer**, etc.

La **pull request** permet de comparer les modifications entre la branche source et la branche cible, de voir les différences entre les deux branches, de voir les fichiers modifiés, etc. 

Les modifications sont affichées ligne par ligne, ce qui permet de commenter directement sur les lignes de code.

---

# Pull Request (suite)

Une fois que tout les `checks` sont passés, il est possible de valider la **pull request** en cliquant sur le bouton **Merge pull request**.

Il est possible de supprimer la branche source après l'intégration des modifications en cochant la case **Delete branch**. Cela permet de garder un historique propre et de ne pas encombrer le repository avec des branches inutiles.

---

# git flow

Pour rappel, le workflow git flow est le suivant:
  - la branche `main` contient la version de production du projet
  - la branche `develop` contient la version de développement du projet
  - les branches `feature/*` contiennent les fonctionnalités en cours de développement
  - les branches `hotfix/*` contiennent les corrections de bugs en production

---

# git flow - release  

Pour préparer une nouvelle version du projet, il est possible de créer une branche `release/*` à partir de la branche `develop`. Cette branche va permettre de préparer la nouvelle version du projet en effectuant les dernières modifications avant de la déployer en production.

Cela permet de geler les fonctionnalités en cours de développement et de se concentrer sur la préparation de la nouvelle version du projet. Pour par exemple, mettre à jour la documentation, les dépendances, etc.

C'est cette branche **release** qui va être déployée en production après validation. C'est l'occasion par exemple d'effectuer une recette utilisateur avant de déployer la nouvelle version en production.

---

# git flow - release (suite)

Nous allons modifier le fichier package.json pour mettre à jour la version du projet, nous allons passer de la version 0.4.0 à la version 1.0.0.rc1.
`rc.` signifie **release candidate**, c'est une version de test avant la version finale.

```json
{
  "version": "1.0.0.rc1"
}
```

---

# git flow - release (suite)

Si cette version est validée, nous allons la déployer en production en la renommant en `1.0.0`, si ce n'est pas le cas nous allons corriger les problèmes et créer une nouvelle version de test `1.0.0.rc2`. 

Le processus est itératif jusqu'à ce que la version soit validée. Les corrections de bugs sont effectuées sur la branche `release/*` et seront à intégrer sur la branche `develop`.

On ne peut pas travailler depuis une feature branch à partir de la branche `develop` car la branche `develop` a potentiellement évoluée avec des fonctionnalités prévues pour la prochaine version.

---

# git flow - release (suite)

Il peut être intéressant de mettre en place les mêmes contrôles que pour les fonctionnalités, c'est à dire de créer une **pull request** pour intégrer les modifications sur la branche `release/*` et de valider la **pull request** avant de merger les modifications vers la branche `release/*`.

Les branches de corrections de bug sur la branche `release/*`  sont généralement appelées `bugfix/*` et devrait être des branches courtes et ciblées (une seule correction de bug par branche et ayant une durée de vie courte). 

Si ce n'est pas le cas c'est que le bug est plus complexe et nécessite une analyse plus approfondie, la fonctionnalité devrait peut-être être repensée et intégrée dans une prochaine version.

---

# git flow - release (suite)

La phase de **release** est l'occasion de mettre à jour la documentation. 

Nous allons ici mettre à jour le fichier `README.md` pour indiquer comment installer et lancer l'application. Nous utilisons le format **Markdown** pour écrire le fichier README.md. Nous verrons plus tard comment utiliser **Markdown**.

---

# git flow - release (suite)

```markdown
# Marvel App

## Installation

git clone url-du-projet
cd marvel-app

npm install

## Lancement

npm run dev

L'application est accessible à l'adresse affichée dans la console.


## Rechargement à chaud

L'application supporte le rechargement à chaud, ce qui signifie que les modifications du code source sont prises en compte sans avoir à recharger la page.


## Point d'entrée

Le point d'entrée de l'application est le fichier `index.html` situé à la racine du projet. C'est ce fichier qui est chargé dans le navigateur et qui charge ensuite le fichier `main.jsx` qui est le point d'entrée de l'application React.

```

<!-- ---

TODO

# Deployment - Netifly

Nous allons déployer l'application sur **netlify**. **netlify** est un service cloud qui permet de déployer des applications web.

Créer un compte sur [netlify](https://app.netlify.com/signup) et se connecter avec son compte GitHub. -->