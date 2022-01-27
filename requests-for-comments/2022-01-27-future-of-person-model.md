# Request for Comments: *topic of the proposal*

- **Created:** 2022-01-27
- **Owner:** @macobo, @marcushyett-ph 

## Problem: Scaling queries horizontally

In posthog, having insights load fast is key for a great experience. However for large projects tracking >10M active persons per month query performance degrades fast.

This can't be solved by scaling horizontally, because a lot of our queries do aggregation on persons - e.g. filtering by user properties, counting daily active persons and calculating funnels. 

To do these queries we need to JOIN `person` and `person_distinct_id` tables. This in clickhouse is really slow (can cause >30x slower queries) and requires these tables to be replicated on every node, setting an upper bound to scalability. Teams with >10M users often run into memory usage problems as well.

This RFC proposes both a technical and product solution to this problem: Store `person_id` and `person_properties` on the main events table.

However, doing so will require breaking changes to the product. These changes are likely to cause some issues for customers, but these have either a negligible impact (affecting around 0.03% of events) or can be mitigated through documentation and education.

## Proposed solution

From a technical standpoint: Store `person_id` and `person_properties` on the main events table.

From a product standpoint storing person_id and person_properties on events requires making the following changes:
1. After the first session we will no longer merge events prior to login with the main person’s events.

2. Person properties become immutable: Changing properties applies only on new events and past data is not updated.

### Customer Impact

#### Benefits
For both changes the benefits are as follows:
Significant short term performance gains
Ability to support the largest tiers of customer now and in the future
Standard analysis such as events from paying users over time will stop yielding misleading results when people start / stop paying (as they do today)


#### 1. After the first session we will no longer merge events prior to login with the main person’s events.

![image](https://user-images.githubusercontent.com/148820/151367498-39637fc6-27ef-4058-b479-1651a4121875.png)

How we'll map persons to events going forward:
1. On initial visit, users device gets assigned an an (anonymous) distinct id `anonid1`. Posthog internally associates person_id `p1` to `anonid1`, which is saved together with their events
2. When after signing up `posthog.identify('userid')` is called, we associate the same person_id `p1` with the new distinct id `userid`
3. If the user later visits the site with a new device, a new anonymous distinct id `anonid2` is created. Posthog internally associates these events with a new person id `p2` as no mapping for that id exists.
4. If after signing in `posthos.identify('userid')` is called, we associate events going forward with person_id `p1`

As a result, events done between steps 1, 2 and 4 will be tracked as from person `p1` but events between steps 3 and 4 will be under person`p2`. As a result no events are lost but some will be considered as coming from a different (anonymous) person.

This affects use cases where users access their product (Logged out) on multiple devices regularly. However, based on our analysis the impact is only on ~0.03% events, some marketing use-cases are more significantly affected.

As a mitigation, if necessary, we would consider a buffer mechanism to reduce affected events to ~0.003%, we can provide further details on this solution, however we would ideally avoid implementing this as it increases the complexity of the system quite significantly.

#### 2. Person properties become immutable: Changing properties applies only on new events and past data is not updated.

Example: Count of active users broken down by `paying` user property each month in 2021.

After this change, if `paying` property of a user in Dec 2021 is changed to `false`, count of `paying` users does not change for e.g. March 2021. The  user will be considered non-paying until their `paying` property changes again2.

It also means users could be counted as both `paying=true` and `paying=false` during a given month when they change state.

User properties will change over time on events, some customers may rely on the existing behavior (where the latest property is assumed for all prior events), however the new behavior is considered much more accurate by our customer feedback.

Our proposed mitigation or most use-cases (e.g. events for users who have paid at some point) cohorts can be used to achieve the same results, we will document this for affected users


### Technical implications

### Benefits
1. Faster queries: remove the need to do JOINs for any insight queries.]
2. Unlock: Our query results no longer change unless new data has come in - we make it possible to later implement time-series caching to speed up repeated queries over long time frames (>10x improvement expected on repeated calculations).
4. We can sample by person_id in queries, which will make it possible to build a quick way to speed up queries in huge (1Bn+ events) installations with no additional query time resource cost.
5. Queries will require less memory and no longer OOM (making it cheaper and more reliable to operate posthog)
3. We can shard by person_id, simplifying the sharding story a lot and removing the need for cross-shard communication when sharding.

Note that some of these changes have been benchmarked [here](https://github.com/PostHog/posthog/issues/7962)


#### Challenges

6. We will need to set up some strong consistency for message ordering in plugin-server and kafka, to make sure messages are processed in the right order per user.
7. We will store more data in the events table. See below for more details.
8. Deleting user properties becomes more expensive due to being stored in a redundant way.

<details><summary>Details on cost of storage challenges</summary>


Storing person properties in events means we’ll be storing more data.

The exact impact of this depends on the dataset. Some factors about our setup which might not make this into a huge deal:

1. The data is low cardinality and clickhouse is great at compression => we should get really good compression ratios
2. When [Clickhouse implements semistructured data support](https://github.com/ClickHouse/ClickHouse/pull/23932), compression ratios will get even better.

I’ve done a measurement on a test benchmarking cluster:
- Original table was 7.56 GiB on disk, 99.13 GiB uncompressed
- New table (with person_properties/person_id joined in) was 9.16 GiB on disk, 167.88 GiB uncompressed
- person_properties column was 1.13 GiB on disk, 51.21 GiB uncompressed with a compression ratio of 0.022
- For reference, `properties` column was 4.49 GiB on disk, 76.28 GiB uncompressed with a compression ratio of 0.059

This measurement was done on a dataset that would compress relatively well (few people), however looking at the uncompressed size, we’d expect at most a 2x disk size increase in most cases.

</details>

## Conclusion
Scalability of Posthog will be critical for our success, this project lays the foundation for high scalability, removes a critical scaling bottleneck and unblocks future improvements for even larger deployments. There are both technical and product risks associated with this project but they are far outweighed by the long-term benefits.

## Open questions

- How do we handle historical data imports?
- How do we implement the change on cloud in a cost-effective way?
