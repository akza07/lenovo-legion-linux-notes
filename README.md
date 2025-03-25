# My Lenovo Legion with Linux Notes 
This is some notes and personal observations for dealing with Linux on Legion Devices. If you are looking for a guide to fix all the weird quirks, then [cszach/linux-on-lenovo-legion](https://github.com/cszach/linux-on-lenovo-legion) is the place to look for. That repository helped me the most.

## About My Setup ( 2024-12-31 )
- Laptop - Legion Slim 5 16AHP9 (AMD Ryzen 7 8845HS + RTX 4060)
```
Operating System: Fedora Linux 41
Kernel Version: 6.12.6-200.fc41.x86_64 (64-bit)
Graphics Platform: Wayland
Processors: 16 × AMD Ryzen 7 8845HS w/ Radeon 780M Graphics
Memory: 30.6 GiB of RAM
Graphics Processor: AMD Radeon Graphics
Manufacturer: LENOVO
Product Name: 83DH
System Version: Legion Slim 5 16AHP9
```
**NOTE: All of these are also automated into a bash script available [here](here)**

## Fix the clock changing when booting to Windows
Run this and it'll tell Linux to use the Local Hardware Clock

`sudo timedatectl set-local-rtc 1 --adjust-system-clock`

## Fix the brightness on AMD GPU
The backlight issue mentioned in the [cszach/linux-on-lenovo-legion](https://github.com/cszach/linux-on-lenovo-legion) didn't work for me.
According to this thread [» [SOLVED] High laptop power usage (Legion slim 5 gen 9)](https://bbs.archlinux.org/viewtopic.php?id=300872)
`acpi_backlight=native` needs to be added to the boot params.

Test it first by adding it during the grub menu by pressing <kbd>E</kbd> and adding it after say `quiet` arg. 

You can check your current one by running

`cat /proc/cmdline`

and it would output something like

```bash
➜  ~ cat /proc/cmdline
BOOT_IMAGE=(hd0,gpt1)/vmlinuz-6.13.8-200.fc41.x86_64 root=UUID=4f15a407-b8f7-4ac6-b873-ed57d5536114 ro rhgb acpi_backlight=native quiet
```

To make it persistent

```bash
sudo grubby --args="acpi_backlight=native" --update-kernel=ALL
```

## Setting Up NVIDIA Graphics

### With Secureboot enabled

I'm using Fedora because they have their guide is easier to work with than ArchWiki for me and I have custom python scripts to automate most stuff for Fedora.

- Setup SecureBoot 
Follow the instructions from [RPM Fusion - HowTo/Secureboot](https://rpmfusion.org/Howto/Secure%20Boot)
    - Import the key to EFI Firmware by running the following.

        `sudo mokutil --import /etc/pki/akmods/certs/public_key.der`

    <details>
    <summary>How to revert?</summary>    

        `sudo mokutil --export`
        and it will export the keys to the current directory.
        `sudo mokutil --delete <THE FILE>` will "un-enroll".

        It's okay even if your linux installation is gone. Using a live USB you can do the same. It's stored in your UEFI Firmware so it's better to not fill the memory with unused keys.
    </details>
    
    ---

    __If it's too confusing, just disable it__
    
    `sudo mokutil --disable-validation`

---

### 1. Install NVIDIA Driver.
    
`sudo dnf install akmod-nvidia -y`

`watch -n 1 modinfo -F version nvidia`

Once it prints NVIDIA driver version, reboot. Fans will spin up and it will happen in the background. If you reboot before that, just re-install and wait again.

_Since we have a Hybrid-GPU configuration, By default it should be using our iGPU. So technically the dGPU should be turned off or on Low Power state. But at the time of writing this, that's kind of broken. Which is the reason I'm writing and documenting this._

### 2. Monitoring the GPU

- Monitor Driver using `watch -n 1 cat /proc/driver/nvidia/gpus/0000:01:00.0/power`
- `cat /sys/bus/pci/devices/0000:01:00.0/power/control` should output `auto`
- `watch -n 1 cat /sys/class/drm/card0/device/power_state` will output the current power state. It's ACPI power state so 
    - DO > D1 > D2 > D3 > D3Cold
- D3Cold means it's barely using any power. Which is what you want when you're not using the dGPU. But you will notice that it's stuck on D0 if the power management isn't working properly.

**\*\*NOTE\*\* :  At the time of writing, For me, it is not working and is always D0 even when I'm not using it**

### 3. Fixing the Power Management

[Refer to this discussion](https://forums.developer.nvidia.com/t/4070-555-and-560-drivers-wont-stay-in-d3cold-lenovo-legion-slim-5/302967)

This should be the default but for some reason, it didn't happen.

- In `/etc/modprobe.d/nvidia.conf `, add following

    ```
    options nvidia_drm modeset=1 fbdev=1
    options nvidia NVreg_DynamicPowerManagementVideoMemoryThreshold=0
    ```

- In `/etc/udev/rules.d/nvidia.rules`
 
    ```
    # Remove NVIDIA USB xHCI Host Controller devices, if present
    ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c0330", ATTR{remove}="1"

    # Remove NVIDIA USB Type-C UCSI devices, if present
    ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c8000", ATTR{remove}="1"
    ```
- Reboot


**\*\*NOTE\*\* : As mentioned in the discussion, Changing Legion Power Profiles or Unplugging while charging etc will Switch to D0 and eat the battery. So restarting at that time is only the way.**
