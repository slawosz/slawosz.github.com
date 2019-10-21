---
title: "How to list postgres procedures"
layout: post
redirect_from: /2015/08/01/how-to-list-postgres-procedures
read_time: short read
categories: ['published', 'featured']
---

I needed to list all stored procedures on my postgres database. As usual, stack overflow helped:
[http://stackoverflow.com/questions/1559008/list-stored-functions-using-a-table-in-postgresql](http://stackoverflow.com/questions/1559008/list-stored-functions-using-a-table-in-postgresql):

{% highlight ruby %}
SELECT  p.proname
FROM    pg_catalog.pg_namespace n
JOIN    pg_catalog.pg_proc p
ON      p.pronamespace = n.oid
WHERE   n.nspname = 'public';
{% endhighlight ruby %}

PS. I am using [pgweb](https://github.com/sosedoff/pgweb) to inspect my postgres database, its awesome!
