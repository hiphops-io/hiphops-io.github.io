# GitHub

_Integrating with GitHub enables you to automate developer flows including approvals, merges, and releases._<br>
_Using Hiphops enables easier and more sophisticated integrations with your other tools, such as Slack._

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`github`|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Add via hiphops.io account page|Credential free (via GitHub App)|


> Note: The structure of GitHub events is mostly identical to the events directly emitted by Github.<br>
> Whilst we'll show examples here, the GitHub docs are usually more exhaustive for exact event structures.

---

## Event: `checksuite`

actions: `completed`, `requested`, `rerequested`

**Example usage:**

```hcl
# example.hops

on checksuite {...} // Will trigger on any checksuite event

on checksuite_completed {...} // Will trigger on any checksuite completed events

on checksuite_requested { // Matches any checksuite requested event...
  if = event.repository.name == "backend" // ... then filters to a specific repo
  ...
} 
```

**Example event:**

[GitHub checksuite](_sample_events/github_check_suite_completed.json ':include')


For full details of this event's possible values, see [GitHub checksuite event docs](https://docs.github.com/en/webhooks/webhook-events-and-payloads#check_suite)

---

## Event: `checkrun`

actions: `completed`, `created`, `requested_action`, `rerequested`

**Example usage:**

```hcl
# example.hops

on checkrun {...} // Will trigger on any checkrun event

on checkrun_completed {...} // Will trigger on any checkrun completed events

on checkrun_created { // Matches any checkrun created event...
  if = event.repository.name == "backend" // ... then filters to a specific repo
  ...
} 
```

**Example event:**

[GitHub checkrun](_sample_events/github_check_run_completed.json ':include')


For full details of this event's possible values, see [GitHub checkrun event docs](https://docs.github.com/en/webhooks/webhook-events-and-payloads#check_run)

---

## Event: `issuecomment`

actions: `created`, `deleted`, `edited`

**Example usage:**

```hcl
# example.hops

on issuecomment {...} // Will trigger on any issuecomment event

on issuecomment_created {...} // Will trigger on any issuecomment created events

on issuecomment_created { // Matches any issuecomment created event...
  if = event.repository.name == "backend" // ... then filters to a specific repo
  ...
} 
```

**Example event:**

[GitHub issuecomment](_sample_events/github_issue_comment_created.json ':include')


For full details of this event's possible values, see [GitHub issue_comment event docs](https://docs.github.com/en/webhooks/webhook-events-and-payloads#issue_comment)

---

## Event: `pullrequest`

actions: `opened`, `closed`, `reopened`, `merged`, `edited`, `assigned`, `unassigned`, `labeled`, `unlabeled`, `synchronize`, `converted_to_draft`, `ready_for_review`, `locked`, `unlocked`, `review_requested`, `review_request_removed`, `auto_merge_enabled`, `auto_merge_disabled`


**Example usage:**

```hcl
# example.hops

on pullrequest {...} // Will trigger on _any_ pullrequest action

on pullrequest_opened {...} // Will trigger when a pullrequest is opened

on pullrequest { // Matches any PR...
  if = event.repository.name == "backend" // ... then filters to a specific repo
}
```

**Example event:**

[GitHub PR sample](_sample_events/github_pull_request.json ':include')


For full details of this event's possible values, see [GitHub pull request event docs](https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#pull_request)

---

## Event: `pullrequestreview`

actions: `dismissed`, `edited`, `submitted`

**Example usage:**

```hcl
# example.hops

on pullrequestreview {...} // Will trigger on any pullrequestreview event

on pullrequestreview_submitted {...} // Will trigger on any pullrequestreview submitted events

on pullrequestreview_submitted { // Matches any pullrequestreview submitted event...
  if = event.repository.name == "backend" // ... then filters to a specific repo
  ...
} 
```

**Example event:**

[GitHub pullrequestreview](_sample_events/github_pull_request_review_submitted.json ':include')


For full details of this event's possible values, see [GitHub pull_request_review event docs](https://docs.github.com/en/webhooks/webhook-events-and-payloads#pull_request_review)

---

## Event: `push`

actions: `N/A`

**Example usage:**

```hcl
# example.hops

on push {...} // Will trigger on push

on push { // Matches any push...
  if = event.ref == "refs/heads/main" // ... then filters by branch `main`
  ...
} 
```

**Example event:**

[GitHub push sample](_sample_events/github_push.json ':include')


For full details of this event's possible values, see [GitHub push event docs](https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#push)

---

## Event: `workflowrun`

actions: `completed`, `requested`, `in_progress`

**Example usage:**

```hcl
# example.hops

on workflowrun {...} // Will trigger on any workflowrun event

on workflowrun_completed {...} // Will trigger on any workflowrun completed events

on workflowrun_completed { // Matches any workflowrun completed event...
  if = event.workflow.path == ".github/workflows/release.yaml" // ... then filters by the release.yaml workflow
  ...
} 
```

**Example event:**

[GitHub workflowrun](_sample_events/github_workflow_run_completed.json ':include')


For full details of this event's possible values, see [GitHub workflowrun event docs](https://docs.github.com/en/webhooks/webhook-events-and-payloads#workflow_run)

---

## Call: `api`

A simple proxy exposing the full GitHub REST API through an authenticated instance of the Hiphops GitHub App.

Whilst we provide multiple helper calls that provide a more polished experience, this ensures any endpoint exposed by GitHub can be used within pipelines.

