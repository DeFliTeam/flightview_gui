# Installation Procedures for DeFli Projects 



## ADSB & ACARS

### Prerequisites

- A Linux system either Linux computer, DragonOS or Pi running Ubuntu
- Docker Compose
- Docker installed
- Git (for cloning the repository)
- One or more RTL-SDR devices (for `readsb`and`acarshub`) note these must be dedicated SDR's for ADSB and Airband VHF respectively.

### Set up Docker

First, set up Docker's apt repository.. Open a terminal and run the following commands:

```bash
# Install prerequisites
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg
```

### Add Key
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
```

### Add Repo (Linux)
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```
### Add Repo (RPi) 
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```
### Install Docker (Linux)
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin
```
### Install Docker (RPI)
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin
```


### Clone the Repository

You can clone this repository to any directory you prefer on your local system. If you are on WarDragon, you can also find it in `/usr/src`.

```bash
git clone https://github.com/alphafox02/flightview_gui.git
```

#### Update the Repository (Optional)

If you have already cloned the repository and want to update it, navigate to the project directory and run the following command:

```bash
cd flightview_gui/
git pull
```
### Change Access Control (On Pi Only)
```bash
export DISPLAY=:0.0
xhost +
```

### Install YAML 
```bash
sudo python -m pip install pyyaml

```

### Launch GUI (Linux)
```bash
cd flightview_gui/
  sudo python3 flightview_gui.py
```
### Launch GUI (RPi) 
```bash
sudo python flightview_gui.py
```

### Using the GUI

The FlightView GUI offers an intuitive interface to configure and manage Aircraft services. It includes three tabs for the following services (for now):

- **tar1090 Configuration:**
  - Configure `docker-compose-tar1090.yml` settings such as latitude, longitude, and timezone.
  - Start and stop the `tar1090` service.
  - Real-time service status display.

- **readsb Configuration:**
  - Configure `docker-compose-readsb.yml` settings, including basic options.
  - Manually configure advanced settings such as RTL-SDR device configuration in the YAML file.
  - Start and stop the `readsb` service.
  - Real-time service status display.

- **acarshub Configuration:**
  - Configure `docker-compose-acars-vhf.yml` settings, including basic options.
  - Manually configure advanced settings such as RTL-SDR device configuration in the YAML file.
  - Start and stop the `acarshub` service.
  - Real-time service status display.

Use these tabs to set basic parameters like latitude (LAT), longitude (LONG), timezone (TZ), and feed ID (FEED_ID). For advanced settings and additional configurations, you'll need to manually edit the corresponding Docker Compose YAML files. The GUI simplifies basic configurations but may not cover all advanced options.

### Access the Web Interfaces

After configuring the services, you can access their web interfaces as follows:

- **ACARS Hub:** [http://localhost](http://localhost)
- **READSB:** [http://localhost:8080](http://localhost:8080)
- **tar1090:** [http://localhost:8078](http://localhost:8078)
- **worldmap** [http://you-ip-address:1880/worldmap](http://localhost:1880/worldmap) (ONLY AVAILABLE AFTER DATA CONNECTOR)

Make sure that you have the required RTL-SDR devices connected and properly recognized by your system for services that use them.

### Links to Docker Repositories

The Docker Compose YAML files in this project pull in the following Docker images:

- [ACARS Hub](https://github.com/sdr-enthusiasts/docker-acarshub)
- [Readsb-protobuf](https://github.com/sdr-enthusiasts/docker-readsb-protobuf)
- [Dump978](https://github.com/sdr-enthusiasts/docker-dump978)
- [DumpVDL2](https://github.com/sdr-enthusiasts/docker-dumpvdl2)
- [Tar1090](https://github.com/sdr-enthusiasts/docker-tar1090)
- [DefliInfluxDB](https://github.com/dealcracker/DefliInfluxDB)

Explore these repositories to learn more about each component and contribute to their development.   


### Troubleshooting

If you encounter issues accessing the web interfaces, check your firewall settings to ensure they allow connections to the specified ports.

Enjoy using the FlightView GUI to effortlessly manage and monitor your aircraft services. If you have any questions or encounter issues, please feel free to open an issue in this repository.


## Data Connector 

If you are running a self-build device your Bucket ID and API Key will be provided in your defli-wallet. If you have a DeFli Device your Bucket ID and API Key can be found on the underneath of the device. 

Run this script  

```bash
sudo bash -c "$(wget -O - https://raw.githubusercontent.com/dealcracker/DefliInfluxDB/master/installInflux.sh)"
```


## Passive Radar
 
This function is only currently available on DeFli Devices however may extend to self-builds. You are however welcome to test on self-builds. 

**Step 1- Install ADSB2**

```bash
sudo git clone http://github.com/30hours/blah2 /opt/adsb2dd
cd /opt/adsb2dd
sudo docker compose up -d
```
API is available at http://localhost:49155

**Step 2- Install Radar** 

```bash
vim docker-compose.yml
--- build: .
+++ image: ghcr.io/30hours/blah2:latest
sudo docker compose up -d
```
Adjusting the docker file: 

```bash
sudo nano config/config.yml
```
In the section titled "truth" change the following: 
TAR1090- Change to the location of your aircraft.json file 
ADSB2DD- Change to 'http://localhost:49155' 

In the first section change your center frequency to the center frequency of your VHF/UHF/FM/DVB-T Antenna. Note this antenna must be oriented at a known source if directional only (such as FM). If using an omni directional then no orientation required.

The radar output is available at http://localhost:49152. 

### Troubleshooting 

- If the RTL-SDR does not capture data, restart the API service (on the host) using sudo systemctl restart sdrplay.api.

## GNSS  

**Step 1 Install Dependencies** 

```bash
$ sudo apt-get install build-essential cmake git pkg-config libboost-dev libboost-date-time-dev \
       libboost-system-dev libboost-filesystem-dev libboost-thread-dev libboost-chrono-dev \
       libboost-serialization-dev liblog4cpp5-dev \
       libblas-dev liblapack-dev libarmadillo-dev libgflags-dev libgoogle-glog-dev \
       libgnutls-openssl-dev libpcap-dev libmatio-dev libpugixml-dev libgtest-dev \
       libprotobuf-dev protobuf-compiler python3-mako
