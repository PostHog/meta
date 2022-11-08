# Request for comments: Data Exploration - Marius Andra

This is a proposal to change the way we query for data in PostHog, and how we display the results.

## Why?

With PostHog, users own their data. Yet when exploring it, they're limited to the tools we've built. Sometimes they want more:

- Insight person modals limit you to looking at a list of people. You can't drill them further (e.g. add a filter, break down by cohorts, etc).
- The live events table supports event properties as columns, but that's it. What about person properties? What about aggregations? Sorting is not supported.
- You can't have insights with non-event data sources (_persons_, event names?), even though some chart types could support them (e.g. pie chart of all persons in the system broken down by country)
- Persons tables are very basic. No column customisation. No math or formulas.
- There are charts in insights that can't be added to dashboards (e.g. retention line chart, funnel correlation results).
- The listing pages (events, persons, etc) support filters, but those can't be saved.
- There are parameters you can't directly filter for or list (such as `distinct_id`, `timestamp`).
- Our `FilterType` has been append-only for 2+ years and is long overdue for a serious cleanup.
- Multiple endpoints accept various types of filters with various capabilities (`/insights/funnel/correlations` vs `/insights/trend/` vs `/events` vs `/persons/trends`). They're all typed by this same object. It's impossible to type the outputs without blindly guessing.

We have an opportunity to build a unified "data exploration" view, which would solve all or most of the above, and open the floodgates to allowing any type of analysis.

The proposed changes below will and also unblock:

- Embed anything on any dashboard (e.g. list of live events or cohort persons)
- Embed anything anywhere else (insights, event lists, etc, all with an easy API)
- Every query and table can have a unique URL.
- Build a fluid interface that can morph between different views or chart types
- Unblock visualisation apps (e.g. the scatterplot plugin, world map v2.0)
- Let us embed data from sources other than clickhouse onto dashboards
- Using any other query engine as our backend

## Analysis of the backend

We capture a stream of events with all sorts of properties:

```js
// this is pseudocode, only for illustration
event = {
    event: 'event name',
    distinct_id: 'unique id',
    ip: '127.0.0.1',
    timestamp: string,
    properties: {
        $withdollar: 'posthog properties',
        '$feature/stuff': 'feature flags',
        'anything else': 'custom properties',
    },
    person: { properties: {} },
    group1: { properties: {} },
}
```

We store this stream of events in a table in ClickHouse, and let users analyse it with bespoke tools like "trends", "retention", "persons" or "live events".

For example, you can:

- Through "live events"
  - List all latest events and show their properties
  - Filter this list by any event, person, or group property
  - Filter this list by event name, person_id, distinct_id, group ids
  - Select custom properties to fetch
- Through an "insight"
  - Aggregate these event lists into time buckets (number of events per day)
  - Support this data with aggregate math (`count(distinct distinct_id)`, `count(*)`, `avg(count(unique property))`)
  - Combine multiple columns with formulas
  - Breakdown by properties (`$browser`)
  - Manipulate these events further with custom queries to generate bespoke graphs (`funnel`, `retention`, `paths`)
- Through the insight persons modal
  - Remove the aggregation (e.g. remove the count by day, and focus on the events of one day)
  - Get the actual persons behind the list.
  - On a funnel, get the success/dropoff for each step.
- Through persons/cohorts
  - Show people who have done various events any number of times
  - Show people whose properties match certain values

All these tools operate by passing to various API endpoints a HUGE object of type `FilterType`:

