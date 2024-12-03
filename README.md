# Install Mainsail for Klipper on OpenMediaVault with Docker Compose

## What is Mainsail
<img src="https://dalaro.design/wp-content/uploads/2024/03/logo-mainsail.png" alt="Mainsail Logo" style="width:300px; height:auto;">

A modern and responsive user interface for Klipper. Control and monitor your printer from everywhere, from any device.

Easy to use. The focus lies on both anticipating what users may need to do and ensuring that the user interface contains elements that are easily accessible, understandable, and user-friendly to make those actions easier.

Always one step ahead. We work closely with developers of other projects so that functions can already be implemented early on.

100% open source. Anyone can join, anyone can contribute.

## What is OpenMediaVault
<img src="https://static-00.iconduck.com/assets.00/openmediavault-icon-96x96-ajxyqhhr.png" alt="OMV Logo" style="width:150px; height:auto;">

OpenMediaVault (OMV) is the next generation network attached storage (NAS) solution based on Debian Linux. It contains services like SSH, (S)FTP, SMB/CIFS, RSync and many more ready to use. Thanks to the modular design of the framework it can be enhanced via plugins. openmediavault is primarily designed to be used in small offices or home offices, but is not limited to those scenarios. It is a simple and easy to use out-of-the-box solution that will allow everyone to install and administrate a Network Attached Storage without deeper knowledge.

## Installation

### Previous steps

This guide assumes that you installed OMV and OMV-EXTRAS yet and you have some experience with it. Anyway, it's well explained **[here](https://wiki.omv-extras.org/)**

### Install Docker Plugin

Install the openmediavault-compose plugin and configure it (*Services/Compose/Settings menu*).

<img src="img/Screenshot from 2024-12-02 19-40-10.png" alt="Network definition" style="width:700px; heght:auto;">

### Define Network for Docker

I prefer to use a **MacVlan** network definition, so in that way your dockerized machine will have its own network address in the same local network you work at home everyday so, as I did it, I explain the method to you:

Open the Networks Menu (*Services/Compose/Networks*) and define a network for your dockerized hosts with the "+" icon.

**Name**: I use "MyMacVlan" for the example, but you could use your preferred name.

**Driver**: Use "macvlan" driver.

**Parent network**: Use the network device your OMV host uses to connect to your LAN (in my case, it's "br1").

**Subnet**: Your LAN network subnet with mask (for the example, a common one is "192.168.0.0/24")

**Gateway**: Your router/gateway address in your LAN (for this example, the usual should be "192.168.0.1")

**IP range**: Here you define the network group of hosts that could be defined into Docker/Compose using this network name. You could calculate this group with any [IP calculator](https://jodies.de/ipcalc?host=192.168.0.0&mask1=27&mask2=) in the web. For this example I use "192.168.0.0/27" which will let us use hosts from 192.168.0.1 to 192.168.0.30.

**Aux address**: Here you define a list of hosts that cannot be used for docker machines. For the example, I define my OMV host machine, which is 192.168.0.2 and another real machine, so I write "omv=192.168.0.2, other=192.168.0.13"

<img src="img/Screenshot from 2024-12-02 19-39-20.png" alt="Network definition" style="width:700px; heght:auto;">

### Create Composer File

Once the network is defined, you could go the Files menu (*Services/Compose/Files*) on OMV to define a new composer file with the "+" icon.

**Name** it as you prefer, and write your own **definition**, and in the file field, write this code or adapt it to your needs:

```yaml
networks:
  mainsail_net:
    name: MyMacVlan
    external: true
services:
  mainsail:
    container_name: mainsail
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    image: ghcr.io/mainsail-crew/mainsail:latest
    networks:
      mainsail_net:
        ipv4_address: 192.168.0.12 # <-- Update to your desired Mainsail IPv4
    volumes:
      - CHANGE_TO_COMPOSE_DATA_PATH/mainsail/mainsail.json:/usr/share/nginx/html/config.json
    restart: unless-stopped
```

<img src="img/Screenshot from 2024-12-02 20-15-05.png" alt="Network definition" style="width:700px; heght:auto;">

In this way, after booting your docker file, the system will download the Mainsail Docker Image, and generate a mainsail/mainsail.json file in your Data folder you defined previously in the *Services/Compose/Files* menu.

You could edit (or create previously) this file, it's well documented in the **[Mainsail Docs](https://docs.mainsail.xyz/overview/quicktips/config-json)** but, for my example, I use two fixed 3D printers (Moonraker services) defined in the same Klipper host, so it is, actually:

```json
{
    "defaultLocale": "en",
    "defaultMode": "dark",
    "defaultTheme": "mainsail",
    "instancesDB": "json",
    "instances": [
        { "hostname": "192.168.0.201", "port": 7125 },
        { "hostname": "192.168.0.201", "port": 7126 }
    ]
}
```
In this way, no matters which OS/browser you use, Mainsail will maintain always both printers defined.

There is an easy way to do it, but you will need to define your printers everytime, on every browser you connect to Mainsail (but you could edit those connections later), and they will be saved in cookies:

```json
{
  "instancesDB": "browser"
}
```

# Addendum for installing **Fluidd**

## What is Fluidd
<img src="https://docs.fluidd.xyz/assets/images/fluidd_icon.svg" alt="Fluidd Logo" style="width:300px; height:auto;">

Fluidd is a lightweight & responsive user interface for Klipper, the 3D printer firmware.

## Installation

You could use the exact installation steps. Obviously, change the name and definitions (from *Mainsail* to *Fluidd*), and in the Container File, change the line "image" from:

```yaml
image: ghcr.io/mainsail-crew/mainsail:latest
```

to:

```yaml
image: ghcr.io/fluidd-core/fluidd:latest
```
So it would finish like:
```yaml
networks:
  fluidd_net:
    name: MyMacVlan
    external: true
services:
  fluidd:
    container_name: fluidd
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    image: ghcr.io/fluidd-core/fluidd:latest
    networks:
      fluidd_net:
        ipv4_address: 192.168.0.13 # <-- Update to your desired Mainsail IPv4
    # volumes:
    #   - CHANGE_TO_COMPOSE_DATA_PATH/fluidd/fluidd.json:/usr/share/nginx/html/config.json
    restart: unless-stopped
```

As Fluidd does not uses config.json, you could just coment those lines.