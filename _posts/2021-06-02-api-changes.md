---
layout: post
title: API Ch-ch-changes
date: 2021-06-02
categories: [python, reproducibility, easydata]
excerpt: A quick guide to the changes in Easydata 2
---
As mentioned in the [last post], the upcoming Easydata 2.0 release is all about API and UX lessons we learned in the last year of using, and developing the Easydata framework.

[last post]: /git-friendly-catalog/

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

* Old: `if 'foo' in available_datasets() ...`
* New: `c = Catalog.load('datsets'); if 'foo' in c ...`

Basically, treat the catalog as a dict.

## Building Transformers

One of our favourite new features is the ability to use a Jupyter Notebook in place of a transformer function.
It's as easy as writing a Dataset to disk inside your notebook and then doing a:
```
dsdict = notebook_as_transformer(notebook_name='my_notebook.ipynb',
                                 input_datasets=[ds_in],
                                 output_datasets=[ds_out],
                                 overwrite_catalog=True)
```

## Eliminating the workflow module

The purpose of `src.workflow` has changed several times. In the end, we ended up using it as a place to test
out new API ideas without exposing the details to the user. By the time we cut our Easydata 2 beta, this file was effectively empty, so it has returned to its original purpose (handling commands like "make datasets" and "make datasources").

For the rest of the functions, that used to be there, import the module you want from easydata directly.

```
from src.data import Catalog, Dataset
from src.helpers import (dataset_from_csv_manual_download,
                         dataset_from_metadata
                         dataset_from_single_function)
```
### Adding Dataset/Datasource to Catalog.

To add a dataset or datasource to its respective catalog, use the "update_catalog()` method of the
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


### "force" flags and other misnamed options
`force` was a terrible name for an option flag, as it meant something slightly different
to every function, leading to some odd bugs. It has been replaced in most cases with a clearer name:

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
* From the dataset itself: Now `dataset.update_catalog()` (which can be handled by `update_catalog`)
* Using the `from_datasource` option: Now `Dataset.from_datasource()`

### Changing the log level
`src.log.debug` is now gone (it did not work correctly anyway). Set the `LOGLEVEL` environment variable instead.


I'm sure there are many other changes I forgot about, but these should get you going. (and the [Easydata documentation] and docstrings should get you the rest of the way!)

[easydata documentation]: https://cookiecutter-easydata.readthedocs.io