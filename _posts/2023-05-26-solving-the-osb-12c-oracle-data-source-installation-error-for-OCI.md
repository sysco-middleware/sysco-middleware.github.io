---
layout: post
title: Solving the OSB 12c Oracle data source installation error for OCI
categories: OSB tips
tags: [OSB 12c, sqlnet.ora, tnsnames.ora, OCI, Wallet, ssl_server_dn_match, SSL_SERVER_DN_MATCH, ssl_server_cert_dn]
author: denzza
---
### INTRODUCTION ###

In the realm of data management and infrastructure, technical difficulties may often arise, one of which is an error message during the installation of a data source in Oracle Service Bus (OSB) 12c to an Oracle Database in Oracle Cloud Infrastructure (OCI). 
The specific error in question reads: 
**"weblogic.application.ModuleException: oracle.security.crypto.asn1.ASN1FormatException: Length is too big: takes 68 bytes"** 

This blog post is dedicated to addressing this error with a solution that worked in our case.

### SOLUTION ###

The process begins with a pre-requisite: it's crucial to ensure you have the correct connection information for Database in the **Domain_Credentials.properties** file. 
Once this is guaranteed, the following steps could potentialy help in resolving the error.

1: Download the appropriate Wallet for the correct environment. Exemple names for different environment: **Wallet_Test, Wallet_Prod**

2: Upload the Wallet to all nodes in your environemnt at the path which could be somenthing like: **/u01/app/oracle/product/fmw/wallets**

3: There are two files that need to be updated. The first file to open is **sqlnet.ora**:

Change the **DIRECTORY** to point to the correct Wallet folder. For example, for Wallet_Test, it would be:
**DIRECTORY="/u01/app/oracle/product/fmw/wallets/Wallet_Test"**

Change **SSL_SERVER_DN_MATCH** from SSL_SERVER_DN_MATCH=**no** to SSL_SERVER_DN_MATCH=**yes**

4: The next file to modify is **tnsnames.ora**:

Replace **ssl_server_dn_match=no** in security with the certificate's information. Example:
Replace: **(security=(ssl_server_dn_match=no))**
To: **(security=(ssl_server_cert_dn="CN=adwc.eucom-central-1.oraclecloud.com,OU=Oracle BMCS FRANKFURT,O=Oracle Corporation,L=Redwood City,ST=California,C=US"))**

5: Run the script to install the data source or do it manually depends on you.

6: If you encounter errors, try rebooting the Admin and Managed servers.

### Conclusion ###

By following these steps, you can address the ModuleException during the installation of a data source in OSB 12c to Oracle Database in OCI. 
Keep in mind that this is a technical solution that worked in our specific case, different scenarios might require different solutions. 
In all cases, always ensure the integrity and security of your data and systems while attempting fixes.