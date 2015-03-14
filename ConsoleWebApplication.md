# The i-jetty Console Web Application #

This webapp is available packaged as an Android apk bundle from the Android Market. When run, this bundle will install the Console webapp into an existing i-jetty setup.

The webapp makes available the on-phone content such as Contacts, Call Logs, Music, Images, Video, System Settings.

## To Install ##

Download the i-Jetty Console Installer app from the Android Market. Click it to install. You will only need to do this once:

![https://i-jetty.googlecode.com/svn/trunk/images/ijetty-console-screenshot.png](https://i-jetty.googlecode.com/svn/trunk/images/ijetty-console-screenshot.png)

After successfully installing, you can restart i-jetty and the Console webapp will be deployed at http://localhost:8080/console.

## Running ##

Accessing the console webapp leads to the welcome page:

![https://i-jetty.googlecode.com/svn/trunk/images/ijetty-console-welcome.png](https://i-jetty.googlecode.com/svn/trunk/images/ijetty-console-welcome.png)

Accessing the webapp from:

**the on-phone browser**, the webapp is available at the URL http://localhost:8080/console. If you selected a different port number, then replace 8080 with that number. If you're using the SSL connector, instead use https://localhost:8443/console.

**a desktop browser**, the webapp will be available at the IP address of the phone on your local wireless network. For example, if your phone is at IP address 10.10.1.14, then access the console webapp at URL http://10.10.1.14:8080/console. Again, if you configured a different port number, then replace 8080 with that number in the URL.

**a device elsewhere on the net**, the webapp will probably not be accessible, as the mobile phone carriers sometimes block inbound connections. There are clever solutions to this problem, so please contact [Webtide](http://www.webtide.com) if you would like further information.

You can find out the IP addresses for your phone by looking at the i-jetty application. Underneath the buttons for Start, Stop, Configure and Download you'll see a list of IP addresses for the phone interfaces. One of them should correspond to an address on your local network. If not, then check the configuration of the phone allows you to connect to your local wifi setup.

## Security ##

In order to view any of the information from the device running i-jetty, you will need to supply a login/password.

The defaults are:
```
     login:    admin
     password: admin
```

To change the password, you will need to edit the file /sdcard/jetty/etc/realm.properties.

The format of this file is:

```
<username>: <password>[,<rolename> ...]
```

The password can be in the clear, obfuscated or checksummed. You can follow the instructions on http://wiki.eclipse.org/Jetty/Tutorial/Passwords to generate obfuscated or checksummed passwords.