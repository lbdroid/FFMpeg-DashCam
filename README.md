# Dashcam for CRAPPY Chinese car radios

This is ~~not~~ now(!!) free software.

~~Read the project Wiki before doing anything else. It describes licensing and distribution limitations.
https://github.com/lbdroid/FFMpeg-DashCam/wiki~~

NEW: The project is now OPEN SOURCE!<br />
picamd: https://github.com/lbdroid/picamd<br />
frontend: https://github.com/lbdroid/dashcam_frontend<br />
(picamd runs on the raspberry pi or other computer running linux, frontend runs on android device like car radio)


# NEW VERSION June 26, 2017, Updated July 10, 2017

WARNING: These instructions are LONG, but not difficult. They're just detailed. However, you still do need to have your own brain in order to understand the details. Please just read through carefully and try to understand. If there is something that is completely unclear, or doesn't work, or just doesn't make any sense *after an appropriate google search for solutions*, then by all means, open a new issue in the "Issues" tab at the top of the screen.


Because of some very serious defects intentionally introduced into the kernel of certain chinese car radios that completely disables the use of "unapproved" USB peripherals, including, specifically, any known UVC camera, it has been necessary to come up with a workaround.

The good news is that the workaround is infinitely superior to the original solution anyway!
1) It is UNIVERSAL, rather than being limited to a specific brand of crappy chinese car radios,
2) It can even run on a phone/tablet you have running in your car.
3) While it does require additional hardware, that additional hardware is CHEAP (only $10)

What you will need;
1) Some android device running in your car. Like a chinese car radio, a tablet, or your cell phone.
2) A Raspberry Pi Zero W, or a Pi 3B. May also work with ANY other RPi, if you plug in a wifi adapter.
3) An SDCARD of at least ~~8 GB~~ 4 GB (though I recommend bigger, note that ~2 GB are used for the OS). Pay attention to the quality of the card, you're looking for one that talks about capturing HD videos. I personally use Sandisk Endurance 64 GB. The reason I don't add this to the "additional hardware" cost, is because you need an sdcard anyway.
4) A CAMERA. Either a UVC camera, or a RPi camera. OR BOTH!!! It will support at least two cameras, possibly more -- I have only tested with 2 cameras. I recommend VERY WIDE ANGLE ("fisheye") cameras. This is for a dashcam, you aren't going for "pretty", your objective is to capture as much as possible in case some drunken meat head crashes into you! NOTE: If you go with a UVC camera, you will also need an "OTG CABLE" -- as low as "$0.99 with free shipping" https://www.amazon.com/Wblue-Adapter-Function-Samsung-BlackBerry/dp/B00Y2OTR72 -- or make one by soldering a micro-usb end onto the camera wire.

The software:
1) Pi-Dashcam.apk
Use DashCam 2.0.0.apk from this repository.

2) The sdcard image.
https://drive.google.com/open?id=0B7EPFZ3mUYvHQUdpSC16QzZUWUk

