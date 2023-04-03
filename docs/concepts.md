## Sensors

Sensors are the entrypoint to everything Hiphops can do. Sensors listen for incoming events (such as a pull request being opened, a slack message being sent, etc) and trigger a set of tasks. Sensors can include a `when` filter to decide which events they apply to.

A basic sensor might look like this:

```yaml
resource: sensor
id: Create release on merge
when:
  event.hiphops.event: "push"
```


## Tasks

Tasks are triggered in response to matching sensors. They can be things such as labelling a pull request, automatically approving or rejecting a change based on metrics, or other common steps in a development workflow.

Tasks can run in parallel or have dependencies between them. Using the `depends` clause allows you to create DAGs, meaning you can easily model complex multi-step processes.

Here's an example of a task on a simple sensor:

```yaml
resource: sensor
tasks:
- name: slack.post_comment
  when:
    some_field: "matches_somevalue"
  input:
    text: "A message to send"
    channel: "general"
```

## Pipelines

When a sensor matches an event, a pipeline run is created within Hiphops. This contains a record of all task statuses, their responses, the original event and the sensor config that triggered it. Pipeline runs can be seen within the Hiphops UI.

##### Example

```json
{
  "id": "69b4b222-0f12-4253-8f8e-ca43199b6fc2", // String - unique id for this pipeline
  "state": { //  Object - cumulative list of pipeline run states. Run has completed with `SUCCESS` (states are `RUNNING`, `SUCCESS`, `FAILURE`). Runs fail if any task fails.
    "RUNNING": "2023-02-23T10:19:18.923Z", // String - timestamp of start
    "SUCCESS": "2023-02-23T10:19:26.725Z" // String - timestamp of completion
  },
  "system": true, // Boolean - false if from user `hiphops.yaml` file; true if from system
  "sensor": { // Object - sensor that triggered pipeline run
    "resource": "sensor", // String - must be "sensor"
    "id": "Prepare releases from Github push events", // String - name of sensor
    "when": { // Object - if all fields match, triggers sensor (a pipeline run is only created if sensor matches)
      "event.hiphops.source": "github", // String - a matching expression
      "event.hiphops.event": "push",
      "event.deleted": "false"
    }
    "tasks": [ // Tasks that pipeline run will execute (if they are able to run). In this case, both tasks can run in parallel as they do not depend on each other
      {
        "name": "system.releasemanager.prepare_release_from_push", //  String - name of task to run. Here a pre-defined integration
        "input": { // Object - input to the task. Tasks require specific inputs
          "(path)push_event": "event" // String - the "(path)" decorator indicates that "event" is read as an object rather than a string
        }
      },
      {
        "name": "slack.post_comment", //  String - name of task to run. Here a pre-defined integration
        "input": {
          "text": "A message to send", // String - message to send (specific to the "slack.post_comment" task)
          "channel": "general" // String - channel to send the message to (specific to the "slack.post_comment" task)
        }
      }
    ],
  },
  "project_id": "0395b0b2-0dcd-4dfb-89f8-65a36d32d9f3", // String - id of the project this pipeline run belongs to (id can be found in the url when viewing a project in the Hiphops UI)
  "lifecycle": { // Object - shows all tasks, and their state.
    "0": { // String - index of the task in the `sensor.tasks` array. States are `PENDING`, `READY`, `RUNNING`, `SUCCESS`, `FAILURE`, `SKIPPED`. All states are present in order (a log of the state changes)
      "PENDING": "2023-02-23T10:19:18.923Z",
      "task": { // Object - a copy of the task that is executed
        "name": "system.releasemanager.prepare_release_from_push",
        "input": { "(path)push_event": "event" },
        "id": "0"
      },
      "READY": "2023-02-23T10:19:20.633Z",
      "RUNNING": "2023-02-23T10:19:23.028Z",
      "SUCCESS": "2023-02-23T10:19:26.062703Z", // String - task completed successfully
      "result": "Preparation of release from Github push successful" //  String or object - result of task. If it fails, then `error_message` will be set instead.
    }
  },
  "vars": {}, // Object - variables set by tasks when they run. Can be used in any expression of a task
  "event": { // Object - event that triggered this run. In this case a github "push" event
    "project_id": "0395b0b2-0dcd-4dfb-89f8-65a36d32d9f3",
    "hiphops": { // Object - hiphops specific event data. Usually used in the `when` clause of a sensor
      "source": "github",
      "event": "push"
    }
    "ref": "refs/heads/release/leap-grow",
    "before": "2549a8d2e3860e334c86d760f2228c168548841d",
    "after": "7e6231e68dfa0465c19242af166deda4fcd6df87",
    "repository": {
      ...
    },
    "pusher": { ... },
    "organization": {
      ...
    },
    "sender": {
      ...
    },
    "installation": {
      ...
    },
    "created": false,
    "deleted": false,
    "forced": false,
    "base_ref": null,
    "compare": "https://github.com/hiphops-io/ui/compare/2549a8d2e386...7e6231e68dfa",
    "commits": [
      {
        ...
        "added": [],
        "removed": [],
        "modified": [
          "app/.env.sandbox",
          "app/package.json",
          "app/src/pipelineRun/component/ToggleCodeToExpand.js",
          "yarn.lock"
        ]
      }
    ],
    "head_commit": {
      ...
      "added": [],
      "removed": [],
      "modified": [
        "app/.env.sandbox",
        "app/package.json",
        "app/src/pipelineRun/component/ToggleCodeToExpand.js",
        "yarn.lock"
      ]
    },
  }
}
```

