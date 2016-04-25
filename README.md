# AltspaceVR Project - TodoMVC Operations


## Overview

This is a sample Django TodoMVC application running in a fairly simple Ubuntu 14.04 environment.

Nginx is used as a reverse proxy for gunicorn which runs the python app in a virtualenv. Supervisor is used to monitor and control gunicorn and Postgresql is used for persistence.

Currently, one machine is provisioned for the web server and another for the database. To be kind to aging Macbooks, the Vagrant Virtualbox VMs are given minimal memory resources.

The instructions below were written with MacOS in mind, but could be adapted to other platforms.

### Design Decisions / Principles

Priority was given to keeping moving parts to a minimum and embracing reliable tools in order to deliver a working demonstration within a reasonable amount of time. The project attempts to be well organized and follow best practices wherever possible. Maintaining parity between local and remote environments is demonstrated with the intention of maintaining parity between dev/test/production environments.

Ansible's ecosystem is quite helpful and permitted playbook reuse. Ansible Galaxy was not directly used in favor of flexibility and customization.

It seems quite reasonable that this basic setup could eventually evolve into a containerized version, but the additional complexity was avoided for this project.

### Provisioning and Configuration

Ansible is used to provision and configure the resources for the TodoMVC app. Dynamic Ansible inventories are utilized and both Vagrant and Digital Ocean were tested as providers. Extending to AWS and other providers would be straightforward.

## Getting Started

1. Verify the requirements below are installed on your local development machine.

2. Clone the project and start the environment with a simple `vagrant up`.

```
git clone git@github.com:mparrett/altspacevr-project-todomvc-ops.git 
cd altvr-project-todomvc-ops/Project
```

```
vagrant up
```

Once Vagrant provisions and configures the machines, visit [https://192.168.33.10](https://192.168.33.10) and enjoy some TODOs.

Note: When running playbooks, Ansible needs to find `ansible.cfg` in the current directory. As a result, you'll need to be in `altvr-project-todomvc-ops` when running them.

### Install Requirements

- Python 2.7.x and pip: `easy_install pip`
- Ansible 1.9+ `sudo pip install ansible`
- Virtualbox (Local) `brew cask install virtualbox` (requires homebrew @ [http://brew.sh/](http://brew.sh/))
- Vagrant (Local) `https://www.vagrantup.com/downloads.html`

These instructions *should* work with the MacOS system python but you may run into complications. If so, it is recommended to use `brew install python` and follow the instructions before installing anything via `pip`.

#### Verify Requirements

Check your software versions:

- `python --version`
- `pip --version`
- `vagrant --version`
- `which VBoxManage`

### Extra Requirements for Digital Ocean

Running on Digital Ocean requires a few extra steps to enable the dynamic inventory script `digital_ocean.py` to access your account. Additionally, if Datadog is enabled via ansible variables, you'll also need to export your Datadog API key before running the playbook.

```
sudo pip install 'dopy>=0.3.5,<=0.3.5'
export DO_API_TOKEN=YOUR_DIGITALOCEAN_TOKEN
export DD_API_KEY=YOUR_DATADOG_KEY
ansible-playbook -i digital_ocean.py digitalocean.yml --user YOUR_USER --extra-vars 'digitalocean_ssh_key_ids=123'
```

You can access this TodoMVC app hosted on my Digital Ocean account here:

[https://107.170.252.197/](https://107.170.252.197/)

## Code Deployment

A simple git-based deployment process is used to deploy new code during development. The repository is configured via ansible variables and can be deployed via `ansible-playbook ansible/vagrant.yml --tags=deploy`. By default this deploys the `master` branch but can be altered via ansible variables. During deployment, the python virtualenv is re-created and necessary dependencies are installed via `pip`.

The sample Django app provided with the challenge is here on [GitHub](https://github.com/mparrett/altvr-todomvc-django) and was slightly modified to meet the requirements.

## Monitoring with Datadog

The project demonstrates basic Datadog integration for monitoring and alerting.

![](http://i.imgur.com/nuAGoDa.png)

### Integrations

Included in the demo are Nginx, Gunicorn, and Postgresql integrations which include many interesting and useful metrics. In addition, gunicorn is configured to speak via StatsD to the local Datadog agent.

### Alerts

A sample alert was created which informs us if the gunicorn worker count drops below the configured amount (3). This could be connected to PagerDuty and Slack. Other examples of useful alerts include watching for non-zero back-end error rates, abnormally high or low request rates, and so forth.

![](http://i.imgur.com/JZSjm5W.png)

### Events

Idea: Deployment events could be sent to DataDog so they can be overlaid on appropriate dashboards and correlated with changes.

## Limitations

Some external dependencies are usually reliable but could potentially cause issues, such as Vagrant boxes, pip, and Ubuntu package repositories. 

## Production Considerations

This sample development environment demonstration is rather basic and would at least require a handful of considerations before promoted to a production environment:

- Usual production considerations (DNS, SSL cert, desired cloud provider, CDN, budget constraints, etc.)
- Secure/harden machines and software
- Load/stability testing
- System and software tuning
- Log aggregation
- Clock synchronization
- Design an implement a deployment pipeline (or more likely integrate with an existing one)

## Next Steps

One next step for production readiness is to split out nginx onto its own instance (possibly multiple instances behind round robin DNS) and use it to load balance to multiple upstream web instances over TCP. Depending on the scale, Postgres connections may become a limiting factor and `pgbouncer` could be investigated for connection pooling between the web instances and the database. Putting the load balancer behind an elastic IP should also be considered. It might eventually be appropriate to give consideration to caching and task queues.

Other thoughts:

- Evaluate producing deployable artifacts using Packer to reduce environment build time and for future integration with a CI pipeline
- Evalulate pros and cons of using containers
- Assess availability requirements, failover plan, and backup strategy
- Evaluate using CoreOS

## Thanks

It was a pleasure to work on this challenge. Thanks for taking the time to evaluate my work!
