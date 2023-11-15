# Quickstart

Get started using Hiphops with your first pipeline.


## Setup

This guide assumes you've followed the instructions in [Installation](start-installation.md) and have `hops` installed locally with an Hiphops.io account key.

The quick version of those instructions is:
1. Download the Hiphops server/CLI from [here](https://github.com/hiphops-io/hops/releases/latest) and place it on your path
1. Get your hiphops.io account key from your account page and add it to hops: `hops config addkey --keydata="COPY_PASTED_ACCOUNT_KEY"`
1. Ensure you have GitHub and Slack connected via that same account page, we'll use those in this guide

> Note: If you don't use Slack you can still follow this guide to understand how things stitch together. The Slack parts just won't do anything

---

Now we'll get our config/folders set up:

1. `mkdir ~/.hops` creates the default folder `hops` uses to look for configs
2. `touch ~/.hops/config.yaml && echo "debug: true" >> ~/.hops/config.yaml` to create a config file with debug enabled.<br>This is just a convenience measure as we'll usually want debug level logging when running locally
3. `touch ~/.hops/quickstart.hops` to create our first empty hops file

> Note: We're using default directory locations and filenames for everything in this guide, but know that you can configure those settings via CLI flags/ENV_VARS/config files to hops

## First pipeline + task

We created an empty .hops config in the previous step `~/.hops/quickstart.hops`

Go ahead and open that in an editor. Using `HCL` for syntax highlighting will make this more pleasant.

Past the following:

```hcl
# quickstart.hops

task greet_world {
  emoji = "🙋🏽‍♀️"
  param greeting {required = true}
}
```

Now start Hiphops with `hops start` (wait for it to say `Console available on http://127.0.0.1:8916/console`) then [click here to open the console](http://127.0.0.1:8916/console)

You should see the task you just created, `Greet World`.<br>
Click on it and you'll see a form with a single field `Greeting` and a `Run task` button.

Right now, the task won't do anything. You can submit it, but there's no pipelines listening for that task's event.

Let's update the config:

```hcl
# quickstart.hops

task greet_world {
  emoji = "🙋🏽‍♀️"
  param greeting {required = true}
}

on task_greet_world {
  call slack_post_message {
    inputs = {
      channel = "random" // Change this to any channel name you like in your Slack workspace
      text = "${event.greeting} World!"
    }
  }
}
```

Now restart the `hops` process and go back to [the task in your console](http://127.0.0.1:8916/console/greet_world)

Enter any greeting and click `Run task`.

If you've connected Slack in your hiphops.io account, you should see your message appear.

:tada: Congratulations, you've just created your first Hiphops pipeline + task! :tada:

## Event triggered pipeline

Not all Hiphops pipelines need a `task`. A task just allows you to trigger manually with a defined set of inputs.

Let's edit the above hops file to look like this:

```
on pullrequest {
  if = glob(event.hops.action, ["opened", "closed"])

  call slack_post_message {
    inputs = {
      channel = "engineering" // Pick whatever channel you like
      text = "PR ${event.number} is ready to review, go here to review it: ${event.pull_request.html_url}"
    }
  }
}
```

Restart hops and this will listen for new PRs.<br>
We can't trigger this task manually, so there's nothing to see in the console. If a PR is opened/reopened in then it will post a message to our Slack #engineering channel and let them know about it.

...and that's it! You've created a pipeline with task, plus a pipeline triggered by a pull request event.

Most pipelines will be much more sophisticated than this, stitching together functionality across multiple SaaS products, custom code via containers, etc.

As a next step, we'd suggest looking through the events and calls offered by each app for inspiration, then create a pipeline of your own.

**Enjoy automating!**

