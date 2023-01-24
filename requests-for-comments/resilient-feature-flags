# Request for comments: Resilient feature flags

## Problem statement
*Who are we building for, what are their needs (include example cases), why is this important?*

We've recently realised a few limitations with feature flags when PostHog goes down. This isn't great for permanent feature flags as it causes things to break
in unexpected ways & forces clients to think really hard about defaults & write code defensively.

Ideally, our client libraries and APIs should 'just work', even if there's no error handling on the user's side.

So, how do we make this happen?

## Context

The (simplified) way things currently work is: 

Whenever a user shows up on a page, we send a request to `/decide` with their randomly generated anonymous ID to parse feature flags. Usually these people don't have
properties attached to them, so only flags that don't depend on user properties resolve to `true` (assuming they're within the rollout bound).

Things change when we call `identify()` on a user, as this updates their ID, and fires another request to `/decide` to get new flags based on not only current
user properties, but any past properties that were set for this user.

We need the database to:
(a) Validate project tokens and find relevant team
(b) Get feature flag definitions for the given team
(c) Evaluate above flags using the person's properties stored in the DB

Note that `decide` can run on any arbitrary anonymous ID.

Caching decide responses doesn't make too much sense, since it's called usually only once per distinct ID per session. Every anonymous user has a new ID, and except
for users coming back to the app again & again on the same day, we don't achieve resiliency as we can't cache for users never seen before, nor can we assume that properties
haven't changed; while introducing new problems like cache invalidation.

### Possible failure modes

1. The database is down/unreachable: `decide` evaluation fails and returns 500
2. The servers are down/unreachable: requests from client libraries time out / error out.

(2) seems hard to defend against without a distributed app distribution (and then a resolver to go to the 'correct' app), but (1) is possible.

## Solution

Introducing, decide v3.

This is a good opportunity to fix other customer frustrations around this endpoint, and pave the way for breaking changes we've wanted to introduce for a while.

In short:

1. Decide v3 returns all flags, with a `true` or `false` based on whether they're enabled or not, vs. implicitly just not returning the false flags.
2. Decide v3 does a best effort calculation of flags when PostHog DBs are down, and tells the client that it faced issues computing all flags.
3. Decide v3 can return a JSON payload with the flag variant (sneaking this into the major version API update)

On the SDK side:

1. We stop obliterating all flags if decide returns a 500, and based on the parameter in (2), update only flags that were computed again, while keeping old values of other flags.
2. We add timeouts to flushes & shutdowns

To make this possible, we cache flag definitions (and not the decide responses) on our side, so these are available even if the DB is down. We need to do the
same for determining the team given a project API key.

It makes sense to include (3) with (1) for decide because the shape of the response is changing anyways, and it's much easier to handle both at the same time in all client libraries, than doing these one by one.

We don't have to build the rest of it out right now, but having this functionality here means when do build it, it will be extraordinarily faster to do.

## Is it worth doing this?

If PostHog never goes down, none of this gives us an advantage (except maybe speeding up `/decide` responses if we decide to put the cache in front).

However, for the times we do go down, doing this ensures that our customers don't have a terrible experience & don't have their apps break. It's a trust-building
activity that ensures customers stick around.

It's an extremely big churn risk. If PostHog going down makes their app go down, we become an extra dependency for them being up.

## Context
*What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?*

## Design 
*What are the key user experience and technical design decisions / trade-offs?*

## Sprints
*How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?*
