---
layout: post
title: The Fork Not Taken
date: 2020-02-11
categories: [reproducibility, cookiecutter, easydata]
excerpt: "The cookiecutter project is facing a fork, and their problem is one found throughout the Open Source world. It's a clash between economies: gift culture, and capitalism."
---

**TL;DR**: The cookiecutter project is facing a fork, and their problem is one found throughout the Open Source world. It's a clash between economies: gift culture, and capitalism. The solution is to pick one. It doesn't matter which. Just know the repercussions of what you pick.

## A Fork in the Road

We make heavy use of [cookiecutter] in the [EasyData framework]. It's
a great way to get a new project up and running quickly, and
more importantly, reproducibly.

But the cookiecutter project has a problem. Some sizeable (and I'd
argue: much-needed) changes [have been proposed][cookiecutter2] to the framework that
allow for branching questions. This would be a huge boon to EasyData,
a huge boon for our upstream [cookiecutter-data-science] project, and,
I expect, a huge boon for a number of other projects, too, as it would
allow for much more succint and relevant questions when configuring a
new project.

[cookiecutter]: https://github.com/cookiecutter/
[cookiecutter2]: https://github.com/cookiecutter/cookiecutter/pull/848

Though I've long been keeping my own list of desirable cookiecutter
features, I hadn't yet headed upstream to see whether others were
proposing similar features. I haven't had the time, and I've been
content with my workarounds to date. Then suddenly, one of our
upstream inspirations, ([cookiecutter-data-science]) made a bit
announcement. Going forward, they would be [monkey-patching new
features into cookiecutter][ccds] and shipping their own (forked) version of
cookiecutter called `ccds`.

[ccds]: https://github.com/drivendata/cookiecutter-data-science/pull/162
[cookiecutter-data-science]: https://github.com/drivendata/cookiecutter-data-science/pull/162
[easydata framework]: https://github.com/hackalog/cookiecutter-easydatadrivendata/cookiecutter-data-science/pull/162

This was a big shift. Forking is rarely a decision taken lightly, so I
headed upstream to see why they were proposing a fork instead of
simply merging the changes upstream.

## The Problem with Progress

In short, a massive (and incredibly useful) change was being proposed
for cookiecutter that changed the underlying `cookiecutter.json` to
permit skipping and branching in the configuration rules. Unfortunately, despite
a great spec, [a working implementation][cc-fork], and 100% test coverage,
the change was put on hold.

[cc-fork]: https://github.com/cookiecutter/cookiecutter/pull/1008

Why?

Because the project's volunteer maintainers predicted that such a
massive change would produce an enormous support burden on the
project's maintainers. Without [sponsorship][raphael-patreon], they were not willing to
take it on.

[raphael-patreon]: https://www.patreon.com/hackebrot

And they're almost certainly right. This is a problem seen throughout
the open source world. How many users are there for your handful of
open source projects?  How many of them submit patches? Now, how many
maintainers are there? I'm guessing it's a pretty ugly ratio. There's
a massive imbalance here, and it leads to a well-documented
[mismatch of expectations][brett-talk].

[brett-talk]: https://pyvideo.org/pycon-ca-2017/setting-expectations-for-open-source-participation.html

The cookiecutter maintainers are being quite honest about the situation they are in,
and should be commended for that. But where does that leave us as users of cookiecutter?

## Darwin, Adam Smith, and Feelings

It could be argued that there is a mechanism within the open source
world for solving this problem: **the fork**. If a given problem
matters enough to someone, and the maintainers can't or won't
implement it, anyone is free to simply fork the project and take over
maintainership themselves. If others agree with the arguments behind
the fork, they will jump to the new project and a new, thriving
community will appear to proceed in the new direction.  It's very
Darwinian (survival of the fittest!). It's very Adam Smith (supply and
demand!).  It also neglects one very import piece of the puzzle:
**people**, and their feelings.

Code is written by people. Forking a project because you don't agree
with the maintainers seems perfectly logical, but it can have real,
emotional repercussions for those involved. Even though it's
completely permissible--even _in the spirit of_--virtually all Open
Source Licenses (including the [BSD-3-clause license used by
cookiecutter][cookiecutter-license]), it can still feel *bad* for the
people involved in the original project. It's like a conversation that
goes like this:

[cookiecutter-license]: https://github.com/cookiecutter/cookiecutter/blob/d9d25d3c735e4b00083c96b279ae98bfc8d3763f/LICENSE

> "Will you support my future work on this project?"
> "No."

> "Seriously, I'd love to help. I just can't keep up with the support burden
of massive, breaking features."
> "Then leave it to someone who can."

> "What about all my over ocntribution sthe years?"
> "It's open source. Deal with it."

These bad feelings have tangible repercussions. If you end up
alienating those who know the project the best, you're greatly
diminishing the pool of people who could conceivably contribute to the
project, either via the fork, or the original project, which seems
like a needless loss.

How do we mitigate the potential for hurt feelings if we decide we
have no choice but to fork?


### Capitalism vs. the Gift Culture

I think what we have here is a clash of economic models.

The open source world was built on the model of a _gift culture_. You
gain status in the community by giving more (code, in this case). You
lose status when you take. Your past contributions determine your
future worth.  It's an abundance culture, and it works well when
everybody agrees on the currency of giving. It also relies on everyone
buying into the currency

Compare with capitalism. You gain status in a capitalist world by
collecting tokens ("money"), exchangable for future benefits.  He or
she with the most tokens has the most worth. It's motivated self
interest. It's a culture of taking. It's a culture of scarcity, but it
works incredibly well when even when everyone focuses only on their
own self-interest. Just like a gift culture, however, it breaks down when
there's disagreement on the value of the currency. Just observe what happens
when deflation occurs.

When a capitalist meets an open source project, it sees **value without cost**.
When an open source community meets a capitalist, it sees a **cost without value**.

These two models don't fit together well. Maybe not *at all*. When you
try to create hybrids ("volunteer to give me money for this thing you
could take for free"), things just don't seem to work out
well. Attempts to bridge the two worlds seems always to end in
failure.

Maybe it's time to admit a truth: the models aren't compatible. Maybe
that's okay. Maybe we can find a path foward even though this is the case.

## The Robert Frost Moment

What's the right road to take? How do we fix the ever rising tide of
support requests and burned out open source maintainers, without
killing off the open source movement as we know it. How do we merge
the models of capitalism and the gift culture in a way that feelings aren't
getting crushed and enemies made?

I don't think we can merge them. In fact, I think that individually,
we have to pick one and be okay with the repercussions.

* If you pick _gift culture_, then you have to be ok with the idea
  that "forks happen, no hard feelings."

* If you pick _capitalism_,
  then you have to be OK paying for support, and the idea that nobody,
  no matter how big the project, owes you anything if you're not
  paying for it.

If you want to participate in open source, treat people
well. Participate. Submit patches, not just issues. Fork if you have to,
but be prepared to fold that fork back upstream if the opportunity
arises. Give value to the currency of giving.

If you want to be a capitalist, **pay for what you need**. Nobody
*owes* you anything. Buy support contracts. Exchange money for goods
and services.

[frost-poem]: https://www.theparisreview.org/blog/2015/09/11/the-most-misread-poem-in-america/

In short, when you come to the fork in the road. Pick one. Just like
the poem, [it doesn't really matter which][frost-poem]. What matters is that you buy
into whatever model fits you the best. All the way in.

> Two roads diverged in a yellow wood, \\
> And sorry I could not travel both

Wise, Robert. Very wise.
