# Automated Installation of Kali Linux 2024.4 using Preseed File

This repository contains a preseed configuration file for automated installation of Kali Linux.
The preseed file automates the installation process by setting various configuration options for localization, time zone, network, account setup, partitioning, software selection, and post-installation commands.
Currently,

1. Download Kali ISO from <https://www.kali.org/downloads/>
2. Create a new VM that boots from the Kali ISO
3. At the Kali boot menu, press `<tab>`.
4. Edit the command line and replace the below

        `preseed/url=/cdrom/simple-cdd/default.preseed simple-cdd/profiles=kali,offline desktop=xfce`

    with

        auto=true url=https://raw.githubusercontent.com/ideologysec/kali-preseed/master/kde-full.cfg priority=critical

## Preseed Configuration

### Localization and Time Zone

```bash
# Set the locale to English (United States)
d-i debian-installer/locale string en_US.UTF-8

# Disable keyboard layout detection
d-i console-setup/ask_detect boolean false

# Set the keyboard layout to US
d-i console-setup/layoutcode string us
```

### Clock and Time Zone Setup

```bash
# Set the hardware clock to UTC
d-i clock-setup/utc boolean true

# Set the time zone to UTC
d-i time/zone string UTC
```

### Network Configuration

```bash
# Automatically select the network interface
d-i netcfg/choose_interface select auto

# Set the hostname for the system
d-i netcfg/get_hostname string kali

# Set the domain name for the system
d-i netcfg/get_domain string

# Set DHCP timeout to 60 seconds
d-i netcfg/dhcp_timeout string 60
```

### Mirror Settings

```bash
# Manually configure the package mirror country
d-i mirror/country string manual

# Set the package mirror hostname
d-i mirror/http/hostname string http.kali.org

# Set the directory on the package mirror
d-i mirror/http/directory string /kali

# Set the HTTP proxy (empty in this case)
d-i mirror/http/proxy string
```

### Account Setup

```bash
# Disable root login
d-i passwd/root-login boolean false

# Create a normal user account
d-i passwd/make-user boolean true

# Set the full name for the new user
d-i passwd/user-fullname string Kali User

# Set the username for the new user
d-i passwd/username string kali

# Set the password for the new user
d-i passwd/user-password password kali
d-i passwd/user-password-again password kali
```

### Disable CDROM Entries After Install

```bash
# Disable CDROM entries in APT sources after installation
d-i apt-setup/disable-cdrom-entries boolean true
```

### Enable contrib and non-free Repositories

```bash
# Enable non-free repositories
d-i apt-setup/non-free boolean true

# Enable contrib repositories
d-i apt-setup/contrib boolean true
```

### Add Kali Security Mirror

```bash
# Add a local security mirror for Kali
d-i apt-setup/local0/repository string http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware

# Comment for the local security mirror
d-i apt-setup/local0/comment string Security updates

# Do not use the source repository
d-i apt-setup/local0/source boolean false
```

### Partitioning

```bash
# Select the disk to partition
d-i partman-auto/disk string /dev/sda

# Use the regular partitioning method
d-i partman-auto/method string regular

# Select the "atomic" partitioning recipe
d-i partman-auto/choose_recipe select atomic

# Confirm writing the new partition table
d-i partman-partitioning/confirm_write_new_label boolean true

# Finish partitioning
d-i partman/choose_partition select finish

# Confirm partitioning
d-i partman/confirm boolean true

# Do not warn about overwriting data on the disk
d-i partman/confirm_nooverwrite boolean true
```

### Custom Partition Recipe

```bash
# Define a custom partitioning recipe
# I use a 100GB VirtIO disk, and separate /home
d-i partman-auto/expert_recipe string                               boot-root ::                                                    1000 5000 10000 ext3                                              $primary{ } $bootable{ }                                      method{ format } format{ }                                    use_filesystem{ } filesystem{ ext3 }                          mountpoint{ /boot }                                       .                                                             2000 10000 1000000000 ext4                                        method{ format } format{ }                                    use_filesystem{ } filesystem{ ext4 }                          mountpoint{ / }                                           .                                                             500 1000 100% linux-swap                                          method{ swap } format{ }                                  .

# Confirm writing the new partition table
d-i partman/confirm_write_new_label boolean true

# Confirm partitioning
d-i partman/confirm boolean true

# Do not warn about overwriting data on the disk
d-i partman/confirm_nooverwrite boolean true
```

### GRUB Boot Loader

```bash
# Install GRUB only on Debian
d-i grub-installer/only_debian boolean true

# Do not install GRUB with other operating systems
d-i grub-installer/with_other_os boolean false

# Install GRUB on the specified disk
d-i grub-installer/bootdev string /dev/sda
```

### Software Selection

```bash
# Select standard and KDE desktop tasks
tasksel tasksel/first multiselect standard, kde-desktop

# Include additional packages
d-i pkgsel/include string kali-linux-core, kali-desktop-xfce

# Do not upgrade installed packages
d-i pkgsel/upgrade select none

# Do not update policies
d-i pkgsel/update-policy select none
```

### Miscellaneous

```bash
# Do not participate in popularity contest
popularity-contest popularity-contest/participate boolean false

# Include the kali-root-login package
d-i pkgsel/include string kali-root-login
```

### Finish Installation

```bash
# Reboot after the installation finishes
d-i finish-install/reboot_in_progress note
```

### Late Command to Install Additional Software and Configure Network

```bash
# Run a late command to configure the system
d-i preseed/late_command string     in-target sh -c 'echo "deb http://http.kali.org/kali kali-rolling main contrib non-free" > /etc/apt/sources.list;                       apt-get update -y || true;                       apt-get install -y kali-linux-default kali-desktop-xfce network-manager openssh-server || true;                       systemctl enable NetworkManager;                       systemctl start NetworkManager;                       systemctl enable ssh;                       systemctl start ssh'
```

### Prevent Prompts from Halting the Installation

```bash
# Set the debconf frontend to noninteractive
d-i debconf debconf/frontend select Noninteractive

# Allow unauthenticated SSL connections
d-i debian-installer/allow_unauthenticated_ssl boolean true

# Do not show the splash screen
d-i debian-installer/splash boolean false

# Set the debconf priority to critical
d-i debconf debconf/priority select critical
```
