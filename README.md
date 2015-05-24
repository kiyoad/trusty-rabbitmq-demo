# trusty-rabbitmq-demo
RabbitMQ cluster demo for Ubuntu trusty by Ansible and Vagrant.

## Abstract

This Vagrantfile will deploy the RabbitMQ cluster on Ubuntu 14.04 LTS virtual machines with VirtualBox.

## Requirements

My development environment is shown below. I think that Ubuntu LinuxBox also works.

    $ cat /etc/redhat-release
    CentOS Linux release 7.1.1503 (Core)
    $ uname -a
    Linux zouk.local 3.10.0-229.4.2.el7.x86_64 #1 SMP Wed May 13 10:06:09 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
    $ cat /proc/meminfo | head -6
    MemTotal:       16243556 kB
    MemFree:          153060 kB
    MemAvailable:    7363000 kB
    Buffers:            4076 kB
    Cached:          7484732 kB
    SwapCached:        15380 kB
    $ vboxmanage --version
    4.3.28r100309
    $ vagrant --version
    Vagrant 1.7.2
    $ ansible --version
    ansible 1.9.1
      configured module search path = None
    $ vagrant box list
    ubuntu/trusty64 (virtualbox, 0)

My Vagrant base box 'ubuntu/trusty64' obtained from the following.
https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box

## How to use

1. run `vagrant up` and `vagrant reload`
1. login to *starter* by `vagrant ssh starter`
1. Setup the RabbitMQ cluster by using script like this.

    ```
    vagrant@starter:~$ ./setup-rabbitmq.sh
    ```

1. The following message is a successful example.

    ```
    vagrant@starter:~$ ./setup-rabbitmq.sh
    + ssh sv2 sudo rabbitmqctl stop_app
    The authenticity of host 'sv2.local (10.0.0.72)' can't be established.
    ECDSA key fingerprint is 85:62:c1:36:d8:96:09:59:ed:13:f8:bd:20:89:32:06.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'sv2.local,10.0.0.72' (ECDSA) to the list of known hosts.
    Stopping node rabbit@sv2 ...
    ...done.
    + ssh sv2 sudo rabbitmqctl join_cluster rabbit@sv1
    Clustering node rabbit@sv2 with rabbit@sv1 ...
    ...done.
    + ssh sv2 sudo rabbitmqctl start_app
    Starting node rabbit@sv2 ...
    ...done.
    + ssh sv3 sudo rabbitmqctl stop_app
    The authenticity of host 'sv3.local (10.0.0.73)' can't be established.
    ECDSA key fingerprint is c8:0b:b5:5f:e2:41:6f:52:c3:9c:16:1e:51:73:9a:45.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'sv3.local,10.0.0.73' (ECDSA) to the list of known hosts.
    Stopping node rabbit@sv3 ...
    ...done.
    + ssh sv3 sudo rabbitmqctl join_cluster rabbit@sv2
    Clustering node rabbit@sv3 with rabbit@sv2 ...
    ...done.
    + ssh sv3 sudo rabbitmqctl start_app
    Starting node rabbit@sv3 ...
    ...done.
    + ssh sv1 sudo rabbitmqctl cluster_status
    The authenticity of host 'sv1.local (10.0.0.71)' can't be established.
    ECDSA key fingerprint is ac:c7:54:a9:0a:ba:30:a8:a4:3f:53:91:ff:1a:3c:80.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'sv1.local,10.0.0.71' (ECDSA) to the list of known hosts.
    Cluster status of node rabbit@sv1 ...
    [{nodes,[{disc,[rabbit@sv1,rabbit@sv2,rabbit@sv3]}]},
     {running_nodes,[rabbit@sv3,rabbit@sv2,rabbit@sv1]},
     {partitions,[]}]
    ...done.
    + ssh sv2 sudo rabbitmqctl cluster_status
    Cluster status of node rabbit@sv2 ...
    [{nodes,[{disc,[rabbit@sv1,rabbit@sv2,rabbit@sv3]}]},
     {running_nodes,[rabbit@sv3,rabbit@sv1,rabbit@sv2]},
     {partitions,[]}]
    ...done.
    + ssh sv3 sudo rabbitmqctl cluster_status
    Cluster status of node rabbit@sv3 ...
    [{nodes,[{disc,[rabbit@sv1,rabbit@sv2,rabbit@sv3]}]},
     {running_nodes,[rabbit@sv1,rabbit@sv2,rabbit@sv3]},
     {partitions,[]}]
    ...done.
    + ssh sv1 sudo rabbitmqctl set_policy ha-all ''\''^(?!amq\.).*'\''' ''\''{"ha-mode":' '"all"}'\'''
    Setting policy "ha-all" for pattern "^(?!amq\\.).*" to "{\"ha-mode\": \"all\"}" with priority "0" ...
    ...done.
    + ssh sv1 sudo rabbitmqctl list_policies
    Listing policies ...
    /       ha-all  all     ^(?!amq\\.).*   {"ha-mode":"all"}       0
    ...done.
    + amqp-declare-queue --url amqp://guest:guest@sv1 -q testq
    testq
    + amqp-publish --url amqp://guest:guest@sv1 -r testq -b ' *** from_sv1 *** '
    + amqp-publish --url amqp://guest:guest@sv2 -r testq -b ' *** from_sv2 *** '
    + amqp-publish --url amqp://guest:guest@sv3 -r testq -b ' *** from_sv3 *** '
    + ssh sv1 sudo rabbitmqctl stop_app
    Stopping node rabbit@sv1 ...
    ...done.
    + amqp-get --url amqp://guest:guest@sv2 -q testq
     *** from_sv1 *** + amqp-get --url amqp://guest:guest@sv3 -q testq
     *** from_sv2 *** + ssh sv1 sudo rabbitmqctl start_app
    Starting node rabbit@sv1 ...
    ...done.
    + amqp-get --url amqp://guest:guest@sv1 -q testq
     *** from_sv3 *** + amqp-delete-queue --url amqp://guest:guest@sv1 -q testq
    0
    vagrant@starter:~$
    ```

