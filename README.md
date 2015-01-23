# ansible-role-munin-server

Installs munin with apache by default, could be changed to nginx with munin_http_server variable (see defaults file)

If there's apache2 or nginx on port 80 – playbook sets this varible to suite discovered http server.

For nginx:

* just installs config file, doesn't install package itself
* listens on munin.{{ ansible_domain }}

# Usage

Example playbook:

```
---
- hosts: munin-client
  sudo: yes
  roles:
      - munin-client

- hosts: munin-server
  sudo: yes
  roles:
      - munin-server
```

If you do not want to set up client and server with a single playbook you still have to run it on the whole group to gather facts, could be done like this:
##### Server playbook

```
- hosts: munin-server
  sudo: yes
  roles:
      - munin-server

- hosts: munin-client
  name: Gather facts from munin-client
  tasks:
    - fail: msg=""
      when: false
```
##### Client playbook

```
---
- hosts: munin-client
  sudo: yes
  roles:
      - munin-client

- hosts: munin-server
  name: Gather facts from munin-server
  tasks:
    - fail: msg=""
      when: false
```

#### Host defenitions

```
[test-munin-server]
munin.test.com

[test-munin-client]
node1.test.com
node2.test.com
node3.test.com
node4.test.com munin_interface_address=10.10.4.8 munin_server_address=10.10.4.9
# munin_extra_allowed_address defined in host_vars as array defenition work only for groups

[munin-server:children]
test-munin-server

[munin-client:children]
test-munin-client

[test-munin:children]
test-munin-server
test-munin-client

[test-munin:vars]
munin_group_search_prefix=test
munin_interface=tap0
munin_extra_allowed_address=['10.10.0.15','10.12.14.2']

```

Server configuration takes client nodes details from ```{{ munin_group_search_prefix }} + munin-client``` group, in this example it's set to ```test-munin-client```, client nodes take server address from the first host in ```{{ munin_group_search_prefix }} + munin-server``` (```test-munin-server```) group.

If ```munin_interface_address``` is set than this client node won't be checked for ```{{ munin_interface }}``` ip address.

```{{ munin_server_address }}``` overrides server address for a node, affects allow list.

# Warnings

* modification of nodes doesn't lead to reload of munin service on master node
* client nodes autodecetc the first server only (uses [0])
* ```munin__server_interface``` has to be the same on server and client. If you have to override – use ```munin_interface_address``` and ```munin_server_address```
* allowed addresses are hardcoded in apache and nginx:

```
    allow 127.0.0.0/8;
    allow 10.8.0.0/24;
    allow 10.0.2.0/24;
    allow 192.168.121.1;
```

Tested in ansible 1.8.1, ubuntu 12.04