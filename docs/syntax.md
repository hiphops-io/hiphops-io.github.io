# Hiphops Config Syntax

## Sensor

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

##### `resource` (required)

Currently only supports a value of `sensor`.

##### `id`

A unique name/id for the sensor. Used to identify it in the UI and in comments, logs, checksuites etc. It's also useful to explain the purpose of the sensor's pipeline to other developers reading your config.

> _Whilst the id field is optional, it's highly recommended._


##### `when`

See [When](#when-1) docs

An empty or missing `when` clause will match all events.


##### `tasks` (required)

A list of [Task](#task) configs

---

## When

The when clause allows you to filter for specific events based on almost anything.

Filter clauses use either javascript expressions or Unix style pattern matching.

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

- When fields are `AND`ed together. All clauses must evaluate to true for the when block to match the event.
- If you have multiple patterns for a field, they run in order on an additive/subtractive basis.
> Imagine putting items into a 'matched' bucket and taking negated items out again. If the bucket is empty at the end it doesn't match. Otherwise it does.
- Every field has a *_not variation. This just flips the boolean result of the match.

---

## Task


---

## Depends
