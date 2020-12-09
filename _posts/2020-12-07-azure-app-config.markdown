---
layout: post
title: "Azure App Config"
date: 2020-12-07 12:00:00 -0000
categories: azure app-config
---
<h1>Azure App Configuration Refreshing</h1>
Azure App Configuration is a fully managed service to centralize the settings of your applications separately from your code, which is very convenient in distributed architectures for updating shared configurations and managing values in one place.

This document outlines the possible solutions to updating app configuration values within deployed APIs during runtime without restarting the app.

The standard tier costs £0.895 per day and includes 200,000 requests, approx out at 138 calls a minute. £0.045 per subsequent 10,000 requests. 

<h2>Sentinel Pattern</h2>
Azure App Configuration has provided the capability to monitor changes and sync them to the configuration within a running application. This functionality allows for on-demand refresh of the configuration, without downtime or restarting the application.

So, to achieve that with the least amount of calls necessary, we use a "sentinel" key-value in the app configuration and trigger a reload for all of our configurations when that value changes.  We need to update the sentinel whenever we want the app to pick up the changes that we made in the app configuration. The app will see that it got updated and doesn't need to worry about the actual value.  The time span for each application can be fine-tuned, for example, batch APIs poll the sentinel key every 20 mins but real-time critical APIs every 30 seconds. 

Below is a code example in C#

![refresh event grid](/images/sentinel.png)

The sentinel key is filtered by API using the ‘label’ feature of app configuration values. 

Event Grid
This concept is event-based, as the application would react to an event from the Azure App Configuration. There are only two possible events to be handled: KeyValueModified and KeyValueDeleted. The assumption is that the application would accept only the first one. Filtering is possible based on labels so further prototyping is needed to allow specific applications to react only to events that are relevant for them (proper configuration construction, including prefixes and labels).

Events would be consumed by an Event Grid, which will propagate it to applications that have configured subscription for specific events (configured based on labels). There is a service called Event Grid System Topic that allows creating multiple subscriptions on the same event generated from a source (topic).

Event example:
```json
{
    "key": "Foo",
    "label": "FizzBuzz",
    "etag": "FnUExLaj2moIi4tJX9AXn9sakm0"
  }
  ```
The last thing needed to make it work is a function of type Event Grid. To be able to debug it locally, the endpoint is of type webhook instead of Azure Function. The configuration is described here.

Jumping to source code below:

![refresh event grid](/images/refresh.png)


Azure Function App Configure Startup method above, shows only setting up a connection to Azure App Configuration in source code and what is worth mentioning, gathering, and registering ConfigurationRefresher.