```ts
export interface FilterType {
    insight?: InsightType
    display?: ChartDisplayType
    interval?: IntervalType

    // Specifies that we want to smooth the aggregation over the specified
    // number of intervals, e.g. for a day interval, we may want to smooth over
    // 7 days to remove weekly variation. Smoothing is performed as a moving average.
    smoothing_intervals?: number
    date_from?: string | null
    date_to?: string | null
    properties?: AnyPropertyFilter[] | PropertyGroupFilter
    events?: Record<string, any>[]
    event?: string // specify one event
    actions?: Record<string, any>[]
    breakdown_type?: BreakdownType | null
    breakdown?: BreakdownKeyType
    breakdowns?: Breakdown[]
    breakdown_value?: string | number
    breakdown_group_type_index?: number | null
    shown_as?: ShownAsValue
    session?: string
    period?: string

    retention_type?: RetentionType
    retention_reference?: 'total' | 'previous' // retention wrt cohort size or previous period
    total_intervals?: number // retention total intervals
    new_entity?: Record<string, any>[]
    returning_entity?: Record<string, any>
    target_entity?: Record<string, any>
    path_type?: PathType
    include_event_types?: PathType[]
    start_point?: string
    end_point?: string
    path_groupings?: string[]
    stickiness_days?: number
    type?: EntityType
    entity_id?: string | number
    entity_type?: EntityType
    entity_math?: string
    people_day?: any
    people_action?: any
    formula?: any
    filter_test_accounts?: boolean
    from_dashboard?: boolean | number
    layout?: FunnelLayout // used only for funnels
    funnel_step?: number
    entrance_period_start?: string // this and drop_off is used for funnels time conversion date for the persons modal
    drop_off?: boolean
    funnel_viz_type?: FunnelVizType // parameter sent to funnels API for time conversion code path
    funnel_from_step?: number // used in time to convert: initial step index to compute time to convert
    funnel_to_step?: number // used in time to convert: ending step index to compute time to convert
    funnel_step_breakdown?: string | number[] | number | null // used in steps breakdown: persons modal
    compare?: boolean
    bin_count?: BinCountValue // used in time to convert: number of bins to show in histogram
    funnel_window_interval_unit?: FunnelConversionWindowTimeUnit // minutes, days, weeks, etc. for conversion window
    funnel_window_interval?: number | undefined // length of conversion window
    funnel_order_type?: StepOrderValue
    exclusions?: FunnelStepRangeEntityFilter[] // used in funnel exclusion filters
    exclude_events?: string[] // Paths Exclusion type
    step_limit?: number // Paths Step Limit
    path_start_key?: string // Paths People Start Key
    path_end_key?: string // Paths People End Key
    path_dropoff_key?: string // Paths People Dropoff Key
    path_replacements?: boolean
    local_path_cleaning_filters?: Record<string, any>[]
    funnel_filter?: Record<string, any> // Funnel Filter used in Paths
    funnel_paths?: FunnelPathType
    edge_limit?: number | undefined // Paths edge limit
    min_edge_weight?: number | undefined // Paths
    max_edge_weight?: number | undefined // Paths
    funnel_correlation_person_entity?: Record<string, any> // Funnel Correlation Persons Filter
    funnel_correlation_person_converted?: 'true' | 'false' // Funnel Correlation Persons Converted - success or failure counts
    funnel_custom_steps?: number[] // used to provide custom steps for which to get people in a funnel - primarily for correlation use
    aggregation_group_type_index?: number | undefined // Groups aggregation
    funnel_advanced?: boolean // used to toggle advanced options on or off
    show_legend?: boolean // used to show/hide legend next to insights graph
    hidden_legend_keys?: Record<string, boolean | undefined> // used to toggle visibilities in table and legend
    breakdown_attribution_type?: BreakdownAttributionType // funnels breakdown attribution type
    breakdown_attribution_value?: number // funnels breakdown attribution specific step value
    breakdown_histogram_bin_count?: number // trends breakdown histogram bin count
    aggregation_axis_format?: AggregationAxisFormat // a fixed format like duration that needs calculation
    aggregation_axis_prefix?: string // a prefix to add to the aggregation axis e.g. £
    aggregation_axis_postfix?: string // a postfix to add to the aggregation axis e.g. %
}
```

This `FilterType` has been "append-only" since 2020, and has passed the point of maintainability.

First some examples, then a proposal for a new approach.