How-to:
1) Install the APK. When you run it, you will find 2 tabs. Files tab shows you existing recordings, if the file is "checked", then it is protected from the reaper. Tap to play it back in some video player (I suggest VLC). The green "send" button in the corner will upload any stored GPS logs. Then there is the settings tab, I suggest enabling "auto-start". If you want it to automatically trigger a "send-to" (for instance, send to google drive to back up important evidence) when you "protect" a file, check the "auto share" switch. If you want to log your GPS, check that switch, if you want the gps logs to be stored on the rpi, check that switch. The last switch is to automatically enable wifi hotspot, which the rpi will need to use to communicate with your android device -- note: Yes, I am aware of the warning. I do mean it. No, the recommended program to use in place of this check is not yet available. IGNORE the warning for now. Don't forget to set up the hotspot with an SSID and password. There will also be a notification with a record or stop button that shows the status of the recording.
2) Install the RPi IMG. Ok, so this is a touch more complex than just installing an APK on an Android device. On Linux, I would run this; "zcat rpicam1.img.gz | dd of=/dev/mmcblk0". I would imagine that Apple would be fairly similar to that. I don't really have any idea where to start suggesting how to do that with mswin. The RPi people have this about windows, which should work (but don't forget to gunzip the file first!!!) https://www.raspberrypi.org/documentation/installation/installing-images/windows.md
~~2b) You will also want to expand the 3rd partition to fill the entire balance of the sdcard. You can use gparted from just about any linux distribution, or the gparted livecd http://gparted.org/livecd.php. If you're a tiny bit more ambitious, you could do it from within the rpi itself. Just delete the third partition, create a new one that fills the remaining space, mkfs.ext4 it, and create a directory in its root called "protected". Hint: If you do it from the rpi itself, make sure to kill picamd first.~~ No longer needed, this happens automatically during first boot.
~~3) Fix the wifi settings so that it will log in to your hotspot. If you can mount the second ext4 partition, you can find /etc/wpa_supplicant/wpa_supplicant.conf. Just change the ssid and psk to match your own and save it. If you do it from the rpi, you will need to do it with a keyboard and monitor, since you don't yet have any network connectivity (well, if you have a pi with an ethernet port, you could log in via ssh for this step).
a) log into the pi. The username is "pi" and the password is "raspberry".
b) sudo mount -o remount,rw /ro
c) sudo nano /ro/etc/wpa_supplicant/wpa_supplicant.conf
Then make the change, save, exit, and...
d) sudo reboot~~ No longer needed, this process has been streamlined somewhat.
3) Update wifi settings on Pi...
a) Set up a wifi AP or hotspot using SSID=PIWIFI, password=defaultpassword, and key_mgmt=WPA_PSK
b) Power on Pi
PATH0: You're good to go if you leave your car radio's hotspot set up with those specs.
PATH1: ssh -l pi pizwcam.local with password "raspberry", and sudo mount -o remount,rw /ro; sudo vim /ro/etc/wpa_supplicant/wpa_supplicant.conf, edit the SSID and password to your preference, save, sudo reboot.
PATH2: (For next version of APK not yet available) In the settings tab of the Android application, hit the wifi setup button, enter the new SSID and password, and send it to the pi. Pi will save it and reboot. Note: Android device and Pi must be on the same wifi network. Android device may be the hotspot.

Now about #4.....
The Pi side software has been updated to make it possible to send camera specs and receive commandline over HTTP. At some point (soon), I'm going to update the Android application to make this all a bunch of clickable choices. Make it much easier.

