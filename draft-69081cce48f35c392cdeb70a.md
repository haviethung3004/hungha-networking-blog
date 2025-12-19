---
title: "How to Build a GCP HA VPN to a Palo Alto Firewall in PNETLab"
slug: how-to-build-a-gcp-ha-vpn-to-a-palo-alto-firewall-in-pnetlab

---

Wellcome everyone come to my first blog, in this blog, i will introduce how to do a VPN lab from your Pnetlab or Eve which behind NAT to reach to GCP using HA VPN from my real case.

## Topology

## Prerequisites

* You can register trial from GCP, you will have 300$ for 3 months to do lab
    

## Setting Up the Environment

## Connecting GCP HA VPN to Palo Alto Firewall

## Configuring GCP HA VPN

1. Create a new VPC for testing
    

In this lab, I create a new VPC to isolate with the name “gcp-havpn-lab”, subnet name “vpc-demo-vpn-gw1” with subnet is 192.168.1.0/24

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1762423202224/928f2687-52c2-4a94-9d6e-ac4d7b092608.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1762422925531/adf29b25-4e92-46e9-bf24-9a39af80eb81.png align="center")

Rest option keep it as default then click create.

2. You will now need to create an HA VPN wizard
    

## Configuring the Palo Alto Firewall

## Testing the Connection

## Conclusion

## Additional Resources