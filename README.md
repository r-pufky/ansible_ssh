# SSH
Secure Shell (SSH/SSHD) global configuration.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_ssh/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_ssh/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_ssh/tree/main/defaults/main/)

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
Read through defaults and [Debian SSH Changes](#debian-ssh-changes) before
proceeding.

All `ssh_config` and `sshd_config` options are supported. A machine may
simultaneously be configured with both.

This will remove DSA host keys, ensure RSA host keys are at least 4096bits,
configure SSHD, and configure SSH for two hosts:

group_vars/all/vars/main.yml
``` yaml
ssh_security_remove_host_keys:
  - 'ssh_host_dsa_key'
ssh_security_regenerate_host_rsa_keys: 4096
```

host_vars/sshd.example.com/vars/sshd_config.yml
``` yaml
ssh_server_allow_agent_forwarding: false
ssh_server_allow_groups: ['ssh']
ssh_server_channel_timeout:
  'session': '5m'
  'tun-connection': '1h'
  '*': '30m'
ssh_server_client_alive_count_max: 5
ssh_server_client_alive_interval: 10
ssh_server_compression: true
ssh_server_deny_groups: ['deny-groups']
ssh_server_deny_users: ['denied-user']
ssh_server_disable_forwarding: true
ssh_server_hostbased_authentication: true
ssh_server_log_level: 'DEBUG'
ssh_server_password_authentication: false
ssh_server_permit_empty_passwords: false
ssh_server_permit_root_login: false
ssh_server_port:
  - 22
  - 2222
ssh_server_x11_forwarding: false
```

host_vars/ssh.example.com/vars/ssh_config.yml
``` yaml
ssh_client_add_keys_to_agent: 'confirm'
ssh_client_address_family: 'inet'
ssh_client_batch_mode: false
ssh_client_forward_agent: false
ssh_client_forward_x11: false
ssh_client_kbd_interactive_authentication: false
ssh_client_port: 22
ssh_client_tcp_keep_alive: true
```

Apply the role
``` yaml
- name: 'Manage SSH'
  hosts: '*'
  become: true
  roles:
     - 'r_pufky.srv.ssh'
```

The ssh role may also deploy local config files in the respective
`/etc/ssh/ssh[d]_config.d` directory (or locations defined by configuration).
Local files are free-form, included before global configuration, and may also
remove existing local files if desired.

```yaml
ssh_server_include_files:
  - config: 'test_config'
    state: 'present'
    options: |
      AllowAgentForwarding yes
  - config: 'remove_this_local_config'
    state: 'absent'
ssh_client_include_files:
  - config: 'test_config'
    state: 'present'
    options: |
      Host *
        EscapeChar ^b
  - config: 'remove_this_local_config'
    state: 'absent'
```

## Debian SSH Changes
Specific Debian SSH changes between major releases.

### ssh group now _ssh
`ssh` group migrated to `_ssh`. `ssh` group must be manually managed if used
with existing users and groups, or migrate users to `_ssh`.

https://salsa.debian.org/ssh-team/openssh/-/commit/18da782ebe789d0cf107a550e474ba6352e68911

#### SSH pubkey authentication with locked accounts
SSH now distinguishes between `!` and `*` password locking:

* `*`: lock password, allow SSH pubkey auth.
* `!`: lock password, deny SSH pubkey auth.

Any other means to lock the password will result in SSH pubkey failures. Do not
enable `ssh_server_use_pam=true` as this leads to security vulnerabilities.

[Reference](https://github.com/r-pufky/ansible_ssh/blob/main/defaults/main/sshd_config.yml)

####
## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```
chacha20-poly1305
### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_ssh/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
