# ansible-example-app

This repository bootstraps environment for python app. Currently it has been optimized for vagrant environment.
Head over to [Test](#test) to perform task testing. 

Current scheme:
```
              +---------------+
     +--------+ example|app|1 |
     |        +---------------+
     |
+----+--+     +---------------+
|Gateway+-----+ example|app|2 |
+----+--+     +---------------+
     |
     |        +---------------+
     +--------+ example|app|3 |
              +---------------+
```

Accomplished tasks:

* Compatibility with Debian 9/10 hosts (All testing is done with Debian 9/10)
* Use of Nginx as a reverse proxy (See group_vars/gw-group)
* Use of Gunicorn as a Python WSGI HTTP Server (See roles/internal/ansible-example-app-deployment)
* Protect all publicly exposed endpoints using basic authentication (See group_vars/gw_group)
* Configure firewall rules to only allow incoming traffic to HTTP/HTTPS ports from Cloudflare’s IP’s (See group_vars/gw_group).
It has been commented, so we could perform tests in vagrant. 

Additional tasks:

* SSH is hardened. No SSH to root, no SSH with user:pass (only key-pair). Tcp forwarding disabled
* Only administrators are allowed to sudo to root
* App VMs 8000 port is only accessible from GW to ensure there is only point of HTTP traffic (GW)
* Closed all ports except 22, 80 in GW and 22, 8000 in App VMs

What could be improved:

* Additional VM for prometheus, grafana and alert manager
* Provision Node exporter, nginx exporter, gunicorn exporter to VMs, so we could scrape and aggregate cluster status
* Configure NTP to ensure time across app distribution is correct
* Implement centralized logging solution (graylog, datadog, ELK)
* Move APP deployment to pipeline
* Dockerize Application to prevent or at least minimize inconsistencies across environments

## Install

Install roles
```sh
ansible-galaxy install -p roles/external -r roles/requirements.yml
```
## Provision

Currently these command won't work as host machine doesn't know what is example-app-[1:3] and gateway (because of DNS for sure).

Provision gateway
```sh
ansible-playbook playbook.yml -i inventories/test.yml -u user --private-key=path/to/key --limit gw-group
```

Provision app
```sh
ansible-playbook playbook.yml -i inventories/test.yml -u user --private-key=path/to/key --limit example-app-web-group
```

## Test
### Requirements
* Ansible (2.8.5)
* Vagrant (2.2.10)
* Curl
* SSH

Install roles
```sh
ansible-galaxy install -p roles/external -r roles/requirements.yml
```

### Perform Debian 9 (strech test)
Provision test environment (It will create virtual machines and provision ansible playbooks)
```sh
DISTRIBUTION=stretch vagrant up --provision
```

Curl gateway
```sh
curl 10.240.0.11     
<html>
<head><title>401 Authorization Required</title></head>
<body bgcolor="white">
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.10.3</center>
</body>
</html>
```

Curl gateway (with basic auth)
```sh
curl -u test:test 10.240.0.11
Hello, World!
```

Curl App VMs (will timeout due to FW)
```sh
for i in {1..3}; do echo "Curling to node ${i}"; curl 10.240.0.2${i} --connect-timeout 3; done
```

SSH to nodes
```sh
ssh test@10.240.0.11 -i files/test_key  -o "StrictHostKeyChecking=no"
ssh test@10.240.0.21 -i files/test_key  -o "StrictHostKeyChecking=no"
ssh test@10.240.0.22 -i files/test_key  -o "StrictHostKeyChecking=no"
ssh test@10.240.0.23 -i files/test_key  -o "StrictHostKeyChecking=no"
```

Sudo to app
```sh
test@example-app-3:~$ sudo -iu app
[sudo] password for test: testtest
app@example-app-3:~$ 
```

Sudo to root
```sh
test@example-app-3:~$ sudo -iu root
[sudo] password for test: testtest
Sorry, user test is not allowed to execute '/bin/bash' as root on example-app-3.
```

Destroy 
```
DISTRIBUTION=stretch vagrant destroy -f
```

### Perform Debian 10 (buster test)
Provision test environment (It will create virtual machines and provision ansible playbooks)
```sh
DISTRIBUTION=buster vagrant up --provision
```

Curl gateway
```sh
curl 10.240.0.11     
<html>
<head><title>401 Authorization Required</title></head>
<body bgcolor="white">
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.10.3</center>
</body>
</html>
```

Curl gateway (with basic auth)
```sh
curl -u test:test 10.240.0.11
Hello, World!
```

Curl App VMs (will timeout due to FW)
```sh
for i in {1..3}; do echo "Curling to node ${i}"; curl 10.240.0.2${i} --connect-timeout 3; done
```

SSH to nodes
```sh
ssh test@10.240.0.11 -i files/test_key  -o "StrictHostKeyChecking=no"
ssh test@10.240.0.21 -i files/test_key  -o "StrictHostKeyChecking=no"
ssh test@10.240.0.22 -i files/test_key  -o "StrictHostKeyChecking=no"
ssh test@10.240.0.23 -i files/test_key  -o "StrictHostKeyChecking=no"
```

Sudo to app
```sh
test@example-app-3:~$ sudo -iu app
[sudo] password for test: testtest
app@example-app-3:~$ 
```

Sudo to root
```sh
test@example-app-3:~$ sudo -iu root
[sudo] password for test: testtest
Sorry, user test is not allowed to execute '/bin/bash' as root on example-app-3.
```

Destroy 
```
DISTRIBUTION=buster vagrant destroy -f
```
