# Compile  Linux Kernel for Cape4All

Christopher Obbard 13/03/2018
Please mail any questions to     chris <at> 64studio.com


Download & Patch kernel sources:
```
chris@pc:~$ git clone -b linux-4.14.y git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
chris@pc:~/linux-stable$ git apply 0001-add-cape4all-driver-v1.patch
```

Configure Kernel:
```
chris@pc:~/linux-stable$ make distclean
chris@pc:~/linux-stable$ ARCH=arm make omap2plus_defconfig
chris@pc:~/linux-stable$ ARCH=arm make menuconfig
```

Enable the CONFIG_SND_SOC_BONEBLACK_AUDIO_EXTENSION option:
```
 > Device Drivers > Sound card support > Advanced Linux Sound Architecture > ALSA for SoC audio support > <M> SoC Audio support for Boneblack Audio Extension
```

This is also required for the Boneblack board:
```
 > File systems > Native language support > <*> ASCII (United States)
```

Cross-compile Kernel, Modules and Device Tree:
```
 > chris@pc:~/linux-stable$ ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j6 zImage modules dtbs
```