```

**Step 2 Clone in to repository** 
```bash
$ git clone https://github.com/gnss-sdr/gnss-sdr
```

**Step 3 Enter Directory, make and add OSMOSDR** 
```bash
cd gnss-sdr/
cd build/
cmake ..
cd build/
cmake -DENABLE_OSMOSDR=ON ..
cd build/
make -j4
```

**Step 4 Create Config File** 
Create a new folder on your desktop called **GPS** and create a file within it, name it **gnss-rtl.conf** 
Paste the following configuration 

```bash
[GNSS-SDR]
;######### GLOBAL OPTIONS ##################
GNSS-SDR.internal_fs_sps=2000000

;######### SIGNAL_SOURCE CONFIG ############
SignalSource.implementation=Osmosdr_Signal_Source
SignalSource.item_type=gr_complex
SignalSource.sampling_frequency=2000000
SignalSource.freq=1575420000
SignalSource.gain=50
SignalSource.rf_gain=40
SignalSource.if_gain=30
SignalSource.AGC_enabled=true
SignalSource.samples=0
SignalSource.repeat=false
SignalSource.dump=false
SignalSource.dump_filename=../data/signal_source.dat
SignalSource.enable_throttle_control=false
SignalSource.osmosdr_args=rtl1,bias=1     **PLEASE NOTE YOU WILL NEED TO ASSIGN THE CORRECT SDR NUMBER THE DEFAULT IS 1 BUT YOUR SYSTEM MAY ASSIGN A DIFFERENT NUMBER, LEAVE BIAS AS 1** 

;######### SIGNAL_CONDITIONER CONFIG ############
SignalConditioner.implementation=Signal_Conditioner

DataTypeAdapter.implementation=Pass_Through

