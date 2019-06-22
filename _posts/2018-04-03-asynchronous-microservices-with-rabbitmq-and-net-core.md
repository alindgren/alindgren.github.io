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

As I mentioned above, there is added complexity when building a microservices application. In this case I have opted to use [RabbitMQ](https://www.rabbitmq.com/), a popular open source message broker. It is easy to install RabbitMQ on a Windows environment for development following the [instructions](https://www.rabbitmq.com/install-windows.html). RabbitMQ supports the AMQP protocol and has a wide range of client libraries. I’ve opted to use [EasyNetQ](http://easynetq.com/), an easy API layer that works on top of the standard [.NET RabbitMQ.Client](https://www.rabbitmq.com/dotnet.html). To verify that your setup is working, I recommend running the EasyNetQTest application described in the [EasyNetQ Quick Start](https://github.com/EasyNetQ/EasyNetQ/wiki/Quick-Start).

For now, I am keeping the Fundraise application essentially monolithic but adding this integration as a microservice. This is done by publishing an event to RabbitMQ from the Fundraise web application and having the Airtable microservice subscribe to that event. I have loosely based my code on Microsoft’s [.NET Microservices Architecture for Containerized .NET Application](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/index) e-book, specifically the chapter on [Implementing event-based communication between microservices](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/integration-event-based-microservice-communications), and the accompanying [sample application](https://github.com/dotnet-architecture/eShopOnContainers).

I created a separate library, [Fundraise.IntegrationEvents](https://github.com/alindgren/Fundraise/tree/master/Fundraise.IntegrationEvents), to define the [integration event](https://github.com/alindgren/Fundraise/blob/master/Fundraise.IntegrationEvents/DonationCreatedIntegrationEvent.cs):

```chsarp
using System;

namespace Fundraise.IntegrationEvents
{
    public class DonationCreatedIntegrationEvent : IntegrationEvent
    {
        public string UserId { get; private set; }
        public Guid FundraiserId { get; private set; }

        /// <summary>
        /// Donation amount in dollars
        /// </summary>
        public double DonationAmount { get; private set; }

        public string DonorDisplayName { get; private set; }

        public DonationCreatedIntegrationEvent(string userId, 
                Guid fundraiserId, 
                double donationAmount, 
                string donorDisplayName)
        {
            UserId = userId;
            FundraiserId = fundraiserId;
            DonationAmount = donationAmount;
            DonorDisplayName = donorDisplayName;
        }
    }
}
```

DonationCreatedIntegrationEvent inherits from IntegrationEvent which is following what was done in [Microsoft’s sample](https://github.com/dotnet-architecture/eShopOnContainers/blob/dev/src/BuildingBlocks/EventBus/EventBus/Events/IntegrationEvent.cs) application:

> Integration events are used for bringing domain state in sync across multiple microservices or external systems. This is done by publishing integration events outside the microservice. When an event is published to multiple receiver microservices (to as many microservices as are subscribed to the integration event), the appropriate event handler in each receiver microservice handles the event.

Note that while the Microsoft ebook does not recommend sharing a common integration events library across multiple microservices as I have done, it seemed necessary for EasyNetQ and since it is in it’s own library, it didn’t seem to me to be coupling the microservice too closely to the Fundraise application.

In the same library, I created an interface called IEventBus which defines the Publish event:

```chsarp
namespace Fundraise.IntegrationEvents
{
    public interface IEventBus
    {
        void Publish<T>(T e) where T : IntegrationEvent;
    }
}
```

In another library, [Fundraise.IntegrationEvents.RabbitMQ](https://github.com/alindgren/Fundraise/tree/master/Fundraise.IntegrationEvents.RabbitMQ), I’ve implemented the IEventBus interface for RabbitMQ. The reason I defined an interface for this is because I will likely create other versions for other message broker such as Azure Service Bus. The Publish method is generic, of type T where T is an IntegrationEvent. The RabbitMQ implementation of IEventBus uses the EasyNetQ library to publish the message.

Currently, I am calling the Publish() method from the [DonateHandler](https://github.com/alindgren/Fundraise/blob/master/Fundraise.RequestHandlers.InProcess/DonateHandler.cs) (which uses the Mediator pattern per my last blog post). This is not ideal as it adds a dependency on [Fundraise.IntegrationEvents.RabbitMQ](https://github.com/alindgren/Fundraise/tree/master/Fundraise.IntegrationEvents.RabbitMQ) for the [request handlers library](https://github.com/alindgren/Fundraise/tree/master/Fundraise.RequestHandlers.InProcess). I plan on fixing this by adding events to the handler library and handling the events in the web application. But for now it is working and publishing events to RabbitMQ whenever a new donation is made.

Finally, I created a new [console application](https://github.com/alindgren/Fundraise.AirtableIntegration/blob/master/Fundraise.AirtableIntegration.ConsoleApp/Program.cs) that subscribes to the DonationCreatedIntegrationEvent and has an event handler that calls the Airtable API to save the donation data to Airtable. The beauty of this is that the microservice is quite simple because it focuses on just doing one thing. It did however require a fair amount of setup for this architecture to work and requires the operational overhead of running RabbitMQ and various services.
