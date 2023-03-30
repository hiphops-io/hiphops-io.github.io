# GitHub

_Integrating with GitHub enables you to automate standard developer flows, view your change and release data in Hiphops and monitor your GitHub account to maintain secure config._

|Name|Incoming events|Has tasks|Integration|
|:-------|:-------|:-------|:-------|
|`github`|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Integrated as part of project creation in the UI|


The structure of GitHub events is mostly identical to the events directly emitted by Github. The only difference is that they also contain the following properties at the event root:

```yaml
project_id: <the current project>
hiphops:
  source: github
  event: <the event name>
  action: <the event action>
```

---

## Event: `pull_request`

actions: `opened`, `closed`, `reopened`, `merged`, `edited`, `assigned`, `unassigned`, `labeled`, `unlabeled`, `synchronize`, `converted_to_draft`, `ready_for_review`, `locked`, `unlocked`, `review_requested`, `review_request_removed`, `auto_merge_enabled`, `auto_merge_disabled`

<details>
<summary>See sample event</summary>

[GitHub PR sample](../_sample_events/github_pull_request.json ':include')

</details>

For the details of the source event's possible values, see [GitHub pull request event docs](https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#pull_request)

---

## Event: `push`

actions: `N/A`

<details>
<summary>See sample event</summary>

[GitHub push sample](../_sample_events/github_push.json ':include')

</details>

