# Request for comments: Cohort Local evaluation in feature flags

## Problem statement
*Who are we building for, what are their needs (include example cases), why is this important?*

Right now, we have several restrictions on cohort local evaluation for flags because of the way flags work: Flags don't support flexible nesting of filters, so you
can't AND-OR things like you can in a cohort definition. So, we can't represent all cohorts as flag conditions, and thus can't evaluate them locally.

This makes it pretty hard for people to judge whether their flags that use cohorts will be locally evaluated or not.

As such, we want to:

(1) Explain clearly when a flag with cohorts wouldn't be locally evaluated.
(2) or, evaluate all cohorts sans restrictions for local evaluation.

(2) sparks a lot more joy, so this RFC is majorly about how to do (2). I mention (1) for completeness sake, that it's a valid option too.


## Context
*What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?*

We haven't done (2) before, because it's a lot more effort than restricting cohort evaluations, and at the time, we weren't fully focused on flags,
so it made a lot of sense to unlock the main use-cases (80/20 it), vs. solving for all.

Open question: Is now the time to invest fully in (2)? Based on the very limited requests for this, I'd say no. Or, not for all libraries atleast.

## Design 
*What are the key user experience and technical design decisions / trade-offs?*

There are two main approaches to evaluate all cohorts locally:

1. Make feature flag conditions more expressive, to support AND-ORs, just like in cohorts (i.e. use property groups to express any arbitrary ANDing and ORing).
2. Special-case more expressive filters only for local evaluation.

On the server side, we don't face the problem of expressive filters since we generate SQL, which can represent any arbitrary AND-ORing.

(1) is significantly more effort than (2), and given the open question above, I'm not very keen on it. The rewards don't justify the cost.

Rather, I'd propose we do (2), and only for `python` and `node` for now. The way it works is:

1. We create a special property type: `property_group_type` only for local evaluation, which is a property group of person properties.
2. We populate the above at `/local_evaluation` call time: it converts all cohorts into `property_group_type` filters.
3. In the SDKs, we implement AND-OR filtering on the property group type. This is relatively simple, because we're only dealing with person properties,
    and we already have the code in place for matching a single property.



## Sprints
*How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?*
