Using GIT with GitHub
======================

Today we will be working with one of the most popular version control systems called [GIT][id_git].
Like [R][id_r], [GIT][id_git] is free and open source, decreasing the boundaries for global collaborations.
We'll be operating [GIT][id_git] from Terminal and using [GitHub][id_github] as our remote online repository.

### GIT works in four main layers:
<img src="figure/unnamed-chunk-1.png" title="plot of chunk unnamed-chunk-1" alt="plot of chunk unnamed-chunk-1" width="504px" height="504px" />


### Step 1: create [GitHub][id_github] account

### Step 2: set up global user configurations on Terminal
* `git config --global user.name "your desired username"`
* `git config --global user.email "your email (academic has advantages)"`

### Step 3: create a new local repo in shell dir:

```
$ git init
```


### Step 4: check status of repo:

```
$ git status
```


### Step 5: create a new file README.md and add it (stage) to the local repo:

```
$ git add filename

#conversely you can add all files (and deletions) by typing
$ git add --all
```


### Step 6: recheck status

### Step 7: ready to commit the change? 

```
$ git commit -m 'your message here'
```


### Step 8: check commit's history:

```
$ git log
```


### Step 9: we want you to gather in pairs, one student will be *student A*, and the other *student B*. Let's collaborate with [GitHub][id_github]:
* student A:
    * create repo on GitHub
    * copy and paste last two lines of code from [GitHub][id_github] to add remote/push
    * look at repo page on [GitHub][id_github]
    * add person B as a collaborator (settings > collaborators)
* student B:
    * clone person A's repo
    * make a change
    * add, commit, push
    * person A:
    * pull

[id_git]: http://git-scm.com/
[id_r]: cran.r-project.org/
[id_github]: https://github.com/
