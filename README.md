# openstack-ovs-dpdk  

## Description  

This repo has serveral patches that should be applied to the following openstack compenents to support ovs-dpdk sriov and ovs-dpdk vdpa:  
- Neutron  
    Adding the required changes in controller nodes (neutron_api service/container) to support creating virtio forwarder ports  
- Nova  
    Adding the required changes in Compute nodes (nova_compute service/container) to convert the nova.network.model to proper os_vif object  
- Os-vif  
    Adding the required changes in Compute nodes (nova_compute service/container) to append some extra devargs to ovs vdpa ports and to set vf-mac address for dpdk sriov ports  

## Apply patches  

After preparing the ovs-dpdk-sriov setup apply the following patches as described above:  

- nova.patch should be applied to  
    - /usr/lib/python3.6/site-packages/nova/network/os_vif_util.py  
    - /usr/lib/python3.6/site-packages/nova/virt/libvirt/vif.py
    ```  
    $ cd /usr/lib/python3.6/site-packages/  
    $ patch -p1 < nova.patch  
    ```  

- os-vif.patch shoule be applied to the following files  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/constants.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/linux_net.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/ovs.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/ovsdb/ovsdb_lib.py  
    - 
    ```  
    $ cd /usr/lib/python3.6/site-packages/  
    $ patch -p1 < os-vif.patch  
    ```  

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
