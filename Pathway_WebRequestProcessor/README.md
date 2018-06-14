# Technical Description of agent Pathway_WebRequestProcessor

This agent is initiated by the server event "after documents are created or modified".

After a web request has completed the document will only be visible in All Documents.  This agent will find new request documents and perform some initial processing to make them visible.  It will also send an email to the requestor acknowledging receipt of the request.

## Step 1 - Set up audit log

There is a separate audit log database.  It displays the parameters of every incoming request.   So, if a request was submitted but for some reason the request document does not exist on the server, the audit log will contain all of the request parameters so the PATHway admin could locally create the request.  

```
Set auditLog = session.CreateLog("WebRequestProcessor")
Call auditLog.Opennoteslog(ProdServer, ProdPath & "pathway_audit_log.nsf")
```

## Step 2 - Check OOO status

There is a custom OOO option in PATHway, under the default Tools menu.  If the agent finds that the admin has set the OOO dates, the response to the user will include text noting this fact.

```
Set docOut = db.GetProfileDocument("OutOfOfficeProfile")
  If (Len(docOut.GetFirstItem("PW_OOO_Start").Text) > 0 And Len(docOut.GetFirstItem("PW_OOO_End").Text) > 0) Then
    If (thisDate.Timedifference(docOut.GetFirstItem("PW_OOO_Start").DateTimeValue) >= 0 And thisDate.Timedifference(docOut.GetFirstItem("PW_OOO_End").DateTimeValue) <= 0) Then
      oooMsg = "Note: The PATHway administrator is currently out of the office.  " & docOut.GetFirstItem("PW_OOO_Text").Text
      oooFlag = True

  End If		
End If
```

## Step 3 - Process new documents

The first step is to determine the document source.  Documents created by the web interface, or by the mobile app, will contain a special field.  It will be removed once it is identified.

```
If (doc.HasItem("PWayWebReq")) Then
  ...
If (doc.HasItem("PWayMobilebReq")) Then
  ...
```

The PWayISV field (name of the partner) will have extra processing.  If this is a new partner name, the value will be set into the existing name field.

```
If (doc.Hasitem("PWayNewISV")) Then
  If (doc.GetFirstItem("PWayNewISV").Text <> "") Then
    Call doc.Replaceitemvalue("PWayISV", doc.GetFirstItem("PWayNewISV").Text)
    Call doc.Removeitem("PWayNewISV")
  End If					
End If
```

Then BluePages is called to retrieve the requestor's Notes email address, if possible.

If a file was included with the request, it is extracted and repopulated into a RichTextItem so it is visible in the request form.

## Step 4 - Send response email

Two helper subroutines are called to build the MIME message body and send the document.

BuildMimeBody() constructs the mail body.

```
Set stream = session.CreateStream
Set body = docReply.CreateMIMEEntity

Set header = body.CreateHeader("Subject")
Call header.SetHeaderVal("Re:  " & doc.GetItemValue("Subject")(0))

Set header = body.CreateHeader("To")
Call header.SetHeaderVal(doc.GetItemValue("PWayAddr")(0))

...
```

SendTheDoc() send the new document.
