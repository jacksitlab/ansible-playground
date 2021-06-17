# ansible playground

  * ansible modules: [https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html)

## Setup

```
                 |---------------|
                 | ansible-host  |
                 |---------------|



    |---------------|           |---------------|
    |   server1     |           |    server2    |
    |---------------|           |---------------|


```

All scripts are executed on ansible host and deployed on server1 and/or server2. In my case two raspberry pis  with enabled ssh public key access and switched python versio to python3 and accessable by hostname.
```
$ sudo rm /usr/bin/python
$ sudo ln -s /usr/bin/python3 /usr/bin/python
```


Prerequisites:

 * Host:
   * ssh-client
   * python3 (v3.8.5)
   * ansible (v2.9.6)
 * servers:
   * openssh-server
   * python3 (v3.7.3)

 * establish at least once ssh connection to have server keys in .ssh/known_hosts

### 01 - update system packages

modules:
 * [apt](https://docs.ansible.com/ansible/2.9/modules/apt_module.html#apt-module)
 * [ping](https://docs.ansible.com/ansible/2.9/modules/ping_module.html#ping-module)

first test if servers are reachable
```
$ ansible all -m ping -i inventory.ini
```

```
$ ansible-playbook -i inventory.ini playbook.yml
```

### 02 - install git

modules:
 * [apt](https://docs.ansible.com/ansible/2.9/modules/apt_module.html#apt-module)

```
$ ansible-playbook -i inventory.ini playbook.yml
```
output:
```
PLAY [Update system packages] ***************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
[WARNING]: Platform linux on host server1 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could change
this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
ok: [server1]
[WARNING]: Platform linux on host server2 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could change
this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
ok: [server2]

TASK [update packages] **********************************************************************************************************************************************************
changed: [server2]
changed: [server1]

PLAY RECAP **********************************************************************************************************************************************************************
server1                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
server2                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### 03 - install and configure docker

modules:
 * [apt](https://docs.ansible.com/ansible/2.9/modules/apt_module.html#apt-module)
 * [shell](https://docs.ansible.com/ansible/2.9/modules/shell_module.html#shell-module)
 * [service](https://docs.ansible.com/ansible/2.9/modules/service_module.html#service-module)

```
$ ansible-playbook -i inventory.ini playbook.yml
```


### 04 - run web server with docker-compose

modules:
  * [copy](https://docs.ansible.com/ansible/2.9/modules/copy_module.html#copy-module)
  * [docker_compose](https://docs.ansible.com/ansible/2.9/modules/docker_compose_module.html#docker-compose-module)
  * [stat](https://docs.ansible.com/ansible/2.9/modules/stat_module.html#stat-module)

```
$ ansible-playbook -i inventory.ini playbook.yml
```


### 06 - backup gitlab server

modules:
  * [copy](https://docs.ansible.com/ansible/2.9/modules/copy_module.html#copy-module)
  * [docker_compose](https://docs.ansible.com/ansible/2.9/modules/docker_compose_module.html#docker-compose-module)
  * [stat](https://docs.ansible.com/ansible/2.9/modules/stat_module.html#stat-module)

```
$ ansible-playbook -i inventory.ini  -e @vars.yml playbook.yml
```

### 07 - backup nextcloud server

modules:
  * [copy](https://docs.ansible.com/ansible/2.9/modules/copy_module.html#copy-module)
  * [docker_compose](https://docs.ansible.com/ansible/2.9/modules/docker_compose_module.html#docker-compose-module)
  * [shell](https://docs.ansible.com/ansible/2.9/modules/shell_module.html#shell-module)
  * [stat](https://docs.ansible.com/ansible/2.9/modules/stat_module.html#stat-module)

```
$ ansible-playbook -i inventory.ini  -e @vars.yml playbook.yml
```

modules:
  * [async](https://docs.ansible.com/ansible/latest/user_guide/playbooks_async.html)
  * [copy](https://docs.ansible.com/ansible/2.9/modules/copy_module.html#copy-module)
  * [docker_compose](https://docs.ansible.com/ansible/2.9/modules/docker_compose_module.html#docker-compose-module)
  * [shell](https://docs.ansible.com/ansible/2.9/modules/shell_module.html#shell-module)
  * [stat](https://docs.ansible.com/ansible/2.9/modules/stat_module.html#stat-module)


what does it do exactly:

  * prepare variables
    * get fix timestamp
    * create filenames for archives and targets from timestamp (db_, public_html_, data_)
  * create backup archives on remote cloud server (server-group: nextcloud-servers)
    * first check if filenames are already exsting
    * if all 3 files are existing, stop creating backup files
    * if not get container image names from running instances
    * check if containers are running/existing
    * fail if one container is not there
    * put nextcloud into maintenance mode
    * create public_html backup
    * stop database container
    * create database backup
    * create data backup
    * restart database container
    * set nextcloud back to operational mode
  * download backup archives (server-group:backup-servers)
    * check file infos on backup target
    * if all files are existing, do not download
    * download db, public_html and data archive

  FAQ:

  **Why creating archives within docker image?** This is the better way to get all the files with the same user rights as the container itself get's them during runtime.

### 08 - Run sdnr and wait for ready state

### 09 - Playing with vars

  references:

    * https://www.mydailytutorials.com/ansible-arithmetic-operations/