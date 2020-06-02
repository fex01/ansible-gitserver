# Ansible Git Server
An example using Ansible to set up a private Git server on a Raspberry Pi.

## Content
* [Intro](#intro)
   * [Why?](#why)
   * [Tools & Platform](#tools--platform)
   * [Steps to Take](#steps-to-take)
* [Preparation](#preparation)
   * [Prepare your Raspberry Pi](#prepare-your-raspberry-pi)
   * [Install Ansible](#install-ansible)
   * [Prepare Work Environment](#prepare-work-environment)
   * [Network Stuff](#network-stuff)
* [Getting Started](#getting-started)
   * [Set IP Address Variables](#set-ip-address-variables)
   * [Run Ansible](#run-ansible)
   * [Setting up a Repository on your new Server](#setting-up-a-repository-on-your-new-server)
* [Getting Useful](#getting-useful)
   * [Ansible Speak](#ansible-speak)
   * [Variables](#variables)
   * [Going back to Start](#going-back-to-start)
* [Deep Dive](#deep-dive)
   * [Execution Command](#execution-command)
   * [Playbook](#playbook)
   * [Role prepare-raspberry](#role-prepare-raspberry)
   * [Role set-connection-parameters](#role-set-connection-parameters)
   * [Role provision-root](#role-provision-root)
   * [Role rename-user](#role-rename-user)
   * [Role rename-host](#role-rename-host)
   * [Role arillso.localization](#role-arillsolocalization)
   * [Role weareinteractive.apt](#role-weareinteractiveapt)
   * [Role GROG.reboot](#role-grogreboot)
   * [Role manala.vim](#role-manalavim)
   * [Role weareinteractive.users](#role-weareinteractiveusers)
   * [Role ssh-server-config](#role-ssh-server-config)
   * [Role weareinteractive.ufw](#role-weareinteractiveufw)
   * [Role gitserver-config](#role-gitserver-config)
   * [Role weareinteractive.users - again?](#role-weareinteractiveusers---again)
   * [Role weareinteractive.git](#role-weareinteractivegit)
* [Testing](#testing)
* [Outlook](#outlook)
* [Sources](#sources)



## Intro
### Why?
To have a repeatable and versionable cooking receipt for setting up servers sounds quite sexy, so even if it might be a bit of a overkill for private users, I got quite intrigued by [Ansible][ansible-intro].

Since the best way to learn something is
1. to use it
2. to teach<sup id="a1">[1](#f1)</sup> it

I decided to take an existing [work log][worklog], to turn it into an Ansible play and to document what I did.

### Tools & Platform
* machine with Ansible: running [Ubuntu 18.04 LTS](https://ubuntu.com)<sup id="a2">[2](#f2)</sup>
* target: [Raspberry Pi 3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)<sup id="a3">[3](#f3)</sup>
* target OS: [Raspbian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/)
* server administration tool: [Ansible 2.9][ansible-intro]

### Steps to Take
(for details see [work log][worklog])
* change username pi to a custom username
* change the default password
* change default hostname
* SSH hardening
* auto updates
* install and configure firewall ([ufw](https://en.wikipedia.org/wiki/Uncomplicated_Firewall))
* install & configure vim (optional, it's just my preferred editor)
* install & configure git
* create additional user git
* restrict user git to git shell
* clone repos from backup



## Preparation
If not stated otherwise all steps have to be done on your Ansible machine, not on the target.

### Prepare your Raspberry Pi
* download Raspbian Lite https://www.raspberrypi.org/downloads/
* write image on SD card https://www.raspberrypi.org/documentation/installation/installing-images/README.md
    * `touch /Volumes/boot/ssh`<sup id="a4">[4](#f4)</sup> enable SSH by writing an empty file called `ssh` on the boot partition<sup id="a5">[5](#f5)</sup>
    * eject & insert SD card into your Raspi
    * connect your Raspberry Pi to LAN and power
* if missing, generate ssh public key on your Ansible machine: `ssh-keygen -o`
* if you are repeating the setup process delete old host keys
    * delete old host key (name): `ssh-keygen -f "/home/<username>/.ssh/known_hosts" -R "raspberrypi"`
    * delete old host key (IP): `ssh-keygen -f "/home/<username>/.ssh/known_hosts" -R "xxx.xxx.xxx.xxx"`
* copy ssh public key to your Pi (password raspberry): `ssh-copy-id pi@raspberrypi`

### Install Ansible
The following steps are for Ubuntu, for details / other OSs have a look into the [Ansible docs][ansible-install].
* `sudo apt update`
* `sudo apt install software-properties-common`
* `sudo apt-add-repository --yes --update ppa:ansible/ansible`
* `sudo apt install ansible`
* `sudo apt install python3-argcomplete`
* `sudo activate-global-python-argcomplete3`

### Prepare Work Environment 
* create an empty folder, e.g. ansible-gitserver, and cd into said folder
* `git clone https://github.com/fex01/ansible-gitserver.git ./`
* download required public roles `ansible-galaxy install -r requirements.yml` (default installation path '/etc/ansible/roles')

### Network Stuff
Recommendation: Assign a permanent IP address to your Raspi, probably done via your router. Example for a [AVM FRITZ!Box](https://en.avm.de/service/fritzbox/fritzbox-7590/knowledge-base/publication/show/201_Configuring-FRITZ-Box-to-always-assign-the-same-IP-address-to-a-network-device/):
* open a browser, open the Web GUI of your router, address might be something like 'http://192.168.xxx.1'
* log in with an admin account
* Home Network -> Network (German GUI says 'Heimnetz -> Netzwerk')
* click the 'Edit' button for the device 'raspberrypi'
* activate the appropriate option, should be something like 'Always assign the same IPv4 address to this network device'
* click 'OK' to save your changes
* since the device is identified by it's MAC address, it doesn't matter that we will rename our Raspi later and it will also not matter if you insert a different SD Card with an different image into your Raspi



## Getting Started
### Set IP Address Variables
For a first test run it's enough to open your ansible-gitserver working directory and change the following values:
* group_vars/all.yml/ssh_public_keys        # public SSH key of your ansible machine
* group_vars/szczecin.yml/location_subnet
* group_vars/gitserver.yml/ansible_host     # IP address of your Raspi

### Run Ansible
Having done that you could now start the setup process with (default vault password is 'my_secret_password')
```
ansible-playbook gitserver.yml -i ./hosts --vault-id git@prompt
```

Ansible will now execute the playbook and you can watch while a not to short list of tasks will be finished.
And voilà, you have a ready to use private Git server. Credentials are as following:
* system account
    * name: link
    * password: my_secret_password
* git account
    * name: git
    * password: my_secret_password

### Setting up a Repository on your new Server
You might want to actually have some repos on your Git server, lets say you want my ansible-gitserver repo on your private server:
* `ssh link@git` ssh into your Git server with the system account
* `cd ../git/` change into user gits homefolder
* `sudo git clone --bare https://github.com/fex01/ansible-gitserver.git ansible-gitserver.git` we clone the repo with the `--bare` option, for details why let me point you to the relevant chapter of the free online book [Pro Git][git-server]
* `sudo chown -R git:git ansible-gitserver.git` make the user git the owner of this repo
* `exit` end your SSH connection

Now what do we have to do on our ansible machine to use our Git server as remote? Let's have a look at two different options:
* clone a fresh copy of the repo from our private server
    * leave your ansible-gitserver working directory and navigate to where you want the new copy
    * `git clone ssh://git@git/home/git/ansible-gitserver.git ansible-gitserver-private`
    * done, `git push` and `git pull`commands will use your private server as target
* add a second remote to our existing ansible-gitserver working directory
    * `git remote add private ssh://git@git/home/git/ansible-gitserver.git`
    * done, `git remote show` will now show two remotes for this repo, origin & private
    * with `git remote show origin` or `git remote show private` you can test the connection to the remote and see details like the remotes URL

While this test run was nice to see that Ansible actually does the job, **I would strongly counsel against using your new server at it is - you are still using user names & passwords chosen by me**!
Let's have a look at the next chapter to change that.


## Getting Useful
An Ansible based setup process would not be very useful if you couldn't change my default values for stuff like user names, passwords, port numbers, etc. ... - thats where Variables come into play. To understand where to set them, we need some common terms.

### Ansible Speak
I aim to keep the vocabulary lessons to a minimum, but lets have a look at three key concepts:
#### Inventory
A list of computers managed by Ansible, sorted into groups (details see [Ansible - How to build your inventory][ansible-inventory]).
As example a commented 'hosts' file:
```
# file: hosts
 
[berlin_raspbian]               

[szczecin_gitserver]             # group
git                              # a managed computer, called host

[szczecin_raspbian]              # different group
git                              # same host

# gitserver in all locations
[gitserver:children]             # group of groups
szczecin_gitserver

# raspbian installations in all locations
[raspbian:children]              # different group of groups
berlin_raspbian
szczecin_raspbian

# everything in berlin
[berlin:children]
berlin_raspbian

# everything in szczecin
[szczecin:children]
szczecin_gitserver
szczecin_raspbian
```
As you can see I have grouped my hosts by location (berlin & szczecin), by function (gitserver) and by OS (raspbian). This enables me to link them with variables based on groups, more about that in [Variables](#variables).
 
#### Playbooks
The previous mentioned repeatable cooking receipts - on which machine(s) do you want to execute which tasks and what are the environment parameters (details see [Ansible - About Playbooks][ansible-playbooks]).
Example: gitserver.yml
```yml
- name: playbook to set up a git server     # name of your play
  hosts: gitserver                          # group of machines to which to apply this play
  gather_facts: false                       # \
  debugger: on_failed                       # - optional parameters
  become: true                              # / 

  tasks:                                    # list of tasks
  - include_role: 
    name: prepare-raspberry
  - include_role: 
    name: gitserver-config
```
#### Roles
Instead of writing the same tasks again and again for different playbooks, you can bundle tasks into roles (details see [Ansible - Roles][ansible-roles]). But the biggest advantage of roles is that they are sharable - just have a look at [Ansible Galaxy][ansible-galaxy] to find roles for almost all common tasks.

### Variables
Comming back to variables - while Ansibles knows a number of options where to set them (details see [Ansible - Using Variables][ansible-variables]), the basic option is to link them to machines in your [Inventory](#inventory). 
Our target is named 'git' and a member of the groups 'szczecin-gitserver', 'szczecin-raspbian', 'gitserver', 'raspbian', 'szczecin' and the special group 'all', which means the variables for all these groups (called group_vars) and the variables for our target (called host_vars) apply. And if you followed [Set IP Address Variables](#set-ip-address-variables), you actually already know where to find them:
* group_vars: working directory/group_vars/<group name>.yml
* host_vars: working_directory/host_vars/<host name>.yml

For an individualized Git server I would recommend to set at least the following variables, for more options look into the variable files (for how to encrypt variables see comments for /group_vars/gitserver.yml/user_password_hash):
```yml
# working directory/group_vars/all.yml

#default system user
ansible_user:

# list of public SSH keys wich should get permission to connect to host
ssh_public_key:
```
```yml
# working directory/group_vars/raspbian.yml

# system user name
user_name:
```
```yml
# working directory/group_vars/gitserver.yml

# hostname will change during bootstrapping, provide IP address
ansible_host:
# custom SSH port
# password for system user
# create: 
#   * create hash: ansible all -i localhost, -m debug -a "msg={{ 'my_secret_password' | password_hash('sha512') }}"
#   * encrypt hash: ansible-vault encrypt_string --vault-id git@prompt --stdin-name 'user_password_hash'
# use in play: ansible-playbook <playbook>.yml --vault-id git@prompt
user_password_hash:
# custom SSH port
ssh_port:
# password for the dedicated git user
git_os_user_password_hash:
# username for git commits
git_commit_user_name:
# user email for git commits
git_commit_user_email:
```
Additionally my play has the option to have Ansible import repos from a backup location. For that follow the instructions in the comments and set the following variables (after initial import the private key will be deleted from the new Git server, you have to delete the public key manually from your backup machine<sup id="a6">[6](#f6)</sup>):
```yml
# working directory/group_vars/gitserver.yml

# [optional] create private / public key pair: ssh-keygen -o
# make public key available on existing git machine (e.g. a backup machine)
# copy content of the private key file and encrypt it
git_private_key:
# git repos on an existing git machine
# example:
#    - repo: ssh://link@{{ location_subnet }}.xxx/path/to/repo/ansible.git
#      dest: /home/git/repos/ansible.git
#      bare: true
git_repositories:
```

Disclaimer: I ignored variable precedence in my explanations (if you set the same variable in multiple location, for example group_vars & host_vars, which value applies?), please look that up at [Ansible - Variable precedence: Where should I put a variable?][ansible-precedence].

### Going back to Start
What? After having finally a running Git server? Yes, but don't, it's quite easy, just repeat the steps of [Prepare your Raspberry Pi](#prepare-your-raspberry-pi) and you are ready to go again.

### Run Ansible
And now everything you have to do to get a brand new individualized Git server is running (remember to enter *your* encryption password when asked):
```
ansible-playbook gitserver.yml -i ./hosts --vault-id git@prompt
```
Done - but this time you have your own individualized Git server :-)  
That also concludes my chapters regarding setting up your own Git server - the next chapters focus more on how to work with Ansible.


## Deep Dive
### Execution Command
Lets start with the command we are executing: `ansible-playbook gitserver.yml -i ./hosts --vault-id git@prompt`
* `ansible-playbook gitserver.yml` execute playbook 'gitserver.yml'
* `-i ./hosts` use the file 'hosts' in the current directory as inventory
* `--vault-id git@prompt` this play has encrypted variables, ask me for the decryption password, password hint is 'git'

### Playbook
So we are executing a playbook, what does it do?
```yml
- name: playbook to set up a git server     # name of your play
  hosts: gitserver                          # group of machines to which to apply this play
  gather_facts: false                       # \
  debugger: on_failed                       # - optional parameters
  become: true                              # / 

  tasks:                                    # list of tasks
  - include_role: 
    name: prepare-raspberry
  - include_role: 
    name: gitserver-config
```
Our playbook should execute two roles with a certain set of variables.

### Role prepare-raspberry

### Role set-connection-parameters

### Role provision-root

### Role rename-user

### Role rename-host

### Role arillso.localization

### Role weareinteractive.apt

### Role GROG.reboot

### Role manala.vim

### Role weareinteractive.users

### Role ssh-server-config

### Role weareinteractive.ufw

### Role gitserver-config

### Role weareinteractive.users - again?

### Role weareinteractive.git


## Testing








## Outlook
* require password for sudo - probably easy, but I still have to find out how to deal with required sudo passwords in Ansible :sweat_smile:
* automate testing -> automate builds (travis?)
* find / create a fitting Raspbian test image for Docker (seems to be non-trivial because of the underlying ARM architecture :thinking:)
* replacing more of my custom roles with public roles from [Ansible Galaxy][ansible-galaxy]
* publishing my surviving roles on [Ansible Galaxy][ansible-galaxy]
    * move each role into it's own repo (necessary for publishing said role)
    * integrate them as dependencies / subrepos into this repo


## Sources
* [Ansible docs][ansible-intro]
* a role which I could not fit exactly to my purpose but which really helped with getting me started: [hannseman.raspbian](https://github.com/hannseman/ansible-raspbian)
* the code from [moodlebox](https://github.com/moodlebox/moodlebox) was quite helpful vor renaming the default user via Ansible
* Ansibles roles repo [Ansible Galaxy](https://galaxy.ansible.com) in general - just have a look into the source code of different roles to learn
* Jeff Geerlings [Testing your Ansible roles with Molecule](https://www.jeffgeerling.com/blog/2018/testing-your-ansible-roles-molecule) (post is from 2028, installation instructions are not up to date, but it helps with getting the concept)
* [Molecule docs](https://molecule.readthedocs.io/en/latest/)
* my git server [work log][worklog]

---
<b id="f1">1</b>: Keep in mind, I neither consider myself an Linux-, Raspberry-, security-, Git- or Ansible-expert. So please let me know about possible improvements and, especially, think twice before you copy-paste security related stuff! [↩](#a1)  
<b id="f2">2</b>: You can install Ansible on a variety of different Linux distributions or macOS, have a look at the [docs](https://docs.ansible.com/ansible/latest/installation_guide/index.html) [↩](#a2)  
<b id="f3">3</b>: An older model shouldn't be a problem, I just had that one laying around. [↩](#a3)  
<b id="f4">4</b>: Your mounting point for the SD Card might differ [↩](#a4)  
<b id="f5">5</b>: If you use etcher either disable auto-eject before writing the image or eject and reinsert your SD Card to mount it again. [↩](#a5)  
<b id="f6">6</b>: The idea is behind deleting the private key is, that user git only has the option to accept inbound connections but has no possibility to initiat outgoing connections. See also [Role gitserver-config](#role-gitserver-config) regarding restricted forwarding / restricted shell. [↩](#a6)  

[ansible-intro]: https://docs.ansible.com/ansible/latest/index.html
[ansible-install]: https://docs.ansible.com/ansible/latest/installation_guide/index.html
[ansible-inventory]: https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#intro-inventory
[ansible-playbooks]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#about-playbooks
[ansible-roles]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#roles
[ansible-galaxy]: https://galaxy.ansible.com
[ansible-variables]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html
[ansible-precedence]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable
[git-server]: https://git-scm.com/book/en/v2/Git-on-the-Server-Getting-Git-on-a-Server
[worklog]: https://community.openhab.org/t/setting-up-my-own-git-server-on-a-pi/
