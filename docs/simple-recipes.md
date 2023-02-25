> When to use iterative development? You should use iterative development only on projects that you want to succeed
>
> <cite>Martin Fowler</cite>

# Simple Recipes

?> Note: The examples below are for information purposes only and may need altering to meet your exact needs. We recommend testing configs in a safe environment before use. This is particularly important for automated approvals or any processes that support critical flows.

## Enforce a specific branch flow

This example config allows a team to reject PRs into `main` branch that do not come from a `release/*` branch.

Because Hiphops is configured at an account rather than repo level, this rule will be applied to all repositories uniformly. This saves you from repeating config
in several places.

```yaml
---
resource: sensor
id: Enforce proper branch flow
when:
  event.hiphops.event: change
  event.branch: main
  event.source_branch: ["*", "!release/*"]
tasks:
- name: github.create_or_update_pr_review
  input:
    (path)repo: event.repo_name
    (path)pr_number: event.pr_number
    review_body: PRs into main must come from a release branch
    approved: false
```

We can create several rules for branch flow in a single sensor.
In this example we enforce flows on long-lived branches per environment.

```yaml
---
resource: sensor
id: Enforce proper branch flow
when:
  event.hiphops.event: change
tasks:
- name: github.create_or_update_pr_review
  when:
    event.branch: main
    event.source_branch: ["*", "!stage"]
  input:
    (path)repo: event.repo_name
    (path)pr_number: event.pr_number
    review_body: PRs into prod must come from stage
    approved: false

- name: github.create_or_update_pr_review
  when:
    event.branch: stage
    event.source_branch: ["*", "!dev"]
  input:
    (path)repo: event.repo_name
    (path)pr_number: event.pr_number
    review_body: PRs into stage must come from dev
    approved: false
```

---

## Auto label PRs

Automatically applying custom labels is useful for triaging. It also can be used to drive workflows e.g. in GitHub actions.

```yaml
resource: change
id: Label style changes
when:
  event.hiphops.event: pull_request
  event.changed_filenames: ["*.css", "*.jss"]
tasks:
- name: github.apply_pr_labels
  input:
    (path)repo: event.repo_name
    (path)pr_number: event.pr_number
    labels: ["styling"]
```

However, Hiphops also generates a set of labels for you. You can choose to apply these to your PR based on prefix. Here we apply the `size/*` and `kind/*` labels for auto-sizing and categorisation.

```yaml
---
resource: sensor
id: Add PR labels
when:
  event.hiphops.event: change
tasks:
  - name: github.apply_pr_labels
    input:
      (path)repo: event.repo_name
      (path)pr_number: event.pr_number
      (path)labels: event.labels
      matching: ["size/*", "kind/*"]
      update: true
```

## Create a release

For every push event, Hiphops prepares a potential release.
Releases will gather all the changes they contain and generate release notes, plus some other useful data.

In this sensor we listen for prepared releases on the `main` branch in all repos and then create that release within Hiphops with a generated CalVer version.

```yaml
---
resource: sensor
id: Create a release
when:
  event.hiphops.source: hiphops
  event.hiphops.event: release_prepared
  event.ref: refs/heads/main

tasks:
  - name: releasemanager.create_release
    id: release
    input:
      (path)release: event
      annotations:
        env: live
      version_template: "v$calyy.$calm.$cald"
```

In this sensor we do the same thing, but we also create the release in GitHub along with generated release notes.

```yaml
---
resource: sensor
id: Create a release
when:
  event.hiphops.source: hiphops
  event.hiphops.event: release_prepared
  event.ref: refs/heads/main

tasks:
  - name: releasemanager.create_release
    id: release
    input:
      (path)release: event
      annotations:
        env: live
      version_template: "v$calyy.$calm.$cald"
  
  - name: github.create_release
    depends:
      $: tasks.release.SUCCESS
    input:
      (path)repo: event.repo_name
      (path)tag: vars.release.version
      (path)release_name: vars.release.version
      (expr)release_body: event.release_note || event.message
      (path)target: event.sha
```
