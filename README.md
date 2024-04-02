# Elastic stack (ELK) on KVM using Ansible

Run the latest version of the [Elastic stack][elk-stack] on KVM using Ansible.

It gives you the ability to analyze any data set by using the searching/aggregation capabilities of Elasticsearch and
the visualization power of Kibana.

The purpose of this Ansible playbook is to:

  - install QEMU/KVM if not already installed
  - provision needed VMs on KVM
  - deploy the different elements of the Elastic stack on previously created VMs

** WARNING **: this playbook has only be tested on a development machine on Ubuntu 22.04, which is also the hypervisor machine where QEMU/KVM will be deployed. 

## Contents

- [Elastic stack (ELK) on KVM using Ansible](#elastic-stack-elk-on-kvm-using-ansible)
  - [Contents](#contents)
  - [Requirements](#requirements)
    - [Ansible](#ansible)
    - [Host setup (on Linux only)](#host-setup-on-linux-only)
    - [Ports](#ports)
  - [Usage](#usage)
    - [Deploying VMs](#deploying-vms)
    - [Deploying Elastic stack](#deploying-elastic-stack)
  - [Initial setup](#initial-setup)
    - [Setting up user authentication](#setting-up-user-authentication)
  - [Configuration](#configuration)
    - [How to configure Elasticsearch](#how-to-configure-elasticsearch)
  - [Using a newer stack version](#using-a-newer-stack-version)

## Requirements

### Ansible

You need to have Ansible installed on your machine (recent version).

### Host setup (on Linux only)

* [Setting Up KVM and Virtualization on Ubuntu](https://www.liquidweb.com/kb/how-to-set-up-virtualization-host-using-kvm-ubuntu/)

A new graphical tool is available: "Virtual machines manager", which allows you to manage VM.

### Ports

By default, the stack exposes the following ports:
* 9200: Elasticsearch HTTP

## Usage

### Deploying VMs

The playbook can be executed with the following method:

  ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass

To deploy only some parts of this playbook, you can use associated tags:

  ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass --tags <my-tag>

For example, if KVM and prerequisites are already installed, and you just need to create VMs, you can use:

  ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass --tags vm

### Deploying Elastic stack

The playbook can be executed with the following method:

  ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts

To deploy only some parts of this playbook, you can use associated tags:

  ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts --tags <my-tag>

For example, if you just need Elasticsearch nodes, you can use:

  ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts --tags es

## Initial setup

### Setting up user authentication

The stack is pre-configured with the following **privileged** bootstrap user:

* user: *elastic*
* account_password: *changeme*

This is done by resetting the ELASTIC_PASSWORD on first node.

## Configuration

> Configuration is not dynamically reloaded, you will need to restart individual components after any configuration change.

### How to configure Elasticsearch

The Elasticsearch configuration is stored in `elasticsearch/config/elasticsearch.yml`.

You can also specify the options you want to override by setting environment variables inside the Compose file.

## Using a newer stack version

To use a different Elastic Stack version than the one currently available in the repository, simply change the `elastic_stack` number inside the `groupvars/all.yml` file, and redeploy the stack with:

> Always pay attention to the [upgrade instructions][upgrade] for each individual component before performing a stack upgrade.

[elk-stack]: https://www.elastic.co/elk-stack

[upgrade]: https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html