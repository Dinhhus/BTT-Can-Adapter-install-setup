
# install requirements

sudo apt-get install cmake gcc-arm-none-eabi  

# clone repo  

git clone https://github.com/LeVan102/CandleLight_FW.git  
cd CandleLight_FW  

# create cmake toolchain  
``
mkdir build  
cd build  
cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake/gcc-arm-none-eabi-8-2019-q3-update.cmake  
``
# compile firmware  
make budgetcan_fw  

# First, the adapter must boot in DFU mode. Please press the boot button and then connect the USB cable  

dfu-util -l  

# If the BTT U2C has booted in DFU mode, you can flash it with this command:  

make flash-budgetcan_fw  

# Now you only have to create the interface in the OS. to do this, create the file   

sudo nano /etc/network/interfaces.d/can0  
# add   
# bitrate = bitrate of firmware.  

allow-hotplug can0  
iface can0 can static  
    bitrate 500000   
    up ifconfig $IFACE txqueuelen 128  

crt x  
y   
enter  

# After a reboot, the can interface should be ready.  
sudo reboot.  
## Test thử xem chạy chưa
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0

# Cài Phần Firmware cho EBB42 (tham khảo tại)
https://docs.meteyou.wtf/btt-ebb/klipper/#flash-klipper-via-usb
## Configure Klipper firmware
Open the config interface of the Klipper firmware with following commands:

    cd ~/klipper
    
    make menuconfig

and set the following settings:

    Enable extra low-level configuration options: check
    
    Micro-controller Architecture: STMicroelectronics STM32
    
    Processor model: STM32G0B1
    
    Bootloader offset: No bootloader (without CanBoot)
    
    Bootloader offset: 8KiB bootloader (with CanBoot)

    Clock Reference: 8 MHz crystal
    
    Communication interface: CAN bus (on PB0/PB1)
    
    CAN bus speed: 500000
    
The result should look like this:

![alt text](https://github.com/Dinhhus/BTT-Can-Adapter-install-setup/blob/main/klipper-make-menuconfig.png)

use q for exit and y for save these settings.

Now clear the cache and compile the Klipper firmware:

    make clean
    
    make 

## Flash Klipper
There are two ways to flash the Klipper firmware to the EBB.

Flash the firmware via USB
Flash the firmware via CAN (recommended) (only with CanBoot)
### Flash Klipper via USB
This is the classic way to flash the firmware to the EBB.

Before you start the flashing process, disconnect the heater from the board!
First, you have to put the board into DFU mode. To do this, press and hold the boot button and then disconnect and reconnect the power supply, or press the reset button on the board. With the command 

    dfu-util -l


, you can check if the board is in DFU mode.

It should then look like this:

![alt text](https://github.com/Dinhhus/BTT-Can-Adapter-install-setup/blob/main/dfu-util_-l.svg)

If your board is in DFU mode, you can flash Klipper with the following command:


    dfu-util -a 0 -D ~/klipper/out/klipper.bin -s 0x08000000:mass-erase:force:leave


![alt text](https://github.com/Dinhhus/BTT-Can-Adapter-install-setup/blob/main/dfu-util_flash_klipper.svg)

### Flash Klipper via CAN
This is the recommended way to flash the firmware, when you use CanBoot on your board.

Find the UUID of your board:

    python3 ~/CanBoot/scripts/flash_can.py -i can0 -q

The output should look like this: 

![alt text](https://github.com/Dinhhus/BTT-Can-Adapter-install-setup/blob/main/klipper_query_can.svg)

With the UUID you have just read, you can now flash the board with:


    python3 ~/CanBoot/scripts/flash_can.py -f ~/klipper/out/klipper.bin -i can0 -u <uuid>

![alt text](https://github.com/Dinhhus/BTT-Can-Adapter-install-setup/blob/main/canboot_flash_klipper.svg)

Add the MCU in Klipper
Finally, you can add the board to your Klipper printer.cfg with its UUID:

printer.cfg


    [mcu EBB]

    canbus_uuid: <uuid>

    #embedded temperature sensor

    [temperature_sensor EBB]

    sensor_type: temperature_mcu

    sensor_mcu: EBB

    min_temp: 0

    max_temp: 100



If you don't know the UUID of your EBB, you can read it out with the following command:

    ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0

The output should look like this:

![alt text](https://github.com/Dinhhus/BTT-Can-Adapter-install-setup/blob/main/klipper_query_can.svg)


