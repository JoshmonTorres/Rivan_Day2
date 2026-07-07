
<!-- Your monitor number = 21 -->


## ⛅ Warm Up for Day 2.

<br>

### 🔧 Physically Connect the following:
| Device   | Port       |  -  | Port   | Switch   |
| ---      | ---        | --- | ---    | ---      |
| PC       | TunayNaLAN |  -  | fa0/1  | CoreBABA |
| WLC      | PoE        |  -  | fa0/2  | CoreBABA |
| AP       | PoE        |  -  | fa0/4  | CoreBABA |
| CUCM     | fe0/0      |  -  | fa0/3  | CoreBABA |
| ePhone 1 | Network    |  -  | fa0/5  | CoreBABA |
| ePhone 2 | Network    |  -  | fa0/7  | CoreBABA |
| Cam6     | Eth        |  -  | fa0/6  | CoreBABA |
| Cam8     | Eth        |  -  | fa0/8  | CoreBABA |
| CoreTAAS | fa0/10     |  -  | fa0/10 | CoreBABA |
| CoreTAAS | fa0/11     |  -  | fa0/11 | CoreBABA |
| CoreTAAS | fa0/12     |  -  | fa0/12 | CoreBABA |


<br>
<br>

---
&nbsp;

## Day 1 via Ansible

<br>
<br>

### Principles
1. __Agent-less Architecture__ - Low maintenance overhead by avoiding the installation of additional software across IT infrastructure.

<br>

2. __Simplicity__ - Automation playbooks use straightforward YAML syntax for code that reads like documentation. Ansible is also decentralized, using SSH with existing OS credentials to access to remote machines.

<br>

3. __Scalability and Flexibility__ - Easily and quickly scale the systems you automate through a modular design that supports a large range of operating systems, cloud platforms, and network devices.

<br>

4. __Idempotence and predictability__ - When the system is in the state your playbook describes, Ansible does not change anything, even if the playbook runs multiple times.


&nbsp;
---
&nbsp;


### Step 1 - Add routing to PC
~~~
!@cmd
route add 10.0.0.0 mask 255.0.0.0 10.21.1.4
route add 200.0.0.0 mask 255.255.255.0 10.21.1.4
~~~

<br>

### Step 2 - Add ip address and routing to device
~~~
!@coreTaas
conf t
 int vlan 1
  no shut
  ip add 10.21.1.2 255.255.255.0
  desc mgmtData-configuredManually
  exit
  enable secret pass
 line vty 0 14
  password pass
  transport input all
  login
  exec-timeout 0 0
  end
~~~

<br>

~~~
!@coreBaba
conf t
 int vlan 1
  no shut
  ip add 10.21.1.4 255.255.255.0
  desc mgmtData-configuredManually
  exit
 vlan 100
  name VOICEVLAN
  exit
 int vlan 100
  no shut
  ip add 10.21.100.4 255.255.255.0
  desc vlanMgmtVoice-configuredManually
  exit
 int fa 0/3
  sw mo ac
  sw ac vlan 100
  exit
 int gi0/1
  no switchport
  no shut 
  ip add 10.21.21.4 255.255.255.0
  exit
 ip routing
 ip route 0.0.0.0 0.0.0.0 10.21.21.1 120
 enable secret pass
 line vty 0 14
  password pass
  transport input all
  login
  exec-timeout 0 0
  end
~~~

<br>

~~~
!@cucm
conf t
 int fa0/0
  no shut
  ip add 10.21.100.8 255.255.255.0
  exit
 ip routing
 ip route 0.0.0.0 0.0.0.0 10.21.100.4 120
 enable secret pass
 line vty 0 14
  password pass
  transport input all
  login
  exec-timeout 0 0
  end
~~~

<br>

~~~
!@edge
conf t
 int gi 0/0/0
  ip add 10.21.21.1 255.255.255.0
  no shut
  exit
 ip routing
 ip route 10.21.0.0 255.255.0.0 10.21.21.4 120
 enable secret pass
 line vty 0 14
  password pass
  transport input all
  login
  exec-timeout 0 0
  end
~~~

&nbsp;
---
&nbsp;

### Step 3 - Telnet to each device
On SecureCRT:
| IP                 | Device   |
| ---                | ---      |
| 10.21.1.2	     | CoreTaas |
| 10.21.1.4      | CoreBaba |
| 10.21.100.8    | CUCM     |
| 10.21.21.1 | EDGE     |

&nbsp;
---
&nbsp;

### Step 4 - Note MAC Addresses
~~~
@CoreBaba
sh mac address-table
~~~

<br>


| Device               | Port  | MAC         |
| ---                  | ---   | ---         |
| Camera 6 MAC Address | fa0/6 | ___.___.___ |
| Camera 8 MAC Address | fa0/8 | ___.___.___ |
| Ephone 1 MAC Address | fa0/5 | ___.___.___ |
| Ephone 2 MAC Address | fa0/7 | ___.___.___ |

&nbsp;
---
&nbsp;

### Step 4 - Enable SSH
For an SSH connection to be established, the device must have:  
| Description                      | Command                                  |
| ---                              | ---                                      |
| a non-default hostname           | hostname coreBaba21                  |
| a domain name                    | ip domain name day1lab.com               |
| a local user account             | username admin privilege 15 secret pass  |
| generated crypto keys            | crypto key generate rsa modulus 2048     |
| SSH enabled                      | ip ssh version 2                         |
| enable remote access             | transport input all                      |
| enable remote user/pass login    | login local                              |

<br>

Extra lines  
| Description             | Command                      |
| ---                     | ---                          |
| password encryption     | service password-encryption  |
| no logs                 | no logging console           |
| no domain lookups       | no ip domain-lookup          |
| no timeout              | exec-timeout 0 0             |

<br>

