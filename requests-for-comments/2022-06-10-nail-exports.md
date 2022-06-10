# Request for comments: Nail Exports - Ben White

## Problem statement

More context here https://github.com/PostHog/product-internal/pull/308/


### What can be Exported

* Insights
  - Visualisation - web, image
  - Data - API, csv
* Dashboards
  - Visualisation - web, image (or collection of images)
  - Data - API, csv
* Persons, Cohorts, Groups

### How can they be Exported
* Embeds 
  - Via an iframe or directly linked to a dedicated webpage for viewing the element 
* Images
  - Server-side rendering of the shared webpage (using chromedriver) to an image (.png)
  - Possibly needs multiple dimension options
* Subscriptions - email:
  - User configured regular reports via email including exported image of the relevant item
* Subscriptions - slack:
  - Slack notification including the exported image(s)
* CSV / Spreadsheet:
  - Relevant data exported to a CSV / Spreadsheet
* Custom Components (React)
  - Integrated directly into customer's code, calling Posthog API for the data 
* Alerts (variation on subscriptions):
  - When certain thresholds are crossed or changes happen, a user is notified with similar channels to that of Subscriptions
* API:
  - Standard retrievel using public API


## Method breakdown

### Embeds
| insights ✅ | dashboards ✅ | person ❌ | cohorts ❌ | groups ❌ |
|------------|--------------|----------|-----------|----------|

Similar to SharedDashboards we currently have, it makes sense to have a unique rendering method for standalone visualisations that can either be accessed:
1. Directly via an authenticated user
2. Directly via an unauthenticated user if "sharing" has been enabled
3. Via an iframe embedded elsewhere
4. Server side via selenium for taking high-quality image exports as "screenshots"

---
#### Technical considerations
- [ ] Embeds should allow customiseable options as query params for showing hiding different elements (e.g. readonly, whitelabel)

#### Current State
- 🤔 Dashboards are embeddable and shareable under a special `shared_dashboards` endpoint
- 🤔 Exports are _almost_ embeddable via the `exporter` endpoint. Currently this is only used for image rendering
- **PROPOSAL**: Merge these endpoints together to a unified endpoint for all standalone viewing of content

---
### Images
| insights ✅ | dashboards ✅ | person ❌ | cohorts ❌ | groups ❌ |
|------------|--------------|----------|-----------|----------|

Images are important either for the user (e.g. to include in a powerpoint presentation) or for use by other export methods such as subscriptions

#### Technical considerations
- [ ] Some things are hard/impossible to render nicely in small sizes (e.g. Funnels).
- [ ] 


---
### Subscriptions - email:
| insights ✅ | dashboards ✅ | person ❌ | cohorts ❌ | groups ❌ |
|------------|--------------|----------|-----------|----------|

Subscriptions via email are essentially Image exports but delivered via email

#### Technical considerations
- [ ] The exported Image needs to be publicly accessible using some sort of token
- [ ] Users need to be able to easily unsubscribe without needing to login

---
### Subscriptions - slack:
| insights ✅ | dashboards ✅ | person ❌ | cohorts ❌ | groups ❌ |
|------------|--------------|----------|-----------|----------|

Subscriptions via slack are essentially Image exports but delivered via slack. We may want to consider additional information to be included

#### Technical considerations
- [ ] The exported Image needs to be publicly accessible using some sort of token
- [ ] Users need to be able to easily unsubscribe without needing to login


---
### CSV / Excel:
| insights ✅ | dashboards ✅ | person ✅ | cohorts ✅ | groups ✅ |
|------------|--------------|----------|-----------|----------|

Resources can be exported directly to a CSV

#### Technical considerations
- [ ] Exporting 10000s of rows can be costly or even impossible syncronously - it needs to be possible to do this in the background and send to the user when complete
- [ ] TTLs: Exported data can include very sensitive information. Ideally we should only keep the exported file for long enough to satisfy the user's request

---
### API:
| insights ✅ | dashboards ✅ | person ✅ | cohorts ✅ | groups ✅ |
|------------|--------------|----------|-----------|----------|

The API should offer all of the same data access as the web app
