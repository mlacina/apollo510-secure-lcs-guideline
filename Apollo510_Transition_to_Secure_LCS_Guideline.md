# Apollo510 Secure Boot — Transition to Secure Boot and LCS Guideline

*A step-by-step guide to configuring Secure Boot and transitioning to Secure LCS*

**Author:** Mariusz Łacina  
**Document version:** 1.2  
**Date:** November 25, 2026

## Disclaimer and Responsibility

This document is an independent technical guide prepared to assist engineers in implementing secure provisioning and secure boot functionality on Ambiq Apollo510 devices.

The author has made every reasonable effort to ensure the accuracy of the information and to provide a practical, functional solution based on available documentation, engineering experience, and laboratory testing. Nevertheless, the procedures and examples are provided "as is", without any warranty, express or implied, regarding their correctness, completeness, suitability, or reliability.

The user is solely responsible for evaluating, validating, and ensuring the correct operation, security, safety, and regulatory compliance of their own product. This includes verifying the suitability of the described procedures, protecting cryptographic assets, and assessing the consequences of irreversible security configuration and provisioning operations.

The author assumes no responsibility for device failures, loss of data, security vulnerabilities, or other consequences arising from the use or implementation of the information provided in this document. Users are strongly encouraged to consult the latest official Ambiq documentation and perform independent validation before applying these procedures to production devices.

