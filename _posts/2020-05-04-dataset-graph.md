---
layout: post
title: Implementing the DatasetGraph
date: 2020-05-04
categories: [python, reproducibility, easydata, hypergraph]
excerpt: How the Dataset DAG became a hypergraph became the TransformerGraph.
---
TL;DR: How the Dataset DAG became a hypergraph became the DatasetGraph.

## DatasetGraph as a top-level object.

Recall from a [few weeks ago], I described a bipartite graph (or Hypergraph), now called a `DatasetGraph`, which describes how `Dataset` objects are generated from other `Dataset` objects. I originally named it a `TransformerGraph`, because that's how the directionality of the edges works out in the bipartite representation, but that turns out to be a little more confusing for the user. In the hypergraph, the `Dataset` objects are the nodes, so `DatasetGraph` it is.

[few weeks ago]: /transformers-and-datasets

One of the unintended consequences of introducing a `DatasetGraph` class in [Easydata] is that it turns out to be the right place to do a lot of things. That’s why we ended up exposing it to the user, instead of just using it internally to the `Dataset`.

Before we created the `DatasetGraph`, we used to have a top-level functions `add_transformer()` to add a dataset transformation to the global catalog. but it turns out a much more natural place to put it is in the `DatasetGraph` class directly.

Sticking with the “edges are functions, nodes are datasets” hypergraph terminology, the API becomes something like this"
```
>>> dag = DatasetGraph()

>>> xp = create_transformer_pipeline([list, of, transformer, functions, or, partials, ...])

>>> dag.add_source(datasource_name="dsrc_name", datasource_opts={}, output_dataset="dset_name")
>>> dag.add_edge(input_dataset=None, input_datasets=(),
                output_dataset=None, output_datasets=(),
                transformer_pipeline=xp, **kwargs)

>>> dataset = dag.generate('node_name')
```

This gives us a clean separation between adding a source node to the graph, and adding an edge. Both technically add edges, but the idea of a “source edge” in this hypergraph just feels weird, so the details are hidden by this API. This is perhaps why describing the dependency graph as a bipartite graph is less troublesome. See [my last post][few weeks ago] for more on that.

[easydata]: https://github.com/hackalog/easydata