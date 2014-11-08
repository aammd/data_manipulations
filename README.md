# Common Data Manipulations in R
Andrew MacDonald  
2014-11-07  

# Simple data manipulations in R

Many years ago, I was introduced to R by [Cam Webb ](http://camwebb.info/).  At the time, his website contained a list of common data manipulations (original [here](http://camwebb.info/blog/2014-04-29/)).  This list dated from Cam's early experience with R, and contained the R-help mailing list responses to a series of data manipulations.  For a long time, I kept this file as a handy reference.  I printed it out.  I recommended it to friends.

Now I have been using R for years, and the state of the art has advanced considerably.  Particulary, [Hadley Wickham's](https://github.com/hadley) `reshape2` and `dplyr` packages have transformed the way most useRs manipulate their data.  I decided that it would be interesting to revisit my favourite resource and try my hand at solving these problems with tools from these two packages.



```r
library(reshape2)
library(dplyr)
library(knitr)
```





## GROUP

Turn this table (A):


| c1 | c2 | c3 |
|----|----|----|
| A  | a  |  1 |
| A  | a  |  3 |
| A  | a  |  1 |
| A  | b  |  1 |
| A  | b  |  2 |
| B  | c  |  2 |
| B  | d  |  1 |


into this (B):

| Group1 | Group2 | Nrows | SumOfCol3 |
|--------|--------|-------|-----------|
| A      | a      |     3 |         5 |
| A      | b      |     2 |         3 |
| B      | c      |     1 |         2 |
| B      | d      |     1 |         1 |




```r
A <- data.frame(
       c1 = c('A', 'A', 'A', 'A', 'A', 'B', 'B'),
       c2 = c('a', 'a', 'a', 'b', 'b', 'c', 'd'),
       c3 = c(1, 3, 1, 1, 2, 2, 1))

B <- A %>%
  group_by(c1,c2) %>%
  summarize(Nrows=n(),
            SumOfCol3=sum(c3))
B %>%
	ungroup %>%
	kable
```



|c1 |c2 | Nrows| SumOfCol3|
|:--|:--|-----:|---------:|
|A  |a  |     3|         5|
|A  |b  |     2|         3|
|B  |c  |     1|         2|
|B  |d  |     1|         1|


## SPLIT

Turn col3 of the second table (B) into this (C):

| row | a | b | c | d |
|-----|---|---|---|---|
| A   | 3 | 2 | . | . |
| B   | . | . | 1 | 1 |



```r
C <- dcast(B,c1~c2,value.var="Nrows")
kable(C)
```



|c1 |  a|  b|  c|  d|
|:--|--:|--:|--:|--:|
|A  |  3|  2| NA| NA|
|B  | NA| NA|  1|  1|


Many original responders suggested the use of `table`, referring to the original dataset:



```r
C_alt<-with(A,table(c1,c2))
kable(C_alt)
```



|   |  a|  b|  c|  d|
|:--|--:|--:|--:|--:|
|A  |  3|  2|  0|  0|
|B  |  0|  0|  1|  1|


although that solution is not ["tidy" in the Hadlian sense](http://vita.had.co.nz/papers/tidy-data.pdf) -- i.e., it does not return a `data.frame`, but rather a `table` object.  You can obtain a data.frame with `dcast` directly:



```r
C_alt2<-dcast(A,c1~c2,value.var="c3",fun.aggregate=length)
kable(C_alt2)
```



|c1 |  a|  b|  c|  d|
|:--|--:|--:|--:|--:|
|A  |  3|  2|  0|  0|
|B  |  0|  0|  1|  1|



## STACK

Turn the above table (C) into this (D):

| c1 | V1 | V2 |
|----|----|----|
| A  | a  | 3  |
| A  | b  | 2  |
| A  | c  | .  |
| A  | d  | .  |
| B  | a  | .  |
| B  | b  | .  |
| B  | c  | 1  |
| B  | d  | 1  |



```r
D <- melt(C,id="c1") %>%
  arrange(c1)
kable(D)
```



|c1 |variable | value|
|:--|:--------|-----:|
|A  |a        |     3|
|A  |b        |     2|
|A  |c        |    NA|
|A  |d        |    NA|
|B  |a        |    NA|
|B  |b        |    NA|
|B  |c        |     1|
|B  |d        |     1|


## JOIN

Join these tables (E, F):

| c1 | c2 |
|----|----|
| A  |  1 |
| B  |  2 |
| C  |  3 |

| c1 | c3 |
|----|----|
| A  | a  |
| B  | a  |
| B  | a  |
| B  | b  |
| C  | c  |
| A  | b  |

to give (G):

| c1 | c3 | c2 |
|----|----|----|
| A  | a  |  1 |
| B  | a  |  2 |
| B  | a  |  2 |
| B  | b  |  2 |
| C  | c  |  3 |
| A  | b  |  1 |



```r
E<-data.frame(c1=c("A","B","C"), c2=1:3)
FF <- data.frame(c1=c("A","B","B","B","C","A"), c3=c("a","a","a","b","c","b"))
G <- left_join(FF, E)
```

```
## Joining by: "c1"
```

```r
kable(G)
```



|c1 |c3 | c2|
|:--|:--|--:|
|A  |a  |  1|
|B  |a  |  2|
|B  |a  |  2|
|B  |b  |  2|
|C  |c  |  3|
|A  |b  |  1|

the `dplyr` package supplies `left_join()`, which preserves the sequence of rows in its left argument.  Alternative, as was originally suggested, one could use `merge()` :



```r
G_merge <- merge(FF,E)
kable(G_merge)
```



|c1 |c3 | c2|
|:--|:--|--:|
|A  |a  |  1|
|A  |b  |  1|
|B  |a  |  2|
|B  |a  |  2|
|B  |b  |  2|
|C  |c  |  3|

Although columns now come out sorted.

## SUBSET 

subset Table G to give H:

| c1 | c3 | c2 |
|----|----|----|
| A  | a  |  1 |
| A  | b  |  1 |




```r
H <- filter(G,c1=="A")
kable(H)
```



|c1 |c3 | c2|
|:--|:--|--:|
|A  |a  |  1|
|A  |b  |  1|

## TRANSPOSE

transpose H to give:

| V1 | V2 |
|----|----|
| A  | A  |
| a  | b  |
| 1  | 1  |



```r
H_transpose <- data.frame(t(H))
kable(H_transpose)
```



|   |X1 |X2 |
|:--|:--|:--|
|c1 |A  |A  |
|c3 |a  |b  |
|c2 |1  |1  |


## SORT

In the original, the question suggested "up to three keys".



```r
A_arranged <- arrange(A,c1,c2,c3)
kable(A_arranged)
```



|c1 |c2 | c3|
|:--|:--|--:|
|A  |a  |  1|
|A  |a  |  1|
|A  |a  |  3|
|A  |b  |  1|
|A  |b  |  2|
|B  |c  |  2|
|B  |d  |  1|


## Conclusion

To my surprise, each of these was actually a single line.  The only exception was the first (GROUP), and that was because there are really two separate steps here -- the first to actually group the data, the second to apply summary functions to the data.  `dplyr` automates both tasks, and supplies great readability.  
