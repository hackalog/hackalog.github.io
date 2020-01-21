---
layout: post
title: Great Myths of the Python Environment Managers
date: 2020-01-20
categories: [Python, reproducibility]
excerpt: In open source land, there's a lot of fighting that goes on about just which virtual environment manager is the right one to use. While not myths exactly, I'm going to bust a few of these opinions, or at least try to convince you that while it may be a perfectly good solution for your use case, that doesn't mean it solves my (or my customers') problems.
---

I spend a lot of time doing Python development, and in the process, I
make pretty heavy use of virtual environments. If you've spent any
time in the space, you know that the Python virtual environment space
is a bit of a mess. Depending what part of the problem space you're
trying to use, you might prefer:

* pip+virtualenv
* [conda]
* [Pipenv]
* [Poetry]
* [virtualenvwrapper]
* [Hatch]
* [pip-tools]
* [pyenv]

{% include pullquote.html quote="Like the mantra 'strong opinions, loosely held', except without the second part." %}
As with everything in open source land, there's a lot of fighting that
goes on about just which virtual environment manager is the right one
to use.

I hear a lot of opinons in this space, stated as facts. Like the mantra
[Strong opinions, loosely held][SOLH], except without the second part.

While not myths exactly, I'm going to bust a few of these opinions, or
at least try to convince you that while it may be a perfectly good
solution for your use case, that doesn't mean it solves **my**
(or my customers') problems.


### Pipenv the officially recommended Python packaging tool from Python.org

Nope. [not even close][warrick-pipenv]. Pipenv is aimed at _managing
application dependencies_ only. Not library development. Not
packaging. Also, it **doesn't handle multiple versions of Python**. As
someone who does a lot of reproducibility work, being able to pick and
choose which version of Python I use is absolutely essential.

I will admit, what pipenv does do nicely is the `pipenv run` command,
which makes scripting things in a Makefile a ton easier. It also
separates dependencies into my desired dependencies (`Pipfile`) and
the complete set of resolved dependencies (`Pipfile.lock`). Then
again, just about every tool than `pip` does this.

Also, I'd be remiss not to point out that the pipenv project appears
to be in quite a [bit of flux][hn-if-its-dead].  To borrow a cheesy
movie line, it's not dead, but it's pretty badly hurt.


### You don't need the ability to split packages between production and development

This claim, e.g. is made by [Chris Warrick][warrick-pipenv], is that
installing development dependencies in production is fine because it:
> ...should only be a waste of HDD space and nothing more in a well-architected system.

Not even close. I write Python all over the place, including embedded
systems, mobile devices, and other bizarre architectures. I also use
things like jupyter to develop my code. I use numpy to generate test
data. Some of these packages are pretty insane to compile and ship on
non-mainline architectures, so in that case, I really *need* to split
my development and deployment environments. It's not a matter of disk
space. It's a vast investment of developer time getting development
dependencies working someplace where they will *never get used*.

(There's also the whole issue of attack surface, but this is Python,
so you probably don't want to open that particular Pandora's box.)

### `pip freeze` is good enough

I'll admit, `pip freeze` is much faster than dependency
resolution. Since it dumps the current list of installed packages, it
can sometimes be used to reproduce an environment (though probably not
across platforms). The problem is, it doesn't separate the
dependencies in a key way: those that I *want* vs the supporting
dependencies that I need to *get there*. This is the idea of lockfiles
that just about every package manager other than `pip` has chosen to adopt.

As a human being, I only ever want to maintain the first list:
*dependencies that I want, or need*. I need `jupyter`, but I don't
want to maintain a list of all its dependencies. I really don't want
to know what it takes to compile `scipy` or `scrapy`, and **please**
never make me peek under the hood of what it takes to get `sage` to
compile.

### `pyproject.toml` is the standardized way to do Python packaging.

Well, there is [PEP 518], sure, and that is a standard. Unfortunately,
the way that package managers (like Poetry) use these files is to put
all the configuration in a custom section, (`[tool.poetry]`). So it's
standard, but everyone's standard is different. Cue [Andew
Tanenbaum][tanenbaum-quotes]:

> The nice thing about standards is that you have so many to choose from.

So take your pick. `requirements.txt`, `setup.cfg`, `MANIFEST.in`,
`Pipfile`, `environment.yml`. They could all be thought of as
*standard*.

[warrick-pipenv]: https://chriswarrick.com/blog/2018/07/17/pipenv-promises-a-lot-delivers-very-little/ "Pipenv: promises a lot, delivers very little"
[reddit-pipenv]: https://np.reddit.com/r/Python/comments/8jd6aq/why_is_pipenv_the_recommended_packaging_tool_by/ "The Reddit thread that killed the 'Pipenv is official' claim"
[hatch]: https://github.com/ofek/hatch
[poetry]: https://github.com/sdispater/poetry
[hn-if-its-dead]: https://news.ycombinator.com/item?id=21781421
[pip-tools]: https://github.com/jazzband/pip-tools
[conda]: https://github.com/conda/conda
[virtualenvwrapper]: https://bitbucket.org/virtualenvwrapper/virtualenvwrapper/src/master/
[pyenv]: https://github.com/pyenv/pyenv
[pipenv]: https://github.com/pypa/pipenv
[pep 518]: https://www.python.org/dev/peps/pep-0518/
[solh]: https://blog.glowforge.com/strong-opinions-loosely-held-might-be-the-worst-idea-in-tech/
*[HDD]: Hard Disk Drive