---
{"dg-publish":true,"dg-path":"Notes/homelab","permalink":"/notes/homelab/","tags":["notes/fern"],"noteIcon":"fern","created":"2025-05-06 20:12","updated":"2025-05-21 22:46"}
---

## Basics

Maintaining a HomeLab can be a rewarding, but time‑consuming hobby - especially once the number of services, containers and virtual machines starts to grow. Early on I found myself repeating the same manual setup steps on every new host and constantly tweaking configuration files directly on production machines. That’s when I decided to embrace Infrastructure as Code (IaC) and let Ansible handle the heavy lifting for me.

Here is a image that should explain the basic setup for my HomeLab:
![Network View.png](/img/user/Media/Network%20View.png)
I recently switched from a basic port forwarding solution to pangolin as my self hosted tunnel/proxy provider.
## The Repo

You can browse the entire setup here -> [Ferdyverse/HomeLab on Github](https://github.com/Ferdyverse/HomeLab) 
The repository is organised like this (simplified):
```
HomeLab/
├── group_vars/
│   ├── all/
│   └── proxmox_nodes/
├── handlers/
├── inventorys/  # inventory files (yes, the repo spells it this way)
├── roles/
│   ├── adguard_home/
│   ├── caddy/
│   ├── grafana/
│   ├── ntfy/
│   ├── pve/
│   ├── semaphore/
│   └── ... (other service roles)
├── tasks/
├── templates/
├── ansible.cfg
├── container_configure.yml
├── container_create.yml
├── nodes_configure.yml
├── requirements.yml  # Ansible Galaxy collections
├── requirements.txt  # Python dependencies
└── setup_environment.sh
```
## Settings
Below are some defaults I rely on throughout the lab. Sticking to a convention means every new service knows exactly where to mount volumes and look for its configuration.
### Docker
For my docker setup I currently use many lxc containers. I thought about switching to Docker Swarm, but I do not see any benefits for my setup because I only have 2 Proxmox nodes.

On each of my docker nodes I have the following storage setup:

| Path                              | Purpose                                                                                          |
| --------------------------------- | ------------------------------------------------------------------------------------------------ |
| `/home/{{ default_user }}/docker` | **Docker Compose files** – One folder per stack or service.                                      |
| `/docker`                         | **Configuration files** – Static configs such as `traefik.yml`, `grafana.ini`, `promtail.yml`, … |
| `/data`                           | **Persistent data** – Bind mounts for databases, Prometheus TSDB, MinIO buckets, etc.            |

> **Note 📝**  All directories are created and owned by **UID `1000`** (the primary unprivileged user on my nodes). This avoids permission headaches when containers drop root privileges.

## Secrets
For my secret management I use 1Password with a special HomeLab-Vault. I recently discovered the possibility to use the [1Password-Connect Stack](https://developer.1password.com/docs/connect/) to be able to use my secrets via a REST-API in Ansible and other services. This stack consists out of two main components . The Connect-API and the Connect-Sync container.

```yaml
services:
  op-connect-api:
    image: 1password/connect-api:latest
    restart: unless-stopped
    volumes:
      - '{{ docker_container_data_folder }}/onepassword/data:/home/opuser/.op/data'
      - './token.json:/home/opuser/.op/1password-credentials.json'
    ports:
      - 8085:8080

  op-connect-sync:
    image: 1password/connect-sync:latest
    restart: unless-stopped
    volumes:
      - '{{ docker_container_data_folder }}/onepassword/data:/home/opuser/.op/data'
      - './token.json:/home/opuser/.op/1password-credentials.json'
```

## CI/CD

I mainly use a Forgejo runner for my pipelines. There I configured a pipeline called `autoupdate`.

The `autoupdate` pipeline runs automatically on each push and checks for certain file changes. Based on the found changes it updates some of my HomeLab services.

```yaml
name: update-homelab-services

on:
  push:
    branches: [ main ]
    paths:
      - '**/roles/glance/files/**'
      - '**/roles/glance/templates/**'
      - '**/roles/caddy/vars/main.yml'
      - '**/roles/caddy/templates/**'
      - '**/roles/ntfy/defaults/main.yml'
      - '**/roles/pve/vars/main.yml'
  workflow_dispatch:
    inputs:
      service:
        description: 'Service to update'
        required: false
        default: auto
        type: choice
        options: [auto, caddy, glance, ntfy, pve, all]

jobs:
  update:
    runs-on: ubuntu-ansible
    container:
      env:
        ANSIBLE_STRATEGY: serverscom.mitogen.mitogen_linear
        OP_CONNECT_HOST: ${{ vars.OP_CONNECT_HOST }}
        OP_CONNECT_TOKEN: ${{ secrets.OP_CONNECT_TOKEN }}
    steps:
      - name: Checkout latest version
        uses: actions/checkout@v4
      - name: Filter changed files
        id: filter
        uses: https://github.com/dorny/paths-filter@v3
        with:
          filters: |
            glance:
              - 'roles/glance/files/**'
              - 'roles/glance/templates/**'
            caddy:
              - 'roles/caddy/vars/main.yml'
              - 'roles/caddy/templates/**'
            ntfy:
              - 'roles/ntfy/defaults/main.yml'
            pve:
              - 'roles/pve/vars/main.yml'
      - name: Prepare Ansible
        run: ansible-galaxy install -r requirements.yml
      - name: Update Glance Pages
        if: |
          (steps.filter.outputs.glance == 'true' && github.event_name != 'workflow_dispatch') ||
          (github.event_name == 'workflow_dispatch' &&
          (github.event.inputs.service == 'glance' ||
            github.event.inputs.service == 'all'))
        run: >
          ansible-playbook -i inventorys
          --limit glance
          --skip-tags user,lxc_config.ssh,glance.setup
          container_configure.yml
      - name: Update Caddy vhosts
        if: |
          (steps.filter.outputs.caddy == 'true' && github.event_name != 'workflow_dispatch') ||
          (github.event_name == 'workflow_dispatch' &&
          (github.event.inputs.service == 'caddy' ||
            github.event.inputs.service == 'all'))
        run: >
          ansible-playbook -i inventorys
          --limit caddy
          --skip-tags user,lxc_config.ssh,caddy.setup,caddy.alloy
          container_configure.yml
      - name: Update ntfy users
        if: |
          (steps.filter.outputs.ntfy == 'true' && github.event_name != 'workflow_dispatch') ||
          (github.event_name == 'workflow_dispatch' &&
          (github.event.inputs.service == 'ntfy' ||
            github.event.inputs.service == 'all'))
        run: >
          ansible-playbook -i inventorys
          --limit ntfy
          --skip-tags user,lxc_config.ssh,ntfy.setup
          container_configure.yml
      - name: Create new container
        if: |
          (steps.filter.outputs.pve == 'true' && github.event_name != 'workflow_dispatch') ||
          (github.event_name == 'workflow_dispatch' &&
          (github.event.inputs.service == 'pve' ||
            github.event.inputs.service == 'all'))
        run: >
          ansible-playbook -i inventorys
          --skip-tags update_lxc
          container_create.yml
      - name: Send notification to Discord
        uses: https://github.com/sarisia/actions-status-discord@v1
        if: always()
        with:
          webhook: ${{ secrets.DISCORD_WEBHOOK }}
          status: ${{ job.status }}
          title: "Auto Update"
          description: "Automatic HomeLab Updates"
          url: "https://git.berger-em.net/bergefe/HomeLab"
          username: Forgejo Actions
          
```

To run this pipeline I had to build my own docker container image. This image includes Ansible, NodeJS, 1Password-CLI and Mitogen. Here you can get the [Dockerfile](https://git.berger-em.net/bergefe/docker-images/src/branch/main/forgejo/ubuntu/24.04-ansible/Dockerfile) to create your own version.

## Future

I am still working on this topic and I try to add more features. The goal for this project is to be able to maintain and update my HomeLab with a single Ansible-Playbook, In the best case, most of the tasks will run on its own and just send me some notification when something went wrong.
