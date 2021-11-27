# Configuring Hiphops

## How to configure

Hiphops can be configured by placing a file named `hiphops.yaml` in the root of your repository.

> Note the extension must be `.yaml` and not `.yml` or variations thereof

For every PR, we will fetch the `hiphops.yaml` found in the head of target branch (A.K.A the `base` branch) and apply it to the incoming change.
If no file can be found, the default pipeline is one which matches all changes and runs analysis on the change, but takes no further action.

## Quick examples

Here's a quick example of a very simple, but potentially useful, hiphops config:

```yaml
version: beta
release_pipelines:
  - name: "Analyse production and staging changes"
    when:
      branch: ["production", "staging"]
```

This would configure change analysis (along with PR commenting) to run on pull requests targeting branches `production` and `staging`.

---

```yaml
version: beta
release_pipelines:
  - name: "Run analysis on changes that include Terraform changes"
    when:
      changed_files:
        - "*.tf"
        - "*.tfvars"
        - "*.tfstate"
        - "*.tf.json"
```

This pipeline runs on changes that include Terraform changes.

---

```yaml
version: beta
release_pipelines:
  - name: "Run Hiphops on main/release changes that only contain CSS"
    when:
      branch:
        - "main"
        - "release/*"
      changed_files_not:
        - "*"
        - "!*.css"
```

We can use actions to apply custom labels to PRs

```yaml
version: beta
release_pipelines:
  - name: "Apply automerge label to docs only changes against main"
    when:
      branch: ["main"]
      changed_files_not:
        - "*"
        - "!*.md"
    do:
      - name: "Apply automerge label"
        action: apply_custom_labels
        config:
          - "automerge"
```

The above pipeline could be used to trigger a GitHub action when the `automerge` label is applied that automatically merges the pull request and skips review.

The `apply_custom_labels` action `config` takes a list of strings to apply as labels to the PR.

---

We can also choose to apply the labels that Hiphops generates to the PR

```yaml
version: beta
release_pipelines:
  - name: "Apply Hiphops labels to PRs against main"
    when:
      branch:
        - "main"
    do:
      - name: Apply all Hiphops labels to PR
        action: apply_labels
        config: ["*"]
```

The config for the `apply_labels` action is a list of glob style patterns that are compared with the labels created in the Hiphops analysis.


This pipeline shows how with Hiphops you can not only filter by the presence/abscence of a specific pattern, but also by PRs *only* including changes to certain files and nothing else.
The branch patterns should be fairly self explanatory, the `changed_files_not` pattern is a bit more interesting. The result is that it will be `true` when the change *only* alters CSS files and nothing else.

Exclusive matching is quite powerful, especially when paired with dynamic reviews and auto labelling coming in the next releases.
Users will be able to leverage this to allow release flows such as not requiring reviews on small, CSS only changes.


## Config properties

The full set of config values allowed in a hiphops config are as follows:

#### `version`

The `version` parameter is optional and defaults to `beta`. It is used to determine which version of Hiphops we'll use to process the pipeline. During the beta any value other than `beta` will be ignored.


#### `release_pipelines`

`release_pipelines` is a list of... well, release pipelines. Release pipelines define when an analysis should run and what other actions should be taken (such as requesting external approvals, applying labels to the PR or automatically passing code review). This field is required and must contain at least one pipeline.

> In `beta.1` we only ever run one action, which is to post the change analysis as a comment on the PR. In the coming weeks we'll be adding support for automatic labelling and dynamically configuring the number of reviews required for a PR to be merged, with more to come in the near future.

#### `release_pipeline[*].name`

A unique name for the pipeline used to identify it in comments/logs/checksuites etc. It's also useful to explain the purpose of the release pipeline to other developers reading your config.

#### `release_pipeline[*].when`

The when clause allows you to conditionally apply a pipeline using Unix style pattern matching. You can specify patterns for `branch`, `changed_files` and `repo`. The fields `branch` and `repo` match against the change's target, rather than the source.


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

  # Whereas this will match any changed files.
  when:
    changed_files:
      - "!*.css"
      - "*"
```

The second `when` block first excludes files ending in .css, but then adds all files to the match (including those that end in .css).

Every field has a `fieldname_not` variation which returns the opposite boolean result for the same patterns.
```yaml
  when:
    # Whenever this evaluates to true...
    branch: ["release/*"]
    #... this will evaluate to false
    branch_not: ["release/*"]
```

#### `release_pipeline[*].do`

The `do` field is a list of actions to run on the pipeline.
Currently we support two action types:
- `apply_labels` which applies Hiphops generated labels to the PR
- `apply_custom_labels` which applies labels of your own choosing to the PR

#### release_pipeline[*].do[*].name

Optional name for the action to run. Will be used in logs/comments/checksuites etc.

#### release_pipeline[*].do[*].action

The action to run. Either `apply_labels` or `apply_custom_labels`.

#### release_pipeline[*].do[*].config

The config to pass to the action, structure and meaning depends on the action.

`apply_labels` config is a list of glob style patterns that are compared with the labels created in the Hiphops analysis. Matching hiphops labels will be applied to the PR. Applied labels will overwrite any existing labels that share a prefix.

`apply_custom_labels` config is a list of strings to apply as labels to the PR.

## Reference

The full spec for the pipeline config is:

[pipeline_spec.yaml](_snippets/pipeline_spec.yaml ':include :type=code')
