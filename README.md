sound_play
=========

Theses are the setup instructions used to configure sound_play to work with a speaker connected to a USB sound card on a Raspberry Pi 3.

## Clone repository

Clone this repository into a catkin workspace and build it.

`cd ~/catkin_ws/src && git clone https://github.com/icclab/audio_common`
`cd .. && catkin_make --pkg sound_play`

## Install and fix dependencies

- python-pygame
- festival
- festvox-don
- alsa-base
- alsa-tools

`sudo apt-get install python-pygame festival festvox-don alsa-base alsa-tools`
`rosdep install sound_play`

## Add user to audio group

Replace $USER with desired user name.

`sudo usermod -a -G audio $USER`

A reboot is necessary.

`sudo reboot`

Double check the command below works without sudo.

`aplay -l`

## Check that the sound card is recognized by the kernel

`cat /proc/asound/cards`

Your card should be in the list. In this case the USB sound card shows up as *USB-Audio - C-Media USB Headphone Set*. Make note of the number in front of the card, it will be used to tell alsa where to play sound from.

## Check ALSA modules

`cat /proc/asound/modules`

One of them should should up as *snd_usb_audio*

## Remove PulseAudio (Optional)

PulseAudio tends to interfere with ALSA.

`sudo apt-get remove pulseaudio`

## Disable the internal sound card

Edit the file /boot/config.txt and change *dtparam=audio=on* to *dtparam=audio=off*.

A reboot is necessary.

`sudo reboot`

Double check above worked by checking the sound modules

`cat /proc/asound/modules & cat /proc/asound/cards`

## Configure ALSA

User config in ~/.asoundrc (assumes USB sound card has index 1):

``` bash
pcm.!default {
        type hw
        card 1
}

ctl.!default {
        type hw
        card 1
}
```

System config in /usr/share/alsa/alsa.conf:

Replace *defaults.ctl.card 0* with *defaults.ctl.card 1* and *defaults.pcm.card 0* with *defaults.pcm.card 1*.

A reboot is necessary.

`sudo reboot`

## Test audio system

Play white noise on Card-1.

`speaker-test -c2 -D hw:1,0`

## Specify Device via ROS Param

Besides setting default device as system wide settings, you can also specify audio device via `rosparam`:

``` xml
<launch>
  <node name="soundplay_node" pkg="sound_play" type="soundplay_node.py">
    <param name="device" value="hw:1,0" />
  </node>
</launch>
```

or simply run: `rosrun sound_play soundplay_node.py _device:="hw:1,0"`

In the launch file above, `~device` parameter is set to `hw:1,0`, which tells `soundplay_node` to use audio device No. `0` connected to audio card No.`1`.

## Useful References

+ https://raspberrypi.stackexchange.com/questions/80072/how-can-i-use-an-external-usb-sound-card-and-set-it-as-default
+ http://wiki.ros.org/sound_play

## Troubleshooting

+ http://wiki.ros.org/sound_play/Troubleshooting

