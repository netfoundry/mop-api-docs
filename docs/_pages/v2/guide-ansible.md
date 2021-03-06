---
permalink: /guides/ansible/
redirect_from:
  - /v2/guides/ansible/
title: "Ansible Guide"
sidebar:
    nav: v2guides
toc: false
classes: wide
---

You may install our [Ansible Galaxy Collection](https://galaxy.ansible.com/netfoundry/platform). Which includes modules like `netfoundry_info` for creating a session and discovering Networks.

```bash
# install collection
ansible-galaxy collection install netfoundry.platform
# read about the info module
ansible-doc netfoundry.platform.netfoundry_info
# read about the Endpoint module
ansible-doc netfoundry.platform.netfoundry_endpoint
# read about the Service module
ansible-doc netfoundry.platform.netfoundry_service
# read about the AppWAN module
ansible-doc netfoundry.platform.netfoundry_appwan
# read about the Router module
ansible-doc netfoundry.platform.netfoundry_router
# read about the Router Policy module
ansible-doc netfoundry.platform.netfoundry_router_policy
```

For more examples please see [the playbook included in the collection](https://github.com/netfoundry/developer-tools/blob/master/ansible_collections/netfoundry/platform/playbooks/example_playbook.yml). The default install path for this file is ~/.ansible/collections/ansible_collections/netfoundry/platform/playbooks/example_playbook.yml.

Here are some working examples to demonstrate a variety of methods for looping over lists of Endpoints, Services, and Edge Routers.

```yaml
{% raw %}
- hosts: localhost
  collections:
  - netfoundry.platform
  - community.general

- hosts: localhost

  collections:
  - netfoundry.platform

  tasks:
  - name: Establish session and find Network resources
    netfoundry_info:
      network: BibbidiBobbidiBoo
      credentials: credentials.json
    register: netfoundry_info

  - name: block and wait for each hosted Edge Router to have a minimum state of PROVISIONING
    netfoundry_router:
      network: "{{ netfoundry_info.network }}"
      name: "{{ item.name }}"
      attributes: 
      - "#defaultRouters"
      datacenter: "{{ item.datacenter  }}"
      state: PROVISIONING
      wait: 200
    loop: "{{hosted_edge_routers}}"

  - name: wait in the background for each hosted Edge Router to become REGISTERED
    netfoundry_router:
      network: "{{ netfoundry_info.network }}"
      name: "{{ item.name }}"
      attributes: 
      - "#defaultRouters"
      datacenter: "{{ item.datacenter  }}"
      state: REGISTERED
    register: router_results
    async: 1000
    poll: 0
    loop: "{{hosted_edge_routers}}"

  - name: declare Endpoints
    netfoundry_endpoint:
      name: "{{ item }}"
      network: "{{ netfoundry_info.network }}"
      attributes:
      - "#workFromAnywhere"
      dest: /tmp/netfoundry
    loop:
    - Client1
    - Exit1
    register: endpoints

  - name: declare an Endpoint-hosted Service
    netfoundry_service:
      network: "{{ netfoundry_info.network }}"
      name: HTTP Echo 1
      endpoints: 
      - Exit1
      attributes: 
      - "#welcomeWagon"
      clientHostName: echo-exit.netfoundry
      clientPortRange: 80
      serverHostName: eth0.me
      serverPortRange: 80
      serverProtocol: TCP

  - name: host a server port range as an array of Services
    netfoundry_service:
      network: "{{ netfoundry_info.network }}"
      name: "spice-console-{{ item }}"
      attributes: "#spice-consoles"
      clientHostName: 192.168.0.188
      clientPortRange: "{{ item|int }}"
      endpoints: 
      - Exit1
      serverHostName: 192.168.0.188
      serverPortRange: "{{ item|int }}"
      serverProtocol: TCP
    with_sequence: 5900-5919
    register: spice_loop

  - name: Read Services from CSV file
    read_csv:
        path: services.csv
    register: services

  # with CSV headings:
  #  name
  #  attribute
  #  serverHostName
  #  port
  #  clientHostName
  #  egressRouterName
  - name: declare Services from CSV
    netfoundry_service:
      network: "{{ netfoundry_info.network }}"
      name: "{{ item.name }}"
      attributes:
      - "{{ item.attribute }}"
      clientHostName: "{{ item.clientHostName }}"
      clientPortRange: "{{ item.port }}"
      egressRouter: "{{ item.egressRouterName }}"
      serverHostName: "{{ item.serverHostName }}"
      serverPortRange: "{{ item.port }}"
      serverProtocol: TCP
    loop: "{{ services.list }}"

  - name: declare a blanket Edge Router Policy
    netfoundry_router_policy:
      network: "{{ netfoundry_info.network }}"
      name: defaultRouters
      routers:
      - "#defaultRouters"
      endpoints:
      - "#all"
    register: blanket_policy

  - name: declare an AppWAN
    netfoundry_appwan:
      network: "{{ netfoundry_info.network }}"
      name: Welcome
      endpoints:
      - "#workFromAnywhere"
      services:
      - "#welcomeWagon"

  - name: Block until completion of background tasks
    async_status:
      jid: "{{ item.ansible_job_id }}"
    retries: 100
    delay: 10
    register: async_job_result
    until: "{{ async_job_result.finished }}"
    loop: "{{ router_results.results }}"

  # lastly, do tasks that depend on an async background task
  - name: declare a Router-hosted Service
    netfoundry_service:
      network: "{{ netfoundry_info.network }}"
      name: HTTP Echo 2
      attributes: 
      - "#welcomeWagon"
      clientHostName: echo2.netfoundry
      clientPortRange: 80
      egressRouter: "Oregon {{increment}}"
      serverHostName: eth0.me
      serverPortRange: 80
      serverProtocol: TCP
    
  vars:
    increment: 12
    hosted_edge_routers:
    - name: Oregon {{increment}}
      datacenter: us-west-2
    - name: Ohio {{increment}}
      datacenter: us-east-2
    - name: Virginia {{increment}}
      datacenter: us-east-1
      

{% endraw %}
```
