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

### Summarizing needs

It looks like **users always need to keep the data isolated** between environments – features: *events*, *people*, *groups*, *recordings*. *Toolbar authorized URLs* should also likely be different, due to separate domains.

**Analysis setup should be shared** – crucially features: *dashboards*, *insights*, *actions*, and *cohorts*. Often also: *event and property definitions*, and *group types*. Shared *feature flags* are also a common ask, with *experiments* sometimes mentioned.

I've never heard of the following features mentioned in terms of environments: *playlists*, *surveys*, *early access features*, *annotations*, *data warehouse tables*, *apps*, or *batch exports*. Playlists seem similar enough to dashboards that they should also be shared. As for the rest, it's hard to say.

Additionally, some projects settings could be shared, but others should not be – I'll skip analyzing this here, as it's not a key problem. For now we can assume all to be separate.

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

We can make environments a first-class entity in PostHog. Each project would be subdivided into one or more environments. The *data* would be isolated between projects, but the *taxonomy* and *analysis setup* would be shared (see "Summarizing needs" for what should be isolated and what should be shared exactly).

To support the most common workflow of drafting dashboards/etc. in dev/staging and then "promoting" to production, entities such as dashboards would be environment-specific by default, but could be rolled out to the whole project with two clicks. When rolled out, the entity's dependencies would be made project-wide as well (e.g. for a cohort, all actions and other cohorts used in the definition). This would involve no copying though, so there would be no effort needed to sync things.

To support large customers with access control needs, we'd need to add environment-specific project membership to the existing project access control system.

To support the common feature flags workflow of rollout conditions being different between dev and production, we'd make flag rollout groups always environment-specific.

To ensure straightforward yet complete separation of data at the ingestion level (incl. persons), *as well as* steady querying performance, each environment would have its own `team_id`. Due to this requirement, environments would have to be predefined by users and not created at ingestion time, thus each environment would also have its own SDK key (aka "project API key" currently). That seems like a safe choice in terms of the UX.

By default, a project would only have one environment implicitly named "Production". A "New environment" button would allow adding a new one, with the environments feature explained in the modal. The current environment would then be selectable from the project selector, roughly looking like this (courtesy of Cory):

### Downsides

Of course, nothing comes for free: adding evironments has its own downsides. Here, it's the amount of effort required – this feature would touch every part of the app, plus ingestion. It would realistically be a few person-sprints of work (as always, it's hard to estimate accurately).

### Pricing

Environments should start out as a paid feature, since they're key for professional users, and more of a nice-to-have for hobbyists. I would suggest it falling under the same restrictions as multiple projects, i.e. multiple environments being available after putting in a card.

### Rough scope of work

1. Backend: We add an `Environment` model (`posthog_environment` table, fields `id`, `team`, `data_team_id`, `name`, `api_token`) and hook it up to a new project-scoped viewset (`/api/projects/:id/environments/`). Every time a new environment is created, we increment the counter of the `Team.id` field and use the obtained value as the environment's `data_team_id`.
1. Backend: For every existing project (`Team`) we create a matching default `Environment`, reusing the project's `id` (for `data_team_id`) and `api_token` (for `api_token`). Any time a new project is created, it also gets a default environment based on the same logic.

  > 	A transition period begins at this step. For some time we'll have to ensure `api_token` stays synced between the `Team` (legacy scheme) and its default `Environment` (future scheme). This only means updating the token reset function to update both.

1. Backend: We add environments to `TeamSerializer`.
1. Backend: We add `current_environment` to the `User` model.
1. Plugin server: We switch to `posthog_environment` as the source of truth for tokens and team IDs. The only thing we continue processing at the project level is event/property definitions.
1. Frontend, behind a flag: We add logic for environment availability and selection to `teamLogic`.
1. Frontend, behind a flag: We add the environment as a breadcrumb item with a dropdown.
1. Backend: To models of all optionally project-wide entities, we add boolean fields `environment_id` and `is_project_wide`:
	- `Dashboard`
	- `Insight`
	- `Action`
	- `Cohort`
	- `FeatureFlag`
	- `SessionRecordingPlaylist`
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

  > The transition period ends at this step. We should no longer be using `posthog_team.api_token` anywhere anymore.

1. Backend + frontend, behind a flag: [UX to be specified] We allow feature flag rollout groups to be environment-specific.
1. Frontend, behind a flag: To prevent the UI from being overwhelming, we turn the organization name into just an icon. If the organization owner has an email address from a non-email-provider domain, we use that domains favicon. Otherwise we show a lettermark (no image uploading for now).
  > At this point we can likely release the feature and unflag it.
1. Backend + frontend: [UX to be specified] We add environment-level permissions to the `ExplicitTeamMembership` system.
