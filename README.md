# Ansible Role: libvirtd

This is an Ansible role to install the packages necessary to manage KVM virtual
machines locally.  Additionally, based upon variables passed to the role, it can
configure the behavior of libvirtd.

I'm not quite sure how this will work with Ubuntu...I still have to set up my
testing environment.

## Overview

At a very high level, this role will:

* install packages which are helpful for managing virtual machines on a local
  system
* [*optional*] if settings are provided, they will modify the default behavior
  accordingly

I am struggling to test this role, so currently the focus is on Fedora and
additional testing will occur for other OSes as time allows (read into that what
you will).

## Important Notes

* the real driver for this role was the fact that the default location for
  storing VMs was not ideal for me.  I prefer to put them where I have more
  storage available so this role was born
* the role allows you to specify a different location for the default pool, but
  does not create the actual partition/space

## Requirements

Any package or additional repository requirements will be addressed in the role.

## Role Variables

All of these variables should be considered **optional** however, be aware that
sanity checking is minimal (if at all).  The configuration of libvirt applies to
the system as a whole and not per-user.

* `libvirt_users` this is a ***list*** of users that will be added to a
  "libvirt" group which can use libvirt without providing root password.  In
  addition to this, these users will have
  `export LIBVIRT_DEFAULT_URI="qemu:///system"` added to their `.bashrc` which
  will change their default URI from `qemu:///session`  ***NOTE:*** *this makes
  sense in my use case, but make sure it does in yours before blindly setting
  this.
  [Here is some good reading on the topic.](https://blog.wikichoon.com/2016/01/qemusystem-vs-qemusession.html)
* `config_for_sat` **DEFAULT = false** libvirt can be configured as a provider
  for a Red Hat Satellite but it requires some specific settings.  Setting
  `config_for_sat` to `yes` will ensure those settings are made. ***NOTE:***
  *this is really probably more appropriate in a `satellite` role but I am still
  putting everything together as of this writing...*
* `libvirt_storage` these settings allow the modification of storage pools:
  * `replace_default` if this is set to `true` then `pool_location` **must** be
    specified and it will replace the location of the `default` storage pool [as
    an aside, the default location of `/var/lib/libvirt/images` has the
    potential to fill a `/` partition if not partitioned appropriately]
  * `pool_name` this will be the name of the new storage pool if it is to
    reside alongside the `default` pool (this will be ignored if `replace_
    default` is `true`)
  * `pool_location` the directory where the storage pool specifies
* `libvirt_network` these settings allow the addition or modification of
  a network
  * `remove_default` if this is set to `true` then the default network will be
    removed.  I cannot recall why you would want to do this
  * `name` this will be the name of the new network
  * `forward_mode` **DEFAULT = nat** if you prefer a mode other than `nat` you
    can set it here
  * `bridge_name` this will largely be dependent on your configuration
  * `ip_address` this is the IP of the bridge for this network.  Used with the
    `netmask` to identify the range of IPs that can communicate on this network
  * `netmask` **DEFAULT = 255.255.255.0** if you prefer a different subnet mask
    you can set it here

**NOTE** *this role will not create the underlaying structure for a custom
storage pool, but it will create a directory...meaning that if you intend to use
a logical volume for this, you must create it, create a filesystem, and mount it
to the new `libvirt_storage.pool_location` outside of this role*

## Example Playbook

Minimal playbook:

```yaml
- hosts: servers
  roles:
    - role: libvirtd
```

Playbook with user-specific settings:
```yaml
- hosts: servers
  roles:
    - role: libvirtd
      libvirt_users:
        - "{{ new_user | default( ansible_user_id ) }}"
      config_for_sat: false
      libvirt_storage:
        replace_default: true
        pool_name: Images
        pool_location: /depot/images/libvirt
      libvirt_network:
        remove_default: false
        name: labtop
        forward_mode: nat
        bridge_name: virbr5
        ip_address: 192.168.111.1
        netmask: 255.255.255.0
```

## Inclusion

I envision this role being included in a larger project through the use of a
`requirements.yml` file.  So here is an example of what you would need in your
file:

```yaml
# get the libvirtd role from github
- src: https://github.com/thisdwhitley/ansible-role-libvirtd.git
  scm: git
  name: libvirtd
```

Have the above in a `requirements.yml` file for your project would then allow
you to "install" it (prior to use in some sort of setup script?) with:

```bash
ansible-galaxy install -p ./roles -r requirements.yml
```

## Testing

The purpose of this project does not lend itself to easy testing.  It is kind
of crappy, sorry about that.

## To-do

* figure out a clean and easy way to test
* test with different OSes
* determine if user modification is necessary
* shift to using 
  [Cockpit](https://cockpit-project.org/guide/latest/feature-virtualmachines)
  instead of virt-manager

## References

* [qemu:///system vs qemu:///session](https://blog.wikichoon.com/2016/01/qemusystem-vs-qemusession.html)

## License

All parts of this project are made available under the terms of the [MIT
License](LICENSE).
