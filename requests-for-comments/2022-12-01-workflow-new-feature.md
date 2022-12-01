# The first "Workflow": MVP for "Creating a new feature"

Owner: @EDsCODE (team experimentation)
Decision to be made: Should we build this in Q1 as part of team experimentation?
Decision date: 7th December 2022

## Context

This RFC proposes that team experimentation builds the MVP of a workflow for "Creating a new feature" as part of their Q1 OKRs.

It should sit with team experimentation as it would be a key driver of feature flag usage.

This could have multiple benefits across the product:

- Making PostHog the first place that product engineers go when building a new feature
- Increasing cross-product usage (feature flags, product analytics, session recordings)
- Encoding the best practices for PostHog and external engineers to use to build check for feature success without using PM time

## What's a workflow?

So far we have given product engineers the individual tools to create trends, create funnels, watch recordings, create feature flags etc.

We have the opportunity to connect these flexible tools with opinionated best practices to focus on specific jobs-to-be-dones in PostHog. These could be called "Workflows".

Two examples of potential workflows:

1. Building a new feature - assisting with rollout and measuring success *More details below*
2. Workbooks - making it easy to come up with questions, save related insights, clip particular bits of recordings and jot down thoughts

## Workflow: Creating a new feature

Here's the vision of what this could be in 3 - 6 months:

Sarah is a product engineer at a high-growth startup building a B2C website to help companies offset their carbon - called Carbon.io. She's working on a new feature for users to watch videos of their carbon-offsetting projects. She hypothesizes that users who watch these videos will feel more connected to the projects and retain at a higher rate.

The first thing Sarah does is open up PostHog and start the workflow for "Create a new feature". It opens up a page to create the new feature where she names it "Videos of offsetting projects". Optionally, she can enter some of the background information on this feature including the hypothesis. She selects "auto-rollout" rather than "A/B test" - meaning once the feature is made it will be used internally and after manual approval will be rolled out to beta users then low-value users then high-value users. She selects "user retention" as the key metric.

Sarah clicks "Done". PostHog automatically creates a feature flag to control the rollout; sets up a dashboard containing session recordings of people using it; and creates the relevant graphs.

Sarah builds the feature between the feature flag and merges it to main branch.

Sarah pings her team to give it a go on a few different devices. When 5 internal users have used the feature PostHog sends her a slack message to the playlist. She reviews it and the error logs - it's all working fine so she gives the thumbs up to escalate to beta users. There are no errors in beta and so it automatically deploys to prod - sending her a slack message to her team to keep them in the loop. PostHog automatically invites 2 users to a user interview after having used the video feature *Luke is working on an MVP of the user interviews feature - can be out of scope for the MVP*.

From the dashboard, Sarah learns most of the users are only watching the first 30 seconds of the 5-minute videos. During the interviews, she asks them what they thought of the videos and while they enjoyed them they said they were too long and dry. Sarah organizes a new set of videos to be made and rolls out an update.

3 months later Sarah gets a slack message from PostHog to view on the retention graph. They can see there's a significant correlation between retention between users that watched a video and users who retained. Success!
