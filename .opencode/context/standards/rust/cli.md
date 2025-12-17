# CLI Standards

## Architecture
- **Bin/Lib Split**: Strictly separate `main.rs` (CLI concerns) and `lib.rs` (core logic).
  - `main.rs`: Argument parsing, logging setup, exit code translation.
  - `lib.rs`: Business logic, typed errors, reusable components.

## Output Streams
- **stdout**: Reserved for the primary output of the tool (data, JSON, protocol payloads).
- **stderr**: Reserved for logs, debug info, and errors.
- **Separation**: Never mix logs with data. This ensures the tool's output is pipeable and machine-readable.

## Testing
- **Integration**: Use `assert_cmd` to verify the compiled binary's behavior as a black box.
- **Coverage**: Test CLI arguments, flags, and verify both stdout and stderr content.
