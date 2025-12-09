# tfmodule-google-cloud-nat

Terraform module for managing Google Cloud NAT (Network Address Translation) with support for automatic or manual IP allocation, custom routing rules, dynamic port allocation, and comprehensive logging configuration.

## Features

- Automatic or manual NAT IP allocation
- Optional Cloud Router creation with BGP configuration
- Configurable timeout settings for ICMP, UDP, and TCP connections
- Dynamic port allocation support
- Endpoint independent mapping
- Flexible subnet configuration (all subnets or specific subnetworks)
- Comprehensive logging with configurable filters
- Custom NAT rules with IP allocation control
- IP draining support for graceful migration
- Random name suffix generation for unique resource naming

## Usage

### Basic Example - Auto IP Allocation
```hcl
module "cloud_nat" {
  source = "./tfmodule-google-cloud-nat"

  project_id = "my-project-id"
  region     = "us-central1"
  router     = "my-router"
  name       = "my-cloud-nat"
}
```

### Cloud NAT with Router Creation
```hcl
module "cloud_nat_with_router" {
  source = "./tfmodule-google-cloud-nat"

  project_id    = "my-project-id"
  region        = "us-central1"
  network       = "my-vpc"
  create_router = true
  router        = "nat-router"
  router_asn    = "64515"
  name          = "cloud-nat-with-router"

  log_config_enable = true
  log_config_filter = "ERRORS_ONLY"
}
```

### Manual IP Allocation with Multiple IPs
```hcl
resource "google_compute_address" "nat_ips" {
  count   = 2
  project = "my-project-id"
  name    = "nat-ip-${count.index}"
  region  = "us-central1"
}

module "cloud_nat_manual_ip" {
  source = "./tfmodule-google-cloud-nat"

  project_id = "my-project-id"
  region     = "us-central1"
  router     = "my-router"
  name       = "cloud-nat-manual"
  nat_ips    = google_compute_address.nat_ips[*].self_link
}
```

### Specific Subnetworks Configuration
```hcl
module "cloud_nat_specific_subnets" {
  source = "./tfmodule-google-cloud-nat"

  project_id                         = "my-project-id"
  region                             = "us-central1"
  router                             = "my-router"
  name                               = "cloud-nat-subnets"
  source_subnetwork_ip_ranges_to_nat = "LIST_OF_SUBNETWORKS"

  subnetworks = [
    {
      name                     = "projects/my-project-id/regions/us-central1/subnetworks/subnet-1"
      source_ip_ranges_to_nat  = ["ALL_IP_RANGES"]
      secondary_ip_range_names = []
    },
    {
      name                     = "projects/my-project-id/regions/us-central1/subnetworks/subnet-2"
      source_ip_ranges_to_nat  = ["PRIMARY_IP_RANGE"]
      secondary_ip_range_names = []
    }
  ]

  log_config_enable = true
  log_config_filter = "TRANSLATIONS_ONLY"
}
```

### Advanced Configuration with Dynamic Port Allocation
```hcl
module "cloud_nat_advanced" {
  source = "./tfmodule-google-cloud-nat"

  project_id = "my-project-id"
  region     = "us-central1"
  router     = "my-router"
  name       = "cloud-nat-advanced"

  # Port allocation
  enable_dynamic_port_allocation = true
  min_ports_per_vm               = "128"
  max_ports_per_vm               = "512"

  # Timeout configurations
  icmp_idle_timeout_sec            = "60"
  udp_idle_timeout_sec             = "60"
  tcp_established_idle_timeout_sec = "1800"
  tcp_transitory_idle_timeout_sec  = "60"
  tcp_time_wait_timeout_sec        = "180"

  # Endpoint independent mapping
  enable_endpoint_independent_mapping = true

  # Logging
  log_config_enable = true
  log_config_filter = "ALL"
}
```

### NAT with Custom Rules
```hcl
resource "google_compute_address" "rule_nat_ips" {
  count   = 3
  project = "my-project-id"
  name    = "rule-nat-ip-${count.index}"
  region  = "us-central1"
}

module "cloud_nat_with_rules" {
  source = "./tfmodule-google-cloud-nat"

  project_id = "my-project-id"
  region     = "us-central1"
  router     = "my-router"
  name       = "cloud-nat-rules"

  rules = [
    {
      description = "Route traffic from subnet-1 through specific IPs"
      match       = "inIpRange(destination.ip, '10.0.1.0/24')"
      rule_number = 100
      action = {
        source_nat_active_ips = [google_compute_address.rule_nat_ips[0].self_link]
        source_nat_drain_ips  = []
      }
    },
    {
      description = "Route traffic from subnet-2 through different IPs"
      match       = "inIpRange(destination.ip, '10.0.2.0/24')"
      rule_number = 200
      action = {
        source_nat_active_ips = [
          google_compute_address.rule_nat_ips[1].self_link,
          google_compute_address.rule_nat_ips[2].self_link
        ]
        source_nat_drain_ips = []
      }
    }
  ]
}
```

### IP Draining for Graceful Migration
```hcl
resource "google_compute_address" "new_nat_ips" {
  count   = 2
  project = "my-project-id"
  name    = "new-nat-ip-${count.index}"
  region  = "us-central1"
}

resource "google_compute_address" "old_nat_ips" {
  count   = 2
  project = "my-project-id"
  name    = "old-nat-ip-${count.index}"
  region  = "us-central1"
}

module "cloud_nat_draining" {
  source = "./tfmodule-google-cloud-nat"

  project_id    = "my-project-id"
  region        = "us-central1"
  router        = "my-router"
  name          = "cloud-nat-draining"
  nat_ips       = google_compute_address.new_nat_ips[*].self_link
  drain_nat_ips = google_compute_address.old_nat_ips[*].self_link
}
```

