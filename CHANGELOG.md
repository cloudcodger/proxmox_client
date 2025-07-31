# Change log

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
