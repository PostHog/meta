# Request for comments: Data Exploration - Marius Andra

"Get your data structures right, and the rest will follow"
- me

## Premise

At PostHog, users own their data. It's theirs. They can do whatever they want with it. 

Ideally we want to support them getting any insight out of it.

This includes export to other
tools to do analysis which PostHog can't. 





We provide the tools to analyse this data.

Users should be able to ask any kind of question about the data they own, and we would provide the tools to get an answer.

Data exploration in PostHog is currently limited by the tools we have built.

Luckily we have built enough tools to answer most common questions, but there are always some that require special
analysis or further exploration to get to the bottom of them.

Currently it feels like we're building an AI using "if" statements, instead of neural networks.

How can we improve it? Well, let's see what's there.

For each user we capture an event. It can have all sorts of properties.

```js
event = {
  event: 'event name',
  distinct_id: 'unique id',
  ip: '127.0.0.1',
  timestamp: string,
  properties: {
    '$withdollar': 'posthog properties',
    '$feature/stuff': 'feature flags',
    'anything else': 'custom properties'
  },
  person_properties: {},
  group_properties: {},
}
```

These are stored in a table. We have then built interfaces to:

- List all the events stored in this table
- Through "live events"
  - Filter this list by any event/person/group property or event name
  - Filter this list by person/group/distinct_id
- Through an insight
  - Aggregate this data by a time bucket 
  - Aggregate this data by some property (`count(distinct distinct_id)`, `count(*)`, `avg()`)
  - Split this aggregation by another property (breakdown by `$browser`)
  - Slice or dice this stream of events in a different way (`funnel`, `retention`, `paths`)
- Through the insight persons modal
  - Remove the aggregation and get the actual events behind the property
  - On a funnel, get the success/dropoff for each step


We want to be able to manipulate each step further. Suppose we get the list of persons who dropped off in a funnel.

This steps should be encapsulated as an input.

```
query = {
  type: 'persons',
  breakdown: ['person.account_type'],
  columns: ['count(*)', 'person.distinct_id', 'person.account_type'],
  display: {
    as: 'pie',
  },
  from: {
    type: 'funnel-dropoff',
    step: '2',
    from: {
      type: 'funnel',
      steps: [{}, {}, {}],
      from: {
        type: 'events'
        filters: {
          date_from: '',
          date_to: '',
        },
      }
    }
  }
}
```

Depending on the type of output, you can have different views.

Most event data types can be sent to a table.
Output type "events" with aggregations and a time column 

The current filters object is a mess


Here's a simple trends line
```json
{
    "events": [
        {
            "id": "$pageview",
            "math": "avg_count_per_actor",
            "name": "$pageview",
            "type": "events",
            "order": 0
        }
    ],
    "actions": [],
    "date_to": null,
    "display": "ActionsLineGraph",
    "insight": "TRENDS",
    "interval": "day",
    "breakdown": "$geoip_city_name",
    "date_from": "-14d",
    "new_entity": [],
    "properties": [],
    "breakdown_type": "event",
    "filter_test_accounts": false
}
```
Here's a funnel:

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
            "math": "avg_count_per_actor",
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
                    "value": [
                        "Chrome"
                    ],
                    "operator": "exact",
                    "type": "event"
                },
                {
                    "key": "email_opt_in",
                    "value": [
                        "true"
                    ],
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

Here's a path with some global filter

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
    "include_event_types": [
        "$pageview"
    ],
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

Here are all the filter keys currently supported

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

Other than scanning for a keyword like "funnel", it's very hard to know when anything is used.

Sometimes we want to use part of an insight as an input to the next query, e.g. in a "persons modal".

```json
{
    "insight": "FUNNELS",
    "date_from": "-30d",
    "actions": [],
    "events": [
        {
            "id": "$pageview",
            "name": "$pageview",
            "type": "events",
            "order": 0
        },
        {
            "id": "$pageview",
            "type": "events",
            "order": 1,
            "name": "$pageview"
        }
    ],
    "display": "FunnelViz",
    "new_entity": [],
    "interval": "day",
    "properties": [],
    "funnel_viz_type": "steps",
    "exclusions": [],
    "breakdowns": [
        {
            "property": "$geoip_country_code",
            "type": "event"
        }
    ],
    "breakdown_type": "event"
}
```

vs

```json
{
    "insight": "FUNNELS",
    "date_from": "-30d",
    "actions": [],
    "events": [
        {
            "id": "$pageview",
            "name": "$pageview",
            "type": "events",
            "order": 0
        },
        {
            "id": "$pageview",
            "type": "events",
            "order": 1,
            "name": "$pageview"
        }
    ],
    "display": "FunnelViz",
    "new_entity": [],
    "interval": "day",
    "properties": [],
    "funnel_viz_type": "steps",
    "exclusions": [],
    "breakdowns": [
        {
            "property": "$geoip_country_code",
            "type": "event"
        }
    ],
    "breakdown_type": "event",
    "/* new stuff below this line */": "/////////",
    "breakdown": ["$geoip_country_code"],
    "breakdown_attribution_type": "first_touch",
    "funnel_step_breakdown": [""],
    "funnel_step": -2,
    "funnel_window_interval": 14,
    "funnel_window_interval_unit": "day",
    "smoothing_intervals": 1,
    "include_recordings": true
}
```

It's the same query, just with this added: 

```json
{
  "funnel_step_breakdown": [""],
  "funnel_step": -2
}
```



