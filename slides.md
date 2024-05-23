---
titleTemplate: '%s'
theme: seriph
background: https://source.unsplash.com/1920x1080/?developer
highlighter: shiki
lineNumbers: true
info: false
css: unocss
hideInToc: true
---

# L'état des profilers PHP en 2024
Introduction, explication et qui sont les acteurs en place ?

<!--
Bienvenue à cette présentation. Aujourd'hui, je vais vous parler des profilers en PHP, un outil essentiel pour tout développeur souhaitant optimiser son code.
-->

---
layout: center
hideInToc: true
---

# De quoi on parle ?

<Toc />

<!--
On va commencer par une introduction aux divers profilers disponibles aujourd'hui, de leurs fonctionnalités uniques, des avantages et inconvénients de chaque profiler et de la manière dont ils peuvent être utilisés pour résoudre les problèmes de performance courants.

Je vais vous dire ce qu'ils ne sont pas dans ce talk,  je vais vous parler de leur fonctionnement, vous présenter les acteurs en place, leurs usages en production et en local ainsi que leur installation. 

Je ferais une petite comparaison de performance ainsi que mes prévisions personnelles concernant le futur.
-->

---
layout: center
class: text-center
src: ./pages/presentations.md
---

---
layout: center
title: La petite histoire
level: 1
---

# La petite histoire

<!-- Depuis quelques clients, j'ai eu l'occasion d'utiliser Sentry et donc d'installer et tester un maximum de feature !
J'ai trouvé le produit très intéressant, il dispose d'énormément de fonctionnalitée. 
J'ai ensuite du faire un audit client, j'ai donc mis en place xdebug et excimer afin d'identifier les ralentissements sur le site puisqu'ils n'avaient plus de license blackfire par manque d'usage. 
Plusieurs conclusions me sont venues et l'envie de creuser le sujet des profilers. Me voila donc ici pour vous en parler !
-->

---
layout: section
title: La définition
level: 1
---

# La définition

## Le _profilage de code_

<!-- Le profilage de code permet de contrôler lors de l'exécution d'un logiciel : la liste des fonctions appelées et le temps passé dans chacune d'elles ; l'utilisation processeur ; l'utilisation mémoire.  - Wikipedia -->

---
layout: quote
hideInToc: true
---

>  Getting started with profiling can be intimidating, especially because the term “profiler” is used to refer to so many specific types of tools. - Sentry

<!--
Je souhaite préciser de quoi on ne parle pas.

"Se lancer dans le profilage peut être intimidant, surtout parce que le terme « profilage » est utilisé pour désigner tant de types spécifiques d'outils." - Sentry

Le profiling dont je vais vous parler ici réfère au profilage de code et non pas les jolis outils tels que celui de Symfony.
-->

---
layout: center
hideInToc: true
---

<img width="550px" src="/symfony-profiler-interface.png"/>

<!-- Donc pas ça ! -->

---
layout: section
hideInToc: true
---

# Historique

Le premier date de 1970 !

<!--
Ils existent depuis déjà un petit moment dans l’informatique et depuis environ 20 ans en PHP.
Pour vous donner un petit historique des solutions que je veux présenter, En 2006 déjà, xdebug permettait de profiler son code. 
XHPROF sera rendu open source en mars 2009 par Facebook puis abandonné après l’adoption de HHVM pour finalement être forké par Tideways. 
Il sera intégré à Tideways Profiler Platform en 2014 afin d’en simplifier son intégration et développer son modèle économique. Blackfire sort autour de 2014 lui. 
En 2018, Wikipedia sortira Excimer, une extension PHP permettant le profiling, développé en interne et qui s’occupe de profiler l’intégralité de Wikipédia.
-->

---
layout: section
title: Fonctionnement
level: 2
---

# Fonctionnement

<!--

Maintenant je vais vous parler du fonctionnement de ces profilers, pas de stress c’est très simple.

Une grande différence au sein de ces outils est leur fonctionnement. Il existe deux grands fonctionnements dont je vais vous parler aujourd'hui, par instrumentation (Deterministic) et par échantillonage (Sampling).
-->

---
layout: two-cols-header
hideInToc: true
---

::left::
<img src="/sentry_explaination_2.png">

::right::

<v-click>
<img src="/sentry-explaination.png">
</v-click>

