# Actor routing table

One Actor powers both Ashby skills; the skills differ in which output mode and question they are shaped for.

| Question | Skill | Actor input shape |
|---|---|---|
| Job rows with salary from named boards | apify-ashby-jobs-scraper | default `jobs` mode, `companies` list, filters |
| New postings since yesterday | apify-ashby-jobs-scraper | `publishedAfter: "25h"` on a schedule |
| Which companies use Ashby | apify-companies-using-ashby | `outputMode: "companiesOnly"`, directory sweep |
| Does this company have an Ashby board | apify-companies-using-ashby | `outputMode: "companiesOnly"`, your `companies` list |
| Cheapest full index of a board | either | `outputMode: "urlsOnly"` |

- Actor: https://apify.com/johnvc/ashby-job-board-scraper?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/ashby-job-board-scraper`
- Related family: Greenhouse Job Board API (https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills) shares the row shape for cross-ATS merging.
