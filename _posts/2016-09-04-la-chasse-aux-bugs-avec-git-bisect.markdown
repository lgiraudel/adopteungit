---
layout: post
title:  "La chasse aux bugs avec git bisect"
date:   2016-09-04 12:00:00 +0200
categories: commande bisect
disqus: true
excerpt_separator: <!--more-->
filename: 2016-09-04-la-chasse-aux-bugs-avec-git-bisect.markdown
---
Quand on nous dit qu'il y a bug dans notre programme, nous avons généralement trois réactions possibles :

- "Ha mais c'est normal que ça fasse ça, on n'a pas du tout prévu ce comportement" (le fameux "It's not a bug, it's a feature").
- "Wow, c'est zarb ça, va falloir que je creuse pour comprendre."
- "Diantre, je suis pourtant sûr que ça eut marché."
<!--more-->

C'est cette 3ème possibilité qui nous intéresse aujourd'hui. D'une part parce que ça m'a permis de te coller un passé antérieur dans ta face, ce qui n'est pas anodin, mais surtout parce qu'il n'y a rien de plus pénible que de réparer un truc qui marchait très bien quelques jours / mois / années plus tôt.

Trouver l'origine d'un bug c'est 80% du travail pour le résoudre et git bisect c'est justement la solution aux p'tits oignons pour retrouver le commit ayant introduit une régression.
Le principe est simple : on fournit à la commande un commit pour lequel on est sûr que le code marchait correctement, un commit pour lequel on est sûr que ça ne marchait plus (par exemple le HEAD) et git bisect va couper l'intervalle en deux pour nous positionner sur le commit du milieu. Il nous revient alors de tester si oui ou non sur ce commit du milieu le bug est présent et de l'indiquer à git bisect pour qu'il réduise de nouveau l'intervalle en deux. Et ainsi de suite jusqu'à retrouver le commit ayant tout ruiné.
Une [recherche dichotomique](https://fr.wikipedia.org/wiki/Recherche_dichotomique) quoi.

Si t'as pas tout capté, t'inquiète on va prendre un exemple.

Supposons que j'ai une application qui me retourne tout simplement `true`. C'est pas l'application du siècle mais y'a quand même moyen de se faire un paquet de brouzoufs avec (tout le monde utilise des booléens après tout).

{% gist 1e16cbd3aef80a02d60e8f31f78e09aa %}

Bref, j'ai cette application qui vivote et un jour, j'ai un gros client dans le domaine du nucléaire qui m'appelle pour me signaler que mon appli est tip-top mais qu'elle bug depuis quelques temps parce qu'elle s'est mise à retourner `false` et qu'ils commencent à avoir un coeur de réacteur qui surchauffe à cause de ça.

T'inquiètes mon lapin, tonton Logit il va te trouver le bug en 4 coups de cuillère à commit et tu vas pouvoir refroidir ton p'tit coeur qui s'emballe.

On va donc simuler ce comportement en générant 100 commits dans mon appli et en faisant en sorte qu'à un moment elle se mette à retourner `false` à la place de `true`.

Je fais ça avec ce petit script Node :

{% gist 42b6077aee515a5c3096acf2a50efdd9 %}

Testons l'appli après ces 100 commits pour voir :

{% highlight text %}
node app.js
> false
{% endhighlight %}

Ok j'ai mon bug, maintenant je dois choisir l'intervalle sur lequel je vais faire mon git bisect. Je sais que mon application marchait au début et ne marche plus au bout de 100 commits donc on ne va pas s'embêter, on va prendre tout l'intervalle.
Je lance donc git bisect avec la commande `start` en lui précisant la fin puis le début de l'intervalle (oui c'est pas super logique et c'est comme les prises USB, on essaye toujours 3 fois avant d'y arriver), autrement dit le HEAD et le 100ème commit avant le HEAD :

{% highlight text %}
> git bisect start HEAD HEAD~100
Bisecting: 49 revisions left to test after this (roughly 6 steps)
[097650fd10877eadb3754ed3170faad5feac7467] Just a random commits
{% endhighlight %}

git m'annonce que pour trouver mon commit, il va devoir couper l'intervalle en deux 6 fois d'affilé. Il m'annonce également qu'il s'est placé sur le commit *097650fd10877eadb3754ed3170faad5feac7467*, qui doit se situer au milieu de l'intervalle que je lui ai passé.

Je dois alors tester si à cet endroit l'application fonctionnait ou était déjà cassée.

{% highlight text %}
> node app.js
false
{% endhighlight %}

La valeur est toujours `false` donc c'était déjà cassé lorsque ce commit du milieu a été créé. Du coup je préviens git en lui disant que ce commit du milieu est "mauvais" :

{% highlight text %}
> git bisect bad
Bisecting: 24 revisions left to test after this (roughly 5 steps)
[f7cc3a58bf2b2407a41870db7e359df0468283f0] Just a random commits
{% endhighlight %}

Hop, git m'annonce qu'il ne reste plus que 5 étapes avant de trouver le commit ayant engendré le bug. Il a également découpé en deux la première moitiée de notre précédent intervalle et s'est positionné sur le commit du milieu. On re-test l'application sur ce nouveau commit :

{% highlight text %}
> node app.js
false
{% endhighlight %}

On indique de nouveau à git que ce commit était mauvais et on retest :

{% highlight text %}
> git bisect bad
Bisecting: 12 revisions left to test after this (roughly 4 steps)
[b69debfd02d4099e5cd009794351800b7ce5bc94] Just a random commit

