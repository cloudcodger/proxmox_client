# `cloud_init` Role

Create PVE VMs using the [Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support) provided in [Proxmox VE](https://pve.proxmox.com/wiki/Main_Page).

By default, if no variables are set, calling this role will not create any VMs.

There are three primary use cases for this role.

1. Create two sets of VMs using all `cloud_init_construct_*` variables.
    - For k8s.
    - Any group of VMs that have two distinct roles. For example, one VM as the primary DNS server (using `ctrl` variables) with small CPU and memory sizes and three secondary DNS servers (using `work` variables) that have different size requirements.
2. Create a set of [identical VMs](#sequencial-vms) using `cloud_init_construct_*` variables.
    - Any group of VMs with a name prefix follow by a sequence number.
3. Create [individually defined](#individually-defined) VMs using the `cloud_init_vms` variable.
    - A single VM.
    - A group of related VMs with different names and/or requirements.

A combination of using both `cloud_init_construct_*` and `cloud_init_vms` is allowed, but discouraged and not recommended. Using both at the same time makes it difficult for others to follow what is being created. To avoid complexity and confusion, it is recommended to use either the `cloud_init_vms` _or_ the `cloud_init_construct_*` settings and not both.

# k8s

This role was originally designed to support the creation of two different groups of VMs as often desired for use with Kubernetes (k8s).
One group for use as k8s control (`ctrl`) nodes and another group for k8s worker (`work`) nodes.

Each group can have different values for the following items:

- the number of CPUs (sockets)
- number of CPU cores
- amount of memory
- disk size
- Proxmox tags

# Sequencial VMs

Recommended configuration is to use the `cloud_init_construct_workers` and `cloud_init_construct_work_*` variables to create a set of identical VMs. For example, when creating three VMs named, `lb1`, `lb2` and `lb3` that all have the same CPU sockets/cores, memory, disk size, and tags.

The `cloud_init_construct_controllers` and `cloud_init_construct_ctrl_*` variables could also be used. Best practice is to pick one and use it for all sequencial VMs.

# Individually Defined

This option provides the creation of multiple VMs that differ in the values set under each item in [`cloud_init_vms`](#cloud_init_vms).

# Requirements

- The `proxmoxer` Python module installed on the Ansible controller.
- Proxmox VE version 8 or above installed on the API host(s).
- Proxmox VE Datacenter Storage with `Disk Image` content allowed.
- A Cloud-Init OS image uploaded to the above storage in a _special_ VMID for this use.

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
you will need to set `cloud_init_pm_api_host`.

At a minimum, set the following to desired values.

- `cloud_init_pm_api_host`
- `cloud_init_vms` and/or the following construct items
- `cloud_init_construct_cidr_start`
- `cloud_init_construct_controllers` and/or `cloud_init_construct_workers`
- `cloud_init_construct_hosts`
- `cloud_init_sshkeys`

## All variables.

- `cloud_init_agent`
  - `true`: enable the QEMU guest agent option.
  - `false`: leave the QEMU guest agent disabled.
  - Default: `0`
  - This can be a bool or string. 0 = false, 1 = true.

- `cloud_init_ansible_inventory_refresh`
  - `true`: Perform a `refresh_inventory` after the last task in this role.
  - `false`: Don't.
  - Default: `false`.
  - This is only done when a VM is created or started.
  - When calling this role in a playbook followed by another play to configure the VM, a newly created host will not be found unless either an inventory refresh is done or it's added using something like `ansible.builtin.add_host`.

- `cloud_init_construct_cidr_start`
  - The first IPv4 address in CIDR notation. For example, `cloud_init_construct_cidr_start: "192.168.1.21/24"`.
  - Default: None.
  - Required when using `cloud_init_construct_*`.

- `cloud_init_construct_cidr_net1_start`
  - The first IPv4 address in CIDR notation for `net1`.
    For example, `cloud_init_construct_cidr_net1_start: "192.168.11.21/24"`.
  - Default: None.

- `cloud_init_construct_cidr_net2_start`
  - The first IPv4 address in CIDR notation for `net2`.
    For example, `cloud_init_construct_cidr_net1_start: "192.168.21.21/24"`.
  - Default: None.

- `cloud_init_construct_cidr_net3_start`
  - The first IPv4 address in CIDR notation for `net3`.
    For example, `cloud_init_construct_cidr_net1_start: "192.168.31.21/24"`.
  - Default: None.

- `cloud_init_construct_cidr6_start`
  - The first IPv6 address in CIDR notation.
  - Default: None.

- `cloud_init_construct_cidr6_net1_start`
  - The first IPv6 address in CIDR notation for `net1`.
  - Default: None.

- `cloud_init_construct_cidr6_net2_start`
  - The first IPv6 address in CIDR notation for `net2`.
  - Default: None.

- `cloud_init_construct_cidr6_net3_start`
  - The first IPv6 address in CIDR notation for `net1`.
  - Default: None.

- `cloud_init_construct_controllers`
  - The number of VMs to create with `control: true`.
  - Default: `0`.

- `cloud_init_construct_hosts`
  - A list of Proxmox host names on which to create the VMs.
  - Default `[]`.
  - Required when using `cloud_init_construct_*` and `cloud_init_find_pm_host` is `false`.
  - VMs get created by repeatedly looping over this list to set `pm_host` for each one. For example, if the total number of VMs to be created is 12 and there are 3 hosts in this list, each host would end up with 4 VMs created on it.

- `cloud_init_construct_padding_width`
  - How many 0 padded charecter to use in names of VMs.
  - Default `2`.

- `cloud_init_construct_vmid_start`
  - The VMID for the first VM and incremented for each one following it.
  - Default: None.

- `cloud_init_construct_workers`
  - The number of VMs to create with `control: false`.
  - Default: `0`.

- `cloud_init_cpu`
  - The CPU type for the vm.
  - Default: None (is unset/omitted).

- `cloud_init_ctrl_cores`
  - The number of CPU Cores for VMs with `control: true`.
  - Default: `4`.

- `cloud_init_ctrl_disk_size`
  - The size for the disk for VMs with `control: true`.
  - Default: `12G`.

- `cloud_init_ctrl_memory`
  - The memory size for VMs with `control: true`.
  - Default: `8192`.

- `cloud_init_ctrl_prefix`
  - The name of all VMs with `control: true` with an index number appended.
  - Default: `ctrl`.

- `cloud_init_ctrl_sockets`
  - The number of CPUs for VMs with `control: true`.
  - Default: `1`.

- `cloud_init_ctrl_tags`
  - A list of Proxmox tags assigned to VMs with `control: true`.
  - Default: `control,{{ cloud_init_k8s_tag }}`.

- `cloud_init_custom`
  - See the [cloud_init_custom](#cloud_init_custom) section below for details.
  - Default: None (is unset/omitted).

- `cloud_init_disk_storage`
  - The storage for holding the VMs cloud-init disks.
  - Default: `local`.
  - This is _not_ where the disk images for the VM are created.

- `cloud_init_find_pm_host`
  - `true`: Set `cloud_init_pm_host` to the node with the most free memory right before creating each VM.
  - `false`: Don't change `cloud_init_pm_host`.

- `cloud_init_genesis_tags`
  - A list of Proxmox tags assigned to the VMs with `genesis: true`.
  - Default: `control,genesis,{{ cloud_init_k8s_tag }}`.
  - This overrides the `cloud_init_ctrl_tags` for VMs with `genesis: true`.

- `cloud_init_image`
  - Name of the Cloud-Init disk image file for all VMs.
  - Default: `ubuntu-24.04-server.qcow2`.
  - The specified image file _must_ be located in `cloud_init_image_storage`, under the `cloud_init_image_storage_content` directory on all PVE nodes where it will be used.

- `cloud_init_image_storage`
  - The storage containing the Cloud-Init image.
  - Default: `local`.
  - This storage must be configured to contain `cloud_init_image_storage_content` content.

- `cloud_init_image_storage_content`
  - The storage content type for the `cloud_init_image` stored in `cloud_init_image_storage`.
  - Default: `import`.

- `cloud_init_k8s_tag`
  - A Proxmox tag added to `cloud_init_ctrl_tags`, `cloud_init_genesis_tags` and `cloud_init_work_tags`.
  - Default: `k8s`.

- `cloud_init_nameservers`
  - A list of name servers for the VMs.
  - Default: `[]`.

- `cloud_init_network_bridge`
  - The bridge network used for all VMs.
  - Default: `vmbr0`.

- `cloud_init_network_gw`
  - The default gateway used for all VMs.
  - Default: None (is unset/omitted).

- `cloud_init_network_gw6`
  - The default IPv6 gateway used for all VMs.
  - Default: None (is unset/omitted).

- `cloud_init_network_ipv6_ula_prefix`
  - The IPv6 Prefix for Unique Local Addresses (ULA).
    For example, `fd01:2345:6789:abcd::/64` or `fd01:2345:6789::/48`.
  - Default: None (is unset/omitted).
  - The interface suffix will be generated from an MD5 hash of the VM name and the SSH public keys and `48` bits long. This allows for an 80 bit prefix like `fd01:2345:6789:abcd:1::/64` to be used without conficting.
  - Four character snippets of the hash will be used to fill in the address.

- `cloud_init_network1_bridge`
  - The bridge network on `net1`.
  - Default: `vmbr1`.

- `cloud_init_network1_gw`
  - The gateway used on `net1`.
  - Default: None (is unset/omitted).

- `cloud_init_network1_gw6`
  - The IPv6 gateway used on `net1`.
  - Default: None (is unset/omitted).

- `cloud_init_network1_ipv6_ula_prefix`
  - The IPv6 Prefix for Unique Local Addresses (ULA) on `net1`.
    For example, `fd01:2345:6789:abcd::/64` or `fd01:2345:6789::/48`.
  - Default: None (is unset/omitted).
  - The interface suffix will be generated from an MD5 hash of the VM name and the SSH public keys and `48` bits long. This allows for an 80 bit prefix like `fd01:2345:6789:abcd:1::/64` to be used without conficting.
  - Four character snippets of the hash will be used to fill in the address.

- `cloud_init_network2_bridge`
  - The bridge network used on `net2`.
  - Default: `vmbr2`.

- `cloud_init_network2_gw`
  - The gateway used on `net2`.
  - Default: None (is unset/omitted).

- `cloud_init_network2_gw6`
  - The IPv6 gateway used on `net2`.
  - Default: None (is unset/omitted).

- `cloud_init_network2_ipv6_ula_prefix`
  - The IPv6 Prefix for Unique Local Addresses (ULA) on `net2`.
    For example, `fd01:2345:6789:abcd::/64` or `fd01:2345:6789::/48`.
  - Default: None (is unset/omitted).
  - The interface suffix will be generated from an MD5 hash of the VM name and the SSH public keys and `48` bits long. This allows for an 80 bit prefix like `fd01:2345:6789:abcd:1::/64` to be used without conficting.
  - Four character snippets of the hash will be used to fill in the address.

- `cloud_init_network3_bridge`
  - The bridge network used on `net3`.
  - Default: `vmbr3`.

- `cloud_init_network3_gw`
  - The gateway used on `net3`.
  - Default: None (is unset/omitted).

- `cloud_init_network3_gw6`
  - The IPv6 gateway used on `net3`.
  - Default: None (is unset/omitted).

- `cloud_init_network3_ipv6_ula_prefix`
  - The IPv6 Prefix for Unique Local Addresses (ULA) on `net3`.
    For example, `fd01:2345:6789:abcd::/64` or `fd01:2345:6789::/48`.
  - Default: None (is unset/omitted).
  - The interface suffix will be generated from an MD5 hash of the VM name and the SSH public keys and `48` bits long. This allows for an 80 bit prefix like `fd01:2345:6789:abcd:1::/64` to be used without conficting.
  - Four character snippets of the hash will be used to fill in the address.

- `cloud_init_pm_api_host`
  - Proxmox host for API connection.
  - Default: `proxmox_master`.

- `cloud_init_pm_api_user`
  - Proxmox user for API connection.
  - Default: `devops@pve`.

- `cloud_init_pm_api_token_file`
  - File name of the API Token Secret.
  - Default: `cloud_init_pm_api_user` + `-` + `cloud_init_pm_api_token_name`, replacing the `@` with `-`. For example, `devops-pve-ansible` from the default values.

- `cloud_init_pm_api_token_name`
  - Proxmox user specific API token for API connection.
  - Default: `ansible`.

- `cloud_init_pm_api_token_secret`
  - The API token secret for API connection.
  - Default: Read from `cloud_init_pm_api_token_file`.
  - Caution! Setting this directly may accidentally expose the value.
  - This is a role var, which changes the override precedence.
    See [understanding-variable-precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence) for more information.

- `cloud_init_pm_host`
  - The default Proxmox host on which to create all the VMs.
  - Defualt: `{{ cloud_init_pm_api_host }}`.

- `cloud_init_searchdomains`
  - A list of domains to search for DNS resolution.
  - Default: `[]`.

- `cloud_init_secrets_dir`
  - Local directory in which the API Token Secret file is located.
  - Default: `~/.pve_tokens`.

- `cloud_init_sshkeys`
  - A multi-line string containing a list of public SSH keys for the default user login on all VMs.
  - Default: None (is unset/omitted).
  - Caution: Leaving this empty will result in VMs to which you cannot login until a password or keys are added using other methods.
  - See `sshkeys` variable for the [community.proxmox.proxmox_kvm](https://docs.ansible.com/ansible/latest/collections/community/proxmox/proxmox_kvm_module.html) module.

- `cloud_init_storage`
  - The Datacenter storage for VM disks.
  - Default: `local-thin`.
  - This storage _must_ exist on each PVE node on which VMs are created and be configured to support the "Disk Image" content type.
  - For PVE clusters, consider using shared storage for better migration support.
  - Common values include: `local-lvm`, `local-thin`, `local-zfs` and `ceph`.

- `cloud_init_tags`
  - List of default tags assigned to `cloud_init_vms` VMs, when `tags` is not provided for an entry.
  - Default: `[]`
  - For construct VMs use `cloud_init_ctrl_tags`, `cloud_init_genesis_tags` and `cloud_init_work_tags` instead.

- `cloud_init_update_known_hosts`
  - `true`: Look up the SSH host keys and update `.ssh/known_hosts` file on the Ansible control host.
  - `false`: Skip these tasks.
  - Default: `true`.

- `cloud_init_vlan_tag`
  - A VLAN tag number for all VM `net0` networks.
  - Default: None (is unset/omitted).
  - This also gets prepended to any provided VMIDs.

- `cloud_init_vlan1_tag`
  - A VLAN tag number for all VM `net1` networks.
  - Default: None (is unset/omitted).

- `cloud_init_vlan2_tag`
  - A VLAN tag number for all VM `net2` networks.
  - Default: None (is unset/omitted).

- `cloud_init_vlan3_tag`
  - A VLAN tag number for all VM `net3` networks.
  - Default: None (is unset/omitted).

- `cloud_init_vms`
  - See the [cloud_init_vms](#cloud_init_vms) section below for details.
  - Default: `[]`.

- `cloud_init_wait_for_connection`
  - `true`: Wait for the new systems to be accessible.
  - `false`: Do not wait.
  - Default: `false`.

- `cloud_init_work_cores`
  - The number of CPU Cores for VMs with `control: false`.
  - Default: `2`.

- `cloud_init_work_disk_size`
  - The size for the disk for VMs with `control: false`.
  - Default: `12`.

- `cloud_init_work_memory`
  - The memory size for VMs with `control: false`.
  - Default: `8192`.

- `cloud_init_work_prefix`
  - The name of all VMs with `control: false` with an index number appended.
  - Default: `work`.

- `cloud_init_work_sockets`
  - The number of CPUs for VMs with `control: false`.
  - Default: `1`.

- `cloud_init_work_tags`
  - A list of Proxmox tags assigned to VMs with `control: false`.
  - Default: `worker,{{ cloud_init_k8s_tags }}`.

# Special Variable Notes

## `cloud_init_custom`

This allows for custom settings to be specied for the VMs using the [Custom Cloud-Init Configuration](https://pve.proxmox.com/wiki/Cloud-Init_Support#_custom_cloud_init_configuration). The command `man qm` on a Proxmox host will provide more information on the `cicustom` parameter to the `qm create` command, which this sets.
An example value would be `vendor=local:snippets/vendor-data.yml`.

Usage observation: When using the `cicustom` option with a value like `user=local:snippets/user-data.yml`, it will override _all_ the user data values that normally get set for the VM. One of those values is `hostname`. If you do not include `hostname` in the snippet file the VM will not get the name of the VM like the default behavior provides. If you include `hostname` in the snippet file, then every VM using the same snippet will get the same hostname. To get around this, each VM would require a unique snippet. Alternatively, if you want to keep all the default user data, use a `vendor=` value instead.

## `cloud_init_vms`

This variable must be a list of hashs that contain specific key/value pairs defining the VMs to create.

Required keys:

- `name`
  - The name of the VM, that also becomes the hostname of the system.

Optional keys:

- `control`
  - Sets if the VM is a control plane node or worker node.
  - Default: `false`.
  - `true`: use the `cloud_init_ctrl_*` values.
  - `false`: use the `cloud_init_work_*` values.

- `custom`
  - Overrides the global value for this machine.
  - See [`cloud_init_custom`](#cloud_init_custom) above.

- `description`
  - Becomes the Notes for a VM in the PVE UI Summary page.

- `genesis`
  - `true`: tag the VM with `cloud_init_genesis_tags`.
  - `false`: tag the VM with tags according to the `control` value.
  - Default: `false`.
  - Used for kubernetes clustering in other places and intended to only be set to `true` on at most one VM, and normally on a VM where `control: true`.

- `ip`
  - Static IPv4 address (in CIDR notation) for `net0`.

- `ip6`
  - Static IPv6 address (in CIDR notation) for `net0`.

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
  - Overrides `cloud_init_pm_host` for this machine.

- `storage`
  - Overrides `cloud_init_storage` for this machine.

- `tags`
  - Overrides `cloud_init_tags` for this machine.

- `vmid`
  - A _unique_ VMID number within the Datacenter.
  - Default: Not set, and let PVE select the next available VMID.
  - When set, this will also have the `cloud_init_vlan_tag` prepended to it.

# Dependencies

This role depends on the [community.proxmox.proxmox_kvm](https://docs.ansible.com/ansible/latest/collections/community/proxmox/proxmox_kvm_module.html) module.

# Example Playbooks

```yaml
---
- name: >-
    Create a single VM using the default control node values and
    not having any Proxmox tags.
  hosts: localhost
  gather_facts: false
  roles:
    - role: cloudcodger.proxmox_client.cloud_init
      cloud_init_pm_api_host: "192.168.1.21"

      cloud_init_ctrl_tags: []
      cloud_init_network_gw: "192.168.1.1"
      cloud_init_pm_host: pve1
      cloud_init_sshkeys: |
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIN+K0KwgKOeeyW519YavKQodVgwWcRUIucZkOfplsKMl devops-guy-mbp
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJk/hSmxsyznAhhsD5cO9wcTgOs+/xz09kZ5woSUUQAY devops-gal-mbp
      cloud_init_vms:
        - name: "tester1"
          control: true
          ip: "192.168.1.119/24"
          vmid: "119"
```

```yaml
- name: >-
    Create a set of VMs for Kubernetes spread across 3 Proxmox hosts with 3 control nodes and 3 worker nodes
    spanning the three hosts.
    Connecting to the API using a different user and token file.
  hosts: localhost
  gather_facts: false
  roles:
    - role: cloudcodger.proxmox_client.cloud_init
      cloud_init_pm_api_host: "192.168.1.21"
      cloud_init_pm_api_user: "s1m0ne@pve"
      cloud_init_pm_api_token_file: lab-s1m0ne-pve-ansible.token

      cloud_init_network_gw: "192.168.1.1"
      cloud_init_pm_host: pve1
      cloud_init_sshkeys: |
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIN+K0KwgKOeeyW519YavKQodVgwWcRUIucZkOfplsKMl devops-guy-mbp
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJk/hSmxsyznAhhsD5cO9wcTgOs+/xz09kZ5woSUUQAY devops-gal-mbp
      cloud_init_storage: 'local-zfs'
      cloud_init_vms:
        - name: "regulator1"
          control: true
          genesis: true
          ip: "192.168.6.121/24"
          vmid: "99121"
        - name: "regulator2"
          control: true
          ip: "192.168.6.122/24"
          pm_host: "pve2"
          vmid: "99122"
        - name: "regulator3"
          control: true
          ip: "192.168.6.123/24"
          pm_host: "pve3"
          vmid: "99123"
        - name: "motor1"
          control: false
          ip: "192.168.6.125/24"
          vmid: "99125"
        - name: "motor2"
          control: false
          ip: "192.168.6.126/24"
          pm_host: "pve2"
          vmid: "99126"
        - name: "motor3"
          control: false
          ip: "192.168.6.127/24"
          pm_host: "pve3"
          vmid: "99127"
```

```yaml
- name: >-
    Create a set of VMs for Kubernetes across 3 Proxmox hosts with 3 control nodes and 6 worker nodes
    spanning the three hosts using the construct variables and configuring IPv6.
    Put VM root disk and Cloud-Init disk on 'ceph' storage.
  hosts: localhost
  gather_facts: false
  roles:
    - role: cloudcodger.proxmox_client.cloud_init
      cloud_init_pm_api_host: "192.168.1.21"

      cloud_init_network_gw: "192.168.1.1"
      cloud_init_network_gw6: "2607:fa18:d7fe:ff00::1"
      cloud_init_pm_host: pve1
      cloud_init_sshkeys: |
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIN+K0KwgKOeeyW519YavKQodVgwWcRUIucZkOfplsKMl devops-guy-mbp
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJk/hSmxsyznAhhsD5cO9wcTgOs+/xz09kZ5woSUUQAY devops-gal-mbp

      cloud_init_construct_cidr_start: "192.168.6.131/24"
      cloud_init_construct_cidr6_start: "2607:fa18:d7fe:ff00::83/64"
      cloud_init_construct_controllers: 3
      cloud_init_construct_hosts: [ 'gadget1', 'gadget2', 'gadget3' ]
      cloud_init_construct_workers: 6
      cloud_init_construct_vmid_start: 99131
      cloud_init_disk_storage: ceph
      cloud_init_storage: ceph
```
