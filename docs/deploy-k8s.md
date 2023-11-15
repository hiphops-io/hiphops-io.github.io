# Deploying Hiphops to Kubernetes

## Simple deploy

Hiphops can be deployed as a container, and for users on Kubernetes we provide a kustomize resource that can be imported and overlayed to get going quickly.

The kustomize resource can be included like so:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- https://github.com/hiphops-io/hops.git//deploy/kustomize/
```

for Hiphops to be fully functional, you'll need to provide a configmap containing your hops configs along with a secret containing your hiphops.io account key.

We have an example of a fully functional kubernetes config with all of this setup in the [hops repo, here](https://github.com/hiphops-io/hops/tree/main/deploy/k8s_example)

Copy the above and edit the key, namespace (if required) and `main.hops`, run `kubectl apply -k path/to/your/folder` and you'll have a fully working hiphops deployment. **Nice.**

### Using the console

By default, hiphops is unauthenticated. As such the base configs do not expose the console outside of your cluster.
You can access the running service UI via `kubectl port-forward`

- Run `kubectl port-forward -n hiphops service/hiphops 8916:8916`
- Visit the console in your browser [http://0.0.0.0:8916/console]

## With auth

Given the nature of Hiphops and the fact most teams have some form of internal auth solution in place, Hiphops delegates auth concerns to the user.

The easiest way of adding auth to Hiphops in Kubernetes is to use an auth proxy, such as [OAuth2 Proxy](https://oauth2-proxy.github.io/oauth2-proxy/).
Setting up OAuth2 Proxy is fairly straightforward, and most of the config will be user specific (for example the GitHub tokens you wish to use, if you wanted to add GitHub login for org members only)

In future, we'll add context variables into tasks and pipelines that expose details about the user that triggered a pipeline, such that you can make authorization decisions before triggering.


## Monitoring

With the exception of debug mode, Hiphops logs are structured JSON. You should be able to ship them to most log aggregators without extra config, enabling you to search and monitor Hiphops activity.

[Elasticsearch](https://www.elastic.co/elasticsearch) and [Loki](https://grafana.com/oss/loki/) are two popular options.
