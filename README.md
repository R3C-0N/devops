# TP 3

## Ansible

### 3-1 Document your inventory and base commands

**Script ping**
```bash
bash ping.sh
```

**Script copy key**
```bash
bash copy_key.sh
```


**For file ansible/inventories/setup.yml**

```yaml
all:
 vars:
   ansible_user: centos
   ansible_ssh_private_key_file: /tmp/id_rsa
 children:
   prod:
     hosts: mathis.medard.takima.cloud
```

**Let's ping the server now**

```bash
$ ansible all -i inventories/setup.yml -m ping
mathis.medard.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

**To get distribution**

```bash
$ ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
mathis.medard.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/centos-release",
        "ansible_distribution_file_variety": "CentOS",
        "ansible_distribution_major_version": "8",
        "ansible_distribution_release": "Stream",
        "ansible_distribution_version": "8",
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```
----> Ansible distribution = "CentOS"

On a donc une distribution CentOS.

**Let's remove apache now**
```bash
$ ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become
mathis.medard.takima.cloud | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Removed: mod_http2-1.15.7-5.module_el8.6.0+1111+ce6f4ceb.x86_64",
        "Removed: httpd-2.4.37-47.module_el8.6.0+1111+ce6f4ceb.1.x86_64"
    ]
}
```

✅ Changed true, results show that httpd and mod_http2 are removed.

## Playbook

**First playbook**

```yaml
- hosts: all
  gather_facts: false
  become: yes

  tasks:
   - name: Test connection
     ping:
```

**Let's run the playbook now**

```bash
$ ansible-playbook -i inventories/setup.yml playbook.yml


PLAY [all] *****************************************************************************************************************************************************

TASK [Test connection] *****************************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

PLAY RECAP *****************************************************************************************************************************************************
mathis.medard.takima.cloud : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

✅ Ok, we have a connection.

## Advanced playbook

**Second playbook**

```yaml
- hosts: all
  gather_facts: false
  become: yes

# Install Docker
  tasks:
    
  ...
```

**Let's run the playbook now**

```bash
$ ansible-playbook -i inventories/setup.yml advanced_playbook.yml

PLAY [all] *****************************************************************************************************************************************************

TASK [Clean packages] ******************************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [Install device-mapper-persistent-data] *******************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [Install lvm2] ********************************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [add repo docker] *****************************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [Install Docker] ******************************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [install python3] *****************************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [Pip install] *********************************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [Make sure Docker is running] *****************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

PLAY RECAP *****************************************************************************************************************************************************
mathis.medard.takima.cloud : ok=8    changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

✅ Ok, Docker seems to be installed and running.

## Using roles

**We create a role docker**

```bash
$ ansible-galaxy init roles/docker
$ ls roles/docker/
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars
```

**We move the playbook to the role**
    
```bash
$ cp advanced_playbook.yml roles/docker/tasks/main.yml
# We remove the first 5 lines
$ nano roles/docker/tasks/main.yml
```

**We run the playbook**

```bash
$ ansible-playbook -i inventories/setup.yml advanced_playbook.yaml 
PLAY [all] *****************************************************************************************************************************************************

TASK [docker : Clean packages] *********************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [docker : Install device-mapper-persistent-data] **********************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : Install lvm2] ***********************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : add repo docker] ********************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [docker : Install Docker] *********************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : install python3] ********************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : Pip install] ************************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : Make sure Docker is running] ********************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [Install Docker] ******************************************************************************************************************************************

TASK [docker : Clean packages] *********************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [docker : Install device-mapper-persistent-data] **********************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : Install lvm2] ***********************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : add repo docker] ********************************************************************************************************************************
changed: [mathis.medard.takima.cloud]

TASK [docker : Install Docker] *********************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : install python3] ********************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : Pip install] ************************************************************************************************************************************
ok: [mathis.medard.takima.cloud]

TASK [docker : Make sure Docker is running] ********************************************************************************************************************
ok: [mathis.medard.takima.cloud]

PLAY RECAP *****************************************************************************************************************************************************
mathis.medard.takima.cloud : ok=16   changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

✅ Ok, We use role docker, and it works well.

**3-2 Document your playbook**

```yaml
# For each host
- hosts: all
  gather_facts: false
  # Use sudo
  become: yes
  # Use docker role
  roles:
    - docker


# Task install Docker
  tasks:
  - name: Install Docker
    # use docker role, it will use tasks/main.yml
    include_role:
      name: docker
```


## Deploy your app

