# Request for comments: PostHogDesign System (Ben White)

## Problem statement
*Who are we building for, what are their needs (include example cases), why is this important?*

PostHog is a teenage codebase who has gone through lots of infant phases, stumbling blocks and covering itself in [Ants](https://ant.design) for fun. Now that it is growing up, it wants a wardrobe of clothes that match, a bedroom where it can actually find what it is looking for and for people to recognize it's work as distinctly "PostHog". Anyways enough of this analogy...

To keep our product development pace high whilst also keeping our sprawling frontend maintainable we have agreed that it makes sense to solidify some of the great work already done in standardising our code (Lemon Components, utility classes). To do this we need to agree on
1. What should our Design System cover and what should it not?
2. How do we ensure Designers and Developers talk in the same language?


## Success criteria
*How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?*

Q3 Objective: `50% reduction in custom and inline CSS styles.`

A clear goal is that Designing and Developing the frontend should be faster, perhaps through it being more restrictive. Developers touching custom CSS files should be a rare occurence, both because the components provide all the flexibility required **and** because Designers are also not deviating from the agreed Design System options unless truly necessary.


## Context
*What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?*

> TODO: Link to some good resources on pitfalls of UI library / Design System creation

### LemonUI Components

> TODO: List our current components and any others we agree should be core (✅ - implemented, 🤔 - partially implented, 💡 - not implemented yet)

### Possible rules we could set (remember - this is a proposal to be commented on!)

* Components never layout themselves - no margin or padding
* Components never expose hacks (e.g. style properties)
* Anything that isn't a Lemon Component should be styled only with utility classes

### Technical questions
* Do we continue building this free-form or do we look into using some base Design System library?
* Do we consider something like Tailwind to aid with tooling, autocomplete, Figma-linking etc?
* Do we try and push developers to the right solution (eslint warnings for usage of `style={}`)


## Design 
*What are the key user experience and technical design decisions / trade-offs?*

## Sprints
*How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?*
