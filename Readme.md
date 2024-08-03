For detailed steps and configurations, follow along with the full video [**here**](https://youtu.be/v5_UdsUgKv0).

## Overview
This document provides the steps and configuration details for setting up a site-to-site VPN between an AWS VPC and an on-premises network using a Fortigate firewall.

## Architecture
![Architecture](/assets/images/projects/fortinetxaws/architecture.png)

## AWS Configuration

### VPC Configuration
- **VPC CIDR**: 10.12.0.0/16
- **Public Subnet CIDR**: 10.12.1.0/24
- **Private Subnet CIDR**: 10.12.2.0/24

> **⚠️ IMPORTANT: Make sure that the CIDR doesn't overlap with your on-prem network.**

### VPN Gateway

1. **Create a Customer Gateway (CGW)**
   - Name: `On-Prem-CGW`
   - IP Address: `<On-Premises Public IP>`
   - BGP ASN: `65000`

2. **Create a Virtual Private Gateway (VGW)**
   - Name: `AWS-VPG`
   - Attach the VGW to the VPC (10.12.0.0/16)

3. **Create a Site-to-Site VPN Connection**
   - Name: `AWS-OnPrem-VPN`
   - VPN Gateway: `AWS-VPG`
   - Customer Gateway: `On-Prem-CGW`
   - Routing Options: Static
   - Static IP prefixes: `10.11.12.0/24`
   - Local IPv4: `10.11.12.0/24`
   - Remote IPv4: `10.12.0.0/16`
   - Download Configuration
   - **Follow the configuration notes**

### Route Table Updates
1. **Update Route Table for Public Subnet**
   - Destination: `10.11.12.0/24`
   - Target: `AWS-VPG`

## On-Premises Configuration

### Fortigate Firewall
1. **Create a custom VPN**
   - Name: `AWS-VPN`
   - Remote Gateway: `<AWS VPN Endpoint IP>`
   - Pre-shared Key: `<Pre-shared Key from AWS>`

2. **Configure Phase 1 and Phase 2**
   - Follow the configuration notes

3. **Set an IP address for tunnel** 
   - Set the IP address based on the configuration notes.

4. **Create Static Routes**
   - Destination: `10.12.0.0/16`
   - Gateway: `AWS-VPG`

### Firewall Policies
1. **Create Policies to Allow Traffic**
   - Incoming: `Local Network (10.11.12.0/24)`
   - Outgoing: `AWS-VPN (10.12.0.0/16)`
   - Source: `All`
   - Destination: `All`
   - Service: `All`
   - NAT: `Disabled`

2. **Clone and Reverse the 1st policy**
   - Enable the new policy

## Verification

### AWS
1. **Check VPN Connection Status**
   - Ensure the VPN status is `UP` on the AWS Management Console.

### On-Premises
1. **Check VPN Tunnel Status**
   - Ensure the VPN tunnel is `UP` on the Fortigate firewall.

2. **Ping Test**
   - From an EC2 instance, ping the firewall and vice versa.