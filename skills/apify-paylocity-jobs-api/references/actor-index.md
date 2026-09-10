# Actor routing table

One Actor, `johnvc/paylocity-jobs-api`, backs two skills. Route by the job at hand.

| The job | Skill | Actor input shape |
|---|---|---|
| Get the open jobs from named Paylocity boards | `apify-paylocity-jobs-api` (this one) | `companies` = GUIDs / URLs / names, filters, `maxJobs` |
| A daily new-postings feed for boards you track | `apify-paylocity-jobs-api` (this one) | add `newerThan: "25h"` on a schedule |
| Find which companies hire on Paylocity | `apify-companies-using-paylocity` | `discoverAll: true`, `discoverOnly: true`, `discoveryQuery` |

Store page: https://apify.com/johnvc/paylocity-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
Hosted MCP server: `https://mcp.apify.com/?tools=actors,docs,johnvc/paylocity-jobs-api`

## Related job-data Actors

- Greenhouse Job Board API: https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills
- Workday Careers API: https://apify.com/johnvc/workday-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
- iCIMS Careers API: https://apify.com/johnvc/icims-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
- Ashby Job Board API: https://apify.com/johnvc/ashby-job-board-scraper?fpr=9n7kx3&fp_sid=awesomeskills
