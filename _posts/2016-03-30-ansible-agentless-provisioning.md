---
layout: post
title: Ansible - agentless provisioning
name: ansible-agentless-provisioning
categories: devops
tags: [ansible, docker]
author: jeqo
---

Ansible is an automation tool that is recognized for be simple and
powerful at the same time. From my experience, I can say this is mainly
because of its scripting language: YAML, and its agentless architecture.

## YAML and Ansible components

> "YAML is a human friendly data serialization standard for all programming
  languages" (Source: http://yaml.org/)

This means that is actually really easy to understand and start working
with it. For example:

```yaml
- hosts: webserver
  tasks:
    - package: apache
        state: latest
```

This **"playbook"** says that a *webserver* host have 1 task: install latest
Apache package, using a package **"module"**.

Pretty simple eh?

To check how powerful Ansible can be, take a look on their Module Index:
http://docs.ansible.com/ansible/modules_by_category.html

To achieve reusability: These tasks can be grouped
as **"roles"**, that are a compilation of tasks
to execute a common goal. e.g: a Java role to install Java SDK on your
node.

Those are the main components of Ansible: Playbooks, Modules, and Roles.

## Agentless architecture

This means that you don't need a "ansible-client" in your node to run
tasks, you can have a master that says what you need to run on your nodes.
This is an important feature compared to other tools where
you need a "\***-client" to make your node translate and run commands:
https://www.ansible.com/benefits-of-agentless-architecture

You don't need a client but you need some packages. But this packages
are ssh and python-related and they are very common:
http://docs.ansible.com/ansible/intro_installation.html#managed-node-requirements

Ansible also have a default "push" approach, where a master sends commands
to your nodes. This is also different from other tools that are based on a
"pull" approach, where the node asks for commands, although this is also
possible with Ansible:
http://docs.ansible.com/ansible/playbooks_intro.html#ansible-pull

There is a final feature I would like to mention: Connection Type.
By default Ansible relies on SSH to send commands to your nodes, but
there are cases where SSH is not an option or you don't need it:
local commands, Windows, Docker.

In these cases, connection type option enables your playbook to run
commands using WinRM in the case of Windows,
or Docker execute commands on Docker containers, or just run local
commands in your workstation.

Let's check some code:

I have implemented a Ansible Role to install Java some time ago:
https://github.com/jeqo/ansible-role-java

Just to explain what it does, let's check the main task file:

```yaml
---
  - debug:
      msg: "This Java Provider will be installed: {{ java_provider }}"

  - include: install-{{ java_provider }}.yml

  - include: set-java-home.yml
```

It will show a message, include a task depending on "java_provider"
variable and finally set JAVA_HOME variable.

Also this role has a "tests" directory where you can add playbooks
to test your role:

```yaml
- name: test install openjdk jdk 8 on centos 7
  hosts: test01
  roles:
    - role: java
      java_provider: openjdk
      java_version: 8
      java_type: jdk
- name: test install openjdk jre 8 on centos 7
  hosts: test02
  roles:
    - role: java
      java_provider: openjdk
      java_version: 8
      java_type: jre
# more tests...
```

And I test this playbooks using Vagrant and VirtualBox:

```ruby
Vagrant.configure(2) do |config|

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "test.yml"
    ansible.galaxy_role_file = "roles.yml"
  end

  config.vm.define "test01" do |node|
    node.vm.box = "jeqo/ansible-centos7"
  end

  config.vm.define "test02" do |node|
    node.vm.box = "jeqo/ansible-centos7"
  end

  # more test nodes...
end
```

So, lets test that OpenJDK 8 is running OK in Centos:

