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

The default values for the above mentioned cloud-init images are `local` and the VMID `00`. This VMID is safe to use for the location path of these images because automatically generated VMIDs start at 100, the UI doesn't allow the creation of a VMID less than 100 and attempting to create one using the CLI with 0 or 00 will select the next available VMID greater than 99.

Hint: The `cloudcodger.proxmox_openssh.proxmox_ve` and `cloudcodger.proxmox_openssh.proxmox_datacenter` roles can be used to create the above directory, upload cloud-init images, and set `local` to allow Disk Images content. However, there is no dependency on these roles as the storage requirements can be satisfied by manual configuration.

# Role Variables

## Default Variables.

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
- `cloud_init_construct_vmid_start`
- `cloud_init_sshkeys`

## All variables.

- `cloud_init_agent`
  - `true`: enable the QEMU guest agent option.
  - `false`: leave the QEMU guest agent disabled.
  - Default: `0`
  - This can be a bool or string. 0 = false, 1 = true.

- `cloud_init_ansible_host`
  - `true`: Create a `host_vars` file for each VM and set `ansible_host` in it.
  - `false`: Don't.
  - Default: `false`.
  - When creating VMs that use DHCP, it can be difficult to get the IP address assigned. This allows the role to set `ansible_host` to the IP address of the guest.

- `cloud_init_ansible_host_vars_paths`
  - List of directory paths in which to look for `host_vars`.
  - Default:
    - "{{ ansible_inventory_sources }}"
    - "{{ ansible_inventory_sources[0] | dirname }}"
  - Make sure to set this when setting `cloud_init_ansible_host` to `true` and the `host_vars` is located at a non-standard path.

- `cloud_init_ansible_host_vars_dir`
  - Directory containing the individual host_vars files.
  - Default: The first `host_vars` directory found in `cloud_init_ansible_host_vars_paths`.

- `cloud_init_ansible_inventory_refresh`
  - `true`: Perform a `refresh_inventory` after the last task in this role.
  - `false`: Don't.
  - Default: `false`.
  - This is only done when a VM is created or started.
  - When calling this role in a playbook followed by another play to configure the VM, a newly created host will not be found in the inventory unless this is done.

- `cloud_init_bypass_wait_for_connection`
  - `true`: Do not wait for the new systems to be accessible.
  - `false`: Do wait.
  - Default: `false`.

- `cloud_init_construct_cidr_start`
  - The first IPv4 address in CIDR notation. For example,
    `cloud_init_construct_cidr_start: "192.168.1.21/24"`.
  - Default: None.
  - Required when using `cloud_init_construct_*`.

- `cloud_init_construct_cidr1_start`
  - The first IPv4 address in CIDR notation for `net1`.
    For example, `cloud_init_construct_cidr1_start: "192.168.11.21/24"`.
  - Default: None.

- `cloud_init_construct_cidr2_start`
  - The first IPv4 address in CIDR notation for `net2`.
    For example, `cloud_init_construct_cidr1_start: "192.168.21.21/24"`.
  - Default: None.

- `cloud_init_construct_cidr3_start`
  - The first IPv4 address in CIDR notation for `net3`.
    For example, `cloud_init_construct_cidr1_start: "192.168.31.21/24"`.
  - Default: None.

- `cloud_init_construct_cidr6_start`
  - The first IPv6 address in CIDR notation.
  - Default: None.

- `cloud_init_construct_controllers`
  - The number of VMs to create with `control: true`.
  - Default: `0`.

- `cloud_init_construct_ctrl_prefix`
  - The name of all VMs with `control: true` with an index number appended.
  - Default: `ctrl`.

- `cloud_init_construct_hosts`
  - A list of Proxmox host names on which to create the VMs.
  - Default `[]`.
  - Required when using `cloud_init_construct_*`.
  - VMs get created by repeatedly looping over this list to set `pm_host` for each one. For example, if the total number of VMs to be created is 12 and there are 3 hosts in this list, each host would end up with 4 VMs created on it.

- `cloud_init_construct_vmid_start`
  - The VMID for the first VM and incremented for each one following it.
  - Default: None (is omitted and PVE will auto assign one).

