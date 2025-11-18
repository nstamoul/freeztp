# FreeZTP - Zero-Touch Provisioning for Cisco Catalyst Switches

![Version](https://img.shields.io/badge/version-v1.4.1-blue.svg)
![Python](https://img.shields.io/badge/python-2.7.5+-green.svg)
![License](https://img.shields.io/badge/license-GPL--3.0-orange.svg)

A powerful, dynamic Zero-Touch Provisioning (ZTP) system designed to automatically configure Cisco Catalyst switches upon first boot. FreeZTP leverages Cisco's built-in AutoInstall feature to provide seamless, template-based switch provisioning with minimal manual intervention.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Architecture](#architecture)
4. [Requirements](#requirements)
5. [Quick Start](#quick-start)
6. [Installation](#installation)
7. [Configuration](#configuration)
8. [ZTP Process Flow](#ztp-process-flow)
9. [Command Reference](#command-reference)
10. [Advanced Features](#advanced-features)
11. [Integrations](#integrations)
12. [Troubleshooting](#troubleshooting)
13. [Contributing](#contributing)
14. [Additional Resources](#additional-resources)

---

## 🎯 Overview

### What is FreeZTP?

FreeZTP is a sophisticated, template-driven TFTP server that automatically provisions Cisco Catalyst switches using the AutoInstall feature built into Cisco IOS. It eliminates manual configuration steps by:

- **Automatic Device Discovery**: Uses SNMP to identify switches by serial number
- **Template-Based Configuration**: Jinja2 templates allow dynamic, customized configurations
- **Intelligent Matching**: Maps device identities to configurations via keystores and ID arrays
- **Image Management**: Handles IOS/IOS-XE software upgrades during provisioning
- **Built-in DHCP**: Integrated DHCP server with required ZTP options
- **External Data Sources**: Supports CSV/JSON keystores for large-scale deployments
- **Third-Party Integrations**: Cisco Webex Teams notifications and more

### Why FreeZTP?

- ✅ **Zero Manual Configuration**: Switches boot fully configured
- ✅ **Scalable**: Handles single switches to large enterprise deployments
- ✅ **Flexible**: Template system adapts to any network design
- ✅ **Stack-Aware**: Intelligently handles switch stacks with multiple serial numbers
- ✅ **Upgrade Capable**: Manages IOS version upgrades during provisioning
- ✅ **Open Source**: Free, customizable, community-driven

---

## ⭐ Key Features

### Core Capabilities

| Feature | Description |
|---------|-------------|
| **AutoInstall Integration** | Leverages Cisco's native ZTP mechanism |
| **Jinja2 Templating** | Powerful template engine with variables, loops, and conditionals |
| **SNMP Discovery** | Automatic device identification via serial number |
| **Keystore System** | Flexible data storage linking device IDs to configurations |
| **ID Arrays** | Map multiple serial numbers to single configuration (for stacks) |
| **Default Keystore** | Fallback configuration for unknown devices |
| **Image Upgrade** | Automated IOS/IOS-XE software deployment |
| **Image Suppression** | Prevents redundant image downloads |

### Advanced Features

| Feature | Description |
|---------|-------------|
| **External Keystores** | CSV/JSON data sources for enterprise-scale deployments |
| **External Templates** | File-based template storage and management |
| **Integrations** | Cisco Webex Teams notifications |
| **Custom Logging** | Configurable logging to files with Jinja2 path templates |
| **DHCP Server** | Full-featured DHCP with auto-configuration |
| **Provisioning Tracking** | Historical record of all provisioned devices |
| **File Caching** | Performance optimization for rapid deployments |

---

## 🏗️ Architecture

FreeZTP is built on a modular Python architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                         CLI Interface                        │
│                   (Command Line + Helpers)                   │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────┴───────────────────────────────────┐
│                   Configuration Manager                      │
│              (Persistent Storage + Settings)                 │
└─────────────────────────┬───────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼────────┐ ┌─────▼──────┐ ┌───────▼────────┐
│  TFTP Server   │ │    DHCP    │ │  Integration   │
│                │ │   Server   │ │    Manager     │
│  - File Cache  │ │  Manager   │ │                │
│  - Dynamic     │ │            │ │  - Webex Teams │
│    Files       │ │  - Auto    │ │  - Webhooks    │
└───────┬────────┘ │    Config  │ └────────────────┘
        │          └────────────┘
┌───────▼────────────────────────────────────────────────────┐
│              Configuration Factory (ZTP Engine)             │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │   Template   │  │   Keystore   │  │  SNMP Discovery │  │
│  │    Merger    │  │   Resolver   │  │                 │  │
│  │   (Jinja2)   │  │              │  │  - Serial Query │  │
│  └──────────────┘  └──────────────┘  └─────────────────┘  │
└────────────────────────────────────────────────────────────┘
        │                 │                     │
┌───────▼──────┐  ┌──────▼───────┐  ┌──────────▼────────┐
│  External    │  │   External   │  │   Tracking &      │
│  Keystores   │  │  Templates   │  │   Provisioning    │
│              │  │              │  │   History         │
│  - CSV       │  │  - File-     │  │                   │
│  - JSON      │  │    based     │  │  - Downloads      │
└──────────────┘  └──────────────┘  └───────────────────┘
```

### Core Components

For detailed architecture information, see [ARCHITECTURE.md](ARCHITECTURE.md).

**Key Modules:**

1. **Config Factory** - ZTP logic engine that generates configurations
2. **TFTP Server** - Serves files to switches during AutoInstall
3. **SNMP Query** - Discovers device identities via SNMP
4. **Keystore System** - Maps device IDs to configuration data
5. **Template Engine** - Jinja2-based configuration generation
6. **DHCP Manager** - Integrated DHCP server with ZTP options
7. **Tracking System** - Records provisioning history
8. **Integration Framework** - Third-party notification system

---

## 📋 Requirements

### Supported Operating Systems

| OS | Versions | Status |
|----|----------|--------|
| **CentOS/RHEL** | 7, 8 | ✅ Recommended |
| **Ubuntu** | 16, 18, 20+ | ✅ Supported |
| **Debian** | 9+ | ✅ Supported |
| **Raspbian** | Stretch+ | ✅ Supported |

### Software Requirements

- **Python**: 2.7.5 or higher
- **Git**: For installation
- **Root Access**: For installation and service management
- **Internet Access**: During installation (for package downloads)

### Network Requirements

- Reachable by target switches via DHCP/TFTP
- UDP port 69 (TFTP) - open
- UDP port 67 (DHCP) - open if using built-in DHCP
- UDP port 161 (SNMP) - outbound for device discovery

### Supported Cisco Devices

FreeZTP supports most Cisco Catalyst switches with AutoInstall capability:

- **Catalyst 2960 Series** (IOS)
- **Catalyst 3560 Series** (IOS)
- **Catalyst 3650 Series** (IOS-XE)
- **Catalyst 3850 Series** (IOS-XE)
- **Catalyst 9200 Series** (IOS-XE)
- **Catalyst 9300 Series** (IOS-XE)
- **Catalyst 9400 Series** (IOS-XE)
- **Catalyst 9500 Series** (IOS-XE)
- **Industrial Ethernet 3200/3400/4000 Series** (IOS/IOS-XE)

---

## 🚀 Quick Start

### 5-Minute Setup

```bash
# 1. Clone the repository
git clone https://github.com/packetsar/freeztp.git
cd freeztp

# 2. Run the installer (requires sudo)
sudo python ztp.py install

# 3. Configure DHCP scope with IP lease range
ztp set dhcpd INTERFACE-ETH0 first-address 192.168.1.100
ztp set dhcpd INTERFACE-ETH0 last-address 192.168.1.200
ztp request dhcpd-commit

# 4. Restart the ZTP service
ztp service restart

# 5. Connect a switch and power it on!
```

The default configuration will now provision switches with a basic configuration.

---

## 💿 Installation

### Detailed Installation Steps

#### 1. Prepare Your System

**For CentOS/RHEL 7/8:**
```bash
# Install Git
sudo yum install git -y

# Install Python2 PIP
sudo yum install python2-pip -y

# For CentOS/RHEL 8, create symlinks
sudo ln -s /usr/bin/python2.7 /usr/bin/python
sudo ln -s /usr/bin/pip2 /usr/bin/pip
```

**For Ubuntu/Debian:**
```bash
# Install Git and Python PIP
sudo apt-get update
sudo apt install -y python-pip git
```

**For Raspbian:**
```bash
# Install Git and Python PIP
sudo apt-get update
sudo apt install -y python-pip git
```

#### 2. Download FreeZTP

```bash
# Clone the repository
git clone https://github.com/packetsar/freeztp.git

# Navigate to the directory
cd freeztp
```

#### 3. Run the Installer

```bash
# Execute the install command
sudo python ztp.py install
```

The installer will:
- Install required Python packages (Jinja2, pysnmp, tftpy)
- Install and configure DHCP server
- Create ZTP configuration directory (`/etc/ztp/`)
- Create TFTP root directory (`/etc/ztp/tftproot/`)
- Install systemd service
- Install CLI completion helpers
- Auto-configure DHCP scopes based on network interfaces

#### 4. Configure DHCP (Required)

The installer creates DHCP scopes but doesn't enable them. Add an IP address range:

```bash
# Set first and last IP addresses for lease range
ztp set dhcpd INTERFACE-ETH0 first-address 192.168.1.100
ztp set dhcpd INTERFACE-ETH0 last-address 192.168.1.200

# Commit DHCP configuration and restart DHCP service
ztp request dhcpd-commit
```

#### 5. Start the Service

```bash
# Restart ZTP to apply configuration
ztp service restart

# Check service status
ztp show status
```

#### 6. Verify Installation

```bash
# View current configuration
ztp show config

# Check ZTP version
ztp version

# Monitor logs
ztp show log tail
```

#### 7. Logout and Login

To activate CLI completion helpers:
```bash
# Logout of SSH session
exit

# Log back in
ssh user@ztp-server
```

---

## ⚙️ Configuration

### Understanding the Configuration Model

FreeZTP uses a hierarchical configuration model:

```
Templates ─┐
           ├──> Association ─┐
Keystores ─┘                  ├──> Final Configuration
                             │
ID Arrays ────────────────────┘
```

### Core Configuration Elements

#### 1. Templates

Templates are Jinja2-formatted Cisco IOS configurations with variables:

**Initial Template** - Used for device discovery (usually left as default):
```cisco
hostname {{ autohostname }}
!
snmp-server community {{ community }} RO
!
end
```

**Final Templates** - Your actual switch configurations:
```cisco
hostname {{ hostname }}
!
interface Vlan1
  ip address {{ vl1_ip_address }} 255.255.255.0
  no shut
!
end
```

**Setting a template:**
```bash
ztp set template MY_TEMPLATE ^
hostname {{ hostname }}
!
interface Vlan1
  ip address {{ mgmt_ip }} {{ mgmt_mask }}
  no shut
!
end
^
```

Note: The `^` character is the delimiter - use any character not in your template.

#### 2. Keystores

Keystores hold the data (key-value pairs) that populate template variables:

```bash
# Create keystore entries
ztp set keystore SWITCH01 hostname ACCESS-SW-01
ztp set keystore SWITCH01 mgmt_ip 10.0.1.10
ztp set keystore SWITCH01 mgmt_mask 255.255.255.0

# For complex data, use JSON
ztp set keystore SWITCH01 vlan_list '[{"id": "10", "name": "Data"}, {"id": "20", "name": "Voice"}]'
```

**Keystore Hierarchy:**
```
Keystore ID (e.g., "SWITCH01")
  ├── Key: hostname → Value: ACCESS-SW-01
  ├── Key: mgmt_ip → Value: 10.0.1.10
  └── Key: mgmt_mask → Value: 255.255.255.0
```

#### 3. ID Arrays

ID Arrays map multiple real device IDs (serial numbers) to a single keystore:

```bash
# For a 3-switch stack
ztp set idarray STACK1 FCW2201A001 FCW2201A002 FCW2201A003
```

When any of these serial numbers is discovered, FreeZTP uses the `STACK1` keystore.

#### 4. Associations

Associations link keystores to templates:

```bash
# Associate SWITCH01 keystore with MY_TEMPLATE
ztp set association id SWITCH01 template MY_TEMPLATE

# Set default template for keystores without associations
ztp set default-template MY_TEMPLATE

# Set default keystore for unknown devices
ztp set default-keystore UNKNOWN_SWITCH
```

### Complete Configuration Example

```bash
#######################################################
# Create Templates
#######################################################
ztp set template ACCESS_SWITCH ^
hostname {{ hostname }}
!
vlan {{ data_vlan }}
  name Data
!
interface Vlan{{ data_vlan }}
  ip address {{ mgmt_ip }} {{ mgmt_mask }}
  no shut
!
interface range GigabitEthernet1/0/1-48
  switchport mode access
  switchport access vlan {{ data_vlan }}
  spanning-tree portfast
!
end
^

#######################################################
# Create Keystores
#######################################################
ztp set keystore SW-FLOOR1-01 hostname ACCESS-FL1-SW01
ztp set keystore SW-FLOOR1-01 mgmt_ip 10.1.1.10
ztp set keystore SW-FLOOR1-01 mgmt_mask 255.255.255.0
ztp set keystore SW-FLOOR1-01 data_vlan 100

ztp set keystore SW-FLOOR2-01 hostname ACCESS-FL2-SW01
ztp set keystore SW-FLOOR2-01 mgmt_ip 10.1.2.10
ztp set keystore SW-FLOOR2-01 mgmt_mask 255.255.255.0
ztp set keystore SW-FLOOR2-01 data_vlan 200

#######################################################
# Create ID Arrays (for serial number mapping)
#######################################################
ztp set idarray SW-FLOOR1-01 FCW2201A001
ztp set idarray SW-FLOOR2-01 FCW2201A002

#######################################################
# Create Associations
#######################################################
ztp set association id SW-FLOOR1-01 template ACCESS_SWITCH
ztp set association id SW-FLOOR2-01 template ACCESS_SWITCH

#######################################################
# Set Defaults
#######################################################
ztp set default-keystore UNKNOWN_SWITCH
ztp set default-template ACCESS_SWITCH

#######################################################
# Configure Image Upgrade
#######################################################
ztp set imagefile cat3k_caa-universalk9.SPA.03.06.06.E.152-2.E6.bin
ztp set image-supression 3600

#######################################################
# Restart Service
#######################################################
ztp service restart
```

### DHCP Configuration

Configure DHCP scopes for switch provisioning:

```bash
# View auto-detected scopes
ztp show config | grep dhcpd

# Configure scope parameters
ztp set dhcpd SCOPE_NAME subnet 192.168.1.0/24
ztp set dhcpd SCOPE_NAME first-address 192.168.1.100
ztp set dhcpd SCOPE_NAME last-address 192.168.1.200
ztp set dhcpd SCOPE_NAME lease-time 3600
ztp set dhcpd SCOPE_NAME ztp-tftp-address 192.168.1.10
ztp set dhcpd SCOPE_NAME imagediscoveryfile-option enable
ztp set dhcpd SCOPE_NAME gateway 192.168.1.1
ztp set dhcpd SCOPE_NAME dns-servers 8.8.8.8 8.8.4.4
ztp set dhcpd SCOPE_NAME domain-name example.com

# Commit changes and restart DHCP
ztp request dhcpd-commit
```

### Advanced Settings

```bash
# SNMP settings for device discovery
ztp set community secretcommunity
ztp set snmpoid C3850_SERIAL 1.3.6.1.2.1.47.1.1.1.1.11.1000
ztp set snmpoid C9300_SERIAL 1.3.6.1.2.1.47.1.1.1.1.11.1000

# TFTP settings
ztp set tftproot /etc/ztp/tftproot/

# Timing settings
ztp set delay-keystore 1000  # Milliseconds to wait before keystore lookup
ztp set file-cache-timeout 10  # Seconds to cache generated files

# AutoInstall file names
ztp set initialfilename network-confg
ztp set suffix -confg
ztp set imagediscoveryfile freeztp_ios_upgrade
```

---

## 🔄 ZTP Process Flow

### Overview

The AutoInstall process follows this sequence:

```
┌─────────────┐
│   Switch    │ Powers On
│  Boots Up   │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ STEP 1: DHCP Discovery                  │
│ - Switch enables all ports              │
│ - Enables VLAN1 SVI                     │
│ - Sends DHCP request                    │
│ - Receives IP + Option 150 (TFTP IP)    │
│ - Receives Option 125 (Image Discovery) │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ STEP 2: Image Upgrade (Optional)        │
│ - Requests imagediscoveryfile via TFTP  │
│ - ZTP returns IOS filename              │
│ - Switch downloads IOS image            │
│ - Installs and reboots (if needed)      │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ STEP 3: Initial Configuration           │
│ - Requests "network-confg"              │
│ - ZTP generates temp hostname           │
│ - ZTP sends initial config with SNMP    │
│ - Switch loads config                   │
└──────┬──────────────────────────────────┘
       │
       ├──────────────────────────┐
       ▼                          ▼
┌──────────────────┐      ┌──────────────────┐
│ STEP 4: SNMP     │      │ STEP 5: Final    │
│ Discovery        │      │ Config Request   │
│                  │      │                  │
│ - ZTP queries    │      │ - Switch         │
│   switch via     │      │   requests       │
│   SNMP           │      │   "<hostname>-   │
│ - Retrieves      │      │   confg"         │
│   serial number  │      │                  │
│ - Maps to temp   │      │                  │
│   hostname       │      │                  │
└──────────────────┘      └──────┬───────────┘
                                 │
       ┌─────────────────────────┘
       ▼
┌─────────────────────────────────────────┐
│ STEP 6: Configuration Generation        │
│ - Find matching keystore                │
│ - Find associated template              │
│ - Merge template + keystore data        │
│ - Send final configuration              │
│ - Switch applies configuration           │
└─────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ STEP 7: Complete                         │
│ - Switch is fully configured            │
│ - Ready for production use              │
│ - Track provisioning event              │
└─────────────────────────────────────────┘
```

### Detailed Process Steps

#### Step 1: DHCP Discovery

1. Switch powers on and boots IOS
2. After boot completes, AutoInstall activates
3. Switch enables all ports as access ports on VLAN 1
4. VLAN 1 SVI is enabled
5. Switch sends DHCP DISCOVER
6. FreeZTP DHCP responds with:
   - IP address
   - Option 150: TFTP server IP (FreeZTP)
   - Option 125: Image discovery file (optional)

#### Step 2: IOS Upgrade (if enabled)

1. If Option 125 was provided, switch requests the image discovery file
2. FreeZTP checks image suppression list to prevent duplicate downloads
3. If not suppressed, FreeZTP returns the configured image filename
4. Switch downloads the .bin or .tar file via TFTP
5. Switch installs the image
6. Switch reboots (process restarts at Step 1, but upgrade is suppressed)

#### Step 3: Initial Configuration

1. Switch requests "network-confg" via TFTP
2. FreeZTP generates a unique temporary hostname (e.g., `ZTP-A1B2C3D4E5`)
3. FreeZTP merges the initial-template:
   - `{{ autohostname }}` = temporary hostname
   - `{{ community }}` = SNMP community string
4. Switch receives and applies the initial configuration

#### Step 4: SNMP Discovery (Parallel)

While the switch loads the initial config, FreeZTP:
1. Initiates SNMP queries to the switch's IP
2. Uses configured SNMP OIDs to retrieve serial number(s)
3. Maps the discovered serial number to the temporary hostname
4. Stores this mapping for Step 6

#### Step 5: Final Configuration Request

1. After loading initial config, switch requests `<temp-hostname>-confg`
2. Example: `ZTP-A1B2C3D4E5-confg`
3. FreeZTP receives the request

#### Step 6: Configuration Generation

FreeZTP performs these steps:

1. **Lookup Real ID**: Uses temp hostname to find discovered serial number
2. **Find Keystore**:
   - Search keystores for matching ID
   - If not found, search ID arrays
   - If still not found, use default keystore
3. **Find Template**:
   - Check if keystore has an association
   - If not, use default template
4. **Merge**: Combine template + keystore data using Jinja2
5. **Return**: Send final configuration to switch

#### Step 7: Complete

1. Switch applies the final configuration
2. FreeZTP logs the provisioning event
3. Switch is ready (may need `write mem` to save config)

### Process Timing

Typical provisioning times:

| Scenario | Time |
|----------|------|
| **No Image Upgrade** | 2-5 minutes |
| **With Image Upgrade (small)** | 10-15 minutes |
| **With Image Upgrade (large)** | 20-30 minutes |
| **Stack (3 switches)** | Add 2-3 minutes for SSO |

---

## 📚 Command Reference

### Service Management

```bash
# Start/stop/restart ZTP service
ztp service start
ztp service stop
ztp service restart
ztp service status

# Control DHCP service
ztp service dhcpd start
ztp service dhcpd stop
ztp service dhcpd restart
```

### Show Commands

```bash
# View configuration
ztp show config          # Formatted configuration
ztp show config raw      # Raw JSON configuration

# View status and logs
ztp show status          # Service status
ztp show version         # FreeZTP version
ztp show log             # View log file
ztp show log tail        # Tail log (follow mode)
ztp show log tail 100    # Tail last 100 lines

# View DHCP information
ztp show dhcpd leases              # Current DHCP leases
ztp show dhcpd leases current      # Active leases only
ztp show dhcpd leases all          # All leases
ztp show dhcpd leases raw          # Raw lease file

# View provisioning data
ztp show provisioning    # Provisioning history
ztp show downloads       # TFTP download history
ztp show downloads live  # Live download monitoring
```

### Set Commands

```bash
# Templates
ztp set template <name> <delimiter>
ztp set initial-template <delimiter>

# Keystores
ztp set keystore <id> <key> <value>

# ID Arrays
ztp set idarray <arrayname> <id1> <id2> ...

# Associations
ztp set association id <id> template <template-name>

# Defaults
ztp set default-keystore <keystore-id|none>
ztp set default-template <template-name|none>

# DHCP
ztp set dhcpd <scope-name> subnet <network/prefix>
ztp set dhcpd <scope-name> first-address <ip>
ztp set dhcpd <scope-name> last-address <ip>
ztp set dhcpd <scope-name> lease-time <seconds>
ztp set dhcpd <scope-name> ztp-tftp-address <ip>
ztp set dhcpd <scope-name> gateway <ip>
ztp set dhcpd <scope-name> dns-servers <ip1> <ip2>
ztp set dhcpd <scope-name> domain-name <domain>
ztp set dhcpd <scope-name> imagediscoveryfile-option <enable|disable>

# SNMP
ztp set community <community-string>
ztp set snmpoid <name> <oid>

# Image Management
ztp set imagefile <filename>
ztp set image-supression <seconds>

# System Settings
ztp set tftproot <path>
ztp set delay-keystore <milliseconds>
ztp set file-cache-timeout <seconds>
ztp set initialfilename <filename>
ztp set suffix <suffix>
ztp set imagediscoveryfile <filename>
```

### Clear Commands

```bash
# Remove configuration items
ztp clear template <name>
ztp clear keystore <id> all
ztp clear keystore <id> <key>
ztp clear idarray <arrayname>
ztp clear association <id>
ztp clear dhcpd <scope-name>
ztp clear snmpoid <name>

# Clear runtime data
ztp clear log
ztp clear downloads
ztp clear provisioning
```

### Request Commands

```bash
# Test and validation
ztp request merge-test <keystore-id>
ztp request initial-merge
ztp request default-keystore-test
ztp request snmp-test <ip-address>

# DHCP operations
ztp request dhcpd-commit
ztp request auto-dhcpd
ztp request dhcp-option-125 <windows|cisco>

# External keystores
ztp request keystore-csv-export <filename>
ztp request external-keystore-test <store-name>

# Integrations
ztp request integration-setup <service-name>
ztp request integration-test <service-name>

# Advanced
ztp request ipc-console
```

### Installation and Upgrade

```bash
# Install FreeZTP
sudo python ztp.py install

# Upgrade FreeZTP
sudo python ztp.py upgrade
```

---

## 🚀 Advanced Features

### External Keystores

For large-scale deployments, store keystore data in external files:

#### CSV Keystores

**Example CSV file (`devices.csv`):**
```csv
keystore_id,association,idarray_1,idarray_2,hostname,mgmt_ip,mgmt_mask
SWITCH01,ACCESS_TEMPLATE,FCW2201A001,,ACCESS-SW-01,10.1.1.10,255.255.255.0
STACK02,ACCESS_TEMPLATE,FCW2201B001,FCW2201B002,CORE-STACK-01,10.1.1.20,255.255.255.0
```

**Configure external keystore:**
```bash
ztp set external-keystore MY_CSV_STORE type csv
ztp set external-keystore MY_CSV_STORE file '/path/to/devices.csv'
ztp service restart

# Test the external keystore
ztp request external-keystore-test MY_CSV_STORE
```

**Reserved CSV Headers:**
- `keystore_id` (Required): The keystore ID
- `association` (Optional): Template to use
- `idarray_X` (Optional): Serial numbers (X = 1, 2, 3, ...)

#### JSON Keystores

**Example JSON file (`devices.json`):**
```json
{
  "SWITCH01": {
    "association": "ACCESS_TEMPLATE",
    "hostname": "ACCESS-SW-01",
    "mgmt_ip": "10.1.1.10",
    "mgmt_mask": "255.255.255.0"
  },
  "STACK02": {
    "association": "ACCESS_TEMPLATE",
    "hostname": "CORE-STACK-01",
    "mgmt_ip": "10.1.1.20",
    "mgmt_mask": "255.255.255.0",
    "idarray": ["FCW2201B001", "FCW2201B002"]
  }
}
```

### External Templates

Store templates as files for easier version control:

**Create template file (`/etc/ztp/templates/access_switch.j2`):**
```jinja2
hostname {{ hostname }}
!
interface Vlan1
  ip address {{ mgmt_ip }} {{ mgmt_mask }}
  no shut
!
end
```

**Configure external template:**
```bash
ztp set external-template ACCESS_SWITCH file '/etc/ztp/templates/access_switch.j2'
ztp service restart
```

### Custom Logging

Log merged configurations to custom files:

```bash
# Log to main log file
ztp set logging merged-config-to-mainlog enable

# Log to custom file with dynamic path
ztp set logging merged-config-to-custom-file '/var/log/ztp/configs/{{ keystore_id }}_{{ local_timestamp }}.txt'
```

**Available variables for log path:**
- `{{ keystore_id }}` - Matched keystore
- `{{ ipaddr }}` - Switch IP
- `{{ temp_id }}` - Temporary hostname
- `{{ local_timestamp }}` - Timestamp (YYYY-MM-DD-HH:MM:SS)
- `{{ epoch_timestamp }}` - Unix timestamp
- `{{ local_year }}`, `{{ local_month }}`, `{{ local_day }}`
- `{{ local_hour }}`, `{{ local_minute }}`, `{{ local_second }}`

### Global Keystore

Define global variables available to all merges:

```bash
ztp set keystore GLOBAL dns_server1 8.8.8.8
ztp set keystore GLOBAL dns_server2 8.8.4.4
ztp set keystore GLOBAL ntp_server 10.0.0.1
ztp set global-keystore GLOBAL
```

Access in templates:
```jinja2
ip name-server {{ GLOBAL.dns_server1 }} {{ GLOBAL.dns_server2 }}
ntp server {{ GLOBAL.ntp_server }}
```

### Advanced Jinja2 Techniques

**Conditionals:**
```jinja2
{% if switch_type == "access" %}
spanning-tree portfast default
{% elif switch_type == "distribution" %}
spanning-tree mode rapid-pvst
{% endif %}
```

**Loops:**
```jinja2
{% for vlan in vlan_list %}
vlan {{ vlan.id }}
  name {{ vlan.name }}
{% endfor %}
```

**Variables:**
```jinja2
{% set mgmt_vlan = 100 %}
interface Vlan{{ mgmt_vlan }}
  ip address {{ mgmt_ip }} 255.255.255.0
```

**ID Array Access:**
```jinja2
{# Individual array members #}
Switch 1 Serial: {{ idarray_1 }}
Switch 2 Serial: {{ idarray_2 }}

{# Full array as list #}
{% for serial in idarray %}
Serial {{ loop.index }}: {{ serial }}
{% endfor %}
```

**SNMP Data Access:**
```jinja2
{# Access discovered SNMP values #}
Discovered Serial: {{ snmpinfo.matched }}

{# Access specific OID responses #}
Serial from OID: {{ snmpinfo.C3850_SERIAL }}
```

---

## 🔌 Integrations

### Cisco Webex Teams

Send provisioning notifications to Webex Teams:

#### Setup Wizard

```bash
ztp request integration-setup MY_WEBEX
```

The wizard will guide you through:
1. Choosing integration type (Spark/Webex)
2. Entering API key
3. Entering Room ID or Email

#### Manual Configuration

```bash
ztp set integration MY_WEBEX type spark
ztp set integration MY_WEBEX roomId <webex-room-id>
ztp set integration MY_WEBEX api-key <your-api-key>
ztp service restart

# Test integration
ztp request integration-test MY_WEBEX
```

#### Getting Webex Credentials

1. **API Key**: https://developer.webex.com/docs/api/getting-started
2. **Room ID**: Use Webex API or bot to list rooms

---

## 🔧 Troubleshooting

### Common Issues

#### 1. Switch Not Getting DHCP Address

**Symptoms:** Switch doesn't receive IP from FreeZTP DHCP

**Checks:**
```bash
# Verify DHCP service is running
ztp show status

# Check DHCP configuration
ztp show config | grep dhcpd

# View DHCP leases
ztp show dhcpd leases current

# Check DHCP service logs
sudo journalctl -u dhcpd -f
```

**Solutions:**
- Ensure DHCP scope has `first-address` and `last-address` configured
- Verify `ztp-tftp-address` matches FreeZTP server IP
- Run `ztp request dhcpd-commit` after configuration changes
- Check network connectivity (VLAN, switch ports, etc.)

#### 2. Switch Not Downloading Initial Config

**Symptoms:** Switch gets DHCP but doesn't download "network-confg"

**Checks:**
```bash
# Monitor live downloads
ztp show downloads live

# Check ZTP service status
ztp show status

# View ZTP logs
ztp show log tail

# Check TFTP is listening
sudo netstat -ulnp | grep :69
```

**Solutions:**
- Verify ZTP service is running: `ztp service restart`
- Check firewall allows UDP port 69
- Verify switch can ping FreeZTP server
- Check Option 150 in DHCP lease points to correct IP

#### 3. SNMP Discovery Failing

**Symptoms:** Switch receives initial config but not final config

**Checks:**
```bash
# Test SNMP manually
ztp request snmp-test <switch-ip>

# Check configured SNMP OIDs
ztp show config | grep snmpoid

# View ZTP logs for SNMP errors
ztp show log | grep -i snmp
```

**Solutions:**
- Verify SNMP community matches initial template
- Add/verify SNMP OID for your switch model
- Ensure switch has loaded initial config (check with console)
- Check network allows UDP port 161

#### 4. Final Config Not Generating

**Symptoms:** SNMP succeeds but no final config sent

**Checks:**
```bash
# Test merge manually
ztp request merge-test <keystore-id>

# Verify keystore exists
ztp show config | grep keystore

# Check associations
ztp show config | grep association

# View provisioning history
ztp show provisioning
```

**Solutions:**
- Verify keystore ID or ID array exists for discovered serial
- Ensure association links keystore to template
- Configure default-keystore as fallback
- Check template syntax with `ztp request merge-test`

#### 5. Template Merge Errors

**Symptoms:** Errors in logs about Jinja2 or template merging

**Checks:**
```bash
# Test template merge
ztp request merge-test <keystore-id>

# View merge errors in logs
ztp show log | grep -i jinja
ztp show log | grep -i template
```

**Solutions:**
- Verify all template variables exist in keystore
- Check Jinja2 syntax (loops, conditionals, etc.)
- Use `{{ variable | default('fallback') }}` for optional variables
- Test templates incrementally

### Debug Mode

Enable detailed logging:

```bash
# Check current log level
ztp show config raw | grep logging

# Monitor logs in real-time
ztp show log tail

# View full log
ztp show log
```

### Getting Help

1. **Check Documentation**: Review this README and TIPS.md
2. **Search Issues**: https://github.com/PackeTsar/freeztp/issues
3. **Community Support**: Create a new issue with:
   - FreeZTP version (`ztp version`)
   - OS version
   - Relevant logs (`ztp show log`)
   - Configuration (sanitized)
   - Switch model

---

## 🤝 Contributing

We welcome contributions! Here's how to help:

### Reporting Issues

1. Check existing issues: https://github.com/PackeTsar/freeztp/issues
2. Create detailed issue with:
   - Clear description
   - Steps to reproduce
   - Expected vs actual behavior
   - FreeZTP version
   - OS details
   - Logs (sanitized)

### Contributing Code

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test thoroughly
5. Commit changes (`git commit -m 'Add amazing feature'`)
6. Push to branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

### Areas for Contribution

- Additional switch model support (SNMP OIDs)
- Integration with other platforms
- Documentation improvements
- Bug fixes
- Performance optimizations

---

## 📖 Additional Resources

### Documentation

- **[TIPS.md](TIPS.md)** - Advanced use cases and techniques
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Detailed architecture documentation
- **[CHANGELOG.md](CHANGELOG.md)** - Version history and changes

### External Links

- **GitHub Repository**: https://github.com/PackeTsar/freeztp
- **Project Website**: http://www.packetsar.com
- **Author's Blog**: http://blog.packetsar.com
- **Video Tutorial**: https://www.youtube.com/watch?v=x4wZvtHiyvE

### Community

- **Issues/Bug Reports**: https://github.com/PackeTsar/freeztp/issues
- **Discussions**: https://github.com/PackeTsar/freeztp/discussions
- **Pull Requests**: https://github.com/PackeTsar/freeztp/pulls

---

## 📄 License

FreeZTP is released under the GNU General Public License v3.0.

See [LICENSE](LICENSE) for details.

---

## 👨‍💻 Author

**John W Kerns**
- Website: http://www.packetsar.com
- Blog: http://blog.packetsar.com
- GitHub: https://github.com/PackeTsar

---

## 🙏 Acknowledgments

- Contributors and community members
- Cisco for AutoInstall feature
- Open source libraries: Jinja2, pysnmp, tftpy

---

## 📊 Version

Current Version: **v1.4.1**

For version history and changelog, see [VERSIONS.md](VERSIONS.md).

---

**Made with ❤️ for network automation**
