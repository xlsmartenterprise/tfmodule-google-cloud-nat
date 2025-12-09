# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-12-09

### Added
- Initial release of Cloud NAT Terraform module
- Support for automatic NAT IP allocation (AUTO_ONLY mode)
- Support for manual NAT IP allocation (MANUAL_ONLY mode) with custom external IPs
- Optional Cloud Router creation with BGP configuration
  - Configurable router ASN (default: 64514)
  - Configurable BGP keepalive interval (default: 20 seconds)
- Comprehensive timeout configurations:
  - ICMP idle timeout (default: 30s)
  - UDP idle timeout (default: 30s)
  - TCP established connection timeout (default: 1200s)
  - TCP transitory connection timeout (default: 30s)
  - TCP TIME_WAIT timeout (default: 120s)
- Dynamic port allocation support
  - Configurable minimum ports per VM (default: 64)
  - Configurable maximum ports per VM (requires dynamic allocation enabled)
- Endpoint independent mapping support
- Flexible subnet configuration options:
  - ALL_SUBNETWORKS_ALL_IP_RANGES (default)
  - ALL_SUBNETWORKS_ALL_PRIMARY_IP_RANGES
  - LIST_OF_SUBNETWORKS with granular control
- Subnetwork-specific NAT configuration:
  - Support for primary IP ranges
  - Support for secondary IP ranges (GKE pods/services)
  - Per-subnet source IP range control
- NAT logging configuration:
  - Enable/disable logging
  - Configurable log filters (ERRORS_ONLY, TRANSLATIONS_ONLY, ALL)
- Custom NAT rules support:
  - Rule-based traffic matching with CEL expressions
  - Per-rule source NAT IP allocation
  - Support for active and drain IPs in rules
- IP draining support for graceful NAT IP migration
- Automatic random suffix generation for unique resource naming
- Outputs for NAT name, IP allocation mode, region, and router name