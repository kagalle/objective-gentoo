
##############
Gentoo Install
##############
The Objective Gentoo Project
----------------------------

\

.. include:: <isonum.txt>

+------------+----------------------------------------------------------------------+
|:Author:    | Kenneth Galle                                                        |
+------------+----------------------------------------------------------------------+
|:Date:      | 12/18/2025                                                           |
+------------+----------------------------------------------------------------------+
|:Version:   | v1.0.0                                                               |
+------------+----------------------------------------------------------------------+
|:License:   || |copy| 2003 by Kenenth Galle                                        |
|            || Use allowed with the bounds of the GNU AGPLv3                       |
|            || No permission granted for use within machine learning (AI) systems. |
+------------+----------------------------------------------------------------------+

----------------------------------------------------------
Notes for installing Gentoo as a basic desktop environment
----------------------------------------------------------


This guide is meant to be used along with the official `Gentoo handbook <https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation>`_ and other Gentoo documentation (e.g. `Gentoo mirrors <https://www.gentoo.org/support/rsync-mirrors>`_). There is no Gentoo installer and the handbook is hard for me to navigate just to get a machine up and running.

As the handbook is very detailed, it is easy to "miss the forest in the trees." This guide is meant to limit installation choices and hide some of the complexity and verbosity of the handbook to ease the process, so that important details are not overlooked.

Using the Gentoo install media to install the system, but note that a current SystemResuceCD (on a flash-drive) can be very helpful, and can also be used to download and create the Gentoo install media as described below.

The details of partitioning the drive using `gdisk` are left out of this guide.

Commands shown are **examples** and should be edited if needed.

* `ken` is the username of the regular user
* `claude` is the name of the machine being setup
* `birchtreefarm.internal` is the name of the local network on the LAN

The steps do not need to be done in one session, start to finish. However if you are anywhere between `Boot the live-CD`_ and `Bootloader` you will need to redo all steps marked with ``***`` up to the point where you left off.

-----------
Constraints
-----------

* x86-64 (AMD64 or Intel 64)

  This is the most common PC environment with 64 bit support. It also includes Apple Mac machines that use Intel processors, however to install there you need to do additional initial steps to the hardware itself.

* Use UEFI to boot (as apposed to BIOS or Compatibility Mode)

  Most recent hardware, within say 10 years or so should support booting using UEFI. Even more recent hardware only supports booting that way. The procedure for installing to hardware that only supports BIOS would differ somewhat and isn't included to improve clarity.

* Use rEFInd as a boot manager (as apposed to trying to boot directly from UEFI or using Grub)

  I attempted to create this guide using EFI stub booting, ie booting directly from the hardware into Linux without a boot manager. However it proved to be needlessly complex to configure and rEFInd is a light-weight alternative to Grub that seems to work well for this.

* Install media

  Create the Gentoo-minimal-install thumb-drive using an existing Linux desktop with a browser available. Other bootable media can be used, but Gentoo's media makes the process more straight forward. It is worth using this method. Also, see the note above about using SystemResuceCD as the existing Linux desktop.

* Downloads from my nearest mirror site

  The biggest problem with mirror lists is that it isn't always clear where a mirror is located, and in some cases a site can be co-located in many locations. There is additional software that can help choosing, but my approach is to search for a university nearest your location.

* Plain GPT partitions

  Create partitions with ``/`` (root), ``/boot`` (including efi), swap (and optionally ``/home``). This is apposed to using any sort of raid or LVM.

  GPT goes with UEFI. GPT is not supported with BIOS. MBR partitioning was used for BIOS (and optionally with compatibility mode) instead. GPT works well and has fewer restrictions and more features.

* OpenRC (as apposed to using `systemd` for the `init` system)

  This a "sensitive" topic - just check the Internet for articles over the last 10 years or more. My success with `systemd` has been less positive than without it, and so I choose to avoid it. This is based strictly my personal opinion. Systemd is a set of software that has enveloped much of the Linux system software. Note that the `init` process is only one area affected by `systemd`. Other areas, especially related to the desktop environment (e.g. elogind) are also `systemd` related, but usable outside of that environment. There are other examples like audio support `pulseaudio`. When I started with Linux, `systemd` didn't exist.

