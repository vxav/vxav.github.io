---
layout: post
title: Change vmnic order on vSphere host
DATE: 

---
Changing the vmnic order is an unusual thing to do and you may rightfully wonder why one would want to do that. Heterogeneous enironments are fully supported but most vSphere administrators aim to have homogeneous hosts with the exact same config in their clusters to simplify operations and avoid human errors.

Uplink are no exception. vSphere gives them labels (vmnic0, 1, 2...) at the first boot that you then use to configure the vSwitches . However, if you make changes to the harware configuration and reorder the vSwitches you may need to change the labels to make them match the other hosts.

VMware [KB2091560](https://kb.vmware.com/s/article/2091560) gives extensive information about the labelling of IO devices. Here is a quick summary of how to change the labels of the uplinks.

* First evacuate the host and put it in maintenance mode (We will need to reboot it.
* Connect to the host via SSH.
* List the uplinks and their labels.

    <code>localcli --plugin-dir /usr/lib/vmware/esxcli/int deviceInternal alias list | grep vmnic</code>

The output contains the list of vmnics with their addresses on the motherboard. If the NIC runs on a native driver (which is most of the time) there will be a logical ID as well. We need to change the aliases of both of these.

    Bus type  Bus address          Alias
    ------------------------------------
    pci       s00000002.00         vmnic4
    pci       s00000002.01         vmnic5
    pci       s00000001.01         vmnic1
    pci       s00000001.00         vmnic0
    logical   pci#s00000002.00#0   vmnic4
    logical   pci#s00000002.01#0   vmnic5
    logical   pci#s00000001.01#0   vmnic1
    logical   pci#s00000001.00#0   vmnic0

Let's say after a hardware change you have a gap between vmnic1 and vmnic4 and you are not happy with that. We want to replace vmnic4 with vmnic2 and vmnic5 with vmnic3.

* Change the PCI alias. You essentially assign an alias to a Bus address.

    <code>
    localcli --plugin-dir /usr/lib/vmware/esxcli/int/ deviceInternal alias store --bus-type pci --alias vmnic2 --bus-address s00000002.00
    </code>
    
    <code>
    localcli --plugin-dir /usr/lib/vmware/esxcli/int/ deviceInternal alias store --bus-type pci --alias vmnic3 --bus-address s00000002.01
    </code>

* Change the logical alias. Same as before with a different parameter.

    <code>
    localcli --plugin-dir /usr/lib/vmware/esxcli/int/ deviceInternal alias store --bus-type logical --alias vmnic2 --bus-address s00000002.00
    </code>
    
    <code>
    localcli --plugin-dir /usr/lib/vmware/esxcli/int/ deviceInternal alias store --bus-type logical --alias vmnic3 --bus-address s00000002.01
    </code>

* Reboot the host.