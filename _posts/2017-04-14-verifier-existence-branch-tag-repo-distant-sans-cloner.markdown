---
layout: post
title:  "Comment vérifier l'existence d'une branche ou d'un tag sur un repository distant sans le cloner"
date:   2017-04-14 12:00:00 +0200
categories: blog
disqus: true
excerpt_separator: <!--more-->
filename: 2017-04-14-verifier-existence-branch-tag-repo-distant-sans-cloner.markdown
---

Quand on commence à écrire des scripts pour automatiser le packaging et le déploiement d'une application, il peut arriver qu'on veuille vérifier, avant de cloner un énorme repository, que la branche ou le tag qu'on cherche à utiliser existe bien sur le repository.

Une commande git permet justement de faire cela assez facilement.
<!--more-->

Et cette commande, c'est `ls-remote`.

Elle porte d'ailleurs bien son nom, puisqu'elle permet de lister (ls) les références nommées (branches, tags, etc) d'un repository distant (remote).

Prenons un repository ayant beaucoup de branches et de tags en guise d'exemple : [https://github.com/facebook/react](https://github.com/facebook/react)

En utilisant la commande `ls-remote`, je vais obtenir ceci :

{% highlight text %}
$ git ls-remote https://github.com/facebook/react
...
88abf18814e6ace664fa3a76011d5d99a93ca8d4  refs/pull/8415/merge
0865273b3e01a3a1d7e8b0da3e82265bde004991  refs/pull/8416/head
5f4f36d21e08602b75b48a1b851e67a1d25e4790  refs/pull/8417/head
968d9bdd03bbd879fac80b1f2a33bdc271fd764f  refs/pull/8417/merge
925f1f2ffc597c83cb3c6befb22c7ef49408a1a9  refs/pull/8420/head
f7ae6e831c63716c57d792a30dfe94994862af67  refs/pull/8421/head
39512ec1b1c61929cce80f343d2892987eb2a653  refs/pull/8422/head
db783e10997c723c30bdbd9300dcada8db5e3146  refs/pull/8424/head
e0d1b8c518219cc2404a745e58ac600ffef3538a  refs/pull/8425/head
1861824615d4b04445386838a4273e99be918991  refs/pull/8426/head
d5d550414dd5b5b1b5aecd9025d97b7ed29ed030  refs/pull/8426/merge
f0571cdca990c8cac6b5bcf806b5a2c6714a3616  refs/pull/8427/head
9af03f7f5cdd84aace3197652e4ea147a244c5fc  refs/pull/8427/merge
c57bfe9fbe15a05e5b1064883d3b6610daadbf38  refs/pull/8428/head
eed7f8370c70a0cb8971837dcdab8ed2e79c614d  refs/pull/8428/merge
e24ec4a1b492a2d7543454afd7228139009924e4  refs/pull/8432/head
...
{% endhighlight %}

Super mais ça fait un poil beaucoup vu que visiblement ça a également sorti les pull-requests.

Pour filtrer sur les branches, il suffit d'utiliser `--heads`, et pour les tags c'est `--tags`.

{% highlight text %}
$ git ls-remote --heads https://github.com/facebook/react
d5b409002b425d2242da0d9bacfad68278856400  refs/heads/0.10-stable
f6fcf385c9fd0df2ad666bc7765c127bd5e1528d  refs/heads/0.11-stable
0fc0871c89963ecaeb1476b82156c2399e765e8e  refs/heads/0.12-stable
76c87da026bdab63b5b109e3c073a1db74896ed6  refs/heads/0.13-stable
894d20744cba99383ffd847dbd5b6e0800355a5c  refs/heads/0.14-stable
f7c56d77dfbb1f7b8217f04e00ff71a26537c9a8  refs/heads/0.3-stable
283f4a8b04db6bbd177e94404476cabaf569d3a7  refs/heads/0.4-stable
7b809ffbc4cdbb9858ba36ec341b4767c7a8543e  refs/heads/0.5-stable
56517a54234379239113a663d2805f75a38b11ab  refs/heads/0.8-stable
b1f66edf4e90527468987eafbaf50a410febdcc9  refs/heads/0.9-stable
fb64f352923383197b2d48385488cdaed09b117e  refs/heads/15-dev
...