* Desktop profile and matching Stage3 file

  Choice of a profile makes a big difference on how large the final system is and how much software is installed by default. However, for a desktop machine, it is best to use the desktop profile, as all that additional software makes for a lot less manual configuration and a much smoother experience. It is well worth if you want to use a desktop environment, like in this case, `xfce4`.

* Use the pre-built Gentoo binary kernel

  Building a kernel, which is essentially what "Linux" is, can result in a much smaller and faster kernel. However the process is long and involved, and also highly specific to the hardware you're using. Just determining what specific hardware is baked into your hardware can be a significant challenge, and then finding the kernel configuration to match it along with side-effect configuration is not straight forward. I wish there was software that would allow you to scan your hardware and output a tailored kernel configuration - I have not come across any that I would consider a solution for this.

* DHCP used for networking

  The Gentoo install process requires connection to the Internet. Using DHCP is a well supported and straight-forward default. If something else is needed, it can be used instead (e.g. dial-up is mostly a thing of the past).

* Install binary packages whenever possible

  Gentoo by default is a source-based distribution, where all of the software you install (outside of the software you get with the Stage3 file) is built from source. This doesn't fit well with the general purpose "I just need to get things done - machine," and binary support in Gentoo is good once configured.

* The choice of system logger, cron, time sync

  These are pieces of system software that run in the background. You can choose between different options. The choices below are a reasonable default.

------------------------------------------------

Create bootable media
=====================

Using an existing Linux installation (or a machine that is started using SystemRescueCD or similar), create a removable thumb-drive that can boot the destination machine. This live-CD environment will contain the scripts and utilities needed to prepare the destination machine to boot Gentoo directly. Existing machines with other operating systems can be used to create this live-CD, but the steps would be somewhat different. The name "live-CD" matches the assigned host-name when the system is running - it can be any sort of removable media, like a CD-ROM or a flash-drive.
 
#. Manually find your nearest mirror - go to ``https://www.gentoo.org/downloads/mirrors/`` and make note of the https:// URL of the mirror you choose.

#. In a browser, go to the nearest mirror

   ::

    http://mirrors.rit.edu/gentoo/releases/amd64/autobuilds/current-install-amd64-minimal/

#. Download latest `.iso` and its `.asc` file. There will be several older `.asc` files along with the current `.iso` and `.iso.asc`, which are the ones you want (right click, save link as...).

   ::

    install-amd64-minimal-20251116T161545Z.iso
    install-amd64-minimal-20251116T161545Z.iso.asc


#. Verify the `.iso` file

   ::

    gpg --auto-key-locate=clear,nodefault,wkd --locate-key releng@gentoo.org
    gpg --verify install-amd64-minimal-20251116T161545Z.iso.asc

#. Insert thumb drive and find its name - make **absolutely sure** you have the correct drive

   ::

    sudo dmesg | grep "\(sd[a-z]\)\|scsi [0-9]"

#. Write the iso image

   ::

    sudo dd if=install-amd64-minimal-20251116T161545Z.iso of=/dev/sdb bs=4096 status=progress
    sudo sync

#. Verify data on thumb-drive. Remove and reinsert the thumb-drive first. Get size of file in bytes from ``ls``, use that in ``cmp`` which should return without any response (if the data on the drive doesn't match, it will tell you).

   ::

    ls -l install-amd64-minimal-20251116T161545Z.iso
    sudo cat /dev/sdb | cmp --bytes 813670400 install-amd64-minimal-20251123T153051Z.iso

Boot the live-CD
================

