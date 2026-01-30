# Changelog

All notable changes to the Postiz Helm Chart are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - Unreleased

### Added
- **Valkey Upgrade**: Switched from Redis to Valkey (v5.1.4) as the caching backend
- Updated PostgreSQL to v18.2.3
- **External Temporal Support**: Comprehensive setup for external Temporal deployment
- Self-hosted Temporal deployment guide with architecture diagram
- Example values files:
  - `examples/temporal-self-hosted.yaml` - Deploy Temporal in same cluster
  - `examples/temporal-migration.yaml` - For future self-hosted migrations
  - `examples/postiz-external-temporal.yaml` - Postiz with external Temporal
- Simplified secret template with extracted helpers for clarity
- URL encoding for database and Redis credentials (security improvement)

### Changed
- **Architecture**: Temporal is now deployed separately (not embedded)
- **REQUIRED**: `env.TEMPORAL_ADDRESS` must be explicitly configured
- Simplified `postiz-secret.yaml` template (removed nested if/else logic)
- Updated helpers: `postiz.secretDatabaseUrl` and `postiz.secretRedisUrl` for cleaner logic
- Chart size reduced from ~270KB to ~184KB (with deps)
- Improved documentation with multiple Temporal deployment options

### Removed
- Temporal as embedded subchart dependency (now external only)
- Automatic `TEMPORAL_ADDRESS` generation based on release name
- Complex `temporal.*` configuration section from values.yaml

### Why Temporal is Now External

1. **Separation of Concerns**: Temporal is infrastructure, managed independently
2. **Production Ready**: Centralize Temporal for multiple applications
3. **Reduced Complexity**: Simpler chart, easier to configure and maintain
4. **Loose Coupling**: Only `TEMPORAL_ADDRESS` env var needed
5. **Flexibility**: Supports Temporal Cloud, self-hosted, or existing instances

### Migration Notes

For users upgrading to v1.1.0:
1. Deploy Temporal separately using provided example values
2. Configure `env.TEMPORAL_ADDRESS` in values
3. See README section "Temporal Workflow Engine Setup" for detailed steps
4. Self-hosted migration guide available in README

## [1.0.5] - 2024

### Added
- **Extra Containers Support**: Added ability to deploy sidecar containers alongside the main Postiz application via `extraContainers` values
- Sidecar pattern support for extending application functionality

### Changed
- Improved deployment configuration to support additional containers

## [1.0.4] - 2024

### Added
- **Volume Management**: Added support for custom volumes and volume mounts
- `extraVolumes` configuration for defining additional volumes
- `extraVolumeMounts` configuration for mounting volumes to the main container
- Enhanced storage flexibility for different deployment scenarios

### Changed
- Improved default volume configuration

## [1.0.2] - 2024

### Added
- **Ingress Support**: Added configurable Ingress resource for routing external traffic
- Support for custom ingress classes, annotations, hosts, and TLS configuration
- `ingress.extraRules` for additional routing rules
- Bitnami dependencies for PostgreSQL and Redis subcharts

### Changed
- Renamed template files to `.yaml` extension for consistency
- Added Bitnami chart repository integration
- Enhanced maintainer metadata

### Fixed
- Chart lint validation issues
- Trailing whitespace and line ending issues

## [1.0.1] - 2024

### Added
- Initial chart release
- Postiz application deployment template
- ConfigMap for non-sensitive environment variables
- Secret template for sensitive data (DATABASE_URL, REDIS_URL, JWT_SECRET)
- Service template (ClusterIP type)
- ServiceAccount template
- Support for PostgreSQL and Redis dependencies
- Helm values configuration for:
  - Deployment settings (replicas, image, security context)
  - Pod configuration (annotations, resource limits)
  - Service and Ingress settings
  - Dependency subchart values
  - Application environment variables

### Changed
- Set appVersion to 1.3.0
- Updated GitHub Actions workflows
- Improved documentation and instructions

## [0.0.1] - Initial

### Added
- Project initialization
- Basic Helm chart structure
