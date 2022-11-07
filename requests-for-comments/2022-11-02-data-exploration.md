# Request for comments: Data Exploration - Marius Andra

_Whenever people ask me to be the life of the party, I go into a long rant about the importance of getting your data structures right.

It's the single most impactful advice you can give junior developers, and it's what we're going to do here._

## Why?

With PostHog, users own their data. Yet when exploring it, they're limited to the tools we've built. Sometimes they want more: 

- You can't explore insight person modals further (e.g. add a filter, break down by cohorts, etc).
- The events table supports event properties as columns, but that's it. What about person properties? What about aggregations? Tables below insights contain different data in insight vs table mode. Sorting is not supported.
- You can't have non-event insights. E.g. show a count of users with property X.
- Persons tables are very basic. No column customisation. No math or formulas.
- There are charts in insights that can't be added to dashboards (e.g. retention line chart).
- Various pages (events, persons, etc) support filters, but those can't be saved.
- There are parameters you can't directly filter for or list (such as `distinct_id`, `timestamp`)
- Our `FilterType` has been append-only for 2+ years and is long overdue for a serious cleanup.
- Multiple endpoints accept various types of filters with various capabilities (`/insights/funnel/correlations` vs `/insights/trend/` vs `/events` vs `/persons/trends`). It's impossible to type the outputs without blindly guessing.

We have an opportunity to build a unified "data exploration" view, which would solve all of the above, and open the floodgates to allowing any type of analysis.

The proposed changes below will and also unblock:

