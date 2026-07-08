## AndrewGodGivens /ansible_patroni_cluster
============

An Ansible role which installs and configures ansible postgres cluster with patroni on Linux

============

## Requirements

Ansible 2.8+

============

## Role Variables

You can see all vars in defaults/main.yml vars file.

Set correct variables `cluster_host_ip`, `cluster_peer1_ip`, `cluster_peer2_ip`, `patroni_name`, `consul_host`.

## Example configuration

### Single-node cluster
Set variable `consul_docker_command`
```yaml
consul_docker_command: "agent -server -advertise {{ cluster_host_ip }} -bootstrap-expect 1 -client 0.0.0.0"
```
Deploy to single node.

### Three-node cluster
#### Deploy to All Nodes Concurrently
Use default `consul_docker_command`. [Set correct variables](#role-variables).

#### One-by-One Node Deployment

In case of rapid node addition, the process consists of three stages:
1. Set `consul_docker_command: "agent -server -advertise {{ cluster_host_ip }} -bootstrap -client 0.0.0.0"` on Node 1 and deploy to Node 1.
2. Set `consul_docker_command: "agent -server -advertise {{ cluster_host_ip }} -client 0.0.0.0  -retry-join {{ cluster_peer1_ip }} -retry-join {{ cluster_peer2_ip }}"` on Node 2 and deploy to Node 2.
3. Revert `consul_docker_command` to default on all 3 nodes and deploy to all simultaneously.

If a long time passes before adding new nodes:
1. `consul_docker_command: "agent -server -advertise {{ cluster_host_ip }} -bootstrap-expect 1 -client 0.0.0.0"` for Node 1.
2. `consul_docker_command: "agent -server -advertise {{ cluster_host_ip }} -bootstrap-expect 2 -client 0.0.0.0" -retry-join {{ cluster_peer1_ip }}` for Node 1 and Node 2 and deploy to both simultaneously. Be careful with the `cluster_peer1_ip` variable.
3. Revert `consul_docker_command` to default on all 3 nodes and deploy to all simultaneously.

### Redeploy a Single Node in a Cluster

If IP address don't change, only deploy to new node.

If IP is changed, you should change the variable `cluster_peer1_ip` or `cluster_peer2_ip` for all nodes, and `cluster_host_ip` for new node, and deploy to all nodes.

### Removing a node from the cluster

Change the variable `consul_docker_command: "agent -server -advertise {{ cluster_host_ip }} -bootstrap-expect 2 -client 0.0.0.0" -retry-join {{ cluster_peer1_ip }}` for Node 1 and Node 2 and deploy to both simultaneously. Be careful with the `cluster_peer1_ip` variable.

### Parallel deployment: 6 nodes, 2 clusters (3 nodes each), single run

There's nothing to change, it all depends on the variables. Just run the playbook.

### Install patroni multi cluster to the same nodes

If you want to deploy multiple Patroni clusters on a single node, you need to use the patroni_clusters list variable. See `default/main.yml` for details. Variables that must differ between clusters are marked with `# need to change` in `patroni_cluster_defaults`.
The same Consul instance is used.

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