### Filter examples

Here's a filter that returns a "trends" insight showing the aggregate count of events of type `$pageview` for the last 14 days:

```json
{
    "insight": "TRENDS",
    "interval": "day",
    "display": "ActionsLineGraph",
    "actions": [],
    "events": [
        {
            "id": "$pageview",
            "name": "$pageview",
            "type": "events",
            "order": 0
        }
    ],
    "properties": [],
    "filter_test_accounts": false,
    "date_from": "-14d",
    "date_to": null
}
```

Here's a funnel `event $pageview -> event $pageview with filters -> action "definitely not an action"`. The second event has a few filters applied.

```json
{
    "insight": "FUNNELS",
    "date_from": "-14d",
    "actions": [
        {
            "id": "2",
            "type": "actions",
            "order": 2,
            "name": "definitely not an action"
        }
    ],
    "events": [
        {
            "id": "$pageview",
            "name": "$pageview",
            "type": "events",
            "order": 0,
            "custom_name": "Visited a page"
        },
        {
            "id": "viewed dashboard",
            "type": "events",
            "order": 1,
            "name": "viewed dashboard",
            "properties": [
                {
                    "key": "created_at",
                    "value": "2022-11-14",
                    "operator": "is_date_exact",
                    "type": "group",
                    "group_type_index": 1
                },
                {
                    "key": "$browser",
                    "value": ["Chrome"],
                    "operator": "exact",
                    "type": "event"
                },
                {
                    "key": "email_opt_in",
                    "value": ["true"],
                    "operator": "exact",
                    "type": "person"
                }
            ]
        }
    ],
    "display": "FunnelViz",
    "new_entity": [],
    "interval": "day",
    "properties": [],
    "funnel_viz_type": "steps",
    "exclusions": []
}
```

That's a funnel. If I open its "step 2 dropoff", the filter gets two extra fields, and the API endpoint is different (`/insights/funnel` vs `/person/funnel`):

```json
{
    "funnel_step_breakdown": [""],
    "funnel_step": -2
}
```

Here's a path with a global filter:

```json
{
    "insight": "PATHS",
    "properties": {
        "type": "AND",
        "values": [
            {
                "type": "AND",
                "values": [
                    {
                        "key": "$anon_distinct_id",
                        "value": "e",
                        "operator": "not_icontains",
                        "type": "event"
                    }
                ]
            }
        ]
    },
    "end_point": "http://amazingstore.com/cart",
    "step_limit": 8,
    "include_event_types": ["$pageview"],
    "path_groupings": [],
    "exclude_events": [],
    "date_from": "-14d",
    "date_to": null,
    "funnel_filter": {
        "date_from": "-14d"
    },
    "local_path_cleaning_filters": []
}
```

Moving away from insights, here's a query under cohorts for "match persons who match criteria: completed `$autocapture` exactly 5 times in the last 30 days"

```json
{
    "properties": {
        "type": "OR",
        "values": [
            {
                "type": "OR",
                "values": [
                    {
                        "key": "$autocapture",
                        "type": "behavioral",
                        "value": "performed_event_multiple",
                        "negation": false,
                        "operator": "exact",
                        "event_type": "events",
                        "time_value": "30",
                        "time_interval": "day",
                        "operator_value": 5
                    }
                ]
            }
        ]
    }
}
```

## Proposal: PostHog Query AST

Instead of a bespoke and scary monolithic filter object, I propose to split this up into **nestable typed query objects**.

