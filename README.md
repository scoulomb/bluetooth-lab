# bluetooth

Dojo fork: https://gitlab.com/jiehong/notes/-/wikis/investigations/bluetooth

## Overview

Following https://en.wikipedia.org/wiki/Bluetooth, bluetooth uses the same radio frequencies as the Wifi.

Low Energy is achieved by using larger and fewer channels.

Maximum throughput seems to be 8Mbits/s (HDR8) on a wide 4Mhz channel (limited by the relatively slow clock used. This slow clock is also the reason for the relative high latency in bluetooth).


## Networks

A single device can communicate with another 7 devices in a piconet. Piconets can be linked together to form scatternets.

## Roles

Devices follow a master/slave approach, but they can exchange roles. A device can be the slave of multiple masters, and a master can talk to up to 7 slaves (switching between each in a round-robin fashion).

## Protocol

The bluetooth protocol is designed differently from the OSI model used for Internet, and doesn't preserve a very clear distinctions between all the layers:

![](./Bluetooth_protokoly.svg)

This seems to be linked to the teams having worked on the protocal and Conway's law (ref: Tanenbaum network book).

### Link Manager

The link manager is in charge of establishing the connection with other link managers (devices), and to also handle authentication.

### Host Controller Interface (HCI)

Mainly finds other bluetooth devices on the network.

Logical Link Control and Adaptation Protocol (L2CAP)
The L2CAP is a protocol used to transmit data.

It has 2 variants:

- Enhanced Retransmission Mode (ERTM): basically equivalent to TCP traffic
- Streaming Mode (SM): basically equivalent to UDP traffic

### Service Discovery Protocol (SDP)
The SDP allows discovering services/profiles a device offers:

- each service has a UUID;
- each service defines a specific profile.

For example, a profile might be there to handle a headset's play/pause communication, while another profile might be there to handle the actual audio data transmission.

📝 some profiles are only visible after a devices has been authentication, for security purposes.

### Audio/Video Control Transport Protocol (AVCTP)

AVCTP defines ways to send/receive commands controlling audio and video streams, such as play, pause, next, etc used by the Audio/Video Remote Control Profile (AVRCP) profile.

### Audio/Video Distribution Transport Protocol (AVDTP)

AVDTP defines ways to send/receive audio and video streams by the advanced audio distribution (A2DP) profile.

### Other protocols

Other protocols can be used such as OBEX, IP/TCP/UDP, PPP, WAP, etc carried through bluetooth.

Those might be used for sending files through bluetooth (some phones do that).

## Connections

### Discovery
Each device can be in discovery mode, and transmit some information if requested, such as name, class, list of services.

📝 the name does not have to be unique, and this can potentially lead to name squatting, or having people try to connect to devices they think they control because the name is identical.

### Pairing (and bonding)

For security reasons, not all devices should automatically connect to each other, and require pairing.

The idea is to share and store a shared secret between 2 devices. Once this is done, the devices are bonded.

This is achieved through the Secure Simple Pairing (SSP) process (unless old bluetooth < 2.1 used). Multiple ways exist for the SSP to succeed, depending on the capabilities of the devices:

- Just Works: aka, automatic secret sharing with optional confirmation by the user. Simple, but not very secure (never saw it in reality);
- Numeric comparison: if both devices have a display, a 6-digit code is shown on both devices, and the user must confirm it is the same on one of the devices (never saw it in reality);
- Passkey Entry: if only one of the devices has a display, and the other one has a way to enter digits (such as a keyboard), the user must enter the 6-digit code in the other devices for confirmation (can be seen when setting up a new iPad with an existing iPhone for example, or some Android TVs);
Out of band: the confirmation is sent on a different medium and/or at a different time (eg: by email, on an NFC token, etc.) (might be used when bringing a phone close to a device to pair them, one of them containing an NFC token);

## Profiles
Reference: https://en.wikipedia.org/wiki/List_of_Bluetooth_profiles

### Advanced Audio Distribution Profile (A2DP)
This allows uni-directional transfer of up to 2 audio channels.

This is done by using some audio codecs, most of which are optional and can be extended by manufacturers:

- SBC (mandatory codex, easy to implement, low quality);
- everything else is optional:
  - MPG 2/3/4;
  - ATRAC;
  - aptX;
  - LDAC;
  = etc.

📝 No lossless codec is used yet (probably due to the lack of bandwidth to use FLAC).

Some of those codecs also support DRMs.

Manufacturers tend to market and differentiate themselves on those supported codecs (especially for headphones).

