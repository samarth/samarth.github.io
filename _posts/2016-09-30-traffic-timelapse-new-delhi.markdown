---
layout: post
title:  "Time lapse of the traffic patterns in New Delhi"
date:   2016-09-30 23:53:25
categories: Tech
tags: javascript,phantomjs,google-maps,google,maps,phantomjs,video,time-lapse
image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.jpg
---


The traffic in New Delhi is quite bad . If you're one of the commuters like me who has to go through the pain staking ordeal , you would know what I mean. I am sure many of you have even tried beating the traffic by leaving early from home / work .

But did it actually help ? I know most of the times I failed. Miserably.

So I tried to find out if there was actually a pattern to all this . Here is my attempt.

<iframe width="560" height="315" src="https://www.youtube.com/embed/uAJmWH8DHXU" frameborder="0" allowfullscreen></iframe><br/>





I was inspired to do this when I looked at a video compiled by Yugi Higaki . He captured the time-lapse of the traffic in San Francisco but unfortunately did not share how he managed to do it .

## Tech
<br/>

I started out by experimenting with the Javascript SDK for Google Maps. There is already a tutorial which helps you put a latitude and longitude on a map , define a zoom level etc . Once that's done , apply the traffic overlay layer . You'll be surprised how easy it is to actually plug into your map. [Here , go try it out .](https://developers.google.com/maps/documentation/javascript/examples/layer-traffic)


Once this is done , the only difficult part is to capture the screenshots . In my case , I wanted to make a time-lapse with frames shot at a minutes interval. This would make 1440 screenshots in a single day .


I started experimenting with [html2canvas](https://html2canvas.hertzen.com) but unfortunately if you've ever done an inspect element on your google maps , you would know in your heart that rendering an image by reading tags and css won't work here . I still gave it a try and all it could manage was to give me a grey screen with google maps written at the bottom. Sad.


### Enter Phantomjs

You lighten up as soon as you see " Phantomjs uses webkit , a real layout and a rendering engine . This engine not only renders html and css , but also svg and canvas . ".
I just passed on my "index.html" to the example mentioned on [this page](http://phantomjs.org/screen-capture.html) and it worked for me pretty well.

To put more perspective , I decided to add a timestamp as well on every screenshot that I took . This can be easily done by the page.evaluate which manipulates the DOM and then do a page.render .



    page.evaluate(function() {
	    var date = new Date();
	    date = date.getHours() + ":" + date.getMinutes();
	    document.getElementById("timestamp").innerHTML = date;
    });

    page.render("./snapshots/" + getTimeStamp('_') + ".png", {
	    quality: 50
    });



Wrap this part in a setInterval and you're live . You don't need to worry about the traffic layer refresh since the API by default has the "autoRefresh" option set on the traffic overlay .

Hope it's simple enough to try.


Until next time ....