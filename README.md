Feature Hashing
================
Niccolò Salvini

[![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/FeatureHashing)](https://cran.r-project.org/package=FeatureHashing/)
[![rstudio mirror
downloads](https://cranlogs.r-pkg.org/badges/FeatureHashing)](https://github.com/metacran/cranlogs.app)

## Introduction

[Feature hashing](https://en.wikipedia.org/wiki/Feature_hashing), also
called as the hashing trick, is a method to transform features to
vector. Without looking the indices up in an associative array, it
applies a hash function to the features and uses their hash values as
indices directly.

The package FeatureHashing implements the method in (Weinberger et al.
2009) to transform a `data.frame` to a sparse matrix. The package
provides a formula interface similar to `model.matrix()` in R and
`Matrix::sparse.model.matrix()`. You can also be splitting concatenated
data. Say you have a string in a cell and you want to parse it and
convert it into a predictor. The hasing algorythm used is the
*MurmurHash3*, initially developed by
[aappleby](https://github.com/aappleby/smhasher/wiki/MurmurHash3) and
then refined by the Wush Wu, the author of the package.

## When should we use Feature Hashing?

Feature hashing is useful when the user does not know the dimension of
the feature vector. For example, the bag-of-word representation in
document problem requires scanning entire dataset to know how many words
we have, i.e. the dimension of the feature vector. Common example of
columns that need feature hasing are urls, CAP (the italian Zip Code),
IPs, cities when they are many.

In general, feature hashing is useful in the following environment:

  - Streaming Environment (all the online streaming platform data
    collection)
  - Distributed environment. (data collection in server, under the hood
    metadata as well)

Because it is expensive or impossible to know the real dimension of the
feature vector.

## Getting Started

The `FeatureHashing` is all about building construct `Matrix::dgCMatrix`
and train a model in packages which supports `Matrix::dgCMatrix` as
input. Since our
[wrapper](https://tidymodels.github.io/parsnip/articles/articles/Models.html)
`parnsip`, by Kuhn hosts a number of different model, we can still use
this as pipeline to build our model (aka setting the computational
engine: `set_engine()`).

If you are not totally sure about what is a hash function you can check
out these interesting quora answers
[here](https://www.quora.com/Can-you-explain-feature-hashing-in-an-easily-understandable-way)

2 widely-used possibilities in which we can perform the hasing tricka
are:

1.  classic logistic regression w/ `glmnet`
2.  `xgboost`
3.  \``sparklyr`
    [here](https://spark.rstudio.com/reference/1.04/ft_feature_hasher/)
    with their own functions. ft\_\_something

## An example

let’s load our data, data here are form the ipinyou (Zhang et al. 2014a)
dataset. this dataset has many *char* variables that actually can’t
really be considered **a-priori** informative. Those are the ones that
in any regular classroom exercises are dropped.

This is a very interesting dataset, it regards the bidding advestisement
market. In case you don’t know large comporates as well as medium try to
place their own advertisement on poluar platforms. Due to the increasing
number of companies that want to exploit the benefits of online
advertisement and also to the lack of widely extended and performing
channel on the web, companies are *bidding* prices to grab their space.
This market is really huge and capitalized. (Zhang et al. 2014b)

> Emerged in 2009 \[?\], Real-Time Bidding (RTB) has become an important
> new paradigm in display advertising. For example, eMarketer estimates
> a 73% spending growth on RTB in United States during 2013, which
> accounts for 19% of the total spending in display advertising.
> Different from the conventional negotiation or pre-setting a fixed bid
> for each campaign or keyword, RTB enables the advertisers to give a
> bid for every individual impression.

and more… (DSP are the platforms where the magic happens)

> Algorithms employed by DSPs are expected to contribute a much higher
> return-on-investment (ROI) comparing with the traditional channels. It
> is crucial that such algorithms can quickly decide whether and how
> much to bid for a specific impression, given the contextual and
> behaviour data (usually referred to as user segments). This is
> apparently also an engineering challenge considering the billion-level
> bid requests that a DSP could normally see in a day.

All of this is real time also because of a very rapid obsolescence of
the advertisement. All of this takes R\&D departements of large and
medium corporates to develop real time base strategies to grap their
space and keep on advertising without fighting so much. Those strategies
are called RTB (Real Time Bidding) (Wikipedia 2020) because they should
be implemented as fastest as possibile, so speed is a key. Due to the
lack of data this special type of machine learnig application has been
left apart for long time:

> Being an emerging paradigm for display advertising, Real-Time Bidding
> (RTB) drives the focus of the bidding strategy from context to users’
> interest by computing a bid for each impression in real time. The data
> mining work and particularly the bidding strategy development becomes
> crucial in this performance-driven business. However, researchers in
> computational advertising area have been suffering from lack of
> publicly available benchmark datasets, which are essential to compare
> different algorithms and systems. Fortunately, a leading Chinese
> advertising technology company iPinYou decided to release the dataset
> used in its global RTB algorithm competition in 2013. The dataset
> includes logs of ad auctions, bids, impressions, clicks, and final
> conversions.

If you are not satistfies with the RTB you can deepn the concept
[here](https://www.groundai.com/project/real-time-bidding-benchmarking-with-ipinyou-dataset/3)

``` r

data(ipinyou)
glimpse(ipinyou.train)
#> Rows: 2,363
#> Columns: 18
#> $ BidID            <chr> "910b91fb4652c453426dd1eb4ddf48df", "53fb8d7cc385f...
#> $ BiddingPrice     <dbl> 277, 277, 277, 277, 294, 277, 294, 294, 294, 294, ...
#> $ IP               <chr> "121.32.131.*", "58.254.172.*", "113.116.63.*", "1...
#> $ Region           <chr> "216", "216", "216", "216", "216", "216", "216", "...
#> $ City             <chr> "217", "219", "219", "217", "228", "217", "217", "...
#> $ AdExchange       <chr> "2", "2", "2", "2", "1", "2", "1", "1", "1", "1", ...
#> $ Domain           <chr> "9f29c106b1894764ef88815ff2d43e2a", "3cf77b0372208...
#> $ URL              <chr> "b9b27f86cd5409b505b66e27470765af", "e17031b01be76...
#> $ AdSlotId         <chr> "2578891183", "887458722", "631212782", "830703526...
#> $ AdSlotWidth      <chr> "468", "468", "300", "250", "300", "300", "300", "...
#> $ AdSlotHeight     <chr> "60", "60", "250", "250", "250", "250", "100", "25...
#> $ AdSlotVisibility <chr> "OtherView", "FirstView", "OtherView", "OtherView"...
#> $ AdSlotFormat     <chr> "Na", "Na", "Na", "Na", "Fixed", "Na", "Fixed", "F...
#> $ CreativeID       <chr> "7328", "7328", "7323", "7321", "7323", "7323", "7...
#> $ PayingPrice      <dbl> 5, 57, 5, 53, 22, 68, 52, 201, 88, 121, 160, 5, 9,...
#> $ Adid             <chr> "2259", "2259", "2259", "2259", "2259", "2259", "2...
#> $ UserTag          <chr> "10129,10024,13866,10111,10146,10120,10115,10063",...
#> $ IsClick          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, F...
```

many columns are not encoded yet but many of them are are *character*
columns as we have already anticipated. They all seem to have pretty all
unique values. The first direct perception is ‘ehi, i really need to
drop the IP, the URL, the domain, and the user tag, they can be
exploited, I already ahve my ID columns’. The idea bihind this is to
tell the model that the most of the time we use as the id-join columns
are valuable and that it can actually exploit them if we make a
trasnformation of them. Hash function helps up doing that. One more
thing to say is that here Data are really sensitive. The more data
becomes sensitive and the more this type of information come up, the
more they have values. see one of them, say: IP, Domain, AdSlotId:

``` r
Ip = ipinyou.test$IP
domain = ipinyou.test$Domain
adslotid = ipinyou.test$AdSlotId
dt = data.frame(Ip,domain,adslotid)
```

| Ip             | domain                           | adslotid                        |
| :------------- | :------------------------------- | :------------------------------ |
| 183.235.61.\*  | 32891a14f72e4f88451e349ed095ba41 | mm\_26632206\_2690592\_10599267 |
| 113.82.111.\*  | 408045890463bb216b49ca36ef97820  | mm\_11402872\_1272384\_3182279  |
| 119.136.133.\* | fdae6842bdadadac7e1acebac6ca9fd9 | 652625647                       |
| 112.90.194.\*  | c686bfc01abf7688c42989656316a8ea | Fashion\_Width5                 |
| 113.104.225.\* | dd4270481b753dde29898e27c7c03920 | Ent\_F\_Width1                  |
| 14.213.115.\*  | dd4270481b753dde29898e27c7c03920 | Ent\_F\_Width1                  |

## logistic regression with the `glmnet`

since data are already preapared in the train and test we really do not
need to split.

``` r
## define the model
f = ~ IP + Region + City + AdExchange + Domain +
  URL + AdSlotId + AdSlotWidth + AdSlotHeight +
  AdSlotVisibility + AdSlotFormat + CreativeID +
  Adid + split(UserTag, delim = ",") # here there is the splitting property that I was mentioning at te beginning 


m.train = hashed.model.matrix(f, ipinyou.train, 2^16)
m.test = hashed.model.matrix(f, ipinyou.test, 2^16)
```

``` r
head(m.train)
#> 6 x 65536 sparse Matrix of class "dgCMatrix"
#>    [[ suppressing 34 column names '1', '2', '3' ... ]]
#>                                                                                
#> <NA> 1 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . ......
#> <NA> 1 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . ......
#> <NA> 1 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . ......
#> <NA> 1 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . ......
#> <NA> 1 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . ......
#> <NA> 1 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . ......
#> 
#>  .....suppressing 65502 columns in show(); maybe adjust 'options(max.print= *, width = *)'
#>  ..............................
```

this is an S4 dg matrix object composed nrows per ncolumns, this example
can be a little bit confusing due to the amount of data considered.
Anyway the purpose was to demonstarte how to integrate such featuring
functions in a common machine learning problem. For this purpose I
developed one other toy example just to have a taste of what the
`hashed.model.function()` is really doing.

Once you have define train and test you can perform directly the
logistic:

``` r

cv.g.lr = cv.glmnet(m.train, ipinyou.train$IsClick, family = "binomial")
p.lr = predict(cv.g.lr, m.test, s="lambda.min")
auc.score = auc(roc(p.lr, factor(ipinyou.test$IsClick)))
auc.score
#> [1] 0.5187244
```

## One other example

Lets take a look at the `mtcars`. It has 32 obervations, it has rownames
with cars names, so I set them equal to a column called rowname.

``` r
mtcars = mtcars %>% 
  rownames_to_column() %>% 
  tibble()
attach(mtcars)
#> The following object is masked from package:ggplot2:
#> 
#>     mpg
knitr::kable(head(rowname), col.names = 'Rowname', "pandoc")
```

| Rowname           |
| :---------------- |
| Mazda RX4         |
| Mazda RX4 Wag     |
| Datsun 710        |
| Hornet 4 Drive    |
| Hornet Sportabout |
| Valiant           |

so at this point I define the model, I am interested in the rowname
variable and I want to dummify it into 15 different features. So what I
am doing is to take a 32 lenght different *char* vector and converting
it into a 15 dummy columns. Here we are One-Hot dumming (just to point
out that the outcome of the algo will be something that is either 0 if
it not present or 1, NOT mean econded alike)

``` r
model =  ~ rowname
matrice = hashed.model.matrix(model, mtcars, 15)
matrice
#> 32 x 15 sparse Matrix of class "dgCMatrix"
#>    [[ suppressing 15 column names '1', '2', '3' ... ]]
#>                                    
#>  [1,] 1 . . . 1 . . . . . . . . . .
#>  [2,] 1 . . . . . . . . . . . . . 1
#>  [3,] 1 . . . . . . . . . . . . 1 .
#>  [4,] 1 . . . . . . . . . . . 1 . .
#>  [5,] 1 . . . . . 1 . . . . . . . .
#>  [6,] 1 . . . . . . . . . . . 1 . .
#>  [7,] 1 . . . . . . . 1 . . . . . .
#>  [8,] 1 . . . . . . . . 1 . . . . .
#>  [9,] 1 . . . . 1 . . . . . . . . .
#> [10,] 1 . . . . . . . . . . . . . 1
#> [11,] 1 . . 1 . . . . . . . . . . .
#> [12,] 1 . 1 . . . . . . . . . . . .
#> [13,] 2 . . . . . . . . . . . . . .
#> [14,] 1 . 1 . . . . . . . . . . . .
#> [15,] 1 . . 1 . . . . . . . . . . .
#> [16,] 1 . . 1 . . . . . . . . . . .
#> [17,] 1 . . . 1 . . . . . . . . . .
#> [18,] 1 . . . . . . 1 . . . . . . .
#> [19,] 1 . . 1 . . . . . . . . . . .
#> [20,] 1 . . . . . . . . . . . . . 1
#> [21,] 2 . . . . . . . . . . . . . .
#> [22,] 1 . 1 . . . . . . . . . . . .
#> [23,] 1 . . . . . . . . . 1 . . . .
#> [24,] 1 . . . . . . . . . 1 . . . .
#> [25,] 1 . . . . . . . 1 . . . . . .
#> [26,] 1 . . . . . . . . 1 . . . . .
#> [27,] 2 . . . . . . . . . . . . . .
#> [28,] 1 . . . . . . . . . . . . 1 .
#> [29,] 1 . . . . . . . . . . . 1 . .
#> [30,] 1 1 . . . . . . . . . . . . .
#> [31,] 1 . . . . . . . . . . . . 1 .
#> [32,] 1 . . . . . 1 . . . . . . . .
```

## More

Some other interesting suggestions comes from the Kuhn 2019 (Kuhn and
Johnson 2019). When rows are too many, this technique can be really
confusing and lower the model predeictive performance hashing all the
values. The best technique from a statistical perspective is to convert
all the id-alike columns into hashed values and then asses perform some
dimension reducion like PCA to lower dimensionality. When the id-alike
column really can not display any sort of aggregative information
feature it highly recommended to set a number of column based on some
assumption, either graphical and xontexstual. Then group up all the rest
in one additional column called ‘other’. To get the concept imagine to
have some separate trash collection task and you have a unordered
multimaterial trash can with inside different materials. Your job is to
sort these materials and throw them in the right bin. When you get
through some paper (the rowname) you throw it in the paper bin (the new
hashed columns), same job with the plastic in the plastic bell and so on
for the rest. Unfortunately some material have to be thrown away in the
unsorted waste, (the column other). In this way the model takes all the
information coming from all the predictors from all the predictors
without being to much stressed on dimensionality. Next, notice that
there were 2 hashes with no collisions (i.e., have a single 1 in their
column). But several columns exhibit collisions. An example of a
collision is hash at col number 3 which encodes for both row 12 row 14
and row 22. In statistical terms, these two categories are said to be
aliased or confounded, meaning that a parameter estimated from this hash
cannot isolate the effect of either car’s name. to solve this issue you
try to exploit *signed* hashing feature as collision resultion
techinique.

## Signed hash feature

Some hash functions can also be signed, meaning that instead of
producing a binary indicator 0 and 1, for the new feature, they can take
possible values could be -1, 0, or +1. Symmetricity here can be a plus.
For ordered cat variables there are other techinques to exploit but are
not the focus here. So now each column in the *signed* approach is a cat
variable and can assume different values according to which car it is.
There are not contraints for each columns, one column can have 2
collisions and can be resolved with a -1 0 +1 signing approach. The
following can still be binary encoded. Anyway uniformity in hashing is a
really good property, the more they are uniformed the more each column
can really express its potential information. Below you see some reasosn
why this could be ineffective:

  - less aliasing can help isolate the information coming from the
    categorical variable.
  - categories involved in collisions are not related in any meaningful
    way. For example, the cars in row 12 *Merc 450SE* and in row 14
    *Merc 450SLC* are not sharing any tangible characteristics with the
    row 22 *Dodge Challenger.* They are simply put together due to the
    randomicity of hashing collisions.
  - the more abundant value will have a much larger influence on the
    effect of that hashing feature. It is conceivable that a category
    that occurs with great frequency is aliased with one that is rare.
    You should get the concept by seeing rowwise the element that appear
    the most.

We are not aware of any statistically conscious competitor to feature
hashing.

## References

<div id="refs" class="references">

<div id="ref-kuhn2019feature">

Kuhn, M., and K. Johnson. 2019. *Feature Engineering and Selection: A
Practical Approach for Predictive Models*. Chapman & Hall/Crc Data
Science Series. CRC Press.
<https://books.google.it/books?id=q5alDwAAQBAJ>.

</div>

<div id="ref-DBLP:conf/icml/WeinbergerDLSA09">

Weinberger, Kilian Q., Anirban Dasgupta, John Langford, Alexander J.
Smola, and Josh Attenberg. 2009. “Feature Hashing for Large Scale
Multitask Learning.” In *Proceedings of the 26th Annual International
Conference on Machine Learning, ICML 2009, Montreal, Quebec, Canada,
June 14-18, 2009*, edited by Andrea Pohoreckyj Danyluk, Léon Bottou, and
Michael L. Littman, 1113–20. <https://doi.org/10.1145/1553374.1553516>.

</div>

<div id="ref-wiki:Real-time_bidding">

Wikipedia. 2020. “Real-time bidding — Wikipedia, the Free Encyclopedia.”
<http://en.wikipedia.org/w/index.php?title=Real-time%20bidding&oldid=939717913>.

</div>

<div id="ref-zhang2014realtime">

Zhang, Weinan, Shuai Yuan, Jun Wang, and Xuehua Shen. 2014a. “Real-Time
Bidding Benchmarking with iPinYou Dataset.”
<http://arxiv.org/abs/1407.7073>.

</div>

<div id="ref-zhang2014real">

———. 2014b. “Real-Time Bidding Benchmarking with iPinYou Dataset.”
*arXiv Preprint arXiv:1407.7073*.

</div>

</div>
