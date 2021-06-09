---
layout: post
title: API Ch-ch-changes
date: 2020-06-03
categories: [python, reproducibility, easydata]
excerpt: Migrating from Easydata1 to Easydata 2
---
TL;DR: API Lessons learned from a year of building (and using) Easydata

After a year of using it, I'd say we got a lot of things right in [Easydata]. We made it to our 1.0 release last summer (introducing the `Dataset.load()` API), and, over the course of several workshops, hammered out a set of changes for working with large datsets (remote data and the EXTRA API), and private data.

[Easydata]: https://github.com/hackalog/easydata

That said, we also got a few things wrong, and now it's time to go ahead and fix most of those things. This is a breaking API change, so we're cranking the major and calling this release Easydata 2.0.

Since there are probably a few existing Easydata users out there, here's a quick guide to migrating to the new API.

## On-disk Catalog Format
We completely changed the on-disk [catalog format]. But you knew that, because we wrote a whole blog post about it :)
[catalog format]: /git-friendly-catalog

### Loading a Catalog
* Old: `load_catalog(catalog_name)`
* New: `Catalog.load(catalog_name)`

Previously, we had defined some helpful (partial) functions to load these; i.e.

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
out new API ideas without exposing the details to the user. By the time we cut our Easydata 2 beta, this file was effectively empty, so it has returned to its original purpose (handling commands like "make datasets" and "make datasources").

For the rest of the functions, that used to be there, import the module you want from easydata directly.

from src.data import Catalog, Dataset
from src.helpers import (dataset_from_csv_manual_download,
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

The same works for DataSource objects.

### Renamed API Calls

* `TransformerGraph` is now `DatasetGraph`
* `create_transformer_pipeline` is now `serialize_transformer_pipeline`

### New Exceptions
We introduced some Easydata-specific exceptions. We had previously been using generic ones.
* EasydataError: base for all other exceptions
* ValidationError: hash check failed
* ObjectCollision: object already exists in object store (more general than a FileExistsError)
* NotFoundError: object not found in object store (more general than a FileNotFound Error)


### Eliminate "force" flags and other misnamed options
`force` was a terrible name for an option flag, as it meant something slightly different
to every function, leading to some odd bugs. It has been replaced

* `Dataset.dump(force=True)` -> `Dataset.dump(exists_ok=True)`
* `DatasetGraph.traverse()`: `force` -> `exhaustive`
* `DatasetGraph.generate()` `force`->`exhaustive`
* `DatasetGraph.add_source()`: `force`->overwrite_catalog
* `DatasetGraph.add_edge()`: `force`->overwrite_catalog

We also cleaned up some other misnamed options:
* `DatasetGraph.generate()`: `write_catalog`->`write_dataset`
* `DatasetGraph.process_edge()`: `write_catalog`->`write_datsets`

### Adding Datasets
`src.data.add_dataset()` is deprecated. It had two forms:
** From the dataset itself: Now `dataset.update_catalog()` (which can be handled by `update_catalog`)
** Using the `from_datasource` option: Now `Dataset.from_datasource()`

### Changing the log level
`src.log.debug` is now gone (it did not work correctly). Set the LOGLEVEL environment variable instead.
