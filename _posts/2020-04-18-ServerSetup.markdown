---
layout: post
title:  "Client Setup"
date:   2020-04-18 12:03:15 +0200
categories: jekyll macOS OpenDirectory Kerberos SSO OpenLDAP Fritzbox
---

# macOS with an OpenDirectory LDAP and Kerberos KDC

I have an OpenLDAP and MIT krb5 Kerberos KDC running on my DSL router (FritzBox 7390). On the LDAP I've created a OpenDirectory schema so that the router - since it is running 24/7 - can mimic a kind of minimal macOS Server. This way I can log on all macs in my home using the same user and even can benefit from single sign on.

## Bind to Directory Server
Let's define some variables:

```
ROOT_CA_FILE="/Volumes/Kiste/Home-IT/Admin/Berkersheim/SSL/Berkersheim_Root_CA.cer"
DIRECTORY_SERVER="fritz.box"
```


Install our CA's root certificate so that we can use SSL when binding to our directory server.

`sudo security add-trusted-cert -d -r trustRoot -p ssl -p basic -k /Library/Keychains/System.keychain "$ROOT_CA_FILE"`

Add our directory server:

	# remove it in case you're already bound
	sudo dsconfigldap -r "$DIRECTORY_SERVER" 
	# add it
	sudo dsconfigldap -vsemgx -a "$DIRECTORY_SERVER" 


## Configure Kerberos

Again some variables:

	REALM="BERKERSHEIM"
	KDC_SERVER="fritz.box"
	DOMAIN=".fritz.box"

Create the /etc/krb5.conf file. Attention it will overwrite an existing file.

```
sudo tee /etc/krb5.conf << EOF

[libdefaults]
  default_realm = $REALM

[realms]
$REALM= {
   kdc = $KDC_SERVER
   admin_server = $KDC_SERVER
   kpasswd_server = $KDC_SERVER
}

[domain_realm]
   .$DOMAIN = $REALM
   # Bonjour
   .local = $REALM

EOF
```

Alternatively create a corresponding record `cn=KerberosClient,cn=config,o=example` with the *apple-xmlplist* attribute:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>edu.mit.kerberos</key>
	<dict>
		<key>domain_realm</key>
		<dict>
			<key>.local</key>
			<string>BERKERSHEIM</string>
			<key>.example.com</key>
			<string>BERKERSHEIM</string>
			<key>example.com</key>
			<string>BERKERSHEIM</string>
		</dict>
		<key>libdefaults</key>
		<dict>
			<key>default_realm</key>
			<string>REALM</string>
		</dict>
		<key>realms</key>
		<dict>
			<key>REALM</key>
			<dict>
				<key>KADM_List</key>
				<array>
					<string>kdc.example.com</string>
				</array>
				<key>KDC_List</key>
				<array>
					<string>kdc.example.com</string>
				</array>
			</dict>
		</dict>
	</dict>
	<key>generationID</key>
	<integer>544631620</integer>
</dict>
</plist>

```

Make sure the user you're testing the SSO features of macOS with is able to obtain a valid kerberos ticket.

 Issue `klist` and make sure you see something like
```
Credentials cache: API:CF42DDF5-9537-4952-AFF9-E123456789
        Principal: matteo@YOUR_REALM

  Issued                Expires               Principal
Apr 18 22:18:21 2020  Apr 19 08:18:18 2020  krbtgt/YOUR_REALM@YOUR_REALM
```

It it does not try to obtain one using `kinit` or `kinit username@YOUR_REALM`.


## Kerberize Services for SSO (Single Sign On)

I do recommend that your ldap directory exposes a KerberosKDC config record such as

```
dn: cn=KerberosKDC,cn=config,o=example
changetype: modify
add: apple-config-realname
apple-config-realname: YOUR_REALM
```

If this entry comes first in */Search* (check with Directory Utility) then SMB file sharing and VNC screensharing will recognize it and grant access if you have a valid kerberios ticket for that realm.

You *could* probably also replace the one in */Local/Default/Config*, by modifying the *apple-config-realname* which by default contains the local KDC. I've used this method before on earlier macOS versions.

You need to create principals in your KDC and export them to your client's keytab. I use the following script - let's name it *add_client_computer_principals.sh* - on my KDC master to do that. Note that I use MIT krb5 on my KDC master so for heimdal you must modify the script accordingly.


```
#!/bin/sh

FQNAME="$1"
KEYTAB="$2"

if [ "$#" -ne 2 ]
then
	echo "Usage $0 fqname keytab"
	exit
fi

for service in "host" "cifs" "afpserver" "vnc" "nfs"
do
	kadmin.local ank -randkey  "$service/$FQNAME"
	kadmin.local ktadd -k "$KEYTAB" "$service/$FQNAME"
done
```

To create principals for a new client mac with hostname *mymac.fritz.box* and Bonjour-Name *MyMac* (please use the technical name without whitespace et cetera) *MyMac* I'd execute:

	add_client_computer_principals.sh mymac.fritz.box mymac.keytab
	add_client_computer_principals.sh MyMac mymac.keytab

which will add principals for ssh, smb (cifs), afp and nfs to the KDC and exports them to a file *mymac.keytab*. The file must than be added to *MyMac*'s `/etc/krb5.keytab` so that the individual services will find the corresponding entry. So I transfer the file from the KDC to *MyMac* and perform:

	sudo cp /etc/krb5.keytab /etc/krb5.keytab.bak
	sudo ktutil copy mymac.keytab /etc/krb5.keytab

You can list the keytab entries with

	sudo ktutil -k krb5.keytab list

It should list the pre-existing entries, that are associated with the mac's local kerberos realm, and the newly created entries of your network's realm.

If you now try to connect (you do not necessarily have to succeed) to a kerberized service like screensharing `klist` should also list something like

	Apr 18 22:18:23 2020  Apr 19 08:18:18 2020  vnc/hostname.example.com@YOUR_REALM

### SMB, Screensharing(VNC) and AFP

If your directory has the KerberosKDC config (recommended see above) then you should already be able to access SMB, AFP shares and VNC using SSO. If VNC seems not to work you may need to reboot (maybe also your client as well) and/or restart the service via `sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist`. I don't know why, but It did not work initially.

If you do not or maybe can't have the KerberosKDC entry in your directory (or you do not have a directory) - but you still have a KDC I believe the following will enable SSO at least for AFP and SMB, although I strongly advice against it:

**Again: You do not need the following, if you have the KerberosKDC in your /Search path (see above)!**

*SMB:*

```
defaults write /Library/Preferences/SystemConfiguration/com.apple.smb.server.plist LocalKerberosRealm "YOUR_REALM"
launchctl unload /System/Library/LaunchDaemons/com.apple.smbd.plist
launchctl load /System/Library/LaunchDaemons/com.apple.smbd.plist
```


*AFP:*

```
defaults write /Library/Preferences/com.apple.AppleFileServer kerberosPrincipal "afpserver/mymac.example.com@YOURREALM"
```

