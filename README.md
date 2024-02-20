# Installation Procedures for DeFli Projects 

## Flightview ADSB + ACARS

### Getting Started

To use this project, follow these steps:

### Prerequisites

- A Linux system either Linux computer, DragonOS or Pi running Ubuntu
- Docker Compose
- Docker installed
- Git (for cloning the repository)
- One or more RTL-SDR devices (for `readsb`, `acarshub`, and `dump978`)

### Set up Docker's apt repository

First, set up Docker's apt repository.. Open a terminal and run the following commands:

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install the Docker packages.
# To install the latest version, run:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Clone the Repository

You can clone this repository to any directory you prefer on your local system. If you are on WarDragon, you can also find it in `/usr/src`.

```bash
git clone https://github.com/alphafox02/flightview_gui.git
```

#### Update the Repository (Optional)

If you have already cloned the repository and want to update it, navigate to the project directory and run the following command:

```bash
git pull
```
### Launch GUI 
```bash
  sudo python3 flightview_gui.py
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
cmake -DENABLE_OSMOSDR=ON -DENABLE ..
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




### Contributing

We welcome contributions! If you have feedback, encounter issues, or want to contribute new features, please open an issue or submit a pull request.
