# ONVIF and WIFI Information NightOwl WNIP2 Camera Series
After searching to find a way to use the WNIP2 Wifi IP Cameras from NightOwl without having to use the packaged WNIP2 NVR (which, is the weakest part of the packaged solution NightOwl sells at discount box stores like Sams Club and Costco) I was looking at trashing the cameras and buying reputable standaling IP cameras that would support ONVIF and BlueIris, etc.

I made one last attempt at trying to prevent these really decent cameras from becoming e-waste.  

Heres what I learned:

## Its a package, sorta.
- These cameras WILL work as standalone units, despite they were packaged with an Paired-NVR.
- NightOwl customer support will not help you not use the paired-NVR.
- You still initially need a NightOwl NVR to temperaily configure the devices
- Once the devices are configured, you never need the paired NVR again.
- There is a model fo camera that is standalone that will natively work.  They are add-on cameras never that were not pre-paried with the NVR.  The pre-paired NVR cameras are locked to the NVR.
- The NVR itself is fully encrypted, don't bother taking it apart.  The disk is not readable without a private key that I couldn't find anywhere.

## The hack.
If you want to use these cameras without the paired NVR, you'll have some work to do to capture network traffic and to design a network that can live without the paired NVR.

### First
- Get all your cameras working as intended by Night Owl. They will operate on their own WIFI SSID.  The  Wifi uses WPA2-PSK.  Don't bother trying to capture the 4-way handshake, you won't crack the PSK.  
- Do yourself a favor and disable the time-stamp overlay on the cameras.  The hack below will allow the cameras to operate without the bundled NVR, however the camears timestamps will revert to 1/1/2000 withotu the NVR being on the network.  Best to just remove the timestamp and let your custom standalong  NVR replacement add timestamps as needed.  NOTE: its liekly that the WinNVR was operating as a NTP server for the cameras.  In future work it might be possible to add an NTP service for the cameras on the replaced-NVR-gateway and see if they update.
- After installing the cameras, launch the NightOwl WinIP client software on your computer and login.  Open one of the live-cameras.  The NVR and software uses beaconing to punch wholes in firewalls.  Its annoying.  But whatever.  This also adds your computer's IP into the Multi-cast channel.
- Make sure your computer and NVR are plugged into the same networks witch. 
- Run wireshark on the same system that has the WINIP Widnows software client software installed.  Begin capturing all traffic in promiscuios mode on the wired interface.
- Login to the NVR using a keyboard plugged into the NVR (have the NVR plugged into a TV) 
- Open the settings panel on the NVR, goto cameras. Attempt to "add-all" cameras.  No new cameras are found, this is ok.    
- Stop the wireshark capture.
- Investigate wireshark capture.  You will have captured a multicast packet from the WinNVR IP to the Multicast network that contains the configuration information for all of the cameras and listening devices assigned to the multicast channel.

The capture will appear similar to below
```
Client-ID:XXXXXXXXXXXXXXXXXXX
Content-Type:application/json
X-Session-Id:1
X-Content-Checksum:#######REDACTED#####
Content-Length:1019

{
"Ver" : "1.1",
"Nonce" :  "", 
"Device-ID" : "", 
"Device-Model" : "WNVR-WNIP2",
"Device-Type" : "WNVR-WNIP2",
"Esee-ID" : "#######REDACTED#####",
"Software-Version" : "WNVR-WNIP2_#######REDACTED#####", 
"Wired" : [
{
"DHCP" : true,
"Connected" : true,
"IP" : "#######REDACTED#####",
"Netmask" : "255.255.255.0",
"Gateway" : "#######REDACTED#####",
"MAC" : "#######REDACTED#####"
}
],
"Wireless" : [
{
"DHCP" : false,
"Connected" : true,
"IP" : "172.#######REDACTED#####.1",
"Netmask" : "255.255.255.0",
"Gateway" : "#######REDACTED#####",
"MAC" : "#######REDACTED#####",
"Mode" : "accessPoint",
"ApMode" : {
"Channel": 11,
"Essid" : "NOPWNVR-#######REDACTED#####",
"Psk" : "#######REDACTED#####"
}
}
],
"Channel-Info" : [
{"id": 0,"Stream-Cnt": 2},
{"id": 1,"Stream-Cnt": 2},
{"id": 2,"Stream-Cnt": 2},
{"id": 3,"Stream-Cnt": 2},
{"id": 4,"Stream-Cnt": 2},
{"id": 5,"Stream-Cnt": 2},
{"id": 6,"Stream-Cnt": 2},
{"id": 7,"Stream-Cnt": 2},
{"id": 8,"Stream-Cnt": 2},
{"id": 9,"Stream-Cnt": 2}
],
"Channel-Cnt": 10,
"Capabilities" : {
"Http-Port" : 80,
"MaxHardDiskDrivers" : 1,
"MaxTFCards" : 0
}}
```

