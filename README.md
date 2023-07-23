# Ansible Cheatsheet

This is a cheatsheet for Ansible.

---
# Install

```bash
# Install ansible (for user).
python3 -m pip install --user ansible

# Upgrade ansible (for user).
python3 -m pip install --upgrade --user ansible

# Display ansible core version.
ansible --version

# Display ansible version.
python3 -m pip show ansible

# Unsnstall ansible.
python3 -m pip uninstall ansible
```

## Inventory
### Host.ini

```txt
# Hosts in group 'all'.
host1
host2

# Hosts in a group.
[group-name]
host1
host2

# Hosts in a group with variables defined.
[group-name]
host1 <var_1>=<value_1> <var_2>=<value_2> ... <var_n>=<value_n>
host2 <var_1>=<value_1> <var_2>=<value_2> ... <var_n>=<value_n>
```

### Host Variables

Variable Name|Description
---|---
`ansible_host`|The name or the ip of the host to connect to.</br>If none, the alias will be used.
`ansible_port`|The port to connect, if not defined, port 12 will used.
`ansible_user`|The user to use when connecting to the host
`ansible_password`|The password to use to authenticate to the host
`ansible_ssh_private_key_file`|Private key file used by ssh.
`ansible_ssh_common_args`|This setting is always appended to the default command line for sftp, scp, and ssh.
`ansible_sftp_extra_args`|This setting is always appended to the default sftp command line.
`ansible_scp_extra_args`|This setting is always appended to the default scp command line.
`ansible_ssh_extra_args`|This setting is always appended to the default ssh command line.
`ansible_ssh_pipelining`|Determines whether or not to use SSH pipelining.</br>This can override the pipelining setting in ansible.cfg.
`ansible_ssh_executable`|This setting overrides the default behavior to use the system ssh.</br>This can override the `ssh_executable` setting in ansible.cfg.
`ansible_become`|Equivalent to `ansible_sudo or ansible_su`, allows to force privilege escalation
`ansible_become_method`|Allows to set privilege escalation method
`ansible_become_user`|Equivalent to `ansible_sudo_user` or `ansible_su_user`, allows to set the user you become through privilege escalation
`ansible_become_password`|Equivalent to `ansible_sudo_password` or `ansible_su_password`, allows you to set the privilege escalation password
`ansible_become_exe`|Equivalent to `ansible_sudo_exe` or `ansible_su_exe`, allows you to set the executable for the escalation method selected
`ansible_become_flags`|Equivalent to `ansible_sudo_flags` or `ansible_su_flags`, allows you to set the flags passed to the selected escalation method.
`ansible_shell_type`|The shell type of the target system.
`ansible_python_interpreter`|The target host python path.
`ansible_*_interpreter`|Works for anything such as ruby or perl and works just like `ansible_python_interpreter`.</br>This replaces shebang of modules which will run on that host.

## Ansible Playbook
### Hello World

```yml
- hosts: all
  tasks:
    - name: Print message
      debug:
        msg: Hello world!
```

### Privilege Escalation

```yml
# Activate privilege escalation for all tasks.
- hosts: all
  become: yes
  tasks:
    - name: Print message
      debug:
        msg: Hello world!


# Activate privilege escalation for a specific task.
- hosts: all
  tasks:
    - name: Print message
      become: yes
      debug:
        msg: Hello world!


# Run command with another user.
- hosts: all
  tasks:
    - name: Print message
      become: yes
      become_user: <another-user>
      debug:
        msg: Hello world!
```

### Commonly Used Modules
#### Package

```yaml
- name: Install a package (state=present is optional)
  apt:
    name: <package-name>
    state: present

- name: Remove a package
  apt:
    name: <package-name>
    state: absent

- name: Install a list of packages
  apt:
    pkg:
    - <package1-name>
    - <package2-name>

- name: Upgrade all packages to the latest version
  apt:
    name: "*"
    state: latest

- name: Update all packages to the latest version
  apt:
    upgrade: dist

- name: Run the equivalent of "apt-get update"
  apt:
    update_cache: yes

- name: Install a .deb package
  apt:
    deb: <deb-filepath>

- name: Install the build dependencies for a package
  apt:
    pkg: <package-name>
    state: build-dep

- name: Install a .deb package from the internet
  apt:
    deb: <deb-url>

- name: Remove useless packages from the cache
  apt:
    autoclean: yes

- name: Remove dependencies that are no longer required
  apt:
    autoremove: yes
```

#### Service

```yaml
- name: Start service <service-name>, if not started
  service:
    name: <service-name>
    state: started

- name: Stop service <service-name>, if started
  service:
    name: <service-name>
    state: stopped

- name: Restart service <service-name>, in all cases
  service:
    name: <service-name>
    state: restarted

- name: Reload service <service-name>, in all cases
  service:
    name: <service-name>
    state: reloaded

- name: Enable service <service-name>, and not touch the state
  service:
    name: <service-name>
    enabled: yes
```

#### File Basics

```yaml
- name: Change file ownership, group and permissions
  file:
    path: <filepath>
    owner: <username>
    group: <group-name>
    mode: <file-mode> # file-mode can be numbers like 0644, or symbols like u=rw,g=r,o=r, or symbol changes u+rw,g-wx,o-rwx.

- name: Create a symbolic link
  file:
    src: <filepath-to-link-to>
    dest: <symlink-path>
    owner: <username>
    group: <group-name>
    state: link

- name: Create two hard links
  file:
    src: <filepath-to-link-to>
    dest: <symlink-path>
    state: hard

- name: Create an empty file.
  file:
    path: <filepath>
    state: touch
    mode: <file-mode>

- name: Create a directory if it does not exist
  file:
    path: <dir-path>
    state: directory
    mode: <file-mode>

- name: Recursively change ownership of a directory
  file:
    path: <dir-path>
    owner: <username>
    group: <group-name>
    state: directory
    recurse: yes

- name: Remove a file
  file:
    path: <filepath>
    state: absent

- name: Recursively remove directory
  file:
    path: <dir-path>
    state: absent
```

