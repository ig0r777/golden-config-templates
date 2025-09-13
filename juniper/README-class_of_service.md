# Juniper Class of Service (CoS) Template

## Overview

The `class_of_service.j2` template implements a comprehensive Quality of Service (QoS) configuration for Juniper devices following enterprise-grade traffic prioritization standards. This template creates a complete CoS framework including traffic classification, queuing, and bandwidth allocation.

## Template Purpose

This template configures **traffic classification, queuing, and scheduling** to prioritize different types of network traffic based on DSCP (Differentiated Services Code Point) markings, ensuring critical traffic receives appropriate treatment.

## Traffic Classes & Prioritization

### Queue Assignment (Priority Order)

| Queue | Class | Purpose | Bandwidth | Priority |
|-------|-------|---------|-----------|----------|
| 5 | **NC** (Network Control) | Routing protocols, management | 5% guaranteed | Highest |
| 4 | **EF** (Expedited Forwarding) | Voice, real-time traffic | 20% shaped | Strict-high |
| 4 | **dscp15** (Custom) | Special applications | Shares EF scheduler | High |
| 3 | **AF41** (Assured Forwarding) | Video, streaming | 15% shaped | Low |
| 2 | **AF31** (Assured Forwarding) | Business applications | 30% shaped | Low |
| 1 | **Default** | Standard traffic | 10% shaped | Low |
| 0 | **BE** (Best Effort) | Internet, background | Remainder | Lowest |

### DSCP to Forwarding Class Mapping

#### Network Control (NC) - Queue 5
- **DSCP 48** (110000) - Network control protocols (OSPF, BGP, etc.)
- **Loss Priority**: Low (highest protection)

#### Expedited Forwarding (EF) - Queue 4  
- **DSCP 40, 32, 41, 43, 33, 35, 49, 45** - Voice traffic
- **Priority**: Strict-high (preempts other traffic)
- **Bandwidth**: 20% shaped

#### Assured Forwarding 41 (AF41) - Queue 3
- **DSCP 24, 46, 28, 30, 38** - Video traffic
- **Bandwidth**: 15% shaped, 12% buffer

#### Assured Forwarding 31 (AF31) - Queue 2
- **DSCP 9, 11, 17, 19, 25, 27, 16** - Business applications
- **Bandwidth**: 30% shaped, 30% buffer

#### Default - Queue 1
- **DSCP 8** - Standard traffic
- **Bandwidth**: 10% shaped, 10% buffer

#### Best Effort (BE) - Queue 0
- **DSCP 0** - Background traffic
- **Bandwidth**: Remainder after other classes

#### Custom (dscp15) - Queue 4
- **DSCP 15** - Special organizational traffic
- **Scheduler**: Shares AF41 scheduler

## Template Variables

### Required Variables
None - all variables have sensible defaults.

### Optional Variables

| Variable | Default | Description | Example |
|----------|---------|-------------|---------|
| `cos_enable` | `true` | Enable/disable CoS configuration | `false` |
| `cos_profile` | `"ASLAN"` | CoS profile name (organizational standard) | `"ENTERPRISE"` |
| `host_outbound_class` | `"NC"` | Forwarding class for host-originated traffic | `"EF"` |
| `host_outbound_dscp` | `"110000"` | DSCP marking for host traffic | `"101000"` |

### Usage Examples

#### Default Configuration
```yaml
# Use all defaults - ASLAN profile with standard settings
cos_enable: true
```

#### Custom Profile
```yaml
cos_enable: true
cos_profile: "ENTERPRISE"
host_outbound_class: "EF"
host_outbound_dscp: "101000"
```

#### Disable CoS
```yaml
cos_enable: false
```

## Key Features

### 🔄 **Dual-Stack Support**
- Separate IPv4 and IPv6 DSCP classifiers
- Consistent treatment across both IP versions

### 🌐 **Interface Coverage**
- Automatic application to all GE (Gigabit Ethernet) interfaces
- Automatic application to all XE (10 Gigabit Ethernet) interfaces
- Unit 0 configuration for standard deployments

### 📊 **Bandwidth Management**
- **Guaranteed bandwidth** for critical traffic (NC: 5%)
- **Shaped bandwidth** with burst capability for real-time traffic (EF: 20%)
- **Remainder bandwidth** for best-effort traffic
- **Buffer management** for application classes

### 🎯 **DSCP Rewrite Rules**
- Outbound DSCP marking for traffic egressing the router
- Consistent marking across IPv4 and IPv6
- Support for dscp15-45 rewrite rule

### ⚡ **Strict Priority**
- EF traffic gets strict-high priority (preempts other classes)
- NC traffic gets guaranteed bandwidth with low loss priority
- Proper priority levels prevent starvation

## Technical Details

### Scheduler Configuration

#### Network Control (NC)
- **Type**: Transmit-rate (guaranteed)
- **Bandwidth**: 5% guaranteed
- **Purpose**: Ensures routing protocols always have bandwidth

#### Expedited Forwarding (EF)
- **Type**: Shaping-rate with strict-high priority  
- **Bandwidth**: 20% shaped
- **Purpose**: Voice traffic with preemption capability

#### Assured Forwarding Classes (AF41, AF31)
- **Type**: Shaping-rate with buffer management
- **Priority**: Low (fair scheduling)
- **Purpose**: Guaranteed bandwidth for business applications

#### Best Effort (BE)
- **Type**: Remainder bandwidth and buffer
- **Purpose**: Uses leftover bandwidth after other classes

### Loss Priority Configuration
- **NC**: Low loss priority (best protection)
- **All others**: High loss priority (standard protection)

## Deployment Considerations

### ✅ **Recommended Use Cases**
- Enterprise network edge routers
- WAN aggregation devices
- Service provider customer edge
- Data center interconnect

### ⚠️ **Important Notes**
- **Voice Traffic**: EF class is optimized for VoIP (low latency, low jitter)
- **Video Traffic**: AF41 provides guaranteed bandwidth with burst capability  
- **Management Traffic**: NC class ensures network control protocols work under congestion
- **Interface Types**: Template applies to physical interfaces (ge-*, xe-*)

### 🔧 **Customization Options**
- Adjust bandwidth percentages for different environments
- Modify DSCP to class mappings for specific applications
- Change cos_profile name for organizational standards
- Add additional interface types if needed

## Integration with Golden Config

### Template Location
```
/golden-config-templates/juniper/class_of_service.j2
```

### Include in Main Template
```jinja2
{# Include CoS configuration #}
{% include 'juniper/class_of_service.j2' %}
```

### Context Variables
Set variables in your Golden Config context or device custom fields:

```yaml
cos_enable: true
cos_profile: "ASLAN"
host_outbound_class: "NC"
host_outbound_dscp: "110000"
```

## Testing & Validation

### Verify Configuration
```bash
# Check CoS configuration
show class-of-service

# Check interface classifiers  
show class-of-service interface ge-*

# Check scheduler maps
show class-of-service scheduler-map map-ASLAN

# Monitor queue statistics
show class-of-service interface ge-0/0/0 
```

### Traffic Testing
- Generate test traffic with different DSCP markings
- Monitor queue utilization and drops
- Verify priority enforcement during congestion

This template provides enterprise-grade QoS suitable for production networks while maintaining flexibility through Jinja2 templating.