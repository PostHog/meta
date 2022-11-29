# Request for comments: Web Performance from ALPHA to BETA, Paul D'Ambra

Desired decision date: December 10th 2022

## Problem statement
*Who are we building for, what are their needs (include example cases), why is this important?*

We're building for engineers who want to correlate performance with product events. Or who want to observe performance in their application in order to avoid adding another tool alongside PostHog to do that.

They want to answer these questions:

* how quickly do my assets loads
* how fast are my API responses
* do third party requests slow my application down
* how large are my assets
* are my assets or server responses compressed 
* have the speed or size of my responses changed over time
* what factors are affecting page speed of my application
* how do these web performance measures correlate with my product analytics

## Success criteria
*How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?*

We'll know this is successful when 5 target customers have adopted this tool, and given positive feedback.

This is ambitious because we're adding a new product to our suite. Where people will have clear expectations of performance and features.

We're not adding performance measurement on a timer, and we're not adding alerting on performance issues.

We're not adding performance measurements to mobile or back-end SDKs

## Context
*What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?*

### Similar tools

#### Sentry

https://sentry.io/for/performance

* backend and front-end performance transactions
* integrated with error reporting
* drill down into traces

#### Log Rocket

https://logrocket.com/features/frontend-performance-monitoring

* network speed measurement
* web performance metrics like first contentful paint
* as metrics and alongside session replay

#### GTMetrix/StatusCake/WebPageTest

various similar-ish tools

* on a timer
* load a web page and capture performance of network requests and browser performance
* report on individual page loads

## Design 
*What are the key user experience and technical design decisions / trade-offs?*

Initial alpha focussed on smallest set of data to draw a performance chart for a single page view. It uses the browser's web performance API to capture whatever performance data is available at point of sending a page view event. And includes the entire dataset as JSON on the pageview event.

This has several drawbacks

1) the JSON blob is large and we can't query individual items within it
2) not all of the interesting information is available at the point we send the pageview
3) we can't (easily) see how much data we're receiving that is specifically performance data

But, the advantage of leaning on the browser's API is gathering the data takes very little code. And lets us gather a wide set of useful information

![an example waterfall chart](/images/rfc-web-perf-waterfall-chart.png)

#### How should we improve this?

#### Gathering

The SDK already tracks a session ID. If we extend this to track a pageview ID (which rotates when we send a $pageview event) then we can send performance data either as a seperate event type, or through a separate channel.

This allows us to separate types of performance data so we can graph speed of API calls or asset requests.

Using a [`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) we can register to be notified when performance entries are available and then send them back as events.

This avoids having to patch fetch without ruling it out doing that in future.

#### Reporting

This gives us enough information to:

* report on API timings and sizes in Insights
* report on Asset timings and sizes in Insights
* report on page timing events in Inisights
* draw performance charts for individual page views or aggregated over a page
* draw performance timelines for individual sessions or aggregated sessions
* place network request logs along session recordings
* highlight slow network requests in session recordings

### This is lots of data

##### Config

We can allow users to configure which types of data they gather. They might not be interested in assets and only want to gather API call performance.

##### Infra / Performance of performance

Since this will send tens or hundreds of events per pageview we could receive a massive amount of traffic if sending each request individually. 

So, similarly to how we process session recordings, it makes sense to send info from the SDK in batches `{sessionId: string, pageViewId: string, entries: PerformanceEntry[]}` and then split them on ingestion. 

The individual performance entries are relatively small but will be numerous. Storing them separately to events would allow TTL deletion as we do with session recordings.

## Sprints
*How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?*

* add page view ID rotation to SDK
* add new API endpoint to accept/validate batches of performance entries
* update SDK to observe performance entries and batch them to new API
* add new route through kafka to a ClickHouse table for performance entries
* add new ingestion route that splits batches to the new kafka topic
* update existing performance waterfall to use new data store
* add network listing to session logs

### Appendix

#### Getting stored performance data currently

```
select formatReadableSize(sum(length(JSONExtractString(properties, '$performance_raw')))) from events 
where event = '$pageview' 
and timestamp < now() 
and timestamp > now() - interval 30 day
and JSONHas(properties, '$performance_raw')
```