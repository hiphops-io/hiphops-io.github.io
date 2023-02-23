## About event sources and task handlers

Hiphops is only useful when integrated with sources of events and task handlers.
By integrating with your dev tooling such as GitHub, you can use Hiphops to orchestrate your processes end-to-end.

Below we give summaries of the events and tasks provided by each source/handler.

Each integration can act as either an event source, a task handler, or both.

Event sources emit events that are available for you to listen for in your sensors.

Task handlers perform the actions you describe in the `tasks` block of a sensor.

## GitHub

_Integrating with GitHub enables you to automate standard developer flows, view your change and release data in Hiphops and monitor your GitHub account to maintain secure config._

|Name|Incoming events|Has tasks|Integration|
|:-------|:-------|:-------|:-------|
|`github`|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Integrated as part of project creation in the UI|

### Events

All Github events have:

source: `github`

Their event structures are nearly identical to the events directly emitted by Github, described in their docs: https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads.

The only difference is that they also contain the following properties at the event root:

```yaml
project_id: <the current project>
hiphops:
  source: github
  event: <the event name>
  action: <the event action>
```

---

##### Event: `pull_request`

actions: `opened`, `closed`, `reopened`, `merged`, `edited`, `assigned`, `unassigned`, `labeled`, `unlabeled`, `synchronize`, `converted_to_draft`, `ready_for_review`, `locked`, `unlocked`, `review_requested`, `review_request_removed`, `auto_merge_enabled`, `auto_merge_disabled`

