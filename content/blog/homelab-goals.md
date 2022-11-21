---
title: "Homelab Goals"
date: 2022-11-20T16:29:44-06:00
draft: false
---

I have had a homelab running in some form or another for the last 20 years. First it was just a couple of machine to test things out on, then it became a way to host my website, email and all sorts of other things. My current homelab is falling apart and I'd like to both simplify it's operations a bit and align it with what I am interested in.

### Current Lab

My lab right now consists of three DL380 G7's with dual CPU's and a bunch of memory plus some direct attached disks. They are all running [Debian](http://debian.org) with the HashiCorp toolset (Consul, Nomad and Vault). Nomad runs the containers using the Docker plguin, they are registered in the Consul service mesh and the secrets are pulled from Vault.
The machines are managed using a bunch of Ansible playbooks which do things like manage the base configuration of the machine, install the base tools, and install and configure all of the Hash. I then use Terraform to manage the Consul/Nomad and Vault services.

### Issues with the setup
This has been working relatively well for the last few years but there are some issues. Some of the issues are technology related and some are just things I don't like about the setup in the order that they frustrate me:
1. The Nomad secret retrieval from Vault is hard to get right, currently if I stop a Job through Nomad and try to restart it, the Vault token is "invalid". I can work around this by removing the job and re-adding it. Not the biggest problem in the world but I haven't found a solution for this yet.
1. The disks attached to the machine are direct attached, so there is no way to share a disk between two machines (ofcourse there are ways to do this, but all of them are super complicated to setup or are slow/resource intensive). This limitation means that services that share storage all need to be on the same host.
1. Ansible playbooks are annoying, Ansible-vault is annoying to use for secrets in code.
1. The backups are cobbled together using Restic + RClone. Nightly Restic jobs kick off on the first two machines to backup to the third, which then uploads the backup to Backblaze. This is workig fine, but not the most efficient and the storage used on Restic's volume and on Backblaze has been growing, it's now 2x as big as the volume it's backing up (not all backed up volumes are uploaded to Backblaze).
1. I'm using auto-unseal on Vault using AWS KMS and I'd like to just stop paying for it.

### Where to go from here
A bunch of the tools that I selected for the current iteration of the lab were selected because I was working with the tools, Ansible, Consul, Nomad and Vault are all things I don't use at my current job.
This is why I'm going to start replacing these components with TrueNAS for storage, Kubernetes (MicroK8s) for compute and keep Terraform for some of the configuration management. My goal with the new setup is to be able to publish the full "infrastructure" Git repository to GitHub (encrypted secrets and all).

- TruesNAS: I'm going to setup two seperate machines (DL360 G7's I have laying around) and setup one primary storage machine with ZFS replication to a backup machine that will replicate some of the more critical volumes to Backblaze.
- Kuberntes: The existing three nodes will be setup with MicroK8s on Debian, I want to write a seperate post about the Kubernetes dream setup. The shared storage will be provided by the TrueNAS machines.
- Terraform: This will manage the configuration of some of the services that will be running on K8s and any external services like Cloudflare.

This homelab rebuild is going to be a work in pgoress and will probably change as things evolve.
