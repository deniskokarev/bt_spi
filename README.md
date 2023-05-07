Experiments with Host and BT controller boards over SPI
==============================================

Install Zephyr as per [Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)
using virtual env.

Install necessary dependencies, such as
```
brew install stlink
```
and [nrf-command-line-tools](https://www.nordicsemi.com/Products/Development-tools/nrf-command-line-tools/download)


Set Zephyr dev env by
```
source ./zephyr.env
```

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

Clean
----------------------------------------------
```
rm -rf host_build ctrl_build
```


TODO
----------------------------------------------

For some reason Host is always repeating logs entries twice
