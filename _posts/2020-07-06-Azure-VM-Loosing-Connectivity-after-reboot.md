---
title: "Azure VM loosing connectivity post reboot"
date: 2020-11-21 17:45:00 +1000
categories: [Azure, Azure Compute]
tags: [Azure VMs, Domain Controllers, Active Directory]
---

## Environment Details
### On-Premise
- Two Domain Controllers runing windows Server 2016
    ```yaml
    OPDC01 - 10.10.1.10
    OPDC02 - 10.10.1.11
    ```

### Azure
- A Hybrid Vnet in Azure connected via VPN to on-premise network.
- Two Domain Controllers running Windows Server 2019 Datacenter edition.
    ```yaml
    AZDC01 - 192.168.1.10
    AZDC02 - 192.168.1.11
    ```
- Other couple of VMS running Windows Server 2019 Datacenter edition as member servers.
- Vnet uses custom DNS settings in the following order.
    ```yaml
    192.168.1.10 - AZDC01
    192.168.1.11 - AZDC02
    10.10.1.10 - OPDC01
    10.10.1.11 - OPDC02
    168.63.129.16 - Azure DNS
    ```
> **Note:** *All the VMS in Azure Vnet are configured as per Microsoft's recommended settings. They use static ip configuration configured via Azure and NOT within the guest OS*



## Behaviour
- When the VM reboots, it looses all connectivity with no pings or Remote desktop.
- you are unable to reset any settings via any of the extensions
- Boot diagnostics from the screenshot below shows that VM started, but connection icon on lower right shows disconnected.
- In conclusion the VM does not have any connectivity.


![Vm-reboot]({{"/assets/img/sample/Vm-reboot.png" | relative_url }})


## Resolution Setps
1. Go to the VM Domain Controller in azure which has lost th connectivity
2. On the left pane navigate to **'Support + Troubleshooting'** and select **'Serial Console'** (screenshot below for refeence)
    ![vm-nav]({{"/assets/img/sample/vm-nav.png" | relative_url }})
3. Log in into serial console, type ```cmd``` and enter
4. Type ```ch -si 1``` and then enter. this will connect to the chanel stream 1
5. Log in using the domain admin account or local admin account. Be careful while typing your credentials. you will not be able to copy and paste niether backspace will work.
6. Enter ```ipconfig /renew``` this will bring back the netwok.


## Summary
I have seen this behavior only when you deploy additional domain controllers in a hybrid scenario and only with the first Domain Controller Deployed ```AZDC01```, where you have your on-prem infrastructure and later extended it to Azure via a Vnet and VPN Gateway. I have not seen any similar issues with standalone Domain Controllers in Azure. No other scenarios have been further tested like migrating all the FSMO roles to the Domain controller in Azure