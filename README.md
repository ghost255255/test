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

### replace raspberry IP to <RPI_IP> (etc: 192.168.4.1)
### Transfer the Bootimages to the Raspberry Pi:
```
scp LoadingLogo.png Newnop@<RPI_IP>:/home/Newnop/
```
```
scp LoadingLogo_with_text.png Newnop@<RPI_IP>:/home/Newnop/
```
### Install fbi:
```
sudo apt-get install fbi
```


## 2.3 remove boot text showing in boot screen

### Modify Plymouth theme:
```
sudo nano /usr/share/plymouth/themes/pix/pix.script
```

### Comment out the following lines:
```
# message_sprite = Sprite();  //comment
# message_sprite.SetPosition(screen_width * 0.1, screen_height * 0.9, 10000);  //comment
 
 fun message_callback (text) {
#        my_image = Image.Text(text, 1, 1, 1);  //comment
#        message_sprite.SetImage(my_image);     //comment
         sprite.SetImage (resized_image);
 }
```

### Update /boot/config.txt:
```
sudo nano /boot/config.txt
```

## 2.3 Configure Boot loader and Display Rotate:

### Display Rotate
#### change display_rotate=0 to display_rotate=3

### add these lines below
```
disable_splash=1
avoid_warnings=1
```

### Update /boot/cmdline.txt:
```
sudo nano /boot/cmdline.txt
```

### Replace console=tty0  with console=tty3:
console=serial0,115200 console=``tty3`` root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait fbcon=map:10 fbcon=font:ProFont6x11


## 2.4 Display IP and WiFi on Screen

### Install required tools:
```
sudo apt-get install fbi imagemagick wireless-tools fonts-dejavu libpng-dev imagemagick-6.q16
```
### Configure script to show IP and SSID:
```
nano ~/.bashrc
```
### Add the following:
```
sudo fbi -T 2 -noverbose -a /home/Newnop/LoadingLogo.png

sleep 1

SSID=$(iwgetid -r)
IP_ADDRESS=$(hostname -I | awk '{print $1}')
TEXT="SSID: $SSID\nIP Address: $IP_ADDRESS\nType the ip address\nin your browser and\nlog into the portal\n\n\nDevice SSID: newnop\nDevice PASS: 12345678\n\n\nNewnopcast pass: 31415926"

sleep 3

convert -size 320x480 xc:none -gravity center -pointsize 16 -fill white -annotate 0 "$TEXT" -rotate 270 /home/Newnop/rotated_text.png
convert /home/Newnop/LoadingLogo_with_text.png /home/Newnop/rotated_text.png -gravity south -composite /home/Newnop/LoadingLogo_with_rotated_text.png
sudo fbi -T 2 -noverbose -a /home/Newnop/LoadingLogo_with_rotated_text.png

sleep 10

cd RPiPlay/build

sudo ldconfig
export LD_LIBRARY_PATH=/opt/vc/lib:$LD_LIBRARY_PATH
find /opt -name "libbrcmEGL.so"
./rpiplay -a hdmi -l -n NewnopCast
```

### Reboot:
```
sudo reboot
```



# 3. For Android Setup

## 3.1 Lazycast Installation

### Update the system and install dependencies:
```
sudo apt update
sudo apt install -y cmake git
```

### Clone and build userland:
```
git clone https://github.com/raspberrypi/userland
cd userland
./buildme
```

### Update the /boot/config.txt` file: 
Replace ``vc4-kms-v3d`` with ``vc4-fkms-v3d``
```
sudo sed -i 's/vc4-kms-v3d/vc4-fkms-v3d/g' /boot/config.txt
```

### Alternatively, edit the file manually:
```
sudo nano /boot/config.txt
```

### Reboot the Raspberry Pi:
```
sudo reboot
```

### Install additional libraries:
```
sudo apt install libx11-dev libasound2-dev libavformat-dev libavcodec-dev python3-evdev
```

### Build LazyCast:
```
cd /opt/vc/src/hello_pi/libs/ilclient/
sudo make
cd /opt/vc/src/hello_pi/hello_video
sudo make
```
```
cd ~/
git clone https://github.com/homeworkc/lazycast
cd lazycast/
make
```


## 3.2 Setup Boot images

### replace raspberry IP to <RPI_IP> (etc: 192.168.4.1)
### Transfer the Boot images to the Raspberry Pi:
```
scp LoadingLogo.png Newnop@<RPI_IP>:/home/Newnop/
```
```
scp LoadingLogo_with_text.png Newnop@<RPI_IP>:/home/Newnop/
```
### Install fbi:
```
sudo apt-get install fbi
```


## 3.3 remove boot text showing in boot screen

### Modify Plymouth theme:
```
sudo nano /usr/share/plymouth/themes/pix/pix.script
```

### Comment out the following lines:
```
# message_sprite = Sprite();  //comment
# message_sprite.SetPosition(screen_width * 0.1, screen_height * 0.9, 10000);  //comment
 
 fun message_callback (text) {
#        my_image = Image.Text(text, 1, 1, 1);  //comment
#        message_sprite.SetImage(my_image);     //comment
         sprite.SetImage (resized_image);
 }
```

### Update /boot/config.txt:
```
sudo nano /boot/config.txt
```

## 3.4 Configure Boot loader and Display Rotate:

### Display Rotate
#### change display_rotate=0 (if its not 0)

### add these lines below
```
disable_splash=1
avoid_warnings=1
```

### Update /boot/cmdline.txt:
```
sudo nano /boot/cmdline.txt
```

### Replace console=tty0  with console=tty3:
console=serial0,115200 console=``tty3`` root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait fbcon=map:10 fbcon=font:ProFont6x11


