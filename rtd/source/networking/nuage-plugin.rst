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

Required VSD Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

When configuring Nuage VSP as the network service provider, Nuage VSD must be added as a CSP user, and this user must be added to the CMS group. See `Enabling the service provider`_.

Zone Configuration
~~~~~~~~~~~~~~~~~~

Select VSP Isolation Method During Zone Creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Nuage VSP solution is NOT supported in Basic zone provisioning mode. 

1. When adding a zone, the ACS administrator should select **Advanced** mode in the zone wizard. 
2. When laying out the physical network configuration during zone provisioning, the **Guest** network traffic should be put in a separate physical network of its own.
3. This physical network carrying the **Guest** traffic should have **VSP** as the **Isolation Method**.


Update Traffic Labels
~~~~~~~~~~~~~~~~~~~~~

**Guest Traffic Type**

Select **Edit** on the **Guest** traffic type panel and update the Traffic Label:

-  For XenServer use **nuageManagedNetwork** as the **XenServer Traffic Label**.
-  For KVM use **alubr0** as the **KVM Traffic Label**, as shown in the screenshot captioned "Specifying the Traffic Type in KVM."

Enabling the service provider
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nuage VSP must be added as a Network Service provider to ACS before it can be used. 

:Step 1: Select **Infrastructure > Zone > [zone name] > Physical Network 2 > Configure Network Service Providers > Nuage Vsp > +**, which brings up the **Add Nuage Virtualized Services Directory (VSD)** panel. 

:Step 2: Enter the VSD **Host Name**, **Username** and **Password** that was previously created; see "Required VSD Configuration" in the Building the Nuage VSP CloudStack Plugin chapter of the current document. 

:Step 3: Specify the VSD API version by entering the API version in the appropriate field (format: ``v3_2``).

:Step 4: *EITHER* Add **Nuage VSD** and click the **OK** button,

         *OR* use API calls to configure Nuage VSP as the Network Provider; see "Nuage VSD API" in the Appendix of the current document.

:Step 5: Go to **Infrastructure > Zones > [zone name] > Physical Network 2 > Network Service Providers > Nuage Vsp > Devices > Details** tab as shown in the figure "Enabling Nuage VSP" below. This indicates the state of Nuage VSP. Enable Nuage VSP by clicking **Enable**.

:Step 6: (Optional) View the Nuage VSP status on the list of Network Service Providers on the **Infrastructure > Zones > [zone name] > Physical Network 2 > Network Service Providers** page;

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

Appendix
--------
Nuage VSD API
~~~~~~~~~~~~~

To add Nuage VSP as Network Service Provider, 

1.  Add the specified network service provider:

::

        http://135.227.147.106:8080/client/api?command=addNetworkSer
        viceProvider&name=NuageVsp&physicalnetworkid=87528ea5-0189-4
        a02-92db-3d1539232e21&response=json&sessionkey=CaDCr2P1qpIqm
        Fsr%2BmMl1T3nLzs%3D&_=1414200788068

2.  Add the specified Nuage VSD:

::

    http://135.227.147.106:8080/client/api?command=addNuageVspDe
    vice&physicalnetworkid=87528ea5-0189-4a02-92db-3d1539232e21&
    hostname=135.227.210.196&username=cloudstackuser1&password=c
    loudstackuser1&port=8443&apiversion=v3_2&retrycount=4&retryinter
    val=60&response=json&sessionkey=CaDCr2P1qpIqmFsr%2BmMl1T3nLz
    s%3D
