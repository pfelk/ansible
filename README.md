# ansible-pfelk [![Build Status](https://travis-ci.org/3ilson/ansible-pfelk.svg?branch=master)](https://travis-ci.org/3ilson/ansible-pfelk)
Ansible playbook automation for deploying pfelk

![PyPI - Python Version](https://img.shields.io/pypi/pyversions/ansible)

You can deploy using [Ansible Galaxy Collection](https://galaxy.ansible.com/fktkrt/ansible_pfelk) or with using the manual deploy process.

Note: When using the Ansible Galaxy Collection, you have to manually create a hosts file, and use the playbook provided in this repository.

## Prerequisites 

### Prerequisites on control nodes

Currently Ansible can be run from any machine with Python 2 (version 2.7) or Python 3 (versions 3.5 and higher) installed. This includes linux distributions e.g. Red Hat, Debian, CentOS, macOS, any of the BSDs. Windows is not supported for the control node.

> Take a look at the following link regarding further details on initial requirements: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

### Add Ansible apt repository and install the package for Ubuntu
```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

> Installation details for further distributions are available [on the official site](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#selecting-an-ansible-version-to-install).

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

To interact with your managed nodes, you will only need ssh. You don't have to install any other additonal agent on those hosts.

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
    │   │   ├── 02-types.conf
    │   │   ├── 03-filter.conf   
    │   │   ├── 05-firewall.conf
    │   │   ├── 10-others.conf
    │   │   ├── 15-squid.conf    
    │   │   ├── 20-suricata.conf
    │   │   ├── 25-snort.conf
    │   │   ├── 30-geoip.conf
    │   │   ├── 35-rules-desc.conf    
    │   │   ├── 45-cleanup.conf
    │   │   ├── 50-outputs.conf
    │   │   ├── patterns
    │   │   │   └── pfelk.grok
    │   │   └── template
    │   │       └── pf-geoip.json
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

### Configure MaxMind geoipupdate
- Create a MaxMind account at: https://www.maxmind.com/en/geolite2/signup
- Login to your MaxMind account; navigate to "My License Key" under "Services" and Generate new license key
- Configure your credentials at playbook level in `deploy-stack.yml`:
```
vars:
  geoipupdate_account_id: 123
  geoipupdate_license_key: "ABCDEF"
```

### Configure your inputs file

#### Enter your pfSense/OPNsense IP address (02-types.conf)

Change line 5:
the `[host] == "192.168.9.1" ` should point to your pfSense/OPNsense IP address

Change line 8:
rename `firewall` (optional) to identify your device (i.e. backup_firewall)

Configure line 11-20 (optional) to point to your second PF IP address if you have any.

At the following section within the 03-filter.conf file, lines 4 & 5

```
# 03-filter.conf
filter {
  grok {
    #OPN# match => {"message" => "<(?<[priority][id]>.*)>%{SYSLOGTIMESTAMP:[event][created]} %{SYSLOGHOST:[observer][name]} %{GREEDYDATA:filter_message}"} #OPNSense#
    #PF# match => {"message" => "<(?<[priority][id]>.*)>%{SYSLOGTIMESTAMP:[event][created]} %{GREEDYDATA:filter_message}"} #pfSense#
...
```
uncomment the instance type's match line you want to monitor (e.g. remove #PF# or #OPN#


### Change current folder to ansible-pfelk/ then deploy the stack
```
$ cd ansible-pfelk/
$ ansible-playbook -i hosts --ask-become deploy-stack.yml
```

This will take care of the following tasks:
 - install java
 - install maxmind geoipupdate
   - download GeoIP databases
   - setup a cron job for automated updates
 - install elasticsearch
 - install kibana
 - install logstash
   - copy the `.conf` files, patterns and templates to their corresponding locations

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
