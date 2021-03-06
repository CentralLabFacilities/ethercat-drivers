EtherCAT  drivers
================

This package contains EtherCAT drivers to integrate with [IgH EtherCAT Master for Linux](http://etherlab.org/en/ethercat/).

For now this only contains **r8169** driver, for kernel **3.8.x**, **3.10.x** and **3.14.x**. They are used to control the Meka robot at Ensta ParisTech at 1Khz with [**rtai 4.0**](https://www.rtai.org/).

## Installation

#### Download EtherCAT 1.5.2
```bash
wget http://etherlab.org/download/ethercat/ethercat-1.5.2.tar.bz2
tar xfvj ethercat-1.5.2.tar.bz2
```

#### Get the drivers
For 3.14 kernel:
```bash
cd ethercat-1.5.2/devices
wget https://raw.githubusercontent.com/ahoarau/ethercat-drivers/master/r8169-3.14-ethercat.c
```

```bash
cd ethercat-1.5.2
wget https://raw.githubusercontent.com/ahoarau/ethercat-drivers/master/r8169-3.8-3.10.patch
## Apply patch
patch -p1 < ethercat-drivers/r8169-3.8-3.10.patch

#### Build the EtherCAT master modules
Example with kernel 3.10 and driver r8169 (Meka robot).
```bash
/configure --enable-cycles --enable-r8169  --with-rtai-dir=/usr/realtime/ --disable-8139too --with-r8169-kernel=3.14
make -j$[$(nproc)+1] all modules
sudo make modules_install install
sudo depmod
```
#### Configure your environnement

```bash
sudo -s
echo KERNEL==\"EtherCAT[0-9]*\", MODE=\"0664\" > /etc/udev/rules.d/99-EtherCAT.rules
cp script/init.d/ethercat /etc/init.d/ethercat # to start the master
cp script/sysconfig/ethercat /etc/sysconfig/ethercat # the config file
exit
```
Then copy the mac address of the ethernet card you'd like to use (use ifconfig, on meka-man it's 00:01:2e:51:ba:b9) to MASTER0_DEVICE and set the driver your card should use to DEVICE_MODULES ( meka-mob uses r8169):

```bash
sudo nano /etc/sysconfig/ethercat
```
>In Meka's RTPC, the file looks like this:
```
MASTER0_DEVICE="00:01:2e:51:ba:b9"
DEVICE_MODULES="r8169"
```

#### Start EtherCAT master
Then when you have configured your environnement (see the install note on ethercat-1.5.2), you can start the driver:
```bash
sudo service ethercat restart
```
See output messages with dmesg:
```bash
dmesg
```

### Start EtherCAT master at startup
```
cd /opt/etherlab
sudo cp etc/init.d/ethercat /etc/init.d/
sudo chmod a+x /etc/init.d/ethercat
sudo update-rc.d ethercat start 51 S .
```

> Contact : Antoine Hoarau <hoarau.robotics@gmail.com>


