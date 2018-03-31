---
layout: post
title:  Graph dynamic visualization
date:   2018-03-09 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: post_6.jpg # Add image post (optional)
tags: [Python, Graph Analysis, Dataviz]
author: # Add name author (optional)
---

My last article covered in depth [how to apply machine learning for link prediction](https://remicnrd.github.io./). It has been the occasion to play with a network dataset on a real-life usecase, but also to use the [NetworkX](https://networkx.github.io/) library.

Despite being easy to use and very complete, NetworkX seemed to me more research oriented than business oriented. For example, **the graph plotting did not appeal me very much**. Hopefully, with a bit of research and determination I found a **very nice alternative**: 

[NetworkxD3](https://bl.ocks.org/Jverma/1c52426ac2fb0ff0e785) is a python package based on NetworkX and the amazing dataviz library [D3.js](https://d3js.org/). 

**This short post will describe how to obtain a dynamic, interactive Graph visualization as html using NetworkxD3.**

We will use as a base the network of my previous article, representing academic papers as nodes and if they quote each other as links.

## Get the graph again

If you read the network analysis, nothing new here: we import the dataset, and define edges and vertices.


```python
import pandas as pd

edges_dataset = pd.read_csv('./data/training_set.txt', sep = ' ', header = None)

node_info = pd.read_csv('./data/node_information.csv', header= None)
node_info.columns = ['id', 'pub_year', 'title', 'authors', 'journal_name', 'abstract']

IDs = [node_id for node_id in node_info.id]
edges_list = edges_dataset.values.tolist() # training dataframe convertion for easy edges list comprehension below
edges = [(node_pair[0], node_pair[1]) for node_pair in edges_list if node_pair[2] == 1]
```

Now we can create the graph with NetworkX, and show some basic information:


```python
import networkx as nx

G = nx.DiGraph()
G.add_nodes_from(IDs)
G.add_edges_from(edges)

print("Number of nodes : " + str(G.number_of_nodes()))
print("Number of edges : " + str(G.number_of_edges()))
```

    Number of nodes : 27770
    Number of edges : 335130


## Create the NetworkX version

Before creating the html version of our network, let us see how the basic plotting version looks like as a comparison.

We can try to plot the full graph, but 28 000 nodes and 334 000 edges is way too much to see anything:


```python
nx.draw(G)
```



![graph-full]({{site.baseurl}}/assets/img/graph-full.jpg)


We need to reduce and take a subset of our initial dataset, only 500 nodes should be sufficient to have a good looking network:


```python
reduced_IDs = IDs[:5000]
reduced_edges = [(node_pair[0], node_pair[1]) for node_pair in edges_list if (node_pair[2] == 1 and node_pair[0] in reduced_IDs and node_pair[1] in reduced_IDs)]
```


```python
G = nx.DiGraph()
G.add_nodes_from(reduced_IDs)
G.add_edges_from(reduced_edges)
nx.draw(G)
```



![graph-reduced]({{site.baseurl}}/assets/img/graph-reduced.jpg)


This looks way better ! Still far from being what you would like to show in order to share interesting discoveries..

## Create the cool version

Wouldn't it be better if we could have more information, and interact with the plotted graph?
Time to use the NetworkxD3 package and create an html dataviz of our network!

This library is not available for installation via pip. It has to be [downloaded via github](https://github.com/Jverma/NetworkxD3), and installed by running `python setup.py install` in a terminal.

Once installed, the code itself is pretty straightforward:


```python
from NetworkxD3.NetworkxD3 import simpleNetworkx
from IPython.display import IFrame 

simpleNetworkx(G)
IFrame('Net.html', width=2000, height=1000)
```
<iframe max- width="100%"  height="1000" src="{{site.baseurl}}/assets/Net.html" frameborder="0"  allowfullscreen ></iframe>
        

Here we are! As a conclusion, I would say that representing a network this way has several benefits compared to `nx.graph`:
- It is overall better looking
- The names of each node is presented in a clear, non-intrusive way
- To hover over a node permits to reveal its neighbors directly
-  Nodes and connected component can be dragged for exploration
- The graph is force directed, which can help the intuition of connectivity of nodes


