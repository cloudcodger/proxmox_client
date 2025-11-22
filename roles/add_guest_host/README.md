# The add_guest_host role.

Loop over PVE guest information, get the IP address and add them to the Ansible inventory. Optionally (by default), perform a key scan and add the hosts to the `~/.ssh/known_hosts` file.

# Prerequisite

For this to work with a VM, it must have the QEMU Guest Agent installed and enabled. Otherwise, the role may cause an error and halt the execution of your playbook.

# Variables

## Default Variables.

Because a Proxmox VE node or cluster will usually set the same values between the `cloud_init` and `lxc` roles
for many varialbes, when set, these variables will be used as the default value for the associated variable.

- `proxmox_client_api_host`
- `proxmox_client_api_user`
- `proxmox_client_api_secrets_dir`
- `proxmox_client_api_token_file`
- `proxmox_client_api_token_name`

## Required variables.

The default values for the following variables will most likely not be desired or possibly even work.
For example, unless you named your Proxmox host `proxmox_master` and can resolve it via the DNS,
you will need to set `add_guest_host_pm_api_host`.

At a minimum, set the following to desired values.

- `add_guest_host_pm_api_host`
- `add_guest_host_list`

## All variables.

- `add_guest_host_list`
  - A list of PVE guest names. The default empty list will cause everything to be skipped.
  - Default: `[]`

- `add_guest_host_pm_api_host`
  - Proxmox host for API connection.
  - Default: `proxmox_master`.

- `add_guest_host_pm_api_user`
  - Proxmox user for API connection.
  - Default: `devops@pve`.

- `add_guest_host_pm_api_token_file`
  - File name of the API Token Secret.
  - Default: `add_guest_host_pm_api_user` + `-` + `add_guest_host_pm_api_token_name`, replacing the `@` with `-`. For example, `devops-pve-ansible` from the default values.

- `add_guest_host_pm_api_token_name`
  - Proxmox user specific API token for API connection.
  - Default: `ansible`.

- `add_guest_host_pm_api_token_secret`
  - The API token secret for API connection.
  - Default: Read from `add_guest_host_pm_api_token_file`.
  - Caution! Setting this directly may accidentally expose the value.
  - This is a role var, which changes the override precedence.
    See [understanding-variable-precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence) for more information.

- `add_guest_host_secrets_dir`
  - Local directory in which the API Token Secret file is located.
  - Default: `~/.pve_tokens`.

- `add_guest_host_to_ssh_known_hosts`
  - When `true`, Scans the host and adds it to `~/.ssh/known_hosts`. Skipped, when `false`.
  - Default: `true`

# Example Playbooks

```yaml
---
- name: Get the IP address of a host and add it to ansible inventory.
  hosts: localhost
  gather_facts: false
  roles:
    - role: cloudcodger.proxmox_client.add_guest_host
      add_guest_host_pm_api_host: "192.168.1.21"
      add_guest_host_list: [myvm]
```
