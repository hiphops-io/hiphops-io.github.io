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
Defining flows in your native language can be powerful, but it's unlikely that the first choice of language is the same for everyone in an org. They also tend to force much more boilerplate.

Hiphops is configured via HCL - the same language used by products such as Terraform and Docker Bake. HCL provides a good mix of expressions, built in functions and a low barrier to entry with a config-like syntax.

Deploy Hiphops configs as you would any other code, so it fits into your existing change flow.


#### Interactive and accessible 👌

Automations are only as valuable as they are accessible. Automations that require technical users to drive them or babysite them effectively force technical team members to spend time on basic admin. They also provide a poor experience to non-technical teams (such as HR) that may want to self-serve for things such as user onboarding.

Whilst Hiphops pipelines can run in response to events (in the same way as your CI/CD), you can also configure custom UIs that are accessible to all team members. 


#### Extensible ➕

A closed ecosystem that solves 90% of your use case often ends up being a false economy. Contorting the tool or your process around arbitrary limitations eats up any early savings. These additional needs often surface after a significant time and money investment in a stack.

Hiphops is fully extensible. You're able to run custom code in containerised steps or create your own integrations from scratch. You only need to code those parts that are custom to you, and your custom code works seamlessly as a first class citizen within the context of Hiphops pipelines.

#### Super fast 🏎

Hiphops running model means pipelines aren't constructed on the fly, and instances are already in place to handle work. This means Hiphops can being to execute work in sub-millisecond times, vs tens of seconds for most workflow tools. This is great for local development feedback, and also means UIs in front of your pipelines feel snappy and responsive to end users.

#### Open Source ☀️

Our automation engine & CLI [hops](https://github.com/hiphops-io/hops) is fully open source under the permissive `Apache 2.0` license.

Whilst Hiphops is most powerful when paired with an [Hiphops.io](https://www.hiphops.io) account, it is possible to use hops without an account. All you need is a [NATS](https://nats.io/) cluster and to create whatever apps your pipelines need.

> Note: Container execution apps (e.g. `k8s`) come packaged with hops already, so pure container workflows only need a NATS setup + hops.


#### Infinitely long pipelines ♾

Hiphops is fully event driven, meaning you only use compute when there's work to be done. If a pipeline waits for input before continuing, that input can take as long as you like. This contrasts with running automations on something like a CI/CD, which generally holds a process open for the duration of the run.


#### Bring your own auth, secrets, etc 🔐

Hiphops attempts to make the fewest assumptions about your security setup as possible. The example configs show how to set up hops behind an auth proxy, allowing you to hook into your own auth source of truth.

Most teams have a full featured secret store in place already via their cloud provider or an internal solution such as Vault. Automation platforms that force you to duplicate secrets into their home-grown (and usually weaker) solution are a nuisance. Because Hiphops is self hosted, you're able to deliver secrets into it via your existing, secure mechanisms.

---

## How it works

You self-host the Hiphops engine, [hops](https://github.com/hiphops-io/hops) - which you deploy with your configs. It's simple to run, with zero additional dependencies. It can be deployed as a docker container, binary, or using our example Kubernetes manifests.

Hops connects with your Hiphops.io account, where we provide pre-built apps to connect with SaaS providers and common helpers. Hiphops.io also provides the messaging, storage, and the other nuts and bolts that makes Hiphops powerful.

Hops itself provides workflow orchestration, expression parsing, idempotency and generally drives the logic and execution of your pipelines.

Everything in Hiphops is communicated via events/messages on a stream. Calls in your pipelines, source events from tasks or third parties, results from your calls. All of it is sent via messages with idempotency safeguards in place.

The actual work Hiphops carries out is handled by workers, and incoming source events are handled by listeners. Hiphops apps can be workers and/or listeners depending on what they integrate with.

[Hiphops.io](https://www.hiphops.io) provides a library of pre-built and hosted apps for use with Hiphops

---

> <small>Editor note:</small>
>
> <small>Yeah, I know. Why is the Open Source emoji a sun? I ran out of ideas here. Shiny sun is nice, Open Source is nice? That's all I got.</small>
