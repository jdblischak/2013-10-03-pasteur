Control Flow in R
==================
    
#### Often when we're coding we want to control the flow of our actions. Control flow is simply the order in which we code and have our statements evaluated. That can be done by setting things to happen only if a condition or a set of conditions are met. Alternatively, we can also set an action to be computed for a particular number of times. There are many ways these can be achieved in R. For conditional statements, the most commonly used approaches are the constructs: 

```r
if (and else)
while
```


#### Say, for example, that we want R to print a message depending on how a number x relates to 5:

```r
x <- 5
if (x >= 5) {
    print(paste0(x, " is greater or equal than 5"))
} else {
    print(paste0(x, "is smaller than 5"))
}
```

```
## [1] "5 is greater or equal than 5"
```


#### We could also have a situation where we want an action to be executed while a condition is being matched:

```r
x <- 0
vec <- vector()
while (x < 10) {
    vec <- c(vec, x^2)
    x <- x + 1
}
vec
```

```
##  [1]  0  1  4  9 16 25 36 49 64 81
```


#### Now, and this is very important. When R evaluates the condition inside if and while statements, it is looking for a logical element, i.e., TRUE or FALSE. For example:

```r
if (4 == 3) {
    "4 equals 3"
}
```


#### The message was not printed because the condition 4 == 3 is FALSE 

```r
4 == 3
```

```
## [1] FALSE
```


#### R also offers a built-in function that is more intuitive to evaluate conditions.

```r
mat <- matrix(0:19, 5, 4)
mat <- ifelse(mat < 5, 0, mat)

# OR

mat <- matrix(sample(c(NA, 1:10), 50, replace = TRUE), 5, 10)
mat <- ifelse(is.na(mat), 0, 1)
```


#### In the first example, ifelse looks for all the conditions there are TRUE in the statement `mat < 5` and replaces them by the first action (transform in 0), otherwise does the altervative action (keep originals). In this case, I told R to keep the original `mat` values where the condition did not occur. Similarly, in the second example, I told R to replace all existing numbers with 1 and replace missing values with 0. *VERY IMPORTANT*: ifelse can be very handy, but for heavy and repetitive tasks, it is considerably slower than regular `if` and `else` statements. 

#### As above, another way of controlling the flow of actions is to set them to occur for a specific number of times. We achieve that using the command `for`

```r
salad <- c("lettuce", "carrots", "tomatos")
amounts <- c("1 bunch", "1 package", "1 bag")
for (vegetable in seq_along(salad)) {
    print(paste0("Our salad has ", amounts[vegetable], " of ", salad[vegetable]))
}
```

```
## [1] "Our salad has 1 bunch of lettuce"
## [1] "Our salad has 1 package of carrots"
## [1] "Our salad has 1 bag of tomatos"
```


#### Notice that at each loop, the index `vegetable` takes a value following the sequence of the vector salad. During the first loop, vegetable, in this particular case, is going to be 1, then 2, and finally 3. 

```r
for (index in 1:3) {
    print(index)
}
```

```
## [1] 1
## [1] 2
## [1] 3
```


#### The index can take any name you want (preferably names that do not overlap with existing objects or functions). Conventionally, when we deal with matrices or data.frames, we use the index `i` to denote rows and the index `j` to denote columns. Example:

```r
mat <- matrix(1:4, 2, 2)
mat
```

```
##      [,1] [,2]
## [1,]    1    3
## [2,]    2    4
```

```r
for (i in 1:nrow(mat)) {
    for (j in 1:ncol(mat)) {
        print(mat[i, j])
    }
}
```

```
## [1] 1
## [1] 3
## [1] 2
## [1] 4
```


#### At this point you may be wondering if we can mix the different approaches and the answer is yes. For a certain `for` loop, you may want to compute your action only if a condition is matched.

```r
set.seed(10)
mat <- matrix(rpois(16, lambda = 11), 4, 4)
mat
```

```
##      [,1] [,2] [,3] [,4]
## [1,]   11   12   14   13
## [2,]   10    6   13   11
## [3,]    6   10   10    7
## [4,]    8    8   14   13
```

```r
for (i in 1:nrow(mat)) {
    for (j in 1:ncol(mat)) {
        if (mat[i, j] >= 10) {
            next
        } else {
            print(paste0("element at row ", i, ", column ", j, ", is smaller than 5"))
        }
    }
}
```

```
## [1] "element at row 2, column 2, is smaller than 5"
## [1] "element at row 3, column 1, is smaller than 5"
## [1] "element at row 3, column 4, is smaller than 5"
## [1] "element at row 4, column 1, is smaller than 5"
## [1] "element at row 4, column 2, is smaller than 5"
```


