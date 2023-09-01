# Implementation of an approximate quantile computation on large-scale data
Realization of the Greenwald & Khanna's quantile algorithm

### Explanation of GK01 algorithm

#### Basic terms

The naive algorithm for finding q-th quantile is to get an element at $⌊qN⌋$-th position (i.e. rank) in a sorted sequence of N elements.
However, this algorithm has a huge space consuption for big amount of stream data (elements arrive one by one). 
To decrease the number of elements stored in the memory, one may allow $\epsilon$ approximation of q-th quantile, when any element with rank in the range $[rank_{min}; rank_{max}]$, where $rank_{min} = ⌊(q-\epsilon)N⌋$ and $rank_{max} = ⌊(q+\epsilon)N⌋$, could be the answer.

A Greenwald & Khannan's algorithm (GK01) is based on the data structure named Summary. In my implementation it is represented as an array of arrays. Each subarray consist of 3 elements: $v, g, d$, where $v$ is a data point, $g$ is a difference between $rank_{min}$ of current element and $rank_{min}$ of element before it, $d$ is a difference between $rank_{max}$ and $rank_{min}$ of current data point. Note that $d=0$ for the first and last elements and $v_{i} ≤ v_{i+1}$ for all elements in the summary.

* $g_{0}=rank_{min}(v_{0})=1$

* $g_{i}=rank_{min}(v_{i})-rank_{min}(v_{i-1})$

* $d_{i}=rank_{max}(v_{i})-rank_{min}(v_{i})$

* $rank_{min}(v_{i})= \sum \limits _{j=0} ^{i} g_{j}$

* $rank_{max}(v_{i})= d_{i} + \sum \limits _{j=0} ^{i} g_{j}$


<!-- !!! * $err=\frac{\max \limits _{j \in N} (g_{j}+d_{j})}{2}$

* $rank(v_{i}) - err <= rank_{min}(v_{i}) <= rank(v_{i}) <= rank_{max}(v_{i}) <= rank(v_{i}) + err$ -->


### Algoritm explanation

Insertion of the element based on 4 steps:
1. Find position i to insert this element, where $ v_{i-1}≤element<v_{i}$;
2. Set $g=1$ and $d=0$ for element that will be inserted in the head or tail of Summary, or $g=1$ and $d=g_i+d_i-1$ otherwise;
3. Insert the element at $i$-th position with the right shift of elements with indexes $≥ i$;
4. Increment the number of arrived elements.

To use $\epsilon$-approximate $q$-th quantile, on each $2 \epsilon N$ step the deletion operation is performed:
1. Look for all possible sequences in the summary, where $d_{i}+\sum \limits_{k=j}^{i} g_{k} ≤ 2 \epsilon N $ for $j<i$, $ j $ is minimal and $ i $ is maximal possible index. 
2. Each sequence will be replaced with the new entry  $[v_{i}, \sum \limits_{k=j}^{i} g_{k}, d_{i}]$. 

To find $q$-th quantile, an algorithm calculates rank $r=⌊qN⌋$. For $r>N-\epsilon N$, the result is the last element in the summary, otherwise the answer is $v_{i} $ with minimum $i $ for which $rank_{max}(v_{i+1})> (q+\epsilon)N $.


### Time and space consumption comparison
> Add comparison of the time and space consumption (with numpy algorithm). Plot them depending on the sample size or time if your algorithm is for time-series quantile calculation.

The GK01 algorithm is a streaming algorithm, it depends on the sample size. The analysis was conducted on the following sample sizes: [10, 100, 1000,5000, 10000, 50000, 100000, 500000,1000000]. 

Evaluated metrics were the running time of the algorithm and the space used for data storage. Note that the amount of memory occupied by objects of classes in which the solution is implemented is the same and equals to 48 bytes. Therefore, the evaluation of occupied space was conducted for structures storing data. 

The obtained results of each metric evaluation for both algorithms are presented in comparison table. For vizualization, 2 plots are shown. The first is the dependence of the metric on the samle size, the second is the dependence of log(metric) and log(sample size). These graphs help to understand how different the values are and how the metric grows with the growing of the sample size. 

The hold analysis showed that the GK01 algorithm is optimal in space, but its execution takes much more time than the NumPy's algorithm. 

### Advantages and disadvantages of GK01

One of the biggest advantage of this algorithm is a small space consumption. It works well on a streaming data, when the number of elements is uncertain and may be large. Additionally, regardless of whether all the data has arrived to GK01 one can obtain a q-th quantile.

However, GK01 has several disadvantages. First of all, the answer is not an exact q-th quantile, but its $\epsilon$ approximate. Also, GK01 computes quantiles regarding all arrived data, while sometimes one can require the quantile of just some recent elements. Moreover, an implementation of this algorithm may have a low time perfomance on a big sample size.