- let us embed anything on any dashboard (e.g. funnel correlation results, retention line graph, list of live events)
- let us link to any piece of data from anywhere (e.g. persons modal, full screen, with extra filters on the list)
- use typescript and a few helpers to strictly validate all query requests and responses
- build a fluid interface that can morph between different views or chart types (it's nodes all the way down)
- unblock visualisation apps (e.g. the scatterplot plugin, world map v2.0)
- let us embed data from sources other than clickhouse onto dashboards
- using any other query engine as our backend

## Part 1. Analysis of the backend

This is a proposal to change the way we query the PostHog API.

### Current state

We capture a stream of events with all sorts of properties:

```js
// this is pseudocode, only for illustration
event = {
  event: "event name",
  distinct_id: "unique id",
  ip: "127.0.0.1",
  timestamp: string,
  properties: {
    $withdollar: "posthog properties",
    "$feature/stuff": "feature flags",
    "anything else": "custom properties",
  },
  person: { properties: {} },
  group1: { properties: {} },
};
```

We store this stream of events in a table, and let users analyse it with bespoke tools like "trends", "retention", "persons" or "live events".

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
  insight?: InsightType;
  display?: ChartDisplayType;
  interval?: IntervalType;

  // Specifies that we want to smooth the aggregation over the specified
  // number of intervals, e.g. for a day interval, we may want to smooth over
  // 7 days to remove weekly variation. Smoothing is performed as a moving average.
  smoothing_intervals?: number;
  date_from?: string | null;
  date_to?: string | null;
  properties?: AnyPropertyFilter[] | PropertyGroupFilter;
  events?: Record<string, any>[];
  event?: string; // specify one event
  actions?: Record<string, any>[];
  breakdown_type?: BreakdownType | null;
  breakdown?: BreakdownKeyType;
  breakdowns?: Breakdown[];
  breakdown_value?: string | number;
  breakdown_group_type_index?: number | null;
  shown_as?: ShownAsValue;
  session?: string;
  period?: string;

  retention_type?: RetentionType;
  retention_reference?: "total" | "previous"; // retention wrt cohort size or previous period
  total_intervals?: number; // retention total intervals
  new_entity?: Record<string, any>[];
  returning_entity?: Record<string, any>;
  target_entity?: Record<string, any>;
  path_type?: PathType;
  include_event_types?: PathType[];
  start_point?: string;
  end_point?: string;
  path_groupings?: string[];
  stickiness_days?: number;
  type?: EntityType;
  entity_id?: string | number;
  entity_type?: EntityType;
  entity_math?: string;
  people_day?: any;
  people_action?: any;
  formula?: any;
  filter_test_accounts?: boolean;
  from_dashboard?: boolean | number;
  layout?: FunnelLayout; // used only for funnels
  funnel_step?: number;
  entrance_period_start?: string; // this and drop_off is used for funnels time conversion date for the persons modal
  drop_off?: boolean;
  funnel_viz_type?: FunnelVizType; // parameter sent to funnels API for time conversion code path
  funnel_from_step?: number; // used in time to convert: initial step index to compute time to convert
  funnel_to_step?: number; // used in time to convert: ending step index to compute time to convert
  funnel_step_breakdown?: string | number[] | number | null; // used in steps breakdown: persons modal
  compare?: boolean;
  bin_count?: BinCountValue; // used in time to convert: number of bins to show in histogram
  funnel_window_interval_unit?: FunnelConversionWindowTimeUnit; // minutes, days, weeks, etc. for conversion window
  funnel_window_interval?: number | undefined; // length of conversion window
  funnel_order_type?: StepOrderValue;
  exclusions?: FunnelStepRangeEntityFilter[]; // used in funnel exclusion filters
  exclude_events?: string[]; // Paths Exclusion type
  step_limit?: number; // Paths Step Limit
  path_start_key?: string; // Paths People Start Key
  path_end_key?: string; // Paths People End Key
  path_dropoff_key?: string; // Paths People Dropoff Key
  path_replacements?: boolean;
  local_path_cleaning_filters?: Record<string, any>[];
  funnel_filter?: Record<string, any>; // Funnel Filter used in Paths
  funnel_paths?: FunnelPathType;
  edge_limit?: number | undefined; // Paths edge limit
  min_edge_weight?: number | undefined; // Paths
  max_edge_weight?: number | undefined; // Paths
  funnel_correlation_person_entity?: Record<string, any>; // Funnel Correlation Persons Filter
  funnel_correlation_person_converted?: "true" | "false"; // Funnel Correlation Persons Converted - success or failure counts
  funnel_custom_steps?: number[]; // used to provide custom steps for which to get people in a funnel - primarily for correlation use
  aggregation_group_type_index?: number | undefined; // Groups aggregation
  funnel_advanced?: boolean; // used to toggle advanced options on or off
  show_legend?: boolean; // used to show/hide legend next to insights graph
  hidden_legend_keys?: Record<string, boolean | undefined>; // used to toggle visibilities in table and legend
  breakdown_attribution_type?: BreakdownAttributionType; // funnels breakdown attribution type
  breakdown_attribution_value?: number; // funnels breakdown attribution specific step value
  breakdown_histogram_bin_count?: number; // trends breakdown histogram bin count
  aggregation_axis_format?: AggregationAxisFormat; // a fixed format like duration that needs calculation
  aggregation_axis_prefix?: string; // a prefix to add to the aggregation axis e.g. £
  aggregation_axis_postfix?: string; // a postfix to add to the aggregation axis e.g. %
}
```

This `FilterType` has been "append-only" since 2020, and has passed the point of maintainability.

I have a proposal for a refactor, but first some examples.

### Filter examples

Here's a filter that returns a trends insight showing the aggregate count of events of type `$pageview` for the last 14 days:

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

Here's a funnel `$pageview -> $pageview with filters -> action "definitely not an action"`. The second event has a few filters applied.

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

If I want to open "step 2 dropoff", the filter just gets these extra fields, and gets sent to a different API endpoint (`/insights/funnel` vs `/person/funnel`):

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

Moving away from insights, here's a query under cohorts for "match persons who match any criteria: completed an event multiple times. $autocapture exactly 5 times in the last 30 days"

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

### Proposal: nestable typed query objects

Instead of a bespoke and scary monolithic filter object, I propose to split this up into **nestable typed query objects**.

The end result should pass as an [AST - an Abstract Syntax Tree](https://ts-ast-viewer.com/#code/GYVwdgxgLglg9mABBAFjCBrApkgFASkQG8AoRZBAZzgBssA6GuAc1wHIUsanEB3OAE40AJm3wkAvkA). This means we could convert it **to and from** any other notation. Think `HogQL` --> `AST` (this RFC ) --> `Clickhouse SQL` or `BigQuery SQL`  output.

For now, I wouldn't focus on `HogQL` input nor `BigQuery` output. These can be implemented in the future.

This RFC is only scoped to the `AST` --> `ClickHouse SQL` transformation.

How would this AST look like?

Let's start with the simplest query that returns all events:

```json
{
  "type": "events"
}
```

```ts
interface Node {
  type: string;
}
```

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
  type: "events";
  filters: EventFilters;
}
```

