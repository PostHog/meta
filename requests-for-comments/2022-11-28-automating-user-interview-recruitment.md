# Request for comments: Automating user interview recruitment

- Owner: Luke Harries
- Deadline: Friday 2nd December 2022
- Feedback I'm looking for: 
  - What's the best way to build this? (I'm looking to do this quick and scrappy)
  - Am I overlooking anything?
- Who'll build it: Likely Luke for MVP (unless an engineer is very keen and has bandwidth)

## Summary

As a PM at PostHog, I want to be able to quickly invite a desired number of customers to a user interview either one-off or on a regular basis. I should be able to filter by the users properties and events. It should be in-app and not feel spammy.

## Problem statement
*Who are we building for, what are their needs (include example cases), why is this important?*

Current situation:

- Recruiting for user interviews at PostHog is done manually.
- The process:
  1. User selection: Find the customers that we want to speak to
       - Use metabase and the revenue server to find High ICP customers. Create a cohort on PostHog. Filter for specific user behavior using insights/trends/retention. Export a CSV of emails
       - Example: product engineer at high growth startup who has created 2+ feature flags more than twice in the last 30 days and hasn't done an interview before
  2. User recruitment: Mass send emails to participants with a calendly link for them to book in (Luke uses Yet Another Mail Merge from a google sheet)
  3. Interview booking: When customers book in, add other teammates to the calendar invite and move the event to the "User interviews" shared calendar.
  4. Follow up with a PostHog merch voucher

- Problems:
  - Primary
    -  It's time intensive (taking ~90 minutes per batch of interviews)
    - It's sometimes seen as spammy (users are getting emails that they don't want)
  - Secondary
    - It's done in bursts of interviews rather than continuous interviewing due to the time required.
    - Accidentally inviting the same user to interview multiple times
    - Engineers don't often book their own interviews due to the time/effort required
    - We often forget to send the voucher
    - Overbooking/underbooking the desired number of interviews

Helping customers use and manage user interviews is likely a feature that should be part of PostHog in future (but doesn't need to be a product focus just yet)

## Success criteria
*How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?*

1. Improve the system for user interviewing at PostHog:
   1. Reduce the time and friction required for booking interviews
   2. Make being invited not feel spammy (likely inviting by in-app rather than email)
   3. Can reliably book in users for interviews (customers click the link)
   4. (Future work) Make it self-serve for the engineers to organize their own interviews
2. Should be quick to build launch and test this
3. (Future work) Launch an MVP for user interviewing system to other companies

## High-level solution

Pre-requesites

- [ ] Connect clearbit and ICP score to PostHog

User selection

- Specify the following:
  - The person/cohort events and properties to filter on
    - Example:
      - user is in our ICP
      - AND has created more than 2 feature flags in the last month
      - AND doesn't have the event "Seen user interview popup" over the last 3 months
  - The desired number of interviews as an absolute amount or on a regular timeframe
    - Example:
      - recruit 5 users OR recruiter 2 users every week

User recruitment

- Potential options:
  - Use Em's customer.io popup
  - Develop a site app for the popup with the invitation message. Control the popup with feature flags
    - Once the desired number of interviews has been booked in, the popup is turned off
    - Nice to have:
      - Control the page that the popup shows on
      - Have the popup show up directly after an event
    - *This is the solution that I'm leaning towards - seems quick and could also test with external users*
  - Custom feature:
    - Build an add-on feature to cohorts
    - 1st class user interview product in PostHog

Interview booking

*Done for product analytics*
- [x] Setup Calendly link
- [x] Create zapier to add other teammates to the calendar invite (done for product analytics)
- [x] Create zapier to add the calendar event to the "User interviews" calendar (done for product analytics)

Follow up

- Require a human trigger to ensure the person showed up
- Potential options:
  - Easy to use email template and a message sent to the interview organizer on slack as a reminder
  - One-click send shopify voucher and thank you

### Solutions Requirements
- Can make more than one user interview recruitment

## Quality bar
*What quality are we aiming for? Is this a v0.1 that we are going to test with a small number of users, or is this a change to core functionality that needs to work perfectly?*

This is a v0.1. Internal use for PostHog with the potential to expand into a product for others to use if it's effective at user recruitment.

From the user side it should meet the quality bar of the general product.

## Context
*What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?*

- Other options I've considered
  - Continuing with manual email outreach
  - User interview recruitment platforms (had a brief look and haven't found anything plus we'll likely want to have this native to PostHog)

## TODOs

- [ ] Sketch of the interface for the product engineer
  - Invitation text I can customise (what feature I look to get feedback on, what do I offer them (voucher)
  - Calendly URL I can customise, that is either in the text or as a button "book now"
  - 'Remind me later' button in popup
  - Nice to have's:
- [ ] Sketch of the popup for the user
- [ ] Outline of tech details
  - [ ] 'Preview mode' where I can see how the popup would look like for the user (to see how long the text is, to test the link?
  - [ ] Events the popup sends - Analytics (How often was the popup shown/link clicked/dismissed)
  - [ ] Investigate "what website & docs have built for applications and direct users to a page on posthog.com/Shopify directly"