# SSH
Secure Shell (SSH/SSHD) global configuration.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_ssh/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_ssh/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_ssh/tree/main/defaults/main/)

## Dependencies
N/A

## Example Playbook
Read through defaults before using; all `ssh_config` and `sshd_config` options
are suppported. A machine may simultaneously be configured with both.

This will remove DSA host keys, ensure RSA host keys are at least 4096bits,
configure SSHD, and configure SSH for two hosts.

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
- name: 'manage ssh'
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

## Unit Testing
Test framework requires molecule and rootless podman setup.

Run all unit tests:
``` bash
molecule test --all
```

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_fonts/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
