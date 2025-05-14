# My Lenovo Legion with Linux Notes 
This is some notes and personal observations for dealing with Linux on Legion Devices. If you are looking for a guide to fix all the usual issues, then [cszach/linux-on-lenovo-legion](https://github.com/cszach/linux-on-lenovo-legion) is the place to look for. That repository helped me the most.

## About My Setup (Updated on 2025-05-14 )
- Laptop - Legion Slim 5 16AHP9 (AMD Ryzen 7 8845HS + RTX 4060)
```
Operating System: Fedora Linux 42
KDE Plasma Version: 6.3.5
KDE Frameworks Version: 6.13.0
Qt Version: 6.9.0
Kernel Version: 6.14.5-300.fc42.x86_64 (64-bit)
Graphics Platform: Wayland
Processors: 16 × AMD Ryzen 7 8845HS w/ Radeon 780M Graphics
Memory: 30.6 GiB of RAM
Graphics Processor 1: AMD Radeon Graphics
Graphics Processor 2: NVIDIA GeForce RTX 4060 Laptop GPU
Manufacturer: LENOVO
Product Name: 83DH
System Version: Legion Slim 5 16AHP9
```

## Index

1. [Fix the clock changing Windows 11 Clock](fix-the-clock-changing-windows-11-clock)
2. [Fix the brightness on AMD iGPU](fix-the-brightness-on-amd-igpu)
3. [Setting Up NVIDIA Graphics](setting-up-nvidia-graphics)
4. [Some debugging commands](some-debugging-commands)



## Fix the clock changing Windows 11 Clock
Run this and it'll tell Linux to use the Local Hardware Clock

```bash
sudo timedatectl set-local-rtc 1 --adjust-system-clock
```

## Fix the brightness on AMD iGPU
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

        ```bash
      sudo mokutil --import /etc/pki/akmods/certs/public_key.der
        ```

    <details>
    <summary>How to revert?</summary>    

        ```bash
      sudo mokutil --export #        and it will export the keys to the current directory.
      sudo mokutil --delete <THE FILE>` # will "un-enroll".
      ```
        It's okay even if your linux installation is gone. Using a live USB you can do the same. It's stored in your UEFI Firmware so it's better to not fill the memory with unused keys.
    </details>
    
    ---

    __If it's too confusing, just disable it__
    
    ```bash
      sudo mokutil --disable-validation
    ```

---

### 1. Install NVIDIA Driver.
    
`sudo dnf install akmod-nvidia -y`

`watch -n 1 modinfo -F version nvidia`

Once it prints NVIDIA driver version, reboot. Fans will spin up and it will happen in the background. If you reboot before that, just re-install and wait again.


### 2. Monitoring the GPU

- Monitor Driver using `watch -n 1 cat /proc/driver/nvidia/gpus/0000:01:00.0/power`
- `cat /sys/bus/pci/devices/0000:01:00.0/power/control` should output `auto`
- `watch -n 1 cat /sys/class/drm/card0/device/power_state` will output the current power state. It's ACPI power state so 
    - DO > D1 > D2 > D3 > D3Cold
- D3Cold means it's barely using any power. Which is what you want when you're not using the dGPU. But you will notice that it's stuck on D0 if the power management isn't working properly.

~~**\*\*NOTE\*\* :  At the time of writing, For me, it is not working and is always D0 even when I'm not using it**~~

### 3. Fixing the Power Management

## Some notable mentions
- [4070 / 555 and 560 drivers wont stay in D3cold, Lenovo Legion Slim 5](https://forums.developer.nvidia.com/t/4070-555-and-560-drivers-wont-stay-in-d3cold-lenovo-legion-slim-5/302967)
- My Post - [Hybrid Nvidia 4060 + AMD iGPU, Nvidia Dynamic Power Management not suspending the PCIe GPU - Laptop](https://www.reddit.com/r/Fedora/comments/1gjya64/hybrid_nvidia_4060_amd_igpu_nvidia_dynamic_power/)
- **SOLUTION - [Solving issues with battery time and hybrid mode on MUX Switch laptops (Lenovo Legion 16AHP9 - Ryzen 8845S+RTX4070)](https://www.reddit.com/r/linux/comments/1klrtxv/solving_issues_with_battery_time_and_hybrid_mode/)**

Ideally this should've worked out of the box. But it doesn't. Maybe it's Lenovo or Maybe it's Nvidia. Both are not helpful to us users. They just point fingers and asks us to use support.

- In `/etc/modprobe.d/nvidia-runtimepm.conf` (any name will do except nvidia.conf since many apps tends to already pollute it), add following

    ```
    # Enable Nvidia Direct Rendering Manager & Kernel Mode Setting
    options nvidia-drm modeset=1 
    # Disables backlightHandler, We have AMD iGPU already doing it for US
    # https://download.nvidia.com/XFree86/Linux-x86_64/570.133.07/README/configlaptop.html
    options NVreg_EnableBacklightHandler=0 # Nvidia takes over the Backlight handling after driver installs
    # Let's us take over the control rather than the firmware in GPU
    options nvidia NVreg_EnableGpuFirmware=0
    # https://download.nvidia.com/XFree86/Linux-x86_64/570.133.07/README/dynamicpowermanagement.html
    options nvidia "NVreg_DynamicPowerManagementVideoMemoryThreshold=100"
    options nvidia "NVreg_DynamicPowerManagement=0x02"

    ```

- Run the following commands
    ```bash
    sudo akmods --rebuild --force
    ```
    ```bash
    sudo dracut --regenerate-all --force
    ```

- Reboot

## Some debugging commands
- Check loaded driver version
    ```bash
    modinfo -F version nvidia
    ```
- Check if your driver is the kernel-Open( **Not nouveau or nova) or the Propriatory ones.
    ```bash
    modinfo nvidia | grep license
    ```
- Check everything ( But avoid sharing output to people )
    ```bash
    modinfo nvidia
    ```
- Informatin about your GPU
    ```bash
    cat /proc/driver/nvidia/gpus/0000:01:00.0/information
    ```
- Current supported features related to RTD3 Power management
    ```bash
    cat /proc/driver/nvidia/gpus/0000:01:00.0/power
    ```
- Watch the dGPU power state. D3cold is low power/suspended D0 is Battery sipper
    ```bash
    watch -n 1 cat /sys/class/drm/card0/device/power_state
    ```
- Watch current status of the dGPU
    ```bash
    watch -n 1 cat /sys/bus/pci/drivers/nvidia/0000:01:00.0/power/runtime_status
    ```

## Already have Nvidia drivers installed?

Run these one by one

```bash
sudo dnf remove *nvidia*
sudo dracut --regenerate-all --force
sudo rm /etc/rpm/macros.nvidia-kmod
```
restart and install drivers from scratch