#### Do not worry that much about the the functions `set.seed` and `rpois`. All you need to know is that `set.seed` will allow us to generate the same random numbers in different computers for comparison matters and `rpois` generates positive integers randomly following a Poisson distribution. Notice that I used `next` to skip the loop in case the condition was TRUE. You can also use the command `break` if you want to stop the loop at once. 

## Exercise
### Step 1: create a matrix called `mat.ex` with 3 rows and 10 columns using `set.seed(20); rpois(30, lambda=12)` as values;
### Step 2: create an empty numeric vector called `means` with length = ncol(mat.ex)
### Step 3: loop through the columns of `mat.ex` and calculate the respective mean values of each column;
### Step 4: if the mean calculated value is greater than 11, store it in `means`;
### Step 5: print vector `means` without zeros

#### solution:

```r
set.seed(20)
mat.ex <- matrix(rpois(30, lambda = 12), 3, 10)
means <- vector(mode = "numeric", length = ncol(mat.ex))
for (j in 1:ncol(mat.ex)) {
    if (mean(mat.ex[, j]) > 11) 
        means[j] <- mean(mat.ex[, j])
}
print(means[means != 0])
```

```
## [1] 14.67 11.33 14.33 12.67 12.00 12.67 11.67 17.00
```


# Speeding up using the family apply
#### One of the common features across most R beginners is to think everything as a loop. While this approach is valid and in many cases the fastest solution to think of, sometimes it can be very, very slow. Try the following:

```r
set.seed(10)
x <- matrix(rpois(500, lambda = 25), 20, 25)
a <- vector(mode = "numeric", length = length(x))
for (index in 1:ncol(x)) {
    a[index] <- mean(x[, index])
}
```


#### That took < 1 second to run. Although it was effectively fast, the code was relatively big (5 lines) for its small task. Fortunately, R has a family of loop functions called the `apply` family, which are slightly different but generally as fast as or faster loops than `for` loops. For the moment, you do not need to fully understand the detailed differences between a regular `for` loop and it's equivalent using the `apply` family. Let's replicate the example above using the mother function `apply`:

```r
set.seed(10)
x <- matrix(rpois(500, lambda = 25), 20, 25)
a <- apply(x, 2, mean)
```


#### `a` takes the same values as in the example before, but with much less code (3 lines) which is also simpler. In this case, `apply` evaluates the object `x` and then apply the function `mean` to its 2nd dimension, i.e., columns - dimension=1 would imply rows. `apply` assumes no margin for dimension for it's evaluations, so you always have to provide it. There are even more convinient and faster ways to calculate sum or mean of dimensions in a matrix or data.frame.

```r
set.seed(10)
x <- matrix(rpois(500, lambda = 25), 20, 25)
a <- colMeans(x)  #or colSums or rowMeans or rowSums
```


#### We can also easily create tables using the function `tapply` from this same family. 

```r
set.seed(10)
dat <- data.frame(genes = rep(paste0("gene_", 1:3), each = 5), species = rep(paste0("species_", 
    1:5), 3), gc_content = runif(15), stringsAsFactors = FALSE)
dat <- dat[-nrow(dat), ]
dat
```

```
##     genes   species gc_content
## 1  gene_1 species_1    0.50748
## 2  gene_1 species_2    0.30677
## 3  gene_1 species_3    0.42691
## 4  gene_1 species_4    0.69310
## 5  gene_1 species_5    0.08514
## 6  gene_2 species_1    0.22544
## 7  gene_2 species_2    0.27453
## 8  gene_2 species_3    0.27231
## 9  gene_2 species_4    0.61583
## 10 gene_2 species_5    0.42967
## 11 gene_3 species_1    0.65166
## 12 gene_3 species_2    0.56774
## 13 gene_3 species_3    0.11351
## 14 gene_3 species_4    0.59593
```


#### Suppose we want to know what the mean gc content per gene is:

```r
tapply(dat$gc_content, dat$genes, mean)
```

```
## gene_1 gene_2 gene_3 
## 0.4039 0.3636 0.4822
```


#### Sometimes, you may also have a very big and messy dataset and you want to obtain a table that makes things clearer to see. Say we want, for example, the gc_content of species (rows) X genes (columns). 

```r
tapply(dat$gc_content, list(dat$species, dat$genes), sum)
```

```
##            gene_1 gene_2 gene_3
## species_1 0.50748 0.2254 0.6517
## species_2 0.30677 0.2745 0.5677
## species_3 0.42691 0.2723 0.1135
## species_4 0.69310 0.6158 0.5959
## species_5 0.08514 0.4297     NA
```


