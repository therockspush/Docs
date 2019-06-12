.. meta::
  :description: Firewall Network
  :keywords: AWS Transit Gateway, AWS TGW, TGW orchestrator, Aviatrix Transit network, Transit DMZ, Egress, Firewall

**A few important notes before we launch the instance:**

1. This document complements the existing deployment guide that was designed to help you to associate a Palo Alto VM-Series. We are going to assume that you have completed all the steps from 1 to 6 before launching this firewall instance. Step 7a is not necessary as it is Palo Alto VM-Series specific.

2. Currently we do not have a full integration between the Aviatrix dashboard and the CloudGuard, which means that you will not be able to update the firewall routing table via API, as it is currently possible with the Palo Alto VM-Series.

3. The Check Point CloudGuard has mainly two types of deployments: standalone or distributed – which basically means whether you are going to have both the Gateway and the Management Server on the same instance. Since we currently CANNOT configure the firewall policies without the management server, we need to configure a Management Server. Also, we need to consider that the Aviatrix gateways within the FireNet VPC will monitor whether the Check Point gateway instance is up or not. That being said, only versions R77.30 and R80.10 support standalone deployments, while another possibility is to have a single R80.20 Management Server inside the management subnet to control multiple CloudGuard instances (although this is not part of the scope. For more information, try this `link <https://supportcenter.checkpoint.com/supportcenter/portal/user/anon/page/default.psml/media-type/html?action=portlets.DCFileAction&eventSubmit_doGetdcdetails=&fileid=24831>`_.

4. For more information on the differences across the available models/versions we suggest the following `link <https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk95746>`_. Check Point has recommended the upgrade to R80 as part of their roadmap. For more information regarding such advisories, please check this `link <https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk110980>`_.



=========================================================
Example Config for Checkpoint VM in AWS
=========================================================

The goal of this document is to provide a step by step guide to launch and configure one or more CloudGuard IaaS[R77.30] Next Gen Firewall instances to be integrated with Aviatrix Firewall Network.

This setup will include basic “allow-all”  policies to serve as initial configuration to validate intended traffic is passing through the firewall instance.
Checkpoint’s documentation should be consulted for configuration of security policies and features.

1. Setup Firewall Network (FireNet)
---------------------------------------
Complete steps 1-5 of Firewall Network Workflow in Aviatrix controller to prepare your Firewall VPC (FireNet VPC). This will also setup the subnets that you will need for launching your Checkpoint instance.

2. Deploy Checkpoint Instance from AWS Marketplace
----------------------------------------------------
2a. Chose your subscription model (BYOL vs PAYG). For more information on this topic, please check page 11 of the Check Point `documentation <http://dl3.checkpoint.com/paid/eb/ebb444ce93242cf3f80f76637678906b/CP_R77.30_SecurityGateway_AmazonVPC_GettingStartedGuide.pdf?HashKey=1559349126_ed97c19f0055aaa62bf0bd69ba4e42ac&xtn=.pdf>`_.

2b. Go to aws.amazon.com/marketplace and search for the chosen instance model/version in AWS Marketplaceand “Continue to Subscribe”

[[HowTos/config_Checkpoint_media/image1.png]]
      or
|image2|
