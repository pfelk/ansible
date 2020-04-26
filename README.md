# ansible-pfelk [![Build Status](https://travis-ci.org/3ilson/ansible-pfelk.svg?branch=master)](https://travis-ci.org/3ilson/ansible-pfelk)
Ansible playbook automation for deploying pfelk

![PyPI - Python Version](https://img.shields.io/pypi/pyversions/ansible)

You can deploy using [Ansible Galaxy Collection](https://galaxy.ansible.com/fktkrt/ansible_pfelk) or with using the manual deploy process.

Note: When using the Ansible Galaxy Collection, you have to manually create a hosts file, and use the playbook provided in this repository, finally configure `GeoIP.conf`.

## Prerequisites 

### Prerequisites on control nodes

Currently Ansible can be run from any machine with Python 2 (version 2.7) or Python 3 (versions 3.5 and higher) installed. Windows is not supported for the control node.

Take a look at the following link regarding further details on initial requirements: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

### Add Ansible apt repository and install the package for Ubuntu
```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

### Create Ansible configuration (optional)

```
$ vi ~/.ansible.cfg

[defaults]
# disable key check if host is not initially in 'known_hosts'
host_key_checking = False

[ssh_connection]
# if True, make ansible use scp if the connection type is ssh (default is sftp)
scp_if_ssh = True
```

### Prerequisites on managed nodes

On the managed nodes, you need a way to communicate, which is normally ssh. By default this uses sftp. If that’s not available, you can switch to scp in ansible.cfg. You also need Python 2 (version 2.6 or later) or Python 3 (version 3.5 or later).

### Tree of Ansible setup
```
ansible-pfelk/
├── deploy-stack.yml
├── group_vars
│   └── all.yml
├── hosts
└── roles
    ├── elasticsearch
    │   ├── files
    │   │   └── elasticsearch.yml
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    ├── java
    │   └── tasks
    │       └── main.yml
    ├── kibana
    │   ├── files
    │   │   └── kibana.yml
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    ├── logstash
    │   ├── files
    │   │   ├── 01-inputs.conf
    │   │   ├── 05-firewall.conf
    │   │   ├── 10-others.conf
    │   │   ├── 20-suricata.conf
    │   │   ├── 25-snort.conf
    │   │   ├── 30-geoip.conf
    │   │   ├── 40-dns.conf
    │   │   ├── 45-cleanup.conf
    │   │   ├── 50-outputs.conf
    │   │   ├── patterns
    │   │   │   └── pfelk.grok
    │   │   └── template
    │   │       └── pf-geoip-template.json
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    └── maxmind
        ├── handlers
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            └── GeoIP.conf.j2

```
## Deploy with Ansible Galaxy Collections
```
$ ansible-galaxy collection install fktkrt.ansible_pfelk
```

## Manual Ansible playbook
## Deploy playbook 
### Clone the repository

```
$ git clone https://github.com/3ilson/ansible-pfelk.git
```

### Define the host you want to deploy the ELK stack to
Provide your target IP address in `ansible-pfelk/hosts` under `elk`, the ELK stack will be installed on this target.

### Configure your inputs file

#### Enter your pfSense/OPNsense IP address (01-inputs.conf)

```
Change line 12; the "if [host] =~ ..." should point to your pfSense/OPNsense IP address
Change line 15; rename "firewall" (OPTIONAL) to identify your device (i.e. backup_firewall)
Change line 18-27; (OPTIONAL) to point to your second PF IP address or ignore
```

#### Revise/Update w/pf IP address (01-inputs.conf)

```
For pfSense uncommit line 34 and commit out line 31
For OPNsense uncommit line 31 and commit out line 34
```

### Change current folder to ansible-pfelk/ then deploy the stack
```
$ cd ansible-pfelk/
$ ansible-playbook -i hosts --ask-become deploy-stack.yml
```

This will take care of the following tasks:
 - install java
 - install maxmind
   - download GeoIP databases
   - setup a cron job for automated updates
 - install elasticsearch
 - install kibana
 - install logstash
   - copy the `.conf` files, patterns and templates to their corresponding locations

### Manually register and configure Maxmind

### Configure Maxmind
- Create a Max Mind Account @ https://www.maxmind.com/en/geolite2/signup
- Login to your Max Mind Account; navigate to "My License Key" under "Services" and Generate new license key
```
$ sudo nano /etc/GeoIP.conf
```
- Modify lines 7 & 8 as follows (without < >):
```
AccountID <Input Your Account ID>
LicenseKey <Input Your LicenseKey>
```
## Finish the configuration

You can follow the steps starting with the Firewall section at https://github.com/a3ilson/pfelk/blob/master/install/configuration.md

## Troubleshooting

### Testing the playbook with dry-run
Include `--check` flag.
 - run `ansible-playbook -i hosts --check deploy-stack.yml`

### Deploy to localhost
To deploy the playbook to your local machine you need the do following:
 - install and setup `openssh`on your machine
 - if you choose not to use ssh keys, install `sshpass` for auth purposes
 - under `hosts` define your IP as `localhost`
 - run the playbook with: `ansible-playbook -i hosts --ask-pass --ask-become deploy-stack.yml`

### Enable verbose mode to debug problems
Include `-vvvv` flag.
 - run `ansible-playbook -i hosts --ask-pass --ask-become -vvvv deploy-stack.yml`
