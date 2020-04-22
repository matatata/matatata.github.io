---
layout: post
title:  "Darwin CalendarServer"
date:   2020-04-22 12:03:15 +0200
categories: jekyll macOS CalendarServer
---

# CalendarServer on macOS

This is tested on High-Sierra but is known to work on El Capitan as well. On the latter It might be necessary to install `pip install pyobjc`, but I could be wrong. On High-Sierra it was not needed.

https://developer.apple.com/support/downloads/macOS-Server-Service-Migration-Guide.pdf is a good start, but it has some flaws. Here's how I have successfully built and have been using it:

Use homebrew and install
`brew install libevent memcached postgresql@10`

install pip
`sudo python -m ensurepip`

Become the calendarserver user:
`sudo -iu calendarserver`
`cd /Users/calendarserver`

Clone my fork that fixes a problem, where the installed openssl is not used. See https://github.com/matatata/ccs-calendarserver/commit/e317507de7e6765c5feb77deca40cd9266ea7b46):

`git clone https://github.com/matatata/ccs-calendarserver.git`
`cd ccs-calendarserver` 


`brew info` is your friend, because it will give important information about how to use them when compiling software:

You'll need at least to do this, before building/packaging calendar server:

```
export PATH="/usr/local/bin:$PATH"
export PATH="/usr/local/opt/postgresql@10/bin:$PATH"
export PATH="/usr/local/opt/openssl/bin:$PATH"
export USE_OPENSSL=1
export LDFLAGS="-L/usr/local/opt/openssl/lib"
export CPPFLAGS="-I/usr/local/opt/openssl/include"
```

Build it
`./bin/package "$HOME/CalendarServer"`

It should NOT build openssl, openldap or posgresqll! It should succeed without errors.

### Quick test

Switch to its virtual python environment
`source "$HOME/CalendarServer/virtualenv/bin/activate"`

You should be able to run the develop version:
`./bin/run -n` and connect to http://server:8008 with a browser. There are several users configured `admin:admin` or `user01:user01`. To do a quick test in Calendar.app on macOS create a 'Manual' account and specify:

```
User: user01
Pass: user01
Server: http://server:8008
```

It should all work then. But for production follow the migration guide... But beware that there are small errors and things that are worth to know:



Step 14: should read `mkdir conf run logs certs`

Step 16: should read `sudo cp org.calendarserver.plist /Library/LaunchDaemons`
Append `/usr/local/bin:/usr/local/opt/postgresql@10/bin` to the PATH variable in the LaundDaemon's org.calendard.plist

## Config
I recomment to protect the config file as it can contain passwords:
`chmod 600 /Users/calendarserver/CalendarServer/conf/calendarserver.plist`

I recommend using an XML accounts.xml file. For that uncomment or the section:

```
<key>DirectoryService</key>
    <dict>
      <key>type</key>
      <string>xml</string>

      <key>params</key>
      <dict>
        <key>xmlFile</key>
        <string>/Users/calendarserver/CalendarServer/conf/accounts.xml</string>
      </dict>
    </dict>
```

The *accounts.xml* file can look like this:

```
<!DOCTYPE accounts SYSTEM "accounts.dtd">
<directory realm="server.example.com">
<!-- define one or more records. Create your own uid and gid using the uuidgen command or reuse existing ids of your system -->
<record>
    <uid>F92AD1D5-94D2-4C0B-94EE-6E6B29EFF478</uid>
    <guid>C2DFC983-DA48-4E44-8866-581CD973A6B8</guid>
    <short-name>matteo</short-name>
    <password>secret</password>
    <full-name>Matteo Ceruti</full-name>
    <email>matteo@example.com</email>
</record>
</directory>

```

Again protect the file:
`chmod 600 /Users/calendarserver/CalendarServer/conf/accounts.xml`


If you use an opendirectory/ldap sercver that does not support Digest, then disable it in the section of the `/Users/calendarserver/CalendarServer/conf/calendarserver.plist` config.

I do recommend to get SSL working first, by creating your certificates. Note that I do not use a self-signed certificate. For that you can always look at the development config `/Users/calendarserver/ccs-calendarserver/conf/caldavd-test.plist`. I generally recommend consulting it or the other files in the same directory.

### SSL
Regarding the calendarserver.chain.pem file (if you use it): It must contain the server, intermediate and root certificate. This order works for me - so the pem files starts with the server certificate, followed by the intermediate and the root certificate. If you use the xca application, the xou can create such a chain file by exporting the server certificate as certificate chain.

## Bonjour
If you have an AirPort router or an AppleTV then they can serve as sleep proxy servers and thus wake your mac from sleep if necessary. For that you'll want to register the calendarserver with it with a LaunchDaemon:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>KeepAlive</key>
        <dict>
                <key>OtherJobEnabled</key>
                <dict>
                        <key>org.calendarserver</key>
                        <true/>
                </dict>
        </dict>
        <key>Label</key>
        <string>org.calendarserver.bonjour</string>
        <key>ProgramArguments</key>
        <array>
                <string>dns-sd</string>
                <string>-R</string>
                <string>http</string>
                <string>_http._tcp</string>
                <string>local</string>
                <string>8443</string>
        </array>
        <key>RunAtLoad</key>
        <false/>
</dict>
</plist>
```

Create a file with the contents above in `/Library/LaunchDaemons/org.calendarserver.bonjour.plist` and load it `sudo launchctl load -w /Library/LaunchDaemons/org.calendarserver.bonjour.plist` .
