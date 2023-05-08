Experiments with Host and BT controller boards over SPI
==============================================

Install Zephyr as per [Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)
using virtual env.

Install necessary dependencies, such as
```
brew install stlink
brew install openocd
```
and [nrf-command-line-tools](https://www.nordicsemi.com/Products/Development-tools/nrf-command-line-tools/download)


Set Zephyr dev env by
```
source ./zephyr.env
```

Have tried 3 STM32 boards as controllers:
- nucleo_f401re
- nucleo_g0b1re
- nucleo_l476rg (doesn't work with nRF controller)

The all work well, except `nucleo_l476rg` works only with x_nucleo_idb05a2 shield. It doesn't want to work with nRF controller. There must be some place where BLUNRG config options is set for entire board. Need to find out where.

STM32 Nucleo F401R Host with STM X-NUCLEO IDB05A2
----------------------------------------------

# Host with shield
Build

```
west build -b nucleo_f401re -s $ZEPHYR_BASE/samples/bluetooth/peripheral_dis -d host_build -- -DBOARD_ROOT=$PWD -DSHIELD=x_nucleo_idb05a2 -DCONF_FILE=$PWD/peripheral_dis.prj.conf
```

Flash
```
west flash -d host_build
```

STM32 Nucleo F401R Host with nRF52840DK Controller
----------------------------------------------

# Host
Build perepheral_dis host example

```
west build -b nucleo_f401re -s $ZEPHYR_BASE/samples/bluetooth/peripheral_dis -d host_build -- -DBOARD_ROOT=$PWD -DSHIELD=red -DCONF_FILE=$PWD/peripheral_dis.prj.conf
```

Flash
```
west flash -d host_build
```

# Controller
Build hci_spi controller example

```
west build -b nrf52840dk_nrf52840 -s $ZEPHYR_BASE/samples/bluetooth/hci_spi -d ctrl_build -- -DBOARD_ROOT=$PWD -DSHIELD=green -DCONF_FILE=$PWD/hci_spi.prj.conf
```

Flash
```
west flash -d ctrl_build
```

NXP RT685 EVK Host with nRF52840DK Controller
----------------------------------------------

It works!!!

**NB!** must switch JP12 pin to enable 3.3V output [MIMXRT685 EVK User Guide](https://www.mouser.com/pdfDocs/NXP_MIMXRT685-EVK_UG.pdf)


# Host
Build perepheral_dis host example

```
west build -b mimxrt685_evk_cm33 -s $ZEPHYR_BASE/samples/bluetooth/peripheral_dis -d host_build -- -DBOARD_ROOT=$PWD -DSHIELD=red -DCONF_FILE=$PWD/peripheral_dis.prj.conf
```

Flash
```
west flash -d host_build
```

Clean
----------------------------------------------
Every time you change the board type you must purge the build directory

```
rm -rf host_build ctrl_build
```

Pinout
----------------------------------------------

Pinout almost identical between host and controller

STM Host
```
(A0)  PA0   EXTI		active Hi
(A1)  PA1   SPI CS		active Lo
(D12) PA6   MISO		active Hi
(D11) PA7   MOSI		active Hi
(D6)  PB10  RESET		active Lo
(D3)  PB3   SPI-SCK		active Hi
```

Only nRF Controller has an extra wire from (D6) to P0.18 reset pin

Debug
----------------------------------------------

In peripheral_dis.prj.conf enable
```
CONFIG_BT_HCI_DRIVER_LOG_LEVEL_DBG=y
```
to see what's going with Host. It's not recommended to leave it on, as there may be crashes associated with that option.


To find the crash line from address like:
```
<err> os: Halting system
...
<err> os: Faulting instruction address (r15/pc): 0x180088a6
```

use `addr2line` like so:
```
% arm-none-eabi-addr2line -e host_build/zephyr/zephyr.elf 0x180088a6
/Users/dkv/zephyrproject/zephyr/subsys/bluetooth/host/hci_core.c:332 (discriminator 3)
```

To debug the host after flashing:
```
west debugserver -d host_build
```

Install [Cortex-Debug](https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug) for VS Code

Then use the launch.json entry similar to this:
```
        {
            "cwd": "${workspaceFolder}",
            "executable": "/Users/dkv/work/arm/bt_spi/host_build/zephyr/zephyr.elf",
            "name": "Attach to BT Host",
            "request": "attach",
            "type": "cortex-debug",
            "servertype": "external",
            "runToEntryPoint": "main",
            "showDevDebugOutput": "none",
            "gdbPath": "/Applications/ARM/bin/arm-none-eabi-gdb",
            "gdbTarget": ":2331",
        },
```

TODO
----------------------------------------------

For some reason Host is always repeating logs entries twice
