
Clustering
========================================================
#author: Matt Wayland
#date: 21 September, 2018
autosize: true
transition: rotate
css: custom.css

Clustering
===

Clustering attempts to find groups (clusters) of similar objects. The members of a cluster should be more similar to each other, than to objects in other clusters. Clustering algorithms aim to minimize intra-cluster variation and maximize inter-cluster variation.

Methods of clustering can be broadly divided into two types:

- **Hierarchic** techniques produce dendrograms (trees) through a process of division or agglomeration.

- **Partitioning** algorithms divide objects into non-overlapping subsets (examples include k-means and DBSCAN)


========================
<img src="clustering-figure/clusterTypes-1.png" title="Example clusters. **A**, *blobs*; **B**, *aggregation* [@Gionis2007]; **C**, *noisy moons*; **D**, *different density*; **E**, *anisotropic distributions*; **F**, *no structure*." alt="Example clusters. **A**, *blobs*; **B**, *aggregation* [@Gionis2007]; **C**, *noisy moons*; **D**, *different density*; **E**, *anisotropic distributions*; **F**, *no structure*." width="80%" style="display: block; margin: auto;" />


Distance metrics
========================================================
type: section

Euclidean distance
========================================================
<img src="clustering-figure/euclideanDistanceDiagram-1.png" title="Euclidean distance." alt="Euclidean distance." width="75%" style="display: block; margin: auto;" />

Distance matrix computation
=======================================================
The function **dist** computes the distances between rows of a data matrix.



```r
x
```

```
  V1 V2 V3
a  6  5  1
b  9  4  7
c  3  2  8
```

```r
dist(x)
```

```
         a        b
b 6.782330         
c 8.185353 6.403124
```

```r
sqrt((6-9)^2 + (5-4)^2 + (1-7)^2)
```

```
[1] 6.78233
```

Hierarchic agglomerative
========================================================
type: section

Building a dendrogram
========================================================
![](img/hclust_demo_0.svg)

Building a dendrogram continued
========================================================
![](img/hclust_demo_1.svg)

Building a dendrogram continued
========================================================
![](img/hclust_demo_2.svg)

Building a dendrogram continued
========================================================
![](img/hclust_demo_3.svg)

Building a dendrogram continued
========================================================
![](img/hclust_demo_4.svg)



Linkage algorithms
========================================================
type: sub-section


========================================================


Distance matrix


|   |A  |B  |C  |D  |
|:--|:--|:--|:--|:--|
|B  |2  |   |   |   |
|C  |6  |5  |   |   |
|D  |10 |10 |5  |   |
|E  |9  |8  |3  |4  |




Merge distances


|Groups        | Single| Complete| Average|
|:-------------|------:|--------:|-------:|
|A,B,C,D,E     |      0|        0|     0.0|
|(A,B),C,D,E   |      2|        2|     2.0|
|(A,B),(C,E),D |      3|        3|     3.0|
|(A,B)(C,D,E)  |      4|        5|     4.5|
|(A,B,C,D,E)   |      5|       10|     8.0|

==================================




























































```
Error in library(ggdendro) : there is no package called 'ggdendro'
```
