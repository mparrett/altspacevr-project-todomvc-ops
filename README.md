# AltspaceVR Project - TodoMVC Operations


## Overview

This is a sample Django TodoMVC application running in a moderately simple environment.

Nginx is used as a reverse proxy for gunicorn which runs the python application in a virtualenv. Supervisor is used to monitor and control gunicorn. Postgresql is used for persistence.

Currently, one machine is provisioned for the web server and another for the database. To be kind to aging Macbooks, the Virtualbox VMs were given minimal memory resources. The instructions below were written with MacOS in mind, but could be adapted to other platforms.

### Design Decisions / Principles

Priority was given to keeping the moving parts to a minimum and embracing reliable tools in order to deliver a working demonstration within a reasonable amount of time. The project attempts to be well organized and follow best practices wherever possible. Maintaining parity between local and remote environments is demonstrated. Chunks of existing playbooks were utilized rather than writing everything from scratch. It seems quite reasonable that this basic setup could eventually evolve into a containerized version.

### Provisioning and Configuration

Ansible is used to provision and configure the resources for the TodoMVC app. Dynamic Ansible inventories are utilized and both Vagrant and Digital Ocean were tested as providers. Extending to AWS would be straightforward.

## Getting Started

Make sure the requirements below are met and then get TodoMVC running on your system with one command (after cloning).

```
git clone git@github.com:mparrett/altvr-todomvc-django.git
cd altvr-todomvc-django
```

```
vagrant up
```

And once that's done, visit [https://192.168.33.15](https://192.168.33.15) and enjoy some TODOs.

### Requirements

- Ansible 1.9+ `pip install ansible --upgrade`
- Virtualbox (Local) `brew cask install virtualbox`
- Vagrant (Local) `https://www.vagrantup.com/downloads.html`

### Extra Requirements for Digital Ocean

When running on Digital Ocean you'll need to do a couple of things to enable the dynamic inventory script `digital_ocean.py` to access your account.

```
pip install dopy
export DO_API_TOKEN=YOUR_TOKEN
ansible-playbook -i digital_ocean.py digitalocean.yml --user YOUR_USER --extra-vars 'digitalocean_ssh_key_ids=123'
```

You can also access TodoMVC hosted on my Digital Ocean account here:

[https://107.170.252.197/](https://107.170.252.197/)

## Code Deployment

A simple git-based deployment system is used to deploy new code during development. The repository is configured via ansible variables and can be deployed via `ansible-playbook vagrant.yml --tags=deploy`. By default this deploys the `master` branch but this is configurable via ansible variables.

The sample Django app provided with the challenge is here on [GitHub](https://github.com/mparrett/altvr-todomvc-django) and was slightly modified to meet the requirements.

## Monitoring with Datadog

The project demonstrates basic Datadog integration for monitoring and alerting.

![](http://i.imgur.com/nuAGoDa.png)

### Integrations

Included in the demo are Nginx, gunicorn, and Postgresql integrations which include many interesting and useful metrics. In addition, gunicorn is configured to speak via StatsD to the local Datadog agent.

### Alerts

A sample alert was created which informs us if the gunicorn worker count drops below the configured amount (3). This could be connected to PagerDuty and Slack.

![](http://i.imgur.com/JZSjm5W.png)

### Events

TODO: Deployment events could be sent to DataDog so they can be overlaid on appropriate dashboards and correlated with changes.


## Production Considerations

This sample development environment demonstration is rather basic and would at least require a handful of considerations before promoted to a production environment:

- Basic production considerations (DNS, SSL cert, desired cloud provider, CDN, budget constraints, etc.)
- Secure/harden machines and software
- Load/stability testing
- Design an implement a deployment pipeline (or more likely integrate with an existing one)

## Next Steps

Architecturally, a next step for production readiness is to split out nginx onto its own instance (possibly multiple behind round robin DNS if high availability is desired) and use it to load balance to multiple upstream web instances over TCP. Depending on the scale, Postgres connections may become a limiting factor and `pgbouncer` could be investigated for connection pooling between the web instances and the database. Putting the load balancer behind an elastic IP should also be considered.

Other thoughts:

- Evalulate pros and cons of using containers
- Evaluate producing deployable artifacts using Packer to reduce environment build time and for future integration with a CI pipeline
- Assess availability requirements, failover plan, and backup strategy

## Thanks

It was a pleasure to work on this challenge. Thanks for taking the time to evaluate my work!
