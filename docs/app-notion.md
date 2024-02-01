# Notion

_Integrating Notion allows Hiphops pipelines access to the full variety of the Notion API._<br>
_Allow querying and updating databases, creating pages and updating blocks, among other capabilities._

| Name     | Listener | Worker                                  | Setup                           | Auth                             |
| :------- | :------- | :-------------------------------------- | :------------------------------ | :------------------------------- |
| `notion` | -        | :white_check_mark:&nbsp;&nbsp;&nbsp;Yes | Add via hiphops.io account page | Credential free (via Notion App) |

---

## Call: `api`

A simple proxy exposing the full Notion API through an authenticated instance of the Hiphops Notion App.

Full documentation for the API can be found at [Notion API Reference](https://developers.notion.com/reference/intro).

**Call structure:**

```hcl
call notion_api {
  inputs = {
    method = "PATCH" // (Optional) string - The HTTP method (default GET)
    path = "/v1/pages/60bdc8bd-3880-44b8-a9cd-8a145b3ffbd7" // The endpoint path, for example: https://developers.notion.com/reference/patch-page
    json = {
      "properties": {
        "In stock": { "checkbox": true }
      }
    } // (Optional) object - The HTTP body as JSON (auto sets Content-Type header to application/json). Cannot be combined with data
    data = "" // (Optional) string - The HTTP body. Cannot be combined with json
    headers = {
      "Content-Type": "multipart/form-data"
    } // (Optional) An array of string keys and values - the HTTP headers
  }
}
```

**Example result:**

```js
{
  "body": "",
  "completed": true,
  "done": true,
  "errored": false,
  "headers": {
    "access-control-allow-origin": "*",
    "connection": "close",
    "content-type": "application/json; charset=utf-8",
    "date": "Tue, 12 Dec 2023 14:10:59 GMT",
    "transfer-encoding": "chunked",
    "vary": "Accept-Encoding",
    "x-notion-request-id": "fa6047aa-706f-49fa-8d78-6dcbb29489ba",
    "x-powered-by": "Express",
    "x-render-origin-server": "cloudflare"
  },
  "hops": {
    "error": null,
    "finished_at": "2023-12-12T14:10:59.264Z",
    "started_at": "2023-12-12T14:10:52.301Z"
  },
  "json": {
    "has_more": false,
    "next_cursor": null,
    "object": "list",
    "request_id": "fa6047aa-706f-49fa-8d78-6dcbb29489ba",
    "results": [
      {
        "avatar_url": "https://lh3.googleusercontent.com/a-/somethingsomething",
        "id": "f3add412-6804-4e44-9f50-2dd7b37c0bd6",
        "name": "Gene Hackman",
        "object": "user",
        "person": {
          "email": "someone@example.com"
        },
        "type": "person"
      },
      {
        "avatar_url": null,
        "id": "37f4d69d-0dd6-4ce7-a7ba-5d94d5eb4ac2",
        "name": "Florence Pugh",
        "object": "user",
        "person": {
          "email": "someoneelse@example.com"
        },
        "type": "person"
      },
      {
        "avatar_url": "https://s3-us-west-2.amazonaws.com/public.notion-static.com/44a0b2f6-14f8-4219-a3ec-ec2e9737af36/Logo_-_square-512.png",
        "bot": {},
        "id": "250ba529-aa52-4c7a-ba98-91393b6c47fc",
        "name": "Hiphops",
        "object": "user",
        "type": "bot"
      },
    ],
    "type": "user",
    "user": {}
  },
  "status_code": 200
}
```

> Note: The top level keys and `hops` object will be consistent across calls, but the content within them will be a transparent forwarding of the result from the Notion API. Refer to [Notion API Reference](https://developers.notion.com/reference/intro) for full result information per endpoint
