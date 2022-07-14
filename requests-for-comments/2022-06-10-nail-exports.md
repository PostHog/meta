# Request for comments: Nail Exports - Ben White

## NOTE: This RFC was closed on 2022-07-14 with these agreed remaining follow up issues
- [ ] Exporting CSVs is not enabled everywhere
- [ ] Exporting large CSVs perhaps still doesn't work
- [ ] Embedding / sharing of images
- [ ] Using shared images in meta for Sharing html
  
## Problem statement

More context here https://github.com/PostHog/product-internal/pull/308/


### What can be Exported

* Insights
  - ✅ Visualisation - web, image
  - ✅ Data - API, csv
* Dashboards
  - ✅ Visualisation - web, image (or collection of images)
  - ✅ Data - API, csv
* Events
  - ✅ the actual underlying data 
* (🚫 out of scope) Session Recordings
* ✅ Persons, Cohorts, Groups

### How can they be Exported
* ✅ Embeds 
  - Via an iframe or directly linked to a dedicated webpage for viewing the element 
* ✅ Images
  - Server-side rendering of the shared webpage (using chromedriver) to an image (.png)
  - (🚫 out of scope) Possibly needs multiple dimension options
* ✅ Subscriptions - email:
  - User configured regular reports via email including exported image of the relevant item
* ✅ Subscriptions - slack:
  - Slack notification including the exported image(s)
* ✅ CSV / (🚫) Spreadsheet:
  - Relevant data exported to a CSV / Spreadsheet
* (🚫 out of scope) Custom Components (React)
  - Integrated directly into customer's code, calling Posthog API for the data 
* (🚫 out of scope) Alerts (variation on subscriptions): _(out of scope for current exporting work)_
  - When certain thresholds are crossed or changes happen, a user is notified with similar channels to that of Subscriptions
* ✅ API:
  - Standard retrievel using public API


## Method breakdown

### Embeds
| insights ✅ | dashboards ✅ | events ❌ | person ❌ | cohorts ❌ | groups ❌ |
|------------|--------------|----------|-----------|-----------|----------|

Similar to SharedDashboards we currently have, it makes sense to have a unique rendering method for standalone visualisations that can either be accessed:
1. Directly via an authenticated user
2. Directly via an unauthenticated user if "sharing" has been enabled
3. Via an iframe embedded elsewhere
4. Server side via selenium for taking high-quality image exports as "screenshots"

---
#### Technical considerations
- [ ] Embeds should allow customiseable options as query params for showing hiding different elements (e.g. readonly, whitelabel)
- [ ] This could be shown immediately to the user when enabling the embed iframe link

#### Current State
- 🤔 Dashboards are embeddable and shareable under a special `shared_dashboards` endpoint
- 🤔 Exports are _almost_ embeddable via the `exporter` endpoint. Currently this is only used for image rendering
- **PROPOSAL**: Merge these endpoints together to a unified endpoint for all standalone viewing of content
  - **ExportedAsset**

    | insight  | dashboard | content                   | content_source  | format        | expiration_date                               |
    |----------|-----------|---------------------------|-----------------|---------------|-----------------------------------------------|
    | relation | relation  | the actual binary content | file\|s3://link | png\|pdf\|csv | time by which the content should be accesible |
  - **Subscription**
    | insight  | dashboard | title                | target_type  | target_value   | {...rules}                              |
    |----------|-----------|----------------------|--------------|----------------|-----------------------------------------|
    | relation | relation  | name of subscription | email\|slack | a@b.co \| #dev | columns todo with subscription calendar |
  
  - **SharingConfiguration**
    | insight  | dashboard | token                   | permissions                    | iframe_origins | deleted                            |
    |----------|-----------|-------------------------|--------------------------------|----------------|------------------------------------|
    | relation | relation  | unique short-ish string | list_of_emails\|password\|sso? | only-me.com    | soft delete (like enable/disabled) |


---
### Images
| insights ✅ | dashboards ✅ | events ❌ | person ❌ | cohorts ❌ | groups ❌ |
|------------|--------------|----------|-----------|-----------|----------|

Images are important either for the user (e.g. to include in a powerpoint presentation) or for use by other export methods such as subscriptions

#### Technical considerations
- [ ] Some things are hard/impossible to render nicely in small sizes (e.g. Funnels).
- [ ] 


---
### Subscriptions - email:
| insights ✅ | dashboards ✅ | events ❌ | person ❌ | cohorts ❌ | groups ❌ |
|------------|--------------|----------|-----------|-----------|----------|

Subscriptions via email are essentially Image exports but delivered via email

#### Technical considerations
- [ ] The exported Image needs to be publicly accessible using some sort of token
- [ ] Users need to be able to easily unsubscribe without needing to login

---
### Subscriptions - slack:
| insights ✅ | dashboards ✅ | events ❌ | person ❌ | cohorts ❌ | groups ❌ |
|------------|--------------|----------|-----------|-----------|----------|

Subscriptions via slack are essentially Image exports but delivered via slack. We may want to consider additional information to be included

#### Technical considerations
- [ ] It may make sense to have support for multiple slack destinations
  - [ ] e.g. Internal metrics to primary Slack, external metrics (e.g. pineapple dashboard to community slack to show off feature) to different Slack community


---
### CSV / Excel:
| insights ✅ | dashboards ✅ | events ✅ | person ✅ | cohorts ✅ | groups ✅ |
|------------|--------------|----------|-----------|-----------|----------|

Resources can be exported directly to a CSV. This may be commonly done adhoc (e.g. from some in-memory filter configured on the frontend.

#### Technical considerations
- [ ] Exporting 10000s of rows can be costly or even impossible syncronously - it needs to be possible to do this in the background and send to the user when complete
  - [ ] This could also incur very high costs. Perhaps we still need some upper bounds.
- [ ] TTLs: Exported data can include very sensitive information. Ideally we should only keep the exported file for long enough to satisfy the user's request

#### 💡 IDEA
To make this scalable we could:
- Create CSV exports like so `POST /exports { "format": "csv", "resource_uri": "/api/persons/funnel?filters=..." }`
- CSV Exporting is now very simple - we use the given URI to iterate the resource_uri until the `next` response param is exhausted.
- With each type of export (person/trend/event) we just need a transformer to map it to a csv-like structure and then we save the chunks to ObjectStorage
- It is now trivial to add CSV exporting to any list based endpoint we expose 
  - Hacky approach - call the actual API with a special JWT
  - Less hacky approach - map the URLs to the ViewSets and use `as_view()` to call the appropriate method without using a network request
  

---
### API:
| insights ✅ | dashboards ✅ | events ✅ | person ✅ | cohorts ✅ | groups ✅ |
|------------|--------------|----------|-----------|-----------|----------|

The API should offer all of the same data access as the web app


# Sprint breakdown
## Sprint 1

- [x] Subscriptions - Email - Insights
- [x] Subscriptions - Email - Dashboards
- [ ] Subscriptions - Slack - Insights
- [ ] Subscriptions - Slack - Dashboards
- [ ] Image export renderer refactoring

## Sprint 2
- [ ] Subscriptions - Slack (finishing)
- [ ] CSV - everything
- [ ] Embeds / Sharing
