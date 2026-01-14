# AGENTS.md - ws-tools

## Project Overview

**ws-tools** is a Rust workspace containing two CLI tools for developer productivity:

1. **ws** - Workspace CLI for git worktrees with tmux layouts
2. **texplore** - Terminal file explorer with git integration

Both tools are designed to work together in a tmux-based development environment.

## Repository Structure

```
ws-tools/
├── Cargo.toml              # Workspace manifest
├── Cargo.lock
├── ws/                     # Main workspace CLI
│   ├── Cargo.toml
│   ├── src/
│   │   ├── main.rs         # CLI entry point, clap command definitions
│   │   ├── commands.rs     # Command implementations (open, new, list, delete, sync, etc.)
│   │   ├── config.rs       # Configuration: AiTool, GitTool, ExplorerTool enums
│   │   ├── git.rs          # Git operations (worktrees, branches)
│   │   ├── tmux.rs         # Tmux session/layout management
│   │   └── onboarding.rs   # First-run setup wizard (ratatui TUI)
│   └── config/
│       └── lazygit.yml     # Lazygit integration config
└── texplore/               # Terminal file explorer
    ├── Cargo.toml
    └── src/
        └── main.rs         # Single-file TUI app (crossterm-based)
```

## Feature Scopes

### ws CLI (`ws/`)

| Feature | Files | Description |
|---------|-------|-------------|
| **Commands** | `main.rs`, `commands.rs` | open, new, list, select, delete, sync, doctor, status, config, init, ai |
| **AI Tool Config** | `config.rs` | AiTool enum - droid, claude, codex, gemini, copilot, vibe |
| **Git Tool Config** | `config.rs` | GitTool enum - lazygit, gitui, tig, custom |
| **Explorer Config** | `config.rs` | ExplorerTool enum - texplore, yazi, ranger, lf, nnn, custom |
| **Tmux Layouts** | `tmux.rs` | Large (5 panes) and small (3 panes) display layouts |
| **Git Operations** | `git.rs` | Worktree CRUD, branch management |
| **Onboarding** | `onboarding.rs` | First-run TUI wizard with ASCII animation |

### texplore (`texplore/`)

| Feature | Location | Description |
|---------|----------|-------------|
| **File Tree** | `main.rs` | Expandable tree with icons, git status |
| **Git Integration** | `main.rs` (`load_git_status`, `GitStatus`) | Shows file status, ahead/behind counts |
| **File Viewer** | `main.rs` (`Viewer`, `open_with_bat`) | View files with bat syntax highlighting |
| **Keyboard Nav** | `main.rs` (`handle_key`) | vim-style navigation (j/k/h/l/g/G) |
| **Mouse Support** | `main.rs` (`handle_mouse`) | Click, double-click, scroll |
| **File Deletion** | `main.rs` (`confirm_delete`) | Trash-based deletion with confirmation |

## Adding a New AI Tool

To add a new AI tool to ws:

1. **Edit `ws/src/config.rs`**:
   - Add variant to `AiTool` enum
   - Add `command()` mapping (CLI command to run)
   - Add `binary()` mapping (binary name for installation check)
   - Add `name()` mapping (display name)
   - Add `from_str()` parsing (config file values)
   - Add to `all()` array
   - Update config file comment in `save()`

2. **Test**: `cargo build && cargo test`

Example (adding Vibe):
```rust
pub enum AiTool {
    // ...existing...
    Vibe,    // Mistral Vibe CLI
}

impl AiTool {
    pub fn command(&self) -> &'static str {
        match self {
            // ...existing...
            AiTool::Vibe => "vibe",
        }
    }
    // ... similar for binary(), name(), from_str(), all()
}
```

## Configuration

- **Config path**: `~/.ws/config.toml`
- **Workspaces dir**: `~/.ws/workspaces/`

Config format:
```toml
ai_tool = "droid"
git_tool = "lazygit"
explorer_tool = "texplore"
```

## Release Process

1. **Update version** in both `ws/Cargo.toml` and `texplore/Cargo.toml`
2. **Commit**: `git commit -am "Bump version to X.Y.Z"`
3. **Tag**: `git tag vX.Y.Z`
4. **Push**: `git push && git push --tags`

The GitHub Actions workflow (`.github/workflows/release.yml`) will:
- Build for Linux x86_64, macOS x86_64, macOS aarch64
- Create tar.gz archives with SHA256 checksums
- Create GitHub release with changelog

The Homebrew workflow (`.github/workflows/update-homebrew.yml`) updates the tap automatically.

## Local Development

Build debug:
```bash
cargo build
```

Build release and install locally:
```bash
cargo build --release
cp target/release/ws ~/bin/ws-test
cp target/release/texplore ~/bin/texplore-test
```

Or use the `ws-dev` shell function (if configured):
```bash
ws-dev  # builds and runs ws-test
```

## Dependencies

Key crates:
- **clap** - CLI argument parsing
- **ratatui** - TUI framework (onboarding, status)
- **crossterm** - Terminal manipulation
- **colored** - Terminal colors
- **anyhow** - Error handling
- **dirs** - Home directory detection
- **which** - Binary detection
- **ignore** - Gitignore parsing (texplore)
- **trash** - Safe file deletion (texplore)

## Testing

```bash
cargo test           # Run all tests
cargo clippy         # Lint
cargo fmt --check    # Format check
```

Note: Most functionality requires manual testing with tmux and git repos.
