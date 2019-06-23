---
layout: post
title: Thin Controllers using MediatR with ASP.NET MVC
tags: [ASP.NET MVC, mediator pattern, MediatR, microservices]
comments: false
---

Last summer I started a project on GitHub called [Fundraise](https://github.com/alindgren/Fundraise). The original intent was to build a .NET core library for building fundraising applications. As someone who works with a small non-profit organization, I thought it would be a good idea to build software to help fundraise and track donations. The project admittedly follows the process of “Resume Driven Development” in that one of my goals is to learn new technologies and programming patterns that I have not yet had much opportunity to explore in my day job.

Much of the work on Fundraise has been done in conjunction with reading blogs and watching training videos to learn something new and then trying it out. I had intended to write a series of blog posts on a variety of topics: ASP.NET Core, Entity Framework Core, Dependency Injection, etc., but so far only one post, [Automated testing ASP.NET applications with Selenium and Appveyor](/posts/thin-controllers-using-mediatr-with-aspnet-mvc/), has been written.

Most recently I’ve been looking into microservices and various design patterns related to microservices. While not directly pertaining to microservices, one of the techniques I came across, and the first I decided to implement, was using the mediator pattern and the [MediatR library](https://github.com/jbogard/MediatR) to create thin controllers.

A fat controller is a controller that has a lot of dependencies and whose action methods contain a lot of business logic. For instance, the Fundraiser controller that handles the donation form had the following objects injected into the constructor:

```csharp
private ICampaignRepository _campaignRepository;
private IFundraiserRepository _fundraiserRepository;
private IDonationRepository _donationRepository;

public FundraiserController(CampaignRepository campaignRepository, 
  FundraiserRepository fundraiserRepository, 
  IDonationRepository donationRepository)
{
   _campaignRepository = campaignRepository;
   _fundraiserRepository = fundraiserRepository;
   _donationRepository = donationRepository;
}
```

The repositories are from the Fundraise library and use the repository pattern to access the database. The Fundraiser controller is still a work in progress and when completed, would likely include other dependencies such as a logger and other objects for other services such as notifications. Thus it is probable that this controller would have gotten much “fatter” and it is easy to imagine that ASP.NET MVC controllers getting out of control. Note that using Dependency Injection makes this more obvious as you see the list of dependencies in the constructor grow, but not using DI would just mean that the problem is pushed into the action method. (Before using DI, I relied a lot more on helper methods and singletons, but the problem of "fat controllers" still existed.)

My [original implementation](https://github.com/alindgren/Fundraise/blob/0436119cce0eed101bc5220b606fba997dbc3438/Fundraise.MvcExample/Controllers/FundraiserController.cs#L22-L27) of the Donate action in the FundraiserController contained the business logic:

```csharp
[HttpPost]
public ActionResult Donate(DonateFormViewModel model)
{
   var fundraiser = _fundraiserRepository.FindById(model.FundraiserId);
   var campaign = _campaignRepository.FindById(fundraiser.CampaignId);

  if (model.DonationAmount > 0)
  {
     var chargeService = new StripeChargeService();
     var charge = chargeService.Create(new StripeChargeCreateOptions()
     {
        Amount = model.DonationAmount * 100,
        Currency = "usd",
        Description = fundraiser.Name,
        SourceTokenOrExistingSourceId = model.StripeToken
    });
   string userid = null;
   if (User.Identity.IsAuthenticated)
   {
       userid = User.Identity.GetUserId();
   }
   _donationRepository.Create(campaign, fundraiser, DonationStatus.Completed, 
      model.DonationAmount, "usd", model.DonationAmount, userid, 
      model.DonorDisplayName, charge.Id);
   }

  var fundraiserViewModel = 
    AutoMapper.Mapper.Map<Fundraiser, FundraiserFormViewModel>(fundraiser);
  return View("Thanks", fundraiserViewModel);
}
```

It uses the FundraiserRepository to get the name of the fundraiser, the CampaignRepository to get the campaign, the StripeChargeService (from [Stripe.net](https://github.com/stripe/stripe-dotnet)) and the DonationRepository to record the donation. It makes sense to move all this out of the controller. To do this, I use the mediator pattern and the MediatR library.

With the MediatR library, I created a [request class called `Donate`](https://github.com/alindgren/Fundraise/blob/c4ed1b8705d2fb8cd917e10d327d210b6aef2c07/Fundraise.MvcExample/Requests/Donate.cs) and a [request handler called `DonateHandler`](https://github.com/alindgren/Fundraise/blob/c4ed1b8705d2fb8cd917e10d327d210b6aef2c07/Fundraise.MvcExample/RequestHandlers/DonateHandler.cs) to handle the above logic. After configuring MediatR, I updated the controller to use the mediator to process the request. The updated [Donate action method](https://github.com/alindgren/Fundraise/blob/c4ed1b8705d2fb8cd917e10d327d210b6aef2c07/Fundraise.MvcExample/Controllers/FundraiserController.cs#L70-L89) looks like this:

```csharp
[HttpPost]
public ActionResult Donate(DonateFormViewModel model)
{
   Donate request = new Donate()
   {
     DonationAmount = model.DonationAmount,
     FundraiserId = model.FundraiserId,
     DonorDisplayName = model.DonorDisplayName,
     StripeToken = model.StripeToken
   };
   request.UserId = User.Identity.IsAuthenticated ? User.Identity.GetUserId() : string.Empty;
   bool success = _mediator.Send<bool>(request).Result;

  var fundraiserViewModel = new FundraiserFormViewModel();
  return View("Thanks", fundraiserViewModel);
}
```

This method would be even shorter if I had used the Donate class as my model, but I decided to leave it so that it uses the existing DonateFormViewModel for now. Once I convert all the action methods in this controller to use the mediator, I can remove the injection of the Repository objects from the controller. This reduction of the use of injected services will mean that I have a thin controller!

I mentioned that I came across this use of the Mediator pattern when researching microservices. While it is not yet using microservices, MediatR provides In-process messaging and supports request/response, commands, queries, notifications and events. By clearly defining requests and their handlers, I think it will be easier to convert the MVC example app to one that uses microservices. I also want to explore the idea that one could implement a microservice version of the request handlers that could be swapped in for the non-microservice version thus allowing a Fundraise application the ability to run in either a simple web hosting environment or a more complex environment with support for queuing and containerized services.
