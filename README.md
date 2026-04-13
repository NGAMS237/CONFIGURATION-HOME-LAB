MikroTik L009 Lab - VLANs + Switch Cisco + TP-Link AP + VPN
[

Lab domestique CCNA : VLANs segmentés, routage inter-VLAN, VPN distant, QoS gaming

📋 Table des matières
Vue d'ensemble

Topologie réseau

Prérequis matériels

Configuration MikroTik L009

Configuration Switch Cisco

Configuration TP-Link AP

Accès distant PC bureau

Vérifications

Dépannage

Vue d'ensemble
Configuration complète d'un lab réseau domestique avec :

5 VLANs segmentés (Admin, House, Gaming, Guest, Wifi)

Routage inter-VLAN sur MikroTik L009

Trunk 802.1Q vers switch Cisco

VPN L2TP/IPsec pour accès distant

Gestion QoS (gaming prioritaire)

Topologie réseau
text
Internet ← Routeur FAI ← ether1 MikroTik L009
                      ↕ ether2 (TRUNK VLANs 10-50)
                    Switch Cisco Gi0/24
                      ↕ Gi0/1-4 VLAN10 ADMIN (PCs local)
                      ↕ Gi0/6-10 VLAN20 HOUSE (TP-Link AP)
PC Bureau ← VPN L2TP ← ether5 MikroTik (10.10.10.x test)
Prérequis matériels
text
✅ MikroTik L009UiGS-RM (8 ports GbE + SFP)
✅ Switch Cisco géré (VLANs/trunking)
✅ TP-Link AP (Omada EAP/EAP225 recommandé)
✅ Câble console RJ45-DB9 (switch Cisco)
✅ Laptop WiFi (config + tests)
✅ Câbles Ethernet Cat5e/Cat6
Configuration MikroTik L009
1. Reset usine
text
WinBox → System → Reset Configuration
☑ No Default Configuration ☑ Skip Backup → Reset
2. Fichier de configuration complet .rsc
bash
# MIKROTIK L009 LAB COMPLET - Copier-coller Terminal WinBox
# =====================================================

# WAN Internet
/ip dhcp-client add interface=ether1 disabled=no

# ether5 = PC test bureau (10.10.10.0/24)
/ip
