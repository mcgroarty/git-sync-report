# Git Sync Report - Design Document

## Overview

`git-sync-report` is a cross-platform Python command-line utility that monitors the synchronization status of Git repositories across multiple directories. It provides an overview of repository states including uncommitted changes, pending pulls/pushes, and synchronization status with remote repositories.

## Features

### Core Functionality
- **Repository Discovery**: Recursively scan configured directories for Git repositories
- **Status Reporting**: Display sync status for each discovered repository
- **Directory Management**: Add, remove, and edit monitored directory lists
- **Cross-Platform Support**: Compatible with macOS, Windows, and Linux
- **Persistent Configuration**: Store settings in platform-appropriate locations

### Command-Line Interface
- **Help System**: Comprehensive usage information via `--help`
- **Directory Management**: Commands to add/remove monitored directory lists
- **Configuration Editing**: Launch system editor to modify directory list via `config-edit`
- **Reporting**: Default report operation with detailed repository status
- **First-Run Behavior**: If no directories are configured, warns user to add directories and exits

## Architecture

### Command Structure

```bash
# Direct execution with shebang
./git-sync-report [COMMAND] [OPTIONS]

Commands:
  report                  Generate repository status report (default command)
  add <directory>         Add directory to monitoring list
  remove <number>         Remove directory by number (from list command)
  config-edit             Edit directory list in default editor
  config-show             Show current configuration and file location
  list                    Show currently monitored directories with numbers
  help                    Show help information

Options:
  --help, -h              Show help message
  --version               Show program version and exit
  --verbose, -v           Verbose output (for report command)
  --quiet, -q             Quiet output (errors only, for report command)
```

### First-Run and No-Directories Behavior
When no directories are configured (first run or after removing all directories):
- Display warning: "No directories configured for monitoring. Use 'add <directory>' to begin."
- Exit with status code 1

### Configuration Management

### Directory Management

#### Directory List Behavior
- **Automatic Sorting**: Directory list is kept sorted alphabetically for consistency
- **Numbered Access**: `list` command displays directories with index numbers for easy reference
- **Index-Based Removal**: `remove` command accepts index numbers instead of full paths
- **Auto-List After Remove**: Successful removal automatically displays updated numbered list to prevent stale index usage
- **Auto-List After Add**: Successful addition automatically displays updated list with marker showing newly added directory

#### Example Directory Management
```bash
# Add directories (automatically sorted with feedback)
git-sync-report add /home/user/projects
# Output:
# Added: /home/user/projects
#
# Monitored directories:
# 1. /home/user/projects ← NEW

git-sync-report add /home/user/work
# Output:
# Added: /home/user/work
#
# Monitored directories:
# 1. /home/user/projects
# 2. /home/user/work ← NEW

git-sync-report add /opt/repositories
# Output:
# Added: /opt/repositories
#
# Monitored directories:
# 1. /home/user/projects
# 2. /home/user/work
# 3. /opt/repositories ← NEW

# List directories with numbers (no markers)
git-sync-report list
# Output:
# Monitored directories:
# 1. /home/user/projects
# 2. /home/user/work  
# 3. /opt/repositories

# Remove by number (easier than typing full path)
git-sync-report remove 2
# Output:
# Removed: /home/user/work
# 
# Updated monitored directories:
# 1. /home/user/projects
# 2. /opt/repositories

# Show current configuration and location
git-sync-report config-show
# Output:
# Configuration File: /home/user/.config/git-sync-report/config.json
# 
# Current Configuration:
# {
#   "version": "0.1",
#   "settings": {
#     "editor": "auto",
#     "ignore_patterns": [".git", "node_modules", "__pycache__"]
#   },
#   "directories": [
#     "/home/user/projects",
#     "/opt/repositories"
#   ]
# }
```

#### Storage Location
Uses `platformdirs` library for cross-platform configuration directories:
- **macOS**: `~/Library/Application Support/git-sync-report/`
- **Linux**: `~/.config/git-sync-report/` or `$XDG_CONFIG_HOME/git-sync-report/`
- **Windows**: `%APPDATA%\git-sync-report\`

#### Configuration Files
- `config.json`: Main configuration file containing all settings and directory list
- Created automatically when the first directory is added

#### Configuration Schema
```json
{
  "version": "0.1",
  "settings": {
    "editor": "auto",
    "ignore_patterns": [".git", "node_modules", "__pycache__"]
  },
  "directories": [
    "/path/to/directory1",
    "/path/to/directory2"
  ]
}
```

### Repository Status Detection

#### Git Status Categories
1. **Up to Date**: No local changes, synchronized with remote
2. **Uncommitted Changes**: Modified files in working directory
3. **Staged Changes**: Files staged for commit
4. **Unpushed Commits**: Local commits not pushed to remote
5. **Unpulled Changes**: Remote changes not pulled locally
6. **Diverged**: Both local and remote have unique commits
7. **No Remote**: Repository has no configured remote
8. **Detached HEAD**: Repository in detached HEAD state
9. **Remote Access Issues**: Cannot check remote status due to network, authentication, or permission problems - display available local information

#### Status Detection Logic
```python
def get_repo_status(repo_path):
    """
    Analyze Git repository status
    Returns: GitStatus object with detailed state information
    """
    status = {
        'path': repo_path,
        'branch': get_current_branch(),
        'remote': get_remote_info(),
        'uncommitted': has_uncommitted_changes(),
        'staged': has_staged_changes(),
        'unpushed': count_unpushed_commits(),
        'unpulled': count_unpulled_commits(),
        'is_clean': is_working_directory_clean(),
        'last_commit': get_last_commit_info(),
        'remote_access_error': None  # Set if remote operations fail
    }
    return GitStatus(status)
