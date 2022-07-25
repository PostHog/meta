# Request for comments: PostHogDesign System (Ben White)

## Problem statement
*Who are we building for, what are their needs (include example cases), why is this important?*

PostHog is a teenage codebase who has gone through lots of infant phases, stumbling blocks and covering itself in [Ants](https://ant.design) for fun. Now that it is growing up, it wants a wardrobe of clothes that match, a bedroom where it can actually find what it is looking for and for people to recognize it's work as distinctly "PostHog". Anyways enough of this analogy...

To keep our product development pace high whilst also keeping our sprawling frontend maintainable we have agreed that it makes sense to solidify some of the great work already done in standardising our code (Lemon Components, utility classes). To do this we need to agree on
1. What should our Design System cover and what should it not?
2. How do we ensure Designers and Developers talk in the same language?



## What IS a Design System?

- Standardised documented Design Language
  - Documentation and guides
  - Codified / visual examples of these standard elements
  - Measured and enforced via code samples and illustations
  - Implementation and Design are using the same words
- Colors / spacing / breakpoints are all defined with a common language
  - Named variables in Design should be completely replicated to the codebase so that we speak the same logic
  - Unique variables (like a special one-off color) should be clearly labelled as unique
- Technical enforcing / encouraging adhering to the Design System
  - Component UI library that only does what the Design System allows
  - Design elements should be as pure and reusable as possible
  - Self-contained and should only layout / worry about itself
- Friction / hard to do something the wrong way **or** Smooth / easy to do things the right way
- Not everything should be in the design system **or** components should be added to the Design System deliberately
- Should allow faster iteration of implementation as both Design and Dev are conforming to the common standards


## Success criteria
*How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?*

Designing and Developing the frontend should be faster, perhaps through it being more restrictive. Developers touching custom CSS files should be a rare occurence, both because the components provide all the flexibility required **and** because Designers are also not deviating from the agreed Design System options unless truly necessary.

* Q3 Objective: `50% reduction in custom and inline CSS styles.` (see metabase graphs for tracking this)
* All elements of the Design System are in use (e.g. no "other" Buttons)
* New engineers should have a frictionless onboarding to the tools and tricks that help them to conform to the Design System
* Designers and Engineers are following the system without needing to be told.
* The Design System and the tools for implementing it should be fully explorable / documented



## What to do

### Design
* Constants - colors / spacing etc. to be codified
* Starting point in Figma containing all components that form LemonUI
  * 
* What missing components do we have that need to be discussed
* Review of the existing "Design System" to ensure it actually follows the rules we have set out
* How to make Storybook the central place
  * e.g. How we use color - in Storybook that links to Figma and vice versa
  * The core documentation and place to be linked to

### LemonUI
* Rework to ensure that styles / classnames can not be added to Lemon components
* Tailwind - evaluate if it or similar libs make sense for our needs
  * Autocompletion of utility classes
  * Easy use of colors

### Tooling
* Linting for rules
  * ERROR - antd components
* PR Bots - "🎉 you used no custom CSS!!"
* Automated testing
  - Cypress accessibility
  - Visual Regression testing

### Communication / Education
* External audiences
  - Open source - other people should be guided to these things as much as internals
  - Bot / PR template to point contributers to the right place
  
### Tidying
* Actually move **everything** to the system
* Remove all AntD elements or at least wrap with Lemon


## Sprints
*How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?*

### Sprint 1
* Evaluate Tailwind (or our utility class approach)
  - Confirm our current multiple build systems is necessary
* Integration of eslint / bot tooling for rules (even if the rules aren't there yet)
* Setup Storybook to be the core documentation point for Design System
* Core components implementation (in Storybook!!) - starting with Form Components


### Sprint 2
* Roll out current System to all of Posthog...
* Remove AntD direct usage behind Lemon components


# References 
> TODO: Link to some good resources on pitfalls of UI library / Design System creation
* https://www.figma.com/file/Y9G24U4r04nEjIDGIEGuKI/PostHog-Design-System-One?node-id=2851%3A4250
* https://github.com/PostHog/posthog/issues/9868
* https://github.com/PostHog/posthog/issues/9867

### Possible rules we could set (remember - this is a proposal to be commented on!)
> Best case we "enforce" or strongly recommend these rules via eslint or otherwise
* Components never layout themselves - no margin or padding
* Components never expose hacks (e.g. style properties)
* Anything that isn't a Lemon Component should be styled only with utility classes
