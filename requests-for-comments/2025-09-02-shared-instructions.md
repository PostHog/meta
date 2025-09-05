# Request for comments: Shared instructions between onboarding, wizard, and docs, Vincent Ge

## Problem statement

### Maintaining accurate instructions

We have similar content used by our onboarding, wizard, and docs. It's hard to keep three different places in sync and up to date. 

These resources are critical for early stage customers, and we need to ensure they are up to date and easy to understand.

We frequently find the in-app onboarding instructions are [out of date compared to docs](https://posthog.slack.com/archives/C011L071P8U/p1738364865046909). 

It's also confusing when instructions are slightly different between the onboarding instructions and docs. We'd ideally like to see a user be able to jump between the two seamlessly.

These instructions are now important for the new installation wizards owned by the growth team. The wizard should generally try to follow the instructions in the onboarding/docs, so that a user trying PostHog can jump between all three seamlessly.

### Better troubleshooting workflow

Another part of the problem is that only the in-app onboarding instructions have real-time feedback on if the user's on the right track. They do this by showing if a event has been received by PostHog.

We can take this concept further by adding clear checkpoints and troubleshooting instructions to the wizard, docs, and in-app onboarding. 

These instructions will be slightly different between the docs, in-app onboarding, and wizard because they're displayed in different contexts. 

## Proposed solution

For maintainability, we'll store the instructions in a single source of truth in the docs repo. We'll then use this file to generate the instructions for the onboarding, wizard, and docs. 

The onboarding instructions are composed of instruction steps and checkpoints. Onboarding, wizard, and docs will share these instruction steps, but will differ mostly in their checkpoints.

We're looking for a stricter, machine-readable format that we can build linting and tooling around so they can be reliably shared between the onboarding, wizard, and docs. 


## Success criteria

- Single source of truth to update getting started instructions for in-app onboarding, docs, and wizard.
- The shared instructions should be stored in a **highly readable by humans** that's also easily parsable.
- Updating instruction once should trigger updates everywhere.
- Warnings should be given when someone tries to update instructions in the wrong place (not at the source).
- Instructions should be rendered properly by onboarding, docs, and wizard. This should include minor differences, especially at checkpoints.

## Proposed format

We need a markup format that's strict enough to be machine-readable (which means markdown is generally hard to parse), but still readable by humans (XML/JSONs break next lines, yamls break indentation). 

We also need it to be reasonably transportable between different platforms, keep in mind that the website and in-app onboarding will likely continue to use very different components. This means MDX with component extensions are not a good fit.

The exact format is not important, some examples are asciidoc, rst, and toml with structured arrays.

For examples below, we'll use toml with structured arrays as examples, but we can discuss exact formats.

### Example with toml

We can transport our instructions as a structured array of tables.

``````toml
# Front matter
title = "Web error tracking installation"
description = "Step-by-step guide to install PostHog web SDK for error tracking"

# Indication of steps
[[steps]]
title = "Install PostHog web SDK"
badge = "required"

# Indication of step content
[[steps.content]]
type = "markdown"
content = """
## Option 1: Add the JavaScript snippet to your HTML Recommended

```sh
npm install --save posthog-js
```

See how I can just write plain markdown here?

It:
- remains readable
- preserves next lines
- preserves indentation
- preserves code blocks
- preserves links
- preserves images
- preserves lists
- preserves headers
"""

[[steps]]
# ... more steps
``````

This is the equivalent of the following JSON (much less readable):

```json
{
  "description": "Step-by-step guide to install PostHog web SDK for error tracking",
  "title": "Web error tracking installation",
  "steps": [
    {
      "badge": "required",
      "title": "Install PostHog web SDK",
      "content": [
        {
          "content": "## Option 1: Add the JavaScript snippet to your HTML Recommended\n\n```sh\nnpm install --save posthog-js\n``` ... \n",
          "type": "markdown"
        }
      ]
    },
    {
        ... more steps
    }
  ]
}
```

### How will we handle custom components?

With structured data + clean/plain markdown blocks for prose, we can easily render them either as markdown or as components.

``````toml
[[steps]]
title = "Install PostHog web SDK"
badge = "required"

# Indication of step content
[[steps.content]]
type = "markdown"
content = """
## Option 1: Add the JavaScript snippet to your HTML Recommended
"""

# Indication of some custom component
[[steps.content]]
type = "calloutbox"
icon = "IconWarning"
title = "Don't `rm -rf ~/`"
type = "warning"
content = """
You should never run `rm -rf ~/`, it will blow up your computer.
"""
``````

The above will be the equivalent of the following MDX:

``````md
## Option 1: Add the JavaScript snippet to your HTML Recommended

```sh
npm install --save posthog-js
```

<CalloutBox icon="IconWarning" title="Don't `rm -rf ~/`" type="warning">

You should never run `rm -rf ~/`, it will blow up your computer.

</CalloutBox>
``````

### K, but why we can't use MDX

MDX has a few issues:

- MDX is really meant for SSG, which is annoying to make work in the docs and in-app. 
  - Since it's literally using and importing JSX, we'd be maintaining some components in two places.
  - Since we'll also have in-app only and docs-only versions of the components, especially checkpoints, this can be come very painful.
- MDX is whitespace sensitive and forgiving.
  - This means what you type may not exactly work but it won’t crash. It's very hard to confidently say if it'll work in both docs and in-app without building both. This is a non-starter since we're trying to avoid having to update/check multiple places.
  - This also means indented code does not work in MDX, which adds all kinds of annoyance.
  - It also means it's painful to lint. (we can add what ever we want to markdown, it just might not render in one place or another )

We _could_ use MDX, but we'd have to rely on humans being humans being careful instead of enforcing a stricter format.

### How do we render different content for docs/in-app/wizard?

With structured data, we can specify platform specific versions of the content, and have defaults. 

```toml
[[steps]]
title = "Verify PostHog is initialized"
subtitle = "Confirm you can capture events with PostHog"
type = "checkpoint"
badge = "checkpoint"

# Wizard specific instructions
[[steps.content.wizard]]
type = "Instruction"
content = """
Before proceeding, run `curl -X GET 'https://app.posthog.com/api/projects/{project_id}/events?event=test_event&after=2025-09-03T00:00:00Z' -H 'Authorization: Bearer {personal_api_key}'Retry`  
"""

[[steps.content.wizard]]
type = "prompt"
content = """
Before proceeding, run `curl -X GET 'https://app.posthog.com/api/projects/{project_id}/events?event=test_event&after=2025-09-03T00:00:00Z' -H 'Authorization: Bearer {personal_api_key}'Retry`  
"""

# Show if event is received in app
[[steps.content.app]]
type = "live"
event = "SOME_EVENT_TO_AWAIT"

# Catch all
[[steps.content]]
type = "markdown"
content = """
Before proceeding, confirm that you can capture events with PostHog using `posthog.capture('test_event')`.
"""
```

### Who will own these?

I think it makes sense for docs to largely own these instructions.

- We already regularly update the docs.
- It'll be in the docs repo.
- A single clear owner will be better than multiple owners.

### How will updates make it to the onboarding, wizard, and docs?

After updating the shared reference instructions in docs, we'll need to propagate the changes to the onboarding, wizard, and docs.

There are options:

- Before build through GitHub Actions and automated PRs triggered when any of the refs are updated. 
- During build by fetching/caching from `posthog/posthog.com/master`. Less manual, requires redeploy to update, could break/block people's builds. (This is how the website builds API docs right now.)
- At run time. Always up to date. Could break without us knowing.

I lean towards fetching and caching the latest known version in each repo and building the page from the reference material. We can minimize the chance of a bad change in docs breaking/blocking people's builds by enforcing reasonably strict linting.

## Sprints

TBD. Likely planned as a goal for Q4.
