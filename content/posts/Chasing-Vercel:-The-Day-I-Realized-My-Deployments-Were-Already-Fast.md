+++
title = "Chasing Vercel: The Day I Realized My Deployments Were Already Fast"
date = 2026-06-01T18:24:43+05:30
draft = false
slug = "chasing-vercel-the-day-i-realized-my-deployments-were-already-fast"
tags = ["self-hosting", "devops", "docker", "vercel", "cloudflare", "github-actions", "ci-cd", "deployment"]
categories = ["Engineering", "Self Hosting"]
description = "After building a self-hosted deployment pipeline with Docker, Traefik, GitHub Actions, and Cloudflare, I set out to understand why Vercel still felt faster."
+++

For the past few months, I have been moving more of my projects away from managed platforms and onto infrastructure I control.

It started with a simple thought: instead of deploying every small project to a different platform, why not run my own server and host multiple production services from there?

That idea slowly turned into a proper self-hosted setup. I had Docker Compose managing services, Traefik acting as a reverse proxy, automatic SSL, GitHub Actions for CI/CD, Cloudflare for DNS and protection, and a clean deployment flow where every push could update the running application.

At some point, I looked at the whole setup and thought:

> This is basically what platforms like Vercel are doing, right?

Not exactly.

That realization became more interesting than the setup itself.

## The Problem I Noticed

My self-hosted deployments were not slow.

That is the important part.

Whenever I pushed code, GitHub Actions would run, build the project, push the Docker image, and my server would pull the latest image and restart the container. With caching and a decent Dockerfile, the whole thing felt smooth enough.

For backend services, APIs, dashboards, and internal tools, it was honestly great.

But whenever I deployed a frontend project on Vercel, the deployment felt faster.

Not just in a marketing sense. It genuinely felt faster.

I would push a small change, and before I could even think too much about it, Vercel already had the preview deployment ready. Production deployments also felt almost instant compared to my Docker-based flow.

That made me curious.

I was not thinking, "My setup is bad."

I was thinking, "What exactly is Vercel doing differently?"

## My Self-Hosted Setup

My deployment setup looked something like this:

```text
Git push
    ↓
GitHub Actions starts
    ↓
Dependencies are installed
    ↓
Docker image is built
    ↓
Image is pushed to registry
    ↓
Server pulls the new image
    ↓
Container is recreated
    ↓
Traefik routes traffic
```

This is a very common self-hosted deployment pattern.

It is clean, reliable, and flexible. It works for almost any application because Docker packages everything the app needs to run.

That is the main advantage of Docker.

The same container can run on my laptop, a VM, a cloud server, or somewhere else. The environment becomes predictable.

But that advantage comes with a cost.

Docker is not just shipping my final website output. It is shipping an image that contains the application, runtime, dependencies, filesystem layers, and everything needed to run the app.

Even when Docker layer caching is enabled, the deployment unit is still heavier than a simple build artifact.

## What I Tried To Improve

At first, I assumed my pipeline just needed more optimization.

So I started improving it step by step.

I moved the build process into GitHub Actions instead of building directly on the server. This already made the server lighter because the server no longer had to install dependencies or run production builds.

Then I started using Docker layer caching so unchanged layers could be reused between builds.

I optimized my Dockerfile with multi-stage builds so the final image contained only what was needed to run the application.

I made sure the server only had to pull the latest image and restart the service.

I also added Cloudflare in front of the server, used Traefik for domain-based routing, and kept the infrastructure clean enough to support multiple services from the same machine.

At that point, I had most of the things people usually recommend:

```text
cached builds
prebuilt Docker images
zero-downtime deploys
rollback support
preview containers
Cloudflare CDN
GitHub Actions CI/CD
```

And honestly, the result was good.

The deployments became fast.

But Vercel was still faster.

That was when I realized the problem was not only about optimization.

It was about architecture.

## The Hidden Difference

The biggest difference is this:

```text
My setup deploys Docker images.
Vercel deploys build artifacts.
```

That single difference explains a lot.

A Docker image is a package containing your application and everything needed to run it. It may include Node.js, dependencies, OS-level layers, configuration, and the built app.

A build artifact is much smaller.

For a frontend website, the final artifact might simply be:

```text
HTML files
CSS files
JavaScript bundles
images
static assets
serverless functions
edge functions
```

Vercel does not always need to ship a full container image for every deployment.

It can build the project, identify the output, upload the required artifacts, and then make that deployment available through its routing and edge infrastructure.

That is much lighter than pushing and pulling Docker image layers.

## Why Vercel Feels Instant

Vercel is not just a server where code is deployed.

It is a platform designed specifically around frontend and full-stack web deployment.

It has deep knowledge of frameworks like Next.js. It knows which parts can be static, which parts need server-side rendering, which functions can run serverlessly, and which assets can be cached aggressively.

That framework awareness matters.

My Docker setup treats the app as a generic container.

Vercel treats the app as a web deployment graph.

That means Vercel can split the application into smaller pieces:

```text
static assets
server-rendered routes
API routes
serverless functions
edge functions
cacheable outputs
```

Instead of replacing one full running container, it can prepare a new immutable deployment and then switch traffic to it.

