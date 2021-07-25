---
layout: post
title:  "Git, les principes de base pour le prendre en main"
lang: fr
ref: git-practical-very-basics
date: 2017-07-02 19:30:00 +0200
categories: methodologie
disqus: true
excerpt_separator: <!--more-->
filename: 2017-07-02-git-les-principes-de-base-pour-le-prendre-en-main.markdown
author: kitty-giraudel
translator: loic-giraudel
---

*Cet article est la traduction d'un article écrit par [Kitty](https://twitter.com/KittyGiraudel), pour aider une de ses amies à prendre en main git. Je lui suis très reconnaissant de nous faire profiter de cet article très utile !*

-----

Git est un logiciel de gestion de versions. En pratique, c'est un petit programme qui maintient un historique des changements sur un projet comme les ajouts, les suppressions ou les éditions…

Git est une bête. C'est l'une de ces choses qui paraissent terrifiantes au premier abord, au point que seules quelques personnes parviennent à la maîtriser. Mais elle n'est pas forcément difficile à comprendre.

Récemment, j'ai eu à enseigner les principes de base de Git à une amie et j'ai pensé qu'il pourrait être intéressant de partager cet enseignement en ligne pour que d'autres personnes puissent en profiter. Voici une esquisse de ce qui est ressorti de notre échange.
<!--more-->

## Création d'un dépôt

La commande `init` initialise un “dépôt git” dans le répertoire courant. Elle crée un répertoire (caché) `.git` avec tout un tas de sous-répertoires dont vous n'avez pas forcément besoin de connaître l'utilité.

{% highlight text %}
$ git init

> Initialized empty Git repository in /Users/Path/To/Your/Folder/.git/
{% endhighlight %}

Git n'a pas nécessairement besoin d'un dépôt en ligne (GitBub, Bitbucket…). Vous pouvez avoir un projet git local n'étant pas sauvegardé en ligne. Mais si vous voulez le sauvegarder, vous aurez besoin d'un dépôt “distant” qui est tout simplement une copie de votre projet sur un serveur (par exemple GitHub).

La commande `remote` a des sous-commandes pour manipuler les dépôts distants. Ce n'est pas une commande que vous aurez à utiliser fréquemment, mais vous en aurez besoin la toute première fois où vous souhaiterez sauvegarder votre projet sur GitHub. Plus exactement, vous aurez besoin d'utiliser la sous-commande `add` pour lier votre dépôt distant à votre projet git local.

Ici, le libellé “origin” sert de nom pour le repository distant et c'est le nom par défaut lorsque vous “clonez” un projet depuis GitHub, du coup nous respecterons cette convention en nommant notre dépôt distant “origin”.

{% highlight text %}
$ git remote add origin https://github.com/VotrePseudoGitHub/VotreProjet
{% endhighlight %}

## Faire un commit

Appliquons une modification dans notre projet. Nous allons le faire via la ligne de commande pour plus de simplicité mais cette modification peut également être faite manuellement avec un éditeur de fichiers.

Créons un fichier “README.md” contenant la chaîne “Hello world”.

{% highlight text %}
$ echo "Hello world" > README.md
{% endhighlight %}

Il est maintenant possible de faire notre tout premier commit. Un commit est un regroupement d'un ou plusieurs changements (ajouts, suppressions, éditions) appliqués à notre projet à un moment précis dans le temps. Une suite de commits est donc un historique des changements sur notre projet. C'est un principe de base.

Ecrire dans l'historique se fait en deux étapes : d'abord, nous devons ajouter les fichiers concernés à une sorte d'état transitoire appelé l'état “staging”. Ensuite nous devons demander à git d'enregistrer tous les changements dans les fichiers de cet état transitoire dans un commit. La première étape se fait grâce à la commande `add`.

Ici, nous allons ajouter le fichier `README.md` à l'état staging pour pouvoir ensuite préparer un commit.

{% highlight text %}
$ git add README.md
{% endhighlight %}

Il est possible de voir quels fichiers ont été mis en staging avec la commande `status`. Elle va afficher la liste des fichiers ayant été modifiés (ou ajoutés ou supprimés) et signaler pour chacun d'entre eux s'il a été ajouté au staging.

{% highlight text %}
$ git status

> Changes to be committed:
>   (use "git rm --cached <file>..." to unstage)
>
>   new file:   README.md
{% endhighlight %}

Maintenant que nous avons nos fichiers en staging, nous pouvons créer un commit avec la commande `commit`. Nous associons un message à ce commit avec l'option `-m`, afin de décrire les changements appliqués et la raison de ces changements.

{% highlight text %}
$ git commit -m "Ajout d'un fichier README.md"

> [master (root-commit) e63214c] Ajout d'un fichier README.md
> 1 file changed, 1 insertion(+)
> create mode 100644 README.md
{% endhighlight %}

Nous pouvons maintenant de nouveau regarder notre état avec la commande `status`. Il doit être vide étant donné que nous n'avons aucun autre changement en cours.

{% highlight text %}
$ git status

> On branch master
> nothing to commit, working tree clean
{% endhighlight %}