For the source event structure, see [GitHub pull request event docs](https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#pull_request)

---

##### Event: `push`

actions: `N/A`

For the source event structure, see [GitHub push event docs](https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#push)


### Tasks

All github tasks are prefixed with `github.`

##### Task: `github.merge_pr`

Merges a PR's source branch into its target branch.

Assuming this is being triggered in response to an event that contains data about the PR, it's likely you'll want to pass the head SHA along in the `head_sha` input - if this is supplied, the merge will only go ahead if no further commits have been pushed since the event was triggered.

###### Task structure

This goes in the `tasks` block of a sensor.

```yaml
tasks:
  - name: github.merge_pr
    input:
      repo: <the name of the repository the PR is in>
      pr_number: <the PR number>
      merge_comment_title: <[optional] the commit message title that will provided with the merge. Defaults to the PR title>
      head_sha: <[optional] the SHA the branch head must be at for the merge to proceed, to prevent race conditions. If not provided the merge will proceed without checking the SHA>
      merge_method: <[optional] the merge method the PR will be merged with - can be “merge”, “squash” or “rebase”. Defaults to “merge” if not provided>
```

###### Example tasks

```yaml
tasks:
  - name: github.merge_pr
    input:
      repo: fabulous-thingy
      pr_number: 55
```

```yaml
tasks:
  - name: github.merge_pr
    input:
      (path)repo: event.repo_name
      (path)pr_number: event.pr_number
      (path)merge_comment_title: event.title
      (path)head_sha: event.sha
      merge_method: squash
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).



##### Task: `github.create_or_update_pr_comment`

Creates or updates a comment on a PR.

If you are only going to have one task post comments to a given PR (e.g. the Hiphops analysis comment), then the first comment posted by the Hiphops app on the given PR is the one that will be updated each time.

However, if you intend to have multiple comments posted by Hiphops, you should add a `comment_identifier` that is distinct for each task - this will be appended to the end of the comment, and subsequent runs of the task (assuming it uses the same identifier) will update the comment on the PR that ends with that identifier.

###### Task structure

This goes in the `tasks` block of a sensor.

```yaml
tasks:
  - name: github.create_or_update_pr_comment
    input:
      repo: <the name of the repository the PR is in>
      pr_number: <the PR number>
      comment_body: <the text to post in the comment>
      comment_identifier: <[optional] an identifier used for making updates to the same comment. This identifier is appended to the end of the comment and subsequent tasks executions that use the same identifier on the same PR will update that comment. This is only needed if you intend to have Hiphops post more than one comment to the same PR.>
```

###### Example tasks

```yaml
tasks:
  - name: github.create_or_update_pr_comment
    input:
      repo: fabulous-thingy
      pr_number: 55
      comment_body: This is a comment
```

```yaml
tasks:
  - name: github.create_or_update_pr_comment
    input:
      (path)repo: event.repo_name
      (path)pr_number: event.pr_number
      (path)comment_body: vars.change_analysis_comment.comment_body
      comment_identifier: ChangeAnalysis
    depends:
      $: vars.change_analysis_comment
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).





##### Task: `github.create_or_update_pr_review`

Creates or updates a review on a PR.

If you mark a review as `approved: true` then the review will be approved. If marked as `approved: false` then it will instead be a request for changes.

If you are only going to have one task post reviews to a given PR, then the first review posted by the Hiphops app on the given PR is the one that will be updated each time.

However, if you intend to have multiple reviews posted by Hiphops, you should add a `review_identifier` that is distinct for each task - this will be appended to the end of the review, and subsequent runs of the task (assuming it uses the same identifier) will update the review on the PR that ends with that identifier.

```yaml
tasks:
  - name: github.create_or_update_pr_review
    input:
      repo: backend # String - The name of the repository the PR is in
      pr_number: 53 # Integer - The PR number
      approved: true # Boolean - Is the review an approval or request for changes
      review_body: Auto approved your PR! # The text to post in the review
      review_identifier: auto-approve-id-foo # Optional string - An identifier used for making updates to the same review. Only needed if you intend to have Hiphops post more than one review to the same PR
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).


##### Task: `github.apply_pr_labels`

Applies labels to a PR.

The input `labels` defines the labels that may be added to the PR. If you don't provide a `matching` input too, then `labels` entries will just be added to the PR.

However, if you do provide a list of glob-matching patterns in `matching` (e.g `["kind/*", "size/*"]`), then only the entries in `labels` that have a match in one or more of the entries in `matching` will be applied.

By setting the update input to true, the task will treat the patterns in `matching` as a list of label patterns to be updated, and will first _remove_ labels matching a pattern before applying those labels from the `labels` input that match it.

So if we have a PR 55 in repository `fabulous-thingy` that currently has the labels `["kind/fix", "size/small", "thingy"]`, and this task is invoked with the input:
```yaml
    input:
      repo: fabulous-thingy
      pr_number: 55
      labels: ["kind/fix", "size/medium", "health/good"]
      matching: ["kind/*", "size/*"]
      update: true
```

Then the task would first remove `size/small`, and then add `size/medium`. No other labels would be changed as they don't meet the criteria in `matching`.

If update had been false (or unset), then it would only add `size/medium`, not attempting to update the existing labels.


###### Task structure

This goes in the `tasks` block of a sensor.

```yaml
tasks:
  - name: github.apply_pr_labels
    input:
      repo: <the name of the repository the PR is in>
      pr_number: <the PR number>
      labels: <an array of strings - the labels to be applied to the PR>
      matching: <an array of strings - each string is a glob matching pattern that will be applied against labels, such that only the labels that have a match will be applied. If update is true, then existing labels on the PR that have a match will be removed too>
      update: <true/false - if true, attempt to update existing labels, or if false just apply labels as new - if matching is provided, this is used to determine which labels will be updated. Defaults to false>
```

###### Example tasks

```yaml
tasks:
  - name: github.apply_pr_labels
    input:
      repo: fabulous-thingy
      pr_number: 55
      labels: ["kind/fix", "size/small"]
      matching: ["kind/*", "size/*"]
      update: true
```

```yaml
tasks:
  - name: github.apply_pr_labels
    input:
      (path)repo: event.repo_name
      (path)pr_number: event.pr_number
      (path)labels: event.labels
      matching: ["kind/*", "size/*"]
      update: true
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).





##### Task: `github.fetch_pr_files`

Fetches details about the files changed in a PR, placing the data in `vars.pr_files` for use by other tasks.

