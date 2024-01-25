# Stability Tweaks
This contains a list of things added to get the laptop to be "stable". Explainations are provided below...

## Wifi
The qualcomm driver has a bug where the performance degrades over time. To combat this we need to disable the power management and also remove/add the ath11k_pci module when resuming from suspend.
```bash
# Add udev rule
mkdir -p /etc/udev/rules.d/
echo 'ACTION=="add", SUBSYSTEM=="net", KERNEL=="wl*", RUN+="/usr/bin/iw dev $name set power_save off"' > /etc/udev/rules.d/81-wifi-powersave.rules
```
System sleep hook to add remove the ath11k_pci module.
```bash
mkdir -p /etc/elogind/system-sleep/
cat << EOF >> /etc/elogind/system-sleep/ath11k.sh
#!/bin/bash
case $1/$2 in
  pre/*)
    # Put here any commands expected to be run when suspending or hibernating.
    ;;
  post/*)
    # Put here any commands expected to be run when resuming from suspension or thawing from hibernation.
    doas modprobe -r ath11k_pci; doas modprobe ath11k_pci
    ;;
esac
EOF
chmod +x /etc/elogind/system-sleep/ath11k.sh
```
We also want to let users do this without a password
```bash
nano /etc/doas.conf
# Add the following...
permit  nopass  :wheel  as root  cmd /usr/bin/modprobe
```
## Video Drivers
There is a strange issue with the video on resume that can be sped up by adding the following to the kernel command line.[^1]
[^1]: https://forums.lenovo.com/t5/Fedora/drm-amdgpu-job-timedout-amdgpu-ERROR-ring-sdma0-timeout/m-p/5227959?page=3
```bash
amdgpu.ppfeaturemask=0xfff7ffff amdgpu.vm_update=3
```
