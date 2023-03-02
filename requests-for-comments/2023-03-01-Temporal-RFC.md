# RFC: Temporal at PostHog

Author: James Greenhill

Date: February 27, 2023 

# For the reader

Hi there, dear reader. I would love for you to scrutinize this RFC and the following RFC with more specifics on how we can build out our workflows. I’m looking for two things in particular from you though:

- Is this a good idea?
- Is getting this done feasible?

I’m specifically looking for feedback from teams that are directly impacted by this, namely:

- Infrastructure
- Billing
- Pipeline

# Problem Statement

We have a number of tasks that we’d like to run and have the confidence that they ran to completion. It would be even nicer to have some confidence that we were able to run it to completion exactly once despite hardware issues, deployments, or James going in and restarting a machine manually. So far we’ve built a few different ways to do this that are all quite different: Celery, Kafka, and Graphile. Each of these is a worker queue or worker queue adjacent that we have to maintain and keep flowing. The goal of this RFC is to propose a new, consolidated solution that we can use to focus more on our business logic (the product that we are actually building) and less on making sure that that logic actually runs. It’s a lot easier to reason about a product if you don’t have to think as much about it’s failure modes, retry logic, and timeouts.

## Requirements

We have a few products that are good candidates for a workflow framework. We’ll cover a few of the requirements here before we jump into any proposals just so that we can have a better understanding of what we are looking for.

### Billing

Billing is one of those things that is really tricky. Ideally you don’t want to bill someone twice (oof). You also don’t want to think you bill someone and then not bill them at all. Ideally you can kick off a flow and at the end of it have confidence that everything worked. The system was able to grab the usage metrics, apply the billing rate, and charge the user without having to double check the usage and the stripe api. Even though I still believe in trust, but verify. Any billing mistakes are generally pretty embarrassing.

Requirements:

- **Idempotent** or **exactly once** - no duplicates
- **Retry-able** - If the thing fails, keep trying to get the money we are owed
- **Resumable** - nice to pick up where you left off
- **Composable** - Query ClickHouse. Calculate bill. Charge stripe.
- **Parallelize-able** - We should be able to run multiple bills at the same time
- **Timeouts** - We should be able to set timeouts on steps
- **Cancelable** - We should be able to cancel any billing gracefully
- **Terminate-able** - We should be able to rip the carpet out from a billing routine
- **Nonblocking failures** - A failure of one billing workflow should not halt the execution of others.

### Celery tasks

Our celery tasks are pretty diverse. We use it for collecting metrics, refreshing caches, and if you didn’t know - we used to even use it to process every single event 😱.

What do we use Celery for now?

In general they are all scheduled tasks:

- Running queries and emitting metrics (statsd, prometheus)
- Warming up caches

Requirements:

- **Retry-able** - If the thing fails, get back on that horse
- **Resumable** - Some of these are heavy and we don’t want to requery CH
- **Parallelize-able** - We should be able to run multiple tasks at once
- **Timeouts** - We should be able to set timeouts on the task
- **Terminate-able** - We should be able to rip the carpet out from under the Task
- **Nonblocking failures** - A failure of one task should not halt the execution of others.

### Historical Exports *(AKA Vanilla ETL)*

We do a lot of moving around data. Actually, a good chunk of our business is just getting data form A to B safe and sound, and in on piece. Some of these jobs are short, some are long running. There are a few requirements and nice to haves around ETLs

Requirements:

- **Idempotent** or **exactly once** - no duplicates
- **Retry-able** - If the thing fails, get back on that horse
- **Resumable** - Nice to pick up where you left off
- **DAG-able** - Most ETLs resemble a Directed Acyclical Graph. Move A → B → C
    - You can think of this as **Composable**
- **Parallelize-able** - We should be able to run multiple bits of the DAG at once
- **Timeouts** - We should be able to set timeouts on the DAG and Nodes
- **Cancelable** - We should be able to cancel the DAG’s execution *gracefully*
- **Terminate-able** - We should be able to rip the carpet out from under the DAG
- **Nonblocking failures** - A failure of one DAG should not halt the execution of others.

### Webhooks

Ahh webhooks. Webhooks are a particularly sticky problem.

Requirements:

- **Idempotent** or **exactly once** - no duplicates
- **Retry-able** - If the thing fails, get back on that horse
- **Resumable** - Helps with idempotency/exactly once
- **Parallelize-able** - We should be able to run multiple webhooks at once and out of order (non-blocking)
- **Timeouts** - We should be able to set timeouts for webhooks
- **Terminate-able** - We should be able to rip the carpet out from under webhooks
- **Nonblocking failures** - A failure of one webhook endpoint should not halt the execution of others.

