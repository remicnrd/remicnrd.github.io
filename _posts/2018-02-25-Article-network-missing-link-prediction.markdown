---
layout: post
title:  Article Network missing link prediction
date:   2018-02-25 19:26:42 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: post_5.jpg # Add image post (optional)
tags: [Python, Graph Analysis, Machine Learning]
author:  # Add name author (optional)
---

The goal of this post is to **predict missing links from a network**.

The dataset is a **citation network of research articles, represented as a graph G = (V, E)**. The nodes correspond to scientific articles
and the existence of a directed edge between nodes u and v indicates that paper u cites paper v. Each article is also associated with information such as title, publication year,
author names, journal name and a short abstract.

We will build a **machine learning model to predict from a pair of nodes if an edge is present (1) or not (0)**. A number of edges have been randomly removed from the initial graph, and the binary classifier performance will be measured on a test subset.

## Get the network as a graph


```python
import pandas as pd

training = pd.read_csv('./data/training_set.txt', sep = ' ', header = None)
training.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9510123</td>
      <td>9502114</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>9707075</td>
      <td>9604178</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9312155</td>
      <td>9506142</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



The training set has 3 columns: the first two indicating nodes IDs and the third one if there is a link between the node pair. The graph is directed: the first node is the source (quoting article) and the second node is the target (quoted article).

We also have access to the file `node_information`, containing additional information about the research articles. The file has no Header and columns can be named for the sake of clarity.

We can use id as index, which will make information retrieval easier later.


```python
node_info = pd.read_csv('./data/node_information.csv', header= None)
node_info.columns = ['id', 'pub_year', 'title', 'authors', 'journal_name', 'abstract']
node_info = node_info.set_index('id')
node_info.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pub_year</th>
      <th>title</th>
      <th>authors</th>
      <th>journal_name</th>
      <th>abstract</th>
    </tr>
    <tr>
      <th>id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1001</th>
      <td>2000</td>
      <td>compactification geometry and duality</td>
      <td>Paul S. Aspinwall</td>
      <td>NaN</td>
      <td>these are notes based on lectures given at tas...</td>
    </tr>
    <tr>
      <th>1002</th>
      <td>2000</td>
      <td>domain walls and massive gauged supergravity p...</td>
      <td>M. Cvetic, H. Lu, C.N. Pope</td>
      <td>Class.Quant.Grav.</td>
      <td>we point out that massive gauged supergravity ...</td>
    </tr>
    <tr>
      <th>1003</th>
      <td>2000</td>
      <td>comment on metric fluctuations in brane worlds</td>
      <td>Y.S. Myung, Gungwon Kang</td>
      <td>NaN</td>
      <td>recently ivanov and volovich hep-th 9912242 cl...</td>
    </tr>
  </tbody>
</table>
</div>



Now that we have everything, we can start to build our network!

We take the IDs from `node_info`, that contains every node of the entire graph (while the other files are just node pairs and would exclude unlinked nodes). We then add the edges from our training set.

