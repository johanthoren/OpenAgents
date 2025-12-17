# Nagios Plugin Standards

## Protocol Compliance
- **Exit Codes**:
  - `0`: OK
  - `1`: WARNING
  - `2`: CRITICAL
  - `3`: UNKNOWN
- **Output Format**:
  - **stdout**: Must follow the standard format: `STATUS - Message | 'label'=value[UOM];[warn];[crit];[min];[max]`.
  - **stderr**: Debugging information only.

## Implementation
- **Status Enum**: Use a typed enum (e.g., `Status`) to represent the state and handle formatting/exit codes centrally.
- **Thresholds**: Implement standard range syntax (e.g., `@10:20`, `~:10`) for warning/critical thresholds.
