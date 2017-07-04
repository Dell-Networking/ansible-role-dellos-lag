LAG Role for Dell EMC Networking OS
====================================

This role facilitates the configuration of Link Aggregation Group (LAG) attributes. It supports the creation and deletion of a LAG and its member ports.
It also supports the configuration of type (static/dynamic), hash scheme in dellos6 devices, and minimum required link. This role is abstracted for dellos9,dellos6 and dellos10 devices.


Installation
------------

```
ansible-galaxy install Dell-Networking.dellos-lag
```

Requirements
------------

This role requires an SSH connection for connectivity to your Dell EMC Networking device. You can use any of the built-in Dell EMC Networking os connection variables, or the ``provider``
dictionary.

Role Variables
--------------

The object described as follows drives the tasks in this role.
This role is abstracted using the variable ``ansible_net_os_name`` that can take the following values: dellos9, dellos6, dellos10.

Any role variable with a corresponding state variable setting to absent negates the configuration of that variable. 
For variables with no state variable, setting an empty value to the variable negates the corresponding configuration.
The variables and its values are case-sensitive.

``dellos_lag`` (dictionary) holds a dictionary with port channel ID key. The key should be in the format `Po <ID>`. The ID can be in the range 1-4096 for dellos9 devices, 1-128 for dellos10 and dellos6 devices.

``port-channel ID`` holds the following keys:

|        Key | Type                      | Notes                                                                                                                                                                                     |
|------------|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type      | choices : static, dynamic      |Configures the interface either as a static or dynamic LAG.            |  
| min_links | integer                       |Configures the minimum number of links in the LAG that must be in 'operup' status. You can configure minimum links in the range 1-64 on dellos9 devices and 1-32 on dellos10 devices and 1-8 on dellos6 devices.|
| hash | integer |Configures hash value for dellos6 devices in the range 0-7.This key is not supported on dellos9 and dellos10 devices.|
| lacp     | dict |Specifies lacp fast-switchover or long timeout options .This key is not supported on dellos6 and dellos10 devices.|
| lacp.fast_switchover | boolean |Configures fast-switchover option if this key is set to true.|
| lacp.long_timeout | boolean |Configure long-timeout option if this key is set to true.|
| lacp_system_priority | integer |Configure lacp-system priority value that can be specified in the range 1-65535 on dellos9 devices. |
| lacp_ungroup_vlt |boolean | Configures all VLT LACP members to be switchports if this key is set to true. |
| lacp_ungroup | list |Specifies the list of port-channels to become switchports .See the lacp_ungroup.* keys for each list item.|
| lacp_ungroup.port_channel | integer(required) |Specifies valid port-channel number. |
| lacp_ungroup.state |choices: present,absent* |Removes the ungroup association if this key is set to absent. |
| channel_members  | list  |Specifies the list of port members to be associated to the port channel. See the channel_members.* keys for each list item .|
| channel_members.port | string  |Specifies valid os9/os6/os10 interface name to be configured as the port channel member. |
| channel_members.state | string, choices: absent, present |Removes the port member association when set to absent .|
| state  | string, choices: absent, present*           |Deletes the LAG corresponding to the port channel ID when set to absent. |


```
Note: Asterisk (*) denotes the default value if none is specified.
```

Connection Variables
--------------------

Ansible Dell EMC Networking roles require the following connection information to establish 
communication with the nodes in your inventory. This information can exist in
the Ansible group_vars or host_vars directories, or in the playbook itself.

|         Key | Required | Choices    | Description                              |
| ----------: | -------- | ---------- | ---------------------------------------- |
|        host | yes      |            | Hostname or address for connecting to the remote device over the specified ``transport``. The value of this key is the destination address for the transport. |
|        port | no       |            | Port to use when building the connection to the remote device. This value applies to either acceptable value of ``transport``. The value of this defaults to the appropriate transport common port if this key provides none (CLI=22, HTTP=80, HTTPS=443). |
|    username | no       |            | Configures the username that authenticates the connection to the remote device. The value of this key authenticates the CLI login. If this key does not specify the value, the value of environment variable ANSIBLE_NET_USERNAME is used instead. |
|    password | no       |            | Specifies the password that authenticates the connection to the remote device. This is a common argument used for either acceptable value of ``transport``. If the task does not specify the value, the value of environment variable ANSIBLE_NET_PASSWORD is used instead. |
|   authorize | no       | yes, no*   | Instructs the module to enter privileged mode on the remote device before sending any commands. If the task does not specify the value, the value of environment variable ANSIBLE_NET_AUTH_PASS is used instead.If not specified, the device attempts to execute all commands in non-privileged mode. |
|   auth_pass | no       |            | Specifies the password to use if required to enter privileged mode on the remote device. If ``authorize`` is set to no, then this key is not applicable. If the task does nf the task does not specify the value, the value of environment variable ANSIBLE_NET_AUTH_PASS is used instead. |
|   transport | yes      | cli*       | Configures the transport connection to use when connecting to the remote device. This key supports connectivity to the device over CLI (SSH).  |
|    provider | no       |            | Convenient method that passes all of the above connection arguments as a dictionary object. All constraints (such as required or choices) must be met either by individual arguments or values in this dictionary. |

```
Note: Asterisk (*) denotes the default value if none is specified.
```
Dependencies
------------

The Dell-Networking.dell-lag role is built on modules included in the core Ansible code.
These modules were added in Ansible version 2.2.0.


Example Playbook
----------------

The following example use the Dell-Networking.dellos-lag role to setup port channel ID and description. This example also configures hash algorithm and minimum links for the LAG. Channel members can be configured for the port channel either in static or dynamic mode. You can also delete the LAG with the port channel ID or delete the members associated to it. This example creates a ``hosts`` file with our switch, a corresponding ``host_vars`` file, and then a simple playbook that references the dellos-lag role.

Sample hosts file:

    leaf1 ansible_host= <ip_address> ansible_net_os_name= <OS name(dellos9/dellos6/dellos10)>

Sample ``host_vars/leaf1``:
    
    hostname: leaf1
    provider:
      host: "{{ hostname }}"
      username: xxxxx
      password: xxxxx
      authorize: yes
      auth_pass: xxxxx 
      transport: cli

    dellos_lag:
        Po 127:
          type: static
          min_links: 3
          lacp:
            long_timeout: true
            fast_switchover: true
          lacp_system_priority: 1
          lacp_ungroup_vlt: true
          lacp_ungroup:
            - port-channel:1
              state: present
          channel_members:
            - port: fortyGigE 1/4
              state: present
            - port: fortyGigE 1/5
              state: present
          state: present

A simple playbook to setup system, ``leaf.yaml``:

    - hosts: leaf1
      roles:
         - Dell-Networking.dellos-lag

Then run with:

    ansible-playbook -i hosts leaf.yaml

License
--------

Copyright (c) 2016, Dell Inc. All rights reserved.
 
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
 
    http://www.apache.org/licenses/LICENSE-2.0
 
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