```

### Output Formatting

#### Default Output Format
```
Git Repository Status Report
Generated: 2025-11-01 10:30:45

📁 /Users/username/projects/
├── 🟢 project1                 [main] ✓ Up to date
├── 🟡 project2                 [develop] ⚠ 3 uncommitted files
├── 🔵 project3                 [feature/new] ↑ 2 commits to push
├── 🟠 project4                 [main] ↓ 5 commits to pull
└── 🔴 project5                 [main] ⚡ Diverged (2↑ 3↓)

📁 /Users/username/work/
├── 🟢 client-app              [main] ✓ Up to date
├── 🟡 internal-tool           [main] 📝 Staged changes ready
└── ⚪ private-repo            [main] ❓ Cannot check remote (auth required)

Summary: 8 repositories found
- 2 up to date
- 2 with uncommitted changes  
- 1 needs push
- 1 needs pull
- 1 diverged
- 1 remote access limited
```

#### Verbose Output
Includes additional information:
- Last commit message and timestamp
- Remote repository URLs
- Specific file counts for changes
- Branch tracking information

## Implementation Details

### Core Classes

#### GitSyncReport (Main Application)
```python
class GitSyncReport:
    def __init__(self):
        self.config = Config()
        self.scanner = RepositoryScanner()
        
    def run(self, args):
        """Main entry point"""
        
    def generate_report(self):
        """Execute repository status report"""
        
    def add_directory(self, path):
        """Add directory to monitoring list, keep sorted, and show updated list with marker"""
        
    def remove_directory(self, index):
        """Remove directory from monitoring list by index number and show updated list"""
        
    def list_directories(self):
        """List all monitored directories with numbers"""
        
    def config_edit(self):
        """Launch editor for directory list"""
        
    def config_show(self):
        """Display current configuration and file location"""
```

#### Config (Configuration Manager)
```python
class Config:
    def __init__(self):
        self.config_dir = get_config_directory()
        self.config_file = self.config_dir / 'config.json'
        
    def load(self):
        """Load configuration from file"""
        
    def save(self):
        """Save configuration to file"""
        
    def get_directories(self):
        """Get list of monitored directories"""
        
    def add_directory(self, path):
        """Add directory to configuration and keep list sorted"""
        
    def remove_directory(self, index):
        """Remove directory from configuration by index number"""
```

#### RepositoryScanner
```python
class RepositoryScanner:
    def __init__(self, config):
        self.config = config
        
    def scan_directories(self):
        """Scan all configured directories for Git repositories"""
        
    def find_git_repos(self, directory):
        """Recursively find Git repositories in directory, skipping ignore patterns"""
        
    def analyze_repository(self, repo_path):
        """Analyze single repository status"""
```

#### GitStatus (Repository Status)
```python
class GitStatus:
    def __init__(self, repo_path):
        self.path = repo_path
        self.branch = None
        self.remote = None
        self.status = 'unknown'
        
    def analyze(self):
        """Perform Git status analysis"""
        
    def get_display_status(self):
        """Get formatted status for display"""
