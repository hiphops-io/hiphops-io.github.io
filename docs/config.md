# Configuring Hiphops

## How to configure

Hiphops can be configured by placing a file named `hiphops.yaml` in the root of your repository.

> Note the extension must be `.yaml` and not `.yml` or variations thereof

For every PR, we will fetch the `hiphops.yaml` found in the head of target branch (A.K.A the `base` branch) and apply it to the incoming change.
If no file can be found, the default pipeline is one which matches all changes and runs analysis on the change, but takes no further action.

## Simple example

```yaml
version: beta
release_pipelines:
  - name: "Analyse production and staging changes"
    when:
      branch: ["production", "staging"]
```

The above config would configure change analysis (along with PR commenting) to run on pull requests against branches `production` and `staging`.
