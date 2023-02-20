# Concepts Overview

## Changes

Changes are tracked and analysed pull requests.

A change always starts with a pull request, but unlike a pull request its lifespan extends beyond merge.

Changes in Hiphops are tracked as they are merged through branches, even if the merge method alters the underlying sha. They also contain lots of extra data useful for making decisions. Size metrics, category, commit signature checks being just some.


## Releases

Releases are versioned and annotated bundles of changes

A release can be created in reaction to any push event (including the push event raised when you merge in a PR).

Releases automatically detect the changes contained within them and make this information available via the Hiphops UI. This means you can see at a glance what work a release contains.

In order to properly gather the *new* changes in a release, we require the push to be to a branch that represents your current state. This allows us to create a delta from the comparison.

If you use persistent branches per environment such as `prod` `dev` `main` etc then this will work automagically.

If you push to fresh release branches e.g. `release/v1.0.0` then change detection will require you to create the branch immediately after the last release.

In future we're adding additional behaviour for release delta detection which will remove this restriction.


## Sensors

Sensors are simply bits of config that match specific events with filters and describe the tasks you'd like to trigger.

An example would be a change sensor matching pull requests against your `main` branch that contain changes to the `migrations/` folder. Perhaps adding a custom label `database` to the source PR.


## Tasks

Tasks are triggered in response to matching sensors. They can be things such as labelling a pull request, automatically approving or rejecting a change based on metrics, or other common steps in a development workflow.

In our next major release we're expanding tasks in a big way, allowing you to call arbitrary tasks on connected services whilst using a powerful expression language for decision making.

This unlocks the ability to create sophisticated, fully automated workflows across as many disparate tools as you have in your stack.

## Change analysis

Hiphops analyses changes created by a PR, collecting several metrics and generating a human-readable summary with some key metrics explained. This summary can be viewed in Hiphops or can be automatically posted to the original pull request.

The analysis summary looks like this:

![Hiphops PR comment](_media/change-details.png ':size=80%')

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

The health of a change can be seen as a heuristic measure of both good working practices and how stringent the review process should be.

We generate the health by looking at three key metrics; `size`, `focus`, and `ease`. These metrics correlate strongly with good change outcomes and are based on three principles:

1. **Small changes are safer than large changes**
2. **Focusing on a smaller set of changes at a time is less risky than changing multiple things at once**
3. **Changes that take less time and fewer contributors to complete are less complex and safer than high effort changes**

Hiphops produces a score for each of the three categories in addition to an overall health score (being a simple average of the other three). For all scores, higher is better. Additionally, each category will produce a label and a description with further details.

Whilst unhealthy changes can be perfectly safe and healthy changes can fail, you should find that most problems are caused by unhealthy changes. If a change has a poor health score then it likely deserves more of your team's attention. You may wish to require additional reviews, or request the approval of more senior team members.

**Size**

We score based on the number of files altered in addition to the number of lines added/deleted. Our algorithm accounts for changes that only alter a few lines of code yet impact a wide area of the codebase.

**Focus**

Focus is the detected workload of the author (the one that raised the PR). We only consider the last 500 PRs against the current repository, analysing for concurrent work in addition to increased failure rates.

High scores denote an author was able to remain focused on a single task for the duration of the work, a low score indictates they may be over-utilised or having to frequently switch context.

**Ease**

Ease looks at the total amount of effort was required to create a change. If a change was the effort of multiple authors over a longer period, that's an indicator that the work was more challenging.

> Health scores are always rounded down when displayed in readable format, but internally we keep the original value. We round down to avoid marking a change as slightly healthier than it really is, erring on the side of caution.

### Labels

Each of the health scores generates a label, as does the overall score and the change category.

The possible labels are:

`kind/fix` `kind/maintenance` `kind/enhancement`

`size/very-small` `size/small` `size/medium` `size/large` `size/very-large`

`focus/very-low` `focus/low` `focus/medium` `focus/high`

`ease/very-low` `ease/low` `ease/medium` `ease/high`

`health/very-low` `health/low` `health/medium` `health/high`

It's possible to automatically add these labels to the underlying pull request, in addition to custom labels of your own choosing. Combined with GitHub actions triggered by specific labels developers can create arbitrary automations.