~~~
!@coreTaas
conf t
 hostname coreTaas-21
 service password-encryption
 no logging console
 no ip domain-lookup
 ip domain name autoday1.com
 username admin privilege 15 secret pass
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
  exit
 crypto key generate rsa modulus 2048 label devs
 ip ssh rsa keypair-name devs
 ip ssh version 2
 end
~~~

<br>

~~~
!@coreBaba
conf t
 hostname coreBaba-21
 service password-encryption
 no logging console
 no ip domain-lookup
 ip domain name autoday1.com
 username admin privilege 15 secret pass
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
  exit
 crypto key generate rsa modulus 2048 label devs
 ip ssh rsa keypair-name devs
 ip ssh version 2
 end
~~~

<br>

~~~
!@cucm
conf t
 hostname cucm-21
 service password-encryption
 no logging console
 no ip domain-lookup
 ip domain name autoday1.com
 username admin privilege 15 secret pass
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
 crypto key generate rsa modulus 2048 label devs
 ip ssh rsa keypair-name devs
 ip ssh version 2
 end
~~~

<br>

~~~
!@edge
conf t
 hostname edge-21
 service password-encryption
 no logging console
 no ip domain-lookup
 ip domain name autoday1.com
 username admin privilege 15 secret pass
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
 crypto key generate rsa modulus 2048 label devs
 ip ssh rsa keypair-name devs
 ip ssh version 2
 end
~~~

&nbsp;
---
&nbsp;

### Step 5 - Run Virtual Machines
NetOps:  
  Name: NetOps-PH  
  
  | NetAdapter   |                    |
  | ---          | ---                |
  | NetAdapter   | NAT                |
  | NetAdapter 2 | VMNet2             |
  | NetAdapter 3 | VMNet3             |
  | NetAdapter 4 | Bridge (Replicate) |

<br>

CSR1000v:  
  Name: UTM-PH  
  
  | NetAdapter   |        |
  | ---          | ---    |
  | NetAdapter   | NAT    |
  | NetAdapter 2 | VMNet2 |
  | NetAdapter 3 | VMNet3 |

&nbsp;
---
&nbsp;


### Step 6 - Set IP address and Routing
~~~
!@UTM-PH
conf t
 hostname UTM-PH
 enable secret pass
 service password-encryption
 no logging cons
 no ip domain lookup
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.11 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.11 255.255.255.0
  no shut
 int g3
  ip add 11.11.11.113 255.255.255.224
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 end
wr
!
~~~


<br>


__NetOps-PH Setup__
> Login: root
> Pass: C1sc0123

<br>

1. Get the MAC Address for the Bridge connection  
VMWare > NetOps-PH Settings > NetAdapter (2, 3, & 4) > Advance > MAC Address  

| NetAdapter   | MAC Address      | VM Interface | ENS     |
| ---          | ---              | ---          | ---     |
| NetAdapter 2 | ___.___.___.___  | ens___       |  ens192 |
| NetAdapter 3 | ___.___.___.___  | ens___       |  ens224 |
| NetAdapter 4 | ___.___.___.___  | ens___       |  ens256 |

<br>

2. Get Network-VM Mapping
~~~
!@NetOps-PH
ip -br link
~~~

<br>

3. Modify Interface IP  
VMNet2:  192.168.102.6/24  
VMNet3:  11.11.11.100/27  
Bridged: 10.21.1.6/24  

<br>

~~~
!@NetOps-PH
ifconfig ens192 192.168.102.6 netmask 255.255.255.0 up
ifconfig ens224 11.11.11.100 netmask 255.255.255.224 up
ifconfig ens256 10.21.1.6 netmask 255.255.255.0 up
~~~

<br>

Verify:
~~~
!@NetOps-PH
ip -4 addr

nmcli connection show
netstat -rn
~~~

<br>

or  

<br>

__Using Network Management CLI for persistent IP.__

<br>

VMNet2:
~~~
!@NetOps-PH
nmcli connection add \
type ethernet \
con-name VMNET2 \
ifname ens192 \
ipv4.method manual \
ipv4.addresses 192.168.102.6/24 \
autoconnect yes

nmcli connection up VMNET2

nmcli connection add \
type ethernet \
con-name VMNET3 \
ifname ens224 \
ipv4.method manual \
ipv4.addresses 11.11.11.100/27 \
autoconnect yes

nmcli connection up VMNET3

nmcli connection add \
type ethernet \
con-name BRIDGED \
ifname ens256 \
ipv4.method manual \
ipv4.addresses 10.21.1.6/24 \
autoconnect yes

nmcli connection up BRIDGED

ip route add 10.0.0.0/8 via 10.21.1.4 dev ens256
ip route add 200.0.0.0/24 via 10.21.1.4 dev ens256
ip route add 0.0.0.0/0 via 11.11.11.113 dev ens224
~~~

&nbsp;
---
&nbsp;

### Remote Access
Connect to Management Interfaces of Devices  
NetOps-PH: 192.168.102.6  
UTM-PH: 192.168.102.11  

&nbsp;
---
&nbsp;

### Step 7 - Exchange SSH Keys
Delete existing SSH Keys  
~~~
!@NetOps
rm -rf /root/.ssh/known_hosts
~~~

<br>

SSH to the ff devices:
~~~
!@NetOps
ssh admin@10.21.1.2
~~~

<br>

| IP                 | Device   |
| ---                | ---      |
| 10.21.1.2      | CoreTaas |
| 10.21.1.4      | CoreBaba |
| 10.21.100.8    | CUCM     |
| 10.21.21.1 | EDGE     |

<br>

- Accept the keys  
- End the SSH session  

&nbsp;
---
&nbsp;

### Step 8 - Download the Configs  
~~~
!@NetOps
git clone https://github.com/4rthurcyber08/SSHAUTOMATE
cd SSHAUTOMATE/_Ansible/Ex\ 02\ -\ Day1/__day1_project/
~~~

&nbsp;
---
&nbsp;

### Step 9 - Specify the MAC address of End devices
Camera MAC Addresses:  
~~~
!@NetOps
nano host_vars/cbaba_21.yml
~~~

