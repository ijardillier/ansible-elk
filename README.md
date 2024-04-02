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
      - [Deploying VMs](#deploying-vms)
      - [Deploying Elastic stack](#deploying-elastic-stack)
    - [Host setup (on Linux only)](#host-setup-on-linux-only)
    - [Ports](#ports)
  - [Usage](#usage)
    - [Bringing up the stack](#bringing-up-the-stack)
    - [Bringing up extensions](#bringing-up-extensions)
    - [Cleanup](#cleanup)
  - [Initial setup](#initial-setup)
    - [Setting up user authentication](#setting-up-user-authentication)
    - [Injecting data](#injecting-data)
    - [Default Kibana data view creation](#default-kibana-data-view-creation)
  - [Configuration](#configuration)
    - [How to configure Elasticsearch](#how-to-configure-elasticsearch)
    - [How to configure Kibana](#how-to-configure-kibana)
  - [JVM tuning](#jvm-tuning)
  - [Using a newer stack version](#using-a-newer-stack-version)

## Requirements

### Ansible

You need to have Ansible installed on your machine (recent version).

#### Deploying VMs

The playbook can be executed with the following method:

  ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass

To deploy only some parts of this playbook, you can use associated tags:

  ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass --tags <my-tag>

For example, if KVM and prerequisites are already installed, and you just need to create VMs, you can use:

  ansible-playbook playbook/deploy-vms.yml -i playbook/inventories/dev/hosts --ask-become-pass --tags vm

#### Deploying Elastic stack

The playbook can be executed with the following method:

  ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts

To deploy only some parts of this playbook, you can use associated tags:

  ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts --tags <my-tag>

For example, if you just need Elasticsearch nodes and Kibana, you can use:

  ansible-playbook playbook/deploy-stack.yml -i playbook/inventories/dev/hosts --tags es,kbn

### Host setup (on Linux only)

* [Setting Up KVM and Virtualization on Ubuntu](https://www.liquidweb.com/kb/how-to-set-up-virtualization-host-using-kvm-ubuntu/)

A new graphical tool is available: "Virtual machines manager", which allows you to manage VM.

### Ports

By default, the stack exposes the following ports:
* 9200: Elasticsearch HTTP
* 5601: Kibana

## Usage

### Bringing up the stack

Clone this repository onto the Docker host that will run the stack, then start services locally using Docker Compose:

```console
$ docker-compose up
```

You can also run all services in the background (detached mode) by adding the `-d` flag to the above command.

If you are starting the stack for the very **first time**, please read the section below attentively.

### Bringing up extensions

You can find extension documentation in dedicated folder.

To launch the complete stack will all extensions:

```console
$ docker compose -f docker-compose.yml -f extensions/logstash/logstash-compose.yml -f extensions/prometheus/prometheus-compose.yml -f extensions/fleet-server/fleet-server-compose.yml -f extensions/apm-server/apm-server-compose.yml -f extensions/beats/beats-compose.yml -f extensions/enterprise-search/enterprise-search-compose.yml up -d
```

### Cleanup

Elasticsearch, Kibana, ... data are persisted inside volumes by default.

In order to entirely shutdown the stack and remove all persisted data, use the following Docker Compose command:

```console
$ docker-compose down -v
```

## Initial setup

### Setting up user authentication

The stack is pre-configured with the following **privileged** bootstrap user:

* user: *elastic*
* account_password: *changeme*

This is done by setting the ELASTIC_PASSWORD in the environment variables on each node (esxx).

Since v8 of Elastic stack, components won't work with this user, so we have to configure the unprivileged built-in users.

The stack automatically configure the following builtin-users:

* user: *kibana_system*
* account_password: *changeme*

* user: *logstash_system*
* account_password: *changeme*

* user: *beats_system*
* account_password: *changeme*

* user: *remote_monitoring_user*
* account_password: *changeme*

In this docker compose, we use a temporary elasticsearch container to setup built-in users passwords (es00).

The passwords are defined in the .env file. **Don't forget to change them before you run the docker compose for the first time**.

The `logstash_system` and `beats_system` users are no more used since logstash and beats monitoring is now done with metricbeat.

The `remote_monitoring_user` password is used in in Metricbeat xpack modules, for Elasticsearch and Kibana monitoring.

> Do not use the `logstash_system` user inside the Logstash *pipeline* file, it does not have sufficient permissions to create indices. [Configuring Security in Logstash]> 
> Follow the instructions at [Secure you connection to Elasticsearch][ls-security] to create a user with suitable roles.

> Learn more about the security of the Elastic stack at [Tutorial: Getting started with security][secure-cluster].

### Injecting data

Give Kibana about a minute to initialize, then access the Kibana web UI by hitting
[http://localhost:5601](http://localhost:5601) with a web browser and use the following default credentials to log in:

* user: *elastic*
* account_password: *\<your generated elastic password>*

Now that the stack is running, you can load the sample data provided by your Kibana installation.

### Default Kibana data view creation

When Kibana launches for the first time, it is not configured with any data view.

> You need to configure each needed data view in order to discover data.

## Configuration

> Configuration is not dynamically reloaded, you will need to restart individual components after any configuration change, except for logstash pipelines, beats modules and inputsn which have auto reload enabled.

### How to configure Elasticsearch

The Elasticsearch configuration is stored in `elasticsearch/config/elasticsearch.yml`.

You can also specify the options you want to override by setting environment variables inside the Compose file.

### How to configure Kibana

The Kibana default configuration is stored in `kibana/config/kibana.yml`.

You can also specify the options you want to override by setting environment variables inside the Compose file.

## JVM tuning

By default, Elasticsearch, Kibana and Logstash are limited to XGb of memory. You can change this value with the MEM_LIMIT variable in the .env file.

For each Elasticsearch node, JVM will automatically be limited to 50% of allocated memory.

For Logstash, we need to limit the JVM to 50% of the allocated memory. This is done by setting the LS_JAVA_OPTS in the .env file.

## Using a newer stack version

To use a different Elastic Stack version than the one currently available in the repository, simply change the version
number inside the `.env` file, and rebuild the stack with:

```console
$ docker-compose up
```

> Always pay attention to the [upgrade instructions][upgrade] for each individual component before performing a stack upgrade.

[elk-stack]: https://www.elastic.co/elk-stack
[prometheus]: https://prometheus.io

[linux-postinstall]: https://docs.docker.com/install/linux/linux-postinstall/

[elasticsearch-docker]: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
[es-sys-config]: https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html

[ls-security]: https://www.elastic.co/guide/en/logstash/current/ls-security.html
[secure-cluster]: https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-cluster.html

[upgrade]: https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html