# Webarchitects apt Ansible role

[![pipeline status](https://git.coop/webarch/apt/badges/master/pipeline.svg)](https://git.coop/webarch/apt/-/commits/master)

This role contains an Ansible role for configuring the `apt` [/etc/apt/sources.list](https://wiki.debian.org/SourcesList) file on Debian and Ubuntu.

See also the [upgrade role](https://git.coop/webarch/upgrade), the [Bullseye role](https://git.coop/webarch/bullseye) for upgrading from Debian Buster and the [localhost repo](https://git.coop/webarch/localhost) which can be used with this role to configure the `sources.list` file locally.

## Usage

In addition to the direct use of this role to configure the `/etc/apt/sources.list` file it can be included by other roles to configure `local_facts` needed when configuring other apt repos, for example to get the `$PATH` for Bash, the CPU Architecture and the version of `gpg`:

```yaml
- name: Include apt role local fact tasks if variables are not defined
  ansible.builtin.include_role:
    name: apt
    tasks_from: local_facts.yml
  when: >-
    ( ansible_local.dpkg.arch is not defined ) or
    ( ansible_local.gpg.version is not defined ) or
    ( ansible_local.bash.path is not defined )
```

The [Yarn Classic Ansible role](https://git.coop/webarch/yarn) provides an example of this usage in the [apt.yml](https://git.coop/webarch/yarn/-/blob/master/tasks/apt.yml) tasks, `ansible_local.bash.path` is used by the `gpg --dearmor` task, the `ansible_local.dpkg.arch` variable is used bu the [yarn.sources](https://git.coop/webarch/yarn/-/blob/master/templates/yarn.sources.j2) template and the `ansible_local.gpg.version` variables is used to conditionally include the `--with-fingerprint --with-subkey-fingerprint` options with `gpg`.

## Role variables

See the [defaults/main.yml](defaults/main.yml) file for the default variables, the [vars/main.yml](vars/main.yml) file for the preset variables and the [meta/argument_specs.yml](meta/argument_specs.yml) file for the variable specification.

### apt_apticron

A required boolean, when `apt_apticron` is true packages listed in `apt_apticron_pkgs` will be installed, by default this causes a outgoing mailserver to be installed so this needs to be false for servers that have other applications listening on `SMTP` ports, `apt_apticron` defaults to `False`.

### apt_apticron_pkgs

A optional list of `.deb` packages to be installed when `apt_apticron` is `True`, by default `apt_apticron_pkgs` is a list containing one package name, `apticron`.

### apt_backports

A required boolean, when `apt_backports` is true the [Debian backports](https://backports.debian.org/) / [Ubuntu backports](https://help.ubuntu.com/community/UbuntuBackports) `apt` repos will be enabled, `apt_backports` defaults to `True`.

### apt_debian_components

A required list of strings for the Debian components that should be be enabled, `apt_debian_components` defaults to:

```yaml
- "contrib"
- "main"
- "non-free"
```

### apt_debian_sources_absent

A optional list of files in `/etc/apt/sources.list.d/` that should be deleted, `apt_debian_sources_absent` defaults to an empty list, `[]`.

### apt_distro_switch

A required boolean, set `apt_distro_switch` to `True` when switching distro, eg Bullseye to Bookworm.

### apt_local_facts

A optional boolean, when `apt_local_facts` is `True` the list of files in the `apt_local_facts_files` array will be processed, to install or remove [files/](files/) from the `/etc/ansible/facts.d` directory.

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

A optional list of `.deb` packages that are required by the local fact scripts, packages listed in `apt_local_facts_packages` will be installed, if absent, when `apt_local_facts` is `True`.

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

A optional list of `.deb` packages that will be installed, by default `apt_pkgs` is set to:

```yaml
  - aptitude
  - apt-listchanges
  - apt-show-versions
  - apt-utils
  - needrestart
```

### apt_src

A required boolean, set `apt_src` to `True` to enable apt source repos, it defaults to `False`.

### apt_ubuntu_components

A required list of Ubuntu components to be enabled, `apt_ubuntu_components` defaults to:

```yaml
- "main"
- "multiverse"
- "restricted"
- "universe"
```

### apt_ubuntu_country

A required string for country code for the Ubuntu mirrors to use, `apt_ubuntu_country` defaults to `uk`.

### apt_ubuntu_partner

A required boolean, when `apt_ubuntu_partner` is `True` Ubuntu partner apt repos will be enabled, it defaults to `False`.

## Repository

The primary URL of this repo is [`https://git.coop/webarch/apt`](https://git.coop/webarch/apt) however it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-apt) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/apt).

If you use this role please use a tagged release, see [the release notes](https://git.coop/webarch/apt/-/releases).

## Copyright

Copyright 2019-2023 Chris Croome, &lt;[chris@webarchitects.co.uk](mailto:chris@webarchitects.co.uk)&gt;.

This role is released under [the same terms as Ansible itself](https://github.com/ansible/ansible/blob/devel/COPYING), the [GNU GPLv3](LICENSE).
