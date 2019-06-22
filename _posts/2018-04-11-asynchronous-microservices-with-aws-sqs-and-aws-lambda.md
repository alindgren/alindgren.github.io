---
layout: post
title: Asynchronous microservices with AWS SQS and AWS Lambda
tags: [microservices]
comments: false
---

In my last post, I wrote about using RabbitMQ to integrate with an external API via a .NET Core microservice. 
It detailed how my [example Fundraise web application](https://github.com/alindgren/Fundraise) published an 
integration event to a RabbitMQ queue to which a 
[.NET Core microservice](https://github.com/alindgren/Fundraise.AirtableIntegration) was subscribed. 
That service then calls an [Airtable](https://airtable.com/invite/r/COvTRNbi) API to save the donation 
information as a record in Airtable. One of the advantages of using a microservices architecture like this is 
that it makes it easy to mix technologies. I thought I would try this out by replacing RabbitMQ and the .NET 
microservice with Amazon Web Services.

Amazon offers a lot of products -- a few of which may have been useful for my scenario. I opted to use 
[Amazon Simple Queue Service (SQS)](https://aws.amazon.com/sqs/) for the message queueing. Based on their 
description, SQS seems ideal for what I am trying to accomplish:

> Amazon Simple Queue Service (SQS) is a fully managed message queuing service that makes it easy to decouple and scale microservices, distributed systems, and serverless applications. Building applications from individual components that each perform a discrete function improves scalability and reliability, and is best practice design for modern applications.

