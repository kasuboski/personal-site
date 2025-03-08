---
title: "Building a Python Docker Image with Distroless and Uv"
date: 2025-03-08T13:03:13-06:00
draft: false
---

The images are too damn big! 📈 Let's use a sane project manager and build an image with minimal dependences.

<!--more-->

## Tooling
This will assume you use [uv](https://docs.astral.sh/uv/) to manage your python project. It's really made me actually consider using python now... Uv has a pretty nice docker tutorial on GitHub that we're going to base off. You can find that [here](https://github.com/astral-sh/uv-docker-example).

It's an example fastapi project that returns `hello world`.

## What's different?
Their example includes a standalone multi-stage build that is pretty good. The standalone aspect comes from allowing uv to install their version of python that will match your project. The multi-stage build ensures you end up with a base debian image with just python and your app.

I wanted to not include the source code of the app separately since it'll be in the virtualenv anyway as well as get rid of debian.

So we'll base on a google distroless image.

## Breaking down the Dockerfile
If you're impatient you can just read the Dockerfile. The first new thing is the builder is installing its own version of python. There are two environment variables that tell uv to only use the version of python that is managed by uv.

`UV_PYTHON_INSTALL_DIR` is the directory where the python version will be installed and `UV_PYTHON_PREFERENCE` tells uv to only use the version of python that is managed by uv.

It then actually installs python. After we get python, we can install the project dependencies. Uv can install only the dependencies with `--no-install-project`. It only needs the lock file and `pyproject.toml` for this. Installing the dependencies separately ensures better caching since you probably don't change dependencies as much as the code.

After dependencies, we can copy the app and install the rest. Using `--no-editable` tells uv to not install the project with any dependency on the source code. Then our final image can be created from just the virtualenv.

The final runtime image is `gcr.io/distroless/cc`. This is a google [distroless](https://github.com/GoogleContainerTools/distroless?tab=readme-ov-file) image that's a little smaller than `debian:slim`. Some of your python dependencies might still expect glibc at runtime so we use `cc` vs a `static` option.

The only thing we need then is to copy the python we installed with uv and the virtualenv. Adding the virtualenv path lets us reference any of the tools like fastapi.

```dockerfile
FROM ghcr.io/astral-sh/uv:bookworm-slim AS builder
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy

# Configure the Python directory so it is consistent
ENV UV_PYTHON_INSTALL_DIR=/python

# Only use the managed Python version
ENV UV_PYTHON_PREFERENCE=only-managed

# Install Python before the project for caching
RUN uv python install 3.12

WORKDIR /app
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --no-install-project --no-dev --no-editable
COPY . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev --no-editable

# Then, use a final image without uv
FROM gcr.io/distroless/cc

# Copy the Python version
COPY --from=builder --chown=python:python /python /python

WORKDIR /app
# Copy the application from the builder
COPY --from=builder --chown=app:app /app/.venv /app/.venv

# Place executables in the environment at the front of the path
ENV PATH="/app/.venv/bin:$PATH"

# Run the FastAPI application by default
CMD ["fastapi", "run", "--host", "0.0.0.0", "/app/.venv/lib/python3.12/site-packages/uv_docker_example"]
```

## But why?
Making the smallest image can help with startup time as you need to download and load less before you get to your app. For me, the biggest reason to go for a smaller image though is to reduce the vulnerability surface. You can't have vulnerabilities if there's nothing to be vulnerable.

Python isn't super conducive to an ultra minimal image. The python interpreter is around 75MB with our virtualenv being around 55MB. The smallest image we could hope for is then 130MB. The distroless base is 23.5 MB so it's quite a fraction of overall size.

The `debian:bookworm-slim` image is around 75MB. It also includes things like bash which can be nice for debugging, but also for vulnerabilities.

It is worth noting, when I was doing this, `debian:bookworm-slim` had no vulnerabilities reported by `trivy`. The base images are generally kept up to date to patch vulnerabilities, but more vulnerabilities are found because of their nature. If you are rebasing your images often it may not matter to you.

I previously explored continously scanning using [`trivy`]({{< ref "image-scanning-trivy" >}}) in a separate post using github actions.


