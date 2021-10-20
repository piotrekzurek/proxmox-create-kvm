Ansible role proxmox_create_kvm
=========

[![Galaxy](https://img.shields.io/badge/galaxy-UdelaRInterior.proxmox__create__kvm-blue.svg)](https://galaxy.ansible.com/udelarinterior/proxmox_create_kvm) ![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/udelarinterior/ansible-role-proxmox-create-kvm?label=release&logo=github&style=social) ![GitHub stars](https://img.shields.io/github/stars/udelarinterior/ansible-role-proxmox-create-kvm?style=social) ![GitHub forks](https://img.shields.io/github/forks/udelarinterior/ansible-role-proxmox-create-kvm?style=social)

A complete role to easily and automagically create/clone KVM virtual machines in a Proxmox Virtual Environment (PVE) cluster, with suppot for [cloud-init](https://pve.proxmox.com/wiki/Cloud-Init_Support) ([see example](#example-playbook)) and multiple network interfaces and storage units. The role wraps the [community.general.proxmox_kvm module](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html#parameter-description)

Requirements
------------

You must act on a Proxmox VE node or cluster already configured, i.e. you need PVE node already installed (tested with PVE 5 and 6), and a Proxmox user with VM creation rights.


Installation
------------

```shell
$ ansible-galaxy install udelarinterior.proxmox_create_kvm
```

To be able to update later and eventually to modify it, prefer using `requirements.yml` with the git source:

```yaml
- name: udelarinterior.proxmox_create_kvm
  src: https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm.git
  ```
And then download it with `ansible-galaxy`:

```shell
$ ansible-galaxy install -r requirements.yml
```

Using `git`, you'll have to be carefull to folder name :

```shell
$ git clone https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm.git udelarinterior.proxmox_create_kvm
```

Role Variables
--------------

The `defaults` variables define the VM parameters. To be specified by host under `host_vars/host_fqdn/vars` and eventually encrypted in `host_vars/host_fqdn/vault`

New interface introduced in v2.0.0 is maintained, with role's variables defined in the `pve_*` namespace when they are shared between [several Proxmox roles](https://github.com/UdelaRInterior?q=proxmox), and in the `pve_kvm_*` namespace when they are specific to th present one. The role is no longer backward's compatible with [v1.X.Y previous interface](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/blob/v1.2.0/README.md#role-variables).

```yaml
#########################################################
### Proxmox API connection and authentication section ###
#########################################################

# Flag to determine the behavior of the variables default values
  # Various module options used to have default values. This cause problems when user expects different behavior from proxmox by default or fill
  # options which cause problems when they have been set. The default value is "compatibility", which will ensure that the default values are used
  # when the values are not explicitly specified by the user. From community.general 4.0.0 on, the default value will switch to "no_defaults".
  # To avoid deprecation warnings, please set proxmox_default_behavior to an explicit value. This affects the acpi, autostart, balloon, boot,
  # cores, cpu, cpuunits, force, format, kvm, memory, onboot, ostype, sockets, tablet, template, vga, options.
  # See: https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html#parameter-proxmox_default_behavior
  # Choices: compatibility - no_defaults
pve_default_behavior: compatibility

# Proxmox node hostname where we create or manage a KVM
pve_node: mynode

# FQDN or IP of the Proxmox API endpoint where we manage the cluster or node
pve_api_host: mynode.mycluster.org

# User to use to connect to the Proxmox cluster API
pve_api_user: automations@pam    # Optional if token based authentication is used
# Password for the previous API user (BETTER PUT THIS IN A VAULT, this dummy example can cause security issues)
pve_api_password: PaSsWd_f0r-AuToMaTi0nS    # Optional if token authentication is used

# pve_api_token_id: automations@pam!ansible                       # Optional if user-password based authentication is used
# pve_api_token_secret: 0b029a23-1ca3-a93f-8d90-5a4c9d064199      # Optional if user-password based authentication is used

# Validate the node's certificates when creating the virtual machine
# pve_validate_certs: no    # Optional.

# Timeout (in seconds) for operations, both clone and create. (Cloning can take a while)
# pve_kvm_timeout: 500


#####################################################
### VM resources and behaviour definition section ###
#####################################################

# By default, we suppose that `inventory_hostname` is the FQDN or the hostname of the VM to create,
# so we set the variable to the hostname. You can arbitrarly define this hostname
pve_hostname: "{{ inventory_hostname.split('.')[0] }}"

# It will ensure that the VM is turned on when provisioning finishes
pve_kvm_started_after_provision: false

# FALSE: create a new VM  -  TRUE: clone an existing VM
pve_kvm_clone_from_existing: false

# With this variable set to 'true', as soon as the cloning of the VM is finished, the resources assigned to the new one will be reconfigured
# (when pve_kvm_update is set to yes or undefined) and the size of its hard drives can be increased. It will be possible to reconfigure the
# same resources (and with the same variables) managed for the creation of a VM from scratch, with the exception of: net, virtio, ide, sata,
# scsi due to limitations of the proxmox_kvm module. This variable will be very useful when cloning templates with support for cloud-init,
# being able to set unique values for each instance, for example IP addresses and passwords.
pve_kvm_clone_post_configure: false

# If yes, the VM will be updated with new value. Cause of the operations of the API and security reasons, the update of the
# following parameters was disabled: net, virtio, ide, sata, scsi. For example updating net update the MAC address and virtio
# create always new disk. Update of pool is disabled. It needs an additional API endpoint not covered by this module.
# Leave undefined or set to 'yes' if you want to reconfigure the resources of a newly cloned VM (pve_kvm_clone_post_configure: true)
pve_kvm_update: no

# With these variables you can make the playbook/role execution pause waiting for SSH connection to the newly
# created VM until it finish booting and provisioning itself (if using cloud-init) to then continue in a continuous flow
pve_kvm_wait_for_ssh_connection: false
pve_kvm_wait_for_ssh_connection_remote_user: "{{ pve_kvm_ciuser | default('root') }}"
pve_kvm_wait_for_ssh_connection_timeout: 90
# pve_kvm_update: no


#####################################################
#### To create a KVM by cloning a pre-existing one
###

## IMPORTANT: If the VM is cloned, the other parameters are not used,
## the assigned hardware characteristics are inherited from the cloned VM

# Create a full copy of all disk. This is always done when you clone a normal VM. For VM templates, we try to create a linked clone by default.
# FALSE: linked clone  -  TRUE: full clone ## See https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/issues/2
pve_kvm_clone_full: true  # true as default value to avoid mentioned error in case of omission

# ID of VM to be cloned
# pve_kvm_clone_vmid: 100

# VMID for the clone. If newid is not set, the next available VMID will be fetched from ProxmoxAPI.
# pve_kvm_clone_newid: 400

# Name of VM to be cloned. If vmid is set with the pve_kvm_clone_vmid variable hereafter, which takes precedence over
# the hostname, pve_kvm_clone_vm can take arbitrary value but it is required for initiating the clone.
# pve_kvm_clone_vm: DebianBusterTemplate

# The name of the snapshot to clone (Don't use/define it at the same time as pve_kvm_clone_vm)
# pve_kvm_clone_snapname: DebianBusterConfigured

# Target storage for full clone.
# pve_kvm_clone_storage: local-lvm

# Target drive's backing file's data format. Use format=unspecified and full=false for a linked clone.
# Choices: cloop - cow - qcow - qcow2 (default) - qed - raw - vmdk
# pve_kvm_clone_format: raw

# Target node. Only allowed if the original VM is on shared storage.
# pve_kvm_clone_target: "{{ pve_node }}"


#####################################################
#### To create a KVM from scratch (Or for reconfigure a clone)
###

###
#### Most important/used vars
###

# Specifies whether a VM will be started during system bootup.
# pve_onboot: no    # Optional

# Specify the boot order: a = floppy, c = hard disk, d = CD-ROM, n = network
# pve_kvm_boot: cnd  # Optional

# Identifier of the hard drive to boot from
# pve_kvm_bootdisk: scsi0  # Optional

# Specify the description for the VM. Only used on the configuration web interface. This is saved as comment inside the configuration file.
# pve_kvm_description: LAMP Server created with Ansible    # Optional

# Enable/disable KVM hardware virtualization.
# pve_kvm_hardware_virtualization: yes  # Optional

# Specifies guest operating system. This is used to enable special optimization/features for specific operating systems.
# pve_kvm_ostype: l26   # Optional. Choices: l24 - l26 (default) - other - wxp - w2k - w2k3 - w2k8 - wvista - win7 - win8 - solaris

# Sets the number of CPU sockets. (1 - N).
# pve_kvm_sockets: 1    # Optional

# Specify number of cores per socket.
# pve_kvm_cores: 1    # Optional

# Specify CPU weight for a VM. You can disable fair-scheduler configuration by setting this to 0
# pve_kvm_cpuunits: 1024    # Optional

# Specify if CPU usage will be limited. Value 0 indicates no CPU limit. If the computer has 2 CPUs, it has total of '2' CPU time
# pve_kvm_cpulimit: 0    # Optional

# Memory size in MB for instance.
# pve_kvm_memory: 512    # Optional

# Specify the amount of RAM for the VM in MB. Using zero disables the balloon driver.
# pve_kvm_balloon: 0    # Optional

# Amount of memory shares for auto-ballooning. (0 - 50000). The larger the number is, the more memory this VM gets.
# Using 0 disables auto-ballooning, this means no limit. The number is relative to weights of all other running VMs.
# pve_kvm_shares: 0    # Optional

# Select VGA type. If you want to use high resolution modes (>= 1280x1024x16) then you should use option 'std' or 'vmware'.
# pve_kvm_vga: std      # Optioanl. Choices: std (default) - cirrus - vmware - qxl - serial0 - serial1 - serial2 - serial3 - qxl2 - qxl3 - qxl4

# Custom structure to define the VM network interfaces in a more human-readable way (these are examples, default is empty)
pve_kvm_net_interfaces:
- id: net0
  model: virtio                 # Optional. Available models: virtio - e1000 - rtl8139 - vmxnet3
  hwaddr: 46:70:B7:26:87:4F     # Optional. If not indicated, Proxmox will assign one automatically.
  bridge: vmbr0
  firewall: true                # Optional
  disconnect: true              # Optional
  multiqueue: 2                 # Optional
  rate_limit: 250               # Optional (In MB/s)
  vlan_tag: 400                 # Optional

- id: net1
  bridge: vmbr0

# Custom structure to define the VM hard disks in a more human-readable way (these are examples, default is empty)
pve_kvm_hard_disks:
- id: 0                         #  0 ≤ n ≤ 15
  bus: virtio                   # Optional. Available buses: virtio (default) - ide - sata - scsi
  storage: local-lvm
  size: 32
  format: raw                   # Optional. Available formats: qcow2 - raw (default) - subvol
  backup: true                  # Optional
  skip_replication: false       # Optional
  cache: none                   # Optional. Available types: none (default) - directsync - writethrough - writeback - unsafe
  io_thread: false              # Optional
  ssd_emulation: false          # Optional

- id: 1
  bus: sata
  storage: local-lvm
  size: 16
  ssd_emulation: true
  backup: false

# A hash/dictionary of volume used as IDE hard disk or CD-ROM
# pve_kvm_ide:    # Optional
#   ide2: 'local:cloudinit,format=qcow2'    # Useful example to add cloud-init Drive


###
#### cloud-init related vars
###

# Specifies the cloud-init configuration format. The default depends on the configured
# operating system type (ostype): nocloud format for Linux, and configdrive2 for Windows
# pve_kvm_citype: nocloud

# Specify custom files to replace the automatically generated ones at start
pve_kvm_cicustom: "user=local:snippets/{{ pve_hostname }}.yaml"

# Username of default user to create.
pve_kvm_ciuser: default

# Password of default user to create.
pve_kvm_cipassword: super-secret

# SSH key to assign to the default user. NOT TESTED with multiple keys but a multi-line value should work
pve_kvm_sshkeys: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}" # Ansible controller default public key

# Set the IP configuration. A hash/dictionary of network ip configurations. ipconfig='{"key":"value", "key":"value"}'
pve_kvm_ipconfig:
  ipconfig0: 'ip=192.168.1.1/24,gw=192.168.1.1'

# DNS server IP address(es). If unset, PVE host settings are used.
pve_kvm_nameservers:
  - '1.1.1.1'
  - '8.8.8.8'

# Sets DNS search domain(s). If unset, PVE host settings are used.
pve_kvm_searchdomains: mycluster.org

# With these variables, a completely customized cloud-init configuration snippet can be uploaded to the proxmox node and then assigned
# to a VM and thus extend the native ones. https://pve.proxmox.com/wiki/Cloud-Init_Support#_custom_cloud_init_configuration
pve_kvm_custom_ci_snippet: |
  hostname: myserver
  manage_etc_hosts: true
  fqdn: myserver.mydomain.com
  user: deploy
  password: $5$WxmZyYwx$LjRDOypMJsF7Ee3y0kK6sboa4mxkTSS1.q.b8pIhWm7
  ssh_authorized_keys:
    - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG0sjq7JWNndpb642fVNXObOHYx0NIASEHW9LMz8IR+v name@host
  chpasswd:
    expire: False
  users:
    - default
  package_upgrade: false
  network:
    version: 1
    config:
        - type: physical
          name: eth0
          mac_address: '42:0c:2a:10:75:e2'
          subnets:
          - type: static
            address: '192.168.0.200'
            netmask: '255.255.255.0'
            gateway: '192.168.0.1'

pve_kvm_custom_ci_snippet_template_src: cloud-init_snippet.yaml.j2
pve_kvm_custom_ci_snippet_dest: "/var/lib/vz/snippets/{{ pve_hostname }}.yaml"

###
#### Other vars
###

# See the module documentation: https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html#parameters
# And the create_kvm.yml tasks: https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/blob/master/tasks/create_kvm.yml


#####################################################
#### To create a KVM by cloning a pre-existing one
###

## IMPORTANT: If the VM is cloned, except pve_kvm_clone_post_configure==true and pve_kvm_update==yes or undefined,
## the other parameters are not used and the assigned hardware characteristics are inherited from the cloned VM

# Create a full copy of all disk. This is always done when you clone a normal VM. For VM templates, we try to create a linked clone by default.
# FALSE: linked clone  -  TRUE: full clone ## See https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/issues/2
pve_kvm_clone_full: true  # true as default value to avoid mentioned error in case of omission

# ID of VM to be cloned
# pve_kvm_clone_vmid: 100

# VMID for the clone. If newid is not set, the next available VMID will be fetched from ProxmoxAPI.
# pve_kvm_clone_newid: 400

# Name of VM to be cloned. If vmid is set with the pve_kvm_clone_vmid variable hereafter, which takes precedence over
# the hostname, pve_kvm_clone_vm can take arbitrary value but it is required for initiating the clone.
pve_kvm_clone_vm: DebianBusterTemplate

# The name of the snapshot to clone (Don't use/define it at the same time as pve_kvm_clone_vm)
# pve_kvm_clone_snapname: DebianBusterConfigured

# Target storage for full clone.
pve_kvm_clone_storage: local-lvm

# Target drive's backing file's data format. Use format=unspecified and full=false for a linked clone.
# Choices: cloop - cow - qcow - qcow2 (default) - qed - raw - vmdk
pve_kvm_clone_format: raw

# Target node. Only allowed if the original VM is on shared storage.
pve_kvm_clone_target: "{{ pve_node }}"

# This list indicates for each identifier of a VM storage unit, the amount of bytes to increase its size.
# To use in combination with pve_kvm_clone_post_configure: true
pve_kvm_clone_grow_disks:
  - disk: scsi0
    increment: 30G
```

Dependencies
------------

We need Ansible version > 2.10 and `community.general` collection to have the appropriate API of Proxmox modules.

Proxmox VE 5 or higher installed in a server node or a cluster of several nodes.

Example Playbook
----------------

Suppose we have a Proxmox VE cluster where `mynode.mycluster.org` is the FQDN of one of the cluster nodes (with the HTTPS API accessible on port 8006) and `deploy` a user in that node with VM creation rights. Then we can define the Ansible inventory group `pve_virtuals_group` and define each of the VMs within it. For this group we will declare the following global variables (`group_vars/pve_virtuals_group/vars/`):
```yaml
pve_node: mynode
pve_api_host: mynode.mycluster.org
pve_api_user: deploy@pam
pve_api_password: "{{ secret_node_password_from_vault }}"
```

Then, for each VM to provision we will define its specific variables at the host level (`host_vars/<machine-name>.mycluster.org/vars/`), similar to how it is done in the following example:
```yaml
# This is a practical, useful and simple example of variable declaration to
# fully provision a VM from an official cloud image with support for
# cloud-init (generally offered by the same distro's maintainers)

# Name of the VM to create
pve_hostname: <machine-name>
# VM description displayable in Proxmox web GUI notes
pve_kvm_description: Cloud image provisioned using Ansible + cloud-init

# The VM will be a full clone of the DebianBusterCloudImage template
# and will be reconfigured after cloning to assign the hardware
# resources and the corresponding cloud-init parameters
pve_kvm_clone_from_existing: true
pve_kvm_clone_vm: DebianBusterCloudImage
pve_kvm_clone_full: true
pve_kvm_clone_post_configure: true

# The size of the disks will be increased as indicated. (Then
# cloud-init will resize partitions and filesystems automatically)
pve_kvm_clone_grow_disks:
  - disk: scsi0
    increment: 30G

# The VM will have 2 CPU cores, 2 GB of RAM and it will
# automatically power on when the node is powered on or rebooted
pve_kvm_cores: 2
pve_kvm_memory: 2048
pve_onboot: yes

# Name of the user to provision and use by cloud-init (can use sudo without password)
pve_kvm_ciuser: deploy
# Password to previously indicated user
pve_kvm_cipassword: "{{ secret_machine_password_from_vault }}"
# SSH authorized key to previously indicated user
pve_kvm_sshkeys: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
# Search domains
pve_kvm_searchdomains: mycluster.org
# Network interfaces and their IP addresses configuration
pve_kvm_ipconfig:
  ipconfig0: 'ip=172.16.0.101/24,gw=172.16.0.1'
# Name resolvers to be used by the virtualized system
pve_kvm_nameservers:
  - '1.1.1.1'
  - '1.0.0.1'

# Power on the VM after provisioning is complete
pve_kvm_started_after_provision: true
# Pause the execution of the role while the new VM is not available to
# access via SSH (until it completes the boot and its auto-provisioning)
pve_kvm_wait_for_ssh_connection: true
# User with which to start the SSH session
pve_kvm_wait_for_ssh_connection_remote_user: "{{ pve_kvm_ciuser }}"
```

Given that:
* virtual machines are named `<machine-name>.mycluster.org`
* name resolutions of Proxmox VE machines and node exist and are configured correctly
* all IPs indicated in vars are allocated and routed

run the following playbook will create all the virtual machines declared in the `pve_virtuals_group`:

```yaml
    - name: create virtual machines declared in pve_virtuals_group
      hosts: pve_virtuals_group
      remote_user: deploy
      become: yes
      gather_facts: no

      roles:
        - udelarinterior.proxmox_create_kvm
```

License
-------

(c) Universidad de la República (UdelaR), Red de Unidades Informáticas de la UdelaR en el Interior.

Licenced under GPL-v3

Author Information
------------------

[@ulvida](https://github.com/ulvida)
[@santiagomr](https://github.com/santiagomr)
[@UdelaRInterior](https://github.com/UdelaRInterior)
https://proyectos.interior.edu.uy/
