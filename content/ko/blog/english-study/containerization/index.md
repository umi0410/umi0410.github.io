---
title: "Containerization explained - IBM Technology"
date: 2022-03-29T20:00:00+09:00
chapter: false
pre: "<b></b>"
weight: 33
noDisqus: false
draft: true
categories:
  - english-study
  - infrastructure
  - container
  - docker
cover:
  image: preview.png
description: |
  IBM Technology에서 Containerization(컨테이너화)에 관해 다룬 영상을 번역해봤어요.
---
*영상 링크: [https://youtu.be/0qotVMX-J5s](https://youtu.be/0qotVMX-J5s)*

*본 게시글은 개발 공부 겸 영어 공부를 하고자 작성되었으며, 잘못된 해석이 존재할 수 있습니다. 잘못된 부분을 발견하신 분이 계시다면 피드백해주시면 감사하겠습니다.*

*편의상 구어체 + 제 마음대로 막 번역합니다.*

# tl;dr

# 번역 내용

Hi everyone, my name is Sai Vennam,

안녕하세요 여러분 저는 Sai Vennam이에요.

and I'm a developer advocate with IBM.

Today, I want to talk about containerization.

Whenever I mention containers,

most people tend to default to something like Docker

- or even Kubernetes these days,

but container technology has actually been around for quite some time.

It was actually back in 2008

that the Linux kernel introduced C-groups, or "control groups",

which paved the way for all the different container technologies we see today.

So, that includes Docker, but also things like Cloud Foundry,

as well as Rocket and other container runtimes out there.

Let's get started with an example.

We'll say that I'm a developer,

and I've created a Node.js application,

and I want to push it into production.

We'll take 2 different form factors to kind of explain

the advantages of containerization.

First, we'll talk about VMs

and then we'll talk about containers.

So, first things first,

let's introduce some of the things that we've got here.

So, we've got the hardware itself,

which is the big box.

We've got the host operating system,

as well as a hypervisor.

The hypervisor is actually what allows us to spin up VMs.

Let's take a look at the shared pool of resources

with the host OS and hypervisor.

We can assume that some of these resources have already been consumed.

Next, let's go ahead and take this .js application and push it in.

And to do that, I need a Linux VM.

So, let's go ahead and sketch out that Linux VM.

And in this VM, there's a few things to note here.

So, we've got another operating system in addition to the host OS

it's going to be the guest OS.

As well as some binaries and libraries.

So, that's one of the things about Linux VMs:

even though we're working with the really lightweight application,

to create that Linux VM,

we have to put that guest OS in there and a set of binaries and libraries.

And so, that really bloats it out.

In fact, I think the smallest Node.js VM that I've seen out there is over 400 MB.

Whereas the Node.js runtime app itself would be under 15.

So, we've got that,

and we go ahead and let's push that .js application into it,

and just by doing that alone,

we're going to consume a set of resources.

Next, let's think about scaling this out.

So, we'll create 2 additional copies of it.

And you'll notice that even though it's the exact same application,

we have to use and deploy that separate guest OS and libraries every time.

And so, we'll do that 3 times.

By doing that,

essentially, we can assume that for this particular hardware,

we've consumed all of the resources.

There's another thing that I haven't mentioned here:

this .js application, I developed on my MacBook.

So, when I pushed it into production to get it going in the VM,

I noticed that there were some issues and incompatibilities.

This is the kind of foundation for a big "he said, she said" issue,

where things might be working on your local machine

and work great, but when you try to push it into production,

things start to break.

This really gets in the way of doing Agile DevOps and continuous integration and delivery.

That's solved when you use something like containers.

There's a 3-step process when doing anything container-related

and pushing or creating containers.

And, it almost always starts with, first, some sort of a manifest.

So, something that describes the container itself.

So, in the Docker world, this would be something like a Dockerfile,

and in Cloud Foundry, this would be a manifest YAML.

Next, what you'll do is create the actual image itself.

So, for the image,

if you're working with something like Docker,

that would be a Docker image.

If you're working with Rocket, it would be an ACI (or, "Application Container Image").

So, regardless of the different containerization technologies,

this process stays the same.

The last thing you end up with is an actual container itself,

which contains all of the runtimes and libraries and binaries needed

to run an application.

That application runs on a very similar setup to the VMs,

but what  we've got on this side is,

again, a host operating system.

The difference here is, instead of a hypervisor,

we're going to have something like a runtime engine.

So, if you're using Docker, this would be the Docker Engine,

and, you know, different containerization  technologies would have a different engine.

Regardless, it's something that runs those containers.

Again, we've got this shared pool of resources;

so we can assume that that alone consumes some set of resources.

Next, let's think about actually containerizing this technology.

So, we talked about the 3-step process:

we create a Dockerfile,

we build out the image,

we push it to a registry, and we have our container,

and we can start pushing this out as containers.

The great thing is, these going to be much more lightweight.

So, deploying out multiple containers

- since you don't have to worry about a guest OS this time,

you really just have the libraries as well as the application itself.

So, we scale that out 3 times,

and because we don't have to duplicate all of those

operating system dependencies and create bloated VMs,

we actually will use less resources.

So, let's use a different color here.

And, scaling that out 3 times,

we still have a good amount of resources left.

Next, let's say that my coworker decides,

"Hey for this .js application, let's take advantage of a third-party,

let's say, a cognitive API to do something

like image recognition."

So, let's say that we've got our third-party service,

and we want to access that using maybe a Python application.

So, he's created that service that accesses the third-party APIs,

and with our Node.js application, we want to access that Python application

to then access that service.

If we wanted to do this in VMs, I'm really tempted to basically create a VM

out of both the .js application

and the Python application

because that would allow me to continue to use the VMs that I have.

But that's not truly cloud-native, right?

Because if I wanted to scale out the .js but not the Python app,

I wouldn't be able to if they're running in the same VM.

So, to do it in a truly cloud-native way,

I would have to free up some of these resources

- basically, get rid of one of these VMs,

and then deploy the Python application in it instead.

And, that's not ideal.

But with the container-based approach, what we can do is simply say,

since we're modular, we can say,

"OK, just deploy one copy of the Python application".

So, we'll go ahead and do that in a different color here.

And that consumes a little bit more resources.

Then, with those remaining resources,

the great thing about container technology,

that actually becomes shared between all the processes running.

In fact, another advantage:

if these container processes aren't actually utilizing the CPU or memory,

all of those shared resources become accessible

for the other containers running within that hardware.

So, with container-based technology,

we can truly take advantage of cloud-native-based architectures.

So, we talked about things like portability of the containers;

talked about how it's easier to scale them out;

and then, overall, with this three-step process and the way we push containers,

it allows for more Agile DevOps and continuous integration and delivery.

Thanks for tuning in for this broad overview of container-based technology.

As always, we're looking for feedback so definitely drop a comment below,

and be sure to subscribe to stay tuned for more videos in the future.

Thank you.

