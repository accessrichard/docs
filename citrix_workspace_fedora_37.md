### Getting Citrix Workspace working on Fedora 37 with Pipewire

### Install Citrix Workspace
*  Download the Citrix [Receiver for Linux (x86_64)](https://www.citrix.com/downloads/citrix-receiver/linux/receiver-for-linux-latest.html)
 - I did not install the citrix usb package since audio and microphone should be redirected by default.

```
dnf install ~/Downloads/<<citrixfilename.rpm>>
```
### Install Certificate

* Go to your company's workspace site and download the root certificate. 
 - In Firefox click the padlock icon to the left of the url, then "Connection is Secure" then "More Information"
 - Hit View Certificate
 - In the Miscellaneous section of the first/root certificate click PEM download.
 - Copy that pem file to the following folder: /opt/Citrix/ICAClient/keystore/cacerts/ 
 - Run /opt/Citrix/ICAClient/util/ctx_rehash 
 - Official instructions are [here](https://support.citrix.com/article/CTX231524/citrix-workspace-app-for-linux-how-to-trust-a-ca-certificate)

### Fix Audio Not Working Due To Fedora Switching to Pipewire - Use Pipewire Pulseaudio Compatability Layer

*  (IF you login and audio is broken) Citrix doesn't support pipewire which Fedora 37 swapped to. In order to fake out citrix so it works with pipewire, in usr/bin make a file called pulseaudio and place the following in it:
```
sudo nano /usr/bin/pulseaudio 
```
Paste this in the file:
```
 echo pulseaudio 16.1
```

* Set the file to executable

```
chmod +x /usr/bin/pulseaudio
```

This is because citrix only supports pulseaudio and not pipewire. Pipewire has a compatability layer pipewire-pulseaudio that will work however the citrix client does not detect this.

### Fix Audio Not Working By Swapping Back To PulseAudio 

* If faking out pulseaudio (instructions above) does not work or you have issues with pipewire:
[You can run the following](https://discussion.fedoraproject.org/t/how-do-i-switch-from-pulseaudio-to-pipewire-and-back/78093) to swap back from pipewire to pulseaudio. 

You have to run the second command on user profile startup. **That is every time you log into Gnome**:
```
sudo dnf swap --allowerasing pulseaudio pipewire-pulseaudio
systemctl --user --now disable wireplumber
```

To re-enable if it doesnt work, do the following. 
```
sudo dnf swap --allowerasing pipewire-pulseaudio pulseaudio
```

### Enable hotkey support by allowing wayland to pass hotkeys to the Citrix app. In a terminal:

```
gsettings set org.gnome.mutter.wayland xwayland-grab-access-rules "['Wfica']"
gsettings set org.gnome.mutter.wayland xwayland-allow-grabs true
```

### Fix DPI Scaling for 4k screens when logging in, Choppy Audio, and Invisible Mouse Issues
### Reference: [Citrix Linux Config Docs](https://docs.citrix.com/en-us/citrix-workspace-app-for-linux/configure-xenapp.html)

In your profile directory:

```
nano ~/.ICAClient/wfclient.ini
```

Under: [WFClient] make sure you have the DPIMatchingEnabled so that windows is scaled when you log in:

```
DPIMatchingEnabled=True ; This setting allows you to login to windows from a 4k monitor without having to adjust the scaling everytime.

```

Under: [Thinwire3.0] I have the following so that i can see the cursor better in the remote screen. e.g. (White cursor and white background)
```
InvertCursorEnabled=True
```


### Fix Choppy Audio

```
nano /opt/Citrix/ICAClient/config/module.ini
```

Under [ClientAudio] enable echo cancellation, and jitter buffer for smoother audio:

```
[ClientAudio]
DriverName=VDCAM.DLL
UseThread=TRUE
ThreadQueueSize=65536
AudioLatencyControlEnabled=FALSE ;Set to false
JitterBufferEnabled=TRUE ; enable
EnableAdaptiveAudio=TRUE ; enable
AudioRedirectionV4=TRUE ; should be on by default
EnableEchoCancellation=TRUE ;enable
PlaybackDelayThreshV4=100 ; you can leave this at the default of 50 if you want.
AudioTempLatencyBoostV4=175 ;you can leave this at the default of 100  if you want.
```