Yet, high-end manufacturers of AV devices may only support the bare minimum codec (SBC), such as the Denon AVR-X2700H stating a support of A2DP 1.2 SBC.

### Audio/Video Remote Control Profile (AVRCP)

Each version of AVRCP adds functionalities to control an audio device (such as a HiFi system, a TV or a car audio system):

- play, pause, volume, etc;
- status (playing, paused, etc);
- track metadata (artist, track name, etc);
- searching and browsing;
- sending albums covers.

### Human Interface Device Profile (HID)

Mouses, keyboards and game controllers use the Human Interface Device (HID) protocol, just like the HID profile is carried through USB otherwise.

## Bluetooth on Linux
Reference: https://wiki.archlinux.org/title/Bluetooth_headset

### Listing devices and connecting

We can check the list of devices, but it's empty:

````
$ bluetoothctl devices
````

Let's scan for devices:

````
$ bluetoothctl scan on
````

After a while, we can stop and check the discovered devices:


````
$ bluetoothctl devices
Device C8:7B:23:2C:EA:A8 LE-Bose Sport Earbuds
Device 22:22:5A:17:FE:F4 Freebox Player POP
Device 3B:F8:CB:04:EE:9A 3B-F8-CB-04-EE-9A
Device 45:C2:B3:B7:14:99 45-C2-B3-B7-14-99
Device 2B:AF:A6:F4:D8:5F 2B-AF-A6-F4-D8-5F
Device 10:60:9C:42:33:5F 10-60-9C-42-33-5F
Device 44:A5:31:9C:56:59 44-A5-31-9C-56-59
Device 1C:B3:C9:2C:1A:26 1C-B3-C9-2C-1A-26
Device 2A:9A:9B:59:68:92 2A-9A-9B-59-68-92
Device 2B:B0:4B:A5:44:BF 2B-B0-4B-A5-44-BF
Device 90:DD:5D:D0:E8:9B 90-DD-5D-D0-E8-9B
Device 4E:8B:81:6B:56:73 4E-8B-81-6B-56-73
Device 5B:59:82:46:D4:A2 5B-59-82-46-D4-A2
We can ask for info about the first device:

