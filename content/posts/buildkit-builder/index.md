---
title: "Running a Buildkit ARM Builder"
date: 2022-05-08T15:34:19-05:00
draft: false
---

I was sick of my hour long ARM docker builds. A 15x speedup using existing infrastructure isn't bad.

<!--more-->

## The Problem
I build my feedreader for ARM in Github Actions. The workflow builds a multi-arch docker image on push. The x86 build was pretty quick, but the ARM build was using qemu which made it take around an hour. QEMU certainly didn't help, but the Github Actions runners aren't exactly the biggest machines at 2 vcpu and 7GB ram.

My build was using the [docker buildx](https://github.com/docker/setup-buildx-action) action. This makes the build use the newish buildkit backend for docker, but it's still running on the actions runner. I wanted to see if I could run my own buildkit backend. There was the option to connect it with a remote docker endpoint or a kubernetes cluster. Neither of which are really that appealing to me, although exposing the docker daemon over Tailscale could be fun. 

## The Solution
Right as I was looking to run my own `buildkitd`, `buildx` had a PR merged that would enable a remote builder driver. This lets you run `buildkitd` somewhere and expose it over tcp. My [kubernetes cluster]({{< ref "multi-region-k3s" >}}) has a free ARM node from Oracle that is pretty big (4 x 24GB). It's usually nowhere near fully utilized.

Running a builder on it seemed like a great way to use the excess resources. Combined with `tailscale` and the recommended mTLS auth I could have a rather secure build runner on my existing infrastructure.

## Setting it up
The [buildkit](https://github.com/moby/buildkit#expose-buildkit-as-a-tcp-service) repo has instructions for running it over TCP. There is also an [example](https://github.com/moby/buildkit/tree/master/examples/kubernetes#deployment--service) that shows how to run it in kubernetes with a deployment. I chose the deployment and service option vs a statefulset with consistent hashing because I was planning to use registry caching anyway and don't have immediate plans for many different builds to use this.

I decided to expose it with Tailscale using the same [process]({{< ref "tailscale-connect-kubernetes-pods.md" >}}) I had previously used for my feedreader. This means connecting to it requires you be on my tailnet (authenticated with Tailscale).

In addition to requiring you be authenticated with Tailscale, the doc still recommends you use mTLS because the steps being built in the builder could potentially access the daemon as well. The example has a script to set up the certs for you, but I wanted to use the step cli from [Smallstep](https://smallstep.com/). It's still very simple, but I could control exactly what is set up.

### Creating Certificates
The first step to run `buildkitd` was to create the certificates it wants. I decided to make a Root CA for this along with an Intermediate CA and then server and client certificates. I didn't spend too long debating this and just followed a Smallstep guide...

Creating the CA
```bash
step certificate create --profile root-ca "Buildkit Root CA" root_ca.crt root_ca.key
```

Creating the Intermediate CA
```bash
step certificate create "Buildkit Intermediate CA 1" \
    intermediate_ca.crt intermediate_ca.key \
    --profile intermediate-ca --ca ./root_ca.crt --ca-key ./root_ca.key
```     

Creating the server cert
```bash
step certificate create buildkitd --san buildkitd --san localhost --san 127.0.0.1 buildkitd.crt buildkitd.key \
    --profile leaf --not-after=8760h \
    --ca ./intermediate_ca.crt --ca-key ./intermediate_ca.key --bundle --no-password --insecure
```
Creating the client cert
```bash
step certificate create client client.crt client.key \
    --profile leaf --not-after=8760h \
    --ca ./intermediate_ca.crt --ca-key ./intermediate_ca.key --bundle --no-password --insecure
```

You'll notice the server has a `buildkitd` san, which is how I'll access it over Tailscale. The `local` ones were for testing while port forwarding to the cluster.

### Running the Server
You can find the example kubernetes yaml [here](https://github.com/moby/buildkit/blob/master/examples/kubernetes/deployment%2Bservice.rootless.yaml). It expects a kubernetes secret with `ca.pem` and `key.pem` keys. You can generate that from below.

```bash
kubectl create secret generic buildkit-daemon-certs --from-file=key.pem=buildkitd.key --from-file=ca.pem=root_ca.crt --dry-run=client -oyaml
```

My actual deployment can be found in [kasuboski/k8s-gitops](https://github.com/kasuboski/k8s-gitops/tree/main/builder/buildkit). It includes the [tailscale-proxy](https://github.com/kasuboski/tailscale-proxy) as well as a `nodeSelector` to make sure it schedules on the ARM node. It requests 1cpu and 512Mi with the limit set to 3.5cpu and 3Gi. It ends up having more cpu than Github Actions and isn't emulated. The memory is less, but hasn't been an issue.

Once the server is running, it will be available in tailscale at `buildkitd` since the proxy uses the deployment name.

### Connecting as a Client
The client needs to have access over tailscale and a client cert. The easiest way is to use `buildctl`.

```bash
buildctl --addr 'tcp://buildkitd:1234' \
    --tlscacert root_ca.crt \
    --tlscert client.crt \
    --tlskey client.key  \
    build --frontend dockerfile.v0 --local context=. --local dockerfile=.
```

Building on multiple platforms with different builders requires `docker buildx`. The remote driver is on `master`, but isn't in a release yet. You can build buildx yourself to get access to that feature, but I only used it from Github Actions.

The [setup buildx action](https://github.com/docker/setup-buildx-action) has the option to build buildx from a specific commit.

### Running in Github Actions
If you want to skip to the workflow it's at [kasuboski/feedreader](https://github.com/kasuboski/feedreader/blob/696debe2da1d26f1e4047806ff5e1f5ca5fbe347/.github/workflows/ci.yaml).

The workflow will need secrets for Tailscale and the certificates. I use [Doppler](https://doppler.com/join?invite=390F66AC) _referral link_ to manage the secrets. It synced super fast and has a nicer interface than doing it per repo in Github imo.

Tailscale has a Github Action that will install and set it up given an auth key. They support ephemeral auth keys so you won't have a bunch of leftover machines in their system. Once installed, your workflow will have access to your tailnet and can reach `buildkitd`. It's worth noting DNS magically works thanks to [Magic DNS](https://tailscale.com/kb/1081/magicdns/). Connecting to a kubernetes pod with a nice name and no other network setup is life changing.

I had problems using the remote buildx driver with a different builder type. I ended up just running another `buildkitd` on the actions runner. In the future, I'd like to run an x86 builder on one of my nodes.

That's setup following inspiration from the buildx tests. This builder doesn't have mTLS setup, but I guess I'm fine for now since it's an ephemeral runner on Github's infrastructure 🤷‍♂️.

```bash
docker run -d --name buildkitd --privileged -p 1234:1234 moby/buildkit:buildx-stable-1 --addr tcp://0.0.0.0:1234
docker buildx create --name gh-builder --driver remote --use tcp://0.0.0.0:1234
docker buildx inspect --bootstrap
```
Adding my arm runner is done after as below. The certs have already been written to disk from the Github secrets.
```bash
docker buildx create --append --name gh-builder \
    --node arm \
    --driver remote \
    --driver-opt key="$GITHUB_WORKSPACE/key.pem" \
    --driver-opt cert="$GITHUB_WORKSPACE/client_cert.pem" \
    --driver-opt cacert="$GITHUB_WORKSPACE/ca_cert.pem" \
    tcp://buildkitd:1234
docker buildx ls
```
The `docker buildx ls` output should then show a `gh-builder` with two nodes, one supporting `amd64` and the other `arm`.
{{< figure src="buildx-ls.png" link="buildx-ls.png" alt="Docker Buildx List" >}}

After setting up the builder, the workflow went from an hour to under four minutes.

## Next Steps
Building on my excess capacity has been great and I want to add an x86 node as well. I could have run my own Github Runners, but that seems much more intense. All of my repos are public as well, so I figure I might as well use the free actions minutes.

I do want to potentially make a service that will just give you a remote buildkit builder on demand. It's particularly helpful for ARM builds since those can be slow in emulation.

I also looked into the cross compilation options, but just getting a native builder seemed easier and more flexible. Your `Dockerfile` still has to not download a specific architecture explicitly, but otherwise most should be able to build multi-arch with this setup.
