# Technical Description of agent PathwayWebViewDisplayDirect

This agent is called from various sources:

* Notes client, with random set of report filters
* Web Navigator page, showing a single selected request page
* Web Navigator page, showing all of the requests either created by the user, or assigned to the user as a resource

The query parameters govern the behavior of the agent.

* Notes client: &ReportName=CustomReport&ReportUser=jhammer@ca.ibm.com&PWayGEO=Latin%20America
* Single report: &ReportName=StatusReport&ReportUser=Joni Mitchell&PWayID=24223
* All for owner or resource: &ReportName=RequestorReport&ReportUser=

## Step 1 - Input Processing

Process the query parameters.

```
Set doc = session.Documentcontext
qs = StrRight(doc.Query_String(0), "&")
```

The first two parameters are fixed so they are removed from the query string and loaded in variables.  The query parameters array is then collapsed so only the custom parameters remain.

## Step 2 - Validation

Look for the PATHway cookie.

```
cookiePathway = getCookieVal(doc.HTTP_COOKIE(0), "PATHwayUser")
cookiePathwayAddr = StrLeft(cookiePathway, "|")
cookiePathwayName = StrRight(cookiePathway, "|")
```

Extract the db's ACL.

```
Set acl = db.Acl
Set acle = acl.Getfirstentry()
```

Now test access.  Users who are ACL Managers don't need a cookie.  Anyone else without a matching cookie will be rejected.  
There is an occasional problem where the HTTP_COOKIE value isn't presented to the agent, even though the user successfully authenticated.  Closing and reopening the browser, or switching the browser, will usually resolve this issue.

## Step 3 - Report Selection

Once the user is validated the requested report is generated.  In the code, 'qs' is the rebuilt query string.

```
Select Case reportName
  Case "CustomReport"
    Call CustomReport(qs, "notes")    
  Case "RequestorReport"
    qs = "PWayTAM=" & cookiePathwayName
    Call CustomReport(qs, "web")
  Case "StatusReport"
    qs = qs & "&PWayTAM=" & cookiePathwayName
    Call StatusDisplay(cookiePathwayName, qs, mgrAccess)
End Select
```

## Step 4 - Report Selection

### Report CustomReport

This report is available in the Notes client and allows the user to provide a wide range of report filters.  Don't think it's used.
Calls the CustomReport() subroutine.

### Report RequestorReport

This report returns a table displaying all of the requests either submitted by the user or where the user is the primary resource.
Calls the CustomReport() subroutine.

### Report StatusReport

This report returns a single page showing a request summary.
Calls the StatusDisplay() subroutine.
