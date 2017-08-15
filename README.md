# Kubernetes cluster setup automation

> This is part of the Hobby Kube project. Functionality of the modules is described in the [guide](https://github.com/hobby-kube/guide).

Deploy a secure Kubernetes cluster on [Scaleway](https://www.scaleway.com/) or [DigitalOcean](https://www.digitalocean.com/) using [Terraform](https://www.terraform.io/).

## Setup

### Requirements

The following packages are required to be installed locally:

```sh
brew install terraform kubectl jq wireguard-tools
```

Modules are using ssh-agent for remote operations. Add your SSH key with `ssh-add -K` if Terraform repeatedly fails to connect to remote hosts.

### Configuration

Export the following environment variables depending on the modules you're using.

#### Using Scaleway as provider

```sh
export TF_VAR_scaleway_organization=<ACCESS_KEY>
export TF_VAR_scaleway_token=<TOKEN>
```

#### Using DigitalOcean as provider

```sh
export TF_VAR_digitalocean_token=<token>
export TF_VAR_digitalocean_ssh_keys=<keys> # e.g. '["121671", "1714133"]'
```

#### Using Cloudflare for DNS entries

```sh
export TF_VAR_domain=<domain> # e.g. example.org
export TF_VAR_cloudflare_email=<email>
export TF_VAR_cloudflare_token=<token>
```

#### Using Amazon Route 53 for DNS entries

```sh
export TF_VAR_domain=<domain> # e.g. example.org shall be already added to hosted zones.
export TF_VAR_aws_access_key=<ACCESS_KEY>
export TF_VAR_aws_secret_key=<SECRET_KEY>
export TF_VAR_aws_region=<region> # e.g. eu-west-1
```

### Usage

```sh
# fetch the required modules
$ terraform get

# see what `terraform apply` will do
$ terraform plan

# execute it
$ terraform apply
```

## Using modules independently

Modules in this repository can be used independently:

```
module "kubernetes" {
  source  = "github.com/hobby-kube/provisioning/service/kubernetes"
}
```

After adding this to your plan, run `terraform get` to fetch the module.


## Upgrade
```
sudo apt-get update
sudo apt-get upgrade
sudo reboot
```
If they don't connect again
```
ip link delete wg0
wg-quick wg0
wg
```


---
approvers:
- pipejakob
title: Upgrading kubeadm clusters from 1.6 to 1.7
---

{% capture overview %}

This guide is for upgrading kubeadm clusters from version 1.6.x to 1.7.x.
Upgrades are not supported for clusters lower than 1.6, which is when kubeadm
became Beta.

**WARNING**: These instructions will **overwrite** all of the resources managed
by kubeadm (static pod manifest files, service accounts and RBAC rules in the
`kube-system` namespace, etc.), so any customizations you may have made to these
resources after cluster setup will need to be reapplied after the upgrade. The
upgrade will not disturb other static pod manifest files or objects outside the
`kube-system` namespace.

{% endcapture %}

{% capture prerequisites %}
You need to have a Kubernetes cluster running version 1.6.x.
{% endcapture %}

{% capture steps %}

## On the master

1. Upgrade system packages.

   Upgrade your OS packages for kubectl, kubeadm, kubelet, and kubernetes-cni.

   a. On Debian, this can be accomplished with:

       sudo apt-get update
       sudo apt-get upgrade

   b. On CentOS/Fedora, you would instead run:

       sudo yum update

2. Restart kubelet.

       sudo systemctl restart kubelet

3. Delete the `kube-proxy` DaemonSet.

   Although most components are automatically upgraded by the next step,
   `kube-proxy` currently needs to be manually deleted so it can be recreated at
   the correct version:

       sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl delete daemonset kube-proxy -n kube-system

4. Perform kubeadm upgrade.

    **WARNING**: All parameters you passed to the first `kubeadm init` when you bootstrapped your
    cluster **MUST** be specified here in the upgrade-`kubeadm init`-command. This is a limitation
    we plan to address in v1.8.

       sudo kubeadm init --skip-preflight-checks --kubernetes-version <DESIRED_VERSION>

   For instance, if you want to upgrade to `1.7.0`, you would run:

       sudo kubeadm init --skip-preflight-checks --kubernetes-version v1.7.0

5. Upgrade CNI provider.

   Your CNI provider might have its own upgrade instructions to follow now.
   Check the [addons](/docs/concepts/cluster-administration/addons/) page to
   find your CNI provider and see if there are additional upgrade steps
   necessary.

## On each node

1. Upgrade system packages.

   Upgrade your OS packages for kubectl, kubeadm, kubelet, and kubernetes-cni.

   a. On Debian, this can be accomplished with:

       sudo apt-get update
       sudo apt-get upgrade

   b. On CentOS/Fedora, you would instead run:

       sudo yum update

2. Restart kubelet.

       sudo systemctl restart kubelet

{% endcapture %}

{% include templates/task.md %}
