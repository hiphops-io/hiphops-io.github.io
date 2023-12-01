# Creating an async standup bot with Slack and schedules

In this walkthrough we use Hiphops schedules paired with Slack to create an async standup bot

## Objective

Our objective is to create an automated standup reminder/update bot for our dev team.

We're pretty committed to dog fooding here, so this is replacing our current subscription for a small bot that solves a similar problem we use today.

When we're finished, we'll have a bot that does the following:

1. Runs daily on a schedule
1. Reminds teammates via Slack that standup updates are due soon, and provides a link to submit them
1. Sends a follow up reminder ten minutes before they're due
1. Provides a form with required and optional fields for devs to provide their update
1. Posts it to slack


## Watch

<iframe width="500" height="500" src="https://www.youtube.com/embed/OFmHsFQmpMc?si=XTEsJqUusnVSprBy" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## The Code

The standup.hops we created in this video is here:

```hcl
schedule standup {
  cron = "23 1 * * 1-5"
}

on schedule_standup {
  call k8s_run {
    name = "timestamp"

    inputs = {
      image = "bash"
      command = [
        "bash",
        "-c",
        <<-BASH
        ASK=$(date '+%Y-%m-%d 09:00:00')
        ASK_TS=$(TZ=Europe/London date -d "$ASK" +%s)

        REMIND=$(date '+%Y-%m-%d 10:00:00')
        REMIND_TS=$(TZ=Europe/London date -d "$REMIND" +%s)

        echo "{\"ask\": $ASK_TS, \"remind\": $REMIND_TS}" > /output/result.json
        BASH
      ]
    }
  }

  call slack_api {
    if = timestamp.completed

    inputs = {
      method = "POST"
      path = "chat.scheduleMessage"
      json = {
        channel = "hiphops-test"
        text = <<-MSG
        <!here> Standups are due by 10:10am, folks!
        
        Submit your update <${env("HIPHOPS_CONSOLE_URL", "")}submit_standup|here>
        MSG
        post_at = timestamp.json.ask
      }
    }
  }

  call slack_api {
    if = timestamp.completed

    inputs = {
      method = "POST"
      path = "chat.scheduleMessage"
      json = {
        channel = "hiphops-test"
        text = <<-MSG
        <!here> Standups are due in 10 minutes!
        
        Submit your update <${env("HIPHOPS_CONSOLE_URL", "")}submit_standup|here>
        MSG
        post_at = timestamp.json.remind
      }
    }
  }
}

task submit_standup {
  description = <<-EOT
  Submit your standup.

  Include links to Trello tickets where applicable
  EOT
  emoji = "🫡"

  param name {
    required = true
    help = "Your name"
  }

  param yesterday {
    type = "text"
    required = true
    help = "What did you work on yesterday?"
  }

  param today {
    type = "text"
    required = true
    help = "What do you plan to do today?"
  }

  param blockers {
    type = "text"
    help = "Any blockers?"
  }
}

on task_submit_standup {
  call slack_post_message {
    inputs = {
      channel = "hiphops-test"
      text = <<-MSG
        *${event.name}'s standup:*

        *Yesterday:*

        ${event.yesterday}

        *Today:*

        ${event.today}

        *Blockers:*

        ${try(event.blockers, "N/A")}
      MSG
    }
  }
}
```


We've also created an environment variable named `HIPHOPS_CONSOLE_URL` for the link to the task. On your local machine that would be something like:

```bash
export HIPHOPS_CONSOLE_URL="http://127.0.0.1:8916/console/
```

when deploying, you'll want to set that to the URL/domain your hops instance is accessible on.

> Note: Remember to place some form of auth (e.g. oauth2proxy) in front of your hops instance before making it accessible over the public internet.
