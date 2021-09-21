# Reference: 
https://gist.github.com/billti/d904fd6124bf6f10ba2c1e3736f0f0f7

------------------------

Below are the steps to get an ARM64 version of Ubuntu running in the QEMU emulator on Windows 10.

## Install QEMU
Install for Windows from <https://qemu.weilnetz.de/w64/> (I used `qemu-w64-setup-20181211.exe`)

Put `C:\Program Files\qemu` on your PATH, and run the below to check it's working (which will list out
the CPUs the AArch64 emulator can emulate):

```
qemu-system-aarch64 -M virt -cpu help
```

## Copy the Firmware and OS images
Into your working directory...

 - Copy the `ubuntu-16.04-server-cloudimg-arm64-uefi1.img` OS image from <https://cloud-images.ubuntu.com/releases/16.04/release/>
 - Copy the `QEMU_EFI.fd` firmware image from <https://releases.linaro.org/components/kernel/uefi-linaro/latest/release/qemu64/>

## Create the configuration data image
The Ubuntu server images require configuration data be provided as an image, such as setting auth credentials.

The tool used to convert the config text file into an image file only runs on Linux, so I've attached a `user-data.img`
file (and the text file used to create it) in a [zip file to this Gist](https://gist.github.com/billti/d904fd6124bf6f10ba2c1e3736f0f0f7/raw/2ba5c964c0be0042b842cd44c2c0a19cdd61c9c4/user-data.zip).
Extract the `user-data.img` file to the working directory.

The `user-data.img` file was created for password authentication as outlined in <https://stackoverflow.com/a/53373376>

## Launch the emulator from the working directory

Run the below to boot the image, you will some some benign errors at startup. Wait until the output settles down (even after you see the login prompt, as the post-boot config may not have completed yet).

```
qemu-system-aarch64 -m 2048 -cpu cortex-a72 -smp 4 -M virt -nographic -bios QEMU_EFI.fd -drive if=none,file=ubuntu-16.04-server-cloudimg-arm64-uefi1.img,id=hd0 -device virtio-blk-device,drive=hd0 -drive file=user-data.img,format=raw -device virtio-net-device,netdev=net0 -netdev user,hostfwd=tcp:127.0.0.1:2222-:22,id=net0
```

To break down these lines:

 - `qemu-system-aarch64 -m 2048 -cpu cortex-a72 -smp 4 -M virt -nographic` - run the ARM64 virtual platform emulator with 2GB RAM and 4 Cortex-A72 cores with no GUI support.
 - `-bios QEMU_EFI.fd` - use the firmware downloaded above.
 - `-drive if=none,file=ubuntu-16.04-server-cloudimg-arm64-uefi1.img,id=hd0` - use the Ubuntu image file
 - `-device virtio-blk-device,drive=hd0` - mount drive from above as a block device
 - `-drive file=user-data.img,format=raw` - use the configuration data image file
 - `-device virtio-net-device,netdev=net0` - create a virtual network device
 - `-netdev user,hostfwd=tcp:127.0.0.1:2222-:22,id=net0` - set up the networking stack and forward the SSH port

Then from a good Terminal emulator (I recommend the new Windows Terminal app with one of the [Powerline fonts](https://github.com/powerline/fonts)) you can connect over SSH with the below, and the configured password (`asdfqwer`):

```
ssh ubuntu@localhost -p 2222
```


## ===========================

Expanding the image size by 8GB

- shutdown the vm
- qemu-img resize ubuntu-16.04-server-cloudimg-arm64-uefi1.img +8G
- start the vm, ssh into it and enter the following command
- sudo growpart /dev/vda 1
- reboot
