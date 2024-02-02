# Syntax

Here's a super simple example of the .hops syntax that shows how pipelines hang together.

```hcl
// Defines the event (pullrequest) and action (closed) that triggers this pipeline
on pullrequest_closed {
  if = event.pull_request.merged

  // Call the slack app on the post_message handler function
  call slack_post_message {
    inputs = {
      channel = "engineering"
      // The text input to Slack, with some data being grabbed from the source event
      text = "PR ${event.number} was merged! View it here: ${event.json.pull_request.html_url}"
    }
  }
}
```

> Note: The events and app calls available to you will depend on the apps you've integrated.<br>
> Look under each app on the left for specific event structures, calls they expose, and other app specific information.

`on` is a type of block. Blocks sometimes have words immediately after them (like `pullrequest_closed` above). This is called the label.<br>

`inputs` is an attribute. Attributes have `=` immediately after them.

Some blocks can appear inside other blocks, as above with `call slack_post_message` inside the `on pullrequest_closed` block.

Each block is described below, along with the attributes and blocks that are valid to appear within them.

## Pipelines `[block:on]`

`on` blocks define a pipeline that will run in response to an event. Events can come from third parties such as GitHub via apps, or from your own tasks.

`on` blocks can be defined at the top level of a .hops config and accepts the following:

| Name      |    Type     |      Required      |      Multiple      |           Example            |
| :-------- | :---------: | :----------------: | :----------------: | :--------------------------: |
| **label** |   `Label`   | :white_check_mark: |         -          |       `on a_label {}`        |
| **name**  | `Attribute` |         -          |         -          | `name = "my_pipeline_name"`  |
| **if**    | `Attribute` |         -          |         -          |         `if = true`          |
| **call**  |   `Block`   | :white_check_mark: | :white_check_mark: | `call slack_post_message {}` |
| **done**  |   `Block`   |         -          | :white_check_mark: |  `done {result = "Woohoo"}`  |

Example `on` block:

```hcl
on event_action {
  name = "my_pipeline" // Optional
  if = true // Optional

  call foo_bar {...}
}
```

---

### Pipeline/on block fields

#### Label <small>`required`</small>

The label defines the event that triggers an `on` block, plus an optional action (effectively a sub-event type).<br>
The event and action are separated by the first underscore, if there is one.

It must meet the following validation rules:

[Label validation](../_snippets/valid_label.md ':include')

Example `label`:

`on pullrequest {...}` would be triggered by any `pullrequest` event, such as `closed`, `opened` etc

`on pullrequest_closed {...}` would be triggered by `pullrequest` `closed` events only.

> Available events and actions are defined in the docs for the corresponding app.

#### Name <small>`optional`</small>

The `name` attribute provides a more readable name for the pipeline in logs and the UI.

It must meet the same validation rules as `labels`:

[Label validation](../_snippets/valid_label.md ':include')

Example `name`:

```hcl
on event {
  name = "handle_event"
  ...
}
```

#### If <small>`optional`</small>

The `if` attribute allows further filtering on events to decide if the pipeline should run.
It accepts an expression or value that resolves to a single boolean value of `true` or `false`.

Example `if`:

```hcl
on event {
 if = event.myvalue == "foo"
 ...
}
```

