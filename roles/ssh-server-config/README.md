ssh-server-config
=========

This role will change the SSH port, set sensible defaults in the SSH server config file and has the option to set additional settings.

### It will:
  * fail when *ssh_user* is undefined to avoid a situation where no user is allowed to connect via SSH
  * change the default SSH port - and set *ansible_port* to the new value to avoid connectivity loss to the host
  * disable SSH *root* login
  * enable SSH strict mode
  * disable X11 forwarding
  * disable SSH password login - **you** should provision the user with keys prior to running this role! (One possible option for that would be the public role [weareinteractive.users](https://galaxy.ansible.com/weareinteractive/users))
  * allow SSH access only to *ssh_users*
  * add an SSH banner
  * set additional settings


Requirements
------------

If you rerun your play you have to set *ansible_port* to your custom SSH port prior to rerunning this role. One option to do that would be my role *set-connection-parameters*.


Role Variables
--------------

```ssh-server-config/defaults/main``` contains variables which should be overwritten to customize this role:
```yaml
# grant SSH access only for
ssh_users: 
  - pi
# path to SSH server config file
ssh_sshd_config: "/etc/ssh/sshd_config"
# Change to a port of your choice
ssh_port: 22
# String to present when connecting to host over ssh
ssh_banner:
# Basic sshd config will be done, use this to set additional values.
# Syntax:
#   - var_name: "<settings name>"
#     value: "<settings value>"
# For details about setting names and values check /etc/ssh/sshd_config.
#
# If more than one user should have SSH access add the following here:
#   - var_name: "AllowUsers"
#     value: "user1, user2, {{ ssh_user }}"
ssh_sshd_config_settings: []
```


Dependencies
------------

* After running this role the variable *ansible_port* will be set to *ssh_port*, overriding other sources for *ansible_port*.
* If you rerun your play, make sure to set *ansible_port* to your custom SSH port before you run tasks connecting to the host. One option to do that would be my role *set-connection-parameters*.
* make sure to provision the user with keys prior to running this role! (One possible option for that would be the public role [weareinteractive.users](https://galaxy.ansible.com/weareinteractive/users))


Example Playbook
----------------

```yaml
- name: SSH hardening
  hosts: raspberrypi
  gather_facts: false
  become: true
  vars:
    - user_name: link
    - ssh_port: 394586
    - ssh_public_keys:
        - ssh-rsa XXXXXXXXXXXXXXXXXX[...]XXXXXXXXXXXXXXXXXXXXXXXXXX link@my-computer.local
    - ssh_sshd_config_settings:
        - var_name: "MaxAuthTries"
          value: "3"
        - var_name: "PubkeyAuthentication"
          value: "yes"
        - var_name: "AuthorizedKeysFile"
          value: ".ssh/authorized_keys"
        - var_name: "IgnoreUserKnownHosts"
          value: "yes"
        - var_name: "IgnoreRhosts"
          value: "yes"
        - var_name: "AllowAgentForwarding"
          value: "no"
        - var_name: "AllowTcpForwarding"
          value: "no"
        - var_name: "GatewayPorts"
          value: "no"
        - var_name: "PermitTTY"
          value: "yes"
        - var_name: "Protocol"
          value: "2"
  
  tasks:
  - include_role: 
      # If this is a rerun the connection might fail since username and / or ssh port might
      # have been changed. 
      # To avoid connection failures this role is setting {{ ansible_port }} & {{ ansible_user }}
      # to the correct values.
      name: set-connection-parameters
    vars:
      connection_custom_user: "{{ user_name }}"
      connection_custom_port: "{{ ssh_port }}"
  - name: gather facts
    gather_facts:
  - include_role: 
      name: weareinteractive.users
    vars:
      users:
        - username: "{{ user_name }}"
          authorized_keys: "{{ ssh_public_keys }}"
  - include_role: 
      name: ssh-server-config
    vars:
      ssh_users: ["{{ user_name }}"]
```


License
-------

MIT