```yaml
tasks:
  - name: github.fetch_pr_files
    input:
      repo: fabulous-thingy # The name of the repository the PR is in
      pr_number: 55 # The PR number
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

If successful it will respond with a `vars` object containing the key `pr_files` an array of data about the PR's files matching the output of [Github's List pull requests files endpoint](https://docs.github.com/en/rest/pulls/pulls?apiVersion=2022-11-28#list-pull-requests-files).

Example:

```json
{
  "vars": {
    "pr_files": [
      {
        "sha": "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07",
        "filename": "e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt",
        "status": "added",
        "additions": 1,
        "deletions": 0,
        "changes": 1,
        "blob_url": "https://github.com/hiphops-io/integration-test/blob/22b88538a0fadf73b427fe749ae7bbbaece2ee57/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt",
        "raw_url": "https://github.com/hiphops-io/integration-test/raw/22b88538a0fadf73b427fe749ae7bbbaece2ee57/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt",
        "contents_url": "https://api.github.com/repos/hiphops-io/integration-test/contents/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt?ref=22b88538a0fadf73b427fe749ae7bbbaece2ee57",
        "patch": "@@ -0,0 +1 @@\n+e1ec5e842183b1878ccc61c8368108b1b1d6198c\n\\ No newline at end of file"
      }
    ]
  }
}
```


## Slack

|Name|Incoming events|Has tasks|Integration|
|:-------|:-------|:-------|:-------|
|`slack`|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Integrated in a few clicks via the UI under project settings|

_Integrating slack allows you to send messages to slack channels. It also allows you to create custom commands using the `/hiphops` slash command. This can be used to trigger sensors and perform tasks by creating a sensor in your `hiphops.yml` file._

### Events

##### Event: `command`

actions: Any command your user provides to the `/hiphops` slash command will be used to populate the `hiphops.action` field.
For example, `/hiphops deploy` will have the action `deploy`.

A command entered on Slack such as `/hiphops foo arg1 arg2` would generate an event with the following structure:

```js
{
  "project_id": "12345677-0123-aaaa-bbbb-123456abcdef", // Your project's UUID
  "hiphops": {
    "source": "slack",
    "event": "command",
    "action": "foo", // User defined command
  },
  "command": "foo", // User defined command
  "args": ["arg1", "arg2"], // An array of strings
  "response_url": "https://hooks.slack.com/commands/T1234567ABC/12345678912345/T123abcDEF1234567", // A time limited URL to respond to message directly
  "trigger_id": "some_id", // Can be used to trigger a modal
  "team_id": "some_team_id", // The team from which the command originated
  "channel_id": "some_channel_id", // The channel from where the command originated
  "user_id": "some_user_id", // The user that sent the command
  "is_enterprise_install": "false" // String "true" | "false" - Whether the slack instance is an enteprise install
  "enterprise_id": "some_enterprise_id" // Optional string, The enterprise ID if set
}
```

###### Example sensor

This sensor simply posts a message to the `#general` slack channel when the `/hiphops` command is used, echoing the command and arguments.

```yaml
---
resource: sensor
id: slack command receiver
when:
  event.hiphops.source: slack
  event.hiphops.event: command
tasks:
  - name: slack.post_message
    input:
      channel: general
      $: "({text: `Command: ${event.command}, Args: ${event.args}`})"
```

---

### Tasks

##### Task: `slack.post_message`

