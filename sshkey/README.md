### Generate SSH Key on Controller

If you do not already have an SSH key on the controller:

```bash
ssh-keygen -t ed25519
```
##

### Add Public Key to Manage nodes
```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.yml setup-ssh-key.yml
```
### Manual Method
```bash
ssh-copy-id vagrant@192.168.5.140
```

##

### Add a User Public Key to Manage nodes (not all of them) 
ansible-playbook -i ansible/inventories/inventory.yml add-ali-key.yml --limit "web1,db"

