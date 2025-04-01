# RFC - Test Pipeline Changes on Real Customer Data w/o Affecting Actual Prod Data

## Background

We now have reasonable traction with our data warehouse product and the cost of making breaking changes with
incremental updates is growing as we accrue more and more external sources, self-managed sources, and saved queries each
day. At this time, we don't have enough confidence in our (mostly unit) test suite and development workloads don't
closely mimic the realities of production scenarios. Not all the sources have similar distributions of use, but one
common theme is that many are SQL based and our data eventually end up in s3 via delta tables and are queryable via
HogQl, so that feels like the area to hone our focus on for biggest impact. Other data sources like HubSpot, Salesforce,
and Chargebee are less generic and also would have a smaller blast radius in the event syncing incremental or full table
syncing gets regressed. These sources are still important and should be incorporated into the test plan as it matures.

## Higher Level Goals

- Provide confidence in the continuous operation of data warehouse syncs and queries for incremental changes to the data
  warehouse product across a broad spectrum of sources, joins, and queries.
- Facilitate the transition of our source code to use more robust inheritance and more generic/unified sets of
  operations, while ensuring that we don't regress on existing functionality as we do so.
- Determine if the keys and secrets we are using will work ahead of time in production and nothing gets regressed in the
  process.

## Testing Goals

- Determinism - We do not want to randomly sample sources/schemas in order to test against. This list of tests should
  grow over time, but remain consistent for feature development.
- Repeatability - A failure one run should fail the same way the next run, A success should also be repeatable.
- Not Ignorable - A scheduled out-of-band test can and will be ignored, so this test sequence should run as part of CI.
  We shall have a designated label in CI called `dw-integration-suite` that will trigger CI to run these tests.
- Uses "real" data - Not anchored to "test" datasets that fail to capture the madness of the real world.
- No Perf Degradation - We don't wish to interfere with any production workloads or queries.
- No Cost/CPU Impact - We don’t wish to run any out of band testing workloads that would impact a live service, with the
  possible exception of testing our own services that are in the purview of PostHog.

## Types of Actions to Test

- Syncs
    - Incremental
    - Full table
- HogQl queries (joins)

> Preface: The main idea behind this strategy is to reverse the direction of our standard syncing procedure in order to
> determine if we can reconstruct our block storage data (delta → s3) back into the original data store it came from (
> Postgres, MySql, etc…), and query it once it’s reconstructed as before once it has been put back together.

> Note: The diagram below can be used as an aid.

## Steps

> Note: For now I'm going to mark this as out of scope in favor of the lean variant, due to several concerns around both
> how maintainable this would be and how much time it would take to implement as well as compliance concerns (GDPR,CCPA).

1. The first step would be to transfer data from a configurable list of sub-buckets under
   `posthog-s3-datawarehouse-us-east-1` into an ephemeral s3 bucket from a list of deltalake tables from prod → dev
   environment. Using this approach we can avoid looking at credentials and our Postgres instance. This also means that
   we have very minimal interactions with production as a whole. This would be a file in a test directory that governs
   the resolved list of tables we wish to consistently test against.

2. Reconstruct stateful instances from the delta tables in our CI flow. This would require different processes for
   various providers. For example, for MySql we could take an official image, deploy it with ArgoCD, then have a program
   move the data that has been temporarily copied into our dev s3 bucket into entities we consider external data
   sources.

3. Run an on demand temporal workflow to test our standard syncing operation as if it’s any other production workflow.

4. The results of these workflows will end up back in s3 via delta as an output. We should be able to run queries at
   this point.

5. Compare the results of the original copied s3 payloads to the s3 payloads that went through the temporal processor
   and were synced. This is mostly a parity check to ensure that our syncing process is working as expected. The parity
   check could do a few things for us like check the behavior of the incremental and full table syncs, ensuring that the
   row counts are the same, and then also verify that some sampled subset of the data matches.

6. Run test case queries on data that has moved from s3 → external_data_source → s3 either with custom queries or test
   queries inherited from production.

7. Reverse of #1 to clean up all fragments of delta tables that have been copied into the dev environment and buckets
   that have. Cleanup fragments in Clickhouse that have been generated by this process as well.

[Data Warehouse Testing Production Flow](./images/dw-testing-production-flo.png)

## Steps (Lean/Lite Variant)

1. Tag a PR with a `dw-integration-engine-all` or `dw-integration-engine-${engine_name}` label which shall trigger a
   GitHub action sync for either all or a subset of our sources within a shared organization in our dev environment.
2. Attempt to programmatically sync our own within our `dev` environment for the source(s) that has been tagged, or all
   sources if the `*-all` label has been used. These sources should already be available already as hand crafted test
   sources As a second step, we could sync some sources from our posthog (team 2 production) environment parquet file to
   a variety of stateful instances. (BigQuery, Postgres, Snowflake, etc...)
3. Ensure that corresponding row counts are correct along with the time that the data was synced.

## Out of Scope

- Temporal Scheduling Logic - We want to ensure that sync works, not that temporal invokes a schedule at the correct
  interval.
- Load testing - For massive queries that exceed thresholds encountered in production. This is mostly meant to test if
  the schemas and queries for syncs are working, not if we can handle massive transactions.

## Tooling

- ArgoCd
    - Source containers (Postgres, MySql, Snowflake)
    - Source destinations (Postgres, MySql, Snowflake)
- S3
- GitHub Actions (Invoke when changing subset of our directories, triggering agent)

## Concerns/Open Questions

- How feasible is it to recreate instances of more hosted type databases, like Snowflake, would something like
  LocalStack be an option for us?
- It is too much work to do this for some many of our integrations, would we become test automation specialists instead
  of software engineers? How can we maintain this balance over time added more and more source types?
- What would be the best way to simulate other sources like Salesforce?
- Should we use a test account for something like Stripe?
- Would we like to test out complicated joins on queries between multiple sources and events? 