A good library to deal with networks is the python package [NetworkX](https://networkx.github.io/). Important for the graph creation: the correct function for this usecase is `DiGraph` and not `Graph` as the network is directed.


```python
import networkx as nx

IDs = [node_id for node_id in node_info.id]

training_list = training.values.tolist() # training dataframe convertion for easy edges list comprehension below
edges = [(node_pair[0], node_pair[1]) for node_pair in training_list if node_pair[2] == 1]

G = nx.DiGraph()
G.add_nodes_from(IDs)
G.add_edges_from(edges)

print("Number of nodes : " + str(G.number_of_nodes()))
print("Number of edges : " + str(G.number_of_edges()))
```

    Number of nodes : 27770
    Number of edges : 335130


## Compute the predictors

The graph contains a lot of nodes and edges. Let's create a subset to keep the learning time of our model more laptop-suitable, it shouldn't impact the performance.


```python
import random

training_reduced = training.sample(frac=0.05) # We keep 5%
training_reduced.columns = ['source', 'target', 'Y']

training_reduced.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>source</th>
      <th>target</th>
      <th>Y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>348093</th>
      <td>110261</td>
      <td>9612133</td>
      <td>1</td>
    </tr>
    <tr>
      <th>496903</th>
      <td>9712054</td>
      <td>9602174</td>
      <td>1</td>
    </tr>
    <tr>
      <th>337809</th>
      <td>1202</td>
      <td>9903249</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Now that the training dataset is ready, time to create our future predictors. We will use the following node descriptive metrics:

- **Degree centrality**: The fraction of nodes of the total graph each node is connected to. We are interested by the *out-degree* centrality of the source node (percentage of papers the article is quoting), and the *in-degree* centrality of the target node (percentage of papers the article is being quoted by).
- **Page ranking**:  The ranking of the node in the graph based on the structure of the incoming links. We are interested in the page ranking of our target.
- **Preferential attachment**: Express the likeliness of a connection between two nodes based on their existing connectiveness. The function is not implemented in NetworkX for directed graph so we define it.
- **HITS algorithm**: Define a Hub and Authority value. A good Hub points to many other nodes, a good Authority is pointed by many Hubs. We are interested in the Hub score for the source, and the Authority score for the target.


```python
# Degree Centrality features
out_degree_centrality = nx.out_degree_centrality(G)
in_degree_centrality = nx.in_degree_centrality(G)
training_reduced['source_out_centrality'] = training_reduced.apply(lambda row: out_degree_centrality[row.source],axis=1)
training_reduced['target_in_centrality'] = training_reduced.apply(lambda row: in_degree_centrality[row.target],axis=1)

# Page rank
page_rank = nx.pagerank_scipy(G)
training_reduced['target_pagerank'] = training_reduced.apply(lambda row: page_rank[row.target],axis=1)

# Preferential Attachment
# For a directed graph, is equal to K_out_source * K_in_target with K the number of neighbors. Which is equivalent to multiply the available centralities.
training_reduced['preferencial_attachment'] = training_reduced.apply(lambda row: row.source_out_centrality * row.target_in_centrality,axis=1)

# HITS algorithm
hub_score, authority_score = nx.hits(G)
training_reduced['source_hub_score'] = training_reduced.apply(lambda row: hub_score[row.source],axis=1)
training_reduced['target_authority_score'] = training_reduced.apply(lambda row: authority_score[row.target],axis=1)
```


```python
training_reduced.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>source</th>
      <th>target</th>
      <th>Y</th>
      <th>source_out_centrality</th>
      <th>target_in_centrality</th>
      <th>target_pagerank</th>
      <th>preferencial_attachment</th>
      <th>source_hub_score</th>
      <th>target_authority_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>348093</th>
      <td>110261</td>
      <td>9612133</td>
      <td>1</td>
      <td>0.001260</td>
      <td>0.000180</td>
      <td>0.000015</td>
      <td>2.269434e-07</td>
      <td>7.067421e-07</td>
      <td>4.779942e-08</td>
    </tr>
    <tr>
      <th>496903</th>
      <td>9712054</td>
      <td>9602174</td>
      <td>1</td>
      <td>0.001008</td>
      <td>0.001765</td>
      <td>0.000064</td>
      <td>1.779236e-06</td>
      <td>5.649810e-05</td>
      <td>7.506221e-05</td>
    </tr>
    <tr>
      <th>337809</th>
      <td>1202</td>
      <td>9903249</td>
      <td>1</td>
      <td>0.000360</td>
      <td>0.000612</td>
      <td>0.000050</td>
      <td>2.204593e-07</td>
      <td>4.574718e-07</td>
      <td>9.512625e-07</td>
    </tr>
  </tbody>
</table>
</div>



We can also extract a lot of information from the features of `node_info`:
- **Publication year**: A paper cannot quote a more recent paper. We compute the difference between the publication year of the target and the publication year of the source.
- **Title**: Title similarity induce similar topics, and enhance the chances of articles being linked. We can use Sentence2Vec (word embedding) similarity with [SpaCy](https://spacy.io/).
- **Abstract**: We follow the same reasoning that for the title.
- **Authors**: Papers sharing authors should have higher chances of being connected. We count the number of common authors.


```python
# Publication year
training_reduced['pub_year_difference'] = training_reduced.apply(lambda row: node_info.pub_year[row.source] - node_info.pub_year[row.target] ,axis=1)
training_reduced['pub_year_difference']=training_reduced['pub_year_difference'].where(training_reduced['pub_year_difference'] < 0, -1)

# Title
import spacy
nlp = spacy.load('en_core_web_lg')
training_reduced['title_similarity'] = training_reduced.apply(lambda row: nlp(node_info.title[row.source]).similarity(nlp(node_info.title[row.target])) ,axis=1)

# Abstract
training_reduced['abstract_similarity'] = training_reduced.apply(lambda row: nlp(node_info.abstract[row.source]).similarity(nlp(node_info.abstract[row.target])) ,axis=1)

# Authors
node_info['authors'] = node_info['authors'].fillna(value='')
training_reduced['common_authors'] = training_reduced.apply(lambda row: len(set(node_info.authors[row.source].split(",")).intersection(set(node_info.authors[row.target].split(",")))) ,axis=1)
```


```python
training_reduced.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>source</th>
      <th>target</th>
      <th>Y</th>
      <th>source_out_centrality</th>
      <th>target_in_centrality</th>
      <th>target_pagerank</th>
      <th>preferencial_attachment</th>
      <th>source_hub_score</th>
      <th>target_authority_score</th>
      <th>pub_year_difference</th>
      <th>title_similarity</th>
      <th>abstract_similarity</th>
      <th>common_authors</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>348093</th>
      <td>110261</td>
      <td>9612133</td>
      <td>1</td>
      <td>0.001260</td>
      <td>0.000180</td>
      <td>0.000015</td>
      <td>2.269434e-07</td>
      <td>7.067421e-07</td>
      <td>4.779942e-08</td>
      <td>-1</td>
      <td>0.781564</td>
      <td>0.972482</td>
      <td>0</td>
    </tr>
    <tr>
      <th>496903</th>
      <td>9712054</td>
      <td>9602174</td>
      <td>1</td>
      <td>0.001008</td>
      <td>0.001765</td>
      <td>0.000064</td>
      <td>1.779236e-06</td>
      <td>5.649810e-05</td>
      <td>7.506221e-05</td>
      <td>-1</td>
      <td>0.839936</td>
      <td>0.657183</td>
      <td>0</td>
    </tr>
    <tr>
      <th>337809</th>
      <td>1202</td>
      <td>9903249</td>
      <td>1</td>
      <td>0.000360</td>
      <td>0.000612</td>
      <td>0.000050</td>
      <td>2.204593e-07</td>
      <td>4.574718e-07</td>
      <td>9.512625e-07</td>
      <td>-1</td>
      <td>0.547695</td>
      <td>0.848389</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



We now have 10 predictors! Some of them express a certain part of common information, we can see it on the correlation matrix of features as well:


```python
%matplotlib inline

import seaborn as sns
import matplotlib.pyplot as plt

plt.figure(figsize=(14,12))
sns.heatmap(training_reduced.corr(),
            vmax=0.5,
            square=True,
            annot=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x14b527dd8>




![png]({{site.baseurl}}/assets/img/output_19_1.jpg)


However, it cannot hurt our model performance so let's keep them for now.

## Build the model

We first want to create a *train,test split* to have an idea of the performance of the classifier.


```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(training_reduced.drop(['source', 'target', 'Y'], axis= 1), training_reduced.Y, test_size=0.2)
```

A Random Forest is often a good idea in this kind of prediction task. Indeed, Random Forest does not require a lot of specific feature engineering, and gives a good idea of important features. We use the [scikit-learn](http://scikit-learn.org/stable/) library.


```python
from sklearn.ensemble import RandomForestClassifier

RF_classifer =  RandomForestClassifier(n_estimators= 1000)
RF_classifer.fit(X_train, y_train)
```




    RandomForestClassifier(bootstrap=True, class_weight=None, criterion='gini',
                max_depth=None, max_features='auto', max_leaf_nodes=None,
                min_impurity_decrease=0.0, min_impurity_split=None,
                min_samples_leaf=1, min_samples_split=2,
                min_weight_fraction_leaf=0.0, n_estimators=1000, n_jobs=1,
                oob_score=False, random_state=None, verbose=0,
                warm_start=False)



We now test on our freshly created X_test:


```python
RF_classifer.score(X_test, y_test)
```




    0.9074074074074074



A score of .9 accuracy is really good here! There is also other interesting metrics we could look at, such as the F1 score for example. It all depends on the priorities in predictions.

In case of prediction on new data, keep in mind that all the previously defined preprocessing tasks need to be done on the new set. The code would look like this:


```python
testing.columns = ['source', 'target']

# Degree Centrality features
testing['source_out_centrality'] = testing.apply(lambda row: out_degree_centrality[row.source],axis=1)
testing['target_in_centrality'] = testing.apply(lambda row: in_degree_centrality[row.target],axis=1)
# Page rank
testing['target_pagerank'] = testing.apply(lambda row: page_rank[row.target],axis=1)
# Preferential Attachment
# For a directed graph, is equal to K_out_source * K_in_target with K the number of neighbors. Which is equivalent to multiply the available centralities.
testing['preferencial_attachment'] = testing.apply(lambda row: row.source_out_centrality * row.target_in_centrality,axis=1)
# HITS algorithm
testing['source_hub_score'] = testing.apply(lambda row: hub_score[row.source],axis=1)
testing['target_authority_score'] = testing.apply(lambda row: authority_score[row.target],axis=1)
# Publication year
testing['pub_year_difference'] = testing.apply(lambda row: node_info.pub_year[row.source] - node_info.pub_year[row.target] ,axis=1)
testing['pub_year_difference']=testing['pub_year_difference'].where(testing['pub_year_difference'] < 0, -1)
# Title
testing['title_similarity'] = testing.apply(lambda row: nlp(node_info.title[row.source]).similarity(nlp(node_info.title[row.target])) ,axis=1)
# Abstract
testing['abstract_similarity'] = testing.apply(lambda row: nlp(node_info.abstract[row.source]).similarity(nlp(node_info.abstract[row.target])) ,axis=1)
# Authors
testing['common_authors'] = testing.apply(lambda row: len(set(node_info.authors[row.source].split(",")).intersection(set(node_info.authors[row.target].split(",")))) ,axis=1)
```


```python
predictions = RF_classifer.predict(testing.drop(['source', 'target'], axis = 1))
predictions_df = pd.DataFrame(predictions)
predictions_df.to_csv('predictions.csv')
```

Here we are! We have **created a model to predict links between nodes, based on both article information and network attributes**.
We also saw how to **test the performance of that model, and make predictions on new data**.

Here, we did not actually represent our network, but if you are interested in such a thing, take a look at [my article on the topic](https://remicnrd.github.io./Graph-dynamic-visualization/)!
