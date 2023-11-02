# Request for comments: Onboarding emails

## Context

We had previously tracked what the email onboarding flow looked like as a series of numbered issues, progressing up to 3.3/3.4. However, this made it difficult for people who don't have access to Customer.io to get a sense of what the onboarding flows looked like - especially because they spread across multiple flows. It also meant the flow was a bit of a hodge-podge patchwork of ideas. 

So, now we have this file. Whenever a change is introduced, we PR and update to this file. In that way, this file always describes what the onboarding flow looks like, changes are easy, and in-line comments are possible. 

It's not the aim of this email to describe the content of every email, but rather to give a functional overview. Screenshots are provided for many, but not all steps. If you want to examine all content, Customer.io is the best place to do it. Ask Joe for access. 

The onboarding email flow is managed by the marketing team, and is implemented with Customer.io - [which does have some issues](https://github.com/PostHog/meta/issues/104).

## Onboarding for US and EU PostHog Cloud
**Campaign in Customer.io:** Onboarding - All cloud users 

The 'full' onboarding experience is delivered across some 'sub' campaigns, below, which trigger based on user properties. This is because we don't want to email users about how to use a feature until we've checked and taken action on whether they are subscribed to other tools.

This campaign triggers when a user completes the `user signed up` event for the first time, on either PostHog US or PostHog EU, provided they also have a valid email address.

1. Wait one hour.
2. Send Welcome Email. 
3. Check users' role in organization, using the `role_at_organization` property.
    1. If `founder` then wait 12 hours, and send Joe's Newsletter Invite email.
        1. Check if user clicked the subscription CTA.
            1. If yes, add `newsletter_cta_clicked: true` to user.
            2. If no, do nothing. 
        2. Wait 12 hours
    2. If `engineering` then wait 12 hours, and send Andy's Newsletter Invite email. 
        1. Check if user clicked the subscription CTA.
            1. If yes, add `newsletter_cta_clicked: true` to user.
            2. If no, do nothing. 
        2. Wait 12 hours
    3. If `product` then wait 12 hours. 
        1. Check if the user has performed `action created`
            1. If yes, do nothing.
            2. If no, send Advice for Product Teams email.
        3. Wait 12 hours.
    4. If `marketing` then wait 12 hours. 
        1. Check if the user has performed `action created`
            1. If yes, do nothing.
            2. If no, send Advice for Marketing Teams email.
        3. Wait 12 hours.
    5. If `sales` then wait 12 hours. 
        1. Check if the user has performed `action created`
            1. If yes, do nothing.
            2. If no, send Advice for Sales Teams email.
        3. Wait 12 hours.
    6. If not `founder` `engineering` `product` `marketing` or `sales` then wait 24 hours
4. Wait until a weekday between 1PM and 5PM in the users' time zone (UTC fallback)
5. Send AARRR intro article email
6. Wait 1 week.
7. Wait until a weekday between 1PM and 5PM in the users' time zone (UTC fallback)
8. Check if the user is in the `Subscribers to Session Replays` segment
    1. If `Yes`, do nothing. (See Session replay onboarding flow)
    2. If `No`, assign to a random cohort branch.
        1. 50% receive Session replay upsell email
        2. 50% recieve Experiment: Personal Invite email
9. Wait 1 week.
10. Wait until a weekday between 1PM and 5PM in the users' time zone (UTC fallback)
11. Check if the user is in the `Subscribers to Feature Flags` segment
    1. If `Yes`, do nothing. (See Feature flag onboarding flow)
    2. If `No`, assign to random cohort branch. 
        1. 50% receive Feature Flag Upsell email 1
        2. 50% recieve Feature Flag Upsell email 2
12. Wait 5 days. 
13. Wait until a weekday between 1PM and 5PM in the users' time zone (UTC fallback)
14. Check if the user is in the `Subscribers to Surveys` segment
    1. If `Yes`, do nothing. (See Survey onboarding flow)
    2. If `No`, send Survey Upsell email
15. Add `completed_onboarding_emails: true` to user. 

### Session replay onboarding flow
**Campaign in Customer.io:** Session replay onboarding

This campaign triggers when a user enters the `Subscribers to Session Replays` segment, provided they also have a valid email address. 

1. Wait until `completed_onboarding_emails: true` for user. 
2. Wait 1 day, and until a weekday between 9AM and 12PM in the users' time zone (UTC fallback)
3. Send Replay Onboarding email
4. Add `replay_onboarding_complete: true` to user. 

### Feature flag onboarding flow
**Campaign in Customer.io:** Feature flag onboarding

This campaign triggers when a user enters the `Subscribers to Feature Flags` segment, provided they also have a valid email address. 

1. Wait until `completed_onboarding_emails: true` for user. 
2. Wait 1 day, and until a weekday between 12PM and 3PM in the users' time zone (UTC fallback)
3. Send Welcome to Flags email
4. Add `flag_onboarding_complete: true` to user.

### Survey onboarding flow
**Campaign in Customer.io:** Surveys onboarding

This campaign triggers when a user enters the `Subscribers to Surveys` segment, provided they also have a valid email address. 

1. Wait until `completed_onboarding_emails: true` for user. 
2. Wait 1 day, and until a weekday between 3PM and 5PM in the users' time zone (UTC fallback)
3. Send Welcome to Surveys email
4. Add `surveys_onboarding_complete: true` to user.

## G2 Review Request Flow
**Campaign in Customer.io:** G2 Review Requester

This campaign exists to invite reviews to our G2 profile. We have limited control over the G2 review process itself and have experimented with lots of different incentives (including merch, charitable donations, and different financial incentives) and approaches in the past. This one works well: We offer the user a $25 gift card for an honest review. Fulfillment is managed through the G2 platform. Need access? Ask Joe. 

The campaign triggers when a user enters the `G2 Review Requests` segment, provided they also have a valid email address, and are not in the `Historic G2 Segment` (from before we moved to Customer.io), or the `Auto unsubscribe segment` (which unsubscribes users who have 2+ emails bounce).

Users enter the `G2 Review Requests` segment if the're `role_at_organization` equals `engineering`, **and** if they fulfill **any** of the following criteria:
    - They have performed `insight analyzed` at least 4 times in the last 30 days
    - They have performed `recording analyzed` at least 5 times in the last 30 days
    - They have performed `feature flag created` at least once in the last 30 days
    - They have performed `experiment launched` at least once in the last 30 days
    - They have performed `survey launched` at least 2 times in the last 30 days

Once the campaign triggers:

1. Wait 3 days
2. Send G2 review request email

## Onboarding for Self-hosted Users
**Campaign in Customer.io:** Onboarding - Self-hosted users

Self-hosted users still trickle in, even though we don't support it anymore. The goal of this campaign is to convert those users to PostHog Cloud for a better experience. 

This campaign triggers when a user enters the `Self-hosted Users` segment, provided they also have a valid email address. 

1. Wait 1 hour. 
2. Send Welcome to Self-hosted email
3. Wait 1 week. 
4. Wait until a weekday between 2:30PM and 4:30PM UTC. 
5. Send Cloud Upsell (Benefits) email. 
6. Wait 2 weeks. 
7. Wait until a weekday between 2:30PM and 4:30PM UTC. 
8. Send Cloud Upsell (Features) email

## Onboarding for Open Source USers
**Campaign in Customer.io:** Onboarding - Open source users

This campaign is similar to the self-hosted onboarding email campaign, but uses different language and tweaks to reflect the open-source experience. 

This campaign triggers when a user enters the `Open source users` segment, provided they also have a valid email address **and** are not in the `All Cloud Users (US + EU)` segment.

1. Wait 1 hour. 
2. Send Welcome Email
3. Wait 2 weeks. 
4. Wait until a weekday between 2:30PM and 4:30PM UTC. 
5. Send Cloud Upsell email. 
6. Wait 2 weeks. 
7. Wait until a weekday between 2:30PM and 4:30PM UTC. 
8. Send Cloud Upsell (Features) email