Introduction to R: Exploring the genes of the human genome
=============================================================

Author: John Blischak

## Description

In this lesson we will learn about the basics of R by inspecting a 
biological dataset. I have created a spreadsheet-like dataset using data on
the human genome from the Ensembl Biomart database. Each row contains the
Ensembl transcript that has the longest coding sequence for a given Ensembl
gene ID. I also removed any genes that were not translated at all in order to
keep the size of the dataset manageable. For the exact details, you can see
`data/create_datasets.R`.

My goal is twofold. First is for you to become comfortable with the basics of
R and get some intuition for how it works. Second I want to provide you the
motivation to actually use R the next time you analyze data instead of using
Excel by showing you how powerful it can be. Please ask a question directly to
me or on the EtherPad if you need more explanation.

**Learning objectives:**
+ Understand the various mode of vectors
+ Inspect objects
+ Filter vectors and data frames
+ Use vectorized functions
+ Make basic plots
+ tapply, table
+ Write R script in place of Excel

## Preparing the workspace

As a first step, let's clear out the working environment and verify that
we are in the correct working directory. We'll use the trick we learned in the
first section, `rm(list = ls())` to ensure that there are no objects in the 
working space that could potentially interfere with our analysis if we 
accidentally referred to them. The other option would be to go to the `Session`
tab and click the option `Restart R`.


```r
rm(list = ls())
getwd()
```

```
## [1] "/home/johnubuntu/repos/2013-10-03-pasteur/lessons/pasteur-bc/introduction_to_R"
```


### Exercise: Set the working directory


```r
# Your code here

```


## Import and inspect the dataset

Instead of opening the file in Excel, we use R's `read.table` function. 


```r
genes <- read.table("data/ensembl_human_genes.txt", header = TRUE, sep = "\t", 
    quote = "", stringsAsFactors = FALSE)
head(genes)
```

```
##   ensembl_transcript_id ensembl_gene_id chromosome_name transcript_start
## 1       ENST00000373020 ENSG00000000003               X         99883667
## 2       ENST00000373031 ENSG00000000005               X         99839799
## 3       ENST00000367770 ENSG00000000457               1        169822215
## 4       ENST00000286031 ENSG00000000460               1        169764550
## 5       ENST00000374003 ENSG00000000938               1         27939180
## 6       ENST00000367429 ENSG00000000971               1        196621008
##   transcript_end external_gene_id percentage_gc_content   gene_biotype
## 1       99891803           TSPAN6                 40.87 protein_coding
## 2       99854882             TNMD                 40.80 protein_coding
## 3      169858029            SCYL3                 40.34 protein_coding
## 4      169823221         C1orf112                 39.22 protein_coding
## 5       27953080              FGR                 52.97 protein_coding
## 6      196716634              CFH                 35.20 protein_coding
##    source                                      name_1006 cds_length
## 1 ensembl                           integral to membrane        738
## 2 ensembl                           integral to membrane        954
## 3 ensembl                                    ATP binding       2229
## 4 ensembl                                                      2562
## 5 ensembl positive regulation of mast cell degranulation       1590
## 6 ensembl     complement activation, alternative pathway       3696
```


