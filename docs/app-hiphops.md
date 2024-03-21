# Hiphops

_The Hiphops app allows you to interact with Hiphops itself, such as creating new source events_

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`hiphops`| - |:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Non, built in|Not required|

> Recipe ideas: Use as a flow control mechanism, for example to create looped pipelines

## Setup instructions

The hiphops app is available to all Hiphops accounts and enabled by default.

---

## Call: `send_event`

Sends a source event, which can in turn be reacted to by `on` blocks in your automations.

**Call structure:**

```hcl
call hiphops_send_event {
  inputs = {
    event = "myevent" // String - Event name (e.g. pullrequest) - lowercase alphanumeric only.
    action = "happened" // String (Optional) - Event action (e.g. opened) - lowercase alphanumeric & underscores only
    content = { // Object with string keys (Optional) - The event payload
      foo = ""
    }
  }
}
```

**Example:**

```hcl
on foo {
  call hiphops_send_event {
    inputs = {
      event = "greeting"
      action = "sent"
      content = {
        greeter_said = "Hello!"
      }
    }
  }
}

// This will be triggered by the above send_event
on greeting {
  call slack_post_message {
    inputs = {
      channel = "random"
      // Here we grab the 'greeter_said' value from the event's content and use it
      // to send a message in Slack
      text = event.greeter_said
    }
  }
}

// This will also be triggered by the above send_event
on greeting_sent {...}

// This will not be triggered, as `received` doesn't match the action
on greeting_received {...}
```


<!-- **Example loop:**

Using `hiphops_send_event` it's possible to create looped workflows. In the near future we'll be adding prettier syntax that hides some of this plumbing away from you, but for now


>! We do not prevent you from creating pipelines that send events that they themselves react to. Be careful with your logic and guard against infinite loops. Looping is an advanced use case and requires caution. -->


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
  "json": {
    "sent": 
  }
}
```

---
