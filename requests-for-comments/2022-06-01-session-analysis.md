# Sessions in insights

The goal of this document is to propose a specific approach to how we add session analysis to our insights product as defined in the [Product Context document](https://github.com/PostHog/product-internal/pull/307/files),

For more details on the problem we're trying to solve and the customer needs, please reference that document.

**Note:** The goal of this project is not to build a feature that allows users to find and explore specific sessions. Rather this feature is targeted at doing analysis on sessions in aggregate

## Requirements

We need to figure out how to allow users to do “session based analysis” inside our product. As defined in the document above, this means allowing users to do analysis on the following:

1. Number of sessions where specific events occur over time
2. Average session length over time
3. Average number of sessions per user over time
4. Session length distributions
5. Count of a specific event per session over time
6. Limit a funnel conversion to a single session
7. Limit a path conversion to a single session

## Possible Solutions

At it’s highest level, there are 2 options here:

1. Fitting session analysis into our existing insights features (mainly trends)
2. Building out a new sessions feature tailor made for this analysis

(tl;dr we should do option 1 because we can lean on lots of existing behavior of trends)

The big trade-offs between these two options are:

- Option 1 allows for lots of analysis that you “get for free” because of the power of trends. For example the following are all easily possible:
    - Plot the number of sessions over time along with the count of users. Then use formulas to divide the 2 and get “sessions per user”
    - Plot average session length where a users does Z broken down by browser, and notice that your feature is broken on IE.
    - Plot the average session length where users do X and compare it to the average session length where users do Y. Notice that users that do X, stick around longer.
    - Run an experiment where the goal is to increase the session length 🤯
- Potentially, option 2 would allow for more complex sessions to be defined. Right now, the language of trends is about defining events, not sessions. As a result, when this language is translated to sessions, it means, “Give me sessions where this specific event occurs”. This restricts users from creating complex session definitions such as “sessions where X happens and the Y happens within 2 minutes”.

My vote is that we **do option 1 and build this analysis into the existing trends product**. The benefits of building these features into trends greatly outweighs the complex sessions a user could define in option 2. Also, there are future paths where we could allow users to define complex sessions within option 1.

## How should these added analyses be built into trends?

Looking at the various analyses that we want to allow, it seems like the requirements above could fit relatively cleanly into our existing insights UX. Below is a proposal for how we would enable all of the above defined requirements.

### 1. Number of sessions where specific events occur over time

Proposal: Inside trends, we add “Unique Session” as an ‘event aggregation’ option.

![image](https://user-images.githubusercontent.com/4813045/171521247-45dfc5d4-d2c5-4ca4-9fec-f81cc0ace5e8.png)


### 2. Average session length over time

Proposal: We allow users to do property aggregations based on “session length”

![image](https://user-images.githubusercontent.com/4813045/171521195-121300f6-48a9-488c-956c-f176243865a7.png)

Details:
- In addition to showing event properties in the taxonomic filter when a property aggregation is selected, we also show “session properties”. Within session properties, we would show session length.
- If a session property is the target of the property aggregation, then the query would be aggregated by sessions.

### 3. Average number of sessions per user over time

Proposal: Users accomplish this using formulas. They would make a series that is (a) Session count and (b) user count, and then divide (a) by (b).

![image](https://user-images.githubusercontent.com/4813045/171521323-74e94328-1b21-45c8-925f-d9cc3aab6725.png)

### 4. Session length distributions

Proposal: We add session length as an option in the breakdown properties. Additionally, we allow users to “auto-bucket” breakdowns that are on numeric properties. 

![image](https://user-images.githubusercontent.com/4813045/171521347-8f3fd16c-28ab-49ac-9baa-c02097d4c42c.png)

Details: The “auto-bucket” functionality would require more work to figure out exactly how this should work + what the UX should be.

### 5. Count of a specific event per session over time

Proposal: Users accomplish this using formulas. They would make a series that is (a) Session count and (b) total count, and then divide (b) by (a).

![image](https://user-images.githubusercontent.com/4813045/171521365-2b353bb0-7634-45fa-84e3-1463eb351fc0.png)

### 6. Limit a funnel conversion to a single session

Proposal: In funnels, we add “Unique sessions” as an aggregation option. Setting this value would enforce that the sequence of events must occur within a specific session.

![image](https://user-images.githubusercontent.com/4813045/171521401-3b7a234c-24b4-45cd-a3d0-15ab4243efb0.png)


### 7. Limit a path conversion to a single session

Proposal: I’d consider this to be a lower priority, but we should update the logic in paths to respect the concept of a session. Today, it uses a 30 min gap between events to split paths - which is essentially the same as sessions, but subtly different.

One issue that we will have to figure out is being backward compatible to data that was generated prior to session support.

## Work breakout:

To accomplish the above, we would need to do the following work:

- Trends:
    - Add “unique sessions” as an event aggregation option in trends
    - Add the concept of “session properties”, and specifically the “session length” property to the taxonomic filter.
    - Allow for session properties to be used as a property aggregation in trends.
    - Allow for session properties to be used in breakdowns
    - Add the ability to “auto bucket” when breaking down by numeric properties
    - Note: while not included in the requirements above, it’d be cool to allow filtering based on session length everywhere. (e.g. total count of pageviews where the corresponding session length > 1 min)
- Funnels
    - Add sessions as a aggregation target
- Paths
    - Use sessions instead of the 30 min gap (need to consider backward compatibility here)

## Questions + notes

1. Do we need a sessions table?
    1. If we add session length as a property option, those queries will have to generate a list of session durations on the fly. This query can’t be limited to specific event types because it has to query across all events to ensure it finds the first and last event in the session. As a result it might be a hefty query. To optimize this, we could create a sessions table in clickhouse (probably using AggregatingMergeTree) ****that contains the session_id and it’s length
    2. My vote is that we don’t over optimize early and try to do without this table at first. We can always add it later.
2. There will be a mismatch in recording duration and session duration
    1. Because event based sessions and recording based sessions are based on different events, there will not be a perfect match when comparing the start and end time of each. This could cause some confusing user experiences.
    2. One solution here could be leaning on a sessions table as described above, and having it reconcile the difference between these two data sources to ensure consistency.
