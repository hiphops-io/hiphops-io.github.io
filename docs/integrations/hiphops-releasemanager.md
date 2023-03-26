# Hiphops - Release manager

|Name|Incoming events|Has tasks|Integration|
|:-------|:-------|:-------|:-------|
|`releasemanager`|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|This integration is built in. No setup is required.|

---

## Event: `change`

Calculates and tracks changes to your codebase. This event is triggered when a pull request is opened, closed, merged, reopened and edited.

The analysis performed is documented at [Change Analysis](./concepts.md#change-analysis).

actions: `open`, `closed`, `merged`

Actions are based on the underlying Github PR event.

<details>
<summary>See sample event</summary>

[Change analysis sample](../_sample_events/releasemanager_change.json ':include')

</details>

---

## Task: `record_release`

Creates releases based on receiving the `change` event which this task processes.

```yaml
tasks:
  - name: releasemanager.record_release
    input:
      (path)change: event # this is the event received from the `change` event
      annotations: # (Optional) key value pairs of strings
        app: myapp
        env: prod
      version_template: v$cal # Template string for how the version should be generated
      is_prerelease: false # (Optional) boolean to flag if this is a pre-release or not. Default: false
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
| `$findsemver` | searches ref, message and base_ref of release for a semver (in that order) |


###### Example tasks and sensors

```yaml
---
resource: sensor
id: Record a dev release
when:
  event.hiphops.source: hiphops
  event.hiphops.event: change
  event.repo_name: backend
  event.branch: ["release/*"]
tasks:
  - name: releasemanager.record_release
    id: release
    input:
      (path)change: event
      annotations:
        env: live
      version_template: "v$calyy.$calm.$cald"
      is_prerelease: false
```

```yaml
tasks:
  - name: releasemanager.record_release
    input:
      (path)change: event
      annotations:
        env: dev
        app: "backend"
      version: "2.5.10"
      title: "Release 2.5.10"
      description: "Backend payment changes"
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
      "id": "6feddbec-4ca8-4e56-8df4-55fc7059e621",
      "created_at": "2023-03-09T23:10:43.884737+00:00",
      "updated_at": "2023-03-09T23:13:38.350729+00:00",
      "project_id": "0395b0b2-0dcd-4dfb-89f8-65a36d32d9f3",
      "source": "GITHUB_COM",
      "version": "v23.3.9",
      "repo_name": "backend",
      "full_repo_name": "hiphops-io/backend",
      "annotations": {
        "app": "backend",
        "env": "prod"
      },
      "is_pre_release": false,
      "release_notes": [ // Array of Objects - List of release notes - can be null
        {
          "is_auto_generated": true, // Bool - are the notes auto generated?
          "note": "Task update: handle applying PR labels in one github task (#610)",
          "author_id": "0ff85ad3-fbc1-407c-9bb4-caed8af51f22"
        }
      ]
    }
  }
}
```


---

## Task: `generate_version`

Creates a version string using the provided version template.

```yaml
tasks:
  - name: releasemanager.generate_version
    input:
      version_template: v$cal # String - template for how the version should be generated. See below for supported variables
      sha: 1234567890 # (Optional) string - the sha to use for $sha variables. Only needed if using a $sha variable in the version template
      semver_source: "String with semver v1.2.3" # (Optional) string - the string that is searched for semver by using the $find_semver template variable. Only needed if using $find_semver
```

###### Version templates

The version template is a string. Any version template variables will be replaced to generate your version.

The supported version template variables for this task are:

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
| `$findsemver` | searches ref, message and base_ref of release for a semver (in that order) |
| `$radjective` | A random adjective |
| `$rnoun` | A random noun |
| `$rword` | A random adjective or noun |


###### Example tasks and sensors

```yaml
---
resource: sensor
id: Record a dev release
when:
  event.hiphops.source: hiphops
  event.hiphops.event: change
  event.branch: ["release/*"]
tasks:
  - name: releasemanager.generate_version
    id: version
    input:
      version_template: "$radjective-$rnoun"
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`). The `result` will contain the generated version string.
Additionally, responds with a `vars` object containing a key which is the ID of the message.

###### Example vars

```js
{
  "0": {
    "version": "v23.3.9"
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
