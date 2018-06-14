# Technical Description of agent PathwayWebFollowEnhanced

This agent is on a fixed schedule, 2AM EST each Sunday.  It looks for requests that should have finished engagements recently and sends the requestor a follow-up email.

## Step 1 - Gather eligible requests

There is a view that is constructed to display request documents where the engagement date is between 7 and 15 days in the past.  
For example, the agent runs on June 17.  The view returns documents with an end date in the range June 3 - June 9.

```
Set vw = db.GetView("Pathway Followup Search")
Set vwc = vw.AllEntries
```

The view formula:

```
rangeStart := @Adjust(@Today; 0; 0; -15; 0; 0; 0);
rangeEnd := @Adjust(@Today; 0; 0; -7; 0; 0; 0);
checkDate := @If(PWayEndNeeded > rangeStart & PWayEndNeeded < rangeEnd; 1; 0);
SELECT Form = "PWayData" & ! @IsAvailable(PWayEngDate) & ! @IsAvailable(PWaySurvey) & checkDate = 1 & PWayStatus != "Cancelled" & PWayClosed = "1"
```

## Step 2 - Document validation

There are a couple of validation checks to verify the request should receive a follow-up.  For example the PATHway admin might have created some requests with the Batch Processor and those would not have an explicit requestor.

## Step 3 - Send the follow-up

Similar to the WebRequestProcessor, an email is constructed and sent via the helper subroutines BuildMimeBody() and SendTheDoc().  In this context there is an additional subroutine that assists with construction of the email body text, PathwayFollowupText().
