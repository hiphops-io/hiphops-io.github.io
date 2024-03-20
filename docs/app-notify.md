# Store

_The Hiphops notify app provides enables your automations to send email notifications_

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`notify`| - |:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Non, built in|Not required|

> Recipe ideas: Use with schedules to periodically generate reports and send them as email attachments

## Setup instructions

The notify app is available to all Hiphops accounts and enabled by default, however you will need to request domains/email addresses to be added to your allowed recipients before use.

Just email support@hiphops.io with the list of domains/specific email addresses you'd like to be approved for.

Note: For public email services such as @gmail.com @hotmail.com etc, we will only approve specific full email addresses.

---

## Call: `send_email`

Saves an object in the store, overwriting the current value if it already exists

**Call structure:**

```hcl
call notify_send_email {
  inputs = {
    to = ["someone@example.com"] // List of strings - Recipient email addresses
    cc = ["someonelse@example.com", "hey@example.com"] // List of strings (Optional) - CC recipient email addresses
    bcc = ["private@example.com"] // List of strings (Optional) - BCC recipient email addresses
    reply_to = "me@example.com" // String (Optional) - Email address for replies
    sender_name = "Me" // String (Optional) - Display name for sender (defaults to "Hiphops Notification")
    subject = "About that thing..." // String (Optional) - Email subject
    content = "I am emailing to let you know that..." // String (Optional) - Plain text content of the email. One or both of `content` or `html_content` must be set
    html_content = "<p>I am emailing to let you know that...</p>" // String (Optional) - HTML content of the email. One or both of `content` or `html_content` must be set
    send_at = 1710950426 // Integer (Optional) - Unix timestamp set in the future, used to schedule the email for future sending.
    attachments = [ // List of attachments (Optional) - File attachments to add to the email
      {
        content = "Some file content" // String - The content of the attachment as a string
        type = "text/plain" // String (Optional) - The MIME type of the attachment e.g. "application/pdf". If unset, Hiphops will attempt to detect the type automatically based on the extension in the filename, falling back to "text/plain" otherwise.
        filename = "notes.txt" // String - The name to give the file attachment
      }
    ]
  }
}
```

**Example call:**

```hcl
call notify_send_email {
  inputs = {
    to = ["bob@fam.com"]
    subject = "Greetings"
    content = "Hi Bob!"
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": ""
}
```

---
