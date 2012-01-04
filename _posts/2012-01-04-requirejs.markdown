---
layout: post
title: Organiser son code JavaScript avec RequireJS
author: filirom1
tags: [requirejs, javascript, webapp]
---

Je suis certain que vous avez connu la frénésie jQuery. Le JavaScript prend de plus en plus de place au sein de l'application. On commence à avoir des difficultés à gérer l'ordre des balises scripts, la navigateur télécharge un nombre incroyable de fichiers js et le navigateur ramme, on perd le compte du nombre de variables globales, on trouve du code javascript dans l'html et du html dans du javascript, le code dupliqué se multiplie et très vite, on ne sait plus qui fait quoi et rien n'est testé...

Alors comment s'en sortir ?
Personnellement à m'en sortir avec [RequireJS](http://requirejs.org/) et cette méthode que je vais vous expliquer.


Il existe également d'autre méthodes qui couvrent un spectre plus ou moins large des problème exposés ci dessus. Je vais commencer par vous les expliquer avant de commencer RequireJS.


Module Pattern
--------------
<http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth>
Il nous permet de ne pas pourrir notre scope gloable. En Javascript un scope est défini à l'aide de fonction. Pour éviter que nos variables arrivent dans le scope globale il suffit de wrapper notre code autours d'une fonction et de l'executer tout de suite après.

    (function(){
      var private = 'I am private, the global scope is free of me !';
    
      function iAmPrivateToo(){
        return 'This function is private and will not be avalaible via the global scope !'
      }
    })()


Object Litteral Pattern
-----------------------
<http://stackoverflow.com/questions/1600130/javascript-advantages-of-object-literal>
Maintenant que nous savons isolé notre code, nous allons apprendre comment le découper pour éviter de se retrouver avec un seul gros fichier.

A partir du moment où l'on découpe notre code, nous allons avoir besoin d'exporter des fonctions et des attributs publiquement dans une variable globale (ça va une seule, on ne crie pas au scandale tout de suite).

    // script1.js
    (function(){
      var myApp = window.myApp = window.myApp || {};
      
      var aPublicVariable = 'I am public, yeahhh';
      
      function aPublicFunction(){
        return 'This function will be public';
      }
      
      var aPrivateVariable = 'I am private, Yeahh !!!';
      
      function aPrivateFunction(){
        return 'I will never export this function';
      }
      
      // exports public functions and variables
      myApp.aPublicVariable = aPublicVariable;
      myApp.aPublicFunction = aPublicFunction;
    })();

    // script2.js use script1 functions
    (function(){
      var myApp = window.myApp = window.myApp || {};
      
      console.log(myApp.aPublicFunction());
    })();

Attention script2 dépend de script1. Il y a un ordre à respecter au niveau de la déclaration des dépendances.

    <script src="script1.js"></script>
    <script src="script2.js"></script>
    
Grouper les fichiers js pour la prod
------------------------------------