<br>

EPhone MAC Addresses:
~~~
!@NetOps
nano host_vars/cucm_21.yml
~~~

&nbsp;
---
&nbsp;

### Step 10 - Run the Playbook
~~~
!@NetOps
ansible-playbook -i rivan_mkt.ini playbooks/deploy_21.yml --skip-tags ivrs
~~~


<br>
<br>

---
&nbsp;


### 🎯 Exercise 01: Add Loopback via Ansible

__Verify Hosts__
~~~
!@NetOps
ssh admin@192.168.102.11
~~~


<br>


__Hosts File__
~~~
!@NetOps
cd /etc/ansible ; nano hosts
~~~


<br>


~~~
[utmph]
192.168.102.11

[utmph:vars]
ansible_user=admin
ansible_password=pass
ansible_port=22
ansible_network_os=ios
ansible_connection=network_cli
# ansible_become=yes
# ansible_become_method=enable
# ansible_become_password=pass
~~~


<br>


__Playbook (addloop.yml)__
~~~
---
- name: addloop
  hosts: utmph
  gather_facts: yes
  tasks:
    - name: "Create Loopbacks"
      ios_command:
        commands:
          - conf t
          - int lo100
          - ip add 100.100.100.100 255.255.255.255
          - exit
          - int lo101
          - ip add 101.101.101.101 255.255.255.255
          - exit
          - int lo102
          - ip add 102.102.102.102 255.255.255.255
      vars:
        ansible_network_os: ios
~~~


<br>


__or__


<br>


__Using ios_config__
~~~
---
- name: addloop
  hosts: utmph
  gather_facts: yes

  tasks:
    - name: Create Loopbacks
      cisco.ios.ios_config:
        lines:
          - ip address 100.100.100.100 255.255.255.255
        parents: interface Loopback100

    - name: Create Loopback101
      cisco.ios.ios_config:
        lines:
          - ip address 101.101.101.101 255.255.255.255
        parents: interface Loopback101

    - name: Create Loopback102
      cisco.ios.ios_config:
        lines:
          - ip address 102.102.102.102 255.255.255.255
        parents: interface Loopback102
~~~


<br>


__or__


<br>


__Using Variables (Best Practice)__
~~~
---
- name: addloop
  hosts: utmph
  gather_facts: yes
  become: yes

  tasks:
    - name: Create loopbacks
      cisco.ios.ios_config:
        parents: "interface Loopback{{ item.id }}"
        lines:
          - ip address {{ item.ip }} 255.255.255.255
      loop:
        - { id: 100, ip: 100.100.100.100 }
        - { id: 101, ip: 101.101.101.101 }
        - { id: 102, ip: 102.102.102.102 }
~~~


<br>


__Run the Playbook__
~~~
!@NetOps
ansible-playbook -i hosts addloop.yml 
~~~


<br>
<br>

---
&nbsp;

## Terraform
~~~
!@Cisco
conf t
 username admin privilege 15 secret pass
 ip http authentication local
 ip http secure-server
 netconf-yang
 restconf
 line vty 0 14
  password pass
  login local
  transport input all
  exec-timeout 0 0
  end
~~~


<br>


~~~
!@NetOps
mkdir /etc/terraform ; cd /etc/terraform ; nano addloop.tf
~~~


<br>


__addloop.tf__
~~~
terraform {
  required_providers {
    iosxe = {
      source = "CiscoDevNet/iosxe"
    }
  }
}

provider "iosxe" {
  username = "admin"
  password = "pass"
  host     = "192.168.102.11"
}

resource "iosxe_interface_loopback" "example" {
  name               = 22
  description        = "Configured via Terraform"
  shutdown           = false
  ipv4_address       = "22.22.22.22"
  ipv4_address_mask  = "255.255.255.255"

}
~~~


<br>
<br>

---
&nbsp;

## SDN (Software Defined Networking)

__Control Plane (vSmart)__ - The part of the device that makes routing and network decisions. builds and maintains the network topology and make decisions on the traffic flows. The vSmart controller disseminates control plane information between WAN Edge devices, implements control plane policies and distributes data plane policies to network devices for enforcement.


<br>
<br>


__Data Plane (vEdge)__ - The part that forwards packets according to rules calculated by the control plane. WAN Edge devices are responsible for establishing secure connections for traffic forwarding, for security, encryption, Quality of Service (QoS) enforcement and more.


<br>
<br>


__Management Plane (vManage)__ - the instance where the user interacts with the device. It is responsible for central configuration and monitoring. The vManage controller is the centralized network management system that provides a single pane of glass GUI interface to easily deploy, configure, monitor and troubleshoot all Cisco SD-WAN components in the network.


<br>
<br>


__Orchestration Plane (vBond)__ - Policy distribution and Overlay control. Centralized Control. Assists in securely onboarding the SD-WAN WAN Edge routers into the SD-WAN overlay. The vBond controller, or orchestrator, authenticates and authorizes the SD-WAN components onto the network.


&nbsp;
---
&nbsp;


## SD-WAN

### Resource Requirements
__EVENG VM__
- 32GB RAM 
- 12 Core 
- Network Adapter 1 : NAT (208.8.8.0/24)
- Network Adapter 2 : VMNet15 (10.255.10.0/24)
- Network Adapter 3 : VMNet16 (10.69.255.0/29)


&nbsp;
---
&nbsp;


### SDWAN Procedure

__SDWAN Appliances Login__
> Login: admin
> Pass: C1sc0123

<br>

| Devices        | Ports |
| ---            | ---   |
| CLOUD          | 32902 |
| PKI SERVER     | 32913 |
|                |       |
| vManage        | 32897 |
| vSmart         | 32898 |
| vBond          | 32899 |
|                |       |
| vEdge-LUZON    | 32900 |
| vEdge-VISAYAS  | 32903 |
| vEdge-MINDANAO | 32904 |
|                |       |
| CSW-LUZON      | 32905 |
| CSW-VISAYAS    | 32906 |
| CSW-MINDANAO   | 32907 |