For the full source event structure, see [GitHub push event docs](https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#push)

---

## Task: `create_pr`

Creates a PR in Github. If the `source_branch` does not exist, it will be created.
Note that if the `source_branch` and `target_branch` have no difference in commits, this will create an empty commit.

```yaml
tasks:
  - name: github.create_pr # String - The name of the task
    input:
      repo: "backend" # String - the repository the PR should be created in
      title: "A PR", # String - the PR title
      source_branch: "release/branch", # String - the source branch the PR will merge in. If it does not exist it will be created
      target_branch: "main", # (Optional) string - the target branch the PR will be merging into. Default: repository's default branch
      body: "This describes the PR", # (Optional) string - the body of the PR
      draft: false # (Optional) boolean - should the PR be created as a draft? Default: false
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

If successful it will respond with a `vars` object, that has - against the key `pr_number` - the number of the newly created PR.

Example:

```js
{
  "vars": {
    "pr_number": 466 // The PR number
  }
}
```


---

## Task: `merge_pr`

Merges a PR's source branch into its target branch.

Assuming this is being triggered in response to an event that contains data about the PR, it's likely you'll want to pass the head SHA along in the `head_sha` input - if this is supplied, the merge will only go ahead if no further commits have been pushed since the event was triggered.

```yaml
tasks:
- name: github.merge_pr
  input:
    repo: backend # String - The name of the repository the PR is in
    pr_number: 55 # Number - The PR number
    merge_comment_title: Auto-merged this PR! # (Optional) string - The commit message title that will provided with the merge. Default: PR title
    head_sha: 939abcd18feaa12345bdb # (Optional) string - The SHA the branch head must be at for the merge to proceed, to prevent race conditions. If not provided the merge will proceed without checking the SHA
    merge_method: merge # (Optional) string - Merge method the PR wull be merged with. One of “merge”, “squash” or “rebase”. Default: “merge”
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

---

## Task: `create_or_update_pr_comment`

Creates or updates a comment on a PR.

If you are only going to have one task post comments to a given PR (e.g. the Hiphops analysis comment), then the first comment posted by the Hiphops app on the given PR is the one that will be updated each time.

However, if you intend to have multiple comments posted by Hiphops, you should add a `comment_identifier` that is distinct for each task - this will be appended to the end of the comment, and subsequent runs of the task (assuming it uses the same identifier) will update the comment on the PR that ends with that identifier.


```yaml
tasks:
  - name: github.create_or_update_pr_comment
    input:
      repo: backend # String - The name of the repository the PR is in
      pr_number: 55 # Number - The PR number
      comment_body: This is a comment # String - The text to post in the comment
      comment_identifier: my-comment-id-foo # (Optional) string - An identifier used for making updates to the same comment. This is only needed if you intend to have Hiphops post more than one comment to the same PR
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

---

## Task: `create_or_update_pr_review`

Creates or updates a review on a PR.

If you mark a review as `approved: true` then the review will be approved. If marked as `approved: false` then it will instead be a request for changes.

If you are only going to have one task post reviews to a given PR, then the first review posted by the Hiphops app on the given PR is the one that will be updated each time.

However, if you intend to have multiple reviews posted by Hiphops, you should add a `review_identifier` that is distinct for each task - this will be appended to the end of the review, and subsequent runs of the task (assuming it uses the same identifier) will update the review on the PR that ends with that identifier.

```yaml
tasks:
  - name: github.create_or_update_pr_review
    input:
      repo: backend # String - The name of the repository the PR is in
      pr_number: 53 # Number - The PR number
      approved: true # Boolean - Is the review an approval or request for changes
      review_body: Auto approved your PR! # String - The text to post in the review
      review_identifier: auto-approve-id-foo # (Optional) string - An identifier used for making updates to the same review. Only needed if you intend to have Hiphops post more than one review to the same PR
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

---

## Task: `apply_pr_labels`

Applies labels to a PR.

The input `labels` defines the labels that may be added to the PR. If you don't provide a `matching` input too, then `labels` entries will just be added to the PR.

However, if you do provide a list of glob-matching patterns in `matching` (e.g `["kind/*", "size/*"]`), then only the entries in `labels` that have a match in one or more of the entries in `matching` will be applied.

By setting the `update` input to `true`, the task will treat the patterns in `matching` as a list of label patterns to be updated, and will first _remove_ labels matching a pattern before applying those labels from the `labels` input that match it.

So if we have a PR 55 in repository `backend` that currently has the labels `["kind/fix", "size/small", "thingy"]`, and this task is invoked with the input:
```yaml
    input:
      repo: backend
      pr_number: 55
      labels: ["kind/fix", "size/medium", "health/good"]
      matching: ["kind/*", "size/*"]
      update: true
```

Then the task would first remove `size/small`, and then add `size/medium`. No other labels would be changed as they don't meet the criteria in `matching`.

If `update` had been `false` (or unset), then it would only add `size/medium`, not attempting to update the existing labels.

```yaml
tasks:
- name: github.apply_pr_labels
  input:
    repo: backend # String - The name of the repository the PR is in
    pr_number: 55 # Number - The PR number
    labels: ["kind/fix", "size/medium", "health/good"] # String array - The labels to be applied to the PR
    matching: ["kind/*", "size/*"] # String array - Each string is a glob matching pattern that will be applied against labels, such that only the labels that have a match will be applied. If update is true, then existing labels on the PR that have a match will be removed too
    update: true # Boolean - If true, attempt to update existing labels, or if false just apply labels as new - if matching is provided, this is used to determine which labels will be updated. Default: false
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

---

## Task: `fetch_pr_files`

Fetches details about the files changed in a PR, placing the data in `vars.pr_files` for use by other tasks.

```yaml
tasks:
  - name: github.fetch_pr_files
    input:
      repo: backend # String - The name of the repository the PR is in
      pr_number: 55 # Number - The PR number
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

If successful it will respond with a `vars` object, that has &mdash; against the key `pr_files` &mdash; an array of data about the PR's files matching the output of [GitHub's list pull requests files endpoint](https://docs.github.com/en/rest/pulls/pulls#list-pull-requests-files).

Example:

```json
{
  "vars": {
    "pr_files": [
      {
        "sha": "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07",
        "filename": "e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt",
        "status": "added",
        "additions": 1,
        "deletions": 0,
        "changes": 1,
        "blob_url": "https://github.com/hiphops-io/integration-test/blob/22b88538a0fadf73b427fe749ae7bbbaece2ee57/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt",
        "raw_url": "https://github.com/hiphops-io/integration-test/raw/22b88538a0fadf73b427fe749ae7bbbaece2ee57/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt",
        "contents_url": "https://api.github.com/repos/hiphops-io/integration-test/contents/e1ec5e842183b1878ccc61c8368108b1b1d6198c.txt?ref=22b88538a0fadf73b427fe749ae7bbbaece2ee57",
        "patch": "@@ -0,0 +1 @@\n+e1ec5e842183b1878ccc61c8368108b1b1d6198c\n\\ No newline at end of file"
      }
    ]
  }
}
```

---

## Task: `create_release`

Creates a release in Github. Will create the corresponding tag if it doesn't already exist.

```yaml
tasks:
  - name: github.create_release # String - The name of the task
    input:
      repo: "integration-test" # String - repository to create the release in
      tag_name: "7eafcf0107" # String - the name of the tag (e.g. v1.0.0). If it already exists, the release will use this. If it doesn’t exist, the tag will be created
      release_name: "a test release" # String - the name of the release
      release_body: "This release contains a fixes" # (Optional) string - the body text of the release
      draft: false # (Optional) boolean - Whether or not to create the release as a draft. Default: false
      prerelease: false # (Optional) boolean - Whether or not to create the release as a prerelease. Default: false
      make_latest: true # (Optional) boolean - Whether or not to mark the release as the latest for the repo. Default: true
      target: "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07" # (Optional) string - the target to create the tag for, either a commit SHA or branch ref - required if the tag specified by tag_name does not exist.
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

---

## Task: `create_tag`

Creates a tag in Github.

```yaml
tasks:
  - name: github.create_tag # String - The name of the task
    input:
      repo: "integration-test" # String - the repository the tag should be created in
      tag: "v23.01.05" # String - the name of the tag
      tag_message: "Final version of v23.01.05" # (Optional) string - The message that will be provided with the tag. Default: same value as tag
      sha: "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07" # String - the SHA to create the tag for
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

If successful it will respond with a `vars` object, that has &mdash; against the key `created_tag` &mdash; an object containing information about the created tag.

Example:

```js
{
  "vars": {
    "created_tag": {
      "tag": "v23.01.05", // The tag name
      "tag_ref": "refs/tags/v23.01.05", // The tag ref
      "tagged_sha": "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07", // The SHA that was tagged
      "tag_sha": "940bd336248efae0f9ee5bc7b2d5c985887b16ac", // The SHA of the tag object
      "tag_message": "Final version of v23.01.05", // The tag message
    }
  }
}
```

---

## Task: `create_branch`

Creates a branch in Github. One of `sha` and `branch_from` must be provided - if both are provided, only `branch_from` will be used.

```yaml
tasks:
  - name: github.create_branch # String - The name of the task
    input:
      repo: "integration-test" # String - the repository the branch should be created in
      branch: "a-nice-branch" # String - the name of the branch
      sha: "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07" # String (optional) - the SHA to create the branch from. One of `sha` and `branch_from` must be provided - if both are provided, only `branch_from` will be used.
      branch_from: "main" # String (optional) - the branch to create the branch from. One of `sha` and `branch_from` must be provided - if both are provided, only `branch_from` will be used.
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

If successful it will respond with a `vars` object, that has - against the key `created_branch` - an object containing information about the created branch.

Example:

```js
{
  "vars": {
    "created_branch": {
      "branch": "a-nice-branch", // The branch name
      "branch_ref": "refs/heads/branches/a-nice-branch", // The branch ref
      "sha": "fd16ba92cbe98ce1747e512ab234d67ce3cc4a07", // The SHA of the branch
    }
  }
}
```

---

## Task: `fetch_pr_commits`

Fetches details about the files changed in a PR, placing the data in `vars.pr_commits` for use by other tasks.

```yaml
tasks:
  - name: github.fetch_pr_commits
    input:
      repo: backend # String - The name of the repository the PR is in
      pr_number: 55 # Number - The PR number
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

If successful it will respond with a `vars` object, that has &mdash; against the key `pr_commits` &mdash; an array of data about the PR's commits matching the output of [Github's list pull requests commits endpoint](https://docs.github.com/en/rest/pulls/pulls#list-commits-on-a-pull-request).

Example:

```json
{
  "vars": {
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
            "url": "https://api.github.com/repos/hiphops-io/integration-test/git/trees/bdb36a18c90eba33e114302fde9c33786be773e4"
          },
          "url": "https://api.github.com/repos/hiphops-io/integration-test/git/commits/59f34daa2771a1e88d88ae442016643923ed85ef",
          "comment_count": 0,
          "verification": {
            "verified": false,
            "reason": "unsigned",
            "signature": null,
            "payload": null
          }
        },
        "url": "https://api.github.com/repos/hiphops-io/integration-test/commits/59f34daa2771a1e88d88ae442016643923ed85ef",
        "html_url": "https://github.com/hiphops-io/integration-test/commit/59f34daa2771a1e88d88ae442016643923ed85ef",
        "comments_url": "https://api.github.com/repos/hiphops-io/integration-test/commits/59f34daa2771a1e88d88ae442016643923ed85ef/comments",
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
            "url": "https://api.github.com/repos/hiphops-io/integration-test/commits/401f0414b010baf45922f0d4253d0070ee27d542",
            "html_url": "https://github.com/hiphops-io/integration-test/commit/401f0414b010baf45922f0d4253d0070ee27d542"
          }
        ]
      }
    ]
  }
}
```

---

## Task: `fetch_repo_prs`

Fetches details about the PRs in a repo, placing the data in `vars.repo_prs` for use by other tasks.

```yaml
tasks:
  - name: github.fetch_repo_prs
    input:
      repo: backend # String - The name of the repository to fetch PRs from
      state: # (Optional) string - either "open", "closed", or "all" to filter by state. Default: "all"
      head: # (Optional) string - filter pulls by head user or head organization and branch name in the format of "user:ref-name" or "organization:ref-name". For example: "github:new-script-format" or "octocat:test-branch".
      base: # (Optional) string - filter pulls by base branch name. Example: gh-pages
      sort: # (Optional) string - what to sort results by. Example: "popularity" will sort by number of comments. "long-running" will sort by date created and will limit to PRs open for more than a month with activity in the past month. One of: "created", "updated", "popularity", "long-running". Default: "created"
      direction: # (Optional) string - direction of sort. Default: "desc" when sort is "created" or sort is not specified, otherwise "asc".
      per_page: # (Optional) integer - number of results per page (max 100). Default: 100
      page: # (Optional) integer - page number of results to fetch. Default: 1
```

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

If successful it will respond with a `vars` object, that has &mdash; against the key `repo_prs` &mdash; an array of data about the repo's PRs matching the output of [Github's list pull requests endpoint](https://docs.github.com/en/rest/pulls/pulls?apiVersion=2022-11-28#list-pull-requests).

Example:

```json
{
  "vars": {
    "repo_prs": [
      {
        "url": "https://api.github.com/repos/hiphops-io/integration-test/pulls/346",
        "id": 1143068882,
        "node_id": "PR_kwDOIZTUWs5EIdjS",
        "html_url": "https://github.com/hiphops-io/integration-test/pull/346",
        "diff_url": "https://github.com/hiphops-io/integration-test/pull/346.diff",
        "patch_url": "https://github.com/hiphops-io/integration-test/pull/346.patch",
        "issue_url": "https://api.github.com/repos/hiphops-io/integration-test/issues/346",
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
        "commits_url": "https://api.github.com/repos/hiphops-io/integration-test/pulls/346/commits",
        "review_comments_url": "https://api.github.com/repos/hiphops-io/integration-test/pulls/346/comments",
        "review_comment_url": "https://api.github.com/repos/hiphops-io/integration-test/pulls/comments{/number}",
        "comments_url": "https://api.github.com/repos/hiphops-io/integration-test/issues/346/comments",
        "statuses_url": "https://api.github.com/repos/hiphops-io/integration-test/statuses/6e7882fd90b7e005eac0ff02a5f1aca85b89b528",
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
            "name": "integration-test",
            "full_name": "hiphops-io/integration-test",
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
            "html_url": "https://github.com/hiphops-io/integration-test",
            "description": null,
            "fork": false,
            "url": "https://api.github.com/repos/hiphops-io/integration-test",
            "forks_url": "https://api.github.com/repos/hiphops-io/integration-test/forks",
            "keys_url": "https://api.github.com/repos/hiphops-io/integration-test/keys{/key_id}",
            "collaborators_url": "https://api.github.com/repos/hiphops-io/integration-test/collaborators{/collaborator}",
            "teams_url": "https://api.github.com/repos/hiphops-io/integration-test/teams",
            "hooks_url": "https://api.github.com/repos/hiphops-io/integration-test/hooks",
            "issue_events_url": "https://api.github.com/repos/hiphops-io/integration-test/issues/events{/number}",
            "events_url": "https://api.github.com/repos/hiphops-io/integration-test/events",
            "assignees_url": "https://api.github.com/repos/hiphops-io/integration-test/assignees{/user}",
            "branches_url": "https://api.github.com/repos/hiphops-io/integration-test/branches{/branch}",
            "tags_url": "https://api.github.com/repos/hiphops-io/integration-test/tags",
            "blobs_url": "https://api.github.com/repos/hiphops-io/integration-test/git/blobs{/sha}",
            "git_tags_url": "https://api.github.com/repos/hiphops-io/integration-test/git/tags{/sha}",
            "git_refs_url": "https://api.github.com/repos/hiphops-io/integration-test/git/refs{/sha}",
            "trees_url": "https://api.github.com/repos/hiphops-io/integration-test/git/trees{/sha}",
            "statuses_url": "https://api.github.com/repos/hiphops-io/integration-test/statuses/{sha}",
            "languages_url": "https://api.github.com/repos/hiphops-io/integration-test/languages",
            "stargazers_url": "https://api.github.com/repos/hiphops-io/integration-test/stargazers",
            "contributors_url": "https://api.github.com/repos/hiphops-io/integration-test/contributors",
            "subscribers_url": "https://api.github.com/repos/hiphops-io/integration-test/subscribers",
            "subscription_url": "https://api.github.com/repos/hiphops-io/integration-test/subscription",
            "commits_url": "https://api.github.com/repos/hiphops-io/integration-test/commits{/sha}",
            "git_commits_url": "https://api.github.com/repos/hiphops-io/integration-test/git/commits{/sha}",
            "comments_url": "https://api.github.com/repos/hiphops-io/integration-test/comments{/number}",
            "issue_comment_url": "https://api.github.com/repos/hiphops-io/integration-test/issues/comments{/number}",
            "contents_url": "https://api.github.com/repos/hiphops-io/integration-test/contents/{+path}",
            "compare_url": "https://api.github.com/repos/hiphops-io/integration-test/compare/{base}...{head}",
            "merges_url": "https://api.github.com/repos/hiphops-io/integration-test/merges",
            "archive_url": "https://api.github.com/repos/hiphops-io/integration-test/{archive_format}{/ref}",
            "downloads_url": "https://api.github.com/repos/hiphops-io/integration-test/downloads",
            "issues_url": "https://api.github.com/repos/hiphops-io/integration-test/issues{/number}",
            "pulls_url": "https://api.github.com/repos/hiphops-io/integration-test/pulls{/number}",
            "milestones_url": "https://api.github.com/repos/hiphops-io/integration-test/milestones{/number}",
            "notifications_url": "https://api.github.com/repos/hiphops-io/integration-test/notifications{?since,all,participating}",
            "labels_url": "https://api.github.com/repos/hiphops-io/integration-test/labels{/name}",
            "releases_url": "https://api.github.com/repos/hiphops-io/integration-test/releases{/id}",
            "deployments_url": "https://api.github.com/repos/hiphops-io/integration-test/deployments",
            "created_at": "2022-11-08T14:37:21Z",
            "updated_at": "2022-11-08T14:37:21Z",
            "pushed_at": "2022-12-02T10:58:05Z",
            "git_url": "git://github.com/hiphops-io/integration-test.git",
            "ssh_url": "git@github.com:hiphops-io/integration-test.git",
            "clone_url": "https://github.com/hiphops-io/integration-test.git",
            "svn_url": "https://github.com/hiphops-io/integration-test",
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
            "name": "integration-test",
            "full_name": "hiphops-io/integration-test",
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
            "html_url": "https://github.com/hiphops-io/integration-test",
            "description": null,
            "fork": false,
            "url": "https://api.github.com/repos/hiphops-io/integration-test",
            "forks_url": "https://api.github.com/repos/hiphops-io/integration-test/forks",
            "keys_url": "https://api.github.com/repos/hiphops-io/integration-test/keys{/key_id}",
            "collaborators_url": "https://api.github.com/repos/hiphops-io/integration-test/collaborators{/collaborator}",
            "teams_url": "https://api.github.com/repos/hiphops-io/integration-test/teams",
            "hooks_url": "https://api.github.com/repos/hiphops-io/integration-test/hooks",
            "issue_events_url": "https://api.github.com/repos/hiphops-io/integration-test/issues/events{/number}",
            "events_url": "https://api.github.com/repos/hiphops-io/integration-test/events",
            "assignees_url": "https://api.github.com/repos/hiphops-io/integration-test/assignees{/user}",
            "branches_url": "https://api.github.com/repos/hiphops-io/integration-test/branches{/branch}",
            "tags_url": "https://api.github.com/repos/hiphops-io/integration-test/tags",
            "blobs_url": "https://api.github.com/repos/hiphops-io/integration-test/git/blobs{/sha}",
            "git_tags_url": "https://api.github.com/repos/hiphops-io/integration-test/git/tags{/sha}",
            "git_refs_url": "https://api.github.com/repos/hiphops-io/integration-test/git/refs{/sha}",
            "trees_url": "https://api.github.com/repos/hiphops-io/integration-test/git/trees{/sha}",
            "statuses_url": "https://api.github.com/repos/hiphops-io/integration-test/statuses/{sha}",
            "languages_url": "https://api.github.com/repos/hiphops-io/integration-test/languages",
            "stargazers_url": "https://api.github.com/repos/hiphops-io/integration-test/stargazers",
            "contributors_url": "https://api.github.com/repos/hiphops-io/integration-test/contributors",
            "subscribers_url": "https://api.github.com/repos/hiphops-io/integration-test/subscribers",
            "subscription_url": "https://api.github.com/repos/hiphops-io/integration-test/subscription",
            "commits_url": "https://api.github.com/repos/hiphops-io/integration-test/commits{/sha}",
            "git_commits_url": "https://api.github.com/repos/hiphops-io/integration-test/git/commits{/sha}",
            "comments_url": "https://api.github.com/repos/hiphops-io/integration-test/comments{/number}",
            "issue_comment_url": "https://api.github.com/repos/hiphops-io/integration-test/issues/comments{/number}",
            "contents_url": "https://api.github.com/repos/hiphops-io/integration-test/contents/{+path}",
            "compare_url": "https://api.github.com/repos/hiphops-io/integration-test/compare/{base}...{head}",
            "merges_url": "https://api.github.com/repos/hiphops-io/integration-test/merges",
            "archive_url": "https://api.github.com/repos/hiphops-io/integration-test/{archive_format}{/ref}",
            "downloads_url": "https://api.github.com/repos/hiphops-io/integration-test/downloads",
            "issues_url": "https://api.github.com/repos/hiphops-io/integration-test/issues{/number}",
            "pulls_url": "https://api.github.com/repos/hiphops-io/integration-test/pulls{/number}",
            "milestones_url": "https://api.github.com/repos/hiphops-io/integration-test/milestones{/number}",
            "notifications_url": "https://api.github.com/repos/hiphops-io/integration-test/notifications{?since,all,participating}",
            "labels_url": "https://api.github.com/repos/hiphops-io/integration-test/labels{/name}",
            "releases_url": "https://api.github.com/repos/hiphops-io/integration-test/releases{/id}",
            "deployments_url": "https://api.github.com/repos/hiphops-io/integration-test/deployments",
            "created_at": "2022-11-08T14:37:21Z",
            "updated_at": "2022-11-08T14:37:21Z",
            "pushed_at": "2022-12-02T10:58:05Z",
            "git_url": "git://github.com/hiphops-io/integration-test.git",
            "ssh_url": "git@github.com:hiphops-io/integration-test.git",
            "clone_url": "https://github.com/hiphops-io/integration-test.git",
            "svn_url": "https://github.com/hiphops-io/integration-test",
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
            "href": "https://api.github.com/repos/hiphops-io/integration-test/pulls/346"
          },
          "html": {
            "href": "https://github.com/hiphops-io/integration-test/pull/346"
          },
          "issue": {
            "href": "https://api.github.com/repos/hiphops-io/integration-test/issues/346"
          },
          "comments": {
            "href": "https://api.github.com/repos/hiphops-io/integration-test/issues/346/comments"
          },
          "review_comments": {
            "href": "https://api.github.com/repos/hiphops-io/integration-test/pulls/346/comments"
          },
          "review_comment": {
            "href": "https://api.github.com/repos/hiphops-io/integration-test/pulls/comments{/number}"
          },
          "commits": {
            "href": "https://api.github.com/repos/hiphops-io/integration-test/pulls/346/commits"
          },
          "statuses": {
            "href": "https://api.github.com/repos/hiphops-io/integration-test/statuses/6e7882fd90b7e005eac0ff02a5f1aca85b89b528"
          }
        },
        "author_association": "CONTRIBUTOR",
        "auto_merge": null,
        "active_lock_reason": null
      }
    ]
  }
}
```

---

## Task: `create_repo_webhook`

Creates a webhook for a particular repo in Github. Documentation for this (especially the events input) can be found [here](https://docs.github.com/en/rest/reference/repos#create-a-repository-webhook).

```yaml
tasks:
  - name: github.github.create_repo_webhook # String - The name of the task
    input:
      repo: "integration-test" # String - repository to create the release in
      url: "https://hiphops.io/github_events" # String - The URL to which the events in the hook are delivered.
      events: # Array - Determines what events the hook is triggered for. Set "*" to receive all events. Default: ['push']
        - push
        - pull_request
      secret: "importantsecret" # String - The shared secret between Github and Hiphops which is used to validate the payload.
```

---

## Task: `create_org_webhook`

Creates a webhook for your organization in Github. Documentation for this (especially the events input) can be found [here](https://docs.github.com/en/rest/orgs/webhooks?apiVersion=2022-11-28#create-an-organization-webhook).

```yaml
tasks:
  - name: github.github.create_org_webhook # String - The name of the task
    input:
      url: "https://hiphops.io/github_org_events" # String - The URL to which the events in the hook are delivered.
      events: # Array - Determines what events the hook is triggered for. Set "*" to receive all events. Default: ['push']
        - push
        - pull_request
      secret: "importantsecret" # String - The shared secret between Github and Hiphops which is used to validate the payload.
```

---

## Task: `dispatch_workflow`

Dispatches a workflow for a given repository.

```yaml
tasks:
- name: github.dispatch_workflow
  input:
    repo: backend # String - The name of the repository the workflow to dispatch is in
    workflow: hello-world.yaml # String or Number - Either the name of the yaml file for the workflow to dispatch, as a string, or the ID of the workflow, as a number
    branch_or_tag: main # String - The name of either a branch or tag to run the workflow against
    inputs:
      foo: bar # (Optional) object, with simple string key-value pairs - The inputs to provide the workflow run with
```

###### Responds with

Only provides standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).


---

## Task: `fetch_workflow_run_logs`

Fetches the logs for a workflow run (e.g the logs of a check suite).
Typically you would run this against a workflow_run event, and get the workflow_run_id input value from the event.workflow_run.id property.

Rather than returning the logs directly, they will be saved in a temporary storage location which can be fed to other tasks such as [`releasemanager.save_material`](integrations/hiphops-releasemanager.md#task-save_material). This storage location is returned in the task's `vars` object.

```yaml
tasks:
- name: github.fetch_workflow_run_logs
  id: workflow_logs
  input:
    repo: backend # String - The name of the repository the workflow run was in
    workflow_run_id: 12345 # Number - The ID of the workflow run, as a number
```

###### Example sensor:

<details>
<summary>Saving workflow run completions (including logs) sensor</summary>

[Saving workflow run completions (including logs) sensor](../_sample_sensors/workflow_run_save_material.yaml ':include')

</details>

###### Responds with

Provides the standard task outputs (`SUCCESS`, `FAILURE`, `result` or `error_message`).

If successful the returned `vars` object will have an object containing the key `log_file_location`, the value of which will be the temporary storage location the logs were saved to. 

Example:

Assumes that the task ID was set as `workflow_logs`.

```js
{
  "vars": {
    "workflow_logs": {
      "log_file_location": "github.fetch_workflow_run_logs/4544497015.zip"
    }
  }
}
```
