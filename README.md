# Task 4 – eSim Upgradation (Ubuntu 25.04 Installation Issues)

## Overview

This repository documents the work done as part of **Task 4 – eSim Upgradation** for the *eSim Semester Long Internship Spring 2026* screening.

The objective of this task is to:
- identify problems faced while installing **eSim 2.5** on **Ubuntu 25.04 and above**
- analyze limitations in the current installer
- implement or propose at least one meaningful fix or mitigation

This work focuses on improving the **robustness of the eSim installer (`install-eSim.sh`)** under newer Ubuntu versions and resource-constrained systems.

---

## System Configuration & Constraints

- Host OS: Windows
- RAM: 8 GB
- CPU: 4 cores
- Virtualization Tool: VMware
- Target OS: Ubuntu 25.04 (Virtual Machine)

### Observed Limitation

During repeated attempts to install Ubuntu 25.04 in a virtualized environment, the **entire host system froze**, requiring a forced reboot.  
This occurred during resource-intensive operations and indicates **resource exhaustion under virtualization**.

Because of this limitation, it was not possible to fully execute the eSim installer inside the VM. Therefore, this task combines **observational findings** with **static analysis of the installer script**.

---

## Analysis of `install-eSim.sh`

The `install-eSim.sh` script acts as a dispatcher that:
1. Detects the Ubuntu version
2. Selects a version-specific installer script
3. Executes it

A review of the script revealed the following issues.

---

## Issues Identified

### Issue 1: No System Resource Validation

The installer does not check whether the system has sufficient:
- RAM
- disk space
- CPU capability

As a result, on low-resource systems (especially virtual machines), dependency installation can lead to:
- severe slowdown
- VM unresponsiveness
- full system freeze

This directly impacts first-time users and students using limited hardware.

---

### Issue 2: Immediate Failure on Ubuntu 25.x

The original script explicitly supports Ubuntu versions up to **24.04**.  
Any newer version (such as **Ubuntu 25.04**) results in immediate termination:


This prevents:
- partial compatibility testing
- meaningful diagnostics
- best-effort installation attempts

---

### Issue 3: Heavy Dependencies Without Safeguards

The installer eventually triggers installation of large dependencies (LLVM toolchain, GUI libraries, Python packages) without:
- staging the installation
- checking intermediate resource usage
- providing fallback or skip options

This increases the risk of system instability during installation.

---

## Fixes and Improvements Implemented

### 1. Pre-Installation System Memory Check

A **RAM validation step** was added at the beginning of `install-eSim.sh` to prevent execution on systems that are likely to fail.

```
MIN_RAM_MB=4096
AVAILABLE_RAM_MB=$(free -m | awk '/Mem:/ {print $2}')

if [ "$AVAILABLE_RAM_MB" -lt "$MIN_RAM_MB" ]; then
    echo "Error: Insufficient system memory."
    echo "Detected: ${AVAILABLE_RAM_MB} MB"
    echo "Required: ${MIN_RAM_MB} MB"
    echo "Aborting eSim installation."
    exit 1
fi
```
Benefits:

- Prevents undefined behavior on low-memory systems
- Provides clear diagnostic feedback to users
- Improves overall installer reliability

Fix 2: Graceful Handling of Ubuntu 25.x

Instead of terminating immediately on unsupported Ubuntu versions, the installer now performs a best-effort fallback.
```
*)
    echo "Warning: Ubuntu version $VERSION_ID ($FULL_VERSION) is not officially supported."
    echo "Attempting to proceed using the Ubuntu 24.04 installer script."
    SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
    ;;

```
Benefits:

- Improves usability on newer Ubuntu releases
- Enables partial installation and testing
- Avoids abrupt termination with no fallback path
- This change does not claim full Ubuntu 25.x support, but significantly improves installer behavior and user experience.

Limitations:
- Due to VM instability and host resource limits, the full eSim installation could not be executed on Ubuntu 25.04.
- The Ubuntu 25.x fallback mechanism was not experimentally validated.
- Disk space and CPU validation were not implemented but are recommended.


===
eSim Packaging
====

It contains all the documentation for packaging eSim for distribution.


# Packaging eSim for Distribution:

1. eSim is currently packaged and distributed for Ubuntu OS (Linux) and MS Windows OS.

2. Refer the [documentation](Version_Change.md) for the changes to be done when a new release is to be made.

> Note: These changes have to be made `first` before proceeding with the packaging on either platform.

3. Refer the [documentation](Ubuntu/README.md) to package eSim for Ubuntu OS.

4. Refer the [documentation](Windows/README.md) to package eSim for Windows OS.
