.. meta::
  :description: Firewall Network
  :keywords: AWS Transit Gateway, AWS TGW, TGW orchestrator, Aviatrix Transit network, Transit DMZ, Egress, Firewall

**Some notes before deployment:**

1. This document complements the existing deployment guide that was designed to help you to associate a Palo Alto VM-Series. We are going to assume that you have completed all the steps from 1 to 6 before launching this firewall instance. Step 7a is not necessary as it is Palo Alto VM-Series specific.

2. Currently we do not have a full integration between the Aviatrix dashboard and the Barracuda CloudGen Firewall, which means that you will not be able to update the firewall routing table via API, as it is currently possible with the Palo Alto VM-Series.


=========================================================
Deploying a Barracuda CloudGen Firewall for use with the Aviatrix FireNet
=========================================================

The goal of this document is to provide a step by step guide to launch and configure one or more Barracuda CloudGen Firewall instances to be integrated with the Aviatrix Firewall Network.

This setup will include basic “allow-all”  policies to serve as initial configuration to validate intended traffic is passing through the firewall instance.


1. Setup Firewall Network (FireNet)
---------------------------------------
Complete steps 1-5 of Firewall Network Workflow in the Aviatrix controller to prepare your Firewall VPC (FireNet VPC). This will also setup the subnets that you will need later for launching Barracuda Firewall instance.

2. Deploy the Barracuda CloudGen Firewall Instance from the AWS Marketplace
----------------------------------------------------
2a. The Barracuda CloudGen Firewall is available as a Pay-As-You-Go(PAYG) or Bring-Your-Own-License(BYOL) model.  For ease of use we will use the PAYG model in this example.

2b. Click the Launch Instance link in the AWS console from the Region your Firenet has been deployed in.  Choose the AWS Marketplace link and search for Barracuda.  Choose the Barracuda CloudGen Firewall for AWS - PAYG.  

|image1|


2c. Click Continue when prompted.

|image2|

2d. Choose an instance size.  A t2.small can be used for the Demo.  Choose Next: Configure Instance Details

|image3|

2e. Choose the VPC created for the Aviatrix firenet from the dropdown.  Choose the Public-FW-ingress-egress-us-east-1b subnet.  In this example we are using the US-East region.

|image4|

2f. Scroll down to adjust the network interfaces.  Click Add Device to add another Network Interface.  On the Subnet dropdown for eth1, choose the aviatrix-fireGW-DMZ-firewall subnet.  

|image5|


2g. Click Next: Add Storage, the Next: Add Tags.  Add any necessary tags then click Next: Configure Security Group.  No changes need to be made to the Security Group now.  Choose Review and Launch, the Launch on the next screen.  You will be prompted for a key pair.  None will be needed for the Firewall.  Choose Launch Instances.

Left off here


2h. On the instance details page, the most relevant setting that are general for any deployment is the subnet selection for the ENIs eth0 and eth1. You will configure eth2 later via AWS Console and the Gaia Portal. The CloudFormation template in R80 does not solve this problem yet.
2i. If you have followed all the steps on the Firewall page, then your subnet selection should follow this logic.
  ▪ Eth0 as the egress interface should be placed in the subnet FireNet-vpc-Public-FW-ingress-egress.

  ▪ Eth1 as the LAN interface should be placed in the subnet aviatrix-FW-GW-dmz-firewall (Same AZ as eth0)
  Eth2 as the management interface should be placed in the subnet FireNet-vpc-Public-gateway-and-firewall-mgmt (same AZ as eth0) will be configure later on step 3e.

  ▪ Also, don’t forget to enable “Auto-assign Public IP”.

|image3|
2j.  At the bottom of this page, click on “Add device” to create eth1 and select the proper subnet.

|image4|

2k. Then click on “Next: Add storage” – the default setting should be fine;
2l. Then click on “Next: Add Tags” – if you use tags in your environment, this is the time;
2m. Then click on “Next: Configure Security Group” – by default you are going to see SSH, HTTPS and the entire TCP port range open to the world. You can then click on “Review and Launch” or (OPTIONAL) you can isolate the instance public interfaces with the following three rules:
  ▪ All inbound traffic allowed for your own public IP (you will have to SSH to the instance and connect to it from the SmartDashboard)

  ▪ All inbound traffic allowed for the controller IP (even though only TCP port 443 and ICMP will be used)

  ▪ All inbound traffic allowed for RFC 1918 addresses (this should cover your spoke CIDRs).

2n. The next page will be a summary containing all of your previous choices, as well as any relevant AWS warning on how you can improve your deployment (e.g: open security groups, AMI usage tier consideration, etc).

2o. Once you click on “Launch” you will be prompted to choose the .prem key – please download the key now if have not done it already and archive it in a directory with proper privileges/restrictions, as you are going to use it to SSH into the instance to enable GUI access. You can now jump to item 2s. if you are deploying R77.30.

