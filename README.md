# REMODO
# Kodi

Kodi is a free and open source media player that is designed to look great on big screen TV but is also fine a small screen. LibreELEC is GNU/Linux distribution for embedded devices that provides Kodi. It is perfect choice for Raspberry Pi. This chapter explains how to:

* Install LibreELEC on Raspberry Pi 4
* Common keyboard shortcuts in Kodi
* Install TED Talks plugin
* Configure custom keyboard shortcut to launch TED Talks plugin

## Required Hardware

The following hardware is required:

* Raspberry Pi 4
* USB keyboard (recommended Raspberry Pi official keyboard)
* microSD card (recommended class 10, 16GB or more)
* HDMI monitor with appropriate cable
* Ethernet cable to connect Raspberry Pi to the LAN and the Internet

## Install LibreELEC

Follow the steps below to download and install LibreELEC with Kodi on Raspberry Pi 4:

* [Download and flash LibreELEC for Raspberry Pi 4](https://libreelec.tv/raspberry-pi-4/) on microSD card
* Connect HDMI monitor, a keyboard and ethernet cable (for internet connectivity) to Raspberry Pi 4. Plug the microSD card and after that turn on Raspberry Pi 4 by plugging in the power cable into the USB-C port.
* Do the initial LibreELEC and Kodi setup following the on-screen instructions and enable **SSH** and **Samba**

## Default Keyboard Shortcuts:

The full list of default keyboard controls supported by Kodi are listed [here](https://kodi.wiki/view/Keyboard_controls).

Most commonly used keyboard shortcuts are `space` to pause or play, `x` to stop, arrow keys to navigate through menus, `T` to turn on/off subtitles, `Ctrl + S` to make a screenshot.

## Install TED plugin

Follow the steps below to install [TED plugin](https://kodi.wiki/view/Add-on:TED_Talks):

* From the left menu on the home screen select **Add-ons**.
* From the left menu on the home screen select **Download**.
* Select **Video Add-ons**.
* Select **TED Talks**.
* Click **Install**.

## Configuring keyboard shortcut for TED plugin

* Login via SSH (default user **root** and password **libreelec**) or Samba.
* Create keyboard configuration file `/storage/.kodi/userdata/keymaps/keyboard.xml` with the following XML content to launch the TED plugin (if it is installed) when key D is pressed:
```
<keymap>
<global>
  <keyboard>
       <d>RunAddon(plugin.video.ted.talks)</d>
   </keyboard>
  </global>
</keymap>
```
* Reboot Raspberry Pi to load the new configurations, from the left menu on Kodi main screen select the button with icon for **Power Options** and click **Reboot**.
* Press key D on the keyboard and verify that the previously installed TED plugin is started.

# Home Assistant

Home Assistant is an open source automation software for smart home. This chapter explains how to:
* Install Home Assistant on Raspberry Pi 4
* Configure Philips Hue Light in Home Assistant
* Configure keyboard remote integration in Home Assistant
* Create a rule to use a keyboard to control Philips Hue lightning bulb through Home Assistant

## Required Hardware

The following hardware is required:
* Raspberry Pi 4
* USB keyboard (recommended Raspberry Pi official keyboard)
* microSD card (recommended class 10, 16GB or more)
* HDMI monitor with appropriate cable
* Ethernet cable to connect Raspberry Pi to the LAN and the Internet
* Philips Hue Bridge
* Philips Hue White ambiance 
* Smartphone (Android or iOS) with installed and configured "Philips Hue App" mobile application

## Home Assistant Installation

Follow the steps below to install Home Assistant on Raspberry Pi 4:
* [Manual installation on a Raspberry Pi](https://www.home-assistant.io/docs/installation/raspberry-pi/).

* Open a terminal and execute hass command to start Home Assistant
```
hass
```

* Open a web browser and access http://homeassistant.local:8123/ If mDNS is not working replace `homeassistant.local` with the actual IP of the Raspberry Pi in your LAN.







## Connect Remodo_X to HomeAssistant

* Open a terminal and execute the following commands 
```
sudo bluetoothctl
```

* type **agent on** and press enter.
```
agent on
```

* Press and hold the III and IV keys at the same time on the Remodo X remote until it beeps. The GREEN LED should be flashing.
* Type **scan on** and press enter. 
```
scan on
```

The unique addresses of all the Bluetooth devices around the Raspberry Pi will appear and look something like **Remodo_X_xxxxxx**. 
* To pair the device, type pair [device mac address]. It should be like this, pair XX:XX:XX:XX:XX:XX 
```
pair 85:80:00:00:00:01
```
* Then, type connect XX:XX:XX:XX:XX:XX (the mac address you have paired in the previous step)
```
connect 85:80:00:00:00:01
```
* Then, you can exit the bluetoothctl process
```
exit
```
* Check the remodo x has successfully conencted or not, check device descriptor by this command, normally it is **event3**
```
ls /dev/input/
```


## Remodo X Integration in Home Assistant

* Go the path as below and edit the file configuration.yaml

`/home/pi/homeassistant/configuration.yaml`

* Add the following lines to the end of configuration.yaml and save it:
```
keyboard_remote:
    device_descriptor: '/dev/input/event3'
    type: 
    -   'key_up'
    -   'key_down'
```
* From the menu of the left go to **Configuration**
* Select **Server Controls**
* Click **CHECK CONFIGURATION**
* Verify that message **Configuration valid!** is shown. Otherwise go back to the previous steps to check the content of `/home/pi/homeassistant/configuration.yaml`.
* From **Server management** click **RESTART** and wait until Home Assistant restarts

**Note:** After enabling **Keyboard Remote** integration you can **not** use your keyboard for typing in the command-prompt because **Home Assistant** intercepts it and evdev will block it.

### Troubleshooting

To verify that the Keyboard Remote integration in Home Assistant is successful select **Developer tools** and click **EVENTS**. In **Listen to events** set `keyboard_remote_command_received` and subscribe. Press keys and observe the detected events to find out key codes. If event is not detected double check `/home/pi/homeassistant/configuration.yaml`


## Philips Hue Integration in Home Assistant

Follow the steps below to integrate Philips Hue in Home Assistant:

For more information about Philips Hue Integration 
(https://www.home-assistant.io/integrations/hue/):

* From the menu of the left go to **Configuration**
* Select **Integrations**.
* Click the plus button (from the lower right corner) and add search for Philips Hue
* Follow the instruction with Home Assistant to Link the Hub with Home Assistant. Press the "Button" on the bridge to register Philips Hue with Home Assistant
* Home Assistant will automatically detect your Philips Hue devices
* From the menu of "Configuration > Integrations > Configured > ", click on "Philips Hue: Philips Hue NT", and you can see Philips Hue devices appears.


## Home Assistant Automations

* Go the path as below and edit the file configuration.yaml

`/home/pi/homeassistant/configuration.yaml`


```
automation:
    - alias: "turn on lights"
    trigger:
        platform: event
        event_type: keyboard_remote_command_received
        event_data:
            device_descriptor: "/dev/input/event3"
            key_code: 2
        action:
            service: light.turn_on
            entity_id: light.hue_color_lamp
    - alias: turn off lights
    trigger:
        platform: event
        event_type: keyboard_remote_command_received
        event_data:
            device_descriptor: "/dev/input/event3"
            key_code: 3
        action:
            service: light.turn_off
            entity_id: light.hue_color_lamp
```

**NOTE:** You need to replace `light.hue_color_lamp` with the actual entity id of the Philips Hue lightning bulb! The keyboard device descriptor might be different from `/dev/input/event3` if several keyboards are connected.

* From the menu of the left go to **Configuration**
* Select **Server Controls**
* Click **CHECK CONFIGURATION**
* Verify that message **Configuration valid!** is shown. Otherwise go back to the previous steps to check the content of `/home/pi/homeassistant/configuration.yaml`.
* From **Server management** click **RESTART** and wait until Home Assistant restarts
* Verify that IKEA TRÅDFRI lightning bulb turns on when key A is pressed and turns off when key S is pressed.

# OpenHAB

OpenHAB is an open source automation software for smart home. The following chapter explains how to:
* Install OpenHAB2 on Raspberry Pi 4
* Configure Philips Hue Light
* Configure Linux Input binding
* Create a rule to use a keyboard to control Philips Hue lightning bulb through OpenHAB2

## Required Hardware

The following hardware is required:
* Raspberry Pi 4
* USB keyboard (recommended Raspberry Pi official keyboard)
* microSD card (recommended class 10, 16GB or more)
* HDMI monitor with appropriate cable
* Ethernet cable to connect Raspberry Pi to the LAN and the Internet
* Philips Hue Bridge
* Philips Hue White ambiance 
* Smartphone (Android or iOS) with installed and configured "Philips Hue App" mobile application

## OpenHAB Installation

Follow the steps below to install OpenHAB on Raspberry Pi 4:

* [Download openHABian](https://github.com/openhab/openhabian/releases), a GNU/Linux distribution for Raspberry Pi pre-configured with openHAB and available as image for microSD card. As of the moment, the recommended release is the latest stable openHABian v1.5 based on Debian Buster with Raspberry Pi 4 support.
* Flash openHABian on a microSD card with [balenaEtcher](https://www.balena.io/etcher/).
* Connect keyboard, HDMI monitor and Ethernet cable for Internet connectivity to the Raspberry Pi. Plug the microSD card and turn on the Raspberry Pi 4 by inserting the USB C cable. Wait until the system boots.
* The openHAB dashboard can be reached from a web browser on any computer in the same local area network at http://openhab:8080 If you experience any issues with mDNS access it directly using the IP, just for example (replace `192.168.1.2` with the actual IP of the Raspberry Pi 4): http://192.168.1.2:8080/

### Default Passwords
* User password needed for SSH or sudo: "openhabian:openhabian"
* openHAB remote console: "openhab:habopen"

**NOTE:** This tutorial is based on **Paper UI** of **OpenHAB2**.

## Connect REMODO X remote with your Raspberry pi with BLE connection

* Before start connecting the Remodo X with Raspberry pi, please make sure batteries are correctly install in the battery compartment of the remodo X. You can see GREEN LED flashing after batteries are correctly installed.

* Execute the following commands in command-prompt
* type **sudo bluetoothctl** then then press enter and input the administrator password

```
sudo bluetoothctl
```

* type **agent on** and press enter.
```
agent on
```

* Press and hold the III and IV keys at the same time on the Remodo X remote until it beeps. The GREEN LED should be flashing.
* Type **scan on** and press enter. 
```
scan on
```

The unique addresses of all the Bluetooth devices around the Raspberry Pi will appear and look something like **Remodo_X_xxxxxx**. 
* To pair the device, type pair [device mac address]. It should be like this, pair XX:XX:XX:XX:XX:XX 
```
pair 85:80:00:00:00:01
```
* Then, type connect XX:XX:XX:XX:XX:XX (the mac address you have paired in the previous step)
```
connect 85:80:00:00:00:01
```

## Install Linux Input Binding 
[Linux Input Binding](https://www.openhab.org/addons/bindings/linuxinput/) allows to you use a keyboard to control your openHAB instance. It works only on Linux and uses libevdev to expose all keys on the keyboard as channels.

* Execute the following commands in command-prompt to install libevdev:
```
sudo apt update
sudo apt install -y libevdev-dev
```
* Execute the following command to allow openhab to access keyboard events:
```
sudo usermod -a -G input openhab
```
* Execute the following command to restart OpenHAB2:
```
sudo systemctl restart openhab2.service
```

* Open a web browser and from the openHAB dashboard select **Paper UI**.
* From the menu on the left in **Paper UI** select **Add-ons**.
* Click the **Bindings** tab.
* Search for **Linux Input Binding**
* Click **INSTALL** and wait until **Linux Input Binding** installs.
* From the menu on the left select **Configurations > Things**.
* Click + button and then select LinuxInputBinding
* Select Remodo_X_Keyboard and add as "things"
* Enable the option of Configuration Parameters and click tick to save the modification and verify that path is `/dev/input/event0`
* Click the button to confirm and verify that **Remodo_X_Keyboard** has been added as thing with status **online**.
* Edit **Configuration > Things > Remodo_X_Keyboard**, in section **Channels** scroll down to **Others** and click **Key Event**. Create a linked item.
* Create a new Item name it **ANY_KEY** with **String** type 

For linked item  
&nbsp;&nbsp;&nbsp;&nbsp;- Name: <e.g. ANY_KEY>  
&nbsp;&nbsp;&nbsp;&nbsp;- Type: String  

**Note:** After installing and enabling **Linux Input Binding** you can **not** use your keyboard for typing in the command-prompt because evdev will block it.


## Philips Hue binding
Follow the steps below to install the [hue Binding](https://www.openhab.org/addons/bindings/hue/):

* Open a web browser and from the openHAB dashboard select **Paper UI**.
* From the menu on the left in **Paper UI** select **Add-ons**.
* Click the **Bindings** tab.
* Search for **hue Binding**
* Click **INSTALL** and wait until **hue Binding** installs.
* From the menu on the left in **Paper UI** select **Inbox**.
* Add **Hue Bridge** as **thing**.
* From the menu on the left select **Configurations > Things**.
* Create a new Item name it **LIGHT ON** with **Switch** type

## Rules

Follow the steps below to enable rules in **Paper UI** of **openHAB2**:

* From the menu on the left in **Paper UI** select **Add-ons**.
* Click the **MISC** tab.
* Select **Rule Engine (Experimental)** and click **INSTALL**.
* After successful installation of **Rule Engine (Experimental)** refresh the web browser and verify **Rules** in the left menu of **Paper UI**.

#### Create a rule to turn on lights

Follow the steps below to create a rule that turns on the lights when key A on the keyboard is pressed:

* From the menu on the left in **Paper UI** go to **Rules**, click the + icon and select **New rule**.
* For **when** select **an item state changes**, for **item** select the linked key event(e.g. ANY_KEY that is you have created in previous steps), for state type in **KEY_1**. (This key must be matched with the key configured in the Remodo X remote)
* For **then** select **send a command**, for **item** select the linked Light_ON (This item should be the one you have created in the previous step when setup Philips Hue Light), for **Command** select **ON**. Then, click **OK** to confirm you modification.
* Verify that the Philips Hue lightning bulb turns on by pressing key 1 on the Remodo X.
