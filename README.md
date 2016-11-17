LAG Role for Dell EMC Networking OS
====================================

This role facilitates the configuration of Link Aggregation Group (LAG) attributes. It supports the creation and deletion of a LAG and its member ports.
It also supports the configuration of type (static/dynamic), hash scheme, and minimum required link. This role is abstracted for OS6, OS9, and OS10.


Installation
------------

```
ansible-galaxy install Dell-Networking.dellos-lag
```

Requirements
------------

Requires an SSH connection for connectivity to your Dell EMC Networking OS device. You can use any of the built-in dellos connection variables, or the ``provider``
dictionary.

Role Variables
--------------

The object described as follows drives the tasks in this role.
``dellos_lag``(dictionary) contains the hostname (dictionary).
The hostname is the value of the variable ``hostname`` that corresponds to the name of the OS device.
This role is abstracted using the variable ``ansible_net_os_name`` that can take the following values: dellos6, dellos9 and dellos10.

Any role variable with corresponding state variable setting to *absent* negates the configuration of that variable. 
For variables with no state variable, setting empty value to the variable negates the corresponding configuration.
The variables and its values are **case-sensitive**.

The hostname (dictionary) holds a dictionary with the port channel ID. Port channel ID can be in the range 1-4096 for OS9 devices.

**port-channel id** (dictionary)  holds the following keys:

|        Key | Type                      | Notes                                                                                                                                                                                     |
|------------|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type      | choices : static,dynamic      |Configures the interface either as a static or dynamic LAG.            |  
| min_links | integer                       |Configures the minimum number of links in the LAG that must be in 'operup' status. Minimum links can be configured to any value in the range 1-64 on OS9 device and 1-32 on OS10 devices.|
| hash      | string   |Configures the hash scheme according to the choices on the OS9 device. OS9: crc16, crc16cc, crc32LSB, crc32MSB, xor1, xor2, xor4, xor8, xor16 ; OS10- crc, xor, random; OS6: hashing mode ranges from 1-7. |
| channel_members  | list  |Specifies the list of port members to be associated to the port channel.|
| state  | string, choices: absent, present*           |Deletes the LAG corresponding to the port channel ID when the setting is *absent*. |
| members_state   | string, choices: absent, present*       |Delete the members of the channel when this setting is *absent*.  |


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
|        host | yes      |            | The host name or address for connecting to the remote device over the specified *transport*. The value of *host* is the destination address for the transport. |
|        port | no       |            | The port to use when building the connection to the remote device. This value applies to either acceptable value of *transport*. The port value defaults to the appropriate transport common port if the task provides none (CLI=22, HTTP=80, HTTPS=443). |
|    username | no       |            | Configures the username that authenticates the connection to the remote device.  The value of *username* authenticates the CLI login. If the task does not specify the value, the value of environment variable ANSIBLE_NET_USERNAME is used instead. |
|    password | no       |            | Specifies the password that authenticates the connection to the remote device. This is a common argument used for either acceptable value of *transport*. If the task does not specify the value, the value of environment variable ANSIBLE_NET_PASSWORD is used instead. |
|   authorize | no       | yes, no*   | Instructs the module to enter privileged mode on the remote device before sending any commands. If not specified, the device attempts to execute all commands in non-privileged mode. If the task does not specify the value, the value of environment variable ANSIBLE_NET_AUTHORIZE is used instead. |
|   auth_pass | no       |            | Specifies the password to use if required to enter privileged mode on the remote device. If *authorize=no*, then this argument does nothing. If the task does not specify the value, the value of environment variable ANSIBLE_NET_AUTH_PASS is used instead. |
|   transport | yes      | cli*       | Configures the transport connection to use when connecting to the remote device. The *transport* argument supports connectivity to the device over CLI (SSH).  |
|    provider | no       |            | Convenient method that passes all of the above connection arguments as a dict object. All constraints (required, choices, etc.) must be met either by individual arguments or values in this dict. |

```
Note: Asterisk (*) denotes the default value if none is specified.
```
Dependencies
------------

The dellos-lag role is built on modules included in the core Ansible code.
These modules were added in Ansible version 2.2.0.


Example Playbook
----------------

The following example use the dellos-lag role to setup port channel ID and description. This example also configures hash algorithm and minimum links for the LAG. Channel members can be configured for the port channel either in static or dynamic mode. You can also delete the LAG with the port channel ID or delete the members associated to it. This example creates a ``hosts`` file with our switch, a corresponding ``host_vars`` file, and then a simple playbook that references the dellos-lag role.

Sample hosts file:

    [leafs]
    leaf1

Sample ``host_vars/leaf1``:
    
    hostname: leaf1
    provider:
      host: "{{ hostname }}"
      username: xxxxx
      password: xxxxx
      authorize: yes
	  auth_pass: xxxxx 
      transport: cli

Sample ``defaults/main.yaml``:

    dellos_lag:
      leaf1:
        Po 127:
        type: static
        min_links: 3
        hash: crc16
        channel_members:
          - fortyGigE 1/4,1/12
        state: present
        members_state: present

A simple playbook to setup system, ``leaf.yml``:

    - hosts: leafs
      roles:
         - Dell-Networking.dellos-lag

Then run with:

    ansible-playbook -i hosts leaf.yml

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
