---
layout: post
title: "Digital Ocean, Terraform and Kubernetes"
date: 2020-11-01 12:00:00 -0000
categories: digitalocean terraform kubernetes
---
<h1>Using the Digital Ocean Terraform provider to deploy a kuberneters cluster</h1> 

This blog post will describe how to run a kubernetes cluster on digital ocean, with the help of Terraform.  

<h3>Digital Ocean</h3>
Back when I first started hosting personal projects and playing around with some PaaS service, Digital Ocean was my go to cloud vendor. Its cheap, reliable, has a clean interface and API. And coming from AWS, which I was using professionally, it was a breath of fresh air. Fewer services but each with a clear purpose, and not the pilots cockpit I was used to on AWS. Caveat now being that I'm an unofficial Azure advocate.

A few (maybe more than a few), years into my dev career I was working on a contract with a team who were migrating onto kubernetes. I had no prior experience with berates so took some plural sight courses to get up to speed, as much as I could, it's a steep learning curve. A lot of the time in the tech world we are introduced to the next latest and greatest tool or new framework (looking at JS), but for the case of Kubernetes I agree. It can support legacy applications, is future proof and all the rest of the out of the box awesomeness it has! It was perfect for the migration contract I was working on.

<h4>Prerequisite</h4>
To follow this tutorial a few things are required, click the link to follow how to get each.
<div class="highlight"><pre class="highlight">
<span class="p">-</span> <a href="https://www.digitalocean.com/">Digital Ocean Account</a> 
<span class="p">-</span> <a href="https://www.digitalocean.com/docs/apis-clis/api/create-personal-access-token/">PAT Token</a> 
<span class="p">-</span> <a href="https://www.digitalocean.com/docs/droplets/how-to/connect-with-ssh/">SSH Key (password less)</a> 
</pre></div>

<h3>Terraform and Kubernetes</h3>
Download and install <a href="https://www.terraform.io/downloads.html"> Terraform</a> and place it on PATH variable to it can be exectured anywhere. Terraform has different providers and APIs the digital ocean documentation is <a href="https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs">here</a>. The PAT token from digtial ocean is required to use the DO provider. To make life easier we should store the token as a variable, never in source control! 

```powershell
export DO_TOKEN=****
```

Both terraform and digital ocean provide templates for deploying resources, below is the simple template to get started for the digital ocean API.

<script src="https://gist.github.com/dixneuf19/808ec11f9251ab9328d65c21848b28a2.js"></script>

Now that our provider file is ready  we need to create a kubernetes cluster in digital ocean, so save the following file, with the extension .tf for terraform. 

```json 
resource "digitalocean_kubernetes_cluster" "kubernetes_cluster" {
  name    = "terraform-do-cluster"
  region  = "ams3"
  version = "1.18.6-do.0"
  tags = ["my-tag"]
  # This default node pool is mandatory
  node_pool {
    name       = "default-pool"
    size       = "s-1vcpu-2gb"
    auto_scale = false
    node_count = 2
    tags       = ["node-pool-one"]
    labels = {
      "jwblog" = "up"
    }
  }
}
```
Validate the file using <i>terraform validate</i> command. Remeber no news is good news for this command.
Then to see the deployment shape you can execute 
```
terraform plan 
```
follow by 
```
terraform apply
```
To get our desired state.

Use <i>terraform show</i> to present the current status of the enviroment. It might take a few minutes to process. But once it has we can start deploying containers to our cluster. Simple!

<h3>Debrief</h3>
I expected this was going to be far more complicated before I started the challenge, especially as I haven't used digital ocean in a while and I've got little terraform experience. But given the amount of resources available, and I'm not one for reinventing the wheel, it went smoothly and took a couple of hours to get the cluster up and running.

<h4>AKS</h4>
Although I've not yet mentioned it, I've spent most of my Kubernetes experience on Azure, using AKS. And I thought I would share a quick compairson based on my first impression of DOKS (digital ocean kubernetes).

AKS has far more choices for the machine that Kubernetes is installed on, Digtial ocean is resrticted to only one Linux OS. 
DOKS, using Terraform, is very simple to set up, and if not using the Azure portal, so script vs script (or ARM) I would say DOKS is simpler to get running.

Networking with AKS can be a bit of a pain, especially when configuring load balancers but it does offer private load balancers which DOKS does not. And there seems to be less control over role management, compared to RBAC on azure. 