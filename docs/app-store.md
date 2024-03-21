# Store

_The Hiphops store app provides key/value file and data storage._

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`store`| - |:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Non, built in|Not required|

> Recipe ideas:
> - Use the store app with results from `template()` to generate and store reports such as security checks.
> - Store is also useful to share data between separate pipelines, or keep track of values.

## Setup instructions

No setup is required for this app. It is available to all Hiphops accounts and enabled by default.

---

## Call: `put`

Saves an object in the store, overwriting the current value if it already exists

**Call structure:**

```hcl
call store_put {
  inputs = {
    key = "some_file_or_data" // String - The name to store the data under
    value = "Hello world!" // String - The data to store
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
  "body": "{\"key\":\"some_file_or_data\"}", // For successful calls it will be populated with the key that was set
  "json": {
    "key": "some_file_or_data"
  }
}
```

---

## Call: `get`

Get fetches data from the store by name (key).

**Call structure:**

```hcl
call store_get {
  inputs = {
    key = "some_key" // String - The key of the data to get
    default = "My default value" // String (Optional) - A default value to return if the key isn't set
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
  "body": "...", // Raw string of the response body - json field below is usually more helpful
  "json": {
    "default": false, // Will be set to 'true' if a default value was returned/the key had no value
    "key": "my_key", // The key that was returned. Will be an empty string if no value found/default returned
    "value": "some value" // String containing the value retrieved from storage
  }
}
```

---

## Call: `delete`

Delete permanently removes an object from the store

**Call structure:**

```hcl
call store_delete {
  inputs = {
    key = "mykeytodelete" // String - The key of of the object to delete
    error_on_missing = false // Boolean (Optional) - Whether the response should be an error if the key doesn't exist. Defaults to true
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
