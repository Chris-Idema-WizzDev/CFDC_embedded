# CAN-FD USB converter

This is an STM32-based CAN-FD USB Converter. CAN-FD is an extension of CAN with options for higher bit rates and larger payloads.
It is compatible with tools such as SocketCAN, can-utils(cansend/candump) and pythonCAN.

# Fork

This is a fork of https://github.com/ucandevices/CFDC_embedded.
Here are some differences:
- It is ported from STM32G431CBUx to STM32G491MET.
- Our board uses a crystal instead of the internal oscillator. 
- The pinout is different. 
- There are two CAN drivers but only 1 is used.
- The CAN-drivers have a standby pin, which is not used.
- it is made with a newer version of STM32CubeMX: 6.4.0

# Build

Build with STM32CubeIde V1.8.0 and flash with an ST-link.

# Run

## Install tools

Install can-utils
```
sudo apt-get install can-utils
```

Install ucan_utils

```
git clone https://github.com/ucandevices/ucan_utils --recurse-submodules
cd ucan_utils
make
```

```
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```
## Start uccbsocketcan with configuration
Run this and keep this running
```
sudo ./uccbsocketcan -cvcan0 -d250 -i250 -fc -mn
```

## Monitor messages
```
candump vcan0
```

## Send messages

Not extended:
```
cansend vcan0 123#112233 
```
Force extended:
```
cansend vcan0 00000123#112233
```
Extended:
```
cansend vcan0 18FEEEFE#112233
```

# Python with python-can

open:
```
import can
with can.interface.Bus(interface="socketcan", channel="vcan0", fd=True) as bus:
```
Send:
```
msg = can.Message(
    arbitration_id=0x123, 
    data=[0x11,0x22,0x33,0x44,0x55,0x66,0x77,0x88], 
    is_extended_id=True
)

bus.send(msg)
```

Read:
```
msg = bus.recv(1)
if msg is not None:
    print(msg)
```

# C

Plenty of examples of socketcan exist such as can-utils. Make sure to activate CAN-FD or you won't be able to receive data.
Activate CAN-FD
```
static const int canfd_on = 1;
setsockopt(s, SOL_CAN_RAW, CAN_RAW_FD_FRAMES, &canfd_on, sizeof(canfd_on));
```

# Todo

- test CAN-FD frameslonger than 8 bytes, right now only classic CAN is tested
- implement bootloader

# Misc

- forked from: https://github.com/ucandevices/CFDC_embedded
- original project this is based on: https://ucandevices.github.io/cfuc.html
- ucan_itils: https://github.com/ucandevices/ucan_utils
- can-utils: https://github.com/linux-can/can-utils
- python-can: https://github.com/hardbyte/python-can
