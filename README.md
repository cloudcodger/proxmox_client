# Ansible Collection - cloudcodger.proxmox_client

A collection of roles for the creation of Proxmox LXC containers and Proxmox KVMs in a [Proxmox VE](https://pve.proxmox.com/wiki/Main_Page) datacenter.

# Roles

## `cloudcodger.proxmox_client.cloud_init`

Called on `localhost` to create a set of Proxmox KVM virtual machines using the [Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support) provided in Proxmox.
See the `README.md` file under the role directory for more detailed information.

## `cloudcodger.proxmox.lxc`

Called on `localhost` to create a set of Proxmox LXC Containers.
See the `README.md` file under the role directory for more detailed information.