## Full Lifecycle

Hiphops is designed from the ground up to support release oriented workflows, the humans in the loop and the environments that code travels through.

In order to support this broad remit, the system handles and processes events, dispatches tasks and tracks results.

Here we descibe the full lifecycle of an event coming into Hiphops. For example, an event could be a Github push event.

#### 1. Event arrives

When an event arrives, it is distributed to both the system sensors (these are used to run internal Hiphops tasks and also create new events for use by users), and user sensors (the ones you define in `hiphops.yaml`).

Both flows run in parallel.

#### 2. Sensors are tested to see if they match

Each sensor (from `hiphops.yaml`) is tested to see if it matches the incoming event. This is done via the `when` clause of the sensor. If the event and sensor match, a `pipeline run` is created (see "Pipelines" above for more) which manages the tasks to be run.

A pipeline run consists of the sensor that triggered it, the event that triggered it, and the tasks that are to be executed. It also manages the state of each task.

#### 3. Each tasks is checked to see if it is ready to run

The system sets the tasks in the pipeline run to pending. Each task remains pending until it is ready.

A task is ready when its `depends` clause (same syntax as the `when` clause in the sensor) evaluates to true and the task's `when` clause (if present) is true.

If the task's `when` clause is false, the task is skipped.

> Tasks can depend on each other (which allows them to run in sequence). For example, you may want to run a task to create a release, and then another task to get the release approved, and another to deploy that release. Each task would depend on the successful completion of the previous task.

For example: an `auto_deploy` task may depend on completion of the `can_auto_deploy` task. If the `can_auto_deploy` task returns `true`, then the `auto_deploy` task is ready. Otherwise, the `auto_deploy` task is skipped.

#### 4. Tasks are executed

Once a task is ready, it is scheduled for execution.

The task is marked as running in the pipeline run and sent to the relevent Hiphops service or integration to execute.

Tasks are not timelimited. Therefore Hiphops can easily support long running tasks (such as waiting for an approval from a human).

#### 5. Tasks return results

