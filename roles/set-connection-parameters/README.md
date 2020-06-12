set-connection-parameters
=========

This role will set the correct SSH port and username (default vs. custom).  
This might be necassary to rerun a playbook that changed either of these.  
*HINT*: In that case you should run *gather_facts* after this role or not at all to avoid a failed play.

### It will:
  * check if the connection is possible with the default SSH port 22 and, if yes, set *{{ ansible_port }}*
  * check if the connection is possible with the custom SSH port *{{ ssh_port}}* and, if yes, set *{{ ansible_port }}*
  * check if the connection is possible with the default Raspbian user *pi* and, if yes, set *{{ ansible_user }}*
  * check if the connection is possible with the custom user *{{ system_user_name}}* and, if yes, set *{{ ansible_user }}*
  * fail if it can't determine the correct SSH port or username


Requirements
------------
  * *ansible_host* or *inventory_hostname* must be set and correct.


Role Variables
--------------
```set-connection-parameters/defaults/main``` contains variables which should be overwritten to customize this role:
```yaml
# Change to the SSH port of your choice
connection_custom_port: 22

# Change to the user name of your choice
connection_custom_user: pi
```

```set-connection-parameters/vars/main``` contains variables which should normally not be touched:
```yaml
# Default user
connection_default_user: pi

# Should facts be gathered after connection parameters are validated?
gather_facts: true
```


Dependencies
------------
None


Example Playbook
----------------
```yaml
- name: Example
  hosts: raspberrypi
  gather_facts: false
  become: true
  vars:
    - user_name: link
    - ssh_port: 5743
  
  tasks:
  - include_role: 
      name: set-connection-parameters
    vars:
      connection_custom_port: "{{ user_name }}"
      connection_custom_user: "{{ ssh_port }}"
```


License
-------

MIT