On se retrouve avec deux fichiers javascripts, pour le développement c'est mieux mais pour la prod c'est moins bien ! On a augmenté le nombre de ressource à faire télécharger par le navigateur. Maintenant, il faut grouper et minifier ses ressources js avant de les servir en prod. Pour cela nous pouvons utiliser un simple script shell: 

    cat js/src/* >> js/build/app.js

Mais attention, les dépendances risquent d'être cassé. Pour cela nous pouvons utiliser des outils plus évolués : <http://code.google.com/p/wro4j/> qui possède même un plugin Tapestry <https://github.com/lltyk/tapestry-wro4j> ;)

Ou sinon utiliser le script de build de [html5 boilerplate](https://github.com/h5bp/html5-boilerplate/wiki/Build-script)

Si vous avez d'autres outils, laissez un commentaire à cet article.


Gestion des dépendances avec RequireJS
--------------------------------------

C'est à ce moment que je commence à vous parler de RequireJS.

RequireJs va vous permettre de faire : 

*  de l'encapsulation (visibilité public / privée )
*  de la modularité 
*  de la gestion de dépendances
*  de ne plus écrire d'HTML dans des fichiers JS
*  de l'optimisation d'assests (grouper et minifier les js et les css)

RequireJS est actuellement la solution mise en avant par : 

* [Rebecca Murphey pour utiliser RequireJS avec jQuery](http://jqfundamentals.com/)
* [Addy Osmani pour utiliser RequireJS avec Backbone](https://github.com/addyosmani/backbone-fundamentals)
* [Alex Sexton le père de YepNope, qui explique pourquoi il préfère RequireJS](http://www.quora.com/What-are-the-use-cases-for-RequireJS-vs-Yepnope-vs-LABjs)

Voici un exemple de code de RequireJS, suivez les commentaires pour les explications.

    /* js/script1.js */
    define(
      id, /* (optionnel) */
      [ 'jquery', 'underscore' ], /* La liste des dépendances */
      function($, _) { 
        /* on retrouve le module pattern ici */
        var privateVariable = 'I am private';
        function privateFunction(){ ... }
        
        var publicVariable = 'I am public';
        function publicFunction(){ ... }
      
        /* Variables et functions à exporter */
        return {
            publicVariable: publicVariable,
            publicFunction: publicFunction
        }
      }
    )

    /* bootstrap.js qui utilise la dépendance defini ci dessus. */
    require({
      paths: {
        jquery: 'js/libs/jquery-1.7.1.min',
        underscore: 'js/libs/underscore-min'
      }
    },['js/script1'], function(script1){
        console.log(script.publicFunction());
    })
    
Reprenons l'exemple précédent tout doucement.

RequireJs définie deux notions : `define`, `require`.

* `define` permet de définir un module avec des dépendances et des methodes/variables à exporter.
* `require` permet de charger dynamiquement des modules.

### Definir un module avec `define`

Si nous prenons l'exemple d'un module simple.

    define([
        'jquery',
        'underscore',
        'js/myScript',
        ...
    ], function($, _, myScript, ...){ 
      
      /* your private methods and variable here */
      
      return { /* your public variables and functions here */ }
    });

Je n'ai pas spécifier l'id, RequireJs va s'en occuper pour moi et je vous conseil de faire de même. Votre code sera réutilisable plus facilement.

Dans mon module, j'ai spécifié une liste de dépendences. 

    [
      'jquery',
      'underscore',
      'js/myScript',
      ...
    ]
    
Nous remarquons que pour myScript, le nom de la dépendance correspond à son chemin sans l'extension '.js'. `js/myScript` va charger `http://myApp.com/js/myScript.js`. Nous verrons par la suite comment configurer RequireJS afin de donner des chemins particulier (comme pour jQuery et underscore).


RequireJs m'assure que les dépendances seront chargées et accessibles dans le scope de la fonction : `function($, _, myScript, ...){ ... }`

Mon module peut exporter des choses: `return { publicFunction: publicFunction, .... }` Tout ce qui n'est pas exporté est privé. Simple n'est-ce pas ;)

Et comme vous l'aurez supposé, tout ce qui est exporté par un module, est disponible par les autres modules via les attributs de la fonction : `$, _, myScript, ...`

