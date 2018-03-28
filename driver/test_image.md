# Instructions on how to use Cape4All test image

Christopher Obbard 13/03/2018
Please mail any questions to     chris <at> 64studio.com


Download the image:
```
chris@pc:~$ wget <image_url>
```

Find SD card path:
```
chris@pc:~$ sudo lsblk
```

Clear whole SD card with 0s:
```
chris@pc:~$ sudo dd if=/dev/zero of=/dev/sdX bs=4M
chris@pc:~$ sudo sync
```

Write the image to SD card (sync is important to flush the buffers)
```
chris@pc:~$ sudo dd if=cape4all_testimg_v3.img of=/dev/sdX bs=4M
chris@pc:~$ sudo sync
```

Calculate SD card checksum:
```
chris@pc:~$ sudo dd if=/dev/sdX bs=4M count=375 | md5sum
```



There is a 5V TTL-level serial port console available on the Beaglebone Black serial header at 115200 baud, 
or SSH/SCP (root enabled). The hostname is beaglebone if you have trouble finding the IP address.
Once the board boots, you can login with either:
```
root:toor  (root user), or
jack:jack  (user with sudo rights & audio realtime membership)
```

Note the Kernel supplied on SD card image is Vanilla Mainline 4.14.8 stable (only change being addition of the soundcard driver):
```
jack@beaglebone:~$ uname -a
> Linux beaglebone 4.14.8-dirty #5 SMP Fri Dec 29 16:54:26 GMT 2017 armv7l GNU/Linux
```

The soundcard is enabled and shows as one ALSA device:
```
jack@beaglebone:~$ aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: boneaudioext [boneaudioext], device 0: Boneblack Audio Extension multicodec-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
jack@beaglebone:~$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 0: boneaudioext [boneaudioext], device 0: Boneblack Audio Extension multicodec-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

Update Debian system & setup some more realtime parameters:
```
jack@beaglebone:~$ sudo apt-get update && sudo apt-get upgrade
jack@beaglebone:~$ sudo apt-get install cpufrequtils
jack@beaglebone:~$ sudo reboot
jack@beaglebone:~$ sudo cpufreq-set -g performance
jack@beaglebone:~$ sudo sysctl -w vm.swappiness=10
jack@beaglebone:~$ sudo sysctl -w fs.inotify.max_user_watches=524288
```

Restore Mixer configuration:
(the outputs are set to headphone with -6dB gain and the inputs are set to microphone with 0dB gain):
```
jack@beaglebone:~# sudo alsactl restore -f cape4all.alsactl.state
```

The controls in alsamixer are mapped in the following way: codec IC1 (MIC1&HP1)=C0, IC2 (MIC2&HP2)=C1 and IC3 (MIC3)=C2. More on this later.
```
jack@beaglebone:~$ alsamixer
```

Play some pink noise on all 8 channels (the driver must transmit to eight channels):
```
jack@beaglebone:~$ speaker-test -c 8 -r 96000 -F S32_LE
```

Record 8-channel WAV from microphones (bias link must be fitted to jumpers between pins 1-2):
```
jack@beaglebone:~$ arecord -c 8 -r 96000 -f S32_LE test.wav
```

Play back recorded sound:
```
jack@beaglebone:~$ aplay -vv test.wav
```

Start JACK server (on separate terminal, with multiple options available):
```
jack@beaglebone:~$ jackd -R -P95 -dalsa -dhw:boneaudioext -r48000 -p256 -n2
```

Confirm JACK sample rate:
```
jack@beaglebone:~$ jack_samplerate
48000
```

Confirm Jack routes (the mapping of these is set through alsamixer, I have included the defaults below)
```
jack@beaglebone:~$ jack_lsp
> system:capture_1      # IC1 Left Playback Data, mapped to LHP_6 (HP1 Left)
> system:capture_2      # IC2 Left Playback Data, mapped to LHP_7 (HP2 Left)
> system:capture_3      # IC3 Left Playback Data, mapped to LHP_8 (Not committed on PCB)
> system:capture_4      # (Not connected)
> system:capture_5      # IC1 Right Playback Data, mapped to RHP_6 (HP1 Right)
> system:capture_6      # IC2 Right Playback Data, mapped to RHP_7 (HP2 Right)
> system:capture_7      # IC3 Right Playback Data, mapped to RHP_8 (Not committed on PCB)
> system:capture_8      # (Not connected)
> system:playback_1     # IC1 Left Record Data, mapped to LINN6 (MIC1 Left)
> system:playback_2     # IC2 Left Record Data, mapped to LINN7 (MIC2 Left)
> system:playback_3     # IC3 Left Record Data, mapped to LINN8 (MIC3 Left)
> system:playback_4     # (Not connected)
> system:playback_5     # IC1 Right Record Data, mapped to RINN6 (MIC1 Right)
> system:playback_6     # IC2 Right Record Data, mapped to RINN7 (MIC2 Right)
> system:playback_7     # IC3 Right Record Data, mapped to RINN8 (MIC3 Right)
> system:playback_8     # (Not connected)
```

Play recorded wav through Jack
> jack@beaglebone:~$ mplayer -ao jack test.wav

In another terminal connect ports:
```
jack@beaglebone:~$ jack_connect system:capture_1 system:playback_1
jack@beaglebone:~$ jack_connect system:capture_2 system:playback_2
jack@beaglebone:~$ jack_connect system:capture_3 system:playback_3
jack@beaglebone:~$ jack_connect system:capture_4 system:playback_4
jack@beaglebone:~$ jack_connect system:capture_5 system:playback_5
jack@beaglebone:~$ jack_connect system:capture_6 system:playback_6
jack@beaglebone:~$ jack_connect system:capture_7 system:playback_7
jack@beaglebone:~$ jack_connect system:capture_8 system:playback_8
```

(jackd may be closed with Ctrl-D)
