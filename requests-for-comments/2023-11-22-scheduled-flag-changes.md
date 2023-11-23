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

To schedule future changes, we can have a scheduling API, specifying the model the operate on, the id, the changes to be executed, and the schedule time.

```
POST /api/projects/:project_id/schedule
{
    "model": "FeatureFlag",
    "id": 57,
    "changes": [ ... ]
    "scheduled_at": <timestamp>
}
```

A change to the record can be expressed as a JSON, specifying the field to be modified and the new value to be set:

**Example 1: Enable the flag**

```
{
    "field": "is_active",
    "value": "false"
}
```

**Example 2: Add a rollout condition to the flag**

```
{
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
```

_At_ or _after_ the time specified, the change needs to be propagated onto the record.

We already use Celery with a Redis queue to execute various background tasks. However, because the Redis queue is not durable, we cannot schedule tasks at the precise point in the future. A possible solution here is to have Postgres table of scheduled changes. Celery could periodically poll this table, e.g. every minute. During each poll, Celery would process any rows where the _scheduled_at_ time has already passed.

The `scheduled_flag_changes` table would have the following schema:

| team_id | id  | changes (JSON) | scheduled_at (timestamp) | status (enum) |
| ------- | --- | -------------- | ------------------------ | ------------- |
| 1       | 57  | [ ... ]        | 2023-11-23 15:30:00      | scheduled     |

The table would be indexed on the `scheduled_at` column to efficiently retrieve the columns that have the scheduled\*at value in the past. For every row retrieved, the task would **atomically**:

-   Modify the field on the feature flag record
-   Mark the row as `completed`

We would need to have separate tables for other models such as `scheduled_survey_changes`, due to the schema differences and possibly different scaling requirements.

## UI

We can add the functionality under the "Schedule" tab within the feature flag view. Users can submit scheduled changes via a form and view currently scheduled changes.

<img width="981" alt="image" src="https://github.com/PostHog/posthog/assets/22996112/ff9f1c4c-c2f6-4dde-a420-bc282c5adcc8">