;######### INPUT_FILTER CONFIG ############
InputFilter.implementation=Freq_Xlating_Fir_Filter
InputFilter.input_item_type=gr_complex
InputFilter.output_item_type=gr_complex
InputFilter.taps_item_type=float
InputFilter.number_of_taps=5
InputFilter.number_of_bands=2
InputFilter.band1_begin=0.0
InputFilter.band1_end=0.85
InputFilter.band2_begin=0.90
InputFilter.band2_end=1.0
InputFilter.ampl1_begin=1.0
InputFilter.ampl1_end=1.0
InputFilter.ampl2_begin=0.0
InputFilter.ampl2_end=0.0
InputFilter.band1_error=1.0
InputFilter.band2_error=1.0
InputFilter.filter_type=bandpass
InputFilter.grid_density=16
InputFilter.sampling_frequency=2000000
InputFilter.IF=14821

######### RESAMPLER CONFIG ############
Resampler.implementation=Pass_Through
Resampler.dump=false
Resampler.item_type=gr_complex

GNSS-SDR.internal_fs_sps=corrected_value
InputFilter.sampling_frequency=corrected_value
```

**Step 5 Install** 

```bash
cd gnss-sdr/build
sudo make install
```

**INSERT SDR HERE** 

**Step 6 Run**

```bash
cd /home/USER/Desktop/GPS/
gnss-sdr -config_file gnss-rtl.conf
```

You should now begin to see satellite data messages, a position lock is indicated in green text 

**Run in background** 

To run GNSS continuously as a background executable follow these instructions 

Once you have secured a GPS lock indicated by a green message on the terminal press **ctrl + z** 

then 

```bash
bg
```

Alternatively you can run this command in step 6 


```bash
gnss-sdr -config_file gnss-rtl.conf &
```

**Troubleshooting** 

You must have Bias T turned on 
You must have assigned the correct number to the RTL-SDR In step 4 Config File 

## WiFi Scanning Drones 

To perform this function you will need a wireless adapter, we recommend the Panda PAU09 USB Adapter 

**Step 1 Install Python** 

```bash
sudo apt update
sudo apt upgrade
sudo apt install python2.7 python-pip
```

**Step 2 Clone Repository and install modules** 

```bash
git clone https://github.com/DeFliTeam/drone-detection.git
cd drone-detection
```
```bash
sudo python3 install -r requirements.txt
```

**Step 3 Identify the WiFi Interface** 

Here we need to indentify the wireless adapter interface. Run command 

```bash
iw dev
```
This will give a list of interfaces like below 

```bash
$ iw dev
phy#0
	Interface wlan0
		ifindex 219
		wdev 0x500000001
		addr a0:f3:xx:xx:xx:xx
		txpower 20.00 dBm
phy#1
	Interface wlan1
		ifindex 180
		wdev 0x1c
		addr b8:08:xx:xx:xx:xx
		type managed
```

You will want use the Interface wlan0 portion, not the phy#0. Note that in some distributions, the interface may be names differently (such as wlxa0f3c11e13c2, for example). This must be your interface card and not your home router. 

**Step 4 Run Programme** 

There are a number of options for running the drone detection 

### Static 
With this command you can scan for drones on a specific WiFi channel. 

```bash
sudo ./dronitor static wlan1 -c 1
```
First it changes the wireless network card to monitor mode and scans all the WiFi packets on channel 1. If it finds a MAC address with an OUI that is on the list an alert will appear. In this command wlan1 is the name of the wireless interface and -c indicates the channel to listen on. You can also use --channel for more verbosity. To exit press ctrl-c.

### Hop 
With this command you can scan for drones on all the WiFi channels 

```bash
sudo ./dronitor hop wlan1 -t 0.5 &
```
First it changes the wireless network card to monitor mode and then scans all the WiFi packets. It changes channel every 0.5 seconds and loops constantly through all the WiFi channels. If it finds a MAC address with an OUI that is on the list an alert will appear. In this command wlan1 is the name of the wireless interface that wil be used and -t or --time is the time to wait before changing to the next channel. To exit press ctrl-c for a few seconds.

### Managed 
With this command you can change your wireless interface back to managed mode. 

```bash
sudo ./dronitor managed wlan1
```
Where wlan1 is the name of the wireless interface that will be changed to managed mode. Use this command when you've finished using the service.  

## Camera Based Drone Detection 

Note this will require this device added to your host board (including a DeFli Device). It uses any camera set as "default" within the board 

### Clone Repository 

```bash
sudo git clone https://github.com/DeFliTeam/Camera-Aerial-Drone-Detection-System.git
```
### Install Python Libraries 

```bash
pip install opencv-python torch numpy pillow
```
### Download YoloV5 Model 

### Download pre-trained weights 
https://github.com/ultralytics/yolov5/releases/download/v7.0/yolov5n-seg.pt 

Add the weights to the project directory by copying and pasting in to a folder named "weights.pt" 

```bash
sudo nano weights.pt
```

### Run 

```bash
python Advanced_Drone_Detection.py
```

The script will open a live video feed from the default camera.

    To create a rectangle, click and drag the mouse on the video feed to define the four corners of the rectangle.
    The rectangle can be adjusted by dragging the corners.
    A warning message will be displayed whenever a drone is detected inside or near the rectangle.

Press 'q' to quit the program.


### Train 

This is optional but if you would like to contribute to our training model 

Use your laptop and the screenshot function to take images from the live video feed 

Annotate these images with the following tags 

"Drone" "Bird" "Aeroplane" "Helicopter" "Balloon" "Litter" 

Save each image to your local computer 

### Then use base64 encoding  

```bash
base64 filename > encoded_filename.txt

