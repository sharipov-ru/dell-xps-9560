## Goal

The main goal of this guide is to help to confugure XPS 9560 for linux distros, particularly for
Ubuntu 16.04. Most of the following settings will be fine for other distributions too.

The main focus is set to:

* Keep nearly max performance when laptop connected to AC power
* Archive max possible battery life when on battery mode
* Decrease overall noise level and temperatures (both for AC power and battery mode)

## Configuration

* Ubuntu 16.04
* XPS 9560 with i7-7700HQ, GTX1050, 4K display, 1TB NVME SSD, 32GB RAM, Intel 9260 Wifi card

## Turn off nvidia card

Hardware/software facts:

* Nvidia GTX 1050 consumes a lot of evergy even when XPS is in idle mode on Ubuntu 16.04
* Default nvidia GUI card switcher does not completely turn off nvidia card: 1050 always remains
active even when Intel graphics mode is enabled.
* I don't need GTX 1050 at all: built-in Intel graphics is working perfectly for 4K video playback,
and I can't think of any other use case when I use heavy-load graphics on Ubuntu.

So, the solution: (originally found [here](https://gist.github.com/jseris/a740f6a3fb0d18064e26dc66f9be4f1d))

1. Remove all nvidia drivers:

> sudo nvidia-uninstall
> sudo apt-get purge nvidia*

2. Install acpi-call-dkms so that we could turn the graphics card manually:

> sudo apt install acpi acpi-call-dkms

3. Create acpi_call module:

> sudo touch /etc/modules-load.d/acpi_call.conf

4. Write `acpi_call` value to the file:
> echo acpi_call > /etc/modules-load.d/acpi_call.conf

5. Create systemd dgpu-off.service:

> sudo cp services/dgpu-off.service /etc/systemd/system

6. Enable dgpu-off service:

> systemctl enable dgpu-off.service

## Update linux-kernel

Default Ubuntu 16.04 kernel is pretty old and I'd recommend updating to
[4.15.10](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.15.10) to archive stability, good 
hardware, software and power saving options support.

1. Download debs:

> wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.15.10/linux-headers-4.15.10-041510_4.15.10-041510.201803152130_all.deb

> wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.15.10/linux-headers-4.15.10-041510-generic_4.15.10-041510.201803152130_amd64.deb<Paste>

> wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.15.10/linux-image-4.15.10-041510-generic_4.15.10-041510.201803152130_amd64.deb

2. Install every fetched deb:

> sudo dpkg -i filename.deb

3. Reboot

## Update GRUB options

Dell XPS may fall into annoying issues when running on latest linux kernels. Possible issues are:

* Suspend/Resume may stuck or be unstable
* Soft lockups possible saying something like "NMI watchdog: BUG: soft lockup - CPU#2 stuck for 23s"

To return everything back to live & working state we need to set acpi_rev_override and pcie_port_pm
in /etc/default/grub:

> GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_rev_override=5 pcie_port_pm=off"

Basically it returns active ACPI revision back to version 5 instead of 6 in latest linux kernels
and disables power management on pcie port.

Edit the file, save, and run `sudo update-grub`

## Install TLP

TLP is an advanced linux power management tool which brings you reasonable defaults and ability to
customize every single power saving option.

> sudo add-apt-repository ppa:linrunner/tlp

> sudo apt-get update

> sudo apt-get install tlp tlp-rdw

I attached my tlp configuration here, some important details:

* Laptop is working at max performance when connected to AC power
* CPU locked to minimum frequency when on battery mode without any ability to be faster
* Hyperthreading/Turboboost is turned off on battery mode
* Bluetooth is turned of on startup, it needs to be turned on manually using default ubuntu switch
* Enabled autosuspend for the fingerprint scanner
* Enabled autosuspend for the touch screen

## Install monitoring tools

i7z helps us to monitor current CPU frequencies/temperatures:

> sudo apt-get install i7z powertop

powerstat shows average energy consumption:

> sudo apt-get install powerstat

## CPU Undervolting

With monitoring tools installed, it's a good time to undervolt your CPU using
[georgewhewell/undervolt](https://github.com/georgewhewell/undervolt) python script.

Beaware, undervolting is the most dangerous section in this guide and you may brick you computer
easily if you don't have strong overclocking/undervolting experience!

All example values below are tested with my computer and may not work for your particular CPU!

Clone python script:

> git clone git@github.com:georgewhewell/undervolt.git

Copy script to /usr/bin:

> sudo cp undervolt/undervolt.py /usr/bin/undervolt

Read initial values:

> sudo undervolt --read

Set voltages for CPU core, CPU cache and CPU uncore:

> sudo undervolt --core -100 --cache -100 --uncore -100

Create undervolt service:

> cp services/undervolt.service /etc/systemd/system/undervolt.service

Create bins:

> cp bin/undervolt_start /usr/bin/undervolt_start

> cp bin/undervolt_stop /usr/bin/undervolt_stop

Don't forget to properly test, be ready to laptop freezes when testing and good luck :)

## Results

Laptop power consumption:

* Idle with screen turned off: 4 wt
* Idle with screen at 40% brightness: 8.5 wt
* Idle with screen at 100% brightness: 14 wt

These settings gives me around 8-9 hours of battery life when I work using:

* 1-2 rails running Ruby on Rails apps 
* Active Webpack
* Neovim and active ruby/js/eslint/rubocop/reek linters
* Few active docker containers active
* TDD & often rspec/mocha/jest executions
* Wifi & VPN enabled

Yeah, the overall performance is noticeable slower, but usually I don't care about the performance 
when I'm working on battery.

# Useful links

* [TLP - Advanced Power Management for Linux](https://github.com/linrunner/TLP)
* [How to update kernel to the latest mainline version without any Distro-upgrade?](https://askubuntu.com/questions/119080/how-to-update-kernel-to-the-latest-mainline-version-without-any-distro-upgrade)
* [Disabling dGPU on Dell XPS 9560 on Ubuntu 16.10](https://gist.github.com/jseris/a740f6a3fb0d18064e26dc66f9be4f1d)
* [XPS 9560 - Battery life optimization](https://www.reddit.com/r/Dell/comments/5y3rii/xps_9560_battery_life_optimization_and_fan/)