```
vagrant up test01
...

PLAY [test install openjdk jdk 8 on centos 7] **********************************

TASK [setup] *******************************************************************
ok: [test01]

TASK [java : debug] ************************************************************
ok: [test01] => {
    "msg": "This Java Provider will be installed: openjdk"
}

TASK [java : include] **********************************************************
included: /home/jeqo/dev/jeqo/ansible-role-java/tests/roles/java/tasks/install-openjdk.yml for test01

TASK [java : set_fact] *********************************************************
skipping: [test01]

TASK [java : set_fact] *********************************************************
ok: [test01]

TASK [java : set_fact] *********************************************************
skipping: [test01]

TASK [java : set_fact] *********************************************************
ok: [test01]

TASK [java : install openjdk (debian)] *****************************************
skipping: [test01]

TASK [java : install openjdk (redhat)] *****************************************

```

But one thing I always want is to reuse this roles on Docker containers,
without prepare a Container with SSH, that is recognized as an
anti-pattern: https://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/

So, since Ansible 2.0 a Docker connection type is included OOTB, and
I give it a try: https://github.com/jeqo/poc-ansible-docker

I added a playbook to create a container:

```yaml
- hosts: 127.0.0.1
  connection: local
  tasks:
    - name: my container
      docker:
        name: poccontainer
        image: centos
        command: sleep infinity
        state: started
```

Here I'm using "connection: local" to execute commands locally.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
a5e49bd032be        centos              "sleep infinity"    About an hour ago   Up About an hour                        poccontainer
```

And once I have a container running, I can run this playbook:

```yaml
- hosts: poccontainer
  connection: docker
  pre_tasks:
    - package: name=sudo
    - command: "sed -i -e \"s/Defaults    requiretty.*/ #Defaults    requiretty/g\" /etc/sudoers"
  roles:
    - role: java
      java_provider: openjdk
      java_type: jdk
      java_version: 8
```

Pre-tasks are required to install sudo package and configure tty. And
then run role as usual:

```
$ ansible-playbook provisioning.yml -vvvv
Using /home/jeqo/dev/jeqo/poc-ansible-docker/ansible.cfg as config file
Loaded callback default of type stdout, v2.0
2 plays in provisioning.yml

PLAY ***************************************************************************

TASK [setup] *******************************************************************
ESTABLISH DOCKER CONNECTION FOR USER: None
<poccontainer> EXEC ['/usr/bin/docker', 'exec', '-i', u'poccontainer', '/bin/sh', '-c', '/bin/sh -c \'( umask 22 && mkdir -p "` echo $HOME/.ansible/tmp/ansible-tmp-1459355431.02-32251179247729 `" && echo "` echo $HOME/.ansible/tmp/ansible-tmp-1459355431.02-32251179247729 `" )\'']
<poccontainer> PUT /tmp/tmpNCOaxi TO /root/.ansible/tmp/ansible-tmp-1459355431.02-32251179247729/setup
<poccontainer> EXEC ['/usr/bin/docker', 'exec', '-i', u'poccontainer', '/bin/sh', '-c', u'/bin/sh -c \'LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 LC_MESSAGES=en_US.UTF-8 /usr/bin/python /root/.ansible/tmp/ansible-tmp-1459355431.02-32251179247729/setup; rm -rf "/root/.ansible/tmp/ansible-tmp-1459355431.02-32251179247729/" > /dev/null 2>&1\'']
ok: [poccontainer]
```

## Conclusions

- This samples show how versatile Ansible is, using roles and connection
type. But there are more platforms where Ansible can fit, as with AWS:
https://aws.amazon.com/blogs/apn/getting-started-with-ansible-and-dynamic-amazon-ec2-inventory-management/ and other Cloud platforms: http://docs.ansible.com/ansible/list_of_cloud_modules.html

- One question can be: Is this a replacement of Dockerfile? Maybe,
depends on you. Dockerfile are very simple and only works with Docker.
Dockerfile also has a nice feature to create an image each step, so
you can distribute images easily. This is missing in Ansible, where
you execute commands on a running Docker container. Also Ansible
is missing commit and push tasks to put containers on Docker Hub,
but you can replace it with local commands as here:

```yaml
- hosts: 127.0.0.1
  connection: local
  tasks:
    - name: commit
      command: docker commit poccontainer
```

Although Ansible also have a module to run Dockerfiles: http://docs.ansible.com/ansible/docker_image_module.html

Hope this helps you to get started with Ansible and Docker.