## The details
The captured packet will show the NVR's configuration and the configuration that the cameras use to connect to the NVR via its dedicated SSID.  The SSID and PSK are presented in plaintext in the packet.

## Next Steps - Network Design
- There are lots of ways of doing this.  The premise at this point is to remove the WinNVR device.  Throw it in a closet and never look at it again.  Add a wireless bridge in your network that  drops to your network and acts in Access-Point mode.  It should be configured with the identified SSID and PSK.
- Add a new network on your firewall/router on a new vlan. The IP should be the same as the old WinNVR device.  This is the stand-in replacement gateway that the WinNVR device used to be.  Tag the VLAN accordingly as untagged on the AP that is acting-as the WinNVR gateway and tagged on your firewall/router.  
- Add routes in your firewall.
- Your LAN devices can now route to all of the cameras that were on the isolated network.  The cameras will use your router as their gateway and the cameras are still technicaly isolated to give you control over their internet access.  (They shouldn't be reaching out to the internet anyhow).

## Alternatiev configuration
- Add a wireless card to where-ever you have your NVR installed.  Use the identified configuration to connect to the camera's network.  This can be done even if the NVR is still on the network.  Once connected to the same WIFI network as the cameras, you can access them directly via IP.

## Cameras capability and OnVIF
- The cameras support onvif out of the box.  
- The ONVIF username is admin and the password is admin
- If using BlueIris
  - Make: Generic/ONvIF
  - Model: *RTSP H.264/H.265/MJPG/MPEG4
  - Cam#: 1
  - Media/video rtsp port: 554
  - Discovery/ONVIF port: 8089
  - Main: Profile_000: /ch0_0.264
  - Audio: 64 kbps G.711 u-law
  - ONVIF source: vstoken0: V_SRC_000
  - Enable "Get ONVIF trigger Events" to allow the cameras to self-trigger.  Leave disabled if you want Blue-Iris to trigger.
  - Enable "Send RTSP keep-alives"
  - Enable "Use RTSP/stream timecode"
  - The cameras support two-way audio, so enable "Send RTSP back-channel for talk support" if you want to use Blue-Iris for two way audio.

## Just works.
As far as I can tell, only the WinNVR device ever reached out to the internet for "zero-config" configuration and punching wholes in your firewall.  Once the cameras are configured, the configuration will survive reboots, even when the WinNVR device is not plugged in.  

## Future work.
I'm a little irrated it was this painful.  Ideally I'd rather have taken the origianl configuration broadcast packet and maniuplated it to be a sane configuration that we would select to match our network preference.  Having a hidden SSID is fine, but it still leakes the name and will self-idenitfy that its a nightowl camera network.  Eventually I might get around to building a conifguration tool that will natively work with the cameras.  But... I've got the camears working now without the nonsense.  Today is not that day.

## Could Night Owl have done better?
- The WinNVR includes a 1TB encrypted drive.  Querying a video from yesterday will timeout of the drive is near capacity.  Encryption, video recorders, and spinning disks aren't a match.  Replacing the included drive with an SSD didn't work. I probably could have dd'd the original drive and got it working, but... the entire WinNVR device is slow and annoying anyway.  Not to mention I don't apprecaite it punching wholes in my firewall for the conviencie of having  cameras access on my phone.  I can do that securely myself.
- The configuration multicast packet is plaintext.  Why they didn't add a hard-coded encryption key on the cameras is beyond me.  That would have truly made these devices e-waste.  
- Seriously, if a customer asks you if they can use the cameras without the pre-paired NVR, be honest.
- Then again, their support team seemed clueless.  Since the WinNVR's connection to the network wasn't wireless (wifi is only used for the isoalted communication to the cameras), the support team refused to ever discuss anything about wireless.  They won't give up the PSK because as far as their documents say, a customer would never need it, so don't give it to them.

Anyway, now you all know.  One of the BlueIris forums will likely pick up on this and take it from where I left off.  Sad to see that these cameras are going to end up in dumpsters because the developer made them so dificult to use outside of their walled-garden.



