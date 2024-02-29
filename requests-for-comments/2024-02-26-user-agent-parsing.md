# Request for comments: User Agent Parsing & Paul D'Ambra

## Decision

Option 3 - extend the user agent parser so it matches (or betters) the JS SDKs behaviour, enable it for everyone, and deprecate from the SDKs

(doing nothing is a fine holding position so this isn't urgent)

## Problem statement

_Who are we building for, what are their needs (include example cases), why is this important?_

-   User agent parsing is ~3% of the posthog-js bundle and needs increasing code golf to keep under control.
-   It is also duplicated in posthog-js-lite and needs people to remember to update one when they change the other
-   Because it is in the SDK other SDKs cannot take advantage of it and so we have things like the user agent populator plugin
-   Any changes are slow to roll out because of the long-tail of SDK versions in the wild

## Success criteria

_How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?_

-   The posthog-js bundle size is reduced by 3% or more
-   The user agent parsing is available universally across all SDKs

## Design

_What are the key user experience and technical design decisions / trade-offs?_

### Option 1 - do nothing

It's been fine for 2 years, we copy the changes from https://github.com/PostHog/posthog-js/pull/1038 to posthog-js-lite and move on.
Fast, but nothing is easier next time we need to change this

### Option 2 - move the code into the plugin server

e.g. in this location https://github.com/PostHog/posthog/pull/20548 (but not that implementation)

We move the code as-is from posthog-js into the plugin server so that any event with `$raw_user_agent` has the various fields parsed out into `$device_type` and `$os` etc.
Then incrementatlly we can remove the code from posthog-js and posthog-js-lite and just use the plugin server to populate the fields.

This doesn't resolve the user_agent_populator plugin, but it does mean that we can remove the code from posthog-js and posthog-js-lite and it is available to all SDKs.

### Option 3 - make the user_agent_populator plugin universal (like GeoIP)

We would extend the [user agent populator plugin](https://posthog.com/docs/cdp/user-agent-populator) so that it handles its existing case and this new one safely. And treat it like GeoIP, enabling it by default and allowing it to be turned off.

This means we don't need to move the code into the plugin server. And lets users opt-out if they want to.
