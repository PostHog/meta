# Request for comments: Improve Web Analytics Performance by Materializing Sessions - Robbie Coomber

## Problem statement
*Who are we building for, what are their needs (include example cases), why is this important?*

Some larger customers are beginning to use Web Analytics, e.g. one with 10M pageviews a week. These customers are not
able to query more than a couple of weeks of data at a time without experiencing timeouts. We also experience this 
ourselves

## Success criteria
*How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?*

The goal of this RFC is for our largest customers to be able to use the Web Analytics dashboard to query their data for
all time without needing to worry about performance at all.

A secondary goal is for the solution to be cheap to run, so that we don't lock ourselves out of offering a very generous
mass-market free tier for Web Analytics.

## Context
Some internal links where customers have reported poor performance:
* https://posthog.slack.com/archives/C05LJK1N3CP/p1703097503435979
* https://posthoghelp.zendesk.com/agent/tickets/8465
* https://posthoghelp.zendesk.com/agent/tickets/8854

### Sampling
I tried adding sampling to the Web Analytics dashboard, but it causes data integrity issues due to how the events table
is sampled. It used the `distinct_id` as the SAMPLE key, but a user may have multiple `distinct_id`s in the same session
due to the way we handle anonymous users. This means that the same user may be sampled in and out of the same session.

In one experiment, I tried sampling by a factor of 0.1x on our data, and it led to an error rate of about 10%, with session duration being the most affected metric.
![posthog sample 0.1x.png](images/2024-01-21-efficient-querying-of-sessions/posthog%20sample%200.1x.png)
![posthog sample 0.5x.png](images/2024-01-21-efficient-querying-of-sessions/posthog%20sample%200.5x.png)
![posthog sample 1.0x.png](images/2024-01-21-efficient-querying-of-sessions/posthog%20sample%201.0x.png)

The speedup was significant, so it'd be extremely useful if we could support sampling without these issues in the
implementation of how we handle sessions.

### Materialized Views
Clickhouse provides some powerful primitives for materialized views, and we use them already for session replay events.
See https://github.com/PostHog/posthog/blob/962756b5bc453149aabbb789fee8f8f48e023e3f/posthog/session_recordings/sql/session_replay_event_sql.py,
in particular see how the table is partitioned and ordered, so there will be one row in the aggregate table per day per
session_id.

### Numbers
We'd

## Design 
*What are the key user experience and technical design decisions / trade-offs?*

## Sprints
*How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?*
