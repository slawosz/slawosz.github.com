--- 
title: "Logging best practices"
layout: post
categories: ['en']
---

Logs are one the most fundamental tools to debug and find issues within your application. Below, I am noting down few important practices which 
would help you to have nice, easy to follow log entries

## Unique identifier on every log entry

Unlike standard apache/nginx logs, modern applications can produce many lines of logs on every request. To make reading and parsing easy, prepend
unique identifier (UID) to every log line:

Instead
```
Started GET /login
Authentication successful
Completed 200 OK in 7ms
```

Logs should be like
```
[unique-id] Started GET /login
[unique-id] Authentication successful
[unique-id] Completed 200 OK in 7ms
```

## Log events and add tags to them

Consider following log entry:
```
Started POST /friends/import
Saved 1334 entries
Completed 200 OK in 7ms
```

We logged in here result of some task - importing friends from facebook. Its useful, but its not easy to find all such entries in log file.
Here is how you can improve your log:

```
[unique-id] Started POST /friends/import
[unique-id][fb-friends-import] Saved 1334 entries
[unique-id] Completed 200 OK in 7ms
```

Tag (lets call it like this) `fb-friends-import` will make searching for all friends import super easy job.

## Timestamp on every log entry

It is important to include timestamp as well on every log entry as follows:

```
[2015/02/12 19:12:12.1233][unique-id] Started GET /login
[2015/02/12 19:12:12.2233][unique-id] Authentication successful
[2015/02/12 19:12:12.3233][unique-id] Completed 200 OK in 7ms
```

Timestamps will help to associate given log entry with ie. customer support request or will give a lot of help to log forwarders.

## Time metrics on log entries

Sometimes its very useful to add 2 time metrics to every log line:
* **Time passed from request start** - can be used to check statistics, finding outliners etc 
* **Time passed from last log entry in given request** - can be used to perform more sophisticated debugging 
 
Consider such entries:
```
[unique-id-123][24 ms][1ms] fb import started
[unique-id-123][1240 ms][1000ms][fb-friends-import] Saved 1334 entries

[unique-id-789][14 ms][1ms][fb-friends-import] fb import started
[unique-id-789][24 ms][10ms][fb-friends-import] Saved 130 entries
```
First timestamp is time from beginning of request, second - from previous log message. From above entries we can see, that our import algoritm took ~ 10 ms for 130 entries, but ~ 1s (1000ms) for 1334 entries. 

Maybe it is O(n2) - probably yes, few more log entries would help us find out? 

## Pass `Unique Identifier` to dependent services - workers etc.

Some systems have complex topology. Consider worker based systems or even system composed from 10-20 microservices. You may want to know the whole trace of request in your logs. So try to pass UID you generated on entry point to every subsystem which is used to finish the job. For example, when you have worker that is used to import facebook friends, your logs would look like this:

```
[unique-id][web] Started POST /friends/import
[unique-id][web] Enqueued facebook import for user_id=123
[unique-id][web] Completed 202 OK in 2ms
[unique-id][facebook-worker] Starting import for user_id=123
[unique-id][facebook-worker][fb-friends-import] Saved 1334 entries
```

Above, I ommited times - from request beginning and from last log entry - logging these information is even more important on system where we have serveral components.

## Switch from INFO log level to DEBUG on production

Sometimes, when your app is failing and you don't know what is happening it may be useful to have a functionality to switch log level from INFO to DEBUG for few minutes. DEBUG should have much more details which may be crucial to find an issue.

## Log forwarders and agregators

When you have more then one servers, it is important to use logs forwarders and agregators.

* [Logstash](https://www.elastic.co/products/logstash)
* [Heka](https://github.com/mozilla-services/heka)
* [Graylog](https://www.graylog.org/)
* [Kibana](https://www.elastic.co/products/kibana)

### Splunk

[Splunk](http://www.splunk.com/) is great tool for logs processing. Its my favourite tools when comes to complex logs analysis.
It has many awesome features:
* advances search
* search based alerts
* data visualization
* dashboard

Splunk is not free, but has 60 day free trial and as they wrote: `When the free trial ends, you can convert to a perpetual Free license`, but I can't get any more info.

If you want to give it a try, check this 2 articles:
* [Splunk best practices](http://dev.splunk.com/view/logging-best-practices/SP-CAAADP6)
* [Splunk for sql users](http://docs.splunk.com/Documentation/Splunk/5.0/SearchReference/SQLtoSplunk)


## Useful resources

* [Google Testing Blog Article](http://googletesting.blogspot.ch/2013/06/optimal-logging.html)
* [Dapper Paper](http://research.google.com/pubs/pub36356.html) - this is great resource about logging in big systems, I really recommend to read.
