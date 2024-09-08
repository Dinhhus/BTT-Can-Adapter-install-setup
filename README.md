
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

# Cài Phần Firmware cho EBB42
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