<br>

### STEP 1 - Set a host route for vManage GUI
~~~
!@cmd 
route add 10.69.255.13 mask 255.255.255.255 10.69.255.6
~~~


&nbsp;
---
&nbsp;


### STEP 2 - Bootstrap Configuration
| Device  | Port  |
| ---     | ---   |
| vManage | 32897 |
| CLOUD   | 32902 |

~~~
!@vManage
conf t
 system
  host-name Rivan-vManage
  site-id 10
  system-ip 10.1.10.2
  organization-name RIVANCORP
  vbond 172.16.10.3
  admin-tech-on-failure
 vpn 0
  int eth0
   ip add 172.16.10.2/29
   no shut
   tunnel-interface
    allow-service all
  ip route 0.0.0.0/0 172.16.10.6
 vpn 512
  int eth1
   ip add 10.69.255.13/30
   no shut
  ip route 0.0.0.0/0 10.69.255.14
  commit
  end
~~~


&nbsp;
---
&nbsp;


### STEP 3 - Access vManage GUI
> [!NOTE]
> It usually takes 10 MINUTES before vManage GUI is accessible

<br>

URL: https://10.69.255.13:8443  
- User: admin  
- C1sc0123  

<br>

Verification
~~~
!@vManage
request nms all status
show system status
~~~


&nbsp;
---
&nbsp;


### STEP 4 - vBond & vSmart
| Device  | Port  |
| ---     | ---   |
| vBond   | 32899 |
| vSmart  | 32898 |

<br>

~~~
!@vBond
conf t
 system
  host-name Rivan-vBond
  site-id 10
  system-ip 10.1.10.3
  organization-name RIVANCORP
  vbond 172.16.10.3 local vbond
  admin-tech-on-failure
 vpn 0
  int ge0/0
   ip add 172.16.10.3/29
   no shut
   tunnel-interface
    encapsulation ipsec
    allow-service all
  ip route 0.0.0.0/0 172.16.10.6
  commit
  end
~~~

<br>

~~~
!@vSmart
conf t
 system
  host-name Rivan-vSmart
  site-id 10
  system-ip 10.1.10.1
  organization-name RIVANCORP
  vbond 172.16.10.3
  admin-tech-on-failure
  vpn 0
   int eth0
    ip add 172.16.10.1/29
	no shut
	tunnel-interface
	 allow-service all
   ip route 0.0.0.0/0 172.16.10.6
   commit
   end
~~~

<br>

> [!IMPORTANT]
> Wait until the SDWAN Controllers are stable before turning on the vEdges

<br>

| Device         | Port  |
| ---            | ---   |
| vEdge-LUZON    | 32900 |
| vEdge-VISAYAS  | 32903 |


&nbsp;
---
&nbsp;


### STEP 5 - Configure BGP on CLOUD to establish connection with controllers
~~~
!@Cloud (BGP Config)
conf t
 int lo8
  ip add 8.8.8.8 255.255.255.255
 router bgp 1
  bgp log-neighbor-changes
  neighbor 192.168.20.1 remote-as 100
  neighbor 192.168.20.5 remote-as 100
  neighbor 192.168.20.9 remote-as 100
  address-family ipv4
   neighbor 192.168.20.1 activate
   neighbor 192.168.20.5 activate
   neighbor 192.168.20.9 activate
   neighbor 192.168.20.1 as-override
   neighbor 192.168.20.5 as-override
   neighbor 192.168.20.9 as-override
   network 8.8.8.8 mask 255.255.255.255
   network 192.168.20.0 mask 255.255.255.0
   network 172.16.10.0 mask 255.255.255.248
   network 10.69.255.0 mask 255.255.255.248
   end
~~~

<br>

~~~
!@vEdge-LUZON
show bgp routes
show bgp neighbors
~~~


&nbsp;
---
&nbsp;


### STEP 6 - Return to the vManage GUI then send the vEDGE list to all controllers

`Configuration` > `Certificates` > `Send to Controllers`


&nbsp;
---
&nbsp;


### STEP 7 - Configure Templates for SYSTEM

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `System`

~~~
Name: VE-SYSTEM
Desc: VE-SYSTEM

Console Baud Rate(bps): 9600
~~~


&nbsp;
---
&nbsp;


### STEP 8 - Configure Templates for BANNER

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `Banner`

~~~
Name: VE-BANNER
Desc: VE-BANNER

Login Banner: Welcome to RIVANCORP
MOTD Banner: Property owned by RIVANCORP
~~~


&nbsp;
---
&nbsp;


### STEP 9 - Configure Templates for VPN0

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN`

~~~
Name: VE-VPN0
Desc: VE-VPN0

VPN: 0
Name: TRANSPORT VPN

Ipv4 Route:
  Prefix: 0.0.0.0/0
  Gateway: NextHop
  Add Next Hop:
    Address: Device Specific
~~~


&nbsp;
---
&nbsp;


### STEP 10 - Configure Templates for VPN512

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN`

~~~
Name: VE-VPN512
Desc: VE-VPN512

VPN: 512
Name: MANAGEMENT VPN
~~~


&nbsp;
---
&nbsp;


### STEP 11 - Configure Templates for Interface IP address (VPN512)

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN Interface Ethernet`

~~~
Name: VE-VPNINT-VPN512-ETH0
Desc: VE-VPNINT-VPN512-ETH0

Shutdown: No
Interface Name: eth0
Description: MANAGEMENT INTERFACE

IPv4Add: Default
~~~


&nbsp;
---
&nbsp;


### STEP 12 - Configure Templates for Interface IP address (VPN0-G0/1)

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN Interface Ethernet`

~~~
Name: VE-VPNINT-VPN0-GIG01
Desc: VE-VPNINT-VPN0-GIG01

