> "It's the how that always stops me. Knives and razors are out for me. I have this thing about blood... Pills? I don't even have a doctor, let alone the feelgood variety. Carbon monoxide sounds good in theory, but in practice it always feels like too much like work. Getting a tube the right size to fit the exhaust pipe and long enough to reach in the side window, all the rest of it. It's like you want to kill yourself, that's fine, but first you have to remodel the basement. You end up thinking, the hell with it, I'll do it tomorrow."
>  
> \- Michael Dibdin (Thanksgiving, 2000)

## Introduction

It is easy to get lost when trying to find [the right software for an IP camera](https://www.youtube.com/watch?v=awQgyKJFYZo&t=84s&ab_channel=TheHookUp). Proprietary vs FOSS, cloud or local, GUI vs command line, "video devices", protocols, configuration, errors...

<table align="center">
    <tr>
    <th align="center"> CCTV Camera "viSar" (April 2025)</th>
    </tr>
    <tr>
    <td>
    <img src="./images/visar-2025-04-04.jpg"  alt="viSar CCTV surveillance IP camera" width="100%" >
    </td>
    </tr>
</table>

Here is a memo how to do automatic motion detection and video storage on Ubuntu 22.04 for a basic ONVIF-compatible or generic RTSP camera assumed to be within the same LAN as the storage PC. Everyting is FOSS and under complete control.

## Test the Network and Camera

Get the network address of the camera somehow:

```bash
sudo apt update
sudo apt install nmap
nmap -sP 192.168.0.0/24
```

In my case, this produces

```bash
Starting Nmap 7.80 ( https://nmap.org ) at 2025-04-06 12:03 EEST
Nmap scan report for _gateway (192.168.0.1)
Host is up (0.0012s latency).
Nmap scan report for 192.168.0.100
Host is up (0.0096s latency).
Nmap scan report for go (192.168.0.101)
Host is up (0.000083s latency).
Nmap scan report for 192.168.0.102
Host is up (0.0024s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 3.02 seconds
```

The camera address turns out to be 192.168.0.100, which is revealed upon testing with

```bash
sudo apt install ffmpeg
ffplay rtsp://admin:admin@192.168.0.100:554/
```

This should display a real-time video stream of the camera. The admin:admin part is likely not essential, 
`ffplay rtsp://192.168.0.100:554/` might do. 

Initially, it is also possible to get the error:

```text
Connection to tcp://192.168.0.100:554?timeout=0 failed: Connection refused
rtsp://admin:admin@192.168.0.100:554/: Connection refused
```

Plug/unplug the camera, turn the camera/router power on/off until ffplay shows the video stream.

## Install Motion

Clean up the previous versions of [Motion](https://motion-project.github.io/3.4.1/motion_guide.html):

```bash
pkill motion
sudo apt purge motion
sudo apt autoremove --purge
sudo rm -rf /etc/motion/
sudo rm -rf /var/log/motion/
sudo rm -rf /var/lib/motion/
rm -rf ~/.motion
rm -f ~/Videos/motion.log
rm -f ~/Videos/*_motion.avi
sudo systemctl disable motion.service
sudo systemctl stop motion.service
sudo systemctl mask motion.service
```

The tilde `~` is not reliable for configuration. From now on I will use the explicit home path which is `/home/tokyo/` in my case.

Create the folder for the video and log files:

```bash
mkdir -p /home/tokyo/Videos
chmod 700 /home/tokyo/Videos # Restrict access to the user only
```

Install Motion:

```bash
sudo apt update
sudo apt install motion
motion -h
motion Version 4.3.2, Copyright 2000-2019 Jeroen Vreeken/Folkert van Heusden/Kenneth Lavrsen/Motion-Project maintainers
Home page :	 https://motion-project.github.io/
```

## Configure and Run Motion

Create `/home/tokyo/.motion/`.

Two configuration options are provided here. 

The first is minimal, simply place `motion.conf` from `config_minimal` inside `/home/tokyo/.motion/`. It should work, but there will be some deprecation warnings, and a few settings missing.

The second one involves placing both `motion.conf` and `cam0.conf` from `config_advanced` inside `/home/tokyo/.motion/`.

Run the whole thing (config_advanced):

```bash
motion -c /home/tokyo/.motion/motion.conf
[0:motion] [NTC] [ALL] conf_load: Processing thread 0 - config file /home/tokyo/.motion/motion.conf
[0:motion] [NTC] [ALL] config_camera: Processing camera config file /home/tokyo/.motion/cam0.conf
[0:motion] [NTC] [ALL] motion_startup: Logging to file (/home/tokyo/Videos/motion.log)
```

Use Ctrl+C to stop the surveillance. Just in case, kill it:

```bash
ps aux | grep motion
kill -9 12345 (process id from ps aux)
```
or
```bash
pkill -f motion
kill -9 $(pidof motion)
```

Check the log file `/home/tokyo/Videos/motion.log`:

```text
[0:motion] [NTC] [ALL] [Apr 06 13:19:50] motion_startup: Motion 4.3.2 Started
[0:motion] [NTC] [ALL] [Apr 06 13:19:50] motion_startup: Using default log type (ALL)
[0:motion] [NTC] [ALL] [Apr 06 13:19:50] motion_startup: Using log type (ALL) log level (NTC)
[0:motion] [NTC] [ENC] [Apr 06 13:19:50] ffmpeg_global_init: ffmpeg libavcodec version 58.134.100 libavformat version 58.76.100
[0:motion] [NTC] [ALL] [bal. 06 13:19:0] translate_init: Language: English
[0:motion] [NTC] [ALL] [bal. 06 13:19:0] motion_start_thread: Camera ID: 1 is from /home/tokyo/.motion/cam0.conf
[0:motion] [NTC] [ALL] [bal. 06 13:19:0] motion_start_thread: Camera ID: 1 Camera Name: VISAR Service: rtsp:
[0:motion] [NTC] [ALL] [bal. 06 13:19:0] main: Waiting for threads to finish, pid: 13698
[1:ml1:VISAR] [NTC] [ALL] [bal. 06 13:19:0] motion_init: Camera 1 started: motion detection Enabled
[1:ml1:VISAR] [NTC] [VID] [bal. 06 13:19:0] vid_start: Opening Netcam RTSP
[1:ml1:VISAR] [NTC] [NET] [bal. 06 13:19:0] netcam_rtsp_connect: Normal resolution: Camera (VISAR) connected
[1:ml1:VISAR] [CRT] [NET] [bal. 06 13:19:0] motion_init: Substream not available.  Image sizes not modulo 16.
[1:ml1:VISAR] [NTC] [ALL] [bal. 06 13:19:0] image_ring_resize: Resizing pre_capture buffer to 1 items
[2:nc2:VISAR] [NTC] [NET] [bal. 06 13:19:0] netcam_rtsp_handler: Normal resolution: Camera handler thread [2] started
[2:nc2:VISAR] [NTC] [NET] [bal. 06 13:19:0] netcam_rtsp_connect: Normal resolution: Camera (VISAR) connected
[1:ml1:VISAR] [NTC] [ALL] [bal. 06 13:19:0] image_ring_resize: Resizing pre_capture buffer to 5 items
[1:ml1:VISAR] [NTC] [EVT] [bal. 06 13:20:0] event_newfile: File of type 8 saved to: /home/tokyo/Videos/2025-04-06_13-20-02.mkv
[1:ml1:VISAR] [NTC] [ALL] [bal. 06 13:20:0] motion_detected: Motion detected - starting event 1
[1:ml1:VISAR] [NTC] [ALL] [bal. 06 13:21:0] mlp_actions: End of event 1
```

## cam0.conf 

* `event_gap 30` - more than 30s. of non-motion completes the video of an event. Non-motion moments during the event are not stored.

* `threshold 1500` - quite sensitive for 2592 x 1944, less so for 640 x 480. Lights on/off or changing the intensity will trigger an event. 1500 @ 2592 x 1944 will trigger an event even from tiny motion reflected from the surface of a matte/glossy door. Use `10000` to make it insensitive to such changes.

* `noise_tune on` - dynamically adjusts the detection based on background noise, might be needed to avoid insect motion-based triggering. One may have to experiment with `noise_level 32` too.

* Roughly 3MB per 10s for 640 x 480, and 3MB per 1s for 2592 x 1944, but this may vary.

* `lightswitch 40` and `smart_mask_speed 5` - ignores sudden light changes (due to clouds or headlights).

* `movie_filename %Y-%m-%d_%H-%M-%S` leads to filenames such as `2025-04-06_13-20-02.mkv`. One can use text in names directly, e.g. `area51_%Y-%m-%d_%H-%M-%S` will prepend `area51_`. `%v` can be used as an event counter, but it is not recommended to name files with it. Running `motion -c...` again after Ctrl+C may restart the counter, but it will append the old `motion.log` with new events.

* `mask_file`. I have not applied it, but one can create a B/W mask `mask.pgm` and add it to `cam0.conf`:

    ```text
    mask_file /home/tokyo/.motion/mask.pgm
    ```
    
    Black color = ignore motion, white color = detect motion. This might enable one to ignore the sky or trees, or save on storage by defining a region of interest.
    
    