4) Set up the camera configuration for recording. You will need to log into the pi (ssh) and get a look at your cameras.
a) First off, you can see all the camera interfaces by "ls /dev/video\*". If you have a raspberry pi CSI camera, it will show up as /dev/video0, with USB cameras appearing as /dev/video1, /dev/video2, etc. If you have no CSI camera, the USB cameras will start at /dev/video0. Some cameras may have multiple entries. Don't be alarmed by this, just pick the most appropriate entry.
b) Check its output capabilities; "ffmpeg -f v4l2 -list_formats all -i /dev/videoX" with "X" replaced by the number corresponding to the camera you are looking at.
--- I suggest only picking cameras that are flagged as "Compressed". Raw video is incredibly large and will not make your sdcard happy, assuming that it can keep up at all. Perhaps at extremely low resolutions that make it a challenge to determine if that was a car, or an iceberg.
--- You are looking for two things from the output, the code, and the supported resolutions. By code, I mean "mjpeg" or "h264". h264 compresses a lot better than mjpeg, but can introduce more compression artifacts. Also, because of the nature of h264, if you are recording from more than 1 camera, you are best off restricting yourself to just ONE h264 stream. This is because videos have to be split on key frames, and on multiple streams, you'll never get the key frames to line up, which means that the second stream will have several broken frames at the start of each segment. For the resolution, these could be output as distinct configurations (like 1280x720, 640x480), or as RANGES. The RPi CSI driver gives ranges and looks like this; "{32-2592, 2}x{32-1944, 2}" -- basically, you can pick anything that satisfies the ranges, for instance, 1280x720.
--- The configuration file is located at /mnt/data/CONFIG, note that this filesystem is mounted ro except when actually recording video. If you look at the default config file, it has a "params=" line that looks like this; "params=-f video4linux2 -input_format h264 -video_size 1280x720 -i /dev/video1 -c:v copy" -- this is a PARTIAL ffmpeg commandline. What it says, is to select an h264 stream with resolution 1280x720 from /dev/video1, and just COPY it (do not re-encode it). You will have to modify this to match YOUR camera!
--- Example 2-camera configuration;
"-f video4linux2 -input_format h264 -video_size 1280x720 -i /dev/video2 -f video4linux2 -input_format mjpeg -video_size 1280x720 -i /dev/video0 -c:v copy -map 0 -map 1"
--- Example 2-camera configuration with the CSI camera recording at 10 fps to reduce file size:
"-f video4linux2 -input_format h264 -video_size 1280x720 -i /dev/video2 -f video4linux2 -input_format mjpeg -video_size 1280x720 -framerate 10 -i /dev/video0 -c:v copy -map 0 -map 1"
Note that not all cameras permit alteration of the framerate. The Pi CSI camera driver does.
--- Example 2-camera + sound configuration;
"-f video4linux2 -input_format h264 -video_size 1280x720 -i /dev/video2 -f video4linux2 -input_format mjpeg -video_size 1280x720 -i /dev/video0 -f oss -i /dev/dsp1 -ac 1 -c:a copy -c:v copy -map 0 -map 1 -map 2"
Note: the sound messes up the timestamps, so I don't recommend that configuration just yet, working on it ;)
--- You can TEST the commandlines by running them through ffmpeg from the commandline like this; "ffmpeg YOUR_PARAMETERS_HERE /home/pi/test.mkv" -- if it errors out, then you need to rework it. If it runs successfully, then copy the file to your computer and see if all your streams are playing correctly. Note: The test will be storing the video to a RAMDISK, so you can't transfer it by pulling out the sdcard. The ramdisk will also fill quickly, so press ctrl-c to stop it after several seconds. On Linux, the "scp" command can be used to retrieve the file, "scp pi@pizwcam.local:/home/pi/test.mkv ./"

What now? Right. Plug it all in.
My cameras;
I have one of these;
https://www.amazon.ca/ELP-Webcams-Surveillance-Customized-fisheye/dp/B0196BPZ1C

And one of these;
https://www.amazon.ca/SainSmart-Fish-Eye-Camera-Raspberry-Arduino/dp/B00N1YJKFS

First off, the ELP captures a much nicer image than the SainSmart. Its also a slightly wider angle (170^ vs 160^). Because of the superior image quality, I mount the ELP on the front, and the SainSmart on the back. I drive a pickup truck, which has a fairly short cab, making it possible to mount the CSI camera on the back with a 1M wire.

For mounting the ELP on the front, I actually screwed it into the front plastic cover of the rearview mirror. Completely invisible except for a thin wire running up from the mirror into the headliner.

The sainsmart is screwed into a box that is glued to the upper edge near the center of the rear window, its wire runs up into the headliner and forward. The pi is screwed into the inside of the garage door opener compartment. The pi power wire is router under the headliner and down the A-piller to the dashboard, plugged in to the radio's USB wire behind the glove compartment.

** TODO: **
1) Make this readme more readable.
2) Add some automation for the camera configurations (via the application). (IN PROGRESS)
~~3) See if there is some sensible way to move the wpa_supplicant to the /boot partition, which would make it practical to modify from windows. Or option 2: set a default SSID and PASSWORD that it will connect to, and configure the application to send an update to the rpi.~~ MOSTLY. Some minor update needed to Pi side.
~~4) Script a first-run process that generates a 3rd partition that fills the sdcard's entire space.~~ DONE
~~5) GPS logging and timekeeping on the rpi.~~ DONE
6) GPS serving to the car radio from the rpi (because the crappy chinese car radio GPS's barely work at all). (IN PROGRESS, PI SIDE DONE)
~~7) Figure out how to fix the timestamps when recording sound.~~ FIXED. Well, in a manner of speaking.