<!--
En utilisant un profilage de code par instrumentation, toute entrée/sortie d’une fonction est suivie, plus on a de fonction appelé, plus l’overhead du profiler est lourd. (L'overhead est le charge induite par le profiler)
Alors que le sampling va regarder à intervalles réguliers quels fonctions tournent afin de déterminer sa présence ou son absence et en déterminer sa durée. 
On voit que l’échantillonage est ici de 10ms ce qui permet de voir A, C et D. On loupe B avec cette valeur, c’est un défaut mais qui permet son usage en environnement de production puisque la charge de travail est bien inférieure.

Si vous n'avez pas tous compris, un autre schéma est disponible juste à côté -->

---
layout: section
hideInToc: true
---

# Comment on visualise nos résultats ?

<!--
Plusieurs formats existent et sont disponibles pour visualiser nos résultats, xdebug s’étant basé sur Cachegrind, un profiler/standard déjà développé. 
La visualisation de résultat se fait comme ceci au sein de KCachegrind :
-->

---
layout: center
hideInToc: true
---

<img height="500px" src="/kcachegrind.png">

<!--
La vue à gauche permet de visualiser de l’appel de la fonction, la classe ou le fichier et le temps passé au total (self) dans les lignes de cette fonction sans compter les appels à d’autres fonctions. 
Elle permet aussi de consulter les cycles d'appels, donc si une boucle se fait 3 fois, on est en capacité de savoir quel cycle est le + lent et identifier quelle boucle est plus lente.
La visualisation à droite est une vue qui s’appelle une Callee map, une vue très visuelle mais qui devient vite brouillon selon moi, (je ne juge personne).

Je vous laisse un peu apprécier 

*pause eau*

Une autre vue disponible est très appréciée est le Call graph.
-->

---
layout: iframe
url: https://svgur.com/i/169a.svg
hideInToc: true
---

<!-- Cette vue bien plus visuelle se compose de plusieurs éléments : -->

---
layout: section
hideInToc: true
---

<img src="/call_graph_details.png" />

<!--
La classe appelé, sa fonction appelée, le temps passé dans la fonction (incl) et self ainsi que le nombre d’appel. Les flèches permettent de facilement voir les fonctions appelés par celle ci ainsi que le nombre d’appel.  Plus le coût est important, plus la flêche est grosse.

Ici par exemple, ProfileThisRequest est présente 99,58% du temps et appelé une fois.
Elle passe 12% de son temps à appeler 10000 fois le persist de Doctrine, une fonction sur le modèle pour 10% du temps aussi et 75% du temps à appeler le flush de doctrine.
Cette vue dispose en revanche d’un soucis, elle nécessite que le profiler fonctionne par instrumentation.

Mais pourquoi ?
-->

---
layout: section
hideInToc: true
---

## Flamegraph 🔥

<v-click>

<img src="/flamegraph_wiki.png" />

</v-click>

<!--
Avant de vous l’expliquer je souhaite vous faire découvrir une autre visualisation, le flame graph !

Voila à quoi elle ressemble, la c’est les données de wikipedia.
Vous vous dites sûrement que c’est illisible et pourtant, je vous ai déjà montré cette vue et vous l’avez comprise. Si on retourne sur le schéma sur le sampling, que voyez vous ?
-->

---
layout: two-cols-header
hideInToc: true
---

# Schéma sampling

<img src="/sentry_flamegraph.png" />

::right::

# Speedscore (visualisation)

<img src="/flamegraph_wiki_uptodown.png" />

<!--
En retournant le flamegraph, l’affichage est très similaire. En fait je vous avais déjà montré ce graph.
-->

---
layout: image
image: /flamegraph_wiki_uptodown_zoom.png
---

<!-- 
Il est d'autant plus simple à lire qu'il est zoomable et on peut voir en détail. 
Donc cette vue est celle “par défaut” pour l’échantillonage car vérifier la présence d’une fonction ne permet pas de savoir avec précision et certitude la quantité d’appel et donc ne permet pas une visualisation du nombre d'itérations etc...
-->


---
layout: center
hideInToc: true
---

- 1,6 milliard/j
- 20k/sec
- 3M de trace par jour

<!--
Pour vous partager un petit chiffre cool, voici une stat de wikipedia :

En décembre 2022, les serveurs de Wikipedia reçoivent environ 1,6 milliard de requêtes par jour soit 20 000 par secondes. Avec une période de sampling de 60 secondes, ils collectent 3 millions de trace d'appels par jour. Ce qui donne 1 trace obtenu de 0,2% des requêtes et 0 trace des autres 99,8%.
-->

---
layout: section
hideInToc: true
---

<img src="/xdebug_profiler.png">

<!-- Une option qui vous est aussi possible est d’interprêter directement le résultat au sein de votre IDE, ici dans PHPStorm. 

Vous profitez donc de la navigation de code et gagnez en efficacité. -->

---
layout: section
level: 1
---

## Profilers PHP les plus utilisés en 2024

<!--
Maintenant que nous en savons plus sur les profilers, parlons de la solution la plus populaire au sein de PHP selon mon expérience personnelle au sein de conversations d’entreprise, d’évènement ou bien de chat de communauté.
Parmis les grands intêrets de Blackfire, nous avons sa triple casquette APM, profiler et test.
La partie APM (monitoring) permet la visualisation de sa santé de serveurs, la partie test permet elle de voir si des tests définis passent lors d’un déploiement et permet d’éviter des régressions.
-->

---
layout: section
hideInToc: true
---

# Blackfire

<img src="/blackfire_ui.png">

<!--La partie nous intéressons ici est sa partie profiling, en installant un client, qu’on appelle probe ou sonde et configurant la solution, une extension (navigateur) ou une commande permet de lancer un profilage de code par instrumentation et obtenir un call graph. 
Ces calls graphs sont historisés et comparable entre eux.
Ce fonctionnement permet d’assurer un impact sur les performances minimales via le fonctionnement en échantillonnage pour leur APM et d’appeler au besoin un profiling précis.
Tideways avec xhprof fonctionne de manière assez similaire.
-->

---
layout: center
hideInToc: true
---

# L'argent & | l'administratif..

<v-click>
<img width="300px" src="/cto.jpg">
</v-click>

<!--
Je vais vous parler de l'argent et ou de l'administratif

Le soucis qui revient souvent lors de cette recommandation est la problématique de prix. L’ensemble de la solution coûte 160€ minimum par mois tandis que le plan de développement limité permettant l’accès entre autres au profiling démarre à 33 euros par mois et par utilisateur. La question de prix se pose donc souvent dans des projets qui ne souhaitent pas investir ou n’ont pas les fonds.
Se pose donc naturellement la question des alternatives gratuite et de préférence, open source.
-->

---
layout: section
level: 1
---

# Cas d'utilisation courants et exemples de mise en œuvre en production

<!-- Les projets au fonctionnement le plus semblable sont xdebug et xhprof. Il est possible d’activer programmatiquement le profiling en ayant dans l’appel la présence d’un header qui est configurable.
Cette solution permet de profiler en local et en environnement de production pour produire un fichier qu’il faut ensuite récupérer pour analyser… Cette solution est assez peu sécurisée et offre une expérience très précaire. Il n’existe à ma connaissance aucune solution open source apportant une expérience similaire à Blackfire ou Tideway pour obtenir un call graph en environnement de production. -->

---
layout: center
hideInToc: true
---

<Tweet id="1786904804331024815" />

<!-- En revanche concernant la visualisation de la production, Pyroscope et excimer-ui proposent un flame graph. Pyroscope est un outil développé par Grafana, il permet d’obtenir des données depuis plusieurs langages, via leur agent ou grâce à des SDK. Il n’existe malheureusement pas encore de SDK PHP. 
Oliver Bonvalet (qui m'a donné son autorisation) a partagé sur twitter son usage et sa frustration liée à Pyroscope. -->

---
layout: section
hideInToc: true
---

# Pyroscope

<img src="/pyroscope.png">

<!-- L’outil permet d’ajouter de la metadonnées aux traces qu’on envoie et permet d’avoir une vision d’ensemble des performances sur un projet, il permet donc du filtre et de naviguer les données. -->

---
layout: section
hideInToc: true
---

# Speedscope

<img src="/speedscope.png">

<!-- 
Excimer-ui est un outil développé par wikipedia, c’est un wrapper d’un autre outil, utile pour visualiser les résultats d’excimer simplement. 
Sorti en 2024, l’outil est encore très récent et se base sur une instance de speedscope. Speedscope étant un outil de visualisation des profilers général (non liés à PHP). 
La seule plu value actuelle du projet est d'ingérer facilement les résultats récupérés par excimer et retire le travail manuel nécessaire. Il n’est pas un remplacement à Blackfire.
-->

---
layout: section
hideInToc: true
---

# Le messi : Sentry

<!-- Pour vous proposer une solution open source, Sentry dispose d’un mode self-hosted et intègre excimer comme profiler. -->

---
layout: image
image: /sentry_ui.png
hideInToc: true
---

<!-- En ajoutant de la métadonnée, il permet de filtrer par route, par utilisateur, device, temps passé dans la fonction depuis par exemple 24 heures. 
Aucun graph n’est disponible au profit d’une interface plus accessible à des utilisateurs non techniques ou non formés. 
Des intégrations Laravel, Symfony et autres sont déjà existants et il supporte une multitude de langage.
-->

---
layout: section
hideInToc: true
---

# Récap

| Tool                    | Call graph | Flame graph | Autre visualisation | Historisation | Open source |
|-------------------------|------------|-------------|---------------------|---------------|-------------|
| Excimer-ui              |            | ✅           |                     |               | ✅           |
| Pyroscope               |            | ✅           |                     | ✅             | ✅           |
| Blackfire               | ✅          |             |                     | ✅             |             |
| Sentry                  |            |             | ✅                   | ✅             | ✅           |
| Outil cachegrind locaux | ✅          |             | ✅                   |               | ✅           |


<!-- Globalement pour de la production sans avoir de fournir trop d'effort, aucune solution ne supporte gratuitement les call graph. -->

---
layout: center
hideInToc: true
---

<img src="/cat_coffee.gif" />

<!-- *pause eau* -->

---
layout: section
level: 1
---

# Cas d'utilisation courants et exemples de mise en œuvre en environnement local

<!-- Pour obtenir des données précises, l’usage d’un profiler en local est très intéressant. Leur mise en place est très simple, pour xdebug et xhprof: -->

---
layout: center
hideInToc: true
---

## XDEBUG
```ini
sudo apt-get install php-xdebug
install-php-extensions xdebug
xdebug.mode = profile
xdebug.output_dir = /app/public/xdebug
xdebug.start_with_request = trigger // démarre avec un header
```

## XHPROF
```ini
install-php-extensions xhprof
xhprof.output_dir = /app/public/xhprof
```
```php
xhprof_enable();
register_shutdown_function(function () {
    $data = xhprof_disable();
    file_put_contents('public/xhprof/'. uniqid() . '.yourapp.xhprof', serialize($data));
});
```

<!-- Globalement, vous installez l'extension grâce aux repos disponible sur toutes les distributions et vous le configurez dans vos .ini. 
Viens ensuite des différences, xdebug va démarrer sur la présence d'un header alors que xhprof doit être activé et arrêté manuellement. 
On utilise register_shutdown_function pour enregistrer une fonction de rappel pour exécution à l'extinction.
-->

---
layout: section
hideInToc: true
---

# XDebug

<v-click>

> docker run --rm -v ./public/xdebug:/tmp -p 800:80 jokkedk/webgrind:latest

</v-click>

<!--
Afin d’exploiter ces données, pour xdebug un fichier cachegrind est généré.
J’ai pu trouver une version dockerisé qui permet de visualiser ces fichiers. 
La visualisation se présente de cette manière :
-->

---
layout: default
hideInToc: true
---

# XDebug

<img src="/webgrind.png">

<!-- Il est possible de voir le nombre d’appels, le coût des fonctions par leur propre code et le temps passé dans cette fonction puis les fonctions appelés.

Il est aussi possible de générer un callgraph pour comprendre mieux ce qu’il se passe en générant un svg. 
-->

---
layout: iframe
url: https://svgur.com/i/169a.svg
hideInToc: true
---

<!--
Je vous ai déjà montré ce graph au tout début
-->

---
layout: default
title: Visualisation de XHProf
level: 2
---

# XHProf & XHGui

<img src="/xhgui.png">

<!-- XHProf lui dispose de xhgui pour sauvegarder sa données dans une base de données et la visualiser. L
e projet dispose d’un docker-compose pour lancer facilement une instance et recommande fortement l’outil php-profiler pour faire ingérer la donnée. 
Il a été crée en 2014 mais n’est pas encore en 1.x pour une raison que je n’ai pas cherché.
Il rajoute de la métadonnée au profiler et permet plusieurs vues.
-->

---
layout: default
title: Impact sur les performances
level: 1
---

# Impact sur les performances (échantillonage vs instrumentation)

| Env  | Pas de profiling | Excimer         | Xhprof           | Xdebug            |
|------|------------------|-----------------|------------------|-------------------|
| Dev  | ~1,76sec         | ~1,78sec (1.1%) | ~2,77sec (57.4%) | ~5,55sec (215.3%) |
| Prod | ~665ms           | ~670ms (0.98%)  | ~1200ms (80.5%)  | ~2200ms (230.8%)  |

<!--
Sur mon projet de test, j’ai pu mesurer ces performances ci sur la même requête. J’utilise en environnement de dev et prod sur ma machine locale. La route appelée s’occupe d’envoyer 100000 objets dans une base de données sqlite. 

*lire les % de résultats*

Sur un autre projet, j’ai pu demandé une comparaison de l’usage de blackfire lorsqu’on fait un profilage de code par instrumentation.
Un impact de 30% environ est vu, son impact est donc bien inférieur à n’importe quel autre profiler de part la quantité d’information récolté et son fonctionnement qui est privé.
-->

---
layout: default
title: Mais pourquoi profiler en production ?
level: 1
---

# Pourquoi profiler en production ?


<v-click>

## Difficulté à être iso

</v-click>

<v-clicks>

- Extensions

- Données

- Variable d'environnement

- Dépendances

</v-clicks>

<!-- Les outils au sein de langage sont pratiques et existent depuis longtemps mais ils sont parfois compliqué ou dangereux à utiliser en production et nécessite un outil supplémentaire pour la visualisation.
Ils sont généralement conçu pour tourner sur nos machines uniquement. Un overhead de 200% est considéré normal alors qu'il est inconcevable en production.
Recréer localement l'environnement de production peut-être compliqué car :
*click*
*click*
*click*
*click*
-->

---
layout: section
title: Perspectives d'avenir des profilers après 2024 & bilan
level: 1
---

# Perspectives d'avenir des profilers après 2024 & bilan

<v-click>

Sentry & excimer c'est tip top !

</v-click>


<!--

L’adoption d’Excimer est grandissante, son intégration à Sentry et sa facilité d’installation le rendant d’autant plus intéressant. Sa compatibilité avec Pyroscope montre aussi la capacité d’export des données tandis OpenTelemetry à ouvert sa spec au profiler depuis mars 2024.

Opentelemetry c’est un ensemble de collection d’api, de sdk et d’outil pour l’intéropabilité au sein de l’observabilité. Il est open source et adopté par de grands acteurs qui existe depuis 2019.

Dans les discussions autour de la spec, Grafana, datadog et d’autres grands acteurs du milieu ont participés à la développer et annonce un futur où l’intéropabilité des outils sera plus grand qu’il ne l’est aujourd’hui et pourrait permettent une plus grande concurrence et une facilité pour les plateformes qui ingère la donnée à utiliser un profiler déjà existant.

C’est un sujet à suivre et un avenir qui se profile .. bien.
*click*
Donc Sentry & excimer c'est top ! mais :
-->

---
layout: section
hideInToc: true
---

# Self hosted != gratuit

<!--
Le self hosted a un coût : le serveur & la maintenance !
En revanche si l'on dispose d'un contrat d'infogérance et que la facture Sentry est élevée (elle scale selon le nombre d'appels, erreurs, profiling...) elle demeure plus intéressante.
-->

---
layout: two-cols-header
hideInToc: true
---

# Pour faire des retours sur les slides

<img src="/qrcode_profiler_feedback.png">

::right::

# Pour retrouver la conf

<img src="/qrcode_profiler_slides.png">

---
layout: end
hideInToc: true
---

<!--
- Afin de conclure, j’aimerais vous partager que mes recherches sur le sujet sont disponibles sur notion et j’y cite toutes mes sources.

Je vous remercie d’être venu aujourd’hui, j’espère que la présentation vous a plu et je remercie aussi toute l’AFUP de se donner pour nous faire de si beaux événements. 

S’il nous reste du temps vous pouvez poser vos questions et si vous le désirez je reste disponible durant les pauses pour échanger avec vous ! Merci beaucoup.
-->
