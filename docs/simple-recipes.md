# Simple Recipes

?> Note: The examples below are for information purposes only and may need altering to meet your exact needs. We recommend testing configs in a safe environment before use. This is particularly important for automated approvals or any processes that support critical flows.

## Require small Terraform changes

```yaml
id: "Request small infra changes"
resource: change_sensor
when:
  labels: ["size/*large"]
  changed_filenames:
    - "*.tf"
    - "*.tfvars"
    - "*.tfstate"
    - "*.tf.json"
add_review:
  approved: false
  comment: This Terraform change is quite large. Please break it up into smaller changes.
```

---

## Label styling changes

```yaml
id: "Label style changes"
resource: change_sensor
when:
  changed_filenames: ["*.css", "*.jss"]
apply_labels:
  labels: ["styling"]
```

Automatically applying custom labels also allows you to run complex workflows in response to Hiphops data. Triggering a GitHub Action based on a specific label is quite straightforward.

---

## Approve changes to "safe" files only

This one takes a bit of understanding. We match all files, then remove files that
are in `docs/`, `*.md` etc. This would give us an empty match if the change *only* altered those files. We then use the `*_not` variant to flip that result, giving us exclusive matching. Nice!

```yaml
id: "Approve safe file changes"
when:
  branch:
    - "main"
  changed_filenames_not:
    - "*"
    - "!docs/*"
    - "!*.md"
    - "!*.MD"
    - "!*.rst"
add_review:
  approved: true
  comment: Automatically approving changes to internal docs
```
