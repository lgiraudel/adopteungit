---
layout: post
title:  "Commits atomiques - la bonne approche"
lang: fr
ref: commits-atomiques-la-bonne-approche
date:   2017-04-26 12:00:00 +0200
categories: methodologie
disqus: true
excerpt_separator: <!--more-->
filename: 2017-04-26-commits-atomiques-la-bonne-approche.markdown
---
L'une des principales difficultés dans nos métiers est, à mon humble avis, la difficulté à découper notre travail en petites unités. Cette difficulté s'applique aussi bien au niveau architectural (il est bien plus facile de faire un gros projet monolithique plutôt que plein de petits projets unitaires) qu'au niveau du code que nous produisons tous les jours dans nos commits.

En un mot comme en cent : **il est difficile de faire de petits commits**.
<!--more-->

C'est difficile de faire des petits commits parce qu'il est facile de ne pas en faire. Ca parait stupide dit comme ça, mais il est bien plus simple, à la fin d'un développement, de faire un gros commit plutôt que de s'amuser à tenter d'en faire plein de petits. C'est d'autant plus difficile que la plus-value des petits commits n'est pas visible sur le moment. 

C'est un peu comme les tests automatisés : sur le moment on se dit que ça ne sert à rien, qu'on va relire le code trois fois et bien vérifier la fonctionnalité pour être certain ou certaine de ne pas envoyer un bug et ça ira bien. Et le jour où ça pète en prod à cause d'un changement plus ou moins anodin, on se dit que s'il y avait eu des tests automatisés, le changement anodin aurait fait passer au rouge un test et le problème aurait été fixé avant même d'atteindre la prod.

Les petits commits c'est pareil : c'est parfois chiant à faire, ça parait inutile mais le jour où ça pète en prod on est bien content/e d'avoir fait l'effort d'en écrire.

On peut se demander quel est le rapport entre la taille des commits et un bug survenu en prod mais ce rapport est tout simple : plus il est difficile de trouver l'origine d'un bug et la manière de le fixer et plus il coûtera cher en temps humain pour le fixer. **Réduire le coût d'un bug consiste donc à le repérer au plus tôt** (idéalement avant qu'il n'atteigne la production) et ça c'est le rôle des tests, mais ça consiste également à réduire le temps qu'il faut à l'équipe de développement pour le fixer.

## La traque des bugs

Faire des petits commits aide à trouver l'origine d'un bug, le contexte dans lequel il a été introduit, éventuellement la raison pour laquelle il a été introduit et du coup facilite le travail de la personne qui va s'occuper de le fixer.

C'est d'autant plus facile avec Git quand on maîtrise la commande `git bisect` permettant de faire une recherche dichotomique sur un grand intervalle de commits. Par exemple sur un de mes vieux projet ayant cumulé 31.000 commits en 6 ans, ça voudrait dire que je devrais être capable de trouver le commit à l'origine d'un bug en 15 manipulations (bon, sous condition que l'environnement capable de faire tourner le projet n'ait pas bougé pendant 6 ans, ce dont je doute fortement).

N'hésitez pas à aller lire mon [article sur git bisect]({% post_url 2016-09-04-la-chasse-aux-bugs-avec-git-bisect %}) si vous souhaitez ajouter une corde à votre arc :)

## Les codes reviews

Avoir des petits commits a un autre intérêt, plus immédiat cette fois : il facilite la relecture.
Sans même parler du monde open-source qui en est très friand, faire des code reviews avant de merger un développement apporte de nombreux avantages :

* ça réduit le nombre d'erreurs bénignes,
* ça encourage la consistance,
* ça facilite l'apprentissage par les autres,
* ça pousse à la discussion pour améliorer la globalité du projet.

Bref c'est super cool.

Par contre c'est difficile à faire parce qu'un code trop long qui fait trop de choses différentes sera très compliqué à comprendre pour le relecteur.

