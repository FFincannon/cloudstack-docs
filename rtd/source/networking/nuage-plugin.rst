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

[QA TO FURTHER EDIT THIS TEXT -- THIS TEXT IS JUST COPY FROM OVS]

Before enabling the OVS plugin the hypervisor needs to be install OpenvSwitch. 
Default, XenServer has already installed OpenvSwitch. However, you must 
install OpenvSwitch manually on KVM. CentOS 6.4 and OpenvSwitch 1.10 are 
recommended.

KVM hypervisor:

-  CentOS 6.4 is recommended.

-  To make sure that the native bridge module will not interfere with 
   openvSwitch the bridge module should be added to the blacklist. See the 
   modprobe documentation for your distribution on where to find the blacklist. 
   Make sure the module is not loaded either by rebooting or executing rmmod 
   bridge before executing next steps.


Zone Configuration
~~~~~~~~~~~~~~~~~~

[QA TO FURTHER EDIT THIS TEXT -- THIS TEXT IS JUST COPY FROM OVS]

CloudStack needs to have at least one physical network with the isolation
method set to “GRE”. This network should be enabled for the Guest
traffic type.

.. note::
   With KVM, the traffic type should be configured with the traffic label
   that matches the name of the Integration Bridge on the hypervisor. For 
   example, you should set the traffic label as following:

   -  Management & Storage traffic: cloudbr0

   -  Guest & Public traffic: cloudbr1
      See KVM networking configuration guide for more detail.


.. figure:: /_static/images/ovs-physical-network-gre.png
   :align: center
   :alt: a screenshot of a physical network with the GRE isolation type


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
