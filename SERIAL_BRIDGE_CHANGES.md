# ExpressLRS Serial Bridge Mode Changes

## Overview
This document describes the changes made to convert ExpressLRS from an RC control link to a pure serial data bridge.

## Changes Made

### 1. RX Main (src/src/rx_main.cpp)

#### Removed RC Packet Handling
- **Removed**: `PACKET_TYPE_RCDATA` packet processing
- **Removed**: `ProcessRfPacket_RC()` function - no longer processes RC channel data
- **Removed**: RC frame available/missed notifications in `HWtimerCallbackTock()`
  - `crsfRCFrameAvailable()` calls removed
  - `servoNewChannelsAvailable()` calls removed
  - `crsfRCFrameMissed()` calls removed

#### Removed Airport Mode
- **Removed**: `#include "rx-serial/SerialAirPort.h"`
- **Removed**: Airport buffer flushing in `GotConnection()`
- **Removed**: Airport mode initialization in `setupSerial()`
- **Removed**: Airport data unpacking in packet processing
- **Modified**: Telemetry data queueing to only use `DataDlSender` (no airport buffers)

#### Removed Servo Output Device
- **Removed**: `#include "devServoOutput.h"`
- **Removed**: `{&ServoOut_device, 1}` from `ui_devices[]` array

#### Simplified Packet Processing
- **Modified**: Switch statement to only handle `PACKET_TYPE_SYNC` and `PACKET_TYPE_DATA`
- All packets are now processed as serial data (no RC channels)

### 2. TX Main (src/src/tx_main.cpp)

#### Removed RC Channel Transmission
- **Removed**: `OtaPackChannelData()` call
- **Removed**: Toggle logic between RC and data packets (`NextPacketIsDataUl`)
- **Modified**: Packet building to ALWAYS send `PACKET_TYPE_DATA` (never RC packets)

#### Removed Airport Mode
- **Removed**: Airport buffer declarations (`apInputBuffer`, `apOutputBuffer`)
- **Removed**: Airport mode packet packing

#### Simplified Packet Building
- **Modified**: Non-SYNC packets now always send serial data via `DataUlSender`
- SYNC packets are still sent for maintaining connection timing
- Data packets are sent with proper acknowledgment handling for MAVLink mode

### 3. What Still Works

The following functionality is preserved:
- **SYNC packets**: Connection establishment and maintenance
- **DATA packets**: Bidirectional serial data transfer
- **Link statistics**: Quality monitoring and reporting
- **FHSS**: Frequency hopping for reliability
- **Telemetry downlink**: Data from RX to TX
- **Binding mode**: Device pairing (optional, can be removed if not needed)
- **Configuration**: WiFi and Lua script configuration interfaces

### 4. What Was Removed

The following RC-related functionality is no longer available:
- **RC channel output**: No CRSF, SBUS, SUMD, or PWM RC channels
- **Servo control**: No servo/motor output
- **RC frame timing**: No periodic RC frame generation
- **Airport mode**: Removed as requested
- **Channel data packing/unpacking**: Functions exist but are unused

## Usage

The modified firmware now operates as a transparent serial bridge:

1. **TX Side**: 
   - Receives serial data from USB/UART
   - Transmits data over RF as DATA packets
   - Receives telemetry data from RX
   - Forwards telemetry to USB/UART

2. **RX Side**:
   - Receives DATA packets over RF
   - Forwards data to serial output (UART)
   - Receives serial data from UART
   - Sends data back to TX as telemetry

## Build Instructions

Build the firmware using PlatformIO as usual:
```bash
cd src
pio run -e <your_target_environment>
```

Example targets:
- `Unified_ESP32_2400_TX_via_UART`
- `Unified_ESP32_2400_RX_via_UART`
- `Unified_ESP32_900_TX_via_UART`
- `Unified_ESP32_900_RX_via_UART`

## Configuration

The serial bridge uses the existing ExpressLRS configuration system:
- RF rate selection (25Hz to 1000Hz)
- TX power settings
- Binding UID
- Serial port configuration

RC-specific settings (servo outputs, switch modes, etc.) are ignored.

## Future Optimizations (Optional)

The following could be removed for further simplification:
1. Binding mode logic (if not needed)
2. Model matching logic (if not needed)
3. OTA RC channel pack/unpack functions (dead code now)
4. RC serial protocol handlers (SBUS, SUMD output classes)
5. Servo output library completely
6. RC-specific configuration options

## Notes

- The changes follow a minimal modification approach
- Existing timing and synchronization mechanisms are preserved
- The system still uses SYNC packets for connection management
- Link quality and statistics are maintained for debugging
- The code maintains compatibility with the existing device framework