Posts a message to slack using the slack message API. The message can be a simple string or a complex object. The message object is documented here: [Slack messaging payload documentation](https://api.slack.com/reference/messaging/payload).

```yaml
tasks:
  - name: slack.post_message
    input:
      channel: general # String - The name of the channel to post to
      text: "Hello world" # String - The message to post, which conforms to the slack payload format
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).


###### Coming soon

_Ability to reply to a message directly using the `response_url` provided in the event._

## Hiphops - Release manager

The Hiphops release manager is both an event source and a task handler. This integration is built in. No setup is required.

### Events

All releasemanager events have:

source: `hiphops`

##### Event: `change`

Calculates and tracks changes to your codebase. Applies the This event is triggered when a pull request is opened, closed, merged, reopened and edited.

The analysis performed is documented at [Change Analysis](./concepts.md#change-analysis).

action: Not used

###### Event structure

See [Change Analysis](./concepts.md#change-analysis) for more information about the areas marked `See change analysis` below.

```json
{
  "project_id": <current project>,
  "hiphops": {
    "event": "change",
    "source": "hiphops",
    "action": ""
  },
  "id": <id of the change>,
  "created_at": <time the change was created>,
  "updated_at": <time the change was last updated>,
  "source": "GITHUB_COM",
  "source_id": <pull request id>,
  "source_url": <pull request url>,
  "pr_number": <pull request number>,
  "title": <pull request title>,
  "body": <pull request body>,
  "sha": <sha of the pull request head>,
  "merge_sha": <merge commit sha>
  "branch": <target branch of pull request>
  "source_branch": <source branch of pull request>,
  "repo_name": <target repo>,
  "source_repo_name": <source repo>,
  "full_repo_name": <target repo including org>,
  "full_source_repo_name": <source repo including org>,
  "additions": <number of lines added>,
  "deletions": <number of lines deleted>,
  "status": <"OPEN" or "CLOSED">,
  Section: see change analysis for scores
  "health": {
    "score": <0 to 100>
    "label": <calculated label>
    "description": <long description>
  },
  "kind": {
    "score": <0 to 100>
    "label": <calculated label>
    "description": <long description>
  },
  "size": {
    "score": <0 to 100>
    "label": <calculated label>
    "description": <long description>
  },
  "ease": {
    "score": <0 to 100>
    "label": <calculated label>
    "description": <long description>
  },
  "focus": {
    "score": <0 to 100>
    "label": <calculated label>
    "description": <long description>
  },
  "changed_filenames": [
    <list of changed filenames>
  ],
  ],
  "commits": [
    List of commits
    {
      "sha": <git sha>
      "id": <id of commit>,
      "url": <github url of commit>,
      "message": <commit message>,
      "author": {
        "name": <name>,
        "email": <email>
      },
      "committer": {
        "name": <name>,
        "email": <email>
      },
      "verification": {
        "verified": <is commit verified>,
        "reason": <reason for verification outcome>,
      }
    }
  ],
  "file_changes": [
    List of file changes
    {
      "sha": <git sha>,
      "filename": <filename>,
      "status": <"modified", "added", "removed">,
      "additions": <lines added>,
      "deletions": <lines deleted>,
      "changes": <total lines changed>
    },
  ],
  "all_commits_verified": <true or false>,
  "shas": [
    <list of shas>
  ],
  "labels": [
    List of labels. See [Change Analysis](./concepts.md#change-analysis).
  ],
  Section: see change analysis for following scores.
  "health_score": <0.0 to 100.0>,
  "size_score": <0 to 100>,
  "focus_score": <0 to 100>,
  "ease_score": <0 to 100>,
  Section: see change analysis for markdown section.
  "markdown": {
    "header": <header for analysis markdown>,
    "labels": <labels markdown>,
    "scores": <scores markdown>,
    "rundown": <rundown (discussion of change) markdown>,
    "footer": <footer markdown>
  }
}
```

###### Example event

```json
{
  "project_id": "5489ff57-07d9-4d39-b849-18c14612f09d",
  "hiphops": {
    "event": "change",
    "source": "hiphops",
    "action": ""
  },
  "id": "6be55b14-14fb-491e-84a8-93d43f1ff78a",
  "created_at": "2023-02-22T11:41:38.517722+00:00",
  "updated_at": "2023-02-22T11:41:59.629179+00:00",
  "source": "GITHUB_COM",
  "source_id": "5550007777",
  "source_url": "https://github.com/hiphops-io/backend/pull/610",
  "pr_number": 610,
  "title": "Task update: handle applying PR labels in one github task",
  "body": "Ensure that the PR labels are applied in one task, rather than one task per label. This should reduce the number of github API calls and speed up the process of applying labels to PRs.",
  "sha": "138ca34a8bf64172db4111a45650edba97f30c26",
  "merge_sha": "06574592c1d3b5335392dc110a8f3e6a993490c3",
  "branch": "release/snappy-cobra",
  "source_branch": "single-label-update-task",
  "repo_name": "backend",
  "source_repo_name": "backend",
  "full_repo_name": "hiphops-io/backend",
  "full_source_repo_name": "hiphops-io/backend",
  "additions": 175,
  "deletions": 93,
  "status": "OPEN",
  "author_id": null,
  "annotations": {},
  "health": {
    "score": 88.21791487932205,
    "label": "very-high",
    "description": "The change's health is rated as `very high`"
  },
  "kind": {
    "score": 80.87165951728821,
    "label": "maintenance",
    "description": "This change has been categorised as maintenance"
  },
  "size": {
    "score": 72,
    "label": "small",
    "description": "This is a small change with 175 addition(s), 93 deletion(s), and 6 file(s) impacted"
  },
  "ease": {
    "score": 100,
    "label": "very-high",
    "description": "The work on this change was submitted in a single commit and is the work of a single author"
  },
  "focus": {
    "score": 100,
    "label": "very-high",
    "description": "The author has worked on multiple changes in this repository whilst completing this work"
  },
  "changed_filenames": [
    "integrations/github/test/app-task-request.e2e-spec.ts",
    "integrations/github/yarn.lock",
    "integrations/github/src/app.service.spec.ts",
    "integrations/github/src/app.service.ts",
    "integrations/github/src/app.schema.ts",
    "integrations/github/package.json"
  ],
  "commits": [
    {
      "sha": "138ca34a8bf64172db4111a45650edba97f30c26",
      "id": "C_kwDOE8oimNoAKDEzOGNhMzRhOGJmNjQxNzJkYjQxMTFhNDU2NTBlZGJhOTdmMzBjMjY",
      "url": "https://github.com/hiphops-io/backend/commit/138ca34a8bf64172db4111a45650edba97f30c26",
      "message": "Task update: handle applying PR labels in one github task",
      "author": {
        "name": "A Coder",
        "email": "acoder@hiphops.io"
      },
      "committer": {
        "name": "A Coder",
        "email": "acoder@hiphops.io"
      },
      "verification": {
        "verified": false,
        "reason": "unsigned"
      }
    }
  ],
  "file_changes": [
    {
      "sha": "b0a17705adccfa147def7cbb0fe1fae45167d47f",
      "filename": "integrations/github/package.json",
      "status": "modified",
      "additions": 1,
      "deletions": 0,
      "changes": 1
    },
    {
      "sha": "6c189c1e7d0d2c8f9e7a5e916a1b51f86103b5e6",
      "filename": "integrations/github/src/app.schema.ts",
      "status": "modified",
      "additions": 14,
      "deletions": 9,
      "changes": 23
    },
    {
      "sha": "6d8a65a35d07a88800f7e5ec7d9e3483fb362734",
      "filename": "integrations/github/src/app.service.spec.ts",
      "status": "modified",
      "additions": 116,
      "deletions": 36,
      "changes": 152
    },
    {
      "sha": "0f6b7b3da35c7e7f6f8177f3ca0644c21c78c932",
      "filename": "integrations/github/src/app.service.ts",
      "status": "modified",
      "additions": 33,
      "deletions": 44,
      "changes": 77
    },
    {
      "sha": "735bb4491850d000be0a1a351351dfcf5636337b",
      "filename": "integrations/github/test/app-task-request.e2e-spec.ts",
      "status": "modified",
      "additions": 4,
      "deletions": 4,
      "changes": 8
    },
    {
      "sha": "865cfc930ee8cc6451c5b435db09489a4c832d0c",
      "filename": "integrations/github/yarn.lock",
      "status": "modified",
      "additions": 7,
      "deletions": 0,
      "changes": 7
    }
  ],
  "all_commits_verified": false,
  "shas": [
    "138ca34a8bf64172db4111a45650edba97f30c26"
  ],
  "labels": [
    "health/very-high",
    "size/small",
    "ease/very-high",
    "focus/very-high",
    "kind/maintenance"
  ],
  "health_score": 88.21791487932205,
  "size_score": 72,
  "focus_score": 100,
  "ease_score": 100,
  "author": null,
  "markdown": {
    "header": "![Hiphops.io banner](https://v1.hiphops.io/external-assets/pr-comment-banner.svg)",
    "labels": "## Labels\n\n![](https://img.shields.io/badge/health-very--high-7C40C8?style=for-the-badge) ![](https://img.shields.io/badge/size-small-7C40C8?style=for-the-badge) ![](https://img.shields.io/badge/ease-very--high-7C40C8?style=for-the-badge) ![](https://img.shields.io/badge/focus-very--high-7C40C8?style=for-the-badge) ![](https://img.shields.io/badge/kind-maintenance-7C40C8?style=for-the-badge) ",
    "scores": "## Scores\n\n\n\n| Overall `88%` | Size `72%` | Focus `100%` | Ease `100%` |\n| :---: | :---: | :---: | :---: |\n| ![9 out of 10 score](https://v1.hiphops.io/external-assets/health-score-9.svg)  | ![7 out of 10 score](https://v1.hiphops.io/external-assets/health-score-7.svg) | ![10 out of 10 score](https://v1.hiphops.io/external-assets/health-score-10.svg) | ![10 out of 10 score](https://v1.hiphops.io/external-assets/health-score-10.svg) |\n",
    "rundown": "## Rundown\n\n**This looks like a `small` `maintenance` change. This change's health is rated as `very-high`.** Given this health score, it may need fewer reviewers.\n\n<!-- Show health descriptions as bullet points -->\n- This is a small change with 175 addition(s), 93 deletion(s), and 6 file(s) impacted\n- The author has worked on multiple changes in this repository whilst completing this work\n- The work on this change was submitted in a single commit and is the work of a single author\n\n\n\n",
    "footer": "> To understand what each section of this analysis means and how Hiphops arrives at its conclusions, read [about change analysis](https://docs.hiphops.io/#/README?id=change-analysis-overview)."
  }
}
```

###### Example sensor

This sensor detects the change events and pulls out the labels to apply to the PR.

```yaml
resource: sensor
id: Update PR in response to change
when:
  event.hiphops.source: hiphops
  event.hiphops.event: change
  event.branch: main
tasks:
  - name: releasemanager.generate_pr_analysis_comment
    input:
      (path)change: event
      include: ["scores", "rundown"]
  - name: github.create_or_update_pr_comment
    input:
      (path)repo: vars.change_analysis_comment.repo
      (path)pr_number: vars.change_analysis_comment.pr_number
      (path)comment_body: vars.change_analysis_comment.comment_body
    depends:
      $: vars.change_analysis_comment
  - name: github.apply_pr_labels
    input:
      (path)repo: event.repo_name
      (path)pr_number: event.pr_number
      (path)labels: event.labels
      matching: ["kind/*", "size/*"]
      update: true
```

---

##### Event: `release_prepared`

This event is emitted when a github `push` occurs. It then prepares a release on the basis of the push. The user then has the option to store the release by using a sensor and applying the approprate task.

action: Not used

###### Event structure

```json
{
  "project_id": <current project>,
  "hiphops": {
    "event": "release_prepared",
    "source": "hiphops",
    "action": ""
  },
  "id": <release id>,
  "created_at": <release created at>,
  "updated_at": <release updated at>,
  "source": "GITHUB_COM",
  "source_id": <release source id>,
  "source_url": <release commit url>,
  "version": <release version>,
  "message": <release message>,
  "is_tag": <true if releaes is a tag, false otherwise>,
  "sha": <release commit sha>,
  "ref": <target ref>,
  "repo_name": <target repo name>,
  "full_repo_name": <target full repo name including org>,
  "git_pusher": {
    "name": <git pusher name>,
    "email": <git pusher email>
  },
  "is_pre_release": <true if release is a pre-release, false otherwise>,
  "shas": [
    <list of commit shas>
  ],
  "pusher": {
    "id": <pusher id>,
    "created_at": <pusher created at>,
    "updated_at": <pusher updated at>,
    "source": "GITHUB_COM",
    "source_id": <source id>,
    "username": <username>,
    "image_url": <avatar for pusher>,
    "url": <github url for pusher>,
  },
  "changes": [
    List of changes
  ],
  "release_notes": [
    List of release notes
  ]
}
```

###### Example event

```json
{
  "project_id": "5489ff57-07d9-4d39-b849-18c14612f09d",
  "hiphops": {
    "event": "release_prepared",
    "source": "hiphops",
    "action": ""
  },
  "id": null,
  "created_at": null,
  "updated_at": null,
  "source": "GITHUB_COM",
  "source_id": "10bba9039292038ad1a3ea3186058326df142cd4",
  "source_url": "https://github.com/hiphops-io/backend/commit/10bba9039292038ad1a3ea3186058326df142cd4",
  "version": null,
  "message": "Fix error when apply labels has no changes to make",
  "is_tag": false,
  "sha": "10bba9039292038ad1a3ea3186058326df142cd4",
  "ref": "refs/heads/release/snappy-cobra",
  "repo_name": "backend",
  "full_repo_name": "hiphops-io/backend",
  "git_pusher": {
    "name": "Shadlington",
    "email": "shadlingtontr@gmail.com"
  },
  "is_pre_release": null,
  "shas": [
    "10bba9039292038ad1a3ea3186058326df142cd4"
  ],
  "pusher": {
    "id": null,
    "created_at": null,
    "updated_at": null,
    "source": "GITHUB_COM",
    "source_id": "2999999",
    "username": "a_coder",
    "image_url": "https://avatars.githubusercontent.com/u/2999999?v=4",
    "url": "https://github.com/a_coder"
  },
  "changes": [],
  "release_notes": [],
}
```

###### Example sensor

This sensor detects the change events and pulls out the labels to apply to the PR.

```yaml
---
resource: sensor
id: Create a release
when:
  event.hiphops.source: hiphops
  event.hiphops.event: release_prepared
  event.repo_name: ["!process"]
  event.ref: refs/heads/main

tasks:
  - name: releasemanager.create_release
    id: release
    input:
      (path)release: event
      annotations:
        env: live
      version_template: "v$calyy.$calm.$cald"
      is_prerelease: false

  - name: github.create_tag
    id: tag
    depends:
      $: tasks.release.SUCCESS
    input:
      (path)repo: event.repo_name
      (path)tag: vars.release.version
      (path)sha: event.sha
      (path)message: event.message

  - name: slack.post_message
    depends:
      $: tasks.tag.SUCCESS
    input:
      (expr)text: |
        `:tada: ${event.git_pusher.name} released ${vars.release.version} on repo ${event.repo_name} :tada:
        > ${event.release_note}`
      channel: "engineering"
```

### Tasks

##### Task: `releasemanager.create_release`

Creates releases based on receiving the `release_prepared` event which this task processes.

###### Task structure

This goes in the `tasks` block of a sensor.

```yaml
tasks:
  - name: releasemanager.create_release
    id: <name of this task step>
    input:
      (path)release: event # this is the event received from the `release_prepared` event
      annotations:
        <[Optional] annotations as key value pair. common example would be env and app. Release will be annotated with these>
      version_template: <template string for how the version should be generated>
      is_prerelease: <true or false>
```

###### Version templates

The version template is a string. Any version template variables will be replaced to generate your version.

These are the supported version template variables:

| Variable | Description |
| --- | --- |
| `$cal` | year.month.day |
| `$calyyyy` | The current four digit year, zero padded |
| `$calyy` | The current two digit year, zero padded |
| `$caly` | The current year 1-2 digits |
| `$calmm` | The current month, zero padded |
| `$calm` | The current month |
| `$calww` | The current week, zero padded |
| `$calw` | The current week |
| `$caldd` | The current day of the month, zero padded |
| `$cald` | The current day of the month |
| `$sha` | The full commit sha |
| `$sha7` | The first 7 characters of the commit sha |
| `$sha12` | The first 12 characters of the commit sha |
| `$find_semver` | searches ref, message and base_ref of release for a semver (in that order) |


###### Example tasks and sensors

```yaml
---
resource: sensor
id: Create a dev release
when:
  event.hiphops.source: hiphops
  event.hiphops.event: release_prepared
  event.repo_name: backend
  event.ref: ["refs/heads/release/*"]
tasks:
  - name: releasemanager.create_release
    id: release
    input:
      (path)release: event
      annotations:
        env: live
      version_template: "v$calyy.$calm.$cald"
      is_prerelease: false
```

```yaml
tasks:
  - name: releasemanager.create_release
    input:
      (path)release: event
      annotations:
        env: dev
        app: "backend"
      version_template: "$sha7"
      is_prerelease: true
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).
Additionally, if successful it will respond with a `vars` object containing the key `release` as described below.

