# ExpressLRS Serial Bridge Mode

This is a modified version of ExpressLRS that operates as a transparent serial port bridge, with all RC-related functionality removed.

## What Changed?

This modification strips away all RC control functionality, leaving only the serial data bridging capability:

- ✅ **Bidirectional serial data transfer** over RF link
- ✅ **Long range** using LoRa modulation (900 MHz or 2.4 GHz)
- ✅ **Low latency** data transmission
- ✅ **Frequency hopping** (FHSS) for reliability
- ✅ **Link quality monitoring** and statistics
- ❌ **No RC channel output** (no CRSF, SBUS, SUMD, PWM)
- ❌ **No servo control**
- ❌ **No airport mode** (removed as requested)

## Use Cases

This serial bridge is ideal for:
- Long-range telemetry links
- MAVLink communication for drones/rovers
- Sensor data transmission
- Remote serial device control
- Any application requiring wireless serial communication

## How It Works

### TX Module
- Receives serial data from USB or UART
- Packages data into RF packets (PACKET_TYPE_DATA)
- Transmits over LoRa radio
- Receives telemetry data from RX
- Forwards telemetry to serial port

### RX Module
- Receives data packets from TX over RF
- Forwards data to UART output
- Receives data from UART input
- Sends data back to TX as telemetry

### Connection Management
- SYNC packets maintain timing and connection
- Automatic frequency hopping for interference resistance
- Link quality statistics for monitoring

## Building

Use PlatformIO to build as normal:

```bash
cd src
pio run -e <target_environment>
```

Example targets:
- TX: `Unified_ESP32_2400_TX_via_UART`
- RX: `Unified_ESP32_2400_RX_via_UART`
- TX: `Unified_ESP32_900_TX_via_UART`
- RX: `Unified_ESP32_900_RX_via_UART`

## Configuration

Configure using:
- **WiFi interface**: Connect to ExpressLRS WiFi AP and use web interface
- **Lua scripts**: If using OpenTX/EdgeTX
- **ExpressLRS Configurator**: Desktop application

Key settings:
- **RF Rate**: 25Hz to 1000Hz (higher = lower latency, shorter range)
- **TX Power**: Adjust based on range requirements
- **Binding UID**: Must match between TX and RX
- **Serial baud rate**: Configure for your application

## Wiring

### TX Module
- Connect USB or UART to your host device
- Data flows: Host ↔ TX Module ↔ RF ↔ RX Module

### RX Module
- Connect UART pins to your target device
- Data flows: TX Module ↔ RF ↔ RX Module ↔ Target Device

## Performance

- **Range**: Up to several kilometers (900 MHz) or ~1km (2.4 GHz)
- **Latency**: As low as 1-5ms at high packet rates
- **Throughput**: Varies by packet rate (higher rate = more data)
  - 1000 Hz: Highest throughput, lowest latency
  - 500 Hz: Good balance
  - 50 Hz: Maximum range, higher latency

## Technical Details

See [SERIAL_BRIDGE_CHANGES.md](SERIAL_BRIDGE_CHANGES.md) for detailed information about:
- Specific code changes made
- What functionality was removed
- What functionality was preserved
- Future optimization opportunities

## Compatibility

- Works with existing ExpressLRS hardware
- TX and RX must both run this modified firmware
- Cannot interoperate with standard ExpressLRS RC firmware

## Support

This is a community modification. For issues:
1. Check that TX and RX are both bound
2. Verify serial baud rate settings
3. Check RF rate and power settings
4. Monitor link quality statistics
5. Ensure line-of-sight or good RF propagation

## License

Same as ExpressLRS (GPL-3.0)

## Credits

Based on the excellent ExpressLRS project by the ExpressLRS team and community.