Quand vous codez une fonctionnalité de A à Z qui traverse toutes les couches d'une application MVC vous touchez déjà à énormément de fichiers : l'IHM (souvent séparée en plusieurs fichiers de templates et de styles), le controlleur, le routing, la couche applicative, la DAO ou les données. Ajoutez à ça les tests unitaires qui accompagnent chacune de ces parties, les tests d'intégration, etc. La Pull Request qui contiendra cette fonctionnalité sera donc déjà suffisamment velue pour éviter de la polluer avec des corrections de checkstyle, des p'tits refactos "en passant", de l'amélioration de la couverture de tests sur des parties peu ou pas testées, de la PHPDoc / JavaDoc / WhatEverDoc qui manquait sur certaines fonctions "en passant aussi" et ainsi de suite.

Même en faisant ces modifications uniquement sur les fichiers modifiés par votre fonctionnalité, leur accumulation rendra la Pull Request indigeste aux yeux de vos relecteurs et relectrices. Dans le meilleur cas, ils liront avec attention les premières lignes de la Pull Request mais finiront par bacler au fur et à mesure de la lecture. Dans le pire des cas, la personne ne prendra pas la peine de relire votre Pull Request faute d'avoir suffisamment de temps pour le faire sérieusement et dans son intégralité, préférant la laisser à celles et ceux ayant le temps pour ça (autrement dit personne).

<section class="post-content">
  <div class="jekyll-twitter-plugin">
    <blockquote class="twitter-tweet">
      <p lang="en" dir="ltr">Pull requests:<br />
        <br />
        - 3 files have changed<br />
        &gt; 25 comments in conversation<br />
        <br />
        - 40 files have changed<br />
        &gt; LGTM!
      </p>
      &mdash; I Am Devloper (@iamdevloper) <a href="https://twitter.com/iamdevloper/status/851445899008106496">April 10, 2017</a>
    </blockquote>
    <script async="" src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
  </div>
</section>


Je bla-blate, je bla-blate, mais je n'ai toujours pas attaqué la partie technique de cet article (oui, il n'y a pas que de longs monologues), mais avant d'expliquer comment faire les choses, il vaut mieux faire comprendre l'intérêt qu'il y a derrière, en particulier pour quelque chose qui semble être une perte de temps pour la plupart des gens.

Ok, parlons technique maintenant.

## Alors, comment qu'on fait ?

Il existe une pléthore de techniques pour mieux organiser ses commits avec Git. La première technique ne repose même pas sur la connaissance d'une commande magique de Git : il suffit de rester concentré/e sur sa tâche et de ne pas se disperser dans les petits changements à droite à gauche.

Bon, disons le clairement, c'est une technique pourrie parce que lorsqu'on peut améliorer le code, il ne faut jamais s'en priver. Donc à moins d'aimer recouvrir votre bureau de post-its "enlever l'espace superflu à la ligne 17 du fichier Toto.class.php" que vous dépilez une fois que vous avez fini votre fonctionnalité, ça n'est vraiment pas une bonne méthode.

Non, une bonne méthode doit laisser le développeur ou la développeuse travailler comme il ou elle l'entend et c'est seulement au moment de commiter qu'il ou elle devra réorganiser tous ses changements pour les rassembler en commits.

### Plein de chtits commits

La technique la plus simple consiste à commiter très souvent.

Une réindentation foireuse sur un fichier ? Faites un commit ne contenant que ça.
Une fonction pas assez testée ? Faites un commit ne contenant que le ou les nouveaux tests.
Votre nouvelle fonctionnalité nécessite de factoriser un bout de code ? Factorisez-le et faites le passer dans un commit ne contenant que ça.

Il faut juste que le commit le plus important, celui contenant votre fonctionnalité, soit épuré de toute cette pollution rendant la lecture difficile.

La seule difficulté avec cette approche consiste à résister à la tentation de faire un `git add -A` pour mettre tous les fichiers modifiés dans le commit. Il est également utile de savoir utiliser la commande `git add -p` (`-p` ou `--patch`) permettant de ciseler amoureusement les modifications dans un fichier pour ne rajouter dans un commit que les modifications qui nous intéressent.

Supposons par exemple que j'ai le fichier PHP suivant :

{% highlight php %}
[...]

/**
 * Retrieves a page of items of a specific list
 *
 * @param $listId
 * @param int $page
 *
 * @return Item[]
 */
