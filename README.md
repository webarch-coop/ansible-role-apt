# Ansible role for Debian Apt configuration

To use this role you need to use Ansible Galaxy to install it into another repository under `galaxy/roles/apt` by adding a `requirements.yml` file in that repo that contains:

```yml
---
- name: apt
  src: https://git.coop/webarch/apt.git
  version: master
  scm: git
```

And a `ansible.cfg` that contains:

```
[defaults]
retry_files_enabled = False
pipelining = True
inventory = hosts.yml
roles_path = galaxy/roles

```

And a `.gitignore` containing:

```
roles/galaxy
```

To pull this repo in run:

```bash
ansible-galaxy install -r requirements.yml --force 
```

The other repo should also contain a `apt.yml` file that contains:

```yml
---
- name: Configure Apt Sources
  become: yes

  hosts:
    - stretch_servers

  roles:
    - apt
```

And a `hosts.yml` file that contains lists of servers and the distros, for example:

```yml
---
all:
  children:
    buster_servers:
      vars:
        apt_distro: buster
      hosts:
        dev.example.org:
          apt_src: yes
    stretch_servers:
      vars:
        apt_distro: squeeze
      hosts:
        cloud.example.com:
          apt_src: yes
        cloud.example.org:
          apt_backports: yes
        cloud.example.net:
          apt_backports: yes
    jessie_servers:
      vars:
        apt_distro: jessie
      hosts:
        host3.example.org:
        host4.example.org:
```

Then it can be run as follows:

```bash
ansible-playbook apt.yml 
```

