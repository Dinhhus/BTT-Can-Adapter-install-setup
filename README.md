# install requirements
sudo apt-get install cmake gcc-arm-none-eabi  

# clone repo  
git clone https://github.com/LeVan102/CandleLight_FW.git  
cd CandleLight_FW  

# create cmake toolchain  
mkdir build  
cd build  
cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake/gcc-arm-none-eabi-8-2019-q3-update.cmake  

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