> [!NOTE]
> Commands are presented as single logical lines unless a multiline script or configuration is explicitly shown. Preserve the original syntax, arguments, indentation, and file paths when copying commands into the terminal or J-Link Commander script.

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Python Environment Setup](#2-python-environment-setup)
  - [2.1. Install System Prerequisites](#21-install-system-prerequisites)
  - [2.2. Install and Configure pyenv](#22-install-and-configure-pyenv)
  - [2.3. Install Python 3.10.12](#23-install-python-31012)
  - [2.4. Create the Python Virtual Environment](#24-create-the-python-virtual-environment)
  - [2.5. Install Python Dependencies](#25-install-python-dependencies)
  - [2.6. Verify and Reuse the Environment](#26-verify-and-reuse-the-environment)
- [3. OpenSSL 1.0.2u Installation](#3-openssl-102u-installation)
  - [3.1. Prepare Installation Directories](#31-prepare-installation-directories)
  - [3.2. Download OpenSSL 1.0.2u](#32-download-openssl-102u)
  - [3.3. Configure OpenSSL](#33-configure-openssl)
  - [3.4. Compile and Install OpenSSL](#34-compile-and-install-openssl)
  - [3.5. Verify the OpenSSL Installation](#35-verify-the-openssl-installation)
  - [3.6. Configure the OpenSSL Environment](#36-configure-the-openssl-environment)
- [4. SEGGER J-Link Installation](#4-segger-j-link-installation)
  - [4.1. Download SEGGER J-Link](#41-download-segger-j-link)
  - [4.2. Install SEGGER J-Link](#42-install-segger-j-link)
  - [4.3. Verify the Installation](#43-verify-the-installation)
- [5. AmbiqSuite SDK 5.2.0 Installation](#5-ambiqsuite-sdk-520-installation)
  - [5.1. Download AmbiqSuite SDK](#51-download-ambiqsuite-sdk)
  - [5.2. Extract AmbiqSuite SDK](#52-extract-ambiqsuite-sdk)
  - [5.3. Verify the SDK Installation](#53-verify-the-sdk-installation)
  - [5.4. Create a Provisioning Working Directory](#54-create-a-provisioning-working-directory)
  - [5.5. Verify the Provisioning Working Directory](#55-verify-the-provisioning-working-directory)
- [6. OEM Security Asset Preparation](#6-oem-security-asset-preparation)
  - [6.1. Generate OEM Cryptographic Keys](#61-generate-oem-cryptographic-keys)
  - [6.2. Generate the OEM Root of Trust (HBK1)](#62-generate-the-oem-root-of-trust-hbk1)
  - [6.3. Generate the OEM Key Request](#63-generate-the-oem-key-request)
  - [6.4. Generate the OEM Security Configuration Asset](#64-generate-the-oem-security-configuration-asset)
    - [6.4.1 Configure Boot Override](#641-configure-boot-override)
    - [6.4.2 Configure the Wired Update Interface](#642-configure-the-wired-update-interface)
    - [6.4.3 Generate the OEM Security Configuration Asset Package](#643-generate-the-oem-security-configuration-asset-package)
  - [6.5. Generate Encrypted OEM Asset Packages](#65-generate-encrypted-oem-asset-packages)
  - [6.6. Generate the OEM Provisioning Data Blob](#66-generate-the-oem-provisioning-data-blob)
  - [6.7. Provisioning Assets Summary](#67-provisioning-assets-summary)
- [7. OEM Certificate Chain Generation](#7-oem-certificate-chain-generation)
  - [7.1. Prepare the Main Firmware Image](#71-prepare-the-main-firmware-image)
  - [7.2. Generate the OEM Root Certificate](#72-generate-the-oem-root-certificate)
  - [7.3. Generate the OEM Key Certificate](#73-generate-the-oem-key-certificate)
  - [7.4. Configure the OEM Content Certificate](#74-configure-the-oem-content-certificate)
  - [7.5. Generate the OEM Content Certificate](#75-generate-the-oem-content-certificate)
  - [7.6. Certificate Chain Memory Allocation](#76-certificate-chain-memory-allocation)
  - [7.7. Verify Generated Certificate Files](#77-verify-generated-certificate-files)
  - [7.8. Certificate Generation Summary](#78-certificate-generation-summary)
- [8. Secure OTA Firmware Package Generation](#8-secure-ota-firmware-package-generation)
  - [8.1. Create the OTA Working Directory](#81-create-the-ota-working-directory)
  - [8.2. Create the Key Configuration File](#82-create-the-key-configuration-file)
  - [8.3. Create the Secure Firmware OTA Configuration](#83-create-the-secure-firmware-ota-configuration)
  - [8.4. Generate the Secure OTA Package](#84-generate-the-secure-ota-package)
  - [8.5. Verify the Generated Package](#85-verify-the-generated-package)
  - [8.6. Secure OTA Package Placement](#86-secure-ota-package-placement)
- [9. INFO0 and Secure Boot Metadata Generation](#9-info0-and-secure-boot-metadata-generation)
  - [9.1. Generate the INFO0 Configuration Image](#91-generate-the-info0-configuration-image)
  - [9.2. Generate the Certificate Chain Pointer Table](#92-generate-the-certificate-chain-pointer-table)
  - [9.3. Generate the OTA Descriptor](#93-generate-the-ota-descriptor)
- [10. OEM Provisioning and Transition to Secure LCS](#10-oem-provisioning-and-transition-to-secure-lcs)
  - [10.1. Single-Pass J-Link Provisioning Script – INFO0-MRAM](#101-single-pass-j-link-provisioning-script--info0-mram)
  - [10.2. Single-Pass J-Link Provisioning Script – INFO0-OTP](#102-single-pass-j-link-provisioning-script--info0-otp)
  - [10.3. Register Readback and Verification](#103-register-readback-and-verification)
  - [10.4. Execute OEM Provisioning](#104-execute-oem-provisioning)
  - [10.5. Verify Secure LCS and Secure Boot Configuration Using UART](#105-verify-secure-lcs-and-secure-boot-configuration-using-uart)
    - [10.5.1 Connect the UART Interface](#1051-connect-the-uart-interface)
    - [10.5.2 Prepare the UART Communication](#1052-prepare-the-uart-communication)
    - [10.5.3 Enter Boot Override Mode](#1053-enter-boot-override-mode)
    - [10.5.4 Execute HELLO and STATUS Commands](#1054-execute-hello-and-status-commands)
    - [10.5.5 Verify the Reported Security Configuration](#1055-verify-the-reported-security-configuration)
    - [10.5.6 Retrieve and Record the Device SOC-ID](#1056-retrieve-and-record-the-device-soc-id)
- [11. Secure Debug Certificate Generation and Installation](#11-secure-debug-certificate-generation-and-installation)
  - [11.1. Secure Debug Certificate Structure](#111-secure-debug-certificate-structure)
  - [11.2. Prepare the Certificate Generation Environment](#112-prepare-the-certificate-generation-environment)
  - [11.3. Generate the OEM Debug-Key Certificate](#113-generate-the-oem-debug-key-certificate)
  - [11.4. Generate the OEM Debug Enabler Certificate](#114-generate-the-oem-debug-enabler-certificate)
  - [11.5. Generate the Device-Specific OEM Debug Developer Certificate](#115-generate-the-device-specific-oem-debug-developer-certificate)
  - [11.6. Secure Debug Certificate Installation Through UART](#116-secure-debug-certificate-installation-through-uart)
    - [11.6.1 Create the Secure Debug Wired Download Configuration](#1161-create-the-secure-debug-wired-download-configuration)
    - [11.6.2 Generate the Secure Debug Wired Package](#1162-generate-the-secure-debug-wired-package)
    - [11.6.3 Install and Activate the Secure Debug Certificate Using UART Boot Override](#1163-install-and-activate-the-secure-debug-certificate-using-uart-boot-override)
    - [11.6.4 Verify Debug Access](#1164-verify-debug-access)
- [12. Removing the Secure Debug Certificate](#12-removing-the-secure-debug-certificate)
  - [12.1. Select the Working Directory](#121-select-the-working-directory)
  - [12.2. Generate a Blank Secure Debug Certificate Image](#122-generate-a-blank-secure-debug-certificate-image)
  - [12.3. Prepare the Wired Download Configuration](#123-prepare-the-wired-download-configuration)
  - [12.4. Generate the Wired Download Image](#124-generate-the-wired-download-image)
  - [12.5. Remove the Secure Debug Certificate Using Boot Override UART](#125-remove-the-secure-debug-certificate-using-boot-override-uart)
  - [12.6. Deactivate Secure Debug](#126-deactivate-secure-debug)
  - [12.7. Verify Secure Debug Deactivation](#127-verify-secure-debug-deactivation)
- [13. References](#13-references)
- [14. Appendix A: J-Link Utilities](#14-appendix-a-j-link-utilities)
- [15. Appendix B – UART Communication Troubleshooting](#15-appendix-b--uart-communication-troubleshooting)
  - [15.1. Verify USB-UART Port Availability](#151-verify-usb-uart-port-availability)
  - [15.2. Verify USB Adapter Detection](#152-verify-usb-adapter-detection)
  - [15.3. Verify User Permissions](#153-verify-user-permissions)
  - [15.4. Verify Python Serial Port Access](#154-verify-python-serial-port-access)
  - [15.5. Verify That the Serial Port Is Not in Use](#155-verify-that-the-serial-port-is-not-in-use)
  - [15.6. Verify Physical UART Connectivity](#156-verify-physical-uart-connectivity)
  - [15.7. Verify Boot Override and UART Communication Timing](#157-verify-boot-override-and-uart-communication-timing)
  - [15.8. Troubleshooting Summary](#158-troubleshooting-summary)

---

## 1. Introduction

This document describes the process of configuring Secure Boot and transitioning an Ambiq Apollo510 device from Device Manufacturer (DM) Life Cycle State to Secure Enabled (SE) Life Cycle State.

The procedure uses security provisioning utilities provided with AmbiqSuite SDK 5.2.0. The original utilities are located in the `./tools/apollo510_scripts` directory. For the purposes of this guide, a separate working copy named `./tools/apollo510_scripts_prov` is used to prepare OEM cryptographic assets, certificates, provisioning packages, and secure firmware images.

The Apollo510 security architecture consists of an immutable Secure BootROM (SBR) and an upgradeable Ambiq Secure Bootloader (SBL) stored in internal Magnetoresistive Random-Access Memory (MRAM).

The Secure BootROM establishes the initial hardware Root of Trust and performs security-critical operations, including authentication of the Ambiq Secure Bootloader and processing of OEM provisioning and Secure Debug certificates.

When OEM Secure Boot is enabled, the Ambiq Secure Bootloader authenticates customer firmware using an OEM certificate chain consisting of three certificates: OEM Root Certificate, OEM Key Certificate, and OEM Content Certificate.

Secure Enabled Life Cycle State (SE LCS) imposes significantly stricter security restrictions than the default development environment. Once the device enters this state, standard debugging interfaces, including SWD/J-Link access, are disabled by default.

Secure Boot and Secure LCS are distinct security configurations. A device can operate in Secure LCS without enforcing Secure Boot authentication. Therefore, both the Life Cycle State transition and Secure Boot configuration must be independently verified.

Debug access may be restored selectively through an authenticated, device-specific Secure Debug Certificate, provided that the necessary security configuration and an appropriate certificate installation mechanism are available.

Secure Debug Certificates can be installed after the device has entered SE LCS using a supported wired update interface, subject to the configured security policies.

Secure Boot additionally enforces authentication of executable firmware. If image authentication fails or the required security assets are invalid, the device may refuse to execute the customer application. This behavior is intentional and prevents unauthorized firmware from running.

Before transitioning a device to SE LCS, it is strongly recommended to establish and verify a reliable firmware recovery mechanism.

The Ambiq Secure Bootloader provides a wired firmware update interface that can be configured for this purpose. Unlike an application-based update mechanism, the Secure Bootloader's wired update functionality does not depend on successful execution of the customer application.

This distinction is particularly important when an invalid or corrupted firmware image prevents the application from starting.

The same wired interface may also be used to install an authenticated Secure Debug Certificate, allowing selected debugging capabilities to be enabled according to the applicable security policy.

An application-level OTA mechanism may download and stage a secure firmware update package containing a new application image and its corresponding OEM Content Certificate. Following a device reset, the Ambiq Secure Bootloader authenticates the update and, if valid, installs the new firmware and the associated Content Certificate.

The application cannot bypass Secure Boot authentication or install arbitrary executable firmware. Furthermore, an application-based update mechanism cannot provide recovery if the current application fails to boot. For this reason, an independent Secure Bootloader wired update interface is strongly recommended.

The security configuration, firmware recovery strategy, certificate management, and Secure Debug access policy should therefore be carefully considered and validated before the device is transitioned to SE LCS.

### MRAM Memory Organization

Table 1 presents the 4 MiB MRAM memory map adopted for the Apollo510 secure provisioning procedure described in this guide. The first 64 KiB of MRAM is reserved for the Ambiq Secure Bootloader (SBL). The remaining memory is divided into separate regions for the main application, OTA firmware update storage, and security-related certificates. The application and OTA image storage regions are allocated 1968 KiB each, providing space to stage a new firmware image without overwriting the currently installed application. A dedicated 16 KiB block is reserved for the OTA Descriptor, which provides the information required by the Secure Bootloader to locate and process the staged update package. The complete OTA package, including its metadata and cryptographic overhead, must fit within the allocated staging region. The uppermost 80 KiB of MRAM is reserved for security-related data. This area contains the Certificate Chain Pointer Array, OEM Root Certificate, OEM Key Certificate, OEM Content Certificate, and a reserved location for the Secure Debug Certificate. Each of these five regions occupies a separate 16 KiB memory block, corresponding to the NVM protection granularity supported by Apollo510. This arrangement allows individual blocks to be protected independently, according to the selected security policy, without unnecessarily restricting access to adjacent certificate regions. The OTA Descriptor is also allocated a separate 16 KiB protection-aligned block, allowing its access permissions to be configured independently of the application and OTA image storage regions. The memory addresses shown in Table 1 are used consistently throughout this guide. Any modifications to the proposed layout require corresponding updates to the INFO0 configuration, certificate pointers, OTA descriptor, firmware image configuration, and provisioning scripts.

**Table 1. Apollo510 MRAM Memory Map**

| Region | Start | Stop | Size (KiB) |
| --- | --- | --- | --- |
| Secure Boot Loader (SBL) | 0x00400000 | 0x0040FFFF | 64 KiB |
| Application | 0x00410000 | 0x005FBFFF | 1968 KiB |
| OTA Descriptor | 0x005FC000 | 0x005FFFFF | 16 KiB |
| OTA Image | 0x00600000 | 0x007EBFFF | 1968 KiB |
| Certificate Chain Pointers | 0x007EC000 | 0x007EFFFF | 16 KiB |
| OEM Root Certificate | 0x007F0000 | 0x007F3FFF | 16 KiB |
| OEM Key Certificate | 0x007F4000 | 0x007F7FFF | 16 KiB |
| OEM Content Certificate | 0x007F8000 | 0x007FBFFF | 16 KiB |
| Secure Debug Reserved | 0x007FC000 | 0x007FFFFF | 16 KiB |

## 2. Python Environment Setup

The Apollo510 secure provisioning utilities are executed using Python on Ubuntu Linux. To provide a consistent execution environment and avoid conflicts with the system Python installation, this guide uses **Python 3.10.12** managed by `pyenv` and a dedicated Python virtual environment.

The following procedure assumes Ubuntu 24.04 LTS and the Bash shell.

### 2.1. Install System Prerequisites

Update the Ubuntu package index:

```bash
sudo apt update
```

Install the packages required to build Python and run the provisioning utilities:

```bash
sudo apt install -y build-essential git curl wget ca-certificates make perl pkg-config zlib1g-dev libssl-dev libbz2-dev libreadline-dev libsqlite3-dev libffi-dev liblzma-dev libncurses-dev libxml2-dev libxmlsec1-dev unzip xxd
```

### 2.2. Install and Configure pyenv

The `pyenv` utility allows multiple Python versions to coexist independently of the operating system's default Python installation.

Install pyenv:

```bash
git clone https://github.com/pyenv/pyenv.git "$HOME/.pyenv"
```

Open the Bash configuration file:

```bash
nano ~/.bashrc
```

Add the following three lines at the end of the file:

```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - bash)"
```

Save the file by pressing Ctrl+O, followed by Enter, and exit the editor using Ctrl+X. These settings make `pyenv` available automatically whenever a new Bash terminal is opened.

Reload the Bash configuration in the current terminal:

```bash
source ~/.bashrc
```

Execute:

```bash
pyenv --version
```

The command should display the installed `pyenv` version.

The environment is now ready for installing Python 3.10.12, as described in the next section.

### 2.3. Install Python 3.10.12

Install the required Python version:

```bash
pyenv install 3.10.12
```

Verify the installation:

```bash
"$HOME/.pyenv/versions/3.10.12/bin/python3" --version
```

Expected result:

```text
Python 3.10.12
```

This installation is separate from the Python interpreter provided by Ubuntu.

### 2.4. Create the Python Virtual Environment

Create a dedicated virtual environment named apollo510-env:

```bash
"$HOME/.pyenv/versions/3.10.12/bin/python3" -m venv "$HOME/apollo510-env"
```

Activate the environment:

```bash
source "$HOME/apollo510-env/bin/activate"
```

After activation, the terminal prompt should include the environment name:

```text
(apollo510-env) user@ubuntu:~$
```

To optionally leave the virtual environment, execute:

```bash
deactivate
```

Verify that the active interpreter is Python 3.10.12:

```bash
python3 --version
```

The expected result:

```text
Python 3.10.12
```

### 2.5. Install Python Dependencies

Upgrade the Python package management utilities:

```bash
python3 -m pip install --upgrade pip setuptools wheel
```

Install the Python packages required by the Apollo510 provisioning tools:

```bash
python3 -m pip install pyserial==3.5 pycryptodome==3.23.0
```

Install Python cryptography library:

```bash
python3 -m pip install cryptography
```

Check that the installed package dependencies are consistent:

```bash
python3 -m pip check
```

### 2.6. Verify and Reuse the Environment

Verify that the Python interpreter and required cryptographic and serial communication modules are available:

```bash
python3 -c "import sys, serial, cryptography; from Crypto.Cipher import AES; print(sys.version); print('cryptography version:', cryptography.__version__); print('Apollo510 Python environment: OK')"
```

Expected confirmation:

```text
Apollo510 Python environment: OK
```

The virtual environment remains installed after closing the terminal. It does not need to be recreated for subsequent provisioning sessions.

Activate the environment before the next steps:

```bash
source "$HOME/apollo510-env/bin/activate"
```

## 3. OpenSSL 1.0.2u Installation

The OEM provisioning utilities included in AmbiqSuite SDK 5.2.0 require the legacy OpenSSL 1.0.2 library for certain cryptographic operations.

Ubuntu 24.04 provides a newer version of OpenSSL, which does not include the legacy libcrypto.so.1.0.0 shared library required by these utilities.

To ensure compatibility, OpenSSL 1.0.2u is installed separately in the user's home directory, without modifying or replacing the system OpenSSL installation.

The following procedure downloads, compiles, and installs OpenSSL 1.0.2u from source.

### 3.1. Prepare Installation Directories

Create a directory for the OpenSSL source files and a separate directory for the compiled library:

```bash
mkdir -p "$HOME/ambiq-legacy/src" "$HOME/ambiq-legacy/openssl-install"
```

The following directories will be used:

| Directory | Purpose |
| --- | --- |
| ~/ambiq-legacy/src | Downloaded archive and extracted source files |
| ~/ambiq-legacy/openssl-install | Compiled OpenSSL binaries and shared libraries |

### 3.2. Download OpenSSL 1.0.2u

Change to the source directory:

```bash
cd "$HOME/ambiq-legacy/src"
```

Download the OpenSSL 1.0.2u source archive:

```bash
curl -fL -o openssl-1.0.2u.tar.gz https://www.openssl.org/source/old/1.0.2/openssl-1.0.2u.tar.gz
```

Extract the downloaded archive:

```bash
tar -xzf openssl-1.0.2u.tar.gz
```

The source files will be extracted into the openssl-1.0.2u directory.

### 3.3. Configure OpenSSL

Change to the extracted source directory:

```bash
cd "$HOME/ambiq-legacy/src/openssl-1.0.2u"
```

Configure the build:

```bash
./config --prefix="$HOME/ambiq-legacy/openssl-install" --openssldir="$HOME/ambiq-legacy/openssl-install" shared no-ssl2 no-ssl3
```

The configuration parameters have the following purposes:

| Parameter | Description |
| --- | --- |
| --prefix | Specifies the installation directory |
| --openssldir | Specifies the OpenSSL configuration directory |
| shared | Enables building shared libraries, including libcrypto.so.1.0.0 |
| no-ssl2 | Disables the obsolete SSLv2 protocol |
| no-ssl3 | Disables the obsolete SSLv3 protocol |

The installation remains independent of the OpenSSL version provided by Ubuntu.

### 3.4. Compile and Install OpenSSL

Generate the required build dependencies:

```bash
make depend
```

Compile OpenSSL using the available processor cores:

```bash
make -j"$(nproc)"
```

The compilation process may display warnings associated with this legacy OpenSSL version.

After compilation, install the compiled binaries and libraries:

```bash
make install_sw
```

The installation does not require sudo, because all files are installed within the user's home directory.

### 3.5. Verify the OpenSSL Installation

Check the installed version:

```bash
"$HOME/ambiq-legacy/openssl-install/bin/openssl" version
```

Expected result:

```text
OpenSSL 1.0.2u 20 Dec 2019
```

This confirms that the required legacy OpenSSL executable has been installed successfully.

### 3.6. Configure the OpenSSL Environment

The Ambiq OEM provisioning utilities require access to the legacy OpenSSL executable and its shared libraries.

Add the OpenSSL installation directory to the current terminal's executable search path:

```bash
export PATH="$HOME/ambiq-legacy/openssl-install/bin:$PATH"
```

Configure the shared library search path:

```bash
export LD_LIBRARY_PATH="$HOME/ambiq-legacy/openssl-install/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

The first command makes the legacy OpenSSL executable available through PATH.

The second command allows applications and Python modules that depend on the legacy OpenSSL shared libraries to locate them during execution.

In particular, it provides access to libcrypto.so.1.0.0, which is required by the OEM certificate generation utilities.

**These settings apply only to the current terminal session**. They do not permanently modify the Ubuntu system configuration.

The same environment variables must be configured when opening a new terminal session for OEM provisioning.

Before executing the Ambiq provisioning utilities, activate the Python virtual environment:

```bash
source "$HOME/apollo510-env/bin/activate"
```

## 4. SEGGER J-Link Installation

SEGGER J-Link is used to communicate with Apollo510 devices through the Serial Wire Debug (SWD) interface.

This guide uses SEGGER J-Link Software and Documentation Pack V9.92.0, installed on Ubuntu 24.04 LTS.

The J-Link Commander utility (JLinkExe) is used later in the secure provisioning procedure to execute the programming script.

### 4.1. Download SEGGER J-Link

Open the official SEGGER download page:

<https://www.segger.com/downloads/jlink/>

Locate the J-Link Software and Documentation Pack and select **version V9.92.0** or newer for Linux.

Download the 64-bit DEB Installer for x86-64 Ubuntu.

The installation package should be saved in the user's Downloads directory:

```text
~/Downloads/JLink_Linux_V992_x86_64.deb
```

### 4.2. Install SEGGER J-Link

Open an Ubuntu terminal and install the downloaded package:

```bash
sudo apt install -y "$HOME/Downloads/JLink_Linux_V992_x86_64.deb"
```

Ubuntu installs the J-Link software and its required package dependencies.

The installation includes J-Link Commander (JLinkExe), which will be used to execute the provisioning commands.

### 4.3. Verify the Installation

Verify that the J-Link Commander executable is available:

```bash
command -v JLinkExe
```

The command should display the path to the installed executable.

Check the installed J-Link package and version:

```bash
dpkg-query -W -f='${Package} ${Version}\n' jlink
```

Confirm that the installed package corresponds to J-Link V9.92.0.

The SEGGER J-Link software is now installed and ready for use with the Apollo510 provisioning procedure.

## 5. AmbiqSuite SDK 5.2.0 Installation

AmbiqSuite SDK provides the software libraries, examples, and utilities required to develop applications and configure secure provisioning for Apollo510 devices.

This guide uses AmbiqSuite SDK 5.2.0.

The SDK includes Python utilities for generating OEM cryptographic assets, certificate chains, provisioning packages, and secure firmware update images.

A separate working copy of the provisioning tools will be created to keep the original SDK files unchanged.

### 5.1. Download AmbiqSuite SDK

Obtain the AmbiqSuite SDK 5.2.0 distribution from Ambiq:

<https://contentportal.ambiq.com/documents/20123/387752/AmbiqSuite_5.2.0.zip>

Download the SDK archive and save it in the Ubuntu Downloads directory:

```text
~/Downloads/AmbiqSuite_5.2.0.zip
```

The following procedure assumes that the unzip utility has already been installed as part of the Ubuntu system prerequisites.

### 5.2. Extract AmbiqSuite SDK

Open an Ubuntu terminal.

Extract the SDK archive into the user's home directory:

```bash
unzip -q "$HOME/Downloads/AmbiqSuite_5.2.0.zip" -d "$HOME"
```

The SDK root directory must be located at:

```text
~/AmbiqSuite_5.2.0
```

If the archive uses a different top-level directory name, rename the extracted SDK directory accordingly, keeping its internal structure unchanged.

### 5.3. Verify the SDK Installation

Verify that the Apollo510 provisioning tools are available:

```bash
ls -ld "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts"
```

The command should display the directory containing the Apollo510 security provisioning utilities.

### 5.4. Create a Provisioning Working Directory

To preserve the original SDK files, create a separate working copy of the provisioning scripts:

```bash
cp -a "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts" "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov"
```

| Directory | Purpose |
| --- | --- |
| tools/apollo510_scripts | Original SDK provisioning utilities |
| tools/apollo510_scripts_prov | Working copy used throughout this guide |

The working copy will contain the configuration files, Python scripts, and generated binary files used during OEM provisioning.

### 5.5. Verify the Provisioning Working Directory

Change to the provisioning working directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov"
```

Verify that the main provisioning utilities and directories are present:

```bash
ls -d oem_tools_pkg create_info0.py create_cust_image_blob.py
```

The command should display all entries without errors.

All subsequent provisioning commands in this guide use

```text
~/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov
```

as the default working directory, unless another directory is explicitly specified. The AmbiqSuite SDK installation and provisioning workspace preparation are now complete.

## 6. OEM Security Asset Preparation

OEM provisioning is the process of preparing the cryptographic keys, Root of Trust, security configuration, and encrypted provisioning data required to transition an Apollo510 device from Device Manufacturer (DM) LCS to Secure Enabled (SE) LCS.

AmbiqSuite SDK 5.2.0 provides Python utilities for generating the required security assets and combining them into a provisioning data blob.

This procedure uses the ICV-encrypted provisioning method, in which OEM assets are encrypted using key material associated with a response package supplied by Ambiq.

The Python environment, OpenSSL 1.0.2u libraries, and provisioning working directory have already been prepared in the preceding chapters.

> [!NOTE]
**NOTE — OEM Key Selection**
>
The AmbiqSuite SDK includes pre-generated OEM keys that can be used for development and testing.
>
**Using SDK-provided keys**: Skip the key-generation procedure and continue with **Section 6.4**, Generate the OEM
>
Security Configuration Asset.
>
**Using newly generated OEM keys**: Obtain a new, matching `icv_response_pkg.bin` from Ambiq before proceeding
with encrypted OEM provisioning. An ICV response package generated for a different key set cannot be reused.

> [!CAUTION]
**WARNING — Production Security**
>
**The pre-generated OEM keys provided with the AmbiqSuite SDK are intended for development and testing only**.
>
These keys must not be used in production devices because they are distributed with the SDK and cannot be considered secret. 
For production, generate your own OEM keys and obtain the corresponding ICV response package from Ambiq.

### 6.1. Generate OEM Cryptographic Keys

Apollo510 uses asymmetric RSA keys for certificate-based authentication and symmetric AES keys for cryptographic operations and OEM asset provisioning.

AmbiqSuite provides the OEM Key Generation Utility for generating these assets. The utility uses the following configuration file:

```text
./oem_tools_pkg/am_oem_key_gen_util/oemKeyGenConfig.cfg
```

The configuration defines the key generation parameters and associated output files.

**Note**: If an existing OEM key set is used together with its matching Ambiq ICV response, do not regenerate the keys. A newly generated key set requires a corresponding new ICV response from Ambiq.

To generate a new OEM key set, change to the key generation directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/oem_tools_pkg/am_oem_key_gen_util"
```

Execute:

```bash
python3 am_oem_keys_gen_util.py ./oemKeyGenConfig.cfg
```

The utility generates the asymmetric and symmetric keys required for OEM provisioning, OEM certificate generation, and Secure Debug certificate generation.

The generated key files are stored in the corresponding RSA and AES output directories.

The OTP key-bank contents must be prepared separately.

### 6.2. Generate the OEM Root of Trust (HBK1)

The Hash of Boot Key 1 (HBK1) establishes the OEM Root of Trust used to authenticate the OEM certificate chain.

HBK1 is derived from the OEM Root Certificate public key using the truncated SHA-256 hash format supported by Apollo510.

Change to the HBK generation directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/oem_tools_pkg/cert_utils/am_hbk_gen"
```

Execute:

```bash
python3 am_hbk_gen_util.py -key ../../am_oem_key_gen_util/oemRSAKeys/oemRootCertPublicKey.pem -endian L -hash_format SHA256_TRUNC
```

The generated files are stored in:

```text
./oem_tools_pkg/cert_utils/am_hbk_gen/hbk_gen_util_outPut
```

The binary file containing HBK1 is:

```text
hbkPubKeyHash_bin.bin
```

This file is subsequently referenced by the OEM provisioning data generator.

The same OEM Root key must be used when generating the OEM Root Certificate.

### 6.3. Generate the OEM Key Request

The ICV-encrypted provisioning method requires an ICV response package generated by Ambiq.

The OEM Key Request Utility creates a request containing the public key information required by Ambiq to prepare the corresponding encrypted response.

The utility uses the configuration file:

```text
./oem_tools_pkg/oem_asset_prov_utils/oem_key_request/am_config/am_dmpu_oem_key_request.cfg
```

Change to the OEM Key Request directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/oem_tools_pkg/oem_asset_prov_utils/oem_key_request"
```

Execute:

```bash
python3 am_oem_key_request_util.py ./am_config/am_dmpu_oem_key_request.cfg
```

The generated request package is:

```text
oem_request_pkg.bin
```

Submit this file to Ambiq through the designated provisioning process.

Ambiq processes the request and returns:

```text
./oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/inputData/icv_response_pkg.bin
```

The request and response form a matched pair. The ICV response must correspond to the OEM key material used for provisioning.

**Note**: If an existing matching OEM request and ICV response pair is available, this step may be skipped. The corresponding files and cryptographic keys must remain unchanged.

### 6.4. Generate the OEM Security Configuration Asset

The OEM Asset Generation Utility generates a binary file containing the security configuration that will be programmed into the device during OEM provisioning.

The configuration file is:

```text
./oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/am_config/oem_asset_gen_sec.cfg
```

It defines several important device security settings, including:

-  MRAM write and copy protection.
-  Secure Boot and firmware update policies.
-  Wired update interface configuration.
-  Boot override configuration.
-  Debug access restrictions.
-  Customer OTP access permissions.

The MRAM protection configuration should be consistent with the memory map presented in Table 1.

Apollo510 supports MRAM protection with 16 KiB granularity. The security-related regions at the end of MRAM have therefore been allocated separate 16 KiB blocks to allow independent protection.

For applications using a Customer Secondary Bootloader (CSBL), the PLONEXIT configuration must also be considered. Its value determines the protection-lock behavior during the handoff from the Ambiq Secure Bootloader.

#### 6.4.1 Configure Boot Override

The Boot Override mechanism allows the Ambiq Secure Bootloader to enter wired update mode independently of the installed customer application.

This functionality is particularly important after the device transitions to Secure Enabled LCS, when unrestricted SWD access is disabled.

In this procedure, Boot Override is configured to use GPIO93 with active-low polarity.

Change to the OEM Asset Package directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/oem_tools_pkg/oem_asset_prov_utils/oem_asset_package"
```

Open the OEM security configuration file:

```bash
nano ./am_config/oem_asset_gen_sec.cfg
```

Locate the OTP_BOOT_OVERRIDE parameters and configure them as follows:

```ini
OTP_BOOT_OVERRIDE.ENABLE = 0x1
OTP_BOOT_OVERRIDE.POL = 0x0
OTP_BOOT_OVERRIDE.GPIO = 0x5D
```

The configuration parameters are described below.

| Parameter | Value | Description |
| --- | --- | --- |
| OTP_BOOT_OVERRIDE.ENABLE | 0x1 | Enable Boot Override |
| OTP_BOOT_OVERRIDE.POL | 0x0 | Active-low polarity |
| OTP_BOOT_OVERRIDE.GPIO | 0x5D | Select GPIO93 |

The OEM Asset Generation Utility combines these parameters into a single 32-bit configuration word.

| Field | Bits | Value |
| --- | --- | --- |
| GPIO | [7:0] | 0x5D |
| POL | [8] | 0 |
| ENABLE | [9] | 1 |

The resulting OTP_BOOT_OVERRIDE configuration word is:

```text
0x0000025D
```

GPIO93 must remain at logic HIGH during normal device startup and be driven LOW during reset when Boot Override is requested.

Unless explicitly required by the selected security policy, leave the remaining parameters in oem_asset_gen_sec.cfg unchanged.

Save the configuration file.

#### 6.4.2 Configure the Wired Update Interface

The wired update interface is used to communicate with the Ambiq Secure Bootloader for device status retrieval, firmware updates, and Secure Debug Certificate installation and removal.

This procedure uses UART0 as the wired update interface.

Locate the OTP_WIRED_CONFIG parameters in the same configuration file and configure them as follows:

```ini
OTP_WIRED_CONFIG.UART = 0x1
OTP_WIRED_CONFIG.SPI = 0x0
OTP_WIRED_CONFIG.I2C = 0x0
OTP_WIRED_CONFIG.SLAVEINTPIN = 0x0
OTP_WIRED_CONFIG.I2CADDR = 0x0
OTP_WIRED_CONFIG.UARTMODULE = 0x0
```

| Parameter | Value | Description |
| --- | --- | --- |
| OTP_WIRED_CONFIG.UART | 0x1 | Enable UART wired update |
| OTP_WIRED_CONFIG.SPI | 0x0 | Disable SPI wired update |
| OTP_WIRED_CONFIG.I2C | 0x0 | Disable I2C wired update |
| OTP_WIRED_CONFIG.SLAVEINTPIN | 0x0 | Not used in this configuration |
| OTP_WIRED_CONFIG.I2CADDR | 0x0 | Not used in this configuration |
| OTP_WIRED_CONFIG.UARTMODULE | 0x0 | Select UART0 |

The UART GPIO assignments, baud rate, pin configuration, and wired update timeout are configured separately through the INFO0 configuration image described in Section 9.1.

Unless explicitly required by the selected security policy, leave the remaining parameters in `oem_asset_gen_sec.cfg` unchanged.

Save the configuration file.

#### 6.4.3 Generate the OEM Security Configuration Asset Package

Review the OEM security configuration before generating the asset.

Remain in the OEM Asset Package directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/oem_tools_pkg/oem_asset_prov_utils/oem_asset_package"
```

Execute:

```bash
python3 oem_asset_gen_util.py ./am_config/oem_asset_gen_sec.cfg
```

The generated file is:

```text
oem_asset_test_data.bin
```

This file contains the OEM security configuration, including the Boot Override and wired update interface settings, and is used as an input to the encrypted asset packaging process described in Section 6.5.

**IMPORTANT — Recovery Interface**

Before transitioning the device to Secure Enabled LCS, ensure that GPIO93 and the selected UART0 pins are physically accessible on the target hardware.

Incorrect Boot Override or wired interface configuration can prevent communication with the Secure Bootloader after the lifecycle transition.

Successful generation of the OEM Security Configuration Asset confirms preparation of the configuration data but does not confirm that the corresponding settings have been programmed into the target device. Boot Override and UART communication must be verified after OEM provisioning.

### 6.5. Generate Encrypted OEM Asset Packages

The OEM Asset Packaging Utility encrypts the security assets required for OEM provisioning.

The ICV-encrypted provisioning procedure uses the corresponding Ambiq response package and the OEM cryptographic assets prepared in the previous sections.

The configuration files specify the required input files, encryption parameters, and output filenames.

Three encrypted asset packages are generated.

**Step 1 — Encrypt the OEM Security Asset**

Configuration file:

```text
./am_config/oem_asset_enc.cfg
```

Execute:

```bash
python3 am_dmpu_oem_asset_pkg_util.py ./am_config/oem_asset_enc.cfg
```

Output:

```text
oem_asset_pkg.bin
```

**Step 2 — Encrypt the KCE Asset**

Configuration file:

```text
./am_config/am_asset_oem_ce.cfg
```

Execute:

```bash
python3 am_dmpu_oem_asset_pkg_util.py ./am_config/am_asset_oem_ce.cfg
```

Output:

```text
oem_prov_asset_kce_pkg.bin
```

**Step 3 — Encrypt the KCP Asset**

Configuration file:

```text
./am_config/am_asset_oem_cp.cfg
```

Execute:

```bash
python3 am_dmpu_oem_asset_pkg_util.py ./am_config/am_asset_oem_cp.cfg
```

Output:

```text
oem_prov_asset_kcp_pkg.bin
```

All three commands are executed from the same OEM Asset Package directory.

The resulting encrypted files are used by the OEM Provisioning Data Generation Utility.

### 6.6. Generate the OEM Provisioning Data Blob

The OEM Provisioning Data Generation Utility combines the encrypted OEM security assets into a single provisioning package.

The resulting package contains the information required by the Ambiq OEM Provisioning Tool (OPT) to initialize the device's security configuration.

The utility uses the following configuration file:

```text
./am_config/am_dmpu_data_gen_sec.cfg
```

The configuration specifies:

-  The OEM Root of Trust (HBK1).
-  The encrypted KCP and KCE assets.
-  The encrypted OEM security configuration.
-  Initial software version parameters.
-  Debug Control Unit (DCU) settings.
-  Additional provisioning configuration fields.
-  Secure boot enabled

Review the configuration and confirm that the referenced files correspond to the OEM key set and security configuration selected for the device.

Remain in the OEM Asset Package directory.

Execute:

```bash
python3 am_dmpu_prov_data_gen_util.py ./am_config/am_dmpu_data_gen_sec.cfg
```

The generated output file is:

```text
dmpu_prov_data_blob_sec.bin
```

This file contains the final OEM provisioning data.

During device provisioning, the blob will be loaded into SRAM and processed by the Ambiq OEM Provisioning Tool.

### 6.7. Provisioning Assets Summary

The following files are generated or prepared during OEM security asset provisioning:

| File | Description |
| --- | --- |
| hbkPubKeyHash_bin.bin | OEM Root of Trust (HBK1) |
| oem_request_pkg.bin | OEM key request submitted to Ambiq |
| icv_response_pkg.bin | Corresponding response received from Ambiq |
| oem_asset_test_data.bin | OEM security configuration |
| oem_asset_pkg.bin | Encrypted OEM security asset |
| oem_prov_asset_kce_pkg.bin | Encrypted KCE asset |
| oem_prov_asset_kcp_pkg.bin | Encrypted KCP asset |
| dmpu_prov_data_blob_sec.bin | Final OEM provisioning data |

The final provisioning data blob is used together with the Ambiq-signed OEM Provisioning Tool:

```text
./oem_tools_pkg/oem_prov_tool/opt_image_pkg.bin
```

During device provisioning, the OPT image and the provisioning data blob are loaded into SRAM at addresses `0x20030000` and `0x20037000`, respectively.

The next chapter describes the generation of the OEM certificate chain required to authenticate firmware during Secure Boot.

## 7. OEM Certificate Chain Generation

The Apollo510 Secure Boot mechanism authenticates customer firmware using a chain of three OEM certificates:

1. **OEM Root Certificate** — establishes trust in the OEM certificate hierarchy through the Root of Trust (HBK1).
2. **OEM Key Certificate** — authenticates the public key used by the OEM Content Certificate.
3. **OEM Content Certificate** — authenticates the application firmware image and defines its associated security attributes.

The certificate chain must be generated using the same OEM RSA keys selected for provisioning in Chapter 6.

### 7.1. Prepare the Main Firmware Image

The OEM Content Certificate contains information about the application firmware, including its location and size. Therefore, the firmware image must be available before the Content Certificate is generated.

For this procedure, the application firmware is named:

```text
main_firmware_image.bin
```

Place the compiled application binary in the following directory:

```text
./oem_tools_pkg/cert_utils/cert_gen_utils/inputData/
```

The application must be linked to execute from MRAM at:

```text
0x00410000
```

According to the memory map defined in Table 1, the application region extends from `0x00410000` to `0x005FBFFF`, providing 1968 KiB of available space.

Verify that the application binary exists and does not exceed the allocated region.

The firmware image is stored unencrypted in its execution region in MRAM. Encryption of a transport or update package, when required, is handled separately during Secure OTA package generation.

**Important**: Whenever the application firmware changes, a corresponding new OEM Content Certificate must be generated.

### 7.2. Generate the OEM Root Certificate

The OEM Root Certificate establishes the first certificate in the OEM authentication chain.

It is associated with the OEM Root of Trust (HBK1) generated in Chapter 6.

The certificate generation utility uses the configuration file:

```text
./am_config/am_oem_root_cert_hbk1.cfg
```

Change to the certificate generation directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/oem_tools_pkg/cert_utils/cert_gen_utils"
```

Generate the OEM Root Certificate:

```bash
python3 am_cert_key_util.py ./am_config/am_oem_root_cert_hbk1.cfg
```

The generated files are:

| File | Description |
| --- | --- |
| am_cert_key_util_output/oem_root_cert_hbk1.bin | Binary OEM Root Certificate |
| am_cert_key_util_output/oem_root_cert_hbk1_Cert.txt | Text representation of the certificate |

The binary certificate will subsequently be installed in the designated MRAM certificate region.

### 7.3. Generate the OEM Key Certificate

The OEM Key Certificate authenticates the public key used for signing the OEM Content Certificate.

It is generated using the same OEM RSA key hierarchy and the following configuration file:

```text
./am_config/am_oem_key_cert.cfg
```

Remain in the certificate generation directory.

Execute:

```bash
python3 am_cert_key_util.py ./am_config/am_oem_key_cert.cfg
```

The generated files are:

| File | Description |
| --- | --- |
| am_cert_key_util_output/oem_key_cert.bin | Binary OEM Key Certificate |
| am_cert_key_util_output/oem_key_cert_Cert.txt | Text representation of the certificate |

The OEM Key Certificate forms the second level of the authentication chain.

### 7.4. Configure the OEM Content Certificate

The OEM Content Certificate authenticates the application image and specifies its location, maximum size, and protection attributes.

The generator uses the following configuration file:

```text
./am_config/am_oem_cnt_cert.cfg
```

The application image information is provided through:

```text
./inputData/images_table.tbl
```

Review the image table and configure the entry corresponding to `main_firmware_image.bin`.

The following parameters must be consistent with the selected application image and MRAM memory map:

| Parameter | Value / Requirement |
| --- | --- |
| ImageName | ./inputData/main_firmware_image.bin |
| RAMloadAdd | 0x00410000 |
| flashStoreAdd | 0xFFFFFFFF |
| Maxsize | Not greater than 0x001EC000 and not smaller than the application image |
| Enc-Scheme | 0 — plaintext image in MRAM |
| WriteProtect | According to the selected firmware protection policy |
| CopyProtect | According to the selected firmware protection policy |
| ExtendedFormat | 0 — unsupported extended format disabled |

For this procedure, the application is executed directly from its installed MRAM location.

If image write protection is enabled, the Secure Bootloader applies protection at 16 KiB granularity.

If copy protection is enabled, the application must be built accordingly. In particular, literal data in protected executable regions may cause execution failures.

The configuration should preserve the selected firmware update strategy, including the ability to install authenticated firmware updates.

### 7.5. Generate the OEM Content Certificate

After preparing the application binary and configuring the image table, generate the OEM Content Certificate.

Remain in the certificate generation directory.

Execute:

```bash
python3 am_cert_content_util.py ./am_config/am_oem_cnt_cert.cfg
```

The generated files are:

| File | Description |
| --- | --- |
| am_cert_content_util_output/content_cert.bin | Binary OEM Content Certificate |
| am_cert_content_util_output/content_cert_Cert.txt | Text representation of the certificate |

The generated Content Certificate is associated with the application image specified in `images_table.tbl`.

The binary Content Certificate will also be included in the Secure OTA package prepared in a subsequent chapter.

### 7.6. Certificate Chain Memory Allocation

The three certificates will be installed in dedicated MRAM regions, according to the memory map defined in Table 1.

Each certificate occupies an independently allocated 16 KiB MRAM protection region.

| Certificate | Start Address | End Address |
| --- | --- | --- |
| OEM Root Certificate | 0x007F0000 | 0x007F3FFF |
| OEM Key Certificate | 0x007F4000 | 0x007F7FFF |
| OEM Content Certificate | 0x007F8000 | 0x007FBFFF |

The certificate pointer table is located at:

```text
0x007EC000
```

It contains three consecutive 32-bit addresses, stored in little-endian format:

| Offset | Certificate | Address |
| --- | --- | --- |
| 0x00 | OEM Root Certificate | 0x007F0000 |
| 0x04 | OEM Key Certificate | 0x007F4000 |
| 0x08 | OEM Content Certificate | 0x007F8000 |

The certificate chain pointer in INFO0 will reference this table.

The remaining MRAM region starting at 0x007FC000 is reserved for the Secure Debug Certificate.

The certificate pointer table, INFO0 image, and MRAM installation procedure will be prepared in the subsequent provisioning chapters.

### 7.7. Verify Generated Certificate Files

Verify that all three binary certificate files have been generated:

```bash
ls -l ./am_cert_key_util_output/oem_root_cert_hbk1.bin ./am_cert_key_util_output/oem_key_cert.bin ./am_cert_content_util_output/content_cert.bin
```

Each certificate must fit within its allocated 16 KiB MRAM region.

Check the certificate file sizes:

```bash
wc -c ./am_cert_key_util_output/oem_root_cert_hbk1.bin ./am_cert_key_util_output/oem_key_cert.bin ./am_cert_content_util_output/content_cert.bin
```

Each generated certificate must be nonempty and no larger than 16384 bytes.

### 7.8. Certificate Generation Summary

The following binary files are required for Secure Boot provisioning:

| File | Target MRAM Address |
| --- | --- |
| oem_root_cert_hbk1.bin | 0x007F0000 |
| oem_key_cert.bin | 0x007F4000 |
| content_cert.bin | 0x007F8000 |

The three certificates establish the OEM authentication chain required by the Apollo510 Secure Bootloader.

Their corresponding addresses will be stored in the certificate pointer table, and the table location will be provided through INFO0.

The next chapter describes the preparation of the Secure OTA firmware package.

## 8. Secure OTA Firmware Package Generation

The Apollo510 Secure Bootloader (SBL) supports authenticated and encrypted firmware update packages.

For this procedure, the firmware update package will contain the application image and its corresponding OEM Content Certificate generated in Chapter 7.

The package is prepared using the AmbiqSuite SDK 5.2.0 `create_cust_image_blob.py` utility.

The resulting binary will be staged in MRAM and subsequently processed by the Secure Bootloader.

### 8.1. Create the OTA Working Directory

Change to the provisioning working directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov"
```

Create the amota directory:

```bash
mkdir -p ./amota
```

The directory will contain the following files:

| File | Description |
| --- | --- |
| keys.ini | RSA and AES key configuration |
| firmware_ota.ini | Secure firmware package configuration |
| main_firmware_ota.bin | Generated Secure OTA package |

### 8.2. Create the Key Configuration File

The keys.ini file specifies the cryptographic keys used for firmware authentication and encryption.

The RSA keys must correspond to the OEM certificate chain generated in Chapter 7.

The AES keys must correspond to the keys included in the OEM provisioning data prepared in Chapter 6.

Create the configuration file:

```bash
nano ./amota/keys.ini
```

Insert the following content:

```ini
[Root Key]
index = 0x0
format = pem
filename = ../oem_tools_pkg/am_oem_key_gen_util/oemRSAKeys/oemRootCertKeyPair.pem
passfile = ../oem_tools_pkg/am_oem_key_gen_util/oemRSAKeys/pwdOemRootCertKey_Rsa.txt

[Key Cert Key]
index = 0x1
format = pem
filename = ../oem_tools_pkg/am_oem_key_gen_util/oemRSAKeys/oemKeyCertKeyPair.pem
passfile = ../oem_tools_pkg/am_oem_key_gen_util/oemRSAKeys/pwdOemKeyCertKey_Rsa.txt

[Content Cert Key]
index = 0x2
format = pem
filename = ../oem_tools_pkg/am_oem_key_gen_util/oemRSAKeys/oemContentCertKeyPair.pem
passfile = ../oem_tools_pkg/am_oem_key_gen_util/oemRSAKeys/pwdOemContentCertKey_Rsa.txt

[Symmetric Keys]
kcp = oem_tools_pkg/am_oem_key_gen_util/oemAESKeys/kcp.bin
kce = oem_tools_pkg/am_oem_key_gen_util/oemAESKeys/kce.bin
kb0 = oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/inputData/keyBank0.bin
kb1 = oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/inputData/keyBank1.bin
kb2 = oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/inputData/keyBank2.bin
kb3 = oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/inputData/keyBank3.bin
```

Save the file.

The key configuration uses the OEM key set prepared in Chapter 6.

### 8.3. Create the Secure Firmware OTA Configuration

The `firmware_ota.ini` file defines the firmware image, destination address, cryptographic algorithms, Content Certificate, and generated output file.

Create the configuration file:

```bash
nano ./amota/firmware_ota.ini
```

Insert the following content:

```ini
[Settings]
ambiq_owned = 0x0
chip = apollo510

app_file = ./oem_tools_pkg/cert_utils/cert_gen_utils/inputData/main_firmware_image.bin
load_address = 0x00410000

enc_algo = 0x1
auth_algo = 0x1
auth_key = 0x2
kek_index = 0x80

image_type = secure-firmware
ota = 0x1
sbl_ota = 0x0
offset = 0x0

certificate = ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_content_util_output/content_cert.bin

output = ./amota/main_firmware_ota.bin
key_table = ./amota/keys.ini
loglevel = 0x2
```

Save the file.

The configuration parameters are described below.

| Parameter | Value | Description |
| --- | --- | --- |
| chip | Apollo510 | Apollo510 tool configuration |
| app_file | main_firmware_image.bin | Application firmware |
| load_address | 0x00410000 | Application destination address |
| enc_algo | 0x1 | AES-128 CTR encryption |
| auth_algo | 0x1 | RSA authentication |
| auth_key | 0x2 | OEM Content Certificate signing key |
| kek_index | 0x80 | First OTP key-bank AES key |
| image_type | secure-firmware | Authenticated firmware image |
| ota | 0x1 | Firmware update package |
| sbl_ota | 0x0 | Customer firmware update |
| certificate | content_cert.bin | Corresponding OEM Content Certificate |

The `load_address` parameter specifies where the application will be installed after successful authentication and decryption.

It does not specify the temporary OTA staging address.

The OEM Content Certificate must correspond to the application binary included in this package.

### 8.4. Generate the Secure OTA Package

Remain in the provisioning working directory:

```text
~/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov
```

Execute:

```bash
python3 create_cust_image_blob.py -c ./amota/firmware_ota.ini
```

The utility processes the application firmware and its Content Certificate using the selected RSA and AES keys.

The generated output file is:

```text
./amota/main_firmware_ota.bin
```

This file is the Secure OTA image that will be staged in MRAM.

The package includes the metadata and cryptographic information required by the Secure Bootloader to authenticate and process the firmware update.

### 8.5. Verify the Generated Package

Verify that the generated file exists:

```bash
ls -l ./amota/main_firmware_ota.bin
```

Display its size:

```bash
stat -c '%n: %s bytes' ./amota/main_firmware_ota.bin
```

The generated package must fit entirely within the allocated OTA staging region.

According to Table 1, the staging area is:

| Parameter | Address / Size |
| --- | --- |
| OTA staging start | 0x00600000 |
| OTA staging end | 0x007EBFFF |
| Available capacity | 0x001EC000 (1968 KiB) |

Check the package size against this limit:

```bash
test "$(stat -c %s ./amota/main_firmware_ota.bin)" -le $((0x1EC000)) && echo "OTA package size: PASS" || echo "OTA package size: FAIL"
```

The Secure OTA package contains additional metadata, authentication information, and the OEM Content Certificate. Therefore, the maximum application image size that fits in the OTA staging area is smaller than the full 1968 KiB application region.

### 8.6. Secure OTA Package Placement

The generated Secure OTA package will be programmed into MRAM at:

```text
0x00600000
```

The OTA descriptor will be stored separately at:

```text
0x005FC000
```

The destination address for the authenticated application is:

```text
0x00410000
```

These addresses serve different purposes:

| Address | Purpose |
| --- | --- |
| 0x00410000 | Installed application firmware |
| 0x005FC000 | OTA descriptor |
| 0x00600000 | Temporary Secure OTA package |

During the provisioning sequence, the Secure OTA package will be staged in MRAM before the device transitions to Secure Enabled LCS.

Following provisioning and reset, the Secure Bootloader will process the OTA descriptor, authenticate and decrypt the update package, and install the application firmware together with its corresponding Content Certificate.

The installed application will execute from its designated MRAM region.

## 9. INFO0 and Secure Boot Metadata Generation

This chapter describes the generation of the INFO0 configuration image, OEM Certificate Chain Pointer Table, and OTA Descriptor required for Apollo510 Secure Boot provisioning.

The memory addresses and binary metadata are based on the MRAM organization defined in Table 1.

All generated files will be used during the device provisioning procedure.

### 9.1. Generate the INFO0 Configuration Image

INFO0 contains the device configuration parameters used by the Ambiq Secure Bootloader, including:

-  Secure Boot configuration and application start address.
-  Wired update interface configuration.
-  UART pin assignments and communication parameters.
-  Boot override configuration.
-  OEM Certificate Chain Pointer Table address.
-  Secure Debug Certificate address.
-  Wired update timeout.

The INFO0 image is generated using the AmbiqSuite SDK 5.2.0 utility:

```text
create_info0.py
```

Remain in the provisioning working directory:

```text
~/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov
```

Execute:

```bash
python3 create_info0.py --valid 1 --u0 0x01C200C0 --u1 0x00001E37 --u2 0x4 --u3 0x4 --u4 0x0 --u5 0x0 --main 0x00410000 --version 1 --wTO 5000 --sdcert 0x007FC000 --cchain 0x007EC000 info0
```

The generated binary file is:

```text
./info0.bin
```

The following parameters establish the memory locations required by this procedure.

| Parameter | Value | Description |
| --- | --- | --- |
| --valid | 1 | Valid INFO0 configuration |
| --u0 | 0x01C200C0 | UART configuration, 115200 baud |
| --u1 | 0x00001E37 | UART0 TX GPIO30, RX GPIO55 |
| --u2 | 0x4 | UART RX pin configuration |
| --u3 | 0x4 | UART TX pin configuration |
| --u4 | 0x0 | Unused UART configuration |
| --u5 | 0x0 | Unused UART configuration |
| --main | 0x00410000 | Main firmware address |
| --version | 1 | INFO0 configuration version |
| --wTO | 5000 | Wired interface timeout, milliseconds |
| --sdcert | 0x007FC000 | Secure Debug Certificate address |
| --cchain | 0x007EC000 | Certificate pointer array address |

The u0 through u5 parameters configure the wired bootloader interface and associated device settings. Their values correspond to the selected UART and Boot Override configuration.

Verify that the INFO0 image has been generated:

```bash
ls -l ./info0.bin
```

Check the file size:

```bash
stat -c '%n: %s bytes' ./info0.bin
```


### 9.2. Generate the Certificate Chain Pointer Table

The OEM Certificate Chain Pointer Table contains the MRAM addresses of the three certificates used by the Secure Boot authentication mechanism.

The table is located at:

```text
0x007EC000
```

It contains three consecutive 32-bit addresses in little-endian format.

| Offset | Certificate | Address |
| --- | --- | --- |
| 0x00 | OEM Root Certificate | 0x007F0000 |
| 0x04 | OEM Key Certificate | 0x007F4000 |
| 0x08 | OEM Content Certificate | 0x007F8000 |

The pointer table is generated as a binary file using Python.

Remain in the provisioning working directory.

Execute:

```bash
python3 -c "from pathlib import Path; import struct; Path('amota/cert_pointers.bin').write_bytes(struct.pack('<III',0x007F0000,0x007F4000,0x007F8000))"
```

The generated file is:

```text
./amota/cert_pointers.bin
```

Verify the file size:

```bash
stat -c '%n: %s bytes' ./amota/cert_pointers.bin
```

The expected size is 12 bytes.

Display its contents:

```bash
xxd -g4 -e ./amota/cert_pointers.bin
```

The three addresses must correspond to the certificate locations defined in Table 1.

The INFO0 configuration generated in Section 9.1 references this table through the cchain parameter.

### 9.3. Generate the OTA Descriptor

The OTA Descriptor provides the information required by the Ambiq Secure Bootloader to locate and process the staged firmware update package.

According to the memory map defined in Table 1, the OTA Descriptor is stored at:

```text
0x005FC000
```

The Secure OTA package generated in Chapter 8 is stored separately at:

```text
0x00600000
```

The descriptor consists of two 32-bit words stored in little-endian format.

| Offset | Value | Description |
| --- | --- | --- |
| 0x00 | 0x00600003 | OTA package address with SBL control bits |
| 0x04 | 0xFFFFFFFF | Second descriptor word |

Generate the descriptor binary.

Remain in the provisioning working directory.

Execute:

```bash
python3 -c "from pathlib import Path; import struct; Path('amota/ota_descriptor.bin').write_bytes(struct.pack('<II',0x00600003,0xFFFFFFFF))"
```

The generated file is:

```text
./amota/ota_descriptor.bin
```

Verify its size:

```bash
stat -c '%n: %s bytes' ./amota/ota_descriptor.bin
```

The expected size is 8 bytes.

Display the binary contents:

```bash
xxd -g4 -e ./amota/ota_descriptor.bin
```

The descriptor must reference the Secure OTA package stored at `0x00600000`.

## 10. OEM Provisioning and Transition to Secure LCS

This chapter describes the OEM provisioning procedure used to transition an Apollo510 device from Device Manufacturer (DM) LCS to Secure Enabled (SE) LCS.

All required cryptographic assets, OEM certificates, INFO0 configuration, Secure OTA package, and binary metadata have been prepared in the preceding chapters.

The provisioning procedure uses a single SEGGER J-Link Commander script to install the security-related data, execute the Ambiq OEM Provisioning Tool (OPT), and initiate the transition to Secure Enabled LCS.

The Apollo510 provides two customer INFO0 storage regions:

| Region | Address | Technology |
| --- | --- | --- |
| INFO0-MRAM | 0x42000000 | Rewritable |
| INFO0-OTP | 0x42004000 | One-time programmable |

Only one region is selected as the active INFO0 configuration.

The selection is controlled by OTP_INFOC_SHDW_TRIM_INFO0_SEL at:

```text
0x400C23FC
```

The selector uses the parity of its programmed bits:

-  Even number of programmed bits: `INFO0-MRAM`
-  Odd number of programmed bits: `INFO0-OTP`

A factory-default selector value of `0x00000000` selects `INFO0-MRAM.`

Choose the required programming procedure:

-  **Option A – INFO0-MRAM: Continue to Section 10.1**
-  **Option B – INFO0-OTP: Continue to section 10.2**

### 10.1. Single-Pass J-Link Provisioning Script – INFO0-MRAM

The provisioning sequence is based on the single-pass procedure successfully used during the Apollo510 laboratory test, with MRAM addresses adapted to the memory organization defined in Table 1.

Change to the provisioning working directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov"
```

Create the following script:

```bash
nano ./jlink-provision.txt
```

Insert the following content:

```text
//
// Apollo510 - Secure LCS Provisioning
// Single-Pass Provisioning Procedure
//
usb
device AP510NFA-CBR
si SWD
speed 4000
eoe 1
r
sleep 10
con
h

//
// STEP 1 - Load INFO0 image into SRAM
//
loadbin ./info0.bin 0x20081000 noreset
verifybin ./info0.bin 0x20081000

//
// STEP 2 - Program INFO0-MRAM using Boot ROM
//
w4 0x20080000 0x00000000 0x00000200 0xD894E09E 0xFFFFFFFF
setPC 0x0200FF15
g
sleep 1000
h
mem32 0x2008000C 1

//
// STEP 3, INFO0 SELECTOR: 0x400C23FC
//
//Enable OTP and Crypto power
w4 0x40021004 0x08200000
sleep 100

// Odd number of bits = OTP
// Even number of bits = MRAM
//
// factory default: 0x00000000 = MRAM
// for OTP, execute:
//                   w4 0x400C23FC 0x00000001
mem32 0x400C23FC 1

//
// STEP 4 - Install OEM Certificate Chain
//
loadbin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_key_util_output/oem_root_cert_hbk1.bin 0x007F0000 noreset
loadbin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_key_util_output/oem_key_cert.bin 0x007F4000 noreset
loadbin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_content_util_output/content_cert.bin 0x007F8000 noreset
loadbin ./amota/cert_pointers.bin 0x007EC000 noreset

//
// STEP 5 - Stage Secure OTA Package and Descriptor
//

loadbin ./amota/main_firmware_ota.bin 0x00600000 noreset
loadbin ./amota/ota_descriptor.bin 0x005FC000 noreset

//
// STEP 6 - Verify Programmed MRAM Contents
//
verifybin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_key_util_output/oem_root_cert_hbk1.bin 0x007F0000
verifybin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_key_util_output/oem_key_cert.bin 0x007F4000
verifybin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_content_util_output/content_cert.bin 0x007F8000
verifybin ./amota/cert_pointers.bin 0x007EC000
verifybin ./amota/main_firmware_ota.bin 0x00600000
verifybin ./amota/ota_descriptor.bin 0x005FC000

//
// STEP 7 - Load OEM Provisioning Assets into SRAM
//
loadbin ./oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/dmpu_prov_data_blob_sec.bin 0x20037000 noreset
loadbin ./oem_tools_pkg/oem_prov_tool/opt_image_pkg.bin 0x20030000 noreset

//
// STEP 8 - Verify SRAM Contents
//

verifybin ./oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/dmpu_prov_data_blob_sec.bin 0x20037000
verifybin ./oem_tools_pkg/oem_prov_tool/opt_image_pkg.bin 0x20030000

//
// STEP 9 - Activate OTA and Initiate Provisioning
//
w4 0x40020264 0x005FC001
mem32 0x40020264 1

exec SetRestartOnClose = 0
w4 0x40000008 0x000000D4
sleep 1000
exit
```

### 10.2. Single-Pass J-Link Provisioning Script – INFO0-OTP

The provisioning sequence is based on the single-pass procedure successfully used during the Apollo510 laboratory test, with MRAM addresses adapted to the memory organization defined in Table 1.

Change to the provisioning working directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov"
```

Create the following script:

```bash
nano ./jlink-provision.txt
```

Insert the following content:

```text
//
// Apollo510 - Secure LCS Provisioning
// Single-Pass Provisioning Procedure
//
usb
device AP510NFA-CBR
si SWD
speed 4000
eoe 1
r
sleep 10
con
h

//
// STEP 1 - Load INFO0 image into SRAM
//
loadbin ./info0.bin 0x20081000 noreset
verifybin ./info0.bin 0x20081000

//
// STEP 2 - Program INFO0-OTP using Boot ROM
//
w4 0x20080000 0x00000000 0x00000040 0xD894E09E 0xFFFFFFFF
setPC 0x0200FF59
g
sleep 1000
h
mem32 0x2008000C 1

//
// STEP 3, INFO0 SELECTOR: 0x400C23FC
//
//Enable OTP and Crypto power
w4 0x40021004 0x08200000
sleep 100

// Odd number of bits = OTP
// Even number of bits = MRAM
//
// factory default: 0x00000000 = MRAM
// for OTP, execute:
w4 0x400C23FC 0x00000001
mem32 0x400C23FC 1

//
// STEP 4 - Install OEM Certificate Chain
//
loadbin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_key_util_output/oem_root_cert_hbk1.bin 0x007F0000 noreset
loadbin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_key_util_output/oem_key_cert.bin 0x007F4000 noreset
loadbin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_content_util_output/content_cert.bin 0x007F8000 noreset
loadbin ./amota/cert_pointers.bin 0x007EC000 noreset

//
// STEP 5 - Stage Secure OTA Package and Descriptor
//

loadbin ./amota/main_firmware_ota.bin 0x00600000 noreset
loadbin ./amota/ota_descriptor.bin 0x005FC000 noreset

//
// STEP 6 - Verify Programmed MRAM Contents
//
verifybin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_key_util_output/oem_root_cert_hbk1.bin 0x007F0000
verifybin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_key_util_output/oem_key_cert.bin 0x007F4000
verifybin ./oem_tools_pkg/cert_utils/cert_gen_utils/am_cert_content_util_output/content_cert.bin 0x007F8000
verifybin ./amota/cert_pointers.bin 0x007EC000
verifybin ./amota/main_firmware_ota.bin 0x00600000
verifybin ./amota/ota_descriptor.bin 0x005FC000

//
// STEP 7 - Load OEM Provisioning Assets into SRAM
//
loadbin ./oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/dmpu_prov_data_blob_sec.bin 0x20037000 noreset
loadbin ./oem_tools_pkg/oem_prov_tool/opt_image_pkg.bin 0x20030000 noreset

//
// STEP 8 - Verify SRAM Contents
//

verifybin ./oem_tools_pkg/oem_asset_prov_utils/oem_asset_package/dmpu_prov_data_blob_sec.bin 0x20037000
verifybin ./oem_tools_pkg/oem_prov_tool/opt_image_pkg.bin 0x20030000

//
// STEP 9 - Activate OTA and Initiate Provisioning
//
w4 0x40020264 0x005FC001
mem32 0x40020264 1

exec SetRestartOnClose = 0
w4 0x40000008 0x000000D4
sleep 1000
exit
```

### 10.3. Register Readback and Verification

The J-Link provisioning script uses `mem32` commands to read back selected registers and operation status values after programming. These readbacks provide diagnostic information in the J-Link execution log but are not automatically compared against expected values.

The `verifybin` commands perform binary comparisons of the programmed memory contents, while `eoe 1` terminates script execution if a J-Link command reports an error.

For automated production provisioning, additional validation should be implemented to prevent the final lifecycle transition if a critical security configuration check fails.

### 10.4. Execute OEM Provisioning

Connect the Apollo510 device to SEGGER J-Link through the SWD interface.

The target device must initially operate in Device Manufacturer (DM) LCS.

Ensure that the target power supply remains stable throughout the provisioning operation.

Remain in the provisioning working directory.

Execute:

```bash
JLinkExe -CommandFile ./jlink-provision.txt
```

The script performs the provisioning sequence in a single J-Link Commander session.

The J-Link log displays the programming operations, binary verification results, Boot ROM status, and final reset sequence.

The eoe 1 setting stops script execution if a J-Link command reports an error.

After successful OEM provisioning, the device transitions to Secure Enabled LCS.

Following reset, the Secure Bootloader processes the staged Secure OTA package, authenticates and decrypts the firmware update, and installs the application with its associated OEM Content Certificate.

The application is installed at:

```text
0x00410000
```

Once the device has entered Secure Enabled LCS, unrestricted SWD debugging is disabled according to the programmed security policy.

### 10.5. Verify Secure LCS and Secure Boot Configuration Using UART

After completing OEM provisioning, verify the device security configuration using the Secure Bootloader UART wired interface.

The Boot Override mechanism allows communication with the Secure Bootloader independently of the installed customer application.

The INFO0 configuration prepared in Chapter 9 specifies a wired update timeout of 5000 ms. The initial HELLO command must therefore be sent within this interval after the Secure Bootloader enters wired update mode.

**Note:** In Secure LCS, SWD and SWO access are disabled by default. Therefore, Secure Bootloader diagnostic output over SWO cannot be assumed to be available.

The UART Boot Override interface provides access to device status and security configuration independently of the customer application.

Verification of firmware authentication and execution requires an additional observation mechanism, such as application-level diagnostics or authorized debug access.

#### 10.5.1 Connect the UART Interface

Connect an external USB-to-UART adapter to the Apollo510 board.

The UART interface uses the pin configuration defined in INFO0.

| Signal | MCU pin | Connection |
| --- | --- | --- |
| UART0 RX | GPIO55 | USB-UART adapter TX |
| UART0 TX | GPIO30 | USB-UART adapter RX |
| Ground | GND | USB-UART adapter GND |
| Boot Override | GPIO93 | Pull LOW during reset |

Use a USB-UART adapter compatible with the target GPIO voltage levels (1.8 V).

The UART communication parameters are:

| Parameter | Value |
| --- | --- |
| Interface | UART0 |
| Baud rate | 115200 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |
| Hardware flow control | Disabled |
| Wired update timeout | 5000 ms |

#### 10.5.2 Prepare the UART Communication

Activate the Python virtual environment:

```bash
source "$HOME/apollo510-env/bin/activate"
```

Change to the provisioning working directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov"
```

Identify the connected USB-UART adapter:

```bash
python3 -m serial.tools.list_ports -v
```

For the laboratory configuration described in this guide, the external FTDI adapter is connected through:

```text
/dev/ttyUSB0
```

The actual serial device name may differ depending on the Ubuntu configuration.

Prepare the following command in the terminal, but do not execute it yet:

```bash
python3 uart_wired_update.py -b 115200 -r 0 -o 0xFFFFFFFF /dev/ttyUSB0
```

The command requests Secure Bootloader status information without downloading firmware, programming an OTA descriptor, or requesting a device reset.

The command uses the following parameters:

| Parameter | Value | Description |
| --- | --- | --- |
| -b | 115200 | UART baud rate |
| -r | 0 | Disables software reset by the utility |
| -o | 0xFFFFFFFF | No OTA descriptor address specified |
| Serial port | /dev/ttyUSB0 | External USB-UART adapter |

No firmware image is specified. The utility is used to establish communication with the Secure Bootloader and retrieve device status information.

#### 10.5.3 Enter Boot Override Mode

The INFO0 configuration specifies:

```text
--wTO 5000
```

This configures a 5000 ms timeout for wired update communication.

Perform the following procedure:

1. Prepare the UART command in the Ubuntu terminal.
2. Pull GPIO93 LOW to assert Boot Override.
3. Reset the Apollo510 device while GPIO93 remains LOW.
4. Immediately execute the prepared UART command by pressing Enter.
5. Confirm reception of the HELLO response and STATUS message.

The initial HELLO message must reach the Secure Bootloader within the configured 5-second communication window. If communication is not established, repeat the Boot Override entry sequence.

#### 10.5.4 Execute HELLO and STATUS Commands

The `uart_wired_update.py` utility communicates with the Secure Bootloader using the Ambiq wired update protocol.

When the Secure Bootloader accepts the request, it responds with a STATUS message containing information about the bootloader, security configuration, and device identification.

A successful communication sequence produces the example response:

```text
Connecting with Apollo5 over serial port /dev/ttyUSB0...
Sending Hello.
Received response for Hello
Received Status
length = 0x60 bytes
SBL Version Major =  1
SBL Version Minor =  13
Max Storage =  0x6fe30
Status =  0x91ee32b7
State =  0xa
chipId0 =  0xcb95aa7b
chipId1 =  0x18c0c644
SDCert =  0x7fc000
Chip Revision =  0xeaf23
SOCId =
0x6bccf9b7
0x862a7e4e
0xf2dc0b6f
0x44e656cd
0x396fca2a
0xb7032c1d
0xaaaf005c
0xbfc6f1
AMInfo =
0x0
0x222002a0
0xb6800000
0x509
0x402000
0x0
Done.
```

The response fields have the following meanings:

| Field | Description |
| --- | --- |
| length | STATUS response length in bytes. |
| SBL version | Secure Bootloader version field. |
| Max Storage | Maximum temporary storage available for wired update operations. |
| Status | Raw Secure Bootloader status field. |
| State | Current Secure Bootloader protocol state. This is not the device Life Cycle State. |
| chipId0, chipId1 | Device identification words. |
| SDCert | Configured Secure Debug Certificate memory address. |
| Chip Revision | Raw chip revision |
| Resv | Reserved field. |
| SOCId | Device-specific 256-bit System-on-Chip Identifier. |
| AMInfo | Additional device and security configuration information. |

#### 10.5.5 Verify the Reported Security Configuration

The Secure Bootloader STATUS response provides information about the device configuration and identity.

Verify the following fields:

| Parameter | Expected result | Description |
| --- | --- | --- |
| HELLO response | Received response for Hello | Confirms successful communication with the Secure Bootloader. |
| STATUS response | Received Status | Confirms that the Secure Bootloader returned its status information. |
| Secure Debug Certificate address | SDCert = 0x7FC000 | Matches the address configured in INFO0 and the memory map defined in Table 1. |
| SOC-ID | Eight 32-bit words | Provides the 256-bit device identifier required for Secure Debug certificate generation. |
| Communication result | Done. | Confirms successful completion of the UART utility. |

The reported `SDCert` value identifies the configured storage location of the Secure Debug Certificate. It does not indicate that a certificate is already installed.

**Secure LCS indication:** In the verified Apollo510 provisioning flow, the device returned an unpopulated SOC-ID (0xFFFFFFFF) before the transition to Secure LCS. After provisioning, the UART STATUS response returned the device-specific 256-bit SOC-ID. This provides an indication that Secure Enabled LCS is active.

Successful HELLO and STATUS communication confirms that the Secure Bootloader wired interface is operational. Firmware authentication and application execution are verified separately

#### 10.5.6 Retrieve and Record the Device SOC-ID

The STATUS response contains the device-specific System-on-Chip Identifier (SOC-ID).

The SOC-ID consists of eight 32-bit words, representing a total of 256 bits.

The identifier is displayed in the following format:

```text
SOCId =
0xXXXXXXXX
0xXXXXXXXX
0xXXXXXXXX
0xXXXXXXXX
0xXXXXXXXX
0xXXXXXXXX
0xXXXXXXXX
0xXXXXXXXX
```

The SOC-ID is required to generate the device-specific OEM Secure Debug Developer Certificate.

The UART STATUS response should be saved to a local text file. Remain in the provisioning directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov"
```

To capture the output automatically, use the following command during the UART communication procedure:

```bash
python3 uart_wired_update.py -b 115200 -r 0 -o 0xFFFFFFFF /dev/ttyUSB0 | tee ./amota/uart_status.log
```

The command saves the complete UART output to:

```text
./amota/uart_status.log
```

The saved log must contain the SOCId and AMInfo fields, with the eight SOC-ID words between them.

The following function extracts the SOC-ID from the saved STATUS response and prepares the text file required for certificate generation.

Copy the entire code block below and paste it into the Ubuntu terminal using **Ctrl+Shift+V**. Press **Enter** to execute the function.

```bash
Apollo510_extract_socid() {
python3 - <<'PY'
from pathlib import Path
import re

source = Path("amota/uart_status.log")
output = Path("oem_tools_pkg/cert_utils/cert_gen_utils/inputData/socid.txt")

if not source.is_file():
    raise SystemExit("FAIL: UART STATUS log not found")

text = source.read_text(encoding="utf-8", errors="replace")

match = re.search(
    r"(?ms)^[ \t]*SOCId[ \t]*=[ \t]*\r?\n(.*?)^[ \t]*AMInfo[ \t]*=",
    text
)

if match is None:
    raise SystemExit("FAIL: SOC-ID section not found")

lines = [
    line.strip()
    for line in match.group(1).splitlines()
    if line.strip()
]

if len(lines) != 8:
    raise SystemExit("FAIL: Expected exactly eight SOC-ID words")

if not all(re.fullmatch(r"0x[0-9A-Fa-f]{1,8}", line) for line in lines):
    raise SystemExit("FAIL: Invalid SOC-ID word format")

words = [int(line, 16) for line in lines]

if all(word == 0 for word in words):
    raise SystemExit("FAIL: Uninitialized SOC-ID (all zeros)")

if all(word == 0xFFFFFFFF for word in words):
    raise SystemExit("FAIL: Uninitialized SOC-ID (all ones)")

formatted = " : ".join(f"0x{word:08X}" for word in words)

output.write_text(formatted + "\n", encoding="ascii")

print("PASS: Eight 32-bit SOC-ID words extracted")
print("PASS: SOC-ID text file generated")
print("Output:", output)
PY
}
Apollo510_extract_socid
```

Expected confirmation:

```text
PASS: Eight 32-bit SOC-ID words extracted
PASS: SOC-ID text file generated
```

The generated file is:

```text
./oem_tools_pkg/cert_utils/cert_gen_utils/inputData/socid.txt
```

It contains eight hexadecimal words separated by colons, following the format used by the Apollo510 SOC-ID conversion utility.

Data format:

```text
0xXXXXXXXX : 0xXXXXXXXX : 0xXXXXXXXX : 0xXXXXXXXX : 0xXXXXXXXX : 0xXXXXXXXX : 0xXXXXXXXX : 0xXXXXXXXX
```

The word order is preserved exactly as reported by the Secure Bootloader.

Change to the certificate generation directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/oem_tools_pkg/cert_utils/cert_gen_utils"
```

Use the `socid_to_bin.py` conversion utility from the Apollo510 Secure Debug certificate preparation procedure.

Execute:

```bash
python3 "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/socid_to_bin.py" ./inputData/socid.txt -o ./inputData/socid.bin
```

The generated output file is:

```text
./inputData/socid.bin
```

The conversion utility prepares the binary SOC-ID representation used by the OEM Debug Developer Certificate generator.

Check the file size:

```bash
stat -c '%n: %s bytes' ./inputData/socid.bin
```

The expected size is:

```text
32 bytes
```

The generated binary file is now available for the OEM Debug Developer Certificate generation procedure.

### Preserve the Device Identification Record

Keep the following files together with the device provisioning record:

| File | Description |
| --- | --- |
| uart_status.log | Original Secure Bootloader UART STATUS output |
| socid.txt | Extracted SOC-ID in hexadecimal text format |
| socid.bin | Binary SOC-ID input for Secure Debug certificate generation |

The SOC-ID must be associated with the correct physical device.

A Secure Debug Developer Certificate generated for another SOC-ID cannot authorize debugging of another device.

## 11. Secure Debug Certificate Generation and Installation

This chapter describes the generation and installation of a device-specific Secure Debug Certificate for an Apollo510 device operating in Secure Enabled Life Cycle State (SE LCS).

Secure Debug allows selected debugging capabilities to be restored through authenticated certificates, according to the device security configuration.

The procedure uses the OEM cryptographic keys prepared in Chapter 6 and the device SOC-ID obtained through the Secure Bootloader UART interface in Section 10.5.6.

The Secure Debug Certificate is installed through the Secure Bootloader wired update interface, independently of the customer application.

### 11.1. Secure Debug Certificate Structure

Apollo510 supports a three-level Secure Debug Certificate chain consisting of:

1. **OEM Debug-Key Certificate** — establishes authentication of the debug certificate hierarchy using the OEM Root of Trust.
2. **OEM Debug Enabler Certificate** — defines the authorized Debug Control Unit (DCU) permissions and authenticates the Developer Certificate public key.
3. **OEM Debug Developer Certificate** — specifies the requested debug permissions and binds the authorization to the device-specific SOC-ID.

### 11.2. Prepare the Certificate Generation Environment

Activate the Python virtual environment:

```bash
source "$HOME/apollo510-env/bin/activate"
```

Configure access to the legacy OpenSSL installation:

```bash
export PATH="$HOME/ambiq-legacy/openssl-install/bin:$PATH"
export LD_LIBRARY_PATH="$HOME/ambiq-legacy/openssl-install/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

Verify the environment:

```bash
python --version
openssl version
command -v openssl
```

Expected results:

```text
Python 3.10.12
OpenSSL 1.0.2u  20 Dec 2019
/home/<username>/ambiq-legacy/openssl-install/bin/openssl
```

Change to the certificate generation directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov/oem_tools_pkg/cert_utils/cert_gen_utils"
```

The procedure uses the OEM RSA keys previously selected for device provisioning.

The same OEM Root key hierarchy must be used for generating the Secure Debug Certificate chain.

### 11.3. Generate the OEM Debug-Key Certificate

The OEM Debug-Key Certificate establishes the first level of the Secure Debug authentication chain.

The certificate is generated using:

```text
am_cert_key_util.py
```

Configuration file:

```text
./am_config/am_oem_dbg_key_cert.cfg
```

Execute:

```bash
python3 am_cert_key_util.py ./am_config/am_oem_dbg_key_cert.cfg
```

The generated files are:

| File | Description |
| --- | --- |
| am_cert_key_util_output/debug_oem_key_cert.bin | Binary OEM Debug-Key Certificate |
| am_cert_key_util_output/debug_oem_key_cert_Cert.txt | Text representation of the certificate |

The certificate authenticates the public key used for the next level of the Secure Debug hierarchy.

### 11.4. Generate the OEM Debug Enabler Certificate

The OEM Debug Enabler Certificate defines the maximum debugging permissions that may be granted to the Developer Certificate.

The configuration includes DCU enable and lock masks, together with the public key used to authenticate the Developer Certificate.

The certificate must authorize the debugging capabilities required for the target device.

Configuration file:

```text
./am_config/am_oem_dbg_enabler_cert.cfg
```

The utility is:

```text
am_cert_dbg_enabler_util.py
```

Execute:

```bash
python3 am_cert_dbg_enabler_util.py ./am_config/am_oem_dbg_enabler_cert.cfg
```

The generated binary file is:

```text
./am_debug_cert_output/oem_dbg_cert_enabler_pkg.bin
```

The Enabler Certificate incorporates the Debug-Key Certificate generated in Section 11.3.

### 11.5. Generate the Device-Specific OEM Debug Developer Certificate

The OEM Debug Developer Certificate is the final certificate in the Secure Debug authentication chain. It contains the device-specific SOC-ID and the selected DCU debug permissions. The SOC-ID was obtained from the Secure Bootloader STATUS response in Section 10.5.6. It consists of eight 32-bit words, representing a 256-bit device identifier. The Developer Certificate configuration must use the SOC-ID of the target device, preserving the required word and byte ordering.

Configuration file:

```text
./am_config/am_oem_dbg_developer_cert.cfg
```

The utility is:

```text
am_cert_dbg_developer_util.py
```

The configuration references the Debug Enabler Certificate generated in Section 11.4 and the corresponding OEM Developer signing key.

Execute:

```bash
python3 am_cert_dbg_developer_util.py ./am_config/am_oem_dbg_developer_cert.cfg
```

The generated files are:

| File | Description |
| --- | --- |
| am_debug_cert_output/oem_dbg_cert_developer_pkg.bin | Complete OEM Secure Debug Certificate package |
| am_debug_cert_output/oemDeveloperCert.h | C header representation of the Developer Certificate |

The binary package contains the certificate chain required by the device security subsystem. The Secure Debug Certificate is device-specific and must correspond to the SOC-ID of the target Apollo510.

### 11.6. Secure Debug Certificate Installation Through UART

The Apollo510 Secure Bootloader supports installation of an OEM Secure Debug Certificate through its UART wired update interface. This mechanism allows a device operating in Secure Enabled LCS to receive a device-specific Secure Debug Certificate without requiring an active SWD connection. The procedure uses the OEM Debug Developer Certificate generated in the preceding sections. The certificate contains the complete Secure Debug Certificate chain and is associated with the target device SOC-ID. The Secure Debug Certificate will be installed in the dedicated MRAM region defined in Table 1.

#### 11.6.1 Create the Secure Debug Wired Download Configuration

The Secure Debug Certificate must be packaged into the wired download format supported by the Ambiq Secure Bootloader.

The AmbiqSuite `create_cust_image_blob.py` utility is used to generate this package.

Change to the provisioning working directory:

```bash
cd "$HOME/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov"
```

Create the configuration file:

```bash
nano ./amota/wired_debug_cert.ini
```

Insert the following configuration:

```ini
[Settings]
ambiq_owned = 0x0
chip = apollo510

app_file = ./oem_tools_pkg/cert_utils/cert_gen_utils/am_debug_cert_output/oem_dbg_cert_developer_pkg.bin
load_address = 0x007FC000

enc_algo = 0x0
auth_algo = 0x0
kek_index = 0x80

image_type = wired
ota = 0x0
sbl_ota = 0x0
offset = 0x0

certificate = None
output = ./amota/debug_cert_wired.bin
key_table = ./amota/keys.ini

loglevel = 0x2
wired_chunk_size = 0x40000
```

Save the file.

The configuration parameters are described below.

| Parameter | Value | Description |
| --- | --- | --- |
| chip | Apollo510 | Apollo510 image generation configuration |
| app_file | oem_dbg_cert_developer_pkg.bin | Complete OEM Secure Debug Certificate package |
| load_address | 0x007FC000 | Destination address in MRAM |
| enc_algo | 0 | No additional wired transport encryption |
| auth_algo | 0 | No additional wired transport RSA authentication |
| kek_index | 0x80 | Key encryption key index; not used when encryption is disabled |
| image_type | wired | Secure Bootloader wired download format |
| ota | 0 | Direct wired download without OTA staging |
| sbl_ota | 0 | SBL firmware update disabled |
| certificate | None | No separate firmware Content Certificate |
| output | debug_cert_wired.bin | Generated wired download package |
| key_table | ./amota/keys.ini | Existing OEM cryptographic key configuration |
| wired_chunk_size | 0x40000 | Maximum wired image chunk size, 256 KiB |

The transport image does not use additional encryption or RSA authentication. The Secure Debug Certificate itself contains the cryptographic signatures required for certificate-chain authentication by the device security subsystem.

The existing `keys.ini` file is reused without modification.

#### 11.6.2 Generate the Secure Debug Wired Package

Remain in the provisioning working directory:

```bash
cd ~/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov
```

Execute:

```bash
python3 create_cust_image_blob.py -c ./amota/wired_debug_cert.ini
```

The utility processes the complete Secure Debug Developer Certificate and generates the wired download image.

The resulting file is:

```text
./amota/debug_cert_wired.bin
```

Verify that the file has been generated:

```bash
ls -l ./amota/debug_cert_wired.bin
```

Display its size:

```bash
stat -c '%n: %s bytes' ./amota/debug_cert_wired.bin
```

The generated file contains the Secure Debug Certificate and the wired download metadata required by the Secure Bootloader.

The binary package is now ready for transfer through UART.

#### 11.6.3 Install and Activate the Secure Debug Certificate Using UART Boot Override

Use the external USB-UART adapter as described in Chapter 10.5.

The INFO0 configuration specifies a wired update timeout of 5000 ms.

The debug certificate installation command must be executed before this timeout expires.

Prepare the following command in the terminal, but do not execute it yet:

```bash
python3 uart_wired_update.py -b 115200 /dev/ttyUSB0 -r 0 -o 0xFFFFFFFF -f ./amota/debug_cert_wired.bin
```

The command parameters are:

| Parameter | Description |
| --- | --- |
| -b 115200 | UART communication speed |
| /dev/ttyUSB0 | External USB-UART adapter |
| -r 0 | Do not send a reset command after image download |
| -o 0xFFFFFFFF | Skip OTA Descriptor transmission |
| -f | Specifies the Secure Debug wired download package |

Perform the following sequence:

1. Assert Boot Override by pulling GPIO93 LOW.
2. Reset the Apollo510 device while GPIO93 remains LOW.
3. Immediately press Enter to execute the prepared UART command.
4. Confirm that the Secure Bootloader responds to the HELLO request.
5. Allow the utility to complete the Secure Debug wired image transfer.
6. Confirm that the transfer has completed successfully.
7. Release GPIO93 to deactivate Boot Override.
8. Perform a complete device reset or power cycle with GPIO93 inactive.
9. Allow the device to complete its normal initialization.

The expected communication begins with:

```text
Connecting with Apollo510 over serial port /dev/ttyUSB0...
Sending Hello.
Received response for Hello
Received Status
```

After receiving the STATUS response, the utility transfers the wired image using the Secure Bootloader update protocol. The transfer consists of an UPDATE command followed by DATA messages containing the prepared image.

A successful transmission must finish without NACK responses, communication timeouts, or other reported errors.

The `-r` parameter controls the reset operation requested by the UART utility after image download.

| Parameter | Operation |
| --- | --- |
| -r 0 | No reset command sent after image download |
| -r 1 | Power-On Initialization (POI) reset |
| -r 2 | Power-On Reset (POR) |

During normal device initialization, the device security subsystem evaluates the installed Secure Debug Certificate and applies the authorized Debug Control Unit (DCU) permissions. The resulting debug access is determined by the certificate chain, device-specific SOC-ID, and configured DCU masks.

**Important**: Successful UART transmission confirms completion of the wired download operation but does not confirm activation of the requested Secure Debug permissions. The resulting debug access must be verified separately.

#### 11.6.4 Verify Debug Access

Connect the SEGGER J-Link debugger to the Apollo510 through SWD.

Open an Ubuntu terminal and execute:

```bash
JLinkExe -device AP510NFA-CBR -if SWD -speed 4000 -autoconnect 1
```

Attempt to establish a debug connection.

If the Secure Debug Certificate has been accepted and the corresponding DCU permissions authorize SWD access, the debugger can connect to the device.

Once SWD communication has been restored, the contents of the protected device memory may be inspected according to the effective security permissions.

The first diagnostic operation should be a read-only examination of the MRAM regions containing the Certificate Chain Pointer Table, OEM certificates, and application firmware.

The Secure Debug Certificate installation does not modify the existing OEM Root, Key, or Content Certificates.

The ability to read individual MRAM regions depends on the effective device protection configuration.

## 12. Removing the Secure Debug Certificate

The installed Secure Debug Certificate can be removed by overwriting its dedicated MRAM region with 0xFF using the UART Wired Download functionality provided by the Ambiq Secure Bootloader (SBL).

This procedure does not require an active SWD connection or an application-level debugger enable mechanism.

The procedure has been successfully validated on the Apollo510 AP510NFA-CBR Rev B2 operating in Secure LCS with SBL version 1.13 and AmbiqSuite SDK 5.2.0.

### 12.1. Select the Working Directory

With the Python virtual environment activated, execute:

```bash
cd ~/AmbiqSuite_5.2.0/tools/apollo510_scripts_prov
```

### 12.2. Generate a Blank Secure Debug Certificate Image

The MRAM region reserved for the Secure Debug Certificate in this guideline is defined as follows:

| Parameter | Value |
| --- | --- |
| Start address | 0x007FC000 |
| End address | 0x007FFFFF |
| Region size | 0x4000 |
| Region size in bytes | 16384 (16 KiB) |
| Blank data value | 0xFF |

Generate a binary file containing 16,384 bytes of 0xFF:

```bash
python3 -c "from pathlib import Path; Path('./amota/blank_cert_16k.bin').write_bytes(b'\xff'*0x4000)"
```

Verify the generated file size:

```bash
stat -c '%n: %s bytes' ./amota/blank_cert_16k.bin
```

Expected result:

```text
./amota/blank_cert_16k.bin: 16384 bytes
```

This binary file represents the desired contents of the entire Secure Debug Certificate region after removal.

### 12.3. Prepare the Wired Download Configuration

Create a new configuration file based on the previously used Secure Debug Certificate installation configuration:

```bash
cp ./amota/wired_debug_cert.ini ./amota/wired_blank_cert.ini
```

Open the configuration file:

```bash
nano ./amota/wired_blank_cert.ini
```

Modify or verify the following three parameters:

```ini
app_file = ./amota/blank_cert_16k.bin

load_address = 0x007FC000

output = ./amota/blank_cert_wired.bin
```

All other configuration parameters must remain unchanged, including the authentication and encryption settings.

The original Secure Debug Certificate installation configuration and its generated binary image must be retained for subsequent restoration of authorized debug access.

**IMPORTANT:** The destination address and image size must correspond to the MRAM region reserved for the Secure Debug Certificate. The operation must not overwrite the OEM Root, Key, or Content Certificates, the certificate chain pointer table, or the application firmware.

### 12.4. Generate the Wired Download Image

Execute the original AmbiqSuite utility:

```bash
python3 create_cust_image_blob.py -c ./amota/wired_blank_cert.ini
```

Expected output file:

```text
./amota/blank_cert_wired.bin
```

Verify the generated image size:

```bash
stat -c '%n: %s bytes' ./amota/blank_cert_wired.bin
```

For the configuration validated in this guideline, the expected result is:

```text
./amota/blank_cert_wired.bin: 16416 bytes
```

The generated Wired Download image consists of the 16 KiB blank binary image and 32 bytes of Wired Download metadata.

### 12.5. Remove the Secure Debug Certificate Using Boot Override UART

Prepare the device for UART Wired Download:

1. Terminate any active J-Link session.
2. Ensure that the application-level debugger enable mechanism is inactive (GPIO94 = LOW, if implemented).
3. Activate Boot Override by pulling GPIO93 LOW.
4. Reset the device.
5. Execute the following command within the configured 5-second Wired Download timeout:

```bash
python3 uart_wired_update.py -b 115200 /dev/ttyUSB0 -r 0 -o 0xFFFFFFFF -f ./amota/blank_cert_wired.bin
```

The SBL receives the Wired Download image and overwrites the Secure Debug Certificate region with the supplied blank data.

The `-r 0` parameter prevents the host utility from issuing an automatic reset command after completing the download.

The transfer must complete without NACK responses, communication timeouts, or other reported errors.

Expected output from a successful transmission:

```text
Connecting with Apollo5 over serial port /dev/ttyUSB0...
Sending Hello.
Received response for Hello
Received Status
...
SDCert =  0x7fc000
...
Found 1 segments with max size 0x4020.
Sending Update Command.
number of updates needed =  1
Sending block of size  0x4020  from  0x0  to  0x4020
Received ACK
0x0  to  0x1ff4
Sending Data Packet of length  8180
0x20 0x40
Received ACK
0x1ff4  to  0x3fe8
Sending Data Packet of length  8180
0xff 0xff
Received ACK
0x3fe8  to  0x4020
Sending Data Packet of length  56
0xff 0xff
Received ACK
Done.
```

Successful completion of the Wired Download confirms that the SBL has acknowledged the transmitted image.

**IMPORTANT:** During laboratory validation with SBL version 1.13, SWD access became unavailable immediately after completion of the Secure Debug Certificate removal operation. No additional reset command or manual power cycle was required to observe the loss of SWD access.

Consequently, an active Secure Debug session or continued availability of the SWD interface must not be assumed after removing the Secure Debug Certificate.

### 12.6. Deactivate Secure Debug

During laboratory validation with SBL version 1.13, SWD access was disabled immediately after successful removal of the Secure Debug Certificate.

A complete device power cycle is nevertheless recommended to verify that debug access remains disabled following normal device initialization. After completing the Wired Download:

1. Release GPIO93 to deactivate Boot Override.
2. Perform a complete device power cycle.
3. Allow the device to complete its normal Secure Boot sequence.

During device initialization, the Secure BootROM (SBR) no longer has a valid Secure Debug Certificate at the configured MRAM address.

The effective debug permissions are consequently determined by the device Life Cycle State and the provisioned security policy, without the overrides previously authorized by the Secure Debug Certificate.

### 12.7. Verify Secure Debug Deactivation

Attempt to establish a new SWD connection using SEGGER J-Link:

```bash
JLinkExe -device AP510NFA-CBR -if SWD -speed 4000 -autoconnect 1
```

For the security configuration established in this guideline, the expected result is:

```text
Failed to attach to CPU. Trying connect under reset.

Error occurred: Could not connect to the target device.
```

The inability to establish an SWD connection, under otherwise unchanged and previously verified connection conditions, provides functional confirmation that the previously authorized Secure Debug access is no longer available.

## 13. References

The following official Ambiq documentation and software resources provide technical background for the Apollo510 security architecture, lifecycle states, OEM provisioning, Secure Boot, firmware update mechanisms, MRAM recovery, and Secure Debug functionality discussed in this guide.

**[1] Apollo5 Family and Apollo330 Plus OEM Provisioning Tools User's Guide**

Ambiq Micro, Document A-SOCAP5-UGGA03EN-2p0, Revision 2.0, July 2026.

Describes OEM provisioning utilities, security asset generation, OEM certificate chains, INFO0 configuration, Secure Debug certificate generation, and the preparation of secure firmware update packages.

<https://contentportal.ambiq.com/documents/20123/387752/Apollo5-Family-and-Apollo330Plus-OEM-Provisioning-Tools-Users-Guide.pdf>

**[2] Apollo5 Family and Apollo330 Plus Secure Update User's Guide**

Ambiq Micro, Document A-SOCAP5-UGGA02EN-2p0, Revision 2.0, July 2026.

Describes secure firmware update mechanisms, image authentication and encryption, OTA image formats, OTA descriptors, and wired update protocols supported by the Secure Bootloader.

<https://contentportal.ambiq.com/documents/20123/387752/Apollo5-Family-and-Apollo330Plus-Secure-Update-UsersGuide.pdf>

**[3] Apollo5 Family and Apollo330 Plus Security User's Guide**

Ambiq Micro, Document A-SOCAP5-UGGA04EN-2p0, Revision 2.0, July 2026.

Provides reference information on the device security architecture, lifecycle states, security policies, OTP configuration, hardware roots of trust, Secure Boot, and Secure Debug functionality.

<https://contentportal.ambiq.com/documents/20123/387752/Apollo5-Family-and-Apollo330Plus-Security-User-Guide.pdf>

**[4] Apollo5 Family and Apollo330 Plus MRAM Recovery User's Guide**

Ambiq Micro, Document A-SOCAP5-UGGA01EN-2p0, Revision 2.0, July 2026.

Provides reference information on MRAM recovery mechanisms, recovery procedures, and the associated device configuration and security considerations.

<https://contentportal.ambiq.com/documents/20123/387752/Apollo5-Family-and-Apollo330Plus-MRAM-Recovery-Users-Guide.pdf>

**[5] Apollo SoC Family Technical Reference Manual**

Ambiq Micro, Revision 1.1.

Applicable to Apollo510, Apollo510B, Apollo510 Lite Series, and Apollo330 Plus Series SoCs.

Provides detailed reference information on the SoC architecture, memory organization, peripheral subsystems, register definitions, reset mechanisms, and hardware configuration.

<https://contentportal.ambiq.com/documents/20123/387752/Apollo_SoC_Family_Technical_Reference_Manual_v1p1.pdf>

**[6] AmbiqSuite SDK 5.2.0**

Ambiq Micro, Software Development Kit, Version 5.2.0.

Provides the Hardware Abstraction Layer (HAL), device headers, firmware examples, OEM provisioning utilities, certificate generation tools, secure firmware update utilities, and J-Link reference scripts used as the software foundation for the procedures described in this guide.

<https://contentportal.ambiq.com/documents/20123/387752/AmbiqSuite_5.2.0.zip>

**Note**: The references above are provided for technical background and further reading. The provisioning workflow, consolidated single-pass J-Link procedure, host environment configuration, Secure Debug installation procedure, and laboratory validation presented in this guide represent the author's independent engineering work.

All referenced materials remain the property of their respective owners. Access to certain Ambiq resources may require registration or acceptance of the applicable license terms.

## 14. Appendix A: J-Link Utilities

SWO console version start:

```bash
JLinkSWOViewerCLExe -cpufreq 96000000 -swofreq 1000000 -device AP510NFA-CBR -itmport 0 -swoattach 1
```

SWO graphics version start:

```bash
JLinkSWOViewerExe -cpufreq 96000000 -swofreq 1000000 -device AP510NFA-CBR -itmport 0 -swoattach 1
```

SWD start:

```bash
JLinkExe -device AP510NFA-CBR -if SWD -speed 4000 -autoconnect 1
```

## 15. Appendix B – UART Communication Troubleshooting

This appendix provides diagnostic procedures for resolving common UART communication problems encountered during Apollo510 OEM provisioning, Secure Debug Certificate installation, and wired firmware update operations. Perform the following checks before attempting to initiate UART communication with the Secure Bootloader.

### 15.1. Verify USB-UART Port Availability

Verify that the USB-UART adapter is recognized by Ubuntu and that the corresponding serial device exists:

```bash
ls -l /dev/ttyUSB0
```

Example output:

```text
crw-rw---- 1 root dialout 188, 0 ... /dev/ttyUSB0
```

This output confirms that the serial device exists and is assigned to the dialout group.

If the specified device does not exist, check the available serial devices:

```bash
ls -l /dev/ttyUSB* /dev/ttyACM*
```

The operating system may assign a different device name, such as

```text
/dev/ttyUSB1 or /dev/ttyACM0.
```

Use the actual device name in all subsequent UART commands.

### 15.2. Verify USB Adapter Detection

If no corresponding serial device is available, verify that Ubuntu detects the USB-UART adapter:

```bash
lsusb
```

Additional diagnostic information can be obtained from the kernel messages:

```bash
sudo dmesg | tail -n 30
```

If Ubuntu is running inside a virtual machine, verify that the USB-UART adapter is attached to the guest operating system rather than the host operating system.

For VirtualBox, check the USB device assignment using:

**Devices → USB**

Select the corresponding USB-UART adapter and repeat the port availability check.

### 15.3. Verify User Permissions

Check the groups assigned to the current user session:

```bash
groups
```

The output should include the `dialout` group.

Example output:

```text
username adm cdrom sudo dip plugdev dialout
```

If the `dialout` group is missing, add the current user to this group:

```bash
sudo usermod -aG dialout "$USER"
```

Log out of Ubuntu and log back in to apply the updated group membership.

Verify the group membership again:

```bash
groups
```

**Note:** Activating or reactivating the Python virtual environment does not refresh the user's Linux group membership.

### 15.4. Verify Python Serial Port Access

Activate the Python virtual environment:

```bash
source "$HOME/apollo510-env/bin/activate"
```

Verify that the `pySerial` module is available:

```bash
python3 -c "import serial; print(serial.__version__)"
```

Next, verify that Python can successfully open the serial port at the required communication speed:

```bash
python3 -c "import serial; s=serial.Serial('/dev/ttyUSB0',115200,timeout=1); print('PASS: UART port accessible'); s.close()"
```

Expected output:

```text
PASS: UART port accessible
```

This result confirms that Python, running within the configured virtual environment, can successfully open and close the specified serial port.

If a Permission denied error is reported, verify the user permissions as described in Section 15.3.

If a ModuleNotFoundError related to the serial module is reported, verify the Python virtual environment configuration and the pySerial installation.

**Important:** Successful completion of this test confirms access to the serial port. It does not confirm physical UART connectivity or successful communication with the Apollo510 Secure Bootloader.

Perform this test before asserting Boot Override and resetting the target device.

### 15.5. Verify That the Serial Port Is Not in Use

Another application or system service may already be using the serial port.

Check whether any process is accessing the port:

```bash
sudo fuser -v /dev/ttyUSB0
```

If a process is listed, identify it and close the corresponding application before proceeding.

Typical sources of serial port conflicts include terminal emulators, other provisioning utilities, serial monitoring applications, and background services such as ModemManager.

Ensure that the serial port is available exclusively to the UART provisioning utility during the wired update operation.

### 15.6. Verify Physical UART Connectivity

If the serial port is accessible but communication with the target device cannot be established, verify the following hardware connections:

-  USB-UART adapter TX connected to the Apollo510 UART RX.
-  USB-UART adapter RX connected to the Apollo510 UART TX.
-  Common ground connection between the USB-UART adapter and the target device.
-  UART logic voltage levels compatible with the target device I/O supply voltage.
-  Correct UART interface and GPIO assignments.
-  Target device correctly powered.

Verify that the USB-UART adapter uses compatible logic-level signalling rather than conventional RS-232 voltage levels.

### 15.7. Verify Boot Override and UART Communication Timing

The INFO0 configuration used in this guide specifies a wired update timeout of 5000 ms.

To initiate communication with the Secure Bootloader:

1. Prepare the required UART command in the Ubuntu terminal.
2. Assert Boot Override by pulling GPIO93 LOW.
3. Reset the Apollo510 device while GPIO93 remains LOW.
4. Immediately execute the prepared UART command.
5. Verify that the Secure Bootloader responds to the HELLO request.

If the timeout expires before communication is established, repeat the Boot Override and reset sequence.

The expected initial communication is:

```text
Connecting with Apollo510 over serial port /dev/ttyUSB0...
Sending Hello.
Received response for Hello
Received Status
```

Successful reception of the HELLO response confirms that UART communication with the Secure Bootloader has been established.

### 15.8. Troubleshooting Summary

| Symptom | Potential Cause | Recommended Action |
| --- | --- | --- |
| Serial device does not exist | USB adapter not detected or incorrect device name | Verify USB detection and available serial devices |
| USB adapter visible on host but not in Ubuntu | Incorrect virtual machine USB assignment | Attach the USB adapter to the guest operating system |
| Permission denied | Missing serial port access permissions | Verify dialout group membership |
| ModuleNotFoundError: No module named 'serial' | Missing pySerial module or incorrect Python environment | Verify virtual environment and pySerial installation |
| Device or resource busy | Serial port used by another process | Identify and close the conflicting application |
| Python serial port test passes, but no HELLO response is received | Incorrect UART wiring, logic voltage levels, Boot Override configuration, communication settings, or expired timeout | Verify hardware connections and repeat the Boot Override sequence |
| HELLO response received, but subsequent communication fails | Wired update protocol error, communication integrity problem, or device-side rejection | Examine the reported error and communication log |
| NACK response received | Secure Bootloader rejected a command or data | Record the exact NACK code and investigate the corresponding error |
| Certificate download succeeds, but debug access remains unavailable | Certificate processing or Secure Debug authorization issue | Verify certificate validity, SOC-ID, DCU masks, and device initialization |

**Important:** UART port accessibility, successful HELLO communication, successful wired image transmission, and activation of Secure Debug permissions represent separate verification stages. Successful completion of one stage does not guarantee successful completion of the subsequent stages.