- `cloud_init_construct_work_prefix`
  - The name of all VMs with `control: false` with an index number appended.
  - Default: `work`.

- `cloud_init_construct_workers`
  - The number of VMs to create with `control: false`.
  - Default: `0`.

- `cloud_init_cpu`
  - The CPU type for the vm.
  - Default: None (is omitted and PVE will auto assign one).

- `cloud_init_ctrl_cores`
  - The number of CPU Cores for VMs with `control: true`.
  - Default: `4`.

- `cloud_init_ctrl_disk_size`
  - The size for the disk for VMs with `control: true`.
  - Default: `12G`.

- `cloud_init_ctrl_memory`
  - The memory size for VMs with `control: true`.
  - Default: `8192`.

- `cloud_init_ctrl_sockets`
  - The number of CPUs for VMs with `control: true`.
  - Default: `1`.

- `cloud_init_ctrl_tags`
  - A list of Proxmox tags assigned to VMs with `control: true`.
  - Default: `control,{{ cloud_init_k8s_tag }}`.

- `cloud_init_custom`
  - See the [cloud_init_custom](#cloud_init_custom) section below for details.
  - Default: omitted.

- `cloud_init_disk_storage`
  - The storage for holding the VMs cloud-init disks.
  - Default: `{{ cloud_init_storage }}`.
  - This is _not_ where the disk images for the VM are created.

- `cloud_init_genesis_tags`
  - A list of Proxmox tags assigned to for VMs with `genesis: true`.
  - Default: `control,genesis,{{ cloud_init_k8s_tag }}`.
    This replaces the `cloud_init_ctrl_tags` for that VM.

- `cloud_init_find_pm_host`
  - `true`: Set `cloud_init_pm_host` to the node with the most free memory.
  - `false`: Don't change `cloud_init_pm_host` using this check.
  - This calculation is performed right before creating each VM.

- `cloud_init_image`
  - Name of the Cloud-Init disk image file for all VMs.
  - Default: `ubuntu-24.04-server.qcow2`.
  - The specified image file _must_ be located in the `cloud_init_image_storage` storage, under the `cloud_init_vmid` directory on all PVE nodes where it will be used.
  - Example directory location using the defaults: `/var/lib/vz/images/00/`.
  - Because of the fictitious VMID, the image files cannot be uploaded in the UI and must be placed there using something like `scp` or downloaded directly on each host system using a utility like `curl` or `wget`.
  - Tip: The Ansible Collection [cloudcodger.proxmox_openssh](https://galaxy.ansible.com/cloudcodger/proxmox_openssh) contains two roles that can be used to create the directory, set the content, and upload images to all the hosts in a cluster.

- `cloud_init_image_storage`
  - The storage containing the Cloud-Init image.
  - Default: `local`.
  - This storage must be configured to contain "Disk Images" content, which is located under the path `/var/lib/vz/images`. This contains a sub-directory for each VMID. The `cloud_init_vmid` indicates the directory to use.
  - The images referenced here are not disks used by a VM.

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
  - Default: unset and omitted.

- `cloud_init_network1_bridge`
  - The bridge network used for all VMs.
  - Default: `vmbr1`.

- `cloud_init_network2_bridge`
  - The bridge network used for all VMs.
  - Default: `vmbr2`.

- `cloud_init_network3_bridge`
  - The bridge network used for all VMs.
  - Default: `vmbr3`.

- `cloud_init_network1_gw`
  - The gateway used on `net1` for all VMs.
  - Default: unset and omitted.

- `cloud_init_network2_gw`
  - The gateway used on `net2` for all VMs.
  - Default: unset and omitted.

- `cloud_init_network3_gw`
  - The gateway used on `net3` for all VMs.
  - Default: unset and omitted.

- `cloud_init_network_gw6`
  - The default IPv6 gateway.
  - Default: unset and omitted.

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
  - Default: unset and omitted.
  - Caution: Leaving this empty will result in VMs to which you cannot login until keys are added using other methods.
  - See `sshkeys` variable for the [community.general.proxmox_kvm](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html) module.

- `cloud_init_storage`
  - The Datacenter storage for VM disks.
  - Default: `local-thin`.
  - This storage _must_ exist on each PVE node on which VMs are created and be configured to support the "Disk Image" content type.
  - For PVE clusters, consider using shared storage for better migration support.
  - Common values include: `local`, `local-thin`, `local-zfs` and `ceph0`.

- `cloud_init_tags`
  - List of tags assigned to non-construct VMs.
  - Default: `[]`
  - Not used for construct VMs which use ``cloud_init_ctrl_tags`, `cloud_init_genesis_tags` and `cloud_init_work_tags`.

- `cloud_init_update_known_hosts`
  - `true`: Look up the SSH host keys and update the local `.ssh/known_hosts` file.
  - `false`: Skip these tasks.
  - Default: `true`.

- `cloud_init_vlan_tag`
  - A VLAN tag number for all VM networks.
  - Default: Not set
  - This also gets prepended to any provided VMIDs.

- `cloud_init_vlan1_tag`
  - A VLAN tag number for all VM `net1` networks.
  - Default: Not set

- `cloud_init_vlan2_tag`
  - A VLAN tag number for all VM `net2` networks.
  - Default: Not set

- `cloud_init_vlan3_tag`
  - A VLAN tag number for all VM `net3` networks.
  - Default: Not set

- `cloud_init_vmid`
  - The image directory containing the Cloud-Init image.
  - Default: `00`.
  - The default is a fictitious VMID that is otherwise invalid for a VM to use.
  - To avoid confusion and collision with an actual VM, either use this default or be careful about the VMID selected.

- `cloud_init_vms`
  - See the [cloud_init_vms](#cloud_init_vms) section below for details.
  - Default: `[]`.

- `cloud_init_work_cores`
  - The number of CPU Cores for VMs with `control: false`.
  - Default: `2`.

- `cloud_init_work_disk_size`
  - The size for the disk for VMs with `control: false`.
  - Default: `12`.

- `cloud_init_work_memory`
  - The memory size for VMs with `control: false`.
  - Default: `8192`.

- `cloud_init_work_sockets`
  - The number of CPUs for VMs with `control: false`.
  - Default: `1`.

- `cloud_init_work_tags`
  - A list of Proxmox tags assigned to VMs with `control: false`.
  - Default: `worker,{{ cloud_init_k8s_tags }}`.

# Special Variable Notes

## `cloud_init_custom`

This allows for custom settings to be specied for the VMs using the [Custom Cloud-Init Configuration](https://pve.proxmox.com/wiki/Cloud-Init_Support#_custom_cloud_init_configuration). The command `man qm` on a Proxmox host will provide more information on the `cicustom` parameter to the `qm create` command, which this sets.
An example value would be `vendor=configs:snippets/vendor-data.yml`.

Usage observation: When using the `cicustom` option with a value like `user=local:snippets/user-data.yml`, it will override _all_ the user data values that normally get set for the VM. One of those values is `hostname`. If you do not include `hostname` in the snippet file the VM will not get the name of the VM like the default behavior provides. If you include `hostname` in the snippet file, then every VM using the same snippet will get the same hostname. To get around this, each VM would require a unique snippet. Alternatively, if you want to keep all the default user data, use a `vendor=` value instead.

Hint: The Ansible Collection [cloudcodger.proxmox_openssh](https://galaxy.ansible.com/cloudcodger/proxmox_openssh) contains the `proxmox_snippet` role that can be used to create a snippet to use here.

## `cloud_init_vms`

This variable must be a list of hashs that contain specific key/value pairs defining the VMs to create.

Required keys:

- `name`
  - The name of the VM and becomes the hostname of the system.

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
  - Default: `dhcp`.

- `ip1`
  - Static IPv4 address (in CIDR notation) for `net1`.
  - Default: unset.
  - The `net1` interface is not configured when this is not set.

- `ip2`
  - Static IPv4 address (in CIDR notation) for `net2`.
  - Default: unset.
  - The `net2` interface is not configured when this is not set.

- `ip3`
  - Static IPv4 address (in CIDR notation) for `net3`.
  - Default: unset.
  - The `net3` interface is not configured when this is not set.

- `ip6`
  - Static IPv6 address (in CIDR notation).

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

This role depends on the [community.general.proxmox_kvm](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html) module.

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