The end result should pass as an AST - an Abstract Syntax Tree ([here's an example TypeScript AST](https://ts-ast-viewer.com/#code/GYVwdgxgLglg9mABBAFjCBrApkgFASkQG8AoRZBAZzgBssA6GuAc1wHIUsanEB3OAE40AJm3wkAvkA)).

An AST is an abstraction, and with some effort and clever tree parsing, we can convert it **to and from** any other query language. Think `HogQL` --> `AST` (this RFC) --> `Clickhouse SQL` or `BigQuery SQL`.

This RFC is only scoped to the `AST` --> `ClickHouse SQL` transformation and does not worry about `HogQL` input nor `BigQuery` output. Those are for the future, if at all.

### Examples

_Note: These examples are illustrative, and possibly not coherent at a query level. The final shapes will be determined when actual work starts._

#### Events example

Let's start with the simplest query that returns all events:

```json
{
    "type": "events"
}
```

```ts
interface Node {
    type: string
}
```

#### Events with filters

Adding a few filters

```json
{
    "type": "events",
    "filters": {
        "event": "$pageview",
        "properties": [{ "key": "$browser", "equals": "Chrome" }],
        "date_from": "-14d",
        "date_to": null
    }
}
```

```ts
interface EventsNode extends Node {
    type: 'events'
    filters: EventFilters
}
```

This query can be used as input wherever an `EventsNode` is expected.

#### Funnel example

Sample insight with 2 steps, both of type `EventsNode`.

```json
{
    "type": "funnel",
    "steps": [
        {
            "type": "events"
        },
        {
            "type": "events",
            "filters": {
                "event": "$pageview",
                "properties": [{ "key": "$browser", "equals": "Chrome" }]
            }
        }
    ]
}
```

```ts
interface FunnelNode extends Node {
    type: 'funnel'
    steps: EventsNode[]
}
```

It's exactly the same type, no confusion.

#### Funnel dropoff persons

If we want people who dropped off at a step, broken down further, it'd be something like:

```json
{
    "type": "persons",
    "breakdown": ["person.account_type"],
    "from": {
        "type": "funnel-dropoff",
        "step": "2",
        "from": {
            "type": "funnels",
            "steps": [
                {
                    "type": "events"
                },
                {
                    "type": "events",
                    "filters": {
                        "event": "$pageview",
                        "properties": [
                            {
                                "key": "$browser",
                                "equals": "Chrome"
                            }
                        ]
                    }
                }
            ]
        }
    }
}
```

```ts
interface PersonsNode extends Node {
    type: 'persons'
    breakdown?: string[]
    from?: FunnelDropoffNode | EventsNode
}

interface FunnelDropoffNode extends Node {
    type: 'funnel-dropoff'
    step: number
    from: FunnelNode
}
```

I'm not sure how many levels of abstractions this requires. It'll depend on how this maps to actual SQL.

#### Funnel correlation

You could get correlation analysis for a funnel in a similar way:

```json
{
    "type": "correlation",
    "correlation": {
        "type": "properties",
        "exclude_names": ["$initial_geoip_postal_code", "$initial_geoip_latitude", "$initial_geoip_longitude"],
        "names": ["$all"]
    },
    "from": {
        "type": "funnels",
        "steps": [
            {
                "type": "events"
            },
            {
                "type": "events",
                "filters": {
                    "event": "$pageview",
                    "properties": [{ "key": "$browser", "equals": "Chrome" }]
                }
            }
        ]
    }
}
```

```ts
interface CorrelationNode extends Node {
    type: 'correlation'
    correlation?: CorrelationOptions
    from?: FunnelNode
}
```

### Types of Nodes

There are two types of AST nodes: 1) Data nodes, 2) Visualization nodes.

Here's a visualization node `barTimeGraph`, which has a data node of type `events` as its source.

```json
{
    "type": "barTimeGraph",
    "options": {
        "legend": false,
        "xAxisLabel": "klingons",
        "xAxisUnit": "kg"
    },
    "source": {
        "type": "events",
        "filter": { "date_from": "-7d" },
        "breakdown": "event.properties.$browser",
        "interval": "day"
    }
}
```

Here's the same data powering an events table

