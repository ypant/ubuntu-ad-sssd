# ubuntu-ad-sssd

## Ubuntu - Active Directory Integration via SSSD

Tried to connect to Active Directory from Ubuntu by following the instructions specified at: https://help.ubuntu.com/lts/serverguide/sssd-ad.html. However encountered a few issues and attempting to specify the steps that helped me to resolve those issue (with some additional notes). For this document, I am using the terminology used in the original guide. 


Active Directory Details:

	Domain Name: MYUBUNTU.EXAMPLE.COM (DC=MYUBUNTU, DC=EXAMPLE, DC=COM)
	Domain Controller FQDN: dc.myubuntu.example.com (dc for domain controller)

Ubuntu Server details:
	Ubuntu server name which is attempting to join to AD: myserver.myubuntu.example.com

(Probably the example would have been a bit clearer if 'myubuntu' was replaced with mydomain and myserver with myubuntu.)

## Samba Configuration: 

for workgroup, I had to specify the first portion of my domain name in Capital:

[global]

workgroup = MYDOMAIN
client signing = yes
client use spnego = yes
kerberos method = secrets and keytab
realm = MYUBUNTU.EXAMPLE.COM
security = ads

## Using Active Directory as the DNS resolver

I had to change the /etc/resolv.conf file for ensuring that the name resolution works for SSSD (ubuntu-AD connectivity).  For this updated /etc/resolvconf/resolv.conf.d/head file and added the following line: 

nameserver 192.168.1.25

and restart the networking service: % service networking restart. 192.168.1.25 is the IP address of the domain controller. This updated the /etc/resolv.conf as follows:

nameserver 192.168.1.25
nameserver 192.168.1.30
nameserver 192.168.1.80


## Joining to the Active Directory

For joining to the AD, I had to change the order specified in the original document as follows:

	sudo systemctl restart ntp.service
	sudo systemctl restart smbd.service nmbd.service 

	sudo kinit Administrator
	sudo klist

	sudo systemctl start sssd.service 

	sudo net ads join -k

As is being documented else where in the internet, by letting the server sit overnight (may be a 10s of minutes or a couple of hours is be  sufficient), ubuntu machine was auto magically appeared in the AD domain. Then after that I was able to execute the getent passwd username command as documented in the original document. 

## Logging on to ubuntu instance

I had to ensure that the following line 

	PasswordAuthentication yes 

was NOT commented  /etc/ssh/sshd_config file. If update required, then restart ssh service by executing % service ssh restart 

Then I was able to connect to the instance using the following syntax:

% ssh ad-id@myubuntu.example.com@myserver.myubuntu.example.com




