# Setup Homebridge-CBus and CGate on Raspberry Pi using MacOS

- You'll need a [Raspberry Pi](https://core-electronics.com.au/raspberry-pi-4-model-b-2gb.html) (recommend Raspberry Pi4 Model B 2GB ram).
- [Case for Raspberry Pi](https://core-electronics.com.au/pimoroni-aluminium-heatsink-case-for-raspberry-pi-4-black.html)
- [Power supply for Raspberry Pi4](https://core-electronics.com.au/raspberry-pi-4-official-power-supply-usb-c-5v-15w-black.html)
- microSD card (recommend [SanDisk Extreme Pro 64GB](https://www.officeworks.com.au/shop/officeworks/p/sandisk-extreme-pro-64gb-microsdxc-memory-card-sdsqxcu064))
- A local network connection (preferably wired).
- CBus network interface eg CNI (5500PCI RS232 or USB are also suitable with additional hardware and or software setup).  
- Your C-Bus project xml file.

Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)  
Write prebuilt Homebridge image to SD card  
Select Operating System >
Other specific-purpose OS > 
Home assistants and home automation > 
Homebridge > 
Offical Homebridge Raspberry Pi image  

Choose Storage (SD)

In Imager advanced settings 
- Would you like to prefill the wifi password from the system keychain? no
- Set hostname `homebridge`
- Enable SSH, use password authentication
- Set username `pi`
- Set password
- do not configure wireless LAN here
- Set locale settings
- Select Eject media when finished
- Save

Write   
1. Once write is finished remove SD card and insert into Raspberry Pi  
2. Connect LAN network cable to Raspberry Pi  
3. Connect power to boot Raspberry Pi   
4. Wait a minute or two for the Pi to boot.

5. Launch a browser to http://homebridge.local  create user `admin` and set a password  
Exit the browser for now. 

## Copy the project xml to the Raspberry Pi

6. Your C-bus network's "Tags file" is a file you'll find where-ever your C-bus network's current instance of "C-Gate" lives. It's essentially a dictionary file, matching the human-readable names you've given the inputs and outputs to the "Group Addresses" that C-Bus uses on the network.

On Windows, the default path for it is `C:\Clipsal\C-Gate2\tag\` and it will be called \<YourNetworkName\>.xml".
  
> Make sure the filename is the name of your network, because the script uses the filename to populate several places in the config where C-Gate and Homebridge need to know the network name.   
> Case matters and CGate is expecting UPPERCASE.xml
  
  7. Copy the project xml to the Pi, placing it in the /home/pi directory.  
  Here is an example how to move the file from the mac desktop to the Raspberry Pi using scp in the mac terminal    
  `scp /Users/<YourMacUserName>/Desktop/<YourProjectName>.xml pi@homebridge.local:/home/pi`

  
## Remote config via SSH

6. SSH to the Pi using terminal: 
```txt 
ssh pi@homebridge.local 
```

7. You should see something like this:
```txt
The authenticity of host '10.10.17.15 (10.10.17.15)' can't be established.
ECDSA key fingerprint is SHA256:Ty0Bw6IZqg1234567899006534456778sFKT6QakOZ5PdJk.
Are you sure you want to continue connecting (yes/no)?
```
8. Enter `yes` and press Return.
9. The response should look like this:
```txt
Warning: Permanently added '10.10.17.15' (ECDSA) to the list of known hosts.
pi@10.10.17.15's password:
```
10. Enter the password and press Return.
  

## Here's where all the software is updated and installed:

11. First let's make sure the Pi is all up-to-date:
```txt
sudo apt update && sudo apt upgrade -y
```

12. `sudo reboot`.

Your SSH session will end here. Wait for the Pi to reboot, sign back in again and continue. 
```txt
ssh pi@homebridge.local 
```
13. Manually install Homebridge-CBus plugin
```txt
sudo hb-shell
```
```txt
sudo hb-service add homebridge-cbus
```
```txt
exit
```

14. We need to install Subversion so we can download *just* the needed bits of the repo from GitHub:
```txt
sudo apt-get install subversion -y
```
15. This downloads the repo, dropping the structure into the home directory:
```txt
svn export https://github.com/greiginsydney/Homebridge-cbus-installer/trunk/code/ ~ --force
``` 

16. All the hard work is done by a script in the repo, but it needs to be made executable first:
```txt
sudo chmod +x setup.sh
```
17. Now run it! 
```txt
sudo -E -H ./setup.sh step1
```

> If any of the script's steps fail, the script will abort and on-screen info should reveal the component that failed. You can simply re-run the script at any time (up-arrow / return) and it will simply skip over those steps where no changes are required. There are a lot of moving parts in the Raspbian/Linux world, and sometimes a required server might be down or overloaded. Time-outs aren't uncommon, hence why simply wait and retry is a valid remediation action.

18. Having installed C-Gate, we now need to edit one of the security files to ensure authorised remote machines - like the one you'll run Toolkit from - are allowed to connect.

The script prompts the user, autofilling a guess at your local network, based upon the IP address of the Pi. Backspace if you want to edit this, or just press return if the value is correct and you want to whitelist that whole network:

```txt
=======================================
C-Gate won't let remote clients connect to it if they're not in the file
/usr/local/bin/cgate/config/access.txt.
Add your Admin machine's IP address here, or to whitelist an entire network
add it with '255' as the last octet. e.g.:
192.168.1.7 whitelists just the one machine, whereas
192.168.1.255 whitelists the whole 192.168.1.x network.
The more IPs you whitelist, the less secure C-Gate becomes.

Enter an IP or network address to allow/whitelist : 10.10.17.255
Enter an IP or network address to allow/whitelist :
```

This menu will loop, allowing you to enter extra IPs. Press Return on its own to break out of this loop.

19. If you overlooked copying the tags file in Step 7, or put it in the wrong location on the Pi, the script will exit:

```
Copy your tags file (i.e. "<ProjectName>.xml)" to /home/pi/ and then run Step2

pi@homebridge:~ $ 
```

20. Do not pass Go, etc. Return to Step 7, then manually run step2:

```txt
sudo -E ./setup.sh step2
```

21. Step 22 here picks up with the output from the script's "step2". (Yes, I probably need to rename them to make this less confusing.)

22. __If everything went OK after step 28, the script proceeds to run step2 automatically.__

23. The script will now move some of the supporting files from the repo to their final homes, and edit some of the default config in the Pi. 

24. It will output its progress to the screen. You'll see it's gone with "19P" which is my network name:

```txt
pi@raspberrypi:~ $ sudo -E ./setup.sh step2
>> Assuming project name = 19P, and setting C-Gate project.start & project.default values accordingly.
renamed 'homebridge.timer' -> '/etc/systemd/system/homebridge.timer'
Added "homebridge-cbus.CBus" to /var/lib/homebridge/config.json OK
Removed /etc/systemd/system/multi-user.target.wants/homebridge.service.
Created symlink /etc/systemd/system/multi-user.target.wants/homebridge.timer → /etc/systemd/system/homebridge.timer.
>> Exited step 2 OK.

Reboot now? [Y/n]:
```
Pressing Return or anything but Y/y will cause the Pi to reboot.

25. Once the Pi reboots, C-Gate and Homebridge will come up. It's this stage that populates your "my-platform.json" file.
Take a break for 10 minutes.

## Tweak the config
At this point you have an almost working Homebridge setup, but some tweaking and fine-tuning is required.

26. To use the script, re-run it with the new 'copy' switch:
```txt
sudo -E ./setup.sh copy
```

27. If the script exits with "Done" immediately, the mostly likely reason is that you've not given Homebridge enough time to populate the my-platform.json file. Wait a couple of minutes and try again.
```txt
sudo -E ./setup.sh copy
Done

The PIN to enter in your iDevice is 031-45-154

Restart Homebridge? [Y/n]:
```

28. Assuming the file has been populated OK, the script will now read through all the GAs in my-platform.json, and if they don't exist in config.json, prompt you one-by-one to Add them, Skip them, and where the "type" of channel is reported as unknown or was guessed incorrectly, Change them to one of the possible types.

29. Where the Group's details are correct, pressing Return will accept the default, Add, which is highlighted in green:
```txt
"type": "dimmer", "id": 18, "name": "Lounge room"
[A]dd, [s]kip, [C]hange & enable, [q]uit?
Added
```

30. All that are reported as "unknown" are highlighted in yellow, and pressing Return defaults to show the Change sub-menu. Choose the highlighted letter of the appropriate type and press Return:
```txt
"type": "unknown", "id": 21, "name": "Exhaust fan"
[a]dd, [s]kip, [C]hange & enable, [q]uit?
Change to:
[l]ight, s[w]itch, [d]immer, [s]hutter, [m]otion, s[e]curity, [t]rigger, [c]ontact: w
Changed to switch
```

31. Press "s" and return to Skip any spare, unknown or unwanted GAs, and then "q" once you're done:
```txt
"type": "light", "id": 22, "name": "Ceiling GPO"
[A]dd, [s]kip, [C]hange & enable, [q]uit? s
Skipped

"type": "unknown", "id": 23, "name": "Group 23"
[a]dd, [s]kip, [C]hange & enable, [q]uit? q
Done

The PIN to enter in your iDevice is 031-45-154

Restart Homebridge? [Y/n]:
```

32. Pressing return or anything but Y/y will restart Homebridge to pick up the new settings.

33. At this point you can turn to your iDevice, launch Home and select "Add Accessory".

34. Scan the QR code from homebridge.local webpage in your browser. 
You should be able to follow your nose from there.

35. You're free to repeat step 39 at any time. You won't be prompted for any of the GAs you added before, so if you want to change the "type" of an existing GA you'll need to do this by hand (`sudo nano /var/lib/homebridge/config.json`). Any GAs that you've recently added to the network or you may have subsequently decided to include can now be added to config.json just by responding to the prompts.

<br>

\- Greig.

