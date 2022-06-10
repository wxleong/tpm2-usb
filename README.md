# Introduction

Access TPM through USB interface.

# Table of Contents

- **[Prerequisites](#prerequisites)**
- **[Hardware Setup](#hardware-setup)**
- **[Software Setup](#software-setup)**
- **[Testing](#Testing)**

# Prerequisites

- Raspberry Pi 4 Model B or Ubuntu 20.04
- Install dependencies:
  - Ubuntu: [[6]](#6)
  - Raspberry: [[7]](#7)
- Iridium 9670 TPM 2.0 board [[1]](#1)
- USB-FTDI-SPI converter (C232HM-DDHSL-0) [[2]](#2)[[3]](#3)

# Hardware Setup

Connect your TPM with FTDI according to [here](media/hardware.pptx)

Plug in the USB:
```
$ dmesg
[35283.252922] usb 3-1: new high-speed USB device number 4 using xhci_hcd
[35283.604846] usb 3-1: New USB device found, idVendor=0403, idProduct=6014, bcdDevice= 9.00
[35283.604913] usb 3-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[35283.604922] usb 3-1: Product: C232HM-DDHSL-0
[35283.604927] usb 3-1: Manufacturer: FTDI
[35283.604931] usb 3-1: SerialNumber: FT1UGJKF
[35283.616827] ftdi_sio 3-1:1.0: FTDI USB Serial Device converter detected
[35283.616989] usb 3-1: Detected FT232H
[35283.625374] usb 3-1: FTDI USB Serial Device converter now attached to ttyUSB0
```

# Software Setup

Set up tpm2_server (modified from [[4]](#4)):
```
$ sudo apt install libftdi-dev libssl-dev

$ git clone https://chromium.googlesource.com/chromiumos/third_party/tpm2 ~/tpm2
$ cd ~/tpm2
$ git checkout a77bf0779e1005c9fd840955193ac7257d67bc05

$ git clone https://github.com/wxleong/tpm2_server ~/tpm2_server
$ cd ~/tpm2_server
$ git checkout develop-tcti-sock
$ make -j${nproc}
```

Set up tpm2-tss:
```
$ git clone https://github.com/wxleong/tpm2-usb ~/tpm2-usb
$ git clone https://github.com/tpm2-software/tpm2-tss ~/tpm2-tss
$ cd ~/tpm2-tss
$ git checkout 3.2.0
$ git am < ~/tpm2-usb/patch/implement-tss2-tcti-sock.patch
$ ./bootstrap
$ ./configure
$ make -j${nproc}
$ sudo make install
$ sudo ldconfig
```

Set up tpm2-tools:
```
$ git clone https://github.com/tpm2-software/tpm2-tools ~/tpm2-tools
$ cd ~/tpm2-tools
$ git checkout 5.2
$ ./bootstrap
$ ./configure
$ make -j${nproc}
$ sudo make install
```

# Testing

Before you continue, remember to plug in the USB.

Start the tpm2_server and keep it running, **do not terminate it**:
```
$ cd ~/build/tpm2_server
$ ./ntpm -h
./ntpm: invalid option -- 'h'
 Command line options:
  -d[d]     - enable debug tracing (more d's - more debug)
  -f NUM    - ftdi clock frequency
  -p NUM    - port number
  -s        - use simulator instead of the USB interface
$ sudo ./ntpm -d -f 10000000
Opening socket on port 9883
Starting MPSSE at 10000 kHz
Connected to device vid:did:rid of 15d1:001b:10

Waiting for new connection...
```

Test the USB TPM with tpm2-tools:
```
# Connect to the tpm2_server using the tcti-sock interface
$ export TPM2TOOLS_TCTI="sock:host=localhost,port=9883"

$ tpm2_startup -c
$ tpm2_getrandom 8 --hex
```

What else can you do? Checkout [[8]](#8).

# References

<a id="1">[1] https://www.infineon.com/cms/en/product/evaluation-boards/iridium9670-tpm2.0-linux/</a> <br>
<a id="2">[2] https://ftdichip.com/products/c232hm-ddhsl-0-2/</a> <br>
<a id="3">[3] https://ftdichip.com/wp-content/uploads/2021/12/DS_C232HM_MPSSE_CABLE.pdf</a> <br>
<a id="4">[4] https://github.com/vbendeb/tpm2_server</a> <br>
<a id="5">[5] https://github.com/tpm2-software/tpm2-tss</a> <br>
<a id="6">[6] https://github.com/wxleong/optiga-tpm-2021-e2-training-day1/tree/master/ubuntu#install-dependencies</a> <br>
<a id="7">[7] https://github.com/wxleong/optiga-tpm-2021-e2-training-day1/tree/master/raspberry#install-dependencies</a> <br>
<a id="8">[8] https://github.com/wxleong/tpm2-cmd-ref</a> <br>

# License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
