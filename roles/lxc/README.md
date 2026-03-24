# `lxc` role

Create a set of identically configured Proxmox LXC Containers (CT) in [Proxmox VE](https://pve.proxmox.com/wiki/Main_Page).

By default, if no variables are set, calling this role will not create any CTs.

There are two primary use cases for this role.

1. Create a set of [identical CTs](#sequencial-cts) using `lxc_construct_*` variables.
    - Any group of CTs with a name prefix follow by a sequence number.
2. Create [individually defined](#individually-defined) VMs using the `lxc_cts` variable.
    - A single CT.
    - A group of related CTs with different names and/or requirements.

A combination of using both `lxc_construct_*` and `lxc_cts` is allowed, but discouraged and not recommended. Using both at the same time makes it difficult for others to follow what is being created. To avoid complexity and confusion, it is recommended to use either the `lxc_cts` _or_ the `lxc_construct_*` settings and not both.

# Sequencial CTs

Use the `lxc_construct_containers` and `lxc_construct_*` variables to create a set of identical CTs. For example, when creating three CTs named, `ns1`, `ns2` and `ns3` that all have the same CPU sockets/cores, memory, disk size, and tags.

# Individually Defined CTs

This option provides the creation of multiple CTs that differ in the values set under each item in [`lxc_cts`](#lxc_cts).

# Requirements

- The `proxmoxer` Python module installed on the Ansible controller.
- Proxmox VE version 8 or above installed on the API host(s).
- Proxmox VE Datacenter Storage with `Container` content allowed.
- An LXC container "template" on each host where CTs are being created.

# Role Variables

## Special Default Variables.

Because a Proxmox VE node or cluster will usually set the same values between the `cloud_init` and `lxc` roles
for many varialbes, when set, these variables will be used as the default value for the associated variable.

- `proxmox_client_api_host`
- `proxmox_client_api_user`
- `proxmox_client_api_secrets_dir`
- `proxmox_client_api_token_file`
- `proxmox_client_api_token_name`
- `proxmox_client_find_pm_host`
- `proxmox_client_nameservers`
- `proxmox_client_searchdomains`
- `proxmox_client_sshkeys`
- `proxmox_client_storage`

## Required variables.

The default values for the following variables will most likely not be desired or possibly even work.
For example, unless you named your Proxmox host `proxmox_master` and can resolve it via the DNS,
you will need to set `lxc_pm_api_host`.

At a minimum, set the following to desired values.

- `lxc_pm_api_host`
- `lxc_cts` and/or the following construct items
- `lxc_construct_cidr_start`
- `lxc_construct_containers`
- `lxc_construct_hosts`

## All variables.

- `lxc_ansible_inventory_refresh`
  - `true`: Perform a `refresh_inventory` after the last task in this role.
  - `false`: Don't.
  - Default: `false`.
  - This is only done when a CT is created or started.
  - When calling this role in a playbook followed by another play to configure the CTs, a newly created host will not be found unless either an inventory refresh is done or it's added using something like `ansible.builtin.add_host`.

- `lxc_construct_cidr_start`
  - The first IPv4 address in CIDR notation. For example, `lxc_construct_cidr_start: "192.168.1.21"`.
  - Default: None.
  - Required when using `lcx_construct_*`.

- `lxc_construct_cidr_net1_start`
  - The first IPv4 address in CIDR notation for `net1`. For example, `lxc_construct_cidr_net1_start: "192.168.1.21"`.
  - Default: None.

- `lxc_construct_cidr_net2_start`
  - The first IPv4 address in CIDR notation for `net2`. For example, `lxc_construct_cidr_net2_start: "192.168.1.21"`.
  - Default: None.

- `lxc_construct_cidr_net3_start`
  - The first IPv4 address in CIDR notation for `net3`. For example, `lxc_construct_cidr_net3_start: "192.168.1.21"`.
  - Default: None.

- `lxc_construct_cidr6_start`
  - The first IPv6 address in CIDR notation.
  - Default: None.

- `lxc_construct_cidr6_net1_start`
  - The first IPv6 address in CIDR notation for `net1`.
  - Default: None.

- `lxc_construct_cidr6_net2_start`
  - The first IPv6 address in CIDR notation for `net2`.
  - Default: None.

- `lxc_construct_cidr6_net3_start`
  - The first IPv6 address in CIDR notation for `net3`.
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
  - Required when using `lcx_construct_*` and `lxc_find_pm_host` is `false`.
  - CTs get created by repeatedly looping over this list to set `pm_host` for each one. For example, if the total number of CTs to be created is 12 and there are 3 hosts in this list, each host would end up with 4 CTs.

- `lxc_construct_padding_width`
  - How many 0 padded charecter to use in names of CTs.
  - Default `2`.

- `lxc_construct_vmid_start`
  - The VMID for the first CT and incremented for each one following it.
  - Default: None.

- `lxc_cores`
  - The number of CPU Cores for all CTs.
  - Default: `4`.

- `lxc_cpus`
  - The limit for the number of CPUs for all CTs.
  - Default: None (is unset/omitted).

- `lxc_cpuunits`
  - The number of CPU Units for all CTs.
  - Default: None (is unset/omitted).

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
  - `true`: Set `lxc_pm_host` to the node with the most free memory right before creating each CT.
  - `false`: Don't change `lxc_pm_host`.

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
  - Default: None (is unset/omitted).

- `lxc_network_gw6`
  - The default IPv6 gateway used for all CTs.
  - Default: None (is unset/omitted).

- `lxc_network_ipv6_ula_prefix`
  - The IPv6 Prefix for Unique Local Addresses (ULA).
    For example, `fd01:2345:6789:abcd::/64` or `fd01:2345:6789::/48`.
  - Default: None (is unset/omitted).
  - The interface suffix will be generated from an MD5 hash of the CT name and the SSH public keys and `48` bits long. This allows for an 80 bit prefix like `fd01:2345:6789:abcd:1::/64` to be used without conficting.
  - Four character snippets of the hash will be used to fill in the address.

- `lxc_network1_bridge`
  - The bridge network on `net1`.
  - Default: `vmbr1`.

- `lxc_network1_gw`
  - The gateway used on `net1`.
  - Default: None (is unset/omitted).

- `lxc_network1_gw6`
  - The IPv6 gateway used on `net1`.
  - Default: None (is unset/omitted).

- `lxc_network1_ipv6_ula_prefix`
  - The IPv6 Prefix for Unique Local Addresses (ULA) on `net1`.
    For example, `fd01:2345:6789:abcd::/64` or `fd01:2345:6789::/48`.
  - Default: None (is unset/omitted).
  - The interface suffix will be generated from an MD5 hash of the VM name and the SSH public keys and `48` bits long. This allows for an 80 bit prefix like `fd01:2345:6789:abcd:1::/64` to be used without conficting.
  - Four character snippets of the hash will be used to fill in the address.

- `lxc_network2_bridge`
  - The bridge network on `net2`.
  - Default: `vmbr2`.

- `lxc_network2_gw`
  - The gateway used on `net2`.
  - Default: None (is unset/omitted).

- `lxc_network2_gw6`
  - The IPv6 gateway used on `net2`.
  - Default: None (is unset/omitted).

- `lxc_network2_ipv6_ula_prefix`
  - The IPv6 Prefix for Unique Local Addresses (ULA) on `net2`.
    For example, `fd01:2345:6789:abcd::/64` or `fd01:2345:6789::/48`.
  - Default: None (is unset/omitted).
  - The interface suffix will be generated from an MD5 hash of the VM name and the SSH public keys and `48` bits long. This allows for an 80 bit prefix like `fd01:2345:6789:abcd:1::/64` to be used without conficting.
  - Four character snippets of the hash will be used to fill in the address.

- `lxc_network3_bridge`
  - The bridge network on `net3`.
  - Default: `vmbr3`.

- `lxc_network3_gw`
  - The gateway used on `net3`.
  - Default: None (is unset/omitted).

- `lxc_network3_gw6`
  - The IPv6 gateway used on `net3`.
  - Default: None (is unset/omitted).

- `lxc_network3_ipv6_ula_prefix`
  - The IPv6 Prefix for Unique Local Addresses (ULA) on `net3`.
    For example, `fd01:2345:6789:abcd::/64` or `fd01:2345:6789::/48`.
  - Default: None (is unset/omitted).
  - The interface suffix will be generated from an MD5 hash of the VM name and the SSH public keys and `48` bits long. This allows for an 80 bit prefix like `fd01:2345:6789:abcd:1::/64` to be used without conficting.
  - Four character snippets of the hash will be used to fill in the address.

- `lxc_pm_api_host`
  - Proxmox host for API connection.
  - Default: `proxmox_master`.

- `lxc_pm_api_user`
  - Proxmox user for API connection.
  - Default: `devops@pve`.

- `lxc_pm_api_token_file`
  - File name of the API Token Secret.
  - Default: `lxc_pm_api_user` + `-` + `lxc_pm_api_token_name`, replacing the `@` with `-`. For example, `devops-pve-ansible` from the default values.

- `lxc_pm_api_token_name`
  - Proxmox user specific API token for API connection.
  - Default: `ansible`.

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
  - Default: None (is unset/omitted).
  - Required. Because leaving this empty will result in CTs to which you cannot login.
  - See `pubkey` variable for the [community.proxmox.proxmox](https://docs.ansible.com/ansible/latest/collections/community/proxmox/proxmox_module.html) module.

- `lxc_storage`
  - The Datacenter storage for all CT disks.
  - Default: `local-thin`.
  - This storage _must_ exist on each PVE node on which VMs are created and be configured to support the "Container" content type.
  - For PVE clusters, consider using shared storage for better migration support.
  - Common values include: `local-lvm`, `local-thin`, `local-zfs` and `ceph`.

- `lxc_swap`
  - The size allocated to swap on all CTs.
  - Default: `0`.

- `lxc_tags`
  - A list of Proxmox tags to give all CTs.
  - Default: None (is unset/omitted).

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
  - A VLAN tag number for all CT `net0` networks.
  - Default: None (is unset/omitted).
  - This also gets prepended to any provided VMIDs.

- `lxc_vlan1_tag`
  - A VLAN tag number for all CT `net1` networks.
  - Default: None (is unset/omitted).

- `lxc_vlan2_tag`
  - A VLAN tag number for all CT `net2` networks.
  - Default: None (is unset/omitted).

- `lxc_vlan3_tag`
  - A VLAN tag number for all CT `net3` networks.
  - Default: None (is unset/omitted).

- `lxc_wait_for_connection`
  - `true`: Wait for the new systems to be accessible.
  - `false`: Do not wait.
  - Default: `true`.

# Special Variable Notes

## `lxc_template` and `lxc_template_storage`

The specified template file must be located in the `lxc_template_storage` storage which must be configured to contain "Containers" content. Using the default `local` storage as an example, this is a directory type storage with the path `/var/lib/vz` and contains the sub-directory `/var/lib/vz/template` under which a sub-directory named `cache` exists to hold the templates. The full default path is `/var/lib/vz/template/cache/` for the template file location.

The template must exist on every host where a CT will be created. They can be uploaded to the storage or downloaded using the "Download from URL" option in the UI or using many other methods of your choosing.

## `lxc_cts`

This variable must be a list of hashs that contain specific key/value pairs defining the CTs to create.

Required keys:

- `name`
  - The name of the CT and becomes the hostname of the system.

Optional keys:

- `description`
  - Becomes the Notes for a CT in the PVE UI Summary page.

- `ip`
  - Static IPv4 address (in CIDR notation) for `net0`.

- `ip6`
  - Static IPv6 address (in CIDR notation) for `net0`.

- `mounts`
  - A list of optional mounts for this CT in the format of a hash. See the first example.

- `net1_ip`
  - Static IPv4 address (in CIDR notation) for `net1`.
  - The `net1` interface is not configured when this and `net1_ip6` are not set.

- `net1_ip6`
  - Static IPv6 address (in CIDR notation) for `net1`.
  - The `net1` interface is not configured when this and `net1_ip` are not set.

- `net2_ip`
  - Static IPv4 address (in CIDR notation) for `net2`.
  - The `net2` interface is not configured when this and `net2_ip6` are not set.

- `net2_ip6`
  - Static IPv6 address (in CIDR notation) for `net2`.
  - The `net2` interface is not configured when this and `net2_ip` are not set.

- `net3_ip`
  - Static IPv4 address (in CIDR notation) for `net3`.
  - The `net3` interface is not configured when this and `net3_ip6` are not set.

- `net3_ip6`
  - Static IPv6 address (in CIDR notation) for `net3`.
  - The `net3` interface is not configured when this and `net3_ip` are not set.

- `pm_host`
  - Overrides `lxc_pm_host` for this CT.

- `storage`
  - Overrides `lxc_storage` for this CT.

- `tags`
  - Overrides `lxc_tags` for this CT.

- `vmid`
  - A _unique_ VMID number within the Datacenter.
  - Default: Not set, and let PVE select the next available VMID.
  - When set, this will also have the `lxc_vlan_tag` prepended to it.

# Dependencies

This role depends on the [community.proxmox.proxmox](https://docs.ansible.com/ansible/latest/collections/community/proxmox/proxmox_module.html) module.

# Example Playbooks

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
