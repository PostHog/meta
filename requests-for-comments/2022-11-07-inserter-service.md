# Inserter Service Requirements

We’ve used the `kafka` table engine on ClickHouse for some time now. It was the perfect table engine to support our ingestion pipeline at the time. We had very few resources and needed a way to safely get events from our `[event.py](https://github.com/PostHog/posthog/blob/master/posthog/api/event.py)` ingestion service into ClickHouse with very little fuss.

It’s time now for us to move on from our friend the `kafka` table engine and move this functionality outside of ClickHouse.

## Why?

- [clickhouse.cloud](http://clickhouse.cloud) does not support `kafka` table engines
- It’s generally well known that the `kafka` table engine is buggy. We’ve seen this a few times with ClickHouse trying to aggressively to retry and pull down Kafka. Altinity and ClickHouse Inc have warned us against using it.
- We’d like to decouple our pipeline from ClickHouse.
- We’d like to be able to insert rows into multiple ClickHouse warehouses (by team) without an explosion of topics.
- We’d like to support dynamic and parametrizable warehouse connection parameters
  - Team 2 → Warehouse with connection details x.y.z pulled from API
- Use whatever serialization we want on ClickHouse without having to ship the schema as a file to ClickHouse and reboot it (Proto and Avro problems)
- Much more configurable error handling.
- We can instrument much better metrics on the health of this step in our pipeline.

# Goals

- [ ] Consumes from topics and inserts data into corresponding tables
  - [ ] Built for Kafka, but is abstracted just enough to work with others (Pulsar)
  - [ ] Built for ClickHouse, but is abstracted just enough to work with others (BigQuery, Snowflake)
- [ ] Ensures only rows destined for ClickHouse are inserted ‘only once’
- [ ] Routes rows to the correct ClickHouse instance based on `team_id`
- [ ] Dynamically load the ClickHouse instance details (hostname, port, credentials) for `team_id`
- [ ] We improve the guarantees of messages on the data bus for typing and serialization and deserialization efficiency (Protobuf and co)
- [ ] Needs to be able to be paused via API to support Async Migrations
- [ ] Extremely granular control over how we route messages depending on the error
- [ ] Sentry is notified of errors with all of the context
- [ ] Exponential backoff for `too many parts` errors as well as other categories of errors
- [ ] Failsafe send messages to DLQ

## Non Goals

- [ ] We will not materialize columns. We will use new `[JSON` datatype](https://clickhouse.com/docs/en/sql-reference/data-types/json/)
- [ ] We will not use `Distributed` or `sharded` tables
- [ ] We will not run arbitrary code on this path. This is not a replacement for the Plugin Service.

## Phased development

### Phase 1

A consumer that consumes from these topics and inserts into corresponding ClickHouse tables only for `team_id = 2` and only supporting one ClickHouse cluster hosted by [ClickHouse.cloud](http://ClickHouse.cloud)

| Topic                               |     | Table                            |
| ----------------------------------- | --- | -------------------------------- |
| clickhouse_session_recording_events | ⇒   | posthog.session_recording_events |
| clickhouse_events_json              | ⇒   | posthog.events                   |
| plugin_log_entries                  | ⇒   | posthog.plugin_log_entries       |
| clickhouse_person                   | ⇒   | posthog.person                   |
| clickhouse_app_metrics              | ⇒   | posthog.app_metrics              |
| clickhouse_groups                   | ⇒   | posthog.groups                   |
| clickhouse_person_distinct_id       | ⇒   | posthog.person_distinct_id2      |
| events_dead_letter_queue            | ⇒   | posthog.events_dead_letter_queue |

The goal of Phase 1 is to simply unblock the team on shipping the SQL Interface for dogfooding. Basic requirements to meet this goal are:

- [ ] Lightweight table creation only if tables don’t exist yet. Do not use schema definitions to build table.
- [ ] Consumes from topics and inserts data into corresponding tables

### Phase 2

The goal of Phase 2 is to enable the routing of events based on `team_id`. This means two things in particular

- [ ] We can dynamically load ClickHouse cluster connection information from an API from PostHog in order to facilitate dynamic inserting into definable clusters
- [ ] We can batch and route rows to the correct cluster based on `team_id`

The crux of this phase will be ensuring that an outage of one `team_id`'s ClickHouse instance does not create backpressure for others. It may be acceptable to drop events at first and punt on this.

### Phase 3

The goal of Phase 3 is to ensure _exactly once_ insertions into ClickHouse for each batch of rows belonging to each team, destined for their ClickHouse cluster.

How do we do this?

- [ ] Abstract the consumer to support data buses other than Kafka (Potentially support Pulsar).
- [ ] Abstract the inserter to support other OLAP databases (BigQuery, Snowflake, etc)
- [ ] Be very deliberate about how we ack messages from the data bus for each batch belonging to each `team_id`

**At any point in time we can tackle the Serialization goal. That doesn’t block anything here. It only improves efficiency both for the computer and for the developers.**
