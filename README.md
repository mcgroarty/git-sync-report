# git-sync-report

A cross-platform Python CLI tool that monitors Git repositories across multiple directories, providing a unified status report of sync states, uncommitted changes, and pending operations.

## Features

- **Multi-directory scanning**: Recursively discovers Git repositories in configured directories
- **Comprehensive status detection**: Identifies uncommitted changes, staged files, unpushed/unpulled commits, diverged branches, and remote access issues
- **Clean visual output**: Uses emojis and tree-style formatting for easy scanning
- **Cross-platform**: Works on macOS, Linux, and Windows with platform-specific configuration storage
- **Simple management**: Add/remove monitored directories with numbered indexing

## Installation

```bash
# Clone the repository
git clone https://github.com/mcgroarty/git-sync-report.git
cd git-sync-report

# Install dependencies
pip install -r requirements.txt

# Make executable (Unix/macOS)
chmod +x git-sync-report
```

## Quick Start

```bash
# Add directories to monitor
./git-sync-report add ~/projects
./git-sync-report add ~/work

# Generate status report (default command)
./git-sync-report

# List monitored directories
./git-sync-report list

# Remove a directory by number
./git-sync-report remove 2
```

## Usage

```bash
./git-sync-report [COMMAND] [OPTIONS]

Commands:
  report          Generate repository status report (default)
  add <dir>       Add directory to monitoring list
  remove <num>    Remove directory by number (see list command)
  list            Show monitored directories with numbers
  config-edit     Edit configuration in default editor
  config-show     Display current configuration and file location
  help            Show help information

Options:
  --help, -h      Show help message
  --version       Show version information
  --show-clean    Show up-to-date repositories in output (default: hidden)
  --verbose, -v   Verbose output
  --quiet, -q     Quiet output, errors only
```

## Example Output

```text
Git Repository Status Report
Generated: 2025-11-01 10:30:45

📁 /Users/username/projects/
├──  project2                 [develop] ⚠ 3 uncommitted files
├── 🔵 project3                 [feature/new] ↑ 2 commits to push
├── 🟠 project4                 [main] ↓ 5 commits to pull
└── 🔴 project5                 [main] ⚡ Diverged (2↑ 3↓)

📁 /Users/username/work/
└── ⚪ private-repo            [main] ❓ Cannot check remote (auth required)

Summary: 7 repositories found
- 2 up to date (hidden)
- 1 with uncommitted changes
- 1 needs push
- 1 needs pull
- 1 diverged
- 1 remote access limited

Note: Up-to-date repositories are hidden by default. Use --show-clean to display them.
```

## Status Indicators

| Icon | Status | Description |
|------|--------|-------------|
| 🟢 | Up to date | No local changes, synchronized with remote |
| 🟡 | Changes | Uncommitted files or staged changes |
| 🔵 | Push needed | Local commits not pushed to remote |
| 🟠 | Pull needed | Remote changes not pulled locally |
| 🔴 | Diverged | Both local and remote have unique commits |
| ⚪ | Limited access | Cannot check remote (auth/network issues) |
| ⚫ | No remote | Repository has no configured remote |
| 🟣 | Detached HEAD | Repository in detached HEAD state |

## Configuration

Configuration is stored in platform-specific locations:

- **macOS**: `~/Library/Application Support/git-sync-report/config.json`
- **Linux**: `~/.config/git-sync-report/config.json`
- **Windows**: `%APPDATA%\git-sync-report\config.json`

## Requirements

- Python 3.7+
- Git installed and accessible in PATH
- Dependencies: `platformdirs`, `GitPython` (see `requirements.txt`)

## License

Licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.

## Contributing

Report bugs and request enhancements at: [GitHub Issues](https://github.com/mcgroarty/git-sync-report/issues)
