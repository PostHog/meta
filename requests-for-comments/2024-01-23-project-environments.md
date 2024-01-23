# Request for comments: Project environments, by Michael Matloka

## Problem statement

I will simply copy this bit from the [December 2022 "Feature flag environments" RFC](https://github.com/PostHog/product-internal/blob/main/requests-for-comments/2022-12-08-feature-flag-environments.md#221115---rfc-feature-flag-environments):

> Customers use environments, such as staging and production, so that they can test features before they are released to production. Multiple customers have asked us for an easier way to manage feature flags across multiple environments. At the moment, customers use separate PostHog projects for their multiple environments. To set up a feature flag in a new environment, it has to be set up from scratch, including key, variables and release conditions. This is time-consuming and can lead to errors.
> Furthermore, it's easy to make mistakes while changing flag definitions to test things, and having the flag exposed to a limited number of people reduces the chances of things going very wrong. By giving customers an easier way to move feature flags and/or flag settings across projects/environments, we are enabling them to release new features more successfully.

We solved this with a feature to copy feature flags between projects. That was the right call, because it's a relatively simple solution and users of feature flags rejoiced.

We've always had signals that the same problem applies to Product Analytics, but this is becoming more and more apparent. [TODO: Zendesk tickets here.] Users want to make sure their analytics instrumentation makes sense in dev/staging before they start using the data from production to make decisions. Setting up insights and dashboards in particular is a pain, because you need to duplicate a lot of work – projects are _entirely_ separate after all.

## Success criteria

[TODO]

## Context

The selling point of environments is that they keep the _data_ separate, while sharing _product setup_. There's some nuance to what counts as what, but that's the outline.

### What has to be separate

- events
- persons
- groups
- session recordings
- annotations
- toolbar sites
- pipeline destinations

The above must be filtered by environment at all times. In addition, there are caching mechanisms which have to account for environments:
- insight result
- cohort people

### What has to be shared

- insights
- dashboards
- feature flags
- experiments
- surveys
- early access features
- group _types_
- event/property definitions
- actions
- cohorts
- pipeline filters and transformations

The "Feature flag environments" RFC brought up one key need users have: often, the test setup needs some deviations from prod. You're likely to play around with instrumentation during development, so feature flags or cohorts might deviate in dev and you probably don't want that setup to be the source of truth. It's the production setup which you want to be a stable source of truth.

[TODO: Research other product analytics tools]

## Design 

[TODO: Probably an environment selector somewhere in the navigation, but needs to spec out the mechanics]

## Sprints

[TODO]