2p. Now, if you are deploying version R80.10, you should be able to use the CloudFormation stack, which saves some time with the basic setup (less clicks). The key point is that the fact that the template defines the first interface (eth0) as “external” and the second (eth1) as “internal” does not mean anything – what matters is the subnet selection. So, for consistency purposes we suggest keeping eth0 as management, eth1 as egress and eth2 for LAN (which will be configured later).

2q. The template should look like this (if you have selected existing VPC). Please make sure your interfaces are in the same AZ.
|image5|
|image6|
|image7|

2r. After you click on “Create” you should go to CloudFormation to monitor the stack creation. Once the status is set to “CREATE_COMPLETE” you should be able to move on. Any different warning can be troubleshooted by checking the details in the “Outputs” tab are they are usually self-explanatory;
2s. If you are installing R77.30, you can now click on the link containing the instance ID as it will redirect you to the Instances page where you can monitor the status check (if you are installing R80.10, just go to the EC2 instances page) – once they are done, you should be able to SSH into the instance
|image8|

3. Login to Firewall and configure interfaces
------------------------------------------------



3a. Now that the instance is up – open your preferred terminal and SSH into the instance using the proper keys and the user “admin”. It takes only two commands to set a new password.

|image9|

3b. Please open a browser and go to https://controller_EIP. You should be prompted with a screen like the one below. Just enter the user name as admin and the password you have just configured on the previous step.

|image10|
3c. IMPORTANT: if you are installing R80.10 via Cloud Formation you can skip step 3d. as the stack took care of these settings already.
3d.The Gaia Portal will take you through the initial Wizard to do some basic setup (the next bullet points were extracted from the Check Point Getting Started Guide):

  ▪ The WebUI shows the First Time Configuration Wizard. Click Next

  ▪ In the Deployment Options window, click Next

  ▪ In the Management Connection window, click Next

  ▪ In Connection to UserCenter, manually configure the IPv4 address of eth0. This information should be correct as we have chosen to auto-assign the IP for eth0. Click Next

  ▪ (OPTIONAL) In Device Information, set the Host name. Click Next

  ▪ (OPTIONAL) Set the Domain name and IPv4 addresses for the DNS servers – if you leave only .2, all your instance DNS traffic will be kept within the FireNet VPC.

  ▪ In Date and Time Settings, set the date and time manually OR if you prefer you can use the VPC NTP server (169.254.169.123). Click Next;

  ▪ In Installation Type, select Security Gateway and Security Management. Click Next.

  ▪ In Products, select Security Gateway or Security Management, or both. Click Next.

     a) If you checked Security Management, in the Security Management Administrator, set the administrator name and password.In the Security Management GUI clients, list the GUI clients that can log into the Security Management Server. Click Next.
     b) If you checked Security Gateway in Dynamically Assigned IP, make sure that ‘No’ is selected. Click Next.If you selected Security Gateway, in Secure Internal Communication (SIC), enter the Activation key. Click Next.

  ▪ Click Finish > Yes.

  ▪ If the Help Check Point Improve Software Updates window opens, click Yes or No. In a few minutes, you can use the WebUI to configure your stand-alone server.


3e. Now you need to add an extra interface to the Check Point instance via `via AWS Console <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#create_eni>`_. . This is going to be your eth2 and it should be associated with the subnet FireNet-vpc-Public-gateway-and-firewall-mgmt. You need to keep eth2 in the same AZ as the other interfaces;
3f. Also, don’t forget to disable “Source/dest. Check” as explained `here <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#change_source_dest_check>`_.
3g. Now that you have the new ENI created and configured, please `attach <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#attach_eni_running_stopped>`_. it to the CloudGuard instance. Please notice that while doing a  `hot attach <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#best-practices-for-configuring-network-interfaces>`_.it is possible that the instance will not recognize it immediately, so a stop/start of the instance might be necessary to address it.
3h. Please log back into the Gaia portal and go to Network Management, Network Interfaces, select eth2 and click on “Edit”. In the popup window, check Enable and also ‘Obtain IPv4 address automatically’. The eth2 IP should be the
same as the one seen in AWS Console. The screen shot below is from R80.10, but the step should be the same on R77.30, just a slightly different layout.

|image11|



4. Create static routes for routing of traffic VPC to VPC
------------------------------------------------------------

4a.The next step is to update the route table. For the purpose of this guide, we suggest adding three routes, each for a RFC1918 address pointing to the private IP of the eth2/ENI of the Aviatrix gateway in question (whether you are attaching the instance to the main or to the backup gateway). Just go to IPv4 Static Routes and click on “Add”. Repeat this step for all three RF1918 subnets:
|image12|
4b. Great. Now please download and install the SmartConsole. You nend to have have access to a Windows client so you can run the SmartConsole, which is provided by Check Point: `R77.30 <https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk104859#Gaia%20Downloads>`_. and `R80.10 <https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk119612>`_. On SmartConsole you need to define a security policy that will allow the traffic to be inspected/logged and register the new interface eth2;


5. Configure basic traffic policy to allow traffic
-----------------------------------------------------------

