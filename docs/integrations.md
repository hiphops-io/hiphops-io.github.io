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

Their event structures are nearly identical to the events directly emitted by Github, described in their docs (https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads).

The only difference is that they also contain, at the event root, the following properties:

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

For the source event structure, see https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#pull_request

---

##### Event: `push`

actions: N/A (no actions exist for this event)

For the source event structure, see https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#push


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

###### Example task

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

## Slack

|Name|Incoming events|Has tasks|Integration|
|:-------|:-------|:-------|:-------|
|`slack`|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Integrated in a few clicks via the UI under project settings|

_Integrating slack allows you to send messages to slack channels. It also allows you to create custom commands using the `/hiphops` slash command. This can be used to trigger sensors and perform tasks by creating a sensor in your `hiphops.yml` file._

### Events

All slack events have:

source: `slack`

##### Event: `command`

actions: Any command your user provids to the `/hiphops` slash command. For example, `/hiphops deploy` will trigger a sensor with the action `deploy`.

###### Event structure

```yaml
project_id: <the current project>
hiphops:
  source: slack
  event: command
  action: <user command>
command: <user command>
args: [<arg1>, <arg2>, ...]
response_url: <time limited URL to respond to message directly>
trigger_id: <can be used to trigger modal>
team_id: <team where the command came from>
channel_id: <channel where the command came from>
user_id: <user to sent the command>
is_enterprise_install: <true or false>
enterprise_id: <enterprise id if set>
```

###### Example event

```yaml
hiphops:
  source: slack
  event: command
  action: deploy
command: deploy
args: [repo, backend, branch, main]
response_url: https://hooks.slack.com/commands/T1234567ABC/12345678912345/T123abcDEF1234567
trigger_id: 123456789.123456789.123456789abcdef12345team_id: T1234567ABC
team_id: T02NVHE2ERJ
channel_id: C1234567ABC
user_id: U1234567ABC
is_enterprise_install: false
```

###### Example sensor

This sensor simply posts a message to the `#slack-integration-dev` slack channel when the `/hiphops` command is used, echoing the command and arguments.

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
      channel: slack-integration-dev
      $: "({text: `Command: ${event.command}, Args: ${event.args}`})"
```

---

### Tasks

All slack tasks are prefixed with `slack.`

##### Task: `slack.post_message`

Posts a message to slack using the slack message API. The message can be a simple string or a complex object. The message object is documented here: [Slack messaging payload documentation](https://api.slack.com/reference/messaging/payload).

###### Task structure

This goes in the `tasks` block of a sensor.

```yaml
tasks:
  - name: slack.post_message
    input:
      channel: <the name of the channel to post to>
      text: <the message to post, which conforms to the slack payload format>
```

###### Example task

```yaml
tasks:
  - name: slack.post_message
    input:
      channel: slack-integration-dev
      text: "Command: ${event.command}, Args: ${event.args}"
```

###### Coming soon

_Ability to reply to a message directly using the `response_url` provided in the event._

## Hiphops - Release manager

The Hiphops release manager is both an event source and a task handler. This integration is built in. No setup is required.

### Events

### Tasks

## Hiphops - Controls

The Hiphops controls service is both an event source and a task handler. This integration is built in. No setup is required.

### Events

### Tasks

## Jira <small>(coming soon)</small>

The Jira integration is high on our roadmap. Integrating with Jira will allow you to create tickets, update their state, or trigger sensors in response to activity on your Jira.

## Datadog <small>(coming soon)</small>

Our Datadog integration is coming soon. Integrating with Datadog will allow you to trigger sensors in reaction to incidents, feed runtime monitoring data back to the relevant changes, alerting in slack and triggering any other internal processes you've modelled within Hiphops.

## ServiceNow <small>(coming soon)</small>

Our ServiceNow integration is coming soon. This integration will enable you to record all changes in ServiceNow, attach test evidence, apply automated categories and ensure all records are always up to date and accurate.
