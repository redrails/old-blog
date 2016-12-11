---
title: "Europe's Major Football Leagues Are Getting Less Competitive"
excerpt: "Using a combination of graph theory and conventional statistics, this post explorers whether Europe's major football (soccer) leagues are becomming less competitive."
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
