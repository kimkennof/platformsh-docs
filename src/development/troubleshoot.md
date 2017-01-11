# Troubleshooting


## HTTP 502 response

If you're receiving HTTP 502 responses from your site,
that means your site is running out of workers.
Here are the typical causes:

* The amount of traffic coming to your site exceeds
  the processing power of your application.
* Certain code path in your application is too slow.

If you're running our PHP containers, you could first use the following
Platform CLI command to find the top 20 slowest requests in the last hour.

```
platform ssh "grep $(date +%Y-%m-%dT%H --date='-1 hours') /var/log/php.access.log | sort -k 4 -r -n | head -20"
```

If you see the processing time of certain requests are slow
(e.g. using more than 1000 ms), you may consider using profiler like
Blackfire to debug the performance issue.

Otherwise, you may check if the following options are applicable:

* Find the most visited pages and see if that can be cached and/or put onto CDN.
* Provide sizing hints to PHP-FPM workers.
* Upgrade your subscription on Platform.sh to get more computing resources.


## MySQL lock wait timeout

If you're receiving SQL error message like below:

```
SQLSTATE[HY000]: General error: 1205 Lock wait timeout exceeded;
```

This means a process running in your application acquired a lock from MySQL
for a long period of time. That is typically caused by one of the followings:

* There are multiple places acquiring locks in different order. For example:
  Code path 1 first locks record A and then locks record B.
  Code path 2, in contrast, first locks record B and then locks record A.
* There is a long running background process executed by your application that
  holds the lock until it ends.

Please make sure that locks are acquired in a pre-defined order and released
as soon as possible.
