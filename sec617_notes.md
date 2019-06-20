# Notes on SEC617 - Wireless Penetration Testing and Ethical Hacking
## Day 1


### Capture 802.3 Traffic (only when assiciated and only data)
Managed Mode Capture
```bash
$ sudo tcpdump -neti wlan0
```

Monitor Mode Capture
```bash
$ sudo tcpdump -neti mon0
```

### RFMON manual setup (Day1 pg. 44)
```
# iw dev wlan0 interface add mon0 type monitro
# ifconfig mon0 up
# iw dev mon0 set channel 1
# iw dev mon0 info
    ...
    # use the interface
# iw dev mon0 del

```

### Airmon-ng (Day1 pg 45)
```
# airmon-ng
    ...
# airmon-ng start wlan0
    ...
# iw dev wlan0 del
```

### Controlling the Physical characteristics of your device: (Day 1 pg. 51)
```
# iw dev mon0 info | grep type
    type monitor
# iw dev mon0 set channel 1
# iw dev mon0 set channel 132
# iw dev mon0 info | grep channel
    Channel 132 (5660 MHz), width: 20 MHz (no HT), centerl: 5660 MHz
# iw dev mon0 set channel 132 HT40+
# iw dev mon0 info | grep channel
    Channel 132 (5660 MHz), width: 40 MHz (no HT), centerl: 5670 MHz
# iw dev mon0 set channel 132 HT40-
# iw dev mon0 info | grep channel
    Channel 132 (5660 MHz), width: 40 MHz (no HT), centerl: 5650 MHz
# iw dev mon0 set channel 36 HT40+
# iw dev mon0 set channel 36 HT40-
    command failed: invalid argument (-22)
```

### Controlling Regulatory Domain settings (Day 1 pg. 52)
```
iw reg get
    country 00: DFS-UNSET...
iw reg set US
iw reg get | grep country
    country US: DFS-FCC
iw reg set CH
iw reg get | grep country
    Country CH: DFS-ETSI
```

### TCPDUMP (Day 1 pg. 56)
tcp dump options:

| flag | description |
|------|-------------|
| -i | Specify capture interface |
| -e | Print link-level header information (MAC address) |
| -n | Don't do DNS lookups on addresses and ports |
| -s | Set capture snap length |
| -X | Print payload in ASCII and hex |
| -r | Read from a capture file |
| -w | Save to a capture file |
| -c | number of packets to process before exit |

```
# tcpdump -n -i mon0 -s 0 -w capture.dump
    tcpdump: listening on mon0
    ^C
    30 packets received by filter
    0 packets dropped by kernel
$ tcpdump -r capture.dump -n -c 2
    ... # prints two packets from the file
$ tcpdump -t -r capture.dump -n -c 1 -X
    ... # prints the output of one packet in hex and ascii
```
tcpdump filters on pg. 58

### Wireshark (Day 1 Pg 60)

| Operator | Description |
|--|--|
| eq, == | Equal |
| ne, != | Not equal |
| gt, > | Greater than |
| lt, < | Less than |
| ge, >= | Greater than or equal |
| le, <= | Less than or equal |
| contains | Contains specific data |
| ~, matches | Protocol or text field match Perl regualar expression. http.host matches "acme\.(org|com|net)" |
| &, bitwise_and | Compare bit field value. tcp.flags & 0x02 |
| &&, and | Logical AND. ip.src==10.0.0.5 and tcp.flags.fin |
| \|\|, or | Logical OR. ip.scr==10.0.0.5 or ip.src==192.1.1.1 |
| ^^, xor | Logical XOR. tr.dst[0:3] == 0.6.29 xor tr.src[0:3] == 0.6.29 |
| !, not | Logical NOT. not llc |
| […​] | “Slice Operator” |
| in | "Membership Operator" |

Example display filters:
- `!wlan.fc.type_subtype == 8` // Will exclude all beacon frames
- `!wlan.fc.protected == 1` // List all frames without the WEP bit (privacy bit) set
- `wlan.bssid == 00:3d:af:24:00:12` // only show packets with this bssid
- `frame contains ORA-` // will search entire packet for the case sensitive string "ORA-"
- `tcp or udp or arp or isakmp` // will not shoe fields that are not of the listed type
- `!(wlan.bssid == 00:10:e7:f5:c3:1f or wlan.bssid == 00:60:1d:f0:47:39)` // will not show traffic from the two bssids

### Kismet (Day 1 pg. 67)
### giskismet (Day 1 pg. 75)
```
# giskismet -x Kismet-20110621-15-28-14-1.netxml
    Checking database...
# giskismet -q "select * from wireless" -o all-nets.kml
# giskismet -q "select * from wireless where Encryption='None'" -o unenc-nets.kml
# giskismet -q "select * from wireless where ESSID='linksys'" -o linksys-nets.kml
```
> Note: if your captures do not have GPS, use `--ignore-gps`
Using SQLlite db output is on Pg. 77


## Lab 1.2 Notes

### Exercise: Capturing Traffic in Monitor Mode
- `# ifconfig -a` command to list all the interfaces on your system
- `# ls /sys/class/ieee80211/` show the wireless physical interface

