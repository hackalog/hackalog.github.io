---
layout: post
title: Making a git-friendly Catalog Format
date: 2021-05-25
categories: [python, reproducibility, easydata, git]
excerpt: API Lessons learned from a year of building (and using) Easydata
---
TL;DR: API Lessons learned from a year of building (and using) Easydata

After a year of using it, I'd say we got a lot of things right in [Easydata]. We made it to our 1.0 release last summer (introducing the `Dataset.load()` API), and, over the course of several workshops, hammered out a set of changes for working with large datsets (remote data and the EXTRA API), and private data.

[Easydata]: https://github.com/hackalog/easydata

That said, we also got a few things wrong, and now it's time to go ahead and fix one of those things: making the catalog format more git-friendy. This is a breaking change, so this change will form the start of what will become Easydata 2.0, the rest of which will be documented in my [next post][API Cleanup].

[API Cleanup]: /api-changes

## On-disk Catalog Format

When we first picked an on-disk catalog format, we hadn't thought about designing for minimizing potential git conflicts. Since a git workflow is a fairly core piece of [Easydata], we're going to right that wrong.

Following in the "implement the obvious thing first" philosophy, our initial serialization format for the `DatasetGraph` hypergraph was  a pair of JSON files: one for the datasets (nodes), and one for the transformers (edges).

While this works in practice, when we use Easydata in a busy workshop, it has a downside: it's a ripe source of **git conflicts**. How? Well, when two participants both make changes to their respective data catalogs, there's a strong possibility of a git conflict when someone goes to merge those changes.

For 2.0, we're going to try a straightforward format change. Instead of a catalog consisting of multiple JSON files with one entry per node/edge, let's make the catalog consist of **multiple directories**, and have one *file* per node/edge. Dataset names are necessarily unique (as are transformer names, though it's *much* less common to refer to them by name), so it seems natural. In other words, instead of

* `catalog/datasets.json`
* `catalog/transformers.json`

we now have
* `catalog/datasets/*.json`
* `catalog/transformers/*.json`

As an added bonus, the Catalog class can be used wherever we need a catalog of serializable objects; e.g. for our `DataSource` objects as well.

## The Catalog class

A `Catalog` object is a serializable, disk-backed git-friendly dict-like object for storing a data catalog.

* **serializable** means anything stored in the catalog must be serializable to/from JSON.
* **disk-backed** means all changes are reflected immediately in the on-disk serialization.
* **git-friendly** means this on-disk format can be easily maintained in a git repo (with minimal
     issues around merge conflicts), and
* **dict-like** means programmatically, it acts like a Python `dict`.

On disk, a Catalog is stored as a directory of JSON files, one file per object The stem of the filename (e.g. `stem.json`) is the key (name) of the catalog entry in the dictionary, so `catalog/key.json` is accessible through the API as `catalog['key']`.

Making this change let us deprecate a whole lot of arbitrary methods in our API, which got the ball rolling on a massive [API cleanup]. More about that soon.
