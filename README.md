 CONFIGURATION COMPLÈTE ZÉRO - TOUS VLANs inclus
RESET + VLAN10,20,30,40,50 + Switch Cisco + TP-Link AP + Accès PC bureau

🛑 ÉTAPE 1 : RESET MIKROTIK L009 (WinBox)
text
**WinBox → System → Reset Configuration**
☑ No Default Configuration  
☑ Skip Backup → Reset Configuration
→ Reconnecte 192.168.88.1 (admin/vide)
🔌 ÉTAPE 2 : CONFIG MIKROTIK COMPLÈTE (Terminal)
Copie-colle TOUT ce bloc :

bash
# =====================================================
# WAN Internet (ether1)
# =====================================================
/ip dhcp-client add interface=ether1 disabled=no

# =====================================================
# ether5 = PC/Laptop TEST (10.10.10.0/24)
/ip address add address=10.10.10.1/24 interface=ether5
/ip pool add name=pool-local ranges=10.10.10.10-10.10.10.100
/ip dhcp-server add name=dhcp-local interface=ether5 address-pool=pool-local
/ip dhcp-server network add address=10.10.10.0/24 gateway=10.10.10.1 dns-server=8.8.8.8,1.1.1.1
/ip dhcp-server enable dhcp-local

# =====================================================
# VLANs TRUNK ether2 (Switch Cisco)
/interface bridge add name=bridge-vlans
/interface bridge port add bridge=bridge-vlans interface=ether2

# Création VLANs
/interface vlan add name=vlan10-admin vlan-id=10 interface=bridge-vlans
/interface vlan add name=vlan20-house vlan-id=20 interface=bridge-vlans
/interface vlan add name=vlan30-gaming vlan-id=30 interface=bridge-vlans
/interface vlan add name=vlan40-guest vlan-id=40 interface=bridge-vlans
/interface vlan add name=vlan50-wifi vlan-id=50 interface=bridge-vlans

# =====================================================
# ADRESSES IP TOUS VLANs (Gateway)
/ip address add address=10.10.10.1/24 interface=vlan10-admin
/ip address add address=10.10.20.1/24 interface=vlan20-house
/ip address add address=10.10.30.1/24 interface=vlan30-gaming
/ip address add address=10.10.40.1/24 interface=vlan40-guest
/ip address add address=10.10.50.1/24 interface=vlan50-wifi

# =====================================================
# DHCP TOUS VLANs
# VLAN10 ADMIN
/ip pool add name=pool-vlan10 ranges=10.10.10.10-10.10.10.100
/ip dhcp-server add name=dhcp-vlan10 interface=vlan10-admin address-pool=pool-vlan10
/ip dhcp-server network add address=10.10.10.0/24 gateway=10.10.10.1 dns-server=8.8.8.8,1.1.1.1

# VLAN20 HOUSE
/ip pool add name=pool-vlan20 ranges=10.10.20.10-10.10.20.100
/ip dhcp-server add name=dhcp-vlan20 interface=vlan20-house address-pool=pool-vlan20
/ip dhcp-server network add address=10.10.20.0/24 gateway=10.10.20.1 dns-server=8.8.8.8,1.1.1.1

# VLAN30 GAMING
/ip pool add name=pool-vlan30 ranges=10.10.30.10-10.10.30.100
/ip dhcp-server add name=dhcp-vlan30 interface=vlan30-gaming address-pool=pool-vlan30
/ip dhcp-server network add address=10.10.30.0/24 gateway=10.10.30.1 dns-server=8.8.8.8,1.1.1.1

# VLAN40 GUEST
/ip pool add name=pool-vlan40 ranges=10.10.40.10-10.10.40.100
/ip dhcp-server add name=dhcp-vlan40 interface=vlan40-guest address-pool=pool-vlan40
/ip dhcp-server network add address=10.10.40.0/24 gateway=10.10.40.1 dns-server=8.8.8.8,1.1.1.1

# VLAN50 WIFI
/ip pool add name=pool-vlan50 ranges=10.10.50.10-10.10.50.100
/ip dhcp-server add name=dhcp-vlan50 interface=vlan50-wifi address-pool=pool-vlan50
/ip dhcp-server network add address=10.10.50.0/24 gateway=10.10.50.1 dns-server=8.8.8.8,1.1.1.1

