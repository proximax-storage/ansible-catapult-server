Run Vagrant Ubuntu VM
```bash
cd ansible-catapult-server/example/vagrant
vagrant up
```

Check if you can connect to the host
```bash
cd ansible-catapult-server/example
ansible-playbook -i inventories/local-virtualbox.yml -b debug.yml -u vagrant -v
```

Download the `ansible-catapult-server` role
```bash
cd ansible-catapult-server/example
ansible-galaxy install --roles-path=roles -r requirements.yml
```

Run the catapult-server playbook
```bash
cd ansible-catapult-server/example
ansible-playbook -i inventories/local-virtualbox.yml -b catapult-server-playbook.yml -u vagrant -v
```