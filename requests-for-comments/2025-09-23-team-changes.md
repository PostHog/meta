# Request for comments: A better process for making team changes

**Decision maker::** James and Tim / blitzscale team

## Overview

Our ability to make team changes quickly is a powerful advantage, but we've slipped into (I think) some not great habits here that will make us less successful in the long run.

This was less of an issue when we were 50 people, but I don't think this behavior is scaling well as we grow. 

This RFC proposes some changes that will maintain our ability to move quickly, but in a slightly healthier way.

## Problem

Problems with the current way of doing things:

1. **It's almost entirely top down.** These decisions, in my experience, are largely driven by the blitzscale team (RIP exec team) and / or private conversations between individuals. This is great for speed, but not for transparency, and often useful context, perspectives, or alternatives don't get considered.

2. **It's become a hammer and nail situation.** Our default behavior has become "here's a problem, a team change will fix this". Often this works, but not always. A specific example of this is our marketing/brand/website teams, where I think we've spun our wheels for much of this year because we've frequently moved chairs around and got distracted from executing.

3. **It's presented as a fait accompli.** While our Slack announcements have a veneer of "let's talk about this", the reality is these are decisions that are already made. That's fine if everyone agrees, but it sucks to be the one person who doesn't agree / like the change, and Slack isn't a great venue for healthy dissent. It's a synchronous environment that trends toward emotional reaction to news, particularly because the need for a response feels urgent.

4. **It goes against our own values.** We say we [write everything down and default to asynchronous communication](https://posthog.com/handbook/company/culture#we-write-everything-down), but this process is the antithesis of that. I always come back to "writing is thinking" as an important value for us, and the way we do this (on Slack very quickly) doesn't allow for this.

5. **It's destabilizing for newer hires.** I think those of us who have been at PostHog for a long time (I appreciate this is a relative term), underestimate how disorientating this can feel people for who are new-ish to the company, especially if it happens to you more than once in a short period. We shouldn't over optimize for making people feel comfortable and secure, but "this is the PostHog way" isn't an appropriate response, either. 

## Proposed alternative

**Important:** I still believe being able to make team changes in a low friction, responsive way is important and needs to be protected. What I'm proposing adds a tiny bit more friction, but I think it's an acceptable trade-off for making better decisions while still doing so quickly.

- **We treat team changes like proposed new hires.** This means we have a GitHub issue template (not an RFC, which is too slow a process imo) with a clear, consistent structure for proposing team changes. The template should be a forcing function for thinking through the reasons for a change + challenging the proposer on the why of it all.

- **We have a strict time box for making a decision.** I think this should be three days - i.e. proposing a change and officially announcing a final decision in Slack should happen within three days of the issue being created. In that time, the people impacted / who care can respond on the issue to highlight unforeseen problems, suggest alternatives, etc.  

- **Team Blitzscale are the ultimate deciders.** No matter who makes the proposal, ultimately it's up to (in most cases James and / or Tim) to make the call and once they have, we move on and execute that change, even if there isn't complete agreement. A perfect consensus isn't the goal here.

## Benefits of doing this

1. **It addresses most of the problems, while still being fast.** We commit to a quick decision, but it creates time for consideration, and proper thinking before we officially announce a change. 

2. **It encourages bottoms up initiative, autonomy, and transparency.** While in theory anyone can already suggest a team change, in practice I think most people who aren't team leads would feel like this isn't for them. If they did have a strong opinion, they're most likely to bring it up in a 1:1 with their manager, where ideas can easily die before they get a proper hearing. This creates a formal way *anyone* can suggest a change. 