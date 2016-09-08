---
layout: post
title:  "Les 1001 façons de référencer un commit"
date:   2016-09-08 23:00:00 +0200
categories: astuces
disqus: true
excerpt_separator: <!--more-->
filename: 2016-09-08-les-1001-facons-de-referencer-un-commit.markdown
---

Des façons de référencer un commit, il en existe des pleines brouettes. Bon, des petites brouettes mais il y en a quand même pas mal. On va essayer de les passer en revue.
<!--more-->

## Le SHA1

La plus classique, c'est bien entendu le SHA1 du commit. Pour le trouver c'est facile, il suffit d'utiliser `git log` pour ça.

{% highlight text %}
> git log -1 aed7bf3c9cdecf3686b80f81eb40d000fa71797a
commit aed7bf3c9cdecf3686b80f81eb40d000fa71797a
Author: Loïc Giraudel <lgiraudel@purch.com>
Date:   Fri Sep 2 16:25:25 2016 +0200

    [BHPR-772] - User with 1 best answer or more should not see vibrant ads
{% endhighlight %}

Mais quand vous référencez un commit par son SHA1, vous n'êtes pas obligé d'utiliser tout le SHA1 mais uniquement les premiers caractères. En fait il faut juste qu'il y ait au moins 4 caractères et qu'il n'y ait qu'un seul commit dans le repository qui commence par la chaîne saisie pour que ça fonctionne.

Sur un tout petit repository, il est donc possible de référencer un commit avec seulement 4 caractères :
{% highlight text %}
> git log 185d
commit 185df75b02129a71f5d62e895957a70fec8630bf
Author: Loïc Giraudel <lgiraudel@purch.com>
Date:   Wed Sep 7 21:49:49 2016 +0200

    First commit
{% endhighlight %}

Exemples de cas d'utilisation :

* `git log [SHA1]`
* `git cherry-pick [SHA1]`

## Le HEAD

Le HEAD, c'est facile, c'est le commit sur lequel vous vous trouvez.

Il existe également :

* FETCH_HEAD => la dernière référence synchronisée lors du dernier `git fetch`
* ORIG_HEAD => la dernière position du HEAD avant la position courante, pour pouvoir y revenir rapidement. Rien à voir avec le remote "origin".
* MERGE_HEAD => le commit de destination lors d'un merge
* CHERRY_PICK_HEAD => le commit cherry-pické.

Exemples de cas d'utilisations :

* `git reset HEAD`

## Les branches

Et oui, les branches ne sont que des références vers des commits, du coup il est possible d'utiliser un nom de branche pour référencer le dernier commit de la-dite branche. Ca marche également avec les branches distantes et il est également possible d'utiliser plusieurs syntaxes pour référencer une branche (`master` ou `heads/master` ou `refs/heads/master`).

Exemples de cas d'utilisation :

* `git checkout ma-branche`
* `git cherry-pick mon-autre-branche`

## Les tags

Comme les branches, les tags sont aussi des références vers des commits. Là aussi on peut utiliser plusieurs syntaxes (`mon-tag`, `tags/mon-tag`, `refs/tags/mon-tag`).

Exemple de cas d'utilisation :

* `git checkout 1.0.0`
* `git log last-version-in-QA`

## Le symbole `^`

Le symbole `^` peut s'utiliser pour référencer le commit parent d'un commit. `HEAD^` représente donc l'avant dernier commit. `1.1.0^` représente l'avant dernier commit du tag (ou de la branche) "1.1.0".

On peut le chaîner : `HEAD^^` est l'avant-avant-dernier commit. Mais ça devient vite illisible donc je vous le déconseille.

Lorsqu'un commit a plusieurs parents, c'est par exemple le cas d'un merge, le symbole `^` permet de choisir le parent quand on lui ajoute un entier. Ainsi `HEAD^1` représente le premier parent du commit alors que `HEAD^2` représente le second parent (et `HEAD^0` représente le commit lui-même). `HEAD^` est donc un raccourci pour `HEAD^1`.

