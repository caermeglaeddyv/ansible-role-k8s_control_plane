Ansible role: Kubernetes Control Plane
=========

This role is used to prepare control plane nodes for kubernetes cluster initialization, as well as for joining other control-plane nodes to existing cluster

For now, it does the following:
- copies etcd certificates to every master
- configures apiserver audit if needed
- configures encryption of secret data at rest if needed
- initializes the cluster on first control plane node and copies admin.conf to localhost
- joins other control plane nodes to the cluster
- installs initial pod security policy configurations if psp admission plugin is in the list of admission plugins


Requirements
------------

This is not strict requirements and it may not work with other versions than tested ones.
Anyway. feel free to test by yourself, suggest addition of new functionality and contribute.

Role is tested with:
- Ansible version >= 2.8.6
- CentOS version >= 7.6 (1803)

Currently supports installation of Kubernetes versions 1.14.1-1.15.10.

caermeglaeddyv.ansible_role_prepare_k8s_cluster role which must be run manually before this one

Kubectl is required on localhost to do management of remote cluster after initialization.

VIP address for api-server must be added to your dns or hosts file with same name as used for cluster initiation ("kubernetes_apiserver_host" variable ), if name used instead of IP.


Role Variables
--------------

Variables and their descriptions copied from defaults/main.yml

```bash

# Variable which is common for most projects, used in
# configuration files or file/directory names.
# By default used as reference for kubernetes_project_dir variable:
kubernetes_project_name: test

# Variable which is common for most projects, used as
# project working directory on the localhost for the role.
# Currently is used for copying etcd certificates to control plane nodes,
# to download admin kube config to localhost and install initial psp if needed:
kubernetes_project_dir: files/{{ kubernetes_project_name }}

# Version of Kubernetes cluster which will be bootstrapped:
kubernetes_version: 1.14.10

# IF to enable apiserver audit or not:
kubernetes_apiserver_audit: false

# If to enable secret encryption at rest or not:
kubernetes_esdar_key:          # Ckcp7YnRWbfta8toDYkjD3jLB08Crgct6oFlrLMZ82k=

# DNS name or IP address which will be listening for apiserver requests,
# this must equal to virtual IP to work correct in HA setup:
kubernetes_apiserver_host: ""

# Port which will listen for apiserser requests on apiserver host:
kubernetes_apiserver_port:

# List of external etcd DNS names or IP addresses to use as backend for apiserver:
kubernetes_external_etcds: "{{ groups.etcdcluster }}"

# List of admission controllers to be enabled in cluster:
kubernetes_admission_plugins:
- NodeRestriction
- AlwaysPullImages
- NamespaceLifecycle
- LimitRanger
- ServiceAccount
- DefaultStorageClass
- DefaultTolerationSeconds
- MutatingAdmissionWebhook
- ValidatingAdmissionWebhook
- ResourceQuota

# Minimum version of TLS for control-plane components, usually
# this must be the same as for load balancer as well as for etcd:
kubernetes_tls_min_version: VersionTLS12

# TLS cipher suites to use for control-plane components, usually
# this must be the same as for load balancer as well as for etcd:
kubernetes_tls_cipher_suites:
- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
- TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
- TLS_RSA_WITH_AES_128_GCM_SHA256
- TLS_RSA_WITH_AES_256_GCM_SHA384

# Token which will be used in cluster initialization, so
# other nodes can join the cluster using it:
kubernetes_bootstrap_token: ""          # 123456.abcdefghijklmnop

# Key used in cluster initialization, so
# other control-plane nodes can join the cluster using it:
kubernetes_certificate_key: ""          # 1a2b3c4d5e6f7g8h9i10j11k12l13m14n15o16p17q18r19s20t21u22v23w24xy

# List of preflight errors which must be ignored while initializing
# cluster or joining node to existing cluster via kubeadm:
kubernetes_ignore_preflight_errors:
# - NumCPU

# Localhost directory used to keep k8s manifests and other role files:
kubernetes_local_dir: files

```


Dependencies
------------

"caermeglaeddyv.ansible_role_prepare_k8s_cluster" role which must be run manually before this one

Kubectl is required on localhost to do management of remote cluster after initialization.

VIP address for api-server must be added to your dns or hosts file with same name as used for cluster initiation ("kubernetes_apiserver_host" variable ), if name used instead of IP.


Example Playbook
----------------

```bash
---
- hosts: localhost
  gather_facts: false
  become: no
  tasks:
  - name: Check ansible version >=2.8.6
    assert:
      msg: Ansible must be v2.8.6 or higher
      that:
      - ansible_version.string is version("2.8.6", ">=")
    tags:
    - check
  vars:
    ansible_connection: local

- hosts: all
  become: yes
  tasks:
  - import_role:
      name: caermeglaeddyv.ansible_role_prepare_k8s_cluster
  - import_role:
      name: caermeglaeddyv.ansible_role_k8s_control_plane

```

More detailed examples ( inventories, playbooks etc. ) of this and other roles can be found [here](https://github.com/caermeglaeddyv/examples/tree/dev/ansible).

It's highly recommended to start you test deploys from there, especially if you use Google Cloud Platform or VMware vCenter as your infrastructure, for now that [repository](https://github.com/caermeglaeddyv/examples) contains [Packer](https://github.com/caermeglaeddyv/examples/tree/dev/packer) and [Terraform](https://github.com/caermeglaeddyv/examples/tree/dev/terraform) examples to build templates and deploy machines on this platforms.


License
-------

[Apache 2.0](https://github.com/caermeglaeddyv/ansible-role-k8s_control_plane/blob/dev/LICENSE)


Author Information
------------------

Copyright 2020 [caermeglaeddyv](https://github.com/caermeglaeddyv)
