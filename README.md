# congenial-barnacle
Running Windows10 with Qemu from a physical hard drive

This is going to be short and sweet. 

Tested with SuSe 12 sp3, kvm-host install
Tested with Manjaro-xfce-17.0.6-stable-x86_64
  -Pacakged required for Manjaro
  -pacman -S qemu
  -pacman -S qemu-block-iscsi
  -pacman -S libvirt

Enviroment 
/dev/sda - Base Windows 10 install
/dev/sdb - SuSe 12 sp3 install



Required Files
Virtio Drivers for Windows
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
UEFI Bios file
https://sourceforge.net/projects/edk2/files/OVMF/OVMF-X64-r15214.zip

In Linux enable Iommu support 
  # edit /etc/default/grub
    - Add intel_iommu=on to the end of GRUB_CMDLINE_LINUX_DEFAULT
    # And then automatically re-generate the grub.cfg file with:
      - grub-mkconfig -o /boot/grub/grub.cfg
      - reboot
      # Verify Iommu support is enabled
        - dmesg | grep iommu

Installation
  - Install Windows 10 to hard drive (or use and existing installation)
  - Boot into Windows to install Virtio Drivers
  - Extract virtio-win.iso 
  # Navigate into each folder then into each w10 subfolder, right-click the inf and select Install, 
      - virtio-win-0.1.141\Balloon\w10\amd64\balloon.inf
      - virtio-win-0.1.141\NetKVM\w10\amd64\netkvm.inf
      - virtio-win-0.1.141\pvpanic\w10\amd64\pvpanic.inf
      - virtio-win-0.1.141\qemufwcfg\w10\amd64\qemufwcfg.inf
      - virtio-win-0.1.141\qemupciserial\w10\amd64\qemupciserial.inf
      - virtio-win-0.1.141\qxldod\w10\amd64\qxldod.inf
      - virtio-win-0.1.141\vioinput\w10\amd64\vioinput.inf
      - virtio-win-0.1.141\viorng\w10\amd64\viorng.inf
      - virtio-win-0.1.141\vioscsi\w10\amd64\vioscsi.inf
      - virtio-win-0.1.141\vioserial\w10\amd64\vioser.inf
      - virtio-win-0.1.141\viostor\w10\amd64\viostor.inf
  # Install the qemu guest-agent 
      - virtio-win-0.1.141\guest-agent\qemu-ga-x64.msi
      
(You can inject viostor.inf into Windows with dism from a booted WinPE USB. - Will update with dism command at a later time.)

Reboot back into your Linux drive
  - Download and extract the UEFI Bios
  - https://sourceforge.net/projects/edk2/files/OVMF/OVMF-X64-r15214.zip
  - rename OVMF.fd --> bios.bin
  
From a terminal run the following command
 - qemu-system-x86_64 -drive file=pflash,format=raw,file=bios.bin -drive if=pflash,format=raw,readonly,file=bins.bin -drive file=/dev/sda,media=disk,format=raw,snapshot=off,cache=none -enable-kvm -cpu host -m 8G

Networking
  - You will need to install the bridge utilities for your specific Linux OS, create a bridge and add it to your qemu-system-x86_64 command
  # SuSe
    - https://www.suse.com/documentation/sles11/book_kvm/data/cha_qemu_running_networking.html
  # Manjaro
    - https://wiki.archlinux.org/index.php/Network_bridge
  # Or create a tap device
    - https://wiki.archlinux.org/index.php/QEMU#Tap_networking_with_QEMU
