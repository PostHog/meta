# Request for comments: Scheduled Feature Flag changes

## Problem

Currently, users can make updates to feature flags only manually and in real-time. This is not ideal, as it potentially leads to overlooked or forgotten changes, particularly when they fall outside normal working hours. This can lead to poorly timed releases, negatively impacting end users' engagement and feedback.

## Solution

Scheduling feature flag changes allows users to automatically change flag properties at a future point in time. For example, a user can schedule an initial release of the flag for internal testing, then enable it to the beta users group 2 days later, and finally roll it out to 100% of the users in 2 weeks.

## Scope

This feature should allow the following scheduled modification to a feature flag:

-   **Enable/disable the flag** (field: `is_active`)
-   **Add rollout conditions to the flag** (field: `filters`)

TBD:

-   **Modify existing rollout condition:** I would advise against adding this. When rolling out to more users, it is considered good practice to add to the existing conditions rather than modifying them. This provides an audit trail of changes and allows one to easily roll back the release to the previous state by removing the most recent condition(s).
-   **Delete existing conditions:** This would allow temporary access to a feature. It might be helpful for some users, but I would advise to wait for more feedback before adding this.

## Implementation

We use Celery to execute tasks on schedule. At the predetermined time, Celery will pick the flag and evaluate scheduled changes. I considered two possible approaches to scheduling:

-   Periodic scheduling - schedule every flag in regular intervals. This simplifies the implementation but creates a lot of unnecessary overhead.
-   Dynamic scheduling - pick the earliest time from the scheduled changes and schedule the flag to be evaluated at this time. After performing the change, reschedule the flag to the new earliest scheduled time. For this RFC, I am assuming the use of dynamic scheduling.

The scheduled changes can be stored on the flag record in the `scheduled_changes` field with the following structure:

```
[
    {
        "scheduled_at": "2023-11-22T12:30:00Z",
        "type": "is_active",
        "value": "false"
    },
    {
        "scheduled_at": "2023-12-01T15:00:00Z",
        "type": “filter”,
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
]
```

At the time specified, the Celery task fetches the flag, and iterates over the `scheduled_changes` items. For any item in this array, the `scheduled_at` field is evaluated, and if it is in the past, the task populates the item on the flag in the following way:

-   When the type `is_active` the task sets the value of the `is_active` field.
-   When the type is `filter`, the given condition is populated in the `filters` field of the flag.

After populating the change on the feature flag, the change item is removed from the `scheduled_changes` array.

To summarize, the end-to-end flow can go as follows:

1. The user specifies the changes in the UI and saves the feature flag.
2. The endpoint saves the record and stores a scheduled task in the Celery queue.
3. Celery executes the scheduled task at the future date. It populates the required fields and reschedules the next task in the future.

## UI

We can add the functionality under the "Schedule" tab within the feature flag view. Users can submit scheduled changes via a form and view currently scheduled changes.

<img width="981" alt="image" src="https://github.com/PostHog/posthog/assets/22996112/ff9f1c4c-c2f6-4dde-a420-bc282c5adcc8">

## Generalizability

The initial implementation idea assumes we schedule the task within the `POST /feature_flags` endpoint. However, we might want to abstract the scheduling functionality to accommodate other models such as `Survey` or `Experiment.` This would involve creating a scheduling API.

```
POST /projects/schedule
{
    type: 'feature_flag',
    scheduled_changes: [ ... ]
    scheduled_at: <timestamp>
}
```

The API would populate the scheduled changes on the record and store a task in the Celery queue.

I considered the option of only storing the scheduled changes in the task itself. This is, however, problematic since we need the list of changes available on the flag for the table view in the UI. That means this API would still need to both store the task and update the record with the list of scheduled changes.

## Possible failure modes

-   When submitting the scheduled changes to perform in the future, we need to store both the list of changes on the flag and the Celery task. The order matters here - we want to store the _task_ first. This is because:
    -   If the flag is stored first, but the saving of the task fails, the list of changes is now saved on the flag without the associated task in the Celery queue.
    -   If the task is stored successfully but the flag update fails, we inform the caller the operation failed. They can simply retry, so no issue here.
-   When processing a scheduled updated of a flag, we also perform two actions - save the flag and complete the task. It is possible the task will fail after the record has been saved. Assuming Celery auto-reschedules failed tasks, the task would re-run and attempt to perform the changes again. At this point however, the scheduled change in question will have been removed from the `scheduled_changes` array. Thus, re-running the task would not lead to issues such as duplicate conditions being saved.
