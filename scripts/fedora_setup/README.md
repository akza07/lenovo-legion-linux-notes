# Fedora Setup Script for Legion - akza07

A set of functions to easy the installation and setup of Fedora

### Table of Contents

1. [Installation Options](#Installation-Options)
2. [Setup Scripts](#setup-scripts)
3. [Monitoring GPU Power State](#monitoring-gpu-power-state)

### Installation Options

The following installation options are available:

* `--secureboot`: Enables secure boot during the installation process.
* `--nvidia`: Installs NVIDIA drivers during the installation process.
* `--fix-clock`: Fixes the system clock to match the local hardware clock.
* `--install-pnpm`: Installs pnpm (a package manager) as part of the setup process.
* `--install-apps`: Installs a list of applications (listed in the script) during the installation process.
* `--base-install`: Runs the base installation command, which updates the system and sets up the necessary packages.
* `--gpu`: Monitors GPU power state and driver setup.

### Setup Scripts

The following setup scripts are used to configure various aspects of Fedora:

* **Secure Boot**: Enables secure boot during the installation process.
* **NVIDIA Drivers**: Installs NVIDIA drivers during the installation process.
* **System Clock Fix**: Fixes the system clock to match the local hardware clock.
* **PNPM Installation**: Installs pnpm (a package manager) as part of the setup process.
* **App Install**: Installs a list of applications (listed in the script) during the installation process.

### Monitoring GPU Power State

The `--gpu` option is used to monitor the power state of NVIDIA GPUs. This can be useful for identifying issues with GPU performance or overheating.

**How to Use:**

1. Clone or download this repository.
2. Navigate to the root directory of the repository.
3. Run the script using the desired installation option:
```bash
$ ./fedora-bootstrap.sh --option_name
```
Replace `option_name` with the desired option (e.g., `--nvidia`, `--fix-clock`, etc.).

**Note:** Make sure you have the necessary dependencies installed before running the script.

By following these instructions, you should be able to successfully install and set up Fedora using this script.