### Secondary IP Range NAT
```hcl
module "cloud_nat_secondary_ranges" {
  source = "./tfmodule-google-cloud-nat"

  project_id                         = "my-project-id"
  region                             = "us-central1"
  router                             = "my-router"
  name                               = "cloud-nat-secondary"
  source_subnetwork_ip_ranges_to_nat = "LIST_OF_SUBNETWORKS"

  subnetworks = [
    {
      name                     = "projects/my-project-id/regions/us-central1/subnetworks/gke-subnet"
      source_ip_ranges_to_nat  = ["LIST_OF_SECONDARY_IP_RANGES"]
      secondary_ip_range_names = ["pods", "services"]
    }
  ]
}
```

## Inputs

| Name | Type | Description | Default | Required |
|------|------|-------------|---------|----------|
| project_id | string | The project ID to deploy to | - | yes |
| region | string | The region to deploy to | - | yes |
| router | string | The name of the router in which this NAT will be configured | - | yes |
| name | string | Name of the Cloud NAT. Defaults to 'cloud-nat-RANDOM_SUFFIX' | "" | no |
| network | string | VPC name, only if router is not passed in and is created by the module | "" | no |
| create_router | bool | Create router instead of using an existing one | false | no |
| router_asn | string | Router ASN, only if router is created by the module | "64514" | no |
| router_keepalive_interval | string | Router keepalive interval in seconds, only if router is created by the module | "20" | no |
| nat_ips | list(string) | List of self_links of external IPs. Value of nat_ip_allocate_option is inferred (MANUAL_ONLY if present, AUTO_ONLY otherwise) | [] | no |
| drain_nat_ips | list(string) | A list of URLs of the IP resources to be drained | [] | no |
| source_subnetwork_ip_ranges_to_nat | string | How NAT should be configured per Subnetwork. Valid values: ALL_SUBNETWORKS_ALL_IP_RANGES, ALL_SUBNETWORKS_ALL_PRIMARY_IP_RANGES, LIST_OF_SUBNETWORKS | "ALL_SUBNETWORKS_ALL_IP_RANGES" | no |
| min_ports_per_vm | string | Minimum number of ports allocated to a VM from this NAT config | "64" | no |
| max_ports_per_vm | string | Maximum number of ports allocated to a VM. Only used when enable_dynamic_port_allocation is enabled | null | no |
| enable_dynamic_port_allocation | bool | Enable Dynamic Port Allocation. If enabled, minPortsPerVm must be a power of two >= 32 | false | no |
| enable_endpoint_independent_mapping | bool | Enable endpoint independent mapping | false | no |
| icmp_idle_timeout_sec | string | Timeout (in seconds) for ICMP connections | "30" | no |
| udp_idle_timeout_sec | string | Timeout (in seconds) for UDP connections | "30" | no |
| tcp_established_idle_timeout_sec | string | Timeout (in seconds) for TCP established connections | "1200" | no |
| tcp_transitory_idle_timeout_sec | string | Timeout (in seconds) for TCP transitory connections | "30" | no |
| tcp_time_wait_timeout_sec | string | Timeout (in seconds) for TCP connections in TIME_WAIT state | "120" | no |
| log_config_enable | bool | Indicates whether or not to export logs | false | no |
| log_config_filter | string | Specifies the desired filtering of logs. Valid values: ERRORS_ONLY, TRANSLATIONS_ONLY, ALL | "ALL" | no |
| subnetworks | list(object) | Specifies one or more subnetwork NAT configurations. Each object contains: name, source_ip_ranges_to_nat, secondary_ip_range_names | [] | no |
| rules | list(object) | Specifies one or more rules associated with this NAT. Each object contains: description, match, rule_number, action (with source_nat_active_ips and source_nat_drain_ips) | [] | no |

## Outputs

| Name | Description |
|------|-------------|
| name | Name of the Cloud NAT |
| nat_ip_allocate_option | NAT IP allocation mode (AUTO_ONLY or MANUAL_ONLY) |
| region | Cloud NAT region |
| router_name | Cloud NAT router name |

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.5.0 |
| google | >= 7.0.0, < 8.0.0 |
| google-beta | >= 7.0.0, < 8.0.0 |
| random | >= 3.0 |

## Implementation Notes

### IP Allocation Modes
The module automatically determines the IP allocation mode based on the `nat_ips` variable:
- If `nat_ips` is empty: Uses AUTO_ONLY (Google automatically allocates IPs)
- If `nat_ips` has values: Uses MANUAL_ONLY (uses specified external IPs)

### Router Creation
When `create_router` is set to true:
- Module creates a new Cloud Router with the name specified in `router` variable
- Optionally configures BGP with custom ASN and keepalive interval
- Requires `network` variable to be set

### Dynamic Port Allocation
When enabled, `min_ports_per_vm` must be a power of two and at least 32. This feature helps optimize NAT gateway usage for environments with many VMs.

### Subnetwork Configuration
The `source_ip_ranges_to_nat` options for subnetworks:
- `ALL_IP_RANGES`: NAT all IP ranges (primary and secondary)
- `PRIMARY_IP_RANGE`: NAT only primary IP range
- `LIST_OF_SECONDARY_IP_RANGES`: NAT only specified secondary ranges (requires `secondary_ip_range_names`)

### NAT Rules
Rules provide granular control over which source IPs are used for specific traffic patterns. The `match` field uses CEL expressions to define traffic matching criteria.

### Logging Filters
- `ERRORS_ONLY`: Log only error events
- `TRANSLATIONS_ONLY`: Log only successful NAT translations
- `ALL`: Log all events (errors and translations)

## Changelog

See [CHANGELOG.md](./CHANGELOG.md) for version history and changes.