---
layout: post
title: The Fork Not Taken
---

**TL;DR**: The cookiecutter project is facing a fork. Their problems boil down to a clash between economies: gift culture, and capitalism.


### The Problem with Progress

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
proposing similar features.  Then our upstream project
([cookiecutter-data-science]) made a bit announcement. Going forward,
they would be monkey-patching new features into cookiecutter and
shipping their own (forked) version of cookiecutter called [ccds].

[ccds]: https://github.com/drivendata/cookiecutter-data-science/pull/162
[cookiecutter-data-science]: https://github.com/drivendata/cookiecutter-data-science/pull/162
[easydata framework]: https://github.com/hackalog/cookiecutter-easydatadrivendata/cookiecutter-data-science/pull/162

This was a big shift, so I headed upstream to see why they were
proposing a fork instead of simply merging the changes upstream.

In short, a massive (and incredibly useful) change was being proposed
for cookiecutter that changed the underlying `cookiecutter.json` to
permit skipping and branching in the configuration rules. Unfortunately, despite
a great spec, [a working implementation][cc-fork], and 100% test coverage,
the change was put on hold.

[cc-fork]: https://github.com/cookiecutter/cookiecutter/pull/1008

Why?

Because the project's volunteer maintainers predicted that such a
massive change would produce an enormous support burden on the
project's maintainers. Without sponsorship, they were not willing to
take it on.

And they're almost certainly right. So where does that leave us as users?

### Darwin and the Fork

It could be argued that there is a mechanism within the open source
world for solving this problem: **the fork**. If a given problem matters
enough to someone, they'll simply fork the project and take over
maintainership themselves. If others agree, they will jump to the new
project and a new, thriving community will appear to replace the old,
stuck one. It's a nice theory. It's very Darwinian (or maybe Keynesian?).
It also neglects one very import piece of the puzzle:
**people**, and their feelings.

People matter. Feelings happen. Forking a project because you don't
agree with the maintainers has repercussions. Even though it's
completely permissible, even _in the spirit of_ virtually all Open Source
Licenses (including the BSD-3 clause license used by cookiecutter), it
can still feel *bad* for the people involved in the original
project. It's like a conversation that goes like this:

> "Will you support my future work on this project?"
> "No."

> "Seriously, I'd love to help. I just can't keep up with the support burden
of massive, breaking features."
> "Then leave it to someone who can."

> "What about all my contributions over the years?"
> "It's open source. Deal with it."

Totally legit in Open Source. Also, totally dickish. Feelings get
hurt, and that matters.


### Capitalism vs the Gift Culture

I think what we have here is a clash of cultures.

The open source world was built on the model of a _gift culture_. You
gain status in the community by giving more (code, in this case). Your
past contributions determine your future worth.  It's an abundance
culture, and it works well when everybody agrees on the currency of
giving.

Compare with capitalism. You gain status in a capitalist world by
collecting tokens ("money").  He or she with the most tokens
wins. It's motivated self interest. It's a culture of taking. It's
a culture of scarcity, and it works well when everyone focuses on their
own self-interest.

When a capitalist meets an open source project, it sees **value without cost**.
When an open source community meets a capitalist, it sees a **cost without value**.

These two worlds don't fit together well. Maybe not *at all*.; When
you try to mix the models ("give me money for this thing you could
take for free"), things just don't seem to work out well. Any attempt
to bridge the two worlds seems to end in failure. The models aren't
compatible.

## The Fork Not Taken

So what's the right road to take? How do we fix the ever rising tide
of support requests, and burned out open source maintainers? How do we
merge the cultures of capitalism and the gift culture?

I'm not sure we do. In fact, I think we have to pick.

* If you pick _gift culture_, then you have to be ok with the idea
  that "forks happen, no hard feelings."
* If you pick _capitalism_, then you have to be ok paying for support.

And if you pick neither? Then you have to be prepared to navigate some pretty
unwinnable scenarios. How do we decide when to fork, and how not to burn bridges when doing it?
I don't know. I'd start with this:

Don't be dickish. Treat people well. Fork if you have to, but be
prepared to fold that fork back upstream if the opportunity
arises. Pay for what you take. Nobody *owes* you anything.
