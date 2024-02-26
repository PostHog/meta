# Request for comments: Project environments, by Michael Matloka
## Context


Our users commonly operate multiple environments of their product. The two common types of split by environment are:

1. By stage, driven by the engineering workflow, e.g. dev/staging/production.
2. By region, driven by data protection requirements, e.g. country A/country B.

When doing that, customers typically want to share setup between environments, while ensuring *data* from one environment doesn't pollute the others.

The multiple environments use case is one of the reasons why projects are a feature of PostHog, and projects are great for tracking separate products, but not ideal for tracking multiple instances of the same product: whatever you set up for one project, you have to manually replicate in all others.

We currently solve this in Feature flags with a feature for syncing flags between projects, based on the flag key. Feature flags make this uniquely easy thanks to the feature flag key being an effective organization-wide identifier. No other entity – dashboards, insights, etc. – has this advantage though.

Because feature flags use cohorts and actions in their definitions, these are also synced in the current solution –but only based on the cohort/action name, so if the name is changed in one environment, the link is silently broken.


Previous context on this topic: [the December 2022 "Feature flag environments" RFC](https://github.com/PostHog/product-internal/blob/main/requests-for-comments/2022-12-08-feature-flag-environments.md#221115---rfc-feature-flag-environments)

## Expectations

### Isolated

The key feature is separation of all of *data*:

- Events
- People
- Groups
- Recordings

Pipeline setup should probably be separate too:

- Apps
- Batch exports

Domain-specific configuration is also going to differ between environments likely:

- Toolbar authorized URLs

### Shared

Sharing of these entities is commonly requested:

- Dashboards
- Insights
- Actions
- Cohorts
- Feature flags

Not mentioned typically, but logically also shared should be:

- Event and property definitions
- Group types
- Annotations
- Playlists
- Experiments
- Surveys
- Early access features
- Project settings

The needs are unclear for:

- Data warehouse tables

## Rejected solution: Cross-project copying

This is what some users have asked for verbatim. What it would probably mean is a feature flags-style "Projects" view on the item level, allowing an item to be copied/synced across projects. The reliability of syncing is key though, and the problem mentioned before is that there's no inherent cross-project ID for this. Just basing on the entity name would be extremely fragile – there needs to be an underlying link better than that.

This means we'd need to maintain such underlying links (e.g. a cross-project ID, adjacent to the primary key ID) for all entities we'd like to work cross-project. In one way, this could be nice if you'd like to develop, say, a dashboard in the test environment and only copy it to the production environment when final. Divergence becomes easy. But there would be an act of manual syncing, making the experience cumbersome for users at scale. This would work, but it wouldn't be turnkey.

[insert metric for how often flags differ within org]

## Proposed solution: Environments

I believe to solve this problem once and for all, we need environments to be a first-class feature, specifically: subdivisions of a project.

#### What does this mean in terms of data storage?

I see two ways to go about making the distinction between environments on the ClickHouse level :

1. Based on an event/person/group property, e.g. `$environment`
2. Based on the `team_id`

Unfortunately without rewriting the events table for a new sorting key that'd include the environment column, only the second option stands. That's because, when aiming to isolate data, we must ensure the isolating column is included in the MergeTree partitioning key and/or sorting key. Ignoring that is a surefire way to slow down queries within the whole project and consequently, for all users of the PostHog instance.

Can we rewrite the events table? It would be a major effort, but may be worth it if there are other tweaks in its definition we could make at the same time.

Let's assume we don't rewrite the events table: Because our data is already sorted in a `team_id`-first way, by reusing `team_id` we get performance for free. In this scenario, for each new environment we just increment the `posthog_team.id` counter without creating a `posthog_team` row. We do, however, create a `posthog_environment` row storing the `team_id` and other metadata, such as environment name.

#### What does this mean in terms of ingestion and SDKs?

We can't rely just on a property for storing the data, but… maybe we could still use it in ingestion and therefore in our SDKs? It doesn't look like we can: the final `team_id` must be known in the plugin server during event processing, and at the same time async operations during event processing are banned – so environments have to be defined manually in the UI beforehand. How would we then handle events with an `$environment` value not matching an already-defined environment? We could force there always being a default catch-all environment, but that feels dangerous. Otherwise such events would be swalled by a black hole.

A safer path is creating a project API key for each environment. We will use all of the existing API key caching mechanisms already present in the plugin server, the only difference being that the `posthog_environment` table becomes the source of truth instead of `posthog_team`. And as a customer, it's much harder to accidentally go off the trail.

#### What does this mean in terms of the UI?

By default, a project will only have one environment implicitly named "Production". A "New environment" button will allow adding a new one, with the environments feature explained in the modal.

The current environment will then be selectable from the project selector, roughly looking like this:

<img>

### Scope of work

1. Backend: We add an `Environment` model (`posthog_environment` table, fields `id`, `team`, `data_team_id`, `name`, `api_token`) and hook it up to a new project-scoped viewset (`/api/projects/:id/environments/`). Every time a new environment is created, we increment the counter of the `Team.id` field and use the obtained value as the environment's `data_team_id`.
1. Backend: For every existing project (`Team`) we create a matching default `Environment`, reusing the project's `id` (for `data_team_id`) and `api_token` (for `api_token`). Any time a new project is created, it also gets a default environment based on the same logic.
  > A transition period begins at this step. For some time we'll have to ensure `api_token` stays synced between the `Team` (legacy scheme) and its default `Environment` (future scheme). This just means updating the token reset function to update both.
1. Backend: We add environments to `TeamSerializer`.
1. Backend: We add `current_environment` to the `User` model.
1. Plugin server: We switch to `posthog_environment` as the source of truth for tokens.
  > The transition period ends at this step. We should no longer be using `posthog_team.api_token` anywhere anymore.
1. Frontend, behind a flag: We add logic for environment availability and selection to `teamLogic`.
1. Frontend, behind a flag: We add the environment as a breadcrumb item with a dropdown.
1. Backend: In all endpoints that query ClickHouse, we add support specifying the environment-specific team ID for querying:
  - `InsightViewSet`
  - `QueryViewSet`
  - `ClickhouseGroupsView`
  - `ClickhouseExperimentsViewSet`
  - `PersonViewSet`
  - `SessionRecordingViewSet`
  - `DashboardsViewSet` (only by passing the `data_team_id` to `InsightSerializer`s)
  We must ensure cache keys use the environment-specific team ID, but besides that no caching changes should be needed.

1. Frontend, behind a flag: We add an "Environments" section to project settings (similar to group types).
1. Frontend, behind a flag: To prevent the UI from being overwhelming, we turn the organization name into just an icon. If the organization owner has an email address from a non-email-provider domain, we use that domains favicon. Otherwise we show a lettermark (no image uploading for now).
1. Backend + frontend behind a flag: We make environment choice explicit in insight & dashboard sharing (`SharingConfiguration` & `SharingViewerPageViewSet`).
1. We unflag environments.

