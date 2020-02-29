# Ubituiti-ERL-OpenVPN
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
Hit the ESC key and type :wq
This will write the file and then quit out of vi
Next create the file userpass.txt
vi userpass.txt
You can retrieve your PIA PPTP/L2TP/SOCKS Username and Password by logging into PIA online, selecting My Account and scrolling down.
You will see PPTP/L2TP/SOCKS Username and Password.
On two lines put your PIA PPT/L2TP/Socks Username on the first line and on the second line put the Password
Example of what iw should look like in the userpass.txt file you creatd in /config/auth directory on the ERL:
x29283933333
WidD92F398msb
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

NOTE: replace your interface with interface your network is on.. i.e. eth0, eth1, eth2, switch etc
So if your network is on eth2, then change the last line above to be:
set interfaces ethernet eth2 firewall in modify pia_route

------------------------

If you want to have only 1 IP go through the VPN, then do exactly the same above, but change the two lines:
set service nat rule 5000 source address 192.168.1.0/24
set service nat rule 5001 source address 192.168.1.0/24
to
set service nat rule 5000 source address 192.168.1.5/31
set service nat rule 5001 source address 192.168.1.5/31

On the ERL Dashboard, you should now see the newly created interface which should be connected
Test to see if the network or individual system is going out the VPN.
A couple of ways is to use a web browser and go to https://ifconfig.me or if you are SSH'd into a shell run: curl ifconfig.me
This will display your external IP, which you should be able to determine if it's your normal assigned ISP IP or the VPN IP.

