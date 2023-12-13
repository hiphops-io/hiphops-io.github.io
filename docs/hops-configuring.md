# Configuring Hiphops Pipelines

## File format

Hiphops pipelines are configured in hops files (extension `.hops`). As hops files are based on HCL, configuring the `.hops` extension to use `.hcl` syntax highlighting will provide a good dev experience.

> A .hops specific language server and set of IDE plugins are in the works, but `HCL` plugins work well in the meantime.


## Config location

Best practice is to split your automations across multiple automation bundles. This will make your automations more manageable.

An automation bundle consists of a sub directory under your main hops config directory. For example, if your main hops config directory is `/home/user/hops`, then your automation bundles should look like the following examples: `/home/user/hops/standup_alert` or `home/user/hops/deploy_cluster`.

Each bundle can only hold a single `.hops` file but as many additional files as needed. When `hops` starts, all bundles are loaded together.

By default, hiphops will search for bundles in `~/.hops`.<br>
This can be configured with the flag `--hops` which accepts a directory. Automations are loaded only from automation bundles under the `--hops` directory.

> Note: File/directory names beginning with `..` will be ignored during search. This is to enable support for Kubernetes configmaps and their symlink layout.

## Best practices

### Code layout

Generally, it is easier to maintain hops configs when they're broken up into files relating to specific tasks/areas of concern, rather than splitting by event type, for example.

In particular, grouping tasks in a file with the pipelines they trigger is better than splitting these closely linked configs across multiple files. If they change, they'll often change together.


### Names and naming

By convention, hops directories and files should be lower snake case `like_this/like_this.hops` and named to represent the value they contribute e.g. `pr_approvals/pr_approvals.hops` or `release/release.hops`.

It's worth avoiding junk words such as `automate`, `workflow`, or `pipelines` in your filenames, as they could be applied to any and all hops configs.

Within your configs, note that whilst the `name` attribute is optional on all resources, using it provides helpful context when debugging or looking at logs.
