---
layout: post
title: Asynchronous microservices with RabbitMQ and .NET Core
tags: [microservices]
comments: false
---

In my spare time I have been developing the [Fundraise](https://github.com/alindgren/Fundraise) application with the 
dual goal of exploring and learning new technologies and techniques while building something that could be used by 
non-profit organizations such as [The World Organization for Positive Action](https://www.worldpositiveaction.org/) 
(for whom I serve as a board member). Previous posts related to this project are Automated testing ASP.NET applications with Selenium and Appveyor and Thin Controllers using MediatR with ASP.NET MVC. This post is about integrating Fundraise with a third-party service using asynchronous microservices with RabbitMQ.

Microservices is an architectural style that decomposes an application into independent smaller services. Microservices can be deployed independently of each other and, if stateful, manage their own state with their own data store. The services can then communicate synchronously or asynchronously to each other through standard protocols such as HTTP and AMQP.

This loose coupling has a few advantages:

* Resiliency/fault isolation -- a failure in one service does not necessarily impact the rest of the application.
* Scalability -- each microservice can be deployed and scaled separately.
* Choice of technology -- because they communicate through standard protocols each microservice can be developed on their own tech stack and you can choose the best technology for the task.
* Ease of Development -- just like it is easier to solve a complex problem by breaking it down into smaller ones, it can be easier to program a complex application by breaking it down into smaller components.

But the advantages come with a couple disadvantages:

* Architectural complexity -- while each microservice might be simple on it’s own, the system as a whole is more complex.
* Operational complexity -- whereas a typical monolithic application may just require a web application server and a database server, microservices require more complex hosting with each service typically hosted in their own containers.

Fortunately cloud computing makes it easier to manage the increased operational complexity and services such as Google’s Kubernetes Engine and Microsoft’s Azure Container Service, Service Fabric and Service Bus, make it possible for even smaller organizations to run microservices.

One of the goals for the Fundraise project is to integrate with third party services. Ultimately I’d like to integrate with QuickBooks Online so that donors and donations made through a Fundraise site would automatically be added to the accounting system. For a proof of concept, I decided to start by integrating with [Airtable](https://airtable.com/invite/r/COvTRNbi), which offers a simple API for a customizable datastore.

As I mentioned above, there is added complexity when building a microservices application. In this case I have opted to use [RabbitMQ](https://www.rabbitmq.com/), a popular open source message broker. It is easy to install RabbitMQ on a Windows environment for development following the [instructions](https://www.rabbitmq.com/install-windows.html). RabbitMQ supports the AMQP protocol and has a wide range of client libraries. I’ve opted to use EasyNetQ, an easy API layer that works on top of the standard .NET RabbitMQ.Client. To verify that your setup is working, I recommend running the EasyNetQTest application described in the EasyNetQ Quick Start.

For now, I am keeping the Fundraise application essentially monolithic but adding this integration as a microservice. This is done by publishing an event to RabbitMQ from the Fundraise web application and having the Airtable microservice subscribe to that event. I have loosely based my code on Microsoft’s .NET Microservices Architecture for Containerized .NET Application e-book, specifically the chapter on Implementing event-based communication between microservices, and the accompanying sample application.

I created a separate library, Fundraise.IntegrationEvents, to define the integration event:
