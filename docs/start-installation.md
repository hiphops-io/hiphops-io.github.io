> Flow is the movement and delivery of customer value through a process. In knowledge work, our whole reason for existence is to deliver value to the customer. Therefore, it stands to reason that our whole process should be oriented around optimizing flow
>
> <cite>Daniel Vacanti</cite>

## Installation

Installing Hiphops locally is simple, and requires no additional dependencies. It's effectively a two step process...

- Download Hiphops (`hops`)
- Add your account key

... and you're good to go!

Those steps are detailed further below.


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

> Note: Whilst it's possible to use Hiphops without an account, these docs will focus on users with accounts as the most common use case. If you do not yet have an account, you can [request early access](https://www.hiphops.io/)

Sign in to your Hiphops.io account, connect any SaaS tools you'd like to automate and grab your account key.

Add your account key to your local Hiphops:

`hops config addkey --keydata="COPY_PASTED_ACCOUNT_KEY"`

!> Important: Protect your account key! It grants full access to Hiphops, which by proxy grants access to any SaaS tools you've connected. It should only be shared with users that require privileges to create automations for these tools.

**All done** :tada:


<!-- ## Quickstart

<!-- Decide which branch/repo you want to store your config. You config will be in a file named `hiphops.yaml` which you place in the root of a repository owned by a connected GitHub account.

Now add the branch/repo name in your project settings.

Finally, create the config file in the repo and branch you specified. Hiphops will automatically detect and apply changes with no further setup required.

If no custom config is set, the default pipeline matches all changes, runs analysis and stores the result, but takes no further action.

> Note the extension must be `.yaml` or `.yml`, not variations thereof -->

## What next?

1. Check out `Writing .hops` in the sidebar to create your first hops configs locally
2. Follow our deployment guides to run a deployed Hiphops instance
