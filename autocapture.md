This is a rough proposal that fits into the category of complex product decisions. I will polish this markdown as a PR.

# Proposal: stop supporting autocapture

We currently support autocapture of front end events.

This helps us with:

* Users can get events in more quickly
* It captures lots of useful pageview data
* Perhaps generates initial sign ups who we retain for _other_ reasons

Hypothesis for why this is hurting us:

Complexity.

* It generates lots of auto labelled events, which will lead to confusing experiences for users or incorrect decision-making
* It doesn't solve the whole problem - you still can't get email addresses for example with it.
  * This makes the solution _more_ confusing: you now need to do manual events AND train users to use autocapture too
* It makes mobile a 2nd class citizen, since it isn't supported with autocapture
* Technical debt maintaining it as a feature

# Should we investigate first?

There will be two outcomes of a completed investigation:

1. Users love it and it helps retention
2. Whilst it may/may not help sign ups, it reduces retention = discard

With 2, we would discard it, so we're left with questioning what would happen with 1. Even if users love it, I think it could be just a substitute for a lack of guidance over which events should come in manually and we should try onboarding people with more guidance on this. It is rational to test this idea out.

# What about the opposite: couldn't we double down on it?

We could go much further with autocapture so it is a complete solution.

We'd need to capture backend events and support mobile. I suspect the reduction in workload setting up events will be reaplced by workloading mapping the underlying database to PostHog. I suspect this will just move the technical work from setting up backend event tracking to tagging what backend events mean in our front end.
