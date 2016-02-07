---
layout: post
title:  "Data Jam with Lastfm!"
date:   2016-02-07 21:53:25
categories: data
tags: python,elasticsearch,d3,data,music
image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.jpg
---


For a music lover, discovering new sources of music is a problem being solved by a lot of recommendation systems . With the proliferation of music content online , the genuine problem is , there is A LOT of music.  Yes I can make a "starred" playlist on a client but lets face it , we've all been too lazy to do that (atleast I was) .

What I instead would have done, was to play the song over and over again . The bad news to this , I can't remember all the tracks that I liked maybe 2 years back .


The good news, I signed up for lastfm's scrobbling service around 5 years back . These kind people exposed their [API](http://www.last.fm/api) and boom , data pushed onto elasticsearch ([the project README](https://github.com/samarthmed/datajam) on how to do that) .


## Thoughts:

I chose the hours of the day to start analyzing my listening history . Also I divided this into two time periods , the college  (2011 to 2013) and the work  (2013 - ).


To start with , I wanted to make a [ histogram](http://bl.ocks.org/mbostock/3048450)  spread over each hour of the day . On top of this , I wanted to know what was the top artist or the least artist that I listened to ? Does this artist pattern change during the day and night ?

I wanted to make sure if the hour of the day and the artist are really correlated . It makes sense that this holds true (at least in my case) when i switch to progressive / rock / blues in the day and switch to classical / slow blues / jazz  in the night.

To achieve this , I had to tweak the response of elasticsearch since the daterange aggregation doesn't support aggregating over the hours of the day . But with python's list comprehensions and inbuilt sorted functions , I could mimic the result exactly. The API now supports the size and the min\_doc\_count too as suggested by elasticsearch .

## Plotting it out :

I gave a few UI controls such as the date range filters , support the minimum document count (basically return artists that have atleast x data points x <- the count of the artist that I listened to in that hour) and limit over the result .

The order option controls the top artist or the least listened to artists ( this is non trivial ). My supposition to this is that the top listened to artists is what you take with you over the years but less listened to remain trapped in a chip somewhere (allying with the fact that this data was found on last.fm) .

# College :

My personal favorites at the peak midnight range from Dylan , Norah Jones , Cat Stevens . This changes in the day time with the top trending artists saying Led Zep , ZZ Top, Hendrix . The data atleast validates that the hours and the artists are dependent.


Another interesting pattern to observe here is the hours that I listened to music were spread out more between 11 and 4  doing the things that you do in college . The rest of the hours are seemingly flattened out because of the classes or what have you . The high peaks at 3 and 4 are also suggestive of the fact that I used to put on switch a playlist and sleep.


![College Peaks Top Artists]({{ site.url }}/assets/article_images/2016-02-07-data-jam-lastfm/College-peak.png)

![College Lows Top Artists]({{ site.url }}/assets/article_images/2016-02-07-data-jam-lastfm/College-low.png)

Also , when I reverse the order of the aggregations , the non trivial part starts to show some results . I discovered a song called Dice by Finley Quaye that I still like . I did not even remember how much I enjoyed Coleman Hawkins all over again .



# Work :
The data shows that the hours are more peaked between  9 and 6  . The frequency drops between 6 and 10 at night which says that I don't push data while I drive a car or while taking the metro (cheaping out on my data pack) and definitely can't listen to music when on the bike .

Also, I get an entirely different result suggesting that my choice of music has changed over the years.

For the top artists in the peak hours , the data suggests that I lie more into extremes . I either am listening to hard rock (ACDC / Malmsteen) or listening to classical stuff (Olafur Arnalds, Brain Crain) . I think this suggests spent coding at work (this kind helps me focus) .

![Work Afternoon Top  Artists]({{ site.url }}/assets/article_images/2016-02-07-data-jam-lastfm/Work-afternoon-top.png)

This data set changes at night with the top artists being more into classical stuff such as Einaudi, Goldmund or OST's (Han zimmer / Zack Hemsey).


# Wishlist :

There still much to do with this data . Some of the ideas at the top of my head are :

- Build an hour based recommendation system.

- Find out which tracks / album do I listen to the most in a loop . Might hint to personal favorites .

- Plot genre patterns during the hour .

- Plotting a lot of Trivia . The lastfm api gives out an mbid for each track , album , artist. MB is a pandoras box when it comes to music data .

Until next time ...