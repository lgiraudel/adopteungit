---
layout: post
title:  "Les bonnes pratiques"
date:   2016-08-16 12:00:00 +0200
categories: methodologie
disqus: true
#excerpt_separator: <!--more-->
filename: 2016-08-16-les-bonnes-pratiques.markdown
---
Une bonne utilisation de Git repose sur un petit nombre de pratiques à respecter pour s'éviter de facheux désagréments, que ce soit sur le moment ou bien plus tard lorsqu'on revient sur d'anciens commits.

### Des p'tits commits
La première règle du git club c'est de faire des petits commits. Je vous encourage à vous faire tatouer cette règle sur l'épaule pour ne jamais l'oublier (cela dit c'est pas facile de lire ce qui est marqué sur sa propre épaule donc faites la tatouer sur l'épaule de votre voisin plutôt).

Oubliez tout ce qu'on a pu vous dire par le passé : si, **la taille ça compte**.

Faire des petits commits vous facilitera la vie dans plein de circonstances :

- ça rendra les commits plus lisibles parce qu'ils se bornent à une seule tache,
- si vous en faites (faites-en !), ça donnera lieu à des code reviews bien plus fluides,
- ça facilitera la manipulation des commits (revert, cherry-pick, rebase)
- ça vous permettra de faire du bisect les doigts dans le tarin
- ça augmentera votre nombre de commits et du coup vous permettra de briller dans les diners mondains,
- ça assainira votre peau et lustrera vos cheveux (ou vos poils de barbe si vous êtes chauve).

Je ne vais pas rentrer dans le détail de chacun de ces points, certains seront probablement couverts par des articles détaillés sur ce blog. Nous essaierons également de donner des astuces pour faire des petits commits parce que ça n'est pas toujours évident.

### git commit -m "Oops, I did it again"
La seconde règle est assez universelle dans le monde de la programmation : facilitez la vie des autres (ou de votre futur vous) en mettant de la documentation et des noms explicites. Dans le cas de Git, ça consiste donc à faire des messages de commits précis.

Arrêtez les "Fix bug" ou "Oups, j'avais oublié un truc" dans les messages de commit. Soyez explicites, décrivez le bug que vous avez fixé, la fonctionnalité que vous avez ajouté. Les seuls messages de commit qui peuvent être vagues sont ceux pour des tâches annexes, genre fix d'erreurs checkstyle.

Si vous utilisez un gestionnaire de tâches, n'hésitez pas à mettre l'ID de votre story / tâche / bug dans votre commit, d'autant plus que les interfaces web des outils permettent parfois de coupler les deux (liens vers les commits depuis la story ou lien vers la story depuis les commits).

![Messages de commits](https://imgs.xkcd.com/comics/git_commit.png)

### Respecter ton master tu dois
Une autre règle que vous pouvez vous faire tatouer (après tout vous avez deux épaules), c'est le fait de ne pas casser le master. Quand je dis "casser", ça veut dire que l'application ou ses tests ne marchent plus.

C'est d'autant plus crucial en équipe si on ne veut pas se retrouver avec des développeurs bloqués suite à un rebase du master et qui pensent que l'appli est cassée à cause de leur code. Je vous encourage d'ailleurs vivement à mettre en place une sécurité sur votre repository : interdire le push dans le master et mettre une exécution des tests **avant** un merge automatisé dans le master. Et comble du luxe, vous pouvez même greffer un système de Pull Requests dans ce workflow.

Si vous faites un commit qui casse le master, plutôt que de faire un second commit qui répare ce qui est pété (avec un "Oups" bien parlant en guise de message de commit) préférez faire un `git commit --amend` pour modifier votre précédent commit. Ca a l'avantage de ne pas casser l'application entre deux commits, ce qui compliquerait un éventuel futur `git bisect`.
Bien entendu, ça sous-entend qu'il faut bien vérifier avant de pusher vers le repository distant.

### L'usage de la force
Utilisez `git push --force` avec parcimonie à moins de bien maîtriser Git et d'être certain(e) de ce que vous êtes en train de faire. Un push forcé est le meilleur moyen de vous faire des ennemis soit en écrasant le travail des autres, soit, et c'est encore plus fréquent, en compliquant la re-synchronisation d'une branche pour les autres. On reviendra en détail sur le `--force` dans un article dédié.

### Prudence est mère de sureté
Allez, un dernier conseil pour la route : n'hésitez pas à faire des branches temporaires avant une opération potentiellement source de conflit (un rebase ou un merge par exemple). Il vaut mieux créer une branche juste avant plutôt que de galérer à annuler une opération compliquée qui se passe mal et il est toujours possible de la supprimer juste après si tout s'est bien passé.

![Les difficultés de Git](https://imgs.xkcd.com/comics/git.png)

### Si on résume...

* des petits commits
* des messages de commit explicites
* ne pas casser le master
* ne pas utiliser `--force`
* ne pas hésiter à créer des branches de sauvegarde
