# Setup Docker and minikube on Debian with Ansible

> Tested on Debian 12 (Bookworm)

## Scope

**TL;DR** you might want to skip to [Pre-requisites](#pre-requisites) or [Running the playbooks](#running-the-playbooks).

It's not too difficult to install Docker on Debian.

However, installing minikube was a bit more of a hassle. The [minikube start](https://minikube.sigs.k8s.io/docs/start/) documentation page tells you how to install minikube using a Debian package, but doesn't explicitly tell you that you have a number of dependencies to install in order to start your cluster.

Namely:

- `conntrack` (which you can install with `sudo apt-get install -y conntrack`)
- `crictl`
- `cri-dockerd`
- CNI plugins

Some of these tools' setup instructions are not quite up-to-date and/or accurate.

After struggling a bit with them, I thought it'd be a good idea to automate their installation with Ansible.

## Pre-requisites

- Have a ready-to-use Debian 12 host (in my case, a VM inside a Proxmox VE host, installed via a home-made "unattended" ISO image)
- If you are to use this:

    - user `debian` has been created during the install process (and is used as `ansible_user` in inventory)
    - target host's IP is `192.168.1.106` (`ansible_host` in inventory)
- Install `sudo`
- Add user `debian` to group `sudo` (run as `root`): `usermod -aG sudo debian`
- Get the host's IP : `ip route`
- Rename VM : `sudo hostnamectl hostname debian-minikube`

## Running the playbooks

### Edit `inventory.ini`

You'll probably want to edit at least these values in `inventory.ini`:

- `debian_user` (I used the `debian` username in my setup)
- `debian_host` (replace `192.168.1.106` with your target host's IP)

### Run the Docker setup playbook

```
ansible-playbook -i inventory.ini playbook-docker.yml -bkK
```

## Run the minikube setup playbook

```
ansible-playbook -i inventory.ini playbook-minikube.yml -bkK
```

## SSH to host and run minikube

Again, use your own host's IP.

```
ssh debian@192.168.1.106
sudo minikube start --driver=none
```