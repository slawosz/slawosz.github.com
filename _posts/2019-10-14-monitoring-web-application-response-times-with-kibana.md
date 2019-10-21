---
title: "Monitoring web application response times with kibana"
layout: post
read_time: 10 min read
categories: ['kibana', 'logs', 'elasticsearch', 'draft']
preface: How to visualize request times using Kibana Dashboard Timelion widget. s Includes short explanation what are percentiles and why they are important in interpretation.
---

Response times are most important metric to monitor web application performance. It is very easy to create kibana dashboard which will give proper insights what is happening to our application. In this blog post, I will show you how.

## Why response time is important metric?

Response times are very good insight to our application overall behaviour. We can have clear picture, how well application behaves, and we can easily spot any anomalies, when values are changing. Based on this metric, we can make a decision, when and how to scale our application.
After scalling, we can compare past and current metrics and see how our changes affected application performance.

## Which metric should I use?

The best metric in most scenarios is using percentiles. For those who don't know, percentiles are statistical measurement based on Gaussian normal distribution. Fortunatelly, we don't need to calculate anything. However, if you would like to learn more you can read about it on wikipedia. For this article, and real life, we just need to know that percentile X denotes value, which is bigger than X percent values in some group. In terms of requests, if we say 90th percentiles of our request time is 200ms, this means, that 90% of requests are faster than 200 ms. In other words, 9 of 10 users has acceptable response time.

## Why not average?

Because average is very unfair metric. There is old statisticans joke "If you take your dog for a walk you and your dog have 3 legs on average each". You can easily cheat by using average for your metric. For example, average salary in some country could be 4000$, but it says nothing about general wealth. In the same country, 50th percentile of all salaries is 1500$, and this number gives us concreate information - half of people earns less than 1500$, which is much, much less then 4000$.

## Pecentiles it is.

Thats why it is important to use percentiles - you will always show Xth % of slowest request. Which percentiles we should look at?
It depends, but good numbers are 50 (half of all requests), 90th, 95th. 99th percentile might be useful as well - especially, if you would like to find outliners - request that are taking lot of time. You can see what the value is and search all request that took longer.

Building request times metric in Kibana

After little bit of theoretical introduction, lets build something. I suspect that
you probably have searchable `request_time` field in your kibana. But if not, I created log file you can use, available here. I also described how to setup full kibana instalation with docker.

When you login to kibana, you will see 'Discovery' page, similar to one on screenshot below. What we want to build, is Visualization, exactly Timelion visualization. To create new one, click ....

Now, we need to create proper timelion search query. I will use `request_time` as my duration field and please note I specified additional query: `q='foo bar'`, to make sure, that I am using proper application. If your Kibana instance is shared with others, ommiting filtering might lead to errors.


```
.es(index=*, q='foo_bar',
  metric='percentiles:request_time:50,95')
```