Si nous souhaitons inspecter l'historique récent du projet, nous pouvons utiliser la commande `log` qui va lister les commits en partant du dernier.

>  • Ajout d'un fichier README.md (e63214c)  
>  └─ Votre Nom &lt;votre.nom@gmail.com&gt;

{% highlight text %}
$ git log

> commit e63214cf6fe03954fd8cbc7ee72ff913ec65c8b9
> Author: Votre Nom <votre.nom@gmail.com>
> Date:   Sat Jun 17 12:37:02 2017 +0200
>
>   Ajout d'un fichier README.md
{% endhighlight %}

## Envoyer tout ça en ligne

Maintenant que nous avons fait notre première contribution dans l'historique du projet, nous voulons sauvegarder ce changement sur notre dépôt GitHub. Cela permettra à d'autre personnes de voir ce changement (dans le cas d'un dépôt public), mais ça permettra également de récupérer ces modifications sur n'importe quel autre ordinateur.

Pour pousser ces changements sur le dépôt distant (configuré précedemment avec la commande `remote`), nous utilisons la commande `push`, suivi par le nom du dépôt distant (`origin`) et le nom de la branche à pousser (`master`).

{% highlight text %}
$ git push origin master

> Counting objects: 3, done.
> Writing objects: 100% (3/3), 237 bytes | 0 bytes/s, done.
> Total 3 (delta 0), reused 0 (delta 0)
> To github.com:VotrePseudoGitHub/VotreProjet.git
>  * [new branch]      master -> master
{% endhighlight %}

## Fusion de branches

Sur un projet git, il est possible de travailler sur plusieurs “branches”. Les branches sont des bifurcations dans l'historique git à certain moment dans le temps. On peut par la suite rassembler plusieurs branches pour regrouper les différents changements appliqués sur chacune des branches.

Il est possible de lister les branches de votre projet avec la commande `branch`.

{% highlight text %}
$ git branch

> * master
{% endhighlight %}

En passant un nom à la commande `branch`, nous pouvons créer une nouvelle branche. Nous pouvons ensuite nous positionner sur cette nouvelle branche avec la commande `checkout`. Il est possible de combiner ces deux actions en une seule avec la commande `checkout -b <nom_de_la_nouvelle_branche>`.

{% highlight text %}
$ git branch ma-nouvelle-branche
$ git checkout ma-nouvelle-branche

> Switched to branch 'ma-nouvelle-branche'
{% endhighlight %}

Regardons la liste de nos branches maintenant que nous en avons plusieurs :

{% highlight text %}
$ git branch

>   master
> * ma-nouvelle-branche
{% endhighlight %}

Ajoutons du contenu au fichier README.md, puis créons un commit avec nos modifications.

{% highlight text %}
$ echo "peekaboo" >> README.md
$ git add README.md
$ git commit -m "Ajout de peekaboo au fichier README"

> [ma-nouvelle-branche e281fd0] Ajout de peekaboo au fichier README
> 1 file changed, 1 insertion(+)
{% endhighlight %}

Nous pouvons regarder les logs pour nous assurer que tout est comme nous le souhaitons. Comme vous le voyez, `ma-nouvelle-branche` contient également le commit de `master` créé précédemment. C'est parce que notre nouvelle branche a été créée depuis le commit sur la branche master, elle contient donc tout l'historique précédant ce commit.

{% highlight text %}
$ git log

> commit e281fd051af7d037486f6eec420664cf08fc9418
> Author: Votre Nom <votre.nom@gmail.com>
> Date:   Sat Jun 17 12:52:26 2017 +0200
> 
>     Ajout de peekaboo au fichier README
> 
> commit e63214cf6fe03954fd8cbc7ee72ff913ec65c8b9
> Author: Votre Nom <votre.nom@gmail.com>
> Date:   Sat Jun 17 12:37:02 2017 +0200
> 
>     Ajout d'un fichier README.md
{% endhighlight %}

Si nous sommes satisfaits de nos changements sur la nouvelle branche, nous pouvons décider d'appliquer ces changements dans `master` (la branche principale). Pour le faire, nous pouvons “incorporer” (“merge” en anglais) la nouvelle branche dans la branche principale. D'abord, nous devons retourner sur la branche `master`, ensuite nous devons utiliser la commande `merge` pour intégrer les commits de `ma-nouvelle-branche` qui ne sont pas déjà présents dans la branche `master`.

{% highlight text %}
$ git checkout master

> Switched to branch 'master'

$ git merge ma-nouvelle-branche

> Updating e63214c..e281fd0
> Fast-forward
> README.md | 1 +
> 1 file changed, 1 insertion(+)
{% endhighlight %}

## Conclusion

C'est à peu prés tout. Bien sûr cela devient plus compliqué quand on commence à revenir dans l'historique pour le modifier rétrospectivement, réappliquer des commits dans un ordre particulier sur une autre branche, éviter les conflits quand deux branches qu'on souhaite fusionner ont toutes les deux modifier le même fichier, mais vous n'avez pas forcément besoin de connaître toutes ces techniques pour débuter avec git.

C'est tout ce que vous avez besoin de connaître pour commencer à jouer avec git. Amusez-vous bien !
