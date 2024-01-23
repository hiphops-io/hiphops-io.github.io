# What is Hiphops?

**Hiphops is an open-source, highly extensible automation platform for tech teams.**

It provides a powerful yet simple config language to define your pipelines, along with the custom UIs with which to trigger them.<br>

Automate and orchestrate your SaaS tools, git repos and manual processes in a single platform. Out of the box SaaS integrations are paired with custom code execution, a powerful workflow syntax and configurable UIs.


---

## Features

#### Control, orchestrate and automate your SaaS products 

Out of the box, credential-free access to your tools means you have automations across your entire process in a few lines of config.


#### Develop and test locally 👩‍💻

Hiphops ships as a single binary that can run almost anywhere. Write and immediately run pipelines locally, replay old events against new code, ship when you're confident.


#### Configure in code 𝍌

UI driven workflow builders or YAML configs are fine for simple pipelines, but become unwieldy and difficult to maintain as complexity grows.
On the other hand, defining automations in full-fledged programming languages is complex and introduces boilerplate setup that isn't useful for most business/process automation work.

Hiphops is configured via an easy to learn syntax that allows you to define precise behaviour without fluff. It's a great balance of control vs simplicity.

> For the curious, our syntax is a simplified HCL - the same language used by products such as Terraform and Docker Bake. HCL provides a good mix of expressions, built in functions and a low barrier to entry.


#### Custom UIs for manually triggered automations

Automations are great, but having the ability to trigger them via a UI is sublime.

Wrap up a multi-step process in a Hiphops automation and have a UI to trigger it in a few lines of config. Perfect for adhoc back office tasks.


#### Extensible ➕

A closed ecosystem that solves 90% of your use case often ends up being a false economy. Contorting the tool or your process around arbitrary limitations eats up any early savings. These additional needs often surface after a significant time and money investment in a stack.

Hiphops is fully extensible. You're able to run custom code in containerised steps or create your own integrations from scratch. You only need to code those parts that are custom to you, and your custom code works seamlessly as a first class citizen within the context of Hiphops pipelines.


#### Super fast 🏎

Hiphops' running model means automations are pre-process and ready to run. This means Hiphops can begin work in sub-millisecond times, vs tens of seconds for most workflow tools. This is great for local development feedback, and also means your pipelines feel snappy and responsive to end users.


#### Open Source ☀️

Our automation engine & CLI [hops](https://github.com/hiphops-io/hops) is fully open source under the permissive `Apache 2.0` license.

Whilst Hiphops is most powerful when paired with an [Hiphops.io](https://www.hiphops.io) account, it is possible to use hops without an account. All you need is a [NATS](https://nats.io/) cluster and to create whatever apps your pipelines need.

> Note: Container execution apps (e.g. `k8s`) come packaged with hops already, so pure container workflows only need a NATS setup + hops.


#### Infinitely long pipelines ♾

Hiphops is fully event driven, meaning you only use compute when there's work to be done. If a pipeline waits for input before continuing, then it won't be using up resource/blocking other pipelines in the meantime. This contrasts with running automations on something like a CI/CD, which generally holds a process open for the duration of the run.


#### Bring your own auth, secrets, etc 🔐

Hiphops attempts to make the fewest assumptions about your security setup as possible. The example configs show how to set up hops behind an auth proxy, allowing you to hook into your own auth source of truth.

Most teams have a full featured secret store in place already via their cloud provider or an internal solution such as Vault. Automation platforms that force you to duplicate secrets into their home-grown (and usually weaker) solution are a nuisance. Because Hiphops is self hosted, you're able to deliver secrets into it via your existing, secure mechanisms.

---

## How it works

You self-host the Hiphops engine, [hops](https://github.com/hiphops-io/hops) - which you deploy alongside your automations. It's simple to run, with zero additional dependencies. It can be deployed as a docker container, binary, or using our example Kubernetes manifests.

Hops connects with your Hiphops.io account, where we provide pre-built apps to connect with SaaS providers and common helpers. Hiphops.io also provides the messaging, storage, and the other nuts and bolts that makes Hiphops powerful.

Hops itself provides workflow orchestration, expression parsing, idempotency and generally drives the logic and execution of your pipelines.

Everything in Hiphops is communicated via events/messages on a stream. Calls in your pipelines, source events from tasks or third parties, results from your calls. All of it is sent via messages with idempotency safeguards in place.

The actual work Hiphops carries out is handled by workers, and incoming source events are handled by listeners. Hiphops apps can be workers and/or listeners depending on what they integrate with.

[Hiphops.io](https://www.hiphops.io) provides a library of pre-built and hosted apps for use with Hiphops

---

> <small>Editor note:</small>
>
> <small>Yeah, I know. Why is the Open Source emoji a sun? I ran out of ideas here. Shiny sun is nice, Open Source is nice? That's all I got.</small>
