# Syntax


Here's a super simple example of the .hops syntax that shows how pipelines hang together.

```hcl
// Defines the event (pullrequest) and action (merged) that triggers this pipeline
on pullrequest_merged {
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

`on` is a type of block. Blocks sometimes have words immediately after them (like `pullrequest_merged` above). This is called the label.<br>

`inputs` is an attribute. Attributes have `=` immediately after them.

Some blocks can appear inside other blocks, as above with `call slack_post_message` inside the `on pullrequest_merged` block.

Each block is described below, along with the attributes and blocks that are valid to appear within them.

## Pipelines `[block:on]`

`on` blocks can be defined at the top level of a .hops config and accepts the following:

|Name|Type|Required|Multiple|Example|
|:---|:--:|:------:|:------:|:----:|
|**label**|`Label`|:white_check_mark:|-|`on a_label {}`|
|**name**|`Attribute`|-|-|`name = "my_pipeline_name"`|
|**if**|`Attribute`|-|-|`if = true`|
|**call**|`Block`|:white_check_mark:|:white_check_mark:|`call slack_post_message {}`|


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

`on pullrequest {...}` would be triggered by any `pullrequest` event, such as `merged`, `opened` etc

`on pullrequest_merged {...}` would be triggered by `pullrequest` `merged` events only.

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

---

## Calls `[block:call]`

`call` blocks can appear within an `on` block and accept the following:

|Name|Type|Required|Multiple|Example|
|:---|:--:|:------:|:------:|:----:|
|**label**|`Label`|:white_check_mark:|-|`call app_handler {}`|
|**name**|`Attribute`|-|-|`name = "do_thing"`|
|**if**|`Attribute`|-|-|`if = true`|
|**inputs**|`Attribute`|:white_check_mark:|-|`inputs = {}`|


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
 if = event.hops.action == "merged" // Call will only execute if the event action is 'merged'
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

## Tasks `[block:task]`

`task` blocks can be defined at the top level of a .hops config and accept the following:

|Name|Type|Required|Multiple|Example|
|:---|:--:|:------:|:------:|:----:|
|**label**|`Label`|:white_check_mark:|-|`task onboard_user {}`|
|**description**|`Attribute`|-|-|`description = "A longer piece of text describing this task's purpose. Can be multi-line."`|
|**display_name**|`Attribute`|-|-|`display_name = "Onboard new user"`|
|**emoji**|`Attribute`|-|-|`emoji = "🙋🏽‍♀️"`|
|**summary**|`Attribute`|-|-|`summary = "Summary of this task's purpose"`|
|**param**|`Block`|-|:white_check_mark:|`param myparam_name {}`|


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
  param user_name = {}
  param email = {required = true}
  param notes = {type = "text"}
  param is_admin = {
    type = "bool"
    help = "Whether this user should be created with admin level perms"
  }
}

on task_onboard_user {
  // Do some work
  ...
}
```

Tasks define user interfaces for your pipelines. A pipeline doesn't _require_ a UI, as you can trigger them based on events from your various apps, but some ad-hoc flows are better driven by a human action. Tasks allow you to easily share common scripts and workflows with a wider team.

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

|Name|Type|Required|Multiple|Example|
|:---|:--:|:------:|:------:|:----:|
|**label**|`Label`|:white_check_mark:|-|`param username {}`|
|**default**|`Attribute`|-|-|`default = "defaultuser123"`|
|**display_name**|`Attribute`|-|-|`display_name = "Username"`|
|**help**|`Attribute`|-|-|`help = "Username to create for new user"`|
|**required**|`Attribute`|-|-|`required = true`|
|**type**|`Attribute`|-|-|`type = "string"`|


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
