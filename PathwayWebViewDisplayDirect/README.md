# Technical Description of agent PathwayWebViewDisplayDirect

This agent is called from various sources:

* Notes client, with random set of report filters
* Web Navigator page, showing a single selected request page
* Web Navigator page, showing all of the requests either created by the user, or assigned to the user as a resource

The query parameters govern the behavior of the agent.

* Notes client: &ReportName=CustomReport&ReportUser=jhammer@ca.ibm.com&PWayGEO=Latin%20America
* Single report: &ReportName=StatusReport&ReportUser=Joni Mitchell&PWayID=24223
* All for owner or resource: &ReportName=RequestorReport&ReportUser=

## Step 1

Process the query parameters.

```
Set doc = session.Documentcontext
qs = StrRight(doc.Query_String(0), "&")
```

The first two parameters are fixed so they are removed from the query string and loaded in variables.  The query parameters array is then collapsed so only the customer paramters remain.