All tasks return a result (in the task response's `result` property). This `result` can be any valid JSON (which includes objects, arrays, strings, numbers, booleans and `null`).
If the task fails, it will return an `error_message` property instead.

The pipeline run is updated with the returned result of the task and then all remaining pending tasks are checked to see if they can now be set to ready.

> Some tasks, while running, create new events, which can then trigger new pipelines (as described in #1).

> Some tasks return `vars` which will be stored in the pipeline run and can be used in the `input` section of another task.

#### 6. Pipeline run completes

Once all tasks have completed successfully or have been skipped, the pipeline run is marked as successful, and completes.

Failures are described in the next point.

#### 7. Errors and failures

If a task fails, the pipeline run is marked as failed and ends, regardless of the outcome of any other tasks.

If at any point a task completes and no pending tasks can be started, the pipeline is marked as failed.

## Changes

Changes are tracked and analysed pull requests.

A change always starts with a pull request, but unlike a pull request its lifespan extends beyond merge.

Changes in Hiphops are tracked as they are merged through branches, even if the merge method alters the underlying sha. They also contain lots of extra data useful for making decisions. For example, they contain size metrics, categorization, and commit signature checks.


## Releases

Releases are versioned and annotated bundles of changes

A release can be created in reaction to any push event (including the push event raised when you merge in a PR).

Releases automatically detect the changes contained within them and make this information available via the Hiphops UI. This means you can see at a glance which changes a release contains.

In order to properly gather the *new* changes in a release, we require the push to be to a branch that represents your current state. This allows us to create a delta from the comparison.

If you use persistent branches per environment such as `prod` `dev` `main` etc then this will work automagically.

If you push to fresh release branches e.g. `release/v1.0.0` then change detection will require you to create the branch immediately after the last release.

In future we're adding additional behaviour for release delta detection which will remove this restriction.

## Change analysis

Hiphops analyses changes created by a PR, collecting several metrics and generating a human-readable summary with some key metrics explained. This summary can be viewed in Hiphops or can be automatically posted to the original pull request.

The analysis summary looks like this:

![Hiphops PR comment](_media/change-details.png ':size=80%')

Here's what each bit means and how you should interpret it.

### Rundown

A description of the analysis and the most important findings in plain english.
It will always include details of:

- The author and any other committers
- The size of the change
- A summary of its health
- The category of change (fix, maintenance, or enhancement)

The rundown will sometimes include other information if it is detected as being particularly relevant.​ The category of change is an important feature to note. Hiphops processes every change through a machine learning pipeline that labels it as either `fix`, `maintenance`, or `enhancement`.

Access to objectively assigned categories for every change enables teams to understand how many new features they're shipping, the amount of time spent on tech debt, the amount of time spent on fixing bugs and more. It also indicates which team members need to be included in reviews. Perhaps product or design want to review all enhancements, but wouldn't be suitable reviewers for refactoring work.​
​
### Health scores

The health of a change can be seen as a heuristic measure of both good working practices and how stringent the review process should be.

We generate the health by looking at three key metrics; `size`, `focus`, and `ease`. These metrics correlate strongly with good change outcomes and are based on three principles:

1. **Small changes are safer than large changes**
2. **Focusing on a smaller set of changes at a time is less risky than changing multiple things at once**
3. **Changes that take less time and fewer contributors to complete are less complex and safer than high effort changes**

Hiphops produces a score for each of the three categories in addition to an overall health score (being a simple average of the other three). For all scores, higher is better. Additionally, each category will produce a label and a description with further details.

Whilst unhealthy changes can be perfectly safe and healthy changes can fail, you should find that most problems are caused by unhealthy changes. If a change has a poor health score then it likely deserves more of your team's attention. You may wish to require additional reviews, or request the approval of more senior team members.

**Size**

We score based on the number of files altered in addition to the number of lines added/deleted. Our algorithm accounts for changes that only alter a few lines of code yet impact a wide area of the codebase.

**Focus**

Focus is the detected workload of the author (the one that raised the PR). We only consider the last 500 PRs against the current repository, analysing for concurrent work in addition to increased failure rates.

High scores denote an author was able to remain focused on a single task for the duration of the work, a low score indictates they may be over-utilised or having to frequently switch context.

**Ease**

Ease looks at the total amount of effort was required to create a change. If a change was the effort of multiple authors over a longer period, that's an indicator that the work was more challenging.

> Health scores are always rounded down when displayed in readable format, but internally we keep the original value. We round down to avoid marking a change as slightly healthier than it really is, erring on the side of caution.

### Labels

Each of the health scores generates a label, as does the overall score and the change category.

The possible labels are:

`kind/fix` `kind/maintenance` `kind/enhancement`

`size/very-small` `size/small` `size/medium` `size/large` `size/very-large`

`focus/very-low` `focus/low` `focus/medium` `focus/high`

`ease/very-low` `ease/low` `ease/medium` `ease/high`

`health/very-low` `health/low` `health/medium` `health/high`

It's possible to automatically add these labels to the underlying pull request, in addition to custom labels of your own choosing. Combined with GitHub actions triggered by specific labels developers can create arbitrary automations.

## Expressions and their execution context

Expressions and variables are evaluated whenever a sensor is run. They can be used to filter events, or to dynamically generate task inputs. These expressions are run in a sandboxed environment, and are not able to access any external resources.

Examples of when expressions are run:

- In the `when` clause of a sensor
- In the `depends` clause of a task
- When setting the `input` of a task

The sandboxed environment is populated with a `context`.

However, a number of structures and variables are available in the expression context. Some variables are only available in certain circumstances. For example, `vars` are only populated by tasks. If nothing has been put in `vars` yet, then it will be empty.

*Note*: changes to the variables will not be persisted between invocations of the expression. This is true even between different steps in a when clause for example.

The variables and structures are:

- `event` - The event that triggered the sensor
- `vars` - A dictionary of variables set by tasks
- `pipeline_run` - The full pipeline run that is currently executing
- `tasks` - A list of all tasks that are defined in the sensor. The tasks can be accessed either by their id or by their index in the list (the order as defined in the `hiphops.yaml` file)
- `input` - The input to the task. This is the same as the `input` field in the `hiphops.yaml` file


## Materials

Materials are any kind of document, file or blob of data that you want to associate with a change (or, in the near future, a release).

The most common reason you would want to do this is to track evidence of processes for audit purposes - for example, storing the logs of runs of PR checks as evidence of tests having been run, or recording a PR review approval as evidence changes are being peer reviewed.

Materials are stored with the [`releasemanager.save_material`](integrations/hiphops-releasemanager.md#task-save_material) task.

### Material type

When saving a material, you can assign a type to it (the default is just `material`). The possible values are: `material`, `logs`, `test`, `evidence`, `approval`, `report`.

### Relationship to changes and SHAs

Currently, a material is related to a change via the commit SHAs of that change. As a change is closely related to a PR, it contains 1 one more SHAs associated with the commits it consists of (and once merged, a merge SHA). When you save a material with [`releasemanager.save_material`](integrations/hiphops-releasemanager.md#task-save_material), you provide a SHA. For example, when saving a workflow run's logs as a material, you'd provide the SHA of the code that the workflow was run against - available in the workflow run event's `workflow_run.head_sha` property.

The sharing of this SHA between material and change is what links them - when viewing a change in the Hiphops app, any material that is associated with a SHA that is part of that change will be displayed.

Relatedly, a material's SHA is central to how we mark it as `unchanging`, `out of date` or `latest`:
* When saving a material, if you set `outdate_on_sha` to `false`, it will always show as 'unchanging'.
* If `outdate_on_sha` is `true`, and the material's SHA matches a change's head or merge SHA, it will show as `latest`.
* If `outdate_on_sha` is `true`, and the material's SHA matches a change's SHA, but not its head or merge SHA, it will show as `out of date`.

In this way, you can see if the code has moved on since the time when material was saved - useful for those materials that relate to a specific snapshot of your change.

### Annotations and hiding older materials

If you have sensors capturing materials related to check runs and other events that occur every time a push is made to a PR, you can very quickly end up with a lot of materials on an actively developed change - many of them simply being more recent versions of a prior material (e.g. the result of the same set of PR checks, for every time they were run in response to a push).

You may instead want to see only the most recent version of such a material. This can be achieved by using annotations - simple key: value pairs of strings that you assign at the time of saving a material.

If a material has annotations assigned to it, then all materials for a particular change that share the same set of annotations will be grouped up, with only the most recent one being shown when viewing the change.

For example, if we are capturing the workflow run logs for two separate PR checks - "Unit tests" and "Terraform plan" - as materials, then we can assign them the following annotations:

Note: these are incomplete snippets from a larger sensor - and in practice you are likely to assign these values with values from the event payloads themselves, with expressions.
```yaml
name: PR unit tests - logs
annotations:
  event: workflow_run
  check: unit tests

name: PR terraform plan - logs
annotations:
  event: workflow_run
  check: terraform plan
```

The result of this will be only seeing a single entry for "PR unit tests - logs" and a single entry for "PR terraform plan - logs" when viewing a change, regardless of how many of them have been saved (as long as its at least one).

### Relevant events

You could feasibly have materials saved in response to any event, however some are more likely to be relevant than others.

The following is a list of those events more likely to be relevant.

 **Check suite**

In Github, a Check Suite is a collection of checks that can be run against a repository - for example, a collection of tests to run in response to pushes on a pull request - which can, collectively, form a CI/CD process.

The event we receive when a Check Suite completes includes a status, indicating whether the checks were successful or not, which we might want to reflect in a material.

<details>
<summary>See sample event and sensor</summary>

[Check suite completed event](_sample_events/github_check_suite_completed.json ':include')

[Saving check suite material sensor](_sample_sensors/check_suite_save_material.yaml ':include')

</details>

**Check run**

In Github, a Check Run is a single check within a Check Suite - for example, one Check Run may run a repository's unit tests.

The event we receive when a Check Run completes includes a status, indicating whether the check was successful or not, which we might want to reflect in a material.

<details>
<summary>See sample event and sensor</summary>

[Check run completed event](_sample_events/github_check_run_completed.json ':include')

[Saving check run material sensor](_sample_sensors/check_run_save_material.yaml ':include')

</details>

**Workflow run**

In Github, a Workflow describes an automated process, and a Workflow Run is an instance of a Workflow's execution. All Check Runs have an associated Workflow Run that describes their execution, but a Workflow Run doesn't have to be a Check Run.

Aside from being more generic than Check Suites/Runs, Workflow Runs can also be used to obtain logs of their execution, so if you require more detailed logging for audit purposes, you should consider having a sensor that fetches a Workflow Run's logs and saves them as a material. This is done by first using the [`github.fetch_workflow_run_logs`](integrations/github.md#task-fetch_workflow_run_logs) task, and then feeding its output to the [`releasemanager.save_material`](integrations/hiphops-releasemanager.md#task-save_material) task.

<details>
<summary>See sample event and sensor (includes log fetching & saving)</summary>

[Workflow run completed event](_sample_events/github_workflow_run_completed.json ':include')

[Saving workflow run logs material sensor](_sample_sensors/workflow_run_save_material.yaml ':include')

</details>

**PR review**

When a review is submitted on a PR, we receive an event about it, containing details such as the user that made the review, and the status they gave it (e.g. `approved` or `changes_requested`). Given that these are often used as one form of approval, you may want to save these as materials.

<details>
<summary>See sample event and sensor</summary>

[PR review submitted event](_sample_events/github_pull_request_review_submitted.json ':include')

[Saving PR review material sensor](_sample_sensors/pull_request_review_save_material.yaml ':include')

</details>

**PR comment**

We also receive events when you leave an individual review comment on a PR. This is less overtly useful than the review itself, but you may still want to record all such comments as materials for audit purposes.

<details>
<summary>See sample event and sensor</summary>

[PR review comment created event](_sample_events/github_pull_request_review_comment_created.json ':include')

[Saving PR review comment material sensor](_sample_sensors/pull_request_review_comment_save_material.yaml ':include')

</details>

