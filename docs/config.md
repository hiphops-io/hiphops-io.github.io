# Configuring Hiphops

## How to configure

Hiphops can be configured by placing a file named `hiphops.yaml` in the root of your repository.

> Note the extension must be `.yaml` and not `.yml` or variations thereof

For every PR, we will fetch the `hiphops.yaml` found in the head of target branch (A.K.A the `base` branch) and apply it to the incoming change.
If no file can be found, the default pipeline is one which matches all changes and runs analysis on the change, but takes no further action.

## Quick examples

Here's a quick example of a very simple, but potentially useful, hiphops config:

```yaml
---
id: "Analyse production and staging changes"
when:
  branch: ["production", "staging"]
```

This would enable change analysis (along with PR commenting) to run on pull requests targeting branches `production` and `staging`.

---

You can create as many pipeline configs in the `hiphops.yaml` file as you like, separated by `---`:

```yaml
---
id: "Analyse production and staging changes"
when:
  branch: ["production", "staging"]

---
id: "Label PRs against main with size and kind"
when:
  branch: ["main"]
apply_labels:
  matching:
    - "size/*"
    - "kind/*"
```

To see more example pipelines, check out our [examples page](examples.md).


## Config properties

The full set of config values allowed in a hiphops config are as follows:

#### `id`

A unique name/id for the pipeline used to identify it in comments/logs/checksuites etc. It's also useful to explain the purpose of the release pipeline to other developers reading your config.

#### `when`

The when clause allows you to conditionally apply a pipeline using Unix style pattern matching. You can specify patterns for `branch`, `changed_files`, `repo` and `labels`. The fields `branch` and `repo` match against the change's target, rather than the source.


|Pattern|Meaning|
|-------|-------|
|`*`|matches everything|
|`?`|matches any single character|
|`[seq]`|matches any character in seq|
|`[!seq]`|matches any character not in seq|
|`!restofstring`|leading `!` negates the pattern|


All fields must evaluate to true for the when clause to be true. i.e. The fields are ANDed together.
```yaml
  # This will match only if the target branch is `production` 
  # AND the change contains files ending in *.css OR *.js
  when:
    branch: ["production"]
    changed_files : ["*.css", "*.js"]
```

Multiple patterns for a field run top to bottom on an additive/subtractive basis, so ordering matters.
```yaml
# This will match any changed files excluding those ending in .css
  when:
    changed_files:
      - "*"
      - "!*.css"

  # Whereas this will match any changed files (because '*' is last, so re-includes everything).
  when:
    changed_files:
      - "!*.css"
      - "*"
  
  # and this won't do much at all, since there's no matched patterns to exclude.
  when:
    changed_files:
      - "!*.css"

  # If you wanted to match everything except css files, you'd do this:
  when:
    changed_files_not:
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

#### `apply_labels`

Apply labels is our first supported action (with many more coming soon). Unsurprisingly it allows you to apply labels to a PR.


#### `apply_labels.when`

The when block configures when this action will run. It uses the exact same structure as the `when` block in release pipelines which is [described above](#when)


#### `apply_labels.matching`

Takes a list of glob style patterns matched against Hiphops generated labels. Each hiphops generated label that matches a pattern will be applied to the PR. (This is handy to decide which Hiphops labels you actually care about on your PRs).


#### `apply_labels.custom`

Takes a list of strings. Each item in the list will be applied to the PR as a label. This is useful to label based on filters we provide (e.g. labelling something as `css-only`)


## Reference


### Pipeline config spec

[pipelineconfig.yaml](_snippets/pipelineconfig.yaml ':include :type=code')

### When spec

[when.yaml](_snippets/when.yaml ':include :type=code')

### Apply labels action spec

[applylabelsaction.yaml](_snippets/applylabelsaction.yaml ':include :type=code')
