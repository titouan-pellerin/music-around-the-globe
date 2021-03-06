# Music Around The Globe

Une application web interactive pour découvrir de la musique à travers le monde. Le principe est assez simple. L'utilisateur commence par renseigner ses artistes favoris. L'algorithme va utiliser cette sélection pour trouver des artistes en lien avec les goûts de l'utilisateur, pour ensuite les afficher autour d'un globe 3D selon leur emplacement sur la planète.

Ce dépôt correspond à l'interface utilisateur. L'interface utilisateur récupère toutes les informations nécessaires depuis une API que j'ai développé. Cette API fait elle-même appel aux API de Spotify, MusicBrainz et Mapbox. 
[Dépôt de l'API](https://github.com/titouan-pellerin/music-around-the-globe-api)

Une première démonstration de l'application est disponible en suivant ce lien :
[http://music-around-the-globe.herokuapp.com/](http://music-around-the-globe.herokuapp.com/)

Le projet est encore en cours de développement. L'étape sur laquelle je travaille actuellement, correspond à l'affichage des informations d'un artiste sur lequel on cliquerait depuis le globe 3D.

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