That is why rollbacks are also fast. It does not need to rebuild everything. It can point traffic back to an older deployment.

This is very different from a traditional Docker deployment where a new image usually means pulling layers, recreating containers, and waiting for the process to become healthy.

## The Container Overhead

Even in a very optimized Docker setup, some work still has to happen.

The CI system builds the image.

The image has to be pushed to a registry.

The server has to pull the image.

Docker has to extract layers.

The old container has to be replaced or a new container has to be started.

Health checks have to pass.

Traffic then shifts to the new version.

None of this is necessarily slow.

But each step adds some amount of time.

Vercel removes or reduces many of these steps for frontend deployments because it is not always moving a full runtime environment around.

It is mostly moving the final output.

That is the theory that finally made things clear to me.

Docker is excellent when I want portability and control.

Vercel is excellent when I want optimized web deployment speed.

They are solving overlapping problems, but not in the same way.

## Could I Fully Recreate Vercel?

Technically, yes.

Practically, no.

To truly recreate Vercel, I would need to build much more than a Docker deployment pipeline.

I would need:

```text
a custom build system
framework-aware caching
preview deployment infrastructure
automatic deployment URLs
global CDN integration
serverless function packaging
edge function support
immutable deployment storage
instant rollback system
routing layer
cache invalidation logic
team/project management
```

At that point, I would no longer be configuring a server.

I would be building a platform.

And that is basically what Vercel is.

This was the part that changed how I looked at the problem.

The goal should not be to "beat Vercel" on a single VM.

That is not a fair comparison.

The better goal is to understand what parts of the Vercel experience I actually need and reproduce those parts.

## What I Managed To Get Close To

I cannot fully recreate Vercel on my own server, but I can get close enough for my needs.

For example, I can build Docker images in GitHub Actions instead of on the server.

I can use Docker layer caching to speed up repeated builds.

I can push images to a registry and make the server only pull already-built releases.

I can use blue-green deployments so a new container starts beside the old one, passes health checks, and only then receives traffic.

I can use Traefik or Nginx to switch routing.

I can keep old image tags so rollback is easy.

I can put Cloudflare in front of everything to cache static content and protect the server.

This gives me a deployment flow like this:

```text
Git push
    ↓
GitHub Actions builds image
    ↓
Image is pushed to registry
    ↓
Server pulls image
    ↓
New container starts beside old one
    ↓
Health check passes
    ↓
Traffic switches
    ↓
Old container is removed later
```

That is already very close to the experience I wanted.

For backend services, it is more than enough.

For APIs, dashboards, workers, internal tools, and small production apps, this setup is powerful.

The server remains under my control, costs stay predictable, and every project follows the same deployment pattern.

## Where Vercel Still Wins

Vercel still wins for frontend-heavy projects.

Especially for frameworks like Next.js.

That is expected because Vercel is built around that ecosystem.

When the output is mostly static files, JavaScript bundles, and serverless functions, shipping a full Docker image is unnecessary overhead.

A better self-hosted approach for such websites might be artifact-based deployment instead of Docker-based deployment.

For example:

```text
build the site
upload the dist folder
switch symlink to new release
reload process manager if needed
purge CDN cache
```

That can be faster than Docker for frontend projects because it avoids image build and image pull overhead.

So the lesson is not "Docker is bad."

The lesson is:

```text
Docker is great for services.
Artifacts are better for static/frontend deployments.
```

## What I Learned

The main thing I learned is that deployment speed is not only about caching or server power.

It is about the deployment unit.

If I deploy a container, I pay the cost of moving and restarting a container.

If I deploy an artifact, I only move the final output.

That difference sounds small until you actually compare the two systems.

Vercel feels faster because it is designed around the smallest useful deployment unit for web apps.

My setup is more general-purpose.

That generality is useful, but it costs time.

Once I understood that, I stopped seeing Vercel as magic.

It is not magic.

It is architecture.

## Why I Still Like Self-Hosting

Even after realizing all this, I still like my self-hosted setup.

It gives me control.

I know where my services are running.

I understand the deployment pipeline.

I can host multiple applications on one server.

I can use custom domains however I want.

I can run databases, backends, dashboards, bots, and internal tools without depending on a specific platform's pricing or limitations.

There is also a different kind of satisfaction in pushing code and knowing exactly what happens after that push.

The code goes through my CI pipeline.

The image is built.

The server pulls it.

The container starts.

Traffic routes through my reverse proxy.

Everything is visible.

Everything is mine to debug.

That feeling is hard to replace.

## Final Thought

I started this by asking why Vercel was faster than my Docker setup.

At first, I thought the answer would be some missing optimization in my pipeline.

Maybe better caching.

Maybe better Docker layers.

Maybe a faster server.

But the real answer was simpler and deeper.

I was deploying containers.

Vercel was deploying artifacts.

That is the difference.

I managed to get close by improving my CI/CD pipeline, using prebuilt images, adding caching, using Cloudflare, and making deployments more release-based.

But fully matching Vercel is not really possible unless I build a platform like Vercel myself.

And honestly, that is fine.

Because the point of self-hosting was never to become Vercel.

The point was to understand the system deeply enough to know what trade-offs I was making.

Now I do.