Manually Configure Monitor Mode:
```
# iw dev wlan0 del  # deletes the managed wlan0 interface
# iw phy phy0 interface add wlan0mon type monitor
# iw dev wlan0mon info
    ...
# ifconfig wlan0mon
# ifconfig wlan0mon up
# ifconfig wlan0mon
    ... # confirm it's in the up state
```
> Note: Run the `ifconfig wlan0mon` command to see the packet number increase so you know the device is working correctly

Change the channel number or frequency:
```
# iw dev wlan0mon set channel 1
# iw dev wlan0mon set freq 2412

or 

# iw dev wlan0mon set channel 1 HT40+
```

### Use airmon-ng to save on some repetitive typing!
```
# airmon-ng
PHY	Interface	Driver		Chipset
phy1	wlan0		rt2800usb	Ralink Technology, Corp. RT5372

# airmon-ng start wlan0
```
Add the channel number when you create it
```
# iw dev wlan0mon del
# airmon-ng start wlan0 11
# iwconfig wlan0mon  <-- to verify the change
```

Stopping the interface with airmon (effectively deleting the interface)
```
# airmon-ng stop wlan0mon
```

### Exercise: Wi-Fi Analysis with Kismet

Unplug and then re-plug the device, then run ifconfig to find where is mounts:
```
# ifconfig -a | grep wlan
wlan0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
```
Move to a directory where you don't mind Kisment spamming logs/dumps before starting kismet:
```
# cd /tmp/
# kismet
```
The first time you run Kismet, it will prompt you to determine if the dark grey text is legible
Press Enter to acknowledge the warning about running as root.
Select Yes to start the Kismet server. Accept the default settings for logging, log title, and console display and select Start.
At the Kismet server console view, navigate to Close Console Window by pressing the Tab key, then press Enter.
After a few seconds, Kismet will prompt you to add a packet capture source. Select Yes to open the Add Source dialog box. Specify wlan0 (or whatever ifconfig spit out) as the interface name in the Intf field, navigate to the Add button with the Tab key, and press Enter to add this source, as shown here.

- Access Menus: Press the backtick (`) to start navigating the Kismet client menu
- Changing the Network Sort Order: **Sort menu**, changing the sort order to **First Seen**
- Locking the Channel: **Kismet | Config channel**

> Remember: Kismet will not refresh/notice the change in channel number if made outside of kismet

Kismet Files:
```
# ls Kismet*
Kismet-20170128-11-11-25-1.alert     Kismet-20170128-11-11-25-1.nettxt
Kismet-20170128-11-11-25-1.pcapdump  Kismet-20170128-11-11-25-1.gpsxml
Kismet-20170128-11-11-25-1.netxml
```
| File Extension | Kismet Purpose |
|-|-|
| .pcapdump | Libpcap capture file of all wireless traffic observed by Kismet |
| .nettxt | Text-file summary of networks and clients observed by Kismet, suitable to be imported into a word processor or viewed with a text editor | 
| .netxml | XML-formatted information collected by Kismet, suitable to be imported into a spreadsheet tool such as Microsoft Excel. |
| .alert | List of Kismet WIDS alerts identified |
| .gpsxml | XML-formatted GPS coordinates for traffic collected by Kismet |



## Lab 1.2 Notes

### Exercise: Identifying Rogue APs
Using Nmap with the rogueap.nse script to perform wired-side scanning to locate a rogue access point.
```
# cd ~
# wget http://www.willhackforsushi.com/code/rogueap.nse
```
| Nmap Target Designation | Effective Host Targets |
|--|--|
| 192.168.15.0/24 | All the hosts between 192.168.15.0 and 192.168.15.255 |
| 192.168.15.0/23 | All the hosts between 192.168.14.0 and 192.168.15.255 |
| 192.168.15.10,11,24 | Three hosts with addresses 192.168.15.10, 192.168.15.11, and 192.168.15.24 |
| 192.168.15.10-15 | Five hosts with addresses 192.168.15.10, 192.168.15.11, 192.168.15.12, 192.168.15.13, 192.168.15.14, and 192.168.15.15 |
| 192.168.15-13.1 | Three hosts with addresses 192.168.15.1, 192.168.14.1, and 192.168.13.1 |

```
# nmap -sS -p 75-85 -O --open --script=rogueap.nse 192.168.15.1-20
Starting Nmap 6.47 ( http://nmap.org ) at 2015-01-29 06:41 EST
Nmap scan report for 192.168.15.1
Host is up (0.048s latency).
Not shown: 10 closed ports
PORT   STATE SERVICE
80/tcp open  http
|_rogueap: Possible Rogue AP Found: "WRT54"
Device type: general purpose|storage-misc|VoIP phone
Running (JUST GUESSING): Microsoft Windows 2008|7 (98%), BlueArc embedded (91%), Pirelli embedded (88%)
Aggressive OS guesses: Microsoft Windows Server 2008 SP1 (98%), Microsoft Windows 7 Enterprise (96%), BlueArc Titan 2100 NAS device (91%), Pirelli DP-10 VoIP phone (88%)
No exact OS matches for host (test conditions non-ideal).

OS detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.70 seconds
```