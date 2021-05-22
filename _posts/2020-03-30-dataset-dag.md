---
layout: post
title: Building a Dataset Dependency DAG for Easydata
date: 2020-03-30
categories: [python, reproducibility, easydata, hypergraph]
excerpt: We thought we were building a graph of dependencies. Turns out we had a hypergraph.
---
TL;DR: We thought we were building a graph of dependencies. Turns out we had a hypergraph.

## Building a Dataset Dependency DAG for Easydata

I spent the last little bit working on really fun problem: dataset dependencies. One of our design goals for Easydata is to be able to start an analysis like this:

```
>>>  ds = Dataset.load(f"covid-19-nlp-{date}", date="2020-04-01")
```

This would do a bunch of magic behind the hood:

* (**Caching**) it would check if a cached version of the dataset already exists, (returning this cached copy if so). Otherwise
* (**Dataset Generation**) it would generate any intermediate files needed to generate this dataset (all the way back to the raw data, if need be), then apply a sequence of **transformer functions** to turn the raw data into a processed dataset, then
* (**Check Hashes**) it would hash and check the generated datasets to ensure that, if this command had been previously executed, nothing about my generated dataset has changed

Until recently, I referred to this process as the **Dataset Dependency DAG**, assuming that it would be implemented as a directed acyclic graph (DAG); i.e. edges would be transformer functions that look like

```
>>> def transformer(input_dset: Dataset, **kwarg) -> Dataset
```
and nodes would be `Dataset` objects. These could be easily pipelined together, as all transformers consumed and generated the same data type, and the remaining kwargs could be serialized to a json blob for the [Easydata] catalog, so Bob's your uncle.

### Well, DUH

Unfortunately, when we started looking at our collection of real-world examples of transformer functions (see [reproallthethings]), we came to the conclusion that what we had wasn't a **directed graph** of data dependencies, it was a **directed hypergraph**, as our real-world collection of data transformations includes such functions as:

[reproallthethings]: https:/github.com/acwooding/reproallthethings
[easydata]: https://github.com/hackalog/easydata

* `train-test-split`: takes as input a single dataset and produces two children: **one parent, two children**.
* `augment-dataset`: takes as input two datasets and joins them along various axes: **two parents, one child**
* `subsample-dataset`: Takes as input a dataset and produces a smaller dataset by subsampling the rows: **one parent, one child**.

In its most general form, therefore, a dataset transformer function *takes in an arbitrary number of datasets, and produces an arbitrary number of datasets*; i.e. a transformer function is a **hyperedge**, not an edge, and so my data dependencies are best described by a "Directed Acyclic Hypergraph" (DAH).

Unfortunately, a "DAH" doesn't have the same ring to it as DAG. I complained about this online, and a colleague fixed this problem for me:

> Obviously acyclic generalizes to a different concept in hypergraphs than what you have. The correct term for the lack of of cycles in your hypergraph is "uncyclic", so, um ... DUH

With the [hardest computer science problem][naming] out already of the way, we came to the next hurdle. I don't have a handy mental model for how to implement this `Dataset` hypergraph (DUH) in python: the actual data structures and algorithms I'll use to issue the sequence of data transformation calls, or the APIs I'll need to be able to chain these transformer functions together in a pipeline.

[naming]: https://martinfowler.com/bliki/TwoHardThings.html

Where as before, I could insist that a transformer be a function:
```
>>> def transformer(input_dset: Dataset, **kwargs) -> Dataset
```
and just chain these together, now I have to something a little more... hyper.
Here's my current thinking:

### Serializing The Dataset Hypergraph

A *transformer function* takes in *input_datasets* and produces *output_datasets*.

Edges can be thought of as directed (parent to child), indicating a dependency. e.g. *output_datasets* depend on *input_datasets*, with an edge from one set to the other.

This will be serialized in the dataset catalog as:
```
{
    "hyperedge_1": {
        "input_datasets":[],
        "output_datasets":[],
        transformations: [
            (function_1, kwargs_dict_1 ),
            (function_2, kwargs_dict_2 ),
            ...
        ],
        "suppress_output": False,  # defaults to True
    },
    "source_edge_1": {
        "datasource_name": "ds_name",
        "datasource_opts": {},
        "output_dataset: "ds_name",
    }
}
```
Notice that source nodes are actual 1-1 edges. This is convenient from an implementation perspective.

In this abstraction, transformer functions become
```
>>> def transformer(dsdict: Dict(str,Dataset), **kwargs) -> Dict(str,Dataset)
```
Where we use kwargs to map function variables to key names in the dsdict as needed. This strings approach is needed, because the kwargs dict needs to be serializable to disk (as a json dump) to be used in the [Easydata] catalog.

### This is a Work In Progress
Of course, there are a few outstanding items from this litle brainstorm

* Does the generalization of the transformer API actually work? Can they be chained together in the way I intend? It works in my head, but my head isn't Turing complete.
* What's the hypergraph traversal algorithm; i.e. I want to give the list of transformer (hyperedges) traversed from sources to any named node in the graph. What's the directed hypergraph equivalent of the depth-first or breadth-first search here? Just do it on the complete bipartite graph and and stop when my list of nodes has been covered?
