---
title: "Logging best practices"
layout: post
redirect_from: /2015/10/14/logging-best-practices
read_time: 3 minutes read
categories: ['published', 'featured']
---

Logs are one of the most fundamental tools for debugging and finding issues
within your application. Below, I am noting down a few important practices which
will help you to have nice and easy to follow log entries

## Unique identifier on every log entry

Unlike standard apache/nginx logs, modern applications can produce several log
entries on every request. To make reading them and parsing them easy prepend
unique identifier (UID) to each log line:

Instead of:

```
Started GET /login
Authentication successful
Completed 200 OK in 7ms
```

Logs should look like:

```
[unique-id] Started GET /login
[unique-id] Authentication successful
[unique-id] Completed 200 OK in 7ms
```


## Log events and tag them

Consider a following log entry:

```
Started POST /friends/import
Saved 1334 entries
Completed 200 OK in 7ms
```

We logged a result of a task here - importing friends from facebook. Its useful,
but its not easy to find all such entries in log file. Here is how you can
improve your log:

```
[unique-id] Started POST /friends/import
[unique-id][fb-friends-import] Saved 1334 entries
[unique-id] Completed 200 OK in 7ms
```

Tag `fb-friends-import` will make searching for all friends imports an easy job.

## Timestamp on every log entry

It is important to include timestamp on every log entry as well:

```
[2015/02/12 19:12:12.1233][unique-id] Started GET /login
[2015/02/12 19:12:12.2233][unique-id] Authentication successful
[2015/02/12 19:12:12.3233][unique-id] Completed 200 OK in 7ms
```

Timestamps may help with associating a log entry with a customer support
request. They may also be useful for log forwarders.

## Time metrics on log entries

Sometimes it's useful to add two time metrics to each log line:

  * **Time passed from request start** - can be used to check statistics, finding outliers etc.
  * **Time passed from last log entry in given request** - can be used to perform more sophisticated debugging

Consider such entries:

```
[unique-id-123][24 ms][1ms] fb import started
[unique-id-123][1240 ms][1000ms][fb-friends-import] Saved 1334 entries

[unique-id-789][14 ms][1ms][fb-friends-import] fb import started
[unique-id-789][24 ms][10ms][fb-friends-import] Saved 130 entries
```

First timestamp is a time from the beginning of a request. Second timestamp is
a time from a previous log message. By looking at above entries we can see that
our import algorithm took ~ 10 ms for 130 entries, but ~ 1s (1000ms) for 1334 entries.

Maybe it is O(n2), probably yes, and maybe a few more log entries would help us find out?

## Pass `Unique Identifier` to dependent services - workers etc.

Some systems have a complex topology. Consider worker based systems or a system
composed from several microservices. You may want to know the whole trace of
a request in your logs. Try to pass UID you generated on an entry point to each
subsystem that is used to finish the job. For example, when you have a worker that
is used to import facebook friends, your logs would look like this:

```
[unique-id][web] Started POST /friends/import
[unique-id][web] Enqueued facebook import for user_id=123
[unique-id][web] Completed 202 OK in 2ms
[unique-id][facebook-worker] Starting import for user_id=123
[unique-id][facebook-worker][fb-friends-import] Saved 1334 entries
```

Above, I ommited timestamps from beginning of a request and from a previous log entry for clarity,
but logging this information is even more important on a system where we have serveral components.

## Switch from INFO log level to DEBUG on production

Sometimes when your app is failing and you don't know what is happening
it may be useful to have a functionality to switch the log level from INFO to DEBUG
for a few minutes. DEBUG level should have much more details, which may be crucial
in solving an issue.

## Log forwarders and agregators

When you have more than one server, it is important to use log forwarders and
agregators. Here are a few services that can do that for you:

* [Logstash](https://www.elastic.co/products/logstash)
* [Heka](https://github.com/mozilla-services/heka)
* [Graylog](https://www.graylog.org/)
* [Kibana](https://www.elastic.co/products/kibana)

### Splunk

[Splunk](http://www.splunk.com/) is a great tool for processing logs.
It's my favourite tool when it comes to complex logs analysis.

It has many awesome features:

* advanced search
* search based alerts
* data visualization
* dashboard

Splunk is not free, but has 60 day free trial, and as they wrote:

> When the free trial ends, you can convert to a perpetual Free license

I couldn't find any more details info on that, though.

If you want to give it a try, check these 2 articles:

* [Splunk best practices](http://dev.splunk.com/view/logging-best-practices/SP-CAAADP6)
* [Splunk for sql users](http://docs.splunk.com/Documentation/Splunk/5.0/SearchReference/SQLtoSplunk)

## Useful resources

* [Google Testing Blog Article](http://googletesting.blogspot.ch/2013/06/optimal-logging.html)
* [Dapper Paper](http://research.google.com/pubs/pub36356.html) - this is great resource about logging in big systems, I really recommend to read.