RequireJs va s'occuper de charger les dépendances des dépendances, ... et au final je n'aurai plus qu'à définir qu'un seul module : myApplication et de ne charger que ce module pour que toutes les dependances de mon application soit chargées automatiquement (ce n'est plus à nous de gérer l'ordre c'est cool ;) ).


### Charger des modules avec `require`

Maintenant c'est sympa nous avons défini des modules, mais il nous manque un point d'entrée à notre application. C'est le rôle de `require` de faire ça. `require` va charger dynamiquement un module, puis ses dépendances, puis appeler la fonction passée en dernier paramètre.

    require({
      paths: {
        jquery: 'js/libs/jquery-1.7.1.min',
        underscore: 'js/libs/underscore-min'
      }
    },['jquery', 'js/script1'], function($, script1){
      $(function(){
        $('#myID').text('Loaded ' + script1.publicFunction());
      });
    })


### Configurer RequireJS

Comme vous l'avez remarqué le premier paramètre de requireJS est un objet de configuration : 

    paths: {
      jquery: 'js/libs/jquery-1.7.1.min',
      underscore: 'js/libs/underscore-min'
    }

Le paramètre `paths` nous permet de definir l'emplacement des librairies utilisés

Tout à l'heure je vous est dit, n'utiliser pas d'ID dans vos modules et vous allez maintenant comprendre pourquoi.

jQuery définie dans son [code ](https://github.com/jquery/jquery/blob/cae1b6174917df3b4db661f20ef4745dd6e7f305/src/core.js#L949) un `define('jquery', function(){ return jQuery });` Le premier paramètre `jquery` est l'id et nous oblige à utiliser `jquery` dans nos dépendances. 

Normalement nous aurions pu faire appel à jquery via son chemin complet : 
`define(['js/libs/jquery-1.7.1.min'], function($){ ... })`, mais à cause de l'ID nous devons faire réference à jquery par son id :`define(['jquery'], function($){ ... })`

C'est le rôle de la configuration `paths: { jquery: 'js/libs/jquery-1.7.1.min', }`. RequireJS va remplacer chaque dépendance `jquery` par son chemin réel afin de trouver la librairie.


### Intégrer RequireJS dans notre HTML

Il faut rajouter dansle HTML les balises script pour require.js et notre bootstrap.js (celui faisant appel à toute nos dépendances).

    <html>
        <head>...</head>
        <body>
          <!-- Ajoutons requireJS et notre script à la fin du body -->
          <script src="require.js"></script>
          <script src="bootstrap.js"></script>
        </body>
    </html>

RequireJs rajoutera les dépendences chagé via des balise script dans le head du HTML et le navigateur les chargera automatiquement.

Maintenant lorsque nous naviguons sur notre site, le navigateur télécharge require.js, le bootstrap, et toute les dependances.


### Optimiser les ressources

Avec RequireJS nous avons découpé notre application en plusieurs fichiers, malheuresement tous les fichiers sont téléchargés un par un par le navigateur.

Nous aurions besoin de grouper et minifier l'ensemble de ces fichiers avant de les servir en production.

RequireJS arrive avec un [script de build](https://github.com/jrburke/r.js/) qui a exactement ce rôle là. 

Si on appelle r.js avec le paramètre -o et en lui spécifiant un fichier de configuration : `r.js -o build.js`

r.js va parcourir l'ensemble des scripts et les minifiers, puis va parcourir nos dépendance afin de les grouper. Les fichiers résultant seront placer dans un nouveau repertoire afin de ne pas écraser les sources.

    ({
        appDir: './src',    /* Repertoire des sources */
        baseUrl: ".",       /* Le repertoire racine */
        dir: "./build",     /* Repertoire de destination */
        modules: [ {
          name: 'bootstrap' /* Une dependance vers le fichier appelant le require */
        } ],
        optimizeCss: "standard.keepLines", /* On va même optimiser les CSS */
        dirExclusionRegExp: /node_modules|test|build/ /* on va exclure du processus de build certain repertoires. */
    })

Le fichier bootstrap.js du réperoire build contiendra l'ensemble de notre code js minifié. 



### Chargement dynamique de code

Il se peut que lors de la consrtuction d'une Single Page Web App vous ayez besoin de definir des ensembles de modules. Par exemple un module qui contient l'ensemble du code JS pour le frontend de votre appli et un autre module qui va contenir le codeJS supplémentaire nécessaire pour charger le backend.

Vous ne souhaitez pas que les internautes qui accèdent à votre frontend soit ralenti par le télechargement des ressources du backend.

C'est pour répondre à cette problématique que le script de build r.js peut être configuré avec plusieurs modules.

### RequireJS sur mobile

Sur mobile on a toujours des contraintes des tailles. Nous ne voulons surtout pas charger de libraries inutilement. Du fait que nous avons utiliser la structuration AMD (Asynchrnous Module Definition) dans notre code, nous sommes obligé d'utiliser une implementation AMD pour lancer notre application et requireJs n'est pas la seule implémentation existante. 

Actuellement il en existe plusieurs dont 3 que j'ai testé personnelement.

* Almond la plus petite (mois de 1Ko gzippé) ne fait pas de chargement dynamique. Elle fonctionne très bien en complément du script builder r.js. Comme tout le code est groupé en un seul fichier, Almond peut faire son travail. Almond est la solution privilégié sur mobile.
* Curl.js qui est deux fois moins gros que RequireJs mais qui fait également moins de choses. Mais de toute façon RequireJS fait beaucoup beaucoup de chose dont vous n'aurez pas forcement besoin et curl.js peut donc être une bonne alternative si vous avez besoin de chargement dynamique de code.

Par contre, sur des projets plus gros, souvent RequireJS est le seul à s'en sortir.

Et voici une comparisons des implémentations en terme de taille

    $ ls -l
    -rw-r--r-- 1 romain users  925 29 nov.  15:41 almond.min.js.gz
    -rw-r--r-- 1 romain users 2,6K 29 nov.  15:41 curl.js.gz
    -rw-r--r-- 1 romain users 5,5K 29 nov.  15:41 require.js.gz


Si vous utilisez almond qui ne fait pas de dynamique loading, vous aurez besoin en prod de remplacer vos dependances requireJS + votre code par un seul fichier minifier contenant tout.

    - <script src="require.js"></script>
    - <script src="boostrap.js"></script>
    + <script src="almond-and-bootstrap.js"></script>

Pour cela vous pouvez utiliser : 

*  soit utiliser [httpBuild](https://github.com/jrburke/r.js/blob/master/build/tests/http/httpBuild.js) afin de toujours compiler en dev vos assets RequireJS côté serveur
* soit utiliser les fonctionnalité de [has avec RequireJS](http://requirejs.org/docs/optimization.html#hasjs) : [exemple](https://github.com/alankligman/gladius/blob/develop/src/gladius.js)

### Stop au JS dans les HTML et à l'HTML dans les JS

Avec requireJs et son plugin text vous avez la chance, de ne plus jamais écrire de Javascript dans des fichiers HTML ni d'écrire du HTML dans les fichiers Javascript.

Voici un exemple de 

    /* bootstrap.js */
    require([
      'jquery',
      'underscore',
      'text!views/template.html'
    ], function($, _, tmpl){
      var compiledTemplate = _.template(tmpl);
      
      $(function(){
        var $el = $('.injectHere');
        $('.btn').click(function(){
            $el.html(compiledtemplat({
                firstName: firstName, 
                lastName: lastName
            })));
        });
      });
    })

Et voci le template en HTML utilisant le [templating underscore](http://documentcloud.github.com/underscore/#template)

    /* views/template.html */
    <div class="personn">
        <p>My name is <%= firstName %> <%= lastName %></p>
    </div>

Si on faisait tourner le script de build r.js on obtiendrait dans le même fichier : 

* jQuery
* undercore
* le template inliner. Voici à quoi cela ressemblerait : define('text!views/template.html', function(){ return '< div class= ....' });
* et notre bootstrap.js

RequireJS possède même des plugins afin de compiler les templates via le script de build r.js : [underscore template](https://github.com/ZeeAgency/requirejs-tpl), [handlebars](https://github.com/SlexAxton/require-handlebars-plugin) . Encore une chose de moins à faire côté client.


Conclusion
----------

Quoi que RequireJs peut parraitre un poil complexe à configurer, RequireJs permet de répondre à pas mal de problématique et pour l'instant je n'ai rien trouvé de mieux pour faire des Single Page Web App.

Si vous souahitez plus de resources sur RequireJS : 

* [Lisez le site officiel de RequireJS](http://requirejs.org/)
* [Plongez vous dans la présentation de AMD, CommonJS et des ES imports/exports par Addy Osmani](http://addyosmani.com/writing-modular-js/)
* [Lisez l'article de mklabs](http://blog.mklog.fr/article/require-js)
* [Si vous souhaitez des plugins supplémentaire pour RequireJS](https://github.com/millermedeiros/requirejs-plugins)
* [Sur mobile essayez Almond](https://github.com/jrburke/almond)
* [sinon curl](https://github.com/unscriptable/curl)

