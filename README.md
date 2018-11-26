Ansible Role: libvirtd - WIP
======================

This is a role to install the packages necessary to manage KVM virtual machines
locally.  Additionally, based upon variables passed to the role, it can
configure the behavior of libvirtd.

I'm not quite sure how this will work with Ubuntu...I still have to set up my
testing environment.

Requirements
------------

* ...

Role Variables
--------------

The variables you can pass into this role are really specific to configuration
at this point.  The configuration of libvirt applies to the system as a whole
and not per-user.  However, in order for a non-root user to run commands, they
must be added to a `libvirt` system group.  The role blindly adds a user based
on `new_user` variable or whoever calls the role.

* `config_for_sat:` **DEFAULT = no** libvirt can be configured as a provider for
  a Red Hat Satellite but it requires some specific settings.  Setting `config
  _for_sat` to `yes` will ensure those settings are made. ***NOTE:*** *this is really
  probably more appropriate in a `satellite` role but I am still putting
  everything together as of this writing...*
* `libvirt_storage:` these settings allow the modification of storage pools:
  * `replace_default:` if this is set to `yes` then `pool_location` must be 
    specified and it will replace the location of the `default` storage pool [as
    an aside, the default location of `/var/lib/libvirt/images` has the
    potential to fill a `/` partition if not partitioned appropriately]
  * `pool_name:` this will be the name of the new storage pool if it is to
    reside alongside the `default` pool (this will be ignored if `replace_
    default` is `yes`)
  * `pool_location:` the directory where the storage pool specifies
* `servers:` a **list** of servers to connect to by default (the network will be
created based on `username`)
* `channels:` this is currently not used, but is included as a way to document
(remind yourself) which channels you should join

**NOTE** *this role will not create the underlaying structure for a custom
storage pool, but it will create a directory...meaning that if you intend to use
a logical volume for this, you must create it, create a filesystem, and mount it
to the new `libvirt_storage.pool_location` outside of this role*

Example Playbooks
-----------------

Minimal playbook:

```yaml
- hosts: servers
  roles:
    - role: hexchat
```

Playbook with user-specific settings:
```yaml
- hosts: servers
  roles:
    - role: libvirtd
      config_for_sat: yes
      libvirt_storage:
        replace_default: yes
        pool_name: Images
        pool_location: /depot/images/libvirt
      libvirt_network:
        remove_default: no
        name: labtop
        forward_mode: nat
        bridge_name: virbr5
        ip_address: 192.168.111.1
        netmask: 255.255.255.0
```

More Roles From dswhitley
-------------------------

You can find more roles from dswhitley on
[GitHub](https://github.com/dswhitley/ansible-roles).

Development & Testing
---------------------

This project uses [Molecule](http://molecule.readthedocs.io/) to aid in the
development and testing; the role is unit tested using
[Testinfra](http://testinfra.readthedocs.io/) and
[pytest](http://docs.pytest.org/).

To develop or test you'll need to have installed the following:

* Linux (e.g. [Ubuntu](http://www.ubuntu.com/))
* [Docker](https://www.docker.com/)
* [Python](https://www.python.org/) (including python-pip)
* [Ansible](https://www.ansible.com/)
* [Molecule](http://molecule.readthedocs.io/)

To run the role (i.e. the `tests/test.yml` playbook), and test the results
(`tests/test_role.py`), execute the following command from the project root
(i.e. the directory with `molecule.yml` in it):

```bash
molecule test
```

License
-------

MIT

Author Information
------------------

Daniel Whitley
