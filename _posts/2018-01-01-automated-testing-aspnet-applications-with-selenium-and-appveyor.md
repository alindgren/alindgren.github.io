---
layout: post
title: Automated testing ASP.NET applications with Selenium and Appveyor
tags: [Appveyor, ASP.NET, Selenium, Test-driven development, unit testing]
comments: false
---
Many years ago I read the first edition of *Extreme Programming Explained* by Kent Beck. While it had a big impact on my programming career -- it was my first introduction to an agile development process -- I never had the opportunity to fully embrace its methodology. Two Extreme Programming practices that I continue to aspire to follow are Test-driven development and Continuous integration.

I am not going to try to fully explain what Test-driven development (TDD) is here as there are a lot of resources that do a better job than I might, but as I understand it, TDD is the practice of first writing unit tests, then writing the code to pass the tests and finally refactoring the code to improve its quality and maintainability. Following TDD, one should always have good test coverage of all code and this gives coders more confidence to make changes. Messy code often comes about when a programmer doesn’t want to touch some code for fear of breaking something else in case they do not fully understand it.

Continuous Integration (CI) is the practice of building code every time a developer pushes their changes to a shared repository. With each build, CI systems can run automated tests so problems are detected early. I have worked on some projects that have CI build process with Appveyor, including a couple Umbraco packages I’ve created. But as I have not done extensive automated testing, none of them ran tests.

Last summer I started developing Fundraise, an open-source side project, primarily as a learning exercise but with the concept of eventually becoming a useful tool for non-profit organizations. It is a .NET core library for building fundraising applications and includes a sample ASP.NET MVC web application. It allows me to explore some new technologies and patterns including Entity Framework Core with the repository pattern, Dependency Injection (using [Simple Injector](https://simpleinjector.org/)) and unit testing with MSTest. Also, last fall at a meetup on testing Umbraco, one of the things I learned from [Lars-Erik Aabech](https://twitter.com/bleedo) is that it is easy to run web browser tests using Selenium from a .NET project. So naturally, I took a little of my holiday time to automate the Fundraise project build and add some Selenium tests.

The first step was setting up automated builds using Appveyor. This is free for open source projects and easy to do. Since I am using Visual Studio 2017, I had to change the Appveyor build worker image to Visual Studio 2017 (defaults to Visual Studio 2015). Also needed to `add nuget restore Fundraise.sln` in "Before build script". For the unit tests, all I needed was to turn it on and set it to automatically discover test assemblies. Appveyor provides automatic discovery, execution and reporting for MSTest, NUnit, xUnit.net and MSpec.

Next, I created automated web browser tests using Selenium WebDriver. Selenium WebDriver directly starts a browser instance and allows you to control it via an API. Appveyor supports testing with the Selenium Firefox WebDriver (see https://www.appveyor.com/docs/how-to/selenium-testing/). To do this, I created [a new MSTest project](https://github.com/alindgren/Fundraise/tree/master/Fundraise.MvcExample.Tests) in the Fundraise Visual Studio solution.

The biggest challenge is to run the website (and in this case SQL Server) as part of the test process. While the Fundraise library is .NET Core, the MVC application is not and thus requires IIS. I found [a helpful post](http://www.michael-whelan.net/testing-mvc-application-with-iis-express-webdriver/) by [Michael Whelan](https://twitter.com/mjmwhelan) to be very useful and largely followed that for setting up IISExpress to run for the tests. The web server can then be started in [my test class Init method](https://github.com/alindgren/Fundraise/blob/master/Fundraise.MvcExample.Tests/AdminTests.cs#L20-L23) as follows:

```csharp
  var app = new WebApplication(ProjectLocation.FromFolder("Fundraise.MvcExample"), 12365); 
  WebServer = new IisExpressWebServer(app);  
  WebServer.Start();
```

And I stop it during [the cleanup method](https://github.com/alindgren/Fundraise/blob/master/Fundraise.MvcExample.Tests/AdminTests.cs#L68). If I had developed the MVC site in .NET Core, then one would simply start Kestral instead.

Now that the web server is setup to start when the tests run, I can create some tests. To do this, I added the Selenium.WebDriver and Selenium.Firefox.WebDriver nuget packages to my test project and create a couple tests. See [https://github.com/alindgren/Fundraise/blob/master/Fundraise.MvcExample.Tests/AdminTests.cs](https://github.com/alindgren/Fundraise/blob/master/Fundraise.MvcExample.Tests/AdminTests.cs).

Since the Fundraise MVC Example site requires a database server, I need to set up a database to use by the tests during the CI build. Fortunately Appveyor supports running databases and comes with SQL Server, MySQL, PostgreSQL and MongoDB pre-installed. See [https://www.appveyor.com/docs/services-databases/](https://www.appveyor.com/docs/services-databases/) for detail. You must specify which database services you are using in Appveyor’s services settings. Then in the “Before tests script” I update the connection string in the site’s web.config and create the empty database:

```powershell
$startPath = "$($env:appveyor_build_folder)\Fundraise.MvcExample"
$sqlInstance = "(local)\SQL2016"
$dbName = "Fundraise"
$config = join-path $startPath "Web.config"
$doc = (gc $config) -as [xml]
$doc.SelectSingleNode('//connectionStrings/add[@name="DefaultConnection"]').connectionString 
     = "Server=$sqlInstance; Database=$dbName; Trusted_connection=true"
$doc.Save($config)
sqlcmd -S "$sqlInstance" -Q "Use [master]; CREATE DATABASE [$dbName]"
```

The MVC Example site employs Entity Framework which will create the necessary tables when the site starts. This is all I need but one could add other setup steps to this script. Also note that on Appveyor, “Build machines are transient, which means the state between builds is not preserved and the next build is started on a fresh build machine - basically, you don’t need any clean-up logic in your build scripts.”

Some final details about Appveyor: First, the configuration can be exported to yml and stored in source control at the root of the repo (see [https://github.com/alindgren/Fundraise/blob/master/appveyor.yml](https://github.com/alindgren/Fundraise/blob/master/appveyor.yml)). Second, it’s nice to display the build status in the project’s GitHub page. This can be done by adding the sample markdown code from the badges tab of the Appveyor configuration to the README.md.