.. meta::
   :description: Aviatrix Cloud Account for Azure
   :keywords: Aviatrix account, Azure, Aviatrix Azure account credential, API credential

===========================================================
Azure ARM
===========================================================

1.0 Overview
=============

This document helps you to setup API credentials on Azure ARM. 
 
Aviatrix Cloud Controller uses Azure APIs extensively to launch Aviatrix
gateways, configure encrypted peering and other features.

In order to use Azure API, you need to first create an Aviatrix Cloud
Account on the Aviatrix Cloud controller. This cloud account corresponds
to a valid Azure account with API credentials.

The new Microsoft Azure (as opposed to Azure Classic) is significantly
different in how applications are authenticated and authorized to
interact with Azure Resource Manager APIs to manage resources, such as
Virtual Machines, Network, Storage Accounts, etc.

This document describes how to obtain the necessary information,
specifically Application ID, Application Key, and
Application Directory ID to create an Aviatrix Cloud Account with step by
step instructions. There are 3 sections, make sure you go through all of
them.

|


2.0 Azure Permission Setup for Aviatrix
========================================

Setting up Azure permission for Aviatrix involves three main steps.

1. Register Aviatrix Controller Application with Azure Active Directory

2. Grant Permissions

3. Get Application ID, Application Key and Directory
   ID

**Important:** Complete the follow steps in order.

2.1  Step 1 – Register Aviatrix Controller Application
-------------------------------------------------------

Login to the Azure Portal.

https://portal.azure.com

***Register Aviatrix Controller***

1. From the Azure portal click on "All services" and search for “Azure Active Directory” and click on “Azure Active Directory”

|Image01|

2. Click “App registrations".  Do not choose "App registrations (Legacy)"

|Image03|

3. Click “+ New registration”

|Image04|

   a. Name = Aviatrix Controller

   b. Supported account types = Accounts in this organizational directory only

   c. Click Register.

3. Done

2.2 Step 2 – Grant Permissions
-------------------------------


***Grant Permissions***

1. Login to the Azure portal

2. On the top left, click All services, search for “Subscriptions”

|Image11|

3. Copy the Subscription ID (to notepad or a convenient location)

|Image12|

4. Click on the Subscription ID

5. Then select “Access control (IAM)”.

|Image13|


6. Click Add and then select the “Contributor” role.


7. In the Select search field, type in “Aviatrix”. The Aviatrix Controller
   app should show up. Select this one and click Select towards to the
   bottom.

2.3 Step 3 – Get Application Information
-----------------------------------------

**Get Application Information**

1. From the Azure portal, click All services and search for “Azure Active Directory”.

   |Image01|

2. Retrieve the **Application (client) ID** and **Directory (tenant) ID**.
   
   A. Copy the Application ID and Directory ID for later use.  


3. Retrieve the **Client Secrets**.

   A. Click Certificates & secrets

   B. Click New client secret

   |Image06|


   |Image07|

   C. Enter in the following

      * Description = Aviatrix

      * Expires = Never

   E. Click Add

   F. Copy the secret

5. Add **API permissions**.

   A. Click API permissions

   |Image08|

   B. Click "+Add a permission"
   
   C. Choose Azure Service Management
   
   |Image09|

   D. Select user_impersonation then Add permissions
   
   |Image10|

6. Done

At this point you should have the following information.

+-----------------------------------+---------------+
| **Subscription ID**               | From step 2   |
+===================================+===============+
| **Directory** **ID**              | From step 3   |
+-----------------------------------+---------------+
| **Application ID**                | From step 3   |
+-----------------------------------+---------------+
| **Client secret**                 | From step 3   |
+-----------------------------------+---------------+


.. |image01| image:: AviatrixAccountForAzure_media/az-ad-01.PNG
   :width: 5.20313in
   :height: 1.50209in
.. |image02| image:: AviatrixAccountForAzure_media/az-ad-directory-id-02.PNG
   :width: 5.65600in
   :height: 2.39763in
.. |image03| image:: AviatrixAccountForAzure_media/az-ad-app-03.PNG
   :width: 6.98958in
   :height: 3.02083in
.. |image04| image:: AviatrixAccountForAzure_media/az-ad-app-create-04.PNG
   :width: 5.20313in
   :height: 1.50209in
.. |image05| image:: AviatrixAccountForAzure_media/az-ad-list-all-apps-05.PNG
   :width: 5.65600in
   :height: 2.39763in
.. |image06| image:: AviatrixAccountForAzure_media/az-ad-application-id-06.PNG
   :width: 6.98958in
   :height: 3.02083in
.. |image07| image:: AviatrixAccountForAzure_media/az-ad-app-keys-save-07.PNG
   :width: 5.20313in
   :height: 1.50209in
.. |image08| image:: AviatrixAccountForAzure_media/az-ad-app-permissions-add-08.PNG
   :width: 5.65600in
   :height: 2.39763in
.. |image09| image:: AviatrixAccountForAzure_media/az-ad-app-permissions-add-api-09.PNG
   :width: 6.98958in
   :height: 3.02083in
.. |image10| image:: AviatrixAccountForAzure_media/az-ad-app-permissions-api-set-10.PNG
   :width: 5.20313in
   :height: 1.50209in
.. |image11| image:: AviatrixAccountForAzure_media/az-ad-sub-role-11.PNG
   :width: 5.65600in
   :height: 2.39763in
.. |image12| image:: AviatrixAccountForAzure_media/az-ad-sub-list-12.PNG
   :width: 6.98958in
   :height: 3.02083in
.. |image13| image:: AviatrixAccountForAzure_media/az-ad-sub-contrib-13.PNG
   :width: 6.98958in
   :height: 3.02083in


.. add in the disqus tag

.. disqus::   
