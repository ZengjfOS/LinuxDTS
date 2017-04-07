# Linux设备树学习笔记

## 一、参考文档：

* [浅析Linux DeviceTree](https://lonzoc.gitbooks.io/device-tree-guide/content/devicetree_basic.html)
* [Device Tree Usage](http://elinux.org/Device_Tree_Usage)
* [Device Tree（二）：基本概念](http://www.wowotech.net/device_model/dt_basic_concept.html)
* [i.MX6 Device Tree customization](https://boundarydevices.com/mx6-device-tree-customization/)
* [linux DTS　分析](http://blog.csdn.net/xmzzy2012/article/details/49514951)

## 二、文件依赖关系

* kernel_imx/arch/arm/boot/dts/:
  * imx6dl-sabresd.dts:
    * #include "imx6dl.dtsi"
      * #include <dt-bindings/interrupt-controller/irq.h>
      * #include "imx6dl-pinfunc.h"
      * #include "imx6qdl.dtsi"
        * #include <dt-bindings/clock/imx6qdl-clock.h>
        * #include <dt-bindings/gpio/gpio.h>
        * #include "skeleton.dtsi"
    * #include "imx6qdl-sabresd.dtsi"
      * #include <dt-bindings/input/input.h>

## 三、依赖文件说明

### 3.1 imx6dl-sabresd.dts

* 定义root节点的module和compatible。
> model = "Freescale i.MX6 DualLite SABRE Smart Device Board";  
> compatible = "fsl,imx6dl-sabresd", "fsl,imx6dl";
* 修改针对imx6dl-sabresd一些特定的设定。

### 3.2 imx6dl.dtsi

* 描述了2个CPU的信息；
* 其soc节点描述了IMX6SDLRM.pdf文档中213也，ARM Platform Memory Map中相关设备信息；

### 3.3 imx6qdl.dtsi

* 芯片默认label是1开始算的，将其重新从0开始计算；
* 定义中断控制器信息；
* 其soc节点描述了IMX6SDLRM.pdf文档中213也，ARM Platform Memory Map中相关设备信息；

### 3.4 imx6qdl-sabresd.dtsi

* 主要是一些常用外设的设定、注册。

## 四、Code Hacking

跟踪源代码的时候采用Linux 3.14.52版本内核，属于Android 5.1的内核源代码。

* [i.MX6 dts gpio-keys hacking](hacking/gpio-keys.md)