protected function retrieveList($listId, $page = 1)
{
    return $this->listDAO->retrieveItems($listId, [
        'offset' => ($page - 1) * self::MAX_ITEMS_PER_PAGE,
        'limit' => self::MAX_ITEMS_PER_PAGE
    ]);
}

[...]
{% endhighlight %}

Vous rajoutez votre bout de code dans ce fichier mais en reparcourant le fichier, vous vous rendez compte que dans la PHPDoc de `retrieveList()` le paramètre `$listId` n'est pas typé. Du coup, vous rajoutez le type du paramètre : c'est bénin, ça mange pas de pain et ça facilitera l'utilisation de cette méthode dans l'IDE donc pourquoi s'en priver.

En bon Atomic Commiter, vous allez vouloir commiter ce petit ajout dans un commit séparé mais vous ne voulez pas ajouter votre nouvelle fonction avec ce commit (en plus la fonctionnalité n'est même pas finie). En utilisant la commande `git add -p path/to/MyFile.php`, Git va découper les changements en plusieurs blocs, vous permettant de choisir, pour chaque bloc, si vous voulez l'ajouter ou non au commit.

{% highlight text %}
git add --patch path/to/MyFile.php
diff --git a/path/to/MyFile.php b/path/to/MyFile.php
index 7c534d6..f7a0e49 100644
--- a/path/to/MyFile.php
+++ b/path/to/MyFile.php
@@ -232,7 +232,7 @@ class MyFile
     }

     /**
-     * @param $listId
+     * @param int $listId
      * @param int $page
      *
      * @return Item[]
Stage this hunk [y,n,q,a,d,/,j,J,g,e,?]?
{% endhighlight %}

Si vous tapez `?` Git va vous décrire toutes les options possibles :

{% highlight text %}
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk or any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk or any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
{% endhighlight %}

Dans la pratique, on se cantonne généralement à `y` pour ajouter la modification au commit, `n` pour ne pas ajouter la modification au commit, `s` pour demander à Git de redécouper en blocs plus petits (jusqu'à ce que Git ne soit pas capable de découper plus finement une modification) et `e` pour choisir manuellement, dans la modification présentée, quelles lignes doivent être ajoutées au commit.

Dans notre cas présent, je vais taper `y` pour rajouter le commentaire PHPDoc au commit. Git va ensuite me présenter la modification suivante dans mon fichier :

{% highlight text %}
@@ -387,4 +387,12 @@ class MyFile
     {
         return new JsonResponse(['redirect' => $redirect]);
     }
+
+    /**
+     * @param int $listId
+     */
+    public function deleteList($listId)
+    {
+        $this->listDAO->delete($listId);
+    }
 }
Stage this hunk [y,n,q,a,d,/,K,g,e,?]?
{% endhighlight %}

Là pour le coup il faut que je réponde `n` vu que je ne veux mettre dans mon commit que le correctif sur le PHPDoc de la fonction `retrieveItems()`.

Une fois que j'ai passé en revue toutes mes modifications (à priori j'ai utilisé `n` pour toutes les autres), il ne me reste plus qu'à préparer un commit et à le pusher.

L'avantage de cette méthode c'est que dans un process intégrant des revues de code, ces petits changements seront rapidement lus et mergés pendant que vous continuez à travailler sur votre fonctionnalité.

### Regrouper les fioritures

Une variante de cette technique consiste à appliquer cette méthode à la fin de votre développement, vous permettant de rassembler dans des commits ces petits changements plutôt qu'à les égrenner en une multitude de minuscules commits. Un commit atomique ça ne veut pas dire qu'on doit mettre chaque correctif PHPDoc / indentation / Coding style  dans un commit dédié, on peut très bien les rassembler dans des commits qui ne contiennent que ça.

Le bémol en faisant ça à la fin de votre développement, c'est la relecture de code : même en faisant 10 commits bien séparés, l'outil de relecture de code risque de faire un gros diff de votre branche par rapport à la destination du merge. Par contre il est toujours possible d'indiquer à vos relecteurs qu'il peuvent lire les commits un à un (voire se concentrer sur les commits importants) pour faciliter leur lecture.

