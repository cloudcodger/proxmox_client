`lxc`
=====

Create a set of identically configured Proxmox LXC Containers (CT) in [Proxmox VE](https://pve.proxmox.com/wiki/Main_Page).

By default, if no variables are set, calling this role will not create any CTs.

There are two primary use cases for this role.

1. Create a set of [identical CTs](#sequencial-cts) using `lxc_construct_*` variables.
    - Any group of CTs with a name prefix follow by a sequence number.
2. Create [individually defined](#individually-defined) VMs using the `lxc_cts` variable.
    - A single CT.
    - A group of related CTs with different names and/or requirements.

A combination of using both `lxc_construct_*` and `lxc_cts` is allowed, but discouraged and not recommended. Using both at the same time makes it difficult for others to follow what is being created. To avoid complexity and confusion, it is recommended to use either the `lxc_cts` _or_ the `lxc_construct_*` settings and not both.

Potential use cases to create;

- a single CT (recommend using `lxc_cts` for this use case)
- a set of CTs from the value of `lxc_cts`
- a set of CTs from a "contructed" list using `lxc_construct_*` values
- a combination of the above settings (not recommended)

To avoid complexity and possible confusion, it is recommended to use either the `lxc_cts` _or_ the `lxc_construct_*` settings and not both.

### Sequencial CTs

Use the `lxc_construct_containers` and `lxc_construct_*` variables to create a set of identical CTs. For example, when creating three CTs named, `ns1`, `ns2` and `ns3` that all have the same CPU sockets/cores, memory, disk size, and tags.

### Individually Defined

This option provides the creation of multiple CTs that differ in the values set under each item in [`lxc_cts`](#lxc_cts).

Requirements
------------

- The `proxmoxer` Python module installed on the Ansible controller.
- Proxmox VE version 8 or above installed on the API host(s).
- Proxmox VE Datacenter Storage with `Container` content allowed.
- An LXC container "template" on each host where CTs are being created.

Role Variables
--------------

### Required variables.

The default values for the following variables will most likely not be desired or possibly even work.
For example, unless you named your Proxmox host `proxmox_master` and can resolve it via the DNS,
you will need to set `lxc_pm_api_host`.

At a minimum, set the following to desired values.

- `lxc_pm_api_host`
- `lxc_cts` and/or the following construct items
- `lxc_construct_cidr_start`
- `lxc_construct_containers`
- `lxc_construct_hosts`
- `lxc_construct_vmid_start`

### All variables.

- `lxc_ansible_host`
  - `true`: Create a `host_vars` file for the CT and set `ansible_host` in it.
  - `false`: Don't.
  - Default: `false`.
  - When creating CTs that use DHCP, it can be difficult to get the IP address assigned. This allows the role to set `ansible_host` to the IP address of the guest.

- `lxc_ansible_host_vars_paths`
  - List of directory paths in which to look for `host_vars`.
  - Default:
    - "{{ ansible_inventory_sources }}"
    - "{{ ansible_inventory_sources[0] | dirname }}"
  - Make sure to set this when setting `lxc_ansible_host` to `true` and the `host_vars` is located at a non-standard path.

- `lxc_ansible_host_vars_dir`
  - The `host_vars` directory.
  - Default: The first `host_vars` directory found in `lxc_ansible_host_vars_paths`.

- `lxc_ansible_inventory_refresh`
  - `true`: Perform a `refresh_inventory` after the last task in this role.
  - `false`: Don't.
  - Default: `false`.
  - This is only done when a CT is created or started.
  - When calling this role in a playbook followed by another play to configure the CTs, a newly created host will not be found in the inventory unless this is done.

- `lxc_bypass_wait_for_connection`
  - `true`: Do not wait for the new systems to be accessible.
  - `false`: Do wait.
  - Default: `false`.

- `lxc_construct_cidr_start`
  - The first IPv4 address in CIDR notation. For example, `lxc_construct_cidr_start: "192.168.1.21"`.
  - Default: None.
  - Default: `omit`, allowing all CTs to use DHCP.
  - Required when using `lcx_construct_*`.

- `lxc_construct_cidr6_start`
  - The first IPv6 address in CIDR notation.
  - Default: None.

- `lxc_construct_container_prefix`
  - The name of all CTs with an index number appended.
  - Default: `new_container`.

- `lxc_construct_containers`
  - The number of CTs to create.
  - Default: `0`.

- `lxc_construct_ct0_tags`
  - Special tags to add to the first and only the first CT in the construct list.
  - Default: `[]`.

- `lxc_construct_hosts`
  - A list of Proxmox host names on which to create the CTs.
  - Default `[]`.
  - CTs get created by repeatedly looping over this list to set `pm_host` for each one. For example, if the total number of CTs to be created is 12 and there are 3 hosts in this list, each host would end up with 4 CTs.
  - Required when using `lcx_construct_*`.

- `lxc_construct_vmid_start`
  - The VMID for the first CT and incremented for each one following it.
  - Default: None (is omitted and PVE will auto assign one).

- `lxc_cores`
  - The number of CPU Cores for all CTs.
  - Default: `4`.

- `lxc_cpus`
  - The limit for the number of CPUs for all CTs.
  - Default: None (is omitted).

- `lxc_cpuunits`
  - The number of CPU Units for all CTs.
  - Default: None (is omitted).

- `lxc_cts`
  - See the [lxc_cts](#lxc_cts) section below for details.
  - Default: `[]`.

- `lxc_disk_size`
  - The size of the root disk for CTs.
  - Default: `9` in GB.

- `lxc_features`
  - A list of LXC features.
  - Default: `[]`.
  - See "features" under [Options](https://pve.proxmox.com/wiki/Linux_Container#pct_options).
  - A list of features, current possible features are; `force_rw_sys`, `fuse`, `keyctl`, `mknod`, `mount` and `nesting`.

- `lxc_find_pm_host`
  - `true`: Set `lxc_pm_host` to the node with the most free memory.
  - `false`: Don't change `lxc_pm_host` using this check.
  - This calculation is performed right before creating each VM.

- `lxc_memory`
  - The memory size for all CTs.
  - Default: `1024` in MB.

- `lxc_nameservers`
  - A list of name servers for the CTs.
  - Default: `[]`.

- `lxc_network_bridge`
  - The bridge network used for all CTs.
  - Default: `vmbr0`.
  - All Proxmox clusters contain a `vmbr0` network bridge by default.

- `lxc_network_gw`
  - The default gateway used for all CTs.
  - Default: unset and omitted.

- `lxc_network_gw6`
  - The default IPv6 gateway.
  - Default: unset and omitted.

- `lxc_pm_api_host`
  - Proxmox host for API connection.
  - Default: `proxmox_master`.

- `lxc_pm_api_user`
  - Proxmox user for API connection.
  - Default: `devops@pve`.

- `lxc_pm_api_token_name`
  - Proxmox user specific API token for API connection.
  - Default: `ansible`.

- `lxc_pm_api_token_file`
  - File name of the API Token Secret.
  - Default: `lxc_pm_api_user` + `-` + `lxc_pm_api_token_name`, replacing the `@` with `-`. For example, `devops-pve-ansible` from the above default values.

- `lxc_pm_api_token_secret`
  - The API token secret for API connection.
  - Default: Read it from `lxc_pm_api_token_file`.
  - Caution! Setting this directly may accidentally expose the value.
  - This is a role var, which changes the override precedence.
    See [understanding-variable-precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence) for more information.

- `lxc_pm_host`
  - The default Proxmox host on which to create all the CTs.
  - Defualt: `{{ lxc_pm_api_host }}`.

- `lxc_searchdomains`
  - A list of domains to search for DNS resolution.
  - Default: `[]`.

- `lxc_secrets_dir`
  - Local directory in which the API Token Secret file is located.
  - Default: `~/.pve_tokens`.

- `lxc_sshkeys`
  - A multi-line string containing a list of public SSH keys for the `root` user login on all CTs.
  - Default: unset and omitted.
  - Required. Because leaving this empty will result in CTs to which you cannot login.
  - See `pubkey` variable for the [community.general.proxmox](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html) module.

- `lxc_storage`
  - The Datacenter storage for all CT disks.
  - Default: `local-thin`.
  - This storage _must_ exist on each PVE node on which VMs are created and be configured to support the "Container" content type.
  - For PVE clusters, consider using shared storage for better migration support.
  - Common values include: `local`, `local-thin`, `local-zfs` and `ceph0`.

- `lxc_swap`
  - The size allocated to swap on all CTs.
  - Default: `0`.

- `lxc_tags`
  - A list of Proxmox tags to give all CTs.
  - Default: `[]`.

- `lxc_template`
  - Name of the CT Template.
  - Default: `ubuntu-22.04-standard_22.04-1_amd64.tar.zst`.

- `lxc_template_storage`
  - The storage containing the above template.
  - Default: `local`.

- `lxc_unprivileged`
  - `true`: CT `unprivileged` set to `true`.
  - `false`: CT `unprivileged` set to `false`.
  - Default: `true`.
  - This is the same default as the module.

- `lxc_update_known_hosts`
  - `true`: Look up the SSH host keys and update the local `.ssh/known_hosts` file.
  - `false`: Skip these tasks.
  - Default: `true`.

- `lxc_vlan_tag`
  - A VLAN tag number for all CT networks.
  - Default: Not set
  - This also gets prepended to any provided VMIDs.

Special Variable Notes
----------------------

### `lxc_template` and `lxc_template_storage`

The specified template file must be located in the `lxc_template_storage` storage which must be configured to contain "Containers" content. Using the default `local` storage as an example, this is a directory type storage with the path `/var/lib/vz` and contains the sub-directory `/var/lib/vz/template` under which a sub-directory named `cache` exists to hold the templates. The full default path is `/var/lib/vz/template/cache/` for the template file location.

The template must exist on every host where a CT will be created. They can be uploaded to the storage or downloaded using the "Download from URL" option in the UI or using many other methods of your choosing.

### `lxc_cts`

This variable must be a list of hashs that contain specific key/value pairs defining the CTs to create.

Required keys:

- `name`
  - The name of the VM and becomes the hostname of the system.

Optional keys:

- `description`
  - Becomes the Notes for a CT in the PVE UI Summary page.

- `ip`
  - Static IPv4 address (in CIDR notation).
  - Default: `dhcp`.

- `ip6`
  - Static IPv6 address (in CIDR notation).

- `mounts`
  - A list of optional mounts for this CT in the format of a hash. See the first example.

- `pm_host`
  - Overrides `lxc_pm_host` for this CT.

- `storage`
  - Overrides `lxc_storage` for this CT.

- `tags`
  - Combined with `lxc_tags` for this CT.

- `vmid`
  - A _unique_ VMID number within the Datacenter.
  - Default: Not set, and let PVE select the next available VMID.
  - When set, this will also have the `lxc_vlan_tag` prepended to it.

Dependencies
------------

This role depends on the [community.general.proxmox](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html) module.

Example Playbooks
-----------------

```yaml
- name: >-
    Create a single CT using default values.
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
    Create a set of 3 CTs spread across 3 Proxmox hosts using the construct variables and configuring IPv6.
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
