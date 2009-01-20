---
title: My Git Workflow
layout: post
---

Lately I've been acting as the unofficial [Git][] consultant for
[Skribit][], usually in response to [@Stammy][] saying something on
Twitter along the lines of "Hey, git just did something totally
nonintuitive and now I can't figure out what to do next." I've been
using Git long enough to know the answer, usually, or at least to be
able to figure it out, plus I still remember how people used to SVN think,
which helps.


<div style="text-align:center;margin:0 auto;width:440px">
<p><img src="http://twictur.es/i/1109499437.gif" /></p>
<p>image provided by <a href="http://twictur.es">twictur.es</a> | original tweet <a href="http://twitter.com/Stammy/statuses/1109499437">here</a></p>
</div>

I think I've mentioned before how much I love Git. I've been a version
control geek for a **really** long time: I remember getting way too
excited about a feature that was like 'git cherry-pick' in some
proprietary VCS we evaluated in 2000 or 2001. I still say that Darcs
is the one I wish we had settled on, but I've grown to love Git over
the past year or so, especially with [GitHub][] making some of the
rough spots so much easier.

The thing that's hard about Git for most people is the same thing
that's hard about all [DVCS][] tools: if you're used to centralized
systems, they work weird. GitHub doesn't actually help with this: it's
really easy to use Git+GitHub like SVN, and not take full advantage of
all the cool stuff you get in a DVCS. It's also really easy to never
grok DVCS, so when something unusual happens, you don't know how to
fix it.

There are lots of really good articles to help you get Git, and I'm
not going to try to duplicate them, nor am I going to make you a
comprehensive list. Ask in the comments if you can't find what you
need, and feel free to share good resources in the comments as
well. One I'll throw out there now is [Git for Computer Scientists][gfcs]
... if you want to really know what Git is doing, this is the way to
go.

What I *will* do is give you a snapshot of my personal Git
workflow(s). I have several, depending on the kind of project.

Small team: one or two person project
----------
A lot of my projects involve either just me, or me and Don, or me and
someone else. For these projects I don't usually take much advantage
of the power of Git. It goes something like this.

* `git pull`
* Hack some stuff
* `git commit -a -m "hacked some stuff"`
* `git push`

I rarely use [topic branches][] when I'm in this mode, and I don't pay
too much attention to keeping a clean commit history. In short, I use it
alot like I would use SVN.

Every now and then I'll get distracted by a quick bug fix when I have a
bunch of code I don't want to commit yet. That's where stashing comes in.

* `git stash` (all my changes are saved away, and I have a clean tree)
* Fix the bug
* `git commit -a -m "bugfix"`
* `git stash pop` (changes are back)
* Continue where I left off

Larger team
----------

Lately, I've been doing some client work to pay the bills, which means
I'm working with a group of other developers with a lot of tickets being
handled all at once. To minimize my pain (and to keep nice and clean commit
logs) I use my own version of the [SSP: Simplified Software Process][ssp].

* `git co master`
* `git pull`
* Find a ticket to work on. Let's say it's "Add flanges to the WangleController"
* `git co -b flanges_in_wangle`
* Write a test, write some code, etc.
* Commit only what I know I want to commit. I use `git add -p` to be selective.
* `git commit --amend` (I write really nice commit messages when there are
  other people looking at the code, and I use amend to bundle all my changes
  into one commit)
* Run the tests and make sure they pass
* `git co master`
* `git pull`
* `git co flanges_in_wangle`
* `git rebase master`
* At this point, either it goes well or I have merge conflicts. If I have
  conflicts, I fix them, and keep going. It's better to have conflicts on
  a topic branch than in master.
* Run the tests again. If there are any code changes, commit, then repeat from
  checking out master again. The idea is to make sure that when you finally
  merge flanges_in_wangle into master, it's going to be a clean fast-forward
  merge with no conflicts.
* Once I'm sure flanges_in_wangle is ready, `git co master`
* `git merge flanges_in_wangle`
* `git push`

Enlightenment
----------

On my most recent large project that I started from scratch, I use something
like the above, with one major exception: there is no *'master'*.

Using *'origin'* as the name of your remote and *'master'* as the name of your
main branch is just the default settings git gives you. Neither name is magic.
So, I renamed *'origin'* to *'github'* in `.git/config` to be explicit about
where the code is going. Then, I created a new branch *'development'* to
reflect the code that is under the mainline of development.

Then I deleted *'master'*.

Yeah, you heard me. Who needs it?

Once I was ready for stuff to go to staging, I made a *'staging'* branch, and
likewise for *'production'*. Topic branches merge into *'development'*, which is
merged periodically into *'staging'*, which gets promoted to *'production'*
when it's ready for deployment. There's never any doubt about what code is on
each environment; just look in the git repo.

The downside to this is that when you first clone the repo, it's broken, because
git tries to checkout *'master'* and it doesn't exist. Not a big deal, just
setup the right branches and all is well.

And you?
----------

So what's your git workflow? Share it in the comments, or blog about it
[and tell us about it](http://skribit.com/suggestions/talk-about-your-daily-git-workflow "Skribit: Talk about your daily git workflow").

**PS**: This blog post was written in direct response to a [Skribit][]
suggestion. I don't promise I'll blog about anything people ask me to,
but it's possible I will. If you'd like to get that kind of feedback
on your blog, check Skribit out. They came out of the first Atlanta
Startup Weekend, and are doing some very cool things.


[Git]: http://git-scm.com
[github]: http://github.com
[Skribit]: http://skribit.com
[@Stammy]: http://twitter.com/Stammy
[DVCS]: http://en.wikipedia.org/wiki/Distributed_revision_control
[gfcs]: http://eagain.net/articles/git-for-computer-scientists/
[topic branches]: http://www.kernel.org/pub/software/scm/git/docs/howto/separating-topic-branches.txt
[ssp]: http://reinh.com/blog/2008/08/27/hack-and-and-ship.html
