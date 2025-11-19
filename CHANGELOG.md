# Change log

# version 2.1.2

- Role `cloud_init` changes.
    - Fix to stop errors on loop labels.

- Role `lxc` changes.
    - Fix to stop errors on loop labels.

# version 2.1.1

- Renamed all `community.proxmox.proxmox_*` modules back to the `community.general` collection modules. This means the roles are not yet compatible with Ansible `version 12.0.0` unless you install the `community.general`, collection which is no longer there by default. The `community.proxmox.proxmox` role was found to always modify the `hwaddr`, even when not specified. This differs from the other one and causes any existing CT to be updated and the networking stops working.

- Role `cloud_init` changes.
    - Changed `cloud_init_pm_api_token_secret` setting in `vars/main.yml`.

- Role `lxc` changes.
    - Added `lxc_cpuunits` with a default of omitted.
    - Changed `lxc_cpus` to be omitted as the default.
    - Changed `lxc_pm_api_token_secret` setting in `vars/main.yml`.

# version 2.1.0

- Renamed all `community.general.proxmox_*` modules to the new `community.proxmox` collection modules.
- Role `cloud_init` changes.
    - Added the ability to configure `net1`, `net2` and `net3`.

# version 2.0.4

- Role `lxc` changes.
    - Changed `lxc_cts`, by adding `description` to the optional items. Fixed it to be like the one for `cloud_init`.

# version 2.0.3

- Both `cloud_init` and `lxc` roles changes.
    - Renamed current highest free variables to fix conflict between roles.

# version 2.0.2

- Both `cloud_init` and `lxc` roles changes.
    - Renamed handlers so playbooks that call both roles call the correct one.
      Since these are the same, it made no difference, but a future change
      expose the issue.
    - Added adding hostkeys to known_hosts file.
      When get IP from DHCP and needing to connect via it,
      leaves the possibility that you will have a conflicting entry
      that prevents the wait_for_connection from connecting.
      Which requires using `-vvv` to debug.
    - Added variable to bypass the `wait_for_connection`.

# version 2.0.1

- Both `cloud_init` and `lxc` roles, minor fix for `*_find_pm_host` calculation.
  Has an issue when calling both roles in the same play.

# version 2.0.0

- Role `cloud_init` changes.
    - Added `cloud_init_ansible_host*` variables. See `README.md` file.
    - Added `cloud_init_agent` to enable the QEMU Guest Agent.
    - Added `cloud_init_ansible_inventory_refresh` to have the role refresh the inventory.
    - Added `cloud_init_cpu` to set the CPU type.
    - Added `cloud_init_find_pm_host`. When set to `true`, `cloud_init_pm_host` is set to the PVE node with the least memory in use for each VM created.
    - Added `cloud_init_tags` for non-construct VMs.
    - Added task to `wait_for_connection` after start loop to wait until VMs are ready.
    - Remvoed `cloud_init_startup_pause`.
    - Removed the default value for `cloud_init_network_gw`.
    - Changed `cloud_init_construct_vmid_start`.
    - Changed `cloud_init_image` default to `ubuntu-24.04-server.qcow2`.
    - Changed `cloud_init_vms`, making the following items optional, `control`, `ip` and `vmid`.
    - Changed `cloud_init_vms`, adding `description` and `tags` to the optional items.
    - Changed VM creation to perform all steps for each item before continuing to the next.

- Role `lxc` changes.
    - Added `lxc_find_pm_host`. When set to `true`, `lxc_pm_host` is set to the PVE node with the least memory in use for each CT created.
    - Added a `wait_for_connection` after start loop to wait until CTs are ready.
    - Changed default value of `lxc_ansible_inventory_refresh` to `false`.
    - Changed CT creation to perform all steps for each item before continuing to the next.

# version 1.4.1

- Role `lxc` bug fix. Missing default for `pm_host`.

# version 1.4.0

- Role `lxc` changes.
    - Made setting `lxc_construct_cidr_start` and `lxc_construct_vmid_start` optional.
    - In `lxc_cts`, the `ip` and `vmid` are now optional.
    - Added the ability to create/update a `host_vars` file for newly created LXC containers and add/update the `ansible_host` variable to it.
    - Changed the `lxc_network_gw` default to `''`.

# version 1.3.1

- Removed `storage` from the `lxc` call to `community.general.proxmox` when creating a container.

# version 1.3.0

- Added `features` to the `lxc` (See the `README.md` for the role).
- Changed the default for `lxc_unprivileged` to `true`, to match the default in Proxmox.

# version 1.2.0

- Added `cloud_init_custom` to the `cloud_init` role for `cicustom` (See the `README.md` for the role).

# version 1.1.0

- Added optional `mounts` for LXC containers.

# version 1.0.0

- The initial release
