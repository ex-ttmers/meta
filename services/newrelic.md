# Newrelic

We use [Newrelic](http://newrelic.com) to track performance.

## Getting Newrelic to record time spent in nginx

Does not work in old versions of nginx. The ${msec} will throw an error, it needs 1.3.9 or above.

Add this to the nginx conf file to get headers added that New Relic will use for tracking time in the nginx layer:

<pre>
        location / {
             ...
             proxy_set_header X-Queue-Start "t=${msec}000";
             ...
        }
</pre>

Recommend using the Unicorn supplied nginx conf file to get started.

See these two URLs for more details:
* https://newrelic.com/docs/features/request-queuing-and-tracking-front-end-time
* https://blog.engineyard.com/2013/new-relic-queuing
