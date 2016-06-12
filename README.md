FriendsOfAnsible.telegraf
=========================

[![Build Status](https://travis-ci.org/FriendsOfAnsible/ansible-role-telegraf.svg?branch=master)](https://travis-ci.org/FriendsOfAnsible/ansible-role-telegraf)

Role to install the [Telegraf]() agent both in Linux and Windows boxes

Requirements
------------

Ansible >= 2.0

Please check [Ansible - Windows support](http://docs.ansible.com/ansible/intro_windows.html) for instructions on how to prepare your Windows hosts to be able to automate them using WinRM from Ansible

Role Variables
--------------

```yaml
- vars:
  telegraf_version: 0.13.1

  # defaults file for telegraf linux
  telegraf_linux_url: https://dl.influxdata.com/telegraf/releases/telegraf_{{ telegraf_version }}_amd64.deb
  telegraf_linux_path: /etc/telegraf

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
     - role: FriendsOfAnsible.telegraf
        telegraf_inputs:
          -
            type: disk
            config:
              mountpoints:
                - "/"
          -
            type: mem

```         

Notes on Windows support
------------------------

Support for Windows in Telegraf is experimental although we have succesfully installed it in Windows 2008, 2012, 2012R2

The win_chocolatey Ansible module fails sometimes in Windows 2008 if chocolatey is not installed in the targetted box. 
A "hack" that we have found useful has been to add these 2 tasks at the beginning:

```yaml
- name: Chcolatey installation
  win_chocolatey:
    name: chocolatey
    state: present
    upgrade: no

- name: Fix problem with Chocolatey (https://github.com/ansible/ansible-modules-extras/issues/378)
  raw: Chocolatey feature enable -n allowGlobalConfirmation
```

Alternatively, you may also try to explicitly install Chocolatey before running the role with:

```yaml
- name: install chocolatey explicitly
  raw: iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))
```

Telegraf inputs
---------------

Multiple inputs are supported, hopefully allowing every combination the Telegraf agent supports.
Some examples we are using:

**Linux host running RabbitMQ. Collection of both system metrics and RabbitMQ metrics**

```yaml
      telegraf_inputs:
        -
          type: cpu
          config:
            percpu: "true"
            totalcpu: "true"
            drop:
              - cpu_time
        -
          type: disk
          config:
            mountpoints:
              - "/"
        -
          type: io
          config:
            skip_serial_number: "true"
        -
          type: mem
        -
          type: system
        -
          type: rabbitmq
          config:
            url: "http://localhost:15672"
            username: admin
            password: changeme
```

**Windows host running MSMQ. Collection of multiple win_perf_counters**

```yaml
      telegraf_inputs:
        -
          type: win_perf_counters
          subinputs:
             -
               type: object
               config:
                 ObjectName: "Memory"
                 Counters: ["Available Bytes","Cache Faults/sec","Demand Zero Faults/sec","Page Faults/sec","Pages/sec","Transition Faults/sec","Pool Nonpaged Bytes","Pool Paged Bytes"]
                 Instances: ["------"]
                 Measurement: "win_mem"

             -
               type: object
               config:
                ObjectName: "Processor"
                Instances: ["*"]
                Counters: ["% Idle Time", "% Interrupt Time", "% Privileged Time", "% User Time", "% Processor Time"]
                Measurement: "win_cpu"

             -
               type: object
               config:
                 ObjectName: "LogicalDisk"
                 Instances: ["C:", "D:"]
                 Counters: ["% Idle Time", "% Disk Time","% Disk Read Time", "% Disk Write Time", "% User Time", "Current Disk Queue Length", "% Free Space", "Free Megabytes"]
                 Measurement: "win_disk"

             -
                type: object
                config:
                  ObjectName: "MSMQ Service"
                  Instances: ["------"]
                  Counters: ["Total messages in all queues", "Total bytes in all queues", "Sessions", "Outgoing Messages/sec", "MSMQ Outgoing Messages", "MSMQ Incoming Messages", "IP Sessions", "Incoming Messages/sec"]
                  Measurement: "msmq_service"

             -
                type: object
                config:
                  ObjectName: "MSMQ Queue"
                  Instances: ["*"]
                  Counters: ["Messages in Queue", "Messages in Journal Queue", "Bytes in Queue", "Bytes in Journal Queue"]
                  Measurement: "msmq_queue"

             -
                type: object
                config:
                  ObjectName: "MSMQ Session"
                  Instances: ["*"]
                  Counters: ["Outgoing Bytes", "Incoming Bytes", "Outgoing Messages", "Incoming Messages", "Outgoing Bytes/sec", "Incoming Bytes/sec", "Outgoing Messages/sec", "Incoming Messages/sec"]
                  Measurement: "msmq_session"
```

**Host listening on a UDP port and also acting as a Statsd listener so that we can send business metrics to these ports and they will get forwarded to the defined outputs**

```yaml
      telegraf_inputs:
        -
          type: statsd
          config:
            service_address: ":8125"
            delete_counters: "true"
            delete_gauges: "false"
            delete_sets: "false"
            delete_timings: "true"
            metric_separator: "_"
            parse_data_dog_tags: "false"
            allowed_pending_messages: 10000
            percentile_limit: 1000

        -
          type: udp_listener
          config:
            service_address: ":8092"
            allowed_pending_messages: 10000
            data_format: "influx"
```

Telegraf outputs
----------------

Multiple outputs are also supported. 

One particularly useful use case is target different influxdb databases which may have different retention policies (for instance, keep system metrics for 1 month but business metrics for 12 months).

Another use case would be for instance sending data to both InfluxDB and AWS Cloudwatch.

Please check [https://github.com/influxdata/telegraf](https://github.com/influxdata/telegraf) to see all possible outputs you can define.

Some examples:

**Linux host running RabbitMQ as described in the inputs section. Targetting different influx databases in the same host using namepass and namedrop tags.**

```yaml
      telegraf_outputs:
        -
          type: influxdb
          config:
            urls: "{{ telegraf_influxdb_urls }}"
            database: "{{ telegraf_influxdb_database }}"
            precision: "{{ telegraf_influxdb_precision }}"
            timeout: "{{ telegraf_influxdb_timeout }}"
            namedrop: ["rabbitmq*"]

        -
          type: influxdb
          config:
            urls: "{{ telegraf_influxdb_urls }}"
            database: "rabbitmq"
            precision: "{{ telegraf_influxdb_precision }}"
            timeout: "{{ telegraf_influxdb_timeout }}"
            namepass: ["rabbitmq*"]
```

**Windows host running MSMQ as described in the inputs section. Targetting different influxdb databases in the same host using namepass tags.**

```yaml
      telegraf_outputs:
        -
          type: influxdb
          config:
            urls: "{{ telegraf_influxdb_urls }}"
            database: "{{ telegraf_influxdb_database }}"
            precision: "{{ telegraf_influxdb_precision }}"
            timeout: "{{ telegraf_influxdb_timeout }}"
            namepass: ["win*"]

        -
          type: influxdb
          config:
            urls: "{{ telegraf_influxdb_urls }}"
            database: "msmq"
            precision: "{{ telegraf_influxdb_precision }}"
            timeout: "{{ telegraf_influxdb_timeout }}"
            namepass: ["msmq*"]
```

License
-------

MIT
