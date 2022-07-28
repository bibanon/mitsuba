## Proxies
Mitsuba can be configured to use one or multiple proxies for requests to 4chan's API as well as the image fetching.
The load balancing system distributes the requests between them, allowing you to circumvent 4chan's rate limiting.
This is mainly intended for when it's strictly necessary, for example when 4chan is under DDoS attack which results in Cloudflare rate limiting clients to a degree that causes issues for archivers. This feature should not be used to abuse 4chan's API.

In order to add proxies, you need to set environment variables `PROXY_URL_{N}` to the URLs of the proxies you intend to use, where N is a number
starting from 0 for the first proxy, then 1 for the second, etc.
For example:
```
PROXY_URL_0=socks5://user:password@example.com:1337
PROXY_URL_1=socks5://user:password@10.0.0.2:1337
```
You have to start from 0 and not skip any numbers, or some or all of the proxies will not be detected.
You can set a weight for each proxy, this is an integer that determines how often it is used in relation to the others.
```
PROXY_WEIGHT_0=3
PROXY_WEIGHT_1=1
```
By default, even if you have set up proxies, Mitsuba **will** also use your machine's regular IP address (bypassing any proxies) alongside the proxies you configured, treating it as if it was a proxy with weight 1. Meaning that, if you set up two proxies, mitsuba will alternate between proxy 0, proxy 1 and not using any proxy for requests, based on the assigned weights.
This can be configured with `PROXY_ONLY`. If set to true, all requests will be routed through the proxies.
The weight of your own IP address as a "proxy" in load balancing can be set with `PROXY_WEIGHT_SELF`:
```
PROXY_ONLY=false
PROXY_WEIGHT_SELF=2
```
Note that the underlying HTTP library we use employs connection pooling. This means that the same connection to the server can be reused many times.
Whenever a connection is reused, the proxy will be the same as for the previous request that used the same connection. So, which proxy is used is only decided when a new connection is created, rather than whenever a request is made.
This means that the weights don't determine exactly how often the proxy is used to make requests, instead they determine how often they are used to open a new connection. Since connections can be reused any number of times, there is no guarantee that all proxies you configured will be used.