## Requirements Grid

| Task | Idempotent / Exactly Once | Retry-able | Resumable | Parallelize-able | Timeouts | Cancelable | Terminate-able | Nonblocking Failures |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Billing | 💯 | 💯 | 💯 | 🙏 | 🙏 | 🙏 | 💯 | 💯 |
| Celery Task |  | 💯 | 🙏 | 💯 | 💯 |  | 💯 | 💯 |
| ETLs | 💯 | 💯 | 💯 | 💯 | 💯 | 💯 | 💯 | 💯 |
| Webhooks | 💯 | 💯 | 🙏 | 💯 | 🙏 |  | 💯 | 💯 |

🙏 - Really nice to have

💯 - Required

# Where are we at now?

Currently our plugin server has been doing all of the heavy lifting and has gotten us quite far in the **************2 years************** that it has existed. I once gave feedback that the plugin server is a diamond encrusted hammer that tempts you into thinking that everything is a nail. Joke was on me because it turned out that it got us way further than I thought.

Since this began we’ve done amazing work in keeping the plugin server going. Chief among the improvements is that we split the plugin server up dependent on the workload into pools of resources that are responsible for different categories of plugin.

### The plugin server, a survey

So what all is the plugin server responsible for? There are really two types of plugins or tasks that we operate on the plugin server: Stateful and Stateless. Stateful plugins make side effects when they are called. This means that an API is called or a mutation is made in a db. Stateless plugins only affect the event as it is in flight. A good example of a stateless plugin is GeoIP where we take the ip that is in the event and based on that information and a static GeoIP db we geolocalize using the ip. 

************************Stateful vs Stateless************************

The practical difference between these two is that a stateless plugin generally executes in a deterministic way. This is mainly because it depends only on its inputs to execute. A stateful plugin on the other hand does not execute so deterministically. This is because of the fact that it depends on something outside of the inputs to the function, like an API. As an example let’s think of a plugin that during execution calls out to an api to either mutate or augment the data that is in flight. This is stateful because depending on the time that this plugin runs the results may vary. If this is a plugin that augments data with the response of an API the data returned by the API may change request to request. The api also may fail or stall. In our case most of these APIs are external to us and may fail or degrade at any point. This makes error handling of stateful plugins very important.

- Plugins
    - Stateless
        - Transformations
    - Stateful
        - Exports
        - API Calls
        - Reverse ETLs
        - Augmenters
- Webhooks (Stateful)

So what’s the pain point currently?

Our biggest pain point currently is with the stateful plugins. Regardless of whether they are scheduled or realtime because they depend on external resources that can degrade or fail they require sophisticated timeout and retry logic that we don’t have currently. Another pain point is a relative lack of visibility into the system when things fail. The final pain point is that we don’t have the ability to easily compose these plugins together. What we need is a workflow framework. We are halfway through building one.

You might ask yourself, James, if we are halfway through building a workflow engine, then why don’t we just finish it!? That’s a great question. The real reason we should not complete it is that frameworks already exist to do just this and building another one is not our competitive advantage against our competitors. Building one is a distraction from us building out our actual business.

What is a compounding factor here is that this isn’t just a plugin server issue but an issue that is felt across the product: Async migrations, billing, and PoE are all tasks that fit a workflow engine really well. There will be other tasks that we need to run in the future that will need a solid framework for executing workflows as well. This allows us to focus on these and not something that is outside of our expertise and product space.

# Temporal

Temporal is a workflow engine that provides a way for developers to build and manage distributed applications. It is designed to help developers focus on their business logic by abstracting away the details of failure modes, retry logic, and timeouts. Temporal allows developers to build workflows that describe complex processes as a set of simple, easy-to-understand steps. It provides a programmable interface that enables developers to write the code that defines their workflows and the logic that controls how they execute.

Temporal is used by a number of companies, including Uber, DoorDash, Instacart, HashiCorp, and Coinbase. It has become a popular choice among developers because it provides a way to manage complex distributed systems without having to worry about the underlying infrastructure. Temporal allows developers to build workflows that are resilient, scalable, and fault-tolerant, which is essential for building applications that can handle large volumes of data and traffic.

One of the key features of Temporal is its ability to provide visibility into the status of workflows. It provides a dashboard that allows developers to monitor the progress of their workflows and to see where errors are occurring. This helps developers to quickly identify and fix problems, which is essential for ensuring that workflows are running smoothly.

