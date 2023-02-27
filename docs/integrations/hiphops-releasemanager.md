# Hiphops - Release manager

|Name|Incoming events|Has tasks|Integration|
|:-------|:-------|:-------|:-------|
|`releasemanager`|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|This integration is built in. No setup is required.|

---

## Event: `change`

Calculates and tracks changes to your codebase. This event is triggered when a pull request is opened, closed, merged, reopened and edited.

The analysis performed is documented at [Change Analysis](./concepts.md#change-analysis).

actions: `N/A`

<details>
<summary>See sample event</summary>

[Change analysis sample](../_sample_events/releasemanager_change.json ':include')

</details>

---

## Event: `release_prepared`

This event is emitted when a github `push` occurs. Hiphops then prepares a release on the basis of the push.

You can choose to store the release in Hiphops with the `releasemanager.create_release` task.

actions: `N/A`

<details>
<summary>See sample event</summary>

[Release prepared sample](../_sample_events/releasemanager_release_prepared.json ':include')

</details>

---

## Task: `create_release`

Creates releases based on receiving the `release_prepared` event which this task processes.

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

The supported version template variables are:

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
Additionally, if successful, responds with a `vars` object containing the key `release` as described below.

###### Example vars

```js
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
      "version": "bbb9206", // String - version generated using the version template
      "message": "Task update: handle applying PR labels in one github task (#610)\n\n* Task update: handle applying PR labels in one github task\r\n\r\n* Don't make no-op changes",
      "is_tag": false, // Bool - is this release based off a tag?
      "sha": "bbb920626b031b2aee41bec40500518624a0bfec",
      "ref": "refs/heads/release/snazzy-cobra",
      "repo_name": "backend",
      "full_repo_name": "hiphops-io/backend", // String - target repo name with org
      "annotations": { // Object - a set of annotations from the task input
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
      "pusher": { // Object - can be null
        "id": "0ff85ad3-fbc1-407c-9bb4-caed8af51f22",
        "created_at": "2023-02-22T14:58:26.416878+00:00",
        "updated_at": "2023-02-22T14:58:26.416878+00:00",
        "source": "GITHUB_COM",
        "source_id": "0ff85ad3-fbc1-407c-9bb4-caed8af51f22",
        "username": "a-coder",
        "image_url": "https://avatars.githubusercontent.com/u/12345678?v=4",
        "url": "https://github.com/a-coder"
      },
      "changes": [ // Array of Objects - List of changes - can be null
        {
          "release_id": "0e8104e1-79a9-4306-8d36-90b6fbf8fb72",
          "change_id": "0e8104e1-79a9-4306-8d36-90b6fbf8fb72"
        }
      ],
      "release_notes": [ // Array of Objects - List of release notes - can be null
        {
          "version": "bbb9206", // String - version generated using the version template
          "is_auto_generated": true, // Bool - are the notes auto generated?
          "note": "Task update: handle applying PR labels in one github task (#610)",
          "release_id": "0e8104e1-79a9-4306-8d36-90b6fbf8fb72",
          "author_id": "0ff85ad3-fbc1-407c-9bb4-caed8af51f22"
        }
      ]
    }
  }
}
```

---

## Task: `generate_pr_analysis_comment`

Generates a comment for a PR based on the analysis of the PR in the `change` event. See the [Change Analysis](./concepts.md#change-analysis) section for more details.

This task is usually followed by the `github.create_or_update_pr_comment` task.

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).
Additionally, if successful, responds with a `vars` object containing the key `change_analysis_comment`.

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
