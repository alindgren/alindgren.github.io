---
layout: post
title: Website hosting considerations
tags: [Hosting]
comments: false
---
In my welcome to my new site post I referred to the fact that I often make web hosting recommendations for our clients at Flightpath. Each client is different and usually their hosting needs are also different. That said, the following framework of considerations has helped me evaluate different hosts in order to make an informed recommendation.

First consideration is services. Since we are discussing web hosting, HTTP is of course the primary service that is required. But often other services are required. Other services that need to be considered are: outgoing SMTP, mail, database, storage, backup, source control, caching and content delivery services.

The next consideration is the technology platform or stack. The decision is usually driven by existing technology requirements such as existing or in development application requirements. We generally deal with a Windows or LAMP stack but additional technologies such as Solr need to be considered.

The next area of consideration I look at is security. Will the site require SSL for HTTPS connections? What kind of firewall should we use and what kind of firewall rules should be in place? Will there be PCI compliance or other security testing? How will we and our client access the servers? While everyone wants to say they are serious about security, in reality there is a continuum of levels of security and more security comes at a cost.

Reliability, defined as the amount of uptime of a system, is the next consideration I look at. Like security, everyone will say they want a reliable system but high reliability comes at a cost which comes down to the cost of redundancy. A lot of redundancy generally results in higher reliability but it costs more.

Scalability, defined as the number of users a system can handle, is another factor to consider. If it is critical that the site can scale to handle a certain amount of users and traffic, then one should verify that with load testing. Cloud infrastructure can also go a long way towards dealing with scalability because sites can be configured to auto-scale or at the very least servers can be resized with minimal effort.

Performance, defined as the speed of how fast the app runs, is another increasingly important factor. Performance is mesured in pages load speed and response times. The performance of a system will be impacted by the software as well as the hardware. Not all servers are equal and often times disk I/O is an overlooked factor in this area. Another factor in considering performance is the location of the server. A server may have super fast response times in North America, but if most of the traffic is coming from Asia or Europe, you should consider having a server there.

Technical support may be one the most important considerations when choosing a web hosting solutions. Since we long ago gave up on hosting with our own hardware, we are dependent on some level of support from the host provider. This will of course depend on the complexity of the system and how much of the system is covered. Some companies such as Acquia provide end-to-end support which includes the applications and not just the hardware. Others, such as Amazon have provided minimal support. In the last year or so I've been working with Liquid Web which so far (knock on wood) has provided a great value in this area.

Finally, one must of course consider cost. If cost was not a factor, then the other factors would not be an issue as you could always through more resources at it. But the fact is cost is always a consideration and often the prime consideration. Fortunately, hosting is very competitive and there are a lot of decent priced options out there which meets the needs of most of my clients.