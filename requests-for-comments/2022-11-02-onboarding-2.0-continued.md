# Request for comments: Onboarding 2.0 (continued) - Emanuele Capparelli / Raquel Smith

## Problem statement

With this effort we're tackling a couple different problems with onboarding:

1. Current activation rates for organizations are lower than we'd like. This means that too many people sign up for PostHog, but then fail to get set up properly or see value before they get tired and leave.
2. Current activation rates for session recordings (even after successful event ingestion) are lower than we'd like. This means that people get set up, but then don't use the session recording product.

### Hypotheses for some things that could cause this to happen (independently or combined)
- Activation / Onboarding:
  - Signup form asks for lots of information (e.g. Where did you hear about us?) which adds friction
  - If you are a non-technical user, onboarding is a daunting experience, and inviting a team member is not exposed enough
  - Email flows are basic and we could do with better flows to reduce churn during onboarding
  - The onboarding flow doesn't provide value after completion, while it could be kept as reference to specific tutorials or docs in the manual
  - The bookmarklet is currently used to skip the onboarding and skews the metrics
  - The onboarding is pretty boring, we could do with more hedgehogs
- Introducing users to other products:
  - Onboarding flow does not take into account session recording or feature flags, but only event tracking
  - We currently have an activation checklist prompt, but it has performed poorly, and we might want to integrate this directly in the onboarding flow as additional non-blocking steps we suggest users to perform
  - We have developed a product tour but it hasn't been released to production yet, it could be added as a final optional step after the onboarding

## Success criteria
*How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?*

