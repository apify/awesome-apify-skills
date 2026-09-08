# Actor routing table

One Actor, `johnvc/paylocity-jobs-api`, backs two skills. Route by the job at hand.

| The job | Skill | Actor input shape |
|---|---|---|
| Find which companies hire on Paylocity | `apify-companies-using-paylocity` (this one) | `discoverAll: true`, `discoverOnly: true`, `discoveryQuery`, `verifyLive` |
| Get the open jobs from named boards | `apify-paylocity-jobs-api` | `companies` = GUIDs / URLs / names, filters, `maxJobs` |
| A daily new-postings feed for boards you track | `apify-paylocity-jobs-api` | add `newerThan: "25h"` on a schedule |

Discovery gives you the `boardGuid`; hand it to `apify-paylocity-jobs-api` to pull that company's jobs.

Store page: https://apify.com/johnvc/paylocity-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
Hosted MCP server: `https://mcp.apify.com/?tools=actors,docs,johnvc/paylocity-jobs-api`

## Related discovery and job-data Actors

- Greenhouse Job Board API: https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills
- Workday Career Sites API: https://apify.com/johnvc/workday-career-sites-api?fpr=9n7kx3&fp_sid=awesomeskills
- iCIMS Careers API: https://apify.com/johnvc/icims-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
