# EdgeOS-ExpressVPN-EdgeRouter-X
# This is a quick guide of how I added OpenVPN ExpressVPN to my network for only a subnet, being a /32 or /24 or whatever needed.

* Login into ExpressVPN
* Go to Setup Device and choose Manual Configuration
* Copy the username and passwords strings you will find on the right side bar.
* Download the vpn configuration to your computer from one of the locations you prefer.
* Open the file and edit the line ```auth-user-pass``` and replace it with ```auth-user-pass /config/auth/user-pass.txt``` and replace ```route-pull``` with ```route-nopull```

Now transfer the file to your EdgeRouter with scp or filezilla or whatever you choose. Even copy and paste works.
```
scp myopenvpnfilelocation.ovpn admin@192.168.3.1:/config/auth/vpn.ovpn
```
SSH into your EdgeRouter and login as root
```
sudo su -l
```
Create a file and insert the expressvpn username and password you got on the Manual Configuration section, pasting one on each line, then save the file
```
vi /config/auth/user-pass.txt
```
Now exit the root user and get into configuration and paste the following lines:
```
configure
set interfaces openvpn vtun0 config-file /config/auth/vpn.ovpn
set interfaces openvpn vtun0 description 'ExpressVPN'
set firewall modify express_vpn_route rule 10 description 'ExpressVPN'
set firewall modify express_vpn_route rule 10 source address 192.168.3.0/24
set firewall modify express_vpn_route rule 10 modify table 1
set protocols static table 1 interface-route 0.0.0.0/0 next-hop-interface vtun0
set interfaces switch switch0 firewall in modify express_vpn_route  
set service nat rule 5001 description 'ExpressVPN'
set service nat rule 5001 log disable
set service nat rule 5001 outbound-interface vtun0
set service nat rule 5001 type masquerade
commit; save;
exit
```
Just make sure you replace the 192.168.3.0/24 with your subnet or if you want to set only one device, use the ip with /32, like 192.168.3.15/32.
The VPN should be enabled by now. You can see the logs with this command:
```
grep openvpn /var/log/messages
```

The original idea was found on this website. Thanks for the ideas!
https://community.ui.com/questions/ExpressVPN-configuration-for-EdgeRouter/cbe24345-5a31-492d-9cd4-30976c26d142
