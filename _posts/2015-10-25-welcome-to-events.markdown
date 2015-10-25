---
layout: post
title:  "Welcome to Events!"
date:   2015-10-25 20:20:25
categories: Tech
tags: node.js,events,javascript
image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.jpg
---


One can think of async environment similar to a clown juggling glass plates in a party making funny faces to keep the kids entertained and simultaneously eyeing all the ladies in the party in the hope that all this isn't a total waste.

Once you get the hang of channeling these events , programming javascript becomes quite fun.

##Lets start ...


Suppose you want to scan 40 lakh rows from a database (I will be considering mysql here for the example) and then take all the rows and maybe make csv out of those . (I understand that there is a mysql command for the same but please bear with me. The idea is that you will be doing something async here).

Lets take the [Felixge Mysql package](https://github.com/felixge/node-mysql ""). Skip quickly over to the section that says `streaming query rows`. The section there is quite self explanatory.

- Register a query
- Register 4 events -> 'result', 'error', 'fields', 'end'.
- Do whatever mojo you want to do in these.

Now lets make our own class which basically does all the heavy duty work of making the csv file.

{% highlight js %}
var util = require('util');
var MYSQL = require('mysql');


function csvConvertor () {
  var self = this;

  EVENTEMITTER.call(self);

  self.mysqlConnection = MYSQL.createConnection({
    host     : 'localhost',
    user     : 'me',
    password : 'secret',
    database : 'my_db'
  });

  self.threshold = 10000;

  self.batch = [];

}

module.exports = csvConvertor;

{% endhighlight %}

I've made a js class called csvConvertor . Whenever a new instance of this class is created , a mysqlConnection is also made available to the instance.

Now , we'll register a query to this connection.


{% highlight js %}
csvConvertor.prototype.registerQuery = function (query) {
  var self = this;

  var query = self.mysqlConnection(query);

  query.on('error', self.handleError.bind(self));
  query.on('field', self.handleFields.bind(self));
  query.on('result', self.handleRows.bind(self));
  query.on('end', self.handleEnd.bind(self));


  self.on('data', self.processData.bind(self));
};

csvConvertor.prototype.handleError = function (error) {
  console.log("Error from mysql " + error.message);
  process.exit(1);
};

csvConvertor.prototype.handleFields = function (fields) {
};

csvConvertor.prototype.handleEnd = function () {
  console.log("Job well done");
  process.exit(0);
};

csvConvertor.prototype.handleRows = function (row) {
  // Some mojo required
};

{% endhighlight %}


The register query function is meant to be only called once. Once this is called , all the events that the mysql package can possibly throw have been bound to individual functions of the csvConvertor class.


Notice that we have used the [bind function ](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind)  while registering all events because if we miss this out , all the csvConvertor functions will get the mysql class reference when we access `this` variable . The bind function ensures that the scope of csvConvertor does not change.


Now when the rows are returned in the handleRows function , we'll maintain an array (self.batch) which keeps on consuming these rows for now. When the threshold reaches, we'll emit an event that processes this bulk of data.

This event is also registered in the registerQuery function where all the event registry happens.


{% highlight js %}

csvConvertor.prototype.handleRows = function (row) {
  // Some mojo required
  var self = this;

  self.batch.push(row);
  
  if (self.batch.length === self.threshold) {
     self.mysqlConnection.pause();
	self.emit('data', self.batch);
  }
};


csvConvertor.prototype.processData = function (batch) {
 /*
   write to csv file here.
 */

 // Clean up for the current batch.

 self.batch = [];
 self.mysqlConnection.resume();

};

{% endhighlight %}



Once the processData is called, it does the async work in a batch size that you've defined . After the async function is done, you can clean up the batch size and resume the mysql stream.

You also have the choice of emitting a resume event from the csvConvertor instance and registering it in the registerQuery function. I've avoided it since it involves only two lines.



I've been using this method to scan databases and indexing that data into elasticsearch. The biggest advantage that this gives me is that I can control the number of documents that I push into ES at a single time since it becomes really sensitive if you exceed the request size or don't meet its expectations. Anyway , once that research is finalized, you'll find a relevant blogpost :)

Until then ...
