Gentoo Install
##############

Notes for installing Gentoo


This guide is meant to be used along with the official Gentoo handbook and other Gentoo documentation (e.g. `EFI Stub <https://wiki.gentoo.org/wiki/EFI_stub/en>`_).

As the handbook is very detailed, it is easy to "miss the forest in the trees." This guide is meant to severly limit installation choices and hide some of the complexity and verbosity of the handbook to ease the process, so that important details are not overlooked.

Commands shown are **examples** and should be edited if needed.

The steps do not need to be done in one session, start to finish. Steps needed to pause and restart for a given point in the process:

a. In `Create bootable media`_ - pick up at the point where you left off.
#. In `Boot the live-CD`_ - redo all steps marked with ``***`` up to the point where you left off.
#. In



Contraints:

* x86-64 (AMD64 or Intel 64)
* UEFI only
* Install using Gentoo minimal install iso on a thumbdrive created on an existing desktop linux install with a browser available
* Downloads from my nearest mirror site
* Plain GPT partitions with / (root), /efi, swap (and optionally /home)
* ext4 for file-systems
* OpenRC
* Desktop profile and matching Stage3 file
* Install binary packages whenever possible
* Use EFI-stub for boot loading
* XFCE4
* DHCP for network setup
* The choice of system logger, cron, time sync

Assumptions:

* Partitioning method
* DHCP used for networking
* 

------------------------------------------------

Create bootable media
=====================

Using an existing linux installation, create a removable thumb-drive that can boot the destination machine. This live-CD environment will contain the scripts and utilities needed to prepare the destination machine to boot Gentoo directly. Existing machines with other operating systems can be used to create this live-CD, but the steps would be somewhat different. The name "live-CD" matches the assigned host-name when the system is running - it can be any sort of removable media, like a CD-ROM or a flash-drive.
 
#. Manually find your nearest mirror - go to ``https://www.gentoo.org/downloads/mirrors/`` and make note of the https URL of the mirror you choose.

#. In a browser, go to the nearest mirror

   ::

    http://mirrors.rit.edu/gentoo/releases/amd64/autobuilds/current-install-amd64-minimal/

#. Download latest iso and its asc file. There will several older ``.asc`` files along with the current ``.iso`` and ``.iso.asc``, which is the ones you want (right click, save link as...).

   ::

    install-amd64-minimal-20251116T161545Z.iso
    install-amd64-minimal-20251116T161545Z.iso.asc


#. Verify iso

   ::

    gpg --auto-key-locate=clear,nodefault,wkd --locate-key releng@gentoo.org
    gpg --verify install-amd64-minimal-20251116T161545Z.iso.asc

#. Insert thumb drive and find its name - make sure you have the correct drive

   ::

    sudo dmesg | grep "\(sd[a-z]\)\|scsi [0-9]"

#. Write the iso image

   ::

    sudo dd if=install-amd64-minimal-20251116T161545Z.iso of=/dev/sdb bs=4096 status=progress
    sudo sync

#. Verify data on thumbdrive. Remove and reinsert the thumbdrive first. Get size of file in bytes from ``ls``, use that in ``cmp`` which should return without any response (if the data on the drive doesn't match, it will tell you).

   ::

    ls -l install-amd64-minimal-20251116T161545Z.iso
    sudo cat /dev/sdb | cmp --bytes 813670400 install-amd64-minimal-20251123T153051Z.iso

Boot the live-CD
================

#. BIOS setup

   Boot the machine and enter the BIOS setup (Del, F1, F10, Esc, etc.)

   a. Change the settings for legacy BIOS boot to support UEFI only. We want to use UEFI and doing this will remove all of the legacy boot options from the boot menu selection, which we don't want to be booting anyway. (recommended)
   #. Find the setting for "Windows" vs "Other OS" and choose "Windows". This will enable additional options for secure boot which you may want.

