# Triple VLAN TPID and Priority Configuration Enhancement

## Overview
This change implements full configurability for all three TPIDs (Tag Protocol Identifiers) and all three priority bits (P-bits) for triple VLAN tagging support in BNG Blaster.

## Files Modified

### 1. Data Structures (`src/bngblaster/src/bbl_protocols.h`)
- Added `vlan_three_priority` field to `bbl_ethernet_header_s` struct
- Added `vlan_tpid_inner` and `vlan_tpid_three` fields for configurable TPIDs
- Maintained backward compatibility with existing `qinq` flag

### 2. Packet Encoding (`src/bngblaster/src/bbl_protocols.c`)
- Updated `encode_ethernet()` function to:
  - Apply priority bits to ALL three VLANs (outer, inner, third)
  - Use configurable TPID for inner and third VLANs
  - Maintain backward compatibility with `qinq` flag for outer VLAN

### 3. Configuration Structures (`src/bngblaster/src/bbl_config.h`)
- Added priority fields: `access_vlan_priority_outer`, `access_vlan_priority_inner`, `access_vlan_priority_three`
- Added TPID fields: `access_tpid_outer`, `access_tpid_inner`, `access_tpid_three`

### 4. JSON Parser (`src/bngblaster/src/bbl_config.c`)
- Added support for: `"outer-tpid"`, `"inner-tpid"`, `"third-tpid"`, `"priority-outer"`, `"priority-inner"`, `"priority-third"`
- Updated JSON validation schema

### 5. Session Initialization (Multiple files):
- `src/bngblaster/src/bbl_tcp.c`
- `src/bngblaster/src/bbl_arp_client.c`
- `src/bngblaster/src/bbl_icmp_client.c`
- `src/bngblaster/src/bbl_tun.c`
- Updated to initialize new TPID and priority fields from access config

## Usage Example

To configure TPIDs [0x8100, 0x88a8, 0x8100] and priority bits [0,0,7]:

```json
{
  "access": [{
    "interface": "eth1",
    "outer-vlan": 100,
    "inner-vlan": 200,
    "third-vlan": 300,
    "outer-tpid": 0x8100,
    "inner-tpid": 0x88a8,
    "third-tpid": 0x8100,
    "priority-outer": 0,
    "priority-inner": 0,
    "priority-third": 7,
    "authentication-protocol": "PAP",
    "access-line-profile-id": 1
  }]
}
```

## Backward Compatibility
- Existing configurations continue to work unchanged
- The `qinq` flag is still supported for backward compatibility
- New fields have sensible defaults when not specified

## Testing
- Code compiles successfully
- Test configuration file created: `test_tpid_priority.json`