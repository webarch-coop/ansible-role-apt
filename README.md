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
- name: Install WP CLI
  become: yes

  hosts:
    - stretch_servers

  vars:
    distro: stretch

  roles:
    - wpcli
```

And a `hosts.yml` file that contains lists of servers, for example:

```yml
---
all:
  children:
    stretch_servers:
      hosts:
        cloud.example.com:
        cloud.example.org:
        cloud.example.net:
    jessie_servers:
      hosts:
        host3.example.org:
        host4.example.org:
```

Then it can be run as follows:

```bash
ansible-playbook apt.yml 
```

TODO: conditionally add files to `sources.list.d` for:
* nodejs
* yarn
* docker
