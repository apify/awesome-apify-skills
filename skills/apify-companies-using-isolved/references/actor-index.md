# Actor routing table

One Actor powers both isolved skills; the skills differ in which output mode and question they are shaped for.

| Question | Skill | Actor input shape |
|---|---|---|
| Which employers use isolved | apify-companies-using-isolved | `outputMode: "tenantsOnly"`, directory sweep |
| Does this employer have an isolved site | apify-companies-using-isolved | `outputMode: "tenantsOnly"`, your `tenants` list |
| Job rows from named tenants | apify-isolved-jobs-api | default `jobs` mode, `tenants` list, filters |
| New or changed postings since yesterday | apify-isolved-jobs-api | `updatedAfter: "25h"` on a schedule |
| Cheapest full index of a tenant | either | `outputMode: "urlsOnly"` |

- Actor: https://apify.com/johnvc/isolved-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/isolved-jobs-api`
- Related family: Greenhouse Job Board API (https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills) and iCIMS Careers API (https://apify.com/johnvc/icims-careers-api?fpr=9n7kx3&fp_sid=awesomeskills) share the row shape for cross-ATS sales maps.
