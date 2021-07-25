---
layout: post
title:  "How to do atomic commits"
lang: en
ref: commits-atomiques-la-bonne-approche
date: 2017-04-26 12:00:00 +0200
categories: en methodology
disqus: true
excerpt_separator: <!--more-->
filename: 2017-04-26-how-to-do-atomic-commits.markdown
hidden: true
translator: kitty-giraudel
---

*One more time, my sibling [Kitty](https://twitter.com/KittyGiraudel), helped me by translating my post "[Commits atomiques - la bonne approche]({% post_url 2017-04-26-commits-atomiques-la-bonne-approche %})". Many thanks to them for their translation and review!*

-----

One of the most difficult parts of our job is to divide our work into small units. Often overlooked, this challenge happens as much on an architectural level (it’s much easier to build a big-ass monolith than small joint projects) as on the code we ship every day through our commits.

In other words: **it is hard to author small commits**.
<!--more-->

It is hard to author small commits because in a way it is easy not to commit. While this may sound stupid, it is usually easier, at the end of a long coding session, to ship a big huge commit rather than doing a lot of small ones. It’s even more tempting to do so when the benefits of small commits are not necessarily obvious.

It’s a bit like automated tests: we tend to find them useless when writing them. After all, proof-reading the code and going through a couple of reviews should be enough to make sure not to introduce a bug. However, the day there is a live bug because of a small thing that should have been Just Fine™, a proper test coverage would have made sure to catch that nasty problem before it hits the production server.

Small commits are somehow similar: they are a pain in the ass to do, they seem useless, but the one time shit gets real on the live server, we thank ourselves having made the efforts.

You might wonder what is the connection between the average commit size and a bug in production: the harder it is to find the source of bug and the way to patch it, the more expensive it will be to fix (be it in time or money). **Therefore reducing the cost of a bug comes down to spotting it early** (admittedly, before hitting live is the ideal scenario). This is where tests intervene. But it also comes down to reducing the amount of time needed by an engineer to fix the problem.

## Hunting bugs down

Authoring small commits help finding the origin of a bug, the context in which it has been introduced and possibly the reason why it has been introduced. Therefore, it makes life easier for the person in charge of fixing it.

This is getting even faster with Git when performing dichotomous search in a large range of commits with `git bisect`. For instance, on a huge project with over 31,000 commits (spread across 6 years), a good ol’ `git bisect` could make it possible to find the origin of a bug in about 15 operations.

If you’d like to learn more about tracking down bugs with `git bisect`, be sure to read my articles about it [in French](http://adopteungit.fr/commande/bisect/2016/09/04/la-chasse-aux-bugs-avec-git-bisect.html) and [in English](https://kittygiraudel.com/2014/03/24/git-tips-and-tricks-part-3/). :)

## Code reviews

Other and more obvious benefit of small commits: they make reviewing code much easier. Without even talking about open source software here, performing code reviews before merging into the main repository present number of advantages.

* It reduces the amount of minor mistakes
* It enforces consistency across a code base
* It promotes knowledge sharing across a team
* It sparks discussions pushing the project further

Anyway, that’s awesome.

On the other hand, it is getting tedious to do consistently when code gets long, complex and dispatched.

When building a feature from start to finish —going through all the layers of an MVC application— a lot of files could be impacted: the view (usually split across templates and styles), the controller, the router, the application layer, the DAO… Plus unit tests for all or most of these chunks and possible integration tests… The pull-request for this feature is going to be huge. And that’s without counting for a bit of refactoring here and there, checkstyle, improvement of test coverage, clarification of documentation, and so on.

The risk is to end up with a huge beast that nobody will want to review. Best case scenario, the reviewers will be careful on the first few lines/files then quickly start botching. Worst case scenario, they will give up hoping someone with more time on their hands will tackle it (hint: nobody does).

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

I write, I write, but we still haven’t really tackled the technical content for this article. But I felt like I had to introduce the matter with a bit of background to explain the rational behind this whole “small commits” approach.

Alright, let’s roll up our sleeves and dig.

## So, what’s up?

There is a ton of ways to organise commits with Git. The first one is not even a technical trick: stay focus on the goal and don’t perform any non-critical change anywhere else.

Let’s be real here, this is quite a terrible technique because it goes actively against improving the code. So unless you like covering your desk with post-its like “remove unnecessary white space line 17 of file Toto.class.php”, I suggest we find something else.

A good method should let the developer work the way they like only to finally reorganise their changes when committing.

### A swarm of commits

The best piece of advice I can give you is to commit very often.

Reindenting some broken tabs? Commit it on its own. New tests for a function? Commit them on their own. Some necessary refactoring for your upcoming feature? Commit that on its own.

The thing that matters is that the critical commits for a feature, the ones which will need to be actively reviewed, are free from all this visual distraction.

The tricky part with this approach is to resist the temptation of doing `git add -A` to ship all modified files in the commit. This is where knowing how to use `git add -p` (or `--patch`) helps, so we can divide multiple updates within a single file to commit them separately.

Let’s say we have the following PHP file:

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

We’re just about to add some code to this file when we realise that the `$listId` parameter is not typed in the PHPDoc from `retrieveList()`. So we add the type. It’s easy, it doesn’t cost much and it will help using this method in the IDE. Cool.

As thorough Atomic Commiters (here is your new Twitter bio, you’re welcome), we’ll want to commit this small addition on its own, without including the new (and possibly non-finished) code. When using `git add -p path/to/MyFile.php`, Git will divide changes into blocks, making it possible for us to choose for each block whether we want to add it to the commit.

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

Pressing `?` displays all possible options:

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

Most of the time, we press `y` to add the block or `n` not to. Sometimes `s` to ask git to be even more granular (until it can’t do anymore) and `e` to manually choose which lines should be added. 

In this scenario, we press `y` to add the PHPDoc comment to the commit. Then git will present the next modification in the file:

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

Now we say `n` since we only want to commit the documentation fix, not the rest. Once we’ve been going through all the modifications (probably setting `n` for all the others), it’s only a matter of finalising the commit and pushing to the remote repository.

The benefit of this approach is that these little changes will likely be quickly reviewed and merged while working on the core feature.

### Bundling the fancy edits

A variation of the aforementioned technique is to bundle these small changes in relatively concise commits rather than having them in tons of teeny tiny ones. Performing atomic commits does not mean each PHPDoc change has to live in its own commit; they can be bundled together as long as they are connected by a unique theme.

The possible issue with this comes up in code review. Even with 10 properly cut commits, the diff is likely to be quite important. This is where it might be helpful to tell reviewers to check commits individually rather than as a whole.

![Guide reviewers towards important commits](/assets/images/posts/commits-atomiques-la-bonne-approche/guider-utilisateurs.png) 

Other potential downside with committing everything at the end of a long working session: it can be time consuming. We might start a commit dedicated to a small thing (such as moving a repeated string to a constant), making us browse through the entire diff to make sure all changes related to this are added to the commit. Once done, we’ll walk through the whole diff to gather all related changes into another commit. And so on, and so on, making us go through the diff quite a few times.

### Rockin’ the interactive rebase

Another approach that has proven to be successful is abusing interactive rebase. This git feature makes it possible to reorder commits, edit them and even merge them.

It means we can perform a lot of small commits that are not directly related to the feature we are currently working on. Once ready to submit the final pull-request for review, we can reorder and squash small commits together to reorganise the git history. For instance, we could create a first pull-request containing all little edits, and then another for the actual feature.

Let’s illustrate with an example.

We are currently working on story *GRE-1234*, whose goal is to allow users to edit their messages in a forum thread. While working on it, we fix a lot of small things here and there, and commit them on the fly. Once done with the actual feature, we end up with these 15 commits:

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

Some of these commits are very connected and could be grouped. Our feature is really just done through 2 commits, prefixed with the story number (`[GRE-1234]`). And unless we have a good reason to split this in half, they probably should be merged into a single commit.

Alright, let’s try to tidy this mess.

The command `git rebase -i HEAD~15` allows us to update the history of these last 15 commits. It will open a text editor displaying the list of commits, each of them prefixed with `pick`. Beware! Unlike `git log`, commits are actually displayed in chronological order: oldest first, newest last.

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

We usually stick to options `pick` and `squash`, as well as the reordering feature. Let’s start with that:

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

We grouped together similar commits and moved our feature to the very end. Now we can decide to merge some commits together. To do so, we replace the `pick` keyword from the relevant lines with `squash` (or `s`).

For instance, the first 4 commits are somehow similar: they add missing descriptions to behat scenarios. We can then keep only the first one and merge the 3 next into the first:

{% highlight text %}
pick ab00199 Missing description in Behat scenario
squash 8593186 Missing description in Behat scenario
squash 1a96fc2 Behat description of scenario
squash ae03f4b Behat description
{% endhighlight %}

Let’s repeat this process for all the other “groups” of commits, until we get something like this:

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

If we save and close our editor, it will reopen asking for the new commit message for the first pack of 4 commits to be merged:

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

We can empty the whole thing and replace it with a single message *“Missing description in Behat scenario”*. When closing the editor, it will open again to ask for the message for the next commit group, and so on until the last group is done after which we’re back on the terminal.

If we wanted to skip this process and keep the message of the first commit of each squashed group, we could have used `fixup` (or `f`) instead of `squash` during the interactive rebase. This is the only difference between them two: `squash` merges commits while giving the ability to edit comments while `fixup` uses the message of the receiving commit.

That’s it! If we have a look at our commit history, we now have 7 commits in place of 15:

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

At this point, we can safely create a pull-request and ask our co-workers to only actively review the commit containing the actual feature without worrying too much about previous unrelated changes.

We could also create a branch on the second to last commit (`git checkout -b small-fixes HEAD~1`) to send all these rather boring edits in a pull-request to free the feature pull-request from visual noise.

Note that interactive rebase works well when commits are not particularely connected to each others. Always be careful with the history order. For instance, the second commit from our feature (`7a9d433 Don't allow answer edition on closed thread`) is likely to be highly dependant on the first as it fixes an edge case omitted from the first one. Swapping the order in the rebase might be quite a bad idea.

As a safety measure, feel free to create a temporary branch (`git branch tmp-save` for instance) before jumping into a rebase, so you can abort and come back to a sane spot in case it turns badly.

### Autosquash

Another nice thing with interactive rebase is the autosquash feature. Rather than reorganising the code and merging commits at the end, we can do it on the fly during development. When creating a commit, we can tell in which commit it will eventually have to be merged.

Let’s go back to our 15 commits from earlier:

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

We start with a first commit (`ab00199 Missing description in Behat scenario`), then a second (`7bc77ef Dead code`). When authoring the third commit (`8593186 Missing description in Behat scenario`), we could have thought *“hey, it’s the same as the first commit, I should merge this change with it”*.

To do so, we only have to use the `--fixup` parameter from the `git commit` command, precising in which commit we want to merge these changes. We can do that by specifying the commit SHA1: `git commit --fixup ab00199`. Or in a relative way: `git commit --fixup HEAD~1` (second to last commit). Or even with a search for the most recent commit matching a keyword: `git commit --fixup :/Behat`. How cool is that?

Once done, the 3 last commits should look like this:

{% highlight text %}
$ git log -3 --oneline
ca9260b fixup! Missing description in Behat scenario
7bc77ef Dead code
ab00199 Missing description in Behat scenario
{% endhighlight %}

Now let’s say we did so on our 15 commits, we should have:

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

We only have to perform an interactive rebase with the `--autosquash` option so all commits tagged `fixup!` gets merged in their relevant commit. git will open the editor with the following content:

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

We can save and close the editor so the rebase gets completed.

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

Note how the order is different from the one we had when reorganising the history without the `--autosquash` option. If it causes any issue, it’s still possible to perform another interactive rebase to reorder commits (`git rebase -i HEAD~7`).

Rather than commiting with `git commit --fixup [SHA1]`, we can also use `git commit --squash [SHA1]` which will work the same way. The only different is the ability to edit messages at the end for commits created with `--squash`.

## Wrapping things up

At the beginning of this huge wall of text, I told you about `git bisect` to perform dichotomic search. Let’s remember that for this command to work properly, it needs a rather clean git history. In theory, you should be able to roll back to any commit and be able to use your application (including running tests).

Therefore, there should be no *“Fixing broken test”*, *“Fixing error seen in CI”* or *“Fixing layout”* commits: if a commit breaks something (be it a test or a part of the app), the fix for this problem should be merged in the same commit, and not be a commit on its own. Otherwise it will be possible to end up in a broken state while performing a `git bisect`. Not cool.

Alright, I hope all this will make you reconsider next time you want to use `git add -A`. Remember to do small commits to ease readability, code review and debugging if there is ever the need!
