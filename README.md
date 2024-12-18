# Installation Manual for Raspberry Pi Setup

> [!NOTE]
> The installation and configuration steps for components, including LazyCast, RPiPlay, LCD setup, framebuffer copying, custom bootloader configurations, and web portal setup on a Raspberry Pi. Follow the steps sequentially.

# OS
![Raspberry Pi OS Selection](1. Initial Installation/Images/os need to download.png)
Select "**Raspberry Pi OS (Legacy, 32-bit)** A port of Debian Bullseye with security updates and desktop environment" when flashing the SD card.
On a fresh OS, follow Instructions below:


# 1. Initial Installation

## 1.1 Update and install dependencies
### Update and install dependencies:

```
sudo apt update
sudo apt install -y cmake git
```


## 1.2 LCD Setup

### Clone and install the Waveshare LCD setup:
```
sudo rm -rf LCD-show
git clone https://github.com/waveshareteam/LCD-show.git
chmod -R 755 LCD-show
cd LCD-show/
chmod -R 755 LCD35B-show-V2
sudo ./LCD35B-show-V2
```

### Set up framebuffer copying:
```
sudo apt-get install cmake -y
cd ~
git clone https://github.com/tasanakorn/rpi-fbcp
cd rpi-fbcp
mkdir build
cd build
cmake ..
make
sudo install fbcp /usr/local/bin/
```

### Automate framebuffer copying on boot:
```
nano ~/.bashrc
```

### Add this line at the end:
```
sudo fbcp &
```
### Reboot:
```
sudo reboot
```

# 2. For IOS Setup
## 2.1 RPiPlay Installation

### Update the system and install dependencies:
```
sudo apt update
sudo apt-get install -y libx264-dev libjpeg-dev libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev libgstreamer-plugins-bad1.0-dev \
    gstreamer1.0-plugins-ugly gstreamer1.0-tools gstreamer1.0-gl \
    gstreamer1.0-gtk3 gstreamer1.0-plugins-good git cmake \
    libavahi-compat-libdnssd-dev libplist-dev libssl-dev
```

### Clone and build RPiPlay:
```
git clone https://github.com/FD-/RPiPlay.git
cd RPiPlay
mkdir build
cd build
cmake ..
make -j2
sudo make install
```

### Install EGL support:
```
sudo apt-get install libegl1-mesa libraspberrypi-dev
```

```
cd RPiPlay/build
sudo ldconfig
export LD_LIBRARY_PATH=/opt/vc/lib:$LD_LIBRARY_PATH
```

## 2.2 Setup Bootimages

### replace raspberry IP to <RPI_IP>
### Transfer the Bootimages to the Raspberry Pi:

```
scp LoadingLogo.png Newnop@<RPI_IP>:/home/Newnop/
```
```
scp LoadingLogo_with_text.png Newnop@<RPI_IP>:/home/Newnop/
```


