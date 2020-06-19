---
layout: post
title: "Application Insights best practice"
date: 2020-06-15 13:00:00 -0000
categories: azure app-insights
---
<h1>Application insights</h1>
This is a guide on general best practice when using Application insights and some best practice guidance when using it alongside Azure Functions apps.

Application insights is an Azure feature offering monitoring and tracking for deployed applications. Its performance footprint is small, as tracking calls are non-blocking and batched with minimal performance overhead. The time and analysis provided far outweighs any minor performance cost it adds.

It can automatically detect anomalies, performance issues and provide analytics to diagnose the root cause of the problem. Application Insights is aimed at the development team, to help understand how your app is performing and how it's being used. It monitors:

*   **Request rates, response times, and failure rates** - Find out which APIs are most popular, at what times of day, and where your users are. If your response times and failure rates go high when there are more requests, then perhaps you have a resourcing problem.
*   **Dependency rates, response times, and failure rates** - Find out whether external services and integrations are slowing you down.
*   **Exceptions** - Analyze the aggregated statistics, or pick specific instances and drill into the stack trace and related requests. Both server and browser exceptions are reported.
*   **Performance counters** from your Windows or Linux server machines, such as CPU, memory, and network usage.
*   **Host diagnostics** from Docker or Azure.
*   **Diagnostic trace logs from your app** - so that you can correlate trace events with requests. Custom events and metrics that you write yourself in the client or server code, to track business events.

### Application Health Monitoring

Azure Application Insights allows you to monitor your application and send you alerts when it is either unavailable, experiencing failures, or suffering from performance issues. Common use cases are to create availability test to continuously check the response of the application and send mail to administrators when a problem occurs.

Application insights in not intended to be an audit system and is not suited for logging each individual request, sampling is preferred and offers the following advantages

*   It will automatically detect performance anomalies. This provides powerful analytics tools to help you diagnose issues and provide an understanding of your website.
*   Help continuously to improve performance and usability.
*   Create custom dashboards and integrate with Power BI

### Performance Diagnostics 

Application Insights collects telemetry from your application to help analyze its operation and performance. You can use this information to identify problems that may be occurring or to identify improvements to the application that would most impact users.

*   Identify the performance of server-side operations
*   Analyze server operations to determine the root cause of slow performance
*   Identify slowest client-side operations
*   Identify exceptions for different components of your application
*   View details of an exception

### Function App Guidance

Function Apps have application insights built in out of the box and can use the integrated ILogger interface to send exceptions, requests and trace messages. Alternatively you can create an implementation of TelemetryClient, and add custom logic within the logging middle ware. 

#### Common SDK usage
**TrackEvent** : User actions and other events. Used to track user behavior or to monitor performance.

**TrackMetric**: Performance measurements such as queue lengths not related to specific events.

**TrackException** : Logging exceptions for diagnosis. Trace where they occur in relation to other events and examine stack traces.

**TrackRequest** : Logging the frequency and duration of server requests for performance analysis.

**TrackTrace**: Resource Diagnostic log messages. You can also capture third-party logs.

**TrackDependency** : Logging the duration and frequency of calls to external components that your app depends on.

#### Recommendations

General best practice rules for using the above are:

1.  StartRequestTracking should be used for the request, each request and subsequent Trace information should be correlated within an operation context, sharing the same CorrelationID
    
2.  TrackExceptions should be used for any exception thrown
    
3.  TrackTrace should be used for logging messages throughout the request
    
4.  TrackEvent should be used for custom events, not tied to a specific request.
    

Some general advice when creating a custom instance of TelemetryClient within a function app.

1.  Don’t create TrackRequest or use the  StartOperation<RequestTelemetry> methods if you don’t want duplicate requests, it’s done automatically.
    
2.  Use the ExecutionContext.InvocationId value as the operation id. To then correlate info together.
    
3.  You have to set the telemetry.Context.Operation.\* items each time your function is started to correlate the items together. 
    

#### Outgoing dependencies tracking

You can track your own integrations or an operation that's not supported by Application Insights.

The general approach for custom dependency tracking is to:

*   Call the TelemetryClient.StartOperation method that populates the required DependencyTelemetry properties that are needed for correlation and some other properties (start time stamp, duration).
*   Set other custom properties on the DependencyTelemetry, such as the name and any other context you need.
*   Make a dependency call and wait for it.
*   Stop the operation with StopOperation when it's finished.
*   Handle exceptions.