#. BIOS setup

   Boot the machine and enter the BIOS setup (by pressing Del, F1, F10, Esc, etc. when initial screen displays)

   a. Change the settings for legacy BIOS boot to support UEFI only. We want to use UEFI and doing this will remove all of the legacy boot options from the boot menu selection, which we don't want to be booting anyway.
   #. Disable secure boot. It may be named that specifically, or it may be implied by a setting for "Windows" vs "Other OS". Secure boot only works for the "Windows" setting, so choosing other-OS can be used to disable secure boot. You can move to secure boot later, but disable it now to limit issues.

#. Boot the thumb-drive \***

   Use whatever BIOS keystroke is needed (Del, F1, F10, Esc, etc. - some machines will have a F-key assigned to get to the boot menu directly, others have override boot options in the "Exit..." menu of setup. Choose the option to boot the thumbdrive.

#. Configure networking \***

   Configure either wireless or wired. A wired connection is easier and generally faster if it is available.

   ::

    ip address
    ip route
    cat /etc/resolv.conf

    See if you already have an ip address, a default route, and name resolution. If not then...

   a. Wireless

      Get name of wireless adapter, enable and configure using Wifi ESSID and password, obtain IP address from DHCP.

      ::

       ip link
       connmanctl enable wifi
       wpa_supplicant -B -i wlp0s20f3 -c <(wpa_passphrase "My ESSID" my-wifi-password)
       dhcpcd wlp0s20f3

   #. Wired

      ::

       ip link
       dhcpcd enp3s0

#. Remote install (optional) \***

   This facilitates being able to do the remainder of install remotely. This is commonly used, so I'm including it in the guide.

   a. Add normal user

      ::

       useradd -m -G users,wheel ken
       passwd ken
       rc-service sshd start

   #. Connect remotely to the machine being installed.

      Note the IP given given to ``livecd`` in the above step, then ssh to the livecd machine from the machine you wish to continue working from.

      If you have a local DHCP server / name server that the live CD gets an IP address from (such as `Technitium`)  then `livecd` may be registered as a host on the network. Otherwise you have to use the machine's IP address.

      Each time that the `livecd` is booted, the ssh service will generate a new key to identify itself, so the `StrictHostKeyChecking` option is to allow repeated connections in this case.

      ::

       ssh -o StrictHostKeyChecking=no ken@livecd
       password: ****
       sudo -i

#. Partition disk and create file systems

   A basic partition scheme is shown below.

   a. Use ``lsblk`` and/or ``dmesg | grep "\(sd[a-z]\)\|scsi [0-9]"`` to determine the drive to install to
   #. Use ``gdisk`` to partition the drive
   #. Use ``mkfs.fat -F 32`` to create boot/efi partition. If the machine already has an EFI partition, then it should be used for the Gentoo install - and no new boot/EFI partition needs to be made. Just be sure to have a backup of that partition (and ideally all other existing partitions that are to be kept). The 1 GB size is sufficient for booting multiple systems, though 200 MB is enough for the Gentoo install.
   #. Use ``mkfs.ext4`` to create root and home partitions. The ``xfs`` format is now preferred over using ext4 - adjust as desired.
   #. Use ``mkswap`` to create swap partition

#. Mount the file systems \***

   The partitions for the new machine need to be mounted under ``/mnt/gentoo`` in the live-CD environment

   #. Use ``mount`` to mount all but swap
   #. Use ``swapon`` to enable the swap partition

   Notes:

   * All partition values shown below are for example only.
   * Partitions should be created in order of partition number (e.g. sdb1, sdb2, sdb3, sdb4). Rows below are in the order that they can be mounted.
   * The root partition size is typical, but can be reduced if space is not available. Generally "steal" from ``home`` until there is no extra available, then reduce ``root`` as needed afterward.  The minimum root size is about 10 GB.
   * The boot partition (where the kernels are stored and read when booting) is shared by the ESP (EFI system partition) which is what UEFI looks for when booting. The kernel needs to be available outside of the root partition if it is encrypted at some point. UEFI always looks for a ``EFI`` directory within the efi partition, and so doesn't need to be separate from `/boot`.  That is: boot items live in ``/boot``, and efi items live in ``/boot/EFI``.

   \ 

   +----+-----+-------------+------------+-----------------+-----------------+-----------------+
   |Code|Label|Size         |filesystem  |partition        |mount point      |partition name   |
   +====+=====+=============+============+=================+=================+=================+
   |8300|root |100 GB       |ext4        |/dev/sdb3        |/mnt/gentoo      |root             |
   +----+-----+-------------+------------+-----------------+-----------------+-----------------+
   |EF00|boot |1 GB         |fat32       |/dev/sdb1        |/mnt/gentoo/boot |efi_boot         |
   +----+-----+-------------+------------+-----------------+-----------------+-----------------+
   |8300|home |<remainder>  |ext4        |/dev/sdb4        |/mnt/gentoo/home |home             |
   +----+-----+-------------+------------+-----------------+-----------------+-----------------+
   |8200|swap |4 GB         |swap        |/dev/sdb2        |<none>           |swap             |
   +----+-----+-------------+------------+-----------------+-----------------+-----------------+

#. Install the Stage3 file

   a. Make sure system date/time is set

      ::

       chronyd -q

   #. Go to to the nearest mirror using the `links` browser (use down-arrow to move, right-arrow to click-links, "d" to download, and "q" to quit)

      ::

       cd /mnt/gentoo
       links http://mirrors.rit.edu/gentoo/releases/amd64/autobuilds/current-stage3-amd64-desktop-openrc/

   #. Download latest stage3 file (amd64 openrc desktop) and its asc file. ``links`` is already installed (use ...

      ::

       stage3-amd64-desktop-openrc-20251116T161545Z.tar.xz
       stage3-amd64-desktop-openrc-20251116T161545Z.tar.xz.asc


   #. Verify the stage3 file

      ::

       gpg --auto-key-locate=clear,nodefault,wkd --locate-key releng@gentoo.org
       gpg --verify stage3-amd64-openrc-20251116T161545Z.tar.xz.asc

   #. Install the stage3 file on the filesystem

      ::

       tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner

Change-root
===========


#. Copy DNS information to new system

   ::

    cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

#. Change-root into the new system \***

   ::

    arch-chroot /mnt/gentoo

#. Create portage database

   ::

    emerge-webrsync

#. Profile selection - if not already set correctly, change the profile to ``/linux/amd64/xx.x/desktop (stable)`` which matches the desktop stage3 file you are using. Use the profile number of the item listed. Be sure to **not** use the profile with "systemd" in the name, only "desktop". The selected item will have an ``*``.
   ::

    eselect profile list | more
    eselect profile set 3
    eselect profile list | more

#. Initial configuring of portage. See handbook for more details on each.

   a. Edit ``/etc/portage/make.conf``

      ::

       cd /etc/portage
       nano make.conf

   #. add/edit these lines...

      ::

       GENTOO_MIRRORS="https://mirrors.rit.edu/gentoo"
       FEATURES="getbinpkg binpkg-request-signature"
       ACCEPT_LICENSE="@FREE @BINARY-REDISTRIBUTABLE"
       USE="-systemd dist-kernel dracut introspection gstreamer colord gnome-online-accounts keyring"

      i. Adjust the mirror URL to match the mirror you chose earlier
      #. Use binary packages whenever possible
      #. Accept binary licenses
      #. USE flags - for best results with binary packages, minimize the number of USE flags you set, but some need to be set. USE flags have multiple purposes

         * choose system options - this will cause certain software to be chosen and installed without explicitly installing it, as well as configuring that software
         * configure / choose between options, in addition to configuring how software should be built.

      Notes/explaination:

      - ``-systemd`` - `systemd` should never be pulled in as a dependency for another package
      - ``dist-kernel`` - needed when using a pre-built binary kernel (`gentoo-kernel-bin`)
      - ``dracut`` - needed for `installkernel` which configures the boot items after a kernel is installed.
      - ``introspection gstreamer colord gnome-online-accounts keyring`` is needed for `xfce4-meta` and `lxdm` install to be binary only

   #. Run ``getuto``
   #. Set binhist mirror.

      i. Determine if you machine supports "v3". Run ``ld.so --help`` and if it returns saying the x86-64-v3 is supported, then use the ``x86-64-v3`` variant for in the `binpackages` URL below, otherwise use ``x86-64``.
      #. Use ``links`` to verify URL for binary packages that match the ``desktop`` profile.
      #. Edit ``/etc/portage/binrepos.conf/gentobinhost.conf`` accordingly...

         ::

          sync-uri = https://mirrors.rit.edu/gentoo/releases/amd64/binpackages/23.0/x86-64

   #. Set rsync mirror - this is used to update the portage tree (which was initially done by emerge-webrsync). Create/edit ``/etc/portage/repos.conf`` accordingly...

      ::

       [gentoo]
       sync-uri = rsync://rsync5.us.gentoo.org/gentoo-portage

   #. Update ebuild repository and portage app

      ::

       emerge --sync
       emerge --ask --verbose --oneshot --changed-use sys-apps/portage

#. Set time-zone

   ::

    cd /etc
    ln -sf ../usr/share/zoneinfo/America/New_York localtime

#. Locale

   a. Generation - edit file so it has (only) ``en_US.UTF-8    UTF-8`` uncommented.

      ::

       nano locale.gen
       locale-gen

   b. Selection - use `eselect` to select the `en_US.UTF-8` profile

      ::

       eselect locale list
       eselect locale set 5
       . ./profile

#. Update fstab

   Run ``blkid >> /etc/fstab`` to add the information needed, then edit that information down to the below format using ``nano /etc/fstab``.

   ::

    UUID=402E-DDEB                             /boot           vfat            umask=0077,tz=UTC        1 2
    UUID=1651c6f4-c3fa-441a-b0d4-6509baf19cdd  /               ext4            defaults,noatime         0 1
    UUID=71222dab-818a-4b34-9d07-3a5cd3a6f9a4  none            swap            sw                       0 0

#. Set the hostname

   ::

    echo claude > /etc/hostname

#. Install and setup DHCP for network

   ::

    emerge --ask --verbose net-misc/dhcpcd
    rc-update add dhcpcd default

#. Edit ``/etc/hosts``

   ::

    127.0.0.1       claude.birchtreefarm.internal claude localhost
    ::1             claude.birchtreefarm.internal claude localhost

#. User setup

   a. Set root password

      ::

       passwd

   #. Create a regular user

      ::

       useradd -G users,wheel ken
       passwd ken

#. System services

   Setup a logger, cron, and ssh, bash completion, time sync

   Note: the emerge line is all one line

   ::

    emerge --ask --verbose app-admin/sysklogd sys-process/cronie app-shells/bash-completion \
       net-misc/chrony sys-fs/e2fsprogs sys-fs/dosfstools app-admin/sudo
    rc-update add sysklogd default
    rc-update add cronie default
    rc-update add sshd default
    rc-update add chronyd default
    rc-update add elogind default
    rc-update add dbus default

   Setup `sudo`

   ::

    EDITOR=nano visudo

   Towards the bottom of the file, find ``# %wheel ALL=(ALL:ALL) NOPASSWD: ALL`` and remove the leading ``#`` to uncomment the line, save.

#. Kernel

   #. Install kernel (binary)

      ::

       emerge --ask --verbose sys-kernel/installkernel
       mkdir -p /boot/EFI/Gentoo

       # these can all be on one typed line and should be emerged as a group
       # intel-microcode is only needed for Intel CPUs
       emerge --ask --verbose linux-firmware sof-firmware     \
                              intel-microcode                 \
                              sys-kernel/gentoo-kernel-bin

       emerge --ask --verbose --config sys-kernel/gentoo-kernel-bin
       eselect kernel list
       eselect kernel set 1
       eselect kernel list


#. Bootloader

   Install the rEFInd package.

   ::

    emerge --ask --verbose sys-boot/refind efibootmgr

   and configure the boot loader. As before, use ``blkid`` to view current partitions and their IDs. This UUID should be the UUID of the root (/) partition. `rEFInd` takes care of adding information for the initrd.


   Install rEFInd to the machine's UEFI and the EFI directory.

   ::

    refind-install

   Configure rEFInd options

   ::

    nano /boot/EFI/refind/refind.conf

   Turn off graphical mode - you get more information in text mode.

   You may need to add ``dont_scan_volumes`` configuration list if your drive contains NTFS (Windows partitions), so that those aren't scanned by rEFInd. These should be either the PARTLABEL or PARTUUID of the partitions listed by `blkid` and comma separated.

   ::

    textonly
    dont_scan_volumes "Basic data partition",f8e9778b-9f0d-4908-96d0-b3c36991dcdf,cc7089ca-153f-4d58-a9e9-3e6df22a2234

   Configure a boot item specifically for Gentoo (in addition to the items that rEFInd will find by scanning). The UUID should be the UUID field of the Gentoo root partition. Note the quotes surround the whole second field, not just the UUID.

   ::

    blkid >> /boot/refind_linux.conf
    nano /boot/refind_linux.conf

   Content

   ::

    "Gentoo" "root=UUID=1651c6f4-c3fa-441a-b0d4-6509baf19cdd"

---------------
New Environment
---------------

#. Restart

   a. Restart the machine (without booting the install disk) and make sure you boot into your new system.

      ::

       exit
       umount home
       umount boot
       cd ..
       umount gentoo
       /sbin/shutdown -r now

   #. Take care of any issues you see while it boots.

#. Desktop environment

   a. Install the meta package for `xfce4` and `lxdm`

      References

      i. https://wiki.gentoo.org/wiki/Xfce
      #. https://www.linuxfromscratch.org/blfs/view/11.3/x/lxdm.html

      ::

       emerge --ask --verbose xfce4-meta lxdm

   #. Edit /etc/lxdm/lxdm.conf and uncomment the `session` line and change it to point to `/usr/bin/startxfce4`.

      ::

       nano /etc/lxdm/lxdm.conf

   #. Add `xdm` to the default runlevel

      ::

       rc-update add xdm default

   #. Configure `xdm` to use `lxdm`.  Edit `/etc/conf.d/display-manager` and change the `DISPLAYMANAGER` setting to `lxdm`.

      ::

       nano /etc/conf.d/display-manager

   #. Restart - I find it easier to simply restart at this point. It verifies you can boot and end up logged into `xfce4`. Log in and open a terminal window.

      At this point the disk useage should be approximately 8.1 G for `root` and under 100 M for `EFI`/`boot`.

      ::

       df -h

      Checking memory use should show that swap space is available

      ::

       free

-----
Notes
-----

#. Portage notes for installing/updating/removing software (not needed now)

   a. Update all packages

      ::

       emerge --ask --verbose --update --deep @world
       emerge --ask --depclean

   b. List all manually installed packages

      ::

       cat /var/lib/portage/world

   c. Remove a package

      ::

       emerge --ask --verbose --depclean package-to-remove
       emerge --ask --verbose --depclean

   d. verify currently installed packages if you have modified configurations or USE flags

      ::

       emerge --ask --verbose --changed-use @world

#. System services notes for managing services running in the background

   a. list all services in `default` run-level

      ::

       rc-status default

   b. list all services

      ::

       ls /etc/init.d

   c. add service to run in the background

      ::

       rc-update add myservice default

   d. start a system service (now, as apposed to when the system restarts)

      ::

       rc-service myservice start

#. Misc.

   #. install tlp if on a laptop
   #. logs are in /var/log/messages
   #. The Void Linux documentation is recomended. Void linux is a non-systemd distribution. https://docs.voidlinux.org/
   #. When in a text terminal, you can switch between the available text terminals using `Alt F1` - `Alt F6`. If you boot into `xfce4`, you can switch to a text terminal using `Alt F1` - `Alt F6` and then back to the graphical environment using `Ctrl Alt F7`.


