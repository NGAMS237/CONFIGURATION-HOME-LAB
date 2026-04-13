# CONFIGURATION-HOME-LAB
Laboratoire personnel

bash
# =====================================================
# MIKROTIK L009 - CONFIGURATION COMPLÈTE 2026-04-13
# VLANs 10,20,30,40,50 + VPN + Accès PC bureau
# =====================================================

# =====================================================
# 1. WAN Internet (ether1)
# =====================================================
/ip dhcp-client add interface=ether1 disabled=no

# =====================================================
# 2. ether5 = PC/Laptop TEST BUREAU (10.10.10.0/24)
# =====================================================
/ip address add address=10.10.10.1/24 interface=ether5
/ip pool add name=pool-local ranges=10.10.10.10-10.10.10.100
/ip dhcp-server add name=dhcp-local interface=ether5 address-pool=pool-local
/ip dhcp-server network add address=10.10.10.0/24 gateway=10.10.10.1 dns-server=8.8.8.8,1.1.1.1
/ip dhcp-server enable dhcp-local

# =====================================================
# 3. VLANs TRUNK ether2 (Switch Cisco Gi0/24)
# =====================================================
/interface bridge add name=bridge-vlans
/interface bridge port add bridge=bridge-vlans interface=ether2

# Création des 5 VLANs
/interface vlan add name=vlan10-admin vlan-id=10 interface=bridge-vlans
/interface vlan add name=vlan20-house vlan-id=20 interface=bridge-vlans
/interface vlan add name=vlan30-gaming vlan-id=30 interface=bridge-vlans
/interface vlan add name=vlan40-guest vlan-id=40 interface=bridge-vlans
/interface vlan add name=vlan50-wifi vlan-id=50 interface=bridge-vlans

# =====================================================
# 4. ADRESSES IP Gateway TOUS VLANs
# =====================================================
/ip address add address=10.10.10.1/24 interface=vlan10-admin
/ip address add address=10.10.20.1/24 interface=vlan20-house
/ip address add address=10.10.30.1/24 interface=vlan30-gaming
/ip address add address=10.10.40.1/24 interface=vlan40-guest
/ip address add address=10.10.50.1/24 interface=vlan50-wifi

# =====================================================
# 5. DHCP COMPLET - TOUS VLANs
# =====================================================
# VLAN10 ADMIN (local technique, câble console)
/ip pool add name=pool-vlan10 ranges=10.10.10.10-10.10.10.100
/ip dhcp-server add name=dhcp-vlan10 interface=vlan10-admin address-pool=pool-vlan10
/ip dhcp-server network add address=10.10.10.0/24 gateway=10.10.10.1 dns-server=8.8.8.8,1.1.1.1

# VLAN20 HOUSE (TP-Link AP, PCs famille)
/ip pool add name=pool-vlan20 ranges=10.10.20.10-10.10.20.100
/ip dhcp-server add name=dhcp-vlan20 interface=vlan20-house address-pool=pool-vlan20
/ip dhcp-server network add address=10.10.20.0/24 gateway=10.10.20.1 dns-server=8.8.8.8,1.1.1.1

# VLAN30 GAMING (PS5/PS4)
/ip pool add name=pool-vlan30 ranges=10.10.30.10-10.10.30.100
/ip dhcp-server add name=dhcp-vlan30 interface=vlan30-gaming address-pool=pool-vlan30
/ip dhcp-server network add address=10.10.30.0/24 gateway=10.10.30.1 dns-server=8.8.8.8,1.1.1.1

# VLAN40 GUEST (visiteurs)
/ip pool add name=pool-vlan40 ranges=10.10.40.10-10.10.40.100
/ip dhcp-server add name=dhcp-vlan40 interface=vlan40-guest address-pool=pool-vlan40
/ip dhcp-server network add address=10.10.40.0/24 gateway=10.10.40.1 dns-server=8.8.8.8,1.1.1.1

# VLAN50 WIFI (WiFi futurs)
/ip pool add name=pool-vlan50 ranges=10.10.50.10-10.10.50.100
/ip dhcp-server add name=dhcp-vlan50 interface=vlan50-wifi address-pool=pool-vlan50
/ip dhcp-server network add address=10.10.50.0/24 gateway=10.10.50.1 dns-server=8.8.8.8,1.1.1.1

# Active TOUS les DHCP VLANs
/ip dhcp-server enable dhcp-vlan10,dhcp-vlan20,dhcp-vlan30,dhcp-vlan40,dhcp-vlan50

# =====================================================
# 6. NAT Internet (TOUS les VLANs vers FAI)
/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1

# =====================================================
# 7. VPN L2TP + SSH (PC bureau accès distant)
/ip pool add name=vpn-pool ranges=192.168.100.10-192.168.100.50
/ppp profile add name=vpn-profile local-address=192.168.100.1 remote-address=vpn-pool dns-server=8.8.8.8,1.1.1.1
/interface l2tp-server server set enabled=yes default-profile=vpn-profile service=l2tp
/ppp secret add name=vpn-user password="VPN123!" profile=vpn-profile service=l2tp

# SSH + WebFig ouverts partout
/ip service set ssh address=0.0.0.0/0 disabled=no
/ip service set www address=0.0.0.0/0 disabled=no
/ip service set winbox address=0.0.0.0/0 disabled=no

# =====================================================
# 8. SAUVEGARDE
# =====================================================
/export compact file=lab-complet-20260413.rsc
/system backup save name=lab-complet-20260413

# =====================================================
# VÉRIFICATIONS (exécute après config)
# =====================================================
# /ip dhcp-server print          # TOUS "R" (running)
# /ip address print              # 6 adresses + WAN
# /interface vlan print          # 5 VLANs
# /interface bridge port print   # ether2 dans bridge-vlans
💾 SAUVEGARDE : Copie ce fichier
1. Copie tout le texte ci-dessus dans Notepad
2. Sauvegarde : mikrotik-lab-complet-20260413.rsc
3. WinBox → Files → Upload ce fichier
4. Terminal : /import file=lab-complet-20260413.rsc

🔌 CONNEXIONS PHYSIQUES
text
ether1 → Routeur FAI (Internet WAN)
ether2 → Switch Cisco Gi0/24 (TRUNK VLANs 10-50)
ether5 → Laptop/PC test bureau (10.10.10.x)
Switch Gi0/1-4 → PCs local technique (VLAN10 ADMIN)
Switch Gi0/6 → TP-Link AP (VLAN20 HOUSE)
📱 VPN PC BUREAU
text
Protocole : L2TP/IPsec PSK
Serveur : IP publique FAI
User : vpn-user
Pass : VPN123!
