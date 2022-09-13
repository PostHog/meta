# Request for comments: Notifications & @pauldambra 

## Problem statement
*Who are we building for, what are their needs (include example cases), why is this important?*

Users don't know when to come back to PostHog. Something important can happen and they might miss it. This can be important things in the data, or that another user has edited your insight.

We could solve for this by extending the audit log. But you have to go check it to find new information. Where notifications can push important information directly to the user regardless of their context in the product. 

Additionally notifications can span multiple contexts, so they can communicate a broader range of information in very specific contexts. Audit logs are often specific to a single feature, so they only cover that scope.

Notifications can be split as active or passive. 

1. Active notifications are sent to you. 
2. Passive notifications are listed in the UI.

Or they can be split by urgency:

1. I want to know when something urgent has happened in slack / custom webhook so I can come to PostHog and investigate. 
2. I want to know what all has changed since I last logged in. These are in-app notifications about things I care about.

## Success criteria
*How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?*

We have users from [the August 2022 customer interviews](https://github.com/PostHog/posthog/issues/11368) that we can revisit.

We can measure:

* number of triggers created
* number of notifications sent
* stickiness of dashboards and insights

## Context
*What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?*

## Design 
*What are the key user experience and technical design decisions / trade-offs?*

```mermaid
graph LR
    A[Ingestion/Celery] -->|property seen| CS
    A -->|read triggers| N
    W-->|"create subscription (time-based trigger)"|N[("Trigger store (subscriptions)")]
    W[Web] -->|create a trigger| N
    Ac[Activity Log] -->|notable activity|CS
    N ---|evaluate triggers| CS[Celery sender]
    CS-->Slack
    CS-->Email
    Web ---|show notification list| NS
    CS -->NS[(notification store)]
```

There are some designs in figma for comment https://www.figma.com/file/Y9G24U4r04nEjIDGIEGuKI/PostHog-Design-System-One?node-id=3112%3A1296

### user considerations

* how to create the triggers for notifications
* where to see configured triggers
* linking from notification to appropriate onward journey

* expectation of performance
  * e.g. evaluate whether insight series have crossed a threshold could be very expensive

### Notifications people mentioned

* chart goes above or below a limit
    * expected to set this on an insight
* for a single event if the value of a property is over a threshold
* on particular calculated event
* notification when same event occurs for a single user/group over time (cohort entry/exit?)
* notification when experiment reaches significance

### Tech considerations

#### Celery

Celery often struggles with load (at least as we have it configured). Subscriptions are a recurring celery task so it makes sense to start there.

But it may be better medium term to have a stream of notifications in kafka so we can completely separate the triggering of a notification from acting on it.

## Sprints
*How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?*

There are several discrete chunks

* create the passive notification UI
* show someone a notification there when a subscription is sent (or fails)
* (use the activity log to) show someone a notification there when someone edits an insight, dashboard, or feature flag they created
* UI to set thresholds on graphs and to highlight when they are crossed
* ingestion creating a notification when event/property is seen
* ingestion creating a notification when property is seen with value over a threshold
* UI for creating event/property triggers
* cohort entry and exit triggers
* experiment status triggers