Shutdown: No
Interface Name: ge0/1
Description: TRANSPORT INTERFACE

IPv4Add: Device Specific

Tunnel Interface: On
Color: BIZ-INTERNET

Allow Service: All, NETCONF, SSH, BGP
NAT: On
~~~


&nbsp;
---
&nbsp;


### STEP 13 - Configure Templates for Interface IP address (VPN0-G0/0)

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN Interface Ethernet`

~~~
Name: VE-VPNINT-VPN0-GIG00
Desc: VE-VPNINT-VPN0-GIG00

Shutdown: No
Interface Name: ge0/0
Description: LAN INTERFACE

IPv4Add: Device Specific

Tunnel Interface: On
Color: Private1
Restrict: On
Allow Service: All, NETCONF, SSH, OSPF
~~~


&nbsp;
---
&nbsp;


### STEP 14 - Configure Templates for BGP

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `BGP`

~~~
Name: VE-BGP-VPN0
Desc: VE-BGP-VPN0

Shutdown: No
AS Number: Global 100

Neigbor:
  Address: 192.168.20.6
  Remote AS: 1
  Address Family: On
    Address Family: IPv4 Unicast
	Shutdown: No
~~~


&nbsp;
---
&nbsp;


### STEP 15 - Configure Device Templates

~~~
Device Model: vEdge Cloud
Device Role: SDWAN Edge
Template Name: VE-TEMP
Desc: VE-TEMP

Basic Info:
  System: VE-SYSTEM
 
Transport & Management VPN:
  VPN0: VE-VPN0
  Add: 
    BGP: VE-BGP-VPN0
	VPN Interface: VE-VPNINT-VPN0-GIG00
	VPN Interface: VE-VPNINT-VPN0-GIG01
  
  VPN512: VE-VPN512
  VPN Interface: VE-VPNINT-VPN512-ETH1
~~~


&nbsp;
---
&nbsp;


### STEP 16 - Attatch Device Tempates to vEdge-LUZON & VISAYAS


&nbsp;
---
&nbsp;


### STEP 17 - Assign Correct Values Per device


&nbsp;
---
&nbsp;


### 🎯 Exercise 02: Add Template for a user account

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>


<br>
<br>

---
&nbsp;

## Establish Connection between LUZON & VISAYAS
| Device      | Port  |
| ---         | ---   |
| CSW-LUZON   | 32905 |
| CSW-VISAYAS | 32906 |


~~~
!@CSW-LUZON
conf t
 hostname CSW-LUZON
 enable secret pass
 service password-encryption
 no logging console
 no ip domain lookup
 username admin priv 15 secret pass
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int lo0
  ip add 1.1.1.1 255.255.255.255
  exit
 int g0/0
  no sw
  ip add 172.16.1.2 255.255.255.252
  no shut
 int g0/1
  no sw
  ip add 10.1.1.2 255.255.255.252
  no shut
 router ospf 1
  router-id 1.1.1.1
  network 172.16.1.0 0.0.0.3 area 0
  network 1.1.1.1 0.0.0.0 area 0
  network 10.1.1.0 0.0.0.3 area 0
  passive-interface lo0
  end
~~~

<br>

~~~
!@CSW-VISAYAS
conf t
 hostname CSW-VISAYAS
 enable secret pass
 service password-encryption
 no logging console
 no ip domain lookup
 username admin priv 15 secret pass
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int lo0
  ip add 2.2.2.2 255.255.255.255
  exit
 int g0/0
  no sw
  ip add 172.16.5.2 255.255.255.252
  no shut
 int g0/1
  no sw
  ip add 10.1.2.2 255.255.255.252
  no shut
 router ospf 1
  router-id 2.2.2.2
  network 172.16.5.0 0.0.0.3 area 0
  network 2.2.2.2 0.0.0.0 area 0
  network 10.1.2.0 0.0.0.3 area 0
  passive-interface lo0
  end
~~~


&nbsp;
---
&nbsp;


### STEP 1 - Configure Templates for VPN1

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN`

~~~
Name: VE-VPN1
Desc: VE-VPN1

VPN: 1
NAME: DATA VPN

IPv4 Route: 
  Prefix: 0.0.0.0/0
  Gateway: VPN
  Enable VPN: On
~~~


&nbsp;
---
&nbsp;


### STEP 2 - Configure Templates VPN1 Interface [MODIFY]  - OPTIONAL

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > MODIFY `VE-VPNINT-VPN0GIG00`

~~~
Name: Retain
Desc: Retain

Shutdown: No
Name: ge0/0
Description: LAN INTERFACE

Ipv4 Address: Device Specific
~~~


&nbsp;
---
&nbsp;


### STEP 3 - Configure Templates for OSPF 

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `OSPF`

~~~
Name: VE-OSPF-VPN1
Desc: VE-OSPF-VPN1

Redistribute:
  Protocol: omp

Area:
  Area Num: 0
  Interface:
    Interface Name: ge0/0
	
	ADD x2

Advance:
  Originate: On
  Always: On
~~~


&nbsp;
---
&nbsp;


### STEP 4 - Update VE-TEMP (Device Template)

~~~
Service VPN:
  Add VPN: VE-VPN1
    OSPF: VE-OSPF-VPN1
	VPN Interface: VE-VPNINT-VPN0-GIG00

Remove GIG00 From VPN 0

Assign G0/0 With the IP based on the topology
~~~


<br>
<br>

---
&nbsp;

## vEDGE Onboarding
| Device       | Port  |
| ---          | ---   |
| CSW-MINDANAO | 32907 |
| PKI-Server   | 32913 |

