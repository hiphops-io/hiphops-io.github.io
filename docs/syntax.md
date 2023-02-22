> Good programmers use their brains, but good guidelines save us having to think out every case
>
> <cite>Francis Glassborow</cite>

## Sensor

A sensor listens for events from your integrations, matching those events you select and executing a pipeline of tasks.


```yaml
id: My sensor # (Optional) Any string. Used to identify the sensor throughout the system
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

See [When](#when-1) syntax.

_Note: An empty or missing `when` clause will match all events._


##### `tasks` _array <small>(required)</small>_

A list of [Task](#task) configs

<div style="text-align: right">
  <small>
    <em>/ sensor</em>
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
  $: "event.size.score >= 90" # A javascript expression. Matches based on js truthiness/falsiness of result
```

##### How matching works

When blocks are sets of key/value pairs defining match clauses. Each individual rule must match for a `when` to evaluate to true (i.e. the clauses are `AND`ed together).

You can filter using javascript expressions or Unix style pattern matching (and any combination of the two) at both the sensor and per task level.

If a `when` block doesn't match, that sensor or task will be skipped for this event.

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

---

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

## Task

<div style="text-align: right">
  <small>
    <em>/ task</em>
  </small>
</div>

---

## Depends

A depends clause will ensure tasks do not run until their conditions are met. If multiple tasks have their dependencies met at the same time, they will be dispatched in parallel.

This flexible syntax allows you to create complex workflows simply by declaring what a task needs to begin executing.

A depends clause follows the exact same syntax rules as a when clause, the difference is at what point they are evaluated and how they are used. (This is described in detail [below](#when-depends)).

Given that the purpose of a `depends` clause is to create dependencies between tasks, you'll also find you reference different areas of the context object.

In particular a `depends` clause will often reference `vars` and `tasks.*.result`.


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

It is expected that all `depends` clauses in a list of tasks will eventually evaluate to true unless an error has occurred. Conversely, `when` clauses have no such expectation. If you wish to make a task conditional (i.e. that task not executing is expected and desired in some cases) you should configure this behaviour within the `when` block.


<div style="text-align: right">
  <small>
    <em>/ when + depends</em>
  </small>
</div>

---