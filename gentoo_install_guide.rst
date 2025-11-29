Gentoo Install
##############

Notes for installing Gentoo

Contraints:

* x86-64 (AMD64 or Intel 64)
* UEFI
* Install using Gentoo minimal install iso on a thumbdrive created on an existing desktop linux install with a browser available
* Downloads from my nearest mirror site
* Plain GPT partitions with / (root), /efi, swap (and optionally /home)
* OpenRC
* Desktop profile and matching Stage3 file
* XFCE4
* 

Assumptions:

* Partitioning method
* DHCP used for networking
* 

Notes:

* This guide is meant to be used along with the official handbook, when the handbook is needed. It is very detailed, and it is easy to get lost in the details. This guide is meant to hide some of the complexity and verbosity of the handbook to ease the process and so that important details are not overlooked.
* Commands shown are **examples** and should be edited if needed.
* 

------------------------------------------------

1. Create bootable media using an existing Linux installation

   a. Goto to nearest mirror

      ::

       http://mirrors.rit.edu/gentoo/releases/amd64/autobuilds/current-install-amd64-minimal/

   #. Download latest iso and its asc file (right click, save link as...).

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

#. Boot the thumbdrive using whatever BIOS keystroke is needed (Del, F3, etc). If given the choice, choose the thumbdrive name that is prefixed with 'UEFI'.

#. Configure networking - either wireless or wired.

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

#. Facilitate being able to do remainder of install remotely (optional)

   a. Add normal user

      ::

       useradd -m -G users,wheel ken
       passwd ken
       rc-service sshd start

   #. Note the IP given given to ``livecd`` in the above step, then ssh to the livecd machine from the machine you wish to continue working from.

      If you have a local DHCP server / name server that the live CD gets an IP address from (such as , then

      ::

       ssh ken@livecd
       password: ****
       sudo -i

#. Determine what disk to install to and setup the disk.

   a. Use ``lsblk`` and/or ``dmesg | grep "\(sd[a-z]\)\|scsi [0-9]"`` to determine the drive to install to
   #. Use ``gdisk`` to partition
   #. Use ``mkfs.fat32`` to create efi partition. If the machine has another disk that already has an EFI partition, then provided it isn't full, it can (and should) be used for the gentoo install - and no new EFI partition needs to be made. Just be sure to have a backup of that partition (and ideally all other existing partitions that are to be kept).
   #. Use ``mkfs.ext4`` to create root and home partitions. the ``xfs`` format is
   #. Use ``mkswap`` to create swap partition
   #. Use ``mount`` to mount all but swap
   #. Use ``swapon`` to swap
   #. Note: All partition values shown below are for example only.  
   #. Note: Rows are in the order that they can be mounted.

\ 

   +----+-----+-------------+------------+-----------------+-----------------+
   |Code|Label|Size         |filesystem  |partition        |mount point      |
   +====+=====+=============+============+=================+=================+
   |8300|root |100 GB       |ext4        |/dev/sdb3        |/mnt/gentoo      |
   +----+-----+-------------+------------+-----------------+-----------------+
   |EF00|efi  |200 MB       |fat32       |/dev/sdb1        |/mnt/gentoo/efi  |
   +----+-----+-------------+------------+-----------------+-----------------+
   |8300|home |<remainder>  |ext4        |/dev/sdb4        |/mnt/gentoo/home |
   +----+-----+-------------+------------+-----------------+-----------------+
   |8200|swap |4 GB         |swap        |/dev/sdb2        |<none>           |
   +----+-----+-------------+------------+-----------------+-----------------+

#. Download the Stage3 file

   a. Goto to nearest mirror

      ::

       http://mirrors.rit.edu/gentoo/releases/amd64/autobuilds/current-install-amd64-minimal/``

   #. Download latest stage3 file (amd64 openrc desktop) and its asc file

      ::

       current-stage3-amd64-openrc/stage3-amd64-openrc-20251116T161545Z.tar.xz
       current-stage3-amd64-openrc/stage3-amd64-openrc-20251116T161545Z.tar.xz.asc


   #. Verify the stage3 file

      ::

       gpg --auto-key-locate=clear,nodefault,wkd --locate-key releng@gentoo.org
       gpg --verify stage3-amd64-openrc-20251116T161545Z.tar.xz.asc

99. attic



