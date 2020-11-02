# openstack-ovs-dpdk  

## Description  

This repo has serveral patches that should be applied to the following openstack compenents to support ovs-dpdk sriov and ovs-dpdk vdpa:  
- Neutron  
    Adding the required changes in controller nodes (neutron_api service/container) to support creating virtio forwarder ports  
- Nova  
    Adding the required changes in Compute nodes (nova_compute service/container) to convert the nova.network.model to proper os_vif object  
- Os-vif  
    Adding the required changes in Compute nodes (nova_compute service/container) to append some extra devargs to ovs vdpa ports and to set vf-mac address for dpdk sriov ports  

## Prerequisites
Make sure you have installed all of the following prerequisites on the host machine:
  - openvswitch-2.13 and above
  - MLNX_OFED_LINUX-5.1-2.3.7.1 and above


## Supported Hardware
Containerized OVS Forwarder has been validated to work with the following Mellanox hardware:
- ConnectX-5 family adapters => min FW is 16.28.2006
- ConnectX-6Dx family adapters => min FW is 22.28.2006

## Enable UCTX
Make sure that you have UCTX_EN enabled in FW configuration
  ```
  $ mlxconfig -d mlx5_0 q UCTX_EN
  ```
if it's disabled run this command and reboot the server
  ```
  $ mlxconfig -d mlx5_0 s UCTX_EN=1
  ```

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
    - /usr/lib/python3.6/site-packages/os_vif/utils.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/constants.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/linux_net.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/ovs.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/ovsdb/ovsdb_lib.py  
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
## Creating an instance:  

### Create network  
```  
$ private_network_id=`openstack network create private --provider-network-type vxlan --share | grep ' id ' | awk '{print $4}'`  
```  
### Create subnet  
```  
$ openstack subnet create private_subnet --dhcp --network private --subnet-range 11.11.11.0/24  
```  
### Create flavors  
```  
$ openstack flavor create --ram 1024 --vcpus 1 --disk 20 --property dpdk=true --property hw:cpu_policy=dedicated --property hw:mem_page_size=1GB --property hw:emulator_threads_policy=isolate  --public dpdk.1g  
```  
Note:  
Please make sure that you have configured hugepages in the host before you create the instance  
You can do it with the following steps:  
  - Edit the file /etc/default/grub and add "intel_iommu=on iommu=pt default_hugepagesz=1G hugepagesz=1G hugepages=8 hugepagesz=2M hugepages=1024" to the existing GRUB_CMDLINE_LINUX line  
  - Run command ``` $ grub2-mkconfig -o /boot/grub2/grub.cfg```  
  - Reboot the host  

### Create sriov port  
```  
$ direct_port=`openstack port create direct --vnic-type=direct --network private --binding-profile '{"capabilities":["switchdev"]}' | grep ' id ' | awk '{print $4}'`  
```  

### Create sriov instance  
```  
$ openstack server create --flavor dpdk.1g --image mellanox --nic port-id=$direct_port --availability-zone nova:overcloud-computesriov-0.localdomain vm_sriov  
```  

### Create vdpa port  
```  
$ virtio_port=`openstack port create virtio_port --vnic-type=virtio-forwarder --network $private_network_id | grep ' id ' | awk '{print $4}'`  
```  

### Create vdpa instance  
```  
$ openstack server create --flavor dpdk.1g --image mellanox --nic port-id=$virtio_port --availability-zone nova:overcloud-computesriov-0.localdomain vm_vdpa  
```  
