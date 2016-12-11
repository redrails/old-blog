---
title: "Europe's Major Football Leagues Are Getting Less Competitive"
excerpt: "Using a combination of graph theory and conventional statistics, this post explorers whether Europe's major football (soccer) leagues are becomming less competitive."
header:
  overlay_image: football-overlay.jpg
  overlay_filter: 0.4
  caption: ""
categories:
  - data science
  - football
tags:
  - football
  - soccer
  - epl
  - la liga
  - visNetwork
  - igraph
  - R
author: "David Sheehan"
date: "12 December 2016"
---

{% include base_path %}

### Background

This work was somewhat motivated by a post I read on [another interesting data science blog](https://longhowlam.wordpress.com/2016/09/12/some-insights-in-soccer-transfers-using-market-basket-analysis/); its combination of network graphs and football seemed both accessible and visualing appealing. Due to the profileration of social media and technological advances, [graph/network based approaches are becoming more common](https://blogs.thomsonreuters.com/answerson/future-graph-shaped/). Graph theory has been employed to study [disease propagation](http://journal.frontiersin.org/article/10.3389/fphy.2015.00071/full), [elephant rest sites](http://onlinelibrary.wiley.com/doi/10.1111/ecog.02379/full), [relationships in The Simpsons](http://thesimpsonsuniverse.weebly.com/network.html) and even [MMA finishes](http://www.fightprior.com/2016/09/29/finishCooccurrence/), so I wanted to try it out for myself.

I watch alot of English Premier League (EPL) football, so I'm actutely aware of its reputation as the [Most Competitive league in the world](http://www.telegraph.co.uk/sport/football/competitions/premier-league/11896600/Is-this-Premier-League-season-the-most-competitive-ever.html), (formerly, [the Best League in the World](https://www.theguardian.com/football/picture/2016/oct/18/david-squires-on-the-return-of-the-best-league-in-the-world)). I'm not aiming to compare the quality of each league ([UEFA coefficients solves that problem](https://en.wikipedia.org/wiki/UEFA_coefficient#Current_ranking)), but rather determine whether the leagues themselves are becoming less competitive. This decade has seen the rise of foreign owned super rich clubs across Europe (Man City, PSG) and the domination of domestic championships by a small elite (Bayern Munich in Germany, Juventus in Italy). Then again, just last year, [relegation favourites Leicester City won the EPL](https://www.theguardian.com/football/2016/may/03/5000-1-outsider-leicester-city-bookmakers), so maybe the EPL has become more competitive than ever.

I suppose we need to quantify the competitiveness of a league. We'll use two approaches: one based on graph theory and another more conventional statistical approach. I'm not particularly expecting the former to beat the latter, I just wanted an excuse to build a network graph populated with football teams.

### Gathering the Data

There are numerous free sources of football data (well, at least for the major European leagues- you might struggle with the Slovakian Third Division or the Irish Premier Division). There's a good summary [here](https://www.jokecamp.com/blog/guide-to-football-and-soccer-data-and-apis/). And if you're interested in R API wrappers, there's the [footballR package](https://github.com/dashee87/footballR). As we want to look at historical trends within leagues, we'll choose the csv route (APIs generally go back only a few years). The data will be sourced from [this site](http://www.football-data.co.uk/data.php). No need to download the files, we can import the data directly into R using the appropriate URL. Let's start with the last year of Alex Ferguson's reign as Man United manager (2012-13 EPL season).

``` r
#loading the packages we'll need
require(RCurl) # import csv from URL
require(dplyr) # data manipulation/filtering
require(visNetwork) # producing interactive graphs
require(igraph) # to calculate graph properties
require(ggplot2) # vanilla graphs
require(purrr) # map lists to functions

options(stringsAsFactors = FALSE)
epl_1213 <- read.csv(text = getURL("http://www.football-data.co.uk/mmz4281/1213/E0.csv"), 
                     stringsAsFactors = FALSE)
head(epl_1213)
```

    ##   Div     Date  HomeTeam   AwayTeam FTHG FTAG FTR HTHG HTAG HTR    Referee
    ## 1  E0 18/08/12   Arsenal Sunderland    0    0   D    0    0   D      C Foy
    ## 2  E0 18/08/12    Fulham    Norwich    5    0   H    2    0   H   M Oliver
    ## 3  E0 18/08/12 Newcastle  Tottenham    2    1   H    0    0   D M Atkinson
    ## 4  E0 18/08/12       QPR    Swansea    0    5   A    0    1   A  L Probert
    ## 5  E0 18/08/12   Reading      Stoke    1    1   D    0    1   A   K Friend
    ## 6  E0 18/08/12 West Brom  Liverpool    3    0   H    1    0   H     P Dowd
    ##   HS AS HST AST HF AF HC AC HY AY HR AR B365H B365D B365A  BWH BWD  BWA
    ## 1 14  3   4   2 12  8  7  0  0  0  0  0  1.40  4.50  8.50 1.35 4.6 9.00
    ## 2 11  4   9   2 12 11  6  3  0  0  0  0  1.80  3.60  4.50 1.80 3.5 4.40
    ## 3  6 12   4   6 12  8  3  5  2  2  0  0  2.50  3.40  2.75 2.60 3.3 2.75
    ## 4 20 12  11   8 11 14  5  3  2  2  0  0  2.00  3.40  3.80 2.00 3.4 3.60
    ## 5  9  6   3   3  9 14  4  3  2  4  0  1  2.38  3.25  3.10 2.40 3.2 3.10
    ## 6 15 14  10   7 10 11  7  3  1  4  0  1  4.20  3.50  1.91 4.10 3.5 1.85
    ##    GBH GBD  GBA  IWH IWD IWA  LBH LBD  LBA  PSH  PSD  PSA  WHH WHD WHA
    ## 1 1.35 4.6 9.00 1.35 4.5 7.3 1.40 4.5 8.00 1.44 4.76 8.78 1.44 4.0 8.0
    ## 2 1.80 3.5 4.40 1.80 3.4 4.0 1.80 3.5 4.50 1.82 3.81 4.80 1.85 3.3 4.5
    ## 3 2.60 3.3 2.75 2.40 3.2 2.7 2.60 3.3 2.70 2.65 3.47 2.82 2.70 3.0 2.8
    ## 4 2.00 3.4 3.60 2.10 3.3 3.1 2.00 3.4 3.75 2.05 3.53 4.01 2.00 3.3 3.8
    ## 5 2.40 3.2 3.10 2.40 3.2 2.7 2.40 3.3 2.88 2.40 3.37 3.26 2.40 3.3 2.9
    ## 6 4.10 3.5 1.85 3.30 3.3 2.0 3.75 3.5 1.95 4.39 3.59 1.94 3.80 3.3 2.0
    ##    SJH SJD  SJA  VCH  VCD  VCA  BSH  BSD  BSA Bb1X2 BbMxH BbAvH BbMxD
    ## 1 1.36 4.5 9.50 1.44 4.75 8.50 1.40 4.33 8.50    39  1.44  1.40  4.89
    ## 2 1.80 3.6 4.50 1.83 3.75 4.75 1.83 3.50 4.33    39  1.85  1.80  3.82
    ## 3 2.50 3.4 2.80 2.62 3.40 2.75 2.50 3.40 2.70    39  2.70  2.55  3.47
    ## 4 2.10 3.3 3.60 2.00 3.50 4.00 2.00 3.40 3.60    39  2.10  2.01  3.55
    ## 5 2.40 3.2 3.00 2.40 3.30 3.25 2.30 3.30 3.10    39  2.45  2.36  3.40
    ## 6 4.00 3.4 1.91 4.33 3.60 1.95 4.00 3.40 1.91    39  4.38  4.01  3.61
    ##   BbAvD BbMxA BbAvA BbOU BbMx.2.5 BbAv.2.5 BbMx.2.5.1 BbAv.2.5.1 BbAH
    ## 1  4.47  9.50  8.37   33     1.83     1.76       2.15       2.05   23
    ## 2  3.59  4.80  4.44   35     1.95     1.89       2.01       1.92   21
    ## 3  3.32  2.85  2.73   35     2.00     1.88       2.01       1.92   23
    ## 4  3.37  4.20  3.76   33     2.18     2.09       1.81       1.73   22
    ## 5  3.27  3.26  3.03   33     2.28     2.18       1.76       1.67   24
    ## 6  3.44  2.00  1.93   33     2.18     2.08       1.81       1.74   22
    ##   BbAHh BbMxAHH BbAvAHH BbMxAHA BbAvAHA PSCH PSCD PSCA
    ## 1 -1.25    2.02    1.96    1.96    1.91 1.44 4.72 8.71
    ## 2 -0.50    1.83    1.80    2.14    2.09 1.84 3.75 4.75
    ## 3  0.00    1.93    1.88    2.03    1.97 2.83 3.35 2.72
    ## 4 -0.25    1.80    1.75    2.21    2.13 2.00 3.53 4.15
    ## 5 -0.25    2.07    2.01    1.91    1.86 2.47 3.30 3.22
    ## 6  0.50    2.00    1.96    1.97    1.92 4.76 3.74 1.84

For each match in a given season, the data frame includes the score and various other data we can ignore (mostly betting odds). First, we must think about our network. Networks are composed of nodes and edges, where an edge connecting two nodes indicates a relationship. In its simplest form, think of a network of people, where two nodes are joined by an edge if they're friends. We can have either undirected or directed networks. The latter means that there's a direction to the relationship (e.g. following someone on Twitter does imply that they follow you, which contrasts with Facebook friends). We'll keep things simple, so we'll opt for an undirected graph.

  

{% include facebook_network.html %}

{% include twitter_network.html %}