```

### Dependencies

#### Required Python Packages
- `platformdirs`: Cross-platform directory paths
- `gitpython`: Git repository interaction

#### Built-in Python Modules Used
- `argparse`: Command-line interface framework
- `json`: Configuration file handling
- `os`: File system operations
- `sys`: System-specific parameters
- `subprocess`: External editor launching
- `pathlib`: Modern path handling

#### Development Dependencies
- `pytest`: Testing framework
- `pytest-cov`: Coverage reporting
- `black`: Code formatting
- `flake8`: Linting
- `mypy`: Type checking

### Error Handling

#### Common Error Scenarios
1. **Configuration Directory Creation Failure**
   - Fallback to temporary directory
   - Clear error messages to user

2. **Git Repository Access Issues**
   - Permission denied
   - Corrupted repositories
   - Network connectivity for remote status
   - SSH key authentication failures
   - HTTP/HTTPS credential issues
   - Private repository access limitations

3. **Editor Launch Failures**
   - Editor selection order: `$EDITOR` → `vi` → `notepad.exe` (Windows) → `TextEdit` (macOS)
   - If no suitable editor found, warn user and advise setting `$EDITOR` environment variable

#### Directory Management Implementation
```python
def add_directory(self, path):
    """Add directory and maintain sorted order, return info for display"""
    normalized_path = os.path.abspath(os.path.expanduser(path))
    if normalized_path not in self.directories:
        self.directories.append(normalized_path)
        self.directories.sort()  # Keep alphabetically sorted
        self.save()
        # Find index of newly added directory for marker
        new_index = self.directories.index(normalized_path) + 1
        return True, normalized_path, self.list_directories(), new_index
    return False, normalized_path, None, None  # Already exists

def remove_directory(self, index):
    """Remove directory by index number (1-based) and return info for display"""
    if 1 <= index <= len(self.directories):
        removed = self.directories.pop(index - 1)
        self.save()
        return removed, self.list_directories()  # Return removed path and updated list
    
    # Invalid index - show list again, print error, and exit
    print("Error: Invalid directory number:", index)
    print("\nCurrent directories:")
    for i, dir_path in enumerate(self.directories, 1):
        print(f"{i}. {dir_path}")
    sys.exit(1)

def list_directories(self):
    """Get numbered list of directories for display"""
    return [(i + 1, dir_path) for i, dir_path in enumerate(self.directories)]

def list_directories_with_marker(self, marker_index=None):
    """Get numbered list with optional marker for newly added/changed item"""
    result = []
    for i, dir_path in enumerate(self.directories):
        number = i + 1
        marker = " ← NEW" if marker_index and number == marker_index else ""
        result.append((number, dir_path, marker))
    return result
```

#### Error Recovery Strategies
```python
def safe_git_operation(repo_path, operation):
    """
    Execute Git operations with error handling
    Returns: (success: bool, result: any, error: str)
    """
    try:
        result = operation(repo_path)
        return True, result, None
    except GitCommandError as e:
        return False, None, str(e)
    except PermissionError:
        return False, None, f"Permission denied: {repo_path}"
    except Exception as e:
        return False, None, f"Unexpected error: {e}"

def check_remote_status_with_fallback(repo_path):
    """
    Check remote status with graceful fallback for access issues
    Returns: (can_access_remote: bool, status_info: dict, error_type: str)
    """
    try:
        # Attempt to fetch remote info with timeout
        success, remote_info, error = safe_git_operation(
            repo_path, 
            lambda path: git.cmd.Git(path).execute(['ls-remote', '--heads', 'origin'], timeout=10)
        )
        
        if success:
            return True, remote_info, None
        else:
            # Classify the error type
            if "authentication" in error.lower() or "permission denied" in error.lower():
                return False, None, "auth_required"
            elif "network" in error.lower() or "timeout" in error.lower():
                return False, None, "network_issue"
            elif "not found" in error.lower():
                return False, None, "repo_not_found"
            else:
                return False, None, "unknown_error"
                
    except Exception as e:
        return False, None, "access_error"
```

### Performance Considerations

#### Simple and Reliable
- Sequential processing of repositories for predictable, maintainable code
- Real-time Git status evaluation for fresh results every run
- Cleanup of subprocess resources
- Full recursive directory scanning as configured

### Testing Strategy

#### Unit Tests
- Configuration management
- Git status analysis
- Directory analysis logic
- Command-line argument parsing

#### Integration Tests
- End-to-end command execution
- Cross-platform configuration paths
- Git repository interaction

#### Test Repository Setup
```python
def create_test_repo(path, status='clean'):
    """Create test Git repository with specified status"""
    repo = git.Repo.init(path)
    # Set up repository state based on status parameter
    return repo
```

### Platform-Specific Considerations

#### macOS
- Use system `git` command
- Respect Gatekeeper and security policies
- Support for Homebrew installations

#### Windows
- Handle Windows path separators
- Support for Git for Windows
- PowerShell compatibility

#### Linux
- Support various distributions
- Handle different Git installations
- Respect XDG Base Directory Specification

### Future Enhancements

#### Planned Features
- **Alternate Report Formats**: Export reports in various formats (JSON, CSV, HTML) for integration with other tools

## Implementation

### File Structure
- Single Python file: `git-sync-report` (executable with shebang)
- Direct execution: `./git-sync-report [COMMAND] [OPTIONS]`
- Git repositories detected by presence of `.git` directory

This design provides a solid foundation for a robust, cross-platform Git repository monitoring tool that is both user-friendly and extensible.