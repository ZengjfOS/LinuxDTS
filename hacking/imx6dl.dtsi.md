# i.MX6 imx6dl.dtsi hakcing

## 原文解析

* 加载相关.h头文件、imx6qdl.dtsi文件

```
#include <dt-bindings/interrupt-controller/irq.h>
#include "imx6dl-pinfunc.h"
#include "imx6qdl.dtsi"
```

* 节点简图
  * / 
    * cpus 
      * cpu0: cpu@0 
      * cpu@1 
    * soc 
      * busfreq 
      * gpu@00130000 
      * ocrams: sram@00900000 
      * ocrams_ddr: sram@00904000
      * ocram: sram@00905000 
      * aips1: aips-bus@02000000 
        * vpu@02040000 
        * iomuxc: iomuxc@020e0000 
        * dcic2: dcic@020e8000 
        * pxp: pxp@020f0000 
      * aips2: aips-bus@02100000 
        * mipi_dsi: mipi@021e0000 
* 从节点简图可以看出，这里并没有实际注册具体的与外界通信的控制器，具体控制器的注册在[imx6qdl.dtsi](imx6qdl.dtsi.md)中注册；
* 定义两个CPU节点信息；
   * 根节点，必须有一个cpus的child node来描述系统中的CPU信息;
   * CPU的编址，我们用一个u32整数就可以描述，所以设定cpus node：
     * #address-cells = <1>，
     * #size-cells = <0>，
   * CPU的node属性可以很多，不过对于ARM，只是定义了compatible属性就OK了。