```json
{
    "type": "eventsTable",
    "options": {
        "columns": [
            "timestamp",
            "event",
            "person",
            "properties.$browser",
            "person.properties.$initial_browser",
            "formula:(properties.$screen_width * properties.$screen_height)"
        ]
    },
    "source": {
        "type": "events",
        "filter": { "date_from": "-7d" },
        "breakdown": "event.properties.$browser",
        "interval": "day"
    }
}
```

The frontend can use this knowledge to persist any screen configuration anywhere.

### API endpoint

Proposed endpoint: `https://app.posthog.com/api/projects/:team_id/query`

The request will be a valid JSON **data node** from any of the queries above. Requesting a visualization node will result in an error.

The response for a query of type `events` will be something like this:

```ts
interface EventsResponse {
    type: 'events'
    query: EventsNode
    count: number
    results: EventsResult[]
}
```

### Performance

The funnel query above can be summarised as `persons.from --> funnel_dropoff.from --> funnels.steps[] -> events`

This feels like it'll be 4 subqueries inside each other. Something like:

```sql
select sum(...)
from (
  select some_cols from (
     select * from events where time filter
  )
  where another_filter
  group by something
)
group by foobar
```

In reality, that's the answer sometimes. Most times we'd like to "unwrap" this and run a more performant query instead:

```sql
select sum(...) from events where time filter group by foobar
```

There are many ways to keep this performant. For example an optimization pass on the AST that merges nodes into faster composed nodes. `funnels.steps[] -> events` becomes `funnels.steps_from_events[]`.

Yet such an optimisation step is overkill at this stage. The easiest solution is to just peek inside the inputs when parsing them.

That basically means, instead of

```ts
function parseFunnelDropoff(node: FunnelDropoffNode, ctx: QueryContext) {
    const sql = ctx.wrap(opts.from)
    return `query something (${sql}) step ${ctx.wrap(step)} power to the people`
}
function parseFunnel(node: FunnelDropoffNode, ctx: QueryContext) {
    const sql = ctx.wrap(opts.steps.map((step) => `(${ctx.wrap(opts.step)})`))
    return `make funnel from (${sql}) get stuff`
}
```

you write

```ts
function parseFunnelDropoff(node: FunnelDropoffNode, ctx: QueryContext) {
    const sql = ctx.wrap(opts.from.steps.map((step) => `(${ctx.wrap(opts.step)})`))
    return `query something (${sql}) step ${ctx.wrap(step)} do other stuff`
}
```

### Tree parsing

I don't know what's the right shape for the parser. I don't know if this should be a bunch of functions, or classes. I do however know we need to pass around a global `ctx` object

```ts
// either this
class FunnelDropoffNode extends Node {
    toSQL({ wrap, ctx, opts }) {
        // opts.from is FunnelNode
        return `query something (${wrap(opts.from, ctx)}) step ${wrap(step, ctx)} do other stuff`
    }
}

// or this
function parseFunnelDropoff(node: FunnelDropoffNode, ctx: QueryContext) {
    const sql = ctx.wrap(opts.from)
    return `query something (${sql}) step ${ctx.wrap(step)} power to the people`
}
```

This `ctx` could be used to carry global filters to the end, e.g. from `funnel` to the `events` themselves.

```ts
function parseFunnel(node: FunnelDropoffNode, ctx: QueryContext) {
    ctx.filters = [...ctx.filters, ...node.globalFilters]
    const sql = ctx.wrap(opts.steps.map((step) => `(${ctx.wrap(opts.step)})`))
    return `make funnel from (${sql}) get stuff`
}
```

This section requires a deeper dive, possibly during implementation.

### Multiple levels

As long as somewhere in the chain we have something that normalises to either `persons` or `events`, we're golden. These should be the common targets, that can act as input to a lot of queries and visualizations. Each level in this chain will require a subquery.

### Pagination

TODO

### Versioning

TODO. We will possibly need an `apiVersion` field in each AST node.

## Next steps for the backend

This is a gradual rollout. There will be no big rewrite. The steps below will also bring no visible changes on the frontend.

