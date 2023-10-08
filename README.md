# OpenShift Container Platform deployment using Ansible
This repository contains an Ansible sample for deploying OpenShift on IBM Z/LinuxONE.
It was created from a system managed by the z/VM ESI tool.

## Using this sample
The playbook does not cover creation of support infrastructure such as DNS or load-balancer.
You need to have these requirements satisfied externally before you can run the playbook to build a cluster.

### Dependencies
As written, the z/VM SMAPI is used to perform guest operations including start and stop.
The z/VM ESI system also provides an `smcli` exec that potentially uses a different command syntax than other versions of `smcli` (such as the one in Feilong, for example).

Some support programs and services are made available in z/VM ESI as well.
* There is a "firstboot" process in ESI that performs configuration of the Linux ELAN system and services such as DNS.
The firstboot also adds additional information to the Ansible inventory, in a `group_vars/all/runtime.yaml` file.
This file is provided here as an empty sample, which will need to be populated (the comments in the file should help with this).
* Some `systemd` services are provided in z/VM ESI, including:
  * `ocp-approve-csrs`, which (as the name suggests) approves OCP CSRs
  * 
  The functons provided by these services can be reproduced in other ways (e.g. manual approving of CSRs when needed).

The playbook does not perform any z/VM unit-record operations to prepare the guests for boot.
In z/VM ESI, the `ZNETBOOT` tool (written by Rick Troth) is used to make the guests "self-booting".
If ZNETBOOT is not used, the playbook would either need to have appropriate changes to add the punching of the CoreOS files needed to boot the installations, or prompt for manual action to prepare and boot the guests.

The values in the `inventory/group_vars/all/default.yml` file need to be filled in for your intended cluster and system environment (z/VM details, etc).

### Run the playbook
Once dependencies are met, you should be able to run the playbook, passing the `cluster_name` variable on the command line:
```
ansible-playbook -i inventory -v -D -e "cluster_name=ocp-z-poc" create-cluster.yml
```

## Other capabilities
There are a couple of other playbooks present as well, which you can either try out yourself or use as models for creating your own.

### `cluster-guests-create.yml`
This reads the details from the inventory and uses the `smcli` program to invoke SMAPI to create the cluster guests.

### `cluster-guests-delete.yml`
This reads the details from the inventory and uses the `smcli` program to invoke SMAPI to delete the cluster guests.

### `regen-certificates.yml`
In its current form, this playbook uses Ansible OpenSSL modules to create certificates for the HTTPS interfaces in the z/VM ESI "ELAN" guest.
It also regenerates the z/VM System SSL and LDAP certificates.
* For the z/VM System SSL certificate, it copies a PKCS#12 bundle containing the certificte to the z/VM BFS.
* For the z/VM LDAP certificate, it uses an `expect` script to drive a `gskkyman` session to store the certificate in a GSKit database in the z/VM BFS.

### `update-dns-domain.yml`
This playbook updates the BIND DNS server configuration (it is driven as part of the `ipconf.sh` "firstboot" script on the z/VM ESI "ELAN" system).
