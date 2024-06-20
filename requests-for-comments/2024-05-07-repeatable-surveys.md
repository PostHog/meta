# Request for comments: Recurring surveys, Phani Raj

Desired decision date: May 10th, 2024

## Problem statement
_Who are we building for, what are their needs (include example cases), why is this important?_


Customers want a way to show a survey on a recurring schedule to track user sentiments over a period of time. [Customer issue](https://github.com/PostHog/posthog/issues/22099)

A premier example of this requirement is tracking NPS over time, 
`are my customers getting happier about the product in the last year?`

## Success Criteria
_How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?_

We will be successful in building this feature if : 

1. The customer's existing pain points with NPS tracking is solved, as described in the above issue. 

2. We see surveys created with a repeated schedule after this feature ships, I'm not sure of an exact number right now, but I assume that once we give customers the ability
   to repeat a survey, they'll start using it.

## Context
_What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?_

Our competitors have already built tools that help customers track user sentiment over a time period, allowing them to repeat surveys and gain insights into changing user behavior.

Our current workarounds of `just repeat the survey manually` might get them to repeat the survey, but there's no way to gain any useful insights since the responses will end up in different surveys.

### Similar tools
1. [Pendo's custom NPS survey type](https://support.pendo.io/hc/en-us/articles/360033527151-Set-up-an-NPS-survey#h_01HGD0SKYSRB6SNKXBFJ1QG7M7)
2. [Surveymonkey's multiple responses](https://help.surveymonkey.com/en/surveymonkey/send/allowing-multiple-responses/)

This is how UserPilot shows NPS scores changing over time 

<img width="1115" alt="image" src="https://github.com/PostHog/meta/assets/82819/3fbaa08d-aa79-4781-b0f4-07b56242486f">


## Design
_What are the key user experience and technical design decisions / trade-offs?_

A survey is assumed to have only one response in the system and we have filter conditions that govern **who** can see the survey.
A user's survey response action is tracked as a boolean, i.e. we add a person property with the survey-id `$survey_response/<surveyID>` when a user submits or dismisses a review.

To support this feature, we need to introduce the concept of `survey iterations` to the UI and person properties.

Here's what we need to do to solve this : 

### Data model changes to the surveys table
1. Add properties to the survey data model to track 

| Column name | type | description | nullable |
|:----|:----|:----|:----|
| `iteration_frequency_days` | integer |  Frequency, in days, at which the survey should be repeated | Yes | 
| `iteration_count` | integer | max number of times the survey should be repeated |  Yes |
| `iteration_start_dates` | []date | Start dates of all iterations for this survey. |  Yes |

This design allows our users to setup such surveys:

1. `repeat this survey 4 times, once every 90 days`.
2. They can also just leave the `iteration_count` as zero to have an infinitely repeating survey `repeat this survey, once every 90 days`

These three properties are nullable and we need to make our work backwards compatible to make existing survey responses function without issues to show NPS scores etc. 

#### UI changes. 
1. Show the user an interface to pick the repeat interval of the survey, this will be a frequency in Days.
2. Show the user an interface to pick the iteration count, how many times should this survey repeat?
3. Once both values are collected, on the server, use `[Recurrence Rule](https://dateutil.readthedocs.io/en/stable/rrule.html)` to save the `iteration_start_dates` of the Survey. 
4. If a survey has an iteration count of more than 1, change the existing UI to run custom clickhouse queries to account for a survey with multiple response sets, one for each iteration.
5. Depending on survey type, introduce a section to show historic results:
   
   4.1. NPS : Display NPS scores bucketed by detractors, passives, promoters, over time. Here's an example.
<img width="1115" alt="image" src="https://github.com/PostHog/meta/assets/82819/3fbaa08d-aa79-4781-b0f4-07b56242486f">
   4.2. CSAT: Display cumulative CSAT over time. Here's an example from https://theacsi.org/
   <img width="489" alt="image" src="https://github.com/PostHog/meta/assets/82819/ff9d9928-862b-4fb3-a718-225922efce4a">

6. Along with historic results, allow users to pick a time period/iteration and show the following information over that time period.
   
   5.1. Users viewed/submitted/dismissed
   
   5.2. for NPS, responses segmented into detractors,passives & promoters.
   
   5.3. for CSAT, responses segmented into dissatisfied, satisfied & very satisfied.

7. Custom new sections to show:

   6.1. Survey response rate over time, i.e., if the number of people responding to a survey increases/decreases over time.
   
   6.2. Survey user base changing over time, i.e., if users who submitted the survey once do it again. 
   
#### posthog-js changes.
1. To decide when to show a survey, see if a user has a survey response submitted for the current survey start date, and show the survey if they haven't.
3. When a survey is submitted, add properties for survey iteration count and survey start date in the current iteration  to the person properties. So this person is accounted as having submitted the survey for this iteration/start date.
  `$survey_response/${survey.id}/${survey.iteration_start_date}/`
   `$survey_response/${survey.id}/${survey.iteration_count}/`

### out of scope
1. Survey creators cannot change the frequency once a survey is created, as this will change the way we calculate the survey response rates. 
2. Changing the survey questions over time, i.e. having a different set of questions for each time period is not supported.

