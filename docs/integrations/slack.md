# Slack

|Name|Incoming events|Has tasks|Integration|
|:-------|:-------|:-------|:-------|
|`slack`|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Integrated in a few clicks via the UI under project settings|

_Integrating slack allows you to send messages to slack channels. It also allows you to create custom commands using the `/hiphops` slash command. This can be used to trigger sensors and perform tasks by creating a sensor in your `hiphops.yml` file._

## Event: `command`

actions: Any command your user provides to the `/hiphops` slash command will be used to populate the `hiphops.action` field.
For example, `/hiphops deploy` will have the action `deploy`.

A command entered on Slack such as `/hiphops foo arg1 arg2` would generate an event with the following structure:

```js
{
  "project_id": "12345677-0123-aaaa-bbbb-123456abcdef", // Your project's UUID
  "hiphops": {
    "source": "slack",
    "event": "command",
    "action": "foo", // User defined command
  },
  "command": "foo", // User defined command
  "args": ["arg1", "arg2"], // An array of strings
  "response_url": "https://hooks.slack.com/commands/T1234567ABC/12345678912345/T123abcDEF1234567", // A time limited URL to respond to message directly
  "trigger_id": "some_id", // Can be used to trigger a modal
  "team_id": "some_team_id", // The team from which the command originated
  "channel_id": "some_channel_id", // The channel from where the command originated
  "user_id": "some_user_id", // The user that sent the command
  "is_enterprise_install": "false" // String "true" | "false" - Whether the slack instance is an enteprise install
  "enterprise_id": "some_enterprise_id" // Optional string, The enterprise ID if set
}
```

###### Example sensor

This sensor simply posts a message to the `#general` slack channel when the `/hiphops` command is used, echoing the command and arguments.

```yaml
---
resource: sensor
id: slack command receiver
when:
  event.hiphops.source: slack
  event.hiphops.event: command
tasks:
  - name: slack.post_message
    input:
      channel: general
      $: "({text: `Command: ${event.command}, Args: ${event.args}`})"
```

---

## Task: `post_message`

Posts a message to slack using the slack message API. The message can be a simple string or a complex object. The message object is documented here: [Slack messaging payload documentation](https://api.slack.com/reference/messaging/payload).

```yaml
tasks:
  - name: slack.post_message
    input:
      channel: general # String - The name of the channel to post to
      text: "Hello world" # String - The message to post, which conforms to the slack payload format
```

###### Responds with

Provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).
Additionally, responds with a `vars` object containing a key which is the ID of the message.

###### Example vars

```js
{
  "0": {
    "ok": true,
    "channel": "C12345678901",
    "ts": "1678955555.666666",
    "message": {
      "bot_id": "B12345678901",
      "type": "message",
      "text": "1 new commit pushed to refs/heads/sandbox\n26209db",
      "user": "U12345678901",
      "ts": "1678955555.666666",
      "app_id": "A12345678901",
      "blocks": [
        {
          "type": "rich_text",
          "block_id": "oADM",
          "elements": [
            {
              "type": "rich_text_section",
              "elements": [
                {
                  "type": "text",
                  "text": "1 new commit pushed to refs/heads/sandbox\n26209db"
                }
              ]
            }
          ]
        }
      ],
      "team": "T12345678901",
      "bot_profile": {
        "id": "B12345678901",
        "app_id": "A12345678901",
        "name": "Hiphops",
        "icons": {
          "image_36": "https://avatars.slack-edge.com/2022-10-15/4227360167490_aa45faf3342d6ce1adf0_36.png",
          "image_48": "https://avatars.slack-edge.com/2022-10-15/4227360167490_aa45faf3342d6ce1adf0_48.png",
          "image_72": "https://avatars.slack-edge.com/2022-10-15/4227360167490_aa45faf3342d6ce1adf0_72.png"
        },
        "deleted": false,
        "updated": 1675441190,
        "team_id": "T12345678901"
      }
    },
    "response_metadata": {
      "scopes": [
        "chat:write",
        "commands",
        "chat:write.public",
        "channels:read"
      ],
      "acceptedScopes": [
        "chat:write"
      ]
    }
  }
}
```


---

## Task: `send_response`

Sends a response message to slack using a `response_url` provided in response to other slack interactions - e.g. as a part of the `command` event, allowing response to be given to the user that used the command. The response can be a simple string or a complex object. The message object is documented here: [Slack messaging payload documentation](https://api.slack.com/reference/messaging/payload).

```yaml
tasks:
  - name: slack.send_response
    input:
      response_url: "https://hooks.slack.com/commands/T02NVHE2ERJ/4902701257719/UNR6kqwSF5fCTH70RxWUe9M9" # String - The slack response URL to post to (will be valid for use 5 times, for 30 minutes from the time you receive it)
      text: "Hello world" # String (optional) - A simple text message to respond with (if not provided, `response_payload` must be provided)
      response_payload: "{ 'text': 'Some text' }" # Object (optional) - A complex object conforming to the Slack messaging format.
      send_to_channel: true # Boolean (optional) - If false, the response will be sent as an ephemeral response, only visible to the user being responded to. If true, it will be sent to the channel the original message is in. Defaults to false.
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).