#### It is very important to notice that the last value is a NA, simply because the species 5 did not have gene 3, and therefore no gc content. fortunately, we already know how to get rid of NAs in tables if we want to.

```r
tab <- tapply(dat$gc_content, list(dat$species, dat$genes), sum)
tab <- ifelse(is.na(tab), 0, tab)  #try also tab[is.na(tab)]  <-  0
```


#### Finally, you can customize your own functions and tomorrow John is going to teach you how to embed your customized functions inside `apply` calls. 

# Dealing with lists
#### R offers a very powerful class of objects called lists. A list can virtually contain anything. You can have many data.frames inside a list, or you can mix a vector, a data.frame, any other type of array, other lists, and so on. Example:

```r
set.seed(10)
our_list <- list(runif(10), matrix(1:10, 2, 5), data.frame(letters = letters, 
    order = 1:26))
class(our_list)
```

```
## [1] "list"
```

```r
str(our_list)
```

```
## List of 3
##  $ : num [1:10] 0.5075 0.3068 0.4269 0.6931 0.0851 ...
##  $ : int [1:2, 1:5] 1 2 3 4 5 6 7 8 9 10
##  $ :'data.frame':	26 obs. of  2 variables:
##   ..$ letters: Factor w/ 26 levels "a","b","c","d",..: 1 2 3 4 5 6 7 8 9 10 ...
##   ..$ order  : int [1:26] 1 2 3 4 5 6 7 8 9 10 ...
```

```r
dim(our_list)  #lists do not have dimensions as arrays, thay are measured by their number of elements
```

```
## NULL
```

```r
length(our_list)
```

```
## [1] 3
```

```r
names(our_list)  #create names for our list
```

```
## NULL
```

```r
names(our_list) <- c("random_numbers", "a_matrix", "a_dataframe")
our_list
```

```
## $random_numbers
##  [1] 0.50748 0.30677 0.42691 0.69310 0.08514 0.22544 0.27453 0.27231
##  [9] 0.61583 0.42967
## 
## $a_matrix
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1    3    5    7    9
## [2,]    2    4    6    8   10
## 
## $a_dataframe
##    letters order
## 1        a     1
## 2        b     2
## 3        c     3
## 4        d     4
## 5        e     5
## 6        f     6
## 7        g     7
## 8        h     8
## 9        i     9
## 10       j    10
## 11       k    11
## 12       l    12
## 13       m    13
## 14       n    14
## 15       o    15
## 16       p    16
## 17       q    17
## 18       r    18
## 19       s    19
## 20       t    20
## 21       u    21
## 22       v    22
## 23       w    23
## 24       x    24
## 25       y    25
## 26       z    26
```


#### If we want to extract an element from a list, we use double squared brackets [[]]. 


```r
our_list[[1]]
```

```
##  [1] 0.50748 0.30677 0.42691 0.69310 0.08514 0.22544 0.27453 0.27231
##  [9] 0.61583 0.42967
```

```r
our_list[["a_matrix"]]
```

```
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1    3    5    7    9
## [2,]    2    4    6    8   10
```

```r
class(our_list[[1]])
```

```
## [1] "numeric"
```

```r
dim(our_list[[2]])
```

```
## [1] 2 5
```


#### Now that we understand the basics of a list, we can use use the function `lapply` from the `apply` family, which is a very powerful tool to loop over vectors and lists. Exmaple:

```r
list_gcs <- list(gene_a = runif(3), gene_b = runif(3), gene_c = runif(3))
means <- lapply(list_gcs, mean)
class(means)
```

```
## [1] "list"
```

```r
names(means)
```

```
## [1] "gene_a" "gene_b" "gene_c"
```

```r
means[[1]]  #or means[['gene_a']], for example
```

```
## [1] 0.4443
```

```r
means
```

```
## $gene_a
## [1] 0.4443
## 
## $gene_b
## [1] 0.4609
## 
## $gene_c
## [1] 0.2383
```


#### Using `our_list`, we can also use lapply to quickly explore the classes of our different objects within it.

```r
lapply(our_list, class)
```

```
## $random_numbers
## [1] "numeric"
## 
## $a_matrix
## [1] "matrix"
## 
## $a_dataframe
## [1] "data.frame"
```


### Notice that lapply returns a list as its default behavior. R offers a second function that simplifies the output whenever possible. It's called `sapply`, which stands for simplify `lapply`. If your input is a mtrix, the output will also be coerced into a matrix, if your input is a vector, R will try to coerce the output into a vector as well.

```r
sap <- sapply(our_list, class)
sap
```

```
## random_numbers       a_matrix    a_dataframe 
##      "numeric"       "matrix"   "data.frame"
```

```r
class(sap)
```

```
## [1] "character"
```

