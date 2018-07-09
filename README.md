# Power2TheWiki
Final project for the Mining Massive Datasets course at UCU.

## Terminology
* **Page** - either an acual page in Wikipedia or a non-existing page having red links pointing to it (see Red Page).
* **Blue Page** - existing page of Wikipedia that has blue links pointing to it.
* **Red Page** - non-existing page of Wikipedia that has red links pointing to it.

## Notation
* **G-L** - graph of all pages of the Wikipedia in language L.
* **G-Lb** - graph of blue pages of the Wikipedia in language L.
* **V-L** - nodes of graph G-L. All pages of the Wikipedia in language L.
* **V-Ls** - pages of the Wikipedia in language L with status s є {b, r} (blue or red pages). V-Lb represents the nodes of graph G-Lb. V-Lb is a set of red pages. Graph G-Lr does not exist, because red pages don't have any outgoing links.
* **E-L** - edges of graph G-L. Links between all pages of the Wikipedia in language L.
* **E-Lb** - edges of graph G-Lb. Links between blue pages of the Wikipedia in language L.

## Problem Statement
Among all red pages in the Wikipedia in language A find those that can be linked to some blue page in the Wikipedia in language B.

## Plan
1. Collect the data
2. Create the following graphs. In all graphs E = {links between pages}
    1. **G-EN**: V = {all pages of English Wikipedia}
    2. **G-UK**: V = {all pages of Ukrainian Wikipedia}
    3. **G-ENb**: V = {blue pages of English Wikipedia} (subset of EN)
    4. **G-UKb**: V = {blue pages of Ukrainian Wikipedia} (subset of UK)
    5. **G-Q**: V = {pages from ENb and UKb that are the translations of each-other}
3. Create the following subsets of nodes:
    1. **V-ENr** = {red pages from G-EN}
    2. **V-UKr** = {red pages from G-UK}
4. Baseline. Find the nearest neighbours for all nodes from V-ENr in V-UKb and for all nodes from V-UKr in V-ENb using the following similarity measures from graph theory
    1. Number of neighbours in common (a.k.a. mutual friends).
5. Create graph embeddings
6. Find the nearest neighbours for all nodes from V-ENr in V-UKb and for all nodes from V-UKr in V-ENb using graph embeddings
7. Compare the results to the baseline

## Research Questions
1. If almost all red pages have only one incoming link from page X, the idea of graph isomorphism will make little sense because we will be looking for similarity based on a single common neighbour. And this means that any other neighbour of X would be a potential candidate.
    1. What is the average number of incoming links of red pages?
    2. What percent of red pages have the number of incoming links greater than some threshold = 3, 4, 5, ...?

## 1. Collecting the data
We have downloaded the full dumps of [English](https://dumps.wikimedia.org/enwiki/20180620/) and [Ukrainian](https://dumps.wikimedia.org/ukwiki/20180620/) Wikipedia articles from 20/06/2018. Then we have parsed those articles to get outgoing red and blue links: the link article, text and position in the current article text. Also we have downloaded [Wiki interlanguage link records](https://dumps.wikimedia.org/ukwiki/20180620/ukwiki-20180620-langlinks.sql.gz) and parsed out all interlingual links between English and Ukrainian Wikipedia articles.

----------

## Diego's comments [TEMPORARY SECTION]

Start with "finding duplicate/similar nodes in one graph".  More formally: 
"Lets consider the two Graphs, with G_en:{V_en, E_en} and  G_ua =:{V_ua, E_ua}, with V_x being the wikipedia pages in language X, and E_x the links between those pages. Each node V_x has an status s  blue (page exists) or red (page doesn't exists). For all nodes in G_en and G_ua with s=blue, there is direct mapping Q. Let's now create a third graph G_q, where all the nodes with the mapping Q(V_en_i == V_ua_i) => Q_i, where Q_i inherit all the links from V_en_i and V_ua_i. The task is learning from G_q_blue the probability of  P of P(V_en_j == V_ua_j ==> True), predict the probability of nodes in V_en_red to have a mapping in a node V_ua_blue"

in other words, you will create the graph for each editions, and then will merge those graphs using the wikidata mapping. 
To train your model, you can just the blue nodes, because all them have mappings (wikidata items), but you can use a set of them for experimentation. So imagine that you have 500K pages that are both in enwiki and uawiki, you will train your model with 400K and use the other 100K as test set. Finally as validation set you will use some manually evaluated pairs, using the red links.

Yes, but here the important part is not in the supervised learning algorithm that you use, but in the notion of node similarity that you use.
To measure the distance/similarity between nodes you can use traditional graph theory (for example number of neighbors in common AKA mutual friends) or if you want to do something more trendy you can try with graph embeddings. There is a very interesting and easy to use python library for Graph embeddings: https://github.com/palash1992/GEM

you are right, maybe in the first step you should just merge the edges, but no the nodes.

Ok, you need to design this, but think in something line a graph with of wikidata items, where which wikidata item is mapped to their corresponding pages in en and uk

take all the nodes that you have both in en and uk, and using the ~80% of this nodes, create your graph
now, for the reaming 20%, you added as two different nodes
then, create the embedding of that graph, and query over the nodes in 20% duplicated, to find nearest neighborhood
so, the probability of two nodes being the same, would be some distance (eg cosine similarity) metric

then you task would be: 1) add some weight in the edges, 2) try with different embedding
then you can report the distribution of probabilities between true positives

if you try multiple embeddings, and ways to weight edges and build the graph, you will need to combine them
I would add layer on top of this, using different embeddings plus other features
but if you just do the first part... it might a contribution

also instead of using the distance metric, you can use some supervised classification to perform the task directly over the embedded vectors
distance is uni dimensional, but maybe the similarity that we are looking for is embed not equally in all dimensions

I would compare standalone approaches (nearest neig with from one embedding) with aggregated systems
we are researching, you usually start with the most simple approach and build on top of this
but it's difficult to know without seen some results.

My overall recommendation, build the dataset as soon as possible, an start computing the most basics statistics such as correlation between variables, or basic classifiers such as logistic regression (if makes sense for your project, obviously)
