# Request for comments: Shared instructions between onboarding, wizard, and docs, Vincent Ge

## Problem statement

Onboarding (in app) instructions are frequently out of date compared to docs. 

These instructions are now important for the new installation wizards owned by the growth team.

We need a way to reduce work needed to maintain common instructions needed in three places, as well as to ensure good continuity between wizard, in-app onboarding, and documentation.

## Success criteria

- Store, write, update **installation** instructions used by in-app onboarding, docs, and wizard in one place.
- The shared instructions should be stored in a machine parsable spec that's also **highly readable by humans**.
- Updating instruction once should trigger updates everywhere.
- Warnings should be given when someone tries to update instructions in the wrong place (not at the source).
- Instructions should be rendered properly by onboarding, docs, and wizard. This should include minor differences, especially at checkpoints.

## Context

Here's some context on existing sources that could consume these shared instructions.

### Existing stepped installation

We moved to strongly structured, stepped, linear [installation instructions](https://posthog.com/docs/error-tracking/installation/web) in docs. These instructions have clear steps and checkpoint concepts to check if each step is successful.

This offers readers clear checkpoints, sense of "quick wins", and clear goals during the most uncertain stages of their onboarding.

These concepts happen to also be important for the wizard, and allow future features like real-time feedback (like if events have been received). 

### In-app onboarding

When you sign up to PostHog, you're show in-app onboarding instructions. These are critical to smooth onboarding for self-serve customers.

In-app onboarding in theory should be able to map to docs 1:1, you should be able to pick up where you left off in-app and follow in docs. The instructions should also be non-conflicting.

The instructions in-app don't have clear ownership, and are often out of date. Their upside is real-time feedback on if events have been received.

### Wizard

PostHog has LLM-powered wizards to help you integrate with PostHog SDKs. These need some accurate source of truth to know how to integrate correctly. They also need more structured checkpoints, to self-check for integration correctness.

Their current challenges are occassional hallucinations, staying up to date with latest changes, and lack of self-checking mechanisms to see if changes made are working currectly.

## Design 

