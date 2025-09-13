# Cisco QoS Policy Template

## Overview

The `qos_policy.j2` template implements a comprehensive Quality of Service (QoS) configuration for Cisco IOS devices that **matches the Juniper ASLAN CoS standard**. This template provides equivalent traffic prioritization and bandwidth allocation using Cisco's Modular QoS CLI (MQC) framework.

## Template Purpose

This template creates **class maps and policy maps** to classify and prioritize network traffic based on DSCP markings, ensuring consistent QoS treatment between Cisco and Juniper devices in a multi-vendor environment.

## Traffic Classes & Cisco Implementation

### Class Map to Queue Mapping (Cisco equivalent to Juniper)

| Cisco Class | Juniper Equivalent | DSCP Values | Bandwidth | Treatment |
|-------------|-------------------|-------------|-----------|-----------|
| **NC-CLASS** | NC (Queue 5) | CS6, CS7 (48, 56) | 5% priority | Network control |
| **EF-CLASS** | EF (Queue 4) | EF (46) | 20% priority | Voice traffic |
| **AF41-CLASS** | AF41 (Queue 3) | AF41-43 (34,36,38) | 15% bandwidth | Video traffic |
| **AF31-CLASS** | AF31 (Queue 2) | AF31-33 (26,28,30) | 30% bandwidth | Business apps |
| **DSCP15-CLASS** | dscp15 (Queue 4) | DSCP 15 | 10% bandwidth | Custom traffic |
| **AF11-CLASS** | Default (Queue 1) | AF11-13 (10,12,14) | 10% bandwidth | Standard traffic |
| **BE-CLASS** | BE (Queue 0) | Default (0) | 10% bandwidth | Best effort |

### QoS Mechanisms Used

#### **Priority Queuing**
- **NC-CLASS**: 5% priority bandwidth (strict priority)
- **EF-CLASS**: 20% priority bandwidth (strict priority)  
- **Purpose**: Guarantees low latency for critical traffic

#### **Guaranteed Bandwidth**
- **AF41-CLASS**: 15% guaranteed bandwidth for video
- **AF31-CLASS**: 30% guaranteed bandwidth for business applications
- **Purpose**: Ensures minimum bandwidth allocation during congestion

#### **WRED (Weighted Random Early Detection)**
- Applied to AF classes for congestion avoidance
- DSCP-based dropping for different drop precedences
- Prevents TCP global synchronization

#### **Queue Limits**
- **AF41-CLASS**: 75 packets (video needs smaller buffers)
- **AF31-CLASS**: 100 packets (applications can buffer more)
- **Purpose**: Controls latency and memory usage

## Template Variables

### Required Variables
None - all variables have sensible defaults.

### Optional Variables

| Variable | Default | Description | Example |
|----------|---------|-------------|---------|
| `qos_enable` | `true` | Enable/disable QoS configuration | `false` |
| `qos_profile` | `"ASLAN"` | QoS profile name (matches Juniper) | `"ENTERPRISE"` |
| `trust_dscp` | `true` | Include DSCP trust configuration comments | `false` |

### Usage Examples

#### Default Configuration (ASLAN Profile)
```yaml
# Use defaults - matches Juniper ASLAN CoS profile
qos_enable: true
qos_profile: "ASLAN"
trust_dscp: true
```

#### Custom Enterprise Profile
```yaml
qos_enable: true
qos_profile: "ENTERPRISE"
trust_dscp: true
```

#### Disable QoS
```yaml
qos_enable: false
```

## Generated Configuration

### Class Maps
The template creates class maps for traffic identification:

```cisco
class-map match-any NC-CLASS
 description Network Control Traffic
 match dscp cs6
 match dscp cs7

class-map match-any EF-CLASS
 description Expedited Forwarding - Voice
 match dscp ef
```

### Policy Map
A comprehensive policy map with bandwidth allocation:

```cisco
policy-map ASLAN-POLICY
 class NC-CLASS
  priority percent 5
  set dscp cs6
 class EF-CLASS  
  priority percent 20
  set dscp ef
```

### Interface Application
Apply to interfaces for QoS enforcement:

```cisco
interface GigabitEthernet0/1
 mls qos trust dscp
 service-policy input ASLAN-POLICY  
 service-policy output ASLAN-POLICY
```

## Key Features

