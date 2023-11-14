# Kubernetes

_The Hiphops Kubernetes app enables you to run custom containers as part of your pipelines, within your own Kubernetes cluster_

|Name|Listener|Worker|Setup|Auth|
|:---|:-------|:-----|:----|:---|
|`k8s`|:hourglass: Coming soon|:white_check_mark:&nbsp;&nbsp;&nbsp;Yes|Run the k8s worker, detailed instructions below|K8s RBAC via `kubeconfig`|


## Setup instructions

For most users that already use Kubernetes regularly, Hiphops will 'just work'.

The Kubernetes app is bundled with hops. It can be ran separately, but if you run
`hops start server` it'll kick off along with all the other components and handle `k8s` calls. This is the easiest way to get going.

The app will look in the default locations (i.e. `~/.kube/config`) for your Kubernetes config file and use the current context to connect to your cluster.

Inside a cluster, it will attempt to use cluster based credentials. For most users this means zero auth config is required.

### Automatic port-forwarding for local runs

If you're running hops locally against a remote cluster, then you'll need to be able to access running pods to get results.

Hiphops handles this for you automatically, you just need to tell it to create ad-hoc port-forwards, like so:

`hops start server --port-forward`

You can also add `port-forward: true` to a local dev Hiphops config to avoid having to set this everytime you run hops.

### Configuring a non-default location for ~/.kube/config 

If your kubeconfig is in a non-default location or if you wish to access one Kubernetes cluster from inside a different Kubernetes cluster, you can pass it through as an argument to hops.

e.g.

`hops start server --kubeconfig=/path/to/kube/config`


---

## Call: `run`

Runs a container image as a pod and fetches the result.

Useful for running custom code or leveraging containers on Docker hub that solve for common tasks.

If a valid `json` output file is produced by your script, it will be returned as an object in the result. Otherwise it will be a string under `body`.

**Call structure:**

```hcl
call k8s_run {
  inputs = {
    image = "bash:5-alpine3.18" // String - The name of the image to run. Your cluster must have access to pull this image. It is best practice to pin to a specific tag
    command = [ // (Optional) string array - Same as `command` in Kubernetes or `ENTRYPOINT` in Docker
      "bash", "-c", "echo '{\"greeting\": \"Hello world!\"}' > /output/result.json"
    ]
    args = [] // (Optional) string array - Same as `args` in Kubernetes or CMD in Docker
    namespace = "" // (Optional) string - Namespace to run the pod in. Defaults to `default` namespace
    in_dir = "" // (Optional) string - absolute path to the dir in which to mount input files. Defaults to "/input"
    in_files = { // (Optional) key/value object (strings for both). Keys represent file names relative to `in_dir` and values represent their content. Will be mounted as files in the filesystem accordingly
      "foo.txt" = "bar"
    }
    out_dir = "" // (Optional) string - absolute path to the dir which will contain results. Defaults to "/output"
    out_path = "" // (Optional) string - path to a result file relative to `out_dir`. Contents upon completion will be returned within the result. Defaults to `result.json`
    cpu = "100m" // Set the CPU limit & request for the created pod
    memory = "500Mi" // Set the memory limit & request for the created pod
    skip_cleanup = false // (Optional) boolean - defaults to false. Set to true to skip deletion of old pods. Useful for debugging, though be careful that it isn't left as `false` in live workloads
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
  "body": "", // Will be populated with pod out_path file contents if they're a plain string
  "json": { "greeting": "Hello World!" } // Will be populated with pod out_path file contents if valid JSON
}
```

---