About the audio glitches... turns out that it wasn't actually a timestamp problem, even though it was spewing timestamp errors. The problem was CPU contention. I actually did end up starting to see the same problem with a multiple-video-stream recording. The problem actually had to do with the packet queue depth. It turns out that the ffmpeg default is to set a packet queue depth of 8 (packets). Well that is fine for reading from disk where the actual disk is an extension of the queue, or with a very fast CPU that isn't going to stall while other processes run... but not so good for capturing a live stream on a weakling CPU like this thing has. While we aren't maxing out the CPU by any stretch, it still has to swap the process in and out in order for other things to run.

So the solution was to crank up the queue length from 8 to 512, and that is done with the parameter "-thread_queue_size 512" added in front of every "-f" parameter in the ffmpeg commandline.

I am currently capturing a 30 fps 1280x720 h264 stream, 10 fps 1280x720 mjpeg stream, and an audio stream using this parameter set; -thread_queue_size 512 -f video4linux2 -input_format h264 -video_size 1280x720 -i /dev/video2 -thread_queue_size 512 -f video4linux2 -input_format mjpeg -video_size 1280x720 -framerate 5 -i /dev/video0 -thread_queue_size 512 -f oss -i /dev/dsp1 -ac 1 -c:a copy -c:v copy -map 0 -map 1 -map 2

And some more info to add to that... also added a gps log into the mix, and its holding steady. I'm starting to push the pi pretty hard though, at least when I'm also retrieving an already recorded video file. It might be a good idea to add in a rate limited for the file retrieval.

I'm going to add a nice(-20) for the ffmpeg thread on next build.

Have to add another feature, which is the ability to run extra user-defined commands immediately prior to starting up ffmpeg. Specifically, v4l2-ctl, which is needed to adjust the characteristics of the cameras. I'm needing this command for my own use, but others may require something more customized to their own needs; "/usr/bin/v4l2-ctl --set-ctrl=video_bitrate=2000000 --set-ctrl=compression_quality=10 --set-ctrl=rotate=180"

**UPDATE JULY 10th:**

I'm going to call the current state "BETA". Its definitely beyond Alpha, all the main features are implemented. There may be some complex configurations to make, but the software is workable and shouldn't place any burden on the crappy chinese car radio, and should more or less "work".

Note: If you are using a CSI camera on the pi, make sure to SHIELD the ribbon wire. The longer the ribbon, the more critical it is to shield it. Use something like aluminum duct tape from home depot in the FURNACE DUCTWORK section. Don't forget to electrically tie the aluminum tape to the car's chassis (negative).

The 2.0.0 Android application fixes a critical bug that I was restling with that was causing the car radio to perform a full shutdown on ignition off instead of going into standby. This was due to it being impossible to have the wifi hotspot shut down when the ignition was switched off. Typically, one registers a receiver for the SCREEN OFF intent, but these pieces of junk don't emit that intent. I also tried polling for the screen state, and whether the device was in an "interactive" state. Apparently, when the screen is actually off, it still thinks it is on, and it never goes non-interactive until it is too late to do anything (about 5 minutes AFTER ignition off).

I ended up having to implement a HACK to solve this problem, and the hack is this; watch the state of the BLUETOOTH adapter. When the bluetooth adapter powers off and remains powered off, I take that to indicate that the ignition is off, and use that information to decide to send a stop recording instruction to the pi, and then power off the wifi hotspot. It is worth noting that the crapware in these car radios automatically re-enables the bluetooth almost instantly when you manually turn off the bluetooth, so what I do is I check that the bluetooth remains turned off for 5 checks that are spaced out 5 seconds apart. This allows the user to turn bluetooth off as they might normally do to reset it when it gets goobered up, but without causing the recording or wifi hotspot to switch off when they do so.

I suspect that it will be necessary to run a customized (hacked) sofia server application, with the KILL EVERYTHING code disabled. Without that, it is likely that this software will get murdered before it has the opportunity to instruct the pi to stop recording and to shut off wifi hotspot.
