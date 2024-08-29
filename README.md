# Elastic stack (ELK) on KVM using Ansible

Run the latest version of the [Elastic stack][elk-stack] on KVM using Ansible.

It gives you the ability to analyze any data set by using the searching/aggregation capabilities of Elasticsearch and
the visualization power of Kibana.

The purpose of this Ansible playbook is to:

  - install QEMU/KVM if not already installed
  - provision needed VMs on KVM
  - deploy the different elements of the Elastic stack on previously created VMs

**WARNING**: this playbook has only be tested on a development machine on Ubuntu 22.04, which is also the hypervisor machine where QEMU/KVM will be deployed. 

## Contents

- [Elastic stack (ELK) on KVM using Ansible](#elastic-stack-elk-on-kvm-using-ansible)
  - [Contents](#contents)
  - [Requirements](#requirements)
    - [Ansible](#ansible)
    - [Host setup (on Linux only)](#host-setup-on-linux-only)
    - [Ports](#ports)
    - [Accessing Kibana](#accessing-kibana)
  - [Usage](#usage)
    - [Deploying VMs](#deploying-vms)
    - [Deploying Elastic stack](#deploying-elastic-stack)
  - [Initial setup](#initial-setup)
    - [Setting up user authentication](#setting-up-user-authentication)
  - [Configuration](#configuration)
    - [How to configure Elasticsearch](#how-to-configure-elasticsearch)
    - [How to configure Kibana](#how-to-configure-kibana)
  - [Using a newer stack version](#using-a-newer-stack-version)

## Requirements

### Ansible

You need to have Ansible installed on your machine (recent version).

### Host setup (on Linux only)

* [Setting Up KVM and Virtualization on Ubuntu](https://www.liquidweb.com/kb/how-to-set-up-virtualization-host-using-kvm-ubuntu/)

A new graphical tool is available: "Virtual machines manager", which allows you to manage VM.

### Ports

Kibana is available on `http://kbn01:5601`

### Accessing Kibana

By default, the stack exposes the following ports:
* 9200: Elasticsearch HTTP
* 5601: Kibana

## Usage

### Deploying VMs

The playbook can be executed with the following method:

```
ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass
```

To deploy only some parts of this playbook, you can use associated tags:

```
ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass --tags <my-tag>
```

For example, if KVM and prerequisites are already installed, and you just need to create VMs, you can use:

```
ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass --tags vm
```

Availables tags:

  - *kvm*: install KVM and prerequisites
  - *vm*: deploy VMs according to specifications (`inventories/dev/group_vars/hypervisor/vars.yml`)

Before deploying VMs, **check specifications to ensure hypervisor host has sufficient resources**.

### Deploying Elastic stack

The playbook can be executed with the following method:

```
ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts --ask-become-pass
```

To deploy only some parts of this playbook, you can use associated tags:

```
ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts --ask-become-pass --tags <my-tag>
```

For example, if you just need Elasticsearch nodes, you can use:

```
ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts --ask-become-pass --tags es
```

Availables tags:

  - *os*: Update all packages, except the Elastic ones
  - *es*: deploy Elasticsearch nodes
  - *kbn*: deploy Kibana instance
  - *mon*: deploy Filebeat/Metricbeat instances for self monitoring
  - *cfg*: update configuration files and restart associated service if needed
  - *certs*: regenerate ca and certs if zip no more present on /tmp and restart associated service if needed
  - *pwd*: reset passwords of all components accounts

## Initial setup

### Setting up user authentication

The stack is pre-configured with the following **privileged** bootstrap user:

* user: *elastic*
* account_password: *changeme*

This is done by resetting the elastic password on first node.

Since v8 of Elastic stack, components won't work with this user, so we have to configure the unprivileged built-in users.

The stack automatically configure the following builtin-users:

* user: *kibana_system*
* account_password: *changeme*

* user: *remote_monitoring_user*
* account_password: *changeme*

The passwords are defined in the `inventories/dev/group_vars/all.yml`.

## Configuration

### How to configure Elasticsearch

The Elasticsearch configuration is deployed with `elasticsearch.yml` j2 template in the elasticsearch role.

You can change any option you want by setting it directly in this file or by changing variable value.

### How to configure Kibana

The Kibana configuration is deployed with `kibana.yml` j2 template in the kibana role.

You can change any option you want by setting it directly in this file or by changing variable value.

## Using a newer stack version

To use a different Elastic Stack version than the one currently available in the repository, simply change the `elastic_stack` number inside the ``inventories/dev/group_vars/all.yml` file of the dev inveotory, and redeploy the stack.

Fo major updates, don't forget to also change repository variable.