# Hiphops - Release manager

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
    "name": "SomeGuy",
    "email": "SomeGuytr@gmail.com"
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
