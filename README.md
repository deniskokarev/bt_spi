Experiments with Host and BT controller boards over SPI
==============================================

This is an experiment of using few boards as BT Hosts:
- nucleo_f401re
- nucleo_g0b1re
- nucleo_l476rg
- mimxrt685_evk_cm33

and 2 boards as BT Controllers:
- x_nucleo_idb05a2 (shield)
- nrf52840dk_nrf52840

connected over SPI


*All STM32 boards work well, except `nucleo_l476rg` works only with x_nucleo_idb05a2 shield. It doesn't want to work with nRF controller. There must be some place where BLUNRG config options is set for entire board. Need to find out where*

--------------------

In order to proceed install Zephyr as per [Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)
using virtual env.

**NB!!! it is very important to use `v3.3-branch`**

After `west init` make sure to do
- `cd ~/zephyrproject`
- `git switch v3.3-branch`
- `west update`

The `main` is known to have problems with GPIO/SPI


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

STM32 Nucleo F401R Host with STM X-NUCLEO IDB05A2
----------------------------------------------

### Host with shield

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

### Host

Build perepheral_dis host example

```
west build -b nucleo_f401re -s $ZEPHYR_BASE/samples/bluetooth/peripheral_dis -d host_build -- -DBOARD_ROOT=$PWD -DSHIELD=red -DCONF_FILE=$PWD/peripheral_dis.prj.conf
```

Flash
```
west flash -d host_build
```

### Controller

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


### Host

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

Pinout is almost identical between stm host, shield and nrf controller.

- CTRL RESET is sent from host to reset the controller
- EXTI is raised when controller has data for host
- 4 SPI wires as usual


**The STM host and nRF controller pinouts are covered by `yellow` shield under `boards`**

**Also matches `x_nucleo_idb05a2` shield**

|  Function  | Arduino Pin | STM32 Pin  | Active Polarity |
|------------|-------------|------------|-----------------|
| CTRL RESET | D7          | PA8        | Lo              |
| EXTI       | A0          | PA0        | Hi              |
| SPI-CS     | A1          | PA1        | Lo              |
| SPI-MISO   | D12         | PA6        | Hi              |
| SPI-MOSI   | D11         | PA7        | Hi              |
| SPI-CLK    | D3          | PB3        | Hi              |


nrf52840dk_nrf52840 has the same pinout, except extra D7 -> P0.18 reset pin


**mimxrt685_evk_cm33 host pinout is covered by `red` shield under `boards`**

|  Function  | Arduino Pin | RT685 pin     |
|------------|-------------|---------------|
| CTRL RESET | D7          | P1_8          |
| EXTI       | D8          | P1_9          | 
| SPI-CS     | D10         | P1_6_SPI_SSO  |
| SPI-MISO   | D12         | P1_5_SPI_MISO |
| SPI-MOSI   | D11         | P1_4_SPI_MOSI |
| SPI-CLK    | D13         | P1_3_SPI_CLK  |


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
