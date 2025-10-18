# Message Of The Day
Each time a user logs in to the server a message will be shown with some info about the server.
In Debian is already a MOTD (Message Of The Day) included at /etc/motd but we’ll make it dynamic and display it.
This will be applied to each server!

## Server Description
First we make an extra file /etc/server-description with the description for that server. 
- csrv1: Control plane
- wsrv1: Worker node 1
- wsrv2: Worker node 2
- wsrv3: Worker node 3

```bash
vi /etc/server-description
```
content (adjust for each server):
```bash
Control plane
```
Save and exit the file

## MOTD Script
To show the message we will make a bash-script that will show the message when a user logs in
The script file 99-custom is placed in the folder /etc/update-motd.d
```bash
vi /etc/update-motd.d/99-custom
```
Paste the following content:
```bash
#!/bin/bash
#
source /etc/os-release
USER=$(logname)
HOME=$(eval echo "~$USER")
echo -e "
*********************************************
*	  KEUTGENS INDUSTRIE 4.0 PLATFORM
*********************************************
* Server 	    : $(hostname).$(dnsdomainname)
* Role 		    : $(cat /etc/server-description)
* IP-Address 	: $(hostname -I)
* Uptime 	    : $(uptime -p)
* OS          : $PRETTY_NAME
*
* You are logged in as 	: $USER
* In directory 	  	    : $HOME
* 
*********************************************
"
```
Save and exit the file.
Now make sure the file can be executed
```bash
chmod a+rx /etc/update-motd.d/99-custom
```
Test the script output manually:
```bash
bash /etc/update-motd.d/99-custom
```
If everything went well, next time you login, the above message will be shown
