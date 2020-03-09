# Ubiquiti Edgerouter (ERL) OpenVPN setting #
Configuration for setting up OpenVPN for PIA

## Steps to set up OpenVPN on a ERL for PIA - Possibly work for other VPNs ##

Download the .zip file directly from PIA - Latest URL is here:

    https://www.privateinternetaccess.com/openvpn/openvpn.zip

Download WinSCP, connect to your ERL and then upload the following files in the zip file to your ERL to this remote path /config/auth 
    
    ca.rsa.2048.crt
    ca.rsa.2048.pem
    *.ovpn (pick the configuration file you want to use and upload it)

SSH into your ERL and run the following commands

    sudo su
    cd /config/auth
vi *.ovpn (replace with the name of the *.ovpn you picked, example Austria.ovpn

    vi Austria.ovpn
hit i < to enter edit mode
when you find auth-user-pass line, make it look like the line below

    auth-user-pass /config/auth/userpass.txt
    
 Also you need to add to the end of your *.ovpn file one more entry
     
     route-nopull
     
This stops the VPN in this case PIA from adding it's on routes, and uses the routing table we configured on the ER.

Hit the ESC key and type :wq
This will write the file and then quit out of vi

You can retrieve your PIA PPTP/L2TP/SOCKS Username and Password by logging into PIA online and select:

    My Account; continue scrolling down.
Locate:

    PPTP/L2TP/SOCKS Username and Password.
On two lines put your PIA PPT/L2TP/Socks Username on the first line and on the second line put the Password

Example of what it should look like in the userpass.txt file you creatd in /config/auth directory on the ERL:
Create the file userpass.txt

    vi userpass.txt
    hit i < to enter edit mode

Example of what your Username and Password looks like.
    
    x29283933333
    WidD92F398msb
    
__________________________________________________
    Hit the ESC key and type :wq

## Configuration lines to add to the ERL ##

    configure

    set interfaces openvpn vtun0 config-file /config/auth/East.ovpn
    set interfaces openvpn vtun0 description 'Private Internet Access'
    set interfaces openvpn vtun0 enable

    set service nat rule 5000 description PIA
    set service nat rule 5000 log disable
    set service nat rule 5000 outbound-interface vtun0
    set service nat rule 5000 source address 192.168.1.0/24
    set service nat rule 5000 type masquerade

    set service nat rule 5001 description default
    set service nat rule 5001 log disable
    set service nat rule 5001 outbound-interface eth0
    set service nat rule 5001 source address 192.168.1.0/24
    set service nat rule 5001 type masquerade

    set protocols static table 1 interface-route 0.0.0.0/0 next-hop-interface vtun0

    set firewall modify pia_route rule 10 description 'PIA'
    set firewall modify pia_route rule 10 source address 192.168.1.5/32
    set firewall modify pia_route rule 10 modify table 1

    set interfaces ethernet eth1 firewall in modify pia_route

    commit
    save

NOTE: eth0 is the WAN (Internet) interface. eth1 is the LAN interface for 192.168.1.0/24 network

If your LAN network is on eth2, you would change the last entry above to read:

set interfaces ethernet eth2 firewall in modify pia_route

------------------------

If you want to have only 1 IP go through the VPN such as your Kodi Box or your Torrent box

Do all of the steps above, except replace the following two lines:

    set service nat rule 5000 source address 192.168.1.0/24
    set service nat rule 5001 source address 192.168.1.0/24

to

    set service nat rule 5000 source address 192.168.1.5/31
    set service nat rule 5001 source address 192.168.1.5/31
      
On the ERL Dashboard, you should now see the newly created interface (vtun0) showing connected

Test to see if the network or individual system is going out the VPN.

A couple of ways is to use a web browser and go to:

    https://ifconfig.me
    
If you are SSH'd into a shell run:

    curl ifconfig.me
This will display your external IP, which you should be able to determine if it's your normal assigned ISP IP or the VPN IP.

__________________________
##### Added March 8th #####
##### Continue to route local networks without going out vtun0 #####
##### If you don't enable these steps, all your traffic will continue to go out vtun0 #####

##### These are the needed steps to route traffic to the appropriate interfaces #####

##### eth0 = WAN (Internet) #####
##### eth1 = 192.168.1.0/24 #####
##### eth2 = 192.168.10.0/24 #####
##### The streaming data from 192.168.10.220 is stored at 192.168.1.5/32 and is routed via eth1 #####

##### NAS 192.168.1.5 eth1 to PoE Camera 192.168.10.220 eth2 #####
##### Notice this is table 1 as we set it up to use eth1 #####
    set protocols static table 1 interface-route 0.0.0.0/0 next-hop-interface eth1
    set firewall modify pia_route rule 10 description 'Camera 220'
    set firewall modify pia_route rule 10 source address 192.168.1.5/32
    set firewall modify pia_route rule 10 destination address 192.168.10.220
    set firewall modify pia_route rule 10 modify table 1
    set interfaces ethernet eth1 firewall in modify pia_route
    set firewall modify pia_route rule 10 action modify

##### NAS 192.168.1.5 eth1 to PoE Camera 192.168.10.221 eth2 #####
    set protocols static table 1 interface-route 0.0.0.0/0 next-hop-interface eth1
    set firewall modify pia_route rule 20 description 'Camera 221'
    set firewall modify pia_route rule 20 source address 192.168.1.5/32
    set firewall modify pia_route rule 20 destination address 192.168.10.221
    set firewall modify pia_route rule 20 modify table 1
    set firewall modify pia_route rule 20 action modify

##### NAS 192.168.1.5 eth1 to LAN on eth1 192.168.1.0/24 #####
    set protocols static table 1 interface-route 0.0.0.0/0 next-hop-interface eth1
    set firewall modify pia_route rule 30 description 'Local NAS to LAN'
    set firewall modify pia_route rule 30 source address 192.168.1.5/32
    set firewall modify pia_route rule 30 destination address 192.168.1.0/24
    set firewall modify pia_route rule 30 modify table 1
    set firewall modify pia_route rule 30 action modify   

##### NAS 192.168.1.5 eth1 to external DSN via eth0 (WAN) #####
##### Notice this is table 2, as we switched from eth1 interface to eth0 #####
    set protocols static table 2 interface-route 0.0.0.0/0 next-hop-interface eth0
    set firewall modify pia_route rule 40 description 'External DNS'
    set firewall modify pia_route rule 40 source address 192.168.1.5/32
    set firewall modify pia_route rule 40 destination port 53
    set firewall modify pia_route rule 40 modify table 2
    set firewall modify pia_route rule 40 protocol tcp_udp
    set firewall modify pia_route rule 40 action modify   

##### NAS 192.168.1.5 eth1 to external DSN via eth0 (WAN) #####
    set firewall modify pia_route rule 50 action modify
    set firewall modify pia_route rule 50 description 'External NTP'
    set firewall modify pia_route rule 50 destination port 123
    set firewall modify pia_route rule 50 modify table 2
    set firewall modify pia_route rule 50 protocol udp
    set firewall modify pia_route rule 50 source address 192.168.1.5/32
    set firewall modify pia_route rule 50 source port 123

##### NAS 192.168.1.5 eth1 to all other traffic not listed above through vtun0 (PIA VPN) #####
##### Notice this is table 3, as we switched from eth0 interface to vtun0 #####
    set protocols static table 3 interface-route 0.0.0.0/0 next-hop-interface vtun0
    set firewall modify pia_route rule 60 description 'PIA Route to vtun0'
    set firewall modify pia_route rule 60 source address 192.168.1.5/32
    set firewall modify pia_route rule 60 modify table 3
    set interfaces ethernet eth1 firewall in modify pia_route
    set firewall modify pia_route rule 60 action modify

##### To get an output which you can paste into the router which are the actual commands run the following: #####
    show configuration commands
    
    You can also grep somethign on the end if you are looking for a specific command(s)
    show configuration commands | grep pia_route
  
##### To view your firewall logs, I normally do this: #####
    tail -f -n 50 /var/logs/messages
    
    This will show you the last 50 lines of the /var/log/messages and tail it or stream it so you can watch in real-time
    This is super useful when trying to figure out why something isn't working or maybe it's blocked
    To see your logs which are blocked, make sure you enable logging on your firewall logs
    
    Example:
    Mar  8 21:16:30 ubnt kernel: [WAN_LOCAL-default-D]IN=eth0 OUT= MAC=fc:dc:dd:24:22:f7:00:3d:45:42:83:19:08:00 SRC=55.177.77.154 DST=44.10.100.101 LEN=40 TOS=0x00 PREC=0x00 TTL=243 ID=54321 PROTO=TCP SPT=56057 DPT=443 WINDOW=65535 RES=0x00 SYN URGP=0
    
    This is someone trying to reach my external interface to probe my system, which in this case is eth0 (WAN/Internet) of device
