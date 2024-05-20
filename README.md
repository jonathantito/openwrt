![OpenWrt logo](include/logo.png)

OpenWrt Project is a Linux operating system targeting embedded devices. Instead
of trying to create a single, static firmware, OpenWrt provides a fully
writable filesystem with package management. This frees you from the
application selection and configuration provided by the vendor and allows you
to customize the device through the use of packages to suit any application.
For developers, OpenWrt is the framework to build an application without having
to build a complete firmware around it; for users this means the ability for
full customization, to use the device in ways never envisioned.

Sunshine!

## Download

Built firmware images are available for many architectures and come with a
package selection to be used as WiFi home router. To quickly find a factory
image usable to migrate from a vendor stock firmware to OpenWrt, try the
*Firmware Selector*.

* [OpenWrt Firmware Selector](https://firmware-selector.openwrt.org/)

If your device is supported, please follow the **Info** link to see install
instructions or consult the support resources listed below.

## 

An advanced user may require additional or specific package. (Toolchain, SDK, ...) For everything else than simple firmware download, try the wiki download page:

* [OpenWrt Wiki Download](https://openwrt.org/downloads)

## Development

To build your own firmware you need a GNU/Linux, BSD or macOS system (case
sensitive filesystem required). Cygwin is unsupported because of the lack of a
case sensitive file system.

### Requirements

You need the following tools to compile OpenWrt, the package names vary between
distributions. A complete list with distribution specific packages is found in
the [Build System Setup](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)
documentation.

```
binutils bzip2 diff find flex gawk gcc-6+ getopt grep install libc-dev libz-dev
make4.1+ perl python3.7+ rsync subversion unzip which
```

### Quickstart

1. Run `./scripts/feeds update -a` to obtain all the latest package definitions
   defined in feeds.conf / feeds.conf.default

2. Run `./scripts/feeds install -a` to install symlinks for all obtained
   packages into package/feeds/

3. Run `make menuconfig` to select your preferred configuration for the
   toolchain, target system & firmware packages.

   For example if you are working with the instruction set [mipsel_24kc](https://openwrt.org/docs/techref/instructionset/mipsel_24kc#devices_with_this_instructionset) you will find the following information as example:

   Package architecture	Target	Subtarget	Brand	  Model	        Version

    mipsel_24kc	        ramips	  mt76x8	GL.iNet	GL-MT300N	    v2
	
	#### IMPORTANT NOTE: Building openssh-sftp-server ipk package locally for the instruction set choosen can show errors but always look for your ipk package in:
		
		'/workspaces/openwrt/bin/packages/mipsel_24kc/packages/openssh-sftp-server_9.7p1-r2_mipsel_24kc.ipk'
	
	IT IS BETTER TO CHOOSE ALL FROM MAIN BRANCH IN A DEVCONTAINER, IT DOESN MATTER IF IT SHOWS YOU AN ERROR

   So your target system will be:
   
     21. MediaTek Ralink MIPS (TARGET_ramips)

  Your subtarget will be:

     3. MT76x8 based boards (TARGET_ramips_mt76x8) (NEW)

  And your target profile will be:

     18. GL.iNet GL-MT300N V2 (TARGET_ramips_mt76x8_DEVICE_glinet_gl-mt300n-v2) (NEW)

In the OpenWrt build system, the TARGET_ROOTFS_INITRAMFS option **controls whether the built firmware image should include an initial RAM disk (initramfs) or not.**

An **initramfs is a temporary root file system that is loaded into memory by the kernel during the boot process. It contains a minimal set of utilities and scripts necessary to mount the actual root file system from a disk or other storage medium**.

When you see the TARGET_ROOTFS_INITRAMFS option during the configuration process (make menuconfig), it provides the following choices:

*N (No): This means that the firmware image will not include an initramfs. The kernel will directly mount the root file system from the specified storage device (e.g., flash memory, USB drive, etc.).* I will choose this one because `cat /proc/cmdline`, `mount` and `lsmod` doesnt show me any initramfs.

y (Yes): This option enables the inclusion of an initramfs in the firmware image. The initramfs will be loaded into memory by the kernel during boot, and it will be responsible for mounting the actual root file system.

? (Help): This option displays additional information and explanations about the TARGET_ROOTFS_INITRAMFS option.

A cpio (copy input/output) archive is a file format that stores a collection of files, directories, and other filesystem data. The cpio.gz format specifically refers to a cpio archive that has been compressed using gzip compression.

When you see the TARGET_ROOTFS_CPIOGZ option during the configuration process (make menuconfig), it provides the following choices:

*N (No): This means that the root filesystem will not be packaged as a cpio.gz archive. The root filesystem data will be included in the firmware image in its regular format.*

y (Yes): This option enables the packaging of the root filesystem as a compressed cpio.gz archive. The archive will be included in the firmware image.
? (Help): This option displays additional information and explanations about the TARGET_ROOTFS_CPIOGZ option.

On my own device I had installed `opkg install tune2fs` and the the command `tune2fs -l /dev/sda2 | grep 'Block size'` gives me:

Block size:               4096


If you enable CONFIG_TARGET_EXT4_JOURNAL (y), it means that when you create an ext4 filesystem using OpenWrt, it will include journaling support. **Journaling filesystems maintain a journal (a special file) that records changes that are intended for the filesystem metadata and data blocks. This helps in ensuring the integrity of the filesystem, particularly in the event of sudden power loss or system crash.**

In simpler terms, when CONFIG_TARGET_ROOTFS_PERSIST_VAR is enabled (y), it means that the contents of the /var directory will persist across reboots. By default, OpenWrt creates symbolic links (symlinks) for certain directories, such as /var, which point to locations in temporary storage (/tmp). This means that any changes made in /var during a session are lost upon reboot.

You would choose to enable CONFIG_TARGET_EXT4_JOURNAL if you want the benefits of journaling, such as improved reliability and faster recovery after unexpected shutdowns or crashes. However, enabling journaling does impose some performance overhead, so if performance is critical and the risk of sudden power loss or crashes is low, you might choose to disable it.

When you encounter this configuration option during the kernel build process, you will be prompted to provide a custom name. This name can be anything you choose, such as your username or any identifier you prefer. It's often used to personalize or identify the kernel build, especially in multi-user or collaborative environments. However, it's optional, and if you leave it empty, the default value might be used or it might not include any custom user name in the kernel build information.


4. Run `make` to build your firmware. This will download all sources, build the
   cross-compile toolchain and then cross-compile the GNU/Linux kernel & all chosen
   applications for your target system.

### Related Repositories

The main repository uses multiple sub-repositories to manage packages of
different categories. All packages are installed via the OpenWrt package
manager called `opkg`. If you're looking to develop the web interface or port
packages to OpenWrt, please find the fitting repository below.

* [LuCI Web Interface](https://github.com/openwrt/luci): Modern and modular
  interface to control the device via a web browser.

* [OpenWrt Packages](https://github.com/openwrt/packages): Community repository
  of ported packages.

* [OpenWrt Routing](https://github.com/openwrt/routing): Packages specifically
  focused on (mesh) routing.

* [OpenWrt Video](https://github.com/openwrt/video): Packages specifically
  focused on display servers and clients (Xorg and Wayland).

## Support Information

For a list of supported devices see the [OpenWrt Hardware Database](https://openwrt.org/supported_devices)

### Documentation

* [Quick Start Guide](https://openwrt.org/docs/guide-quick-start/start)
* [User Guide](https://openwrt.org/docs/guide-user/start)
* [Developer Documentation](https://openwrt.org/docs/guide-developer/start)
* [Technical Reference](https://openwrt.org/docs/techref/start)

### Support Community

* [Forum](https://forum.openwrt.org): For usage, projects, discussions and hardware advise.
* [Support Chat](https://webchat.oftc.net/#openwrt): Channel `#openwrt` on **oftc.net**.

### Developer Community

* [Bug Reports](https://bugs.openwrt.org): Report bugs in OpenWrt
* [Dev Mailing List](https://lists.openwrt.org/mailman/listinfo/openwrt-devel): Send patches
* [Dev Chat](https://webchat.oftc.net/#openwrt-devel): Channel `#openwrt-devel` on **oftc.net**.

## License

OpenWrt is licensed under GPL-2.0