$ git ls-remote --tags https://github.com/facebook/react.git
dedf0c20da67872b5dff21a25cb9075e6019c12e  refs/tags/v0.10.0
7f24943e5af5ee4b14ec002d45df315af94adb75  refs/tags/v0.10.0-rc1
95d82cacd6e9cc6a2fe6366d79510cc9133886cb  refs/tags/v0.11.0
0f9cec2e78c09e81dc3dac764788589a07903411  refs/tags/v0.11.0-rc1
7e946bcb9c5fba7fbeba7301c212d27ed653a06b  refs/tags/v0.11.1
3845e214a481381694de66ef6ce16ff7f5f139d5  refs/tags/v0.11.2
3e925822a6c3b7a2447a537563e66793383f3cc9  refs/tags/v0.12.0
2b4e35870b7a0c4d681bc3c86641790dd828f0a0  refs/tags/v0.12.0-rc1
a067fc0feef9e8b02cc3e9331360c57082afea19  refs/tags/v0.12.1
1e1f02a83ab2972de72acab90b7cf4769adba9e1  refs/tags/v0.12.2
edb8f7f4af980d2859582ed243b7f9dd6701a48e  refs/tags/v0.13.0
06126ad3f4d063e89b3168abce79c9cd9961831c  refs/tags/v0.13.0-rc1
...
{% endhighlight %}

Du coup, si vous avez un script ou un job d'intégration continue qui prend en paramètre un tag ou une branche à déployer, il suffit de faire un grep sur la sortie de ls-remote pour vérifier que le paramètre d'entrée existe bien :

{% highlight text %}
$ version=v15.0.0
$ git ls-remote --heads --tags https://github.com/facebook/react.git | grep -E "refs/(heads|tags)/${version}$"
94a1ae06852d388f081e7d3ba87d3065dda7ee6a  refs/tags/v15.0.0
{% endhighlight %}

Dans un script bash, vous pouvez ensuite vérifier le résultat de la dernière commande pour savoir si la version fournie a bien été trouvée ou non, par exemple pour déclencher le clonage du repository ou bien pour afficher un message d'erreur à l'utilisateur.

Voici à quoi ressemblerait un tel script :

{% highlight shell %}
#!/bin/sh

repo="https://github.com/facebook/react.git"

if [[ "$#" -ne "1" ]]; then
  echo "Usage: $0 [version]"
fi

version=$1
git ls-remote --heads --tags $repo | grep -E "refs/(heads|tags)/${version}$" >/dev/null

if [[ "$?" -eq "0" ]]; then
  git clone $repo --branch $version --depth 1 --single-branch
else
  echo "Unknown version $version"
fi
{% endhighlight %}

Et maintenant si j'essaye de l'utiliser avec un tag qui n'existe pas puis avec un tag qui existe :

{% highlight text %}
$ ./clone_repo.sh bliblablou
Unknown version bliblablou

$ ./clone_repo.sh v15.0.0
Cloning into 'react'...
remote: Counting objects: 1118, done.
remote: Compressing objects: 100% (1078/1078), done.
remote: Total 1118 (delta 58), reused 495 (delta 10), pack-reused 0
Receiving objects: 100% (1118/1118), 37.91 MiB | 2.66 MiB/s, done.
Resolving deltas: 100% (58/58), done.
Checking connectivity... done.
Note: checking out 'd1c08f11d5e1ad03eb92a58b599562a010a68734'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

$ cd react
$ git log -1 --decorate
commit d1c08f11d5e1ad03eb92a58b599562a010a68734 (grafted, HEAD, tag: v15.0.0)
Author: Paul O’Shannessy <paul@oshannessy.com>
Date:   Thu Apr 7 12:07:50 2016 -0700

    v15.0.0
{% endhighlight %}

Bon, reconnaissons toutefois que mon script ne sert pas à grand chose vu que git, depuis la version 1.7.0, vérifie l'existence de la branche avant de faire un git clone :

{% highlight text %}
$ git clone https://github.com/facebook/react.git --branch bliblablou
Cloning into 'react'...
fatal: Remote branch bliblablou not found in upstream origin
{% endhighlight %}