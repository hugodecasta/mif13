## TP 8 : Progressive Web apps

[TOC]



### Audit de l'application

Dans un premier temps charger votre application (idéalement une version déployée) dans Chrome et l'examiner avec l'outil d'audit [Lighthouse](https://developers.google.com/web/tools/lighthouse) disponible dans les outils de développement.

![](lighthouse.png)

Voici un exemple de résultat attendu à ce stade du projet :

![](audit.png)



Nous allons maintenant traiter des problèmes signalés par LightHouse

### HTTPS

Depuis l'arrivée de [Let's Encrypt](https://letsencrypt.org/) il n'y a plus d'excuses pour ne pas servir de pages en `https`. 

Si vous utilisez surge, voir la page suivante pour forcer un déploiement en https : [https://surge.sh/help/using-https-by-default](https://surge.sh/help/using-https-by-default)

### Chargements bloquants

Vérifier que les chargements de police de caractère et de css ne sont pas bloquants.

Permettez le téléchargement et le rendu asynchrone des polices et styles pour accélérer la première peinture. Si vous incluez des polices depuis votre index, chargez les avec `rel="preload"`, en suivant [ces indications](https://alligator.io/html/preload-prefetch/).

**Attention tester plutôt dans Chrome**

### App shell

La manière la plus propre de fournir un appshell efficace, est de faire une partie du rendu de l'application, i.e. la partie sans contenu, ni personalisation côté serveur (ce sera la même pour tout le monde). 1. Cela permet de ne faire le travail qu'une fois; 2. Le contenu peut être mis en cache côté serveur et transféré rapidement; 3. Il y a moins de calculs et de freins pour la première peinture côté client.

Pour des raisons de simplicité nous allons créer un appshell simple en éditant directement l' `index.html` de notre projet.

##### Bannière / menu

- Créer une banière temporaire directement dans le index.html
- Style la banière inline pour ne pas rajouter de dépendance

##### Feedback de chargement 

Afficher un spinner indiquant que le chargement est en cours.

##### Gestion de l'absence de JS

```html
<noscript>
  <h3 style="color: #673ab7; font-family: Helvetica; margin: 2rem;">
    Sorry, but app is not available without javascript
  </h3>
</noscript>
```

### Manifest

Nous allons maintenant rédiger un [Web App Manifest](https://www.w3.org/TR/appmanifest/). Voir l'[exemple de la MDN](https://developer.mozilla.org/en-US/docs/Web/Manifest) ou celui de [Google](https://developers.google.com/web/fundamentals/web-app-manifest/)

<!-- Tu pourrais aussi pointer la spec, stp : https://www.w3.org/TR/appmanifest/ ? -->

Vous pouvez utiliser ce [générateur d'icônes](https://romannurik.github.io/AndroidAssetStudio/icons-launcher.html) (mais de très nombreux autres sont disponibles).

Enfin inclure le manifest dans le head de votre `index` :

```html
<link rel="manifest" href="/manifest.json">
```

Dans les outils de développement de Chrome il est possible d'inspecter son manifest.



### Service Worker et Offline

**Nous allons travailler sur la version de production.**  Nous ne voulons pas gérer les caches et le offline sur la version de dev. Idéalement il faudrait plutôt créer une version de pré-production.

##### Chargement des services workers 

Ajouter un script simple qui chargera votre service worker depuis votre index.html
```js
      // This is the service worker with the combined offline experience (Offline page + Offline copy of pages)

      // Add this below content to your HTML page, or add the js file to your page at the very top to register service worker
      if (navigator.serviceWorker.controller) {
        console.log('[PWA Builder] active service worker found, no need to register')
      } else {
      //Register the ServiceWorker
        navigator.serviceWorker.register('./pwabuilder-sw.js', {
          scope: './'
        }).then(function(reg) {
          console.log('Service worker has been registered for scope:'+ reg.scope);
        });
      }
```

Pour des raisons de portée lié aux services workers le fichier `pwabuilder-sw.js` devra se trouver au même niveau que le fichier index.html dans le dossier `dist`.

Pour cela il faut modifier le fichier de build prod webpack pour copier le fichier `pwabuilder-sw.js` dans `dist`. On utilise pour cela `CopyWebpackPlugin`. Trouver l'endroit ou il est utilisé dans le fichier webpack.config.prod.js, ou sinon l'ajouter ([voir la doc](https://github.com/webpack-contrib/copy-webpack-plugin)).

```js
      {
        from: path.resolve(__dirname, '../pwabuilder-sw.js'),
        to: 'pwabuilder-sw.js',
        toType: 'file'  
      }
```

##### Création du service worker

En s'aidant de [PWAbuilder](https://preview.pwabuilder.com), générer un fichier `pwabuilder-sw.js`. Que vous adapterez au besoin pour mettre en cache les éléments de votre choix.

##### Créer une page dédiée au Offline 

Créer une nouvelle page et une nouvelle route `offline`. Cette page (et ses assets) sera toujours mise en cache et disponible hors-ligne.

Y afficher le [trajet du T1](https://fr.wikipedia.org/wiki/Fichier:Tramway_Lyon_1-plan.svg) et ses [horaires à Université Lyon 1](http://www.tcl.fr/Me-deplacer/Toutes-les-lignes/T1/Horaire-a-l-arret?arret=tcl5505&sens=1).

### Rendu

À rendre pour le lundi 25 juin à 23h59.

Penser à mettre à jour votre README, incluant à minima : les identifiants du binome (n° étudiant, nom, prénom), les instruction de build, les dépendances implicites (i.e. les choses installées en global), toute autre chose facilitant la compréhension du projet par le correcteur.

Créer une branche rendu-tp8, même si ceci est le dernier rendu du cours. Toute erreur sur la gestion des branches sera pénalisée.

Reporter le numéro Tomuss (pas de ‘p’ devant) de votre binome sur Tomuss.

Reporter le lien vers votre dépôt git qui permette de le cloner facilement, au format suivant : `forge.univ-lyon1.fr:idutilisateur/projet.git`

La mise en place des PWA étant plus avancée dans Chrome,  **ce TP sera corrigé avec Chrome**. 

### Barême

Qualité de l'application - 10 pts

- README 
- Rendu **propre** via Tomuss et la branche rendu-tp8
- Structure de projet Vue et structure sur la forge propre (bon usage de .gitignore)
- npm run dev lance le projet sur une machine “vierge” (à défaut toutes les dépendances globales sont explicitées dans le README)
- npm run build construit un projet dans le **dossier dist** prêt à être déployé (sera corrigé sur surge.sh)
- Qualité de l’exécution (code propre, ESLint ne soulève pas d’erreurs, qualité d’usage de l’application)
- Travail réparti au sein du binome : push équitables sur la forge (nombre de commits, nombre de lignes, etc.)
- Tous les éléments sont des composants
- Les composants ont le bon niveau de responsabilité 
- Utilisation appropriée de Vuetify

Capteurs mobile et PWA - 17 pts

- Utiliser la position géographique pour recherche un trajet (3 pts)
- pseudo Réalité Augmenté (3 pts)
- Détection des capacités du dispositif (2pts)
- App Shell (3pts)
- App Manifest (3pts)
- Service workers (3pts)
