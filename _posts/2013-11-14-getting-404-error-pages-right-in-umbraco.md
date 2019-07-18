---
layout: post
title: Getting 404 error pages right in Umbraco
tags: [404, Umbraco]
comments: false
---
When a browser requests a URL that does not exist, they are [supposed to get a "404 Not Found" response](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.5). It is a good design practice to have a custom 404 design for your site. There are some good [creative examples](http://designmodo.com/404-error-pages-examples/) on the web.

A [basic search](https://duckduckgo.com/?q=configuring+404+error+pages+in+umbraco) for configuring 404 error pages in Umbraco returns results that mostly describe [how to set the id of the not found node](http://our.umbraco.org/wiki/install-and-setup/configuring-404-pages), the node that will be displayed when there are no other nodes that match a requested URL. Essentially, you need to specify the node id in the errors section of /config/umbracosettings.config as follows:

```xml
<errors>
  <error404>1826</error404>
</errors>
```

Since I am running IIS 8, I followed the instructions that to insert the following element before the end of the system.webServer section of web.config:

```xml
<httpErrors existingResponse="PassThrough"/>
```

This sort of works. For .aspx and extensionless URLs I was getting the content of the page I had set in umbracosettings.config. But other URLs, such as .html or .jpg were returning a 404 Not Found response with an empty body.

```
httpErrors>
   <remove statusCode="404" subStatusCode="-1" />
   <error statusCode="404" prefixLanguageFilePath="" 
      path="/missing.aspx" responseMode="ExecuteURL" />
</httpErrors>
```

With this setting, I now get the 404 page content returned regardless of the file extension.

