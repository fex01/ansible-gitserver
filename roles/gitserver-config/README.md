gitserver-config
=========


Create a dedicated user for git operations, install git & clone your repos.

### It will
* create an non-sudo user {{ git_os_user_name }}
  * shell restricted to git-shell and without interactive login
  * provisioned with authorized keys
  * provisioned with a private key for initial cloning
  * with restricted SSH access (no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty)
* add a second user {{ user_name }} without special restrictions to group {{ git_os_user_name }}
* install and configure git
* clone specified repos (make sure that the source accepts the private key)
* deletes the private key after cloning (the idea is that, after initial cloning, your git server should not be able to access your backup source)

Role Variables
--------------

`gitserver-config/defaults/main` contains variables which should be overwritten to customize this role:
```yml
# defaults file for gitserver-config
#   * additional defaults can be found in the <subrole-name>/defaults folders

# git connections will not be done with the system user but with a dedicated git user
git_os_user_name: git

# password for the dedicated git user
# create: 
#   * create hash: ansible all -i localhost, -m debug -a "msg={{ 'my_secret_password' | password_hash('sha512') }}"
#   * encrypt hash: ansible-vault encrypt_string --vault-id git@prompt --stdin-name 'git_os_user_password_hash'
# use in play: ansible-playbook <playbook>.yml --vault-id git@prompt
git_os_user_password_hash: "{{ 'my_secret_password' | password_hash('sha512') }}"

# The dedicated git user should not be able to use the standard shell, but should be
# restricted to the git-shell. To get the path to the git shell execute 'which git-shell'
# on the host.
git_os_user_shell: /usr/bin/git-shell
# username for git commits
git_commit_user_name: link
# user email for git commits
git_commit_user_email: link@example.org
# [optional]
# private key will be deploy by w.users to /home/git/.ssh/id_rsa
#   * create private / public key pair: ssh-keygen -o
#   * make public key available on existing git machine (e.g. a backup machine)
#   * copy content of the private key file and encrypt it (see {{ git_os_user_password_hash }})
git_private_key:
# keys to authorize repo SSH access
git_authorized_keys: []
# [optional]
# git repos on an existing git machine (e.g. a backup machine)
# example:
#    - repo: ssh://link@{{ location_subnet }}.xxx/path/to/repo/ansible.git
#      dest: /home/git/repos/ansible.git
#      bare: true
git_repositories: []
```

Dependencies
------------

This role is builds up on `weareinteractive.users` & `weareinteractive.git`. These roles can be installed by executing
```
ansible-galaxy install weareinteractive.users
ansible-galaxy install weareinteractive.git
```

Example Playbook
----------------

```yml
- name: playbook to set up a git server
  hosts: gitserver
  debugger: on_failed
  become: true

  tasks:
  - include_role: 
      name: gitserver-config
```

License
-------

MIT
