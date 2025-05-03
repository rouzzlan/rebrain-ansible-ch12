upload files

```bash
ansible-playbook upload-files_configure.yaml -i hosts.ini
```

run on ansible controller

```bash
ansible-playbook ./DVPS-ANS-12/install_nginx.yml -i inventory.yaml
```

other

```bash
sudo chown user:user /home/user/task/
```

list installed modules

```bash
ansible-galaxy list
```

lint 

```bash
ansible-lint install_nginx.yml
```