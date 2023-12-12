# OpenAI

_The Hiphops OpenAI app exposes the full OpenAI REST API, usable within your pipelines._<br>
_Use OpenAIs capabilities for flows such as generating reports on your own data, automated release notes, generating readable explainers for PRs or even suggesting code changes._

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`openai`|-|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|No setup required|OpenAI API key passed into calls|

---

## Call: `api`

A simple proxy exposing the full OpenAI REST API.

Full documentation for the REST API can be found at [OpenAI API Reference](https://platform.openai.com/docs/api-reference).

**Call structure:**

```hcl
call openai_api {
  inputs = {
    method = "POST" // (Optional) string - The HTTP method (default GET)
    api_key = "the_api_key" // string - The OpenAI API key
    path = "chat/completions" // The endpoint path, for example: https://api.openai.com/v1/chat/completions
    json = {
      "model": "gpt-3.5-turbo",
      "messages": [
        {
          "role": "user",
          "content": "Quote the first line of Hamlets famous speech"
        }
      ]
    } // (Optional) object - The HTTP body as JSON (auto sets Content-Type header to application/json). Cannot be combined with data
    data = "" // (Optional) string - The HTTP body. Cannot be combined with json
    headers = {
      "Content-Type": "multipart/form-data"
    } // (Optional) An array of string keys and values - the HTTP headers
    params = {
      "engine": "davinci"
    } // (Optional) An array of string keys and values - the HTTP query parameters
    org_id = "the_org_id" // (Optional) string - The OpenAI organization ID
  }
}
```

**Example result:**

```js
{
  "context": "AppController",
  "hops": {
    "started_at": "2023-11-13T22:30:11.094Z",
    "finished_at": "2023-11-13T22:30:11.954Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "id": "chatcmpl-3FM7ANTHQZE118c3vTSEQWDuXdhIX",
    "object": "chat.completion",
    "created": 1699914611,
    "model": "gpt-3.5-turbo-0613",
    "choices": [
      {
        "index": 0,
        "message": {
          "role": "assistant",
          "content": "\"To be, or not to be: that is the question\""
        },
        "finish_reason": "stop"
      }
    ],
    "usage": {
      "prompt_tokens": 16,
      "completion_tokens": 13,
      "total_tokens": 29
    }
  },
  "status_code": 200,
  "headers": {
    "date": "Mon, 13 Nov 2023 22:30:11 GMT",
    "content-type": "application/json",
    "transfer-encoding": "chunked",
    "connection": "close",
    "access-control-allow-origin": "*",
    "cache-control": "no-cache, must-revalidate",
    "openai-model": "gpt-3.5-turbo-0613",
    "openai-organization": "hiphops-io",
    "openai-processing-ms": "521",
    "openai-version": "2020-10-01",
    "strict-transport-security": "max-age=15724800; includeSubDomains",
    "x-ratelimit-limit-requests": "5000",
    "x-ratelimit-limit-tokens": "90000",
    "x-ratelimit-remaining-requests": "4999",
    "x-ratelimit-remaining-tokens": "89971",
    "x-ratelimit-reset-requests": "12ms",
    "x-ratelimit-reset-tokens": "19ms",
    "x-request-id": "aCb7cdCADA30F6FBBe1f5ff6F81639Ba",
    "cf-cache-status": "DYNAMIC",
    "server": "cloudflare",
    "cf-ray": "6uGOyuRzisCUO19N-QSO",
    "alt-svc": "h3=\":443\"; ma=86400"
  }
}
```

> Note: The top level keys and `hops` object will be consistent across calls, but the content within them will be a transparent forwarding of the result from the OpenAI API. Refer to [OpenAI API Reference](https://platform.openai.com/docs/api-reference) for full result information per endpoint
