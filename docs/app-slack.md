# Slack

_Integrating Slack enables you to automate common flows such as notifying when a pipeline ends or sending messages to users._<br>
_As Hiphops exposes the full Slack REST API, the vast majority of Slack based flows can be created as part of Hiphops pipelines._

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`slack`|-|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Add via hiphops.io account page|Credential free (via Slack App)|


> Note: The structure of Slack events is mostly identical to the events directly emitted by Slack.<br>
> Whilst we'll show examples here, the Slack docs are usually more exhaustive for exact event structures.

---

## Call: `post_message`

Posts a message to slack.

**Call structure:**

```hcl
call slack_post_message {
  inputs = {
    channel = "general" // (Optional) string - The channel to post to. Must provide one of channel and channel_id
    channel_id = "C12345678901" // (Optional) string - The ID of the channel to post to. Must provide one of channel and channel_id
    text = "Hello world" // (Optional) string - The message to post. One of text, attachments and blocks must be provided
    attachments = [] # (Optional) array - An array of attachments. One of text, attachments and blocks must be provided
    blocks = [] # (Optional) array - An array of blocks. One of text, attachments and blocks must be provided
    thread_ts = "1699915731.367659" // (Optional) string - The timestamp of the message to reply to. This comes from a previous post_message call
  }
}
```

**Example result:**

```js
{
  "context": "AppService",
  "hops": {
    "started_at": "2023-11-13T22:48:50.970Z",
    "finished_at": "2023-11-13T22:48:51.433Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "ok": true,
    "channel": "CC7VAL05QBX",
    "ts": "1699915731.367659",
    "message": {
      "bot_id": "B1PW699QRLS",
      "type": "message",
      "text": "Hello world",
      "user": "UDITT4ER6UG",
      "ts": "1699915731.367659",
      "app_id": "ALTWA3CWFIH",
      "blocks": [
        {
          "type": "rich_text",
          "block_id": "4rV1",
          "elements": [
            {
              "type": "rich_text_section",
              "elements": [{ "type": "text", "text": "Hello world" }]
            }
          ]
        }
      ],
      "team": "TMDM4I31SFD",
      "bot_profile": {
        "id": "B1PW699QRLS",
        "app_id": "ALTWA3CWFIH",
        "name": "Hiphops",
        "icons": {
          "image_36": "https://avatars.slack-edge.com/2023-10-18/6059604623780_f8fa96b9cbe29d7f98e3_36.png",
          "image_48": "https://avatars.slack-edge.com/2023-10-18/6059604623780_f8fa96b9cbe29d7f98e3_48.png",
          "image_72": "https://avatars.slack-edge.com/2023-10-18/6059604623780_f8fa96b9cbe29d7f98e3_72.png"
        },
        "deleted": false,
        "updated": 1697626969,
        "team_id": "TMDM4I31SFD"
      }
    },
    "response_metadata": {
      "scopes": [
        "chat:write",
        "chat:write.public",
        "commands",
        "channels:read"
      ],
      "acceptedScopes": ["chat:write"]
    }
  }
}
```

For full details on the payload see [Slack messaging payload documentation](https://api.slack.com/reference/messaging/payload)

---

## Call: `update_message`

Updates an existing message.

```hcl
call slack_update_message {
  inputs = {
    channel_id = "C12345678901" // (Optional) string - The ID of the channel to post to
    text = "Hello world" // (Optional) string - The message to post. One of text, attachments and blocks must be provided
    attachments = [] # (Optional) array - An array of attachments. One of text, attachments and blocks must be provided
    blocks = [] # (Optional) array - An array of blocks. One of text, attachments and blocks must be provided
    ts = "1699915731.367659" // string - The timestamp of the message to update. This comes from a previous call
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-14T17:27:16.834Z",
    "finished_at": "2023-11-14T17:27:17.536Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "ok": true,
    "channel": "CC7VAL05QBX",
    "ts": "1699982264.011329",
    "text": "Updated Hello world now",
    "message": {
      "bot_id": "B1PW699QRLS",
      "type": "message",
      "text": "Updated Hello world now",
      "user": "UDITT4ER6UG",
      "app_id": "ALTWA3CWFIH",
      "blocks": [
        {
          "type": "rich_text",
          "block_id": "4rV1",
          "elements": [
            {
              "type": "rich_text_section",
              "elements": [
                { "type": "text", "text": "Updated Hello world now" }
              ]
            }
          ]
        }
      ],
      "team": "TMDM4I31SFD",
      "bot_profile": {
        "id": "B1PW699QRLS",
        "app_id": "ALTWA3CWFIH",
        "name": "Hiphops Sandbox",
        "icons": {
          "image_36": "https://avatars.slack-edge.com/2023-10-18/6059604623780_f8fa96b9cbe29d7f98e3_36.png",
          "image_48": "https://avatars.slack-edge.com/2023-10-18/6059604623780_f8fa96b9cbe29d7f98e3_48.png",
          "image_72": "https://avatars.slack-edge.com/2023-10-18/6059604623780_f8fa96b9cbe29d7f98e3_72.png"
        },
        "deleted": false,
        "updated": 1697626969,
        "team_id": "TMDM4I31SFD"
      },
      "edited": { "user": "B1PW699QRLS", "ts": "1699982837.000000" }
    },
    "response_metadata": {
      "scopes": [
        "chat:write",
        "chat:write.public",
        "commands",
        "channels:read"
      ],
      "acceptedScopes": ["chat:write"]
    }
  }
}
```

For full details on the payload see [Slack messaging payload documentation](https://api.slack.com/reference/messaging/payload)
