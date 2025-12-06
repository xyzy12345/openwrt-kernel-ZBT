# openwrt-kernel-ZBT

OpenWrt kernel compilation workflow for MT7981 chipset devices.

## Overview

This repository contains GitHub Actions workflow for compiling custom OpenWrt kernel for MT7981-based devices (ZBT routers).

## Features

- Automated kernel compilation using GitHub Actions
- Custom kernel configuration for MT7981 chipset
- Disk space optimization for CI/CD environment
- Manual workflow trigger support

## Workflow

The compilation workflow includes:

1. **Repository Checkout** - Checks out the repository to access kernel configurations
2. **Disk Space Management** - Frees up ~10-20GB by removing unnecessary packages:
   - .NET SDK
   - Android SDKs
   - GHC (Haskell compiler)
   - CodeQL tools
   - APT cache
   - Temporary files
3. **Kernel Compilation** - Compiles kernel version 6.6.73 with custom configuration

## Usage

### Manual Trigger

1. Go to the [Actions tab](../../actions)
2. Select "Compile OpenWrt Kernel for MT7981" workflow
3. Click "Run workflow"
4. Wait for compilation to complete

### Kernel Configuration

Kernel configurations are stored in `kernel/config_path/`:
- `config-6.6.73` - Configuration for kernel 6.6.73
- `config-6.12` - Configuration for kernel 6.12

## Technical Details

- **Kernel Version**: 6.6.73
- **Kernel Sign**: -mt7981-custom
- **Base Repository**: padavanonly/immortalwrt-mt798x-6.6
- **Build Tool**: ophub/amlogic-s9xxx-armbian

## Requirements

- GitHub Actions runner with ubuntu-latest
- Minimum 15GB free disk space after cleanup
- Repository checkout access

## Troubleshooting

### Disk Space Issues

If the workflow fails due to insufficient disk space:
1. The cleanup step should free up 10-20GB automatically
2. Check the "Check disk space after cleanup" step in workflow logs
3. If still insufficient, consider using larger runners or reducing cache size

### Kernel Configuration Issues

Ensure the kernel configuration file matches the kernel version specified in the workflow.