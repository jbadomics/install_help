

This page describes how to configure a 2015 MacBook Pro to triple boot OS X El Capitan, Windows 8, and Ubuntu 14.04 LTS. It assumes that you have a clean, working installation of OS X 10.11 El Capitan already installed. If you prefer to start with a clean hard drive and reinstall OS X, follow the steps under Installing OS X below; otherwise, skip ahead to Installing Windows 8.

tl;dr?


What you need

*   a stable Internet connection, preferably wired. Apple's USB Ethernet adapter was recognized by all 3 operating systems and worked reliably during setup.

*   a downloaded `.iso` file of Windows 8 (does not require its own dedicated thumb drive or partition)

*   a dedicated Ubuntu 14.04 installation USB stick. Instructions for making one can be found here.

*   sufficient disk space for each OS.

*   about 1.5 hours to get everything installed.

Optional

*   a USB mouse for right-clicking in Windows and Ubuntu while you're setting things up.

*   additional disk space for a shared data partition

*   a printed copy of these instructions :)

Installing OS X

## Create bootable OS X installer

Find a USB stick or external hard drive with at least 8 GB capacity, and transfer any important data off of it as this procedure will erase the disk.

From the App Store, download OS X El Capitain *but do not begin the installation process*. In Finder, open Applications. You should see an app called "Install OS X El Capitan." Insert your (bootable) USB stick or external hard drive and open Finder to determine the name of your external disk. If you prefer, you can also erase the disk using Disk Utility and simply select the default "Untitled" name. The command below assumes the disk is named "Untitled" but modify as needed. Open Terminal and type: (**NOTE:** this will erase the disk!):

    sudo /Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app --nointeraction

and replace `/Volumes/Untitled` with the name of your external disk.

## Install OS X

Reboot the computer and, with your installer disk connected, hold down the Option key. You will see a menu with different boot options. Select the yellow icon labeled "Install OS X El Capitan."

When the installer loads, run Disk Utility. Select your internal hard drive and click "erase." Give the disk a name, and select GUID partition table and OS X Extended (Journaled). When the command completes, quit Disk Utility.

Select "Install OS X" from the menu and select your hard disk (i.e. the one you just erased and reformatted) as the installation target. The system will reboot and eventually ask you for configuration/preferences. **DO NOT transfer any files at this time.** Do not enable FileVault encryption.

Install Windows 8

A few other tutorials for triple booting call for arresting the Windows Boot Camp installation process in order to install Ubuntu. None of them worked for me. I could only get Windows booting properly if I let Boot Camp run from start to finish, without interruption. Boot Camp requires that the hard drive have only one partition, and so Windows needs to be installed *before* Ubuntu. (This is a bit misleading, since an OS X-only partition actually has 3 partitions: the EFT system partition (ESP), the OS itself, and the OS X recovery partition. See for yourself: before proceeding, open a Terminal and type `diskutil list`.)

1.  Open Applications --> Utilities --> Boot Camp Assistant.

2.  Select the `.iso` file you downloaded for Windows 8.

3.  Drag the slider to give **only as much disk space as you want for Windows 8**. In my case, I left ~80 GB for Windows at the end of the disk.

4.  Click continue. Boot Camp Assistant will download the required software and drivers, and will reboot automatically into the Windows 8 installer.

5.  Follow the prompts and select the option to customize the installation. You will then be shown a list of partitions. **DO NOT** add or remove any partitions. Click on the advanced options button, then select `Disk 0 Partition 5: BOOTCAMP`, and click Format. This will erase the Boot Camp partition that the Assistant created and reformat it as NTFS.

6.  Select `Disk 0 Partition 5: BOOTCAMP` and click Next.

7.  Grab a cup of coffee while Windows finishes its installation. It will reboot several times.

8.  The Windows installer will eventually bring up a Boot Camp dialog box. Click through the prompts to install drivers. When the Boot Camp installer finishes, click Reboot.

You should now have a stable dual-boot system, and (for the time being) there are two ways of selecting which OS you want to boot:

*   Press and hold the Option key on reboot

*   Select the Startup Disk before you reboot. In OS X, go to System Preferences --> Startup Disk, and select BOOTCAMP. In Windows, search for 'boot camp' and open the Windows version of the same utility.

**Before you proceed,** make sure that you can safely reboot into both operating systems.

Prepare for Ubuntu Installation

By default, Apple does not allow existing partitions to be resized. This is because OS X occupies a "Core Storage" partition, which Google tells me no one really understands. Fortunately, this can be disabled.

To review, we now have OS X taking up the first 920 GB, with Windows 8 taking up the last 80 GB. We want to shrink the existing OS X partition without modifying anything else.

1.  Open a Terminal (Applications --> Utilities --> Terminal) and type:

        diskutil cs list
 
    You should see an output that looks something like this:

        CoreStorage logical volume groups (1 found)
        |
        +-- Logical Volume Group C2356C06-A89F-4334-AE29-D7390FB36A11
            =========================================================
            Name:         mac
            Status:       Online
            Size:         918372315136 B (918.4 GB)
            Free Space:   18984960 B (19.0 MB)
            |
            +-< Physical Volume CB7CE669-6B7C-44C4-9EC3-71E9A6072BF3
            |   ----------------------------------------------------
            |   Index:    0
            |   Disk:     disk0s2
            |   Status:   Online
            |   Size:     918372315136 B (918.4 GB)
            |
            +-> Logical Volume Family EA474DAB-C993-4A67-8DC8-18E4B1BF06E1
                ----------------------------------------------------------
                Encryption Type:         None
                |
                +-> Logical Volume B3ECE9A1-8E51-444B-9D46-9E470BC92BC4  <---- COPY THIS STRING
                    ---------------------------------------------------
                    Disk:                  disk1
                    Status:                Online
                    Size (Total):          918001008640 B (918.0 GB)
                    Revertible:            Yes (no decryption required)  <--- This must say 'yes'
                    LV Name:               mac
                    Volume Name:           mac
                    Content Hint:          Apple_HFS

2.  You should see a revertible logical volume. In a Terminal window type

        diskutil cs revert B3ECE9A1-8E51-444B-9D46-9E470BC92BC4

    where `B3ECE9A1-8E51-444B-9D46-9E470BC92BC4` is the UUID for your logical volume.

3.  Reboot.

4.  Reopen Terminal and type `diskutil list` to show all of your current disks and partitions.

5.  Type `diskutil cs list`. The output should say

        No CoreStorage logical volume groups found

6.  Reboot again, but this time boot into Recovery Mode by holding down Command + R as the system boots.

7.  Disable System Integrity Protection. This is required for installing GPT fdisk and rEFInd below. This is a security feature Apple introduced with El Capitan that protects certain low-level system directories, but fortunately this can be disabled. Within Recovery Mode, select Terminal from the Utilities pull down menu and type

        csrutil disable
        reboot

8. The computer should boot normally into OS X. Let's check our disk partitioning scheme again with `diskutil list`:

        /dev/disk0 (internal, physical):
           #:                       TYPE NAME                    SIZE       IDENTIFIER
           0:      GUID_partition_scheme                        *1.0 TB     disk0
           1:                        EFI EFI                     209.7 MB   disk0s1
           2:                  Apple_HFS mac                     918.4 GB   disk0s2
           3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
           4:       Microsoft Basic Data BOOTCAMP                81.3 GB    disk0s4



