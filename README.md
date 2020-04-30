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
2009) to transform a `data.frame` to sparse matrix. The package provides
a formula interface similar to model.matrix in R and
Matrix::sparse.model.matrix in the package Matrix. Splitting of
concatenated data, check the help of `test.tag` for explanation of
concatenated data, during the construction of the model matrix.

## When should we use Feature Hashing?

Feature hashing is useful when the user does not know the dimension of
the feature vector. For example, the bag-of-word representation in
document (R Core Team 2019) problem requires scanning entire dataset to
know how many words we have, i.e. the dimension of the feature vector.
Common example of columns that need feature hasing are urls, CAP (the
italian Zip Code), IPs, cities when they are many.

In general, feature hashing is useful in the following environment:

  - Streaming Environment (all the online streaming platform data
    collection)
  - Distributed environment. (data collection in server, under the hood
    metadata as well)

Because it is expensive or impossible to know the real dimension of the
feature vector.

## Getting Started

The `FeatureHashing` is all about biuulding construct
`Matrix::dgCMatrix` and train a model in packages which supports
`Matrix::dgCMatrix` as input. Since our
[wrapper](https://tidymodels.github.io/parsnip/articles/articles/Models.html)
`parnsip`, by Kuhn hosts a number of different model, we can still use
this as pipeline to build our model (aka setting the computational
engine: `set_engine()`).

2 widely-used possibilities in which we can perform the hasing tricka
are:

1.  classic logistic regression w/ `glmnet`
2.  `xgboost`

## An example

let’s load our data, data here are form the ipinyou (Zhang et al. 2014)
dataset. this dataset has many *char* variables that actually can’t
really be considered **a-priori** informative. Those are the ones that
in any regular classroom exercises are dropped.

This is a very interesting dataset, it regards the bidding advestisement
market. In case you don’t know large comporates as well as medium try to
place their own advertisement on poluar platforms. Due to the increasing
number of companies that want to exploit the benefits of online
advertisement and also to the lack of widely extended and performing
channel on the web, companies are *bidding* prices to grab their space.
This market is really huge and capitalized.

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
more they have values. see one of them:

``` r
Ip = ipinyou.test$IP
knitr::kable(head(Ip), "pandoc")
```

| x              |
| :------------- |
| 183.235.61.\*  |
| 113.82.111.\*  |
| 119.136.133.\* |
| 112.90.194.\*  |
| 113.104.225.\* |
| 14.213.115.\*  |

## logistic regression with the `glmnet`

since data are already preapared in the train and test we really do not
need to split.

``` r
## define the model
f = ~ IP + Region + City + AdExchange + Domain +
  URL + AdSlotId + AdSlotWidth + AdSlotHeight +
  AdSlotVisibility + AdSlotFormat + CreativeID +
  Adid + split(UserTag, delim = ",")


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

Once you have define train and test you can perform directly the
logistic:

``` r

cv.g.lr = cv.glmnet(m.train, ipinyou.train$IsClick, family = "binomial")
p.lr = predict(cv.g.lr, m.test, s="lambda.min")
auc.score = auc(roc(p.lr, factor(ipinyou.test$IsClick)))
auc.score
#> [1] 0.5187244
```

## Reference

<div id="refs" class="references">

<div id="ref-R-base">

R Core Team. 2019. *R: A Language and Environment for Statistical
Computing*. Vienna, Austria: R Foundation for Statistical Computing.
<https://www.R-project.org>.

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

Zhang, Weinan, Shuai Yuan, Jun Wang, and Xuehua Shen. 2014. “Real-Time
Bidding Benchmarking with iPinYou Dataset.”
<http://arxiv.org/abs/1407.7073>.

</div>

</div>
