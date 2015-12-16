.. Licensed to the Apache Software Foundation (ASF) under one
   or more contributor license agreements.  See the NOTICE file
   distributed with this work for additional information#
   regarding copyright ownership.  The ASF licenses this file
   to you under the Apache License, Version 2.0 (the
   "License"); you may not use this file except in compliance
   with the License.  You may obtain a copy of the License at
   http://www.apache.org/licenses/LICENSE-2.0
   Unless required by applicable law or agreed to in writing,
   software distributed under the License is distributed on an
   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
   KIND, either express or implied.  See the License for the
   specific language governing permissions and limitations
   under the License.


The Nuage-VSP Plugin
====================

Introduction to the Nuage-VSP Plugin
------------------------------------

The Nuage-VSP plugin is the Nuage Networks SDN
implementation in CloudStack, which integrates with Release 3.2 of the
Nuage Networks Virtualized Services Platform.
The plugin can be used by CloudStack to implement

* Isolated Guest Networks
* Virtual Private Clouds (VPC)
* Shared Networks

and leverage the scalability and feature-richness of Advanced SDN.

For more info about Nuage Networks, visit www.nuagenetworks.net.


Features of the Nuage-VSP Plugin
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following table lists the CloudStack network services provided by
the Nuage-VSP Plugin.

.. cssclass:: table-striped table-bordered table-hover

+----------------------+----------------------+
| Network Service      | CloudStack version   |
+======================+======================+
| Virtual Networking   | >= 4.5               |
+----------------------+----------------------+
| VPC                  | >= 4.5               |
+----------------------+----------------------+
| Source NAT           | >= 4.5               |
+----------------------+----------------------+
| Static NAT           | >= 4.5               |
+----------------------+----------------------+
| Firewall             | >= 4.5               |
+----------------------+----------------------+
| Network ACL          | >= 4.5               |
+----------------------+----------------------+
| User Data (*)        | >= 4.7               |
+----------------------+----------------------+

(*) Through the use of VR Provider

Table: Supported Services

.. note::
   The Virtual Networking service was originally called 'Connectivity'
   in CloudStack 4.0

The following hypervisors are supported by the Nuage-VSP Plugin.

.. cssclass:: table-striped table-bordered table-hover

+--------------+----------------------+
| Hypervisor   | CloudStack version   |
+==============+======================+
| XenServer    | >= 4.5               |
+--------------+----------------------+
| VmWare ESXi  | >= 4.5               |
+--------------+----------------------+
| KVM          | >= 4.7               |
+--------------+----------------------+

Table: Supported Hypervisors


Configuring the Nuage-VSP Plugin
--------------------------------

Prerequisites
~~~~~~~~~~~~~

Before building and using the Nuage plugin for ACS 4.7, verify that the platform you intend to use is supported.

.. Note:: Only the release notes for Nuage VSP contain the most up-to-date information on supported versions. Please check them to verify that the information below is actually correct.

Supported Versions
~~~~~~~~~~~~~~~~~~

* Nuage VSP 3.2.R2
* Apache CloudStack 4.7
* Citrix XenServer 6.2
* KVM on Enterprise Linux 7.x


Zone Configuration
~~~~~~~~~~~~~~~~~~

Select VSP Isolation Method During Zone Creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Nuage VSP solution is NOT supported in Basic zone provisioning mode. 

1. When adding a zone, the ACS administrator should select **Advanced** mode in the zone wizard. 
2. When laying out the physical network configuration during zone provisioning, the **Guest** network traffic should be put in a separate physical network of its own.
3. This physical network carrying the **Guest** traffic should have **VSP** as the **Isolation Method**, as shown in the figure captioned "Setting Isolation Method to VSP during Zone Creation" below.


Update Traffic Labels
~~~~~~~~~~~~~~~~~~~~~

**Guest Traffic Type**

Select **Edit** on the **Guest** traffic type panel and update the Traffic Label:

-  For XenServer use **nuageManagedNetwork** as the **XenServer Traffic Label**.
-  For KVM use **alubr0** as the **KVM Traffic Label**, as shown in the screenshot captioned "Specifying the Traffic Type in KVM."

Agent Configuration
~~~~~~~~~~~~~~~~~~~

[QA TO FURTHER EDIT THIS TEXT -- THIS TEXT IS JUST COPY FROM OVS]

.. note::
   Only for KVM hypervisor

