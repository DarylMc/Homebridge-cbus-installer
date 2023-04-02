
[<img src="https://user-images.githubusercontent.com/28332686/229351337-3717be4c-e0b8-4a7b-aa8f-677401fc2fa9.jpg">](https://youtu.be/x28SieXtzN8)


# Setup Homebridge-CBus and CGate on Raspberry Pi using MacOS  Tested on M1 MacBookPro
 
- You will need a [Raspberry Pi](https://core-electronics.com.au/raspberry-pi-4-model-b-2gb.html) (recommend Raspberry Pi4 Model B 2GB ram).
- [Case for Raspberry Pi](https://core-electronics.com.au/pimoroni-aluminium-heatsink-case-for-raspberry-pi-4-black.html)
- [Power supply for Raspberry Pi4](https://core-electronics.com.au/raspberry-pi-4-official-power-supply-usb-c-5v-15w-black.html)
- microSD card (recommend [SanDisk Extreme Pro 64GB](https://www.officeworks.com.au/shop/officeworks/p/sandisk-extreme-pro-64gb-microsdxc-memory-card-sdsqxcu064))
- A local network connection (preferably wired).
- CBus network interface eg CNI (5500PCI RS232 or USB are also suitable with additional hardware and or software setup).  
- Your C-Bus project xml file.

## 1. Download and install 
[Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager/releases/tag/v1.7.3) 
version 1.7.3 recommended as of April 2023  
https://github.com/raspberrypi/rpi-imager/releases/download/v1.7.3/Raspberry.Pi.Imager.1.7.3.dmg

## 2. Use Imager to write Official Homebridge image to SD card    
Select Operating System >
Other specific-purpose OS > 
Home assistants and home automation > 
Homebridge > 
Offical Homebridge Raspberry Pi image  

## 3. Choose storage
- Select SD card 

## 4. Imager advanced options  
- Would you like to prefill the wifi password from the system keychain? no
- Set hostname `homebridge`
- Enable SSH, use password authentication
- Set username `pi`
- Set password
- do not configure wireless LAN here
- Set locale settings
- Select Eject media when finished
- Save

## 5. Write 
- Select write
- Once write and verification is finished remove SD card and insert into Raspberry Pi  
- Connect Raspberry Pi to LAN
- Connect power to boot Raspberry Pi   
- Wait a few minutes since it's the raspberry Pi's first boot.

- Launch a browser and navigate to http://homebridge.local  create user `admin` and set a password  
- Exit the browser for now. 

## 6. Update the Raspberry Pi

SSH to the Pi using mac terminal 
```txt 
ssh pi@homebridge.local 
```
Update the system
```txt
sudo apt update && sudo apt upgrade -y
```
Reboot
```txt
sudo reboot
```
 
## 7. Copy the project xml to the Raspberry Pi

Copy the project xml to the Pi, placing it in the /home/pi directory.  
  Here is an example how to move the file from the mac desktop to the Raspberry Pi using scp in the mac terminal    
  ```txt 
  scp /Users/<YourMacUserName>/Desktop/<YourProjectName>.xml pi@homebridge.local:/home/pi
  ```

## 8. Install CBus plugin and script components 

SSH to the Raspberry Pi
```txt
ssh pi@homebridge.local 
```
Manually install Homebridge-CBus plugin
```txt
sudo hb-shell
```
```txt
sudo hb-service add homebridge-cbus
```
```txt
exit
```

Install Subversion
```txt
sudo apt install subversion -y
```
This downloads the repo, dropping the structure into the home directory:
```txt
svn export https://github.com/greiginsydney/Homebridge-cbus-installer/trunk/code/ ~ --force
``` 

Make setup.sh executable
```txt
sudo chmod +x setup.sh
```

## 9. Run the script step1

```txt
sudo -E -H ./setup.sh step1
```

Reboot once completed 
```txt
sudo reboot
```

Once the Pi reboots, C-Gate and Homebridge will come up. It's this stage that populates your "my-platform.json" file.
Take a break for 10 minutes.

## 10. Re-run the script with the copy switch
```txt
sudo -E ./setup.sh copy
```

Assuming the file has been populated OK, the script will now read through all the GAs in my-platform.json, and if they don't exist in config.json, prompt you one-by-one to Add them, Skip them, and where the "type" of channel is reported as unknown or was guessed incorrectly, Change them to one of the possible types.

Where the Group's details are correct, pressing Return will accept the default, Add, which is highlighted in green:
```txt
"type": "dimmer", "id": 18, "name": "Lounge room"
[A]dd, [s]kip, [C]hange & enable, [q]uit?
Added
```

All that are reported as "unknown" are highlighted in yellow, and pressing Return defaults to show the Change sub-menu. Choose the highlighted letter of the appropriate type and press Return:
```txt
"type": "unknown", "id": 21, "name": "Exhaust fan"
[a]dd, [s]kip, [C]hange & enable, [q]uit?
Change to:
[l]ight, s[w]itch, [d]immer, [s]hutter, [m]otion, s[e]curity, [t]rigger, [c]ontact: w
Changed to switch
```

Press "s" and return to Skip any spare, unknown or unwanted GAs, and then "q" once you're done:
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

Pressing return or anything but Y/y will restart Homebridge to pick up the new settings.

## 11. Add accessory to Home

At this point you can turn to your iDevice, launch Home and select "Add Accessory".

Scan the QR code from http://homebridge.local webpage in your browser. 
You should be able to follow your nose from there.

## 12. Additional setup for USB serial CBus Interface

```txt
git clone https://github.com/nutechsoftware/ser2sock.git && sudo mv ser2sock /usr/local/bin && cd /usr/local/bin/ser2sock && chown -R pi:pi . && mv config.h.in config.h && cc -o ser2sock ser2sock.c && sudo nano /etc/systemd/system/ser2sock.service
```
Paste the following into nano
```txt
[Unit]
Description=ser2sock

[Service]
ExecStart=/usr/local/bin/ser2sock/ser2sock -p 10001 -s /dev/ttyUSB0 -b 9600
Restart=always
User=root
Group=root
Environment=PATH=/usr/bin:/usr/local/bin
Environment=NODE_ENV=production
WorkingDirectory=/usr/local/bin/ser2sock/

[Install]
WantedBy=multi-user.target
``` 
Ctl x y enter to save 

Enable and Start ser2sock service 
```txt
sudo systemctl enable ser2sock.service && sudo systemctl start ser2sock.service
``` 
Restart cgate service 
```txt
sudo systemctl restart cgate.service
```

## 13. Fix error connecting to remote CGate
Allow TLS V1  
sudo nano /usr/lib/jvm/java-8-openjdk-armhf/jre/lib/security/java.security   
Go To Line (control shift underscore) 704,40   
Remove TLSv1 in the line "jdk.tls.disabledAlgorithms="  
Ctl x y enter to save


<br>