[bluetooth]# info C8:7B:23:2C:EA:A8
Device C8:7B:23:2C:EA:A8 (public)
        Name: Bose Sport Earbuds
        Alias: Bose Sport Earbuds
        Class: 0x00240418
        Icon: audio-card
        Paired: no
        Trusted: no
        Blocked: no
        Connected: no
        LegacyPairing: no
        UUID: Bose Corporation          (0000febe-0000-1000-8000-00805f9b34fb)
        UUID: Advanced Audio Distribu.. (0000110d-0000-1000-8000-00805f9b34fb)
        UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)
        UUID: Audio Source              (0000110a-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control        (0000110e-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control Cont.. (0000110f-0000-1000-8000-00805f9b34fb)
        UUID: Phonebook Access          (00001130-0000-1000-8000-00805f9b34fb)
        UUID: Phonebook Access Client   (0000112e-0000-1000-8000-00805f9b34fb)
        UUID: Handsfree                 (0000111e-0000-1000-8000-00805f9b34fb)
        UUID: Headset                   (00001108-0000-1000-8000-00805f9b34fb)
        UUID: Headset HS                (00001131-0000-1000-8000-00805f9b34fb)
        UUID: Vendor specific           (00000000-deca-fade-deca-deafdecacaff)
        Modalias: bluetooth:v009Ep402Dd0207
        ManufacturerData Key: 0x1903
        ManufacturerData Value:
  d1 18 0d 81 ff e3 bd 57 a6 bb 88                 .......W...
`````

We can connect and trust:

`````
# connect C8:7B:23:2C:EA:A8
Attempting to connect to C8:7B:23:2C:EA:A8
[NEW] Device 2B:AF:A6:F4:D8:5F 2B-AF-A6-F4-D8-5F
[CHG] Device C8:7B:23:2C:EA:A8 Connected: yes
[CHG] Device C8:7B:23:2C:EA:A8 Name: Bose Sport Earbuds
[CHG] Device C8:7B:23:2C:EA:A8 Alias: Bose Sport Earbuds
[NEW] Device 7D:11:28:E0:BD:45 7D-11-28-E0-BD-45
[CHG] Device 62:9D:80:54:6D:ED RSSI: -66
[CHG] Device C8:7B:23:2C:EA:A8 Modalias: bluetooth:v009Ep402Dd0207
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 00000000-deca-fade-deca-deafdecacaff
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 00001101-0000-1000-8000-00805f9b34fb
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 00001108-0000-1000-8000-00805f9b34fb
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 0000110b-0000-1000-8000-00805f9b34fb
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 0000110c-0000-1000-8000-00805f9b34fb
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 0000110e-0000-1000-8000-00805f9b34fb
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 0000111e-0000-1000-8000-00805f9b34fb
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 00001200-0000-1000-8000-00805f9b34fb
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 0000eb03-d102-11e1-9b23-00025b00a5a5
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 85dbf2f9-73e3-43f5-a129-971b91c72f1e
[CHG] Device C8:7B:23:2C:EA:A8 UUIDs: 9b26d8c0-a8ed-440b-95b0-c4714a518bcc
[CHG] Device C8:7B:23:2C:EA:A8 ServicesResolved: yes
[CHG] Device C8:7B:23:2C:EA:A8 Name: LE-Bose Sport Earbuds
[CHG] Device C8:7B:23:2C:EA:A8 Alias: LE-Bose Sport Earbuds
[CHG] Device C8:7B:23:2C:EA:A8 Paired: yes
[CHG] Device C8:7B:23:2C:EA:A8 ManufacturerData Key: 0x1903
[CHG] Device C8:7B:23:2C:EA:A8 ManufacturerData Value:
  51 18 0d 1b 92 ef ba 2b 2d ae 9e                 Q......+-..
Connection successful

`````

``````
[LE-Bose Sport Earbuds]# trust C8:7B:23:2C:EA:A8
[CHG] Device C8:7B:23:2C:EA:A8 Trusted: yes
Changing C8:7B:23:2C:EA:A8 trust succeeded
``````

``````
[LE-Bose Sport Earbuds]# info
Device C8:7B:23:2C:EA:A8 (public)
        Name: LE-Bose Sport Earbuds
        Alias: LE-Bose Sport Earbuds
        Paired: yes
        Trusted: yes
        Blocked: no
        Connected: yes
        LegacyPairing: no
        UUID: Vendor specific           (00000000-deca-fade-deca-deafdecacaff)
        UUID: Serial Port               (00001101-0000-1000-8000-00805f9b34fb)
        UUID: Headset                   (00001108-0000-1000-8000-00805f9b34fb)
        UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control Target (0000110c-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control        (0000110e-0000-1000-8000-00805f9b34fb)
        UUID: Handsfree                 (0000111e-0000-1000-8000-00805f9b34fb)
        UUID: PnP Information           (00001200-0000-1000-8000-00805f9b34fb)
        UUID: Vendor specific           (0000eb03-d102-11e1-9b23-00025b00a5a5)
        UUID: Vendor specific           (85dbf2f9-73e3-43f5-a129-971b91c72f1e)
        UUID: Vendor specific           (9b26d8c0-a8ed-440b-95b0-c4714a518bcc)
        Modalias: bluetooth:v009Ep402Dd0207
        ManufacturerData Key: 0x1903
        ManufacturerData Value:
  71 18 63 39 fe cf 94 8f af 90 df 98 24 f9        q.c9........$.
        RSSI: -52
        TxPower: -10
``````

Now the headset is paired, and ready to be used:

``````
$ bluetoothctl paired-devices
Device C8:7B:23:2C:EA:A8 LE-Bose Sport Earbuds
``````

### Bluetooth headset as an audio card

Pulseaudio now sees this as an audio card that can use the SBC codec:

``````
$ pactl list
Card #1
        Name: bluez_card.C8_7B_23_2C_EA_A8
        Driver: module-bluez5-device.c
        Owner Module: 22
        Properties:
                device.description = "LE-Bose Sport Earbuds"
                device.string = "C8:7B:23:2C:EA:A8"
                device.api = "bluez"
                device.class = "sound"
                device.bus = "bluetooth"
                bluez.path = "/org/bluez/hci0/dev_C8_7B_23_2C_EA_A8"
                bluez.class = "0x000000"
                bluez.alias = "LE-Bose Sport Earbuds"
                device.icon_name = "audio-card-bluetooth"
                bluetooth.codec = "sbc"
        Profiles:
                headset_head_unit: Headset Head Unit (HSP) (sinks: 1, sources: 1, priority: 30, available: no)
                a2dp_sink: High Fidelity Playback (A2DP Sink) (sinks: 1, sources: 0, priority: 40, available: yes)
                handsfree_head_unit: Handsfree Head Unit (HFP) (sinks: 1, sources: 1, priority: 30, available: yes)
                off: Off (sinks: 0, sources: 0, priority: 0, available: yes)
        Active Profile: a2dp_sink
        Ports:
                unknown-output: Bluetooth Output (type: Bluetooth, priority: 0, latency offset: 0 usec, availability unknown)
                        Part of profile(s): headset_head_unit, a2dp_sink, handsfree_head_unit
                unknown-input: Bluetooth Input (type: Bluetooth, priority: 0, latency offset: 0 usec, availability unknown)
                        Part of profile(s): headset_head_unit, handsfree_head_unit
``````

We can ask what codecs are supported:

``````
$ pactl send-message /card/bluez_card.C8_7B_23_2C_EA_A8/bluez list-codecs | jq
[
  {
    "name": "sbc",
    "description": "SBC"
  },
  {
    "name": "sbc_xq_453",
    "description": "SBC XQ 453kbps"
  },
  {
    "name": "sbc_xq_512",
    "description": "SBC XQ 512kbps"
  },
  {
    "name": "sbc_xq_552",
    "description": "SBC XQ 552kbps"
  }
]
``````

The headset should also support AAC (https://www.boseindia.com/en_in/products/headphones/earbuds/bose-sport-earbuds.html#ProductTabs_tab1), but is not shown in Linux (why?).

### Bluetooth traffic capture

We can check tcpdump to see the interfaces:

``````
$ tcpdump -D
1.wlo1 [Up, Running, Wireless, Associated]
2.flannel.1 [Up, Running, Connected]
3.cni0 [Up, Running, Connected]
4.veth4c454470 [Up, Running, Connected]
5.vetha8e0767b [Up, Running, Connected]
6.veth38350bbc [Up, Running, Connected]
7.veth4010a94d [Up, Running, Connected]
8.veth8b1b8fbb [Up, Running, Connected]
9.veth69fade00 [Up, Running, Connected]
10.vetha72f680f [Up, Running, Connected]
11.veth33d7c581 [Up, Running, Connected]
12.veth6115b03c [Up, Running, Connected]
13.vethb5899382 [Up, Running, Connected]
14.vethea5b9545 [Up, Running, Connected]
15.veth1c2d4f7b [Up, Running, Connected]
16.veth750baacd [Up, Running, Connected]
17.veth7252a6d1 [Up, Running, Connected]
18.veth11be456f [Up, Running, Connected]
19.veth736b34e7 [Up, Running, Connected]
20.veth3b2cafa5 [Up, Running, Connected]
21.vethbc914ab2 [Up, Running, Connected]
22.veth7efab993 [Up, Running, Connected]
23.vetheed385b2 [Up, Running, Connected]
24.any (Pseudo-device that captures on all interfaces) [Up, Running]
25.lo [Up, Running, Loopback]
26.eno1 [Up, Disconnected]
27.br-4cdc7334da31 [Up, Disconnected]
28.docker0 [Up, Disconnected]
29.br-5460f29249a0 [Up, Disconnected]
30.br-5c12fb711107 [Up, Disconnected]
31.br-6208c1507ce0 [Up, Disconnected]
32.br-e2f8819c1dea [Up, Disconnected]
33.br-1eebc5551699 [Up, Disconnected]
34.br-4b1253ce589f [Up, Disconnected]
35.bluetooth0 (Bluetooth adapter number 0) [Wireless, Association status unknown]
36.bluetooth-monitor (Bluetooth Linux Monitor) [Wireless]
37.nflog (Linux netfilter log (NFLOG) interface) [none]
38.nfqueue (Linux netfilter queue (NFQUEUE) interface) [none]
39.dbus-system (D-Bus system bus) [none]
40.dbus-session (D-Bus session bus) [none]
``````

We can start recording packets, and push the "play/pause" button:

``````
$ tcpdump -i bluetooth0 -w bluetooth0_capture.pcap
tcpdump: listening on bluetooth0, link-type BLUETOOTH_HCI_H4_WITH_PHDR (Bluetooth HCI UART transport layer plus pseudo-header), snapshot length 262144 bytes
^C49 packets captured
27059 packets received by filter
0 packets dropped by kernel
``````

Let's copy the captured packet locally:

``````
$ scp scoulomb@host:/home/scoulomb/bluetooth0_capture.pcap .
``````

However, this only shows HCI_EVT packets, and does not show anything useful (why? unknown).

Instead, we try and can install [hcidump](https://linux.die.net/man/8/hcidump:

``````
$ hcidump
HCI sniffer - Bluetooth packet analyzer ver 5.66
device: hci0 snap_len: 1500 filter: 0xffffffffffffffff
> HCI Event: Command Status (0x0f) plen 4
    Create Connection (0x01|0x0005) status 0x00 ncmd 1
> HCI Event: Connect Complete (0x03) plen 11
    status 0x00 handle 21 bdaddr C8:7B:23:2C:EA:A8 type ACL encrypt 0x00
> HCI Event: Command Status (0x0f) plen 4
    Read Remote Supported Features (0x01|0x001b) status 0x00 ncmd 1
> HCI Event: Command Complete (0x0e) plen 4
    Write Scan Enable (0x03|0x001a) ncmd 1
    status 0x00
> HCI Event: Read Remote Supported Features (0x0b) plen 11
    status 0x00 handle 21
    Features: 0xff 0xfe 0x8f 0xfe 0xdb 0xff 0x5b 0x87
> HCI Event: Command Status (0x0f) plen 4
    Read Remote Extended Features (0x01|0x001c) status 0x00 ncmd 1
> HCI Event: Read Remote Extended Features (0x23) plen 13
    status 0x00 handle 21 page 1 max 2
    Features: 0x03 0x00 0x00 0x00 0x00 0x00 0x00 0x00
> HCI Event: Command Status (0x0f) plen 4
    Remote Name Request (0x01|0x0019) status 0x00 ncmd 1
> HCI Event: Remote Name Req Complete (0x07) plen 255
    status 0x00 bdaddr C8:7B:23:2C:EA:A8 name 'Bose Sport Earbuds'
> HCI Event: Command Status (0x0f) plen 4
    Authentication Requested (0x01|0x0011) status 0x00 ncmd 1
> HCI Event: Command Complete (0x0e) plen 10
    Link Key Request Reply (0x01|0x000b) ncmd 1
    status 0x00 bdaddr C8:7B:23:2C:EA:A8
> HCI Event: Auth Complete (0x06) plen 3
    status 0x00 handle 21
> HCI Event: Command Status (0x0f) plen 4
    Set Connection Encryption (0x01|0x0013) status 0x00 ncmd 1
> HCI Event: Encrypt Change (0x08) plen 4
    status 0x00 handle 21 encrypt 0x01
``````

📝 However, no data is shown after this connection setup.

Install package on old Ubuntu when servers are down (Click to expand)

On another computer, both using wireshark and btmon, we can only capture the non-data packets (in this case the ATT protocol for bluetooth low energy):

``````
$ sudo btmon
Bluetooth monitor ver 5.66
= Note: Linux version 5.15.114 (x86_64)                                                                                                               0.079036
= Note: Bluetooth subsystem version 2.22                                                                                                              0.079037
= New Index: AC:67:5D:49:54:2A (Primary,USB,hci0)                                                                                              [hci0] 0.079038
= Open Index: AC:67:5D:49:54:2A                                                                                                                [hci0] 0.079038
= Index Info: AC:67:5D:49:54:2A (Intel Corp.)                                                                                                  [hci0] 0.079038
@ MGMT Open: bluetoothd (privileged) version 1.21                                                                                            {0x0001} 0.079039
> ACL Data RX: Handle 3585 flags 0x02 dlen 22                                                                                               #1 [hci0] 0.194587
      ATT: Handle Value Notification (0x1b) len 17
        Handle: 0x0021
          Data: 000000000000000000000000000000
> ACL Data RX: Handle 3585 flags 0x02 dlen 8                                                                                                #2 [hci0] 2.279466
      ATT: Handle Value Notification (0x1b) len 3
        Handle: 0x0012
          Data: 51
> ACL Data RX: Handle 3585 flags 0x02 dlen 22                                                                                               #3 [hci0] 7.949616
      ATT: Handle Value Notification (0x1b) len 17
        Handle: 0x0021
          Data: 000000800000000000000000000000
``````

Many other tools exist to check for bluetooth traffic: https://www.kali.org/tools/bluez/, each specialized with one protocol.


## Optional notes

- https://en.wikipedia.org/wiki/List_of_Bluetooth_profiles

> Each A2DP service, of possibly many, is designed to uni-directionally transfer an audio stream in up to 2 channel stereo, either to or from the Bluetooth host.[2] This profile relies on AVDTP and GAVDP

- Do not check if Atoll has apt-x 

- Parallel with service discovery and co here: https://github.com/scoulomb/home-assistant/blob/main/sound-video/README.md#sound-1

- OK CCL