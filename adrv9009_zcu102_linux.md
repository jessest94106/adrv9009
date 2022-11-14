# ADRV9009+ZCU102 on PYNQ
## Requirement

+ linux ubuntu 18.04
+ petalinux2019.1
+ Vivado 2019.1
+ ADI hdl-2019_r2
+ meta-xilinx-pynq in meta-xilinx-rel-v2019.2 (only using meta-xilinx-pynq here)
+ meta-adi-2019_R2
+ pylibiio-0.23.1
+ pyadi-iio-0.0.13
:::warning
Make sure all the version are matching, or it's highly suggested to follow the setup above.
:::

## WorkFlow of Building ADI supported PYNQ for ZCU102 and ADRV9009

+ petalinux-create -t project --template zynqMP --name ADIpynq2

```
Do not use "petalinux-create -t project --name pynq1 -s xilinx-zcu102-v2019.2-final.bsp"
```
+ cd ADIpynq2
+ petalinux-config --get-hw-description=~/Desktop/hdl-2019_r2/projects/adrv9009/zcu102/adrv9009_zcu102.sdk

```
Do not turn on FPGA Manger. It's already included in meta-adi.
```

+ user-layer
    + ~/Desktop/meta-adi-2019_R2/meta-adi-core
    + ~/Desktop/meta-adi-2019_R2/meta-adi-xilinx
    + ~/Desktop/meta-xilinx-pynq
+ Subsystem AUTO Hardware Settings
    + Advanced bootable images storage Settings
        + u-boot env partition setting image storage media=primary sd
+ image packaging configuration
    + root filesystem type=sd card
    
+ petalinux-config -c kernel
    + Device Drivers
        + Staging Drivers
            + Xilinx APF Accelerator driver
                + Xilinx APF DMA engines
                
+ project-spec/meta-user/conf/petalinuxbsp.conf
    add"KERNEL_DTB="zynqmp-zcu102-rev10-adrv9009-jesd204-fsm"
    
+ project-spec/meta-user/recipes-core/images
```shell=
IMAGE_INSTALL_append = " python3-pynq"
IMAGE_INSTALL_append = " libiio"
IMAGE_INSTALL_append = " libiio-tests"
IMAGE_INSTALL_append = " libiio-iiod"
IMAGE_INSTALL_append = " fru-tools"
IMAGE_INSTALL_append = " libad9361-iio"
IMAGE_INSTALL_append = " jesd-status"
IMAGE_INSTALL_append = " adrv9009-zu11eg-fan-control"
```
+ petalinux-config -c rootfs
    + Filesystem Packages
        + base
            + tar 
    + misc
        + packagegroup-core-buildessential
        + python3
    + User packages
        Enable all.

+ copy xlnk bsp into file folder
+ add following into "/project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi"
``` shell=
xlnk {
  compatible = "xlnx,xlnk-1.0";
};
```

+ cd build
+ petalinux-build
+ petalinux-package --boot --fsbl --fpga --u-boot
