#!/usr/bin/bash

echo "Set of functions to easy the installation of Fedora - AKZA07"

# Useful for dual-boot setups.
# This will tell linux to use the hardware clock
# like how Windows does so they don't override each other
fixClock() {
  echo "Setting system-clock to local"
  sudo timedatectl set-local-rtc 1 --adjust-system-clock
}

setupNode() {
  echo "Installing pnpm"
  curl -fsSL https://get.pnpm.io/install.sh | sh -
  source ~/.config/fish/config.fish
  pnpm env use latest --global
}

setupApps() {
  APPS=(
    'btop'
    'helix'
    'kitty'
    'lazygit'
    'bat'
    'zoxide'
    'fish'
    'tealdeer'
    'eza'
    'wl-clipboard'
    'trash-cli'
  )
  sudo dnf copr enable atim/lazygit -y
  sudo dnf install "${APPS[@]}" -y
}

baseInstall(){
  if [[ $1 == "--secureboot" ]]; then
    sudo dnf update -y
  elif [[ $1 == "--nvidia" ]]; then
    echo "Installing NVIDIA Drivers..."
    # sudo dnf copr enable sunwire/envycontrol -y
    # sudo dnf install python3-envycontrol
    # sudo dnf install akmod-nvidia
    # sudo dnf install xorg-x11-drv-nvidia-cuda
    echo "Waiting for kmod to finish..."
    while true; do
      modinfo -F version nvidia > /dev/null 2>&1
      if [ $? -eq 0 ]; then
        echo "Completed. Initiating reboot"
        sleep 2
        sudo reboot
      else
        echo "In progress, Checking in 3 seconds"
        sleep 3
      fi
    done
  fi
}

if [[ $1 == "--fix-clock" ]]; then
  fixClock
elif [[ $1 == "--install-pnpm" ]]; then
  setupNode
elif [[ $1 == "--install-apps" ]]; then
  setupApps
elif [[ $1 == "--base-install" ]]; then
  baseInstall $2
elif [[ $1 == "--gpu" ]]; then

  if [[ $2 == "power" ]]; then
    watch -n 1 cat /proc/driver/nvidia/gpus/0000:01:00.0/power
  elif [[ $2 == "control" ]]; then
    watch -n 1 /sys/bus/pci/devices/0000:01:00.0/power/control
  elif [[ $2 == "state" ]]; then
    watch -n 1 /sys/class/drm/card0/device/power_state
  elif [[ $2 == "version" ]]; then
    watch -n 1 modinfo -F version nvidia
  else
    echo "Error: Bad Usage"
    echo "$0 --gpu [power, control, state, version]"
    exit
  fi

else
  echo "Error: Bad Usage"
  echo "$0 --setup-node | --fix-clock | --install-apps | --gpu"
fi 