We have provided R the following information to retrieve our dataset:
+ Open the file `data/ensembl_human_genes.txt`
+ The file has a header, so the first row contains the column names
+ The file is tab-separated (`"\t"` stands for "tab")
+ Ignore all quotation marks (" and ')
+ Do not convert strings to factors (explanined later in this lesson)

The function `head` gave us a preview of the dataset, which does look very
much like a spreadsheet. There are column names like `ensembl_transcript_id`
and `chromosome_name`, and there are row numbers for each gene entry. 

But how is R representing the data? Let's use some R function to further
inspect this dataset. As we explore what we have imported, we'll learn about 
the different data types in R.


```r
class(genes)
```

```
## [1] "data.frame"
```

```r
dim(genes)
```

```
## [1] 21056    11
```

```r
str(genes)
```

```
## 'data.frame':	21056 obs. of  11 variables:
##  $ ensembl_transcript_id: chr  "ENST00000373020" "ENST00000373031" "ENST00000367770" "ENST00000286031" ...
##  $ ensembl_gene_id      : chr  "ENSG00000000003" "ENSG00000000005" "ENSG00000000457" "ENSG00000000460" ...
##  $ chromosome_name      : chr  "X" "X" "1" "1" ...
##  $ transcript_start     : int  99883667 99839799 169822215 169764550 27939180 196621008 143816614 53362139 41040684 24683527 ...
##  $ transcript_end       : int  99891803 99854882 169858029 169823221 27953080 196716634 143832827 53409927 41067715 24740230 ...
##  $ external_gene_id     : chr  "TSPAN6" "TNMD" "SCYL3" "C1orf112" ...
##  $ percentage_gc_content: num  40.9 40.8 40.3 39.2 53 ...
##  $ gene_biotype         : chr  "protein_coding" "protein_coding" "protein_coding" "protein_coding" ...
##  $ source               : chr  "ensembl" "ensembl" "ensembl" "ensembl" ...
##  $ name_1006            : chr  "integral to membrane" "integral to membrane" "ATP binding" "" ...
##  $ cds_length           : int  738 954 2229 2562 1590 3696 1404 1914 1044 1005 ...
```



Thus, the data is in a data.frame with 21,056 rows and 11 columns. The 11
columns each contain a different type of information, which is summarized by
the function `str`, which shows the **str**ucture of an object. `str` is an
amazing tool for both beginner and advanced programmers (it was voted the
most useful R trick on a [StackOverflow post](http://stackoverflow.com/questions/1295955/what-is-the-most-useful-r-trick)),
so use it often! For our data.frame `genes`, it explains the data in each
column. The individual columns are each a specific type of vector.

## The vector

One big difference between R and other programming languages you may be 
familiar with like C++, Java, Python, Perl, etc. is that there are no scalar
data types. In other words, the base object is a vector. The code below 
demonstrates that even though a variable may look like a scalar, it is 
actually a vector with a length of one. We will see the ramifications of this
soon.


```r
x <- 3
is.vector(x)
```

```
## [1] TRUE
```

```r
length(x)
```

```
## [1] 1
```


A discussion of data types can be an extremely advanced topic. As a 
pragmatic consideration, I will present a **simplified** view of the main types
of vectors in R. Below I will describe four types of vectors:
+ numeric
+ character
+ factor
+ logical

## Numeric vector

A numeric vector in our dataset is `percentage_gc_content`, which as the name
implies is the percentage of G and C bases in the transcript sequence. As the 
output from `str(genes)` hinted, we can access the columns of a data.frame
using the dollar sign character, `$`. Let's learn more about what it contains:


```r
is.numeric(genes$percentage_gc_content)
```

```
## [1] TRUE
```

```r
head(genes$percentage_gc_content)
```

```
## [1] 40.87 40.80 40.34 39.22 52.97 35.20
```

```r
summary(genes$percentage_gc_content)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    25.0    40.6    46.0    47.1    52.8    83.7
```


Note that R has designated some vectors as int for integers. In our simplified
view, we are not going to worry about this distintion because R considers
them to be numeric as well, e.g.:


```r
is.numeric(genes$cds_length)
```

```
## [1] TRUE
```


Since R is designed for exploratory data analysis, it is very simple to 
visualize data.


```r
plot(genes$percentage_gc_content)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 


The graph generated by `plot` is not very informative in this case since the
x-axis is simply the order of the data in the vector, i.e. the index, and
also the plot is extremely over-plotted. 


```r
hist(genes$percentage_gc_content)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 


On the other hand, the histogram created by `hist` provides a nice
visualization of the information returned by the `summary` function.

R has the typical mathematical functions you would expect:
+ `+`, `-`  addition, subtraction
+ `*`, `/`  multiplication, division
+ `^`  exponentiation
+ `%/%`  integer division
+ `%%`  modulo (returns the remainder of a division operation)

But first, let's create some toy vectors that are more manageable to work with.
There are multiple ways to create a numeric vector. 


```r
(num_vec_one <- c(1, 5, 8, 3))
```

```
## [1] 1 5 8 3
```

```r
(num_vec_two <- 1:8)
```

```
## [1] 1 2 3 4 5 6 7 8
```

```r
(num_vec_three <- seq(from = 3, to = 12, by = 3))
```

```
## [1]  3  6  9 12
```


**Note**: I used a trick above. Normally for an assignment, R generates no
output. In order to see what is contained in the variable as it is assigned,
you can surround the expression with parentheses.

In other programming languages, in order to do something simple like add one to
each number of a vector, you would have to use some sort of looping
mechanism to add each number separately. However, because R is vector-based
this is extremely simple.


```r
num_vec_one + 1
```

```
## [1] 2 6 9 4
```

```r
num_vec_two/num_vec_three
```

```
## [1] 0.3333 0.3333 0.3333 0.3333 1.6667 1.0000 0.7778 0.6667
```

```r
num_vec_three/num_vec_three
```

```
## [1] 1 1 1 1
```

```r
c(0, 0, 0, 0) + 1:3  # alternatively could have used rep(0, 4)
```

```
## Warning: longer object length is not a multiple of shorter object length
```

```
## [1] 1 2 3 1
```


The examples above demonstrate the comcept of **recycling**. The first element
of the first vector is paired with the first element of the second, and then
the second elements of each vector, and so on. If one vector is shorter than
than the other, it starts again at the beginning of the vector. As we saw in 
the last example, if the shorter vector is not a multiple of the longer 
vector, R issues a warning since it is likely a mistake.

As you may have deduced, many R functions are "vectorized."


```r
sqrt(c(2, 4, 9, 16))
```

```
## [1] 1.414 2.000 3.000 4.000
```

```r
abs(c(-4, 6, -987))
```

```
## [1]   4   6 987
```


Utilizing vectorized functions will not only make your code easier to
understand, but it also will speed up your code. This is because many
base R functions are actually written in C, so one function call to a C
function is much faster than multiple calls to C and/or R functions (see
this blog [post](http://yihui.name/en/2010/10/on-the-gory-loops-in-r/)) for
details).

Lastly, you may have noticed that there is always a "[1]" next to the output.
This tells us the index of the first element on the line. It is useful when
the output spans more than one line, e.g.


```r
1:100
```

```
##   [1]   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17
##  [18]  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34
##  [35]  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51
##  [52]  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68
##  [69]  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85
##  [86]  86  87  88  89  90  91  92  93  94  95  96  97  98  99 100
```


This also implies that R indexing starts at 1, instead of at 0 like most other
programming languages. We can use this index information to extract specific
elements of a vector.


```r
num_vec_three
```

```
## [1]  3  6  9 12
```

```r
num_vec_three[2:4]
```

```
## [1]  6  9 12
```

```r
num_vec_three[c(1, 4)]
```

```
## [1]  3 12
```

```r
genes$external_gene_id[c(1, 4, 15)]
```

```
## [1] "TSPAN6"   "C1orf112" "CFTR"
```


**Exercise**

The column `cds_length` contains the number of base pairs that constitute
the protein-coding portion of the gene. Thus, we would expect each entry to
be a multiple of three base pairs. Check to see if our assumption is
correct.


```r
# your code here

```


## character vectors

The other main mode of vector that we observe in the `genes` data.frame is
character. Character vectors contain non-numeric data, often referred to as
"strings."

**Question:** Why is `chromosome_name` a character vector and not a numeric
vector?

Create character vectors using `c`. As expected, the functions for strings
are also vectorized.


```r
dna_seqs <- c("AGTCTATGCTAGC", "ACT", "ATCGTCTCTCGGCTGGCGGCAA")
dna_len <- nchar(dna_seqs)  # counts the number of characters in each string
dna_len
```

```
## [1] 13  3 22
```


One very useful string function is `paste`. It pastes together the vectors it
is supplied, recycling shorter vectors if necessary. Below we convert the
chromosomes to the necessary format to make a BED file.


```r
chroms <- c(1:23, "X", "Y", "MT")  # the numbers are converted to characters
# Add the prefix 'chr'
paste("chr", chroms, sep = "")  # alternatively could use paste0
```

```
##  [1] "chr1"  "chr2"  "chr3"  "chr4"  "chr5"  "chr6"  "chr7"  "chr8" 
##  [9] "chr9"  "chr10" "chr11" "chr12" "chr13" "chr14" "chr15" "chr16"
## [17] "chr17" "chr18" "chr19" "chr20" "chr21" "chr22" "chr23" "chrX" 
## [25] "chrY"  "chrMT"
```


`substr` extracts a substring of each element. Below we extract the last six
elements of each string (all the gene ID's start with ENSG00000) to create
shorter unique identifiers.


```r
ensembl_gene_id_short <- substr(genes$ensembl_gene_id, 10, 15)
head(ensembl_gene_id_short)
```

```
## [1] "000003" "000005" "000457" "000460" "000938" "000971"
```


`gsub` replaces a specific pattern in each string. 


```r
(biotypes <- unique(genes$gene_biotype))
```

```
##  [1] "protein_coding"         "polymorphic_pseudogene"
##  [3] "IG_V_gene"              "TR_V_gene"             
##  [5] "TR_J_gene"              "TR_C_gene"             
##  [7] "TR_D_gene"              "IG_D_gene"             
##  [9] "IG_C_gene"              "IG_J_gene"
```

```r
gsub("_", "-", biotypes)
```

```
##  [1] "protein-coding"         "polymorphic-pseudogene"
##  [3] "IG-V-gene"              "TR-V-gene"             
##  [5] "TR-J-gene"              "TR-C-gene"             
##  [7] "TR-D-gene"              "IG-D-gene"             
##  [9] "IG-C-gene"              "IG-J-gene"
```


`grep` searches for a specific pattern and returns the indexes that contain
that pattern. This is useful for selecting specific subsets of a dataset. Below
we search for the genes whose gene ontology (GO) description includes the
string "mast cell".


```r
head(genes$name_1006)  # the cryptic name Ensembl uses for GO description
```

```
## [1] "integral to membrane"                          
## [2] "integral to membrane"                          
## [3] "ATP binding"                                   
## [4] ""                                              
## [5] "positive regulation of mast cell degranulation"
## [6] "complement activation, alternative pathway"
```

```r
grep("mast cell", genes$name_1006)
```

```
## [1]     5  1934 14726
```


If you find yourself performing lots of `grep` searches, you will want to
invest some time in learning regular expressions. These allow you to create
more complex search patterns. See `?regex` for details.

## factor

Factors represent categorical variables. Unfortunately they are complicated.
They can be very useful at times, especially when
building statistical models or graphing. However, they can behave in very
unexpected ways to produce strange results. This is why when we orginally
read in our dataset, we set the read.table option `stringsAsFactors = FALSE`.
It is best to keep everything as strings, and only convert to a factor when
you need it (and usually R will automatically convert a character vector
to a factor when it makes sense).

Here we convert the character vector `gene_biotype` to a factor. The main 
feature of a factor is the levels, which correspond to the set of categorical
variables. 


```r
genes$gene_biotype <- factor(genes$gene_biotype)
str(genes$gene_biotype)
```

```
##  Factor w/ 10 levels "IG_C_gene","IG_D_gene",..: 6 6 6 6 6 6 6 6 6 6 ...
```

```r
levels(genes$gene_biotype)
```

```
##  [1] "IG_C_gene"              "IG_D_gene"             
##  [3] "IG_J_gene"              "IG_V_gene"             
##  [5] "polymorphic_pseudogene" "protein_coding"        
##  [7] "TR_C_gene"              "TR_D_gene"             
##  [9] "TR_J_gene"              "TR_V_gene"
```

```r
table(genes$gene_biotype)
```

```
## 
##              IG_C_gene              IG_D_gene              IG_J_gene 
##                      9                     28                      6 
##              IG_V_gene polymorphic_pseudogene         protein_coding 
##                     45                     29                  20760 
##              TR_C_gene              TR_D_gene              TR_J_gene 
##                      5                      3                     74 
##              TR_V_gene 
##                     97
```

```r
boxplot(genes$percentage_gc_content ~ genes$gene_biotype)
```

![plot of chunk unnamed-chunk-21](figure/unnamed-chunk-21.png) 


If you wanted to change the order of the x-axis on the boxplot, we can
order the factor.


```r
genes$gene_biotype <- ordered(genes$gene_biotype, levels = c("protein_coding", 
    "polymorphic_pseudogene", "IG_C_gene", "IG_D_gene", "IG_J_gene", "IG_V_gene", 
    "TR_C_gene", "TR_D_gene", "TR_J_gene", "TR_V_gene"))
boxplot(genes$percentage_gc_content ~ genes$gene_biotype)
```

![plot of chunk unnamed-chunk-22](figure/unnamed-chunk-22.png) 


To obtain the median value for the percent GC content for each of the factor
levels that we observe in the boxplot, we can use the function `tapply`.


```r
tapply(genes$percentage_gc_content, genes$gene_biotype, median)
```

```
##         protein_coding polymorphic_pseudogene              IG_C_gene 
##                  45.89                  49.06                  62.99 
##              IG_D_gene              IG_J_gene              IG_V_gene 
##                  45.08                  59.16                  53.36 
##              TR_C_gene              TR_D_gene              TR_J_gene 
##                  43.66                  55.56                  46.55 
##              TR_V_gene 
##                  47.44
```


Here `tapply` takes the first argument, `genes$percentage_gc_content`, and 
splits these values into the categories as specified by the factor
`genes$gene_biotype` provided as the second. Then it applies the third 
argument, the function `median`, to each of the subcategories.

So factors are clearly powerful. But again you'll need to be careful since
they are more complicated than other vectors. In fact, even though in many
ways they are similar to vectors, they aren't even a vector. This is because
factors have the additional attribute `levels`.


```r
is.vector(genes$gene_biotype)
```

```
## [1] FALSE
```

```r
is.factor(genes$gene_biotype)
```

```
## [1] TRUE
```



## logical vectors

We have already seen that some functions return `TRUE` or `FALSE`. In R these
are of mode "logical" (also more generally know as Boolean values). And of 
course all the same vector properties apply to these as well.

relational operator
+ x < y
+ x > y
+ x <= y
+ x >= y
+ x == y
+ x != y

Logical operators
+ ! x  NOT x
+ x & y  x AND y (element-wise)
+ x && y  x AND y (first element only)
+ x | y  x OR y (element-wise)
+ x || y  x OR y (first element only)
+ xor(x, y)  x OR y but not x AND Y (exlusive OR)

Here are some examples:


```r
1:10 < 5
```

```
##  [1]  TRUE  TRUE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
```

```r
!c(TRUE, FALSE, TRUE, TRUE)
```

```
## [1] FALSE  TRUE FALSE FALSE
```

```r
c(TRUE, TRUE, FALSE) & c(TRUE, FALSE, FALSE)
```

```
## [1]  TRUE FALSE FALSE
```

```r
c(TRUE, TRUE, FALSE) | c(TRUE, FALSE, FALSE)
```

```
## [1]  TRUE  TRUE FALSE
```

```r
xor(c(TRUE, TRUE, FALSE), c(TRUE, FALSE, FALSE))
```

```
## [1] FALSE  TRUE FALSE
```

```r
c(TRUE, TRUE, TRUE) && c(TRUE, FALSE, TRUE) && c(FALSE, FALSE, TRUE)
```

```
## [1] FALSE
```


**Best practice:** `TRUE` and `FALSE` can also be abbreviated to `T` and `F`,
respectively. However, you should avoid this. `TRUE` and `FALSE` cannot be
used as variables names, but `T` and `F` can. This could lead to strange errors.


```r
T == TRUE
```

```
## [1] TRUE
```

```r
T <- "abc"
T == TRUE
```

```
## [1] FALSE
```

```r
# TRUE <- 'abc' # throws an error
rm(T)
```


Another useful aspect of logical vectors is that they can be used for counting.
This is because `TRUE` is synonymous with 1, and `FALSE` with 0. 


```r
TRUE == 1
```

```
## [1] TRUE
```

```r
FALSE == 0
```

```
## [1] TRUE
```

```r
# How many are numbers in my vector are greater than 10?
my_nums <- c(5, 54, 9, 4, 1, 84, 47, 35, 18)
my_nums > 10
```

```
## [1] FALSE  TRUE FALSE FALSE FALSE  TRUE  TRUE  TRUE  TRUE
```

```r
sum(my_nums > 10)
```

```
## [1] 5
```

```r
# How many are greater than 10 but less than 40?
sum(my_nums > 10 & my_nums < 40)
```

```
## [1] 2
```


Logical vectors are also great for indexing. We already saw how we can select
specific subsets of data by referring to the index numbers. However, we can
also select elements of a list with a logical vector: elements in the vector
that match up with TRUE in the logical vector are selected.


```r
x <- 1:10
(divisible_by_2 <- x%%2 == 0)
```

```
##  [1] FALSE  TRUE FALSE  TRUE FALSE  TRUE FALSE  TRUE FALSE  TRUE
```

```r
x[divisible_by_2]
```

```
## [1]  2  4  6  8 10
```

```r
# This can also be done in just one line of code
x[x%%2 == 0]
```

```
## [1]  2  4  6  8 10
```


## Converting vector modes

The various vector modes can be converted to other modes.


```r
as.character(-5:5)
```

```
##  [1] "-5" "-4" "-3" "-2" "-1" "0"  "1"  "2"  "3"  "4"  "5"
```

```r
as.character(c(TRUE, T, F, FALSE))
```

```
## [1] "TRUE"  "TRUE"  "FALSE" "FALSE"
```

```r
as.numeric(c("1", "2", "3"))
```

```
## [1] 1 2 3
```

```r
as.numeric(c(TRUE, T, F, FALSE))
```

```
## [1] 1 1 0 0
```

```r
as.logical(c("TRUE", "T", "F", "FALSE"))
```

```
## [1]  TRUE  TRUE FALSE FALSE
```

```r
as.logical(-10:10)
```

```
##  [1]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE FALSE
## [12]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
```


But be careful! Especially when using factors. See below for an error that can
be disatrous since R will not even give a warning.


```r
chroms <- c(1, 5, 3, 1, "X")
chroms_fac <- as.factor(chroms)
as.numeric(chroms_fac)
```

```
## [1] 1 3 2 1 4
```

```r
str(chroms_fac)
```

```
##  Factor w/ 4 levels "1","3","5","X": 1 3 2 1 4
```


If you are sure it is supposed to be numeric, first convert it to a character
vector before converting to a numeric vector.


```r
numeric_fac <- factor(c(87, 3, 1, 7, 25, 3, 87))
as.numeric(as.character(numeric_fac))
```

```
## [1] 87  3  1  7 25  3 87
```


### Exercise: Saving a plot

Recall the histogram we made earlier of the distribution of percent GC content
for human genes, `hist(genes$percentage_gc_content)`. There are lots of
options available to customize the appearance of the plot. See `?hist` for all
of the available options. Here are some ideas to get you started:
+ Change the label of the x-axis
+ Change the title
+ Color the bars of the histogram (R accepts normal names like "red" and "blue")
+ Change the number of breaks


```r
# png('figs/distribution-gc-content.png')

# Customize the histogram here
hist(genes$percentage_gc_content)
```

![plot of chunk unnamed-chunk-32](figure/unnamed-chunk-32.png) 

```r

# dev.off()
```


After you have customized the plot, uncomment the first and last lines of code
above. Before you were always sending the plot to the plotting area of
RStudio. The function `png` directs the plot to be saved in the given filename.
The last line, `dev.off`, tells R to stop sending information to the external
file. 

**Extra challenge:** Use the function `abline` to add a vertical line at
50% GC content. Then add another vertical line at the median percentage GC
content (the vertical lines should be different colors). Re-run the code and
confirm that the file `figs/distribution-gc-content.png` was updated.

## Indexing a Data frame

We've already seen that we can extract a specific column of a data.frame using
the dollar sign character, `$`. Additionally we can extract specific rows of
that column using square brackets, `[]`. But a data.frame can also be indexed
specifically using its 2D structure. Again the square brackets are used, but 
this time there are two arguments separated by a comma. The first vector
specifies which rows to select, and the second vector specifies the columns.


```r
genes[158:165, 4:6]
```

```
##     transcript_start transcript_end external_gene_id
## 158        133634113      133702765            CDKL3
## 159        158958382      158992478             UPP2
## 160          2867228        2871720           PRSS21
## 161         45754516       45806951            MARK4
## 162         15969849       16077741            PROM1
## 163         18043825       18054746          CCDC124
## 164         42082601       42092916         CEACAM21
## 165         26083792       26127525             NOS2
```


Also similar to vectors, we can choose rows and columns with logical vectors.
A row/column that matches `TRUE` in the logical vector is kept and one that
matches `FALSE` is removed.


```r
# Only keep every other column
genes[1:10, c(TRUE, FALSE)]
```

```
##    ensembl_transcript_id chromosome_name transcript_end
## 1        ENST00000373020               X       99891803
## 2        ENST00000373031               X       99854882
## 3        ENST00000367770               1      169858029
## 4        ENST00000286031               1      169823221
## 5        ENST00000374003               1       27953080
## 6        ENST00000367429               1      196716634
## 7        ENST00000002165               6      143832827
## 8        ENST00000229416               6       53409927
## 9        ENST00000341376               6       41067715
## 10       ENST00000337248               1       24740230
##    percentage_gc_content  source cds_length
## 1                  40.87 ensembl        738
## 2                  40.80 ensembl        954
## 3                  40.34 ensembl       2229
## 4                  39.22 ensembl       2562
## 5                  52.97 ensembl       1590
## 6                  35.20 ensembl       3696
## 7                  39.90 ensembl       1404
## 8                  40.16 ensembl       1914
## 9                  40.00 ensembl       1044
## 10                 44.09 ensembl       1005
```


If the row or column argument is omitted, all rows or columns are included,
respectively.


```r
dim(genes[3, ])  # select only the 3rd row, returns a data.frame
```

```
## [1]  1 11
```

```r
length(genes[, 8])  # select only the 8th column, returns a vector
```

```
## [1] 21056
```


We usually want to subset the data based on the values contained in the
data.frame.


```r
# Select only genes with the biotype 'protein_coding'
dim(genes[genes$gene_biotype == "protein_coding", ])
```

```
## [1] 20760    11
```


The argument `genes$gene_biotype == "protein_coding"` creates a logical vector,
thus only rows where this condition is `TRUE` are added to the new variable. All
the columns are copied as well. 

### Exercise: Subsetting

1) Create a new data.frame called `genes_sub` that only contains genes that are
on the X chromosome and are labeled "protein_coding."


```r
# Your code here

```


2) Now make `genes_sub` contain only genes who have a GO category with the
word "development" in it. Recall that the GO column is named "name_1006"
and you can search for words in strings using the function `grep`.


```r
# Your code here

```


**Note:** There is a convenience function called `subset` that makes these
sorts of operations more succint. However, it comes with the warning that it is
"intended for use interactively," so you do not want to become too dependent
on it. See this [StackOverflow post](http://stackoverflow.com/questions/9860090/in-r-why-is-better-than-subset) if you are curious about the dangers of 
subset.

Lastly, you can also subset a data.frame by specifically referring to the
row or column names. Our data.frame `genes` does not have informative row
names (they are the same as the index), but it does have column names.


```r
colnames(genes)
```

```
##  [1] "ensembl_transcript_id" "ensembl_gene_id"      
##  [3] "chromosome_name"       "transcript_start"     
##  [5] "transcript_end"        "external_gene_id"     
##  [7] "percentage_gc_content" "gene_biotype"         
##  [9] "source"                "name_1006"            
## [11] "cds_length"
```

```r
genes[1:10, c("ensembl_transcript_id", "cds_length")]
```

```
##    ensembl_transcript_id cds_length
## 1        ENST00000373020        738
## 2        ENST00000373031        954
## 3        ENST00000367770       2229
## 4        ENST00000286031       2562
## 5        ENST00000374003       1590
## 6        ENST00000367429       3696
## 7        ENST00000002165       1404
## 8        ENST00000229416       1914
## 9        ENST00000341376       1044
## 10       ENST00000337248       1005
```


Referring to the column specifically by its name not only saves you from having
to figure out which column it is, but it allows you to change the order of the
columns of your input data without effecting the code exectution.

## Creating a bed file

Let's create [BED](http://www.genome.ucsc.edu/FAQ/FAQformat.html#format1) file
of our data.frame `genes_sub`, which contains protein-coding genes on the X
chromosome that are involved in any developmental process. This BED file could
then be used for a variety of purposes, e.g. find the number of ChIP-seq reads
that fall within each gene.

First we need to add the prefix "chr" to all of our chromosome names.


```r
genes_sub$chromosome_name <- paste0("chr", genes_sub$chromosome_name)
```


Second, we have to convert from Ensembl coordinates, which are 1-based and 
both start and end coordinats are inclusive, to UCSC coordinates, which are
0-based and the end coordinate is exclusive 
([details](http://genome.ucsc.edu/FAQ/FAQtracks#tracks1)). Thus we need to
subtract one from our starting positions.


```r
genes_sub$transcript_start <- genes_sub$transcript_start - 1
```


Third, we need to select the right columns in the right order.


```r
genes_bed <- genes_sub[, c("chromosome_name", "transcript_start", "transcript_end", 
    "ensembl_gene_id")]
```


And fourth, we write the data to a file.


```r
write.table(genes_bed, "data/x_dev_genes.bed", quote = FALSE, row.names = FALSE, 
    col.names = FALSE)
```



### Exercise

Instead of searching for ChIP-seq reads across the whole gene body, let's 
instead focus just on the transcription start site (TSS), which corresponds to
the `transcript_start` variable*. Modify `genes_bed` so
that the intervals are +/- 1000 bp from the TSS, and then write the output to
a file called `x_dev_genes_tss.bed` in the directory `data`.


```r
# Your code here

```


*This is technically inacurate. The TSS is the `transcript_start` only for the
genes on the plus strand. This would be the transcription end site (TES) for
genes on the minus strand. We haven't yet covered the flow control statements
that you would need to accomplish this task correctly. We can talk between
sessions if you are curious.
