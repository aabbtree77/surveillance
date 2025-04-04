> "It's the how that always stops me. Knives and razors are out for me. I have this thing about blood... Pills? I don't even have a doctor, let alone the feelgood variety. Carbon monoxide sounds good in theory, but in practice it always feels like too much like work. Getting a tube the right size to fit the exhaust pipe and long enough to reach in the side window, all the rest of it. It's like you want to kill yourself, that's fine, but first you have to remodel the basement. You end up thinking, the hell with it, I'll do it tomorrow."
>  
> \- Michael Dibdin (Thanksgiving, 2000)

## Introduction

It is easy to get lost when trying to find the right software for an IP camera. Proprietary vs FOSS, cloud or local, "video devices", protocols, configuration, errors...

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

Here is a simple memo to do automatic motion detection and video storage on Ubuntu 22.04 for a basic ONVIF-compatible or generic RTSP camera assumed to be within the same LAN as the storage PC.

## Setup

Get the network address of the camera somehow:

```bash
sudo apt update
sudo apt install nmap
nmap -sP 192.168.0.0/24
```

Test the camera. In my case the address is 192.168.0.100:

```bash
sudo apt install ffmpeg
ffplay rtsp://admin:admin@192.168.0.100:554/
```

This should display a real time video stream of the camera. The admin:admin part is likely not essential, 
`ffplay rtsp://192.168.0.100:554/` might do.

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

The tilde gets interpreted literally at times. From now on I will use the explicit path which is `/home/tokyo/` in my case.

Create a folder for the video and log files:

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

Create `/home/tokyo/.motion/motion.conf` with the following content:

```text
# No daemon, run in foreground
daemon off

# Log file
log_file /home/tokyo/Videos/motion.log
log_level 6

# Camera from RTSP stream
netcam_url rtsp://admin:admin@192.168.0.100:554/
# netcam_userpass admin:admin  # (optional if in URL above)

# Disable web control
webcontrol_port 0
stream_port 0

# Motion detection settings
threshold 1500
minimum_motion_frames 5
event_gap 30

# Video output
ffmpeg_output_movies on
ffmpeg_filename %Y%m%d_%H%M%S_motion
target_dir /home/tokyo/Videos
output_pictures off
```

Run the whole thing:

```bash
motion -c /home/tokyo/.motion/motion.conf

[0:motion] [NTC] [ALL] conf_load: Processing thread 0 - config file /home/tokyo/.motion/motion.conf
[0:motion] [ALR] [ALL] conf_cmdparse: "ffmpeg_output_movies" replaced with "movie_output" after version 4.1.1
[0:motion] [ALR] [ALL] conf_cmdparse: Unknown config option "ffmpeg_filename"
[0:motion] [ALR] [ALL] conf_cmdparse: "output_pictures" replaced with "picture_output" after version 4.1.1
[0:motion] [NTC] [ALL] motion_startup: Logging to file (/home/tokyo/Videos/motion.log)

```

Use Ctrl+C to stop streaming/surveillance.

## Remarks

* event_gap 30  - more than 30s. of non-motion completes/stops the video of the event.
* Lights on/off or merely changing the intensity is also an event.
* Roughly 3MB per 10s with the current settings, but this may vary.
* Running `motion -c...` again after Ctrl+C appends the old `motion.log` with new events.
 
