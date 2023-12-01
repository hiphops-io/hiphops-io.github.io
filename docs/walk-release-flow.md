# Creating an automated release flow with GitHub and Slack

Here we show how you can use Hiphops paired with GitHub and Slack to automate a multi-environment release process

## Objective

Our objective in this walkthrough is to create an automated release/pre-release flow for all the repos in our org.

This is Hiphops.io's actual internal release process for the time being (we'll be enhancing this pretty regularly as we add more features)

When we're finished, we'll have a flow that does the following:

1. On push/merge into any `main` branch, we generate a pre-release + tag with an automatic, memorable version name
1. We run the `deploy.yaml` workflow for that repo against that newly created tag
1. We create a custom UI using a Hiphops task, responsible for triggering a live/production release
1. When that task is submitted, a release is created with the given version and auto generated release notes
1. In addition, each step results in notifications to Slack, including links to debug failed workflows and view releases


The end state is that we can release to our stage environments (or create a pre-release for hops) just by merging. We can create full releases with just a click.


## Watch

<iframe width="500" height="500" src="https://www.youtube.com/embed/BtiBp53XHDo?si=IuM2ALpJlTtW8f0O" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## The Code

The .hops we created in this video is here:

```hcl
on push {
  name = "prerelease"

  if = event.ref == "refs/heads/main"

  call github_create_tag {
    name = "create_tag"

    inputs = {
      repo = event.repository.name
      tag = versiontmpl("[yy][mm]-[adj]-[pet]")
      sha = event.after
    }
  }

  call github_create_release {
    name = "create_release"

    if = create_tag.completed

    inputs = {
      repo = event.repository.name
      tag_name = create_tag.json.created_tag.tag
      release_name = "Pre-release ${create_tag.json.created_tag.tag}"
      prerelease = true
      make_latest = false
    }
  }

  call github_dispatch_workflow {
    name = "run_workflow"

    if = create_release.completed

    inputs = {
      repo = event.repository.name
      workflow = "deploy.yaml"
      branch_or_tag = "main"
      inputs = {
        ref = create_tag.json.created_tag.tag
        env = "stage"
      }
    }
  }

  call slack_post_message {
    if = run_workflow.done

    inputs = {
      channel = "engineering"
      text = (run_workflow.errored ? 
      format(":warning: Pre-release workflow failed to start on ${ event.repository.name }, error is: %v", run_workflow.hops.error) : 
      ":white_check_mark: Pre-release <${event.repository.html_url}/releases/tag/${create_tag.json.created_tag.tag}|${create_tag.json.created_tag.tag}> created for ${event.repository.name}")
    }
  }
}

// Live release process
task release_live {
  display_name = "Release To Live"
  emoji = "🚀"

  param version {
    help = "Enter the version for the next release. Format is v0.0.0"
    required = true
  }

  param repo {
    help = "Enter the repo name, e.g. 'hops'"
    required = true
    default = "hops"
  }
}

on task_release_live {
  call github_api {
    name = "create_release"

    inputs = {
      path = "/repos/{owner}/{repo}/releases"
      method = "POST"
      params = {
        repo = event.repo
        tag_name = event.version
        name = "Release ${event.version}"
        generate_release_notes = true
      }
    }
  }

  call github_dispatch_workflow {
    name = "run_workflow"

    if = create_release.completed

    inputs = {
      repo = event.repo
      workflow = "deploy.yaml"
      branch_or_tag = "main"
      inputs = {
        ref = event.version
        env = "live"
      }
    }
  }

  call slack_post_message {
    if = run_workflow.done

    inputs = {
      channel = "engineering"
      text = (run_workflow.errored ? 
      format(":warning: Release workflow failed to start on %s, error is: %v", event.repo, run_workflow.hops.error) : 
      ":rocket: Release <${create_release.json.html_url}|${event.version}> created for ${event.repo}")
    }
  }
}

// On workflow run completion (stage or live), notify slack of the outcome
on workflowrun_completed {
  // Filter by runs that are release deploys
  if = alltrue(
    glob(event.workflow.path, "**/deploy.yaml"),
    event.workflow_run.head_branch == "main"
  )

  call slack_post_message {
    if = event.workflow_run.conclusion == "success"

    inputs = {
      channel = "engineering"
      text = ":information_source: <${event.workflow_run.html_url}|${event.workflow_run.display_title}> completed on ${event.workflow_run.repository.name}"
    }
  }

  call slack_post_message {
    if = event.workflow_run.conclusion != "success"

    inputs = {
      channel = "engineering"
      text = <<-EOT
      :warning: ${event.workflow_run.display_title} unsuccessful. Status was `${event.workflow_run.conclusion}`

      <${event.workflow_run.html_url}|Click here to debug/re-run>
      EOT
    }
  }
}
```


Additionally, we have worklows in each of our GitHub repos named deploy.yaml that look like this:

```yaml
run-name: Release to ${{ inputs.env }} @ ${{ inputs.ref }}

on:
  workflow_dispatch:
    inputs:
      ref:
        description: "The ref/tag to deploy"
        required: true
        type: string
      env:
        description: "The environment to deploy to"
        required: true
        type: choice
        options:
          - live
          - stage

concurrency:
  group: deploy-${{ inputs.env }}
  cancel-in-progress: true

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ inputs.ref }}

    # ... rest of pipeline
```
