FriendsOfAnsible.telegraf
=========================

[![Build Status](https://travis-ci.org/FriendsOfAnsible/ansible-role-telegraf.svg?branch=master)](https://travis-ci.org/FriendsOfAnsible/ansible-role-telegraf)

Role to install the [Telegraf]() agent both in Linux and Windows boxes

Requirements
------------

Ansible >= 1.9

Please check [Ansible - Windows support](http://docs.ansible.com/ansible/intro_windows.html) for instructions on how to prepare your Windows hosts to be able to automate them using WinRM from Ansible

Role Variables
--------------

```yaml
- vars:

```

Example Playbook
----------------

```yaml
- hosts: servers
  roles:
     - FriendsOfAnsible.telegraf
```         

License
-------

MIT
