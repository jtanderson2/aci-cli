# Cisco ACI CLI

CLI commands to run from the APIC or switches to glean information around the configuration and operational state of the fabric. Quicker than the GUI in most cases and useful for troubleshooting and planning.

> NOTE: The emulated NXOS running on the APIC can be used for configuration purposes, though due to the complex underlying obect model it's easy to get in a muddle! My advice, stick to the GUI for configuration until you're ready to use the API!

Far from comprehensive, work in progress...

---
### APIC Commands

Display the address and state of nodes registered with the fabric (useful to gain a quick snapshot):

```
acidiag fnvread

      ID   Pod ID                 Name    Serial Number         IP Address    Role        State   LastUpdMsgId
--------------------------------------------------------------------------------------------------------------
     101        1         AAALSW101      F123456789   10.0.152.66/32    leaf         active   0
     102        1         AAALSW102      G123456789   10.0.152.64/32    leaf         active   0
     201        1         AAASSW201      H123456789   10.0.152.65/32    spine        active   0
```

Show overview of APICs and their health:

```
show controller
```

Show detailed information fo APICs and cluster info; useful for troubleshooting cluster issues:

```
acidiag avread
```

Show tenant info:

```
show tenant
```

Show VRFs and associated tenants:

```
show vrf
```

Show VPC mappings across fabric (policy group - pc id - vpc id - ports - leafs):

```
show vpc map
```

Show running config on leaf (of limited use, but interesting!):

```
show running-config leaf 101
```

Show physical domains, associated VLANs and which EPGs are using them:

```
show vlan-domain
show vlan-domain name BOB_PD
show vlan-domain name BOB_PD detail
```

Show VMM domain EPGs and VLANS:

```
show vmware domain name AAAVDS101 epg
```

---
### NXOS L1/L2 show commands

Many standard nxos commands work on the leaf switches or can be run from the APIC when proceeding with 'fabric \<node-id\>'

Examples from leafs:

```
show port-channel summary
show interface ethernet 1/24
show mac address-table interface ethernet 1/24
```

Example from APIC:

```
fabric 101 show port-channel summary
```

Any commands that output a vlan (eg 'show mac address-table') will be referencing the internal vlan and not the encap used on leaf ports; use the follow command to translate:

```
show system internal epm vlan 20

+----------+---------+-----------------+----------+------+----------+-----------
   VLAN ID    Type      Access Encap     Fabric    H/W id  BD VLAN    Endpoint
                        (Type Value)     Encap                          Count
+----------+---------+-----------------+----------+------+----------+-----------
 20           FD vlan 802.1Q         22 10413      55     14         5
```

Where:
- VLAN ID = internal VLAN
- Access  Encap = vlan used on leaf ports
- Fabric encap = VXLAN Network Identifier (VNID)

Show detailed information about endpoints; 'all' can be replaced with 'ip, mac, vlan, interface' to drill down

```
show system internal epm endpoint all 
```

---
### NXOS L3 show commands

VRF format is \<TENANT\>:\<VRF\>, except when running show commands in the underlay network, where it is just 'overlay-1' (obviously :-) 

Examples from leafs:

```
show ip route vrf AAA:PROD_VRF
show ip interface brief vrf AAA:PROD_VRF
show bgp ipv4 unicast vrf AAA:PROD_VRF
show bgp ipv4 unicast summary vrf AAA:PROD_VRF
show bgp ipv4 unicast neighbors 1.1.1.1 advertised-routes vrf AAA:PROD_VRF
show bgp ipv4 unicast neighbors 1.1.1.1 routes vrf AAA:PROD_VRF

! show all vrfs present on leaf
show vrf
```

Example from APIC:

```
fabric 101 show bgp ipv4 unicast vrf AAA:PROD_VRF
```

---
### iPing

Traditional pings from the switches only work in mgmt tenant/VRF. For any other VRF, you need to use iping (assumes valid L3 interface in VRF):

```
iping -V AAA:PROD_VRF 1.1.1.1

! add source ip or interface
iping -V AAA:PROD_VRF -S 2.2.2.2 1.1.1.1
iping -V AAA:PROD_VRF -S vlan123 1.1.1.1

```

---
### Moquery

A CLI based tool running on the APIC to query the fabric via the REST API. This is a massive subject; this article is a great starting point and below a few examples I use regularly:

https://learnwithsalman.com/aci-moquery/

Find where vlan encaps are used on the fabric (with regex matching which can be used with egrep on any APIC command):
```
! match any vlan starting 21
moquery -c fvIfConn | grep dn | egrep 'vlan-21'

! match exactly vlan 21
moquery -c fvIfConn | grep dn | egrep '\[vlan-21\]'

! match any vlan 13xx
moquery -c fvIfConn | grep dn | egrep 'vlan-13[0-9]{2}\]'
```

Display all instances of active faults for a given code:
```
moquery -c faultInst -f 'fault.Inst.code=="F0467"'
```

