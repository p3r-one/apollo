# What is apollo?

apollo builds k3s clusters from scratch.

It's Ansible-based and allows you to deploy Kubernetes clusters on any machine with a supported OS (Ubuntu, Debian, Centos) and architecture (x64, armhf, arm64) that you can access via SSH.

Depending on the provider of your machines (bare-metal or a supported cloud-provider), additional enhancements like CSI-integration are available.

[!["Version"](https://img.shields.io/github/v/tag/wearep3r/apollo?label=version)](https://github.com/wearep3r/apollo)
[!["p3r. Slack"](https://img.shields.io/badge/slack-@wearep3r/general-purple.svg?logo=slack&label=Slack)](https://join.slack.com/t/wearep3r/shared_invite/zt-d9ao21f9-pb70o46~82P~gxDTNy_JWw)
[!["GitLab Stars"](https://img.shields.io/badge/dynamic/json?color=orange&label=GitLab%20stars&query=%24.star_count&url=https%3A%2F%2Fgitlab.com%2Fapi%2Fv4%2Fprojects%2F17046783)](https://gitlab.com/p3r.one/apollo)
[!["GitHub Stars"](https://img.shields.io/github/stars/wearep3r/apollo?logo=github)](https://github.com/wearep3r/apollo)
[!["Docker Image Size"](https://img.shields.io/docker/image-size/wearep3r/apollo?logo=docker&label=Image)](https://hub.docker.com/r/wearep3r/apollo)

# Prerequisites

- ansible >= 2.10
- helm / kubectl

## Install with pip

```bash
pip3 install -r requirements.txt
```

# Get started

## Create your inventory

Your inventory defines the machines you want to build your cluster upon.

Create a directory (the `inventory_dir`) for your new cluster:

```bash
mkdir -p inventory/my-cluster
```

Create an inventory file for your new cluster:

```bash
edit inventory/my-cluster/hosts.yml
```

> NOTE: this must be a valid Ansible inventory; it doesn't matter if it's YAML, JSON or INI format

Setup your nodes:

```bash
# inventory/my-cluster/hosts.yml
all:
  hosts:
    master-0:
      ansible_host: 1.2.3.4
    node-0:
      ansible_host: 4.5.6.7
    node-1:
      ansible_host: 5.6.7.8
  children:
    master:
      hosts:
        master-0:
    node:
      hosts:
        node-0:        
        node-1:
    k3s_cluster:
      children:
        master:
        node:
```

> NOTE: this is pure Ansible using the defaults. Depending on your setup your inventory might look more complex or you'll need to specify additional configuration for the SSH user or a specific SSH key file to use.

## Create your configuration files

In your `inventory_dir`, create a vars-file:

```bash
edit inventory/my-cluster/vars.yml
```

Setup your cluster configuration:

```bash
# inventory/my-cluster/vars.yml
ansible_ssh_user: root
csi:
  provider: longhorn
```

## Run apollo

apollo consumes an inventory and optional configuration to build a k3s cluster:

```bash
ansible-playbook provision.yml -e @inventory/default-cluster/vars.yml -i inventory/default-cluster/hosts.yml --flush-cache
```

Upon completion, apollo will save your Kubernetes credentials in `$inventory_dir/kubeconfig.yml`. Use this file to connect to the cluster.

# Configuration

Default configuration options can be found in `defaults.yml`. This file will be loaded by `provision.yml`. Additional configuration options can be set in multiple ways:

- changing the value in `defaults.yml`
- creating an additional configuration file (e.g. `staging.yml`) containing your configuration and feeding it to ansible when executing the playbook: `ansible-playbook provision.yml -e @inventory/default-cluster/vars.yml -i inventory/default-cluster/staging.yml --flush-cache`
- setting environment variables (see table below) before running the playbook
- overwriting variables directly in the playbooks or roles
- injecting configuration by using ansible's `--extra-vars` flag: `ansible-playbook provision.yml -e @inventory/default-cluster/vars.yml -i inventory/default-cluster/hosts.yml -e "k3s_version=v1.20.2+k3s1"`

> NOTE: sensitive configuration options like secrets and credentials that don't have a default need to be set before executing the playbook - either via EnvVar or as an Ansible extra-var. The playbook fails when these are not set correctly. You need to specifiy these configs **EVERY TIME** you run the playbook - if you change values for a configuration option Ansible will reflect this by changing the data saved to Kubernetes. Make sure to keep configuration scoped and available each time you run the playbook or you might get unxepected results

## Configuration options

| Configuration Option        | Description | Environment Variable           | Default  |
| ------------- |:-------------:| -----:|-----:|
| `ansible_user` | The user Ansible should use to connect to the inventory hosts | `ANSIBLE_USER` | `root` |
| `k3s_extra_agent_args` | Additional arguments for the k3s agent |    `K3S_EXTRA_AGENT_ARGS` | `` |
| `k3s_extra_server_args` | Additional arguments for the k3s server |    `K3S_EXTRA_SERVER_ARGS` | `--disable traefik` |
| `k3s_version` | k3s version |    `K3S_VERSION` | `v1.20.2+k3s1` |
| `provider.name` | The name of the provider of the inventory hosts |    `PROVIDER_NAME` | `generic` |
| `provider.csi` | Enable/Disable CSI integration for this provider |    `PROVIDER_CSI` | `False` |
| `systemd_dir` | The directory on the remote nodes where k3s service files should go to      |    `SYSTEM_DIR` | `/etc/systemd/system` |

# Issues / Troubleshooting

- `+[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called. We cannot safely call it or ignore it in the fork() child process. Crashing instead.`:  https://stackoverflow.com/questions/50168647/multiprocessing-causes-python-to-crash-and-gives-an-error-may-have-been-in-progr

# Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull/merge requests to us. This software is primarily developed and maintained on [GitLab](https://gitlab.com/p3r.one/apollo).

# Versioning

We use SemVer for versioning, automated by [shipmate](https://gitlab.com/peter.saarland/shipmate). For the versions available, see the tags on this repository.

# Authors

- Fabian Peter

See also the list of contributors who participated in this project.

# License

apollo is [fair-code](http://faircode.io/) licensed under [Apache 2.0 with Commons Clause](LICENSE.).

Additional information about license can be found in the FAQ.

Which license does n8n use?
n8n is fair-code licensed under Apache 2.0 with Commons Clause

## Is apollo Open-Source

No. The Commons Clause that is attached to the Apache 2.0 license takes away some rights. Hence, according to the definition of the Open Source Initiative (OSI), apollo is not open-source. Nonetheless, the source code is open and everyone (individuals and companies) can use it for free. However, it is not allowed to make money directly with apollo.

For instance, one cannot charge others to host or support apollo. However, to make things simpler, we grant everyone (individuals and companies) the right to offer consulting or support without prior permission as long as it is less than 20,000 EUR (€20k) per annum. If your revenue from services based on apollo is greater than €20k per annum, we'd invite you to become a partner and apply for a license. If you have any questions about this, feel free to reach out to us at info@p3r.one.

## Why is apollo not open-source but fair-code licensed instead

We love open-source and the idea that everybody can freely use and extend what we wrote. Our community is at the heart of everything that we do and we understand that people who contribute to a project are the main drivers that push a project forward. So to make sure that the project continues to evolve and stay alive in the longer run, we decided to attach the Commons Clause. This ensures that no other person or company can make money directly with apollo. Especially if it competes with how we plan to finance our further development. For the greater majority of the people, it will not make any difference at all. At the same time, it protects the project.

As apollo itself depends on and uses a lot of other open-source projects, it is only fair that we support them back. That is why we have planned to contribute a certain percentage of revenue/profit every month to these projects.

We have already started with the first monthly contributions via Open Collective. It is not much yet, but we hope to be able to ramp that up substantially over time.

## Disclaimer

This software is maintained and commercially supported by [p3r.](https://www.p3r.one). You can reach us here:

- [Web](https://www.p3r.one)
- [Slack](https://join.slack.com/t/wearep3r/shared_invite/zt-d9ao21f9-pb70o46~82P~gxDTNy_JWw)
- [GitLab](https://gitlab.com/p3r.one)
- [GitHub](https://github.com/wearep3r/)
- [LinkedIn](https://www.linkedin.com/company/wearep3r)

