# Request for comments: Renaming our plans by @raquelmsmith

## Problem statement
_Who are we building for, what are their needs (include example cases), why is this important?_

Plan names are something seemingly minor that actually get used quite a lot, both internally and by our customers (oftentimes just inside their own heads when evaluating). However, our current plan names leave room for confusion:

* The names don't indicate right at the get-go which plan you are probably interested in based on your company size/scale/needs. 
* The names can include information about deployment but lack any real plan name (eg. PostHog Cloud is a paid tier but it's also a deployment)
* We use the term self-serve in the docs but no one really knows what this means

## Proposal
The Cloud environment was new a couple years ago, so it made sense to call the plans on Cloud the "Cloud" plans. However, the Cloud environment is by far the default way to use PostHog now, and calling it Cloud is confusing (I can imagine thinking, "should I be here? or is there a different non-cloud thing I should use?").

So, I'd propose that our Cloud plans are the _default_ plans and are named:

- **Starter**
  - I prefer Starter over "Free" because "free" can get confused with the FOSS version
  - I prefer Starter over "Hobby" because someone might be getting a new startup up and running, and they are very serious about it and working full time on it and are definitely not considering themselves a hobbyist, but this is still the correct plan for them
- **Scale**
  - I think this speaks to our goal to have post-PMF companies who are wanting to grow quickly be on paid plans
- **Enterprise**

Since we are sunsetting the k8s deployment, I don't think we will be selling licenses for self-hosted versions any more, correct? If that is the case then the only other option is the self-hosted docker deployment. We could simply call this the **Open-source** plan.

## Success criteria
_How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?_

I think some internal consensus, along with feedback from customer success, can let us know if this has cleared things up.

## Context
_What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?_

Our competitors generally use some hierarchy of Free/Starter -> Growth -> Enterprise. 

## Design
_What are the key user experience and technical design decisions / trade-offs?_

N/A

## Sprints
_How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?_

- Update pricing tables (should be easy with the work currently going on in Growth)
- Update docs
- Update blog posts, etc
- Where else?
