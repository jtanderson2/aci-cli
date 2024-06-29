# Cisco ACI CLI

Useful CLI commands to run from the APIC or switches to glean information around the configurational and operational state of the fabric. 

> NOTE: The emulated NXOS running on the APIC can be used for confiugration purposes, though due to the complex underlying obect model it's easy to get in a muddle! My advice, stick to the GUI for configuration until you're ready to use the API!

---

### NXOS L1/L2 show commands

Many standard nxos commands work on the leaf switches or can be run from the apic when proceeding with 'fabric <node-id>'

Examples from leafs:

```
show port-channel summary
show interface ethernet 1/24
show mac address-table interface ethernet 1/24
```

Example from APIC

```
fabric 101 show port-channel summary
```

Any commands that output a vlan (eg show mac address-table) will be referencing the internal vlan and not the encap used on leaf ports; use the follow command to translate:

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



