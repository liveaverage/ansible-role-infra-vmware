ansible-role-infra-vmware
=========

A role to bootstrap VMware infrastructure using Cobbler for OpenShift, k8s, or Docker workloads

Requirements
------------

No extra requirements.

Role Variables
--------------

The following vars should be configured for proper targetting of VMware and Cobbler environments:
```
    esxi_hostname: "{{ vsphere_host | default('esx.gxr.me') }}"
    esxi_datastore: "{{ vsphere_ds | default('localDS') }}"
    esxi_datacenter: "{{ vsphere_dc | default('ha-datacenter') }}"
    vmstate: "{{ vsphere_vmstate | default('poweredon')"
    cobbler_hostname: "{{ cobbler_srv_hostname | default ('cobbler.gxr.me') }}"
    cobbler_profile: "{{ cobbler_srv_profile | default ('centos7-min-x86_64') }}"
```

The following vars should be configured for proper host/VM configuration:
```
    disks:
      - datastore: localDS
        type: thin
        size_gb: '25'
      - datastore: localDS
        type: thin
        size_gb: '15'

    memory: 4096
```
Note that only the first disk in the array [of dictionaries] is provisioned as the root disk. Supplemental disks are added after PXE provisioning is complete.

Dependencies
------------

No extra dependencies, though you can use https://github.com/bertvv/ansible-role-cobbler to quickly configure a new Cobbler server.

Example Playbook
----------------

Sample Playbook:

    - name: Create ESXi Guest
      hosts: localhost
      connection: local
      vars:
        esxi_hostname: "{{ vsphere_host | default('esx.gxr.me') }}"
        esxi_datastore: "{{ vsphere_ds | default('localDS001') }}"
        esxi_datacenter: "{{ vsphere_dc | default('ha-datacenter') }}"
        vmstate: "poweredon"
        cobbler_hostname: "{{ cobbler_srv_hostname | default ('cobbler.gxr.me') }}"
        cobbler_profile: "{{ cobbler_srv_profile | default ('centos7-min-x86_64') }}"
      roles:
        - ansible-role-infra-vmware


Sample inventory:

```
[OSEv3:children]
masters
nodes
etcd
nfs

[OSEv3:vars]

cobbler_srv_profile="rhel-server-7.5-x86_64"
esxi_datastore=localDS001
disk_config=[{'size_gb': '25', 'type': 'thin', 'datastore': 'localDS001' }, {'size_gb': '15', 'type': 'thin', 'datastore': 'localDS001' }]
mem_config=4096
ocp_version='3.9'

[masters]
r1.gxr.me

[etcd]
r1.gxr.me

[nodes]
r1.gxr.me openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}" disks="{{ disk_config }}" memory="8096"
r2.gxr.me openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}" disks="{{ disk_config }}" memory="{{ mem_config }}"
r3.gxr.me openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}" disks="{{ disk_config }}" memory="{{ mem_config }}"

```

License
-------

BSD

Author Information
------------------

J.R. Morgan (https://github.com/liveaverage) - https://liveaverage.com
