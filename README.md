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
  telegraf_version: 0.13.1

  # defaults file for telegraf linux
  telegraf_linux_url: https://dl.influxdata.com/telegraf/releases/telegraf_{{ telegraf_version }}_amd64.deb
  telegraf_linux_path: /etc/telegraf/

  # defaults values for telegraf windows
  telegraf_win_url: https://dl.influxdata.com/telegraf/releases/telegraf-{{ telegraf_version }}_windows_amd64.zip

  # Folder where telegraf will be installed in Windows
  telegraf_win_path: C:\telegraf
  # The telegraf zip file we download has a folder. 
  # This has changed over time although telegraf maintainers say this will stay with the value "telegraf"
  telegraf_zip_folder: telegraf

  # Some sensible defaults for the telegraf agent
  telegraf_debug: "false"
  telegraf_agent_interval: 30s
  telegraf_round_interval: "false"
  telegraf_flush_interval: 30s
  telegraf_flush_jitter: 15s

  # Some sensible defaults for influxdb outputs
  telegraf_influxdb_urls: 
    - http://localhost:8086
  telegraf_influxdb_database: telegraf
  telegraf_influxdb_precision: s
  telegraf_influxdb_timeout: 5s

  # Please check telegraf inputs section or the tests folder for advanced examples
  telegraf_inputs: []
  
  # Please check telegraf outptus section for advanced examples
  telegraf_outputs:
    -
      type: influxdb
      config:
        urls: "{{ telegraf_influxdb_urls }}"
        database: "{{ telegraf_influxdb_database }}"
        precision: "{{ telegraf_influxdb_precision }}"
        timeout: "{{ telegraf_influxdb_timeout }}"

```

Example Playbook
----------------

```yaml
- hosts: servers
  roles:
     - FriendsOfAnsible.telegraf
```         

Telegraf inputs
---------------

Docs coming soon

Telegraf outputs
----------------

Docs coming soon

License
-------

MIT