#### File Copy

```yaml
- name: Copy file with owner and permissions.
  copy:
    src: <src-filepath>
    dest: <dst-filepath>
    owner: <username>
    group: <group-name>
    mode: <file-mode> # file-mode can be numbers like 0644, or symbols like u=rw,g=r,o=r, or symbol changes u+rw,g-wx,o-rwx.

- name: Copy a file, backing up the original one if it differs from the copied version
  copy:
    src: <src-filepath>
    dest: <dst-filepath>
    owner: <username>
    group: <group-name>
    mode: <file-mode>
    backup: yes

- name: Copy a new "sudoers" file into place, after passing validation with visudo
  copy:
    src: /mine/sudoers
    dest: /etc/sudoers
    validate: /usr/sbin/visudo -csf %s

- name: Copy a "sudoers" file on the remote machine for editing
  copy:
    src: /etc/sudoers
    dest: /etc/sudoers.edit
    remote_src: yes
    validate: /usr/sbin/visudo -csf %s

- name: Copy using inline content
  copy:
    content: 'this is the file content'
    dest: <filepath>
```

#### LineInFile

```yaml
- name: If a line matches <line-pattern>, replace it with <line>
  lineinfile:
    path: <filepath>
    regexp: <line-pattern>
    line: <line>

- name: If a line matches <line-pattern>, remove it
  lineinfile:
    path: <filepath>
    regexp: <line-pattern>
    state: absent

- name: If a line matches <line-pattern>, replace it with <line> and place it after the line <insert-line-pattern>
  lineinfile:
    path: <filepath>
    regexp: <line-pattern>
    insertafter: <insert-line-pattern>
    line: <line>

- name: If a line matches <line-pattern>, replace it with <line> and place it before the line <insert-line-pattern>
  lineinfile:
    path: <filepath>
    regexp: <line-pattern>
    insertbefore: <insert-line-pattern>
    line: <line>

- name: Add <line> to a file. Create the file if it doesn't exist
  lineinfile:
    path: <filepath>
    line: <line>
    create: yes
```

#### Commands

```yaml
- name: Execute a command, and send the output to a variable
  command: <command>
  register: <var-name>

- name: Run command if filepath does not exist
  command: <command>
  args:
    creates: <filepath>

- name: Run command in a specific directory
  command: <command>
  args:
    chdir: <dir>
```

#### Git

```yaml
- git:
    name: Check out a git repo to a specific version (which can be HEAD, a branch, or a tag)
    repo: <git-repo-url>
    dest: <checkout-dir-path>
    version: <version>

- git:
    name: Ensure a git repo checkout exists.
    repo: <git-repo-url>
    dest: <checkout-dir-path>
    update: no
```

#### User

```yaml
- name: Add a specific user with a uid (e.g., 1040)
  user:
    name: <username>
    comment: <comment>
    uid: <uid>
    group: <group>

- name: Add a uesr and append groups to the user's groups
  user:
    name: <username>
    groups: <group_1>,<group_2>,...,<group_n>
    append: yes

- name: Remove a user
  user:
    name: <username>
    state: absent
    remove: yes

- name: Add a user with an expire time
  user:
    name: <username>
    groups: <group_1>,<group_2>,...,<group_n>
    expires: <expire-timestamp>

- name: Remove user's expire time
  user:
    name: <username>
    expires: -1
```

#### Debug

```yaml
- name: Print hostname and uuid
  debug:
    msg: System {{ inventory_hostname }} has uuid {{ ansible_product_uuid }}

- name: print hostname and uuid conditionally
  debug:
    msg: System {{ inventory_hostname }} has gateway {{ ansible_default_ipv4.gateway }}
  when: ansible_default_ipv4.gateway is defined
```

### All Modules

A list of all modules can be found [here](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html).

## Commands
### Run Command For A Single Module

```bash
# Ping hosts in group all.
ansible all -m ping

# Ping hosts in group all with a given hosts.ini.
ansible -i hosts.ini all -m ping

# Ping hosts in a group with a given hosts.ini.
ansible -i hosts.ini <group-name> -m ping

# Run a task with a specific module.
ansible -i hosts.ini <group-name> -m <module-name> -a "<module-arg-1-key>=<module-arg-1-value> <module-arg-2-key>=<module-arg-2-value> ... <module-arg-n-key>=<module-arg-n-value>"

# Touch a file.
ansible -i hosts.ini all -m file -a "path=~/file-to-touch mode=0644 state=touch"
```

### Run Command For A Playbook

```bash
# Run a playbook for group all.
ansible-playbook -i hosts.ini <playbook-yml-file>

# Run a playbook for a group.
ansible-playbook -i hosts.ini <playbook-yml-file> <group-name>

# Run a playbook for a host.
ansible-playbook -i hosts.ini <playbook-yml-file> <hostname>

# Show hosts information.
ansible-playbook -i hosts.ini <playbook-yml-file> --list-hosts

# Show tasks information.
ansible-playbook -i hosts.ini <playbook-yml-file> --list-tasks

# Perform playbook syntax check.
ansible-playbook --syntax-check <playbook-yml-file>
```
