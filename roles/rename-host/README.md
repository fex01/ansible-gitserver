rename-host
=========

This role will change the hostname of a Raspbian installation.

### It will:
  * change the hostname to *host_name*
  * update `/etc/hosts`
  * reboot the host if *host_reboot* is set to *true*


Requirements
------------

*ansible_host* should be set to the hosts IP address instead of the hosts name so that you can connect to your host before and after executing this role.


Role Variables
--------------

```rename-host/defaults/main``` contains variables which should be overwritten to customize this role:
```yaml
# Change to the hostname of your choice
host_name: "raspberrypi"
# Reboot after renaming
host_reboot: true
```


Dependencies
------------

None


Example Playbook
----------------

```yaml
- name: playbook to change the default hostname of a Raspbian Lite installation
  hosts: myserver
  become: true
  vars:
    - host_name: myserver
  
  tasks:
  - include_role:
      name: rename-host
```


License
-------

MIT