~~~
!@CSW-MINDANAO
conf t
 hostname CSW-MINDANAO
 enable secret pass
 service password-encryption
 no logging console
 no ip domain lookup
 username admin priv 15 secret pass
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int lo0
  ip add 3.3.3.3 255.255.255.255
  exit
 int g0/0
  no sw
  ip add 172.16.9.2 255.255.255.252
  no shut
 int g0/1
  no sw
  ip add 10.1.3.2 255.255.255.252
  no shut
 router ospf 1
  router-id 3.3.3.3
  network 172.16.9.0 0.0.0.3 area 0
  network 3.3.3.3 0.0.0.0 area 0
  network 10.1.3.0 0.0.0.3 area 0
  passive-interface lo0
  end
~~~


<br>
<br>

---
&nbsp;


### 🎯 Exercise 03: Setup Trunking, Etherchannel, & MST on CoreTAAS & CoreBABA
~~~
!@CoreTAAS
conf t
 hostname coreTaas-21
 enable secret pass
 service password-encryption
 no logging console
 no ip domain-lookup
 line cons 0
  password pass
  login
  exec-timeout 0 0
 line vty 0 14
  password pass
  login
  exec-timeout 0 0
 int vlan 1
  no shut
  ip add 10.21.1.2 255.255.255.0
  desc DEFAULT-VLAN
 int vlan 10
  no shut
  ip add 10.21.10.2 255.255.255.0
  desc WIFI-VLAN
 int vlan 50
  no shut
  ip add 10.21.50.2 255.255.255.0
  desc CCTV-VLAN
 int vlan 100
  no shut
  ip add 10.21.100.2 255.255.255.0
  desc VOICE-VLAN
 end
~~~


<br>


~~~
!@CoreBABA
conf t
 hostname coreBaba-21
 enable secret pass
 service password-encryption
 no logging console
 no ip domain-lookup
 line cons 0
  password pass
  login
  exec-timeout 0 0
 line vty 0 14
  password pass
  login
  exec-timeout 0 0
 int gi 0/1
  no shut
  no switchport
  ip add 10.21.21.4 255.255.255.0
 int vlan 1
  no shut
  ip add 10.21.1.4 255.255.255.0
  desc DEFAULT-VLAN
 int vlan 10
  no shut
  ip add 10.21.10.4 255.255.255.0
  desc WIFI-VLAN
 int vlan 50
  no shut
  ip add 10.21.50.4 255.255.255.0
  desc CCTV-VLAN
 int vlan 100
  no shut
  ip add 10.21.100.4 255.255.255.0
  desc VOICE-VLAN
 end
~~~



<br>
<br>

---
&nbsp;


### Interactive Voice Response System
~~~
!@CUCM
config t
dial-peer voice 69 voip
 service rivanaa out-bound
 destination-pattern 2169
 session target ipv4:10.21.100.1
 incoming called-number 2169
 dtmf-relay h245-alphanumeric
 codec g711ulaw
 no vad
!
telephony-service
 moh "flash:/en_bacd_music_on_hold.au"
!
application
 service rivanaa flash:app-b-acd-aa-3.0.0.2.tcl
  paramspace english index 1        
  param number-of-hunt-grps 2
  param dial-by-extension-option 8
  param handoff-string rivanaa
  param welcome-prompt flash:en_bacd_welcome.au
  paramspace english language en
  param call-retry-timer 15
  param service-name rivanqueue
  paramspace english location flash:
  param second-greeting-time 60
  param max-time-vm-retry 2
  param voice-mail 1234
  param max-time-call-retry 700
  param aa-pilot 2169
 service rivanqueue flash:app-b-acd-3.0.0.2.tcl
  param queue-len 15
  param aa-hunt1 2100
  param aa-hunt2 2177
  param aa-hunt3 2101
  param aa-hunt4 2133
  param queue-manager-debugs 1
  param number-of-hunt-grps 4
  end
~~~


<br>
<br>

---
&nbsp;


## Quality of Service
*Why need QoS? Periodic Congestion*




&nbsp;
---
&nbsp;


### DSCP IDs

| #   | Name                       | Decimal | Binary  |
| --- | ---                        | ---     | ---     |
| 1   | Default                    | 0       | 00 0000 |
| 2   | Expedited Forwarding (EF)  | 46      | 10 1110 |
| 3   | Class Selector 1 (CS1)     | 8	     | 00 1000 |
| 4   | Class Selector 2 (CS2)     | 16	     | 01 0000 |
| 5   | Class Selector 3 (CS3)     | 24	     | 01 1000 |
| 6   | Class Selector 4 (CS4)	   | 32	     | 10 0000 |
| 7   | Class Selector 5 (CS5)     | 40      | 10 1000 |
| 8   | Class Selector 6 (CS6)     | 48      | 11 0000 |
| 9   | Class Selector 7 (CS7)     | 56	     | 11 1000 |

<br>
<br>

| #   | Low Drop Probability   | Medium Drop Probability   | High Drop Probability  |
| --- | ---                    | ---                       | ---                    |
| 1   | AF11(10) 001 01_0      | AF12(12) 001 10_0         | AF13(14) 001 11_0      |
| 2   | AF21(18) 010 01_0      | AF22(20) 010 10_0         | AF23(22) 010 11_0      |
| 3   | AF31(26) 011 01_0      | AF32(28) 011 10_0         | AF33(30) 011 11_0      |
| 4   | AF41(34) 100 01_0      | AF42(36) 100 10_0         | AF43(38) 100 11_0      |


<br>
<br>

### SPAN
~~~
!@CoreBABA
conf t
 monitor session 1 source interface fa0/3,fa0/5,fa0/7
 monitor session 1 destination interface fa0/1,fa0/9
 end
~~~

<br>

Remove SPAN
~~~
conf t
 no monitor session 1
 end
~~~


<br>
<br>

---
&nbsp;


### Modify QoS Marking
| Protocol | DSCP |
| ---      | ---  |
| SKINNY   |      |
| RTCP     |      |
| RTP      |      |

<br>

__Remove Trust QOS__
~~~
!@CoreBABA
conf t
 int range fa0/5,fa0/7
  no mls qos trust device cisco-phone
  end
~~~


<br>


__Manual Markings__








<br>
<br>

---
&nbsp;


