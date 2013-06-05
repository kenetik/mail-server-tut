Free Perfect Email Server - Detailed Tutorial
===========================================
This tutorial covers all the details steps for creating your own custom email server using a Virtual Private Server running Ubuntu 12.10 x64 with iRedMail 0.8.4, iRedAdmin, PostgreSQL, Roundcubemail, Awstats, Apache and SSL Certificates.
-------------------------------------------
_I used [Digital Ocean SSD VPS](https://www.digitalocean.com/?refcode=297ac3dfc7bf) for this tutorial. (Included in this tutorial is a coupon[^1] for 2 months of free service.)_ 

If you already have a perfectly configured server, you can skip to: 'Install iRedMail'.
- - - 
###Create a Droplet (aka VPS)
####I highly recommend using [Digital Ocean SSD VPS]( ) for this setup. If you are creating a new account, use [this coupon](#fn:1) for 2 months of free service.
After creating a new account, click `Create Droplet`  
    * [Screen Shot of Creating a Droplet via Digial Ocean](http://grab.by/n7w6)

1. Assign hostname (for this tutorial I will be using mail.yourdomain.com)
From my tests 2GB/2CPUS/40GBSSD configuration works the best, but for trial and error we will use the smallest configuration.
[Screen Shot of Assigning Hostname](http://grab.by/n7wa)

2. Select Region - _This is based on your location, or potenial users location._ I chose San Francisco 
    * Select Image - _We will be using Ubuntu 12.10 x64 for this tutorial._ 
        + [Screen Shot of Selecting Image and Location](http://grab.by/n7wE)
    * Click Create Droplet - You will be greeted with a message about your droplet being created. (Wait approximately 60 seconds, usual much less.) You will be automatically redirected to your droplet upon it's creation. Check your email for the root password.
        + [Screen Shot of Create Button](http://grab.by/n7wK)
        + [Screenshot of Droplet After Creation](http://grab.by/n7wU)
3. Change your server's password
    * Login to your server using a SSH Client (PuTTY/Terminal/Digital Ocean's Console Access) with the details given from the email. In your SSH Client type the following command:
        + ```
          ssh root@yourip
          ```
    * You will be prompted about a RSA key fingerprint. Type 'yes' as the prompt. 
        + ```
          yes
          ```
        + [Screenshot of RSA Prompt](http://grab.by/n7xw) 
  * Enter the random password generated during droplet creation. You should now be logged into your server and see something similar to this: 
        + [Screenshot of Successful Server Login](http://grab.by/n7xI) 
    * __It is very important__ to now change the password to something secure of your choice. At the command prompt type the following command: 
	    + ```
          passwd
          ```
        + [Screenshot of Password Change](http://grab.by/n7xM)
4. Reboot your server by typing the following command:

Now your in! Let's start configuring!
- - -
###Setup 2GB of Swap Memory
_This helps with server stability and is optional but __highly recommended__._

1. Login back into your server and type the following command:
    * ```
	  dd if=/dev/zero of=/swap bs=1024 count=2097152
	  mkswap /swap && chown root. /swap && chmod 0600 /swap && swapon /swap
	  echo /swap swap swap defaults 0 0 >> /etc/fstab
	  echo vm.swappiness = 0 >> /etc/sysctl.conf && sysctl -p
	  ```
        + [Screen Shot of Command](http://grab.by/n7yI)
2. Check to make sure your swap file is active by typing the following command:
    * ```
	  free -m
      ```
      will show Swap: 2047
        + [Screen Shot of Swap: 2047 Displayed](http://grab.by/n7yO)

Swap (virtual ram) is setup!
- - -
###Set your domain's DNS via Digital Ocean's Control Panel
_Please ensure that your domain's dns are forwarding to your server if you are not using Digital Ocean_

1. Add a domain to your account by visiting [Digital Ocean's Domain Control Panel](https://www.digitalocean.com/domains) | [Screen Shot of DCP](http://grab.by/n7z2)
	1. Click the [Add Domain] button
	2. Input your domain, droplet's ip address, and select your droplet.
	    + [Screen Shot of Domain Inout](http://grab.by/n7z8)
	3. Click the [Create Domain] button. You should see 'Domain was successfully created'.
	4. Click the [Add Record] button
        * Select MX as the Record Type
		* In Hostname input:
            + `mail.yourdomain.com.` _Ensure the trailing `.` after your domain._
        * In Priority
        * You also want to add a CNAME record for 'mail', '@'. _This may seem a little redundant, but it ensures SSL Certification ease later._
            + `10`
        * Click the [CREATE] button
		 	+ [Screen Shot of Domain Information Input](http://grab.by/n7IM)

DNS is set!
- - -
###Ensure VPS is Updated
_Again, if this is a pre-configured VPS and you now everything is good to go, you may skip this step, but is still recommended_

1. From your SSH Client use the following command:
    * `apt-get update`
        + [Screen Shot of Update Results](http://grab.by/n7zG)
        + and then this command
    * `apt-get upgrade`
        + Depending on how many items need to be updated, you will see something similar to the following screenshot and be prompted to continue. Type `Y`, and your server will begin updating.
        + [Screen Shot of Update Process](http://grab.by/n7zO)     

Your up to date!
- - -
###Set your Fully Qualified Domain Name (FQDN)
_If its already set, it would be a good idea to confirm it_

1. Edit the hosts file by typing the following command:
    * `nano /etc/hosts`
        + [Screen Shot of the Hosts file via Nano](http://grab.by/n7A2)
2. Change the default line to:
    * `127.0.0.1 mail.yourdomain.com mail localhost`
    * _You can verify this by rebooting, and typing:_
        + `hostname -f`

Your FDQN is now set!
- - -
###Install iRedMail
_This is the magic software and step for all users_

1. From the command prompt type the following command:
    * ```cd /tmp
        wget https://bitbucket.org/zhb/iredmail/downloads/iRedMail-0.8.4.tar.bz2
        tar jxvf iRedMail-0.8.4.tar.bz2
        rm iRedMail-0.8.4.tar.bz2
        mv iRedMail-0.8.4 iRedMail
        cd iRedMail
        bash iRedMail.sh```
    * Your server will begin installing requirements. 
2. Once you see the screen 'Welcome and thank you for your use':
    * Select: 'Yes'
	    + [Screen Shot of iRedMail Welcome Screen](http://grab.by/n7Aa)
	* _/var/vmail is the default storage path, and what I use_
    * Select: 'Next'
	    + [Screen Shot of Default Storage Path](http://grab.by/n7Ac)
	* Select PostgreSQL with your arrow keys and the spacebar, then 'Next'
	    + [Sceen Shot of Selecting Database Manager(PostgreSQL)](http://grab.by/n7Ak)
	* Enter a password for your PostgreSQL admin (_you'll need this later_)
        + `password`, then 'Next'
	        + [Screen Shot of Password Choice](http://grab.by/n7Ao)
	* Enter your first virtual domain name, `yourdomain.com`, then 'Next'
	    + [Screen Shot of Virtual Domain Name](http://grab.by/n7Au)
	* Enter a password for your administrator account (_you'll need this later, and will eventually will need to change it_)
        + `password`, then 'Next'
	        + [Screen Shot of Admin Password](http://grab.by/n7AC)
	* Select optional components:
		+ Fail2ban, and phpPGAdmin are optional, I am chosing to install phpPgAdmin and not Fail2Ban, the rest are required for this tutorial, then 'Next'
		    + [Screen Shot of Optional Component](http://grab.by/n7AI)
	* _The selected configurations are complete. Take note, we'll need to move /tmp/iRedMail/config later_
    * Type `Y` to continue
        + [Screen Shot of ?](http://grab.by/n7AO)
	* iRedMail will begin downloading and installing required files. (_Approximately ~2 minutes_) 
3. iRedMail will prompt for firewall rules:
    * Select 'N'
	* iRedMail is now installed, _take note of the url's given_
		+ [Screen Shot of Installed Configuration](http://grab.by/n7B8)
	* Open iRedMail.tips for configuration specifics by typing the following command:
        + `nano /tmp/iRedMail/iRedMail.tips` (_Save this information incase you don't receive the email_)
    * Now reboot and log back in!
	    + `reboot`
		+ [Screen Shot of Reboot](http://grab.by/n7Bi)
		+ _Upon reboot, you should notice "You have mail."_

You've just installed iRedMail! Feel accomplished yet?
- - -
###Create a Secure SSL Certificate
_For tutorial purposes we will use [InstantSLL](http://www.instantssl.com/ssl-certificate-products/free-ssl-certificate.html) for a free certificate_

1. Create a Certificate Signing Request by typing the following command:
    * ```
	   cd /etc/ssl
	   openssl req -out mail.yourdomain.com.csr -new -newkey rsa:2048 -nodes -keyout mail.yourdomain.com.key
      ```
	* You will be prompted for information about your certificate, fill them out as requested. The 'extra' attributes are not required (Do not give a challenge password)
	    + [Screen Shot of Certificate Information](http://grab.by/n7DA)
2. Open your Certificate Signing Request by typing the following command:
	* `nano mail.yourdomain.com.csr`  
        + [Screen Shot of CSR Results](http://grab.by/n7DQ) _I have removed some information and replaced it with *'s for security purposes_
3. Copy all of the information located here, into the CSR Box from InstantSSL.
    * Select Apache-ModSSL from the server software drop-down, uncheck Opt in? Then click 'Next >'
    * [Screen Shot of CSR Box @ InstantSSL](http://grab.by/n7E4)
	* After [InstantSLL](http://www.instantssl.com/ssl-certificate-products/free-ssl-certificate.html) validates your request (more steps) you will receive a .zip containing two files. [ mail_yourdomain_com.ca-bundle and mail_yourdomain_com.crt] ([Screen Shot of the files](http://grab.by/n7GQ)) _To rush the validation, you can log into the comodo account you created and download the .zip of certificate files from your account panel._
4. Place both of these files in /etc/ssl on your server
	* After unzipping the document, open each file with your favorite text editor.
	* Copy and paste the information in each file into the same file name on your server.
5. Modify Apache's default-ssl to reflect these SSL Certificates
    * Type the following command `nano /etc/apache2/sites-available/default-ssl`
	* Replace the default information so that the following is set

                SSLCertificateFile /etc/ssl/mail_yourdomain_com.crt
        	    SSLCertificateKeyFile /etc/ssl/mail.yourdomain.com.key
        	    SSLCACertificateFile /etc/ssl/mail_yourdomain_com.ca-bundle
    
    * _Please ensure to remove the `#` before SSLCACertificateFile and do not place the `>>` in the file, as these are indicators for your benefit_
        + [Screen Shot of Default-SSL Configuration](http://grab.by/n7HC)
		+ Use `Ctrl+X` and `Y + Enter` to save the adjustments

6. Modify Postfix and Dovecot's configuration files
    * Postfix: `nano /etc/postfix/main.cf`
        + [Screen Shot of Postfix main.cf](http://grab.by/n8Gw)
        + Under #TLS Parameters change: 
    
                smtpd_tls_cert_file = /etc/ssl/mail_yourdomain_com.crt 
                and 
                smtpd_tls_key_file = /etc/ssl/mail.yourdomain.com.key
                
                
        + Use `Ctrl+X` and `Y + Enter` to save the adjustments
        
	* Dovecot: `nano /etc/postfix/main.cf`
        + [Screen Shot of Dovecot main.cf](http://grab.by/n8GQ)
		+ Under # SSL: Global settings change: 


                ssl = required 
                verbose_ssl = yes 
                (this is optional, but added for debug help) and, 
                ssl_cert = </etc/ssl/mail_yourdomain_com.crt 
                ssl_key = </etc/ssl/mail.yourdomain.com.key  


        + Use `Ctrl+X` and `Y + Enter` to save the adjustments

7. Reboot by issuing the command `reboot`
	* [Screen Shot of Reboot](http://grab.by/n8GW)
    
8. Verify your SSL is working by visiting https://mail.yourdomain.com
_Depending on which browser/OS you are using, you will see a lock icon next to your URL similar to the screenshot_	[Screen Shot of SSL Secured Page](http://grab.by/n7J4)

You have your very own SSL Secured Address!
- - -
###Configure iRedAdmin Accounts

1. Login to iRedAdmin and configure accounts
	* https://mail.yourdomain.com/iRedAdmin
	* Username:` postmaster@yourdomain.com` and Password: `password` (_or whatever you set earlier, located in iRedMail.tips_) then click [Login]
        + _This is your main mail server admin panel or configuration portal_
    	+ [Screen Shot of Login Screen](http://grab.by/n7Je)
2. Change your password! 
    * Click Preferences in the top right, then select Password to the right of General
		+ [Screen Shot of Preferences](http://grab.by/n7Js)
		+ After changing your password, I would also recommend removing your Mailbox Quota [0] and changing your User ID to [admin] or your preference
		+ [Screen Shot of Preferences Settings](http://grab.by/n7Jw)
		+ Logout and back in
3. Disable Greylisting!
_This is my personal preference, it's only given me problems in the past_
	* iRedAPD (aka Cluebringer)
	* Go to https://mail.yourdomain.com/cluebringer/greylisting-main.php
		+ It will prompt for a Username and Password. This is the same as your postmaster address (eg. postmaster@yourdomain.com:password)
			- [Screen Shot of Cluebringer Login](http://grab.by/n7Lu)
			- Once here, select Default Inbound, Action >> Delete
    * Disable Policies
    _Also personal preferene_
    * Go to https://mail.yourdomain.com/cluebringer/policy-main.php	
		+ Select the policy of your choice (Action Change)
		+ On the policy edit page, Disable < Yes
4. Create a new email address.
	* Return to https://mail.yourdomain.com and login
	* Select [+Add...] > >User
		
            Mail Address* [anythingyouwish]@[yourdomain.com]
		    New password* [********]
		    Confirm new password [********]
		    Display Name [not required but suggested]
		    Mailbox Quota [0-99999] 

		+ _Determine Mailbox Quota depending on the user of the account, and your server's storage space (Thankfully DigitalOcean is resizable!)_
        + [Screen Shot of New User Preferences](=http://grab.by/n7JY])

You just created your first email account on your new server!
- - -
###Using your new email!
After all of that, you finally get to use your email server for personal email, or professional!

1. Webmail Access
	* https://mail.yourdomain.com _This is the url for you to be able to access your e-mail from any web-enabled device!_
		+ Enter the Username and Password you just created
			- [Screen Shot of L/P For Webmain Access](http://grab.by/n7K0)
		+ You should now be welcomed by a beautiful roundcube webmail user interface.
			- [Screen Shot of Roundcube Webmail](http://grab.by/n7K2)
		+ From this point I typically like to test the send/receive functions.
			- Select Compose +, to create a new message to whomever you like.
				* Before sending, open your SSH client with the following command: `tail -f mail.log mail.err` for debugging!
				* I would also suggest doing the same by sending an email to your new account from another email address.

2. Mail Client
    * _Coming Soon_

[^1]: Coupon Code: SSDPOWER