* 定义soc中部分外设信息；
  * 节点busfreq请参考：[Documentation/devicetree/bindings/arm/imx/busfreq-imx.txt](http://lxr.free-electrons.com/source/Documentation/devicetree/bindings/arm/imx/busfreq-imx.txt)

```
    / {
        cpus {
            #address-cells = <1>;
            #size-cells = <0>;
    
            cpu0: cpu@0 {
                compatible = "arm,cortex-a9";
                device_type = "cpu";
                /**
                 * <0> 
                 *     #address-cells = <1>     --> 0;
                 */
                reg = <0>;
                next-level-cache = <&L2>;
                operating-points = <
                    /* kHz    uV */
                    996000  1275000
                    792000  1175000
                    396000  1150000
                >;
                fsl,soc-operating-points = <
                    /* ARM kHz  SOC-PU uV */
                    996000    1175000
                    792000    1175000
                    396000    1175000
                >;
                clock-latency = <61036>; /* two CLK32 periods */
                clocks = <&clks IMX6QDL_CLK_ARM>,
                     <&clks IMX6QDL_CLK_PLL2_PFD2_396M>,
                     <&clks IMX6QDL_CLK_STEP>,
                     <&clks IMX6QDL_CLK_PLL1_SW>,
                     <&clks IMX6QDL_CLK_PLL1_SYS>,
                     <&clks IMX6QDL_PLL1_BYPASS>,
                     <&clks IMX6QDL_CLK_PLL1>,
                     <&clks IMX6QDL_PLL1_BYPASS_SRC> ;
                clock-names = "arm", "pll2_pfd2_396m", "step",
                          "pll1_sw", "pll1_sys", "pll1_bypass", "pll1", "pll1_bypass_src";
                arm-supply = <&reg_arm>;
                pu-supply = <&reg_pu>;
                soc-supply = <&reg_soc>;
            };
    
            cpu@1 {
                compatible = "arm,cortex-a9";
                device_type = "cpu";
                reg = <1>;
                next-level-cache = <&L2>;
            };
        };
    
        soc {
            busfreq {
                compatible = "fsl,imx_busfreq";
                clocks = <&clks IMX6QDL_CLK_PLL2_BUS>, <&clks IMX6QDL_CLK_PLL2_PFD2_396M>,
                    <&clks IMX6QDL_CLK_PLL2_198M>, <&clks IMX6QDL_CLK_ARM>,
                    <&clks IMX6QDL_CLK_PLL3_USB_OTG>, <&clks IMX6QDL_CLK_PERIPH>,
                    <&clks IMX6QDL_CLK_PERIPH_PRE>, <&clks IMX6QDL_CLK_PERIPH_CLK2>,
                    <&clks IMX6QDL_CLK_PERIPH_CLK2_SEL>, <&clks IMX6QDL_CLK_OSC>,
                    <&clks IMX6QDL_CLK_AXI_ALT_SEL>, <&clks IMX6QDL_CLK_AXI_SEL> ,
                    <&clks IMX6QDL_CLK_PLL3_PFD1_540M>;
                clock-names = "pll2_bus", "pll2_pfd2_396m", "pll2_198m", "arm", "pll3_usb_otg", "periph",
                    "periph_pre", "periph_clk2", "periph_clk2_sel", "osc", "axi_alt_sel", "axi_sel", "pll3_pfd1_540m";
                interrupts = <0 107 0x04>, <0 112 0x4>;
                interrupt-names = "irq_busfreq_0", "irq_busfreq_1";
                fsl,max_ddr_freq = <400000000>;
            };
    
            gpu@00130000 {
                compatible = "fsl,imx6dl-gpu", "fsl,imx6q-gpu";
                reg = <0x00130000 0x4000>, <0x00134000 0x4000>,
                      <0x0 0x0>;
                reg-names = "iobase_3d", "iobase_2d",
                        "phys_baseaddr";
                interrupts = <0 9 IRQ_TYPE_LEVEL_HIGH>,
                         <0 10 IRQ_TYPE_LEVEL_HIGH>;
                interrupt-names = "irq_3d", "irq_2d";
                clocks = <&clks IMX6QDL_CLK_OPENVG_AXI>, <&clks IMX6QDL_CLK_GPU3D_AXI>,
                     <&clks IMX6QDL_CLK_GPU2D_CORE>, <&clks IMX6QDL_CLK_GPU3D_CORE>,
                     <&clks IMX6QDL_CLK_DUMMY>;
                clock-names = "gpu2d_axi_clk", "gpu3d_axi_clk",
                          "gpu2d_clk", "gpu3d_clk",
                          "gpu3d_shader_clk";
                resets = <&src 0>, <&src 3>;
                reset-names = "gpu3d", "gpu2d";
                power-domains = <&gpc 1>;
            };
    
            ocrams: sram@00900000 {
                compatible = "fsl,lpm-sram";
                reg = <0x00900000 0x4000>;
                clocks = <&clks IMX6QDL_CLK_OCRAM>;
            };
    
            ocrams_ddr: sram@00904000 {
                compatible = "fsl,ddr-lpm-sram";
                reg = <0x00904000 0x1000>;
                clocks = <&clks IMX6QDL_CLK_OCRAM>;
            };
    
            ocram: sram@00905000 {
                compatible = "mmio-sram";
                reg = <0x00905000 0x1B000>;
                clocks = <&clks IMX6QDL_CLK_OCRAM>;
            };
    
            aips1: aips-bus@02000000 {
                vpu@02040000 {
                    iramsize = <0>;
                };
    
                iomuxc: iomuxc@020e0000 {
                    compatible = "fsl,imx6dl-iomuxc";
                };
    
                dcic2: dcic@020e8000 {
                    clocks = <&clks IMX6QDL_CLK_DCIC1 >,
                            <&clks IMX6QDL_CLK_DCIC2>; /* DCIC2 depend on DCIC1 clock in imx6dl*/
                    clock-names = "dcic", "disp-axi";
                };
    
                pxp: pxp@020f0000 {
                    compatible = "fsl,imx6dl-pxp-dma";
                    reg = <0x020f0000 0x4000>;
                    interrupts = <0 98 IRQ_TYPE_LEVEL_HIGH>;
                    clocks = <&clks IMX6QDL_CLK_IPU2>, <&clks IMX6QDL_CLK_DUMMY>;
                    clock-names = "pxp-axi", "disp-axi";
                    status = "disabled";
                };
    #if 0
                epdc: epdc@020f4000 {
                    compatible = "fsl,imx6dl-epdc";
                    reg = <0x020f4000 0x4000>;
                    interrupts = <0 97 IRQ_TYPE_LEVEL_HIGH>;
                    clocks = <&clks IMX6QDL_CLK_IPU2>, <&clks IMX6QDL_CLK_IPU2_DI1>;
                    clock-names = "epdc_axi", "epdc_pix";
                };
    
                lcdif: lcdif@020f8000 {
                    reg = <0x020f8000 0x4000>;
                    interrupts = <0 39 IRQ_TYPE_LEVEL_HIGH>;
                };
    #endif
            };
    
            aips2: aips-bus@02100000 {
                mipi_dsi: mipi@021e0000 {
                    compatible = "fsl,imx6dl-mipi-dsi";
                    reg = <0x021e0000 0x4000>;
                    interrupts = <0 102 0x04>;
                    gpr = <&gpr>;
                    clocks = <&clks IMX6QDL_CLK_HSI_TX>, <&clks IMX6QDL_CLK_VIDEO_27M>;
                    clock-names = "mipi_pllref_clk", "mipi_cfg_clk";
                    status = "disabled";
                };
    #if 0
                i2c4: i2c@021f8000 {
                    #address-cells = <1>;
                    #size-cells = <0>;
                    compatible = "fsl,imx6q-i2c", "fsl,imx21-i2c";
                    reg = <0x021f8000 0x4000>;
                    interrupts = <0 35 IRQ_TYPE_LEVEL_HIGH>;
                    clocks = <&clks IMX6DL_CLK_I2C4>;
                    status = "disabled";
                };
    #endif
            };
        };
    };
```

* label前面加&符号，相当于引用别处定义好的节点，在这里可以修改、增加属性

```
    &ecspi1 {
        dmas = <&sdma 3 7 1>, <&sdma 4 7 2>;
        dma-names = "rx", "tx";
    };
    
    &ecspi2 {
        dmas = <&sdma 5 7 1>, <&sdma 6 7 2>;
        dma-names = "rx", "tx";
    };
    
    &ecspi3 {
        dmas = <&sdma 7 7 1>, <&sdma 8 7 2>;
        dma-names = "rx", "tx";
    };
    
    &ecspi4 {
        dmas = <&sdma 9 7 1>, <&sdma 10 7 2>;
        dma-names = "rx", "tx";
    };
    
    &ldb {
        compatible = "fsl,imx6dl-ldb", "fsl,imx53-ldb";
    
        clocks = <&clks IMX6QDL_CLK_LDB_DI0>, <&clks IMX6QDL_CLK_LDB_DI1>,
             <&clks IMX6QDL_CLK_IPU1_DI0_SEL>, <&clks IMX6QDL_CLK_IPU1_DI1_SEL>,
             <&clks IMX6QDL_CLK_IPU2_DI0_SEL>,
             <&clks IMX6QDL_CLK_LDB_DI0_DIV_3_5>, <&clks IMX6QDL_CLK_LDB_DI1_DIV_3_5>,
             <&clks IMX6QDL_CLK_LDB_DI0_DIV_7>, <&clks IMX6QDL_CLK_LDB_DI1_DIV_7>,
             <&clks IMX6QDL_CLK_LDB_DI0_DIV_SEL>, <&clks IMX6QDL_CLK_LDB_DI1_DIV_SEL>;
        clock-names = "ldb_di0", "ldb_di1",
                  "di0_sel", "di1_sel",
                  "di2_sel",
                  "ldb_di0_div_3_5", "ldb_di1_div_3_5",
                  "ldb_di0_div_7", "ldb_di1_div_7",
                  "ldb_di0_div_sel", "ldb_di1_div_sel";
    };
```
