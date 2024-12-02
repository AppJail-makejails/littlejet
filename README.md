# LittleJet

LittleJet is an open source, easy-to-use orchestrator for managing, deploying, scaling and interconnecting FreeBSD jails anywhere in the world.

<img src="https://raw.githubusercontent.com/DtxdF/LittleJet/refs/heads/main/assets/img/Isotype.png" width="60%" height="auto" />

## How to use this Makejail

### Standalone

```sh
appjail makejail \
    -j littlejet \
    -f gh+AppJail-makejails/littlejet \
    -o virtualnet=":<random> default" \
    -o nat \
        -- \
        --littlejet_ssh_key "ssh-ed25519 ..."
```

If you use SSH, you should have in your `ssh_config(5)` file:

```
Host control-jet
        HostName <ip-or-hostname>
        User littlejet
        LogLevel ERROR
        RequestTTY yes
```

### Deploy using appjail-director

**appjail-director.yml**:

```yaml
options:
  - virtualnet: ':<random> default'
  - nat:

services:
  c2c:
    name: littlejet
    makejail: gh+AppJail-makejails/littlejet
    arguments:
      - littlejet_ssh_key: !ENV '${SSH_KEY}'
      - littlejet_tag: !ENV '${TAG}'
    options:
      - expose: !ENV '${PORT}:22'
    volumes:
      - conf: littlejet-conf
      - data: littlejet-data
      - ssh_srv: littlejet-ssh
      - ssh_cli: /user/.ssh

default_volume_type: '<volumefs>'

volumes:
  conf:
    device: .volumes/conf
  data:
    device: .volumes/data
  ssh_srv:
    device: .volumes/ssh/srv
  ssh_cli:
    device: .volumes/ssh/cli
    type: nullfs
    owner: 1001
    group: 1001
    mode: 0700
```

**.env**:

```
DIRECTOR_PROJECT=orchestrator
TAG=14.1
PORT=2022
SSH_KEY=ssh-ed25519 ...
```

**Create an SSH key**:

```sh
mkdir -p .volumes/ssh/cli
ssh-keygen -t ed25519 -N "" -f .volumes/ssh/cli/id_ed25519 -C "LittleJet-c2c"
chown -Rf 1001:1001 .volumes/ssh/cli
```

### Arguments

* `littlejet_ssh_conf` (default: `files/sshd_config`): `sshd_config(5)` configuration file.
* `littlejet_ssh_key`: (optional): If this argument is set, the SSH server is started and configured to use LittleJet remotely. This argument is the authorized key.
* `littlejet_tag` (default: `13.4`): See [#tags](#tags).

### Volumes

| Name           | Owner | Group | Perm | Type | Mountpoint              |
| -------------- | ----- | ----- | ---- | ---- | ----------------------- |
| littlejet-conf | 1001  | 1001  |  -   |  -   | /user/.config/littlejet |
| littlejet-data | 1001  | 1001  |  -   |  -   | /user/.littlejet        |
| littlejet-ssh  | 0     | 0     |  755 |  -   | /etc/ssh                |

## Tags

| Tag    | Arch    | Version        | Type   |
| ------ | ------- | -------------- | ------ |
| `13.4` | `amd64` | `13.4-RELEASE` | `thin` |
| `14.1` | `amd64` | `14.1-RELEASE` | `thin` |