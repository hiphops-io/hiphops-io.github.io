# Configuring Hiphops

## Quickstart

Hiphops can be configured by placing a file named `hiphops.yaml` in the root of a repository owned by a connected GitHub account.

Once you've created a config, add the branch/repo name in your project settings. Hiphops will automatically detect and apply changes with no further setup required.

If no custom config is set, the default pipeline matches all changes, runs analysis and stores the result, but takes no further action.

> Note the extension must be `.yaml` and not `.yml` or variations thereof

### A simple example

Here's a quick example of a simple, but potentially useful, hiphops config that automatically approves small PRs that have been detected as fixes:

```yaml
---
id: Approve small fixes
resource: sensor
when:
  $: event.size.score >= 90 && event.kind.label === "fix"
  event.hiphops.event: "change_analysed"
tasks:
- name: github.create_or_update_pr_review
  input:
    (path)pr_number: event.pr_number
    message: "Auto approved a small fix"
    approved: true
```

---

You can create as many sensors in `hiphops.yaml` as you like, separated by `---`:

```yaml
---
id: "Analyse production and staging changes"
resource: sensor
...

---
id: "Do some other thing"
resource: sensor
when:
  branch: ["main"]
...

```

<!-- To see more example pipelines, check out our [examples page](examples.md). -->


## Config syntax

The full set of config values allowed in a hiphops config are as follows:

#### `id`

A unique name/id for the pipeline used to identify it in the UI and comments/logs/checksuites etc. It's also useful to explain the purpose of the release pipeline to other developers reading your config.


#### `resource`

Currently on supports a value of `sensor`.

#### `when`

The when clause allows you to filter for specific events using Unix style pattern matching.

|Pattern|Meaning|
|-------|-------|
|`*`|matches everything|
|`?`|matches any single character|
|`[seq]`|matches any character in seq|
|`[!seq]`|matches any character not in seq|
|`!restofstring`|leading `!` negates the pattern|

You can specify a list of patterns for the following fields:

- `title` The title of the pull request
- `body` The pull request description
- `branch` The name of the target/base branch for the PR e.g. **main**
- `source_branch` The name of the head/source branch for the PR e.g. **feature/new-feature**
- `repo_name` The name of the target/base repo
- `source_repo_name` The name of the source repo
- `full_repo_name` The name of the target/base repo including the account/org name e.g. **hiphops-io/widgets**
- `full_source_repo_name` The name of the source repo including the account/org name
- `status` The status of the PR. Either `OPEN`, `CLOSED`, or `MERGED`
- `changed_filenames` A list of filenames changed in this PR
- `labels` A list of Hiphops generated labels applied to this PR. All possible labels are described [here](concepts.md#labels)


Some other notes on matching:

- When fields are `AND`ed together. All clauses must evaluate to true for the when block to match the event.
- If you have multiple patterns for a field, they run in order on an additive/subtractive basis.
> Imagine putting items into a 'matched' bucket and taking negated items out again. If the bucket is empty at the end it doesn't match. Otherwise it does.
- Every field has a *_not variation. This just flips the boolean result of the match.

**Examples of the above:**


All fields must evaluate to true for the when clause to be true. i.e. The fields are ANDed together.
```yaml
  # This will match only if the target branch is `production` 
  # AND the change contains files ending in *.css OR *.js
  when:
    branch: ["production"]
    changed_filenames : ["*.css", "*.js"]
```

Multiple patterns for a field run top to bottom on an additive/subtractive basis, so ordering matters.
```yaml
# This will match any changed files excluding those ending in .css
  when:
    changed_filenames:
      - "*"
      - "!*.css"

  # Whereas this will match any changed files (because '*' is last, so re-includes everything).
  when:
    changed_filenames:
      - "!*.css"
      - "*"
  
  # and this won't do much at all, since there's no matched patterns to exclude.
  when:
    changed_filenames:
      - "!*.css"

  # If you wanted to match everything except css files, you'd do this:
  when:
    changed_filenames_not:
      - "*.css"
```

Every field has a `FIELDNAME_not` variation which returns the opposite boolean result for the same patterns.
```yaml
  when:
    # Whenever this evaluates to true...
    branch: ["release/*"]
    #... this will evaluate to false
    branch_not: ["release/*"]
```

#### `apply_auto_labels`

Apply labels allows you selectively apply generated Hiphops labels to the PR. All possible Hiphops labels are described [here](concepts.md#labels)


#### `apply_auto_labels.when`

The when block configures when this task will run. It uses the exact same structure as the `when` block in release pipelines which is [described above](#when)


#### `apply_auto_labels.matching`

Takes a list of glob style patterns matched against Hiphops generated labels. Each hiphops generated label that matches a pattern will be applied to the PR. (This is handy to decide which Hiphops labels you actually care about on your PRs).


#### `apply_labels`

This task allows you to automatically apply custom labels to your PR. This is useful to label based on data from the change (e.g. labelling something as `css-only`)

#### `apply_labels.when`

The when block configures when this task will run. It uses the exact same structure as the `when` block in release pipelines which is [described above](#when)

#### `apply_labels.labels`

List of strings to be applied as labels. Must be valid labels for your source host (e.g. GitHub)


#### `add_review`

The add review task adds a review to the pull request that triggered the pipeline. It will either add a new review if the result (approve/request changes) has changed, or update the old review's comment otherwise. This action enables you to do things such as skip reviews for small mantainence changes or auto reject PRs that are too big.

#### `add_review.when`

The when block configures when this task will run. It uses the exact same structure as the `when` block in release pipelines which is [described above](#when)

#### `add_review.approved`

A `boolean` field. If true, the PR will be approved. If false, the review will request changes (which is essentially rejecting it in the current form).

#### `add_review.comment`

A `string` field. The comment to add to the review, if blank a default comment will be used explaining the review was automated.


#### `add_analysis_comment`

Add a summary of the Hiphops analysis to the PR as a comment

#### `add_analysis_comment.include`

An optional list of strings describing which sections to include in the comment. If not provided all sections will be included. Options are `rundown`, `scores`, and `labels`
