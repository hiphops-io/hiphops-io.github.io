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

Keys can be dot notation paths to properties in the source event payload, e.g.:

`event.some.path: "foo"`

---

|Pattern|Meaning|
|-------|-------|
|`*`|matches everything|
|`?`|matches any single character|
|`[seq]`|matches any character in seq|
|`[!seq]`|matches any character not in seq|
|`!restofstring`|leading `!` negates the pattern|

<!-- - `title` The title of the pull request
- `body` The pull request description
- `branch` The name of the target/base branch for the PR e.g. **main**
- `source_branch` The name of the head/source branch for the PR e.g. **feature/new-feature**
- `repo_name` The name of the target/base repo
- `source_repo_name` The name of the source repo
- `full_repo_name` The name of the target/base repo including the account/org name e.g. **hiphops-io/widgets**
- `full_source_repo_name` The name of the source repo including the account/org name
- `status` The status of the PR. Either `OPEN`, `CLOSED`, or `MERGED`
- `changed_filenames` A list of filenames changed in this PR
- `labels` A list of Hiphops generated labels applied to this PR. All possible labels are described [here](concepts.md#labels) -->


Some other notes on matching:

- If you have multiple patterns for a field, they run in order on an additive/subtractive basis.
> Imagine putting items into a 'matched' bucket and taking negated items out again. If the bucket is empty at the end it doesn't match. Otherwise it does.


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
