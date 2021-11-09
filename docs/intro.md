# Introduction

## What is Hiphops?

Hiphops is an automated release management platform. Empowering teams to automate away time consuming release admin tasks.

The Hiphops platform:

- Instantly analyses changes for health and impact
- Provides high level summaries to assist with code reviews
- Objectively categorises changes for better team analytics and deeper automations
- Auto labels PRs, opening the door to further automation such as with GitHub Actions

## Getting started

> We're currently in beta, which you will need to join before being able to install. Join the beta [here](https://hiphops.io)

Hiphops is installable as a GitHub app via the marketplace. GitHub apps are installed on a per user/org basis.

With zero config Hiphops will analyse all new PRs and post an analysis as a comment.
To refine this behaviour, you can [configure Hiphops](config.md)

## Change analysis overview

Hiphops analyses changes created by a PR, collecting several metrics and a generating a human-readable summary on the PR itself.

When you've installed the Hiphops GitHub app and created your first pull request, Hiphops will post a comment like this:

![Hiphops PR comment](_media/pr-comment.png ':size=80%')
