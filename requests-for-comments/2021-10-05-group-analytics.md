# Request for Comments: *topic of the proposal*

- **Created:** 2021-10-05
- **Owner:** @macobo
- **GitHub issues:** https://github.com/PostHog/posthog/issues/2247

> While commenting on this, please comment on specific lines if possible, to thread the discussion.

## Overview

This RFC proposes a solution which would allow PostHog users to leverage insights and feature flags not only by users but also by groups (e.g. organizations, projects).

## Background
B2B products don’t deal directly with users, but with other companies and hence have more complicated needs around what kinds of metrics are most important for them.


Lack of this feature has been a major driver for users wanting data exports in different analytics products, forcing users to roll their own analytics on top of data warehouses to get at their key metrics.

### Example usecases


- Show me how many active “organizations” there are
- Show only pageviews from organizations who are paying us (filtering by group properties)
- Have an overview of groups and their relations with users
- Feature flags: roll out feature group-by-group

For PostHog the product, the 3 group types we would use are:

- Instances (e.g. cloud, posthog.someproduct.org, …)
- Organizations
- Projects

## Semantics & Constraints

### 1. Users can belong to multiple groups

Some examples:

1. Contractor type work (e.g. for farm management agronomists work for multiple farms at once)
2. Billing happens on a different level (e.g. our projects vs organization)
3. Users clients are global companies with a lot of local “branches” where execs need access to everything
4. Internal users have access to debug issues on multiple accounts

### 2. User is active in only “group” at any point in time

Whenever an event is made, we can (and should) make the assumption that it belongs to a single group (e.g. organization). This is needed to avoid “double-counting” in stats.

Example: If Yakko has been added to 20 different organizations and is active, we don’t want to count all of them as active.

### 3. Limited amount of group types

From speaking with clients, their internal amount of different “group types” (e.g. `organization`) is limited, though the number of groups under a group type is unlimited (e.g. Facebook if it was an organization).

This constraint allows us to implement things slightly more efficiently and have a nicer UX compared to unbounded amount of groups.

### 4. Feature flags in groups should be consistent

For the users who are using feature flags, they want to ensure a consistent experience within a group during rollout.

This is to avoid situations where some users see a new feature and some do not.

This also factors into their later analysis - for example, they’d like to treat two different “organizations” both as a single datapoint even if one has 1500 users and the other has 1.

## Proposal

### Groups are event-level

Users can define up to N (5?) group types. For us these would be `instance` , `project` and `organization`.

Each event we ingest can have one group per group type associated.

We will have N new columns in the `events` table, containing the group ids - first one for the first group type of the project, second for the second one etc.

### Group properties API

We will expose a new API for creating and adding properties groups. This API can be exposed on the client level, e.g.

```python
    import posthog

    posthog.groups.add_properties('organization', 'org:5', { name: 'Swedish Bakery', subscription_level: 'gold' })
```

There will not be any functionality for merging two groups, as that behavior is user-specific.

### Capture API

Capturing events in client libraries will allow users to pass group metadata alongside with an event.

Here’s how it would look like in python:

```python
    import posthog

    posthog.capture(
      user_id,
      'purchase'
      { plan: 'silver' },
      groups={
        'organization': 'org:6'
      }
    )
```

`groups` metadata will be sent to us under event properties under new the `$groups` key.

Our data ingestion in plugin-server will perform the neccessary mappings from group ids to column indexes or other keys as needed.


### In-App: filtering by group properties in insights

In-app, users will be able to do analytics by groups and filter by them.

To give an example:

- Throughout insights, we’ll allow filtering insights with group properties. This will be implemented as a new tab per group type in the filters modal.
- In trends, users can also select “unique organizations” after first capturing events for an “organization”
- In funnels, users can select to analyze by a group type (e.g. “instance”). Different users within the group can have completed different steps of the funnel.
- In retention, users can select “organization” over “user” for the unit to analyze retention by
- And so on…


### In-App: profile pages for groups

We’ll also expose pages for different groups in-app, with:


1. Listing all your different group types
2. Listing and allowing searching all different groups of a type (e.g. organizations)
3. Group page with
    1. Group properties
    2. Events from that group
    3. Related users and groups (calculated via users’ other events)

Initially, the design and behavior will be taken from the person pages and iterated later on.


### Autocapture & posthog-js

To make autocapture work with groups and to make posthog-js usage a bit simpler, we can provide the following API:

```js
    // Normal posthog usage
    posthog.identify(userId)
    // New proposed method, to be called right after .identify
    posthog.group('instance', 'posthog.yourapp.com', { instance_prop: 5})
```

After calling this method, we will automatically associate events from that session with that group until either:

- `posthog.group` is called with the same group type
- `posthog.reset()` is called.
### Feature flags

Users should be able to add group-level property filters to feature flags and be able to ensure rollouts at a group level (?).

However feature flags will not be part of the initial scope of the project, see below.

## Open questions
### What pricing tier this belongs to

Does this sound like a collaboration-related feature we should immediately upsell? This is what mixpanel and amplitude do.

Based on discussion on RFC we'll make this a paid feature.

### Feature flags and posthog-js timing

Feature flags should work with group filters eventually. One tricky part there is behavior of posthog-js and ensuring we “know” what groups are currently active at the time of the request.

### How would “managing” groups go, if users manage to mess up?

TBD, can focus on this after the initial MVP

## Other details

### Initial scope

This project is quite large. Rather than update all of our client libraries, ingestion, queries, feature flags and insights pages at once, the plan would be to do a gradual rollout in phases, gathering feedback from stakeholders at each step.

The first step would be an internal MVP, where we can experiment with the feature ourselves behind a feature flag.

Some tasks involve implementing new logic in posthog-js and posthog-python client libraries, get queries working for trends/funnels, implement very simple group profile pages.

Work on feature flags specifically will be done later due to the extra complexity involved in making sure feature flags would work in posthog-js as expected.

### Competitor solutions:

Two other analytics products advocate group analytics:

- Mixpanel https://mixpanel.com/group-analytics/
- Amplitude https://help.amplitude.com/hc/en-us/articles/115001765532-Accounts

The two approaches are different: Mixpanels groups are user-level, while Amplitudes approach is similar to this proposal with groups being on events.

Other resources:

- https://www.avo.app/docs/data-design/groups

### Why groups on events not users?

When initially thinking about groups, it's tempting to think of them being associated with users. After all, it's users who belong to various organizations and other things that can be considered groups.

Worse, groups on events has some clear trade-offs involved:
- It's "harder" to get started for our clients
- More work for us building queries and feature flag support

However during research we found that:
- All B2B companies we spoke to had some form of "log in as user" functionality. This would cause issues when trying to figure out key metrics such as "is this group active".
- All of the B2B companies we spoke to didn't have a strict hierarchy - even ignoring internal users, there were special cases where some users belonged to multiple companies. In the best case they were trying to move away from this model, but it was confusing nonetheless.
- Data quality is integral - e.g. you want to avoid double-counting conversion/revenue because one user belongs to multiple groups or if an existing user got added to a new group. See also semantics/constraints above.

This matches up with prior experience from @marcushyett-ph and @macobo doing data work at startups small (eAgronom) and big (facebook).

In the end doing groups on events rather than users builds a baseline which can be trusted long-term without running into data trust issues or committing to a solution which does not solve the right problems..
