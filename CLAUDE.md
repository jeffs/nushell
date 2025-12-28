# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands

```bash
# Build
cargo build                    # Debug build
cargo build --release          # Optimized release build

# Run tests
cargo test --workspace         # All tests
cargo test --workspace --profile ci  # CI profile (faster, no debug info)
cargo test -p nu-parser        # Single crate
cargo test --package nu-cli --test main -- commands::<command_name>  # Single command

# Code quality (run before every commit)
cargo fmt --all -- --check     # Check formatting
cargo fmt --all                # Apply formatting
cargo clippy --workspace -- -D warnings -D clippy::unwrap_used

# Using toolkit.nu (recommended)
nu -c "use toolkit.nu; toolkit check pr"   # Full PR check: fmt + clippy + test + stdlib
nu -c "use toolkit.nu; toolkit test"       # Run all tests
nu -c "use toolkit.nu; toolkit test stdlib"  # Standard library tests only
nu -c "use toolkit.nu; toolkit fmt"        # Format code
nu -c "use toolkit.nu; toolkit clippy"     # Lint check

# Debugging
cargo run --release -- --log-level trace   # Run with trace logging
cargo run --release -- --log-level trace --log-target file  # Log to file
```

## Architecture Overview

Nushell is a modern shell that treats data as structured objects (tables, records, lists) rather than text streams. It uses a type-directed parser where command signatures define how arguments are parsed.

### Core Crates (in `crates/`)

- **nu-protocol** - Core type definitions, `Value` enum, `Command` trait, errors (`ShellError`, `ParseError`)
- **nu-parser** - Type-directed parser: lexing → lite parsing → full parsing with command signature awareness
- **nu-engine** - Expression evaluator that executes the parsed AST
- **nu-command** - Built-in commands (file ops, data manipulation, system interaction)
- **nu-cli** - REPL functionality using `reedline` library
- **nu-std** - Standard library written in pure Nushell

### Plugin System

- **nu-plugin** - Public API for plugin authors
- **nu-plugin-core/engine/protocol** - Plugin infrastructure (JSON-RPC over stdin/stdout)
- **nu_plugin_*** - Plugin implementations (example, formats, polars, etc.)

### Data Flow

```
Input → Lexer → Lite Parser → Parser (uses command signatures) → Engine → Output
```

The parser looks up command signatures to determine how to parse arguments. For example, `where $x > 10` parses differently depending on whether the signature expects three strings or a math expression.

## Code Style Requirements

**Critical rules:**
- **Never use `.unwrap()`** - Always handle errors with `ShellError` or `ParseError`
- **No panicking on user input** - Nushell must not crash on bad input
- **No nightly features** - Stay ~2 Rust releases behind stable (see `rust-toolchain.toml`)
- **No GPL dependencies** - MIT license only
- **Avoid `unsafe`** - If necessary, include `// SAFETY:` comments
- **No custom macros** - Use functions/generics instead

**Dependencies:**
- Use workspace dependencies from root `Cargo.toml`
- Exact semver: `"1.2.3"` not `^1.2.3`
- No git dependencies in PRs (except `reedline`)
- Check for duplicates: `cargo tree --duplicates`

## Implementing Commands

Commands implement the `Command` trait in `crates/nu-command/src/`:

```rust
impl Command for MyCommand {
    fn name(&self) -> &str { "my-command" }
    fn signature(&self) -> Signature { /* define args and types */ }
    fn usage(&self) -> &str { "Description" }
    fn examples(&self) -> Vec<Example> { /* these become tests! */ }
    fn run(&self, call: &Call, input: PipelineData) -> Result<PipelineData, ShellError> { /* impl */ }
}
```

Examples in `examples()` are automatically run as tests, ensuring documentation stays in sync.

## PR Requirements

- Title format: `Fix URL parsing in \`http get\` (#1234)`
- Must pass: `toolkit check pr` (fmt, clippy, test, stdlib)
- Include release notes summary (brief, user-focused)
- Link issues: "Fixes #1234" or "Closes #5678"
- One functional change per PR - don't mix unrelated changes

## Reusable Scripts (`var/tools/`)

When a task requires repeated attempts, complex shell escaping, or multi-step operations that could silently fail as one-liners, write a reusable `.nu` script in `var/tools/` instead of a fragile shell command. This reduces trial-and-error and makes future sessions faster.

For complex transformations—especially those involving AST manipulation or batch refactoring—consider writing a small Rust program that uses crates like `nu-parser` directly. This is more reliable than text-based shell pipelines and leverages the existing toolchain.