#. Boot the thumb-drive \***

   Use whatever BIOS keystroke is needed (Del, F1, F10, Esc, etc. - some machines will have a F-key assigned to get to the boot menu directly, others have override boot options in the "Exit..." menu of setup. Choose the option to boot the thumbdrive.

#. Configure networking \***

   Configure either wireless or wired. Wired is easier and generally faster if you have a wired connection.

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

   This facilitatea being able to do the remainder of install remotely.

   a. Add normal user

      ::

       useradd -m -G users,wheel ken
       passwd ken
       rc-service sshd start

   #. Connect remotely to the machine being installed.

      Note the IP given given to ``livecd`` in the above step, then ssh to the livecd machine from the machine you wish to continue working from.

      If you have a local DHCP server / name server that the live CD gets an IP address from (such as Technitium)  then ``livecd`` may be registered as a host on the network. Otherwise you have to use the machine's IP address.

      Each time that the livecd is booted, the ssh service will generate a new key to identify itself, so the StrictHostKeyChecking option is to allow connection in this case.

      ::

       ssh -o StrictHostKeyChecking=no ken@livecd
       password: ****
       sudo -i

#. Partition disk and create file systems

   A basic partition scheme is shown below.

   a. Use ``lsblk`` and/or ``dmesg | grep "\(sd[a-z]\)\|scsi [0-9]"`` to determine the drive to install to
   #. Use ``gdisk`` to partition the drive
   #. Use ``mkfs.fat -F 32`` to create efi partition. If the machine has another disk that already has an EFI partition, then provided it isn't full, it can (and should) be used for the gentoo install - and no new EFI partition needs to be made. Just be sure to have a backup of that partition (and ideally all other existing partitions that are to be kept). The 1 GB size is sufficient for booting multiple systems, though 200 MB is enough for the Gentoo install.
   #. Use ``mkfs.ext4`` to create root and home partitions. The ``xfs`` format is now preferred over using ext4 - adjust as desired.
   #. Use ``mkswap`` to create swap partition

#. Mount the file systems \***

   The partitions for the new machine need to be mounted under ``/mnt/gentoo`` in the live-CD environment

   #. Use ``mount`` to mount all but swap
   #. Use ``swapon`` to enable the swap partition

   Notes:

   * All partition values shown below are for example only.
   * Partitions should be created in order of partition number (e.g. sdb1, sdb2, sdb3, sdb4). Rows below are in the order that they can be mounted.
   * The root partition size is typical, but can be reduced if space is not available. Generally "steal" from ``home`` until there is no extra available, then reduce ``root`` as needed afterward.  The minimimum root size is about 8 GB.

   \ 

   +----+-----+-------------+------------+-----------------+-----------------+-----------------+
   |Code|Label|Size         |filesystem  |partition        |mount point      |partition name   |
   +====+=====+=============+============+=================+=================+=================+
   |8300|root |100 GB       |ext4        |/dev/sdb3        |/mnt/gentoo      |root             |
   +----+-----+-------------+------------+-----------------+-----------------+-----------------+
   |EF00|efi  |1 GB         |fat32       |/dev/sdb1        |/mnt/gentoo/efi  |efi              |
   +----+-----+-------------+------------+-----------------+-----------------+-----------------+
   |8300|home |<remainder>  |ext4        |/dev/sdb4        |/mnt/gentoo/home |home             |
   +----+-----+-------------+------------+-----------------+-----------------+-----------------+
   |8200|swap |4 GB         |swap        |/dev/sdb2        |<none>           |swap             |
   +----+-----+-------------+------------+-----------------+-----------------+-----------------+

#. Install the Stage3 file

   a. Go to to the nearest mirror

      ::

       http://mirrors.rit.edu/gentoo/releases/amd64/autobuilds/current-stage3-amd64-desktop-openrc/

   #. Download latest stage3 file (amd64 openrc desktop) and its asc file. ``links`` is already installed (use down-arrow to move, right-arrow to click-links, "d" to download, and "q" to quit...

      ::

       cd /mnt/gentoo
       chronyd -q
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
===========================


#. Copy DNS information to new system

   ::

    cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

#. Change-root into the new system \***

   ::

    arch-chroot /mnt/gentoo

#. Create portage database

   ::

    emerge-webrsync

#. Profile selection - if not already set correctly, change the profile to ``/linux/amd64/xx.x/desktop`` which matches the desktop stage3 file you are using. Use the profile number of the item listed.
   ::


    eselect profile list | more
    eselect profile set 3

#. Initial configuring of portage. See handbook for more details on each.

   a. Edit ``/etc/portage/make.conf`` and add these lines...

      i. Adjust the mirror URL to match the mirror you chose earlier
      #. Use binary packages whenever possible
      #. Accept binary licenses
      #. USE flags - for best results with binary packages, minimize the number of USE flags you set, but some need to be set. USE flags are used as a way to configure / choose between options, in addition to configuring how software should be built.

         - ``-systemd`` - systemd should never be pulled in as a dependancy for another package
         - ``dist-kernel`` - needed when using ``gentoo-kernel-bin``
         - ``dracut``, ``efistub`` - boot system using an EFI stub (instead of using grub). Both flags needed for ``installkernel`` which configures the boot items after a kernel is installed.

      ::

       GENTOO_MIRRORS="https://mirrors.rit.edu/gentoo"
       FEATURES="getbinpkg binpkg-request-signature"
       ACCEPT_LICENSE="@FREE @BINARY-REDISTRIBUTABLE"
       USE="-systemd dracut dist-kernel efistub"

   #. Run ``getuto``
   #. Set binhist mirror.

      i. Determine if you machine supports "v3". Run ``ld.so --help`` and if it returns saying the x86-64-v3 is supported, then use the ``x86-64-v3`` variant for in the binpackages URL below, otherwise use ``x86-64``.
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
       emerge --ask --verbose --oneshot sys-apps/portage

#. Set time-zone

   ::

    cd /etc
    ln -sf ../usr/share/zoneinfo/America/New_York localtime

#. Locale

   a. Generation - edit file so it has (only) ``en_US.UTF-8 UTF-8`` uncommented.

      ::

       nano locale.gen
       locale-gen

   b. Selection - use ``eselect`` to select the ``en_US.UTF-8`` profile

      ::

       eselect locale list
       eselect locale set 5
       . ./profile

#. Update fstab

   Run ``blkid >> /etc/fstab`` to add the information needed, then edit that information down to the below format using ``nano /etc/fstab``.

   ::

    UUID=402E-DDEB                             /efi            vfat            umask=0077,tz=UTC        1 2
    UUID=1651c6f4-c3fa-441a-b0d4-6509baf19cdd  /               ext4            defaults,noatime         0 1
    UUID=71222dab-818a-4b34-9d07-3a5cd3a6f9a4  none            swap            sw                       0 0

#. Set the hostname

   ::

    echo claude > /etc/hostname

#. Install and setup DHCP for network

   ::

    emerge --ask --verbose net-misc/dhcpcd
    rc-update add dhcpcd default
    rc-service dhcpcd start             # ***

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

#. Setup a logger, cron, and ssh, bash completion, time sync

   Note: the emerge line is all one line

   ::

    emerge --ask --verbose app-admin/sysklogd sys-process/cronie app-shells/bash-completion \
       net-misc/chrony sys-fs/e2fsprogs sys-fs/dosfstools
    rc-update add sysklogd default
    rc-update add cronie default
    rc-update add sshd default
    rc-update add chronyd default

#. Bootloader

   Steps to all booting and to keep the EFI settings updated when kernels are updated.

   #. Unmask EFI stub booting

      Create ``/etc/portage/package.accept_keywords/installkernel`` and fill with:

      ::

       sys-kernel/installkernel
       sys-boot/uefi-mkconfig
       app-emulation/virt-firmware
       sys-kernel/dracut

    #. Install ``installkernel``

       ::

        emerge --ask --verbose sys-kernel/installkernel
        mkdir -p /efi/EFI/Gentoo


.. Left off here

#. Firmware and Kernel (``intel-microcode`` is only needed for Intel CPUs)

   ::

    emerge --ask --verbose linux-firmware sof-firmware         # --newuse, --changed-use if needed
    emerge --ask --verbose intel-microcode                     # if needed

    emerge --ask --verbose --config sys-kernel/gentoo-kernel-bin
    eselect kernel list
    eselect kernel set 1
    eselect kernel list

    emerge --ask --verbose sys-boot/efibootmgr
    efibootmgr --create --disk //dev/sdb --part 1 --label "gentoo --loader "\EFI\Gentoo\bzImage.efi"

..





#. Portage notes (not needed now)

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


99. attic

  a. ``emerge --ask --verbose ``

    # emerge --ask --verbose --update --deep --newuse @world
