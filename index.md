# Links
- [This web page](https://itrobinson.github.io/novian/) on GitHub Pages.
- Anne Gentle's [instructions](https://www.docslikecode.com/) on how to use GitHub Pages.
- The ArchWiki [WirePlumber instructions](https://wiki.archlinux.org/title/WirePlumber)

# Get the hardware
You need

- a Raspberry Pi
- something that can play audio

The device which plays audio could be an amplifier, headphones or a Bluetooth speaker.

# Install the operating system
https://www.raspberrypi.com/software/

Install Raspberry Pi OS Lite (64-bit) on a Secure Digital (SD) card.

When doing the installation, ensure that Secure Shell access is enabled and set up keys for login.

# Log in
Connect to your new internet radio by typing

```
ssh me@myradio
```

where `me` is your username and `myradio` is the hostname. You set both of these when you created the SD card.

# Check version
Check which version of Debian your Raspbian installation is based on.

```
cat /etc/os-release
```

This will provide information about Debian, the operating system that Raspbian is based on. These instructions apply to Debian 12 (codename Bookworm) and later.

# Pipewire
Check that Pipewire is running

```
systemctl --user status pipewire.service
```

May need to install Pipewire and Wireplumber here.

# Play a test sound
Play some noise to test the sound output

```
pw-play /usr/share/sounds/alsa/Noise.wav
```

# Check the volume
```wpctl status```

```wpctl xx set-volume 100%```

where `xx` is the number of the sink.

# Music Player Daemon
Install the Music Player Daemon.

```
sudo apt --assume-yes install mpd
```

The Music Player Daemon is by default configured to use the Advanced Linux Sound Architecture (ALSA). Disable ALSA and tell it to use PipeWire instead. Edit the file `/etc/mpd.conf` and delete (or comment out) the ALSA audio output. Then add a new one for PipeWire that looks like this

```
audio_output {
        type    "pipewire"
        name    "PipeWire Sound Server"
}
```

It may also be necessary to change the name of the user from the default to whatever use is running PipeWire.

# Bluetooth
Run the following command

```
bluetoothctl
```

(There is no need for `sudo` here.)

Put your speaker into pairing mode. Then do

```
scan on
```

A list fo devices will appear. Figure out which is your speaker. If it is not obvious more information is available

```
info mac
```

where `mac` is the address of the device.

Then pair with the device and trust it.

```
pair mac
trust mac
```

Switch scanning off

```
scan off
```

Now it is ready to connect

```
connect mac
```

Look for a message that looks like this

```
[CHG] Device xx:xx:xx:xx:xx:xx ServicesResolved: yes
```

which incidactes that the Bluetooth audio is connected.

# Play on two devices simultaneously
By defulat all audio output switches to Bluetooth when the Bluetooth speaker is connected. This behaviour can be changed.

Create a new node and give it a name. Let's call it *Groovie*

```
pw-cli create-node adapter '{ factory.name=support.null-audio-sink node.name="Groovie" node.description="Groovie" media.class=Audio/Sink object.linger=true audio.position=[FL FR] }'
```

Now connect the audio sinks to the new device.

```
wpctl inspect xx
```

where `xx` is the number of a *sink*. Look for the `node.name` in the output. Suppose the `node.name` is X. Then connect playback to

```
pw-link "Groovie:monitor_FL" X:playback_FL
pw-link "Groovie:monitor_FR" X:playback_FR
```
