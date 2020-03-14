---
name: Bug/issue template
about: Describe this issue template's purpose here.
title: ''
labels: ''
assignees: ''

---

---
name: 'Bug/Error '
about: Create a report to help us improve
title: ''
labels: ''
assignees: ''

---

**Describe the bug**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

**Screenshots**
If applicable, add screenshots to help explain your problem.

**Platform Version informations (please complete the following information):**
 - OS (`printf "$(uname -srm)\n$(cat /etc/os-release)\n"`):
 - Version of Ansible (`ansible --version`):
 - Version of Python (`python --version`):

**Elastic, Logstash, Kibana:**
 - Version of ELK (`dpkg -l elasticsearch|logstash|kibana`):

**Verbose output of running Ansible**
 - Output of failing Ansible task when run with `-vvvv` flag:

**Relevant entries of ELK service logs/status**
 - `service elasticsearch|logstash|kibana status`
 - `tail`ing corresponding logs eg. `/var/log/logstash-plain.log`

**Additional context**
Add any other context about the problem here.
