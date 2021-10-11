
En cours de DEV

Branche Docker-voltronic-hommeassistant transformer pour communiquer sous jeedom via JMQTT


## Prerequisites

- Docker
- Docker-compose
- [Voltronic/Axpert/MPPSolar](https://www.ebay.com.au/sch/i.html?_from=R40&_trksid=p2334524.m570.l1313.TR11.TRC1.A0.H0.Xaxpert+inverter.TRS0&_nkw=axpert+inverter&_sacat=0&LH_TitleDesc=0&LH_PrefLoc=2&_osacat=0&_odkw=solar+inverter&LH_TitleDesc=0) based inverter that you want to monitor
- Home Assistant [running with a MQTT Server](https://www.home-assistant.io/components/mqtt/)


## Configuration & Standing Up

It's pretty straightforward, just clone down the sources and set the configuration files in the `config/` directory:

```bash
# Clone down sources on the host you want to monitor...
git clone https://github.com/tititg34/docker-voltronic-Jeedom.git /opt/jeedom-inverter-mqtt-agent
cd /opt/jeedom-inverter-mqtt-agent

# Configure the 'device=' directive (in inverter.conf) to suit for RS232 or USB.. 
vi config/inverter.conf

# Configure your MQTT server's IP/Host Name, Port, Credentials, HA topic, and name of the Inverter that you want displayed in Home Assistant...
# If your MQTT server does not need a username/password just leave these values empty.

vi config/mqtt.json
```

Then, plug in your Serial or USB cable to the Inverter & stand up the container:

```bash
docker-compose up -d

```

_**Note:**_

  - builds on docker hub are currently for `linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64,linux/386` -- If you have issues standing up the image on your Linux distribution (i.e. An old Pi/ARM device) you may need to manually build the image to support your local device architecture - This can be done by uncommenting the build flag in your docker-compose.yml file.
  
  - The default `docker-compose.yml` file includes Watchtower, which can be  configured to auto-update this image when we push new changes to github - Please **uncomment if you wish to auto-update to the latest builds of this project**.

## Integrating into Home Assistant.

Providing you have setup [MQTT](https://www.home-assistant.io/components/mqtt/) with Home Assistant, the device will automatically register in your Home Assistant when the container starts for the first time -- You do not need to manually define any sensors.

From here you can setup [Graphs](https://www.home-assistant.io/lovelace/history-graph/) to display sensor data, and optionally change state of the inverter by "[publishing](https://www.home-assistant.io/docs/mqtt/service/)" a string to the inverter's primary topic like so:

![Example, Changing the Charge Priority](images/mqtt-publish-packet.png "Example, Changing the Charge Priority")
_Example: Changing the Charge Priority of the Inverter_

**COMMON COMMANDS THAT CAN BE SENT TO THE INVERTER**

_(see [protocol manual](http://forums.aeva.asn.au/uploads/293/HS_MS_MSX_RS232_Protocol_20140822_after_current_upgrade.pdf) for complete list of supported commands)_



```
DESCRIPTION:                PAYLOAD:  OPTIONS:
----------------------------------------------------------------
Set output source priority  POP00     (Utility first)
                            POP01     (Solar first)
                            POP02     (SBU)

Set charger priority        PCP00     (Utility first)
                            PCP01     (Solar first)
                            PCP02     (Solar and utility)
                            PCP03     (Solar only)

Set the Charge/Discharge Levels & Cutoff
                            PBDV26.9  (Don't discharge the battery unless it is at 26.9v or more)
                            PBCV24.8  (Switch back to 'grid' when battery below 24.8v)
                            PBFT27.1  (Set the 'float voltage' to 27.1v)
                            PCVV28.1  (Set the 'charge voltage' to 28.1v)

Set other commands          PEa / PDa (Enable/disable buzzer)
                            PEb / PDb (Enable/disable overload bypass)
                            PEj / PDj (Enable/disable power saving)
                            PEu / PDu (Enable/disable overload restart);
                            PEx / PDx (Enable/disable backlight)
```

*NOTE:* When setting/configuring your charge, discharge, float & cutoff voltages for the first time, it's worth  understanding how to optimize charging conditions to extend service life of your battery: https://batteryuniversity.com/learn/article/charging_the_lead_acid_battery


### Using `inverter_poller` binary directly

This project uses heavily modified sources, from [manio's](https://github.com/manio/skymax-demo) original demo, and be compiled to run standalone on Linux, Mac, and Windows (via Cygwin).

Just head to the `sources/inverter-cli` directory and build it directly using: `cmake . && make`.

Basic arguments supported are:

```
USAGE:  ./inverter_poller <args> [-r <command>], [-h | --help], [-1 | --run-once]

SUPPORTED ARGUMENTS:
          -r <raw-command>      TX 'raw' command to the inverter
          -h | --help           This Help Message
          -1 | --run-once       Runs one iteration on the inverter, and then exits
          -d                    Additional debugging

```

### Bonus: Lovelace Dashboard Files

_**Please refer to the screenshot above for an example of the dashboard.**_

I've included some Lovelace dashboard files in the `homeassistant/` directory, however you will need to need to adapt to your own Home Assistant configuration and/or name of the inverter if you have changed it in the `mqtt.json` config file.

Note that in addition to merging the sample Yaml files with your Home Assistant, you will need the following custom Lovelace cards installed if you wish to use my templates:

 - [vertical-stack-in-card](https://github.com/custom-cards/vertical-stack-in-card)
 - [circle-sensor-card](https://github.com/custom-cards/circle-sensor-card)
