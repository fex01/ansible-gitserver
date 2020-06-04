provision-root
=========

Provision *root* for passwordless SSH auth.

### It will:
  * fail if *root_key* is undefined
  * write the provided SSH public key *root_key* into *root*s `authorized_keys` file


Requirements
------------

This role connects as *ansible_user*, to facilitate that, you need to provision the default account with the public ssh key of your ansible machine / with your ansible public ssh key. One way to do that is:

### prepare SD card
  * download the [Raspbian Lite image](https://www.raspberrypi.org/downloads/)
  * write image on SD card (instructions at [raspberrypi.org](https://www.raspberrypi.org/documentation/installation/installing-images/README.md))
  * enable ssh by creating a file called *ssh* on the boot partition
  * [optional] enable wifi by creating a file called *wpa_supplicant.conf* on the boot partition with the following content:
  ```
  network={
          ssid="your ssid"
          psk="your password"
  }
  ```
  * eject SD card & insert it into your Raspberrypi

### enable key-based ssh auth for pi
  * [optional] generate ssh public key on your ansible machine: `ssh-keygen -o`
  * [optional] delete old host key (name): `ssh-keygen -f "/home/<your user name>/.ssh/known_hosts" -R "raspberrypi"`
  * copy ssh public key to your Pi (password raspberry): `ssh-copy-id pi@raspberrypi`


Role Variables
--------------

```provision-root/defaults/main``` contains variables which should be overwritten to customize this role:
```yaml
# Public key used by ansible - role will fail if the key is not provided
root_key:
```


Dependencies
------------

To run this role on a fresh Raspbian installation *ansible_user* should be set to *pi*, for example by running my role *set-connection-parameters* prior to running this role:
```yaml
tasks:
- include_role:
    name: set-connection-parameters
  vars:
    connection_custom_user: "{{ user_name }}"
    connection_custom_port: "{{ ssh_port }}"
- name: gather facts
  gather_facts:
- include_role: 
    name: provision-root
  vars:
    root_key: "{{ ssh_ansible_key }}"
```


Example Playbook
----------------

```yaml
- name: playbook to provision root for passwordless SSH auth
  hosts: raspberrypi
  gather_facts: false
  become: true
  vars:
    - user_name: link
    - ssh_port: 394586
    - ssh_ansible_key: ssh-rsa XXXXXX[...]XXX ansible_user@ansible-machine.local
  
  tasks:
  - include_role:
      name: set-connection-parameters
    vars:
      connection_custom_user: "{{ user_name }}"
      connection_custom_port: "{{ ssh_port }}"
  - name: gather facts
    gather_facts:
  - include_role: 
      name: provision-root
    vars:
      root_key: "{{ ssh_ansible_key }}"
```

License
-------

MIT