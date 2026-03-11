# SSH
Secure Shell (SSH/SSHD) global configuration.

## [Requirements][i]
Requires [r_pufky.deb][g] galaxy-ng collection. See
[reference documentation][h] for specific SSH configuration.

SSH Hardening Guidance:

* OpenSSH [security advisories][k].
* [NSA Hardening Guide][l].
* [SSH Hardening for Audit][m].

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [ssh_config][q] - SSH configurable options.

* [sshd_config][r] - SSHD configurable options.

## Usage

### Example Playbooks
Read through defaults and [Debian SSH Changes](#debian-ssh-changes) before
proceeding. All **ssh_config** and **sshd_config** options are supported.

This will remove DSA host keys, ensure RSA host keys are at least 4096bits,
configure SSHD, and configure SSH for two hosts:

group_vars/all/vars/main.yml
``` yaml
ssh_security_remove_host_keys:
  - 'ssh_host_dsa_key'
ssh_security_regenerate_host_rsa_keys: 4096
```

host_vars/node.example.com/vars/sshd_config.yml
``` yaml
ssh_cfg_server_allow_agent_forwarding: false
ssh_cfg_server_allow_groups: ['_ssh']
ssh_cfg_server_channel_timeout:
  'session': '5m'
  'tun-connection': '1h'
  '*': '30m'
ssh_cfg_server_client_alive_count_max: 5
ssh_cfg_server_client_alive_interval: 10
ssh_cfg_server_compression: true
ssh_cfg_server_deny_groups: ['deny-groups']
ssh_cfg_server_deny_users: ['denied-user']
ssh_cfg_server_disable_forwarding: true
ssh_cfg_server_hostbased_authentication: true
ssh_cfg_server_log_level: 'DEBUG'
ssh_cfg_server_password_authentication: false
ssh_cfg_server_permit_empty_passwords: false
ssh_cfg_server_permit_root_login: false
ssh_cfg_server_port:
  - 22
  - 2222
ssh_cfg_server_x11_forwarding: false
```

host_vars/node.example.com/vars/ssh_config.yml
``` yaml
ssh_cfg_client_add_keys_to_agent: 'confirm'
ssh_cfg_client_address_family: 'inet'
ssh_cfg_client_batch_mode: false
ssh_cfg_client_forward_agent: false
ssh_cfg_client_forward_x11: false
ssh_cfg_client_kbd_interactive_authentication: false
ssh_cfg_client_port: 22
ssh_cfg_client_tcp_keep_alive: true
```

``` yaml
- name: 'Renegerate host keys for all hosts, and set ssh/sshd config for node.'
  hosts: 'all'
  become: true
  roles:
     - 'r_pufky.deb.ssh'
```

### Local configuration
The ssh role may also deploy local config files in the respective
**/etc/ssh/{ssh,sshd}_config.d** directory (or locations defined by
configuration). Local files are free-form, included before global
configuration, and may also remove existing local files if desired.

```yaml
- name: 'Set local SSH and SSHD configuration.'
  ansible.builtin.import_role:
     - 'r_pufky.deb.ssh'
  vars:
    ssh_cfg_server_include_files:
      - config: 'test_config'
        state: 'present'
        options: |
          AllowAgentForwarding yes
      - config: 'remove_this_local_config'
        state: 'absent'
    ssh_cfg_client_include_files:
      - config: 'test_config'
        state: 'present'
        options: |
          Host *
            EscapeChar ^b
      - config: 'remove_this_local_config'
        state: 'absent'
```

## Debian Specific SSH Changes

### [ssh group now _ssh][n]
**ssh** group migrated to **_ssh**. **ssh** group must be manually managed if
used with existing users and groups, or migrate users to **_ssh**.

### SSH pubkey authentication with locked accounts
SSH now distinguishes between ! and * password locking:

* `*`: Lock password - allow SSH pubkey auth.
* `!`: Lock password - deny SSH pubkey auth.

Any other means to lock the password will result in SSH pubkey failures. Do
NOT set ssh_cfg_server_use_pam=true as this leads to
[security vulnerabilities][o] with potential [mitigataions][p] if absolutely
necessary.

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

### [Releases][b]

  Release | Debian | Ansible | Notes
 ---------|--------|---------|-------
  3.x.x   | 13     | 2.20    | Ansible 2.20, semantic versioning.
  2.x.x   | 13     | 2.18    | Migrate to Debian Trixie.
  1.x.x   | 12     | 2.18    | Use standardized libraries.
  0.x.x   | 12     | 2.11    | Migration from private repository.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]


[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_ssh/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_deb
[h]: http://r-pufky.github.io/docs/service/ssh
[i]: https://github.com/r-pufky/ansible_ssh/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_ssh/tree/main/defaults/main/main.yml
[k]: https://www.openssh.com/security.html
[l]: https://media.defense.gov/2022/Jun/15/2003018261/-1/-1/0/CTR_NSA_NETWORK_INFRASTRUCTURE_SECURITY_GUIDE_20220615.PDF
[m]: https://www.sshaudit.com/hardening_guides.html#os_12
[n]: https://salsa.debian.org/ssh-team/openssh/-/commit/18da782ebe789d0cf107a550e474ba6352e68911
[o]: https://arlimus.github.io/articles/usepam
[p]: https://unix.stackexchange.com/questions/193066/how-to-unlock-account-for-public-key-ssh-authorization-but-not-for-password-aut
[q]: https://github.com/r-pufky/ansible_ssh/tree/main/defaults/main/ssh_config.yml
[r]: https://github.com/r-pufky/ansible_ssh/tree/main/defaults/main/sshd_config.yml
