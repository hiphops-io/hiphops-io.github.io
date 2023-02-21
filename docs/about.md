## What is Hiphops?

> Software development is not a rational process. It’s a process made by people with feelings, with bodies and with thinking. And by putting all of those together I can be a more effective software developer.
>
> <cite>Kent Beck</cite>

Hiphops is a platform that gives you the ability to create _secure_, highly automated development flows.

By using Hiphops you can **address the gaps between CI/CD and your full dev process.**

Sometimes the gap is small. It might be as simple as a developer having to manually tag a release every time they merge into `main`.

In other cases the gap is vast. Regulated teams might have several manual hoops to jump through before they can kick off a deployment pipeline, or be left manually gathering test evidence for audit purposes.

These gaps represent a tax you pay every time you ship new features to your users. They're a burden on your team and hamper productivity, reliability and developer satisfaction.

Using Hiphops allows you to code up all the manual bits, the annoying bits, the 'we really should automate this but we don't have the time' bits. Which means your team has more time to focus on the 'let's build an awesome product' bits. :tada:



## What isn't it?

#### A CI/CD system

Your CI/CD platform is a _terrible_ place to automate process. People processes need fully asynch systems, reliable retries, and UIs that are friendly to all team members, technical or not. Process automated within CI/CD will always be a brittle pile of scripts that are a pain to work with and limited by the platforms they run on.

#### A static analysis tool

We do analyse your changes, but we care more about the people and working methods that created them. Other static analysers already solve for that part of the equation pretty well.


## How does it work?

Hiphops integrates with your team's tools such as GitHub and Slack. We provide inbound events from each integration along with automated tasks. 

Combining this with our powerful expressions syntax for filtering, event enrichment via ML, and our DAG workflows means you can automate sophisticated flows in a few lines of config.

Things like 'automatically merge a PR if it contains maintenance work, is small and is from an experienced author' become trivial to implement.

Building an incredible dev experience becomes achievable even for small teams. Using Hiphops you could have your own suite of slack commands. e.g. A command `/hiphops pr-to-prod 1234` to kick off your workflow, carrying a PR through a multi-stage release process all the way to live.
