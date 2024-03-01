# RFC: Project environments, by Michael Matloka

## Problem

Our users commonly operate multiple environments of their product. The two common types of split are:

1. By stage, driven by the engineering workflow, e.g. dev/staging/production.
2. By region, driven by data protection requirements, e.g. country A/country B.

When doing that, customers typically want to share part of the *setup* between environments, while at the same time ensuring *data* from one environment doesn't pollute the others.

The multiple environments use case is one of the reasons why projects are a feature of PostHog, and projects are great for tracking separate products, but not ideal for tracking multiple instances of the same product: whatever you set up for one project, you have to manually replicate in all others.

### Prior art and current partial solution

This problem was explored in the [December 2022 "Feature flag environments" RFC](https://github.com/PostHog/product-internal/blob/main/requests-for-comments/2022-12-08-feature-flag-environments.md#221115---rfc-feature-flag-environments). That work was focused on feature flags, resulting in a feature flags-specific solution: syncing flags between projects, based on the flag key.

Feature flags make cross-project syncing uniquely easy thanks to the feature flag key effectively working as an organization-wide identifier for copying. No other entity – dashboards, insights, etc. – has this advantage of a "key".

Because feature flags use cohorts and actions in their definitions, these are also synced in the current solution – but only based on the cohort/action name, so if the name is changed in one environment, the link silently ceases to exist (on re-sync, a new separate cohort/action copy is created, though the user isn't informed about this).

### What customers are asking for 

#### Tickets

*The initials before the Zendesk ticket signatures are anonymized company names*

- [Sentry-style environments - F ZEN-25](https://posthoghelp.zendesk.com/agent/tickets/25)

	No extra context.

- [Insight & dashboard copying/templates - E ZEN-8327](https://posthoghelp.zendesk.com/agent/tickets/8496)

	They run separate environments per country and currently use projects, copying dashboards manually. They have a set of dashboards that need to be available for all those environments, while a lot of others should just be specific to a single environment. As a large organization with strict privacy requirements, they also need access to each country's production data to be granted individually.

- [Insight & dashboard copying/templates - F ZEN-8496](https://posthoghelp.zendesk.com/agent/tickets/8327)

	They use PostHog to offer analytics to *their* customers. They'd like some dashboards to be copied across projects. Unclear if they should stay synced or separate.

- [Insight & dashboard copying/templates - ? ZEN-2633](https://posthoghelp.zendesk.com/agent/tickets/2633)

	Running staging and production of the same product as separate projects. Would like the setup to just be shared.

- [Insight & dashboard copying/templates - S ZEN-10093](https://posthoghelp.zendesk.com/agent/tickets/10093)

	They run separate environments per country and would like to copy dashboards across projects.

- [Dashboards shared between projects - H ZEN-8724](https://posthoghelp.zendesk.com/agent/tickets/8724)

	They run dev and production separately. The user suggested *sharing* dashboards between projects, rather than copying.

- [Insight & dashboard copying/templates + feature flag copying + sharing event definitions - D ZEN-3433](https://posthoghelp.zendesk.com/agent/tickets/3433)

	A lot of useful feedback, but insights & dashboards take the cake. They want to keep development, staging and production separate, with an easy way to "promote" dashboards/insights to the higher level once ready.

	They also highlighted a few other things as annoying to replicate between projects manually:

	- feature flags
	- actions, specifically webhook message formats
	- some project settings, specifically person display name, webhook integration, group types

	They also noted it's annoying how switching projects from a dashboard takes you back to the homepage. Sadly, this is currently the only thing we can do, because a dashboard is strictly accessible in only one project.

- [Sharing event definitions - P ZEN-5939](https://posthoghelp.zendesk.com/agent/tickets/5939) and [ZEN-5937](https://posthoghelp.zendesk.com/agent/tickets/5937) (two from one org)

	They don't want event and property definitions to get out of sync between staging and production projects, since they intend the shape of data to be the same for all environments. 

- [Sharing event definitions - I ZEN-4637](https://posthoghelp.zendesk.com/agent/tickets/4637)

	Same as above, with a focus on event descriptions.

- [Experiment copying - S ZEN-8439](https://posthoghelp.zendesk.com/agent/tickets/8439)

	They would like to copy experiments between staging and production. This is framed as "would be nice" though.

- [Feature flag copying - O ZEN-10377](https://posthoghelp.zendesk.com/agent/tickets/10377)

	Wanted to copy flags from dev to production.

- [Feature flag copying - K ZEN-10652](https://posthoghelp.zendesk.com/agent/tickets/10652)

	Same as above.

#### Issues

- [Using projects as environments: syncing actions/insights/dashboards/playlists etc. #13418](https://github.com/PostHog/posthog/issues/13418)

	5 users have chimed in to express their need for a better UX around environments, and 11 more have upvoted a comment suggesting handling of "environments". All commenters mention insights and dashboards as a problem. Feature flags and event/property definitions are also mentioned.

- [Copy / Move dashboards across projects #19476](https://github.com/PostHog/posthog/issues/19476)

	3 users commented to express their need, one specifically mentioned dashboards, insights, and actions as currently problematic.

#### User interviews

- [Feature flag & experiment & dashboard copying - R, by Annika](https://posthog.slack.com/archives/C011L071P8U/p1671640760520999)

	They run development and staging, same as customers above they find manual replication tedious and error-prone.

### Summarizing requirements

It looks like **users always need to keep the _data_ isolated** between environments – features: *events*, *people*, *groups*, *recordings*. Data sources and destinations should be environment-specific as well: *data warehouse tables*, *apps*, and *batch exports*.

**Analysis setup should be shareable** – crucially features: *dashboards*, *insights*, *actions*, and *cohorts*. Often also: *event and property definitions*, and *group types*. *Playlists* don't seem to have come up, but should act analoguously to dashboards.
Shareable means that by default all of these entities would be environment-specific by default, but could easily be made synced between environments.

**Shared *feature flags* are also a common ask**, with *experiments* sometimes mentioned. *Early access features* or *surveys* are self-serve feature flags, so should be shareable as well.
Flags are a bit special, because they control your end users' experience. It's typical to want different rollout conditions in dev and different ones in prod, although there is sometimes a common base. To support this workflow, flags must still be able to diverge between environments.

*Annotations* could possibly be scoped by environment _or_ project, but to reduce the scope of work, we can stay at just project scoping for now.

Additionally, select projects settings should be moved into a new environment level of settings – certainly *Toolbar authorized URLs* and *Authorized Domains for Replay* due to varying domains.

Let me know if I missed any other customer context.

## Dismissed solution A: Dashboard/insights/etc. copying across projects

This is what some users have asked for verbatim. What it would probably mean is a feature flags-style "Projects" view on the item level, allowing it to be copied/synced across projects. The reliability of syncing separate instances of a dashboard/insight/action/cohort is key though, and the problem mentioned before is that there's no inherent cross-project ID for entities other than flags. Just basing sync on the entity name would be extremely fragile – there needs to be an underlying link better than that. And then an entity may have many dependencies, e.g. for a dashboard: tiles → insights → actions → cohorts.

#### Why not?

Copying would mean a need to maintain such underlying links (e.g. a cross-project ID, adjacent to the primary key ID) for all entities we'd like to work across projects, which would be dashboards, tiles, insights, actions and cohorts at the minimum. In one way, this could be nice if you'd like to develop, say, a dashboard in the test environment and only copy it to the production environment when final, while still being able to diverge between projects. The major UX tradeoff would be the act of manual syncing, making the experience cumbersome for users at scale. This would work, but it wouldn't be turnkey. And the whole tree of dependencies would be massive in terms of entities in Postgres. Support reports around inconsistencies with dependency syncing would be hard to avoid.

## Dismissed solution B: Dashboards/insights/etc. sharing between projects

What if instead of copying whole trees of entities we shared them directly? This could be even less work for the user – no need to sync manually – and for us – no need to copy dozens of entities to share a single dashboard between two projects. Also no extra work in terms of permissioning or ingestion.

#### Why not?

The UX would still be a little clunky, with a need to manually set up multiple full projects for each environment. It's the taxonomy that's the killer though – shared taxonomy seems to be a relatively common ask in itself, but it's also key for analysis in an implicit way. Group type indexes in particular MUST match exactly, or insights will show results for completely different data. Property types matter – numbers are treated differently from strings, and then there are also dates. A deeper level of integration is needed for this to work reliably.

## Proposed solution: Environments

We can make environments a first-class entity in PostHog – each project would be subdivided into one or more environments. The *data* would be isolated between projects, but the *taxonomy* and *analysis setup* would be shared (see "Summarizing needs" for what should be isolated and what should be shared exactly).

To support the most common workflow of drafting dashboards/etc. in dev/staging and then "promoting" to production, entities such as dashboards would be environment-specific by default, but could be rolled out to the whole project with two clicks. When rolled out, the entity's dependencies would be made project-wide as well (e.g. for a cohort, all actions and other cohorts used in the definition would be instantly made project-wide as well). We'd introduce checks to make sure you can't make an entity environment-specific if it's currently being used project-wide. Sharing in this way would involve no copying though, so there would be no effort needed to sync things.

To support large customers with access control needs, we'd need to add environment-specific project membership to the existing project access control system.

To support the common feature flags workflow of rollout conditions being different between dev and production, we'd make flag rollout groups always environment-specific.

To ensure straightforward yet complete separation of data at the ingestion level (incl. persons), *as well as* steady querying performance, each environment would have its own `team_id`. Due to this requirement, environments would have to be predefined by users and not created at ingestion time, thus each environment would also have its own SDK key (aka "project API key" currently). That seems like a safe choice in terms of the UX.

By default, a project would only have one environment implicitly named "Production". A "New environment" button would allow adding a new one, with the environments feature explained in the modal. The current environment would then be selectable from the project selector, roughly looking like this (courtesy of Cory):

Extra internal requirement: Because Team Pipeline plans to implement multiple SDK keys per project, it would be simplest to move SDK keys to their own model, rather than moving to the environment model.

<img width="939" alt="image" src="https://github.com/PostHog/meta/assets/4550621/ec966c6b-0a1c-4cb0-a593-0234f4807ab1">

### Downsides

Of course, nothing comes for free: adding evironments has its own downsides. Here, it's the amount of effort required – this feature would touch every part of the app, plus ingestion. It would realistically be a few person-sprints of work (as always, it's hard to estimate accurately).

### Pricing

Environments should start out as a paid feature, since they're key for professional users, and more of a nice-to-have for hobbyists. I would suggest it falling under the same restrictions as multiple projects, i.e. multiple environments being available after putting in a card.

### Rough scope of work

1. Backend: We add an `Environment` model (`posthog_environment` table, columns int `id`, `team_id`, `data_id` `name`, `api_token`) and hook it up to a new project-scoped viewset (`/api/projects/:id/environments/`). Every time a new environment is created, we determine its `id` by incrementing the counter of the `posthog_team.id` column and using that value as the environment's `data_id`.
1. Backend: For every project existing currently we create a matching default `Environment`, reusing the project's `id` the environment's `data_id`. Any time a new project is created, it also gets a default environment based on the same logic.
1. Backend: We add an `SDKKey` model (columns UUID `id`, `environment_id`, `value`, `created_by`, `created_at`) and populate it with current `api_token` values from `posthog_team` (for each project using the project ID as `environment_id`).
    > Until we replace the project API key in project settings with environments, we'll have to sync the project API key being reset with its `posthog_sdkkey` equivalent.
1. Backend: We add environments to `TeamSerializer`.
1. Backend: We add `current_environment` to the `User` model.
1. Plugin server: We switch to `posthog_environment` as the source of truth for tokens and event/person `team_id`. From now on, `team_id` in ClickHouse will mean `posthog_environment.data_id`. The only thing we continue processing at the project level is event/property definitions.
1. Frontend, behind a flag: We add logic for environment availability and selection to `teamLogic`.
1. Frontend, behind a flag: We add the environment as a breadcrumb item with a dropdown + creation modal.
    > We must make it crystal clear for customers splitting by region that cross-environment analytics isn't possible – environments are designed specifically to solve cross-contamination.
1. Backend: To models of all optionally project-wide entities, we add boolean fields `environment_id` and `is_project_wide`:
	- `Dashboard`
	- `Insight`
	- `Action`
	- `Cohort`
	- `FeatureFlag`
	- `SessionRecordingPlaylist`
1. Backend: In all endpoints that query ClickHouse, we add support for filtering by environment ID instead of project ID for querying:
	- `InsightViewSet`
	- `QueryViewSet`
	- `ClickhouseGroupsView`
	- `ClickhouseExperimentsViewSet`
	- `PersonViewSet`
	- `SessionRecordingViewSet`
	- `DashboardsViewSet`

	We must ensure cache keys use the environment ID, but besides that no caching changes should be needed.

1. Frontend, behind a flag: We add an "Environments" section to project settings (similar to group types).
1. Backend + frontend, behind a flag: [UX to be specified] We allow feature flag rollout groups to be environment-specific.
1. Frontend, behind a flag: To prevent the UI from being overwhelming, we turn the organization name into just an icon. If the organization owner has an email address from a non-email-provider domain, we use that domains favicon. Otherwise we show a lettermark (no image uploading for now).
  > At this point we can likely release the feature and unflag it.
1. Backend + frontend: [UX to be specified] We add environment-level permissions to the `ExplicitTeamMembership` system.