This query can be used as input wherever an `EventsNode` is expected. If it's at the root of a query, the frontend will
receive an `EventsResponse`,  

```ts
interface EventsResponse {
  type: "events";
  query: EventsNode,
  response: { count: number, results: EventResult[], }
}
```

and will likely render the "events list" scene.

If the result comes back with `type: "funnel"`, we'll instead render an insight with the funnel:

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
  type: "funnel";
  steps: EventsNode[];
}
```

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
  type: "persons";
  breakdown?: string[];
  from?: FunnelDropoffNode | EventsNode;
}

interface FunnelDropoffNode extends Node {
  type: "funnel-dropoff";
  step: number;
  from: FunnelNode;
}
```

I'm not sure how many levels of abstractions this requires. It'll depend on how this maps to actual SQL.

You could get correlation analysis for a funnel in a similar way:

```json
{
  "type": "correlation",
  "correlation": {
    "type": "properties",
    "exclude_names": [
      "$initial_geoip_postal_code",
      "$initial_geoip_latitude",
      "$initial_geoip_longitude"
    ],
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
  type: "correlation";
  correlation?: CorrelationOptions;
  from?: FunnelNode;
}
```

### Considerations

- The exact shape of this new API is not yet fixed. It will change somewhat when starting implementation.
- We will likely need an `apiVersion` field in each AST node.
- Not each node in the request (e.g. `{ type: 'people', from: { type: 'events', ... } }`) will map to a SQL subquery in the response ([discussion](https://github.com/PostHog/meta/pull/66#discussion_r1013178284)). 
  While we can be verbose in describing the query, we'll need a different approach on the backend, where merging queries together in bespoke ways often yields a lot higher performance.  

### Next steps for backend

The plan is to slowly transform the existing queries into the new approach. 

Here's a rough list of steps we can take on the backend. These will bring no visible changes on the frontend, though they will require some refactoring. 

1. Refactor the huge `FilterType` into `InsightTrendsFilter`, `InsightFunnelsFilter`, etc. It's still the same backend, just with cleaner queries (e.g. remove all `funnel*` fields for the trends filter).
2. Create flat query nodes like `insightTrendsQuery`, `insightPathsQuery`, and document their structure. 
3. Create a "backend router". This will be one endpoint, which takes a query object, and returns the matching results. For now, we will just pass these queries on to the existing insight queries.
4. Create a new query node type `"events"`, with support for various filters. Use this to power the "live events" page. 
5. Replace the bespoke steps in `insightTrendsQuery`, `insightFunnelQuery`, etc with nodes of type `events`.

The next steps after these will be determined when we work on this. However these are some steps:

1. Create a new node type `persons`, and connect it to the existing persons/cohorts/etc filters.
2. Create a new node type `persons from events`, something like: `{ type: 'persons', from: { query: { type: 'insightTrends', ... } } }`, where you get a list of persons for any events query
3. Refactor the person modals to use this new query. 


## Part 2. Analysis of the frontend

## Frontend

### Iteration 1

The "data exploration" view will basically be one big query editor. The first steps will look similar to the existing [MVP](https://github.com/PostHog/posthog/pull/11981) and expose a big table. 

- Two select boxes and a large table below. 
- You can select "events", apply "matching filters" and see a list of rows
- You can switch "events" to "persons" and get a different list.
- Switching to "recordings" might open a different looking interface (recordings might come later)

The goal is to replace the existing events and persons tables, including the persons table in insight modals.

These person modals will now have an "Expand" button that will open them in the "data explorer" view.

All insights should otherwise look the same, yet be powered by the new query engine.

### Iteration 2. 

Start bringing in new features:

- Add grouping, breakdowns, formulas and person properties to the "events" and "persons" pages.
- Make it possible to also see trends and funnels in the same interface. The filters should transform smoothly,
  e.g. the full "events" filter moves into the "step 1" box of the "funnels" filter when you click "start funnel from here".
- This will require a lot of ideation and a big design sprint. 


## Part 3. Future directions.

All of the above is already a lot of work. Here are some ways to take it even further:

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

### Visualization apps



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

If it's better, I don't know.