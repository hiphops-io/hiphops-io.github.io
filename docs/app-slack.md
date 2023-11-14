# Slack

_Integrating Slack enables you to automate common flows such as notifying when a pipeline ends or sending messages to users._<br>
_As Hiphops exposes the full Slack REST Web API, the vast majority of Slack based flows can be created as part of Hiphops pipelines._

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`slack`|-|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Add via hiphops.io account page|Credential free (via Slack App)|

---

## Call: `api`

Calls the Slack REST Web API with the given parameters.

**Call structure:**

```hcl
call slack_api {
  inputs = {
    method = "POST" // (Optional) string - The HTTP method (default GET)
    path = "conversations.list" // The endpoint path
    json = {} // (Optional) object - The HTTP body as JSON (auto sets Content-Type header to application/json). Cannot be combined with data
    data = "" // (Optional) string - The HTTP body. Cannot be combined with json
    headers = {
      "Content-Type": "multipart/form-data"
    } // (Optional) An array of string keys and values - the HTTP headers
    params = {
      "key": "value"
    } // (Optional) An array of string keys and values - the HTTP query parameters
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-14T17:44:45.172Z",
    "finished_at": "2023-11-14T17:44:45.403Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "ok": true,
    "channels": [
      {
        "id": "C043KSV1F1S",
        "name": "general",
        "is_channel": true,
        "is_group": false,
        "is_im": false,
        "is_mpim": false,
        "is_private": false,
        "created": 1638108664,
        "is_archived": true,
        "is_general": false,
        "unlinked": 0,
        "name_normalized": "general",
        "is_shared": false,
        "is_org_shared": false,
        "is_pending_ext_shared": false,
        "pending_shared": [],
        "context_team_id": "T02NVHE2ERJ",
        "updated": 1652366946835,
        "parent_conversation": null,
        "creator": "U043AR3145C",
        "is_ext_shared": false,
        "shared_team_ids": ["T02NVHE2ERJ"],
        "pending_connected_team_ids": [],
        "is_member": false,
        "topic": {
          "value": "General discussion.",
          "creator": "U043AR3145C",
          "last_set": 1641981040
        },
        "purpose": {
          "value": "General",
          "creator": "U043AR3145C",
          "last_set": 1638108665
        },
        "previous_names": ["general1"],
        "num_members": 0
      },
      {
        "id": "C02NVHE2M52",
        "name": "specific",
        "is_channel": true,
        "is_group": false,
        "is_im": false,
        "is_mpim": false,
        "is_private": false,
        "created": 1638108476,
        "is_archived": false,
        "is_general": true,
        "unlinked": 0,
        "name_normalized": "specific",
        "is_shared": false,
        "is_org_shared": false,
        "is_pending_ext_shared": false,
        "pending_shared": [],
        "context_team_id": "T02NVHE2ERJ",
        "updated": 1689072147105,
        "parent_conversation": null,
        "creator": "U043AR3145C",
        "is_ext_shared": false,
        "shared_team_ids": ["T02NVHE2ERJ"],
        "pending_connected_team_ids": [],
        "is_member": false,
        "topic": {
          "value": "",
          "creator": "U043AR3145C",
          "last_set": 1688127252
        },
        "purpose": {
          "value": "Specific discussion.",
          "creator": "U043AR3145C",
          "last_set": 1638108476
        },
        "properties": {
          "canvas": {
            "file_id": "F1PW699QRLS",
            "is_empty": true,
            "quip_thread_id": "VF9X546HG8F"
          }
        },
        "previous_names": ["specific1"],
        "num_members": 6
      }
    ],
    "response_metadata": { "next_cursor": "" }
  },
  "status_code": 200,
  "headers": {
    "date": "Tue, 14 Nov 2023 17:44:45 GMT",
    "server": "Apache",
    "vary": "Accept-Encoding",
    "x-slack-req-id": "SOOT8UH76NVEXFJ6W9X4E965MNA9KB7I",
    "x-content-type-options": "nosniff",
    "x-xss-protection": "0",
    "pragma": "no-cache",
    "cache-control": "private, no-cache, no-store, must-revalidate",
    "expires": "Sat, 26 Jul 1997 05:00:00 GMT",
    "content-type": "application/json; charset=utf-8",
    "x-accepted-oauth-scopes": "channels:read,groups:read,mpim:read,im:read,read",
    "x-oauth-scopes": "chat:write,chat:write.public,commands,channels:read",
    "access-control-expose-headers": "x-slack-req-id, retry-after",
    "access-control-allow-headers": "slack-route, x-slack-version-ts, x-b3-traceid, x-b3-spanid, x-b3-parentspanid, x-b3-sampled, x-b3-flags",
    "strict-transport-security": "max-age=31536000; includeSubDomains; preload",
    "referrer-policy": "no-referrer",
    "x-slack-unique-id": "ZJFK4924QMJTC8QE5MN4S1EYXWH",
    "x-slack-backend": "r",
    "access-control-allow-origin": "*",
    "content-length": "2124",
    "x-envoy-attempt-count": "1",
    "x-envoy-upstream-service-time": "168",
    "x-server": "slack-www-hhvm-main-iad-iysm",
    "x-slack-shared-secret-outcome": "no-match",
    "x-edge-backend": "envoy-www",
    "x-slack-edge-shared-secret-outcome": "no-match",
    "connection": "close"
  }
}
```

Full documentation for the REST Web API can be found [Using the Slack Web API](https://api.slack.com/web). Methods can be found at [Web API Methods](https://api.slack.com/methods).

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
