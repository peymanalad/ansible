### Add Public Key to Manage nodes
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.yml setup-ssh-key.yml
### Manual Method
ssh-copy-id vagrant@192.168.5.140
##

### Add a User Public Key to Manage nodes (not all of them) 
ansible-playbook -i ansible/inventories/inventory.yml add-ali-key.yml --limit "web1,db"

