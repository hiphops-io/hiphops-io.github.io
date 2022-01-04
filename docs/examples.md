# Examples


## Sample analysis pipelines

Here are some examples of various Hiphops configs to illustrate some common use cases.

```yaml
id: "Run analysis on changes that include Terraform changes"
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
id: "Auto approve small fixes"
when:
  labels: ["kind/fix", "size/*small"]
add_review:
  approved: true
  comment: "Automatically approved small fix"
```

This pipeline will automatically add an approving review to the PR if it is is either `size/very-small` or `size/small` and detected as a fix by our ML pipelines.

---

```yaml
id: "Run Hiphops on main/release changes that only contain CSS"
when:
  branch:
    - "main"
    - "release/*"
  changed_files_not:
    - "*"
    - "!*.css"
```

This pipeline shows how with Hiphops you can select PRs that alter certain files and *only* alter that type of file.

The branch patterns should be fairly self explanatory, the `changed_files_not` pattern is a bit more interesting. The result is that it will be `true` when the change *only* alters `.css` files and nothing else.

Exclusive matching is quite powerful, especially when paired with dynamic reviews (coming soon) and auto labelling.
Using exclusive matching enables you to create release flows such as not requiring reviews on small, styling only changes.

---

We can use actions to apply custom labels to PRs

```yaml
id: "Apply automerge label to docs only changes against main"
when:
  branch: ["main"]
  changed_files_not:
    - "*"
    - "!*.md"
apply_labels:
  custom:
    - "automerge"
```

The above pipeline could be used to trigger a GitHub action when the `automerge` label is applied that automatically merges the pull request and skips review.

The `apply_labels` action `custom` field takes a list of strings to apply as labels to the PR.

---

We can also choose to apply the labels that Hiphops generates to the PR

```yaml
id: "Apply Hiphops labels to PRs against main"
when:
  branch:
    - "main"
apply_labels:
  matching: ["*"]
```

The `apply_labels.matching` field is a list of glob style patterns used to filter which Hiphops generated labels should be applied to the PR.