## Private & Public WAN
~~~
!@EDGE
conf t
 no router ospf 1
 router ospf 1
  router-id 21.0.0.1
  network 10.21.21.0 0.0.0.255 area 0
  default-information originate always
  exit
 ip domain lookup
 ip name-server 8.8.8.8
 ip domain lookup source-int g0/0/0
 ip route 0.0.0.0 0.0.0.0 200.0.0.1
 end
~~~

<br>

~~~
!@EDGE
conf t
 int g0/0/0 
  ip nat inside
  exit
 int g0/0/1
  ip nat outside
  exit
 !
 ip access-list extended NAT-POLICY
  permit ip any any
  exit
 !
 ip nat inside source list NAT-POLICY int g0/0/1 overload
 end
~~~


&nbsp;
---
&nbsp;


### Multi-Point GRE Tunnel







<br>
<br>

---
&nbsp;


# S2S-VPN via Active Directory Certificate Services
`VMWare` > `Edit` > `Virtual Network Editor`  

<br>

Add/Edit the following VMNets:  

| VMNet 2    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.102.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 3    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.103.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 4    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.104.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 8    |               |
| ---        | ---           |
| VMNet Info | NAT           |
| IP address | 208.8.8.0     |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 	

<br>

| VMNet 15   |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 10.255.10.0   |
| Net Mask   | 255.255.255.0 |
| DHCP       | Checked       | 

<br>

| VMNet 16   |                 |
| ---        | ---             |
| VMNet Info | Host-only       |
| IP address | 10.69.255.0     |
| Net Mask   | 255.255.255.248 |
| DHCP       | Checked         | 


<br>
<br>

---
&nbsp;


## CERTIFICATE AUTHORITY

### Deployment
~~~
!@UTM-PH
conf t
 hostname UTM-PH
 enable secret pass
 service password-encryption
 no logging cons
 ip domain lookup
 ip domain lookup source-interface G2
 ip name-server 192.168.102.8
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.11 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.11 255.255.255.0
  no shut
 int g3
  ip add 11.11.11.113 255.255.255.224
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 end
wr
!
~~~

 
<br>


~~~
!@UTM-JP
conf t
 hostname UTM-JP
 enable secret pass
 service password-encryption
 no logging cons
 ip domain lookup
 ip domain lookup source-interface G2
 ip name-server 192.168.102.8
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.12 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.12 255.255.255.0
  no shut
 int g3
  ip add 21.21.21.213 255.255.255.240
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 end
wr
!
~~~


<br>


~~~
!@BLDG-PH
sudo su
ifconfig eth0 11.11.11.111 netmask 255.255.255.224 up
route add default gw 11.11.11.113
ping 11.11.11.113
~~~


<br>


~~~
!@BLDG-JP
sudo su
ifconfig eth0 21.21.21.211 netmask 255.255.255.240 up
route add default gw 21.21.21.213
ping 21.21.21.213
~~~


&nbsp;
---
&nbsp;


### STEP 1 - Setup WinServer

| NetAdapter  | VMNet  | IP Address    | 
| ---         | ---    | ---           |
| 1           | NAT    | 208.8.8.8     |
| 2           | VMNet2 | 192.168.102.8 |

<br>

~~~
!@Powershell
set-netfirewallprofile -name private,public,domain -enabled false
rename-computer ccnp21.com
ncpa.cpl
~~~


&nbsp;
---
&nbsp;


### STEP 2 - Install Active Directory Domains and Services
Create a Service Account:
- Active Directory and Users and Computers


&nbsp;
---
&nbsp;


### STEP 3 - Install Active Directory Certificate Services


&nbsp;
---
&nbsp;


### STEP 4 - Create DNS Mapping for both Device


&nbsp;
---
&nbsp;


### STEP 5 - Configure Certificates on Cisco UTM-PH & UTM-JP

~~~
!@UTM-PH
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://192.168.102.8/certsrv/mscep/mscep.dll
  serial-number
  fqdn utmph.ccnp21.com
  ip-address 208.8.8.11
  subject-name CN=UTM-PH,OU=NOC,O=RIVANCORP,L=MAKATI,ST=NCR,C=PH
  subject-alt-name utmph.ccnp21.com
  revocation-check none
  source interface GigabitEthernet2
  rsakeypair CERTKEY
  end
~~~

<br>

~~~
!@UTM-JP
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://192.168.102.8/certsrv/mscep/mscep.dll
  serial-number
  fqdn utmjp.ccnp21.com
  ip-address 208.8.8.12
  subject-name CN=UTM-JP,OU=NOC,O=RIVANCORP,L=TOKYO,ST=KANTO,C=JP
  subject-alt-name utmjp.ccnp21.com
  revocation-check none
  source interface GigabitEthernet2
  rsakeypair CERTKEY
  end
~~~



&nbsp;
---
&nbsp;


### STEP 6 - Access CA Web Enrollment

Set Routes for trustpoints:
~~~
!@cmd
route add 208.8.8.11 mask 255.255.255.255 192.168.102.11
route add 208.8.8.12 mask 255.255.255.255 192.168.102.12
~~~


<br>
<br>


http://192.168.102.8/certsrv/mscep/mscep.dll  

<br>
<br>

Grab the Hash & Challenge Password    
- Hash: ___________    
- Pass: ___________  


&nbsp;
---
&nbsp;


### STEP 7 - Enroll Network Devices

~~~
!@UTM-PH,UTM-JP
conf t
 crypto pki enroll CCNPTRUST
~~~


<br>
<br>

---
&nbsp;


## S2S-VPN

__Phase 1 (IKEv2) & Phase 2 (IPSec)__
- Encryption
- Integrity
- DH

<br>

__Tunnel Properties__
- IP
- Source Interface
- Destination Peer IP
- Remote Subnets


&nbsp;
---
&nbsp;


