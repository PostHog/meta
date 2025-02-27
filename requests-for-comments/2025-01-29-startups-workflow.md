# Request for comments: Startups workflow streamlining

## Problem statement
We need to streamline the startup application and approval workflow. It’s a multidisciplinary job and we need some hands who good at integrations (in between Zapier, Stripe, Vitally, Slack and PostHog-the-app and PostHog.com). Doing this will serve a core audience better, increase our speed.

Right now, when startups and YC members apply for our programs and deals, it's a manual process that results in friction, delay, data loss, missed opportunities, bad reporting and general inefficiency.

## What's wrong with current flow:
- **It's slow for founders:** Manual effort, email-based (which we hate), asking for duplicate info, all of which gets in the way of their JTBD (shipping). 
- **It's slow for us:** This program matters to PostHog, and this is unnecessary friction. (Like, poor Scott. There's at least 30 of these a day.)

## How the current flow works:
1. People fill out the forms on https://posthog.com/startups 
2. An email (via Zap) goes out from Scott Lewis, asking participants to manually submit their info
3. Participants manually submit, over email, the date they were founded, their project ID, last 4 digits of their CC, amount they've fundraised, the email address associated with their PostHog account
4. Over email, Scott receives this, manually reviews the info across Stripe and Vitally to see if they are eligible, and to check whether they have the credits or not.
5. He then presses Approve in Zapier (which applies the credits in Stripe, and triggers the next workflow of emailing the person with their update + merch stuff)

## What can we do about it? 
- **Steps 2 and 3,** above, should not have to happen. (We already have a record of *most* of the info, because a PostHog account is required to apply.)
- I think this means the actual **sign-up form should be gated**, accessible from the app itself — requiring a user to be in a logged-in state to submit. 
- I think the system should **auto-filter applications:** if they are not eligible, do not apply the credits. *(We can write nice rejections: “Did we get that wrong? Contact Scott if you think there’s been a mistake. We like being generous, but sometimes the our auto-bots are fierce defenders of the realm.”)*
- I think the system should **auto-approve people:** if they pass the above criteria check, the credits should simply be applied. We can be alerted to abuses if needed, starting with daily manual reviews. 

## Proposed flow:
 * Remove the actual application form fields (just the form fields) from [the page](https://posthog.com/startups/apply)
* Replace it with a call-to-action to “Sign in or sign up to apply”
* Use PostHog-the-app to filter by whether a user has clicked that or not, and show a banner that says “🎉 You’re eligible for the startup plan! Get started here.”
* Build a new page on the app: /project/###/startups
* Host the completion steps of the form here. 
* * All we need to ask for is what we don't already know. I think that's just "How much in total funding have you raised (USD)" and "The date that your company was incorporated."
* I get a little lost in the plumbing after this and am asking for support thinking through the path.

## Three other considerations:
* **Solving for the YC program too:** It needs similar care, as the form at [yc-onboarding](https://posthog.com/yc-onboarding) generates all the same challenges. It has the same issue of manual back and forth that causes too much friction. The same issues apply, and I think a similar solution can be considered.
* **Solving for other deals:** Besides the YC deal, PostHog will continue to set up other deals with other accelerators / VC programs. It would be smart to think of this as a workflow we can replicate and edit the minor details for emerging deals.
* **Solving for internal reporting:** Because so many systems are talking to one another here, there is a lot of manual reporting between Zapier tables, Stripe data, Salesforce, Vitally and PostHog to get a source-of-truth view of how our startups programs and YC adoption is doing. I think if we can find ways to let the systems do the talking, we should hopefully result naturally in cleaner data, but if we can undertake this job with this in mind, we'd be even better off. 