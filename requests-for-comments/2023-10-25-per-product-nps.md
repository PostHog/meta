Per-product NPS

We would like to run permanent per-product NPS surveys in the PostHog app, for the following paid-for products:
- Product Analytics
- Feature Flags
- A/B Testing
- Session Replay
- Surveys

## Why?

- Trend of customer satisfaction, we see how customer satisfaction develops over time. If we find that user satisfaction for a certain product is low or trending down, we can react
- By asking follow-up questions to explain the score, we can learn which areas of the product in particular need improvement
- By asking on a per-product basis, we gain more actionable feedback than having a blanket survey for general PostHog usage
- The customer success team has an additional data point for individual customers if they are happy or not and can react quickly
- NPS is an industry-known benchmark and allows us to compare ourselves to other products. The scale itself allows for more granularity than the [PMF survey](https://posthog.com/founders/measure-product-market-fit#indicator-2-pmf-survey), which is similarly known, but only offers a 3-point scale

## Setup

- 2-step survey, NPS and open-ended question for feedback
- Link to live survey for feature flags: https://app.posthog.com/surveys/018bcec8-6cf5-0000-c724-a51a86a4e8b1
- Question 1:
  
    <img width="359" alt="Screenshot 2023-11-16 at 09 04 50" src="https://github.com/PostHog/meta/assets/14750837/5640a505-5898-4632-a6bc-6cbc374190fe">
    
- Question 2:
  
<img width="363" alt="Screenshot 2023-11-16 at 09 05 12" src="https://github.com/PostHog/meta/assets/14750837/619a9bc6-6cb6-4f30-8de3-4149ed1fe1c9">
  
- Thank you: Get a free PostHog sticker:
  
<img width="366" alt="Screenshot 2023-11-16 at 09 05 36" src="https://github.com/PostHog/meta/assets/14750837/d7b88484-ae41-45c6-8231-0d69f71a9671">

## Release conditions

- Users on any feature flags page (app & eu). I am using the following regex: `https://(app|eu)\.posthog\.com/feature_flags`
- Users who match the 'product usage' cohort, see definitions below, excluding PostHog team
- Users who have signed up more than 14 days ago
- Don’t display to users who have seen another survey in the last 60 days (set as survey setting)
- Don't display to users who have dismissed or answered this survey (set as property, because the survey setting is not working 100%)
- Roll out to a small number of users, e.g. 10%, so that we can track changes over time, rather than asking every user right now. (If we would roll this out to 100%, there is a risk that no user will see another survey for a while, if we keep the ‘survey wait period’ controls

## Appendix: How do we define 'product usage'?

In [this document](https://docs.google.com/document/d/16Z6m4kdgxui53SuSq-PeZ30lYfEsyict3u2fPm2vlsE/edit#heading=h.9kfq6mtxomjq), there is a proposal for what counts as an activated organisation for each product. I am slightly adjusting those definitions to define 'product usage' **per user**.

Product Analytics
- Completed event `insight created` AND event `insight analyzed` in the last 90 days: [Cohort](https://app.posthog.com/cohorts/48711)

Session Replay
- Completed event `recording analyzed` at least 5 times in the last 90 days: [Cohort](https://app.posthog.com/cohorts/48712)

Feature Flags
- Completed action `clicked-save-feature-flag (existing flags)` OR action `clicked-save-feature-flag (new flags)`
 in the last 90 days: [Cohort](https://app.posthog.com/cohorts/48720)

Experiments
- Completed event `experiment created` OR event `experiment launched` in the last 90 days: [Cohort](https://app.posthog.com/cohorts/48729)

Surveys
- Completed event `survey created` OR event `survey launched` in the last 90 days [Cohort](https://app.posthog.com/cohorts/48730)
