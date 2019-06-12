.. meta::
  :description: Firewall Network
  :keywords: AWS Transit Gateway, AWS TGW, TGW orchestrator, Aviatrix Transit network, Transit DMZ, Egress, Firewall


=========================================================
Example Config for FortiGate VM in AWS 
=========================================================

The goal of this document is to provide a step by step guide to launch and configure one or more Fortigate Next Generation Firewall instances to be integrated with Aviatrix Firewall Network. 
This setup will include basic “allow-all”  policies to serve as initial configuration to validate intended traffic is passing through the firewall instance. 
Fortinet’s documentation should be consulted for configuration of security policies and features.

Setup details
--------------
The instance will have to be launched in Firewall Network VPC and appropriate subnets based on the availibity zone. Each FW instance will have 3 interfaces that will have the following roles:

Eth0 → Management Interface
Etn1 → Transit Gateways traffic
Eth2 → Internet ingress/egress traffic

1. Setup Firewall Network (FireNet)
---------------------------------------
Complete steps 1-5 of Firewall Network Workflow in Aviatrix controller to prepare your Firewall VPC (FireNet VPC). This will also setup the subnets that you will need for launching your Fortigate instance. 

2. Deploy Fortigate Instance from AWS Marketplace
----------------------------------------------------

Launch Fortigate from AWS Marketplace, with appropriate subscription level. The type of subscription and/or version does not affect the fucntionaly from Aviatrix
perspective. However, it may affect steps outline from FG's UI.

Launch Fortinet FortiGate Next-Generation Firewall from AWS Marketplace with following details:

- Select “FireNet VPC” for the VPC
- Primary interface (Eth0): Assign this interfate to a public subnet of the FireNet VPC that is designated for management. If you created this subnet using Aviatrix VPC creator, the subnet is named in this format: 

    *<VPC Name>-Public-gateway-and-firewall-mgmt-<AvailabilityZone>*

- Disable public IP association during launch

|Network Interface and Subnet Information|

3. Configure Dataplane interfaces
------------------------------------------

- Using AWS console, create two additional Network Interfaces in AWS console with following subnet association (Naming is optional):


 - “WAN” Port (Eth1)

  Subnet: Designated Egress Public Subnet. This interface will be used for Egress/Ingress to and from the internet. Therefore, it is either going to be talking to a load balancer or internet gateway .If you created this subnet using Aviatrix VPC creator, the subnet is named in this format: 

    *<VPC Name>-Public-FW-ingress-egress-<AvailabilityZone>*

  Security Groups: Allow inbound to a supernet for your VPC and OnPrem CIDRs

 - “LAN” Port (Eth2)

  Subnet: This interface will be facing Aviatrix GW Eth2. This subnet is automatically created by Aviatrix Controller with following naming format: 

    *<Gateway Name>-dmz-firewall*

  Security Groups: Ensure we allow appropriate inbound if Ingress is to be used

  Note: If you need further clarification on FireNet subnets, please see this link:  `FireNet Subnets <https://www.lucidchart.com/publicSegments/view/f0bbe123-cbf7-4339-88df-a51eee2da631/image.pdf>`_ 


- Using AWS console assing an Elastic IP address to Management and WAN interfaces (Eth0 and Eth2) 



4. Login to Firewall and configure interfaces 
------------------------------------------------

- Using a web browser, connet to management IP of the instance with HTTPS. You should see a login prompt as seen below, continue to login.

|login|

::

  At the time of this writing, default login for Fortigate is admin as username and instance ID is the password

- To configure both Port1 and Port2, go to "Interfaces" tab:

  - Select an interface and clink on "Edit".  Enter following details:

    - Enter an Alias (i.e: LAN/WAN) for the interface
    - Enable DHCP to ensure FW retrieve private IP information from AWS console
    - Disable “Retrieve default gateway from server" 
    - Specify appropriate role (LAN/WAN)

|editInterface|


5. Create static routes for routing of traffic VPC to VPC 
------------------------------------------------------------
Go to Network -> State Routes to create A Static Route -> click on "Create New"

|createStaticRoute|

Packets to and from TGW VPCs, as well as on-premises, will be hairpinned off of the LAN interface. As such, we will need to configure appropate route ranges that you expect traffic for packets that need to be forward back to TGW. 
For simplicity, you can configure the FW to send all RFC 1918 packets to LAN port, which sends the packets back to TGW. 

In this example, we configure all traffic for 172.16.0.0/12 to be sent out of LAN interace.

In the Edit dialoge, you need to enter the following:

- Enter destiantion route in the "Destination" box
- In the "Gateway" box, you will need to enter IP address of Eth2 interface of Aviatrix gateway that this firewall will be attached to
- Interface will be LAN port
- Configure appropriate admin distance if you expect overlapping routes that need to be prioritized
- Enter comments as necessary

|editStaticRoute|

6. Configure basic traffic policy to allow traffic
----------------------------------------------------------------

In this step we will need to configure a basic traffic security policy that allows traffic to pass through the firewall. Give that aviatrix gateways will only forward internal traffic to LAN port of the Firewall, 
we can simply base our policy on packets that are being recieved on the LAN interface. 

In the Edit Policy dialoge, you need to enter the following:

- Go to Policy & Objects -> IPv4 Policy -> Edit
- Name: Configure any name for this policy
- Incoming Interface: LAN
- Outgoing Interface: LAN
- Source: Click on the + sign and add all
- Destination: Click on the + sign and add all
- Schedule: always
- Service: ALL
- Action: Accept

|editPolicy|

8. Ready to go!
---------------

Now your firewall instance is ready to receive packets! 

The next step is specify which Security Domain needs packet inspection by defining a connection policy that connects to
the firewall domain. This is done by `Step 8 <https://docs.aviatrix.com/HowTos/firewall_network_workflow.html#specify-security-domain-for-firewall-inspection>`_ in the Firewall Network workflow. 

For example, deploy Spoke-1 VPC in Security_Domain_1 and Spoke-2 VPC in Security_Domain_2. Build a connection policy between the two domains. Build a connection between Security_Domain_2 to Firewall Domain. 

Launch one instance in Spoke-1 VPC and Spoke-2 VPC. From one instance to ping the other instance. The ping should go through. . 

9. View Traffic Log
----------------------

You can view if traffic is forwarded to firewall instance by going to FortiView 

|showTraffic|


.. |login| image:: config_FortiGate_media/login.png
   :scale: 40%
.. |Interfaces.png| image:: config_FortiGate_media/Interfaces.png.png
   :scale: 40%
.. |editInterface| image:: config_FortiGate_media/editInterface.png
   :scale: 40%
.. |editPolicy| image:: config_FortiGate_media/editPolicy.png
   :scale: 40%
.. |createStaticRoute| image:: config_FortiGate_media/createStaticRoute.png
   :scale: 40%
.. |editStaticRoute| image:: config_FortiGate_media/editStaticRoute.png
   :scale: 40%
.. |editStaticRoute| image:: config_FortiGate_media/editStaticRoute.png
   :scale: 40%
.. |showTraffic| image:: config_FortiGate_media/showTraffic.png
   :scale: 40%
.. disqus::
