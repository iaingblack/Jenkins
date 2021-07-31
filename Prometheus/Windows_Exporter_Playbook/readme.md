# Basic Setup
```
username='myusername'
password='MachinePWD!'
cd WMI_Exporter_Playbook
```

# Test Hosts Respond to a Ping
```
ansible servers -i "variables/azure-nprd-windows-hosts.yml" -m win_ping  -e ansible_user=$username -e ansible_password=$password
```

# Run
```
ansible-playbook playbooks/install-wmi-exporter.yml -i "variables/hosts.yml" -e ansible_user=$username -e ansible_password=$password
```
