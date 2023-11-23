## AndrewGodGivens /ansible_patroni_cluster
============

An Ansible role which installs and configures ansible postgres cluster with patroni on Linux

============

## Requirements

Ansible 2.8+

============

## Role Variables

You can see all vars in defaults/main.yml vars file.

## Example Playbook

```yaml
---
- name: Install and configure wal-g
  hosts: patroni_cluster
  remote_user: root
  become: yes

  roles:
    - ansible_patroni_cluster

```