Overall, Temporal is a powerful tool that enables developers to build complex distributed applications and workflows. It provides a way to manage the complexity of distributed systems and to build applications that are scalable, resilient, and fault-tolerant.

[What is temporal?](https://docs.temporal.io/temporal)

[How does temporal work?](https://temporal.io/how-temporal-works)

### Why is it a good fit?

Temporal is an especially good fit because it provides things that are genuinely hard to build and maintain ourselves:

- Retry logic per Activity and Workflow
- Timeout logic per activity and workflow
- Exactly-once semantics (!!)
- Go, [Python](https://docs.temporal.io/kb/python-sandbox-environment), and Typescript libraries that support deterministic code execution and retries (!!)
- Composability of Activities and Workflows
- Dashboard for observing Workflow status

Finally, Temporal is a good fit because we are a small team. We should be focusing on the business of building our product. Nothing about what we are building that would be run on Temporal is novel enough to justify us to invest in a competing runtime.

### Who uses it?

- **Uber** (original developer - originally called [Cadence](https://github.com/uber/cadence))
- **Coinbase**
- **Doordash**
- **Instacart**
- **Hashicorp**
- **Netflix**
- **Snapchat**
- **Stripe**
- **Datadog**
- **Box**
- **Airbyte**
- A whole lot more…

### Quick Concept Overview

A workflow in [Temporal.io](http://temporal.io/) is a set of simple, easy-to-understand steps that describe a complex process. Workflows are made up of activities, which are individual steps in the workflow that perform specific tasks. Activities can be executed in parallel, and Temporal provides retry logic, timeout logic, and exactly-once semantics. Workflows and activities can be composed together, and Temporal provides a dashboard for observing workflow status.

# Example Workflow

We currently use it for our manual backfills to S3 for certain clients. You can see an example workflow here:

[https://github.com/posthog/energize](https://github.com/posthog/energize)

Note: that this will be folded into the Posthog/Posthog repo. For now this is separate for example purposes.

The specific workflow for S3 exporting can be found [here](https://github.com/PostHog/energize/blob/main/src/energize/workflows/exports/s3.py).

# Temporal Cloud

Why ☁️?

There is one critical piece of the Temporal topology that causes loss if it goes down: The scheduler. As long as the scheduler is up. When you schedule a job - it will be completed. If the scheduler is down for an extended period of time there is a chance that the requesting service would drop the intent and that workflow would never be run. As long as the scheduler is up and we are with the tunable TTL of the workflow, as soon as a worker comes back online we will process that workflow.

Luckily [Temporal.io](http://Temporal.io) has a hosted version of their scheduler where we can just submit our workflows and connect our workers to and the rest should be relatively smooth sailing. We only have to worry about writing the workflows themselves and keeping the workers happy.

# Ownership

This is why I’ve wanted to get you all here. Here is how ownership here would most likely shake out:

**Infrastructure**

- K8s Deployment of the workers that execute Workflows and Actions
- CD of these workers

**Billing**

- Workflows and Actions written to be executed on Temporal workers

**Pipeline**

- Same, Workflows and actions written and scheduled to be executed on Temporal workers

**Temporal cloud**

- The scheduler itself. As long as the workflows are scheduled, they will be executed at some point in the future as long as a worker starts back up and the TTL does not expire the job.

# Phases of work

**Phase 1: Manual mode -** We disable historical exports in the app interface. This is the buggiest feature that larger high value customers are complaining about. This transitions into a manual mode where if a customer wants a historical export they request that we run one for them and we do that manually for them using a Temporal workflow. This allows us to be iterative and to start gathering operational knowledge around Temporal.

**Phase 2: Build the Platform -** We stand up Temporal cloud as our scheduler and add worker deployments to K8s. This will build the foundation for migrating tasks to Temporal and unblock those that are interested in migrating their workflows to Temporal. Manual exports will run on the Temporal workers that are deployed on the K8s cluster. Temporal worker code is in the PostHog/PostHog monorepo

**Phase 3: API-ification -** We rewire the historical exports interfaces to schedule Temporal workflows. The api is already there (built for the manual backfills). The only difference now is that we are scheduling workflows from the PostHog application through the user interface

**Phase 4: Scheduling -** We make our ‘realtime’ export jobs batch temporal scheduled jobs that run from previous high water marks.

**Phase 5: Webhooks -** We migrate our webhooks to be workflows to be executed on Temporal.

**Phase 6: Celery -** We migrate our Celery tasks to be operated on Temporal and decomission our Celery workers.

**Phase TBD ≥ 3 -** Billing is migrated to temporal workers