---
layout: post
title: "Web Page Performance Tip: minimize CDNs used for external resources to minimize DNS lookups"
tags: [Page Speed, Performance]
comments: false
---
Minor front-end dev performance tip: when loading external resources from CDNs, try to limit the number of CDNs. For example, instead of loading the following two resources:

```
https://ajax.aspnetcdn.com/ajax/jQuery/jquery-3.1.0.min.js
https://cdnjs.cloudflare.com/ajax/libs/jsrender/0.9.74/jsrender.min.js
```

Load just from CloudFlare:

```
https://cdnjs.cloudflare.com/ajax/libs/jquery/3.1.0/jquery.min.js
https://cdnjs.cloudflare.com/ajax/libs/jsrender/0.9.74/jsrender.min.js
```

This is good because it will reduce the number of DNS requests the browser will make thus shaving off a few milliseconds. In one test I made, the aspnetcdn.com DNS look up tool 97ms:

![Chrome Tools showing DNS lookup speed](https://alindgren.github.io/img/posts/2017/cdn.png "Chrome Tools showing DNS lookup speed")

In general, I opt for CloudFlare CDN because it’s one of the most popular (which means it is more likely to be cached in a client’s browser) and has a ton of libraries. Also, I've heard that Google, which also offers CDN hosting for popular libraries, is blocked in China.