For more details, see the [section on 'if' attributes](#conditional-attributeif)

#### Call <small>`required`</small>

Nested `call` blocks define the actual work to be done within a pipeline. They call apps and their handlers with input to perform work.

Calls are executed in parallel by default, but you have full control over dependencies between them. This is described in detail in the `call` docs below.

> 'App' doesn't necessarily mean a SaaS app. It's simply something that listens for work and/or provides source events. `k8s` is an example of an app that is baked into hiphops, which allows you to execute container based workloads on your own Kubernetes cluster.

Example `call`:

```hcl
on event {
 call app_function {
  inputs = {...}
 }

 call otherapp_do_thing {
  inputs = {...}
 }
}
```

For more details, see the [section on 'Call' blocks](#calls-blockcall)

#### Done <small>`optional`</small>

Nested `done` blocks define the exit scenarios for a pipeline, with their result or error.

Example `done`:

```hcl
on event {
 call app_do_thing {
  name = "one"
  ...
 }

 call otherapp_do_thing {
  name = "two"
  ...
 }

 // Pipeline will immediately exit and error if either call errors
 // else it will return the result of the second call as an object
 done {
  error = one.hops.error || two.hops.error
  result = two.json
 }
}
```

For more details, see the [section on 'Done' blocks](#done-blockdone)

---

## Calls `[block:call]`

`call` blocks are how you execute work in a pipeline. They call app workers (e.g. `github_create_pr`) and present results back into your pipeline.

`call` blocks can appear within an `on` block and accept the following:

| Name       |    Type     |      Required      | Multiple |        Example        |
| :--------- | :---------: | :----------------: | :------: | :-------------------: |
| **label**  |   `Label`   | :white_check_mark: |    -     | `call app_handler {}` |
| **name**   | `Attribute` |         -          |    -     |  `name = "do_thing"`  |
| **if**     | `Attribute` |         -          |    -     |      `if = true`      |
| **inputs** | `Attribute` | :white_check_mark: |    -     |     `inputs = {}`     |

Example `call` block:

```hcl
on event_action {
  call foo_bar {
    name = "barred_foo" // Optional
    if = true // Optional

    inputs = {
      hello = "world"
      meaning = 42
    }
  }
}
```

By default, `call` blocks are executed in parallel.<br>
Using the `if` statement allows you to create dependencies between `calls` in addition to deciding if a call will execute at all for a given pipeline run.

A single `call` block will only ever execute once per pipeline run, even if the workload is shared across multiple hops instances.

---

### Call block fields

#### Label <small>`required`</small>

The label defines the app and handler that a `call` block executes.
The app and handler are separated by the first underscore.

It must meet the following validation rules:

[Label validation](../_snippets/valid_label.md ':include')

Example `label`:

`call slack_post_message {...}` would call the slack app's post_message handler.

> Available events and actions are defined in the docs for the corresponding app.

#### Name <small>`optional`</small>

The `name` attribute provides a more readable name for the pipeline in logs and the UI. Additionally, names are used to reference the results of earlier calls in a pipeline.

It must meet the same validation rules as `labels`:

[Label validation](../_snippets/valid_label.md ':include')

Example `name`:

```hcl
call github_create_pr {
  name = "new_pr"
  ...
}
```

#### If <small>`optional`</small>

The `if` attribute decides whether (and when) a call should be executed.
It accepts an expression or value that resolves to a single boolean value of `true` or `false`.

Using `if` in `call` blocks allows you to create full DAG workflows.

Example `if`:

```hcl
call github_api {
 if = event.hops.action == "closed" // Call will only execute if the event action is 'closed'
 ...
}
```

```hcl
call someapp_run {
  name = "apprun"
}

call otherapp_run {
  if = apprun.done // Call will execute when the first call finishes, creating an ordering between the two calls
}

call otherapp_run {
  if = apprun.completed // Call will execute if/when the first call successfully completes
}
```

For more details, see the [section on 'if' attributes](#conditional-attributeif)

#### Inputs <small>`required`</small>

The `inputs` attribute is a key value object defining the inputs to be passed to a call. The exact inputs required by any call are defined in the app's own docs.

Example `inputs`:

```hcl
call anapp_run {
  inputs = {
    property = "value"
    foo = event.some_value
    bool_val = event.branch ==
  }
}
```

## Done `[block:done]`

`done` blocks are used to define when a pipeline is complete, either due to error
or completion. They also allow a completed pipeline to declare a result object.

`done` blocks can appear within an `on` block and accept the following:

| Name       |    Type     | Required | Multiple |             Example             |
| :--------- | :---------: | :------: | :------: | :-----------------------------: |
| **error**  | `Attribute` |    -     |    -     | `error = "Bad thing happened"`  |
| **result** | `Attribute` |    -     |    -     | `result = {myfield = "Great!"}` |

Example `done` block:

```hcl
on event_action {
  call foo_bar {...}

  done {
    error = foo_bar.hops.error
    result = foo_bar.result
  }
}
```

A `done` block is satisfied when:

- `error` evaluates to anything other than `null` or `false` **OR**
- `result` evaluates to anything other than `null`.

If the `done` block is satisfied, the pipeline will exit.

> Note: `error` takes precedence over `result` in determining the outcome.<br>
> If `error` is not `null` or `false`, the pipeline will always be marked as `errored`.

Multiple `done` blocks may be used in a single pipeline, but only one will become the final result. This allowings defining multiple/early exit scenarios.<br>
If multiple `done` blocks match the first will be used.

Pipelines do not require a `done` block to complete. The default behaviour is:

- Mark as `done` when existing `calls` have their results and no further `calls` can be dispatched
- The outcome will be `completed` with an empty object `result`

---

### Done block fields

#### Error <small>`optional`</small>

The `error` attribute defines the error (if any) for a pipeline. If `error` evaluates to `false` or `null`, the pipeline has not errored. All other values including the empty string mean the pipeline will error immediately.

`error` can be set to any string value, which will be used as the error message for the pipeline

Example `error`:

```hcl
on event {
  done {error = "I'm going to immediately error the pipeline"}
  ...
}
```

`error` evaluation is relaxed to avoid having to guard against as yet unset variables. If `error` evaluation fails, it will be defaulted to `null`.<br>
In debug mode this will be logged with a reason, in normal running mode it will not.


#### Result <small>`optional`</small>

The `result` attribute defines the result (if any) for a pipeline. If `result` evaluates to `null` the pipeline has no `result`. All other values (including the empty string and `false`) mean the pipeline will complete immediately.

`result` can be set to any value that is serializable as JSON

Example `result`:

```hcl
on event {
  done {result = "Success"}
  ...
}
```

`result` evaluation is relaxed to avoid having to guard against as yet unset variables. If `result` evaluation fails, it will be defaulted to `null`.<br>
In debug mode this default will be logged with a reason, in normal running mode it will not.

If your pipeline should exit successfully, but you do not need to return a `result`, it is best to set `result` to the empty string (or a nice message).

---

## Tasks `[block:task]`

`task` blocks can be defined at the top level of a .hops config and accept the following:

| Name             |    Type     |      Required      |      Multiple      |                                           Example                                           |
| :--------------- | :---------: | :----------------: | :----------------: | :-----------------------------------------------------------------------------------------: |
| **label**        |   `Label`   | :white_check_mark: |         -          |                                   `task onboard_user {}`                                    |
| **description**  | `Attribute` |         -          |         -          | `description = "A longer piece of text describing this task's purpose. Can be multi-line."` |
| **display_name** | `Attribute` |         -          |         -          |                             `display_name = "Onboard new user"`                             |
| **emoji**        | `Attribute` |         -          |         -          |                                       `emoji = "🙋🏽‍♀️"`                                        |
| **summary**      | `Attribute` |         -          |         -          |                        `summary = "Summary of this task's purpose"`                         |
| **param**        |   `Block`   |         -          | :white_check_mark: |                                   `param myparam_name {}`                                   |

Example `task` block:

```hcl
task onboard_user {
  display_name = "Onboard Internal User" // Optional
  emoji = "🙋🏽‍♀️" // Optional
  summary = "Onboard a new user with default access" // Optional
  description = <<-EOT
  Use this task to onboard a new internal user.
  The user will be provisioned with the email address given and access to all
  default SaaS applications.
  EOT // Optional

  // Params are optional. A task without params will just have a submit button.
  param user_name {}
  param email {required = true}
  param notes {type = "text"}
  param is_admin {
    type = "bool"
    help = "Whether this user should be created with admin level perms"
  }
}

on task_onboard_user {
  // Do some work
  ...
}
```

Tasks define user interfaces for your pipelines. A pipeline doesn't _require_ a UI, as you can trigger them based on events from your various apps or via a `schedule`, but some ad-hoc flows are better driven by a human action. Tasks allow you to easily share common scripts and workflows with a wider team.

You'll notice `task` doesn't define any work to do, just the inputs it accepts and validates.
The `on` block is the single place to define pipelines and their calls.

This decoupling means a `task` can execute multiple pipelines, and that these pipelines can be added/altered without changing the interface.

---

### Task block fields

#### Label <small>`required`</small>

The label names the `task` and will be used to auto-generate a `display_name` if none is given.
The label given to a task will also be used as the `action` for events it generates, with `task` being the event.

Example `label`:

```hcl
task hello {...}
```

You can trigger pipelines with a matching sensor like so:

```hcl
on task_hello {...}
```

The task label must meet the following validation rules:

[Label validation](../_snippets/valid_label.md ':include')

#### Display Name <small>`optional`</small>

The `display_name` for a task gives a nice, human readable name that will be used in the UI.
If a `display_name` is not provided, the task `label` is instead converted to title case with underscores swapped for spaces.

Example `display_name`:

```hcl
task hello {
  display_name = "Say Hello!"
}
```

#### Summary <small>`optional`</small>

The `summary` for a task provides a short summary of its purpose. It will be used in UIs where the longer `description` would be cumbersome

Example `summary`:

```hcl
task hello {
  summary = "Say Hello via Slack!"
}
```

#### Description <small>`optional`</small>

The `description` for a task provides a full description of its purpose. It will be shown to users where appropriate. It may be a multi-line string.

Example `description`:

```hcl
task hello {
  description = "Say Hello via Slack! All messages will be sent to the slack #greetings channel"
}
```

or as a multi-line string:

```hcl
task hello {
  description = <<-EOT
  Say Hello via Slack!

  All messages will be sent to the slack #greetings channel
  EOT
}
```

#### Emoji <small>`optional`</small>

The `emoji` attribute takes an emoji that will be displayed in the UI `task` list.
It can be very helpful when you have a lot of tasks to make them visually distinct.

Example `emoji`:

```hcl
task hello {
  emoji = "👋"
}
```

#### Params <small>`optional`</small>

`param` blocks describe the parameters a task accepts. A task can take zero to many parameters.

Example `param`:

```hcl
task hello {
  param name {required = true}
  param greeting {
    type = "text"
  }
}
```

For more details, see the [section on 'param' blocks](#parameters-blockparam) below.

---

## Parameters `[block:param]`

`param` blocks can be defined within `task` blocks and accept the following:

| Name             |    Type     |      Required      | Multiple |                  Example                   |
| :--------------- | :---------: | :----------------: | :------: | :----------------------------------------: |
| **label**        |   `Label`   | :white_check_mark: |    -     |            `param username {}`             |
| **default**      | `Attribute` |         -          |    -     |        `default = "defaultuser123"`        |
| **display_name** | `Attribute` |         -          |    -     |        `display_name = "Username"`         |
| **help**         | `Attribute` |         -          |    -     | `help = "Username to create for new user"` |
| **required**     | `Attribute` |         -          |    -     |             `required = true`              |
| **type**         | `Attribute` |         -          |    -     |             `type = "string"`              |

Example `param` block:

```hcl
task onboard_user {
  param username {
    type = "string" // Optional. Defaults to "string"
    required = true // Optional. Defaults to false
    help = "Username to create for new user" // Optional
    display_name = "New username" // Optional. Defaults to title-cased label
    default = "defaultuser123" // Optional
  }
}
```

---

### Param block fields

#### Label <small>`required`</small>

The label names the `param` and will be used to auto-generate a `display_name` if none is given.
Params values can be referenced by their label in pipelines.

Example `label`:

```hcl
task hello {
  param email {}
}
```

You can reference param values in pipelines like so:

```hcl
on task_hello {
  if = event.email == "joe@example.com"
  ...
}
```

The `param` label must meet the following validation rules:

[Label validation](../_snippets/valid_label.md ':include')

#### Default <small>`optional`</small>

The `default` attribute defines the default value for a param.
It accepts boolean, numeric, or string inputs. The input type must correspond with the param type.

Example `default`:

```hcl
param greeting {default = "Hello"}
param is_true {
  type = "bool"
  default = true
}
param amount {
  type = "number"
  default = 0.56
}
```

> A given `default` value must be of the same type as the param itself

#### Display Name <small>`optional`</small>

The `display_name` for a param gives a nice, human readable name that will be used in the UI.
If a `display_name` is not provided, the param `label` is used - converted to title case with underscores swapped for spaces.

Example `display_name`:

```hcl
param hello {display_name = "Say Hello!"}
```

#### Help <small>`optional`</small>

`help` will be shown next to a param in the UI, providing extra information about the field.

Example `help`:

```hcl
param email {help = "Enter the email to be created"}
```

#### Required <small>`optional`</small>

`required` declares a param as required or not. By default `required` is false.
Required accepts a boolean value.

Example `required`:

```hcl
param hello {required = true}
param goodbye {required = false} // Since this is the default, it could be omitted like so:
param goodbye {} // Functionally equivalent to above
```

#### Type <small>`optional`</small>

The `type` attribute defines the input type for the field. This will also alter the form field in the UI. Type is defined as a string.

Valid types and their form representation:

- `string`: `input` - This is the default param type
- `text`: `text area` - Handles multi-line strings, but otherwise identical to `string`
- `bool`: `check box`
- `number`: `input with number validation`

Example `type`:

```hcl
param is_good {type = "bool"}
```

> `type` and `default` must correspond for any param where `default` is defined

---

### Pass values to params via URL

You can pass pre-defined values to params via a task's URL. This works for any param by referencing its label.

Example URLS:

```hcl
// String
hops.mydomain.com/console/my_task?foo_string=Hello

//Textarea
hops.mydomain.com/console/my_task?foo_text=Hello World

//Number
hops.mydomain.com/console/my_task?foo_number=001

//Bool
hops.mydomain.com/console/my_task?foo_bool=true

//Combining values
hops.mydomain.com/console/my_task?foo_string=Hello&foo_number=001
```

---

## Schedules `[block:schedule]`

`schedule` blocks can be defined at the top level of a .hops config and accept the following:

| Name       |    Type     |      Required      | Multiple |         Example          |
| :--------- | :---------: | :----------------: | :------: | :----------------------: |
| **label**  |   `Label`   | :white_check_mark: |    -     | `schedule run_report {}` |
| **cron**   | `Attribute` | :white_check_mark: |    -     |   `cron = "0 0 * * *"`   |
| **inputs** | `Attribute` |         -          |    -     |      `inputs = {}`       |

Example `schedule` block:

```hcl
schedule run_report {
  cron = "@daily"
}

on schedule_run_report {
  // Do some work
  ...
}
```

Schedules create scheduled source events with which you can trigger pipelines. Schedules allow you to run automation flows periodically.

You'll notice `schedule` doesn't define any work to do, just the schedule on which to run via `cron` and optional `inputs`.
An `on` block can listen to events created by a `schedule` and perform work in reaction to them (as shown above).

This decoupling means a `schedule` can execute multiple pipelines (or none)

> Note: Hiphops won't catch up missed schedules. If you have no hiphops instance running, the scheduled will not be triggered.

Schedules have a resolution of 1 minute and are subject to idempotency measures (meaning multiple hops instances can serve the same schedules without duplication).

> Note: Idempotency only works for _identical_ schedules. If your input mutates _and_ the schedule is triggered twice, then it may cause duplicate events to be created.

---

### Schedule block fields

#### Label <small>`required`</small>

The label names the `schedule` and will also be used as the `action` for events it generates, with `schedule` being the event.

Example `label`:

```hcl
schedule hello {...}
```

You can trigger pipelines with a matching sensor like so:

```hcl
on schedule_hello {...}
```

The schedule label must meet the following validation rules:

[Label validation](../_snippets/valid_label.md ':include')

#### Cron <small>`required`</small>

The `cron` attribute defines when a `schedule` will run.

> syntax!

Example `cron`:

```hcl
schedule party_time {
  cron = "0 0 * * FRI" // Traditional cron format. Runs midnight every Friday
}

schedule hello {
  cron = "@daily" // Uses friendly pre-defined schedule. Runs at midnight every day
}

schedule ping {
  cron = "@every 1m" // A helper syntax to run every 'duration'
}
```

> As noted above, hiphops does not trigger new schedule events more frequently than every minute. This is part of the idempotency protections so overrides any config given in `cron`.

Hiphops accept typical cron spec format (see [this site](https://crontab.guru/examples.html) for some good examples) in addition to pre-defined schedules:

| Entry                      | Description                                | Equivalent To |
| -------------------------- | ------------------------------------------ | ------------- |
| `@yearly` (or `@annually`) | Run once a year, midnight, Jan. 1st        | `0 0 0 1 1 *` |
| `@monthly`                 | Run once a month, midnight, first of month | `0 0 0 1 * *` |
| `@weekly`                  | Run once a week, midnight between Sat/Sun  | `0 0 0 * * 0` |
| `@daily` (or `@midnight`)  | Run once a day, midnight                   | `0 0 0 * * *` |
| `@hourly`                  | Run once an hour, beginning of hour        | `0 0 * * * *` |
| `@every duration`          | Run every duration (e.g. 1m, 1h25m)        | `0 0 * * * *` |

Most users familiar with cron (or happy with the predefined schedules) won't need the full spec. For completeness, the exhuastive spec of what Hiphops supports is [here](https://pkg.go.dev/github.com/robfig/cron#hdr-CRON_Expression_Format)

#### Inputs <small>`optional`</small>

The `inputs` attribute is a key value object defining the values to be passed with the schedule event. These are inputs of your own choosing and are not required.

Example `inputs`:

```hcl
schedule daily_greeting {
  cron = "@daily"

  inputs = {
    greeting = "Hello"
    name = "World"
  }
}

// Example pipeline using the inputs to do some 'work'
on schedule_daily_greeting {
  call slack_post_message {
    inputs = {
      channel = "random"
      text = "${event.greeting} ${event.name}!" // Hello World!
    }
  }
}
```

---

## Conditional `[attribute:if]`

`if` attributes can be defined within `on` and `call` blocks and accept any expression that evaluates to a single boolean value.

`if` attributes are highly permissive. In most areas of the pipeline an invalid expression (such as referencing a property that doesn't exist) will throw an error. In `if` clauses however, it will simply default to `false` and consider the block non-matching.

This permissiveness allows you to freely reference values that are not yet set (e.g. they're the result of a pending call) without having to write boilerplate checks first.

> Note: `if` statements that successfully evaluate must _always_ evaluate to a boolean value. Anything else is an error.

> Note: In debug mode, the logs will detail when an `if` evaluated to false due to failed evaluation in addition to giving the exact failure.

Example `if` attribute:

```hcl
on pullrequest {
  if = event.action == "opened" || event.action == "reopened"
}

// This is functionally equivalent to the above, but shows how the built-in functions
// can help write `if` statements
on pullrequest {
  if = anytrue(
    event.action == "opened",
    event.action == "reopened"
  )
}
```

There are plenty of built-in functions that are useful when writing .hops configs, and these are defined in the Functions section on the left.<br>
A few common ones to look at first are:

- `alltrue()` accepts n-many boolean args and returns true if they are all true
- `anytrue()` accepts n-many boolean args and returns true if any of them are true
- `can()` evaluates a given expression and returns true if successful, false otherwise. This is useful to check if a property exists
- `try()` similar to `can`, but accepts n-many expressions and returns the result of the first one that succeeds (or an error, if none succeed)
- `glob()` Accepts a string or list of strings and a glob pattern or list of patterns. Returns true if any strings match any patterns
- `xglob()` Similar to `glob` but returns true only if all strings match at least one pattern