###### Example vars

```json
{
  "vars": {
    "release": {
      "id": "0e8104e1-79a9-4306-8d36-90b6fbf8fb72",
      "created_at": "2023-02-22T14:58:26.416878+00:00",
      "updated_at": "2023-02-22T14:58:26.416878+00:00",
      "project_id": "0395b0b2-0dcd-4dfb-89f8-65a36d32d9f3",
      "source": "GITHUB_COM",
      "source_id": "bbb920626b031b2aee41bec40500518624a0bfec",
      "source_url": "https://github.com/hiphops-io/backend/commit/bbb920626b031b2aee41bec40500518624a0bfec",
      "version": "bbb9206", # String - version generated using the version template
      "message": "Task update: handle applying PR labels in one github task (#610)\n\n* Task update: handle applying PR labels in one github task\r\n\r\n* Don't make no-op changes",
      "is_tag": false, # Bool - is this release based off a tag?
      "sha": "bbb920626b031b2aee41bec40500518624a0bfec",
      "ref": "refs/heads/release/snazzy-cobra",
      "repo_name": "backend",
      "full_repo_name": "hiphops-io/backend", # String - target repo name with org
      "annotations": { # Object - a set of annotations from the task input
        "app": "backend",
        "env": "dev"
      },
      "pusher_id": "0ff85ad3-fbc1-407c-9bb4-caed8af51f22",
      "git_pusher": {
        "name": "A Coder",
        "email": "a_coder@hiphops.io"
      },
      "is_pre_release": true,
      "shas": [
        "bbb920626b031b2aee41bec40500518624a0bfec"
      ],
      "pusher": { # Object - can be null
        "id": "0ff85ad3-fbc1-407c-9bb4-caed8af51f22",
        "created_at": "2023-02-22T14:58:26.416878+00:00",
        "updated_at": "2023-02-22T14:58:26.416878+00:00",
        "source": "GITHUB_COM",
        "source_id": "0ff85ad3-fbc1-407c-9bb4-caed8af51f22",
        "username": "a-coder",
        "image_url": "https://avatars.githubusercontent.com/u/12345678?v=4",
        "url": "https://github.com/a-coder"
      },
      "changes": [ # Array of Objects - List of changes - can be null
        {
          "release_id": "0e8104e1-79a9-4306-8d36-90b6fbf8fb72",
          "change_id": "0e8104e1-79a9-4306-8d36-90b6fbf8fb72"
        }
      ],
      "release_notes": [ # Array of Objects - List of release notes - can be null
        {
          "version": "bbb9206", # String - version generated using the version template
          "is_auto_generated": true, # Bool - are the notes auto generated?
          "note": "Task update: handle applying PR labels in one github task (#610)",
          "release_id": "0e8104e1-79a9-4306-8d36-90b6fbf8fb72",
          "author_id": "0ff85ad3-fbc1-407c-9bb4-caed8af51f22"
        }
      ]
    }
  }
}
```

