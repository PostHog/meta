# RFC: Getting our client libraries up to scratch

## Summary

Eric raised a number of problems around the developer experience for our libraries (listed below). Additionally, Cameron and Simon have mentioned that often customers are requesting feature parity for the libraries that we have. As there’s a large number of languages and spikey amounts of work this is likely a task better suited to a few contractors. We will do a trial with enlisting contractors to first improve the developer experience and get to feature parity. And then setup a process for the contractors to maintain feature parity for the non-primary libraries.

What's the decision: Should we try 2 months of working with contractors on improving the developer experience and getting to feature parity for the libraries. If this works, we will continue to work with the contractors. If not, we will re-evaluate.

## Problems

Developer experience problems:

- Incomplete CI/CD
- Lack of test coverage
- Random legacy dependencies/development environments that don’t work
- Patchy documentation
- “Cross compatibility nuances (kotlin -> java, swift -> objc)”

Custom problems:

- Lack of 1st class support for React and [NextJS](https://github.com/PostHog/posthog-js/issues/492) (very common for high-potential and high-growth customers)
- Missing feature parity for a number of libraries:
    - [https://posthog.com/docs/integrate/libraries](https://posthog.com/docs/integrate/libraries)

## Where we want to get to

- Test coverage, CI/CD, and quality docs across all the repos
- PostHog core team implements the features for the “Primary” libraries (the ones we use ourselves): Javascript, Python and Node.js
- Contractors implement new features and the core team reviews the PRs for the “Secondary” libraries (all the other ones). Aiming to keep feature parity within a few weeks
- Each team is responsible for reviewing their features.
- TBD who reviews the core dependencies? Should we instead go with individuals that have good knowledge of those libraries
- Luke to get the system setup for working with the contractors via a shared slack and github project. Each team as they ship features work directly with the contractors for feature implementation. Luke responsible for making sure the ongoing core work and maintenance happens.

## What does success look like?

- Stable libraries that are kept up to date with the core features
- Good developer experience for the libraries
- Reliable contractors that don't require too much management or excessive PR reviewing
- Not spending too much money

## Open questions

- Which team should be responsible for reviewing the PRs for core features?
- Would we want a contractor implementing autocapture for mobile?
- We seem to have posthog-js-lite and posthog-js, what’s the plan and timeline for these? Do we want contractor support? (This seems more core)
- For test coverage, do we want to aim for a % test coverage? Are we also wanting cypress style tests?
- What profile do we want? Ideally we’d have some overlap of people e.g. 1 person to do all the mobile libraries, and up to 2 people for the server libraries
    - I’d assume we’d want a senior engineer
- How do we want to solve: “Cross compatibility nuances (kotlin -> java, swift -> objc)”? Do we want to build a swift library?
- Eric mentioned that the segment libraries have gone through some larger changes. Is there anything that we can learn from this?
- Are there any libraries that we are missing that we want to add?

## Rough plan

**Owned by Luke**

1. Recruit the contractors: Decide hourly rate, create the JD, share within our next first, and hire the people.
2. Improve the dev experience (make sure they are working, improve test coverage and implement CI/CD)
3. Develop the React and NextJS library (in parallel with below)
4. Identify which the libraries and the feature parity we should do and then get started on them

## Appendix:
- [Library use by customers](https://posthog.slack.com/archives/C034XD440RK/p1676404093156699)