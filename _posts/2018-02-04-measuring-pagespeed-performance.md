---
layout: post
title: Measuring PageSpeed performance
tags: [page speed, performance]
comments: false
---

Pagespeed is an important factor in providing a good web user experience. A commonly quoted statistic for this is that 40% of people abandon a website that takes more than 3 seconds to load. It is common for sites to see increased conversions after implementing performance optimizations. Pagespeed is not just a factor for user experience but also effects search. Google has stated that page speed is now a factor in their web search ranking algorithm.

`<aside>`Visit [WPO Stats](https://wpostats.com/) to view case studies and experiments demonstrating the impact of web performance optimization (WPO) on user experience and business metrics.`</aside>`

There are so many factors that go into providing good pagespeed for a site, that sometimes optimizing a site seems more like art than science. Most people start with Google’s [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) - a tool that scores the performance of a page using [several rules](https://developers.google.com/speed/docs/insights/rules):

* Avoid landing page redirects
* Enable compression
* Improve server response time
* Leverage browser caching
* Minify resources
* Optimize images
* Optimize CSS Delivery
* Prioritize visible content
* Remove render-blocking JavaScript

I often tell people to take the Google PageSpeed score with “a grain of salt” because a page that seems to load slowly can have a good score while a page that seems to load fast can have a lower score. This is because the score measures how well a site is optimized for these rules and not its actual performance, and these do not necessarily correspond. Recently Google acknowledged this in a [blog post](https://developers.googleblog.com/2018/01/real-world-data-in-pagespeed-insights.html) announcing changes to PageSpeed Insights:

> In the past, these recommendations were presented without the context of how fast the page performed in the real world, which made it hard to understand when it was appropriate to apply these optimizations. Today, we're announcing that PageSpeed Insights will use data from the [Chrome User Experience Report](https://blog.chromium.org/2017/10/introducing-chrome-user-experience-report.html) to make better recommendations for developers and the optimization score has been tuned to be more aligned with the real-world data.

With these changes, Google PageSpeed Insights reports now distinguish between speed and optimization. The optimization score is what previously displayed, as shown in this old screenshot:

![Old PageSpeed insights screen](https://alindgren.github.io/img/posts/2018/old_pagespeedinsights.jpg "Old PageSpeed insights screen")

