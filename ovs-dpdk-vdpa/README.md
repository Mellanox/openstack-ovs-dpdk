# Openstack ovs-dpdk-vdpa  

## Apply patches  

After preparing the ovs-dpdk-sriov setup apply the following patches to nova_compute service/contianer on compute nodes:  

- nova.patch should be applied to  
    - /usr/lib/python3.6/site-packages/nova/network/os_vif_util.py  
    ```  
    $ cd /usr/lib/python3.6/site-packages/  
    $ patch -p1 < nova.patch  
    ```  

- os-vif.patch shoule be applied to the following files  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/ovs.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/constants.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/ovsdb/ovsdb_lib.py  
    ```  
    $ cd /usr/lib/python3.6/site-packages/  
    $ patch -p1 < os-vif.patch  
    ```  
And the following patch to neutron_api service/contianer on controller node:  

- neutron.patch should be applied to  
    - /usr/lib/python3.6/site-packages/neutron/plugins/ml2/drivers/openvswitch/mech_driver/mech_openvswitch.py  
    ```  
    $ cd /usr/lib/python3.6/site-packages/  
    $ patch -p1 < neutron.patch  
    ```  
After that restart the nova_compute and neutron_api services/contianers  

## Configure OVS  

Add this configuration to ovs on each compute node  
- <pci_adress>	PCI of uplink interface like 0000:03:00.0  
- <vfs_range>	Virtual Functions range like 0-7  
```  
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-extra="-w <pci_adress>,representor=[<vfs_range>],dv_flow_en=1,dv_esw_en=1,dv_xmeta_en=1"  
$ service openvswitch restart  
```  

## Configure vhostuser socket directory  

```  
$ mkdir -p /var/lib/vhost_sockets/  
$ chmod 775 /var/lib/vhost_sockets/  
$ chown qemu:hugetlbfs /var/lib/vhost_sockets/  
```
