# Embedded Communication Protocols Handbook

## Table of Contents

1. [How to Read This Handbook](#1-how-to-read-this-handbook)
2. [Master Protocol Index](#2-master-protocol-index)
3. [Detailed Protocol Entries](#3-detailed-protocol-entries)
4. [Protocol Stacks](#4-protocol-stacks)
5. [Comparison Tables](#5-comparison-tables)
6. [Physical Layer Reference](#6-physical-layer-reference)
7. [Serial Communication Fundamentals](#7-serial-communication-fundamentals)
8. [Embedded Driver Design](#8-embedded-driver-design)
9. [Linux Device Driver View](#9-linux-device-driver-view)
10. [RTOS View](#10-rtos-view)
11. [Packet / Frame Analysis](#11-packet--frame-analysis)
12. [Debugging Tools](#12-debugging-tools)
13. [Common Embedded Protocol Debugging Workflow](#13-common-embedded-protocol-debugging-workflow)
14. [Testing](#14-testing)
15. [Security](#15-security)
16. [Real-World System Architectures](#16-real-world-system-architectures)
17. [Protocol Selection Guide](#17-protocol-selection-guide)
18. [Learning Roadmap](#18-learning-roadmap)
19. [Projects](#19-projects)
20. [Glossary](#20-glossary)
21. [Final Master Table](#21-final-master-table)

---

## 1. How to Read This Handbook

### Key Terminology

| Term | Definition |
|------|------------|
| **Protocol** | Rules for exchanging information (e.g., CAN, MQTT, SPI) |
| **Interface** | Physical or logical boundary enabling access (e.g., UART peripheral, USB controller) |
| **Bus** | Shared communication medium with multiple participants (e.g., I2C bus, CAN bus) |
| **Standard** | Formally published specification governing interoperability (e.g., IEEE 802.3, PCIe, USB) |

### Layering (OSI Model Adapted for Embedded)

| Layer | Description | Examples |
|-------|-------------|----------|
| **Physical** | Signaling, voltage, cabling, connectors, termination | RS-485, LVDS, MIPI PHY |
| **Data-Link** | Framing, addressing, error detection within a link | CAN, Ethernet MAC, I2C, SPI |
| **Network** | Routing and logical addressing | IP, ARP |
| **Transport** | End-to-end communication | TCP, UDP, TLS |
| **Application** | Higher-level semantics | MQTT, Modbus, HTTP, DICOM |

### Communication Characteristics

- **Synchronous vs Asynchronous**: Clock-based vs timing-based communication
- **Serial vs Parallel**: Single-bit vs multi-bit data paths
- **Topologies**: Point-to-point, multi-drop, multi-master, star, mesh
- **Duplex**: Full-duplex, half-duplex, simplex
- **Control Models**: Polling, interrupt-driven, DMA-based
- **Terminology**: Master/Slave vs Controller/Peripheral vs Host/Device
- **Data Units**: Bit, byte, frame, packet, message, transaction
- **Performance**: Bandwidth vs throughput vs latency
- **Timing**: Deterministic vs non-deterministic communication

---

## 2. Master Protocol Index

| Technology | Category | Layer | Physical Medium | Topology | Duplex | Typical Speed | Distance | Typical Devices | Industry | Complexity | Linux Support | MCU Support | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| GPIO | Interface/Peripheral | Physical | Electrical wires | Point-to-point | N/A | N/A | N/A | Microcontrollers, SBCs | All | Low | Varies | High | Medium |
| UART | Serial bus | Data-Link/Physical | Conductors | Point-to-point | Full/Half | 1–5 Mbit/s common; up to 10–100 Mbit/s in specialized links | Short | Microcontrollers, SoCs | All | Medium | Yes | Yes | Medium |
| USART | Serial interface | Physical | Differential/single-ended | Point-to-point | Full/Half | Similar to UART with additional modes | Short | MCU peripherals | All | Medium | Yes | Yes | Medium |
| I2C | Serial bus | Data-Link | Two-wire | Multi-master multi-slave | Half | 100–400 kbit/s standard; up to 3.4 Mbit/s | Short | MCU peripherals, sensors | Industrial/Consumer | Medium | Yes | Yes | Medium |
| I3C | Serial bus | Data-Link | Differential-like over I2C-phy | Multi-master | Full | 12.5–33 Mbit/s (exceeding I2C) | Short | Sensors, MCU bridges | Industrial/Consumer | High | Growing | Yes | High | Advanced |
| SPI | Serial bus | Data-Link | Four or more wires | Point-to-point or multi-slave | Full | 1–50 Mbit/s common; higher in specialized PHYs | Short | Flash, sensors, sensors, DACs | All | Medium | Yes | Yes | Medium-High |
| QSPI | Serial bus | Data-Link | Quad SPI lines | Point-to-point | Full | 50–200+ Mbit/s | Short | Flash memory | Embedded storage | Medium | Yes | Yes | Medium-High |
| OSPI | Serial bus | Data-Link | Octal SPI / wide bus | Point-to-point | Full | Hundreds of MHz | Short | NOR/NAND/flash | Embedded storage | High | Yes | Yes | High |
| I2S | Audio/Serial | Data-Link | Multi-wire | Point-to-point | Full | 64–768 kHz sample rates; bit depths up to 32-bit | Short | DAC/ADC, audio codecs | Audio/MCU | Medium | Yes | Yes | Medium |
| TDM | Audio/Serial | Data-Link | Time-Division multiplex | Point-to-point | Full | Variable | Short | Audio pipelines | Audio | Medium | Yes | Yes | Medium |
| PDM | Audio/Serial | Data-Link | Pulse density modulated | Point-to-point | Mono | kHz ranges (audio) | Short | MEMS microphones | Consumer/IoT | Low | Yes | Yes | Low |
| SAI | Serial audio | Data-Link | SAI blocks | Multi-channel | Full | Varies | Short | SoC audio subsystems | Embedded Audio | Medium | Yes | Yes | Medium |
| 1-Wire | Serial bus | Data-Link | Single wire | Point-to-point | Half | 16 kbit/s–16 Mbit/s depending on mode | Short | 1-Wire devices | Consumer/Industrial | Low | Yes | Yes | Low |
| PWM | Interface/Control | Physical | PWM lines | N/A | N/A | kHz range | N/A | MCU timers | All | Low | Yes | Yes | Low |
| ADC/DAC interfaces | Interfaces | Physical/Data-Link | Analog/Digital physical | N/A | N/A | N/A | N/A | Analog peripherals | All | Low | Yes | Yes | Low |
| RS-232 | Serial electrical | Electrical/Physical | Single-ended | Point-to-point | Full | Up to ~115 kbit/s typical; higher with modern transceivers | Point-to-point | PC/telecom gear, embedded systems | All | Medium | Yes | Yes | Medium |
| RS-422 | Serial electrical | Electrical/Physical | Differential | Multi-drop | Full | 100 kbit/s–10 Mbit/s | Medium | Industrial devices | Industrial | Medium | Yes | Yes | Medium |
| RS-485 | Serial electrical | Electrical/Physical | Differential | Multi-drop/multi-point | Half/Full | 1 kbit/s–20 Mbit/s | Medium | Industrial controllers, sensors | Industrial | Medium | Yes | Yes | Medium |
| RS-423 | Serial electrical | Electrical/Physical | Single-ended | Point-to-point | N/A | Similar to RS-232 | Short/Medium | Legacy devices | Industrial | Medium | Limited | Yes | Low |
| CAN | Automotive/Industrial | Data-Link/Network | Differential | Multi-master multi-drop | Multi | 1 Mbit/s (classic); up to 8 Mbit/s (CAN FD) | Short to medium | ECUs, sensors | Automotive/Industrial | High | Yes | Yes | High |
| CAN-FD | CAN extension | Data-Link/Network | Differential | Multi-master | Multi | Higher data rates than CAN | Short | Automotive/Industrial | Automotive | High | Yes | Yes | High |
| CAN XL | CAN extension | Data-Link/Network | Differential | Multi-master | Multi | Very high raw rates | Short | Automotive | Automotive | High | Yes | Yes | High |
| LIN | Automotive | Data-Link | Single-wire | Star/line | Half | 20 kbit/s–33.3 kbit/s | Short | Body networks | Automotive | Medium | Yes | Yes | Medium |
| FlexRay | Automotive | Data-Link | Twin-axial | Multi-drop | Dual channel | 10–40 Mbit/s | Medium | High-end ECUs | Automotive | High | Emerging | Yes | High | High |
| Automotive Ethernet | Automotive | Network/Link | Ethernet PHY | Star/mesh | Full/Duplex | 100 Mbps–10 Gbps | Vehicle length scale | ECUs, gateways | Automotive | High | Yes | Yes | High |
| 100BASE-T1 | Automotive Ethernet | Link | Twisted pair | Point-to-point | Full | 100 Mbit/s | Short to medium | ECUs | Automotive | Medium | Yes | Yes | Medium |
| 1000BASE-T1 | Automotive Ethernet | Link | Twisted pair | Point-to-point | Full | 1 Gbit/s | Short | ECUs | Automotive | Medium | Yes | Yes | Medium |
| UDS | Automotive protocol | Application/Transport | Ethernet/ CAN | Node-to-node | N/A | Variable | Vehicle network | Diagnostics | Automotive | High | Yes | Yes | High |
| DoIP | Automotive | Transport/Application | Ethernet | Star | Full | 100 Mbps+ | Long | Diagnostic gateways | Automotive | High | Yes | Yes | High |
| OBD-II | Automotive diagnostics | Application | CAN/Ethernet | Master/slave | N/A | 1 kbit/s–125 kbit/s (CAN); higher on Ethernet variants | Vehicle-wide | Diagnostic tools | Automotive | Medium | Yes | Yes | Medium |
| J1939 | Automotive higher-layer | Application | CAN | Multi-node | Full | Up to 250 kbit/s typical (CAN FD higher) | Medium | Heavy-duty vehicles | Automotive | Medium | Yes | Yes | Medium |
| SOME/IP | Automotive service protocol | Application | Ethernet | Star/Hub | Full | High | Vehicle domain | Service discovery | Automotive | Medium-High | Yes | Yes | Medium |
| AUTOSAR communication | Automotive standard concepts | Application/Architecture | Varied | Varied | Varied | Varied | Varied | Varied | Automotive | High | Yes | Yes | High |
| CANopen | Industrial/Automotive | Application | CAN | Multi-drop | Full | Up to 1 Mbit/s | Short | Motion controllers, sensors | Industrial | Medium | Yes | Yes | Medium |
| Modbus RTU | Industrial | Application/Transport | RS-485/RS-232 | Multi-drop | Full | 7–115200 bps | Medium | PLCs, RTUs | Industrial | Medium | Yes | Yes | Medium |
| Modbus ASCII | Industrial | Application/Transport | RS-232/RS-485 | Multi-drop | Full | 7–115200 bps | Medium | PLCs | Industrial | Medium | Yes | Yes | Medium |
| Modbus TCP | Industrial | Application/Transport | Ethernet | Star | Full | 10/100 Mbps | Medium | PLCs, gateways | Industrial | Medium | Yes | Yes | Medium |
| PROFIBUS | Industrial | Fieldbus | copper | Bus | Multi-master | 9.6 kbps–12 Mbit/s | Medium | Controllers, devices | Industrial | High | Yes | Yes | High |
| PROFINET | Industrial Ethernet | Application/Transport | Ethernet | Star/daisy-chain | Full | 100 Mbps–1 Gbps | Medium | PLCs, IO devices | Industrial | High | Yes | Yes | High |
| EtherCAT | Industrial | Fieldbus/Network | Ethernet | Line/Tree | Full | 100 Mbps | Short | Motion, I/O | Industrial | High | Yes | Yes | High |
| EtherNet/IP | Industrial Ethernet | Application/Transport | Ethernet | Star/Tree | Full | 10 Mbps–1 Gbps | Medium | PLCs, IO devices | Industrial | Medium-High | Yes | Yes | Medium |
| DeviceNet | Industrial | Fieldbus | CAN | Multi-drop | Full | 125–500 kbps | Short | PLCs, I/O | Industrial | Medium | Yes | Yes | Medium |
| USB | Computer / Peripheral | Data-Link/Transport | USB cable | Point-to-point / hub | Full | 12 Mbit/s (USB 1.x) up to 40 Gbit/s (USB4) | Short | Peripherals, mass storage | Computer/Embedded | Medium-High | Yes | Yes | High |
| USB 1.x | USB | Data-Link/Physical | USB 1.x signaling | Point-to-point | Full | 1.5–12 Mbit/s | Short | Legacy devices | General | Low | Partial | Partial | Low |
| USB 2.0 | USB | Data-Link/Physical | Hi-Speed USB | Point-to-point | Full | 480 Mbit/s | Short | Mass storage, HID | General | Medium | Yes | Yes | Medium |
| USB 3.x | USB | Data-Link/Physical | SuperSpeed | Point-to-point | Full | 5–40 Gbit/s | Short | Modern peripherals | General | High | Yes | Yes | High |
| USB-C / USB OTG | USB | Physical/Transport | USB-C connector | Point-to-point | Full | 480 Mbps–40 Gbps | Short | Reversible connectors | General | High | Yes | Yes | High |
| USB HID | USB class | Application | USB | N/A | N/A | 1.5–12 Mbit/s | Short | Human input devices | General | Low | Yes | Yes | Medium |
| USB CDC | USB class | Application | USB | N/A | N/A | 1.5–12 Mbit/s | Short | Virtual COM ports | General | Medium | Yes | Yes | Medium |
| USB MSC | USB class | Application | USB | N/A | N/A | 1.5–12 Mbit/s | Short | Mass storage | General | Medium | Yes | Yes | Medium |
| USB Audio | USB class | Application | USB | N/A | N/A | 1.5–12 Mbit/s | Short | Audio devices | Audio | Medium | Yes | Yes | Medium |
| USB Video Class | USB class | Application | USB | N/A | N/A | 1.5–12 Mbit/s | Short | Video devices | General | Medium | Yes | Yes | Medium |
| USB DFU | USB class / bootloader | Application | USB | N/A | N/A | 1.5–12 Mbit/s | Short | Firmware upgrade | General | Medium | Yes | Yes | Medium |
| PCI Express | Computer/PCIe | Link/Physical | PCIe cables | Point-to-point | Full | Gen1–Gen6 rates up to multi-Gbps | Short to medium | PCIe devices | Computing | High | Yes | Yes | High |
| PCI | Computer | Link/Physical | Parallel/serial (older) | Point-to-point | N/A | 33–133 MHz | Short | Expansion cards | General | Medium-High | Partial | Partial | Medium |
| SD / SDIO | Storage | Interface | SD/SDIO cards | Point-to-point | Full | 12.5–104 Mbit/s (SDR50/SDR104) | Short | Memory cards | General | Low | Yes | Yes | Low |
| eMMC | Storage | Interface | eMMC flash | Point-to-point | Full | 52–200+ Mbit/s | Short | Embedded storage | General | Medium | Yes | Yes | Medium |
| UFS | Storage | Interface | Universal Flash | Point-to-point | Full | 1–2.9 Gbit/s | Short | High-performance storage | General | High | Yes | Yes | High |
| NVMe | Storage | Protocol/Transport | PCIe over PCIe/NVMe | Point-to-point | Full | 1–8+ Gbps | Short | SSDs | Data Center/Industrial | High | Yes | Yes | High |
| NAND/NOR Flash Interfaces | Storage | Interface | SPI/NAND/NOR | Point-to-point | Full | Varies by device | Short | Flash memory | General | Medium | Yes | Yes | Medium |
| SPI NOR Flash | Storage | Interface | SPI | Point-to-point | Full | Several tens of MHz | Short | Flash memory | General | Low | Yes | Yes | Low |
| NAND Flash | Storage | Interface | Parallel/8/16-bit | Point-to-point | Full | Varies | Short | Flash memory | General | Medium | Yes | Yes | Medium |
| MIPI CSI-2 | Camera/Imaging | Interface | High-speed serial lanes | Point-to-point | Full | 1–12 Gbit/s per lane | Short | Image sensors | Medical/Camera | High | Yes | Yes | High |
| MIPI DSI | Display | Interface | High-speed lanes | Point-to-point | Full | Gbps | Short | Displays | Consumer/Embedded | High | Yes | Yes | High |
| LVDS | Display/Video | Interface | Differential pair | Point-to-point | Full | 1–4 Gbit/s | Medium | Video panels | Industrial/Automotive | Medium | Yes | Yes | Medium |
| HDMI / DisplayPort | Display/Video | Interface | Differential | Point-to-point | Full | 2–18+ Gbit/s | Short | Displays/monitors | General | High | Yes | Yes | High |
| Ethernet | Networking | Data-Link/Network/Physical | Twisted pair / fiber | Star/Tree | Full | 10 Mbps–100 Gbps | Medium | Switches, hosts | IT/Industrial | High | Yes | Yes | High |
| IPv4 | Networking | Network | IP | N/A | N/A | 10 Mbps–10 Gbps+ | Global | Routers, hosts | Internet/Industrial | Medium | Yes | Yes | Medium |
| IPv6 | Networking | Network | IP | N/A | N/A | 10 Mbps–10 Gbps+ | Global | Routers, hosts | Internet/Industrial | Medium | Yes | Yes | Medium |
| ARP | Networking | Data-Link/Network | Broadcast | N/A | N/A | N/A | Local | IP_ADDRESS resolution | General | Low | Yes | Yes | Low |
| TCP | Transport | Transport | Layered over IP | N/A | Full | 10 Mbps–10 Gbps+ | End-to-end | Web servers, MQTT over TCP | Internet/Industrial | Medium | Yes | Yes | Medium |
| UDP | Transport | Transport | Layered over IP | N/A | Full | Similar to TCP | End-to-end | Real-time datagrams | General | Low-Medium | Yes | Yes | Medium |
| TLS/DTLS | Transport Security | Transport/App | Over TCP/UDP | End-to-end | Full | TLS overhead varies | End-to-end | Secure channels | Internet/Industrial | High | Yes | Yes | High |
| HTTP(S) | Application | App | TCP/TLS | Star | Full | 0.5–10 Gbps practical | Global | Web services | Internet/IoT | Medium | Yes | Yes | Medium-High |
| MQTT | Application | App/Protocol | TCP/TLS/UDP | Pub/Sub | Full | 1–1000 kbit/s typical | Global | IoT buses | IoT | Low-Medium | Yes | Yes | Medium |
| CoAP | Application | App/Protocol | UDP/TCP | Client/Server | Full | 1–250 kbps typical | Local/Regional | IoT devices | IoT | Low-Medium | Yes | Yes | Medium |
| AMQP | Application | Messaging | TCP/TLS | Brokered | Full | High reliability | Global | Enterprise apps | Enterprise | High | Yes | Yes | High |
| OPC UA | Industrial/Automation | Application | TCP/HTTPS | Client/Server | Full | High | Global | PLCs, SCADA | Industrial | High | Yes | Yes | High |
| BACnet | Building Automation | Application/Protocol | IP/MS/RTU | Networked | Full | Varies | Building-scale | Building devices | Building automation | Medium | Yes | Yes | Medium |
| KNX | Building Automation | Application/Protocol | Twisted pair/Wireless | Networked | Full | Varies | Building-scale | Builders, sensors | Building | Medium | Yes | Yes | Medium |
| Thread | Wireless Mesh | Network/Link/Application | IEEE 802.15.4 | Mesh | Full | 20 kbps–250 kbps | Home/Building | IoT devices | IoT | Medium | Yes | Yes | Medium |
| Zigbee | Wireless Mesh | Network/Link/Application | IEEE 802.15.4 | Mesh | Full | 250 kbps | Home/Building | Home devices | IoT | Low-Medium | Yes | Yes | Low |
| BLE (Classic) | Wireless | Link/Application | 2.4 GHz | Star/Point-to-point | Full | 1–3 Mbps | Short | HID, peripherals | Consumer | Medium | Yes | Yes | Medium |
| BLE (LE) / BLE GATT | Wireless | Link/Application | 2.4 GHz | Star/Mesh | Full | 125 kbps–2 Mbps | Short | Peripherals | IoT | Medium | Yes | Yes | Medium |
| Zigbee IP | Wireless | Network | IP over Zigbee | Mesh | Full | 250 kbit/s | Short | Home networks | IoT | Medium | Yes | Yes | Medium |
| LoRa / LoRaWAN | Wireless Wide Area | Physical/Data-Link/Application | Sub-GHz | Star/LoRaWAN | Half | 0.3–50 kbps | Long | Remote sensors | IoT | Medium | Partial | Partial | Medium |
| NFC / RFID | Wireless | Interface | Near-field | Proximity | Half | 424 kbit/s–848 kbit/s | cm-scale | Access cards, tags | Security/Logistics | Low | Yes | Yes | Low |
| UWB | Wireless | Physical/Link | Ultra-wideband | Point-to-point/mesh | Full | 110–4800 Mbps | Short | Precise ranging devices | Industrial/Automotive | High | Yes | Yes | High |
| Cellular / LTE / 5G | Wireless Wide Area | Network/Link | RF cellular | Star | Full | Mbps–Gbps | Global | IoT devices, gateways | Telecom/IoT | High | Yes | Yes | High |
| DICOM / HL7 / FHIR | Medical protocols | Application/Data | Hospital networks | Star/Client-Server | Full | Varied | Hospital scale | Medical devices | Healthcare | High | Limited | Limited | High |
| Matter | IoT / Home | Application/Network | Wi‑Fi/Ethernet/Thread | Mesh/Hybrid | Full | Varies | Home scale | Smart devices | IoT/Home | Medium | Yes | Yes | Medium |
| TSN / IEEE 802.1Qbv etc. | Real-time Ethernet | Data-Link/Physical | Ethernet | Star/Tree/Hybrid | Full | 100 Mbps–10 Gbps | Local | Industrial | Industrial | High | Yes | Yes | High |
| Ethernet Powerlink | Real-time Ethernet | Protocol/Layer | Ethernet | Networked | Full | Real-time | Local | Motion controllers | Industrial | High | Yes | Yes | High |

---

## 3. Detailed Protocol Entries

### 3.1 GPIO

#### Overview
- **What it is**: General-purpose input/output; a configurable digital pin used to read sensors or drive outputs
- **Why it exists**: Provides flexible, low-level control of hardware
- **Where it used**: MCU/SoC peripherals, baseboard signaling, simple bit-banging interfaces

#### Classification

| Property | Value |
|---|---|
| Category | Interface / Peripheral |
| OSI Layer | Physical (conceptual at pin level) |
| Communication Type | N/A (digital I/O) |
| Physical Medium | Copper trace, package pins |
| Topology | Point-to-point (pin-level) |
| Duplex | N/A |
| Synchronization | N/A |
| Addressing | Pin number / GPIO bank |
| Typical Speed | Dependent on software; typically tens of kHz to MHz when toggled in software |
| Maximum/Typical Distance | On-board only (trace length) |
| Deterministic | Deterministic in timing if driven by hardware clocks or tight loops; otherwise non-deterministic in software |
| Multi-device Support | Shared across multiple pins in a port; no intrinsic bus protocol |

#### Architecture
```
Application
    ↓
GPIO driver calls
    ↓
HAL/LL
    ↓
GPIO peripheral
    ↓
Pad/Pin
```

ASCII:
```
Device A (MCU GPIO) ----> Device B (driven pin or sampled pin)
```

#### Physical Layer
- **Voltage levels**: depends on MCU (typically 0–VDD, Vih/Vil thresholds)
- **Signaling**: single-ended
- **Connectors**: PCB traces; headers
- **Termination**: usually none
- **Isolation**: none by default

#### Communication Model
- **Sender**: CPU/peripheral writes output high/low
- **Receiver**: reads input state
- **Addressing**: pin identifier
- **Arbitration**: none
- **Synchronization**: software-driven

#### Frame / Packet / Message Format
- Not applicable; single-bit signaling

#### Timing
- **Clocking**: software-timed or hardware-timed toggling
- **Setup/Hold**: depends on CPU cycles
- **Timeout**: software timeouts

#### Error Detection / Recovery
- None inherent; rely on software validation

#### Addressing
- Pin number mapping

#### Example Communication Sequence
1. Configure pin as output
2. Set high/low to drive signal
3. Read input when needed

#### C Example
```c
// simple GPIO blink example (pseudo-C)
void gpio_init(uint32_t pin) { /* enable clock, set mode */ }
void gpio_write(uint32_t pin, bool val) { /* write register */ }
bool gpio_read(uint32_t pin) { /* read register */ return (reg & (1<<pin)); }

int main(void) {
  gpio_init(13);
  while (1) {
    gpio_write(13, true);
    delay_ms(500);
    gpio_write(13, false);
    delay_ms(500);
  }
}
```

#### C++ Example
```cpp
class GpioPin {
public:
  GpioPin(uint32_t pin) : pin_(pin) { init(); }
  void set(bool v) { /* write hardware register */ }
  bool read() const { return /* read register */; }

private:
  uint32_t pin_;
  void init(); // RAII-style init
};
```

#### Embedded Driver Architecture
```
Application -> GPIO API -> HAL -> Peripheral Register -> Hardware
```

#### Interrupt / DMA
- IRQs allow edge/level detection; DMA not typically used for simple GPIO

#### RTOS Usage
- Tasks may toggle pins; ISRs for input events; queues for event signaling

#### Embedded Linux
- GPIO sysfs or character device interfaces; device tree binding

#### Tools

| Tool | Purpose |
|---|---|
| Logic Analyzer | Capture pin state over time |
| Oscilloscope | Measure timing of toggles |

#### Debugging Checklist
- [ ] Verify power rails
- [ ] Verify pin multiplexing
- [ ] Check pin direction and pull-ups/downs
- [ ] Capture edge timing

#### Common Mistakes
- Misconfigured pin modes
- Floating inputs without pull resistors
- Missing debouncing

#### Security Considerations
- Physical tampering; not a typical attack surface unless used for control channels

#### Advantages
- Simple, flexible, low overhead

#### Disadvantages
- No inherent protocol; no standard framing

#### When to Use
- Simple signaling, bit-banging, wakeup lines

#### When NOT to Use
- Structured communication with error handling

#### Similar Technologies
- Digital I/O abstraction vs wireless signaling

#### Related Technologies
- [SPI](#36-spi), [I2C](#34-i2c) for actual data networks

#### Learning Priority
- **Beginner**

---

### 3.2 UART

#### Overview
- **What it is**: Asynchronous serial data transmission using a pair of lines (TX/RX); optionally includes parity, stop bits, and a baud rate
- **Why it exists**: Simple, low-cost, widely supported serial communications
- **Where it used**: Console ports, device communication, bootloaders, debugging interfaces

#### Classification

| Property | Value |
|---|---|
| Category | Serial data link / Protocol (asynchronous) |
| OSI Layer | Data-Link (conceptual) / Physical |
| Communication Type | Asynchronous serial |
| Physical Medium | Copper trace or cable |
| Topology | Point-to-point |
| Duplex | Full/half (depends on wiring) |
| Synchronization | Asynchronous with baud timing |
| Addressing | Not defined by UART itself; higher-layer addressing if used |
| Typical Speed | 9600–115200 bps common; up to several Mbps with modern transceivers |
| Maximum/Typical Distance | Short to moderate (meters with proper transceivers) |
| Deterministic | Not strictly deterministic; depends on baud and buffering |
| Multi-device Support | Point-to-point, multiple UARTs per MCU |

#### Architecture
```
Application
    ↓
Protocol/Application Layer
    ↓
Transport/Serial Driver
    ↓
UART Controller
    ↓
Hardware TX/RX pins
```

ASCII:
```
Device A (MCU) ---- UART ---- Device B (MCU)
```

#### Physical Layer
- **Signaling**: Single-ended TTL or RS-232 level shifters depending on standard
- **Wires**: TX, RX, GND (often RTS/CTS for hardware flow control)
- **Termination**: Not required in short links
- **Connectors**: DB9/DB25 (RS-232) or 3.3V/5V TTL headers

#### Communication Model
- **Sender**: Transmitter UART
- **Receiver**: Receiver UART
- **Arbitration**: None
- **Synchronization**: Based on agreed baud rate
- **Flow control**: Optional hardware (RTS/CTS) or software

#### Frame / Packet / Message Format
- Serial frames defined by protocol on top of UART (e.g., simple start/stop framing)

#### Timing
- **Baud rate** controls bit timing; errors produce framing errors

#### Error Detection / Recovery
- Parity optionally; higher-layer protocols provide CRC/ACK/NACK

#### Addressing
- Not inherent in UART; use higher-layer addressing if needed

#### Example Communication Sequence
1. Both sides set same baud and frame format
2. Transmitter sends bytes
3. Receiver reads and validates framing

#### C Example
```c
void uart_send(const uint8_t* data, size_t len);
size_t uart_recv(uint8_t* buf, size_t maxlen);
```

#### C++ Example
```cpp
class Uart {
public:
  Uart(uint32_t baud);
  void write(const std::vector<uint8_t>& data);
  std::vector<uint8_t> read(size_t n);
private:
  uint32_t baud_;
  // HAL handles
};
```

#### Embedded Driver Architecture
```
Application -> UART API -> HAL/LL -> UART peripheral -> Hardware
```

#### Interrupt / DMA
- IRQs when data arrives
- DMA can be used for large blocks to reduce CPU load

#### RTOS Usage
- ISR for RX; task reads from queue; TX via blocking/non-blocking calls

#### Embedded Linux
- UART driver stack; /dev/ttyS* or /dev/ttyAMA*, termios API

#### Tools

| Tool | Purpose |
|---|---|
| Logic Analyzer | Decode serial frames |
| Oscilloscope | View signal integrity |
| Baud rate tester | Validate baud matching |

#### Debugging Checklist
- [ ] Check baud, parity, stop bits
- [ ] Verify wiring and ground
- [ ] Verify flow control settings
- [ ] Capture and decode frames

#### Common Mistakes
- Mismatched baud/parity
- Missing ground return
- Buffer overflows

#### Security Considerations
- Eavesdropping risk on unencrypted UART; protect with encryption in higher layers

#### Advantages
- Simple, ubiquitous

#### Disadvantages
- No inherent security or framing

#### When to Use
- Debug consoles, low-speed device comms

#### When NOT to Use
- High-throughput or multi-device networks needing framing

#### Similar Technologies
- [RS-232](#313-rs-232), [RS-485](#315-rs-485) as alternative serial standards

#### Related Technologies
- [SPI](#36-spi), [I2C](#34-i2c) for bus-based devices

#### Learning Priority
- **Beginner**

---

### 3.3 I2C

#### Overview
- **What it is**: Two-wire synchronous serial bus with multi-master and multi-slave support
- **Why it exists**: Simple, low-pin-count bus for connecting multiple sensors and peripherals
- **Where it used**: Sensors, EEPROMs, real-time clocks, GPIO expanders

#### Classification

| Property | Value |
|---|---|
| Category | Serial bus |
| OSI Layer | Data-Link |
| Communication Type | Synchronous serial |
| Physical Medium | Two-wire (SDA, SCL) |
| Topology | Multi-master, multi-slave |
| Duplex | Half-duplex |
| Synchronization | Clock (SCL) |
| Addressing | 7-bit or 10-bit device address |
| Typical Speed | 100 kbit/s (Standard), 400 kbit/s (Fast), 1 Mbit/s (Fast+), 3.4 Mbit/s (High-Speed) |
| Maximum/Typical Distance | Short (cm to ~1m with proper design) |
| Deterministic | No (arbitration-based) |
| Multi-device Support | Yes (up to 127 devices with 7-bit addressing) |

#### Architecture
```
Application
    ↓
I2C Driver
    ↓
I2C Controller
    ↓
SDA/SCL lines
    ↓
Multiple slave devices
```

ASCII:
```
        SDA ----+----+----+----+
                |    |    |    |
        SCL ----+----+----+----+
               MCU  Dev1 Dev2 Dev3
```

#### Physical Layer
- **Signaling**: Open-drain with pull-up resistors
- **Wires**: SDA (data), SCL (clock), GND
- **Voltage levels**: Typically 3.3V or 5V; I2C level shifters available
- **Pull-ups**: Required on SDA and SCL (value depends on bus capacitance and speed)
- **Termination**: Not required; pull-ups serve as termination

#### Communication Model
- **Master**: Initiates transactions, generates clock
- **Slave**: Responds to address, sends/receives data
- **Arbitration**: Multi-master supported via clock synchronization and data monitoring
- **Synchronization**: Clock provided by master

#### Frame / Packet / Message Format

| Field | Size | Description |
|---|---:|---|
| START | 1 bit | Start condition (SDA falls while SCL high) |
| Address | 7 or 10 bits | Slave address |
| R/W | 1 bit | Read (1) or Write (0) |
| ACK/NACK | 1 bit | Acknowledge from slave |
| Data | 8 bits | Data byte |
| STOP | 1 bit | Stop condition (SDA rises while SCL high) |

#### Timing
- **Clock**: Generated by master
- **Setup/Hold times**: Defined by I2C specification (e.g., 100ns for Standard mode)
- **Timeout**: Bus timeout detection (optional)

#### Error Detection / Recovery
- **ACK/NACK**: Slave acknowledges each byte
- **Bus error detection**: START/STOP condition monitoring
- **Arbitration loss**: Masters detect when another master drives bus

#### Addressing
- **7-bit addressing**: Up to 127 devices
- **10-bit addressing**: Up to 1023 devices
- **General call address**: 0x00 (broadcast)

#### Example Communication Sequence
1. Master sends START
2. Master sends address + R/W bit
3. Slave sends ACK
4. Master sends/receives data bytes
5. Master sends STOP

#### C Example
```c
// Pseudo-code for I2C read
uint8_t i2c_read_reg(uint8_t dev_addr, uint8_t reg_addr) {
    i2c_start();
    i2c_write(dev_addr << 1);  // Write
    i2c_write(reg_addr);
    i2c_start();
    i2c_write((dev_addr << 1) | 1);  // Read
    uint8_t data = i2c_read_nack();
    i2c_stop();
    return data;
}
```

#### C++ Example
```cpp
class I2cDevice {
public:
  I2cDevice(uint8_t addr, I2cBus& bus) : addr_(addr), bus_(bus) {}

  uint8_t readReg(uint8_t reg) {
    bus_.start();
    bus_.write(addr_ << 1);
    bus_.write(reg);
    bus_.start();
    bus_.write((addr_ << 1) | 1);
    uint8_t data = bus_.readNack();
    bus_.stop();
    return data;
  }

private:
  uint8_t addr_;
  I2cBus& bus_;
};
```

#### Embedded Driver Architecture
```
Application -> I2C API -> HAL -> I2C Controller -> SDA/SCL -> Devices
```

#### Interrupt / DMA
- IRQs for transfer complete, arbitration lost, bus error
- DMA for large transfers (optional)

#### RTOS Usage
- ISR for completion; task waits on semaphore; mutex for bus access

#### Embedded Linux
- I2C subsystem; /dev/i2c-*, ioctl interface
- Device tree bindings for I2C devices

#### Tools

| Tool | Purpose |
|---|---|
| Logic Analyzer | Decode I2C transactions |
| Oscilloscope | View SDA/SCL waveforms |
| I2C scanner | Discover devices on bus |

#### Debugging Checklist
- [ ] Check pull-up resistors
- [ ] Verify device addresses
- [ ] Check clock stretching
- [ ] Verify bus capacitance
- [ ] Capture and decode transactions

#### Common Mistakes
- Missing pull-up resistors
- Wrong device address
- Exceeding bus capacitance
- Not handling clock stretching

#### Security Considerations
- No inherent security; bus can be monitored
- Physical access allows device impersonation

#### Advantages
- Simple, low pin count
- Multi-device support
- Multi-master capability

#### Disadvantages
- Limited speed
- Limited distance
- No inherent security

#### When to Use
- Connecting multiple low-speed sensors
- Limited GPIO availability

#### When NOT to Use
- High-speed data transfer
- Long-distance communication
- Noisy environments without isolation

#### Similar Technologies
- [I3C](#35-i3c), [SPI](#36-spi)

#### Related Technologies
- [I3C](#35-i3c), [SMBus](#320-smbus)

#### Learning Priority
- **Beginner**

---

### 3.4 I3C

#### Overview
- **What it is**: Improved Inter-Integrated Circuit; enhanced version of I2C with higher speed and additional features
- **Why it exists**: Overcome I2C limitations while maintaining backward compatibility
- **Where it used**: Modern sensors, mobile devices, IoT applications

#### Classification

| Property | Value |
|---|---|
| Category | Serial bus |
| OSI Layer | Data-Link |
| Communication Type | Synchronous serial |
| Physical Medium | Two-wire (SDA, SCL) |
| Topology | Multi-master, multi-slave |
| Duplex | Full-duplex (in some modes) |
| Synchronization | Clock (SCL) |
| Addressing | Dynamic address assignment |
| Typical Speed | 12.5 Mbit/s (Standard), 33 Mbit/s (High-Speed) |
| Maximum/Typical Distance | Short (similar to I2C) |
| Deterministic | Partial (improved over I2C) |
| Multi-device Support | Yes (up to 50 devices) |

#### Architecture
```
Application
    ↓
I3C Driver
    ↓
I3C Controller
    ↓
SDA/SCL lines
    ↓
Multiple I3C/I2C devices
```

#### Physical Layer
- **Signaling**: Push-pull (faster than I2C open-drain)
- **Wires**: SDA, SCL, GND
- **Voltage levels**: 1.2V, 1.8V, 3.3V
- **Backward compatibility**: I2C devices supported

#### Communication Model
- **Master**: Initiates transactions, generates clock
- **Slave**: Responds to address
- **Arbitration**: Improved over I2C
- **Synchronization**: Clock provided by master

#### Frame / Packet / Message Format

| Field | Size | Description |
|---|---:|---|
| START | 1 bit | Start condition |
| Address | 7 bits | Device address |
| R/W | 1 bit | Read/Write |
| Parity | 1 bit | Parity bit |
| Data | 8 bits | Data byte |
| CRC | 8 bits | CRC for command/data |
| STOP | 1 bit | Stop condition |

#### Timing
- **Clock**: Push-pull for higher speed
- **Setup/Hold**: Tighter than I2C
- **Timeout**: Bus timeout detection

#### Error Detection / Recovery
- **CRC**: Error detection for commands and data
- **Parity**: Single-bit error detection
- **ACK/NACK**: Acknowledge mechanism

#### Addressing
- **Dynamic address assignment**: Master assigns addresses at boot
- **I2C compatibility**: Static I2C addresses supported

#### Example Communication Sequence
1. Master sends START
2. Master sends address + R/W + parity
3. Slave sends ACK
4. Data transfer with CRC
5. Master sends STOP

#### C Example
```c
// Pseudo-code for I3C read
uint8_t i3c_read_reg(uint8_t dev_addr, uint8_t reg_addr) {
    i3c_start();
    i3c_write_command(dev_addr, reg_addr);
    i3c_read_data(&data, 1);
    i3c_stop();
    return data;
}
```

#### C++ Example
```cpp
class I3cDevice {
public:
  I3cDevice(uint8_t addr, I3cBus& bus) : addr_(addr), bus_(bus) {}

  uint8_t readReg(uint8_t reg) {
    bus_.start();
    bus_.writeCommand(addr_, reg);
    uint8_t data = bus_.readData();
    bus_.stop();
    return data;
  }

private:
  uint8_t addr_;
  I3cBus& bus_;
};
```

#### Embedded Driver Architecture
```
Application -> I3C API -> HAL -> I3C Controller -> SDA/SCL -> Devices
```

#### Interrupt / DMA
- IRQs for transfer complete, error conditions
- DMA for large transfers

#### RTOS Usage
- ISR for completion; task waits on semaphore

#### Embedded Linux
- I3C subsystem (growing support)
- Device tree bindings

#### Tools

| Tool | Purpose |
|---|---|
| Logic Analyzer | Decode I3C transactions |
| Oscilloscope | View SDA/SCL waveforms |
| I3C scanner | Discover devices on bus |

#### Debugging Checklist
- [ ] Check pull-up/pull-down configuration
- [ ] Verify device addresses
- [ ] Check clock frequency
- [ ] Verify CRC calculations

#### Common Mistakes
- Incorrect pull-up/pull-down configuration
- Not handling dynamic address assignment
- Mixing I2C and I3C devices incorrectly

#### Security Considerations
- No inherent security; bus can be monitored
- Dynamic addressing provides some protection

#### Advantages
- Higher speed than I2C
- Lower power consumption
- Full-duplex capability
- Backward compatible with I2C

#### Disadvantages
- More complex than I2C
- Limited device support
- Higher cost

#### When to Use
- High-speed sensor networks
- Mobile and IoT applications
- When I2C speed is insufficient

#### When NOT to Use
- Simple low-speed applications
- Cost-sensitive designs
- Legacy I2C-only systems

#### Similar Technologies
- [I2C](#34-i2c), [SPI](#36-spi)

#### Related Technologies
- [I2C](#34-i2c), [SMBus](#320-smbus)

#### Learning Priority
- **Advanced**

---

### 3.5 SPI

#### Overview
- **What it is**: Synchronous serial peripheral interface with full-duplex communication
- **Why it exists**: High-speed, simple point-to-point communication
- **Where it used**: Flash memory, displays, sensors, DACs, ADCs

#### Classification

| Property | Value |
|---|---|
| Category | Serial bus |
| OSI Layer | Data-Link |
| Communication Type | Synchronous serial |
| Physical Medium | 4+ wires (MOSI, MISO, SCLK, CS) |
| Topology | Point-to-point or multi-slave |
| Duplex | Full-duplex |
| Synchronization | Clock (SCLK) |
| Addressing | Chip select (CS) lines |
| Typical Speed | 1–50 Mbit/s common; higher in specialized PHYs |
| Maximum/Typical Distance | Short (cm to ~1m) |
| Deterministic | No (master-controlled) |
| Multi-device Support | Yes (via chip select) |

#### Architecture
```
Application
    ↓
SPI Driver
    ↓
SPI Controller
    ↓
MOSI/MISO/SCLK/CS
    ↓
Slave devices
```

ASCII:
```
        MOSI ----+----+----+
        MISO ----+----+----+
        SCLK ----+----+----+
         CS1 -----+    |    |
         CS2 ----------+    |
         CS3 ---------------+
        MCU   Dev1  Dev2  Dev3
```

#### Physical Layer
- **Signaling**: Push-pull
- **Wires**: MOSI (Master Out Slave In), MISO (Master In Slave Out), SCLK (clock), CS (chip select), GND
- **Voltage levels**: Typically 3.3V or 5V
- **Termination**: Not required for short distances

#### Communication Model
- **Master**: Generates clock, controls chip select
- **Slave**: Responds when CS is active
- **Arbitration**: None (master-controlled)
- **Synchronization**: Clock provided by master

#### Frame / Packet / Message Format

| Field | Size | Description |
|---|---:|---|
| CS Active | N/A | Chip select goes low |
| Data | 8/16/32 bits | Data transferred MSB or LSB first |
| CS Inactive | N/A | Chip select goes high |

#### Timing
- **Clock**: Generated by master
- **Modes**: 4 SPI modes (CPOL, CPHA combinations)
- **Setup/Hold**: Defined by device specifications

#### Error Detection / Recovery
- None inherent; higher-layer protocols provide CRC/ACK

#### Addressing
- Chip select lines (one per slave)

#### Example Communication Sequence
1. Master activates CS
2. Master sends clock and data on MOSI
3. Slave sends data on MISO
4. Master deactivates CS

#### C Example
```c
// Pseudo-code for SPI write
void spi_write(uint8_t cs, const uint8_t* data, size_t len) {
    gpio_write(cs, 0);  // Activate CS
    for (size_t i = 0; i < len; i++) {
        spi_transfer(data[i]);
    }
    gpio_write(cs, 1);  // Deactivate CS
}
```

#### C++ Example
```cpp
class SpiDevice {
public:
  SpiDevice(GpioPin& cs, SpiBus& bus) : cs_(cs), bus_(bus) {}

  void write(const std::vector<uint8_t>& data) {
    cs_.set(false);
    bus_.transfer(data);
    cs_.set(true);
  }

private:
  GpioPin& cs_;
  SpiBus& bus_;
};
```

#### Embedded Driver Architecture
```
Application -> SPI API -> HAL -> SPI Controller -> MOSI/MISO/SCLK/CS -> Devices
```

#### Interrupt / DMA
- IRQs for transfer complete
- DMA for large transfers

#### RTOS Usage
- ISR for completion; task waits on semaphore

#### Embedded Linux
- SPI subsystem; /dev/spidev*, ioctl interface
- Device tree bindings

#### Tools

| Tool | Purpose |
|---|---|
| Logic Analyzer | Decode SPI transactions |
| Oscilloscope | View waveforms |
| SPI flasher | Program SPI flash |

#### Debugging Checklist
- [ ] Check chip select polarity
- [ ] Verify SPI mode (CPOL, CPHA)
- [ ] Check clock frequency
- [ ] Verify data order (MSB/LSB first)

#### Common Mistakes
- Wrong SPI mode
- Incorrect chip select polarity
- Clock frequency too high
- Data order mismatch

#### Security Considerations
- No inherent security; bus can be monitored
- Physical access allows device impersonation

#### Advantages
- High speed
- Simple protocol
- Full-duplex
- No addressing overhead

#### Disadvantages
- Requires multiple CS lines for multiple devices
- No inherent error detection
- Limited distance

#### When to Use
- High-speed data transfer
- Point-to-point communication
- Flash memory, displays

#### When NOT to Use
- Multi-device networks without CS expansion
- Long-distance communication
- Noisy environments

#### Similar Technologies
- [QSPI](#37-qspi), [I2C](#34-i2c), [UART](#32-uart)

#### Related Technologies
- [QSPI](#37-qspi), [OSPI](#38-ospi)

#### Learning Priority
- **Beginner**

---

### 3.6 QSPI

#### Overview
- **What it is**: Quad SPI; SPI with 4 data lines for higher throughput
- **Why it exists**: Faster flash memory access for code execution (XIP)
- **Where it used**: NOR flash, embedded storage

#### Classification

| Property | Value |
|---|---|
| Category | Serial bus |
| OSI Layer | Data-Link |
| Communication Type | Synchronous serial |
| Physical Medium | 6+ wires (4 data, clock, CS) |
| Topology | Point-to-point |
| Duplex | Full-duplex (in some modes) |
| Synchronization | Clock |
| Addressing | Chip select |
| Typical Speed | 50–200+ Mbit/s |
| Maximum/Typical Distance | Short (on-board) |
| Deterministic | No |
| Multi-device Support | Limited |

#### Architecture
```
Application
    ↓
QSPI Driver
    ↓
QSPI Controller
    ↓
IO0-IO3, CLK, CS
    ↓
QSPI Flash
```

#### Physical Layer
- **Signaling**: Push-pull
- **Wires**: IO0-IO3 (data), CLK, CS, GND
- **Voltage levels**: Typically 3.3V or 1.8V
- **Termination**: May require termination for high speeds

#### Communication Model
- **Master**: MCU/QSPI controller
- **Slave**: QSPI flash device
- **Arbitration**: None
- **Synchronization**: Clock

#### Frame / Packet / Message Format

| Field | Size | Description |
|---|---:|---|
| Command | 8 bits | Read/Write command |
| Address | 24/32 bits | Memory address |
| Data | Variable | Data payload |
| Dummy | Variable | Dummy cycles |

#### Timing
- **Clock**: Up to 100+ MHz
- **XIP mode**: Execute-in-place from flash

#### Error Detection / Recovery
- CRC (device-dependent)
- ECC (in flash controller)

#### Addressing
- Memory-mapped or command-based

#### Example Communication Sequence
1. Activate CS
2. Send command
3. Send address
4. Transfer data (4 bits per clock)
5. Deactivate CS

#### C Example
```c
// Pseudo-code for QSPI read
void qspi_read(uint32_t addr, uint8_t* data, size_t len) {
    qspi_set_mode(QSPI_MODE_READ);
    qspi_set_address(addr);
    qspi_transfer(data, len);
}
```

#### C++ Example
```cpp
class QspiFlash {
public:
  QspiFlash(QspiController& ctrl) : ctrl_(ctrl) {}

  void read(uint32_t addr, uint8_t* data, size_t len) {
    ctrl_.setMode(QspiMode::Read);
    ctrl_.setAddress(addr);
    ctrl_.transfer(data, len);
  }

private:
  QspiController& ctrl_;
};
```

#### Embedded Driver Architecture
```
Application -> QSPI API -> HAL -> QSPI Controller -> Flash
```

#### Interrupt / DMA
- DMA for large transfers
- IRQs for completion

#### RTOS Usage
- ISR for completion; task waits on semaphore

#### Embedded Linux
- SPI NOR flash subsystem
- MTD (Memory Technology Device) layer

#### Tools

| Tool | Purpose |
|---|---|
| Logic Analyzer | Decode QSPI transactions |
| Oscilloscope | View waveforms |
| Flash programmer | Program QSPI flash |

#### Debugging Checklist
- [ ] Check clock frequency
- [ ] Verify command sequences
- [ ] Check XIP configuration
- [ ] Verify termination

#### Common Mistakes
- Wrong command sequences
- Clock frequency too high
- Incorrect XIP configuration

#### Security Considerations
- No inherent security
- Flash can be read/modified with physical access

#### Advantages
- High speed
- XIP capability
- Simple interface

#### Disadvantages
- Limited to point-to-point
- No inherent error detection
- Requires careful timing

#### When to Use
- Code storage with XIP
- High-speed flash access

#### When NOT to Use
- Multi-device networks
- Low-speed applications

#### Similar Technologies
- [OSPI](#38-ospi), [SPI](#36-spi)

#### Related Technologies
- [OSPI](#38-ospi), [SPI NOR Flash](#350-spi-nor-flash)

#### Learning Priority
- **Intermediate**

---

### 3.7 OSPI

#### Overview
- **What it is**: Octal SPI; SPI with 8 data lines for very high throughput
- **Why it exists**: Maximum flash performance for modern embedded systems
- **Where it used**: High-speed NOR/NAND flash, embedded storage

#### Classification

| Property | Value |
|---|---|
| Category | Serial bus |
| OSI Layer | Data-Link |
| Communication Type | Synchronous serial |
| Physical Medium | 10+ wires (8 data, clock, CS) |
| Topology | Point-to-point |
| Duplex | Full-duplex |
| Synchronization | Clock |
| Addressing | Chip select |
| Typical Speed | 100–400+ Mbit/s |
| Maximum/Typical Distance | Short (on-board) |
| Deterministic | No |
| Multi-device Support | Limited |

#### Architecture
```
Application
    ↓
OSPI Driver
    ↓
OSPI Controller
    ↓
IO0-IO7, CLK, CS
    ↓
OSPI Flash
```

#### Physical Layer
- **Signaling**: Push-pull, differential (in some implementations)
- **Wires**: IO0-IO7 (data), CLK, CS, GND
- **Voltage levels**: Typically 1.8V or 3.3V
- **Termination**: Required for high speeds

#### Communication Model
- **Master**: MCU/OSPI controller
- **Slave**: OSPI flash device
- **Arbitration**: None
- **Synchronization**: Clock

#### Frame / Packet / Message Format

| Field | Size | Description |
|---|---:|---|
| Command | 8 bits | Read/Write command |
| Address | 24/32 bits | Memory address |
| Data | Variable | Data payload (8 bits per clock) |
| Dummy | Variable | Dummy cycles |

#### Timing
- **Clock**: Up to 200+ MHz
- **XIP mode**: Execute-in-place from flash

#### Error Detection / Recovery
- CRC (device-dependent)
- ECC (in flash controller)

#### Addressing
- Memory-mapped or command-based

#### Example Communication Sequence
1. Activate CS
2. Send command
3. Send address
4. Transfer data (8 bits per clock)
5. Deactivate CS

#### C Example
```c
// Pseudo-code for OSPI read
void ospi_read(uint32_t addr, uint8_t* data, size_t len) {
    ospi_set_mode(OSPI_MODE_READ);
    ospi_set_address(addr);
    ospi_transfer(data, len);
}
```

#### C++ Example
```cpp
class OspiFlash {
public:
  OspiFlash(OspiController& ctrl) : ctrl_(ctrl) {}

  void read(uint32_t addr, uint8_t* data, size_t len) {
    ctrl_.setMode(OspiMode::Read);
    ctrl_.setAddress(addr);
    ctrl_.transfer(data, len);
  }

private:
  OspiController& ctrl_;
};
```

#### Embedded Driver Architecture
```
Application -> OSPI API -> HAL -> OSPI Controller -> Flash
```

#### Interrupt / DMA
- DMA for large transfers
- IRQs for completion

#### RTOS Usage
- ISR for completion; task waits on semaphore

#### Embedded Linux
- SPI NOR flash subsystem
- MTD layer

#### Tools

| Tool | Purpose |
|---|---|
| Logic Analyzer | Decode OSPI transactions |
| Oscilloscope | View waveforms |
| Flash programmer | Program OSPI flash |

#### Debugging Checklist
- [ ] Check clock frequency
- [ ] Verify command sequences
- [ ] Check XIP configuration
- [ ] Verify termination and impedance matching

#### Common Mistakes
- Wrong command sequences
- Clock frequency too high
- Incorrect termination
- Signal integrity issues

#### Security Considerations
- No inherent security
- Flash can be read/modified with physical access

#### Advantages
- Very high speed
- XIP capability
- Efficient bus utilization

#### Disadvantages
- More pins required
- Complex PCB routing
- Signal integrity challenges
- Higher cost

#### When to Use
- Maximum flash performance
- High-speed code execution
- Large embedded storage

#### When NOT to Use
- Cost-sensitive designs
- Simple applications
- Multi-device networks

#### Similar Technologies
- [QSPI](#37-qspi), [SPI](#36-spi)

#### Related Technologies
- [QSPI](#37-qspi), [NAND Flash](#351-nand-flash)

#### Learning Priority
- **Advanced**

---

### 3.8 CAN

#### Overview
- **What it is**: Controller Area Network; robust multi-master serial bus for automotive and industrial
- **Why it exists**: Reliable communication in noisy environments with deterministic behavior
- **Where it used**: Automotive ECUs, industrial automation, medical devices

#### Classification

| Property | Value |
|---|---|
| Category | Fieldbus / Data-Link / Network |
| OSI Layer | Data-Link (with some Network characteristics) |
| Communication Type | Synchronous serial with arbitration |
| Physical Medium | Differential pair (CAN_H, CAN_L) |
| Topology | Multi-master, multi-drop (linear bus) |
| Duplex | Half-duplex |
| Synchronization | Hard synchronization + resynchronization |
| Addressing | Message ID (11-bit or 29-bit) |
| Typical Speed | 1 Mbit/s (CAN 2.0), up to 8 Mbit/s (CAN FD) |
| Maximum/Typical Distance | Up to 40m at 1 Mbit/s; up to 1km at lower speeds |
| Deterministic | Yes (priority-based arbitration) |
| Multi-device Support | Yes (up to 127 nodes theoretically) |

#### Architecture
```
Application (UDS, J1939, CANopen)
    ↓
CAN Driver
    ↓
CAN Controller
    ↓
CAN Transceiver
    ↓
CAN Bus (differential pair)
```

ASCII:
```
        CAN_H ----+----+----+----+
                  |    |    |    |
        CAN_L ----+----+----+----+
                 ECU1 ECU2 ECU3 ECU4
                  |              |
               Term           Term
               120Ω           120Ω
```

#### Physical Layer
- **Signaling**: Differential (CAN_H, CAN_L)
- **Wires**: CAN_H, CAN_L, GND (optional)
- **Voltage levels**: Differential voltage represents logic states
- **Termination**: 120Ω at each end of bus
- **Transceivers**: ISO 11898 compliant

#### Communication Model
- **Nodes**: All nodes are equal (multi-master)
- **Arbitration**: Priority-based (lower ID = higher priority)
- **Synchronization**: Hard sync at start, resync on edges
- **Broadcast**: All nodes receive all messages

#### Frame / Packet / Message Format (CAN 2.0)

| Field | Size | Description |
|---|---:|---|
| Start of Frame | 1 bit | Dominant bit |
| Arbitration Field | 11/29 bits | Message ID |
| Control Field | 4 bits | DLC, RTR, etc. |
| Data Field | 0–8 bytes | Payload |
| CRC Field | 15 bits | CRC + delimiter |
| ACK Field | 2 bits | ACK slot + delimiter |
| End of Frame | 7 bits | Recessive |
| Interframe Space | 3 bits | Separation |

#### Timing
- **Bit time**: Configurable (e.g., 1μs at 1 Mbit/s)
- **Propagation delay**: Bus length dependent
- **Sample point**: Typically 75% of bit time

#### Error Detection / Recovery
- **CRC**: 15-bit CRC
- **ACK**: Acknowledge from receivers
- **Frame check**: Format, stuffing errors
- **Bus-off**: Error counter overflow recovery

#### Addressing
- **Standard ID**: 11-bit (2048 IDs)
- **Extended ID**: 29-bit (536 million IDs)
- **Remote Frame**: Request data from specific ID

#### Example Communication Sequence
1. Node waits for bus idle
2. Node sends Start of Frame
3. Arbitration on ID (lower ID wins)
4. Winner sends data
5. Receivers send ACK
6. All nodes update error counters

#### C Example
```c
// Pseudo-code for CAN send
void can_send(uint32_t id, const uint8_t* data, uint8_t len) {
    can_frame_t frame;
    frame.id = id;
    frame.len = len;
    memcpy(frame.data, data, len);
    can_transmit(&frame);
}
```

#### C++ Example
```cpp
class CanNode {
public:
  CanNode(CanController& ctrl) : ctrl_(ctrl) {}

  void send(uint32_t id, const std::vector<uint8_t>& data) {
    CanFrame frame{id, data};
    ctrl_.transmit(frame);
  }

  void receive(std::function<void(CanFrame)> callback) {
    ctrl_.onReceive(callback);
  }

private:
  CanController& ctrl_;
};
```

#### Embedded Driver Architecture
```
Application -> CAN API -> HAL -> CAN Controller -> Transceiver -> Bus
```

#### Interrupt / DMA
- IRQs for receive, transmit complete, error
- DMA for high-throughput applications

#### RTOS Usage
- ISR for receive; task waits on queue/semaphore
- Priority handling for time-critical messages

#### Embedded Linux
- SocketCAN; /dev/can0, netlink
- ip link set can0 up type can bitrate 500000

#### Tools

| Tool | Purpose |
|---|---|
| Logic Analyzer | Decode CAN frames |
| Oscilloscope | View differential signals |
| CAN analyzer | Monitor bus traffic |
| Wireshark | Capture CAN over USB |

#### Debugging Checklist
- [ ] Check termination (120Ω at each end)
- [ ] Verify wiring (CAN_H, CAN_L)
- [ ] Check baud rate configuration
- [ ] Monitor error counters
- [ ] Check for bus-off state

#### Common Mistakes
- Missing termination resistors
- Wrong baud rate
- Exceeding bus length for speed
- Not handling bus-off recovery
- Incorrect bit timing

#### Security Considerations
- No authentication or encryption
- Any node can send any message
- Physical access allows spoofing
- Consider CAN security extensions

#### Advantages
- Robust in noisy environments
- Deterministic (priority-based)
- Multi-master
- Error detection and recovery
- Long distance at lower speeds

#### Disadvantages
- Limited payload (8 bytes in CAN 2.0)
- No inherent security
- Complex bit timing configuration
- Bus-off recovery required

#### When to Use
- Automotive applications
- Industrial control
- Noisy environments
- Deterministic communication needed

#### When NOT to Use
- High-bandwidth applications
- Secure communication required
- Point-to-point only
- Very long distances at high speed

#### Similar Technologies
- [CAN-FD](#39-can-fd), [LIN](#310-lin), [FlexRay](#311-flexray)

#### Related Technologies
- [CAN-FD](#39-can-fd), [CANopen](#324-canopen), [J1939](#328-j1939), [UDS](#326-uds)

#### Learning Priority
- **Intermediate**

---

## 4. Protocol Stacks

### 4.1 Embedded MCU Stack
```
Application
    ↓
Driver
    ↓
HAL (Hardware Abstraction Layer)
    ↓
Peripheral Register Access
    ↓
Hardware
```

### 4.2 TCP/IP Stack
```
Application (HTTP, MQTT, DNS)
    ↓
Transport (TCP, UDP)
    ↓
Network (IPv4, IPv6)
    ↓
Data-Link (Ethernet MAC, Wi-Fi)
    ↓
Physical (PHY, RF)
```

### 4.3 CAN Stack
```
Application (UDS, J1939, CANopen)
    ↓
CAN (CAN 2.0, CAN FD)
    ↓
CAN Controller
    ↓
CAN Transceiver
    ↓
Physical Bus (differential pair)
```

### 4.4 USB Stack
```
Application
    ↓
USB Class (HID, CDC, MSC)
    ↓
USB Device Stack
    ↓
USB Controller Driver
    ↓
PHY
    ↓
USB Cable
```

### 4.5 Linux Driver Stack
```
Application
    ↓
POSIX API / Socket API
    ↓
Kernel Subsystem (tty, net, i2c, spi)
    ↓
Kernel Driver
    ↓
Hardware
```

---

## 5. Comparison Tables

### 5.1 UART vs SPI vs I2C vs I3C

| Feature | UART | SPI | I2C | I3C |
|---|---|---|---|---|
| Wires | 2–3 (TX, RX, GND) | 4+ (MOSI, MISO, SCLK, CS) | 2 (SDA, SCL) | 2 (SDA, SCL) |
| Speed | Up to Mbps | 1–50+ Mbit/s | 100 k–3.4 Mbit/s | 12.5–33 Mbit/s |
| Addressing | None | Chip select | 7/10-bit address | Dynamic assignment |
| Duplex | Full/Half | Full | Half | Full (some modes) |
| Topology | Point-to-point | Point-to-point/multi-slave | Multi-master multi-slave | Multi-master |
| Distance | Short–Medium | Short | Short | Short |
| Controller count | 1 | 1 | 1–N | 1–N |
| Peripherals | Serial devices | Flash, displays, sensors | Sensors, EEPROMs | Modern sensors |
| DMA | Optional | Yes | Optional | Optional |
| Interrupt support | Yes | Yes | Yes | Yes |
| Complexity | Low | Medium | Medium | Medium-High |
| Common applications | Console, debug | High-speed peripherals | Sensor buses | High-speed sensors |

### 5.2 RS-232 vs RS-422 vs RS-485

| Feature | RS-232 | RS-422 | RS-485 |
|---|---|---|---|
| Signaling | Single-ended | Differential | Differential |
| Topology | Point-to-point | Multi-drop | Multi-drop/multi-point |
| Distance | Short (≤15m) | Medium (≤1200m) | Medium–Long (≤1200m) |
| Speed | Up to ~1 Mbit/s | 100 k–10 Mbit/s | 1 k–20 Mbit/s |
| Termination | Not required | Required for long runs | Required for long runs |
| Noise immunity | Low | High | Very high |
| Drivers | 1 | 1 | Multiple |
| Receivers | 1 | 1 | Multiple (up to 32) |

### 5.3 CAN vs CAN-FD vs LIN vs FlexRay

| Feature | CAN | CAN-FD | LIN | FlexRay |
|---|---|---|---|---|
| Speed | 1 Mbit/s max | Up to 8 Mbit/s | 20–33.3 kbit/s | 10–40 Mbit/s |
| Payload | 8 bytes | Up to 64 bytes | 8 bytes | 254 bytes |
| Topology | Multi-drop | Multi-drop | Master/slave | Multi-drop |
| Determinism | Priority-based | Priority-based | Scheduled | Time-triggered |
| Cost | Low | Low-Medium | Very low | High |
| Use case | General automotive | High-speed automotive | Body electronics | Safety-critical |

### 5.4 TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable (ACK, retransmit) | Unreliable |
| Ordering | In-order delivery | No ordering guarantee |
| Flow control | Yes | No |
| Congestion control | Yes | No |
| Overhead | High | Low |
| Use case | Web, file transfer | Streaming, real-time |

### 5.5 MQTT vs CoAP vs HTTP

| Feature | MQTT | CoAP | HTTP |
|---|---|---|---|
| Transport | TCP/TLS | UDP/TCP | TCP/TLS |
| Model | Pub/Sub | Client/Server | Client/Server |
| Overhead | Low | Very low | High |
| QoS | 0, 1, 2 | Confirmable/Non-confirmable | Request/Response |
| Use case | IoT messaging | Constrained IoT | Web services |

### 5.6 BLE vs Wi-Fi vs Zigbee vs Thread vs LoRaWAN

| Feature | BLE | Wi-Fi | Zigbee | Thread | LoRaWAN |
|---|---|---|---|---|---|
| Range | Short (10–100m) | Medium (50–100m) | Medium (10–100m) | Medium (10–100m) | Long (km) |
| Power | Very low | High | Low | Low | Very low |
| Data rate | 125 k–2 Mbps | 11–600 Mbps | 250 kbit/s | 250 kbit/s | 0.3–50 kbit/s |
| Topology | Star | Star | Mesh | Mesh | Star |
| IP support | No | Yes | Optional | Yes | No |
| Use case | Wearables, sensors | High-speed data | Home automation | Smart home | Remote sensors |

---

## 6. Physical Layer Reference

### 6.1 Signaling Types

| Type | Description | Examples |
|------|-------------|----------|
| **Single-ended** | Signal referenced to ground | UART, RS-232, GPIO |
| **Differential** | Signal is voltage difference between two wires | RS-485, CAN, Ethernet, LVDS |

### 6.2 Why Differential Signaling?
- **Noise immunity**: Common-mode noise cancels out
- **Longer distance**: Better signal integrity
- **EMI reduction**: Reduced electromagnetic interference

### 6.3 Termination
- **Purpose**: Prevent signal reflections
- **Types**: Parallel, series, Thevenin
- **When needed**: High-speed, long-distance

### 6.4 Impedance
- **Characteristic impedance**: Typically 50Ω, 75Ω, 100Ω, 120Ω
- **Matching**: Prevent reflections
- **Controlled impedance**: PCB trace design

### 6.5 Pull-up / Pull-down Resistors
- **Pull-up**: Resistor to VDD (I2C, open-drain)
- **Pull-down**: Resistor to GND
- **Purpose**: Define default state

### 6.6 Open-Drain vs Push-Pull
| Type | Description | Examples |
|------|-------------|----------|
| **Open-drain** | Can pull low or float; requires pull-up | I2C, 1-Wire |
| **Push-pull** | Can drive high or low | SPI, GPIO, UART |

### 6.7 Common-Mode Voltage
- **Definition**: Average voltage on differential pair
- **Range**: Specified by standard (e.g., RS-485: -7V to +12V)

### 6.8 Noise, EMI, Crosstalk
- **Noise**: Unwanted electrical interference
- **EMI**: Electromagnetic interference
- **Crosstalk**: Coupling between adjacent traces

### 6.9 Ground Loops
- **Problem**: Multiple ground paths cause current flow
- **Solution**: Single-point grounding, isolation

### 6.10 Isolation
- **Galvanic isolation**: No direct electrical connection
- **Methods**: Optocouplers, transformers, capacitive isolation
- **Purpose**: Safety, noise immunity, ground loop prevention

### 6.11 Transceivers
- **Purpose**: Convert logic levels to bus levels
- **Examples**: RS-485 transceiver, CAN transceiver, Ethernet PHY

---

## 7. Serial Communication Fundamentals

### 7.1 Baud Rate vs Bit Rate
- **Baud rate**: Symbols per second
- **Bit rate**: Bits per second
- **Relationship**: Bit rate = Baud rate × bits per symbol

### 7.2 Start/Stop Bits
- **Start bit**: Indicates beginning of frame (typically 1 bit, low)
- **Stop bit(s): Indicates end of frame (typically 1–2 bits, high)

### 7.3 Parity
- **Purpose**: Single-bit error detection
- **Types**: None, Even, Odd, Mark, Space
- **Calculation**: XOR of data bits

### 7.4 Framing
- **Frame**: Complete unit of transmission
- **Components**: Start, data, parity, stop

### 7.5 Clock Recovery
- **Synchronous**: Clock provided separately
- **Asynchronous**: Clock recovered from data edges

### 7.6 Synchronous vs Asynchronous
| Feature | Synchronous | Asynchronous |
|---|---|---|
| Clock | Separate line | Embedded in data |
| Speed | Higher | Lower |
| Complexity | Lower | Higher |
| Examples | SPI, I2C | UART |

### 7.7 Flow Control
- **Hardware**: RTS/CTS signals
- **Software**: XON/XOFF characters
- **Purpose**: Prevent buffer overflow

### 7.8 Buffering
- **FIFO**: First-In-First-Out buffer
- **Ring buffer**: Circular buffer for continuous data
- **DMA buffer**: Direct memory access for large transfers

### 7.9 DMA (Direct Memory Access)
- **Purpose**: Transfer data without CPU involvement
- **Benefits**: Lower CPU load, higher throughput
- **Types**: Single transfer, block transfer, circular DMA

---

## 8. Embedded Driver Design

### 8.1 Driver Architecture Layers
```
Application
    ↓
Protocol/Application Layer
    ↓
Driver (HAL/LL)
    ↓
Peripheral Register Access
    ↓
Hardware
```

### 8.2 Register-Level Programming
- **Memory-mapped I/O**: Direct register access
- **Volatile**: Prevent compiler optimization
- **Bit manipulation**: Set, clear, toggle bits

### 8.3 HAL vs LL vs Vendor SDK
| Layer | Description | Example |
|---|---|---|
| **HAL** | Hardware Abstraction Layer | STM32 HAL |
| **LL** | Low-Layer (closer to hardware) | STM32 LL |
| **Vendor SDK** | Vendor-provided libraries | ESP-IDF, Nordic SDK |

### 8.4 Interrupt Handlers
```c
void UART_IRQHandler(void) {
    if (UART_SR & UART_SR_RXNE) {
        uint8_t data = UART_DR;
        ring_buffer_push(&rx_buf, data);
    }
    if (UART_SR & UART_SR_TXE) {
        if (ring_buffer_pop(&tx_buf, &data)) {
            UART_DR = data;
        } else {
            UART_CR &= ~UART_CR_TXEIE;  // Disable TX interrupt
        }
    }
}
```

### 8.5 DMA Usage
```c
// Circular DMA for continuous reception
DMA_Channel->CCR |= DMA_CCR_CIRC;
DMA_Channel->CNDTR = BUFFER_SIZE;
DMA_Channel->CCR |= DMA_CCR_EN;
```

### 8.6 Ring Buffers
```c
typedef struct {
    uint8_t* buffer;
    size_t size;
    volatile size_t head;
    volatile size_t tail;
} ring_buffer_t;

void ring_buffer_push(ring_buffer_t* rb, uint8_t data) {
    rb->buffer[rb->head] = data;
    rb->head = (rb->head + 1) % rb->size;
}

bool ring_buffer_pop(ring_buffer_t* rb, uint8_t* data) {
    if (rb->head == rb->tail) return false;
    *data = rb->buffer[rb->tail];
    rb->tail = (rb->tail + 1) % rb->size;
    return true;
}
```

### 8.7 State Machines
```c
typedef enum {
    STATE_IDLE,
    STATE_WAITING_FOR_HEADER,
    STATE_RECEIVING_DATA,
    STATE_WAITING_FOR_CRC
} uart_state_t;

uart_state_t state = STATE_IDLE;
```

### 8.8 Timeout Handling
```c
uint32_t start = millis();
while (!condition) {
    if (millis() - start > TIMEOUT_MS) {
        return ERROR_TIMEOUT;
    }
}
```

### 8.9 Concurrency and Thread Safety
- **Critical sections**: Disable interrupts
- **Mutexes**: RTOS mutex for resource protection
- **Atomic operations**: Hardware-supported atomic access

### 8.10 Memory Barriers
- **Purpose**: Ensure memory operation ordering
- **When needed**: DMA, multi-core, interrupt contexts

### 8.11 Cache Coherency
- **Problem**: Cache and memory out of sync
- **Solution**: Cache flush/invalidate before/after DMA

### 8.12 Memory-Mapped I/O
```c
#define UART_BASE  0x40013800
#define UART_DR    (*(volatile uint32_t*)(UART_BASE + 0x00))
#define UART_SR    (*(volatile uint32_t*)(UART_BASE + 0x04))
```

---

## 9. Linux Device Driver View

### 9.1 Linux Subsystems

| Hardware | Linux Subsystem | Typical Userspace Interface |
|---|---|---|
| UART | TTY subsystem | /dev/ttyS*, /dev/ttyAMA* |
| I2C | I2C subsystem | /dev/i2c-*, ioctl |
| SPI | SPI subsystem | /dev/spidev*, ioctl |
| CAN | CAN subsystem | can0, SocketCAN |
| USB | USB subsystem | /dev/bus/usb-*, libusb |
| Ethernet | Networking stack | sockets, netlink |
| GPIO | GPIO subsystem | /dev/gpiochip*, sysfs |
| PCIe | PCIe subsystem | /dev, sysfs |
| Video | V4L2 | /dev/video* |
| Audio | ALSA | /dev/snd/* |
| Display | DRM | /dev/dri/* |

### 9.2 Device Tree
```dts
&i2c1 {
    status = "okay";
    sensor@68 {
        compatible = "vendor,sensor";
        reg = <0x68>;
    };
};
```

### 9.3 Character Devices
```c
static struct file_operations fops = {
    .open = device_open,
    .read = device_read,
    .write = device_write,
    .ioctl = device_ioctl,
};
```

### 9.4 ioctl Interface
```c
#define IOCTL_READ  _IOR(MAJOR, 0, int)
#define IOCTL_WRITE _IOW(MAJOR, 1, int)

long device_ioctl(struct file *filp, unsigned int cmd, unsigned long arg) {
    switch (cmd) {
        case IOCTL_READ:
            // Handle read
            break;
        case IOCTL_WRITE:
            // Handle write
            break;
    }
}
```

### 9.5 poll/select/epoll
- **poll**: Check if device is ready for I/O
- **select**: Monitor multiple file descriptors
- **epoll**: Scalable I/O event notification

### 9.6 DMA in Linux
- **DMA coherent**: CPU-cache coherent memory
- **DMA streaming**: One-way transfers
- **DMA mapping**: Map buffers for DMA

### 9.7 Interrupts in Linux
```c
static irqreturn_t device_irq(int irq, void *dev_id) {
    // Handle interrupt
    return IRQ_HANDLED;
}

request_irq(irq, device_irq, IRQF_SHARED, "device_name", dev_id);
```

---

## 10. RTOS View

### 10.1 RTOS Concepts
- **Task/Thread**: Independent execution unit
- **ISR**: Interrupt Service Routine
- **Queue**: FIFO for inter-task communication
- **Semaphore**: Counting synchronization
- **Mutex**: Mutual exclusion
- **Event Group**: Multiple event flags
- **Notification**: Task-to-task notification

### 10.2 Producer-Consumer Architecture
```
[Producer Task] --> [Queue] --> [Consumer Task]
       |                           |
    ISR (RX)                   Application
```

### 10.3 Priority and Inversion
- **Priority**: Task execution order
- **Priority inversion**: Low-priority task blocks high-priority
- **Solution**: Mutex with priority inheritance

### 10.4 DMA with RTOS
- **ISR**: DMA complete interrupt
- **Semaphore**: Signal task on completion
- **Task**: Process received data

### 10.5 Example: UART Driver with RTOS
```c
QueueHandle_t uart_queue;

void UART_IRQHandler(void) {
    if (UART_SR & UART_SR_RXNE) {
        uint8_t data = UART_DR;
        xQueueSendFromISR(uart_queue, &data, NULL);
    }
}

void uart_task(void *pvParameters) {
    uint8_t data;
    while (1) {
        if (xQueueReceive(uart_queue, &data, portMAX_DELAY)) {
            process_data(data);
        }
    }
}
```

---

## 11. Packet / Frame Analysis

### 11.1 Identifying Frame Structure
Given a byte stream:
```
AA 55 10 03 01 02 7F 4C
```

### 11.2 Common Patterns
| Pattern | Likely Meaning |
|---|---|
| **AA 55** | Synchronization bytes |
| **10** | Length (16 bytes) |
| **03** | Command/Message type |
| **01 02** | Address/Sequence |
| **7F 4C** | CRC/Checksum |

### 11.3 Analysis Steps
1. **Identify sync pattern**: Look for repeated sequences
2. **Find length field**: Correlate with actual frame size
3. **Identify command**: Look for consistent values
4. **Locate address**: Device/node identifier
5. **Extract payload**: Variable data
6. **Verify checksum**: CRC, XOR, sum

### 11.4 Reverse Engineering Tips
- **Capture multiple frames**: Identify patterns
- **Vary inputs**: Observe changes in output
- **Check documentation**: Look for protocol specs
- **Use protocol analyzer**: Decode known protocols
- **Legal considerations**: Only analyze devices you own or have permission to test

---

## 12. Debugging Tools

| Tool | Protocols | Purpose | Hardware/Software |
|---|---|---|---|
| **Logic Analyzer** | UART, SPI, I2C, CAN, etc. | Decode digital signals | Hardware |
| **Oscilloscope** | All | View analog waveforms | Hardware |
| **Protocol Analyzer** | USB, CAN, Ethernet, etc. | Deep protocol decode | Hardware/Software |
| **Wireshark** | Ethernet, TCP/IP, MQTT, HTTP | Packet capture and analysis | Software |
| **USBPcap** | USB | USB packet capture | Software |
| **usbmon** | USB | Linux USB monitoring | Software |
| **can-utils** | CAN | CAN frame utilities | Software |
| **SocketCAN tools** | CAN | Linux CAN tools | Software |
| **sigrok / PulseView** | Multiple | Multi-protocol capture | Software/Hardware |
| **Saleae Logic** | Digital | Logic analyzer | Hardware |
| **minicom / picocom** | UART | Serial terminal | Software |
| **screen** | UART | Serial terminal | Software |
| **stty** | UART | Serial port configuration | Software |
| **iproute2** | Ethernet | Network configuration | Software |
| **ethtool** | Ethernet | Ethernet diagnostics | Software |
| **tcpdump** | Ethernet | Packet capture | Software |
| **Vendor analyzers** | Proprietary | Vendor-specific protocols | Hardware/Software |

---

## 13. Common Embedded Protocol Debugging Workflow

1. **Understand requirements**: What should the protocol do?
2. **Identify physical layer**: What wires/signals are involved?
3. **Identify electrical characteristics**: Voltage, termination, impedance
4. **Identify protocol**: What protocol is being used?
5. **Identify frame structure**: Header, payload, checksum
6. **Verify wiring**: Check connections, continuity
7. **Verify power**: Check voltage levels
8. **Verify clock/baud**: Check timing configuration
9. **Capture traffic**: Use logic analyzer, oscilloscope
10. **Decode traffic**: Analyze frames
11. **Check errors**: CRC, ACK, timeouts
12. **Check driver**: Register configuration, ISR
13. **Check ISR/DMA**: Interrupt handling, DMA configuration
14. **Check application layer**: Protocol logic, state machine
15. **Stress test**: High load, edge cases
16. **Fault injection**: Simulate errors
17. **Long-duration test**: Endurance testing

---

## 14. Testing

### 14.1 Unit Testing
- Test individual functions
- Mock hardware interfaces
- Verify logic

### 14.2 Integration Testing
- Test subsystems together
- Verify interfaces
- Check timing

### 14.3 Hardware-in-the-Loop (HIL)
- Test with real hardware
- Simulate environment
- Verify complete system

### 14.4 Loopback Testing
- Connect TX to RX
- Verify transmit/receive
- Check echo

### 14.5 Fault Injection
- Corrupt packets
- Simulate timeouts
- Disconnect devices
- Inject noise

### 14.6 Packet Corruption
- Flip bits
- Truncate frames
- Insert errors
- Verify error handling

### 14.7 Timeout Testing
- Delay responses
- Omit ACKs
- Verify timeout handling

### 14.8 Disconnect Testing
- Unplug devices
- Break connections
- Verify reconnection

### 14.9 Bus Overload
- Flood bus with traffic
- Test arbitration
- Verify priority handling

### 14.10 Malformed Packets
- Invalid headers
- Wrong lengths
- Bad checksums
- Verify error detection

### 14.11 Fuzzing
- Random inputs
- Boundary values
- Stress conditions

### 14.12 Stress Testing
- Maximum load
- Continuous operation
- Resource exhaustion

### 14.13 Endurance Testing
- Long-duration operation
- Wear and tear
- Memory leaks

---

## 15. Security

### 15.1 Security Concepts
- **Authentication**: Verify identity
- **Authorization**: Verify permissions
- **Encryption**: Protect data confidentiality
- **Integrity**: Detect tampering
- **Replay protection**: Prevent message reuse

### 15.2 Secure Boot
- Verify firmware signature
- Chain of trust
- Prevent unauthorized code

### 15.3 Secure Firmware Update
- Signed updates
- Encrypted transfer
- Rollback protection
- Secure storage

### 15.4 Key Storage
- Secure element
- TPM (Trusted Platform Module)
- Encrypted storage
- Key derivation

### 15.5 Certificates
- X.509 certificates
- Certificate authority
- Certificate validation

### 15.6 TLS/DTLS
- **TLS**: Transport Layer Security (TCP)
- **DTLS**: Datagram TLS (UDP)
- **Purpose**: Secure communication

### 15.7 Secure CAN Concepts
- Message authentication
- Encryption (optional)
- Intrusion detection
- Secure gateways

### 15.8 Network Segmentation
- Separate networks
- Firewalls
- Access control

### 15.9 Attack Surface
- Minimize exposed interfaces
- Disable unused features
- Regular updates

### 15.10 Protocol Security vs Application Security
| Aspect | Protocol Security | Application Security |
|---|---|---|
| **Built-in** | TLS, DTLS, secure CAN | Application-layer encryption |
| **Authentication** | Certificate-based | Username/password, tokens |
| **Encryption** | Protocol-level | Application-level |
| **Examples** | HTTPS, MQTT over TLS | Custom encryption |

---

## 16. Real-World System Architectures

### 16.1 Example 1: Sensor Device
```
Sensor (I2C temperature)
    ↓
I2C
    ↓
MCU (STM32)
    ↓
UART
    ↓
Linux Gateway (Raspberry Pi)
    ↓
MQTT over TLS
    ↓
Cloud (AWS IoT)
```

**Why this architecture?**
- I2C: Simple sensor interface
- UART: Reliable MCU-to-gateway communication
- MQTT: Lightweight IoT messaging
- TLS: Secure cloud communication

### 16.2 Example 2: Automotive ECU
```
Sensor (wheel speed)
    ↓
MCU (Automotive grade)
    ↓
CAN-FD
    ↓
Gateway ECU
    ↓
Automotive Ethernet (100BASE-T1)
    ↓
Central Compute ECU
```

**Why this architecture?**
- CAN-FD: Robust, deterministic
- Automotive Ethernet: High bandwidth
- Gateway: Protocol translation

### 16.3 Example 3: Industrial Machine
```
Sensor (pressure)
    ↓
RS-485
    ↓
Modbus RTU
    ↓
PLC (Siemens S7)
    ↓
Ethernet
    ↓
PROFINET
    ↓
SCADA System
```

**Why this architecture?**
- RS-485: Long distance, noisy environment
- Modbus RTU: Simple, widely supported
- PROFINET: Real-time industrial Ethernet
- SCADA: Centralized monitoring

### 16.4 Example 4: Medical Imaging Device
```
Detector (X-ray sensor)
    ↓
High-speed interface (LVDS/MIPI)
    ↓
Embedded processing (FPGA/SoC)
    ↓
Ethernet / USB 3.0
    ↓
Workstation
    ↓
DICOM
    ↓
PACS (Picture Archiving System)
```

**Why this architecture?**
- High-speed interface: Large data volumes
- Embedded processing: Real-time image processing
- DICOM: Medical imaging standard
- PACS: Centralized storage

---

## 17. Protocol Selection Guide

| Requirement | Recommended Technology | Why |
|---|---|---|
| Cheapest MCU communication | GPIO, UART | Simple, minimal hardware |
| Few sensors (1–3) | UART, SPI | Point-to-point, simple |
| Many sensors (4+) | I2C, I3C | Multi-device, low pin count |
| High speed (>10 Mbit/s) | SPI, QSPI, OSPI, Ethernet | High throughput |
| Long distance (>100m) | RS-485, CAN, Ethernet | Robust, differential |
| Noisy environment | RS-485, CAN, Ethernet with shielding | Differential signaling |
| Deterministic communication | CAN, CAN-FD, TSN, EtherCAT | Priority-based, time-triggered |
| Automotive | CAN, CAN-FD, Automotive Ethernet, LIN | Industry standards |
| Industrial | Modbus, PROFINET, EtherCAT, EtherNet/IP | Industrial protocols |
| Wireless (short range) | BLE, Zigbee, Thread | Low power, mesh |
| Wireless (long range) | LoRaWAN, Cellular | Long distance |
| Battery powered | BLE, Zigbee, Thread | Low power consumption |
| Camera | MIPI CSI-2, USB UVC, GigE Vision | High bandwidth |
| Display | MIPI DSI, HDMI, LVDS | Video interfaces |
| Storage | SPI NOR, QSPI, OSPI, eMMC, UFS, NVMe | Non-volatile storage |
| Audio | I2S, TDM, PDM, USB Audio | Audio-specific interfaces |
| Linux device | Ethernet, USB, UART, I2C, SPI | Linux support |
| Cloud IoT | MQTT, CoAP, HTTP over TLS | Internet protocols |
| Medical imaging | DICOM, GigE Vision, USB3 Vision | Medical standards |
| Building automation | BACnet, KNX, MQTT | Building protocols |
| Real-time networking | TSN, EtherCAT, PROFINET IRT | Deterministic Ethernet |

---

## 18. Learning Roadmap

### Stage 1: Fundamentals
**Concepts**: GPIO, digital I/O, basic electronics
**Protocols**: UART, I2C, SPI
**Projects**:
- Blink LED with GPIO
- UART terminal (printf/scanf)
- I2C temperature sensor
- SPI flash read/write
**Debugging tools**: Logic analyzer, oscilloscope
**Expected skills**: Basic register programming, simple drivers

### Stage 2: Serial Communication
**Concepts**: Asynchronous vs synchronous, framing, error detection
**Protocols**: RS-232, RS-485, CAN, Modbus RTU
**Projects**:
- RS-232 PC communication
- RS-485 multi-drop network
- CAN node
- Modbus RTU slave device
**Debugging tools**: Protocol analyzer, CAN analyzer
**Expected skills**: Multi-device networks, error handling

### Stage 3: USB and Ethernet
**Concepts**: USB enumeration, Ethernet framing, TCP/IP
**Protocols**: USB (HID, CDC, MSC), Ethernet, TCP, UDP, IP
**Projects**:
- USB HID device (keyboard/mouse)
- USB CDC virtual COM port
- Ethernet TCP server
- UDP client
**Debugging tools**: Wireshark, USB analyzer
**Expected skills**: USB device stack, network programming

### Stage 4: Linux and Advanced Topics
**Concepts**: Linux kernel, device drivers, DMA, interrupts
**Protocols**: SocketCAN, I2C/SPI in Linux, netlink
**Projects**:
- Linux CAN application
- Linux I2C driver
- Linux SPI driver
- DMA-based UART driver
**Debugging tools**: Kernel debugging, ftrace, perf
**Expected skills**: Kernel programming, DMA, advanced debugging

### Stage 5: Wireless and IoT
**Concepts**: RF, mesh networking, IoT protocols
**Protocols**: BLE, Wi-Fi, MQTT, CoAP, TLS
**Projects**:
- BLE peripheral
- Wi-Fi station
- MQTT sensor node
- Secure MQTT gateway
**Debugging tools**: RF analyzer, Wireshark
**Expected skills**: Wireless protocols, IoT architectures

### Stage 6: Specialization
**Concepts**: Industry-specific protocols, safety, security
**Protocols**:
- **Automotive**: CAN-FD, Automotive Ethernet, UDS, SOME/IP
- **Industrial**: PROFINET, EtherCAT, OPC UA
- **Medical**: DICOM, HL7
**Projects**:
- CAN-FD ECU
- Automotive Ethernet node
- EtherCAT slave
- DICOM client
**Debugging tools**: Industry-specific analyzers
**Expected skills**: Industry standards, safety-critical systems

### Stage 7: High-Speed and Advanced
**Concepts**: High-speed signaling, signal integrity, PCIe
**Protocols**: MIPI CSI-2/DSI, PCIe, NVMe, UFS
**Projects**:
- MIPI camera pipeline
- PCIe endpoint
- NVMe driver
- High-speed storage system
**Debugging tools**: High-speed oscilloscope, protocol analyzer
**Expected skills**: High-speed design, signal integrity

---

## 19. Projects

### Beginner Projects

#### 1. UART Terminal
**Objective**: Create a simple serial terminal
**Hardware**: MCU, USB-to-UART adapter
**Protocols**: UART
**Software**: printf/scanf over UART
**Architecture**:
```
PC <--UART--> MCU
```
**Skills learned**: UART configuration, serial communication

#### 2. I2C Temperature Sensor
**Objective**: Read temperature from I2C sensor
**Hardware**: MCU, I2C temperature sensor (e.g., TMP102)
**Protocols**: I2C
**Software**: I2C driver, sensor library
**Architecture**:
```
MCU <--I2C--> Temperature Sensor
```
**Skills learned**: I2C communication, sensor integration

#### 3. SPI Flash Driver
**Objective**: Read/write SPI NOR flash
**Hardware**: MCU, SPI flash chip
**Protocols**: SPI
**Software**: SPI driver, flash commands
**Architecture**:
```
MCU <--SPI--> Flash Memory
```
**Skills learned**: SPI communication, flash memory commands

#### 4. PWM Motor Controller
**Objective**: Control motor speed with PWM
**Hardware**: MCU, motor driver, motor
**Protocols**: PWM, GPIO
**Software**: PWM generation, motor control
**Architecture**:
```
MCU --PWM--> Motor Driver --Power--> Motor
```
**Skills learned**: PWM generation, motor control

### Intermediate Projects

#### 5. RS-485 Modbus Device
**Objective**: Create Modbus RTU slave device
**Hardware**: MCU, RS-485 transceiver
**Protocols**: RS-485, Modbus RTU
**Software**: Modbus stack, RS-485 driver
**Architecture**:
```
Master <--RS-485/Modbus--> Slave Device
```
**Skills learned**: Multi-drop networks, Modbus protocol

#### 6. CAN Node
**Objective**: Create CAN communication node
**Hardware**: MCU, CAN transceiver
**Protocols**: CAN
**Software**: CAN driver, message handling
**Architecture**:
```
ECU1 <--CAN--> ECU2 <--CAN--> ECU3
```
**Skills learned**: CAN communication, multi-master networks

#### 7. USB HID Device
**Objective**: Create USB keyboard/mouse
**Hardware**: MCU with USB, USB connector
**Protocols**: USB HID
**Software**: USB device stack, HID class
**Architecture**:
```
PC <--USB--> HID Device
```
**Skills learned**: USB device, HID class

#### 8. Ethernet TCP Server
**Objective**: Create TCP server on embedded device
**Hardware**: MCU with Ethernet, Ethernet PHY
**Protocols**: Ethernet, TCP, IP
**Software**: TCP/IP stack, server application
**Architecture**:
```
Client <--Ethernet/TCP--> Server
```
**Skills learned**: Network programming, TCP server

#### 9. MQTT Sensor
**Objective**: Create MQTT sensor node
**Hardware**: MCU, sensor, Wi-Fi or Ethernet
**Protocols**: MQTT, TCP/IP, Wi-Fi/Ethernet
**Software**: MQTT client, sensor driver
**Architecture**:
```
Sensor --> MCU --MQTT--> Broker
```
**Skills learned**: IoT protocols, cloud connectivity

### Advanced Projects

#### 10. Linux CAN Application
**Objective**: Create CAN application on Linux
**Hardware**: Linux device with CAN (e.g., Raspberry Pi with CAN hat)
**Protocols**: CAN, SocketCAN
**Software**: SocketCAN API, application
**Architecture**:
```
Application <--SocketCAN--> CAN Device
```
**Skills learned**: Linux networking, SocketCAN

#### 11. Linux I2C/SPI Driver
**Objective**: Write Linux kernel driver
**Hardware**: Linux device with I2C/SPI
**Protocols**: I2C/SPI in Linux
**Software**: Linux kernel driver
**Architecture**:
```
Userspace <--ioctl--> Kernel Driver <--Hardware--> Device
```
**Skills learned**: Linux kernel programming, device drivers

#### 12. USB Linux Driver
**Objective**: Write USB driver for Linux
**Hardware**: USB device
**Protocols**: USB
**Software**: Linux USB driver
**Architecture**:
```
Userspace <--libusb/usbfs--> Kernel Driver <--USB--> Device
```
**Skills learned**: USB protocol, Linux USB subsystem

#### 13. CAN Bootloader
**Objective**: Create bootloader over CAN
**Hardware**: MCU with CAN, flash memory
**Protocols**: CAN, bootloader protocol
**Software**: Bootloader, flash programming
**Architecture**:
```
Host --CAN--> Bootloader --> Flash
```
**Skills learned**: Bootloaders, firmware update

#### 14. OTA Firmware Updater
**Objective**: Create over-the-air update system
**Hardware**: MCU with Wi-Fi/Ethernet, flash
**Protocols**: HTTP/HTTPS, MQTT, bootloader
**Software**: Update client, server
**Architecture**:
```
Device --HTTP/MQTT--> Server
```
**Skills learned**: Secure updates, OTA systems

#### 15. MIPI Camera Pipeline
**Objective**: Process MIPI camera data
**Hardware**: MIPI camera, SoC with MIPI CSI-2
**Protocols**: MIPI CSI-2
**Software**: Camera driver, image processing
**Architecture**:
```
Camera --MIPI--> SoC --Processing--> Display/Storage
```
**Skills learned**: High-speed interfaces, image processing

#### 16. Industrial EtherCAT System
**Objective**: Create EtherCAT slave device
**Hardware**: MCU/Ethernet, EtherCAT PHY
**Protocols**: EtherCAT, Ethernet
**Software**: EtherCAT stack
**Architecture**:
```
Master --EtherCAT--> Slave1 --EtherCAT--> Slave2
```
**Skills learned**: Industrial Ethernet, real-time systems

---

## 20. Glossary

| Term | Definition |
|------|------------|
| **ACK** | Acknowledge; confirmation of receipt |
| **NACK** | Negative Acknowledge; rejection of data |
| **CRC** | Cyclic Redundancy Check; error detection code |
| **Checksum** | Sum of data bytes; error detection |
| **FIFO** | First-In-First-Out buffer |
| **DMA** | Direct Memory Access; hardware data transfer |
| **ISR** | Interrupt Service Routine |
| **PHY** | Physical layer device (e.g., Ethernet PHY) |
| **MAC** | Media Access Control; data-link layer address |
| **MTU** | Maximum Transmission Unit |
| **MSS** | Maximum Segment Size |
| **TTL** | Time To Live; packet lifetime |
| **CMOS** | Complementary Metal-Oxide-Semiconductor |
| **LVDS** | Low-Voltage Differential Signaling |
| **UART** | Universal Asynchronous Receiver-Transmitter |
| **USART** | Universal Synchronous/Asynchronous Receiver-Transmitter |
| **SCL** | Serial Clock (I2C) |
| **SDA** | Serial Data (I2C) |
| **MOSI** | Master Out Slave In (SPI) |
| **MISO** | Master In Slave Out (SPI) |
| **SCLK** | Serial Clock (SPI) |
| **CS** | Chip Select (SPI) |
| **CAN ID** | Message identifier in CAN |
| **Arbitration** | Process to determine bus access priority |
| **Bus-off** | CAN error state; node removed from bus |
| **Baud rate** | Symbols per second |
| **Bit rate** | Bits per second |
| **Jitter** | Timing variation |
| **Latency** | Time delay |
| **Throughput** | Actual data rate |
| **Bandwidth** | Maximum data rate |
| **Frame** | Data unit at data-link layer |
| **Packet** | Data unit at network layer |
| **Payload** | Actual data (excluding headers) |
| **Header** | Control information at start of frame |
| **Footer** | Control information at end of frame |
| **Endpoint** | USB communication endpoint |
| **Descriptor** | USB device information structure |
| **Socket** | Network communication endpoint |
| **Port** | TCP/UDP port number |
| **Node** | Network device |
| **Gateway** | Device connecting networks |
| **Controller** | Master device |
| **Peripheral** | Slave device |
| **Target** | Slave device (alternative term) |
| **Transceiver** | Transmitter + Receiver |
| **HAL** | Hardware Abstraction Layer |
| **BSP** | Board Support Package |
| **RTOS** | Real-Time Operating System |
| **HIL** | Hardware-in-the-Loop |
| **OTA** | Over-The-Air (update) |
| **XIP** | Execute-In-Place (from flash) |
| **ECC** | Error Correction Code |
| **TSN** | Time-Sensitive Networking |
| **UDS** | Unified Diagnostic Services |
| **DoIP** | Diagnostic over IP |
| **OBD-II** | On-Board Diagnostics II |
| **PLC** | Programmable Logic Controller |
| **SCADA** | Supervisory Control and Data Acquisition |
| **HMI** | Human-Machine Interface |
| **ECU** | Electronic Control Unit |
| **PACS** | Picture Archiving and Communication System |
| **DICOM** | Digital Imaging and Communications in Medicine |
| **HL7** | Health Level 7 (medical data standard) |
| **FHIR** | Fast Healthcare Interoperability Resources |

---

## 21. Final Master Table

| Technology | Layer | Category | Medium | Speed | Distance | Topology | Duplex | Addressing | Error Detection | Deterministic | Typical Use | Difficulty |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| GPIO | Physical | Interface | Copper trace | N/A | On-board | Point-to-point | N/A | Pin number | None | No | Simple I/O, control lines | Beginner |
| UART | Data-Link/Physical | Serial bus | Copper | 1–10+ Mbit/s | Short–Medium | Point-to-point | Full/Half | None | Parity, framing | No | Console, debug, bootloaders | Beginner |
| USART | Physical/Data-Link | Serial interface | Copper | Similar to UART | Short–Medium | Point-to-point | Full/Half | None | Parity, framing | No | MCU serial comms | Beginner |
| I2C | Data-Link | Serial bus | 2-wire copper | 100 k–3.4 Mbit/s | Short | Multi-master, multi-slave | Half | 7/10-bit address | ACK/NACK, optional CRC | No | Sensors, EEPROMs | Beginner |
| I3C | Data-Link | Serial bus | 2-wire (I3C PHY) | 12.5–33 Mbit/s | Short | Multi-master | Full | Dynamic address assignment | CRC, ACK/NACK | Partial | High-speed sensors | Advanced |
| SPI | Data-Link | Serial bus | 4+ wires | 1–50+ Mbit/s | Short | Point-to-point / multi-slave | Full | Chip-select | None (higher-layer CRC) | No | Flash, displays, sensors | Beginner |
| QSPI | Data-Link | Serial bus | 4 data lines | 50–200+ Mbit/s | Short | Point-to-point | Full | Chip-select | CRC (device) | No | NOR flash, XIP | Intermediate |
| OSPI | Data-Link | Serial bus | 8 data lines | 100–400+ Mbit/s | Short | Point-to-point | Full | Chip-select | CRC (device) | No | High-speed flash | Advanced |
| I2S | Data-Link | Audio serial | Multi-wire | Audio rates (kHz–MHz) | Short | Point-to-point | Full | None (channel-based) | None (higher-layer) | No | Audio codecs | Beginner |
| TDM | Data-Link | Audio serial | Multi-wire | Audio rates | Short | Point-to-point | Full | Time-slot | None | No | Multi-channel audio | Intermediate |
| PDM | Data-Link | Audio serial | 1–2 wires | Audio rates | Short | Point-to-point | Half | None | None | No | MEMS mics | Beginner |
| SAI | Data-Link | Audio serial | Multi-wire | Audio rates | Short | Multi-channel | Full | Slot-based | None | No | SoC audio blocks | Intermediate |
| 1-Wire | Data-Link | Serial bus | Single wire | 16 k–16 Mbit/s | Short | Point-to-point / multi-drop | Half | 64-bit ROM ID | CRC | No | ID, sensors | Beginner |
| PWM | Physical | Control interface | Copper | kHz–MHz | On-board | Point-to-point | N/A | None | None | No | Motor control, LEDs | Beginner |
| ADC/DAC interfaces | Physical/Data-Link | Analog/digital interface | Copper | N/A | On-board | Point-to-point | N/A | Channel | None | No | Analog sensing | Beginner |
| RS-232 | Electrical/Physical | Serial electrical | Single-ended | Up to ~1 Mbit/s | Short–Medium | Point-to-point | Full | None | Parity (optional) | No | Legacy serial, consoles | Beginner |
| RS-422 | Electrical/Physical | Serial electrical | Differential | 100 k–10 Mbit/s | Medium | Multi-drop | Full | None | Parity (optional) | No | Industrial serial | Intermediate |
| RS-485 | Electrical/Physical | Serial electrical | Differential | 1 k–20 Mbit/s | Medium–Long | Multi-drop | Half/Full | Node address (higher-layer) | Parity/CRC (higher-layer) | No | Modbus, industrial | Intermediate |
| RS-423 | Electrical/Physical | Serial electrical | Single-ended | Similar to RS-232 | Short–Medium | Point-to-point | Full | None | Parity (optional) | No | Legacy devices | Beginner |
| CAN | Data-Link/Network | Fieldbus | Differential | 1 Mbit/s (classic) | Medium | Multi-master, multi-drop | Multi | 11/29-bit ID | CRC, ACK, error frames | Yes (with arbitration) | Automotive, industrial | Intermediate |
| CAN-FD | Data-Link/Network | Fieldbus | Differential | Up to 8 Mbit/s | Medium | Multi-master | Multi | 11/29-bit ID | CRC, ACK | Yes | Automotive, industrial | Intermediate |
| CAN XL | Data-Link/Network | Fieldbus | Differential | Very high | Medium | Multi-master | Multi | Extended ID | CRC, ACK | Yes | Next-gen automotive | Advanced |
| LIN | Data-Link | Fieldbus | Single-wire | 20–33.3 kbit/s | Short | Master/slave | Half | Node address | Parity, checksum | No | Body electronics | Beginner |
| FlexRay | Data-Link | Fieldbus | Twin-axial | 10–40 Mbit/s | Medium | Multi-drop | Full | Slot-based | CRC, ACK | Yes | High-end automotive | Advanced |
| Automotive Ethernet (100BASE-T1) | Data-Link/Physical | Ethernet PHY | Twisted pair | 100 Mbit/s | Vehicle-scale | Star/Point-to-point | Full | MAC/IP | CRC (Ethernet) | Yes (TSN) | ECUs, gateways | Advanced |
| Automotive Ethernet (1000BASE-T1) | Data-Link/Physical | Ethernet PHY | Twisted pair | 1 Gbit/s | Vehicle-scale | Star/Point-to-point | Full | MAC/IP | CRC | Yes (TSN) | ECUs, gateways | Advanced |
| UDS | Application/Transport | Diagnostic protocol | CAN/Ethernet | Varies | Vehicle-scale | Client/server | Full | ECU address | CRC (underlying) | No | Diagnostics | Advanced |
| DoIP | Transport/Application | Diagnostic protocol | Ethernet | 100 Mbps+ | Vehicle-scale | Client/server | Full | IP/MAC | CRC (Ethernet) | No | Remote diagnostics | Advanced |
| OBD-II | Application | Diagnostic standard | CAN/Ethernet | Varies | Vehicle-scale | Client/server | Full | PID-based | CRC (underlying) | No | Vehicle diagnostics | Intermediate |
| J1939 | Application | Higher-layer protocol | CAN | Up to 250 k–1 Mbit/s | Medium | Multi-drop | Full | PGN/SA | CRC (CAN) | No | Heavy-duty vehicles | Intermediate |
| SOME/IP | Application | Service protocol | Ethernet | 100 Mbps+ | Vehicle-scale | Client/server | Full | Service ID | CRC (Ethernet) | No | Service discovery | Advanced |
| AUTOSAR comms | Application/Architecture | Standard concepts | Varied | Varies | Varied | Varied | Varied | Varied | Varied | No | Automotive SW architecture | Advanced |
| CANopen | Application | Higher-layer protocol | CAN | Up to 1 Mbit/s | Short–Medium | Multi-drop | Full | Node ID | CRC (CAN) | No | Industrial automation | Intermediate |
| Modbus RTU | Application/Transport | Fieldbus protocol | RS-485/RS-232 | 7–115.2 kbit/s | Medium | Multi-drop | Full | Slave address | CRC/LRC | No | PLCs, RTUs | Beginner |
| Modbus ASCII | Application/Transport | Fieldbus protocol | RS-232/RS-485 | 7–115.2 kbit/s | Medium | Multi-drop | Full | Slave address | LRC | No | Legacy PLCs | Beginner |
| Modbus TCP | Application/Transport | Fieldbus protocol | Ethernet | 10/100 Mbps | Medium | Star | Full | IP + Unit ID | CRC (Ethernet) | No | Industrial gateways | Intermediate |
| PROFIBUS | Data-Link/Application | Fieldbus | Copper | 9.6 k–12 Mbit/s | Medium | Multi-master | Full | Station address | CRC | Yes | Industrial automation | Advanced |
| PROFINET | Application/Transport | Industrial Ethernet | Ethernet | 100 Mbps–1 Gbps | Medium | Star/daisy-chain | Full | MAC/IP | CRC | Yes (IRT) | PLCs, IO | Advanced |
| EtherCAT | Data-Link/Network | Fieldbus | Ethernet | 100 Mbps | Short–Medium | Line/tree | Full | Station alias | CRC | Yes | Motion control | Advanced |
| EtherNet/IP | Application/Transport | Industrial Ethernet | Ethernet | 10 Mbps–1 Gbps | Medium | Star/tree | Full | MAC/IP | CRC | No | PLCs, IO | Intermediate |
| DeviceNet | Data-Link/Application | Fieldbus | CAN | 125–500 kbit/s | Short | Multi-drop | Full | MAC ID | CRC (CAN) | No | Industrial IO | Intermediate |
| USB 1.x | Data-Link/Physical | Peripheral bus | USB cable | 1.5–12 Mbit/s | Short | Star (hub) | Full | Endpoint | CRC | No | Legacy peripherals | Beginner |
| USB 2.0 | Data-Link/Physical | Peripheral bus | USB cable | 480 Mbit/s | Short | Star (hub) | Full | Endpoint | CRC | No | Mass storage, HID | Intermediate |
| USB 3.x | Data-Link/Physical | Peripheral bus | USB cable | 5–40 Gbit/s | Short | Star (hub) | Full | Endpoint | CRC | No | High-speed peripherals | Advanced |
| USB-C / OTG | Physical/Transport | Connector/role-swap | USB-C cable | 480 Mbps–40 Gbps | Short | Point-to-point | Full | Endpoint | CRC | No | Modern devices | Intermediate |
| USB HID | Application | USB class | USB | 1.5–12 Mbit/s | Short | Point-to-point | Full | Endpoint | CRC | No | Keyboards, mice | Beginner |
| USB CDC | Application | USB class | USB | 1.5–12 Mbit/s | Short | Point-to-point | Full | Endpoint | CRC | No | Virtual COM | Beginner |
| USB MSC | Application | USB class | USB | 1.5–12 Mbit/s | Short | Point-to-point | Full | Endpoint | CRC | No | Mass storage | Beginner |
| USB Audio | Application | USB class | USB | 1.5–12 Mbit/s | Short | Point-to-point | Full | Endpoint | CRC | No | Audio devices | Intermediate |
| USB Video Class | Application | USB class | USB | 1.5–12 Mbit/s | Short | Point-to-point | Full | Endpoint | CRC | No | Webcams | Intermediate |
| USB DFU | Application | Bootloader class | USB | 1.5–12 Mbit/s | Short | Point-to-point | Full | Endpoint | CRC | No | Firmware update | Intermediate |
| PCI Express | Data-Link/Physical | High-speed bus | Copper/fiber | Gen1–Gen6 multi-Gbps | Short–Medium | Point-to-point | Full | Device/function | CRC | No | High-speed peripherals | Advanced |
| PCI | Data-Link/Physical | Bus (legacy) | Parallel/serial | 33–133 MHz | Short | Multi-drop | Full | Device/function | CRC | No | Legacy expansion | Intermediate |
| SD / SDIO | Data-Link/Physical | Storage interface | Copper | 12.5–104 Mbit/s | Short | Point-to-point | Full | Card select | CRC | No | Memory cards | Beginner |
| eMMC | Data-Link/Physical | Storage interface | Copper | 52–200+ Mbit/s | Short | Point-to-point | Full | Chip-select | CRC | No | Embedded storage | Intermediate |
| UFS | Data-Link/Physical | Storage interface | Copper | 1–2.9 Gbit/s | Short | Point-to-point | Full | LUN | CRC | No | High-performance storage | Advanced |
| NVMe | Transport/Application | Storage protocol | PCIe | 1–8+ Gbps | Short | Point-to-point | Full | Namespace | CRC | No | SSDs | Advanced |
| SPI NOR Flash | Data-Link/Physical | Storage interface | SPI | Tens of MHz | Short | Point-to-point | Full | Chip-select | CRC (device) | No | Code storage | Beginner |
| NAND Flash | Data-Link/Physical | Storage interface | Parallel/SPI | Varies | Short | Point-to-point | Full | Chip-select | ECC | No | Mass storage | Intermediate |
| MIPI CSI-2 | Data-Link/Physical | Camera interface | Differential lanes | 1–12 Gbit/s/lane | Short | Point-to-point | Full | Virtual channel | CRC | No | Image sensors | Advanced |
| MIPI DSI | Data-Link/Physical | Display interface | Differential lanes | Gbps | Short | Point-to-point | Full | Virtual channel | CRC | No | Displays | Advanced |
| LVDS | Data-Link/Physical | Video interface | Differential pair | 1–4 Gbit/s | Medium | Point-to-point | Full | None | None | No | Video panels | Intermediate |
| HDMI | Data-Link/Physical | Display interface | Differential | 2–18+ Gbit/s | Short | Point-to-point | Full | None | CRC | No | Monitors, TVs | Intermediate |
| DisplayPort | Data-Link/Physical | Display interface | Differential | 2–18+ Gbit/s | Short | Point-to-point | Full | None | CRC | No | Monitors | Intermediate |
| Ethernet | Data-Link/Network/Physical | Networking | Twisted pair/fiber | 10 Mbps–100 Gbps | Medium–Long | Star/tree | Full | MAC/IP | CRC | No (TSN yes) | General networking | Intermediate |
| IPv4 | Network | Networking protocol | IP | N/A | Global | N/A | N/A | IP address | Checksum | No | Internet/industrial | Intermediate |
| IPv6 | Network | Networking protocol | IP | N/A | Global | N/A | N/A | IP address | Checksum | No | Internet/industrial | Intermediate |
| ARP | Data-Link/Network | Resolution protocol | Broadcast | N/A | Local | N/A | N/A | MAC/IP mapping | None | No | IP→MAC resolution | Beginner |
| TCP | Transport | Transport protocol | IP | N/A | End-to-end | Full | Port | Checksum, ACK | No | Reliable streams | Intermediate |
| UDP | Transport | Transport protocol | IP | N/A | End-to-end | Full | Port | Checksum | No | Real-time datagrams | Beginner |
| TLS/DTLS | Transport/Application | Security protocol | TCP/UDP | N/A | End-to-end | Full | Port | MAC, encryption | No | Secure channels | Advanced |
| HTTP/HTTPS | Application | Web protocol | TCP/TLS | N/A | Global | Full | URL/port | TLS (HTTPS) | No | Web services | Intermediate |
| MQTT | Application | Messaging protocol | TCP/TLS/UDP | N/A | Global | Full | Topic | TLS (optional) | No | IoT pub/sub | Intermediate |
| CoAP | Application | Messaging protocol | UDP/TCP | N/A | Local/Global | Full | URI | DTLS (optional) | No | Constrained IoT | Intermediate |
| AMQP | Application | Messaging protocol | TCP/TLS | N/A | Global | Full | Queue/exchange | TLS | No | Enterprise messaging | Advanced |
| OPC UA | Application | Industrial protocol | TCP/HTTPS | N/A | Global | Full | Endpoint | TLS | No | SCADA, PLCs | Advanced |
| BACnet | Application | Building protocol | IP/MS/RTU | N/A | Building-scale | Full | Device ID | CRC (underlying) | No | Building automation | Intermediate |
| KNX | Application | Building protocol | Twisted pair/wireless | N/A | Building-scale | Full | Physical address | CRC | No | Smart buildings | Intermediate |
| Thread | Network/Link/Application | Wireless mesh | 2.4 GHz RF | 20–250 kbit/s | Home/building | Mesh | Full | IPv6 | AES-128 | Yes (mesh) | Smart home | Intermediate |
| Zigbee | Network/Link/Application | Wireless mesh | 2.4 GHz RF | 250 kbit/s | Home/building | Mesh | Full | 16-bit short/64-bit long | CRC, AES | No | Home automation | Beginner |
| BLE Classic | Data-Link/Application | Wireless | 2.4 GHz RF | 1–3 Mbps | Short | Star/Point-to-point | Full | MAC | CRC | No | Audio, HID | Intermediate |
| BLE LE / GATT | Data-Link/Application | Wireless | 2.4 GHz RF | 125 k–2 Mbps | Short | Star/Mesh | Full | MAC, handle | CRC, AES | No | Sensors, wearables | Intermediate |
| LoRa / LoRaWAN | Physical/Data-Link/Application | Wireless WAN | Sub-GHz RF | 0.3–50 kbit/s | Long | Star | Half | DevAddr | CRC, MIC | No | Remote sensors | Intermediate |
| NFC / RFID | Physical/Data-Link | Wireless proximity | Near-field RF | 424–848 kbit/s | cm-scale | Point-to-point | Half | UID | CRC | No | Access, tags | Beginner |
| UWB | Physical/Data-Link | Wireless | Ultra-wideband RF | 110–4800 Mbps | Short | Point-to-point/mesh | Full | MAC | CRC | Yes | Ranging, positioning | Advanced |
| Cellular (LTE/5G) | Network/Link | Wireless WAN | RF cellular | Mbps–Gbps | Global | Star | Full | IMSI/IMEI | CRC, encryption | No | IoT gateways | Advanced |
| DICOM / HL7 / FHIR | Application | Medical protocols | Hospital networks | N/A | Hospital-scale | Client/server | Full | AE titles, URIs | TLS (optional) | No | Medical imaging, EHR | Advanced |
| Matter | Application/Network | IoT protocol | Wi‑Fi/Ethernet/Thread | N/A | Home-scale | Mesh/Hybrid | Full | IPv6 | TLS, AES | No | Smart home | Intermediate |
| TSN (IEEE 802.1Qbv etc.) | Data-Link/Physical | Real-time Ethernet | Ethernet | 100 Mbps–10 Gbps | Local | Star/tree | Full | MAC/IP | CRC | Yes | Industrial real-time | Advanced |
| Ethernet Powerlink | Data-Link/Application | Real-time Ethernet | Ethernet | Real-time | Local | Star | Full | Node ID | CRC | Yes | Motion control | Advanced |

---

*Document generated: Thursday, August 13, 2026*
*Version: 1.0*
*Target audience: Embedded C/C++ developers, firmware engineers, embedded Linux developers, device driver developers*
