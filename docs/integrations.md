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

Slack is both an event source and a task handler. You can integrate with Slack in a few clicks via the Hiphops UI under your project settings page.

### Events

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
