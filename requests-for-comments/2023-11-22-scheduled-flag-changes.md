# Request for comments: Scheduled Feature Flag changes

## Problem

Currently, users can make updates to feature flags only manually and in real-time. This is not ideal, as it potentially leads to overlooked or forgotten changes, particularly when they fall outside normal working hours. This can lead to poorly timed releases, negatively impacting end users' engagement and feedback.

## Solution

Scheduling feature flag changes allows users to automatically change flag properties at a future point in time. For example, a user can schedule an initial release of the flag for internal testing, then enable it to the beta users group 2 days later, and finally roll it out to 100% of the users in 2 weeks.

## Scope

This feature should allow the following scheduled modifications to a feature flag:

-   **Enable/disable the flag** (field: `is_active`)
-   **Add rollout conditions to the flag** (field: `filters`)

We want to be able to _generalize_ this implementation to other models such as `Survey` or `Experiment`.

## Implementation

### API

To schedule future changes, we can have a scheduling API, specifying the model to operate on, the id, the changes to be executed, and the schedule time.

```
POST /api/projects/:project_id/schedule
{
    "model": "FeatureFlag",
    "id": 57,
    "changes": [ ... ]
    "scheduled_at": <timestamp>
}
```

A scheduled change can be expressed as a JSON, specifying the operation to be performed and any payload required for the operation. In the context of scheduled changes to feature flags, the operation will be a `db_update`. For other scheduling use cases in the future, the operation may be again a database update or some other side effect.

**Example 1: Enable the flag**

```
{
    "operation": "db_update",
    "payload": {
        "field": "is_active",
        "value": "false"
    }
}
```

**Example 2: Add a rollout condition to the flag**

```
{
    "operation": "db_update",
    "payload": {
        "field": “filter”,
        "value": {
            "variant": null,
            "properties": [{
                "key": "$geoip_country_name",
                "type": "person",
                "value": ["Australia"],
                "operator": "exact"
            }],
            "rollout_percentage": 20
        }
    }
}
```

### Queue processing

_At_ or _after_ the time specified, the change needs to be propagated onto the record.

We already use Celery with a Redis queue to execute various background tasks. However, because the Redis queue is not durable, we cannot schedule tasks at the precise point in the future. A possible solution here is to have Postgres table of scheduled changes. Celery could periodically poll this table, e.g. every minute. During each poll, Celery would process any rows where the _scheduled_at_ time has already passed.

The `scheduled_changes` table would have the following schema:

| team_id | record_id | model       | changes (JSON) | scheduled_at (timestamp) | executed_at (timestamp, null) |
| ------- | --------- | ----------- | -------------- | ------------------------ | ----------------------------- |
| 1       | 57        | FeatureFlag | [ ... ]        | 2023-11-23 15:30:00      | null                          |

The table would be indexed on the `scheduled_at` column to efficiently retrieve the columns that have the scheduled\*at value in the past. For every row retrieved, the task would **atomically**:

-   Modify the field on the model record
-   Mark the row as completed by setting the `executed_at` value

#### Concurrency and retries

To tackle the issue of concurrency and retries, let's consider the following questions.

**What delay in mutations is acceptable?**

-   A delay up to 1 minute should be acceptable; above that we might receive complaints. Example: users want to precisely time a feature release such as a ticket sale at noon.

**Can mutations fail?**

Mutations will fail in the following scenarios:

-   The flag has been deleted in the meantime - no retry, just mark the row as completed.
-   Some field is invalid (e.g. filters) - no retry, but record the reason for the failure. (To guard against this, we can validate the fields in the “schedule changes” API)
-   PG is down - need to retry.

Considering the above, our approach should be as follows:

-   Retries: If PG downtime is the only reason for a failed mutation, we can simply wait for the next scheduled run of Celery at which point the schedule change will be retried.
-   Concurrency: Given that our current/foreseeable count of records is low, we can afford concurrency=1. Even during peak loads, we should be able to handle all scheduled changes within 1 minute. To implement concurrency=1, we can use a transaction with the NOWAIT clause. The first worker that selects the rows "wins", while the queries from subsequent workers will fail with an error.

### Model updates

Every model that needs scheduled updates will implement a `scheduled_changes_dispatcher()` method. This method will receive the payload from the Celery task and update the record. After the successful update, the worker marks the row as completed by setting `executed_at`.

## Monitoring

We will need to monitor the backlog of tasks and trigger an alert when the delay is causing too much degradation. For guidance, the `ingestion_lag` Celery task is a relevant example.

## UI

We can add the functionality under the "Schedule" tab within the feature flag view. Users can submit scheduled changes via a form and view currently scheduled changes.

<img width="981" alt="image" src="https://github.com/PostHog/posthog/assets/22996112/ff9f1c4c-c2f6-4dde-a420-bc282c5adcc8">
