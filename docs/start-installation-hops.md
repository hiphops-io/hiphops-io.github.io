## Installation (Standalone CLI/Server)

> Note: If you're using the desktop app, you do not need to follow these instructions. The hops server/CLI is already included and managed directly via the app.

Installing the hops server locally is simple, and requires no additional dependencies. It's a two step process...

- Download Hiphops (`hops`)
- Add your account key


### Download Hiphops

The Hiphops server and CLI is distributed as a single binary. Download the latest release for your platform [here](https://github.com/hiphops-io/hops/releases/latest).

Place the downloaded binary on your path, either by adding it to a location such as `/usr/local/bin` or updating your `~/.bash_profile` (or whatever the equivalent is for your system)

Example bash:

```bash
export PATH=$PATH:/path/to/hops
```

Test you've set up correctly:

```bash
which hops # Should show path to the hops file
hops -h # Should print the hops help message
```

### Connecting to Hiphops.io

> Note: Whilst it's possible to use hops without an account, these docs will focus on users with accounts as the most common use case. If you do not yet have an account, you can [sign up here](https://www.hiphops.io/)

Sign in to your Hiphops.io account, connect any SaaS tools you'd like to automate and grab your account key.

Add your account key to your local Hiphops:

`hops config addkey --keydata=COPY_PASTED_ACCOUNT_KEY`

!> Important: Keep your account key safe. It should only be shared with trusted users that require privileges to create automations for your connected tools.

**Now you're fully installed** :tada:

## Setup

Now we'll get our config/folders set up:

1. `touch ~/.hops/config.yaml && echo "debug: true" >> ~/.hops/config.yaml` to create a config file with debug enabled.<br>This is just a convenience measure as we'll usually want debug level logging when running locally
1. `mkdir -p ~/.hops/hello_world` to create the directory for our first automation
1. `touch ~/.hops/hello_world/main.hops` to add an empty hops file.

> Note: We're using default directory locations and filenames for everything in this guide, but know that you can configure those settings via CLI flags/ENV_VARS/config files to hops

---

> Note: The desktop app includes a basic built-in editor with side-by-side previews. If you are on MacOS and don't need the full capabilities of an IDE, you can follow the [desktop installation guide](start-installation.md) instead

Go ahead and open `~/.hops/hello_world/main.hops` in an editor. Using `HCL` for syntax highlighting will make this more pleasant.

Paste the following:

```hcl
# main.hops

task greet_world {
  emoji = "🙋🏽‍♀️"
  param greeting {required = true}
}
```

Now start Hiphops with `hops start --watch` (wait for it to say `Console available on http://127.0.0.1:8916/console`) then [click here to open the console](http://127.0.0.1:8916/console)

> Note: You'll see a lot of logging as we're in debug mode. Unless you see an error/fatal level log, it's nothing to worry about and should help you understand how work is flowing through your pipelines.

You should see the task you just created in the UI, `Greet World`.<br>
Click on it and you'll see a form with a single field `Greeting` and a `Run task` button.

Right now, the task won't do anything. You can submit it, but there's no pipelines configured to make it _do_ anything.

Let's update the config:

```hcl
# main.hops

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

Since we started hops with `--watch`, it will automatically load the new config.

Enter any greeting on the task form and click `Run task`.

If you've connected Slack in your hiphops.io account, you should see your message appear.

:tada: Congratulations, you've just created your first Hiphops automation! :tada:
