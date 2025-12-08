# The remove role.

Remove PVE guest machines from Proxmox VE cluster. Optionally (done by default), remove hosts from the `~/.ssh/known_hosts` file. Starts by force stopping the guest machines.

This can be a distructive role if called with the wrong variable settings. Make sure that the `remove_host_patterns` is correct. For this reason, the default value for `remove_host_patterns` is `!all`, which will find no `inventory_hostnames`.

# Variables

## Default Variables.

Because a Proxmox VE node or cluster will usually set the same values between the `cloud_init` and `lxc` roles
for many varialbes, when set, these variables will be used as the default value for the associated variable. Otherwise, the values under [All Variables](#all-variables) will be the default.

- `proxmox_client_api_host`
- `proxmox_client_api_user`
- `proxmox_client_api_secrets_dir`
- `proxmox_client_api_token_file`
- `proxmox_client_api_token_name`

## Required variables.

The default values for many variables will most likely not be desired or possibly even work.
For example, unless you named your Proxmox host `proxmox_master` and can resolve it via the DNS,
you will need to set `remove_pm_api_host`.

## All variables.

- `remove_from_ssh_known_hosts`
  - When `true`, Scans the host and adds it to `~/.ssh/known_hosts`. Skipped, when `false`.
  - Default: `true`

- `remove_host_patterns`
  - A `hosts` pattern that will match all guest machines to be removed.
  - Default: `'!all'`
  - The default pattern will return an empty list and avoid any removals.

- `remove_list`
  - A list of PVE guest names.
  - Default: Created from an `inventory_hostnames` query, using `remove_host_patterns`.
  - This is a role var, which changes the override precedence.
    See [understanding-variable-precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence) for more information.

- `remove_pm_api_host`
  - Proxmox host for API connection.
  - Default: `proxmox_master`.

- `remove_pm_api_user`
  - Proxmox user for API connection.
  - Default: `devops@pve`.

- `remove_pm_api_token_file`
  - File name of the API Token Secret.
  - Default: `remove_pm_api_user` + `-` + `remove_pm_api_token_name`, replacing the `@` with `-`. For example, `devops-pve-ansible` from the default values.

- `remove_pm_api_token_name`
  - Proxmox user specific API token for API connection.
  - Default: `ansible`.

- `remove_pm_api_token_secret`
  - The API token secret for API connection.
  - Default: Read from `remove_pm_api_token_file`.
  - Caution! Setting this directly may accidentally expose the value.
  - This is a role var, which changes the override precedence.
    See [understanding-variable-precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence) for more information.

- `remove_secrets_dir`
  - Local directory in which the API Token Secret file is located.
  - Default: `~/.pve_tokens`.

# Example Playbooks

```yaml
---
- name: Remove hosts matching a pattern in the ansible inventory from Proxmox.
  hosts: localhost
  gather_facts: false
  roles:
    - role: cloudcodger.proxmox_client.remove
      remove_host_patterns: myvm
      remove_pm_api_host: "192.168.1.21"
```