### STEP 1 - Phase 1 (IKEv2)
~~~
!@UTM-PH
conf t
 crypto ikev2 proposal IKEV2-PROP
  encryption ______
  integrity ______
  group ______
 !
 crypto ikev2 policy IKEV2-POL
  proposal IKEV2-PROP
 !
 crypto ikev2 profile IKEV2-PROF
  match identity remote address __.__.__.__
  authentication remote rsa-sig
  authentication local rsa-sig
  pki trustpoint CCNPTRUST
  end
~~~

<br>

~~~
!@UTM-JP
conf t
 crypto ikev2 proposal IKEV2-PROP
  encryption ______
  integrity ______
  group ______
 !
 crypto ikev2 policy IKEV2-POL
  proposal IKEV2-PROP
 !
 crypto ikev2 profile IKEV2-PROF
  match identity remote address __.__.__.__
  authentication remote rsa-sig
  authentication local rsa-sig
  pki trustpoint CCNPTRUST
  end
~~~


&nbsp;
---
&nbsp;


### STEP 2 - Phase 2 (IPSEC)
~~~
!@UTM-PH, UTM-JP
conf t
 crypto ipsec transform-set TSET _____  _____
  mode ____
 !
 crypto ipsec profile VPN-IPSEC-PROF
  set transform-set TSET
  set ikev2-profile IKEV2-PROF
  end
~~~


&nbsp;
---
&nbsp;


### STEP 3 - Tunnel Properties

~~~
!@UTM-PH
conf t
 int tun1
  ip add __.__.__.__  __.__.__.__
  tunnel source ___
  tunnel destination __.__.__.__
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile VPN-IPSEC-PROF
  end
~~~

<br>

~~~
!@UTM-JP
conf t
 int tun1
  ip add __.__.__.__  __.__.__.__
  tunnel source ___
  tunnel destination __.__.__.__
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile VPN-IPSEC-PROF
  end
~~~


&nbsp;
---
&nbsp;


### STEP 4 - Remote Subnets / Interesting Traffic
~~~
!@UTM-PH
conf t
 ip route __.__.__.__  __.__.__.__  __.__.__.__
 end
~~~

<br>

~~~
!@UTM-JP
conf t
 ip route __.__.__.__  __.__.__.__  __.__.__.__
 end
~~~





<br>
<br>

---
&nbsp;








## DMVPN

### PRECONFIGS

~~~
!@UTM-PH
conf t
 hostname UTM-PH
 enable secret pass
 service password-encryption
 no logging cons
 ip domain lookup
 ip domain lookup source-interface G2
 ip name-server 192.168.102.8
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.11 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.11 255.255.255.0
  no shut
 int g3
  ip add 10.11.11.113 255.255.255.224
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 end
wr
!
~~~

~~~
!@UTM-JP
conf t
 hostname UTM-JP
 enable secret pass
 service password-encryption
 no logging cons
 ip domain lookup
 ip domain lookup source-interface G2
 ip name-server 192.168.102.8
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.12 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.12 255.255.255.0
  no shut
 int g3
  ip add 10.21.21.213 255.255.255.240
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 end
wr
!
~~~

~~~
!@UTM-US
conf t
 hostname UTM-US
 enable secret pass
 service password-encryption
 no logging cons
 ip domain lookup
 ip domain lookup source-interface G2
 ip name-server 192.168.102.8
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.13 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.13 255.255.255.0
  no shut
 int g3
  ip add 10.0.0.2 255.255.255.252
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 end
wr
!
~~~

~~~
!@BLDG-PH
sudo su
ifconfig eth0 10.11.11.101 netmask 255.255.255.224 up
route add default gw 10.11.11.113
ping 10.11.11.113
~~~

~~~
!@BLDG-JP
sudo su
ifconfig eth0 10.21.21.211 netmask 255.255.255.240 up
route add default gw 10.21.21.213
ping 10.21.21.213
~~~

~~~
!@BLDG-US
sudo su
ifconfig eth0 10.0.0.1 netmask 255.255.255.252 up
route add default gw 10.0.0.2
ping 10.0.0.2
~~~


<br>
<br>

---
&nbsp;


### STEP 1 - IKEV2
HUB
~~~
!@UTM-US
conf t
 crypto ikev2 proposal IKEV2-PROP
  encryption ___
  integrity ___
  group ___
 !
 crypto ikev2 policy IKEV2-POLICY
  proposal ___
 !
 crypto ikev2 keyring DMVPN-KEYRING
  peer SPOKES
   address ___.___.___.___
   pre-shared-key ___
 !
 crypto ikev2 profile DMVPN-IKEV2
  ___
  ___
  ___
  ___
  end
~~~

SPOKE
~~~
!@UTM-PH,UTM-JP
conf t
 crypto ikev2 proposal IKEV2-PROP
  encryption ___
  integrity ___
  group ___
 !
 crypto ikev2 policy IKEV2-POLICY
  proposal ___
 !
 crypto ikev2 keyring DMVPN-KEYRING
  peer SPOKES
   address ___.___.___.___
   pre-shared-key ___
 !
 crypto ikev2 profile DMVPN-IKEV2
  ___
  ___
  ___
  ___
  end
~~~


<br>
<br>

---
&nbsp;


### STEP 2 - IPSEC

HUB
~~~
!@UTM-US
conf t
 crypto ipsec transform-set TS ___
  mode ___
  !
 crypto ipsec profile DMVPN-IPSEC
  ___
  ___
 !
 interface Tunnel1
  
  end
~~~


SPOKE
~~~
!@UTM-PH
conf t
 crypto ipsec transform-set TS ___
  mode ___
  !
 crypto ipsec profile DMVPN-IPSEC
  ___
  ___
 !
 interface Tunnel1
  
  end
~~~



~~~
!@UTM-JP
conf t
 crypto ipsec transform-set TS ___
  mode ___
  !
 crypto ipsec profile DMVPN-IPSEC
  ___
  ___
 !
 interface Tunnel1
  
  end
~~~
