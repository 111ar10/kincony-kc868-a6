# KC868-A6 ESPHome Integration

## Overview
This project provides a comprehensive ESPHome configuration for the KC868-A6 relay controller board, 
enabling seamless integration with Home Assistant and other smart home platforms. 

The KC868-A6 features 6 relay outputs and 6 digital inputs, making it perfect for home automation projects like lighting control, appliance automation, and more.

## Features

- ✅ 6 individually controllable relay outputs
- ✅ 6 digital inputs with configurable triggers
- ✅ Integrated I2C, SPI, UART (RS232 & RS485) support
- ✅ DS1307 real-time clock integration
- ✅ Wi-Fi connectivity with fallback AP
- ✅ OTA (Over-the-Air) updates
- ✅ Home Assistant API integration with encryption
- ✅ Web server for direct control
- ✅ Status LED for visual feedback
- ✅ System monitoring (WiFi signal, uptime)

Here are the instructions to integrate board Kincony KC868-A6 with Esphome and Home Assistant.

For more details check https://devices.esphome.io/devices/KinCony-KC868-A6

## Hardware Requirements

* KC868-A6 relay controller board
* USB 3 cable for firmware flashing
* 12V Power supply appropriate for your relays

# Kincony KC868-A6 firmware upload
![Alt text](https://devices.esphome.io/static/3b5b4421510fc5f23252af03ecde6091/e46b2/KC868-A6.webp?raw=true "KC868-A6")

1. Connect the board with a USB-C cable to your commputer. 
2. While pressing the S2 switch, attach the external power supply (12V) to the board
3. Use esphome to upload new firmware

## Installation

### Prerequisites

1. Install ESPHome. You can do this via Home Assistant or standalone:
   ```bash
   pip install esphome
   ```

2. Clone this repository:
   ```bash
   git clone https://github.com/111ar10/kincony-kc868-a6.git
   cd kc868-a6-esphome
   ```

### Configuration

Before flashing the firmware, you need to customize the configuration file:

1. Copy the example configuration file:
   ```bash
   cp kc868-a6.example.yaml kc868-a6.yaml
   ```

2. Edit the configuration file to set your Wi-Fi credentials, API key, and passwords:
   ```yaml
   # WiFi Configuration
   wifi:
     ssid: "YOUR_WIFI_SSID"
     password: "YOUR_WIFI_PASSWORD"
     
     # Fallback hotspot
     ap:
       ssid: "KC868-A6 Fallback"
       password: "YOUR_FALLBACK_PASSWORD"
   
   # API encryption key
   api:
     encryption:
       key: "YOUR_32_BYTE_ENCRYPTION_KEY"
   
   # OTA password
   ota:
     password: "YOUR_OTA_PASSWORD"


### Flashing

#### Option 1: Using ESPHome Dashboard

1. Add the device to your ESPHome dashboard
2. Upload the YAML configuration
3. Click "Install" and follow the prompts

#### Option 2: Command Line

1. Connect the ESP32 to your computer via USB
2. Run the ESPHome command:
   ```bash
   esphome run kc868-a6.yaml
   ```

## Integration with Home Assistant

After flashing the firmware:

1. Open Home Assistant
2. Navigate to Configuration → Integrations
3. Click the "+ ADD INTEGRATION" button
4. Search for and select "ESPHome"
5. Enter the IP address of your KC868-A6 device
6. The device should be automatically discovered and added to your Home Assistant instance

## Pin Configuration

### ESP32 Board Connections

| Function | Pin |
|----------|-----|
| RS485 TX | GPIO27 |
| RS485 RX | GPIO14 |
| RS232 TX | GPIO17 |
| RS232 RX | GPIO16 |
| SPI CLK | GPIO18 |
| SPI MOSI | GPIO23 |
| SPI MISO | GPIO19 |
| I2C SDA | GPIO4 |
| I2C SCL | GPIO15 |
| Status LED | GPIO2 |

### I/O Expander Configuration

The KC868-A6 uses PCF8574 I/O expanders for input and output control:

- **Inputs PCF8574**: Address 0x22
  - Pins 0-5: Digital inputs 1-6

- **Outputs PCF8574**: Address 0x24
  - Pins 0-5: Relay outputs 1-6

## Usage

### Controlling Relays

Relays can be controlled through:

1. **Home Assistant UI**: Each relay appears as a switch entity
2. **Web Interface**: Access `http://<device-ip>/` in your browser
3. **API Calls**: Use the ESPHome API to control the relays programmatically
4. **Physical Inputs**: Each input is configured to toggle its corresponding relay

### Automation Examples

#### Basic Light Control

```yaml
# Home Assistant automation to turn on Relay 1 at sunset
automation:
  - alias: "Turn on lights at sunset"
    trigger:
      - platform: sun
        event: sunset
        offset: "-00:15:00"
    action:
      - service: switch.turn_on
        target:
          entity_id: switch.kc868_a6_relay_1
```

#### Time-Based Control

```yaml
# Home Assistant automation to toggle Relay 3 on a schedule
automation:
  - alias: "Garden Irrigation Schedule"
    trigger:
      - platform: time
        at: "06:00:00"
    action:
      - service: switch.turn_on
        target:
          entity_id: switch.kc868_a6_relay_3
      - delay: "00:30:00"
      - service: switch.turn_off
        target:
          entity_id: switch.kc868_a6_relay_3
```

## Customization

### Adding Sensors

You can expand the functionality by adding additional sensors to the available interfaces:

```yaml
# Example: Adding a DHT22 temperature/humidity sensor
sensor:
  - platform: dht
    pin: 13
    temperature:
      name: "Room Temperature"
    humidity:
      name: "Room Humidity"
    update_interval: 60s
```

### Modifying Input Behavior

You can change how the inputs behave by modifying their trigger actions:

```yaml
# Example: Modified input to control multiple relays
binary_sensor:
  - platform: gpio
    name: "KC868-A6-IN-1"
    id: input_1
    pin:
      pcf8574: inputs
      number: 0
      mode: INPUT
      inverted: true
    on_press:
      - switch.turn_on: relay_1
      - switch.turn_on: relay_2
    on_release:
      - switch.turn_off: relay_1
      - switch.turn_off: relay_2
```

## Troubleshooting

### Device Not Connecting to Wi-Fi

1. Check if the fallback AP is available (SSID: "KC868-A6 Fallback")
2. Connect to the fallback AP using the configured password
3. Access the web interface at `http://192.168.4.1/`
4. Configure the correct Wi-Fi credentials

### Failed OTA Updates

1. Ensure the device has a stable Wi-Fi connection
2. Verify you're using the correct OTA password
3. Try flashing via USB if OTA is completely non-functional

## Advanced Configuration

### Enabling MQTT (Alternative to Home Assistant API)

```yaml
# Add MQTT support
mqtt:
  broker: 192.168.1.100
  username: mqtt_user
  password: mqtt_password
  
  # Make the switch available via MQTT
  switch:
    - platform: mqtt
      name: "KC868-A6-RELAY-1-MQTT"
      command_topic: "kc868/relay/1/set"
      state_topic: "kc868/relay/1/state"
```

### Adding Additional Security

```yaml
# Add basic authentication to the web server
web_server:
  port: 80
  auth:
    username: admin
    password: admin_password
```

## Hardware Details

The KC868-A6 board features:

- ESP32 microcontroller
- 6 relay outputs (rated for 10A at 250VAC)
- 6 optically isolated digital inputs
- RS485 and RS232 interfaces
- I2C and SPI interfaces
- DS1307 real-time clock with battery backup
- Status LEDs for each relay
- 12V DC power input

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- ESPHome team for creating an amazing framework
- Home Assistant community for their continuous support
- KC868 team for manufacturing versatile relay boards


