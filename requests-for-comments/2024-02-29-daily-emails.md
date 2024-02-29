# Request for comments: Daily emails for users
**Owner:** @joethreepwood

We've launched our onboarding flow, and that's great. But we also want to [give users more help to get started if they want it](https://github.com/PostHog/meta/issues/171).

One way to do this which has been suggested by the community is to send _daily_ emails to users offering them tips on how to get started. 

This has proven [unpopular internally](https://posthog.slack.com/archives/C01FHN8DNN6/p1709031034701129) because it would feel like too many emails.

Rather than testing this within the main onboarding flow, the solution would be to create an additional _opt-in_ email campaign that gives users daily email tips only if they want them. We could then use this flow to reinforce important actions and pro-actively solve any setup support issues. 

## How it would work

We would add an additional CTA to the Welcome emails within Onboarding 5.0, asking users if they want to receive daily tips for their first week in order to help them get setup.

If users click the link they are sent to a subscription confirmation page on posthog.com, and we fire an event to Customer.io. Users who click the link and visit the page are added to a `Daily Tips subscribers` segment. 

Users in this segment then get daily emails for the following eight days. The conversion goal for this flow is `billing product activated` within seven days of opening any email in the campaign. 

Emails in this flow would have to be explicitly labelled in the subject so users recognize them as part of this campaign and know why they are getting additional emails. This is because they may receive the next two onboarding emails within the 8-day daily tips period. 

We'd also create a new subscription topic so that users can opt out of these emails while remaining in the normal onboarding flow. 

## Proposed email content

Every day we'd send one email to users in the segment. Emails would begin the day _after_ a user is signed up to PostHog. 

**Day 1:** How to create actions from the toolbar 
**Day 2:** How to setup a reverse proxy
**Day 3:** Common formulas: bounce rates, DAU, MAU, etc
**Day 4:** Intro to HogQL and time-based breakdowns
**Day 5:** How to create actually useful user surveys
**Day 6:** Integrating replays with Intercom, Zendesk and Sentry
**Day 7:** How to run your first A/B test
**Day 8:** Thanks, invite to betas, link to PMF guide

## Feedback requested

Would love feedback on any common support issues we think we can head off here, especially. 

All other thoughts appreciated too. 