## 3.5 Display IP and WiFi on Screen

### Install required tools:
```
sudo apt-get install fbi imagemagick wireless-tools fonts-dejavu libpng-dev imagemagick-6.q16
```
### Configure script to show IP and SSID:
```
nano ~/.bashrc
```
### Add the following:
```
sudo fbi -T 2 -noverbose -a /home/Newnop/LoadingLogo.png

sleep 1
SSID=$(iwgetid -r)
IP_ADDRESS=$(hostname -I | awk '{print $1}')
TEXT="SSID: $SSID\nIP Address: $IP_ADDRESS\nType the ip address\nin your browser and\nlog into the portal\n\n\nDevice SSID: newnop\nDevice PASS: 12345678\n\n\nNewnopcast pass: 31415926"

sleep 3
convert -size 320x480 xc:none -gravity center -pointsize 16 -fill white -annotate 0 "$TEXT" -rotate 270 /home/Newnop/rotated_text.png
convert /home/Newnop/LoadingLogo_with_text.png /home/Newnop/rotated_text.png -gravity south -composite /home/Newnop/LoadingLogo_with_rotated_text.png
sudo fbi -T 2 -noverbose -a /home/Newnop/LoadingLogo_with_rotated_text.png

cd lazycast
./all.sh
```

### Reboot:
```
sudo reboot
```


# 4. For Both OS Setup
## 4.1 RPiPlay Installation

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

## 4.2 Lazycast Installation

### Update the system and install dependencies:
```
sudo apt update
sudo apt install -y cmake git
```

### Clone and build userland:
```
git clone https://github.com/raspberrypi/userland
cd userland
./buildme
```

### Update the /boot/config.txt` file: 
Replace ``vc4-kms-v3d`` with ``vc4-fkms-v3d``
```
sudo sed -i 's/vc4-kms-v3d/vc4-fkms-v3d/g' /boot/config.txt
```

### Alternatively, edit the file manually:
```
sudo nano /boot/config.txt
```

### Reboot the Raspberry Pi:
```
sudo reboot
```

### Install additional libraries:
```
sudo apt install libx11-dev libasound2-dev libavformat-dev libavcodec-dev python3-evdev
```

### Build LazyCast:
```
cd /opt/vc/src/hello_pi/libs/ilclient/
sudo make
cd /opt/vc/src/hello_pi/hello_video
sudo make
```
```
cd ~/
git clone https://github.com/homeworkc/lazycast
cd lazycast/
make
```

## 4.3 Setup Bootimages and Web Portal

### replace raspberry IP to <RPI_IP> (etc: 192.168.4.1)
### Transfer the Bootimages to the Raspberry Pi:
```
scp LoadingLogo.png Newnop@<RPI_IP>:/home/Newnop/
```
```
scp LoadingLogo_with_text.png Newnop@<RPI_IP>:/home/Newnop/
```
### Install fbi:
```
sudo apt-get install fbi
```

### Transfer the web portal to the Raspberry Pi:
```
scp -r web_portal Newnop@<RPI_IP>:/home/Newnop/
```

### Install Flask:
```
sudo apt update
sudo apt install python3-flask
```

## 4.4 remove boot text showing in boot screen

### Modify Plymouth theme:
```
sudo nano /usr/share/plymouth/themes/pix/pix.script
```

### Comment out the following lines:
```
# message_sprite = Sprite();  //comment
# message_sprite.SetPosition(screen_width * 0.1, screen_height * 0.9, 10000);  //comment
 
 fun message_callback (text) {
#        my_image = Image.Text(text, 1, 1, 1);  //comment
#        message_sprite.SetImage(my_image);     //comment
         sprite.SetImage (resized_image);
 }
```

### Update /boot/config.txt:
```
sudo nano /boot/config.txt
```

## 4.5 Configure Boot loader and Display Rotate:

### Display Rotate
#### change display_rotate=0 (if its not 0)

### add these lines below
```
disable_splash=1
avoid_warnings=1
```

### Update /boot/cmdline.txt:
```
sudo nano /boot/cmdline.txt
```

### Replace console=tty0  with console=tty3:
console=serial0,115200 console=``tty3`` root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait fbcon=map:10 fbcon=font:ProFont6x11


## 4.6 Display IP and WiFi on Screen

### Install required tools:
```
sudo apt-get install fbi imagemagick wireless-tools fonts-dejavu libpng-dev imagemagick-6.q16
```
### Configure script to show IP and SSID:
```
nano ~/.bashrc
```
### Add the following:
```
sudo fbi -T 2 -noverbose -a /home/Newnop/LoadingLogo.png

sleep 1

SSID=$(iwgetid -r)
IP_ADDRESS=$(hostname -I | awk '{print $1}')
TEXT="SSID: $SSID\nIP Address: $IP_ADDRESS\nType the ip address\nin your browser and\nlog into the portal\n\n\nDevice SSID: NewnopSM\nDevice PASS: 12345678\n\n\nNewnopcast pass: 31415926"

sleep 3

convert -size 320x480 xc:none -gravity center -pointsize 16 -fill white -annotate 0 "$TEXT" -rotate 270 /home/Newnop/rotated_text.png
convert /home/Newnop/LoadingLogo_with_text.png /home/Newnop/rotated_text.png -gravity south -composite /home/Newnop/LoadingLogo_with_rotated_text.png
sudo fbi -T 2 -noverbose -a /home/Newnop/LoadingLogo_with_rotated_text.png

sleep 1

sudo python3 web_portal/wifi_config_portal.py
```

### Reboot:
```
sudo reboot
```
