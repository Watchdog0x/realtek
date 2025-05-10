# rtw88 

## Compatibility
Compatible with **Linux kernel versions 5.4 and newer** as long as your distro hasn't modified any kernel APIs. RHEL and all distros based on RHEL will have modified kernel APIs and are unlikely to be compatible with this driver.


#### Supported Chipsets
- **PCIe**: RTL8723DE, RTL8814AE, RTL8821CE, RTL8822BE, RTL8822CE
- **SDIO**: RTL8723CS, RTL8723DS, RTL8821CS, RTL8822BS, RTL8822CS
- **USB** : RTL8723DU, RTL8811AU, RTL8811CU, RTL8812AU, RTL8812BU, RTL8812CU
- **USB** : RTL8814AU, RTL8821AU, RTL8821CU, RTL8822BU, RTL8822CU

---

## Installation Guide

### Prerequisites 📋
Below are prerequisites for common Linux distributions __before__  installing this driver.

#### Ubuntu
```bash
sudo apt update && sudo apt upgrade
```
```bash
sudo apt install linux-headers-generic build-essential git
```

#### Fedora
```bash
sudo dnf update
```
```bash
sudo dnf install kernel-devel git
```

#### openSUSE
```bash
sudo zypper install make gcc kernel-devel kernel-default-devel git libopenssl-devel
```

#### Arch
```bash
sudo pacman -Syu
```
```bash
sudo pacman -Sy base-devel git linux-firmware
```
Remember to install the corresponding [kernel headers](https://archlinux.org/packages/?q=Headers+and+scripts+for+building+modules) which is also needed for compilation.

#### Raspberry Pi OS
```bash
sudo apt update && sudo apt upgrade
```
```bash
sudo apt install -y raspberrypi-kernel-headers build-essential git
```

---

### Installation Using DKMS 🔄
It's highly recommended to install this driver via DKMS especially Secure Boot is enabled on your machine. Using DKMS (Dynamic Kernel Module Support) ensures that the `rtw88` kernel modules are automatically rebuilt and re-signed whenever the Linux kernel is updated. Without DKMS, these drivers would stop working after each kernel update, requiring manual re-compilation and re-signing. DKMS should be available through your distribution’s package manager. You can learn more about DKMS [here](https://github.com/dell/dkms).

1. Install `dkms` and all its required dependencies using your preferred package manager.

2. Clone the `rtw88` GitHub repository
   ```
   git clone https://github.com/lwfinger/rtw88
   ```

3. Build, sign and install the rtw88 driver
   ```
   cd rtw88
   ```
   ```
   sudo dkms install $PWD
   ```

4. Install the firmware necessary for the rtw88 driver
   ```
   sudo make install_fw
   ```

5. Enroll the MOK (Machine Owner Key), this is needed **ONLY IF** Secure Boot is enabled on your machine.
   ```
   sudo mokutil --import /var/lib/dkms/mok.pub
   ```
   For Ubuntu-based distro users, run this command instead
   ```
   sudo mokutil --import /var/lib/shim-signed/mok/MOK.der
   ```
   
   Note: At this point, you will be requested to enter a password. Remember this password and re-enter it after rebooting your system in order to enroll your new MOK into your system's UEFI. Please see [this tutorial](https://github.com/dell/dkms?tab=readme-ov-file#secure-boot) for more details.


---

### Installation Using make🛠

You will need to rebuild and reinstall the driver manually after each kernel updates if you choose this way to install the driver. This method is **NOT RECOMMENDED** for systems with Secure Boot enabled.

```bash
git clone https://github.com/lwfinger/rtw88
```
```bash
cd rtw88
```
```bash
make
```
```bash
sudo make install
```
```bash
sudo make install_fw
```
---

### Installation for Arch-based Distros 🛠

This is the best way for Arch-based distro users to install this driver, one more step is required after running `makepkg -si` if Secure Boot is enabled on your machine: Enroll the MOK. Please see the step 5 in [Installation Using DKMS](#installation-using-dkms-) for details.

```bash
git clone https://aur.archlinux.org/rtw88-dkms-git.git
```
```bash
cd rtw88-dkms-git
```
```bash
makepkg -si
```
---

## Important Information
Below is important information for using this driver.

### 1. Blacklisting 🚫
A file called `rtw88.conf` will be installed into `/etc/modprobe.d`. It will blacklist all in-kernel rtw88 drivers, however, it will not blacklist out-of-kernel vendor drivers. You will need to uninstall any out-of-kernel vendor drivers that you have installed that may conflict.

### 2. Recovery Problems After Sleep/Hibernation 🛌
Some BIOSs have trouble changing power state from D3hot to D0. If you have this problem, run 

``` bash
sudo cp reload_rtw88.sh /usr/lib/systemd/system-sleep/
```

That script will unload the driver before sleep or hibernation, and reload it following resumption.

### 3. How To Load/Unload Kernel Modules (aka Drivers)🪛

The kernel modules for RTL8821CU are used to demonstrate here, you need to change the module name according to the chip that your Wi-Fi adapter uses.

This unloads the modules for RTL8821CU, due to some pecularities in the modprobe utility, two steps are required. If you are not sure the names of the loaded modules for your Wi-Fi adapter, run `lsmod | grep -i rtw`
```
sudo modprobe -r rtw_8821cu
sudo modprobe -r rtw_core

# This command will show nothing because the modules for RTL8821CU were unloaded.
lsmod | grep -i rtw
```

This loads the modules for RTL8821CU, only a single modprobe call is required.
```
sudo modprobe rtw_8821cu

# This command will show that the modules for RTL8821CU were loaded.
lsmod | grep -i rtw
rtw_8821cu             16384  0
rtw_usb                36864  1 rtw_8821cu
rtw_8821c              98304  1 rtw_8821cu
rtw_core              294912  2 rtw_usb,rtw_8821c
mac80211             1220608  2 rtw_usb,rtw_core
cfg80211             1064960  2 rtw_core,mac80211
```

### 4. How To Update The Driver Installed via DKMS

1. Remove the installed rtw88 drivers completely, please see [Q1](#q1-how-to-remove-this-driver-if-it-doesnt-work-as-expected) in Q&A for details.

2. Run this command in the rtw88 source directory to pull the latest code 
   ```
   git pull
   ```

3. Build, sign, install the rtw88 driver from the latest code
   ```
   sudo dkms install $PWD
   ```
---

## Q&A

### Q1: How to remove this driver if it doesn't work as expected?

For users who installed this driver via DKMS, run
```
sudo dkms remove rtw88/0.6 --all
```
```
sudo rm -rf /var/lib/dkms/rtw88
```
```
sudo rm -rf /usr/src/rtw88-0.6
```
```
sudo rm -f /etc/modprobe.d/rtw88.conf
```

For users who installed this driver via `make`, run this command in the rtw88 source directory and then the rtw88 driver will be unloaded and removed.

```
sudo make uninstall
```

For Arch-based distro users, run

```
sudo pacman -Rn rtw88-dkms-git
```