![Guider vos utilisateurs vers les commits importants](/assets/images/posts/commits-atomiques-la-bonne-approche/guider-utilisateurs.png) 

Un autre inconvénient quand on effectue ce découpage à la fin, c'est le fait que cela prend pas mal de temps : en voyant le diff, vous allez commencer un commit dédié à un petit truc (par exemple le passage d'une chaîne sous forme de constante) qui va vous obliger à scruter à la loupe tout le contenu de votre diff pour ajouter à ce commit toutes les éditions qui concernent ce changement et uniquement celles-là. Quand vous aurez fini, vous allez recommencer avec un autre commit atomique et de nouveau relire tout votre diff (du moins tout ce qu'il reste) pour de nouveau ajouter tout ce qui concerne ce second changement. Et ainsi de suite. Du coup vous allez relire votre diff un paquet de fois.

### Le rebase interactif

Une autre technique qui marche super bien, c'est via l'utilisation du rebase interactif.
Le rebase interactif permet de changer l'ordre des commits, de les éditer et de les fusionner ensemble afin de les réorganiser à loisir.

Cela signifie que le développeur ou la développeuse, une fois son travail terminé ou même au fil de son développement, va pouvoir créer plein de petits commits contenant les petites choses qu'il ou elle aura fixées à droite à gauche et qui ne sont pas directement liées à son travail.

Une fois qu'il ou elle sera prêt à pusher sa Pull Request, le rebase interactif lui permettra de fusionner les commits pouvant être contenus dans un unique commit ou éventuellement réorganiser ses commits, par exemple pour faire une première Pull Request contenant toutes les fioritures afin de la merger rapidement et avoir ensuite une seconde Pull Request ne montrant que la vraie fonctionnalité attendue.

Prenons un exemple.

Je suis en train de travailler sur la tâche *GRE-1234*, dont le but est d'autoriser un utilisateur ou une utilisatrice à éditer ses réponse dans un sujet de forum.
Au cours de mon travail, je corrige tous les petits trucs que je vois à droite à gauche et je commit fréquemment ces petits trucs. Une fois content de ma fonctionnalité, je me retrouve avec 15 commits :

{% highlight text %}
$ git log -15 --oneline
a5c4c94 misnamed variable
eee1b54 phpdoc
9e90a6f dead code
68b3bc8 Missing property declaration
9234889 dead code
e531cce phpdoc
7a9d433 [GRE-1234] - Don't allow answer edition on closed thread
ae03f4b Behat description
c69f370 Reference to obsolete class
e7e0f59 phpdoc
1a96fc2 Behat description of scenario
5f8fe5f [GRE-1234] - User can edit his answers
8593186 Missing description in Behat scenario
7bc77ef Dead code
ab00199 Missing description in Behat scenario
{% endhighlight %}

On voit que dans ces 15 commits, certains sont très similaires et pourraient être regroupés. Ma fonctionnalité en elle-même n'est répartie que dans deux commits préfixés par l'identifiant de ma tâche. A moins d'avoir une bonne raison de séparer la fonctionnalité en deux, je devrais fusionner ces deux commits en un seul.

Bref, essayons de réorganiser tout ça.

La commande `git rebase -i HEAD~15` va me permettre de peaufiner facilement le rebase des 15 derniers commits.

En lançant cette commande, cela va m'ouvrir l'éditeur de texte dont le contenu sera la liste de mes 15 commits précédés par `pick` suivi d'instructions en commentaire pour me guider. Attention, l'ordre des commits est l'ordre chronologique : mon plus vieux commit est le premier, le plus récent est le dernier. C'est l'inverse de l'affichage du `git log` avec lequel on est plus familier.

{% highlight text %}
pick ab00199 Missing description in Behat scenario
pick 7bc77ef Dead code
pick 8593186 Missing description in Behat scenario
pick 5f8fe5f [GRE-1234] - User can edit his answers
pick 1a96fc2 Behat description of scenario
pick e7e0f59 phpdoc
pick c69f370 Reference to obsolete class
pick ae03f4b Behat description
pick 7a9d433 [GRE-1234] - Don't allow answer edition on closed thread
pick e531cce phpdoc
pick 9234889 dead code
pick 68b3bc8 Missing property declaration
pick 9e90a6f dead code
pick eee1b54 phpdoc
pick a5c4c94 misnamed variable

# Rebase 0b9574c..a5c4c94 onto 0b9574c (15 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
{% endhighlight %}

Se contenter des options `pick` et `squash` et réordonner les commits est généralement suffisant. Commençons déjà par réordonner les commits pour obtenir ceci :

{% highlight text %}
pick ab00199 Missing description in Behat scenario
pick 8593186 Missing description in Behat scenario
pick 1a96fc2 Behat description of scenario
pick ae03f4b Behat description
pick 7bc77ef Dead code
pick 9234889 dead code
pick 9e90a6f dead code
pick e7e0f59 phpdoc
pick e531cce phpdoc
pick eee1b54 phpdoc
pick c69f370 Reference to obsolete class
pick 68b3bc8 Missing property declaration
pick a5c4c94 misnamed variable
pick 5f8fe5f [GRE-1234] - User can edit his answers
pick 7a9d433 [GRE-1234] - Don't allow answer edition on closed thread
{% endhighlight %}

J'ai rassemblé les commits similaires et j'ai mis ma fonctionnalité en tout dernier.

Maintenant, je peux choisir de merger certains commits entre eux. Pour ça, il suffit que je remplace le libellé `pick` en début des lignes qui m'intéressent par `squash` (ou `s`).
Par exemple les quatre premiers commits sont similaires : ils rajoutent des descriptions manquantes dans des scénarios Behat. Je peux donc me contenter de garder le premier des quatre commits tel quel et demander à ce que les trois suivants soient mergés dans le premier :

{% highlight text %}
pick ab00199 Missing description in Behat scenario
squash 8593186 Missing description in Behat scenario
squash 1a96fc2 Behat description of scenario
squash ae03f4b Behat description
{% endhighlight %}

Je répète ce procédé sur les autres "groupes" de commits, jusqu'à obtenir quelque chose qui ressemble à ça :

{% highlight text %}
pick ab00199 Missing description in Behat scenario
squash 8593186 Missing description in Behat scenario
squash 1a96fc2 Behat description of scenario
squash ae03f4b Behat description
pick 7bc77ef Dead code
squash 9234889 dead code
squash 9e90a6f dead code
pick e7e0f59 phpdoc
squash e531cce phpdoc
squash eee1b54 phpdoc
pick c69f370 Reference to obsolete class
pick 68b3bc8 Missing property declaration
pick a5c4c94 misnamed variable
pick 5f8fe5f [GRE-1234] - User can edit his answers
squash 7a9d433 [GRE-1234] - Don't allow answer edition on closed thread
{% endhighlight %}

Si je sauve mon éditeur et que je le ferme, celui-ci va se rouvrir, cette fois-ci pour me demander d'éditer le commentaire du premier pack de quatre commits que je vais merger :

{% highlight text %}
# This is a combination of 4 commits.
# The first commit's message is:
Missing description in Behat scenario

# This is the 2nd commit message:

Missing description in Behat scenario

# This is the 3rd commit message:

Behat description of scenario

# This is the 4th commit message:

Behat description

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon Apr 11 23:02:10 2016 +0200
#
# interactive rebase in progress; onto 0b9574c
# Last commands done (4 commands done):
#    squash 1a96fc2 Behat description of scenario
#    squash ae03f4b Behat description
# Next commands to do (11 remaining commands):
#    pick 7bc77ef Dead code
#    squash 9234889 dead code
# You are currently editing a commit while rebasing branch 'my-branch' on '0b9574c'.
{% endhighlight %}

Je peux choisir d'enlever tout le contenu de l'éditeur et le remplacer par un message unique *"Missing description in Behat scenario"*.

En fermant l'éditeur, il va de nouveau se rouvrir pour recommencer avec le groupe de commits suivant et
ainsi de suite jusqu'au dernier groupe, où il me rend la main.

Si j'avais voulu sauter cette étape et me contenter à chaque fois d'utiliser le commentaire du tout premier commit de chaque groupe, j'aurais pu utiliser `fixup` (ou `f`) à la place de `squash` dans le rebase interactif. C'est l'unique différence entre les deux : `squash` va également fusionner les commentaires des commits tout en laissant la possibilité à l'utilisateur de réditer ce commentaire agrégé alors que fixup va utiliser le commentaire du commit dans lequel on se merge.

C'est fini, si je regarde mon historique de commits, j'ai maintenant 7 commits là où j'en avais 15 :

{% highlight text %}
$ git log -7 --oneline
421123f [GRE-1234] - User can edit his answers
03b59fe misnamed variable
0fa14f1 Missing property declaration
4ed3679 Reference to obsolete class
94d3208 PHPDoc
5220758 Dead code
86855ae Missing description in Behat scenario
{% endhighlight %}

A ce stade, je peux envoyer ça sous forme de Pull Request en précisant à mes collègues de se cantonner au commit contenant la fonctionnalité et à ne pas s'embêter avec les commits précédents.

Je peux aussi créer une branche sur l'avant dernier commit (`git checkout -b small-fixes HEAD~1`) pour envoyer tous ces petits commits sans grand intérêt dans une Pull Request qui sera rapidement mergée (voire que je mergerai moi-même), pour ensuite faire une Pull Request ne contenant que le commit de ma fonctionnalité.

Attention, le rebase interactif marche bien quand les commits ne sont pas étroitement liés ou bien en prenant garde à ce que l'on fait. Par exemple sur les deux commits de ma fonctionnalité, le second est probablement très dépendant du code du premier vu que visiblement le second commit fixe un cas que j'avais oublié dans le premier commit. Si dans le rebase interactif je change l'ordre de ces deux commits, ça risque d'assez mal se passer.

Par mesure de précaution, n'hésitez pas à créer une branche temporairement (`git branch tmp-save` par exemple) avant de vous lancer dans un rebase (interactif ou non) pour pouvoir revenir à cet endroit en cas de pépin.

### Autosquash

Un autre truc sympa avec le rebase interactif, c'est le mécanisme d'autosquash. Plutôt que de réorganiser et merger les commits tout à la fin, vous pouvez préparer le terrain pendant la phase de développement. Lorsque vous créez un commit, vous pouvez d'ores et déjà signaler dans quel commit il devra être mergé ultérieurement.

Reprenons mes 15 commits de tout à l'heure :

{% highlight text %}
$ git log -15 --oneline
a5c4c94 misnamed variable
eee1b54 phpdoc
9e90a6f dead code
68b3bc8 Missing property declaration
9234889 dead code
e531cce phpdoc
7a9d433 [GRE-1234] - Don't allow answer edition on closed thread
ae03f4b Behat description
c69f370 Reference to obsolete class
e7e0f59 phpdoc
1a96fc2 Behat description of scenario
5f8fe5f [GRE-1234] - User can edit his answers
8593186 Missing description in Behat scenario
7bc77ef Dead code
ab00199 Missing description in Behat scenario
{% endhighlight %}

J'ai commencé par créér le tout premier commit (`ab00199 Missing description in Behat scenario`), puis le second (`7bc77ef Dead code`). En créant le 3ème commit (`8593186 Missing description in Behat scenario`) j'aurais pu me dire *"Hey, c'est la même chose que le premier commit, je devrais fusionner ce code dedans"*.

Pour faire ça, c'est très facile, il suffit d'utiliser le paramètre `--fixup` à la commande `git commit` en lui précisant le commit dans lequel on veut se merger.

On peut faire ça en précisant le SHA1 du commit : `git commit --fixup ab00199`. Ou bien de manière relative : `git commit --fixup HEAD~1` (l'avant dernier commit). Ou encore en lui demandant de rechercher le plus récent commit matchant une recherche : `git commit --fixup :/Behat`

Une fois ceci fait, en regardant les trois derniers commits vous devriez avoir ceci :

{% highlight text %}
$ git log -3 --oneline
ca9260b fixup! Missing description in Behat scenario
7bc77ef Dead code
ab00199 Missing description in Behat scenario
{% endhighlight %}

Supposons que j'ai appliqué cette façon de procéder sur mes 15 commits, j'obtiens normalement ceci à la fin :

{% highlight text %}
$ git log -15 --oneline
93cae94 Misnamed variable
d649734 fixup! PHPDoc
85327bf fixup! Dead code
ea21220 Missing property declaration
173c7ac fixup! Dead code
1dace99 fixup! PHPDoc
8ccf516 fixup! [GRE-1234] - User can edit his answers
b8a4e94 fixup! Missing description in Behat scenario
cad8d41 Reference to obsolete class
b68e1e2 PHPDoc
223d27e fixup! Missing description in Behat scenario
4f867ae [GRE-1234] - User can edit his answers
ca9260b fixup! Missing description in Behat scenario
8e6a009 Dead code
b95f418 Missing description in Behat scenario
{% endhighlight %}

Il ne me reste plus qu'à utiliser le rebase intéractif en lui précisant l'option `--autosquash` pour qu'il merge les commits tagués avec `!fixup` dans les commits adéquats. Il va m'ouvrir l'éditeur dont le contenu sera le suivant :

{% highlight text %}
pick b95f418 Missing description in Behat scenario
fixup ca9260b fixup! Missing description in Behat scenario
fixup 223d27e fixup! Missing description in Behat scenario
fixup b8a4e94 fixup! Missing description in Behat scenario
pick 8e6a009 Dead code
fixup 173c7ac fixup! Dead code
fixup 85327bf fixup! Dead code
pick 4f867ae [GRE-1234] - User can edit his answers
fixup 8ccf516 fixup! [GRE-1234] - User can edit his answers
pick b68e1e2 PHPDoc
fixup 1dace99 fixup! PHPDoc
fixup d649734 fixup! PHPDoc
pick cad8d41 Reference to obsolete class
pick ea21220 Missing property declaration
pick 93cae94 Misnamed variable
{% endhighlight %}

Il me reste plus qu'à fermer l'IDE pour que le rebase s'achève.

{% highlight text %}
$ git log -7 --oneline
f0232ac Misnamed variable
7716fcf Missing property declaration
bbc9a87 Reference to obsolete class
3160938 PHPDoc
10396ac [GRE-1234] - User can edit his answers
53f5b7b Dead code
fa1188f Missing description in Behat scenario
{% endhighlight %}

L'ordre des commits est différent de celui qu'on a obtenu sans l'option `--autosquash` mais si cela pose problème il est toujours possible de refaire un rebase intéractif sur les commits pour les réordonner (`git rebase -i HEAD~7`).

Plutôt que de commiter avec `git commit --fixup [SHA1]`, je peux aussi utiliser `git commit --squash [SHA1]` qui va fonctionner de la même manière. La seule différence c'est la possibilité de remodifier les commentaires des commits à la fin du rebase pour les commits ayant été créés avec `--squash`.

## Le mot de la fin

Au début de ce pavé je vous ai parlé de `git bisect` afin de rechercher de manière dichotomique un bug dans le code. Il faut savoir que pour que cette commande fonctionne correctement il est nécessaire que les commits ne soit pas mal découpés. En théorie, vous devez pouvoir prendre n'importe quel duo de commits qui se suivent dans votre repository, vous placer entre les deux commits et être capable d'utiliser votre application sans que cela ne pose le moindre problème (y compris faire tourner les tests). 

Du coup, plus de commits *"Fixing broken test"* / *"Fixing error seen in CI"* / *"Fixing layout"* : si un commit casse quelque chose (un test ou une partie de l'application), le fix de ce problème doit être mergé dans le même commit et non faire l'objet d'un second commit, sans quoi il sera possible, au détour d'un git bisect, de se retrouver entre les deux commits et avoir une application cassée plus ou moins partiellement.

Bon, j'espère que tout ce bla-bla et ces techniques vous encourageront à ne plus utiliser `git add -A` ou `git commit -a` ainsi qu'à découper minutieusement vos commits pour augmenter leur lisibilité et la facilité à trouver un bug en cas de pépin.