### 🔄 **Juniper Compatibility**
- **Matching bandwidth allocation** to Juniper CoS schedulers
- **Equivalent DSCP classification** for consistent treatment
- **Same traffic prioritization** across multi-vendor environment

### 📊 **Bandwidth Management**
- **Priority queuing** for critical traffic (NC: 5%, EF: 20%)
- **Guaranteed bandwidth** for application classes
- **Fair queuing** for default traffic
- **Remaining bandwidth** shared proportionally

### 🎯 **DSCP Preservation & Marking**
- **Trust DSCP** on ingress (preserves existing markings)
- **Set DSCP** on egress (ensures consistent marking)
- **Class-based marking** for traffic conditioning

### ⚡ **Congestion Management**
- **WRED** for TCP traffic optimization
- **Queue limits** to control latency
- **Fair queuing** for default class

## Traffic Flow & Processing

### 1. **Classification** (Ingress)
```
Packet → DSCP Check → Class Map Match → Apply Policy
```

### 2. **Queuing** (During Congestion)
```
Priority Queue: NC-CLASS (5%) + EF-CLASS (20%)
Bandwidth Queue: AF41 (15%) + AF31 (30%) + Others
Default Queue: class-default (fair-queue)
```

### 3. **Marking** (Egress)
```
Traffic → DSCP Remarking → Egress Interface
```

## Deployment Guide

### Step 1: Enable QoS Globally
```cisco
! Enable QoS processing globally
mls qos
```

### Step 2: Apply Policy to Interfaces
```cisco
interface range GigabitEthernet0/1-24
 description User Access Ports
 mls qos trust dscp
 service-policy input ASLAN-POLICY
 service-policy output ASLAN-POLICY
```

### Step 3: Configure Uplink Ports
```cisco
interface GigabitEthernet0/25
 description Uplink to Core
 mls qos trust dscp
 service-policy output ASLAN-POLICY
```

## Integration with Golden Config

### Template Location
```
/golden-config-templates/cisco/qos_policy.j2
```

### Include in Main Template
```jinja2
{# Include QoS configuration #}
{% include 'cisco/qos_policy.j2' %}
```

### Context Variables
Set in Golden Config context or device custom fields:

```yaml
qos_enable: true
qos_profile: "ASLAN"
trust_dscp: true
```

## Verification Commands

### Check QoS Configuration
```cisco
! Show class maps
show class-map

! Show policy maps  
show policy-map ASLAN-POLICY

! Show interface QoS
show mls qos interface GigabitEthernet0/1

! Show QoS statistics
show policy-map interface GigabitEthernet0/1
```

### Monitor QoS Performance
```cisco
! Real-time interface statistics
show policy-map interface GigabitEthernet0/1 input
show policy-map interface GigabitEthernet0/1 output

! Check for drops and congestion
show interfaces GigabitEthernet0/1 | include drops|errors
```

## Multi-Vendor QoS Consistency

### Cisco ↔ Juniper Mapping

| Traffic Type | Cisco Class | Juniper Class | DSCP | Bandwidth |
|--------------|-------------|---------------|------|-----------|
| Network Control | NC-CLASS | NC | 48,56 | 5% priority |
| Voice | EF-CLASS | EF | 46 | 20% priority |
| Video | AF41-CLASS | AF41 | 34,36,38 | 15% guaranteed |
| Business Apps | AF31-CLASS | AF31 | 26,28,30 | 30% guaranteed |
| Custom | DSCP15-CLASS | dscp15 | 15 | 10% guaranteed |
| Standard | AF11-CLASS | Default | 10,12,14 | 10% guaranteed |
| Best Effort | BE-CLASS | BE | 0 | Remainder |

### End-to-End QoS Flow
```
Cisco Edge → [DSCP preserved] → Juniper Core → [DSCP preserved] → Cisco Edge
```

## Best Practices

### ✅ **Recommended Settings**
- **Always enable** `mls qos trust dscp` on inter-switch links
- **Apply policies** to both input and output directions
- **Monitor regularly** for drops and congestion
- **Test thoroughly** with traffic generators

### ⚠️ **Important Considerations**
- **Hardware dependency**: Some features require specific line cards
- **Platform limits**: Check platform-specific QoS capabilities  
- **Buffer tuning**: May need adjustment for high-bandwidth interfaces
- **DSCP transparency**: Ensure DSCP is preserved across WAN links

This template ensures consistent QoS treatment in mixed Cisco/Juniper environments while leveraging platform-specific optimizations.