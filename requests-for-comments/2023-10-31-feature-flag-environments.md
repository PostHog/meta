# Request for comments: Feature Flag environments

## Problem statement

_Who are we building for, what are their needs (include example cases), why is this important?_

Users find it difficult to work with Feature Flags across different projects. Currently, to use a flag in a different project, they must create it from scratch and manually copy the configuration.

## Context

_What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?_

See the earlier [RFC Feature flag environments](https://github.com/PostHog/product-internal/blob/main/requests-for-comments/2022-12-08-feature-flag-environments.md#221115---rfc-feature-flag-environments) from @annikaschmid for more context.

## Design

_What are the key user experience and technical design decisions / trade-offs?_

### Option a) Add the ability to copy a feature flag to another project.

#### Pros

-   Our users already use projects as environments and copy their flags manually. This solution would free them from manual work (prone to errors).
-   Easiest to implement

#### Cons

-   Projects are semantically different from environments, which may not align with the needs of larger clients with multiple apps (projects).

### Option b) Introduce environments to Feature Flags only

#### Pros

-   Cleaner separation between projects and environments.
-   Ability to roll out environment to other parts of the app incrementally and as needed.

#### Cons

-   May introduce confusion by having environments for Feature Flags but not other features like Surveys.

### Option C) Introduce environments app-wide (i.e. dashboards, insights, etc.)

#### Pros

-   Cleanest separation of concerns, respects the distinction between projects and environments.

#### Cons

-   Most complex to implement
-   Introduces another layer of complexity where it might not be needed (e.g. dashboards).

Based on Annika's RFC and some preliminary discussion, I recommend implementing **Option a) Add the ability to copy a feature flag to another project.** This is the quickest win which addresses the users' requirements from the [RFC user feedback](https://github.com/PostHog/product-internal/blob/main/requests-for-comments/2022-12-08-feature-flag-environments.md#user-feedback).

## Implementation

In the Feature Flag view page, users will have the option to "Sync" the feature flag to other projects. To do so, they'll click the Sync button, after which a modal pops up. They can choose the project to sync the flag to via a dropdown.

If they select a project where the flag with the identical ID already exists, a warning will show telling them they are about to overwrite the other project's flag configuration.

Depending on whether the flag exists in the other project, we'll either `PATCH` that existing flag, or `POST` a new feature flag. Alternatively, we can can implement a `PUT` endpoint to handle both scenarios.

<img width="1068" alt="Screenshot 2023-10-31 at 09 53 08" src="https://github.com/PostHog/posthog/assets/22996112/432abf9a-b8cc-4d46-a6e6-a5e5ab586b7e">

_Note that the original [RFC](https://github.com/PostHog/product-internal/blob/main/requests-for-comments/2022-12-08-feature-flag-environments.md#user-feedback) also mentions the ability to `move` a flag to another project. I believe we don't need to implement this explicitely. To achieve this, users can simply copy the flag to another environment, and immediately delete the flag in the current environment._

### Nice to have

It would be useful for users to have an overview of projects that use the current flag - otherwise, they would have to switch projects to check whether a flag exists in a particular environment. We can add an overview to the vacant space below the "Insights that use this feature flag" list.

<img width="1166" alt="Screenshot 2023-10-31 at 09 53 21" src="https://github.com/PostHog/posthog/assets/22996112/86df3710-60c5-48e0-b1f7-7ab27b1fd80e">

Alternatively, we can add this overview to the new tab "Projects" within the flag view page. If we decide to do so, it would make sense to also move the Sync button there. Having a separate Projects tab might make sense if we want to add more "project flags" operations in the future. This way, the projects list and the related operations would be nicely grouped under a tab.
