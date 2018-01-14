

Clustering
========================================================
#author: Matt Wayland
#date: 14 January, 2018
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

Hierarchic agglomerative
========================================================
type: section


========================================================
Building a dendrogram
![](img/hclust_demo_0.svg)

========================================================
Building a dendrogram continued
![](img/hclust_demo_1.svg)

========================================================
Building a dendrogram continued
![](img/hclust_demo_2.svg)

========================================================
Building a dendrogram continued
![](img/hclust_demo_3.svg)

========================================================
Building a dendrogram continued
![](img/hclust_demo_4.svg)

========================================================
Building a dendrogram continued
![](img/hclust_demo_5.svg)

========================================================

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

<img src="clustering-figure/linkageComparison-1.png" title="Dendrograms for the example distance matrix using three different linkage methods. " alt="Dendrograms for the example distance matrix using three different linkage methods. " width="100%" style="display: block; margin: auto;" /><img src="clustering-figure/linkageComparison-2.png" title="Dendrograms for the example distance matrix using three different linkage methods. " alt="Dendrograms for the example distance matrix using three different linkage methods. " width="100%" style="display: block; margin: auto;" /><img src="clustering-figure/linkageComparison-3.png" title="Dendrograms for the example distance matrix using three different linkage methods. " alt="Dendrograms for the example distance matrix using three different linkage methods. " width="100%" style="display: block; margin: auto;" />

Hierarchical clustering examples
====================================
type: prompt

- Clustering synthetic data sets (6.3.2)
- Gene expression profiling of human tissues (6.3.3)


K-means clustering
========================================================
type: section

Algorithm
========================================================
**Pseudocode for the K-means algorithm**

```
randomly choose k objects as initial centroids
while true:

1. create k clusters by assigning each object to closest centroid

2. compute k new centroids by averaging the objects in each cluster

3. if none of the centroids differ from the previous iteration: return the current set of clusters
```


K-means clustering examples
====================================
type: prompt

- Clustering synthetic data sets (6.4.4)
- Gene expression profiling of human tissues (6.4.5)

Randomly choose initial centroids
===================================

![plot of chunk unnamed-chunk-3](clustering-figure/unnamed-chunk-3-1.png)

First iteration
========================================================

![plot of chunk unnamed-chunk-4](clustering-figure/unnamed-chunk-4-1.png)
***
![plot of chunk unnamed-chunk-5](clustering-figure/unnamed-chunk-5-1.png)

Second iteration
========================================================

![plot of chunk unnamed-chunk-6](clustering-figure/unnamed-chunk-6-1.png)
***
![plot of chunk unnamed-chunk-7](clustering-figure/unnamed-chunk-7-1.png)

Third iteration
========================================================

![plot of chunk unnamed-chunk-8](clustering-figure/unnamed-chunk-8-1.png)
***
![plot of chunk unnamed-chunk-9](clustering-figure/unnamed-chunk-9-1.png)

Fourth and final iteration
========================================================

![plot of chunk unnamed-chunk-10](clustering-figure/unnamed-chunk-10-1.png)
***
![plot of chunk unnamed-chunk-11](clustering-figure/unnamed-chunk-11-1.png)

DBSCAN
========================================================
type: section

Density-based spatial clustering of applications with noise


DBSCAN examples
====================================
type: prompt

- Clustering synthetic data sets (6.5.4)
- Gene expression profiling of human tissues (6.5.5)


Prompt Slide
====================================
type: prompt


Alert Slide
====================================
type: alert

First Slide
========================================================

For more details on authoring R presentations please visit <https://support.rstudio.com/hc/en-us/articles/200486468>.

- Bullet 1
- Bullet 2
- Bullet 3

Slide With Code
========================================================


```r
summary(cars)
```

```
     speed           dist       
 Min.   : 4.0   Min.   :  2.00  
 1st Qu.:12.0   1st Qu.: 26.00  
 Median :15.0   Median : 36.00  
 Mean   :15.4   Mean   : 42.98  
 3rd Qu.:19.0   3rd Qu.: 56.00  
 Max.   :25.0   Max.   :120.00  
```

Slide With Plot
========================================================

![plot of chunk unnamed-chunk-13](clustering-figure/unnamed-chunk-13-1.png)
