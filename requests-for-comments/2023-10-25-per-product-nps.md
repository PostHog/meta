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
- Link to draft survey for feature flags: https://app.posthog.com/surveys/018b665b-7396-0000-25ad-59cd7c843b4e
- Question 1:
    
  <img width="346" alt="Screenshot 2023-10-25 at 14 28 21" src="https://github.com/PostHog/meta/assets/14750837/5c58d7f9-706f-4407-adcd-0d4571b3f460">
    
- Question 2:

  <img width="343" alt="Screenshot 2023-10-25 at 14 29 05" src="https://github.com/PostHog/meta/assets/14750837/e2388e9b-0eb5-4749-ad18-00ef3ee2c3f2">
  
- Thank you: Get a free PostHog sticker (optional):      

    <img width="350" alt="Screenshot 2023-10-25 at 14 29 55" src="https://github.com/PostHog/meta/assets/14750837/5661f99a-e0d3-470e-92ba-d0f3d83d53a8">

  Note: There is a bug with the button, which is why there isn’t a more engaging CTA yet

## Release conditions

- Visitors on any feature flags page (app & eu). I am using the following regex: `https://(app|eu)\.posthog\.com/feature_flags`
- Visitors who have completed an action that stands for feature usage/activation (set up as a new property)
    - Product analytics: Insight created = true
    - Session replay: 5 recordings watched = true
    - Feature Flags: Feature flag created = true
    - Experiments: Experiment created = true
    - Surveys: Survey created = true

- Visitors who have signed up at least 30 days ago: `joined_at` before `date`
    - Note: Once we have behavioural surveys, we can update the targeting to only show to users who have created a flag / watched 10 recordings / created a survey / created an insight / created an experiment
- Don’t display to users who have seen another survey in the last 60 days
- Roll out to a small number of users, e.g. 10%, so that we can track changes over time, rather than asking every user right now. (If we would roll this out to 100%, there is a risk that no user will see another survey for a while, if we keep the ‘survey wait period’ controls

## Blockers / Dependencies

- [Fix long-running surveys issue](https://github.com/PostHog/posthog/issues/17863)
- [Fix product announcements ↔ surveys issue](https://posthog.slack.com/archives/C034XD440RK/p1698235239337499)
- If we want to give away a sticker: [Fix link/button in the last step of multi-question surveys](https://posthog.slack.com/archives/C034XD440RK/p1698167499567089)