5a.The SmartConsole layout is different across the main versions. On R77.30, please go the Firewall tab, Policy and change the default policy to ‘accept’ traffic and to ‘Log’ it as well. This can (and SHOULD) be customized to comply with your project requirements. Finally, install the policy on the gateway(s) in question. Your basic policy should look like this:
|image13|
5b.Then click on “Install Policy” on the top menu, and click OK to commit this change.
|image14|
5c.The last step is to register the new eth2 that was manually added via attachment to your SmartConsole topology: click on the upper-high-left menu button, select Manage, Network Objects.
|image15|
Then find the referred gateway in the list and click on Edit:
|image16|
5d. On the next screen, please click on Topology and then on “Get…” and “Interfaces…”. Just select eth2 and hit “Accept”.
|image17|
5e. (OPTIONAL) On this same screen you can update the “Network Type” of interfaces eth0 and eth2 to “External” and enable “Anti-Spoofing” in the Topology tab under the “Edit” section;
5f. That is it – the next steps will refer the R80.10 SmartConsole instead, but they are pretty much the same thing: the basic policy can be accessed via Security Policies and then Policy.
|image18|
5g. As per the topology page, it can be reached via Gateways & Servers and a double-click on the gateway itself. Then click on Network Management, Get Interfaces

|image19|

6. Ready to go!
---------------

Now your firewall instance is ready to receive packets!

The next step is specify which Security Domain needs packet inspection by defining a connection policy that connects to
the firewall domain. This is done by `Step 8 <https://docs.aviatrix.com/HowTos/firewall_network_workflow.html#specify-security-domain-for-firewall-inspection>`_ in the Firewall Network workflow.

For example, deploy Spoke-1 VPC in Security_Domain_1 and Spoke-2 VPC in Security_Domain_2. Build a connection policy between the two domains. Build a connection between Security_Domain_2 to Firewall Domain.

Launch one instance in Spoke-1 VPC and Spoke-2 VPC. From one instance to ping the other instance. The ping should go through. .

7. View Traffic Log
----------------------
7a. The final step is to monitor your traffic to confirm that the inspection is being performed as configured. If you deployed the R77.30 instance, then you should open the SmartView Tracker and filter the logs accordingly
|image20|
|image21|
7b. On the R80.10 SmartConsole, go to Logs & Monitor instead.
|image22|
7c. Now, we added a third interface as currently our dashboard requires 3 separate interfaces, but CloudGuard will use eth0 for both management and egress traffic by default. If you would like to move the Gaia management interface to eth2, please use this `link <https://sc1.checkpoint.com/documents/R80.20_GA/WebAdminGuides/EN/CP_R80.20_Installation_and_Upgrade_Guide/html_frameset.htm?topic=documents/R80.20_GA/WebAdminGuides/EN/CP_R80.20_Installation_and_Upgrade_Guide/205119>`_.as a reference.
7d. Great. You are now good to repeat this process to add more instances to talk to the active gateway and also to the backup gateway. The difference regarding the backup gateway attachment is that the subnets will likely be in a different AZ.
You can view if traffic is forwarded to firewall instance by going to FortiView
8e. For more information on the Firewall network solution, please refer to this `link <https://docs.aviatrix.com/HowTos/firewall_network_faq.html>`_.


.. |image1| image:: ./config_Checkpoint_media/image1.png
    :width: 100%
.. |image2| image:: ./config_Checkpoint_media/image2.png
    :width: 100%
.. |image3| image:: ./config_Checkpoint_media/image3.png
    :width: 100%
.. |image4| image:: ./config_Checkpoint_media/image4.png
    :width: 100%
.. |image5| image:: ./config_Checkpoint_media/image5.png
    :width: 100%
.. |image6| image:: ./config_Checkpoint_media/image6.png
    :width: 100%
.. |image7| image:: ./config_Checkpoint_media/image7.png
    :width: 100%
.. |image8| image:: ./config_Checkpoint_media/image8.png
    :width: 100%
.. |image9| image:: ./config_Checkpoint_media/image9.png
    :width: 100%
.. |image10| image:: ./config_Checkpoint_media/image10.png
    :width: 100%
.. |image11| image:: ./config_Checkpoint_media/image11.png
    :width: 100%
.. |image12| image:: ./config_Checkpoint_media/image12.png
    :width: 100%
.. |image13| image:: ./config_Checkpoint_media/image13.png
    :width: 100%
.. |image14| image:: ./config_Checkpoint_media/image14.png
    :width: 100%
.. |image15| image:: ./config_Checkpoint_media/image15.png
    :width: 100%
.. |image16| image:: ./config_Checkpoint_media/image16.png
    :width: 100%
.. |image17| image:: ./config_Checkpoint_media/image17.png
    :width: 100%
.. |image18| image:: ./config_Checkpoint_media/image18.png
    :width: 100%
.. |image19| image:: ./config_Checkpoint_media/image19.png
    :width: 100%
.. |image20| image:: ./config_Checkpoint_media/image20.png
    :width: 100%
.. |image21| image:: ./config_Checkpoint_media/image21.png
    :width: 100%
.. |image22| image:: ./config_Checkpoint_media/image22.png
    :width: 100%
