# Openstack ovs-dpdk-sriov  

## Apply patches  

After preparing the ovs-dpdk-sriov setup apply the following patches to nova_compute service/contianer:  

- nova.patch should be applied to the following files  
    - /usr/lib/python3.6/site-packages/nova/network/os_vif_util.py  
    - /usr/lib/python3.6/site-packages/nova/virt/libvirt/vif.py  
    ```  
    $ cd /usr/lib/python3.6/site-packages/  
    $ patch -p1 < nova.patch  
    ```  

- os-vif.patch shoule be applied to the following files  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/ovs.py  
    - /usr/lib/python3.6/site-packages/vif_plug_ovs/linux_net.py  
    ```  
    $ cd /usr/lib/python3.6/site-packages/  
    $ patch -p1 < os-vif.patch  
    ```  

After that restart the nova_compute service/contianer
