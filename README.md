# Home Assistant VPD Control System

A plug-and-play Vapor Pressure Deficit (VPD) control system for Home Assistant, designed for hydroponic grow rooms, greenhouses, and controlled environment agriculture.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Quick Start](#quick-start)
- [Device Setup](#device-setup)
- [Configuration](#configuration)
- [Dashboard](#dashboard)
- [Troubleshooting](#troubleshooting)
- [Advanced Usage](#advanced-usage)

## 🌱 Overview

VPD (Vapor Pressure Deficit) is crucial for optimal plant growth, representing the difference between water vapor pressure in the air and at the leaf surface. This system automatically calculates target humidity levels based on temperature and your desired VPD, then controls humidification/dehumidification devices accordingly.

### What this system does:
- **Calculates real-time VPD** using temperature sensors
- **Determines optimal humidity levels** for your target VPD
- **Automatically controls** humidifiers and dehumidifiers
- **Provides detailed logging** and monitoring
- **Supports multiple growth stages** with different VPD targets

## ✨ Features

- 🌡️ **Real-time VPD calculation** using Magnus-Tetens formula approximation
- 🎯 **Multi-stage VPD targets** (seedling to flowering)
- 🔧 **Multi-protocol device support** (Zigbee, Z-Wave, WiFi, SmartThings)
- 📊 **Pre-configured dashboard** with charts and notifications
- 🔄 **Dead-band control** to prevent device oscillation
- 📱 **Mobile-friendly interface**
- 🔍 **Detailed logging** for troubleshooting
- 🐳 **Docker-ready** for easy deployment

## 🚀 Quick Start

### Prerequisites
- Docker installed on your machine
- Temperature and humidity sensors
- Controllable humidifier/dehumidifier devices

### Installation

#### For Linux/Mac:
```bash
git clone <your-repo-url>
cd VPDRegulator

# Create Home Assistant directory structure
mkdir -p homeassistant/config

# Copy the VPD configuration
cp VPD_Regulator.yaml homeassistant/config/configuration.yaml

# Start Home Assistant with Docker
docker run -d \
  --name homeassistant-vpd \
  --restart=unless-stopped \
  -p 8123:8123 \
  -v $(pwd)/homeassistant/config:/config \
  -v /etc/localtime:/etc/localtime:ro \
  -e TZ=Europe/Rome \
  ghcr.io/home-assistant/home-assistant:stable
```

#### For Windows (Command Prompt):
```cmd
git clone <your-repo-url>
cd VPDRegulator

:: Create Home Assistant directory structure
mkdir homeassistant\config

:: Copy the VPD configuration
copy VPD_Regulator.yaml homeassistant\config\configuration.yaml

:: Start Home Assistant with Docker
docker run -d ^
  --name homeassistant-vpd ^
  --restart=unless-stopped ^
  -p 8123:8123 ^
  -v %cd%\homeassistant\config:/config ^
  -e TZ=Europe/Rome ^
  ghcr.io/home-assistant/home-assistant:stable
```

#### For Windows (PowerShell):
```powershell
git clone <your-repo-url>
cd VPDRegulator

# Create Home Assistant directory structure
New-Item -ItemType Directory -Force -Path homeassistant\config

# Copy the VPD configuration
Copy-Item VPD_Regulator.yaml homeassistant\config\configuration.yaml

# Start Home Assistant with Docker
docker run -d `
  --name homeassistant-vpd `
  --restart=unless-stopped `
  -p 8123:8123 `
  -v ${PWD}\homeassistant\config:/config `
  -e TZ=Europe/Rome `
  ghcr.io/home-assistant/home-assistant:stable
```

2. **Access Home Assistant:**
   - Open http://localhost:8123
   - Complete the initial setup wizard
   - Your VPD system is ready to use!

3. **Connect your devices** (see Device Setup section below)

4. **Configure your sensors** (see Configuration section below)

### Useful Docker Commands

These commands work the same on all platforms:

```bash
# View logs
docker logs -f homeassistant-vpd

# Stop the container
docker stop homeassistant-vpd

# Start the container
docker start homeassistant-vpd

# Restart the container
docker restart homeassistant-vpd

# Remove container (keeps your config files)
docker rm homeassistant-vpd

# Check container status
docker ps -a | grep homeassistant-vpd
```

## 🔌 Device Setup

### Required Devices

You need these devices for a complete VPD control system:

#### Temperature & Humidity Sensors
Any Home Assistant compatible sensor will work:

**Zigbee Sensors (Recommended)**
- Setup via Settings > Devices & Services > Add Integration > Zigbee Home Automation (ZHA)
- Follow your sensor's pairing instructions
- Sensors will auto-discover in Home Assistant

**WiFi Sensors**
- Configure according to manufacturer instructions
- Add via appropriate integration (Shelly, ESPHome, etc.)
- Sensors will appear in Home Assistant automatically

**SmartThings Sensors**
- Setup via Settings > Devices & Services > Add Integration > SmartThings
- Follow OAuth authentication process
- Import existing sensors from your SmartThings hub

#### Humidity Control Devices

**Option 1: Smart Plugs + Regular Devices (Easiest)**
- Use any Zigbee, Z-Wave, or WiFi smart plug
- Connect regular humidifier/dehumidifier to the smart plug
- Control through Home Assistant as switches

**Option 2: Native Smart Devices**
- Use WiFi-enabled humidifiers/dehumidifiers
- Integrate through manufacturer-specific integrations
- Provides additional features like water level monitoring

### Device Naming Best Practices

Use this naming convention for easy identification:
```
{room}_{device_type}_{function}
```

**Examples:**
```
sensor.grow_room_temperature
sensor.grow_room_humidity
switch.grow_room_humidifier
switch.grow_room_dehumidifier
```

**How to rename entities:**
1. Go to Settings > Devices & Services > Entities
2. Click on the entity you want to rename
3. Click the gear icon ⚙️
4. Update "Entity ID" field
5. Save changes

## ⚙️ Configuration

### Step 1: Update Sensor Entity IDs

Edit `VPD_Regulator.yaml` and replace the mock sensors with your real ones:

```yaml
# Find this section and update with your actual sensor entity IDs
sensor:
  - platform: template
    sensors:
      temperature:
        friendly_name: "Grow Room Temperature"
        value_template: "{{ states('sensor.your_actual_temp_sensor') }}"
        unit_of_measurement: "°C"
        device_class: temperature
        
      humidity:
        friendly_name: "Grow Room Humidity"
        value_template: "{{ states('sensor.your_actual_humidity_sensor') }}"
        unit_of_measurement: "%"
        device_class: humidity
```

### Step 2: Update Device Entity IDs

In the same file, find the switch control sections and update:

```yaml
# Find these sections in script.vpd_control and update entity IDs
- service: switch.turn_on
  target:
    entity_id: switch.your_actual_humidifier    # Update this line

- service: switch.turn_on
  target:  
    entity_id: switch.your_actual_dehumidifier  # Update this line
```

### Step 3: Set VPD Targets for Your Growth Cycle

The system comes pre-configured with optimal VPD targets:

```yaml
input_select:
  vpd_target:
    options:
      - "0.4"  # Seedlings/Clones - High humidity needed
      - "0.6"  # Early Vegetative - Building structure
      - "0.8"  # Late Vegetative - Strong growth
      - "1.0"  # Early Flowering - Transition phase
      - "1.2"  # Late Flowering - Prevent mold/mildew
      - "1.4"  # Harvest/Drying - Low humidity
```

### Step 4: Adjust Control Parameters (Optional)

Fine-tune the system behavior:

```yaml
# In script.vpd_control, adjust these values:
# Humidity dead-band (prevents device cycling)
conditions:
  - condition: template
    value_template: "{{ current_rh < (rh_target - 2) }}"  # -2% = turn on humidifier
  - condition: template  
    value_template: "{{ current_rh > (rh_target + 3) }}"  # +3% = turn on dehumidifier

# Update frequency in automation section:
automation:
  - alias: "VPD Control Automation"
    trigger:
      - platform: time_pattern
        minutes: "/30"  # Run every 30 minutes (adjust as needed)
```

### Step 5: Restart and Verify

```bash
# Restart Home Assistant to load new configuration (all platforms)
docker restart homeassistant-vpd

# Check logs for any errors (all platforms)
docker logs -f homeassistant-vpd
```

## 📊 Dashboard

The system includes a pre-configured dashboard accessible at:
**Settings > Dashboards > VPD Control**

### Dashboard Features:
- **Real-time environmental readings**
- **VPD target selection**
- **Device status indicators** 
- **Historical charts** (4-hour trends)
- **Manual control options**
- **Recent event log**

### Using the Dashboard:

1. **Monitor current conditions** in the top section
2. **Adjust VPD target** based on growth stage
3. **Check device status** to ensure proper operation
4. **Review charts** for trends and optimization
5. **Use manual controls** for testing or overrides

## 🔧 Troubleshooting

### Quick Diagnostic Steps

1. **Check entity status:**
   - Go to Developer Tools > States
   - Verify all sensor entities show current values
   - Confirm switch entities respond to manual control

2. **Verify script execution:**
   - Go to Developer Tools > Services
   - Call `script.vpd_control` manually
   - Check Logbook for calculation results

3. **Review automation:**
   - Go to Settings > Automations & Scenes
   - Confirm "VPD Control Automation" is enabled
   - Check automation traces for execution history

### Common Issues and Solutions

#### "Entity not available" Errors
**Problem:** Device offline or entity ID incorrect

**Solution:**
- Check device connectivity/battery
- Verify entity ID spelling in configuration
- Re-pair device if necessary

#### VPD Calculations Not Updating  
**Problem:** Automation not running or script disabled

**Solution:**
- Check Settings > Automations - ensure automation is enabled
- Manually trigger: Developer Tools > Services > `automation.vpd_control_automation`
- Review logs: `docker-compose logs homeassistant`

#### Devices Not Responding
**Problem:** Switch entity IDs incorrect or devices offline

**Solution:**
- Test manual control: Developer Tools > States > click switch entity
- Verify device responds to manual control
- Update entity IDs in configuration if needed

#### Humidity Control Too Aggressive
**Problem:** Dead-band values too small

**Solution:**
- Increase dead-band values in script configuration
- Change from ±2%/±3% to ±5%/±7% for less frequent switching

### Getting Help

If you encounter issues:

1. **Check the logs:**
   ```bash
   # Works on all platforms
   docker logs homeassistant-vpd | grep -i error
   ```

2. **Enable debug logging** (add to VPD_Regulator.yaml):
   ```yaml
   logger:
     default: info
     logs:
       homeassistant.components.script: debug
       homeassistant.helpers.template: debug
   ```

3. **Test individual components:**
   - Use Developer Tools > Template to test calculations
   - Use Developer Tools > Services to test device control
   - Use Developer Tools > States to verify sensor readings

## 🔬 Advanced Usage

### Multiple Grow Rooms

Create separate instances for multiple rooms:

```yaml
# Duplicate and rename the script for each room
script:
  vpd_control_veg_room:
    # ... configure with veg room sensors
    
  vpd_control_flower_room:  
    # ... configure with flower room sensors

# Create separate automations for each room
automation:
  - alias: "VPD Control Veg Room"
    # ... trigger veg room script
    
  - alias: "VPD Control Flower Room"
    # ... trigger flower room script
```

### Custom VPD Schedules

Automatically adjust VPD based on time:

```yaml
automation:
  - alias: "Daytime VPD Schedule"
    trigger:
      - platform: time
        at: "06:00:00"  # Lights on
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vpd_target
        data:
          option: "0.8"  # Active growth VPD

  - alias: "Nighttime VPD Schedule"  
    trigger:
      - platform: time
        at: "18:00:00"  # Lights off
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vpd_target
        data:
          option: "1.0"  # Higher VPD for night
```

### Environmental Alerts

Get notified of critical conditions:

```yaml
automation:
  - alias: "Critical VPD Alert"
    trigger:
      - platform: template
        value_template: "{{ states('sensor.calculated_vpd') | float > 1.6 }}"
    action:
      - service: persistent_notification.create
        data:
          title: "🚨 VPD Critical"
          message: "VPD too high: {{ states('sensor.calculated_vpd') }}kPa - Check devices!"

  - alias: "Device Offline Alert"
    trigger:
      - platform: state
        entity_id: 
          - sensor.grow_room_temperature
          - sensor.grow_room_humidity
        to: "unavailable"
        for: "00:05:00"
    action:
      - service: persistent_notification.create
        data:
          title: "⚠️ Sensor Offline"
          message: "{{ trigger.entity_id }} has been offline for 5 minutes"
```

### Data Export

Export VPD data for analysis:

```yaml
# Log data to CSV format
automation:
  - alias: "VPD Data Logger"
    trigger:
      - platform: time_pattern
        minutes: "/15"  # Every 15 minutes
    action:
      - service: notify.file
        data:
          filename: "/config/vpd_data.csv"
          message: >
            {{ now().isoformat() }},{{ states('sensor.temperature') }},{{ states('sensor.humidity') }},{{ states('input_select.vpd_target') }},{{ states('switch.humidifier') }},{{ states('switch.dehumidifier') }}
```

## 📈 Optimization Tips

### Sensor Placement
- **Temperature sensor**: 18-24 inches from canopy level
- **Humidity sensor**: Same height as temperature, good air circulation
- **Avoid**: Direct light, heat sources, stagnant air pockets

### VPD Target Guidelines
- **Seedlings**: 0.4-0.6 kPa (gentle conditions)
- **Vegetative**: 0.6-0.8 kPa (encourage growth)  
- **Flowering**: 0.8-1.2 kPa (prevent mold/mildew)
- **Late flower**: 1.0-1.4 kPa (reduce humidity)

### System Tuning
- **Start conservative**: Use wider dead-bands initially
- **Monitor device cycles**: Avoid short on/off cycles
- **Adjust gradually**: Small changes over several days
- **Log everything**: Use data to optimize settings

## 📝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test thoroughly with real devices
4. Submit a pull request with clear description

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**🌿 Ready to grow? Clone, configure, and cultivate! 🌿**

For support, please open an issue on GitHub with your configuration details and logs.
