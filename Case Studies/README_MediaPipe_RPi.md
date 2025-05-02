# Case Study: MediaPipe Object Detection on Raspberry Pi

##  Overview

This case study outlines the setup and execution of **Google’s MediaPipe object detection** framework on a **Raspberry Pi 4** using a USB webcam. MediaPipe is an open-source, cross-platform framework for building multimodal machine learning pipelines.

The project is divided into three main stages:
1. Preparing the Raspberry Pi for computer vision tasks.
2. Installing MediaPipe and supporting dependencies.
3. Running object detection on a live camera feed.



##  Part 1: Preparing Raspberry Pi for Vision-Based Projects

###  Hardware Requirements
- Raspberry Pi 4 (4GB+ recommended)
- USB Camera
- 32GB or larger SD Card
- Power supply
- Internet access

###  Install Raspberry Pi OS
1. Recommended OS: Raspberry Pi OS (Debian Bullseye – Legacy version)
2. Use Raspberry Pi Imager to flash the OS onto the SD card
3. During OS setup:
   - Enable SSH
   - Configure locale, Wi-Fi, and hostname
   - Update system software:
     ```bash
     sudo apt-get update && sudo apt-get upgrade
     ```


##  Part 2: Installing Dependencies and MJPG-Streamer

###  Install Core Dependencies
```bash
sudo apt-get install nodejs npm git software-properties-common
sudo apt-get install build-essential imagemagick libv4l-dev cmake git -y
sudo apt-get install aptitude
sudo aptitude install libjpeg62-dev
```

###  Install MJPG-Streamer
```bash
git clone https://github.com/jacksonliam/mjpg-streamer.git
cd mjpg-streamer/mjpg-streamer-experimental
make
sudo make install
```

###  System Configuration

1. Enable serial interface:
   ```bash
   sudo raspi-config
   ```
   - Enable Serial Interface
   - Set auto-login as root
   - Reboot the system

2. Modify `/boot/config.txt`:
   ```bash
   sudo nano /boot/config.txt
   ```
   Add:
   ```
   dtoverlay=dwc2
   ```

3. Modify `/etc/modules`:
   ```bash
   sudo nano /etc/modules
   ```
   Add:
   ```
   dwc2
   g_serial
   ```



##  Part 3: Auto-Starting MJPG-Streamer on Boot

###  Create Startup Script

Create the script:
```bash
sudo vi /etc/init.d/livestream.sh
```

Paste the following:

```bash
#!/bin/sh
# /etc/init.d/livestream.sh

### BEGIN INIT INFO
# Provides:          livestream.sh
# Required-Start:    $network
# Required-Stop:     $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: mjpg_streamer for webcam
# Description:       Streams /dev/video0 to http://IP/?action=stream
### END INIT INFO

f_message(){
    echo "[+] $1"
}

case "$1" in
    start)
        f_message "Starting mjpg_streamer"
        /usr/local/bin/mjpg_streamer -b -i "input_uvc.so -f 30 -r 480x320" -o "output_http.so -w /usr/local/share/mjpg-streamer/www"
        sleep 2
        f_message "mjpg_streamer started"
        ;;
    stop)
        f_message "Stopping mjpg_streamer…"
        killall mjpg_streamer
        f_message "mjpg_streamer stopped"
        ;;
    restart)
        f_message "Restarting daemon: mjpg_streamer"
        killall mjpg_streamer
        /usr/local/bin/mjpg_streamer -b -i "input_uvc.so -f 30 -r 480x320" -o "output_http.so -w /usr/local/share/mjpg-streamer/www"
        sleep 2
        f_message "Restarted daemon: mjpg_streamer"
        ;;
    status)
        pid=`ps -A | grep mjpg_streamer | grep -v "grep" | grep -v mjpg_streamer. | awk '{print $1}' | head -n 1`
        if [ -n "$pid" ]; then
            f_message "mjpg_streamer is running with pid ${pid}"
            f_message "mjpg_streamer was started with the following command line"
            cat /proc/${pid}/cmdline ; echo ""
        else
            f_message "Could not find mjpg_streamer running"
        fi
        ;;
    *)
        f_message "Usage: $0 {start|stop|status|restart}"
        exit 1
        ;;
esac
exit 0
```

###  Enable and Start the Service
```bash
sudo chmod 755 /etc/init.d/livestream.sh
sudo update-rc.d livestream.sh defaults
sudo service livestream start
```



##  Install and Run MediaPipe

### 1. Create a Virtual Environment
```bash
python -m venv <environment_name>
source <environment_name>/bin/activate
```

### 2. Clone the MediaPipe Repository
```bash
git clone https://github.com/google-ai-edge/mediapipe-samples.git
cd mediapipe/examples/object_detection/raspberry_pi
```

### 3. Install Dependencies
```bash
sh setup.sh
```

### 4. Run the Example
```bash
python3 detect.py --model efficientdet_lite0.tflite
```

>  You can replace the model with any `.tflite` version of your choice for different performance profiles.



##  Conclusion

This pipeline allows Raspberry Pi users to run **real-time object detection** using **MediaPipe’s efficient models**, combined with lightweight **MJPG-streamer** video handling. It is an ideal setup for low-power edge applications like surveillance, smart kiosks, and IoT-based computer vision tasks.
