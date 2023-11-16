# Pipeline Context/Variables

Hiphops is fully event driven, with every pipeline triggered by events and every call/result being transmitted via events.

Your pipelines will have access to context variables (which come from these events) so here we show how you can address them in your code.

> Note: We dive into the details of the evaluation loop in the [Evaluation loop](#evaluation-loop) section, but you will rarely need that level of detail when writing pipelines.

## Context overview

### Source event

Every single pipeline that runs will have an `event` context. This is the source event that triggers the run, and without it no run can begin.

Source events contain a `hops` metadata object at the top level, which has info on the event itself. The rest of the event content is specific to the event type.

The `hops` metadata for a source event looks like this:

```json
{
  "hops": {
    "source": "github",
    "event": "checksuite",
    "action": "completed"
  },
  // ... Rest of the event data
}
```

To illustrate accessing the `event`, here's a super simple Hiphops pipeline that automatically labels a `fix` PR, using the event context throughout.<br>
It is triggered by a `pullrequest` event from our GitHub app.

```hcl
# label_prs.hops

on pullrequest_opened {
  // We check the source event to run this pipeline for PRs against the `backend` repo and fix/ branches only
  if = event.repository.name == "backend" && glob(event.pull_request.head.ref, "fix/*")

  call github_apply_pr_labels {
    inputs = {
      repo = event.repository.name
      pr_number = event.number
      labels = ["kind/fix"]
    }
  }
}
```

Since this pipeline uses the `pullrequest` event, we'd usually look at the docs for that event as we create the pipeline. This helps us to understand what data is available.

### Result events

Result events are returned by finished calls in your pipeline. You should receive a result even if the call failed, in which case it will have the property `errored = true`

Most app calls will have a similar result structure, though you should always consult the docs for that specific call. The common structure is:

```json
{
   "hops": {
    "started_at": "2023-11-13T23:57:20.336Z", // When the app started handling the call (_not_ when your pipeline dispatched it)
    "finished_at": "2023-11-13T23:57:38.869Z", // When the app finsihed handling the call (_not_ when your pipeline received it)
    "error": null // Any extra error info, if the call errored
  },
  "errored": false, // True if the call failed to complete
  "completed": true, // True if the call completed successfully
  "done": true, // True whenever the call is finished (successful or otherwise). Provided as syntactic sugar e.g. allowing if = mycall.done to run a step regardless of outcome
  "body": "", // See specific calls to understand what this will contain. Will always be a string
  "json": null // See specific calls to understand with this will contain. Will always be valid decoded JSON, (Remember: JSON is not necessarily an object with keys)
}
```

Now, let's add another step to the above pipeline and show how we can access the results of a previous call.

```hcl
# label_prs.hops

on pullrequest_opened {
  if = event.repository.name == "backend" && glob(event.pull_request.head.ref, "fix/*")

  call github_apply_pr_labels {
    name = "apply_label" // Note that we've added a name to this step, allowing us to reference it later

    inputs = {
      repo = event.repository.name
      pr_number = event.number
      labels = ["kind/fix"]
    }
  }

  call slack_post_message {
    // ...and now we access the previous result by the call's `name`. The `slack_post_message` call will now only run after our labels have applied successfully
    if = apply_label.completed

    inputs = {
      channel = "engineering-spam"
      // here we access the source event again and use string interpolation to construct a lovely and definitely-useful-not-at-all-spammy message to send to Slack.
      text = "PR ${event.number} was labelled as a fix! View the PR here: ${event.pull_request.html_url}"
    }
  }
}
```

(Click the above to expand fully)

The `github` app's `apply_pr_labels` call doesn't return much data, so we just use it to create a sequence of events.<br>
You can see the event format for every call's result in their docs (in the left sidebar).

> Note: Calls run in parallel by default. Above we create an explicit dependency between calls using the `if` statement, which will make then run one after the other.
> A call won't run until its if statement evaluates to true.


## Evaluation loop

As mentioned above, Hiphops pipelines are fully event driven. Expressions are only evaluated when there's a new event (such as a `call` returning a result).

This means, for example, that an `if` statement which only triggers when the time is 12 O'clock has a good chance of never evaluating to true. The statement will only be evaluated when a new event is received (i.e. there's a change in the pipeline that makes evaluation worthwhile).

Under the hood, Hiphops actually presents the entire aggregate of all events in a sequence as a big blob to your pipeline code. We call this the message bundle or event bundle.<br>
In more practical terms, it's the current state of your pipeline run.

> Note: If you're familiar with event sourcing, this is a very similar pattern (though a radically simple version of it)

Your entire pipeline code is evaluated top to bottom against this state, with any calls being dispatched if they are ready (any `if = true`, or no `if` present).

Technically this means Hiphops dispatches every `call` several times as a pipeline proceeeds, but as a pipeline creator you can ignore this due to idempotency measures we have in place and simply focus on the first condition which should make a `call` execute.

> Note on idempotency: Hiphops ensures all calls in a single run of a pipeline are unique and idempotent. Effectively a `call` can only ever execute once per pipeline run, first call wins.

#### Speed :zap:

One advantage of this 'stupid simple' approach is speed. Whilst we evaluate your full pipeline for every state change, the evaluation is usually nothing more than a bunch of conditional statements executed by a persistent `go` process. As such it's generally completed in sub-millisecond times.

All the complex stuff happens outside of your pipelines by app handlers specifically suited to that task.

