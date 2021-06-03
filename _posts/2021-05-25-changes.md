---
layout: post
title: API Ch-ch-changes
date: 2020-06-03
categories: [python, reproducibility, easydata]
excerpt: API Lessons learned from a year of building (and using) Easydata
---
TL;DR: API Lessons learned from a year of building (and using) Easydata

After a year of using it, I'd say we got a lot of things right in [Easydata]. We made it to our 1.0 release last summer (introducing the `Dataset.load()` API), and, over the course of several workshops, hammered out a set of changes for working with large datsets (remote data and the EXTRA API), and private data.

That said, we also got a few things wrong, and now it's time to go ahead and fix one of those things: making the catalog format more git-friendy. This is a breaking change, so this change will form the start of what will become Easydata 2.0, where we do some serious API cleanup.1

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

Making this change let us deprecate a whole lot of arbitrary methods in our API.

## Migrating to Easydata 2
Since there's probably a few older Easydata repos out there, here's a quick guide to migrating to the proposed Easydata 2 way of doing things:

### Loading a Catalog
* Old: `load_catalog(catalog_name)`
* New: `Catalog.load(catalog_name)`

In addition, we had defined some helpful (partial) functions to load these; i.e.

* `dataset_catalog = partial(load_catalog, catalog_file='datasets.json')  # Old way`
* `transformer_catalog = partial(load_catalog, catalog_file='transformers.json') # Old way`
* `datasource_catalog = partial(load_catalog, catalog_file='datasources.json') # Old way`


We've deprecated them, because the new form is just as clear:

* `Catalog.load('datasets')`
* `Catalog.load('transformers')`
* `Catalog.load('datasources')`

### Deleting a key
* Old: `del_from_catalog(key, catalog_file=foo)`
* New: `c = Catalog.load(foo); del c[key]`

Basically, treat the catalog as a dict, and changes will be serialized to disk automatically.

### Available catalog entries
We used to have functions like `available_datasets()`, `available_transformers()`, `available_datasources()` but again, we now simply treat these as a dict, so

Old: `if 'foo' in available_datasets() ...`
New: `c = Catalog.load('datsets'); if 'foo' in c ...`

Basically, treat the catalog as a dict.

### Eliminate the workflow module

The purpose of `src.workflow` has changed several times. In the end, we ended up using it as a place to test
out new API ideas without exposing the details to the user. By the time we cut our Easydata 2 beta, this file was effectively empty, so we eliminated it for now. Instead, import the module you want from easydata directly.

from src.data import Catalog, Dataset
from src.data.helpers import (dataset_from_csv_manual_download,
                              dataset_from_metadata
                              dataset_from_single_function)

### Add Dataset/Datasource to Catalog.

To add a dataset or datasource to its respective catalog, use the "update_catalog()" method of the
Dataset / Datasource object respectively; e.g.

```
c = Catalog.load('datsets')
ds = Dataset('new_dataset_name')
ds.update_catalog()
```

### Renamed API Calls

* `TransformerGraph` is now `DatasetGraph`
* `create_transformer_pipeline` is now `serialize_transformer_pipeline`

### New Exceptions
We introduced some Easydata-specific exceptions. We had previously been using generic ones.


### Still to-do

Easydata 2 is still in beta, as there's still some API cleanup to come. In particular:

* `add_dataset()` should be deprecated. It has two forms:
** From the dataset itself (which can be handled by `update_catalog`)
** `from_datasource`, which can be a helper in `src.data.helpers`.

* `.update_catalog()` should be write_catalog or add_to_catalog. Or both, where `add_to_catalog` would set the `create` flag, and `update` would by default omit it.