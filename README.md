# Linux EFI Loader
Simple EFI bootloader for Linux kernels written in C. For Macbian and other distributions.

### Project goals
* Support all modern Linux kernels
* Boot Linux kernels from the non-ESP (`systemd-boot` eat your heart out)
* Secure Boot signing support (WITHOUT using a shim)
* Load and use an EFI file system driver to find the kernel
* Boot a Linux kernel **traditionally** (without EFI Stub) to allow for booting a 64-bit kernel on a hybrid EFI system
* Chainload additional EFI applications (Windows Boot Manager, EFI Shell, etc)
* Small interactive shell
* Modular design (e.g. can use config file or hardcode, silent/verbose boot, Macbian OS-bundle support)

### Building
Dependencies: gcc, make, gnu-efi, cmake

```shell
git clone https://github.com/macbian-linux/LinuxEFILoader
cd LinuxEFILoader
mkdir build && cd build
cmake ..
make
```

