---
{"dg-publish":true,"dg-path":"HowTo/matebook","permalink":"/how-to/matebook/","tags":["notes/fern"],"noteIcon":"fern","created":"2025-06-26 17:44","updated":"2025-06-26 18:31"}
---

These are the steps that are required to fix the sound on my HUAWEI matebook. Before them I just had a **Dummy Device** shown in the sound settings:

1. Install the following packages system:
```bash
# You can use pacman to install the packages below
yay -S intel-ucode sof-firmware

# Maybe you also require the following packages
# You should first try it without those
sudo pacman -S alsa-utils alsa-tools pavucontrol pulsemixer
```

2. Create a file in `/etc/modprobe.d/intelDSP.conf` with the following content:
```
options snd-intel-dspcfg dsp_driver=1
```

3. Reboot