##### Task: `releasemanager.generate_pr_analysis_comment`

Generates a comment for a PR based on the analysis of the PR in the `change` event. See the [Change Analysis](./concepts.md#change-analysis) section for more details.

This task is usually followed by the `github.create_or_update_pr_comment` task.

###### Example tasks and sensors

```yaml
---
resource: sensor
id: Update PR in response to change
when:
  event.hiphops.source: hiphops
  event.hiphops.event: change
tasks:
  - name: releasemanager.generate_pr_analysis_comment
    input:
      (path)change: event # Event - the `change` event
      include: ["scores", "rundown"] # Array of Strings - At least one of "scores", "rundown" and "labels"
  - name: github.create_or_update_pr_comment
    input:
      (path)repo: vars.change_analysis_comment.repo
      (path)pr_number: vars.change_analysis_comment.pr_number
      (path)comment_body: vars.change_analysis_comment.comment_body
    depends:
      $: vars.change_analysis_comment
  - name: github.apply_pr_labels
    input:
      (path)repo: event.repo_name
      (path)pr_number: event.pr_number
      (path)labels: event.labels
      matching: ["kind/*", "size/*"]
      update: true
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).
Additionally, if successful it will respond with a `vars` object containing the key `change_analysis_comment`.

###### Example vars

```json
{
  "vars": {
    "change_analysis_comment": {
      "repo": "backend",
      "pr_number": 611,
      "comment_body": "![Hiphops.io banner](https://stage-app.hiphops.io/external-assets/pr-comment-banner.svg)\n## Rundown\n\n**This looks like a `small` `maintenance` change. This change's health is rated as `very-high`.** Given this health score, it may need fewer reviewers.\n\n<!-- Show health descriptions as bullet points -->\n- This is a small change with 246 addition(s), 108 deletion(s), and 7 file(s) impacted\n- The author appears to have focused on one change for the duration of the work\n- The work on this change took over a month to complete and is the work of a single author\n\n\n\n\n## Scores\n\n\n\n| Overall `86%` | Size `63%` | Focus `90%` | Ease `97%` |\n| :---: | :---: | :---: | :---: |\n| ![9 out of 10 score](https://stage-app.hiphops.io/external-assets/health-score-9.svg)  | ![6 out of 10 score](https://stage-app.hiphops.io/external-assets/health-score-6.svg) | ![9 out of 10 score](https://stage-app.hiphops.io/external-assets/health-score-9.svg) | ![10 out of 10 score](https://stage-app.hiphops.io/external-assets/health-score-10.svg) |\n\n> To understand what each section of this analysis means and how Hiphops arrives at its conclusions, read [about change analysis](https://docs.hiphops.io/#/README?id=change-analysis-overview)."
    }
  }
}
```

## Jira <small>(coming soon)</small>

The Jira integration is high on our roadmap. Integrating with Jira will allow you to create tickets, update their state, or trigger sensors in response to activity on your Jira.

## Datadog <small>(coming soon)</small>

Our Datadog integration is coming soon. Integrating with Datadog will allow you to trigger sensors in reaction to incidents, feed runtime monitoring data back to the relevant changes, alerting in slack and triggering any other internal processes you've modelled within Hiphops.

## ServiceNow <small>(coming soon)</small>

Our ServiceNow integration is coming soon. This integration will enable you to record all changes in ServiceNow, attach test evidence, apply automated categories and ensure all records are always up to date and accurate.