### Metrics
- Improve the organization signup to discovery activation metric by 100% as compared to the current baseline ([see here, internal only](https://app.posthog.com/dashboard/54139))
- Improve the organization session recordings activation metric by 50% as compared to current baseline (see [this insight, internal only](https://app.posthog.com/insights/YdcM8sBK), from Step 2 to Step 3)

### Desired state
- New users are immediately vetted for technical ability (to install the snippet) and, if they are not technical, they invite someone who is
- Email sequences get the right people into the product to support successful onboarding
- Customers are introduced to other products in a secondary product onboarding (eg. Sentry's QuickStart panel)
- Onboarding flow is not visually boring and has more hedgehogs

#### New onboarding flow
- Signup flow is seamless, only email field and SSO are shown to users, any other data capturing happens within the onboarding flow
- At the beginning of the onboarding flow, we should ask the users if they need technical help, and suggest to invite a team member
- At any step during the onboarding, the CTA to invite members should be prominent, to avoid users getting stuck
- The bookmarklet should be removed, and we could add an option to create some basic test events
- Event ingestion should be the first step of a multi-step onboarding flow, as well as the only mandatory one
- After event ingestion is completed, we should show an activation checklist button within the sidebar, to prompt the user to complete optional onboarding steps like creating a dashboard, action, and/or cohort
- Setting up session recordings should be an optional step after event ingestion, with feature flags to be added in a second iteration
- The current activation checklist steps should be refactored in in this new onboarding (invite users to track their marketing websites, invite users to track custom actions, to install apps, and to invite other team members)
- Optional, add the current product tour as a step
- Each step of the onboarding checklist, if completed, should turn into a reference to the relevant content in the manual

#### Emailing
- We should integrate an external tool (e.g. Customer.io) to allow marketing to create and test email flows
- At minimum, email flows should be reworked following suggestions in [the previous RFC](https://github.com/PostHog/meta/issues/64)


## Context
*What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?*

**Onboarding:**

- According to the linked dashboard, those who get their events ingesting properly have a decent rate of activation (see [this insight](https://app.posthog.com/insights/yj7DIktG) and show conversion rate from Step 2 to Step 3)
- People seem to struggle more with getting their events ingested (view above linked insight from Step 1 to Step 2)

**Introducing users to other products (specifically session recording):**

- After successfully ingesting events, activation for the session recording product is lower than we'd like (see [this insight](https://app.posthog.com/insights/YdcM8sBK), from Step 2 to Step 3) 
  - We will look at activation rates only after someone has successfully started ingesting events, since that is a necessary prerequisite and ingestion success is being worked on with the onboarding tasks

Fortunately, there are lots of examples of onboarding on the internet! This is what Sentry currently does:
1. Sentry asks if you need to invite a team member right away.
    - ![image](https://user-images.githubusercontent.com/18598166/199791647-b3af837d-dd53-4bf3-8d1e-4be1139247dd.png)
2. They also let you skip onboarding by importing a sample error.
    - ![image](https://user-images.githubusercontent.com/18598166/199792853-277ad016-2edb-4531-b120-fd6590c62102.png)
3. After initial onboarding, there is a Quick Start that is triggered in the left menu. 
4. The Quick Start panel contains info about the onboarding steps you already completed and more tasks to intro you to other parts of the product.
    - ![image](https://user-images.githubusercontent.com/18598166/199791737-cd5dc488-324a-4cee-8544-e42a9e03235e.png)
5. Interestingly, they gamify it a bit by "locking" some tasks until you complete others.
    - ![image](https://user-images.githubusercontent.com/18598166/199791879-ba9df04a-23b8-481d-a02e-5b2b3d7c5938.png)


## Design 
*What are the key user experience and technical design decisions / trade-offs?*

1. We'll ask people if they need help installing the snippet right away
    - ![Screen Shot 2022-11-02 at 3 51 28 PM](https://user-images.githubusercontent.com/18598166/199792106-33eb8f0b-0f88-458e-b071-5d621f270c36.png)
2. We'll also let people skip onboarding by importing some dummy events for them
    - This is open to discussion. People might not like having dummy data in their project. However, they could always create a clean new project when they are ready to actually start using PostHog with their real data.
3. Every step of the onboarding will give the user another opportunity to invite a team member for help.
    - ![image](https://user-images.githubusercontent.com/18598166/199793207-b8e45d38-b68f-4480-9bfd-c1f091e9dce8.png)
4. After onboarding we'll have a panel similar to Sentry's to allow people to go back to previous onboarding steps, or explore other parts of the product.
5. When clicking on a todo in the sidebar, the relevant instructions can directly be shown in there, e.g. if users click on "Setup Session Recording", we would show the session recording config options directly in the sidebar, and links to the relevant docs in the manual.
6. If users have already performed a task, e.g. "Setup Session Recording", clicking again on the task would show them again the links to the manual / tutorials. 
  

## Tasks
*How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?*

This can be broken down into 3 distinct projects, which can be shipped independently:
1. Redo the existing onboarding to encourage more technical people to help get set up
    - Lead: Raquel
    - [ ] Start tracking signup form usage and attrition
      - [ ] If the above seems to indicate a problem, then A/B test moving the extra fields (Name, where did you hear about us) to a secondary screen in the signup flow, or into the onboarding
    - [ ] A/B test: remove extra signup form fields
    - [ ] A/B test: Add a first step in the onboarding to ask if someone can install the snippet, or if they need help from a team member
    - [ ] A/B Test: Let people skip event ingestion to just explore the product
        - Right now there is a bookmarklet people can use to record some events. This is kind of confusing to use. Maybe we can just create some test events for them?
        - Maybe if someone wants to skip we can jump them over to the demo project? (it seems the demo is currently broken, so that would need to be fixed)
    - [ ] Work with design to visually improve the existing steps
2. Create the "QuickStart" sidebar with additional tasks for people to complete after onboarding
    - Lead: Emanuele
    - [ ] Create a new sidebar container behind a feature flag and add it to the main sidebar
    - [ ] Create a completion indicator to show how many steps have been completed and how many are still to go
    - [ ] Add onboarding tasks that we already track and show their completion in the indicator:
      - Ingesting events
      - Billing info added
      - Team invited
      - Session recording setup
    - [ ] A/B test: Add tasks for creating a dashboard, action, and cohort
    - [ ] A/B test: Add a task for trying out session recordings
      - See if the task actually gets used, and if it actually gets people to successfully use session recordings more than baseline
3. Integrate with Customer.io so marketing/other teams can easily create and send email sequences
    - Lead: Emanuele
    - [ ] Refactor the existing Customer.io app to sync PostHog data
      - Data would be sent to Customer.io using the API, and Customer.io events would be synced back using webhooks to the PostHog API
    - [ ] Move email flows to Customer.io ([everything here](https://github.com/PostHog/meta/issues/64) + the new billing emails, currently unmapped)

