---
title: Add route for split VPN
description: Adding an additional route to a VPN connection in Windows 11
date: 2024-11-30
categories:
  - VPN
  - Quick Tips
---
# Adding an additional route to a VPN connection in Windows 11

If you're running a split VPN in Windows 11 (i.e. one which sends only certain traffic over the VPN and does not use the VPN as the default gateway), you may wish to add specific IP addresses to also be routed over the VPN.  An example of this might be a service which is only accessible via a certain IP address.

Assuming you've already set up your VPN connection in Windows, you can add a route using the following PowerShell command:

```powershell
Add-VpnConnectionRoute -Name '<Split VPN Name>' -DestinationPrefix <123.45.67.89>/32
```

Where:

- `<Split VPN Name>` is the name of the VPN connection you wish to add the route to
- `<123.45.67.89>` is the IP address you wish to route over the VPN rather than via your default gateway/normal internet connection

For example, if you have a VPN connection named `My VPN` and you wish to route traffic to `216.58.204.68` over the VPN, you would use the following command:

```powershell
Add-VpnConnectionRoute -Name 'My VPN' -DestinationPrefix 216.58.204.68/32
```