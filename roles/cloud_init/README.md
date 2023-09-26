`cloud_init`
============

A role to create two distinct sets of identically configured Proxmox virtual machines using the [Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support) provided in [Proxmox VE](https://pve.proxmox.com/wiki/Main_Page).

This role is designed to support the creation of two different groups of VMs as often desired for use with Kubernetes.
One group for use as control (`ctrl`) nodes and another group for worker (`work`) nodes as seen by the choice in variable naming.
Although these groups were designed specifically for creating VMs to be used with Kubernetes, they are not limited to this use.
You could use this for any group of VMs that have two distinct roles for specific systems.
For example, one VM as the primary DNS server (using `ctrl` variables) with small CPU and memory sizes and three secondary DNS servers (using `work` variables) that have different size requirements.

Each group can have different values for the following items:

- the number of CPUs (sockets)
- number of CPU cores
- amount of memory
- disk size
- Proxmox tags

Different configurations can create;

- a single VM (recommend using `cloud_init_vms` for this use case)
- a set of VMs using the `cloud_init_vms` variable
- two sets of VMs using the `cloud_init_vms` variable
- a "contructed" set of VMs using `cloud_init_construct_*` variables
- two "contructed" sets of VMs using `cloud_init_construct_*` variables
- a combination of the above settings (not recommended)

To avoid complexity and possible confusion, it is recommended to use either the `cloud_init_vms` _or_ the `cloud_init_construct_*` settings and not both.

Requirements
------------

- The `proxmoxer` Python module installed on the Ansible controller.
- Proxmox VE version 8 or above installed on the API host(s).
- Proxmox VE Datacenter Storage with `Disk Image` content allowed.
- A Cloud-Init OS image uploaded to the above storage in a _special_ VMID for this use.

The default values for the above mentioned cloud-init images are `local` and the VMID `00`. This VMID is safe to use for the location path of these images because automatically generated VMIDs start at 100, the UI doesn't allow the creation of a VMID less than 100 and attempting to create one using the CLI with 0 or 00 will select the next available VMID greater than 99.

Hint: The `cloudcodger.proxmox_openssh.proxmox_ve` and `cloudcodger.proxmox_openssh.proxmox_datacenter` roles can be used to create the above directory, upload cloud-init images, and set `local` to allow Disk Images content. However, there is no dependency on these roles as the storage requirements can be satisfied by manual configuration.

Role Variables
--------------

### Required variables.

The default values for the following variables will very likely not be desired or possibly even work.
For example, unless you named your Proxmox host `proxmox_master` and can resolve it via the DNS,
you will need to set `cloud_init_pm_api_host`.

At a minimum, set the following to desired values.

- `cloud_init_pm_api_host`
- `cloud_init_vms` and/or the following construct items
- `cloud_init_construct_cidr_start`
- `cloud_init_construct_controllers` and/or `cloud_init_construct_workers`
- `cloud_init_construct_hosts`
- `cloud_init_construct_vmid_start`
- `cloud_init_network_gw`
- `cloud_init_sshkeys`

### All variables.

- `cloud_init_pm_api_host` - Proxmox host for API connection (default: `proxmox_master`).
- `cloud_init_pm_api_user` - Proxmox user for API connection (default: `devops@pve`).
- `cloud_init_pm_api_token_name` - Proxmox user specific API token for API connection (default: `ansible`).
- `cloud_init_secrets_dir` - Local directory in which the API Token Secret file is located (default: `~/.pve_tokens`).
- `cloud_init_pm_api_token_file` - File name of the API Token Secret.
  The default is to construct this value from the `cloud_init_pm_api_user` and `pm_api_token`, replacing the `@` with `-`.
  For example, `devops-pve-ansible` from the above default values.
- `cloud_init_pm_api_token_secret` - Read from the above file be default.
  Caution! Setting this directly may accidentally expose the value.
- `cloud_init_vms` - See the sub-section below for details (default: `[]`).
- `cloud_init_construct_cidr_start` - The first IPv4 address in CIDR notation.
- `cloud_init_construct_cidr6_start` - The first IPv6 address in CIDR notation.
- `cloud_init_construct_ctrl_prefix` - The name of all VMs with `control: true` with an index number appended (default: `ctrl`).
- `cloud_init_construct_controllers` - The number of VMs to create with `control: true` (default: `0`).
- `cloud_init_construct_hosts` - A list of Proxmox host names on which to create the VMs (default `[]`).
  VMs get created by repeatedly looping over this list to set `pm_host` for each one.
  For example, if the total number of VMs to be created is 12 and there are 3 hosts in this list, each host would end up with 4 VMs created on it.
- `cloud_init_construct_work_prefix` - The name of all VMs with `control: false` with an index number appended (default: `work`).
- `cloud_init_construct_workers` - The number of VMs to create with `control: false` (default: `0`).
- `cloud_init_construct_vmid_start` - The VMID for the first VM and incremented for each one following it.
- `cloud_init_ctrl_cores` - The number of CPU Cores for VMs with `control: true` (default: `4`).
- `cloud_init_ctrl_disk_size` - The size for the disk for VMs with `control: true` (default: `12G`).
- `cloud_init_ctrl_memory` - The memory size for VMs with `control: true` (default: `8192`).
- `cloud_init_ctrl_sockets` - The number of CPUs for VMs with `control: true` (default: `1`).
- `cloud_init_ctrl_tags` - A list of Proxmox tags assigned to VMs with `control: true` (default: `control,{{ cloud_init_k8s_tag }}`).
- `cloud_init_genesis_tags` - A list of Proxmox tags assigned to for VMs with `genesis: true` (default: `control,genesis,{{ cloud_init_k8s_tag }}`).
- `cloud_init_work_cores` - The number of CPU Cores for VMs with `control: false` (default: `2`).
- `cloud_init_work_disk_size` - The size for the disk for VMs with `control: false` (default: `12`).
- `cloud_init_work_memory` - The memory size for VMs with `control: false` (default: `8192`).
- `cloud_init_work_sockets` - The number of CPUs for VMs with `control: false` (default: `1`).
- `cloud_init_work_tags` - A list of Proxmox tags assigned to VMs with `control: false` (default: `worker,{{ cloud_init_k8s_tags }}`).
- `cloud_init_disk_storage` - The storage for holding the VMs cloud-init disks (default: `local`).
- `cloud_init_image` - Name of the Cloud-Init disk image file for all VMs (default: `ubuntu-22.04-server.qcow2`) [See note below].
- `cloud_init_image_storage` - The storage containing the Cloud-Init image (default: `local`) [See note below].
- `cloud_init_vmid` - The image directory containing the Cloud-Init image (default: `00`) [See note below].
- `cloud_init_k8s_tag` - A Proxmox tag added to all the VMs.
- `cloud_init_nameservers` - A list of name servers for the VMs (default: `[]`).
- `cloud_init_network_bridge` - The bridge network used for all VMs (default: `vmbr0`).
- `cloud_init_network_gw` - The default gateway used for all VMs.
- `cloud_init_network_gw6` - The default IPv6 gateway (default: unset).
- `cloud_init_pm_host` - The default Proxmox host on which to create all the VMs (defualt: `{{ cloud_init_pm_api_host }}`).
- `cloud_init_searchdomains` - A list of domains to search for DNS resolution (default: `[]`).
- `cloud_init_sshkeys` - A multi-line string containing a list of public SSH keys for the default user login on all VMs (No default. See `sshkeys` variable for the [community.general.proxmox_kvm](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html) module)
- `cloud_init_startup_pause` - The number of seconds to pause between starting all VMs (default: `10`).
- `cloud_init_storage` - The Datacenter storage for VM disks (default: `local-thin`).
- `cloud_init_vlan_tag` - A VLAN tag number for all VM networks that also gets prepended to VMIDs (default: `''`).

Special Variable Notes
----------------------

### `cloud_init_image`, `cloud_init_image_storage` and `cloud_init_vmid`

The specified image file must be located in the `cloud_init_image_storage` storage which must be configured to contain "Disk Images" content. Using the default `local` storage as an example, this is a directory type storage with the path `/var/lib/vz` and contains the sub-directory `/var/lib/vz/images` under which a sub-directory is created for each VM to hold disk images. Although the Cloud-Init disk image must be located in this directory structure, it is not in use by a VM and for this reason the fictitious VMID of `00` has been used to hold these disk image files. The full default path is `/var/lib/vz/images/00/` for the Cloud-Init image file location.

The cloud-init image must exist on every host where a VM will be created. Because of the fictitious VMID, these cannot be uploaded in the UI and must be placed there using something like `scp` or downloaded directly on each host system using `curl` or using many other methods of your choosing.

Hint: The Ansible Collection [cloudcodger.proxmox_openssh](https://galaxy.ansible.com/cloudcodger/proxmox_openssh) contains two roles that can be used to create the directory, set the content, and upload images to all the hosts in a cluster.

### `cloud_init_vms`

This variable must be a list of hashs that contain specific key/value pairs defining the VMs to create. Because many values must be unique among all containers and VMs within a Proxmox Datacenter, several of these are required and do not have default values.

Required keys:

- `name` - The name of the VM and becomes the hostname of the system.
- `control` - Sets if the VM is a control plane or worker node.
  For `true`, the `control_*` settings are used and
  for `false`, the `worker_*` settings are used.
  For non-kubernetes use, just set all VMs to the same value.
- `ip` - Static IPv4 address (in CIDR notation) or `dhcp`.
- `vmid` - A _unique_ VMID number within the Datacenter.

Optional keys:

- `genesis` - Tag with `cloud_init_genesis_tags`.
  Used for kubernetes clustering in other places.
  Intended to only be set to `true` on at most one VM and normally on a VM where `control: true`.
  For non-kubernetes use, just set or leave all VMs to `false` (the default).
- `ip6` - Static IPv6 address (in CIDR notation).
- `pm_host` - Overrides the global value for this machine.
- `storage` - Overrides the global value for this machine.

Dependencies
------------

This role depends on the [community.general.proxmox_kvm](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html) module.

Example Playbooks
-----------------

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
