+++
draft = "false"
title = "Cheap Managed Kubernetes with Terraform"
date = "2019-04-18"
author = "Josh"
cover = "img/gcp-tf-kube.png"
description = "Deploying a cheap kubernetes cluster on GKE with Terraform"
+++

Kubernetes is the way to deploy your services in a scalable and reliable way. However, it's a pretty complex system to manage yourself. Thankfully, cloud providers are offering managed versions where you only pay for the worker nodes.

We'll use [GKE](https://cloud.google.com/kubernetes-engine/), Google's managed kubernetes offering, to deploy a cluster so we can test out kubernetes.

We'll use [Terraform](https://www.terraform.io/) to make sure we have a repeatable deployment process.

If you just want to skip to the code it's on [GitHub](https://github.com/kasuboski/cheap-managed-kubernetes).

The resources we'll deploy use the Google Cloud [free-tier](https://cloud.google.com/free/) extensively. If you leave it running, it should cost a little over $5 a month.

If you're not familiar with Terraform or haven't used the Google Provider, you can get started [here](https://www.terraform.io/docs/providers/google/getting_started.html). All of the resources it deploys will be in the free tier.

Terraform has a concept of remote backends which allow you to save the state of your deployments (not just on your machine). This is especially helpful if you have multiple team members.

Since we're already using Google Cloud we can use Google Cloud Storage to house our state. Once change some defaults we can run a few commands and have our cluster running.

* Create a Google Cloud Storage Bucket following these [instructions](https://cloud.google.com/storage/docs/creating-buckets)
* Clone the cheap-managed-kubernetes [repo](https://github.com/kasuboski/cheap-managed-kubernetes)
* Modify `terraform.tfvars.example` with your gcp project and rename to `terraform.tfvars`
* Modify `backend.hcl.example` with the gcs bucket you created above and rename to `backend.hcl.example`

You should now be set up to deploy with Terraform. We'll initialize Terraform with our remote backend and run a plan. This plan will output what will be created (or destroyed). You can verify the output of the plan is correct and then run the apply.

* `terraform init -backend-config=backend.hcl`
* `terraform plan` This should say it will create a cluster and node pool.
* `terraform apply` This will actually create the cluster and node pool.

The output of the apply will give you the info you need to create a `kubeconfig` to be able to connect to your cluster. Since we're using GKE though, I find it easier to just use the `gcloud` command that will set your `kubeconfig` for you.

It should look something like `gcloud container clusters get-credentials my-poor-gke-cluster` where `my-poor-gke-cluster` is the name of the cluster resource in `main.tf`

Once you have your `kubeconfig` set up, you can access your cluster like you normally would. Maybe try running `kubectl get pods --all-namespaces`. You should see the pods that make up `kube-system`.