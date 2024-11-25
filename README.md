# Webarchitects apt Ansible role

[![pipeline status](https://git.coop/webarch/apt/badges/master/pipeline.svg)](https://git.coop/webarch/apt/-/commits/master)

This role contains an Ansible role for configuring the `apt` [/etc/apt/sources.list](https://wiki.debian.org/SourcesList) or the [DEB822-style format](https://manpages.debian.org/bookworm/apt/sources.list.5.en.html#DEB822-STYLE_FORMAT) file on Debian and Ubuntu LTS.

This role has been automatically tested using [GitLab CI](.gitlab-ci.yml) and [Molecule](molecule/default/molecule.yml) on Debian Trixie, Bookworm, Bullseye and Ubuntu Jammy and Noble, it should also work on Debian Buster and Ubuntu Focal but it hasn't been tested on these versions as the version of Python is too old for recent versions of Ansible and Molecule to be used to run the tests (testing on Bullseye requires old versions of `ansible`, `ansible-lint` and `molecule`).

See also the [upgrade role](https://git.coop/webarch/upgrade), the [Bullseye role](https://git.coop/webarch/bullseye) for upgrading from Debian Buster, the [Bookworm role](https://git.coop/webarch/bookworm) for upgrading from Debian Bullseye and the [localhost repo](https://git.coop/webarch/localhost) which can be used with this role to configure the `sources.list` or the DEB822-style file locally.

## Role variables

Set `apt` to `true` for the tasks in this role to be run. it defaults to `false`.

See the [defaults/main.yml](defaults/main.yml) file for the default variables, the [vars/main.yml](vars/main.yml) file for the preset variables and the [meta/argument_specs.yml](meta/argument_specs.yml) file for the variable specification.

### apt_apticron

A required boolean, when `apt_apticron` is true packages listed in `apt_apticron_pkgs` will be installed, by default this causes a outgoing mailserver to be installed so this needs to be false for servers that have other applications listening on `SMTP` ports, `apt_apticron` defaults to `false`.

### apt_apticron_pkgs

A optional list of `.deb` packages to be installed when `apt_apticron` is `true`, by default `apt_apticron_pkgs` is a list containing one package name, `apticron`.

### apt_distro

An optional string for the distro, if this is not set then it is automatically set to `ansible_distribution_release`, if `apt_distro_switch` is `true` and the disto is set to the next version then the apt sources will be updated in preperation for a manual`update` and `dist-upgrade`.

### apt_distro_switch

A required boolean, set `apt_distro_switch` to `true` when switching distro, eg Bullseye to Bookworm, it will update apt sources list for the server when `apt_distro` is also set to the new version.

### apt_local_facts

A optional boolean, when `apt_local_facts` is `true` the list of files in the `apt_local_facts_files` array will be processed, to install or remove [files/](files/) from the `/etc/ansible/facts.d` directory.

### apt_local_facts_files

A optional list of local fact files and their state, `apt_local_facts_files` defaults to:

```yaml
- name: bash.fact
  state: present
- name: dpkg.fact
  state: present
- name: gpg.fact
  state: present
```

When `state` is set to `absent` the file `name` in `/etc/ansible/facts.d` will be deleted.

The [bash.fact](files/bash.fact) script outputs the `$PATH` to Bash, this is `/bin/bash` on older servers and `/usr/bin/bash` on recent ones, in JSON format, for example:

```bash
bash /etc/ansible/facts.d/bash.fact | jq
```

```json
{
  "state": "present",
  "path": "/bin/bash"
}
```

The [dpkg.fact](files/dpkg.fact) script outputs the CPU architecture for use in `apt` configuration, this is generated using `dpkg --print-architecture`, also a list of installed packages generated using `dpkg --get-selections`, in JSON format, for example:

```bash
bash /etc/ansible/facts.d/dpkg.fact | jq
```

```json
{
  "state": "present",
  "arch": "amd64",
  "installed": [
    "adduser",
    "adwaita-icon-theme",
    "ansible",
    "zip",
    "zlib1g:amd64",
    "zlib1g-dev:amd64"
  ]
}
```

The [gpg.fact](files/gpg.fact) script outputs the version of `gpg`, this is generated using `gpg --version | head -n1 | gwak '{ print $3 }'`, in JSON format, for example:

```bash
bash /etc/ansible/facts.d/gpg.fact | jq
```

```yaml
{
  "state": "present",
  "version": "2.2.27"
}
```

### apt_local_facts_packages

A optional list of `.deb` packages that are required by the local fact scripts, packages listed in `apt_local_facts_packages` will be installed, if absent, when `apt_local_facts` is `true`.

The default value of `apt_local_facts_packages` is:

```yaml
- gawk
- bash
- coreutils
- dpkg
- gpg
- jo
```

### apt_pkgs

A required list of `.deb` packages that will be installed for each supported distro, for example:

```yaml
apt_pkgs:
  - distro: bookworm
    pkgs:
      - apt-listchanges
      - apt-show-versions
      - apt-utils
      - aptitude
      - aptitude-common
      - file
      - needrestart
```

### apt_config

The main configuration list for `.list` files and also `.sources` file, the writing of the `.sources` files use the [ansible.builtin.deb822_repository module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/deb822_repository_module.html), this is combined with the `apt_default_config` so you only need to list values that you want to differ, for example:

```yaml
apt_config:
  - distro: noble
    lists:
      - path: /etc/apt/sources.list
        state: absent
    sources:
      - name: ubuntu
        state: present
        types: deb
        uris:
          - http://archive.ubuntu.com/ubuntu/
        suites:
          - noble
          - noble-backports
          - noble-updates
        components:
          - main
          - multiverse
          - restricted
          - universe
        signed_by: /usr/share/keyrings/ubuntu-archive-keyring.gpg
      - name: ubuntu-security
        state: present
        types: deb
        uris:
          - http://security.ubuntu.com/ubuntu/
        suites:
          - noble-security
        components:
          - main
          - multiverse
          - restricted
          - universe
        signed_by: /usr/share/keyrings/ubuntu-archive-keyring.gpg
  - distro: jammy
    lists:
      - path: /etc/apt/sources.list
        state: present
        repos:
          - deb http://mirror.hetzner.de/ubuntu/packages/ jammy main restricted
          - deb http://mirror.hetzner.de/ubuntu/packages/ jammy-updates main restricted
          - deb http://mirror.hetzner.de/ubuntu/packages/ jammy universe
          - deb http://mirror.hetzner.de/ubuntu/packages/ jammy-updates universe
          - deb http://mirror.hetzner.de/ubuntu/packages/ jammy multiverse
          - deb http://mirror.hetzner.de/ubuntu/packages/ jammy-updates multiverse
          - deb http://security.ubuntu.com/ubuntu jammy-security main restricted
          - deb http://security.ubuntu.com/ubuntu jammy-security universe
          - deb http://security.ubuntu.com/ubuntu jammy-security multiverse
```

## Usage

In addition to the direct use of this role to configure the `/etc/apt/sources.list` file it can be included by other roles to set `local_facts` that are needed when configuring other apt repos, for example to get the `$PATH` for Bash, the CPU Architecture and the version of `gpg`:

```yaml
- name: Include apt role local fact tasks if variables are not defined
  ansible.builtin.include_role:
    name: apt
    tasks_from: local_facts.yml
  when: >-
    ( ansible_facts.ansible_local.dpkg.arch is not defined ) or
    ( ansible_facts.ansible_local.gpg.version is not defined ) or
    ( ansible_facts.ansible_local.bash.path is not defined )
```

The [Yarn Classic Ansible role](https://git.coop/webarch/yarn) provides an example of this usage in the [apt.yml](https://git.coop/webarch/yarn/-/blob/master/tasks/apt.yml) tasks, `ansible_local.bash.path` is used by the `gpg --dearmor` task, the `ansible_local.dpkg.arch` variable is used by the [yarn.sources](https://git.coop/webarch/yarn/-/blob/master/templates/yarn.sources.j2) template and the `ansible_local.gpg.version` variable is used to conditionally include the `--with-fingerprint --with-subkey-fingerprint` options for the `gpg --show-keys` command.

## Notes

This role removes `/etc/apt.conf` files as this role previously created them using the wrong format and in addition any required apt config should be written to files in `/etc/apt/apt.conf.d/`.

## Repository

The primary URL of this repo is [`https://git.coop/webarch/apt`](https://git.coop/webarch/apt) however it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-apt) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/apt).

If you use this role please use a tagged release, see [the release notes](https://git.coop/webarch/apt/-/releases).

## Copyright

Copyright 2019-2024 Chris Croome, &lt;[chris@webarchitects.co.uk](mailto:chris@webarchitects.co.uk)&gt;.

This role is released under [the same terms as Ansible itself](https://github.com/ansible/ansible/blob/devel/COPYING), the [GNU GPLv3](LICENSE).
