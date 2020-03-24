---
layout: post
title: Setting up email server using Postfix
categories: Mail Server Deployment
tags: [Mail Server, Postfix, Ubuntu]
author: AshishKapoor
---

# Introduction
The purpose for this post is to enable you to setup your own email server. For this purpose, we chose Postfix as a mail server. Postfix is a free and open-source mail transfer agent that routes and delivers electronic mail.

In this Post, we will explain how to install and configure Postfix mail server on Ubuntu 18.4 server.

# Prerequisites: 
For installing Postfix server, your user should have sudo privileges (admin rights for installing postfix). If you want to receive emails from internet you should have DNS Records for Your Mail Server. You need to make sure Port 25 is open so that Postfix can receive emails from other SMTP servers.

# Overview of steps
- **Goal 1**: Setting up mail server for receiving mails from local network.
- **Goal 2**: Setting up mail server for receiving mails from internet.


# Goal 1: Setting up mail server for receiving mails from local network
## Background: Selection of Cypress as the test framework

Before start configuring Postfix server, you need to make sure that proper entries in /etc/hosts. As most of programs will not accept an email using just @localhost as domain. 
```bash
 	vi /etc/hosts
```
Your host file will look something like this.

![message](/images/2020-03-24-Setting-up-email-sever-using-Postfix/hosts.png)

## Installing and Configuring Postfix server on ubuntu with Command line interface

Installing Postfix on Ubutu server is quite simple task, but after installing postfix you need make sure that you configure postfix server properly.

Install the postfix package if it is not installed already.
```bash
 	sudo apt-get install postfix
```
**Configure a Catch-all Address**

Enabling this, you can use any email address ending with "@localhost" or "@localhost.com".

 if not exists, create file /etc/postfix/virtual: 

```bash
    sudo nano /etc/postfix/virtual
```
Add the following 2 lines content, replacing <your-user> with your Unix user account:

```bash
    @localhost <your-user>
    @localhost.com <your-user> 
```
Save and close the file.

**Configure a Catch-all Address**

To Change the configuration of postfixx server we need to read this file:
```bash
    sudo vi /etc/postfix/main.cf
```
And check if this line is enabled in main.cf, or add it if not exists:
```bash
    virtual_alias_maps = hash:/etc/postfix/virtual
 ```
Save and close this file and to activate these changes by running below commands:
```bash
    sudo postfix reload
    sudo service postfix restart
 ```
**Configure Postfix to use Maildir-style mailboxes and other settings.**
Open Postfix configuration file
```bash
    sudo vi /etc/postfix/main.cf
 ```
Add below lines after inet_protocoles and save main.cf

```bash
    luser_relay = sysco@localhost
    local_recipient_maps =
    home_mailbox = Maildir/
    disable_mime_output_convesion = yes
    strict_8bitmime = yes
    strict_7bit_headers = no
    strict_8bitmime_body = yes
 ```

Reload postfix and restart it:
```bash
    sudo postfix reload
    sudo service postfix restart
 ```
Your /etc/postfix/main.cf will look like this:

![message](/images/2020-03-24-Setting-up-email-sever-using-Postfix/maincf.png)

**Optional steps**
Let Postfix know about the domains that it should consider local:
```bash
    sudo postconf -e "mydestination = gitlab.example.com, localhost.localdomain, localhost"
 ```
Let Postfix know about the IPs that it should consider part of the LAN:
We’ll assume 192.168.1.0/24 is your local LAN. You can safely skip this step if you don’t have other machines in the same local network.
 ```
    sudo postconf -e "mynetworks = 127.0.0.0/8, 192.168.1.0/24"
 ```

Remember to relaod Postfix configuration and restart it.

Checking Postfix server status.
```bash
    sudo service postfix status
 ```
Starting Postfix server.
```bash
    sudo service postfix start
 ```
 Stoping Postfix server.
```bash
    sudo service postfix stop
 ```

**Testing Postfix sever**

For testing postfix server first, we will install mailutils for sending mail to local server
```bash
    sudo apt install mailutils
```
Execute below command to send mail to local mail sever
```bash
		mail -s "Test Subject" 27148485@localhost.com < /dev/null
```
and check mail /Maildir/new , if you receive any file in this folder then your postfix server is configured properly and you mail server setup is complete for receiving emails from local network.

# Goal 2: Setting up mail server for receiving mails from internet 

Postfix requires server’s hostname to identify itself when communicating with other MTAs. Host name can have Single word or Fully Qualified Domain Name(FQDN). FQDN is usually used for internet facing servers. You should setup DNS records for your mail server in your DNS hosting service. In this you have to setup MX record and A record.

**MX reco*rd**: An MX record tells other MTAs that your mail server mail. syscotest.com is responsible for email delivery for your domain name.
```bash		
    MX record    @           mail.syscotest.com
```
**A record**: An A record maps a FQDN to an IP address.
```bash
    mail.syscotest.com        <IP-address>
 ```
**Installing Postfix**

Run below command in terminal to install Postfix server.
```bash
    sudo apt-get update
    sudo apt-get install postfix -y
 ```
The installation process will ask you to select mail configuration. You need to select Internet Site for this.

![message](/images/2020-03-24-Setting-up-email-sever-using-Postfix/PostfixInternetSite.png)

**No configuration** means the installation process will not configure any parameters.

**Internet Site** means using Postfix for sending emails to other MTAs and receiving email from other MTAs.

**Internet with smarthost** means using postfix to receive email from other MTAs, but using another smart host to relay emails to the recipient.

**Satellite system** means using smart host for sending and receiving email.

**Local only** means emails are transmitted only between local user accounts. (For Goal 1 select this option)

In step it will ask you to enter domain name for the system mail name. In our case, we have entered “syscotest.com” as system mail server.

![message](/images/2020-03-24-Setting-up-email-sever-using-Postfix/syscotest.png)

Once installed check Postfix configuration file  “/etc/postfix/main.cf” by opening it in your favourite editor. Your main.cf will look like this:

```bash
    myhostname = syscotest.com
    mydomain = syscotest.com
    myorigin = $mydomain
    mydestination = $myhostname, localhost, $mydomain, localhost.localdomain
    mynetworks = 127.0.0.0/8, /32
    relay_domains = $mydestination
    inet_interfaces = all
    inet_protocols = all
    home_mailbox = Maildir/
 ```
Check Postfix status by executing below command:
```bash
    sudo service postfix status
```
![message](/images/2020-03-24-Setting-up-email-sever-using-Postfix/status.png)

if this command returns status running, then postfix is up and running.

**Testing mail server**
Before testing this setup make sure you have mapped your mail server user in “/etc/postfix/virtual”. Your virtual file should look like this
```bash
contact@syscotest.com sysco
admin@syscotest.com Sysco
```
We can apply the mapping by typing:
```bash
		sudo postmap /etc/postfix/virtual
```
Restart the Postfix process to be sure that all our changes have been applied:
```bash
sudo systemctl restart postfix
```
Goto you gmail account and send mail to admin@syscotest.com. Mail should be sent successfully, and you should receive mail in “/Maildir/new”.

# Conclusion

Configuring and Managing mail servers can be a difficult task for new administrators, but with this configuration, you should have basic MTA email functionality to get you started. Success factor for mail server configuration depends upon how you configure and run your mail server.