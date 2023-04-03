# SmartThings to Home Assistant Migration notes:
## Overview
In February 2023, I migrated our house from Samsung SmartThings to Home Assistant for orchestraion of IoT devices.

## Networking
Default setup of DHCP, with router enforcing a static address. Attempts to force static address from PI/HA cause "homeassistant.local" to no longer resolve.

External access via DynDNS. Attempted to utilize Google Domains dyndns support, but built in HA add on do not support it. UDM states it supports it, but appears broken in interface as of 4/1/23. Not shown as a valid provider option. Additionally unclear how this works with primary T-Mobile 5G home service as we have only a dedicated IPv6 address. Revisit at a later date.

## Devices
### Home Assistant Yellow PoE w/ CM4 compute module
Flashed HA specific distribution from Home Assistant site through USB

### Zooz 700 Series Z-Wave Plus S2 USB
Flashed to latest firmware via HA web interface. Firmware from Zooz website. Installed Z-Wave JS home assistant integration to add devices. To add devices in secure mode requires a pin, for switches this means pulling the face plate.

### Venstar Thermostat
The instructions in the book are fairly unclear. To enable authentication on the local API, from the thermostate you must switch to HTTPS, then hit the back key to get a username/password option. This never appears with the http option. Once set you can toggle between the two and then enable the API. HA detected after and the Venstart Integration was installed.

### Unifi Dream Machine (UDM)
Create a local user account on UDM with "view options" this enables showing status and the same account may be used for Unifi Protect.

### Unifi Protect 
Use same account for UDM to view the protect camera information and get camera detections.

### Sonoff RF Bridge
The Bridge was pre-flashed with Tasmota and had pre-learned buttons for our Sauana lighting. The instructions and videos present around the web mostly show them used for sensors not sending.

- Make sure HA account has "Advanced mode enabled"
- Create an account for an MQTT broker to run as, and another account to use with MQTT clients.
- Under Add-On install MQTT Broker and set it to use the broker account.
- Add integrations for Tasmota and MQTT (set server to local host and use the MQTT broker cred above.
- In Tasmota on device enable MQTT with client credentials and change topic it publishes to so that it is descriptive (I used: tasmota_rf-bridge)
- At this point you should be able to see status from the device in HA, but not trigger/send.
- Using File Editor in HA (installed Add-on) add a second to MQTT buttons that will send the proper presaved key command when pressed.
```
mqtt:
    button:
    # RF Bridge - Sauna Lights On/Off
      - name: "Sauna Lights On"
        command_topic: "cmnd/tasmota_rf-bridge/RfKey1"
        availability_topic: "tele/tasmota_rf-bridge/LWT"
        payload_available: "Online"
        payload_not_available: "Offline"
        payload_press: ""
        retain: false
        qos: 2
    # RF Bridge - Sauna Lights On/Off
      - name: "Sauna Lights Off"
        command_topic: "cmnd/tasmota_rf-bridge/RfKey2"
        availability_topic: "tele/tasmota_rf-bridge/LWT"
        payload_available: "Online"
        payload_not_available: "Offline"
        payload_press: ""
        retain: false
        qos: 2
  ```
  - In developer tools, reload the MQTT custom configured.
  - Add buttons to a dashboard and test.
  - You can view the console from Tasmota RF Bridge web interface to see if it recieves the commands.

The lights controller on the recieving end does not send status, so we can't do much better out of the box. We do have an option of storing the state in HA. The lighting controller loses state and defaults to on in power outage, to work around we send an off signal every day at 3am.

### Sonoff Wifi Smart Switch w/ Temp/Humidity sensor
- See Tasmota Configuration as above for MQTT, but it shouls register the switch automatically with proper status.

### Jasco/GE Smart Toggle
For new switches which were not added to SmartThings previously, press the up switch three times fast to put it in discovery mode. In HA under the Z-Wave JS add on discover and add the device. It will prompt you for start of the pin, which is on each light switch itself. This required removing some installed wall plates. Without the pin you can only use older security modes.

For switches attached to SmartThings, you must send an unlearn command first then follow the above procedure. Details to follow.

### Jasco/GE Smart Dimmer
- As above. Has the ability to set the light behind the switch state through configuration.

### Relay 3 switch

### Fire HD7 Wall Tablets

### Athom WSLED Light controllers



