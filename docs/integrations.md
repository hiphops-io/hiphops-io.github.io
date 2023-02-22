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


---

##### Event: `pull_request`

actions: `opened`, `closed`, `merged`, `edited`

---

##### Event: `push`

actions: ?



### Tasks

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