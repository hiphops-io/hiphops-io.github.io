# Introduction

Hiphops gives you the ability to create secure, highly automated development flows.

By hooking into the lifecycle events of your dev tooling, you're able to create automations in reaction to events such as pull requests, pushes and GitHub config changes.

However, Hiphops is more than just an event trigger. Hiphops is built to understand the SDLC process, enriching events in ways that make automated decision making possible.

You aren't just told a pull request is open, you're given a machine learning generated summary of the change's health, predicted category, and hundreds of data points that are critical to understanding what an event _actually_ means to you and your team.


## Getting started

Setting up Hiphops is straighforward. You will need a GitHub account and around 2 minutes.

1. Sign up on the app [app.hiphops.io](https://app.hiphops.io)
2. If you signed in with anything other than GitHub, you also need to [connect your personal GitHub account](https://app.hiphops.io/account)
    > We use this to connect you to your GitHub installation and set up your project. The GitHub installation you create next handles all the privileged access.
3. Install the GitHub app by clicking 'Add project' on the [project page](https://app.hiphops.io/projects). Make sure to select the correct org/account when installing.


:tada: You're done! :tada:

By default Hiphops will monitor all pull requests in the account and take no further action. If you'd like to automate some actions or control what is tracked, you can [configure Hiphops](using.md)


## Viewing your data

Pull requests created in your GitHub org/account will be analysed and stored as changes within Hiphops. A change is essentially a pull request that is tracked beyond close/merge and has been enriched with analysis data. Hiphops follows pull requests through your branches, even when the merge method creates a new sha.

You can see your changes, releases, and development insights in [Hiphops](https://app.hiphops.io/)

Faceted search allows you to see across all of your repos and get a more complete view of your development efforts than is possible via GitHub.

Using faceted search enables you to do things such as see all recently merged pull requests across every repository in your project, or see all changes for a specific author.


## What next?

- Read about the core Hiphops [concepts](concepts.md)
- To customise what events are captured and create some automations, check out the [config docs](using.md)
- See what you can do with Hiphops through [example configs](examples.md)
