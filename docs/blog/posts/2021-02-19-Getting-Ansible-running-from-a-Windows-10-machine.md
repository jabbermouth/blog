---
title: Getting Ansible running from a Windows 10 machine
description: A quick guide to getting Ansible up and running on a Windows 10 machine.
date: 2021-02-19
categories:
  - Ansible
  - Tutorials
---
# Getting Ansible running from a Windows 10 machine

From the Microsoft Store, install the latest version of Ubuntu (just follow the instructions on screen) and then apply any updates using the following:

```bash
sudo apt update
sudo apt upgrade -y
```

And then, to install Ansible itself, run the following commands:

```bash
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

You can then confirm it's installed by running the following command:

```bash
ansible --version
```