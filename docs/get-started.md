> Flow is the movement and delivery of customer value through a process. In knowledge work, our whole reason for existence is to deliver value to the customer. > Therefore, it stands to reason that our whole process should be oriented around optimizing flow
>
> <cite>Daniel Vacanti</cite>

## Installation

Setting up Hiphops is straighforward. You will need a GitHub account and around 2 minutes.

1. Sign up here :point_right: [app.hiphops.io](https://app.hiphops.io)
2. If you signed in with anything other than GitHub, you also need to [connect your personal GitHub account](https://app.hiphops.io/account)
    > We only use your personal GitHub auth to connect you to your GitHub installation and set up your project. The GitHub installation you create next handles all the privileged access.
3. Install the GitHub app by clicking 'Add project' on the [project page](https://app.hiphops.io/projects). Make sure to select the correct org/account when installing.

?> Integrating with GitHub will become optional in a future release, but for now it remains a required step in creating a new project.

By default Hiphops will monitor all pull requests in the account and take no further action. If you'd like to automate some actions or control what is tracked, you can [Configure Hiphops](syntax.md)

---

Hiphops will analyse changes without any custom config, but that's not very interesting. Creating a `hiphops.yaml` file is how you use all the good stuff.

## Quickstart

Hiphops can be configured by placing a file named `hiphops.yaml` in the root of a repository owned by a connected GitHub account.

Once you've created a config, add the branch/repo name in your project settings. Hiphops will automatically detect and apply changes with no further setup required.

If no custom config is set, the default pipeline matches all changes, runs analysis and stores the result, but takes no further action.

> Note the extension must be `.yaml` and not `.yml` or variations thereof

### Hello world

What docs would be complete without a nice hello world to get you started?

```yaml
---
id: Hello world
resource: sensor
tasks:
- name: slack.post_message
  input:
    channel: "general"
    text: "Hello world!"
```

Of course, this isn't particularly _useful_ unless you happen to want your Slack channel to be relentlessly spammed every time an event is received from your integrations.

> Hint: You do not want that.

---

You can create as many sensors in `hiphops.yaml` as you like, separated by `---`:

```yaml
---
id: "Analyse production and staging changes"
resource: sensor
...

---
id: "Do some other thing"
resource: sensor
when:
  branch: ["main"]
...

```

## Viewing your data

Pipeline runs are visible in the [Hiphops UI](https://app.hiphops.io/) along with records of your changes, release, development insights and control scans.

Faceted search allows you to see across all of your repos and get a more complete view of your development efforts than is possible via GitHub.

Using faceted search enables you to do things such as see all recently merged pull requests across every repository in your project, or see all changes for a specific author.


## What next?

- Read about the core Hiphops [concepts](concepts.md#sensors)
- To customise what events are captured and create some automations, check out the [config docs](syntax.md#sensor)
- See what you can do with Hiphops through [example configs](simple-recipes.md#simple-recipes)