# Active TOUS DHCP VLANs
/ip dhcp-server enable dhcp-vlan10, dhcp-vlan20, dhcp-vlan30, dhcp-vlan40, dhcp-vlan50

# =====================================================
# NAT Internet TOUS réseaux
/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1

# =====================================================
# VPN + SSH Accès PC bureau
/ip pool add name=vpn-pool ranges=192.168.100.10-192.168.100.50
/ppp profile add name=vpn-profile local-address=192.168.100.1 remote-address=vpn-pool dns-server=8.8.8.8,1.1.1.1
/interface l2tp-server server set enabled=yes default-profile=vpn-profile
/ppp secret add name=vpn-user password="VPN123!" profile=vpn-profile service=l2tp

# SSH ouvert partout
/ip service set ssh address=0.0.0.0/0 disabled=no
/ip service set www address=0.0.0.0/0 disabled=no

# =====================================================
# SAUVEGARDE
/export compact file=lab-complet-20260413.rsc
/system backup save name=lab-complet-20260413
🔌 ÉTAPE 3 : CONNEXIONS PHYSIQUES
text
ether1 → Routeur FAI (WAN Internet)
ether2 → Switch Cisco Gi0/24 (TRUNK VLANs 10-50)  
ether5 → Laptop/PC test (10.10.10.x)
Switch Gi0/1-4 → PCs local technique (VLAN10 ADMIN)
Switch Gi0/6 → TP-Link AP (VLAN20 HOUSE)
⚙️ ÉTAPE 4 : SWITCH CISCO (Câble console)
text
# === RESET USINE ===
delete vlan.dat
erase startup-config
reload

# === CONFIG COMPLÈTE ===
enable
conf t
hostname Switch-Lab

# VLANs
vlan 10
 name ADMIN
exit
vlan 20
 name HOUSE
exit
vlan 30
 name GAMING
exit
vlan 40
 name GUEST
exit
vlan 50
 name WIFI
exit

# TRUNK MikroTik
int gi0/24
switchport trunk encapsulation dot1q
switchport mode trunk  
switchport trunk allowed vlan 10,20,30,40,50
no shutdown
exit

# VLAN10 ADMIN
int range gi0/1-4
switchport mode access
switchport access vlan 10
no shutdown
exit

# VLAN20 HOUSE + AP
int range gi0/6-10
switchport mode access
switchport access vlan 20
no shutdown
exit

# Management VLAN10
ip default-gateway 10.10.10.1
interface vlan 10
ip address 10.10.10.254 255.255.255.0
no shutdown
exit

username admin privilege 15 secret Admin123!
enable secret Cisco123!
line vty 0 15
 login local
 transport input ssh telnet
exit

wr
📡 ÉTAPE 5 : TP-LINK AP
text
AP LAN → Switch Gi0/6
http://192.168.0.254 → Mode AP
IP statique : 10.10.20.10/24 | Gateway 10.10.20.1
SSID : Maison-WiFi (WPA2-PSK)
✅ ÉTAPE 6 : TESTS
text
**Vérifications MikroTik** :
/ip dhcp-server print          # TOUS "R" (running)
/ip address print              # 6 adresses (ether5 + 5 VLANs)
/interface vlan print           # 5 VLANs

**PC ether5** : 10.10.10.10-100
**Switch VLAN10** : ssh admin@10.10.10.254
**WiFi VLAN20** : 10.10.20.x
**VPN PC bureau** : L2TP → IP publique → vpn-user/VPN123!
📋 PLAN ADRESSES FINAL
text
ether5 test     : 10.10.10.1/24
VLAN10 ADMIN    : 10.10.10.1/24 (Switch 10.10.10.254)
VLAN20 HOUSE    : 10.10.20.1/24 (AP 10.10.20.10)  
VLAN30 GAMING   : 10.10.30.1/24
VLAN40 GUEST    : 10.10.40.1/24
VLAN50 WIFI     : 10.10.50.1/24
VPN             : 192.168.100.1/24
Copie-colle le BLOC MIKROTIK COMPLET → RESET switch → Branche → TEST VPN = LAB 100% !
