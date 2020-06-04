rename-user
=========

Change a username / change a users password. Can handle the default user *pi* including all relevant files.

### It will:
  * connect as *user_executing_account*
  * rename *user_name_old* to *user_name_new*
  * set a new password *user_password* (if defined)
  * rename the home dir */home/{{ user_name_old }}* to */home/{{ user_name_new }}*
  * create a symlink */home/{{ user_name_old }}* linking to */home/{{ user_name_new }}*
  * set the value of *ansible_user* to *user_name_new* as to not disrupt your plays connection to the host
  * fail gracefully (-> skip & continue play) when *user_ignore_connection_errors* is set to *true*


Requirements
------------

This role connects as *root*, to facilitate that, you need to provision the root account with the public ssh key of your ansible machine / with your ansible public ssh key. One way to do that is my role *provision-root*.


Role Variables
--------------

```rename-user/defaults/main``` contains variables which should be overwritten to customize this role:
```yaml
# Name that should be changed
user_name_old: "pi"
# Username of your choice
user_name_new: "pineapple"
# Password for your user - if you leave this empty the password will not be changed
user_password_hash:
# Its necessary to use a second account to rename an account. This
# could be done with root or a different sudo-account
user_executing_account: root
# If your play should go on after this role fails to connect to the host,
# set this to true.
# This might be useful if you are sure this role would fail on a rerun due to changed
# security settings later on in your play.
# Example:
# - you run this role as root (user_executing_account: root)
# - later on in your play you set the sshd-setting 'PermitRootLogin no'
user_ignore_connection_errors: false
```

```rename-user/vars/main``` contains variables which should normally not be touched:
```yaml
# list of files in which the username will be changed
# for a default Raspbian Lite installation you don't have to change this list
username_files:
  - /etc/passwd
  - /etc/group
  - /etc/shadow
  - /etc/gshadow
  - /etc/sudoers
  - /etc/systemd/system/autologin@.service
  - /etc/sudoers.d/010_pi-nopasswd
```


Dependencies
------------

* Passwordless SSH auth for *root* must be enabled prior to running this role.
* After running this role the variable *ansible_user* will be set to *user_name_new*, overriding other sources for *ansible_user* (e.g. inventory).


Example Playbook
----------------

```yaml
- name: playbook to change the default username of a Raspbian Lite installation
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
  - include_role: 
      name: rename-user
    vars:
      user_name_new: "{{ user_name }}"
```


License
-------

MIT