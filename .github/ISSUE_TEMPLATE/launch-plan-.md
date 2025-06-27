---
name: 'Launch Plan'
about: For planning marketing launches and product announcements.
title: 'Launch Plan: [Product Name]'
labels: marketing
assignees: ''
---

_Start by saying what is launching, when and who is involved._

## Best practices

-   Ensure the product has at least one customer story within 3 weeks of launch
-   Ensure we publish best practice content for the product and link to it from docs
-   Ensure that products have at least one pre-made template (or similar) for users
-   Ensure the product has at least one tutorial at launch
-   Ensure the product has a robust docs page, and product page if needed
-   Ensure the product is added to email and in-app onboarding flows

## Launch plan

_Keep this list updated with current plans. Add linked PRs and issues, and assign people ASAP._

### Before launch - Deadline: [DATE]

<!-- Add / Remove tasks below depending on your needs. -->

-   [ ] Create a Slack channel called #{product-name}-launch - _team lead_
    -   Add @PostHog/{team-name} @joethreepwood, @PostHog/team-billing and the relevant exec
-   [ ] Find a case study customer for @joethreepwood - _team lead_
-   [ ] Ensure docs are up to snuff - @ivanagas
-   [ ] (Optional) Decide a reward for beta users - @joethreepwood, @PostHog/team-billing and _team lead_
    -   Note: one month free usage is plenty generous

#### (Optional) Product

<!-- Remove this section if you are not launching an entirely new product. -->

-   [In-app onboarding]() ([example](https://github.com/PostHog/posthog/pull/30071)) - _team lead_
-   [Product intents]() ([example](https://github.com/PostHog/posthog/pull/30099)) - _team lead_
-   [Website product page]() ([example](https://github.com/PostHog/posthog.com/pull/10988)) - @joethreepwood or @corywatilo
-   [Homepage]() ([example](https://github.com/PostHog/posthog.com/pull/11215)) - @joethreepwood

#### Pricing

<!-- Remove this section if you are not charging for the product. -->

-   [ ] [Write a pricing RFC]() [template](https://github.com/PostHog/billing/blob/main/notes/pricing-rfc.md) - _team lead_
    -   Model plans ([template](https://docs.google.com/spreadsheets/d/1Ue4qlfGyEz8EmwDIgB4ro2e9LLRoxM40lXTFA8ZLFXg))
-   [ ] Setup plans in Stripe - @PostHog/team-billing
-   [ ] Create plans in billing ([example](https://github.com/PostHog/billing/pull/1186)) - _team lead_
-   [ ] [Usage report]() ([example](https://github.com/PostHog/posthog/pull/28313)) - _team lead_
-   [ ] [Quota limiting]() ([example](https://github.com/PostHog/posthog/pull/30459)) - _team lead_
-   [ ] [Pricing calculator]() ([example](https://github.com/PostHog/posthog.com/pull/11143)) - _team lead_

### Launch day - Deadline: [DATE]

<!-- Add / Remove tasks below depending on your needs. -->

-   [ ] Merge any remaining PRs - _team lead_
-   [ ] Enable feature flag for all - _team lead_
-   [ ] Disable beta feedback emails - @joethreepwood

### Follow-on activity

-   [ ] Delete early access feature (& flag) - _team lead_
-   [ ] Move everyone over and delete the beta plans - @PostHog/team-billing
-   [ ] Reach out to #team-support to get them involved in triaging tickets - _team lead_

## Feedback and other ideas

All feedback and additional ideas from @marketing is appreciated.

_You can add additional out-there ideas here_.
