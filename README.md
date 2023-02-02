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


**For ansible/inventories/setup.yml**

```yaml
all:
 vars:
   ansible_user: centos
   ansible_ssh_private_key_file: /tmp/id_rsa
 children:
   prod:
     hosts: mathis.medard.takima.cloud
```

**ansible all -i inventories/setup.yml -m ping**

```json
mathis.medard.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

