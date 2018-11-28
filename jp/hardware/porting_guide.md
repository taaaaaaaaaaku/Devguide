# Flight Controller Porting Guide

This topic is for developers who want to port PX4 to work with *new* flight controller hardware.

## PX4 Architecture

PX4 consists of two main layers: The [board support and middleware layer](../middleware/README.md) on top of the host OS (NuttX, Linux or any other POSIX platform like Mac OS), and the applications (Flight Stack in [src/modules](https://github.com/PX4/Firmware/tree/master/src/modules)\).  Please reference the [PX4 Architectural Overview](../concept/architecture.md) for more information.

This guide is focused only on the host OS and middleware as the applications/flight stack will run on any board target.

## Flight Controller Configuration File Layout

Board startup and configuration files are located under [/boards](https://github.com/PX4/Firmware/tree/master/boards/) in each board's vendor-specific directory (i.e. **boards/_VENDOR_/_MODEL_/**)).

For example, for FMUv5:
* (All) Board-specific files: [/boards/px4/fmu-v5](https://github.com/PX4/Firmware/tree/master/boards/px4/fmu-v5). 
* Build configuration: [/boards/px4/fmu-v5/default.cmake](https://github.com/PX4/Firmware/blob/master/boards/px4/fmu-v5/default.cmake).
  
In addition there are several groups of configuration files for each board located throughout the code base:
* The boot file system (startup script) is located in: [ROMFS/px4fmu\_common](https://github.com/PX4/Firmware/tree/master/ROMFS/px4fmu_common)
* Driver files are located in: [src/drivers](https://github.com/PX4/Firmware/tree/master/src/drivers).

## Host Operating System Configuration

This section describes the purpose and location of the configuration files required for each supported host operating system to port them to new flight controller hardware.

### NuttX

In order to port PX4 on NuttX to a new hardware target, that hardware target must be supported by NuttX.  The NuttX project maintains an excellent [porting guide](http://www.nuttx.org/Documentation/NuttxPortingGuide.html) for porting NuttX to a new computing platform.

For all NuttX based flight controllers (e.g. the Pixhawk series) the OS is loaded as part of the application build.

The configuration files for all boards, including linker scripts and other required settings, are located under [/boards](https://github.com/PX4/Firmware/tree/master/boards/) in a vendor- and board-specific directory (i.e. **boards/_VENDOR_/_MODEL_/**)).

The following example uses FMUv5 as it is a recent [reference configuration](../debug/reference-design.md) for NuttX based flight controllers:
* Running `make px4_fmu-v5_default` from the **Firmware** directory will build the FMUv5 config
* The base FMUv5 configuration files are located in: [/boards/px4/fmu-v5](https://github.com/PX4/Firmware/tree/master/boards/px4/fmu-v5).
* Board specific header: [/boards/px4/fmu-v5/nuttx-config/include/board.h](https://github.com/PX4/Firmware/blob/master/boards/px4/fmu-v5/nuttx-config/include/board.h). 
* NuttX OS config (created with Nuttx menuconfig): [/boards/px4/fmu-v5/nuttx-config/nsh/defconfig](https://github.com/PX4/Firmware/blob/master/boards/px4/fmu-v5/nuttx-config/nsh/defconfig).
* Build configuration: [PX4/Firmware/boards/px4/fmu-v5/default.cmake](https://github.com/PX4/Firmware/blob/master/boards/px4/fmu-v5/default.cmake).

The function of each of these files, and perhaps more, will need to be duplicated for a new flight controller board.

#### NuttX Menuconfig

If you need to modify the NuttX OS configuration, you can do this via [menuconfig](https://bitbucket.org/nuttx/nuttx) using the PX4 shortcuts:
```sh
make px4_fmu-v5_default menuconfig
make px4_fmu-v5_default qconfig
```

For fresh installs of PX4 onto Ubuntu using [ubuntu_sim_nuttx.sh](https://raw.githubusercontent.com/PX4/Devguide/master/build_scripts/ubuntu_sim_nuttx.sh) you will also need to install *kconfig* tools from [NuttX tools](https://bitbucket.org/nuttx/tools/src/master/).

> **Note** The following steps are not required if using the [px4-dev-nuttx](https://hub.docker.com/r/px4io/px4-dev-nuttx/) docker container or have installed to macOS using our normal instructions (as these include`kconfig-mconf`). 

Run the following commands from any directory:
```sh
git clone https://bitbucket.org/nuttx/tools.git
cd tools/kconfig-frontends
sudo apt install gperf
./configure --enable-mconf --disable-nconf --disable-gconf --enable-qconf --prefix=/usr
make
sudo make install
```

The `--prefix=/usr` is essential as it determines the specific installation location where PX4 is hardcoded to look for `kconfig-tools`.
The `--enable-mconf` and `--enable-qconf` options will enable the `menuconfig` and `qconfig` options respectively.

To run `qconfig` you may need to install additional Qt dependencies.


### Linux

Linux boards do not include the OS and kernel configuration. These are already provided by the Linux image available for the board (which needs to support the inertial sensors out of the box).

* [cmake/configs/posix\_rpi\_cross.cmake](https://github.com/PX4/Firmware/blob/master/cmake/configs/posix_rpi_cross.cmake) - RPI cross-compilation.

## Middleware Components and Configuration

This section describes the various middleware components, and the configuration file updates needed to port them to new flight controller hardware.

### QuRT / Hexagon

* The start script is located in [posix-configs/](https://github.com/PX4/Firmware/tree/master/posix-configs).
* The OS configuration is part of the default Linux image (TODO: Provide location of LINUX IMAGE and flash instructions).
* The PX4 middleware configuration is located in [src/drivers/boards](https://github.com/PX4/Firmware/tree/master/src/drivers/boards). TODO: ADD BUS CONFIG
* Drivers: [DriverFramework](https://github.com/px4/DriverFramework).
* Reference config: Running `make eagle_default` builds the Snapdragon Flight reference config.

## Related Information

* [Device Drivers](../middleware/drivers.md) - How to support new peripheral hardware (device drivers)
* [Building the Code](../setup/building_px4.md) - How to build source and upload firmware 
* Supported Flight Controllers:
  * [Autopilot Hardware](https://docs.px4.io/en/flight_controller/) (PX4 User Guide)
  * [Supported boards list](https://github.com/PX4/Firmware/#supported-hardware) (Github)
* [Supported Peripherals](https://docs.px4.io/en/peripherals/) (PX4 User Guide)


## RC UART Wiring Recommendations

It is generally recommended to connect RC via separate RX and TX pins to the microcontroller. 
If however RX and TX are connected together, the UART has to be put into singlewire mode to prevent any contention. 
This is done via board config and manifest files. 
One example is [px4fmu-v5](https://github.com/PX4/Firmware/blob/master/src/drivers/boards/px4fmu-v5/manifest.c).
