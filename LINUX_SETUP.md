# Linux Setup Guide for AI_RFIC_workflow

This guide explains how to set up and use the AI_RFIC_workflow project on Linux systems.

## Platform Support

The project now supports:
- **Windows**: Original support maintained
- **Linux**: Full support for Keysight ADS on Linux
- **macOS**: Partial support (ADS availability may vary)

## Prerequisites

### System Requirements

- Linux (Ubuntu 18.04+, CentOS 7+, or equivalent)
- Keysight ADS 2024 or later with Linux support
- Python 3.7 or later
- Git

### Required Environment Variables

Before running any scripts, you must set the following environment variables:

```bash
# ADS Installation
export ADS_PYTHON=/opt/Keysight/ADS/tools/python/python3
export ADS_INSTALL_DIR=/opt/Keysight/ADS

# PDK (Optional, but recommended for real workflows)
export PDK_DIR=/home/user/PDK/pdk_2024
export PDK_TECH_DIR=/home/user/PDK/pdk_tech_2024

# Substrate name (Optional)
export SUBSTRATE=microstrip_substrate
```

## Setup Method 1: Using .env File (Recommended)

### 1. Copy and Configure .env

```bash
cp .env.example .env
```

Edit `.env` with your system paths:

```bash
nano .env
```

Example Linux configuration:

```dotenv
# ADS Installation
ADS_PYTHON=/opt/Keysight/ADS/tools/python/python3
ADS_INSTALL_DIR=/opt/Keysight/ADS

# PDK Paths
PDK_DIR=/home/user/PDK/pdk_2024
PDK_TECH_DIR=/home/user/PDK/pdk_tech_2024

# Design Parameters
SUBSTRATE=microstrip_substrate
```

### 2. Source the .env File

```bash
source .env
```

Or add to your `.bashrc` for permanent setup:

```bash
echo "source $(pwd)/.env" >> ~/.bashrc
source ~/.bashrc
```

## Setup Method 2: Using direnv (Alternative)

If you prefer automatic environment loading, install direnv:

```bash
# Ubuntu/Debian
sudo apt-get install direnv

# CentOS/RHEL
sudo yum install direnv

# macOS
brew install direnv
```

Then add this to your `.bashrc` or `.zshrc`:

```bash
eval "$(direnv hook bash)"
# or for zsh:
eval "$(direnv hook zsh)"
```

The `.env` file will be automatically loaded when you enter the project directory.

## Setup Method 3: Manual Shell Commands

If you don't want to use .env or direnv, set variables manually in each terminal session:

```bash
export ADS_PYTHON=/opt/Keysight/ADS/tools/python/python3
export ADS_INSTALL_DIR=/opt/Keysight/ADS
export PDK_DIR=/home/user/PDK/pdk_2024
export PDK_TECH_DIR=/home/user/PDK/pdk_tech_2024
```

## Creating Python Environments

### 1. Setup ML Environment (for HDF5/PyTorch)

```bash
# Create virtual environment
python3 -m venv ml_env

# Activate it
source ml_env/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Setup Orchestration Environment

```bash
# Create another virtual environment for CLI orchestration
python3 -m venv orchestration_env

# Activate it
source orchestration_env/bin/activate

# Note: This environment does NOT need torch or heavy ML deps
# It only needs basic utilities for workflow management
```

## Quick Start

### 1. Check ADS Detection

```bash
source .env
cd parallel_version
python subprocess_cli_parallel.py --help
```

If successful, you should see help text mentioning Linux/Windows support.

### 2. Create a Workspace and Library

```bash
python subprocess_cli_parallel.py create-workspace-lib \
  --workspace-dir ~/my_workspace \
  --library-name MyLib \
  --use-pdk \
  --pdk-dir $PDK_DIR \
  --pdk-tech-dir $PDK_TECH_DIR
```

### 3. Create a Design

```bash
python subprocess_cli_parallel.py create-design-only \
  --workspace-dir ~/my_workspace \
  --library-name MyLib \
  --cell-name design1 \
  --json-file ../parallel_version/config_examples/json_layout/example.json \
  --substrate microstrip_substrate
```

### 4. Run Simulation

```bash
python subprocess_cli_parallel.py run-simulation-only \
  --workspace-dir ~/my_workspace \
  --library-name MyLib \
  --cell-name design1 \
  --export-path ~/results \
  --export-touchstone \
  --export-dataset \
  --export-csv
```

## Troubleshooting

### Issue: "ADS Python environment not found"

**Solution**: Check that environment variables are set correctly.

```bash
# Verify variables are set
echo $ADS_PYTHON
echo $ADS_INSTALL_DIR

# Check if Python executable exists
ls -la $ADS_PYTHON
```

If paths are wrong, update `.env` and source it again:

```bash
source .env
```

### Issue: Permission Denied on Python Scripts

**Solution**: Make scripts executable:

```bash
chmod +x serial_version/*.py
chmod +x parallel_version/*.py
```

### Issue: "ModuleNotFoundError" when running scripts

**Solution**: Make sure you're in the correct Python environment:

```bash
# Check which Python is active
which python3
python3 --version

# Activate the correct virtual environment if needed
source ml_env/bin/activate
```

### Issue: Workspace Creation Fails

**Solution**: Ensure the directory path is writable:

```bash
mkdir -p ~/my_workspace
chmod 755 ~/my_workspace
```

## Path Handling

The code automatically handles path differences between Windows and Linux:

| Component | Windows | Linux |
|-----------|---------|-------|
| Path separator | `\` | `/` |
| ADS Python | `.exe` file | executable without extension |
| ADS install | `C:\Keysight\ADS` | `/opt/Keysight/ADS` |
| Home directory | `C:\Users\username` | `~` or `/home/username` |
| Temp directory | `C:\Temp` | `/tmp` |

## Environment Variable Discovery

The system automatically searches for ADS in these locations (Linux):

1. `$ADS_PYTHON` environment variable
2. `$ADS_INSTALL_DIR/tools/python/python3`
3. `/opt/Keysight/ADS/tools/python/python3`
4. `/opt/keysight/ADS/tools/python/python3`
5. `/usr/local/Keysight/ADS/tools/python/python3`
6. `$HOME/.local/Keysight/ADS/tools/python/python3`

## Next Steps

After setup:

1. Read [QUICKSTART.md](docs/core/QUICKSTART.md) for workflow overview
2. Check [CONFIG_REFERENCE.md](docs/core/CONFIG_REFERENCE.md) for configuration details
3. Review [ARCHITECTURE.md](docs/core/ARCHITECTURE.md) for system design

## Support

For platform-specific issues:

- Check that Python 3 is used (not Python 2)
- Verify all paths use forward slashes on Linux
- Ensure file permissions are correct (755 for executables)
- Use absolute paths to avoid relative path issues

## Differences from Windows

- Use `source .env` instead of PowerShell's environment commands
- Use `/` instead of `\` in paths
- Python executable is `python3` by default
- Home directory is `~` or `$HOME`, not `%USERPROFILE%`
- Temp directory is `/tmp`, not `C:\Temp`
