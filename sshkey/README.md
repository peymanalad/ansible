##
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.yml setup-ssh-key.yml

## 
ansible-playbook -i ansible/inventories/inventory.yml add-ali-key.yml --limit "web1,db"