Exemples de cas d'utilisation :

* `git reset HEAD^`
* `git diff HEAD^`

## Le symbole `~`

Le symbole `~` s'utilise avec un entier pour indiquer l'ancêtre d'un commit. `HEAD~10` représente le 10ème commit avant le dernier. `1.1.0~3` représente le 3ème commit avant le commit pointé par le tag (ou la branche) "1.1.0".

Vous l'aurez compris, `HEAD~1` est équivalent à `HEAD^` et personnellement j'ai tendance à utiliser le `~` plutôt que le `^`.

Exemples de cas d'utilisation :

* `git reset HEAD~1`
* `git rebase -i HEAD~5`

## Le symbole `@`

`@` est un raccourci pour le HEAD. Pratique !

## Les références temporelles

On peut référencer un commit basé sur sa date avec la syntaxe @{YYYY-MM-DD}. Par exemple `git log @{2016-07-04}` va me faire un git log à partir du 4 juillet.

Il est également possible de faire du relatif :

* `git log @{yesterday}`
* `git log origin/master@{one month ago}`

Note : pour que ça fonctionne, il faut qu'il y ait un reflog existant pour la branche concernée.

## La dernière / avant-dernière / avant-avant-dernière position d'une branche

La syntaxe `@{X}` permet de référencer la Xème ultérieure position sur laquelle se trouvait le commit référencé.

Par exemple `HEAD@{1}` ou simplement `@{1}` représente la position qu'avait le HEAD avant sa position courante (soit ORIG_HEAD en toute logique). `master@{5}` représente la 5ème position qu'avait la branche master avant sa position courante.

Note : pour que ça fonctionne, il faut qu'il y ait un reflog existant pour la branche concernée.

## Le dernier commit ou la dernière branche checkouté

La syntaxe @{-X} permet de récupérer le Xème commit (ou branche) récupéré avec un checkout. Très pratique quand on switch temporairement de branche. Du coup généralement on se cantonne à `@{-1}` vu qu'il est rare de jongler avec plus de deux branches à la fois.

## @{upstream}

`@{upstream}` ou `@{u}` représente la branche trackée par notre branche courante. On peut même référencer la branche traquée par une autre branche : `myOtherBranch@{u}`

## @{push}

`@{push}` représente la destination de la branche courante (ou de la branche référencée avant `@{push}`) si on faisait un `git push`. La plupart du temps ça a la même valeur que `@{upstream}`, sauf si la branche a été configurée pour être pusher vers une destination particulière (ou si la configuration par défaut de git a été modifiée).

## La référence basée sur le message de commit

Il est possible de référencer un commit en se basant sur son message. `git show :/foobar` va m'afficher le commit le plus récent dans l'historique (quelle que soit la branche) contenant "foobar" dans son message.

Et si on ne veut pas que ça s'applique sur n'importe quelle branche mais que ça soit forcément l'ancêtre d'un commit particulier, on peut utiliser la syntaxe suivante : `commit^{/foobar}`.

## Le résultat de `git describe`

`git describe` retourne une chaîne de caractères décrivant le nom d'un commit par rapport au dernier tag annoté. S'il y a un tag annoté sur le commit ça retourne le nom de ce tag annoté, sinon ça retourne le nom du tag annoté le plus proche suivi de la distance (en nombre de commits) avec ce tag annoté suivi du SHA1 (abrégé) du commit.

Honnêtement je me suis jamais servi de cette commande et j'ignore quels cas d'utilisation elle peut avoir mais en tout cas il est possible de référencer un commit en utilisant la sortie de cette commande.

Voilà, ça vous fait une belle jambe.

## Et d'autres syntaxes exotiques

Je ne vous parlerai pas des syntaxes `HEAD^{}`, `HEAD^{<type>}`, n'étant pas sur de bien les comprendre moi-même. Mais elles fonctionnent aussi :)