e.g base64 img1.jpg > img1.txt
```
### Send the file to us using InfluxDB  

```bash
# influxdata-archive_compat.key GPG fingerprint:
#     9D53 9D90 D332 8DC7 D6C8 D3B9 D8FF 8E1F 7DF8 B07E
wget -q https://repos.influxdata.com/influxdata-archive_compat.key
echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list

sudo apt-get update && sudo apt-get install influxdb2-cli
```
```bash
influx
```

```bash
influx write \
  -b bucketName \
  -o orgName \
  -a API Token
  -p s \
  --format=lp
  -f /path/to/line-protocol.txt
```
Example 
```bash
influx write \
  -b Test1 \       (obtain this from @OreoMCL on our discord server)
  -o DEFliData \
  -a API Token     (obtain this from @OreoMCL on our discord server)
  -p s \
  --format=lp
  -f /path/to/img1.txt
```



### Screen Drivers 
```bash
sudo git clone https://github.com/osoyoo/HDMI-show.git
sudo chmod -R 777 HDMI-show
cd HDMI-show/
sudo ./hdmi35-480x320-show
reboot
```
## Real VNC 
Install Real VNC to remotely manage your mini screen on the DeFli Device 

### Raspberry Pi 
Real VNC is pre-installed on Raspberry Pi 

1) Open Menu and choose security> Encryption = Prefer Off   Authentication = VNC_Password
2) Users & Permissions > Select Standard User and create password.
3) Restart the Pi

### Linux 
To download to Linux follow these additional steps and the use the steps outlined in "Raspberry Pi" 

1) Navigate to https://www.realvnc.com/en/connect/download/combined/ and choose the linux version
2) Extract the file and open
3) Run the installer

### On Host Device
This is the device where you want to control your DeFli mini-screen from, assuming this is the Mini-PC within your DeFli Device 

1) Navigate here https://www.realvnc.com/en/connect/download/viewer/linux/ and download the x64 Linux version
then

 ```bash
cd /Downloads
sudo apt install ./VNC-Viewer-7.10.0-Linux-x64.deb
```
2) Search for "VNC" in your application menu.

### Starting a Session 

1) Open VNC Viewer app
2) Input Pi or mini-pc IP address (note you can find this on the Real VNC app on your target device (RPI or Mini-PC) and click connect
3) Enter password created in "step 2" of the instructions and click connect

You will now have a full screen on your local device that replicates the screen shown on your mini-screen in the DeFli Device. Use the browser function to input the URL of the map you wish to see.



### Contributing

We welcome contributions! If you have feedback, encounter issues, or want to contribute new features, please open an issue or submit a pull request. 

### Thanks to 

@alphafox02 
@dealcracker 
@wiedhopf 
@30hours
