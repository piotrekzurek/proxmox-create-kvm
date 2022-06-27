# Change Log

## [v4.1.1](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v4.1.1)

* Installing the `python3-pip` package directly from apt instead of the ambiguous `python-pip`. This change allows the role to run correctly on all Proxmox versions supported (PVE >= 5), since `python3-pip` is available on all of them, but `python-pip` is not on the latest ones.

## [v4.1.0](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v4.1.0)

Added the ability to use a seamless flow to fully provision a virtual machine from a template that integrates cloud-init.

To set the cloud-init metadata, both the native parameters handled by Proxmox and a custom snippet can be used.

In addition, it's now possible to reallocate/update the hardware resources assigned to a cloned VM (with the exception of net, virtio, ide, sata and scsi due to the limitations of the proxmox_kmv module) and increase the size of its disks.

## [v4.0.1](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v4.0.1)

* Minor fixes in README

## [v4.0.0](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v4.0.0)

Updated to Ansible v2.10 (use of collections) and several improvements:
* Now there is a variable associated with each of the Proxmox modules parameters.
* Implemented authentication in the PVE node via tokens.
* Numerous variables renamed to be mnemonic.
* Unification of criteria for variable name prefixes.

## [v3.0.0](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v3.0.0)

* End of v1.0.0 variables' API backward's compatibility, no longer needed and not considered clean code

## [v2.0.0](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v2.0.0)

* New role's variables interface, with a uniform namespace pve_kvm_*
* Now possible to define the proxmox hostname as a variable, `pve_kvm_hostname`

## [v1.2.0](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v1.2.0)

* Added variable to parameterize proxmox_kvm module timeout

## [v1.1.0](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v1.1.0)

* Added the possibility to turn on the VM after create it

## [v1.0.1](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v1.0.1)

* Added `clone_target`, `clone_vmid` and `clone_newid` variables to fully support cloning VMs. See [#4](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/pull/4) [#6](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/pull/6)


## [v1.0.0](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/tree/v1.0.0)

* First stable version. VM creation from scratch with multiple storage units and network interfaces or cloning from an existing template