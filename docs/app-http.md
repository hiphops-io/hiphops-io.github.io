# HTTP

_The Hiphops HTTP app enables you to make calls via http/s. Useful for calling custom API endpoints, or fetching data from websites._

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`http`| - |:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Non, built in|Not required|

> Recipe ideas: Pair the HTTP app with Hiphops `schedule` and `slack` messages to create a downtime detector, or even automated smoke tests against your own deployed stack.

## Setup instructions

No setup is required for this app. It is bundled with Hiphops and enabled by default. You can disable this with `hops start --serve-httpapp=false`.

## Via config.yaml

You can configure all options via Hiphops' `config.yaml` too:

```yaml
http:
  serve: true # Default true. Set to false to disable the app in a hops instance
```

---

## Call: `do`

Performs an http request and responds with the result

**Call structure:**

```hcl
call http_do {
  inputs = {
    url = "https://example.com/foo" // String - The full URL including protocol and path you wish to call. Query parameters are configured separately
    method = "GET" // (Optional) string - defaults to "GET". One of "GET" "POST" "PUT" "DELETE" "PATCH" "OPTIONS" "HEAD"
    params = { // (Optional) key/value string pairs - will be added as query string params and URL encoded
      "foo" = "bar"
    }
    headers = { // (Optional) key/value string pairs
      "content-type" = "application/json"
    }
    retries = 0 // (Optional) integer from 0-3. Number of retries (with exponential backoff). 1 retry = 2 attempts (initial request + a retry)
    data = "Hello" // (Optional) string - request body - If JSON is also set, it will take precedence
    json = { // (Optional) object - will be sent as json encoded request body - If data is also set, JSON will take precedence
      greeting = hello
    }
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
  "status_code" 200,
  "body": "", // Will be populated with the raw string body from the response
  "json": { "greeting": "Hello World!" } // If response content-type is application/json and the body is valid json, this will contain that body as an object
  "url": "https://example.com/foo" // The URL that was called
  "headers": { // Response headers as key/value string pairs. Multiple values per header are concatenated as a single string separated by ", "
    "Content-Type": "application/json; charset=utf-8",
    "Access-Control-Allow-Credentials":"true",
  }
}
```

---
