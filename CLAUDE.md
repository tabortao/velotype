# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build Commands
```bash
# Build in debug mode (default)
cargo build

# Build in release mode (optimized, single binary)
cargo build --release

# Run the application
cargo run

# Run with specific features disabled (e.g., without code highlighting)
cargo run --no-default-features
```

### Development Commands
```bash
# Check code formatting
cargo fmt --check

# Auto-format code
cargo fmt

# Run clippy linter
cargo clippy

# Run tests
cargo test

# Run specific test file
cargo test --test tests
```

### Feature Management
Velotype uses Cargo features for optional functionality:
- `code-highlight-core`: Basic syntax highlighting infrastructure
- `code-highlight-official`: Official language support (Rust, JavaScript, TypeScript, etc.)
- `code-highlight-config`: Configuration file highlighting (YAML, TOML)
- `html-native`: Native HTML parsing support

Default features include all code highlighting options. Disable with `--no-default-features`.

## Architecture Overview

### High-Level Architecture
Velotype is a native desktop Markdown editor built with Rust and GPUI, following a layered architecture:

1. **Application Layer** (`main.rs`, `app_menu.rs`, `app_identity.rs`)
   - Window management and application lifecycle
   - Cross-platform menu system
   - Command-line argument processing

2. **Editor Core** (`src/editor/`)
   - `document.rs`: Core document model and block tree management
   - `render.rs`: GPUI rendering pipeline for the editor interface
   - `tree.rs`: Block tree manipulation and source mapping
   - `selection.rs`: Text and block selection handling
   - `history.rs`: Undo/redo functionality
   - `events.rs`: Input event handling and command processing

3. **Component System** (`src/components/`)
   - `block/`: Editable block runtime and rendering
   - `markdown/`: Markdown parsing, serialization, and data models
   - `latex/`: LaTeX math rendering support

4. **Export System** (`src/export/`)
   - `html.rs`: HTML export with CSS theming
   - `pdf.rs`: PDF export via HTML pipeline

5. **Configuration & Theming**
   - `theme/`: Visual theme system with JSONC configuration
   - `i18n/`: Internationalization and language packs
   - `config.rs`: Application settings and recent files

6. **Network Layer** (`src/net/`)
   - HTTP client for remote image loading
   - Update checking functionality

### Key Design Patterns

- **Block-based Editing**: Documents are represented as editable blocks rather than raw text, enabling structured editing while preserving source compatibility
- **Dual-mode Editing**: Supports both WYSIWYG rendered editing and raw Markdown source editing
- **Theme Inheritance**: Custom themes can partially override defaults, with missing values inherited from built-in themes
- **Progressive Enhancement**: Graceful fallback from structured blocks to raw Markdown for unsupported syntax

### Data Flow
1. **Import**: Markdown → Block tree (structured blocks + preserved source)
2. **Editing**: Block tree manipulation with real-time rendering
3. **Export**: Block tree → Canonical Markdown or themed HTML/PDF

### File Structure Conventions
- `mod.rs`: Module declarations and public API
- Feature-specific files grouped in directories (e.g., `components/block/`)
- Test files use `_tests.rs` suffix or are in `tests/` modules
- Configuration examples in `assets/` directory

### Dependencies of Note
- **GPUI 0.2**: Native UI framework (primary GUI dependency)
- **tree-sitter**: Syntax highlighting and parsing
- **pulldown-cmark**: Markdown parsing fallback
- **ratex-layout/parser/svg**: LaTeX math rendering
- **mermaid-rs-renderer**: Diagram rendering
- **ironpress**: Compression for export

### Platform Support
Targets Windows, Linux, and macOS as portable single-binary applications. Uses platform-specific resource compilation on Windows via `embed-resource`.

### Testing Strategy
- Unit tests embedded in modules with `#[cfg(test)]`
- Integration tests in `src/editor/tests.rs`
- Test support via GPUI's `test-support` feature
- Focus on block tree manipulation and Markdown round-trip fidelity

### Configuration Files
- `Cargo.toml`: Package configuration with optional feature flags
- `build.rs`: Windows resource compilation
- Theme files: JSONC format in user config directory
- Language files: JSONC format for internationalization

### Performance Considerations
- Release builds use LTO and optimization level "s" for size
- Single codegen unit for better optimization
- Panic abort strategy to reduce binary size
- Tree-sitter parsing for efficient syntax highlighting