1. Refactor the huge `FilterType` into `InsightTrendsFilter`, `InsightFunnelsFilter`, etc. It's still the same backend, just with cleaner queries (e.g. remove all `funnel*` fields for the trends filter).
2. Create flat query nodes like `insightTrendsQuery`, `insightPathsQuery`, and document their structure.
3. Create a "backend router". This will be one endpoint, which takes a query object, and returns the matching results. For now, we will just pass these queries on to the existing insight queries.
4. Create a new query node type `"events"`, with support for various filters. Use this to power the "live events" page.
5. Replace the bespoke steps in `insightTrendsQuery`, `insightFunnelQuery`, etc with nodes of type `events`.

The next steps after these will be determined when we work on this. However, these are some options:

1. Create a new node type `persons`, and connect it to the existing persons/cohorts/etc tables.
2. Create a new node type `persons from events`, something like: `{ type: 'persons', from: { query: { type: 'insightTrends', ... } } }`, where you get a list of persons for any events query.
3. Refactor the person modals to use this new query.

## The Frontend

### Phase 1 - keeping up with the refactors

The changes above will require a lot of frontend refactors. To keep this manageable, we will not introduce (almost?) any new frontend functionality in the first phase. We will refactor the frontend to support the new queries as they are implemented.

Some new things that could be included:

- You can now put all different tables and parts of insights (e.g. correlation analysis) onto a dashboard
- Same with event tables and the like

### Phase 2 - new live events table

Combining work from the [Lemon Data Grid](https://github.com/PostHog/posthog/pull/11817), build and release a new events table. Add configurable person properties.

### Phase 3 - live events aggregrations

Finally a big new feature: add support for aggregating values in the events view.

### Phase 4 - non-event visualizations

Make visualizations like `pieChart` accept different mappable inputs, so it could be used to render any type of data.

### Phase 5 - escape the persons modal

Add an "explore further" button to break out of the persons modal. This would effectively load a new data table (think persons list), but with certain filters applied.

### Phase 6 - start converging things

- Implement universal search. [MVP](https://github.com/PostHog/posthog/pull/11981)
- Introduce a big search bar that lets you swoop between different views. (maybe two selects at first?)
- Easily switch between events, persons, filters, etc.
- We could store a lot of stuff in the URL now, including frontend filters: `app.posthog.com/query#q={big-json-object}`

### Phase 7 - implement more things

- Add grouping, breakdowns, formulas and person properties to the "events" and "persons" pages.
- Make a nifty query editor that knows what to suggest. Perhaps use this to build the edit interface?
- ???

## Future directions

Here even more ways to take this further.

### "Query Apps"

We could build various apps that modify the query, like you'd modify an AST.

```json
{
    "type": "funnels",
    "steps": [
        {
            "type": "action",
            "action": {
                "id": 1
            }
        },
        {
            "type": "events",
            "filters": {
                "event": "$pageview",
                "properties": [
                    {
                        "key": "$browser",
                        "equals": "Chrome"
                    }
                ]
            }
        }
    ]
}
```

A hypothetical "action inliner" app could convert the `{ type: 'action' }` node into a corresponding `{ type: 'events', filters: {} }` node.

### "Visualization apps"

Custom JS apps that get data and show it in many new formats. World map 2.0.

### HogQL

An AST can be converted into any language.

For example this:

```json
{
    "type": "funnel",
    "steps": [
        {
            "type": "events"
        },
        {
            "type": "events",
            "filters": {
                "event": "$pageview",
                "properties": [{ "key": "$browser", "equals": "Chrome" }]
            }
        }
    ],
    "breakdown": ["event[0].$browser"]
}
```

could be equivalent to this:

```sql
SELECT * FROM funnel WITH STEPS (SELECT * FROM events) as events1, (SELECT * FROM events WHERE event='$pageview' AND properties.$browser = 'Chrome') as events2 BREAKDOWN BY events1.properties.$browser
```

If it's better, I don't know. Probably just more error prone.
