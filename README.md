# volker-raschek.unix-users

![Ansible Role](https://img.shields.io/ansible/role/d/volker-raschek/unix-users)

The ansible role `volker-raschek.unix-users` create and manage users on Linux based distributions. For example for Arch
Linux, Fedora and Ubuntu. Furthermore, the role can also be used to create groups, `~/.forward`, `~/.netrc` and to
manage the `~/.ssh` directory.

## Examples

### User and group

The following example create the user `toor` and group `toor`. Booth with a specific id.

```yaml
unix_groups:
  toor:
    gid: "1001"
    state: present

unix_users:
  toor:
    state: present
    name: Toor
    uid: "1000"
    home: /home/toor
    shell: /bin/bash
    password: toor
    group: toor
```

### Btrfs home dir

Optionally, the home directory of a user can also be created as dedicated btrfs subvolume. This make it possible to
create snapshots of the home directory, for example via `btrbk`.

```yaml
unix_users:
  toor:
    state: present
    name: Toor
    uid: "1000"
    home: /home/toor
    btrfs: true
    shell: /bin/bash
    password: toor
    group: toor
```

### .netrc

The ansible role supports the creation and management of the `.netrc` file in a user's home directory. The `.netrc` file
for the user `toor` is created below. This contains entries for GitHub.

```yaml
unix_users:
  toor:
    state: present
    name: Toor
    uid: "1000"
    home: /home/toor
    netrc:
    - machine: github.com
      login: octocat
      password: pat_12345
    - machine: api.github.com
      login: octocat
      password: pat_12345
    shell: /bin/bash
    password: toor
    group: toor
```

### .ssh

The SSH client directory `~/.ssh` can also be managed via the Ansible role. This supports the creation and management of
`~/.ssh/config`, `~/.ssh/authorized_keys` as well as the maintenance of private and public SSH keys.

The following example create two entries in `~/.ssh/authorized_keys`. One normal SSH access for `claire`. If `bob`
establish a SSH connection the command `/usr/local/bin/upload-file.sh` will be executed and exited. Furthermore,
environment variables can be espcilitly defined, to consume it during execution of the command.

> [!IMPORTANT]
> To allow consuming environment variables must be set `PermitUserEnvironment yes` in `/etc/ssh/sshd_config`.

The private key `toor@toor-pc.ed25519.key` must be stored in `ssh/private_keys`. The public key will be automatically
extracted from the private key.

The public keys `claire@claire-pc.pub` as well as `bob@bob-pc.pub` must be stored in `ssh/authorized_keys`.

```yaml
unix_users:
  toor:
    state: present
    name: Toor
    uid: "1000"
    home: /home/toor
    ssh:
      config:
      - Host: "*"
        StrictHostKeyChecking: "no"
        UserKnownHostFile: /dev/null
      authorized_keys:
      - filename: claire@claire-pc.pub
      - command: /usr/local/bin/upload-file.sh
        environments:
        - key: SSH_KEY_NAME
          value: bob@bob-pc
        filename: bob@bob-pc.pub
      private_keys:
      - toor@toor-pc.ed25519.key
    shell: /bin/bash
    password: toor
    group: toor
```

### .forward

If on the system is postfix installed, postfix will respect the `~/.forward`
[file](https://www.postfix.org/local.8.html). This allows to forward local emails to external email addresses. The
following example create the `~/.forward` file for `toor` to forward emails to `toor@company.example.local`.

```yaml
unix_users:
  toor:
    state: present
    name: Toor
    uid: "1000"
    home: /home/toor
    email: toor@company.example.local
    shell: /bin/bash
    password: toor
    group: toor
```

### shell_rc files

The role also supports the creation of bashrc drop-in files. These are created in `~/.bashrc.d` and included by
`~/.bashrc` via `source`.

Program-related configurations can be made via a drop-in file. For example, the configuration of the bash history via
the environment variables `HISTCONTROL` or `HISTFILE`. In addition to environment variables, aliases and complete
functions can also be defined.

```yaml
unix_users:
  toor:
    state: present
    name: Toor
    uid: "1000"
    home: /home/toor
    email: toor@company.example.local
    shell: /bin/bash
    shell_rc_files:
    - file: "/home/toor/.bashrc.d/10-docker.bashrc" # absolute or relative path to home dir
      aliases:
      - key: "dcd"
        value: "docker-compose down"
      envs:
      - export: true
        key: "PATH"
        value: "/home/toor/workspace/docker-compose/bin:${PATH}" # Add local compiled docker-compose into $PATH
      functions:
      - name: "foo"
        value: |
          if ! which docker 1> /dev/null; then
            echo "ERROR: docker not found" 1>&2
            exit 1
          fi
    password: toor
    group: toor
```

## Further ansible roles

This ansible role is used in combination with other ansible roles of `volker-raschek`. You can search for the other
ansible roles via the following command.

```bash
$ ansible-galaxy role search --author "volker-raschek"

Found roles matching your search:

 Name                      Description
 ----                      -----------
 volker-raschek.bind9      Role to install and configure bind9 on different distributions
 volker-raschek.dhcpd      Role to install and configure dhcpd on different distributions
 volker-raschek.renovate   Role to configure renovate as container image
 ...
```
