# rpi_network_script
The Raspberry Pi tends to drop network connection (especially wireless wifi) rather fast, which is a real pain when you're trying to do anything that has the RPi running constantly from a remote location (like our RaspEye does).  

However, it's possible to detect wifi connection loss and perform upon it. It's easiest to just do a full system reboot.  

checkwifi.sh. 
Store this script in /usr/local/bin/checkwifi.sh.  

```
ping -c4 192.168.1.1 > /dev/null. 
 
if [ $? != 0 ] 
then
  sudo /sbin/shutdown -r now
fi
```

Change the IP on the first line to the IP of your router, or some other device on your network that you can assume will be always online.

First step is to ping your IP.

On line three, the $? represents the exit code of the previous command, in this case the ping.

If it failed (not 0), the script assumes something's wrong with the wireless connection and just reboots.

Make sure the script has the correct permissions to run (thanks for the tip Jason Reibelt)

`sudo chmod 775 /usr/local/bin/checkwifi.sh`

crontab
SSH into the Raspberry Pi and open up the crontab editor by typing `crontab -e`.  

Add the following line:

`*/5 * * * * /usr/bin/sudo -H /usr/local/bin/checkwifi.sh >> /dev/null 2>&1`
This runs the script we wrote every 5 minutes as sudo (so you have permission to do the shutdown command), writing its output to /dev/null so it won't clog your syslog.  

Done.  
Exit the crontab editor, reboot your Raspberry Pi (just to make sure) and from now on it'll reboot when it drops connection. Combine this with running your scripts on boot and you have a powerful, standalone system running.  

Just restart network instead of reboot  
Because many things could have gone wrong with a network loss and our application itself needs to adapt we prefer a full reboot. But after alot of people were shouting "that's just too much", you can also try to just restart the wireless connection:  

```
ping -c4 192.168.1.1 > /dev/null
 
if [ $? != 0 ] 
then
  echo "No network connection, restarting wlan0"
  /sbin/ifdown 'wlan0'
  sleep 5
  /sbin/ifup --force 'wlan0'
fi
```
