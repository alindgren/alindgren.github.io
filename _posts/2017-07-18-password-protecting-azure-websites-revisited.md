---
layout: post
title: Password Protecting Azure Websites Revisited
tags: [Azure, Basic Authentication]
comments: false
---
A couple years ago, I wrote about how to Password Protect Azure Websites with Basic Authentication. As I stated, I typically use basic authentication to password protect development or staging sites to block search engines and others from accessing the non-production sites. My post explained how I used a simple [Basic Authentication module](https://github.com/devbridge/AzurePowerTools/tree/master/Devbridge.BasicAuthentication) to implement basic authentication on Azure App Service (since basic authentication is not offered there 'out of the box').

While this module works fine, I’ve started using another one called [HttpAuthModule](https://github.com/nabehiro/HttpAuthModule) which has a couple advantages. First, it is published on nuget.org which makes it more convenient to install (at least for ASP.NET projects). Second, and most importantly for me, it is easy to turn on and off with an App Setting. This allows me to use Azure App Service’s slot settings for configuring the production site to not be password protected. I just create an App Setting for the production slot for HttpAuthModuleEnabled set to false. Finally, I like the ability to whitelist my office IP address (using the IgnoreIPAddresses setting). The module also supports Digest Authentication which I haven’t used as I’m not particularly concerned about the weakness of basic auth.

The HttpAuthModule is easy to use. I just installed via nuget and modified the updated web.config. The comments in the config are pretty straight forward as is the GitHub readme.

*Update:* I recently became aware of an Umbraco package SiteLock that will password protect a site so only logged in backoffice users can view the site. I haven't used it but wanted to list it here as another option.