prepare-raspberry
=========

Bootstrap and customize a Raspbian Lite system. To achieve that this role is using a bunch of subroles (see [Dependencies](##Dependencies)).

### It will:
  * rename the default user *pi* to *user_name*
  * set a new user password *user_password*
  * rename the home dir */home/{{ user_name_old }}* to */home/{{ user_name_new }}*
  * create a symlink */home/{{ user_name_old }}* linking to */home/{{ user_name_new }}*
  * change the hostname to *host_name*
  * configure locale
  * set timezone to *localization_timezone*
  * activate auto-updates / -upgrades
  * install and configure vim
  * set sensible defaults for the SSH server
  * write public SSH keys to *user_name*'s *authorized_keys* file
  * install and configure a firewall (ufw)


Requirements
------------
  
  * *ansible_host* should be set to the hosts IP address instead of the hosts name so that you can connect to your host before and after changing the hostname.
  * Setup
    ### prepare SD card
      * download the [Raspbian Lite image](https://www.raspberrypi.org/downloads/)
      * write image on SD card (instructions at [raspberrypi.org](https://www.raspberrypi.org/documentation/installation/installing-images/README.md))
      * enable ssh by creating a file called *ssh* on the boot partition
      * [optional] enable WIFI by creating a file called *wpa_supplicant.conf* on the boot partition with the following content:
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

```prepare-raspberry/defaults/main``` contains variables which should be overwritten to customize this role:
```yml
# These defaults mention only the most commonly overwritten variables of dependencies.
# For additional options have a look into the individual role defaults.

# rename-user
# Change to the username of your choice
user_name: "pi"
# Password for your user
user_password_hash: "raspberry"

#rename-host
# Change to the hostname of your choice
host_name: "raspberrypi"

# localization
# Set your timezone
localization_timezone: 'Europe/Berlin'
# Default locale. See `man 7 locale`
localization_default_locale: 'en_US.UTF-8'

# apt
# upgrade system: safe | full | dist
apt_upgrade: full
# Do “apt-get upgrade –download-only” every n-days (0=disable)
apt_download_upgradeable_packages: 1
# Do “apt-get autoclean” every n-days (0=disable)
apt_auto_clean_interval: 1
# Split the upgrade into the smallest possible chunks so that
# they can be interrupted with SIGUSR1. This makes the upgrade
# a bit slower but it has the benefit that shutdown while a upgrade
# is running is possible (with a small delay)
apt_unattended_upgrades_minimal_steps: yes
# Send email to this address for problems or packages upgrades
# If empty or unset then no email is sent, make sure that you
# have a working mail setup on your system. A package that provides
# 'mailx' must be installed. E.g. "user@example.com"
apt_mails: []
# Automatically reboot *WITHOUT CONFIRMATION*
# if the file /var/run/reboot-required is found after the upgrade
apt_unattended_upgrades_automatic_reboot: yes
# If automatic reboot is enabled and needed, reboot at the specific
# time instead of immediately
# Values: now | 02:00 | ...
apt_unattended_upgrades_automatic_reboot_time: now

# vim
# set manala.vim config to development
manala_vim_config_template: config/default.dev.j2
# set additional config options for vim
manala_vim_config:
  - incsearch:  true  # Search as soon as characters are entered
  - hlsearch:   true  # Highlight search results

# SSH
# Sets the ssh port
ssh_port: 22
# Required field, list of ssh public keys to update ~/.authorized_keys. 
# Note: If you don't provision your user with keys, your user will no longer be able to access the host via SSH.
# !!!The first key has to be the key ansible is using!!!
ssh_public_keys: []
# String to present when connecting to host over ssh
ssh_banner: "Welcome to {{ user_name }}'s {{ host_name }} server\n"
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

# ufw
ufw_rules:
  - { rule: "allow", port: "{{ ssh_port }}", proto: "tcp", comment: 'SSH' }

# Internal variable used when running tests - should not be used.
ansible_raspbian_testing: false
```


Dependencies
------------

```yml
dependencies:
    - role: set-connection-parameters
      vars:
        connection_custom_port: "{{ ssh_port }}"
        connection_custom_user: "{{ user_name }}"
    - role: provision-root
      vars:
        root_key: "{{ ssh_public_keys | first }}"
    - role: rename-user
      vars:
        user_name_new: "{{ user_name }}"
        user_ignore_connection_errors: true
    - role: rename-host
      vars:
        host_reboot: not ansible_raspbian_testing
    - role: arillso.localization
      vars:
        localization_timezone_linux: "{{ localization_timezone }}"
    - role: weareinteractive.apt
    - role: manala.vim
    - role: ssh-server-config
      vars:
        ssh_user: "{{ user_name }}"
    - role: weareinteractive.ufw
```


Example Playbook
----------------

```yml
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