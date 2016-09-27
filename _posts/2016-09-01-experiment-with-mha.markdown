---
layout: post
title:  "Experiments with MHA !"
date:   2016-09-01 12:00:00
categories: data
tags: high-availability,ha,d3,data,mysql,failover
image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.jpg
---

This blog post is about my experiment with *M*ysql *H*igh *A*vailability . This is not a complete walkthrough and will highlight in a gist what all has to be kept in mind when setting MHA and what to expect when MHA runs . A detailed documentation with configurations and setting up will follow . People interested may further continue.

The need for MHA has been quite prevalent since mysql on production systems has been a single point of failure.

MHA in brief has the capability to monitor the master database and incase of failure (host unreachable , mysql shutdown) promotes a candidate slave to become the new master. It also ensures all slaves point to the new master taking into account any differentials that may have occurred during the master switch .

### What did I expect when MHA ran (Positive Test Case) :

* Master database fails .
* Mha manager detects failover
* Mha manager does the promotes the candidate master slave to the new master .
* All slaves now read from the new master
* All slaves are consistent with the master
* Mha runs a master_failover_script (customisable by us)
* The aws secondary ip shifts from the master to the candidate master ( this part was written in the master_failover_script itself)
* Apps reconnect to the new master and start running as they were .

### The setup :

The MHA setup leverages the power of virtual ips on AWS . This document provides an excellent explanation of how this is done. The idea is to have a virtual ip for all apps to connect. Incase of a failover the virtual IP will start point to the new database ( a candidate master incase a failover occurs) .

I used three mysql instances ( 1 master , 1 candidate master - nothing but a slave of the current master and 1 slave) and one instance for the app . The master database was assigned a secondary ip from the aws console itself.

![Setting up virtual ip on aws]({{ site.url }}/assets/article_images/2016-09-01-experiment-with-mha/virtual-ip-menu.png)

![Setting up virtual ip on aws]({{ site.url }}/assets/article_images/2016-09-01-experiment-with-mha/virtual-ip-selection.png)

You also need to ensure that the master and the candidate master instances should have a virtual ip setup in addition to their primary ips. [Reference Link](http://askubuntu.com/questions/313877/how-do-i-add-an-additional-ip-address-to-etc-network-interfaces) .

Please ensure that while setting up the slaves , both of them should have the exact same configuration . Mha manager will not run if even a single parameter is a mismatch . My guess is in production , all slaves must read from a slave which has the exact same configuration as the candidate master . I am not going in depth with the installation instructions as they are well written inside the project wiki . The configurations that I used will be available with this document as separate files .

MHA exposes a script known as the "master_failover_script" which is executed when the mha_failover is completed. This is the time when we do our aws specific work of switching the secondary ip to the new master . This script is called by mha manager with some parameters by default . An example is

```
/etc/mha/master_ip_failover_new --command=start --ssh_user=root --orig_master_host=10.0.42.208 --orig_master_ip=10.0.42.208 --orig_master_port=3306 --new_master_host=10.0.42.209 --new_master_ip=10.0.42.209 --new_master_port=3306 --new_master_user='testuser' --new_master_password='123123'
```

The application that I wrote to test this setup inserted records into the master database. There were two separate programs running in parallel which read from the other two slaves . The idea was to detect if there was any discrepancy in the records on both the slaves but my rate of inserting documents was one in three seconds so the consistency check is still something I have to get back with .

### The experiment :

I chose to run the MHA manager on one of the candidate masters . Ideally this should be a separate box on production . On starting MHA, I saw two warnings :
```
[warning] secondary_check_script is not defined. It is highly recommended setting it to check master reachability from two or more routes.
[warning] shutdown_script is not defined.

```

Although they didn't conflict with my experiment but I think both of them are actionables . The first one is quite self explanatory which basically ensures that even if the connectivity between the mha manager instance and master db fails , there is another route in a separate network to validate this claim . If the second route establishes a connection , a failover does not take place.

The shutdown script is a good point to switch off the old master instance .

### Now the juice .

Mysql was stopped on master database , mha_manager detects the failover .

Tue Aug 30 14:54:16 2016 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '10.0.42.208' (111))

Tue Aug 30 14:54:16 2016 - [warning] Connection failed 1 time(s)..

Tue Aug 30 14:54:19 2016 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '10.0.42.208' (111))

Tue Aug 30 14:54:19 2016 - [warning] Connection failed 2 time(s)..

Tue Aug 30 14:54:22 2016 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '10.0.42.208' (111))

Tue Aug 30 14:54:22 2016 - [warning] Connection failed 3 time(s)..

Tue Aug 30 14:54:22 2016 - [warning] Master is not reachable from health checker!

Tue Aug 30 14:54:22 2016 - [warning] Master 10.0.42.208(10.0.42.208:3306) is not reachable !




The application starts throwing the "Econnection refused , unable to connect " and the program reading from the slave does not find a new ID that from the master . After 15 seconds or so , I get this message :
Tue Aug 30 14:54:29 2016 - [info] Master failover to 10.0.42.209(10.0.42.209:3306) completed successfully.

All my expectations listed above were met . The app reconnected and started inserting records on the new master . The slave received these records from the new master.

What next ?
* I still have to check the consistency check for the slaves when the rate of insertion is quite high .
* Look out for cases where mha might run an exception in the middle of something . Will compile a more exhaustive list in such a scenario .


Incase you need any help with the mha_config and setting up the master_ip_failover , feel free to holler :) . I used [ node js' mysql package ](https://github.com/mysqljs/mysql) to connect to the database .