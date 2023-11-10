> Good programmers use their brains, but good guidelines save us having to think out every case
>
> <cite>Francis Glassborow</cite>

## Sensor

A sensor listens for events from your integrations, matching those events you select and executing a pipeline of tasks.


```yaml
id: My sensor # (Optional) string - Used to identify the sensor throughout the system
resource: sensor # (Required) Currently only accepts a value of 'sensor'
when: # (Optional)
  $: '"some expression" === event.a_property'
  event.some_property: "some_matching_pattern"
  event.some_other.property: ["a", "list", "of matching", "patterns*"]
tasks: # (Required) A list of tasks to perform
- name: foo.task_name
```

##### `resource` _string <small>(required)</small>_

Currently only supports a value of `sensor`.

##### `id` _string_

A unique name/id for the sensor. Used to identify it in the UI and in comments, logs, checksuites etc. It's also useful to explain the purpose of the sensor's pipeline to other developers reading your config.

_Note: Whilst the id field is optional, it's highly recommended._


##### `when` _object_

See [When](#when) syntax.

!> Sensors will almost always need a `when` clause in practice, as without it the sensor will trigger for every single event in your project. The `when` block schema is defined below.


##### `tasks` _array <small>(required)</small>_

A list of [Task](#task) configs

<div style="text-align: right">
  <small>
    <em>/ sensor</em>
  </small>
</div>

---

## Task

A task can be defined within the `tasks` array on a sensor. Tasks will perform work and can have dependencies between them. Tasks that have their conditions met are triggered in parallel. Sequential execution can be defined using `depends` which is documented in the next section.

Available task names and their inputs are defined in the Integrations section

```yaml
tasks:
- name: slack.post_comment # Required string - The name of the task to run
  when: # (Optional) - A when block. Remember events are already filtered by the sensor's own when block, so this isn't always necessary
    ...
  depends: # (Optional) - A depends block. If no depends block is set the task will trigger immediately on event
    ...
  input: # Optional, depending on the task being called
    ... # The required input structure is defined by the task being called
```


<div style="text-align: right">
  <small>
    <em>/ task</em>
  </small>
</div>

---

## Input

Many tasks take inputs - arguments that dictate the work the task should perform.
Some inputs are optional, some are required, and they can be a variety of types (including primitives, complex objects and arrays). See the documentation for the task in question for details.

For example, the following task's inputs specify that the task should create/update a comment saying "This is a comment" on PR 55 in the backend repo:
```yaml
tasks:
  - name: github.create_or_update_pr_comment
    input:
      repo: backend
      pr_number: 55
      comment_body: This is a comment
```

Additionally, you can use the `(path)` decorator as a prefix to the input name (e.g. `(path)repo:`) to specify that the input value is a dot.path to a property on the context object. If you do this, and the path is valid at the time of evaluation, the value of that path will be looked up and substituted into the task input at the time of dispatch.

Returning to the above example:
```yaml
tasks:
  - name: github.create_or_update_pr_comment
    input:
      (path)repo: event.repo_name
      (path)pr_number: event.pr_number
      comment_body: This is a comment
```

In this task definition, the repo name and PR number are now coming via the `event` object within the context object (if, for example, the event in question was a `change` event). If the event was for PR 55 in the backend repo, the task would evaluate to the same values as the prior example.

<div style="text-align: right">
  <small>
    <em>/ input</em>
  </small>
</div>

---

## When

The when clause allows you to filter for specific events based on almost anything.

```yaml
when:
  event.hiphops.event: "pull_request" # Simple pattern match filtering for events of type `pull_request`
  event.hiphops.action: "opened" # Simple pattern match. With the above, matches when a pull request was opened.
  event.branch: ["feature/*", "main"] # Simple pattern match with array of matches
  event.source_branch: ["*", "!feature/*"] # Simple pattern match with a negative pattern
  (not)event.changed_filenames: ["!*.md"] # An inverted (not) match condition. Matches when all changed files are *.md files.
  $: "event.size.score >= 90" # A javascript expression. Matches based on js truthiness/falsiness of result
```

##### How matching works

`when` blocks are sets of key/value pairs defining match clauses. Each individual rule must match for a `when` to evaluate to true (i.e. the clauses are `AND`ed together).

You can filter using javascript expressions or Unix style pattern matching (and any combination of the two) at both the sensor and per task level.

If a `when` block doesn't match, that sensor or task will be skipped for this event.

If prefixed with the `(not)` decorator, the result of the match condition is inverted - matching when all of the criteria on the RHS evaluate as false. Effectively just a boolean toggle.

Keys are either `$` or paths to properties on the pipeline run context, their values are the conditions that property must meet. Some examples given the following context:

```js
// Example context
{
  "event": {
    "project_id": "01234567-0123-abcd-1234-12345678abcd",
    "hiphops": {
      "source": "hiphops",
      "event": "demo_event",
      "action": "ping"
    },
    "greeting": "Hello",
    "greeting_length": 5,
    "alt_greetings": [
      "Hey",
      "Hi",
      "Ello guvnor!"
    ]
  },
  "tasks": {...}
}
```

```yaml
# This clause would match as all key/value conditions match
when:
  $: "event.greeting_length == event.greeting.length"
  event.hiphops.event: "demo_event"
  event.alt_greetings: ["*", "!goodbye"]
```

##### Pattern matches

Normal dot notation keys will be evaluated using a familiar Unix style syntax. They can either take a single string or array of strings.
Under the hood, a single string is simply treated as an array with one entry.

The pattern syntax is as follows:

|Pattern|Meaning|
|-------|-------|
|`*`|matches in a glob-like fashion without matching across `/`|
|`?`|matches any single character|
|`[seq]`|matches any character in seq|
|`[!seq]`|matches any character not in seq|
|`!restofstring`|leading `!` negates the pattern|


Some other notes on matching:

- If you have an array of patterns for a field, they run top to bottom on an additive/subtractive basis.
> Imagine putting items into a 'matched' bucket and taking negated items out again. If the bucket is empty at the end it doesn't match. Otherwise it does.

---

##### Expression matches

Matches can be evaluated as Javascript expressions, giving you the ability to make more complex decisions around the flow of your pipelines.

To define an expression use the key `$`. An expression result will be evaluated for truthiness/falsiness to decide if the condition matches.

Bear in mind that truthiness/falsiness is as defined by Javascript, which may not be the same as whatever language you are most familiar with.

A quick cheat sheet:

The following values are falsy:

```js
false
0 // Zero (including negative zero & big int zero)
undefined
null
'', "", `` // The empty string
NaN
```

Everything else is truthy, but that includes some common gotchas:

```js
[] // An empty array <- Python devs take note!
{} // An empty object <- Python devs take note!
"0" // A string containing a single zero
"false" // A string with the text “false”
```

<div style="text-align: right">
  <small>
    <em>/ when</em>
  </small>
</div>

---

## Depends

A `depends` clause will ensure tasks do not run until their conditions are met. If multiple tasks have their dependencies met at the same time, they will be dispatched in parallel.

This flexible syntax allows you to create complex workflows simply by declaring what a task needs in order to begin executing.

A `depends` clause follows the exact same syntax rules as a `when` clause, the difference is at what point they are evaluated and how they are used. (This is described in detail [below](#when-depends)).

Given that the purpose of a `depends` clause is to create dependencies between tasks, you'll also find you reference different areas of the context object.

In particular a `depends` clause will often reference `vars` and `tasks.*.result`.

A simple example:

```yaml
tasks:
- name: foo.some_first_task
  id: first
- name: foo.some_second_task
  depends:
    $: tasks.first.COMPLETE # Once the `first` task has successfully completed this property will be populated, meaning the task can trigger
```


<div style="text-align: right">
  <small>
    <em>/ depends</em>
  </small>
</div>

---


## When + depends

In a task config, it is possible and useful to define both a `when` and `depends` clause.

In this scenario the `when` block will only be evaluated after the `depends` condition has been met.
This ensures that the pre-conditions for the task have been met before a decision is made on whether the task is relevant.

> To give an example:
>
> - You wouldn't want a task that alerts on test failure to trigger before the tests have completed (`depends`)
>
> - Once the tests _have_ completed, you only want to alert if there was a failure (`when`)

It is expected that all `depends` clauses in a list of tasks will eventually evaluate to true unless an error has occurred. Conversely, `when` clauses have no such expectation. If you wish to make a task conditional (i.e. you don't want that task to execute in some cases) you should configure this behaviour within the `when` block.


<div style="text-align: right">
  <small>
    <em>/ when + depends</em>
  </small>
</div>

---

## Decorators e.g. `(not)` & `(path)`

You will notice in samples and recipes that some keys under `when`, `depends`, and `input` are prefixed with words in parentheses.

We call these decorators, and they're useful shortcuts that alter how a field is resolved. Using decorators means you can often avoid having to use more complex javascript expressions entirely.

### `(not)`

The `(not)` decorator is valid under `when` and `depends` blocks.

It simply toggles the boolean result of a match.

e.g. `(not)branches: ["release/*"]` would return true if `release/*` did _not_ match the field `branches`

### `(path)`

The `(path)` decorator is valid under `input` blocks.

It causes the value given to be evaluated as a json dot notation path. If a value exists in the context object on that path, then its value will be substituted in.

This is super useful when you just want to wire in values from previous tasks or the incoming event.

e.g. `(path)message: event.commit_message` would send the input field `message` with whatever value was present on the `commit_message` of the incoming event.

### `(expr)`

The `(expr)` decorator is valid under `input` blocks.

This decorator causes the value given to be evaluated as a javascript expression.

e.g. `(expr)some_input: "'yes' === 'no'"` would cause the value of `some_input` to be set to `false`


<div style="text-align: right">
  <small>
    <em>/ decorators</em>
  </small>
</div>

---
