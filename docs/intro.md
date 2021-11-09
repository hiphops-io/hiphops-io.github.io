# Introduction

## What is Hiphops?

Hiphops is an automated release management platform, empowering teams to automate away time consuming release admin tasks.

The Hiphops platform:

- Instantly analyses changes for health and impact
- Provides high level summaries to assist with code reviews
- Objectively categorises changes for better team analytics and deeper automations
- Auto labels PRs, opening the door to further automation such as with GitHub Actions

## Getting started

> We're currently in beta, which you will need to join before being able to install. Join the beta [here](https://hiphops.io)

Hiphops is installable as a GitHub app via the marketplace. GitHub apps are installed on a per user/org basis.

Without any config Hiphops will analyse all new PRs and add a summary as a comment.
To refine this behaviour, you can [configure Hiphops](config.md)

## Change analysis overview

Hiphops analyses changes created by a PR, collecting several metrics and a generating a human-readable summary on the PR itself.

When you've installed the Hiphops GitHub app and created your first pull request, Hiphops will post a comment like this:

![Hiphops PR comment](_media/pr-comment.png ':size=80%')

Here's what each bit means and how you should interpret it.

### Rundown

A description of the analysis and the most important findings in plain english.
It will always include details of:

- The author and any other committers
- The size of the change
- A summary of its health
- The category of change (fix, maintenance, or enhancement)

The rundown will sometimes include other information if it is detected as being particularly relevant.​ The category of change is an important feature to note. Hiphops processes every change through a machine learning pipeline that labels it as either `fix`, `maintenance`, or `enhancement`.

Access to objectively assigned categories for every change enables teams to understand how many new features they're shipping, the amount of time spent on tech debt, the amount of time spent on fixing bugs and more. It also indicates which team members need to be included in reviews. Perhaps product or design want to review all enhancements, but wouldn't be suitable reviewers for refactoring work.​
​
### Health scores

The health of a change can be seen as a measure of both good working practices and how stringent the review process should be.

We generate the health by looking at three key metrics; `size`, `author`, and `ease`. These metrics correlate strongly with good change outcomes and are based on three principles:

1. **Small changes are safer than large changes**
2. **Authors that are focused on a single task at a time produce fewer errors than authors with a larger workload**
3. **Changes that take less time and fewer contributors to complete are less complex and safer than high effort changes**

Whilst unhealthy changes can be perfectly safe and healthy changes can fail, you should find that most problems are caused by unhealthy changes. If a change has a poor health score then it likely deserves more of your team's attention. You may wish to require additional reviews, or the approval of more senior team members.

Hiphops produces a score for each of the three categories in addition to an overall health score (being a simple average of the other three). For all scores, higher is better. Additionally, each category will produce a label and a description with further details.

**Size**

We score based on the number of files altered in addition to the number of lines added/deleted. Our algorithm accounts for changes that only alter a few lines of code yet impact a wide area of the codebase.

**Author**

Author is based on the recent activity of the author of the change (the one that raised the PR). We consider the last 500 pull requests against the current repository only.

!> We currently score authors based on recent successfully approved changes. This is being modified in the next algorithm update, where we will account for a sustainable rate of work and lower context switching.

**Ease**

Ease looks at the total amount of effort was required to create a change. If a change was the effort of multiple authors over a longer period, that's an indicator that the work was more challenging.

> Health scores are always rounded down when displayed in readable format, but internally we keep the original value. We round down to avoid marking a change as slightly healthier than it really is, erring on the side of caution.

### Labels

Each of the health scores generates a label, as does the overall score and the change category. Our next release will allow users to apply these labels to your PRs with optional re-writing. Labels can also be used to trigger GitHub Actions workflows.

The possible labels are:​​

`kind/fix` `kind/maintenance` `kind/enhancement`

`size/tiny` `size/small` `size/medium` `size/medium-large` `size/large​` `size/very-large​`

`author/new` `author/pending` `author/known​` *author/pending is an author that has requested changes without any being accepted so far*

`ease/very-low` `ease/low` `ease/medium` `ease/high​`

`health/very-low` `health/low` `health/medium` `health/high`