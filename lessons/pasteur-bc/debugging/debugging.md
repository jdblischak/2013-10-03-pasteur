Debugging
========================================================

#### One of the most common things that happens when you are writing your computer code is to create many functions that interdepend on each other. Sometimes, when you give a certain input, the desired output does not look like what you expected. The best way of avoiding this is to write tests for your functions. Unfortunately we do not have time to cover code testing in our course, but please do check our online lectures on [software testing][swc_test]. For the moment, we will teach a nice way of debugging code when errors or unexpected outputs occur. 

#### Say that we have three functions. The first function breaks a linear dna string into smaller bp fragments given a certain restriction site. The second function takes a certain dna string and return to us the number of bps. The third function depends on the first and second functions - given a dna string, it first breaks it into different fragments following a restriction site and then, for each fragment, return us the number of base pairs.


```r
find_genes <- function(dna_string, restriction_site) {
    genes <- strsplit(dna_string, restriction_site)
    return(genes)
}

count_bp <- function(gene) {
    return(nchar(gene))
}

count_bp_linear_seq <- function(dna_string, ...) {
    genes <- find_genes(dna_string, ...)
    names(genes) <- genes
    return(count_bp(genes))
}
```


#### We can now test if the function `count_bp_linear_seq` works:

```r
bps <- count_bp_linear_seq(dna_string = "cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg", 
    restriction_site = "actgctgtg")
bps
```

```
## cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg 
##                                            45
```


#### Unfortunately the output was not what we expected. We gave one dna string and expected to have different fragments out of it. Since we know that the function `find_genes` is responsible for this part, we can look closer into it inside the function itself. The way we do it, is by using the function `browser()`:

```r
find_genes <- function(dna_string, restriction_site) {
    genes <- strsplit(dna_string, restriction_site)
    browser()
    return(genes)
}

bps <- count_bp_linear_seq(dna_string = "cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg", 
    restriction_site = "ACTGTCGATGT")
```

```
## Called from: find_genes(dna_string, ...)
```

```r
bps
```

```
## cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg 
##                                            45
```


#### `browser()` allows us to navigate within the function environment, explore it's objects in order to figure out what is going on. In our case, we now know that when we give a restriction site that does not exist, the function does not return an error, it rather returns the entire original string. If, for some reason, we do not want this to be the default behavior, we can tell R to stop running the function and returning an error message - that is called defensive programming. 

```r
find_genes <- function(dna_string, restriction_site) {
    genes <- strsplit(dna_string, restriction_site)
    if (genes == dna_string) 
        stop("Restriction site not found in dna_string") else return(genes)
}

bps <- count_bp_linear_seq(dna_string = "cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg", 
    restriction_site = "ACTGTCGATGT")
```

```
## Error: Restriction site not found in dna_string
```


#### Things should be working fine now. Let's try the same string with a different restriction site:

```r
bps <- count_bp_linear_seq(dna_string = "cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg", 
    restriction_site = "atg")
bps
```

```
## c("cgtagctag", "cgtagctgatcgtagtg", "c", "tcgtagtcg") 
##                                                    53
```


### Hummm... still not good. Now we can see that the first function `find_genes` is doing it's job, but somehow the final function is not returning the character count of each fragment. Let's include another `browser()` call inside `count_bp_linear_seq`:

```r
count_bp_linear_seq <- function(dna_string, ...) {
    genes <- find_genes(dna_string, ...)
    names(genes) <- genes
    browser()
    return(count_bp(genes))
}
bps <- count_bp_linear_seq(dna_string = "cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg", 
    restriction_site = "atg")
```

```
## Called from: count_bp_linear_seq(dna_string = "cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg", 
##     restriction_site = "atg")
```


#### It turns out that the returning portion of our last function was not doing what it was supposed to. This happened because of the default behavior of the function `nchar`. In this case, our input `genes` for the function `count_bp` is a list. `nchar` counts the total number oh characters of an entire list regardless the number of elements in it's vector. We can easily modify that by making the `genes` object a vector and not a list:

```r
count_bp_linear_seq <- function(dna_string, ...) {
    genes <- find_genes(dna_string, ...)[[1]]
    names(genes) <- genes
    return(count_bp(genes))
}
bps <- count_bp_linear_seq(dna_string = "cgtagctagatgcgtagctgatcgtagtgatgcatgtcgtagtcg", 
    restriction_site = "atg")
bps
```

```
##         cgtagctag cgtagctgatcgtagtg                 c         tcgtagtcg 
##                 9                17                 1                 9
```


[swc_test]: http://software-carpentry.org/v4/test/index.html
