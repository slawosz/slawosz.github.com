---
title: "Monitoring web application response times with kibana"
layout: post
read_time: 10 min read
categories: ['kibana', 'logs', 'elasticsearch', 'draft']
preface: 'Visualizing request times with Kibana Timelion, easy and effective way to understand and monitor application performance.'
---

## Why response time is an important metric?

Response times are an excellent insight into web application overall behavior. We can have a clear picture, how well application behaves, and we can easily spot any anomalies when values are changing. Based on this metric, we can make a decision, when and how to scale our application.
After scaling, we can compare past and current metrics and see how our changes affected application performance.

## Which metric should I use and why percentiles?

The best metric in most scenarios is using response time percentiles. For those who don't know, percentiles are statistical measurements based on Gaussian normal distribution. Fortunately, we don't need to calculate anything. However, if you would like to learn more, you can read about it on Wikipedia. For this article and real life, percentile splits sorted set of values into two sub-sets. Percentile number, lets call it X, denotes how many percent of all original sets elements are in first set. Then, the biggest value of this set is value of percentile X. For example, in sets of requests, if we say 90th percentile of our request time is 200ms, this means that 90% of requests are faster than 200 ms. In other words, 9 of 10 users has an acceptable (200 ms) response time.

## Why not average?

Because the average is a very unfair metric, there is an old statisticians joke, "If you take your dog for a walk, you and your dog have three legs on average each." You can easily cheat by using the average value for your metric. For example, the average salary in some country could be 2500$, but it says nothing about general wealth. In the same country, the 50th percentile of all salaries is 1500$, and this number gives us concreate information - half of the people earns less than 1500$, which is much, much less than 2500$.

## Percentiles it is.

That is why it is essential to use percentiles - you will always show Xth % of slowest requests. Which percentiles should we look at?
It depends, but good numbers are 50 (half of all requests), 90th, 95th. 99th percentile might be useful as well - especially if you would like to find outliners - request that is taking a lot of time. You can see what the value is and search for all requests that took longer.

## Building request times metric in Kibana

After a little bit of theoretical introduction, let's build something. To create your diagram, you need to have a numeric field which contains request duration.


### Sample data and local ELK stack

If you don't have access to local kibana, don't worry. Follow these short instructions to start your own ELK stack:
1. Clone docker composed ELK stack from `git clone git@github.com:deviantony/docker-elk.git`
2. Start stack by running `docker-compose up` in `docker-elk` directory.
3. Create kibana index pattern: ``. Index pattern informs kibana, which Elasticsearch indices will be used as data sources.
4. Download sample logs:

### Building the graph

When you log in to kibana, you will see 'Discovery' page, similar to one on the screenshot below. What we want to build is Visualization, exactly Timelion visualization. To create a new one, click the dashboard icon:

and from available options, please select timelion.

Now, we need to create timelion search query. I will use `duration` as my duration field. The following query will generate a proper diagram:

```
.es(index=*, metric='percentiles:request_time:50,90')
```

That's it! Let's take a look at the diagram below:

![Kibana percentiles diagram](/images/kibana-request-percentiles/kibana-percentiles.png)

For readability, I included only 50th and 90th percentile on diagram.

## Reading the diagram

We can learn a few things immediately after looking at the diagram:
* 50 th percentile looks quite good. Only few requests took longer than 250 ms.
* 90 th percentile looks not that good. Most of the time these requests are not exceeding 600 ms,
however, the slowest are taking more then one second.

## Next steps

There are several actions you can do to improve application response time.

### Temporailry increase number of servers

Increasing number of application servers, is good way to find out if our response times are depending from server capacity.
If the times will drop, it is an evidence that we can scale our application by adding server. Of course, there is limit of
servers we can add, beyong which response times won't drop, despite increased servers count.

### Take a look into outliners

Checking outliners, requests which are taking the most time, is good way to learn, why these particular requests are very slow.
Of course, our logs should have additional information, which will help retrace request in code. In typical web application environment,
it could be `user_id` field. Having such field, will help us recreate particular request on development environment.

Common reason of slow requests can be:
* slow database queries
* loading lot of data from database
* complex algorithms, which are slow with large sets of data.
* slow network requests
* lot of network requests

### Application optimization

With measurement in place, you can start improving application. Add network response caching where requests are slow, change http server threadpool size,
and see how it affects performance.

### One change at the time

When you are improving your application performance, it is very important to introduce one change at the time, as this will let you measure real impact of the change.
Wait until you will see how your change affects application performance before you will introduce next performance improvement. Otherwise, you won't be able to tell, which improvements were real and which made only small impact.

## Summary

Measuring application request times is simple but powerful metric. With Kibana, they can be easily visualized.