1. My RabbitMQ cluster version is show below.

    ```
    vagrant@starter:~$ ssh sv1 sudo rabbitmqctl status
    Status of node rabbit@sv1 ...
    [{pid,1114},
     {running_applications,
         [{rabbitmq_management,"RabbitMQ Management Console","3.2.4"},
          {rabbitmq_web_dispatch,"RabbitMQ Web Dispatcher","3.2.4"},
          {webmachine,"webmachine","1.10.3-rmq3.2.4-gite9359c7"},
          {mochiweb,"MochiMedia Web Server","2.7.0-rmq3.2.4-git680dba8"},
          {rabbitmq_management_agent,"RabbitMQ Management Agent","3.2.4"},
          {rabbit,"RabbitMQ","3.2.4"},
          {os_mon,"CPO  CXC 138 46","2.2.14"},
          {mnesia,"MNESIA  CXC 138 12","4.11"},
          {amqp_client,"RabbitMQ AMQP Client","3.2.4"},
          {inets,"INETS  CXC 138 49","5.9.7"},
          {xmerl,"XML parser","1.3.5"},
          {sasl,"SASL  CXC 138 11","2.3.4"},
          {stdlib,"ERTS  CXC 138 10","1.19.4"},
          {kernel,"ERTS  CXC 138 10","2.16.4"}]},
     {os,{unix,linux}},
     {erlang_version,
         "Erlang R16B03 (erts-5.10.4) [source] [64-bit] [async-threads:30] [kernel-poll:true]\n"},
     {memory,
         [{total,39492152},
          {connection_procs,5264},
          {queue_procs,5264},
          {plugins,446112},
          {other_proc,13517920},
          {mnesia,61376},
          {mgmt_db,8880},
          {msg_index,33704},
          {other_ets,1037488},
          {binary,382248},
          {code,19586901},
          {atom,703377},
          {other_system,3703618}]},
     {vm_memory_high_watermark,0.4},
     {vm_memory_limit,839688192},
     {disk_free_limit,50000000},
     {disk_free,38859227136},
     {file_descriptors,
         [{total_limit,924},{total_used,3},{sockets_limit,829},{sockets_used,1}]},
     {processes,[{limit,1048576},{used,193}]},
     {run_queue,0},
     {uptime,763}]
    ...done.
    vagrant@starter:~$
    ```

## Restrictions

1. `default_user` and `default_pass` are not changed.(guest:guest)

## Memo

1. When the entire cluster is brought down, the last node to go down must be the first node to be brought online. If this doesn't happen, the nodes will wait 30 seconds for the last disc node to come back online, and fail afterwards. If the last node to go offline cannot be brought back up, it can be removed from the cluster using the forget_cluster_node command - consult the rabbitmqctl manpage for more information. (see https://www.rabbitmq.com/clustering.html)

1. If all cluster nodes stop in a simultaneous and uncontrolled manner (for example with a power cut) you can be left with a situation in which all nodes think that some other node stopped after them. In this case you can use the force_boot command on one node to make it bootable again - consult the rabbitmqctl manpage for more information. (see https://www.rabbitmq.com/clustering.html)
