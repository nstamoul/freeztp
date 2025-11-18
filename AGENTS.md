# FreeZTP Architecture & Component Guide

This document provides detailed information about FreeZTP's internal architecture, components (agents), and how to extend the system.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Core Components](#core-components)
3. [Component Interactions](#component-interactions)
4. [Data Flow](#data-flow)
5. [Extension Points](#extension-points)
6. [Developer Guide](#developer-guide)
7. [Module Reference](#module-reference)

---

## Architecture Overview

FreeZTP is built on a modular Python 2.7 architecture with clearly separated concerns. The system follows an event-driven model where TFTP requests trigger a series of coordinated actions across multiple components.

### High-Level Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                      External Interface                       │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐ │
│  │     CLI      │  │    TFTP      │  │       DHCP         │ │
│  │  Interface   │  │   Clients    │  │      Clients       │ │
│  │              │  │  (Switches)  │  │    (Switches)      │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬─────────────┘ │
└─────────┼──────────────────┼──────────────────┼───────────────┘
          │                  │                  │
┌─────────▼──────────────────▼──────────────────▼───────────────┐
│                      Service Layer                            │
│                                                               │
│  ┌──────────────────┐  ┌──────────────────┐                  │
│  │   CLI Parser     │  │  TFTP Server     │                  │
│  │   & Executor     │  │   (tftpy)        │                  │
│  └──────────────────┘  └─────────┬────────┘                  │
│                                  │                            │
│                           ┌──────▼────────┐                   │
│                           │  Interceptor  │                   │
│                           │   Function    │                   │
│                           └──────┬────────┘                   │
└──────────────────────────────────┼────────────────────────────┘
                                  │
┌─────────────────────────────────▼────────────────────────────┐
│                      Core Engine Layer                        │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │         Configuration Factory (cfact)                 │    │
│  │                                                       │    │
│  │  ┌─────────────┐  ┌────────────┐  ┌──────────────┐  │    │
│  │  │  Keystore   │  │  Template  │  │   SNMP       │  │    │
│  │  │  Resolver   │  │   Merger   │  │  Discovery   │  │    │
│  │  └─────────────┘  └────────────┘  └──────────────┘  │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
│  ┌──────────────┐  ┌────────────────┐  ┌─────────────────┐  │
│  │  File Cache  │  │   Tracking     │  │   Integration   │  │
│  │              │  │    System      │  │    Framework    │  │
│  └──────────────┘  └────────────────┘  └─────────────────┘  │
└───────────────────────────┬───────────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────────┐
│                      Data Layer                               │
│                                                               │
│  ┌──────────────┐  ┌────────────────┐  ┌─────────────────┐  │
│  │   Config     │  │   External     │  │    External     │  │
│  │   Manager    │  │   Keystores    │  │    Templates    │  │
│  │              │  │   (CSV/JSON)   │  │    (Files)      │  │
│  └──────────────┘  └────────────────┘  └─────────────────┘  │
│                                                               │
│  ┌──────────────┐  ┌────────────────┐                        │
│  │  Persistent  │  │   Log          │                        │
│  │    Store     │  │  Management    │                        │
│  └──────────────┘  └────────────────┘                        │
└───────────────────────────────────────────────────────────────┘
```

### Design Principles

1. **Modularity**: Each component has a single, well-defined responsibility
2. **Separation of Concerns**: Data, logic, and presentation are separated
3. **Event-Driven**: Components respond to events (TFTP requests, SNMP responses)
4. **Thread-Safe**: Multi-threaded operations with proper synchronization
5. **Extensible**: Plugin architecture for integrations and external data sources

---

## Core Components

### 1. Configuration Factory (`config_factory`)

**Location**: `ztp.py:233-758`

**Purpose**: The heart of the ZTP engine - generates configurations based on device discovery and keystore matching.

#### Responsibilities

- **TFTP Request Processing**: Determines if requested file should be served
- **Initial Config Generation**: Creates discovery configurations with temp IDs
- **SNMP Coordination**: Initiates and manages SNMP discovery requests
- **Keystore Resolution**: Matches device IDs to keystores/ID arrays
- **Template Merging**: Combines templates with keystore data using Jinja2
- **Image Management**: Handles IOS upgrade file requests
- **Logging**: Records provisioning events and merged configurations

#### Key Methods

```python
class config_factory:
    def lookup(self, filename, ipaddr)
        # Determines if file request is valid for serving

    def request(self, filename, ipaddr, test=False)
        # Generates and returns file content

    def merge_base_config(self, tempid)
        # Creates initial discovery configuration

    def merge_final_config(self, keystoreid, snmpinfo)
        # Generates final device configuration

    def get_keystore_id(self, identifiers)
        # Resolves device ID to keystore

    def create_snmp_request(self, tempid, ipaddr)
        # Initiates SNMP discovery

    def id_configured(self, serial)
        # Checks if serial exists in keystore/ID arrays
```

#### Configuration Flow

```
TFTP Request
    ↓
lookup() → Is this a valid request?
    ↓ (Yes)
request() → What type of file?
    ↓
    ├─→ Initial Config → merge_base_config()
    │                    create_snmp_request()
    │
    ├─→ Image File → Return image filename
    │               (check suppression)
    │
    └─→ Final Config → get_keystore_id()
                       merge_final_config()
                       send to switch
```

---

### 2. SNMP Query System (`snmp_query`)

**Location**: `ztp.py:759-821`

**Purpose**: Discovers device identity (serial numbers) via SNMP polling.

#### Architecture

```
                     ┌──────────────────┐
                     │  config_factory  │
                     │  creates request │
                     └────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │   snmp_query      │
                    │   instance        │
                    └────────┬──────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
    ┌───────▼───────┐ ┌─────▼──────┐ ┌──────▼───────┐
    │  OID 1 Query  │ │ OID 2 Query│ │ OID 3 Query  │
    │   (Thread)    │ │  (Thread)  │ │   (Thread)   │
    └───────┬───────┘ └─────┬──────┘ └──────┬───────┘
            │                │                │
            └────────────────┼────────────────┘
                             │
                    ┌────────▼──────────┐
                    │  Responses Dict   │
                    │  {oid: serial}    │
                    └───────────────────┘
```

#### Key Features

- **Multi-OID Support**: Queries multiple OIDs in parallel (for different switch models)
- **Threaded Execution**: Each OID query runs in its own thread
- **Auto-Complete Detection**: Marks query complete when any OID returns a value
- **Timeout Handling**: Configurable timeout per OID query
- **Error Tolerance**: Continues if some OIDs fail

#### Usage Pattern

```python
# Create SNMP query
snmp_req = snmp_query(
    tempid="ZTP-ABC123",
    host="192.168.1.50",
    oids={"C3850_SERIAL": "1.3.6.1.2.1.47.1.1.1.1.11.1000"},
    community="public"
)

# Wait for completion
while not snmp_req.complete:
    time.sleep(0.1)

# Get results
serial_number = snmp_req.responses["C3850_SERIAL"]
```

---

### 3. Configuration Manager (`config_manager`)

**Location**: `ztp.py:823-1700`

**Purpose**: Manages all FreeZTP configuration - reading, writing, validating, and providing access to settings.

#### Configuration Storage

FreeZTP uses a JSON-based configuration stored in `/etc/ztp/ztp.cfg`:

```json
{
  "running": {
    "initialfilename": "network-confg",
    "suffix": "-confg",
    "community": "secretcommunity",
    "snmpoid": {...},
    "starttemplate": {...},
    "templates": {...},
    "keyvalstore": {...},
    "idarrays": {...},
    "associations": {...},
    "default-keystore": "...",
    "default-template": "...",
    "imagefile": "...",
    "dhcpd": {...},
    ...
  }
}
```

#### Key Responsibilities

- **Configuration I/O**: Load and save configuration to disk
- **Schema Validation**: Ensure configuration integrity
- **Default Values**: Provide sensible defaults for new installations
- **Update Handling**: Merge new settings with existing configuration
- **Access Layer**: Provide thread-safe access to configuration data

#### Important Methods

```python
class config_manager:
    def save()
        # Write configuration to disk

    def load()
        # Read configuration from disk

    def update(path, value)
        # Update specific configuration item

    def get(path)
        # Retrieve configuration value

    def validate()
        # Check configuration integrity
```

---

### 4. File Cache (`file_cache`)

**Location**: `ztp.py:127-175`

**Purpose**: Caches generated TFTP files to improve performance and reduce redundant processing.

#### Caching Strategy

```
Request → Cache Hit? → Yes → Reset position → Return cached file
             │
             No
             ↓
        Generate New File → Store in Cache → Return new file
             ↑
             │
        (Background maintenance thread cleans expired entries)
```

#### Configuration

```python
# Set cache timeout (seconds)
ztp set file-cache-timeout 10

# Disable caching
ztp set file-cache-timeout 0
```

#### Cache Entry Structure

```python
{
    timestamp: {
        "filename": "network-confg",
        "ipaddr": "192.168.1.50",
        "file": <ztp_dyn_file object>
    }
}
```

#### Maintenance

- **Background Thread**: Runs continuously checking for expired entries
- **Timeout-Based**: Entries expire based on `file-cache-timeout` setting
- **Auto-Cleanup**: Expired entries are automatically removed

---

### 5. Dynamic File Object (`ztp_dyn_file`)

**Location**: `ztp.py:179-229`

**Purpose**: Provides a file-like interface for dynamically generated configurations to the TFTP server.

#### File-Like Interface

Implements methods required by `tftpy` library:

```python
class ztp_dyn_file:
    def read(size)
        # Return next 'size' bytes

    def tell()
        # Return current position

    def seek(arg1, arg2)
        # Seek to position (logged but not used)

    def close()
        # Mark file as closed

    def len()
        # Return remaining bytes
```

#### Lifecycle

```
1. Instantiation → Generate content via cfact.request()
2. TFTP reads chunks → track position, report to tracking system
3. Transfer complete → close() called
4. Object cached or discarded
```

---

### 6. Tracking System (`tracking_class`)

**Location**: `ztp.py:2671-3075`

**Purpose**: Records all provisioning events, TFTP downloads, and device history.

#### Tracked Data

**Downloads**:
```python
{
    timestamp: {
        "ipaddr": "192.168.1.50",
        "filename": "network-confg",
        "port": 5000,
        "position": 1024,
        "source": "ztp_dyn_file"
    }
}
```

**Provisioning**:
```python
{
    timestamp: {
        "Temp ID": "ZTP-ABC123",
        "IP Address": "192.168.1.50",
        "Matched Keystore": "SWITCH01",
        "Status": "Complete",
        "Real IDs": {"C3850_SERIAL": "FCW1234A001"},
        "Timestamp": 1234567890.0
    }
}
```

#### Commands

```bash
# View provisioning history
ztp show provisioning

# View download history
ztp show downloads

# Live monitoring
ztp show downloads live

# Clear history
ztp clear provisioning
ztp clear downloads
```

---

### 7. External Keystore System

**Location**: `ztp.py:3333-3563`

**Purpose**: Allows keystores to be stored in external CSV or JSON files for large-scale deployments.

#### Supported Formats

**CSV** (`external_keystore_csv`):
- Columnar format with header row
- Special columns: `keystore_id`, `association`, `idarray_*`
- All other columns become keystore keys

**JSON** (`external_keystore_json`):
- Nested object structure
- Top-level keys are keystore IDs
- Special keys: `association`, `idarray`

#### Load Process

```
Service Start → external_keystore_main.load()
                        ↓
        ┌───────────────┴───────────────┐
        │                               │
    CSV Files                       JSON Files
        │                               │
        └───────────────┬───────────────┘
                        ↓
            Merged into single structure:
            {
                "keyvalstore": {...},
                "idarrays": {...},
                "associations": {...}
            }
                        ↓
            Available to config_factory
```

---

### 8. Integration Framework

**Location**: `ztp.py:3126-3332`

**Purpose**: Sends notifications to third-party services (Cisco Webex Teams, webhooks, etc.).

#### Architecture

```
                   ┌─────────────────────┐
                   │ integration_main    │
                   └─────────┬───────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
    ┌───────▼─────────┐  ┌──▼───────┐  ┌────▼──────┐
    │ integration_    │  │ Future:  │  │  Future:  │
    │    spark        │  │ Webhook  │  │   Email   │
    │ (Webex Teams)   │  │          │  │           │
    └─────────────────┘  └──────────┘  └───────────┘
```

#### Message Flow

```python
# 1. Event occurs (e.g., provisioning complete)
event_data = {
    "device": "SWITCH01",
    "status": "Complete"
}

# 2. Create integration message
msg = integration_message(
    integration=integration_config,
    message="Switch SWITCH01 provisioned successfully"
)

# 3. Send via appropriate integration
msg.send()
```

#### Adding New Integrations

1. Create new class inheriting from base integration class
2. Implement `send()` method
3. Add integration type to `integration_main.load()`
4. Update configuration schema

---

## Component Interactions

### Complete Provisioning Flow

```
┌──────────┐
│  Switch  │ Powers On
└────┬─────┘
     │
     ▼
┌──────────────────────────────────────┐
│ 1. DHCP Request                      │
│    Switch → DHCP Server              │
│    Response: IP + TFTP Server IP     │
└────┬─────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ 2. Initial Config Request            │
│    Switch → "network-confg" → TFTP   │
└────┬─────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────────────────┐
│ 3. TFTP Interceptor                              │
│    interceptor() → Checks cache                  │
│    → cache.get("network-confg", "192.168.1.50")  │
│    → Returns None (not cached)                   │
└────┬─────────────────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────────────────┐
│ 4. Config Factory Lookup                         │
│    cfact.lookup("network-confg", "192.168.1.50") │
│    → Returns True (valid request)                │
└────┬─────────────────────────────────────────────┘
     │
     ▼
┌───────────────────────────────────────────────────┐
│ 5. Generate Initial Config                        │
│    cfact.request("network-confg", "192.168.1.50") │
│    → Generate temp ID: ZTP-ABC123                 │
│    → Merge initial-template                       │
│    → Create SNMP request                          │
│    → Log to tracking system                       │
└────┬──────────────────────────────────────────────┘
     │
     ├──────────────────────────────┐
     ▼                              ▼
┌─────────────────┐      ┌──────────────────────┐
│ 6a. Return File │      │ 6b. SNMP Discovery   │
│ to Switch       │      │ (Parallel)           │
│                 │      │                      │
│ file_object     │      │ snmp_query created   │
│ → ztp_dyn_file  │      │ → Query all OIDs     │
│ → Cached        │      │ → Get serial number  │
│ → Switch loads  │      │ → Map to temp ID     │
└─────────────────┘      └──────────┬───────────┘
                                    │
     ┌──────────────────────────────┘
     ▼
┌──────────────────────────────────────┐
│ 7. Final Config Request              │
│    Switch → "ZTP-ABC123-confg"       │
│    → TFTP → interceptor()            │
│    → cfact.lookup() → True           │
└────┬─────────────────────────────────┘
     │
     ▼
┌────────────────────────────────────────────┐
│ 8. Keystore Resolution                     │
│    cfact.get_keystore_id()                 │
│    → Look up serial in SNMP responses      │
│    → Search keystores for match            │
│    → Search ID arrays for match            │
│    → Fall back to default-keystore         │
│    → Returns: ("SWITCH01", "FCW1234A001")  │
└────┬───────────────────────────────────────┘
     │
     ▼
┌────────────────────────────────────────────┐
│ 9. Template Resolution                     │
│    → Check associations for SWITCH01       │
│    → Find template: ACCESS_TEMPLATE        │
│    → Load template from config             │
└────┬───────────────────────────────────────┘
     │
     ▼
┌────────────────────────────────────────────┐
│ 10. Configuration Merge                    │
│     cfact.merge_final_config()             │
│     → Load keystore data                   │
│     → Load template                        │
│     → Jinja2 merge                         │
│     → Return final configuration           │
└────┬───────────────────────────────────────┘
     │
     ▼
┌────────────────────────────────────────────┐
│ 11. Delivery & Tracking                    │
│     → Create ztp_dyn_file                  │
│     → Cache file object                    │
│     → Send to switch via TFTP              │
│     → Log to tracking.provision()          │
│     → Log merged config (if enabled)       │
│     → Send integration notifications       │
└────────────────────────────────────────────┘
     │
     ▼
┌──────────┐
│  Switch  │ Fully Configured
└──────────┘
```

---

## Data Flow

### Configuration Data Flow

```
┌─────────────────┐
│   CLI Command   │  ztp set keystore SWITCH01 hostname ACCESS-SW
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│  CLI Parser/Executor    │  Parse command, validate syntax
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  config_manager.update()│  Update in-memory configuration
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  config_manager.save()  │  Write to /etc/ztp/ztp.cfg
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  Persistent Storage     │  JSON file on disk
└─────────────────────────┘
```

### Runtime Data Flow

```
┌────────────────┐
│  Service Start │
└───────┬────────┘
        │
        ├──────────────────────────────┐
        │                              │
        ▼                              ▼
┌────────────────┐          ┌──────────────────────┐
│  Load Config   │          │  Start TFTP Server   │
│  Manager       │          │  (Thread)            │
└───────┬────────┘          └──────────────────────┘
        │
        ▼
┌───────────────────────────────────┐
│  Initialize Components            │
│  - config_factory                 │
│  - file_cache                     │
│  - tracking                       │
│  - external_keystores             │
│  - external_templates             │
│  - integrations                   │
└───────┬───────────────────────────┘
        │
        ▼
┌────────────────────────────────────┐
│  TFTP Server Running               │
│  → Waiting for requests            │
│  → interceptor() handles requests  │
│  → cfact processes each request    │
└────────────────────────────────────┘
```

---

## Extension Points

### 1. Adding New Integration Types

**File**: `ztp.py:3234-3332`

**Steps**:

1. Create new integration class:
```python
class integration_webhook:
    def __init__(self, name, config):
        self.name = name
        self.url = config["url"]
        self.api_key = config.get("api_key", "")

    def send(self, message):
        # Implement webhook POST
        import requests
        headers = {"Authorization": f"Bearer {self.api_key}"}
        response = requests.post(
            self.url,
            json={"text": message},
            headers=headers
        )
        return response.status_code == 200
```

2. Register in `integration_main.load()`:
```python
def load(self):
    for name in config.running["integrations"]:
        itype = config.running["integrations"][name]["type"]
        if itype == "spark":
            self.integrations[name] = integration_spark(name, config)
        elif itype == "webhook":  # Add this
            self.integrations[name] = integration_webhook(name, config)
```

3. Add CLI commands in interpreter for configuration

### 2. Adding External Keystore Types

**File**: `ztp.py:3343-3563`

**Steps**:

1. Create new keystore class:
```python
class external_keystore_database:
    def __init__(self, config):
        self.config = config
        self.data = {"keyvalstore": {}, "idarrays": {}, "associations": {}}

    def load(self):
        # Connect to database
        # Query data
        # Populate self.data
        pass
```

2. Register in `external_keystore_main.load()`:
```python
if store_type == "csv":
    store = external_keystore_csv(store_config)
elif store_type == "json":
    store = external_keystore_json(store_config)
elif store_type == "database":  # Add this
    store = external_keystore_database(store_config)
```

### 3. Custom SNMP OIDs

**Configuration**:
```bash
# Add OID for new switch model
ztp set snmpoid C9200_SERIAL 1.3.6.1.2.1.47.1.1.1.1.11.1000

# Test SNMP
ztp request snmp-test 192.168.1.50
```

**Automated Discovery**:
The SNMP query system automatically tries all configured OIDs in parallel.

### 4. Custom Jinja2 Filters

**File**: Add to `config_factory` or separate module

**Example**:
```python
def custom_filter_uppercase(text):
    return str(text).upper()

def merge_final_config(self, keystoreid, snmpinfo=None):
    # Register custom filter
    env = j2.Environment()
    env.filters['uppercase'] = custom_filter_uppercase

    # Use in template: {{ hostname | uppercase }}
```

---

## Developer Guide

### Setting Up Development Environment

```bash
# Clone repository
git clone https://github.com/PackeTsar/freeztp.git
cd freeztp

# Install dependencies
sudo pip install jinja2 pysnmp tftpy

# Run in test mode (no installation)
sudo python ztp.py run
```

### Code Structure

```
freeztp/
├── ztp.py                      # Main application file
│   ├── Classes (29-3585)
│   │   ├── os_detect           # OS/distribution detection
│   │   ├── file_cache          # File caching system
│   │   ├── ztp_dyn_file        # Dynamic TFTP files
│   │   ├── config_factory      # ZTP engine
│   │   ├── snmp_query          # SNMP discovery
│   │   ├── config_manager      # Configuration management
│   │   ├── log_management      # Logging system
│   │   ├── installer           # Installation routines
│   │   ├── tracking_class      # Provisioning tracking
│   │   ├── persistent_store    # Data persistence
│   │   ├── integration_*       # Integration framework
│   │   ├── external_keystore_* # External data sources
│   │   └── external_templates  # External template system
│   └── Functions (3586-end)
│       ├── start_tftp()        # TFTP server startup
│       └── interpreter()       # CLI interface
├── nstam_ipaddr.py             # IP address utilities (from Ansible)
├── json_sample_keystore.json   # Example external keystore
├── README.md                   # User documentation
├── TIPS.md                     # Advanced usage tips
└── AGENTS.md                   # This file
```

### Adding New Features

#### Example: Add Custom Validation

```python
# In config_manager class
def validate_keystore(self, keystore_id):
    """Validate keystore has required keys"""
    required_keys = ["hostname", "mgmt_ip"]
    keystore = self.running["keyvalstore"].get(keystore_id, {})

    for key in required_keys:
        if key not in keystore:
            raise ValueError(
                f"Keystore {keystore_id} missing required key: {key}"
            )
    return True
```

### Testing

#### Unit Test Example

```python
# Test keystore resolution
def test_keystore_resolution():
    # Setup
    config.running["keyvalstore"]["TEST01"] = {"hostname": "TEST-SW"}
    cfact = config_factory()

    # Test
    result = cfact.get_keystore_id({"serial": "TEST01"})

    # Verify
    assert result == ("TEST01", "TEST01")
    print("Test passed")
```

#### Integration Test

```bash
# 1. Configure test keystore
ztp set keystore TEST01 hostname TEST-SWITCH
ztp set keystore TEST01 mgmt_ip 10.0.0.1

# 2. Test merge
ztp request merge-test TEST01

# 3. Verify output
```

### Logging and Debugging

```python
# Add debug logging
log("module.function: Debug message here")

# Log important events
log("module.function: Important event occurred: %s" % data)

# View logs
ztp show log tail
```

### Performance Considerations

1. **File Caching**: Reduces repeated generation
2. **SNMP Parallelization**: Multiple OIDs queried simultaneously
3. **Thread Safety**: Use locks for shared resources
4. **Database-like Access**: Avoid full config reloads

---

## Module Reference

### Import Dependencies

```python
# Native Python modules
import os
import re
import sys
import csv
import time
import json
import curses
import socket
import logging
import platform
import commands
import threading
from inspect import getmembers, isfunction

# External dependencies
import jinja2 as j2              # Template engine
from jinja2 import meta          # Template introspection
import pysnmp                    # SNMP library
import tftpy                     # TFTP server

# Internal modules
import nstam_ipaddr              # IP address utilities
```

### Global Objects

```python
# Created at runtime
config = config_manager()          # Configuration management
cfact = config_factory()           # ZTP engine
cache = file_cache()               # File caching
tracking = tracking_class()        # Provisioning tracking
external_keystores = external_keystore_main()
external_templates = external_templates_main()
integrations = integration_main()
```

### Thread Model

FreeZTP uses multiple threads:

```
Main Thread
├── TFTP Server Thread
│   └── Request Handler Threads (spawned per request)
├── File Cache Maintenance Thread
│   └── Cache Cleanup Sub-thread
├── SNMP Query Threads (per device)
│   └── Individual OID Query Threads
└── Integration Threads (for notifications)
```

**Synchronization**: Uses Python threading locks for shared resources

---

## Best Practices

### For Developers

1. **Follow Existing Patterns**: Match coding style of existing components
2. **Add Logging**: Use `log()` function liberally
3. **Error Handling**: Catch exceptions and log them
4. **Documentation**: Add docstrings to new classes/functions
5. **Testing**: Test new features thoroughly before PR
6. **Backward Compatibility**: Don't break existing configurations

### For Operators

1. **Version Control**: Store ZTP configuration in git
2. **Testing**: Use `merge-test` before deploying changes
3. **Monitoring**: Watch logs during provisioning
4. **Backups**: Regular backups of `/etc/ztp/` directory
5. **External Data**: Use external keystores for large deployments

---

## Troubleshooting Components

### Component Health Checks

```bash
# Config Manager
ztp show config raw

# TFTP Server
sudo netstat -ulnp | grep :69

# File Cache
ztp show config raw | jq '.running["file-cache-timeout"]'

# Tracking System
ztp show provisioning
ztp show downloads

# External Keystores
ztp request external-keystore-test <name>

# Integrations
ztp request integration-test <name>
```

### Common Component Issues

**SNMP Discovery Fails**:
- Check OIDs: `ztp show config | grep snmpoid`
- Test manually: `ztp request snmp-test <ip>`
- Verify community string in initial-template

**Template Merge Errors**:
- Test merge: `ztp request merge-test <id>`
- Check Jinja2 syntax
- Verify all variables exist in keystore

**Cache Issues**:
- Clear cache: restart service
- Disable caching: `ztp set file-cache-timeout 0`

**Integration Failures**:
- Test integration: `ztp request integration-test <name>`
- Check API credentials
- Review logs: `ztp show log | grep -i integration`

---

## Contributing

### Submission Guidelines

1. **Fork & Branch**: Create feature branch
2. **Code Standards**: Follow PEP 8 (where practical)
3. **Testing**: Test on supported OS platforms
4. **Documentation**: Update this file if adding components
5. **Pull Request**: Clear description of changes

### Code Review Checklist

- [ ] Follows existing code patterns
- [ ] Includes appropriate logging
- [ ] Handles errors gracefully
- [ ] Tested on CentOS/Ubuntu
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
- [ ] Thread-safe (if applicable)

---

## Future Enhancements

### Planned Features

1. **Enhanced Integrations**
   - Webhook support
   - Email notifications
   - Syslog integration

2. **Database Backend**
   - PostgreSQL keystore support
   - MySQL keystore support

3. **API Interface**
   - RESTful API for configuration
   - API for provisioning status

4. **Enhanced Discovery**
   - LLDP-based discovery
   - MAC address discovery
   - Flexible discovery plugins

5. **Web Interface**
   - Configuration management UI
   - Real-time monitoring dashboard
   - Provisioning visualization

---

## References

### External Documentation

- **Jinja2**: https://jinja.palletsprojects.com/
- **PySNMP**: http://snmplabs.com/pysnmp/
- **TFTPy**: https://github.com/msoulier/tftpy
- **Cisco AutoInstall**: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/fundamentals/configuration/15mt/fundamentals-15-mt-book/cf-autoinstall.html

### Internal Documentation

- **README.md**: User guide and setup instructions
- **TIPS.md**: Advanced use cases and techniques
- **Code Comments**: Inline documentation in `ztp.py`

---

## Conclusion

FreeZTP's modular architecture makes it extensible and maintainable. Understanding these components and their interactions is key to effectively using, troubleshooting, and extending the system.

For questions or contributions, visit: https://github.com/PackeTsar/freeztp

---

**Document Version**: 1.0
**Last Updated**: 2025
**Maintainer**: FreeZTP Community