> node app.js
false
{% endhighlight %}

On recommence :

{% highlight text %}
> git bisect bad
Bisecting: 5 revisions left to test after this (roughly 3 steps)
[67bc83324eda649968ae89e5f643c23cf8ac02d2] Just a random commit

> node app.js
true
{% endhighlight %}

Ha ! Cette fois on voit que sur ce commit le bug n'était pas encore présent parce que l'application retournait encore `true`. Du coup on signale à git que ce commit était "bon" et on continue à tester :

{% highlight text %}
> git bisect good
Bisecting: 2 revisions left to test after this (roughly 2 steps)
[5f64b5e75ef0be301e27b9bd160f2e6d9a3af659] Just a random commit

> node app.js
true
{% endhighlight %}

Là aussi l'appli n'était pas encore cassée, continuons :

{% highlight text %}
> git bisect good
Bisecting: 0 revisions left to test after this (roughly 1 step)
[642a24f53eae8a51f1bd94a665ac34533c19c0b1] Just a random commit

> node app.js
false
{% endhighlight %}

On a de nouveau le bug, signalons à git que le commit est mauvais :

{% highlight text %}
> git bisect bad
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[c9382d3817df15c332a83a3fe3a8baec7c1f71fe] Commit introducing the bug

> node app.js
false
{% endhighlight %}

Dernière étape :

{% highlight text %}
> git bisect bad
c9382d3817df15c332a83a3fe3a8baec7c1f71fe is the first bad commit
commit c9382d3817df15c332a83a3fe3a8baec7c1f71fe
Author: Loïc Giraudel <lgiraudel@purch.com>
Date:   Fri Sep 2 22:57:13 2016 +0200

    Commit introducing the bug

:100644 100644 73f737c59c37ba81651b6187c1a42bf082050ef7 82e47dd52f32855e369243a0be37e7aeeb389399 M  true-generator.js
{% endhighlight %}

Ca y est, on en est venu à bout. Lors de la toute dernière étape, git nous indique le SHA1 du tout premier commit ayant engendré le problème, à savoir *c9382d3817df15c332a83a3fe3a8baec7c1f71fe*.
De plus git s'est positionné sur le premier commit ne contenant pas le problème.

Il ne me reste plus qu'à analyser le commit ayant introduit le bug et à le corriger après être retourné sur le master (ce qui peut être fait avec la commande `git bisect reset`).

{% highlight diff %}
> git log -p -1 c9382d3817df15c332a83a3fe3a8baec7c1f71fe
commit c9382d3817df15c332a83a3fe3a8baec7c1f71fe
Author: Loïc Giraudel <l.giraudel@gmail.com>
Date:   Fri Sep 2 22:57:13 2016 +0200
 
    Commit introducing the bug
 
diff --git true-generator.js true-generator.js
index 73f737c..82e47dd 100644
--- true-generator.js
+++ true-generator.js
@@ -1 +1 @@
-module.exports = true;
\ No newline at end of file
+module.exports = false;
\ No newline at end of file
{% endhighlight %}

On fix, on commit et sans s'en rendre compte on vient de sauver une bonne partie du territoire qui n'aura pas à subire une explosion nucléaire.

Tout ça grâce à git bisect.

Pas mal, non ?

### Les questions qui fachent, les réponses qui rassurent

- __Q) Ca marche tout le temps ?__
- R) Oui et non. Disons qu'il faut que l'application soit toujours exécutable. Par exemple si sur l'intervalle du bisect l'environnement a changé en matière de dépendances, d'environnement d'exécution, etc, ça devient difficile de vérifier si le bug était déjà présent ou non.

- __Q) Que faire si git se repositionne sur un commit ou exceptionnellement l'appli ne marchait pas ?__
- R) Il est possible de zapper une étape du git bisect en mettant `skip` à la place de `good` ou `bad`. git va alors chercher un commit proche pour remplacer celui qu'il avait choisi.
Bien sûr, si on *skip* trop souvent, à la fin git peut nous dire qu'il n'est pas capable de trouver le commit exact qui a engendré le bug et nous fournira plutôt le plus petit intervalle sur lequel le bug a été introduit. Ca encourage à faire des petits commits et à faire en sorte que la branche reste en permanence opérationnelle pour éviter ce genre de désagrément.

- __Q) Vaut mieux faire tourner l'outil sur un petit intervalle, non ?__
- R) Pas forcément vu qu'il s'agit d'une recherche dichotomique. Là il nous a fallu 7 étapes pour traiter un intervalle de 100 commits. Un intervalle de 1.000 commits ça n'est "que" 10 étapes et 100.000 commits ne représentent "que" 17 étapes. Bref ça va très vite, à condition d'avoir peu d'actions à effectuer entre chaque étape (tirer les éventuels changements dans les dépendances, builder, déployer pour pouvoir tester, etc). Après c'est sûr que ça sert à rien de parser tout votre historique si votre bug est survenu au cours de 2 dernières semaines.

- __Q) En parlant de ça, c'est possible de donner un intervalle basé sur des dates ?__
- R) Yep : `git bisect start HEAD@{one month ago} HEAD@{2016-01-01}`

- __Q) La succession des différentes étapes du git bisect est un peu longue, il y a moyen de l'automatiser ?__
- R) Ouiiiiiii :) On y reviendra sur un article dédié mais le principe consiste à avoir un test qui fait les vérifications pour nous, ce qui va automatiser le choix `good` / `bad`.