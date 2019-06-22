---
layout: post
title: Asynchronous microservices with AWS SQS and AWS Lambda
bigimg: /img/path.jpg
tags: [microservices, AWS SQS, AWS Lambda]
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

And for processing the queue, it seemed like a good idea to try out AWS Lambda:

> AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources for you.

Lambda supports Java, Node.js, Go, C# (.NET Core), and Python. While I could have stuck with .NET Core, since my goal is a proof-of-concept that demonstrates how easy it is to mix technologies, I opted to use Node.js. It didn’t hurt that Airtable’s API documentation includes dynamic node.js examples.

I was surprised to learn that AWS Lambda does not automatically hook into AWS SQS events. Perhaps I should have used another service such as Amazon Kinesis. According to AWS their documentation:

> You can create a Kinesis stream to continuously capture and store terabytes of data per hour from hundreds of thousands of sources such as website click streams, financial transactions, social media feeds, IT logs, and location-tracking events.
> 
> You can subscribe Lambda functions to automatically read batches of records off your Kinesis stream and process them if records are detected on the stream. AWS Lambda then polls the stream periodically (once per second) for new records.

But there are a few ways to make my architecture work with SQS. What I opted for was to set up a CloudWatch event to trigger my Lambda function to poll the queue every so often. I could set it up to run every 5 minutes, every hour or just once a day. If you aren’t too concerned about having your data updated too quickly, this works fine. It also allows you to reduce the number of requests Lambda is making, but given that their free tier includes 1 million requests per month and 400,000 GB-seconds of compute time per month and SQS also provides 1 million free requests per month, the cost per request may not be a concern.

The implementation was fairly straightforward. First, I set up an SQS queue using the AWS console. Then I created [EventBusAmazonSQS](https://github.com/alindgren/Fundraise/blob/master/Fundraise.IntegrationEvents.AmazonSQS/EventBusAmazonSQS.cs). Like [EventBusRabbitMQ](https://github.com/alindgren/Fundraise/blob/master/Fundraise.IntegrationEvents.RabbitMQ/EventBusRabbitMQ.cs), EventBusAmazonSQS implements the IEventBus interface which defines the Publish event:

```csharp
namespace Fundraise.IntegrationEvents
{
    public interface IEventBus
    {
        void Publish<T>(T e) where T : IntegrationEvent;
    }
}
```

Instead of using the EasyNetQ library, EventBusAmazonSQS uses the AmazonSQSClient (from the [AWSSDK.SQS](https://www.nuget.org/packages/AWSSDK.SQS/) nuget package) to send a message to the SQS queue:

```csharp
var sendMessageRequest = new SendMessageRequest()
{
    QueueUrl = "https://sqs.us-west-2.amazonaws.com/852229429830/FundraiseDonations",
    MessageBody = JsonConvert.SerializeObject(e)
};
var sendMessageResponse = sqsClient.SendMessageAsync(sendMessageRequest);
```

The example app uses SimpleInjector for dependency injection, and I replaced the RabbitMQ implementation with the AmazonSQS implementation for IEventBus:

```csharp
container.Register<IntegrationEvents.IEventBus, 
IntegrationEvents.AmazonSQS.EventBusAmazonSQS>(Lifestyle.Singleton);
```

I still call the Publish() method directly in the [DonateHandler](https://github.com/alindgren/Fundraise/blob/master/Fundraise.RequestHandlers.InProcess/DonateHandler.cs) and I still plan to move this out of that library by adding events to the handler library. But at least with dependency injection, DonateHandler is unaware of what implementation of IEventBus it is using.

It took awhile for me to get my AWS credentials configured correctly. See [their docs](https://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/net-dg-config-creds.html) for details, but basically I created a .aws folder in my user folder with a credentials file storing my default credentials. Once working, I was able to see the messages in the SQS queue via the AWS console.

Next, I created a Lambda function in the AWS console. I found a post on [Lambda for Asynchronous Message Processing through SQS](https://medium.com/@manjulapiyumal/lambda-for-asynchronous-message-processing-through-sqs-9b798a6c509c) by Manjula Piyumal that was very helpful. The Lambda function uses the [Airtable.js](https://github.com/airtable/airtable.js) (installed with npm) to create the record in Airtable.

```javascript
let AWS = require('aws-sdk');
let sqs = new AWS.SQS();
let Airtable = require('airtable');
let base = new Airtable({apiKey: '[API_KEY]'}).base('[APP_ID]');

exports.handler = (event, context, callback) => {
    
    sqs.receiveMessage({
        QueueUrl: 'https://sqs.us-west-2.amazonaws.com/852229429830/FundraiseDonations',
        AttributeNames: ['All'],
        MaxNumberOfMessages: '10',
        VisibilityTimeout: '30',
        WaitTimeSeconds: '20'
    }).promise()
        .then(data => {
            data.Messages.forEach(message => { // iterate through all  fetched messages
                let messageBody = JSON.parse(message.Body);
                console.log("FundraiserId: " + messageBody.FundraiserId);

                base('Donations').create({
                  "Id": messageBody.Id,
                  "Creation Date": messageBody.CreationDate,
                  "Donor Id": messageBody.UserId,
                  "Fundraiser Id": messageBody.FundraiserId,
                  "Donation Amount": messageBody.DonationAmount,
                  "Donor Display Name": messageBody.DonorDisplayName
                }, function(err, record) {
                    if (err) { console.error(err); return; }
                    console.log(record.getId());
                });
            
                sqs.deleteMessage({ // Delete message
                    QueueUrl: "https://sqs.us-west-2.amazonaws.com/123/Donations",
                    ReceiptHandle: message.ReceiptHandle

                }).promise()
                    .then(data => {
                        console.log("Successfully deleted message with ReceiptHandle : " 
                          + message.ReceiptHandle +
                            "and booking reference : " + messageBody.bookingRef 
                          + " with response:" + JSON.stringify(data));
                    })
                    .catch(err => {
                        console.log("Error deleting message with ReceiptHandle: " 
                          + message.ReceiptHandle +
                            "and booking reference : " + messageBody.bookingRef, err);
                });
                
            });
        })
        .catch(err => {
            console.log("Error while fetching messages from the sqs queue", err);
        });


    callback(null, 'Lambda execution completed');
};
```

Without little to no experience with AWS Lambda and SQS I was able to quickly implement the Fundraise microservice for integration with Airtable.
