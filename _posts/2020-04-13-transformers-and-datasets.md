---
layout: post
title: Building the Dataset Hypergraph: Transformers and Datasets
date: 2020-04-13
categories: [python, reproducibility, easydata, hypergraph]
excerpt: "Easydata's Dataset dependency hypergraph, described"
permalink: /transformers-and-datasets/
---
TL;DR: Easydata's Dataset dependency hypergraph, described.

### Hypergraph or Bipartite Graph?

For this post, I'm still talking about the hypergraph of data dependencies that I mentioned [last time], however for this discussion, I’ll switch from a hypergraph-based description to a bipartite graph-based description of the dependencies.

[last time]: /dataset-dag/

Why? For starters, there's not necessarily a commonly accepted notion of a "directed hypergraph."
When I use the term, I mean that a *directed hypergraph*, where the vertices of an edge are partitioned into two sets: the *head-set* and *tail-set* of the edge.

It’s perhaps interesting (and often surprising) to note the constructs that appear when trying to describe data flow as a directed hypergraph. In our case, we often end up with a hypergraph where data originates from a transformer (like when we have synthetic, or downloaded data). This leads to a directed hyperedge with **no input nodes**, only output nodes; i.e the head-set is empty, but the tail-set is not. (What does one even call that. A “source edge”?)

Anyway, to avoid some of these rabbit holes, we can switch to a bipartite graph representation of this construct. These representations (hypergraph, bipartite graph) are interchangable. Just draw a bipartite graph with transformers (the hyper "edges") down one side of the page, Datasets (the hyper "nodes") down the other, and join them with directed edges to indicate data dependencies (inbound edges to a transformer are input datasets, outbound edges are output datasets).


### More on the Transformer Graph

A `Transformer` is a function that takes in zero or more `Dataset` objects, and produces one or more `Dataset` objects. While the functions themselves are stored in the source module (default src/user/transformers.py), metadata describing these functions and their inputs/outputs `Dataset` objects are serialzed to the catalog file `transformers.json`.

A `Dataset` is an on-disk object representing a point-in-time snapshot (a cached copy) of data and its associated metadata (a `Dataset`). The `Dataset` objects themselves are serialized (cached) to data/processed. Metadata about these objects are serialized to the catalog file datasets.json

Recall that a `Transformer Graph` is a bipartite graph with two distinct sets of vertices—corresponding to `Dataset`s, and `Transformer`s as described above—that describes the transformation of data from data sources to a processed data set.

Edges are directed, indicating the direction of dependency. e.g. `output_datasets` depend on `input_datasets`, so arrows are directed from input to output.

The whole goal of this exercise is to *capture the information about data transformations from raw data to processed data, in a way that can be serialized to disk, and committed as if it was code*. These instructions are stored in the data catalog in JSON format. There is some trickiness here, as function objects don’t serialize in a platform-independent way, so we just some make assumptions about namespace (we set up a standard location in the python module for user-generated functions: `src.user.*`), and use Python introspection to map the serialization to function objects when the pipeline is loaded.

### Transformer Serialization
Note that transformers can take *zero* datasets as input (but must produce at least one output). This special case occurs in one of two ways:

* **Synthetic Data**": The data is synthetic, and the transformer is actually generates a `Dataset` object from scratch. The JSON in this case looks like:
```
    "synthetic_source": {
        "output_dataset: "ds_name",
        "transformations": [
            (synthetic_generator, kwargs_dict),
            (optional_function_2, kwargs_dict_2 ),
            ...
        ],
}
```
* **Data Conversion**: The data originates from something that isn’t a `Dataset` (e.g. a DataSource object), and the transformer converts it to a `Dataset`. This is really no different than the synthetic data case, except we supply a dataset_from_datasource() wrapper so the user doesn’t have to constantly reimplement it:
```
    "datasource_edge": {
        "output_dataset: "ds_name",
        "transformations": [
            (dataset_from_datasource, (datasource_name), **datasource_opts} ),
            (optional_function_2, kwargs_dict_2 ),
            ...
        ],
}
```
In all other cases, a transformer consumes and emits datasets as follows:
```
    "hyperedge": {
        "input_datasets":[in_dset_1, in_dset_2],
        "output_datasets":[out_dset_1, out_dset_2],
        "transformations": [
            (function_1, kwargs_dict_1 ),
            (function_2, kwargs_dict_2 ),
            ...
        ],
        "suppress_output": False,  # defaults to True
},
```
### Dataset Serialization
A complete `Dataset` object contains both the data itself and an associated metadata dictionary. On disk, this is serialized to two files, typically located in paths['processed_data_path']. This serialization is called the cached copy of the dataset:

`dataset_name.dataset`: The complete `Dataset` Object

`dataset_name.metadata`: A copy of the metadata portion of the `Dataset`. As the `Dataset` can be quite large, metadata-only operations save time and memory by reading this file instead. If the `Dataset` has been reproducibly generated, this metadata should match whatever is serialized into the dataset catalog.

One of the design goals of [Easydata] is that this cached copy can be deleted at any time and (reproducibly and deterministically) recreated when needed.

### Dataset Metadata
The master copy of the generated metadata is stored in the catalog file: `datasets.json`.

`Dataset` metadata is fairly freeform. It is based on scikit-learn’s [Bunch] object (basically a dictionary where the keys can be accessed as attributes). This object typically contains 4 attributes: data, target (which is often None for unsupervised learning problems), metadata, and hashes. The latter contains a hash of all the non-metadata attributes of the `Dataset`; e.g.
```
    "hashes": {
        "data":"sha1:d1d5ac9a5872e09b3a88618177dccc481df022d1",
        "target":"sha1:38f65f3b11da4851aaaccc19b1f0cf4d3806f83b",
},where data and target are whatever data type makes sense for the problem at hand (e.g. matrix, pandas dataframe, nparray, etc.)
```
[bunch]: https://github.com/adrinjalali/scikit-learn/blob/bea2e2414f93fdf4558f1288377d2aa0351727b4/sklearn/utils/__init__.py#L60-L80
[easydata]: https://github.com/hackalog/easydata