Refer to the full [GitHub REST API docs](https://docs.github.com/en/rest) for exhaustive information.

> Note: The app will set `owner` for you and handle auth, but everything else is transparent.

**Call structure:**

```hcl
call github_api {
  inputs = {
    path = "/repos/{owner}/{repo}/issues/{issue_number}/labels" // String - the endpoint path as detailed in the GitHub REST API docs
    method = "POST" // (Optional) string - Default: "GET" The HTTP method to use
    params = { // (Optional) object containing params required for the endpoint as detailed in the GitHub REST API docs
      repo = "backend"
      issue_number = 55
    }
    headers = { // (Optional) object containing headers to set
      "Content-Type": "foo"
    }
    data = "" // (Optional) String payload to send. Only one of `data` and `json` may be set
    json = { // (Optional) object that will be encoded as JSON and used as the request payload
      labels = ["kind/bugfix"]
    }
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T17:55:11.650Z",
    "finished_at": "2023-11-13T17:55:12.708Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": [
    {
      "name": "abranch",
      "commit": {
        "sha": "4f66071305da15ea4a61c02d02971ebad59d0f4a",
        "url": "https://api.github.com/repos/example-com/integration-test/commits/e1bbed5043a542239376c3ef3436d842b4b1d965"
      },
      "protected": false,
      "protection": {
        "enabled": false,
        "required_status_checks": {
          "enforcement_level": "off",
          "contexts": [],
          "checks": []
        }
      },
      "protection_url": "https://api.github.com/repos/example-com/integration-test/branches/f890ccaafb52ac89aac5/protection"
    },
    {
      "name": "main",
      "commit": {
        "sha": "0ed9b759504bbce0a8160f032455be4637220373",
        "url": "https://api.github.com/repos/example-com/integration-test/commits/80854325b37ee7add353bd37c7208c9160f5b0e3"
      },
      "protected": false,
      "protection": {
        "enabled": false,
        "required_status_checks": {
          "enforcement_level": "off",
          "contexts": [],
          "checks": []
        }
      },
      "protection_url": "https://api.github.com/repos/example-com/integration-test/branches/main/protection"
    }
  ],
  "status_code": 200,
  "headers": {
    "access-control-allow-origin": "*",
    "access-control-expose-headers": "ETag, Link, Location, Retry-After, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Used, X-RateLimit-Resource, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval, X-GitHub-Media-Type, X-GitHub-SSO, X-GitHub-Request-Id, Deprecation, Sunset",
    "cache-control": "private, max-age=60, s-maxage=60",
    "connection": "close",
    "content-encoding": "gzip",
    "content-security-policy": "default-src 'none'",
    "content-type": "application/json; charset=utf-8",
    "date": "Mon, 13 Nov 2023 17:55:12 GMT",
    "etag": "W/\"a931b0134f658adecd402996bf78eafe25f99533c7d693cf1dbfe17f722d\"",
    "link": "<https://api.github.com/repositories/125fb2e10/branches?per_page=2&page=2>; rel=\"next\", <https://api.github.com/repositories/125fb2e10/branches?per_page=2&page=2>; rel=\"last\"",
    "referrer-policy": "origin-when-cross-origin, strict-origin-when-cross-origin",
    "server": "GitHub.com",
    "strict-transport-security": "max-age=31536000; includeSubdomains; preload",
    "transfer-encoding": "chunked",
    "vary": "Accept, Authorization, Cookie, X-GitHub-OTP, Accept-Encoding, Accept, X-Requested-With",
    "x-accepted-github-permissions": "contents=read",
    "x-content-type-options": "nosniff",
    "x-frame-options": "deny",
    "x-github-api-version-selected": "2022-11-28",
    "x-github-media-type": "github.v3; format=json",
    "x-github-request-id": "A3F4:12345:1A2B3C4D:5E6F7G8H:9I0J1K2L",
    "x-ratelimit-limit": "5150",
    "x-ratelimit-remaining": "5149",
    "x-ratelimit-reset": "1699901712",
    "x-ratelimit-resource": "core",
    "x-ratelimit-used": "1",
    "x-xss-protection": "0"
  },
  "url": "https://api.github.com/repos/example-com/integration-test/branches?per_page=2",
  "context": "AppService"
}
```

> Note: The top level keys and `hops` object will be consistent across calls, but the content within them will be a transparent forwarding of the result from the GitHub API. Refer to [GitHub's docs](https://docs.github.com/en/rest) for full result information per endpoint

---


## Call: `apply_pr_labels`

Label a PR.

The input `labels` defines the labels that may be added to the PR. 

If you also provide the `matching` input, then only labels that match one of the patterns will be applied.

If you provide `matching` and `update = true` inputs, then the task will first remove any labels that match `matching` before applying the new labels.

This allows you to update structured labels, e.g. `matching = ["kind/*"]` and `update = true` would allow you to modify the `kind/*` labels, rather than simply appending a new label.

**Call structure:**

```hcl
call github_apply_pr_labels {
  inputs = {
    repo = "backend" // String - The name of the repository the PR is in
    pr_number = 55 // Number - The PR number
    labels = ["kind/fix", "size/medium", "health/good"] // String array - The labels to be applied to the PR
    matching = ["kind/*", "size/*"] // (Optional) string array - Each string is a glob matching pattern that has the behaviour described above
    update = true // (Optional) boolean - Defaults to false. If true, attempt to update existing labels. If false just apply labels directly
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": null
}
```

---

## Call: `create_branch`

Creates a branch in Github.

One of `sha` or `branch_from` must be provided - if both are provided, only `branch_from` will be used.

**Call structure:**

```hcl
call github_create_branch {
  inputs = {
    repo = "backend" // String - the repository the branch should be created in
    branch = "a-nice-branch" // String - the name of the branch
    sha = "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07" // String (optional) - the SHA to create the branch from
    branch_from = "main" // String (optional) - the branch to create the branch from
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "created_branch": {
      "branch": "a-branch", // The branch name
      "branch_ref": "refs/heads/a-branch", // The branch ref
      "sha": "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07", // The SHA of the branch
    },
    "message": "Successfully created branch refs/heads/a-branch",
  }
}
```

---

## Call: `create_or_update_pr_comment`

Creates or updates a comment on a PR.

By default, this handler will update the first comment that was posted by Hiphops, or create a new comment if not found.

If you intend to have multiple, distinct comments posted by Hiphops you should add a `comment_identifier` that is unique for each comment.

Passing this same `comment_identifier` to any `create_or_update_pr_comment` will result in that comment being updated if it exists.

**Call structure:**

```hcl
call github_create_or_update_pr_comment {
  inputs = {
    repo = "backend" // String - The name of the repository the PR is in
    pr_number = 55 // Number - The PR number
    comment_body = "This is a comment" // String - The text to post in the comment
    comment_identifier = "my-comment-id-foo" // (Optional) string - An identifier used for making updates to the same comment. This is only needed if you intend to have Hiphops post more than one comment to the same PR
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": null
}
```

---

## Call: `create_or_update_pr_review`

Creates or updates a review on a PR.

If you mark a review as `approved = true` then the review will be approved. If marked as `approved = false` then it will instead be a request for changes.

By default, this handler will update the first review that was created by Hiphops, or create a new review if not found.

If you intend to have multiple, distinct reviews posted by Hiphops you should add a `review_identifier` that is unique for each review.

Passing this same `review_identifier` to any `create_or_update_pr_review` will result in that review being updated if it exists.


**Call structure:**

```hcl
call github_create_or_update_pr_review {
  inputs = {
    repo = "backend" // String - The name of the repository the PR is in
    pr_number = 53 // Number - The PR number
    approved = true // Boolean - Is the review an approval or request for changes
    review_body = "Auto approved your PR!" // String - The text to post in the review
    review_identifier = "auto-approve-id-foo" // (Optional) string - An identifier used for making updates to the same review. Only needed if you intend to have Hiphops post more than one review to the same PR
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": null
}
```

---

## Call: `create_pr`

Creates a PR in Github.

If the `source_branch` does not exist, it will be created.<br>
If the `source_branch` and `target_branch` have no difference in commits, it will create an empty commit.


**Call structure:**

```hcl
call github_create_pr {
  inputs = {
    repo = "backend" // String - the repository the PR should be created in
    title = "A PR" // String - the PR title
    source_branch = "release/branch" // String - the source branch the PR will merge in. If it does not exist it will be created
    target_branch = "main" // (Optional) string - the target branch the PR will be merging into. Default: repository's default branch
    body = "This describes the PR" // (Optional) string - the body of the PR
    draft = false // (Optional) boolean - should the PR be created as a draft? Default: false
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": { "pr_number": 70, "message": "Successfully created PR 70" }
}
```

---

## Call: `create_release`

Creates a release in Github. Will create the corresponding tag if it doesn't already exist.

**Call structure:**

```hcl
call github_create_release {
  inputs = {
    repo = "backend" // String - repository to create the release in
    tag_name = "7eafcf0107" // String - the name of the tag (e.g. v1.0.0). If it already exists, the release will use this. If it doesn’t exist, the tag will be created
    release_name = "a test release" // String - the name of the release
    release_body = "This release contains a fixes" // (Optional) string - the body text of the release
    draft = false // (Optional) boolean - Whether or not to create the release as a draft. Default = false
    prerelease = false // (Optional) boolean - Whether or not to create the release as a prerelease. Default = false
    make_latest = true // (Optional) boolean - Whether or not to mark the release as the latest for the repo. Default = true
    target = "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07" // (Optional) string - the target to create the tag for, either a commit SHA or branch ref - required if the tag specified by tag_name does not exist.
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": null
}
```

---

## Call: `create_tag`

Creates a tag in Github.

**Call structure:**

```hcl
call github_create_tag {
  inputs = {
    repo = "backend" // String - the repository the tag should be created in
    tag = "v23.01.05" // String - the name of the tag
    tag_message = "Final version of v23.01.05" // (Optional) string - The message that will be provided with the tag. Default = same value as tag
    sha = "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07" // String - the SHA to create the tag for
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "created_tag": {
      "tag": "v23.01.05", // The tag name
      "tag_ref": "refs/tags/v23.01.05", // The tag ref
      "tagged_sha": "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07", // The SHA that was tagged
      "tag_sha": "940bd336248efae0f9ee5bc7b2d5c985887b16ac", // The SHA of the tag object
      "tag_message": "Final version of v23.01.05", // The tag message
    },
    "message": "Successfully created tag v23.01.05",
  }
}
```

---

## Call: `dispatch_workflow`

Dispatches a workflow for a given repository.

**Call structure:**

```hcl
call github_dispatch_workflow {
  inputs = {
    repo = "backend" # String - The name of the repository containing the workflow to dispatch
    workflow = "hello-world.yaml" # String or Number - Either the string name of the workflow yaml file to dispatch, or the number ID of the workflow
    branch_or_tag = "main" # String - The name of either a branch or tag to run the workflow against
    inputs = { // (Optional) object with string key-value pairs - The inputs to provide to the workflow run
      foo = "bar"
    }
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": null
}
```

---

## Call: `fetch_pr_commits`

Fetches details about the commits in a PR.


**Call structure:**

```hcl
call github_fetch_pr_commits {
  inputs = {
    repo = "backend" // String - the repository the branch should be created in
    pr_number = 55 // Number - The PR number
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "pr_commits": [
      {
        "sha": "59f34daa2771a1e88d88ae442016643923ed85ef",
        "node_id": "C_kwDOIZTUWtoAKDU5ZjM0ZGFhMjc3MWExZTg4ZDg4YWU0NDIwMTY2NDM5MjNlZDg1ZWY",
        "commit": {
          "author": {
            "name": "Some guy",
            "email": "someguy@gmail.com",
            "date": "2022-12-02T10:53:56Z"
          },
          "committer": {
            "name": "Some guy",
            "email": "someguy@gmail.com",
            "date": "2022-12-02T10:53:56Z"
          },
          "message": "creating random file for testing",
          "tree": {
            "sha": "bdb36a18c90eba33e114302fde9c33786be773e4",
            "url": "https://api.github.com/repos/hiphops-io/backend/git/trees/bdb36a18c90eba33e114302fde9c33786be773e4"
          },
          "url": "https://api.github.com/repos/hiphops-io/backend/git/commits/59f34daa2771a1e88d88ae442016643923ed85ef",
          "comment_count": 0,
          "verification": {
            "verified": false,
            "reason": "unsigned",
            "signature": null,
            "payload": null
          }
        },
        "url": "https://api.github.com/repos/hiphops-io/backend/commits/59f34daa2771a1e88d88ae442016643923ed85ef",
        "html_url": "https://github.com/hiphops-io/backend/commit/59f34daa2771a1e88d88ae442016643923ed85ef",
        "comments_url": "https://api.github.com/repos/hiphops-io/backend/commits/59f34daa2771a1e88d88ae442016643923ed85ef/comments",
        "author": {
          "login": "SomeGuy",
          "id": 2988379,
          "node_id": "MDQ6VXNlcjI5ODgzNzk=",
          "avatar_url": "https://avatars.githubusercontent.com/u/2988379?v=4",
          "gravatar_id": "",
          "url": "https://api.github.com/users/SomeGuy",
          "html_url": "https://github.com/SomeGuy",
          "followers_url": "https://api.github.com/users/SomeGuy/followers",
          "following_url": "https://api.github.com/users/SomeGuy/following{/other_user}",
          "gists_url": "https://api.github.com/users/SomeGuy/gists{/gist_id}",
          "starred_url": "https://api.github.com/users/SomeGuy/starred{/owner}{/repo}",
          "subscriptions_url": "https://api.github.com/users/SomeGuy/subscriptions",
          "organizations_url": "https://api.github.com/users/SomeGuy/orgs",
          "repos_url": "https://api.github.com/users/SomeGuy/repos",
          "events_url": "https://api.github.com/users/SomeGuy/events{/privacy}",
          "received_events_url": "https://api.github.com/users/SomeGuy/received_events",
          "type": "User",
          "site_admin": false
        },
        "committer": {
          "login": "SomeGuy",
          "id": 2988379,
          "node_id": "MDQ6VXNlcjI5ODgzNzk=",
          "avatar_url": "https://avatars.githubusercontent.com/u/2988379?v=4",
          "gravatar_id": "",
          "url": "https://api.github.com/users/SomeGuy",
          "html_url": "https://github.com/SomeGuy",
          "followers_url": "https://api.github.com/users/SomeGuy/followers",
          "following_url": "https://api.github.com/users/SomeGuy/following{/other_user}",
          "gists_url": "https://api.github.com/users/SomeGuy/gists{/gist_id}",
          "starred_url": "https://api.github.com/users/SomeGuy/starred{/owner}{/repo}",
          "subscriptions_url": "https://api.github.com/users/SomeGuy/subscriptions",
          "organizations_url": "https://api.github.com/users/SomeGuy/orgs",
          "repos_url": "https://api.github.com/users/SomeGuy/repos",
          "events_url": "https://api.github.com/users/SomeGuy/events{/privacy}",
          "received_events_url": "https://api.github.com/users/SomeGuy/received_events",
          "type": "User",
          "site_admin": false
        },
        "parents": [
          {
            "sha": "401f0414b010baf45922f0d4253d0070ee27d542",
            "url": "https://api.github.com/repos/hiphops-io/backend/commits/401f0414b010baf45922f0d4253d0070ee27d542",
            "html_url": "https://github.com/hiphops-io/backend/commit/401f0414b010baf45922f0d4253d0070ee27d542"
          }
        ]
      }
    ],
    "message": "Github PR commit data retrieved"
  }
}
```

If `completed = true` the result will contain the key `pr_commits`, an array of data about the PR's commits matching the output of [Github's list pull requests commits endpoint](https://docs.github.com/en/rest/pulls/pulls#list-commits-on-a-pull-request).


---

## Call: `fetch_pr_files`

Fetch details about the files changed in a PR.

Useful to make decisions about automated approvals, pipelines to run, other actions to take.

**Call structure:**

```hcl
call github_fetch_pr_files {
  inputs = {
    repo = "backend" // String - The name of the repository the PR is in
    pr_number = 55 // Number - The PR number
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "pr_files": [
      {
        "sha": "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07",
        "filename": "README.md",
        "status": "added",
        "additions": 1,
        "deletions": 0,
        "changes": 1,
        "blob_url": "https://github.com/example-com/backend/blob/22b88538a0fadf73b427fe749ae7bbbaece2ee57/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt",
        "raw_url": "https://github.com/example-com/backend/raw/22b88538a0fadf73b427fe749ae7bbbaece2ee57/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt",
        "contents_url": "https://api.github.com/repos/example-com/backend/contents/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt?ref=22b88538a0fadf73b427fe749ae7bbbaece2ee57",
        "patch": "@@ -0,0 +1 @@\n+e1ec5e842183b1878ccc61c8368108b1b1d6198c\n\\ No newline at end of file"
      }
    ],
    "message": "Github PR file data retrieved"
  }
}
```

A successful (`completed = true`) call will contain the key `pr_files`, an array of data about the PR's files matching the output of [GitHub's list pull requests files endpoint](https://docs.github.com/en/rest/pulls/pulls#list-pull-requests-files).


---

## Call: `fetch_repo_prs`

Fetches details about the PRs in a repo.


**Call structure:**

```hcl
call github_fetch_repo_prs {
  inputs = {
    repo = "backend" // String - The name of the repository to fetch PRs from
    state = "open" // (Optional) string - either "open", "closed", or "all" to filter by state. Default = "all"
    head = "octocat:test-branch" // (Optional) string - filter pulls by head user or head organization and branch name in the format of "user =ref-name" or "organization:ref-name"
    base = "gh-pages" // (Optional) string - filter pulls by base branch name.
    sort = "updated" // (Optional) string - what to sort results by. One of: "created", "updated", "popularity", "long-running". Default: "created"
    direction = "desc" // (Optional) string - direction of sort. Default = "desc" when sort is "created" or not specified, otherwise "asc".
    per_page = 1 // (Optional) integer - number of results per page (max 100). Default = 100
    page = 2 // (Optional) integer - page number of results to fetch. Default = 1
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": {
    "repo_prs": [
      {
        "url": "https://api.github.com/repos/hiphops-io/backend/pulls/346",
        "id": 1143068882,
        "node_id": "PR_kwDOIZTUWs5EIdjS",
        "html_url": "https://github.com/hiphops-io/backend/pull/346",
        "diff_url": "https://github.com/hiphops-io/backend/pull/346.diff",
        "patch_url": "https://github.com/hiphops-io/backend/pull/346.patch",
        "issue_url": "https://api.github.com/repos/hiphops-io/backend/issues/346",
        "number": 346,
        "state": "open",
        "locked": false,
        "title": "pull request title",
        "user": {
          "login": "SomeGuy",
          "id": 2988379,
          "node_id": "MDQ6VXNlcjI5ODgzNzk=",
          "avatar_url": "https://avatars.githubusercontent.com/u/2988379?v=4",
          "gravatar_id": "",
          "url": "https://api.github.com/users/SomeGuy",
          "html_url": "https://github.com/SomeGuy",
          "followers_url": "https://api.github.com/users/SomeGuy/followers",
          "following_url": "https://api.github.com/users/SomeGuy/following{/other_user}",
          "gists_url": "https://api.github.com/users/SomeGuy/gists{/gist_id}",
          "starred_url": "https://api.github.com/users/SomeGuy/starred{/owner}{/repo}",
          "subscriptions_url": "https://api.github.com/users/SomeGuy/subscriptions",
          "organizations_url": "https://api.github.com/users/SomeGuy/orgs",
          "repos_url": "https://api.github.com/users/SomeGuy/repos",
          "events_url": "https://api.github.com/users/SomeGuy/events{/privacy}",
          "received_events_url": "https://api.github.com/users/SomeGuy/received_events",
          "type": "User",
          "site_admin": false
        },
        "body": "pull request description",
        "created_at": "2022-12-02T10:58:05Z",
        "updated_at": "2022-12-02T10:58:05Z",
        "closed_at": null,
        "merged_at": null,
        "merge_commit_sha": "1eaed5bbfb0c22ee7418265b04bdd563432cb3a1",
        "assignee": null,
        "assignees": [],
        "requested_reviewers": [],
        "requested_teams": [],
        "labels": [],
        "milestone": null,
        "draft": false,
        "commits_url": "https://api.github.com/repos/hiphops-io/backend/pulls/346/commits",
        "review_comments_url": "https://api.github.com/repos/hiphops-io/backend/pulls/346/comments",
        "review_comment_url": "https://api.github.com/repos/hiphops-io/backend/pulls/comments{/number}",
        "comments_url": "https://api.github.com/repos/hiphops-io/backend/issues/346/comments",
        "statuses_url": "https://api.github.com/repos/hiphops-io/backend/statuses/6e7882fd90b7e005eac0ff02a5f1aca85b89b528",
        "head": {
          "label": "hiphops-io:07e5b03c49497777021f",
          "ref": "07e5b03c49497777021f",
          "sha": "6e7882fd90b7e005eac0ff02a5f1aca85b89b528",
          "user": {
            "login": "hiphops-io",
            "id": 77837455,
            "node_id": "MDEyOk9yZ2FuaXphdGlvbjc3ODM3NDU1",
            "avatar_url": "https://avatars.githubusercontent.com/u/77837455?v=4",
            "gravatar_id": "",
            "url": "https://api.github.com/users/hiphops-io",
            "html_url": "https://github.com/hiphops-io",
            "followers_url": "https://api.github.com/users/hiphops-io/followers",
            "following_url": "https://api.github.com/users/hiphops-io/following{/other_user}",
            "gists_url": "https://api.github.com/users/hiphops-io/gists{/gist_id}",
            "starred_url": "https://api.github.com/users/hiphops-io/starred{/owner}{/repo}",
            "subscriptions_url": "https://api.github.com/users/hiphops-io/subscriptions",
            "organizations_url": "https://api.github.com/users/hiphops-io/orgs",
            "repos_url": "https://api.github.com/users/hiphops-io/repos",
            "events_url": "https://api.github.com/users/hiphops-io/events{/privacy}",
            "received_events_url": "https://api.github.com/users/hiphops-io/received_events",
            "type": "Organization",
            "site_admin": false
          },
          "repo": {
            "id": 563401818,
            "node_id": "R_kgDOIZTUWg",
            "name": "backend",
            "full_name": "hiphops-io/backend",
            "private": true,
            "owner": {
              "login": "hiphops-io",
              "id": 77837455,
              "node_id": "MDEyOk9yZ2FuaXphdGlvbjc3ODM3NDU1",
              "avatar_url": "https://avatars.githubusercontent.com/u/77837455?v=4",
              "gravatar_id": "",
              "url": "https://api.github.com/users/hiphops-io",
              "html_url": "https://github.com/hiphops-io",
              "followers_url": "https://api.github.com/users/hiphops-io/followers",
              "following_url": "https://api.github.com/users/hiphops-io/following{/other_user}",
              "gists_url": "https://api.github.com/users/hiphops-io/gists{/gist_id}",
              "starred_url": "https://api.github.com/users/hiphops-io/starred{/owner}{/repo}",
              "subscriptions_url": "https://api.github.com/users/hiphops-io/subscriptions",
              "organizations_url": "https://api.github.com/users/hiphops-io/orgs",
              "repos_url": "https://api.github.com/users/hiphops-io/repos",
              "events_url": "https://api.github.com/users/hiphops-io/events{/privacy}",
              "received_events_url": "https://api.github.com/users/hiphops-io/received_events",
              "type": "Organization",
              "site_admin": false
            },
            "html_url": "https://github.com/hiphops-io/backend",
            "description": null,
            "fork": false,
            "url": "https://api.github.com/repos/hiphops-io/backend",
            "forks_url": "https://api.github.com/repos/hiphops-io/backend/forks",
            "keys_url": "https://api.github.com/repos/hiphops-io/backend/keys{/key_id}",
            "collaborators_url": "https://api.github.com/repos/hiphops-io/backend/collaborators{/collaborator}",
            "teams_url": "https://api.github.com/repos/hiphops-io/backend/teams",
            "hooks_url": "https://api.github.com/repos/hiphops-io/backend/hooks",
            "issue_events_url": "https://api.github.com/repos/hiphops-io/backend/issues/events{/number}",
            "events_url": "https://api.github.com/repos/hiphops-io/backend/events",
            "assignees_url": "https://api.github.com/repos/hiphops-io/backend/assignees{/user}",
            "branches_url": "https://api.github.com/repos/hiphops-io/backend/branches{/branch}",
            "tags_url": "https://api.github.com/repos/hiphops-io/backend/tags",
            "blobs_url": "https://api.github.com/repos/hiphops-io/backend/git/blobs{/sha}",
            "git_tags_url": "https://api.github.com/repos/hiphops-io/backend/git/tags{/sha}",
            "git_refs_url": "https://api.github.com/repos/hiphops-io/backend/git/refs{/sha}",
            "trees_url": "https://api.github.com/repos/hiphops-io/backend/git/trees{/sha}",
            "statuses_url": "https://api.github.com/repos/hiphops-io/backend/statuses/{sha}",
            "languages_url": "https://api.github.com/repos/hiphops-io/backend/languages",
            "stargazers_url": "https://api.github.com/repos/hiphops-io/backend/stargazers",
            "contributors_url": "https://api.github.com/repos/hiphops-io/backend/contributors",
            "subscribers_url": "https://api.github.com/repos/hiphops-io/backend/subscribers",
            "subscription_url": "https://api.github.com/repos/hiphops-io/backend/subscription",
            "commits_url": "https://api.github.com/repos/hiphops-io/backend/commits{/sha}",
            "git_commits_url": "https://api.github.com/repos/hiphops-io/backend/git/commits{/sha}",
            "comments_url": "https://api.github.com/repos/hiphops-io/backend/comments{/number}",
            "issue_comment_url": "https://api.github.com/repos/hiphops-io/backend/issues/comments{/number}",
            "contents_url": "https://api.github.com/repos/hiphops-io/backend/contents/{+path}",
            "compare_url": "https://api.github.com/repos/hiphops-io/backend/compare/{base}...{head}",
            "merges_url": "https://api.github.com/repos/hiphops-io/backend/merges",
            "archive_url": "https://api.github.com/repos/hiphops-io/backend/{archive_format}{/ref}",
            "downloads_url": "https://api.github.com/repos/hiphops-io/backend/downloads",
            "issues_url": "https://api.github.com/repos/hiphops-io/backend/issues{/number}",
            "pulls_url": "https://api.github.com/repos/hiphops-io/backend/pulls{/number}",
            "milestones_url": "https://api.github.com/repos/hiphops-io/backend/milestones{/number}",
            "notifications_url": "https://api.github.com/repos/hiphops-io/backend/notifications{?since,all,participating}",
            "labels_url": "https://api.github.com/repos/hiphops-io/backend/labels{/name}",
            "releases_url": "https://api.github.com/repos/hiphops-io/backend/releases{/id}",
            "deployments_url": "https://api.github.com/repos/hiphops-io/backend/deployments",
            "created_at": "2022-11-08T14:37:21Z",
            "updated_at": "2022-11-08T14:37:21Z",
            "pushed_at": "2022-12-02T10:58:05Z",
            "git_url": "git://github.com/hiphops-io/backend.git",
            "ssh_url": "git@github.com:hiphops-io/backend.git",
            "clone_url": "https://github.com/hiphops-io/backend.git",
            "svn_url": "https://github.com/hiphops-io/backend",
            "homepage": null,
            "size": 2,
            "stargazers_count": 0,
            "watchers_count": 0,
            "language": null,
            "has_issues": true,
            "has_projects": false,
            "has_downloads": true,
            "has_wiki": true,
            "has_pages": false,
            "has_discussions": false,
            "forks_count": 0,
            "mirror_url": null,
            "archived": false,
            "disabled": false,
            "open_issues_count": 2,
            "license": null,
            "allow_forking": false,
            "is_template": false,
            "web_commit_signoff_required": true,
            "topics": [],
            "visibility": "private",
            "forks": 0,
            "open_issues": 2,
            "watchers": 0,
            "default_branch": "main"
          }
        },
        "base": {
          "label": "hiphops-io:main",
          "ref": "main",
          "sha": "401f0414b010baf45922f0d4253d0070ee27d542",
          "user": {
            "login": "hiphops-io",
            "id": 77837455,
            "node_id": "MDEyOk9yZ2FuaXphdGlvbjc3ODM3NDU1",
            "avatar_url": "https://avatars.githubusercontent.com/u/77837455?v=4",
            "gravatar_id": "",
            "url": "https://api.github.com/users/hiphops-io",
            "html_url": "https://github.com/hiphops-io",
            "followers_url": "https://api.github.com/users/hiphops-io/followers",
            "following_url": "https://api.github.com/users/hiphops-io/following{/other_user}",
            "gists_url": "https://api.github.com/users/hiphops-io/gists{/gist_id}",
            "starred_url": "https://api.github.com/users/hiphops-io/starred{/owner}{/repo}",
            "subscriptions_url": "https://api.github.com/users/hiphops-io/subscriptions",
            "organizations_url": "https://api.github.com/users/hiphops-io/orgs",
            "repos_url": "https://api.github.com/users/hiphops-io/repos",
            "events_url": "https://api.github.com/users/hiphops-io/events{/privacy}",
            "received_events_url": "https://api.github.com/users/hiphops-io/received_events",
            "type": "Organization",
            "site_admin": false
          },
          "repo": {
            "id": 563401818,
            "node_id": "R_kgDOIZTUWg",
            "name": "backend",
            "full_name": "hiphops-io/backend",
            "private": true,
            "owner": {
              "login": "hiphops-io",
              "id": 77837455,
              "node_id": "MDEyOk9yZ2FuaXphdGlvbjc3ODM3NDU1",
              "avatar_url": "https://avatars.githubusercontent.com/u/77837455?v=4",
              "gravatar_id": "",
              "url": "https://api.github.com/users/hiphops-io",
              "html_url": "https://github.com/hiphops-io",
              "followers_url": "https://api.github.com/users/hiphops-io/followers",
              "following_url": "https://api.github.com/users/hiphops-io/following{/other_user}",
              "gists_url": "https://api.github.com/users/hiphops-io/gists{/gist_id}",
              "starred_url": "https://api.github.com/users/hiphops-io/starred{/owner}{/repo}",
              "subscriptions_url": "https://api.github.com/users/hiphops-io/subscriptions",
              "organizations_url": "https://api.github.com/users/hiphops-io/orgs",
              "repos_url": "https://api.github.com/users/hiphops-io/repos",
              "events_url": "https://api.github.com/users/hiphops-io/events{/privacy}",
              "received_events_url": "https://api.github.com/users/hiphops-io/received_events",
              "type": "Organization",
              "site_admin": false
            },
            "html_url": "https://github.com/hiphops-io/backend",
            "description": null,
            "fork": false,
            "url": "https://api.github.com/repos/hiphops-io/backend",
            "forks_url": "https://api.github.com/repos/hiphops-io/backend/forks",
            "keys_url": "https://api.github.com/repos/hiphops-io/backend/keys{/key_id}",
            "collaborators_url": "https://api.github.com/repos/hiphops-io/backend/collaborators{/collaborator}",
            "teams_url": "https://api.github.com/repos/hiphops-io/backend/teams",
            "hooks_url": "https://api.github.com/repos/hiphops-io/backend/hooks",
            "issue_events_url": "https://api.github.com/repos/hiphops-io/backend/issues/events{/number}",
            "events_url": "https://api.github.com/repos/hiphops-io/backend/events",
            "assignees_url": "https://api.github.com/repos/hiphops-io/backend/assignees{/user}",
            "branches_url": "https://api.github.com/repos/hiphops-io/backend/branches{/branch}",
            "tags_url": "https://api.github.com/repos/hiphops-io/backend/tags",
            "blobs_url": "https://api.github.com/repos/hiphops-io/backend/git/blobs{/sha}",
            "git_tags_url": "https://api.github.com/repos/hiphops-io/backend/git/tags{/sha}",
            "git_refs_url": "https://api.github.com/repos/hiphops-io/backend/git/refs{/sha}",
            "trees_url": "https://api.github.com/repos/hiphops-io/backend/git/trees{/sha}",
            "statuses_url": "https://api.github.com/repos/hiphops-io/backend/statuses/{sha}",
            "languages_url": "https://api.github.com/repos/hiphops-io/backend/languages",
            "stargazers_url": "https://api.github.com/repos/hiphops-io/backend/stargazers",
            "contributors_url": "https://api.github.com/repos/hiphops-io/backend/contributors",
            "subscribers_url": "https://api.github.com/repos/hiphops-io/backend/subscribers",
            "subscription_url": "https://api.github.com/repos/hiphops-io/backend/subscription",
            "commits_url": "https://api.github.com/repos/hiphops-io/backend/commits{/sha}",
            "git_commits_url": "https://api.github.com/repos/hiphops-io/backend/git/commits{/sha}",
            "comments_url": "https://api.github.com/repos/hiphops-io/backend/comments{/number}",
            "issue_comment_url": "https://api.github.com/repos/hiphops-io/backend/issues/comments{/number}",
            "contents_url": "https://api.github.com/repos/hiphops-io/backend/contents/{+path}",
            "compare_url": "https://api.github.com/repos/hiphops-io/backend/compare/{base}...{head}",
            "merges_url": "https://api.github.com/repos/hiphops-io/backend/merges",
            "archive_url": "https://api.github.com/repos/hiphops-io/backend/{archive_format}{/ref}",
            "downloads_url": "https://api.github.com/repos/hiphops-io/backend/downloads",
            "issues_url": "https://api.github.com/repos/hiphops-io/backend/issues{/number}",
            "pulls_url": "https://api.github.com/repos/hiphops-io/backend/pulls{/number}",
            "milestones_url": "https://api.github.com/repos/hiphops-io/backend/milestones{/number}",
            "notifications_url": "https://api.github.com/repos/hiphops-io/backend/notifications{?since,all,participating}",
            "labels_url": "https://api.github.com/repos/hiphops-io/backend/labels{/name}",
            "releases_url": "https://api.github.com/repos/hiphops-io/backend/releases{/id}",
            "deployments_url": "https://api.github.com/repos/hiphops-io/backend/deployments",
            "created_at": "2022-11-08T14:37:21Z",
            "updated_at": "2022-11-08T14:37:21Z",
            "pushed_at": "2022-12-02T10:58:05Z",
            "git_url": "git://github.com/hiphops-io/backend.git",
            "ssh_url": "git@github.com:hiphops-io/backend.git",
            "clone_url": "https://github.com/hiphops-io/backend.git",
            "svn_url": "https://github.com/hiphops-io/backend",
            "homepage": null,
            "size": 2,
            "stargazers_count": 0,
            "watchers_count": 0,
            "language": null,
            "has_issues": true,
            "has_projects": false,
            "has_downloads": true,
            "has_wiki": true,
            "has_pages": false,
            "has_discussions": false,
            "forks_count": 0,
            "mirror_url": null,
            "archived": false,
            "disabled": false,
            "open_issues_count": 2,
            "license": null,
            "allow_forking": false,
            "is_template": false,
            "web_commit_signoff_required": true,
            "topics": [],
            "visibility": "private",
            "forks": 0,
            "open_issues": 2,
            "watchers": 0,
            "default_branch": "main"
          }
        },
        "_links": {
          "self": {
            "href": "https://api.github.com/repos/hiphops-io/backend/pulls/346"
          },
          "html": {
            "href": "https://github.com/hiphops-io/backend/pull/346"
          },
          "issue": {
            "href": "https://api.github.com/repos/hiphops-io/backend/issues/346"
          },
          "comments": {
            "href": "https://api.github.com/repos/hiphops-io/backend/issues/346/comments"
          },
          "review_comments": {
            "href": "https://api.github.com/repos/hiphops-io/backend/pulls/346/comments"
          },
          "review_comment": {
            "href": "https://api.github.com/repos/hiphops-io/backend/pulls/comments{/number}"
          },
          "commits": {
            "href": "https://api.github.com/repos/hiphops-io/backend/pulls/346/commits"
          },
          "statuses": {
            "href": "https://api.github.com/repos/hiphops-io/backend/statuses/6e7882fd90b7e005eac0ff02a5f1aca85b89b528"
          }
        },
        "author_association": "CONTRIBUTOR",
        "auto_merge": null,
        "active_lock_reason": null
      }
    ],
    "message": "Github repo PR data retrieved",
  }
}
```

If `completed = true` result  will contain the key `repo_prs`, an array of data about the repo's PRs matching the output of [Github's list pull requests endpoint](https://docs.github.com/en/rest/pulls/pulls?apiVersion=2022-11-28#list-pull-requests).

---

## Call: `merge_pr`

Merges a PR's source branch into its target branch.

**Call structure:**

```hcl
call github_merge_pr {
  inputs = {
    repo = "backend" // String - The name of the repository the PR is in
    pr_number = 55 // Number - The PR number
    merge_comment_title = "Auto-merged this PR!" // (Optional) string - The commit message title that will provided with the merge. Default = PR title
    head_sha = "939abcd18feaa12345bdb" // (Optional) string - The SHA the branch head must be at for the merge to proceed, to prevent race conditions. If not provided the merge will proceed without checking the SHA
    merge_method = "merge" // (Optional) string - Merge method the PR wull be merged with. One of “merge”, “squash” or “rebase”. Default = “merge”
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "",
  "json": null
}
```

---

## Call: `merge_pr_when_ready`

Merges a PR's source branch into its target branch with auto-merge.

When auto merge and branch protections are enabled on the repo, this sets the PR to automatically merge when ready.

However, if the PR can be immediately merged, it will be immediately merged.

(This task functions the same as `merge_pr` if auto-merge is not enabled or the PR is merge ready.)


**Call structure:**


```hcl
call github_merge_when_ready {
  inputs = {
    repo = "backend" // String - The name of the repository the PR is in
    pr_number = 55 // Number - The PR number
    merge_comment_title = "Auto-merged this PR!" // (Optional) string - The commit message title that will provided with the merge. Default = PR title
    head_sha = "939abcd18feaa12345bdb" // (Optional) string - The SHA the branch head must be at for the merge to proceed, to prevent race conditions. If not provided the merge will proceed without checking the SHA
    merge_method = "merge" // (Optional) string - Merge method the PR will be merged with. One of “merge”, “squash” or “rebase”. Default = “merge”
  }
}
```

**Example result:**

```js
{
  "hops": {
    "started_at": "2023-11-13T23:57:20.336Z",
    "finished_at": "2023-11-13T23:57:38.869Z",
    "error": null
  },
  "errored": false,
  "completed": true,
  "done": true,
  "body": "Successfully merged PR 55.", // Could also say "PR 55 will merge when ready." in the event the merge has been queued
  "json": {}
}
```
