`lxc`
=====

A role to create a set of identically configured Proxmox LXC Containers in [Proxmox VE](https://pve.proxmox.com/wiki/Main_Page).

Potential use cases to create;

- a single container (recommend using `lxc_cts` for this use case)
- a set of containers from the value of `lxc_cts`
- a set of containers from a "contructed" list using `lxc_construct_*` values
- a combination of the above settings (not recommended)

To avoid complexity and possible confusion, it is recommended to use either the `lxc_cts` _or_ the `lxc_construct_*` settings and not both.

Requirements
------------

- The `proxmoxer` Python module installed on the Ansible controller.
- Proxmox VE version 8 or above installed on the API host(s).
- Proxmox VE Datacenter Storage with `Container` content allowed.
- An LXC container "template" on each host where containers are being created.

Role Variables
--------------

### Required variables.

The default values for the following variables will very likely not be desired or possibly even work.
For example, unless you named your Proxmox host `proxmox_master` and can resolve it via the DNS,
you will need to set `lxc_pm_api_host`.

At a minimum, set the following to desired values.

- `lxc_pm_api_host`
- `lxc_cts` and/or the following construct items
- `lxc_construct_cidr_start`
- `lxc_construct_containers`
- `lxc_construct_hosts`
- `lxc_construct_vmid_start`
- `lxc_network_gw`
- `lxc_sshkeys`

### All variables.

- `lxc_pm_api_host` - Proxmox host for API connection (default: `proxmox_master`)
- `lxc_pm_api_user` - Proxmox user for API connection (default: `devops@pve`)
- `lxc_pm_api_token_name` - Proxmox user specific API token for API connection (default: `ansible`)
- `lxc_secrets_dir` - Local directory in which the API Token Secret file is located (default: `~/.pve_tokens`)
- `lxc_pm_api_token_file` - File name of the API Token Secret
  The default is to construct this value from the `lxc_pm_api_user` and `pm_api_token`, replacing the `@` with `-`.
  For example, `devops-pve-ansible` from the above default values.
- `lxc_pm_api_token_secret` - Read from the above file be default.
  Caution! Setting this directly may accidentally expose the value.
- `lxc_cts` - See the sub-section below for details (default: `[]`)
- `lxc_construct_cidr_start` - The first IPv4 address in CIDR notation.
- `lxc_construct_cidr6_start` - The first IPv6 address in CIDR notation.
- `lxc_construct_container_prefix` - The name of all containers with an index number appended (default: `new_container`)
- `lxc_construct_containers` - The number of containers to create (default: `0`)
- `lxc_construct_ct0_tags` - Special tags to add to the first and only the first container in the construct list (default: `[]`).
- `lxc_construct_hosts` - A list of Proxmox host names on which to create the containers (default `[]`).
  Containers get created by repeatedly looping over this list to set `pm_host` for each one.
  For example, if the total number of containers to be created is 12 and there are 3 hosts in this list, each host would end up with 4 containers created on it.
- `lxc_construct_vmid_start` - The VMID for the first VM and incremented for each one following it.
- `lxc_cores` - The number of CPU Cores for all containers (default: `4`).
- `lxc_cpus` - The number of CPUs (sockets) for all containers (default: `1`).
- `lxc_disk_size` - The size of the root disk for containers (default: `9` in GB).
- `lxc_memory` - The memory size for all containers (default: `1024` in MB).
- `lxc_storage` - The storage for all container disks (default: `local-thin`).
- `lxc_swap` - The size allocated to swap on all containers (default: `0`).
- `lxc_tags` - A list of Proxmox tags to give all containers (default: `[]`).
- `lxc_template` - Name of the LXC Container Template (default: `ubuntu-22.04-standard_22.04-1_amd64.tar.zst`).
- `lxc_template_storage` - The storage containing the above template (default: `local`).
- `lxc_unprivileged` - Indicate if the container should be unprivileged (default: `false`).
- `lxc_nameservers` - A list of name servers for the containers (default: `[]`).
- `lxc_network_bridge` - The bridge network used for all containers (default: `vmbr0`).
- `lxc_network_gw` - The default gateway used for all containers (default: `192.168.1.1`).
- `lxc_network_gw6` - The default IPv6 gateway (default: unset).
- `lxc_pm_host` - The default Proxmox host on which to create all the containers (defualt: `{{ lxc_pm_api_host }}`).
- `lxc_searchdomains` - A list of domains to search for DNS resolution (default: `[]`).
- `lxc_sshkeys` - A multi-line string containing a list of public SSH keys for the `root` user login on all containers (No default. See `pubkey` variable for the [community.general.proxmox](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html) module).
- `lxc_vlan_tag` - A VLAN tag number for all VM networks that also gets prepended to VMIDs (default: `''`).

Special Variable Notes
----------------------

### `lxc_template` and `lxc_template_storage`

The specified template file must be located in the `lxc_template_storage` storage which must be configured to contain "Containers" content. Using the default `local` storage as an example, this is a directory type storage with the path `/var/lib/vz` and contains the sub-directory `/var/lib/vz/template` under which a sub-directory named `cache` exists to hold the templates. The full default path is `/var/lib/vz/template/cache/` for the template file location.

The template must exist on every host where a container will be created. They can be uploaded to the storage or downloaded using the "Download from URL" option in the UI or using many other methods of your choosing.

### `lxc_cts`

This variable must be a list of hashs that contain specific key/value pairs defining the containers to create. Because many values must be unique among all containers and VMs within a Proxmox Datacenter, several of these are required and do not have default values.

Required keys:

- `name` - The name of the VM and becomes the hostname of the system.
- `ip` - Static IPv4 address (in CIDR notation) or `dhcp`.
- `vmid` - A _unique_ VMID number within the Datacenter.

Optional keys:

- `ip6` - Static IPv6 address (in CIDR notation).
- `mounts` - A list of optional mounts for this container in the format of a hash. See the first example.
- `pm_host` - Overrides the global value for `pm_host` for this container.
- `storage` - Overrides the global value for this container.
- `tags` - Combined with the global value for this container.

Dependencies
------------

This role depends on the [community.general.proxmox](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html) module.

Example Playbooks
-----------------

```yaml
- name: >-
    Create a single container using default values.
  hosts: localhost
  gather_facts: false
  roles:
    - role: cloudcodger.proxmox_client.lxc
      lxc_pm_api_host: "192.168.1.21"

      lxc_network_gw: "192.168.1.1"
      lxc_pm_host: pve1
      lxc_sshkeys: |
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIN+K0KwgKOeeyW519YavKQodVgwWcRUIucZkOfplsKMl devops-guy-mbp
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJk/hSmxsyznAhhsD5cO9wcTgOs+/xz09kZ5woSUUQAY devops-gal-mbp
      lxc_cts:
        - name: "testerct1"
          ip: "192.168.6.219/24"
          mounts:
            mp0: "local-lvm:400,mp=/mnt_point/"
          vmid: "219"
```

```yaml
- name: >-
    Create a set of 3 containers spread across 3 Proxmox hosts using the construct variables and configuring IPv6.
    Put the root disk on 'local-zfs' storage. Using the Ubuntu 20.04 template.
  hosts: localhost
  gather_facts: false
  roles:
    - role: cloudcodger.proxmox_client.lxc
      lxc_pm_api_host: "192.168.1.21"

      lxc_network_gw: "192.168.1.1"
      lxc_network_gw6: "2607:fa18:d7fe:ff00::1"
      lxc_pm_host: pve1
      lxc_sshkeys: |
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIN+K0KwgKOeeyW519YavKQodVgwWcRUIucZkOfplsKMl devops-guy-mbp
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJk/hSmxsyznAhhsD5cO9wcTgOs+/xz09kZ5woSUUQAY devops-gal-mbp
      lxc_storage: 'local-zfs'
      lxc_template: 'ubuntu-20.04-standard_20.04-1_amd64.tar.gz'

      lxc_construct_cidr_start: "192.168.6.141/24"
      lxc_construct_cidr6_start: "2607:fa18:d7fe:f000::8d/64"
      lxc_construct_container_prefix: "ct"
      lxc_construct_containers: 3
      lxc_construct_ct0_tags: ['primary']
      lxc_construct_hosts: [ 'gadget1', 'gadget2', 'gadget3' ]
      lxc_construct_vmid_start: 90141
```