-  Configure network interfaces:

   ::
      
      /etc/sysconfig/network-scripts/ifcfg-eth0
      DEVICE=eth0
      BOOTPROTO=none
      IPV6INIT=no
      NM_CONTROLLED=no
      ONBOOT=yes
      TYPE=OVSPort
      DEVICETYPE=ovs
      OVS_BRIDGE=cloudbr0
    
      /etc/sysconfig/network-scripts/ifcfg-eth1
      DEVICE=eth1
      BOOTPROTO=none
      IPV6INIT=no
      NM_CONTROLLED=no
      ONBOOT=yes
      TYPE=OVSPort
      DEVICETYPE=ovs
      OVS_BRIDGE=cloudbr1
    
      /etc/sysconfig/network-scripts/ifcfg-cloudbr0
      DEVICE=cloudbr0
      ONBOOT=yes
      DEVICETYPE=ovs
      TYPE=OVSBridge
      BOOTPROTO=static
      IPADDR=172.16.10.10
      GATEWAY=172.16.10.1
      NETMASK=255.255.255.0
      HOTPLUG=no
    
      /etc/sysconfig/network-scripts/ifcfg-cloudbr1
      DEVICE=cloudbr1
      ONBOOT=yes
      DEVICETYPE=ovs
      TYPE=OVSBridge
      BOOTPROTO=none
      HOTPLUG=no
    
      /etc/sysconfig/network
      NETWORKING=yes
      HOSTNAME=testkvm1
      GATEWAY=172.10.10.1

-  Edit /etc/cloudstack/agent/agent.properties

   ::
      
      network.bridge.type=openvswitch
      libvirt.vif.driver=com.cloud.hypervisor.kvm.resource.OvsVifDriver


Enabling the service provider
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

[QA TO FURTHER EDIT THIS TEXT -- THIS TEXT IS JUST COPY FROM OVS]

The OVS provider is disabled by default. Navigate to the "Network
Service Providers" configuration of the physical network with the GRE
isolation type. Navigate to the OVS provider and press the
"Enable Provider" button.

.. figure:: /_static/images/ovs-physical-network-gre-enable.png
   :align: center
   :alt: a screenshot of an enabled OVS provider


Network Offerings
~~~~~~~~~~~~~~~~~

[QA TO FURTHER EDIT THIS TEXT -- THIS TEXT IS JUST COPY FROM OVS]

Using the OVS plugin requires a network offering with Virtual
Networking enabled and configured to use the OVS element. Typical
use cases combine services from the Virtual Router appliance and the
OVS plugin.

.. cssclass:: table-striped table-bordered table-hover

+----------------------+-----------------+
| Service              | Provider        |
+======================+=================+
| VPN                  | VirtualRouter   |
+----------------------+-----------------+
| DHCP                 | VirtualRouter   |
+----------------------+-----------------+
| DNS                  | VirtualRouter   |
+----------------------+-----------------+
| Firewall             | VirtualRouter   |
+----------------------+-----------------+
| Load Balancer        | OVS             |
+----------------------+-----------------+
| User Data            | VirtualRouter   |
+----------------------+-----------------+
| Source NAT           | VirtualRouter   |
+----------------------+-----------------+
| Static NAT           | OVS             |
+----------------------+-----------------+
| Post Forwarding      | OVS             |
+----------------------+-----------------+
| Virtual Networking   | OVS             |
+----------------------+-----------------+

Table: Isolated network offering with regular services from the Virtual
Router.

.. figure:: /_static/images/ovs-network-offering.png
   :align: center
   :alt: a screenshot of a network offering.


.. note::
   The tag in the network offering should be set to the name of the
   physical network with the OVS provider.

Isolated network with network services. The virtual router is still
required to provide network services like dns and dhcp.

.. cssclass:: table-striped table-bordered table-hover

+----------------------+-----------------+
| Service              | Provider        |
+======================+=================+
| DHCP                 | VirtualRouter   |
+----------------------+-----------------+
| DNS                  | VirtualRouter   |
+----------------------+-----------------+
| User Data            | VirtualRouter   |
+----------------------+-----------------+
| Source NAT           | VirtualRouter   |
+----------------------+-----------------+
| Static NAT           | OVS             |
+----------------------+-----------------+
| Post Forwarding      | OVS             |
+----------------------+-----------------+
| Load Balancing       | OVS             |
+----------------------+-----------------+
| Virtual Networking   | OVS             |
+----------------------+-----------------+

Table: Isolated network offering with network services



Dedicated features that come with Nuage-VSP Plugin
--------------------------------------------------

Need to talk here about Domain Template Feature



Revision History
----------------

<add>
