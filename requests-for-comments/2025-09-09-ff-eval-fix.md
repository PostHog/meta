### Customer reach out email

Subject: Feature Flag Evaluation Order Fix - Action May Be Required

Hello,

We're writing to inform you about an important fix we're deploying to our feature flag evaluation system that will affect your current flag configurations.

## What Changed

We discovered and fixed a bug in how feature flags evaluate conditions when variant overrides are present. Previously, conditions with variant overrides were incorrectly being evaluated before conditions without variant overrides, regardless of their defined order. This has now been corrected to respect the order you define in your flag configuration.

## Who Is Affected

We have identified that at least one of your active flags will be affected. To identify potentially affected flags, look for multi-variant flags where:
- Earlier conditions have no variant override (rely on default multivariate distribution)
- Later conditions have explicit variant overrides
- The order of these conditions matters for your use case

Example of an affected configuration:
Condition 1: If country = "United States" → (no variant override, uses default
distribution)
Condition 2: Everyone else → variant = "test"

Previously, Condition 2 would incorrectly be evaluated first. Now conditions are evaluated in the order you defined them (1, then 2).

## What You Need To Do

**Most customers require no action.** The fix ensures flags work as originally intended.

However, please review your affected feature flags if:
1. You noticed unexpected variant assignments in the past and worked around them
2. You have complex flags with multiple conditions mixing variant overrides

## Fix Release Timeline
- **PostHog Servers**: Fix deployment [2025-09-12]
- **Server SDKs if you are using [local-evaluation](https://posthog.com/docs/feature-flags/local-evaluation) (Python, Go, Node, PHP, Ruby, .NET)**: Please check the changelog for the specific version and release date

## Questions or Concerns?

If you have any questions or need assistance reviewing your feature flags, please don't hesitate to reach out to our support team.

We apologize for any confusion this may have caused and appreciate your understanding as we continue to improve PostHog.
