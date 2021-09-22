# Ubuntu OS Initial Tasks

## Removing snap package

Free resources (computing, memory and storage) since I am not going to use snap package manager.

### Step 1. List snap packages installed

    sudo snap list

The output will something like

    sudo snap list
    Name    Version   Rev    Tracking       Publisher   Notes
    core18  20210611  2074   latest/stable  canonical✓  base
    lxd     4.0.7     21029  4.0/stable/…   canonical✓  -
    snapd   2.51.1    12398  latest/stable  canonical✓  snapd

### Step 3. Remove snap packages

    snap remove <package>

    snap remove lxd && snap remove core18 && snap remove snapd

### Step 4. Remove snapd package

    sudo apt purge snapd

Remove packages not required

    sudo apt autoremove

# Raspberry PI specific configuration

## Installing Fake RTC clock

Raspberry PI does not have by default a RTC (real-time clock) keeping the time when the Raspberry PI is off. A RTC module can be added to each RaspberryPI but we won`t do it here since we will use NTP to keep time in sync.

Even when NTP is used to synchronize the time and date, when it boots takes as current-time the time of the first-installation and it could cause problems in boot time when the OS detect that a mount point was created in the future and ask for manual execution of fscsk

> NOTE: I have detected this behaviour with my Raspberry PIs when mounting the iSCSI LUNs in `node1-node4` and after rebooting the server, the server never comes up.

As a side effect the NTP synchronizatio will also take longer since NTP adjust the time in small steps.

For solving this [`fake-hwclock`](http://manpages.ubuntu.com/manpages/focal/man8/fake-hwclock.8.html) package need to be installed. `fake-hwclock` keeps track of the current time in a file and it load the latest time stored in boot time.

## Installing Utility scripts

Raspberry PI OS contains several specific utilities such as `vcgencmd` that are also available in Ubuntu 20.04 through the package [`libraspberrypi-bin`](https://packages.ubuntu.com/focal-updates/libraspberrypi-bin)

    sudo apt install libraspberrypi-bi

Two scripts, using `vcgencmd` command for checking temperature and throttling status of Raspberry Pi, can be deployed on each Raspberry Pi (in `/usr/local/bin` directory)

`pi_temp` for getting Raspeberry Pi temperature
`pi_throttling` for getting the throttling status

Boths scripts can be executed remotely with Ansible

    ansible -i inventory.yml -b -m shell -a "pi_temp" raspberrypi
    
    ansible -i inventory.yml -b -m shell -a "pi_throttling" raspberrypi

## Change default GPU Memory Split

The Raspberry PI allocates part of the RAM memory to the GPU (76 MB of the available RAM)

Since the Raspberry PIs in the cluster are configured as a headless server, without monitorm and using the server Ubuntu distribution (not desktop GUI) Rasberry PI reserved GPU Memory can be set to lowest possible (16M).

- Step 1. Edit `/boot/firmware/config.txt` file, adding at the end:

    gpu_mem=16

- Step 2. Reboot the Raspberry Pi

    sudo reboot