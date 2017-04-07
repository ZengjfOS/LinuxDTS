# i.MX6 imx6qdl-sabresd.dtsi hacking

## 原文解析

* 加载相关.h头文件

```
#include <dt-bindings/input/input.h>
```

* 节点简图
  * / 
    * aliases 
    * battery: max8903@0 
    * hannstar_cabc 
      * lvds0 
      * lvds1 
    * leds 
      *   charger-led 
    * memory: memory 
    * regulators 
      * reg_usb_otg_vbus: regulator@0 
      * reg_usb_h1_vbus: regulator@1 
      * reg_audio: regulator@2 
      * reg_mipi_dsi_pwr_on: mipi_dsi_pwr_on 
      * reg_sensor: regulator@3 
      * wlreg_on: fixedregulator@100 
      * bcmdhd_wlan_1: bcmdhd_wlan@0 
    * gpio-keys 
      * power 
      * volume-up 
      * volume-down 
    * sound 
    * sound-hdmi 
    * mxcfb1: fb@0 
    * mxcfb2: fb@1 
    * mxcfb3: fb@2 
    * mxcfb4: fb@3 
    * lcd@0 
    * pwm-backlight 
    * v4l2_cap_0 
    * v4l2_cap_1 
    * v4l2_out 
    * mipi_dsi_reset: mipi-dsi-reset 
    * minipcie_ctrl 
    * bt_rfkill 
    * ramoops_device 
    * caam_keyblob 
    * v4l2_cap_0 
    * v4l2_cap_1 
    * v4l2_out 
    * mipi_dsi_reset: mipi-dsi-reset 
* 后面主要是一些外设的注册。

```
    / {
        aliases {
            mxcfb0 = &mxcfb1;
            mxcfb1 = &mxcfb2;
            mxcfb2 = &mxcfb3;
            mxcfb3 = &mxcfb4;
        };
    #if 0
        battery: max8903@0 {
            compatible = "fsl,max8903-charger";
            pinctrl-names = "default";
            dok_input = <&gpio2 24 1>;
            uok_input = <&gpio1 27 1>;
            chg_input = <&gpio3 23 1>;
            flt_input = <&gpio5 2 1>;
            fsl,dcm_always_high;
            fsl,dc_valid;
            fsl,usb_valid;
            status = "okay";
        };
    #endif
        hannstar_cabc {
            compatible = "hannstar,cabc";
    
            lvds0 {
                gpios = <&gpio6 15 GPIO_ACTIVE_HIGH>;
                cabc-enable;    //active
            };
    
            lvds1 {
                gpios = <&gpio6 16 GPIO_ACTIVE_HIGH>;
                cabc-enable;    //active
            };
        };
    
        leds {
            compatible = "gpio-leds";
    
            charger-led {
                gpios = <&gpio1 2 0>;
                linux,default-trigger = "max8903-charger-charging";
                retain-state-suspended;
            };
        };
    
        memory: memory {
            reg = <0x10000000 0x40000000>;
        };
    
        regulators {
            compatible = "simple-bus";
            #address-cells = <1>;
            #size-cells = <0>;
    
            reg_usb_otg_vbus: regulator@0 {
                compatible = "regulator-fixed";
                reg = <0>;
                regulator-name = "usb_otg_vbus";
                regulator-min-microvolt = <5000000>;
                regulator-max-microvolt = <5000000>;
                gpio = <&gpio3 22 0>;
                enable-active-high;
            };
    
            reg_usb_h1_vbus: regulator@1 {
                compatible = "regulator-fixed";
                reg = <1>;
                regulator-name = "usb_h1_vbus";
                regulator-min-microvolt = <5000000>;
                regulator-max-microvolt = <5000000>;
                //gpio = <&gpio1 29 0>;
                //enable-active-high;
            };
    
            reg_audio: regulator@2 {
                compatible = "regulator-fixed";
                reg = <2>;
                regulator-name = "wm8962-supply";
                //gpio = <&gpio4 10 0>;
                gpio = <&gpio1 30 0>;    //MX6QDL_PAD_ENET_TXD0__GPIO1_IO30
                enable-active-high;
            };
    
            reg_mipi_dsi_pwr_on: mipi_dsi_pwr_on {
                compatible = "regulator-fixed";
                regulator-name = "mipi_dsi_pwr_on";
                gpio = <&gpio6 14 0>;
                enable-active-high;
            };
    
            reg_sensor: regulator@3 {
                compatible = "regulator-fixed";
                reg = <3>;
                regulator-name = "sensor-supply";
                regulator-min-microvolt = <3300000>;
                regulator-max-microvolt = <3300000>;
                gpio = <&gpio2 31 0>;
                startup-delay-us = <500>;
                enable-active-high;
            };
            wlreg_on: fixedregulator@100 {
                compatible = "regulator-fixed";
                regulator-min-microvolt = <5000000>;
                regulator-max-microvolt = <5000000>;
                regulator-name = "wlreg_on";
                gpio = <&gpio4 7 0>;
                startup-delay-us = <100>;
                enable-active-high;
            };
    
            bcmdhd_wlan_1: bcmdhd_wlan@0 {
                compatible = "android,bcmdhd_wlan";
                wlreg_on-supply = <&wlreg_on>;
            };
        };
    #if 0//disable
        gpio-keys {
            compatible = "gpio-keys";
            pinctrl-names = "default";
            pinctrl-0 = <&pinctrl_gpio_keys>;
    
            power {
                label = "Power Button";
                gpios = <&gpio3 29 1>;
                gpio-key,wakeup;
                linux,code = <KEY_POWER>;
            };
    
            volume-up {
                label = "Volume Up";
                gpios = <&gpio1 4 1>;
                gpio-key;
                linux,code = <KEY_VOLUMEUP>;
            };
    
            volume-down {
                label = "Volume Down";
                gpios = <&gpio1 5 1>;
                gpio-key;
                linux,code = <KEY_VOLUMEDOWN>;
            };
        };
    
        sound {
            compatible = "fsl,imx6q-sabresd-wm8962",
                   "fsl,imx-audio-wm8962";
            model = "wm8962-audio";
            cpu-dai = <&ssi2>;
            audio-codec = <&codec>;
            asrc-controller = <&asrc>;
            audio-routing =
                "Headphone Jack", "HPOUTL",
                "Headphone Jack", "HPOUTR",
                "Ext Spk", "SPKOUTL",
                "Ext Spk", "SPKOUTR",
                "MICBIAS", "AMIC",
                "IN3R", "MICBIAS",
                "DMIC", "MICBIAS",
                "DMICDAT", "DMIC",
                "CPU-Playback", "ASRC-Playback",
                "Playback", "CPU-Playback",
                "ASRC-Capture", "CPU-Capture",
                "CPU-Capture", "Capture";
            mux-int-port = <2>;
            mux-ext-port = <3>;
            hp-det-gpios = <&gpio7 8 1>;
            mic-det-gpios = <&gpio1 9 1>;
        };
    #endif
    
        sound-hdmi {
            compatible = "fsl,imx6q-audio-hdmi",
                     "fsl,imx-audio-hdmi";
            model = "imx-audio-hdmi";
            hdmi-controller = <&hdmi_audio>;
        };
    
        mxcfb1: fb@0 {
            compatible = "fsl,mxc_sdc_fb";
            disp_dev = "ldb";
            interface_pix_fmt = "RGB666";
            default_bpp = <16>;
            int_clk = <0>;
            late_init = <0>;
            status = "disabled";
        };
    
        mxcfb2: fb@1 {
            compatible = "fsl,mxc_sdc_fb";
            disp_dev = "hdmi";
            interface_pix_fmt = "RGB24";
            mode_str ="1920x1080M@60";
            default_bpp = <24>;
            int_clk = <0>;
            late_init = <0>;
            status = "disabled";
        };
    
        mxcfb3: fb@2 {
            compatible = "fsl,mxc_sdc_fb";
            disp_dev = "lcd";
            interface_pix_fmt = "RGB565";
            mode_str ="CLAA-WVGA";
            default_bpp = <16>;
            int_clk = <0>;
            late_init = <0>;
            status = "disabled";
        };
    
        mxcfb4: fb@3 {
            compatible = "fsl,mxc_sdc_fb";
            disp_dev = "ldb";
            interface_pix_fmt = "RGB666";
            default_bpp = <16>;
            int_clk = <0>;
            late_init = <0>;
            status = "disabled";
        };
    
        lcd@0 {
            compatible = "fsl,lcd";
            ipu_id = <0>;
            disp_id = <0>;
            default_ifmt = "RGB565";
            pinctrl-names = "default";
            pinctrl-0 = <&pinctrl_ipu1>;
            status = "okay";
        };
    
        pwm-backlight {
            compatible = "pwm-backlight";
            pwms = <&pwm2 0 50000>;    //default pwm1
            brightness-levels = <
                0  /*1  2  3  4  5  6*/  7  8  9
                10 11 12 13 14 15 16 17 18 19
                20 21 22 23 24 25 26 27 28 29
                30 31 32 33 34 35 36 37 38 39
                40 41 42 43 44 45 46 47 48 49
                50 51 52 53 54 55 56 57 58 59
                60 61 62 63 64 65 66 67 68 69
                70 71 72 73 74 75 76 77 78 79
                80 81 82 83 84 85 86 87 88 89
                90 91 92 93 94 95 96 97 98 99
                100
                >;
            default-brightness-level = <94>;
        };
    
        v4l2_cap_0 {
            compatible = "fsl,imx6q-v4l2-capture";
            ipu_id = <0>;
            csi_id = <0>;
            mclk_source = <0>;
            status = "okay";
        };
    
        v4l2_cap_1 {
            compatible = "fsl,imx6q-v4l2-capture";
            ipu_id = <0>;
            csi_id = <1>;
            mclk_source = <0>;
            status = "okay";
        };
    
        v4l2_out {
            compatible = "fsl,mxc_v4l2_output";
            status = "okay";
        };
    
        mipi_dsi_reset: mipi-dsi-reset {
            compatible = "gpio-reset";
            reset-gpios = <&gpio6 11 GPIO_ACTIVE_LOW>;
            reset-delay-us = <50>;
            #reset-cells = <0>;
        };
    
        minipcie_ctrl {
            power-on-gpio = <&gpio3 19 0>;
        };
    #if 0
        bt_rfkill {
                    compatible = "fsl,mxc_bt_rfkill";
                            bt-power-gpios = <&gpio1 2 0>;
                                    status ="okay";
        };
    #endif
        ramoops_device {
            compatible = "fsl,mxc_ramoops";
            record_size = <524288>; /*512K*/
            console_size = <262144>; /*256K*/
            ftrace_size = <262144>;  /*256K*/
            dump_oops = <1>;
            status = "okay";
        };
    
        caam_keyblob {
            compatible = "fsl,sec-v4.0-keyblob";
        };
    
        v4l2_cap_0 {
            compatible = "fsl,imx6q-v4l2-capture";
            ipu_id = <0>;
            csi_id = <0>;
            mclk_source = <0>;
            status = "okay";
        };
    
        v4l2_cap_1 {
            compatible = "fsl,imx6q-v4l2-capture";
            ipu_id = <0>;
            csi_id = <1>;
            mclk_source = <0>;
            status = "okay";
        };
    
        v4l2_out {
            compatible = "fsl,mxc_v4l2_output";
            status = "okay";
        };
    
        mipi_dsi_reset: mipi-dsi-reset {
            compatible = "gpio-reset";
            reset-gpios = <&gpio6 11 GPIO_ACTIVE_LOW>;
            reset-delay-us = <50>;
            #reset-cells = <0>;
        };
    };

```

* label前面加&符号，相当于引用别处定义好的节点，在这里可以修改、增加属性

```
    &audmux {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_audmux>;
        status = "okay";
    };
    
    &cpu0 {
        arm-supply = <&sw1a_reg>;
        soc-supply = <&sw1c_reg>;
    };
    
    &clks {
        fsl,ldb-di0-parent = <&clks IMX6QDL_CLK_PLL2_PFD0_352M>;
        fsl,ldb-di1-parent = <&clks IMX6QDL_CLK_PLL2_PFD0_352M>;
    };
    #if 0//disable
    &ecspi1 {
        fsl,spi-num-chipselects = <1>;
        cs-gpios = <&gpio4 9 0>;
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_ecspi1>;
        status = "okay";
    
        flash: m25p80@0 {
            #address-cells = <1>;
            #size-cells = <1>;
            compatible = "st,m25p32";
            spi-max-frequency = <20000000>;
            reg = <0>;
        };
    };
    #endif
    #if 1//enable
    &ecspi2 {
        fsl,spi-num-chipselects = <1>;
        cs-gpios = <&gpio2 20 0>;
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_ecspi2>;
        status = "okay";
        ar1021@0x00 {
            compatible = "microchip,ar1021-spi";
            spi-max-frequency = <200000>;
            spi-cpha;
            reg = <0>;
            interrupt-parent = <&gpio1>;
            interrupts = <17 IRQ_TYPE_EDGE_RISING>;
        };
    };
    #endif
    #if 1//active fec
    &fec {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_enet>;
        phy-mode = "rgmii";
        phy-reset-gpios = <&gpio1 25 0>;  
        fsl,magic-packet;
        status = "okay";
    };
    #endif
    &i2c1 {
        clock-frequency = <100000>;
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_i2c1>;
        status = "okay";
    #if 0
        codec: wm8962@1a {
            compatible = "wlf,wm8962";
            reg = <0x1a>;
            clocks = <&clks 201>;
            DCVDD-supply = <&reg_audio>;
            DBVDD-supply = <&reg_audio>;
            AVDD-supply = <&reg_audio>;
            CPVDD-supply = <&reg_audio>;
            MICVDD-supply = <&reg_audio>;
            PLLVDD-supply = <&reg_audio>;
            SPKVDD1-supply = <&reg_audio>;
            SPKVDD2-supply = <&reg_audio>;
            amic-mono;
            gpio-cfg = <
                0x0000 /* 0:Default */
                0x0000 /* 1:Default */
                0x0013 /* 2:FN_DMICCLK */
                0x0000 /* 3:Default */
                0x8014 /* 4:FN_DMICCDAT */
                0x0000 /* 5:Default */
            >;
           };
    
        mma8451@1c {
            compatible = "fsl,mma8451";
            reg = <0x1c>;
            position = <0>;
            vdd-supply = <&reg_sensor>;
            vddio-supply = <&reg_sensor>;
            interrupt-parent = <&gpio1>;
            interrupts = <18 8>;
            interrupt-route = <1>;
        };
    
        ov564x: ov564x@3c {
            compatible = "ovti,ov564x";
            reg = <0x3c>;
            pinctrl-names = "default";
            pinctrl-0 = <&pinctrl_ipu1_2>;
            clocks = <&clks IMX6QDL_CLK_CKO>;
            clock-names = "csi_mclk";
            DOVDD-supply = <&vgen4_reg>; /* 1.8v */
            AVDD-supply = <&vgen3_reg>;  /* 2.8v, on rev C board is VGEN3,
                            on rev B board is VGEN5 */
            DVDD-supply = <&vgen2_reg>;  /* 1.5v*/
            pwn-gpios = <&gpio1 16 1>;   /* active low: SD1_DAT0 */
            rst-gpios = <&gpio1 17 0>;   /* active high: SD1_DAT1 */
            csi_id = <0>;
            mclk = <24000000>;
            mclk_source = <0>;
        };
    #endif
    };
    
    &i2c2 {
        clock-frequency = <100000>;
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_i2c2>;
        status = "okay";
    
        hdmi: edid@50 {
            compatible = "fsl,imx6-hdmi-i2c";
            reg = <0x50>;
        };
    #if 0//disable
        max11801@48 {
            compatible = "maxim,max11801";
            reg = <0x48>;
            interrupt-parent = <&gpio3>;
            interrupts = <26 2>;
            work-mode = <1>;/*DCM mode*/
        };
    #endif
        pmic: pfuze100@08 {
            compatible = "fsl,pfuze100";
            reg = <0x08>;
    
            regulators {
                sw1a_reg: sw1ab {
                    regulator-min-microvolt = <300000>;
                    regulator-max-microvolt = <1875000>;
                    regulator-boot-on;
                    regulator-always-on;
                    regulator-ramp-delay = <6250>;
                };
    
                sw1c_reg: sw1c {
                    regulator-min-microvolt = <300000>;
                    regulator-max-microvolt = <1875000>;
                    regulator-boot-on;
                    regulator-always-on;
                    regulator-ramp-delay = <6250>;
                };
    
                sw2_reg: sw2 {
                    regulator-min-microvolt = <800000>;
                    regulator-max-microvolt = <3300000>;
                    regulator-boot-on;
                    regulator-always-on;
                    regulator-ramp-delay = <6250>;
                };
    
                sw3a_reg: sw3a {
                    regulator-min-microvolt = <400000>;
                    regulator-max-microvolt = <1975000>;
                    regulator-boot-on;
                    regulator-always-on;
                };
    
                sw3b_reg: sw3b {
                    regulator-min-microvolt = <400000>;
                    regulator-max-microvolt = <1975000>;
                    regulator-boot-on;
                    regulator-always-on;
                };
    
                sw4_reg: sw4 {
                    regulator-min-microvolt = <800000>;
                    regulator-max-microvolt = <3300000>;
                };
    
                swbst_reg: swbst {
                    regulator-min-microvolt = <5000000>;
                    regulator-max-microvolt = <5150000>;
                };
    
                snvs_reg: vsnvs {
                    regulator-min-microvolt = <1000000>;
                    regulator-max-microvolt = <3000000>;
                    regulator-boot-on;
                    regulator-always-on;
                };
    
                vref_reg: vrefddr {
                    regulator-boot-on;
                    regulator-always-on;
                };
    
                vgen1_reg: vgen1 {
                    regulator-min-microvolt = <800000>;
                    regulator-max-microvolt = <1550000>;
                };
    
                vgen2_reg: vgen2 {
                    regulator-min-microvolt = <800000>;
                    regulator-max-microvolt = <1550000>;
                };
    
                vgen3_reg: vgen3 {
                    regulator-min-microvolt = <1800000>;
                    regulator-max-microvolt = <3300000>;
                };
    
                vgen4_reg: vgen4 {
                    regulator-min-microvolt = <1800000>;
                    regulator-max-microvolt = <3300000>;
                    regulator-always-on;
                };
    
                vgen5_reg: vgen5 {
                    regulator-min-microvolt = <1800000>;
                    regulator-max-microvolt = <3300000>;
                    regulator-always-on;
                };
    
                vgen6_reg: vgen6 {
                    regulator-min-microvolt = <1800000>;
                    regulator-max-microvolt = <3300000>;
                    regulator-always-on;
                };
            };
        };
    #if 0
        ov564x_mipi: ov564x_mipi@3c { /* i2c2 driver */
            compatible = "ovti,ov564x_mipi";
            reg = <0x3c>;
            clocks = <&clks 201>;
            clock-names = "csi_mclk";
            DOVDD-supply = <&vgen4_reg>; /* 1.8v */
            AVDD-supply = <&vgen3_reg>;  /* 2.8v, rev C board is VGEN3
                            rev B board is VGEN5 */
            DVDD-supply = <&vgen2_reg>;  /* 1.5v*/
            pwn-gpios = <&gpio1 19 1>;   /* active low: SD1_CLK */
            rst-gpios = <&gpio1 20 0>;   /* active high: SD1_DAT2 */
            csi_id = <1>;
            mclk = <24000000>;
            mclk_source = <0>;
        };
    
        egalax_ts@04 {
            compatible = "eeti,egalax_ts";
            reg = <0x04>;
            interrupt-parent = <&gpio6>;
            interrupts = <8 2>;
            wakeup-gpios = <&gpio6 8 0>;
        };
    #endif
    };
    
    &i2c3 {
        clock-frequency = <100000>;
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_i2c3>;
        status = "okay";
    #if 0
        egalax_ts@04 {
            compatible = "eeti,egalax_ts";
            reg = <0x04>;
            interrupt-parent = <&gpio6>;
            interrupts = <7 2>;
            wakeup-gpios = <&gpio6 7 0>;
        };
    
        /* Add by qinzd 2016-11-09 */
        Goodix-TS@5d {
            compatible = "goodix,gt9xx";
            reg = <0x5d>;
            goodix,rst-gpio = <&gpio1 23 0>;
            goodix,irq-gpio = <&gpio6 7 0>;
        };
    #endif
        rtc-ds1307@68 {
            compatible = "dallas,ds1337";
            reg = <0x68>;
        };
    #if 0
        mag3110@0e {
            compatible = "fsl,mag3110";
            reg = <0x0e>;
            position = <1>;
            vdd-supply = <&reg_sensor>;
            vddio-supply = <&reg_sensor>;
            interrupt-parent = <&gpio3>;
            interrupts = <16 1>;
        };
    
        isl29023@44 {
            compatible = "fsl,isl29023";
            reg = <0x44>;
            rext = <499>;
            vdd-supply = <&reg_sensor>;
            interrupt-parent = <&gpio3>;
            interrupts = <9 2>;
        };
    #endif
    };
    #if 1
    &i2c_gpio {
        compatible = "i2c-gpio";
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_i2c4>;
        gpios =
            <&gpio1 29 0>, /* sda */
            <&gpio1 28 0>; /* scl */
        //i2c-gpio,sda-open-drain;
        //i2c-gpio,scl-open-drain;
        i2c-gpio,delay-us = <50>;    /* ~100 kHz */
        #address-cells = <1>;
        #size-cells = <0>;
        status = "okay";
        eeprom@50 {
            compatible = "at24,24c02";
            reg = <0x50>;
        };
    };
    #endif
    &iomuxc {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_hog>;
    
        imx6qdl-sabresd {
            pinctrl_hog: hoggrp {
                fsl,pins = <
                    MX6QDL_PAD_NANDF_D0__GPIO2_IO00 0x80000000
                    MX6QDL_PAD_NANDF_D1__GPIO2_IO01 0x80000000
                    MX6QDL_PAD_NANDF_D2__GPIO2_IO02 0x80000000
                    MX6QDL_PAD_NANDF_D3__GPIO2_IO03 0x80000000
                    MX6QDL_PAD_GPIO_0__CCM_CLKO1    0x130b0
                    MX6QDL_PAD_NANDF_CLE__GPIO6_IO07 0x80000000
                    MX6QDL_PAD_NANDF_ALE__GPIO6_IO08 0x80000000
                    //MX6QDL_PAD_ENET_TXD1__GPIO1_IO29 0x80000000
                    MX6QDL_PAD_EIM_D22__GPIO3_IO22  0x80000000
                    MX6QDL_PAD_ENET_CRS_DV__GPIO1_IO25 0x80000000
                    //MX6QDL_PAD_EIM_D26__GPIO3_IO26 0x80000000//for uart2
                    //MX6QDL_PAD_EIM_CS1__GPIO2_IO24 0x80000000//for spi2
                    MX6QDL_PAD_ENET_RXD0__GPIO1_IO27 0x80000000
                    MX6QDL_PAD_EIM_A25__GPIO5_IO02 0x80000000
                    MX6QDL_PAD_EIM_D23__GPIO3_IO23 0x80000000
                    MX6QDL_PAD_EIM_EB3__GPIO2_IO31 0x80000000
                    //MX6QDL_PAD_SD1_CMD__GPIO1_IO18 0x80000000
                    MX6QDL_PAD_SD1_CMD__PWM4_OUT 0x1b0b1
                    MX6QDL_PAD_EIM_D16__GPIO3_IO16 0x80000000
                    MX6QDL_PAD_SD3_RST__GPIO7_IO08    0x80000000
                    //MX6QDL_PAD_GPIO_9__GPIO1_IO09     0x80000000//for wdog1
                    MX6QDL_PAD_GPIO_9__WDOG1_B 0x80000000//WDOG1
                    MX6QDL_PAD_EIM_DA9__GPIO3_IO09 0x80000000
                    //MX6QDL_PAD_GPIO_1__WDOG2_B 0x80000000//for usb_otg_id
                    MX6QDL_PAD_GPIO_2__GPIO1_IO02 0x1b0b0
                    MX6QDL_PAD_NANDF_CS0__GPIO6_IO11 0x80000000
                    MX6QDL_PAD_NANDF_CS1__GPIO6_IO14 0x80000000
                    MX6QDL_PAD_NANDF_CS2__GPIO6_IO15 0x80000000
                    MX6QDL_PAD_NANDF_CS3__GPIO6_IO16 0x80000000
                    //MX6QDL_PAD_GPIO_1__WDOG2_B 0x80000000//for usb_otg_id
                    MX6QDL_PAD_GPIO_4__GPIO1_IO04 0x80000000//for uart3 mode select 0
                    MX6QDL_PAD_GPIO_5__GPIO1_IO05 0x80000000//for uart3 mode select 1
                    MX6QDL_PAD_GPIO_16__GPIO7_IO11 0x80000000//for uart3 mode terminate
                    MX6QDL_PAD_SD2_DAT0__GPIO1_IO15        0x80000000//gpio_in0
                    MX6QDL_PAD_SD2_DAT1__GPIO1_IO14        0x80000000//gpio_in1
                    MX6QDL_PAD_SD2_DAT2__GPIO1_IO13        0x80000000//gpio_in2
                    MX6QDL_PAD_SD2_DAT3__GPIO1_IO12        0x80000000//gpio_in3
                    MX6QDL_PAD_NANDF_D4__GPIO2_IO04        0x80000000//gpio_out0
                    MX6QDL_PAD_NANDF_D5__GPIO2_IO05        0x80000000//gpio_out1
                    MX6QDL_PAD_NANDF_D6__GPIO2_IO06        0x80000000//gpio_out2
                    MX6QDL_PAD_NANDF_D7__GPIO2_IO07        0x80000000//gpio_out3
                    MX6QDL_PAD_SD1_DAT1__GPIO1_IO17        0x80000000//touch_int
                >;
            };
    
            pinctrl_audmux: audmuxgrp {
                fsl,pins = <
                    MX6QDL_PAD_CSI0_DAT7__AUD3_RXD        0x130b0
                    MX6QDL_PAD_CSI0_DAT4__AUD3_TXC        0x130b0
                    MX6QDL_PAD_CSI0_DAT5__AUD3_TXD        0x110b0
                    MX6QDL_PAD_CSI0_DAT6__AUD3_TXFS        0x130b0
                >;
            };
    #if 0
            pinctrl_ecspi1: ecspi1grp {
                fsl,pins = <
                    MX6QDL_PAD_KEY_COL1__ECSPI1_MISO    0x100b1
                    MX6QDL_PAD_KEY_ROW0__ECSPI1_MOSI    0x100b1
                    MX6QDL_PAD_KEY_COL0__ECSPI1_SCLK    0x100b1
                >;
            };
    #endif
    #if 1
            pinctrl_ecspi2: ecspi2grp {
                fsl,pins = <
                    MX6QDL_PAD_EIM_LBA__ECSPI2_SS1    0x100b1
                    MX6QDL_PAD_EIM_OE__ECSPI2_MISO    0x100b1
                    MX6QDL_PAD_EIM_RW__ECSPI2_SS0    0x100b1
                    MX6QDL_PAD_EIM_CS0__ECSPI2_SCLK    0x100b1
                    MX6QDL_PAD_EIM_CS1__ECSPI2_MOSI    0x100b1
                >;
            };
    #endif
            pinctrl_enet: enetgrp {
                fsl,pins = <
                    MX6QDL_PAD_ENET_MDIO__ENET_MDIO        0x1b0b0
                    MX6QDL_PAD_ENET_MDC__ENET_MDC        0x1b0b0
                    MX6QDL_PAD_RGMII_TXC__RGMII_TXC        0x1b0b0
                    MX6QDL_PAD_RGMII_TD0__RGMII_TD0        0x1b0b0
                    MX6QDL_PAD_RGMII_TD1__RGMII_TD1        0x1b0b0
                    MX6QDL_PAD_RGMII_TD2__RGMII_TD2        0x1b0b0
                    MX6QDL_PAD_RGMII_TD3__RGMII_TD3        0x1b0b0
                    MX6QDL_PAD_RGMII_TX_CTL__RGMII_TX_CTL    0x1b0b0
                    MX6QDL_PAD_ENET_REF_CLK__ENET_TX_CLK    0x1b0b0
                    MX6QDL_PAD_RGMII_RXC__RGMII_RXC        0x1b0b0
                    MX6QDL_PAD_RGMII_RD0__RGMII_RD0        0x1b0b0
                    MX6QDL_PAD_RGMII_RD1__RGMII_RD1        0x1b0b0
                    MX6QDL_PAD_RGMII_RD2__RGMII_RD2        0x1b0b0
                    MX6QDL_PAD_RGMII_RD3__RGMII_RD3        0x1b0b0
                    MX6QDL_PAD_RGMII_RX_CTL__RGMII_RX_CTL    0x1b0b0
                    //MX6QDL_PAD_GPIO_16__ENET_REF_CLK    0x4001b0a8
                >;
            };
    
    #if 0
            pinctrl_enet_irq: enetirqgrp {
                fsl,pins = <
                    MX6QDL_PAD_GPIO_6__ENET_IRQ        0x000b1
                >;
            };
    #endif
    #if 0
            pinctrl_gpio_keys: gpio_keysgrp {
                fsl,pins = <
                    MX6QDL_PAD_EIM_D29__GPIO3_IO29 0x80000000
                    MX6QDL_PAD_GPIO_4__GPIO1_IO04  0x80000000
                    MX6QDL_PAD_GPIO_5__GPIO1_IO05  0x80000000
                >;
            };
    #endif
            pinctrl_hdmi_cec: hdmicecgrp {
                fsl,pins = <
                    MX6QDL_PAD_KEY_ROW2__HDMI_TX_CEC_LINE 0x1f8b0
                >;
            };
    
            pinctrl_hdmi_hdcp: hdmihdcpgrp {
                fsl,pins = <
                    MX6QDL_PAD_KEY_COL3__HDMI_TX_DDC_SCL 0x4001b8b1
                    MX6QDL_PAD_KEY_ROW3__HDMI_TX_DDC_SDA 0x4001b8b1
                >;
            };
    
            pinctrl_i2c1: i2c1grp {
                fsl,pins = <
                    MX6QDL_PAD_CSI0_DAT8__I2C1_SDA        0x4001b8b1
                    MX6QDL_PAD_CSI0_DAT9__I2C1_SCL        0x4001b8b1
                >;
            };
    
            pinctrl_i2c2: i2c2grp {
                fsl,pins = <
                    MX6QDL_PAD_KEY_COL3__I2C2_SCL        0x4001b8b1
                    MX6QDL_PAD_KEY_ROW3__I2C2_SDA        0x4001b8b1
                 >;
            };
    
            pinctrl_i2c3: i2c3grp {
                fsl,pins = <
                    MX6QDL_PAD_GPIO_3__I2C3_SCL        0x4001b8b1
                    MX6QDL_PAD_GPIO_6__I2C3_SDA        0x4001b8b1
                >;
            };
    #if 1
            pinctrl_i2c4: i2c4grp {
                fsl,pins = <
                    MX6QDL_PAD_ENET_TXD1__GPIO1_IO29    0x80000000    /* sda */
                    MX6QDL_PAD_ENET_TX_EN__GPIO1_IO28    0x80000000    /* scl */
                >;
            };
    #endif
            pinctrl_ipu1: ipu1grp {
                fsl,pins = <
                    MX6QDL_PAD_DI0_DISP_CLK__IPU1_DI0_DISP_CLK 0x10
                    MX6QDL_PAD_DI0_PIN15__IPU1_DI0_PIN15       0x10
                    MX6QDL_PAD_DI0_PIN2__IPU1_DI0_PIN02        0x10
                    MX6QDL_PAD_DI0_PIN3__IPU1_DI0_PIN03        0x10
                    MX6QDL_PAD_DI0_PIN4__IPU1_DI0_PIN04        0x80000000
                    MX6QDL_PAD_DISP0_DAT0__IPU1_DISP0_DATA00   0x10
                    MX6QDL_PAD_DISP0_DAT1__IPU1_DISP0_DATA01   0x10
                    MX6QDL_PAD_DISP0_DAT2__IPU1_DISP0_DATA02   0x10
                    MX6QDL_PAD_DISP0_DAT3__IPU1_DISP0_DATA03   0x10
                    MX6QDL_PAD_DISP0_DAT4__IPU1_DISP0_DATA04   0x10
                    MX6QDL_PAD_DISP0_DAT5__IPU1_DISP0_DATA05   0x10
                    MX6QDL_PAD_DISP0_DAT6__IPU1_DISP0_DATA06   0x10
                    MX6QDL_PAD_DISP0_DAT7__IPU1_DISP0_DATA07   0x10
                    MX6QDL_PAD_DISP0_DAT8__IPU1_DISP0_DATA08   0x10
                    MX6QDL_PAD_DISP0_DAT9__IPU1_DISP0_DATA09   0x10
                    MX6QDL_PAD_DISP0_DAT10__IPU1_DISP0_DATA10  0x10
                    MX6QDL_PAD_DISP0_DAT11__IPU1_DISP0_DATA11  0x10
                    MX6QDL_PAD_DISP0_DAT12__IPU1_DISP0_DATA12  0x10
                    MX6QDL_PAD_DISP0_DAT13__IPU1_DISP0_DATA13  0x10
                    MX6QDL_PAD_DISP0_DAT14__IPU1_DISP0_DATA14  0x10
                    MX6QDL_PAD_DISP0_DAT15__IPU1_DISP0_DATA15  0x10
                    MX6QDL_PAD_DISP0_DAT16__IPU1_DISP0_DATA16  0x10
                    MX6QDL_PAD_DISP0_DAT17__IPU1_DISP0_DATA17  0x10
                    MX6QDL_PAD_DISP0_DAT18__IPU1_DISP0_DATA18  0x10
                    MX6QDL_PAD_DISP0_DAT19__IPU1_DISP0_DATA19  0x10
                    MX6QDL_PAD_DISP0_DAT20__IPU1_DISP0_DATA20  0x10
                    MX6QDL_PAD_DISP0_DAT21__IPU1_DISP0_DATA21  0x10
                    MX6QDL_PAD_DISP0_DAT22__IPU1_DISP0_DATA22  0x10
                    MX6QDL_PAD_DISP0_DAT23__IPU1_DISP0_DATA23  0x10
                >;
            };
    #if 0
            pinctrl_ipu1_2: ipu1grp-2 { /* parallel camera */
                fsl,pins = <
                    MX6QDL_PAD_CSI0_DAT12__IPU1_CSI0_DATA12    0x80000000
                    MX6QDL_PAD_CSI0_DAT13__IPU1_CSI0_DATA13    0x80000000
                    MX6QDL_PAD_CSI0_DAT14__IPU1_CSI0_DATA14    0x80000000
                    MX6QDL_PAD_CSI0_DAT15__IPU1_CSI0_DATA15    0x80000000
                    MX6QDL_PAD_CSI0_DAT16__IPU1_CSI0_DATA16    0x80000000
                    MX6QDL_PAD_CSI0_DAT17__IPU1_CSI0_DATA17    0x80000000
                    MX6QDL_PAD_CSI0_DAT18__IPU1_CSI0_DATA18    0x80000000
                    MX6QDL_PAD_CSI0_DAT19__IPU1_CSI0_DATA19    0x80000000
                    MX6QDL_PAD_CSI0_DATA_EN__IPU1_CSI0_DATA_EN 0x80000000
                    MX6QDL_PAD_CSI0_PIXCLK__IPU1_CSI0_PIXCLK   0x80000000
                    MX6QDL_PAD_CSI0_MCLK__IPU1_CSI0_HSYNC      0x80000000
                    MX6QDL_PAD_CSI0_VSYNC__IPU1_CSI0_VSYNC     0x80000000
                >;
            };
    #endif
            pinctrl_pwm1: pwm1grp {
                fsl,pins = <
                    MX6QDL_PAD_SD1_DAT3__PWM1_OUT        0x1b0b1
                >;
            };
    #if 1
            pinctrl_pwm2: pwm2grp {
                fsl,pins = <
                    MX6QDL_PAD_SD1_DAT2__PWM2_OUT        0x1b0b1
                >;
            };
    #endif
    #if 1
            pinctrl_pwm4: pwm4grp {
                fsl,pins = <
                    MX6QDL_PAD_SD1_CMD__PWM4_OUT        0x1b0b1
                >;
            };
    #endif
            pinctrl_uart1: uart1grp {
                fsl,pins = <
                    MX6QDL_PAD_CSI0_DAT10__UART1_TX_DATA    0x1b0b1
                    MX6QDL_PAD_CSI0_DAT11__UART1_RX_DATA    0x1b0b1
                >;
            };
    
    #if 1
            pinctrl_uart2: uart2grp {
                fsl,pins = <
                    MX6QDL_PAD_EIM_D26__UART2_TX_DATA    0x1b0b1
                    MX6QDL_PAD_EIM_D27__UART2_RX_DATA    0x1b0b1
                >;
            };
    
            pinctrl_uart3: uart3grp {
                fsl,pins = <
                    MX6QDL_PAD_EIM_D24__UART3_TX_DATA    0x1b0b1
                    MX6QDL_PAD_EIM_D25__UART3_RX_DATA    0x1b0b1
                >;
            };
    
            pinctrl_uart4: uart4grp {
                fsl,pins = <
                    MX6QDL_PAD_KEY_COL0__UART4_TX_DATA    0x1b0b1
                    MX6QDL_PAD_KEY_ROW0__UART4_RX_DATA    0x1b0b1
                >;
            };
    #endif
    #if 0
            pinctrl_uart5_1: uart5grp-1 {
                fsl,pins = <
                    MX6QDL_PAD_KEY_COL1__UART5_TX_DATA    0x1b0b1
                    MX6QDL_PAD_KEY_ROW1__UART5_RX_DATA    0x1b0b1
                    MX6QDL_PAD_KEY_COL4__UART5_RTS_B    0x1b0b1
                    MX6QDL_PAD_KEY_ROW4__UART5_CTS_B    0x1b0b1
                >;
            };
    
            pinctrl_uart5dte_1: uart5dtegrp-1 {
                fsl,pins = <
                    MX6QDL_PAD_KEY_ROW1__UART5_TX_DATA    0x1b0b1
                    MX6QDL_PAD_KEY_COL1__UART5_RX_DATA    0x1b0b1
                    MX6QDL_PAD_KEY_ROW4__UART5_RTS_B    0x1b0b1
                    MX6QDL_PAD_KEY_COL4__UART5_CTS_B    0x1b0b1
                >;
            };
    #endif
    
            pinctrl_usbotg: usbotggrp {
                fsl,pins = <
                    //MX6QDL_PAD_ENET_RX_ER__USB_OTG_ID    0x17059
                    MX6QDL_PAD_GPIO_1__USB_OTG_ID    0x17059
                >;
            };
    #if 0
            pinctrl_usdhc2: usdhc2grp {
                fsl,pins = <
                    MX6QDL_PAD_SD2_CMD__SD2_CMD        0x17059
                    MX6QDL_PAD_SD2_CLK__SD2_CLK        0x10059
                    MX6QDL_PAD_SD2_DAT0__SD2_DATA0        0x17059
                    MX6QDL_PAD_SD2_DAT1__SD2_DATA1        0x17059
                    MX6QDL_PAD_SD2_DAT2__SD2_DATA2        0x17059
                    MX6QDL_PAD_SD2_DAT3__SD2_DATA3        0x17059
    //                MX6QDL_PAD_KEY_ROW0__GPIO4_IO07        0x13069    /* WL_REG_ON */
                >;
            };
    #endif
            pinctrl_usdhc3: usdhc3grp {
                fsl,pins = <
                    MX6QDL_PAD_SD3_CMD__SD3_CMD        0x17059
                    MX6QDL_PAD_SD3_CLK__SD3_CLK        0x10059
                    MX6QDL_PAD_SD3_DAT0__SD3_DATA0        0x17059
                    MX6QDL_PAD_SD3_DAT1__SD3_DATA1        0x17059
                    MX6QDL_PAD_SD3_DAT2__SD3_DATA2        0x17059
                    MX6QDL_PAD_SD3_DAT3__SD3_DATA3        0x17059
    //                MX6QDL_PAD_SD3_DAT4__SD3_DATA4        0x17059
    //                MX6QDL_PAD_SD3_DAT5__SD3_DATA5        0x17059
    //                MX6QDL_PAD_SD3_DAT6__SD3_DATA6        0x17059
    //                MX6QDL_PAD_SD3_DAT7__SD3_DATA7        0x17059
                >;
            };
    
            pinctrl_usdhc4: usdhc4grp {
                fsl,pins = <
                    MX6QDL_PAD_SD4_CMD__SD4_CMD        0x17059
                    MX6QDL_PAD_SD4_CLK__SD4_CLK        0x10059
                    MX6QDL_PAD_SD4_DAT0__SD4_DATA0        0x17059
                    MX6QDL_PAD_SD4_DAT1__SD4_DATA1        0x17059
                    MX6QDL_PAD_SD4_DAT2__SD4_DATA2        0x17059
                    MX6QDL_PAD_SD4_DAT3__SD4_DATA3        0x17059
                    MX6QDL_PAD_SD4_DAT4__SD4_DATA4        0x17059
                    MX6QDL_PAD_SD4_DAT5__SD4_DATA5        0x17059
                    MX6QDL_PAD_SD4_DAT6__SD4_DATA6        0x17059
                    MX6QDL_PAD_SD4_DAT7__SD4_DATA7        0x17059
                >;
            };
    
        };
    };
    
    &dcic1 {
        dcic_id = <0>;
        dcic_mux = "dcic-hdmi";
        status = "okay";
    };
    
    &dcic2 {
        dcic_id = <1>;
        dcic_mux = "dcic-lvds1";
        status = "okay";
    };
    
    &gpc {
        /* use ldo-bypass, u-boot will check it and configure */
        fsl,ldo-bypass = <1>;
        fsl,wdog-reset = <1>;
    };
    
    &hdmi_audio {
        status = "okay";
    };
    
    &hdmi_cec {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_hdmi_cec>;
        status = "okay";
    };
    
    &hdmi_core {
        ipu_id = <0>;
        disp_id = <0>;
        status = "okay";
    };
    
    &hdmi_video {
        fsl,phy_reg_vlev = <0x0294>;
        fsl,phy_reg_cksymtx = <0x800d>;
        status = "okay";
    };
    
    &ldb {
        status = "okay";
    //    split-mode;
    //    dual-mode;
    
        lvds-channel@0 {
            fsl,data-mapping = "spwg";
            fsl,data-width = <18>;
            status = "okay";
    
            display-timings {
                native-mode = <&timing0>;
                timing0: hsd100pxn1 {
                    clock-frequency = <65000000>;
                    hactive = <1024>;
                    vactive = <768>;
                    hback-porch = <220>;
                    hfront-porch = <40>;
                    vback-porch = <21>;
                    vfront-porch = <7>;
                    hsync-len = <60>;
                    vsync-len = <10>;
                };
            };
        };
    
        lvds-channel@1 {
            fsl,data-mapping = "spwg";
            fsl,data-width = <18>;
            primary;
            status = "okay";
    
            display-timings {
                native-mode = <&timing1>;
                timing1: hsd100pxn1 {
                    clock-frequency = <65000000>;
                    hactive = <800>;
                    vactive = <480>;
                    hback-porch = <220>;
                    hfront-porch = <40>;
                    vback-porch = <21>;
                    vfront-porch = <7>;
                    hsync-len = <60>;
                    vsync-len = <10>;
                };
            };
        };
    };
    
    &mipi_csi {
        status = "okay";
        ipu_id = <0>;
        csi_id = <1>;
        v_channel = <0>;
        lanes = <2>;
    };
    
    &mipi_dsi {
        dev_id = <0>;
        disp_id = <1>;
        lcd_panel = "TRULY-WVGA";
        disp-power-on-supply = <&reg_mipi_dsi_pwr_on>;
        resets = <&mipi_dsi_reset>;
        status = "okay";
    };
    
    &pcie {
        power-on-gpio = <&gpio3 19 0>;
        reset-gpio = <&gpio7 12 0>;
        status = "okay";
    };
    
    &pwm1 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_pwm1>;
        status = "okay";
    };
    #if 1
    &pwm2 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_pwm2>;
        status = "okay";
    };
    #endif
    &ssi2 {
        fsl,mode = "i2s-slave";
        status = "okay";
    };
    
    &uart1 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_uart1>;
        status = "okay";
    };
    #if 1
    &uart2 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_uart2>;
        status = "okay";
    };
    
    &uart3 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_uart3>;
        status = "okay";
    };
    
    &uart4 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_uart4>;
        status = "okay";
    };
    #endif
    #if 0
    &uart5 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_uart5_1>;
        fsl,uart-has-rtscts;
        status = "okay";
    };
    #endif
    &usbh1 {
        vbus-supply = <&reg_usb_h1_vbus>;
        status = "okay";
    };
    
    &usbotg {
        vbus-supply = <&reg_usb_otg_vbus>;
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_usbotg>;
        disable-over-current;
        srp-disable;
        hnp-disable;
        adp-disable;
        status = "okay";
    };
    #if 0
    &usdhc2 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_usdhc2>;
        bus-width = <4>;
    #if 0
        cd-gpios = <&gpio2 2 0>;
        wp-gpios = <&gpio2 3 0>;
    #endif
        no-1-8-v;
        wifi-host;
        pm-ignore-notify;
        keep-power-in-suspend;
        enable-sdio-wakeup;
        status = "okay";
    };
    #endif
    &usdhc3 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_usdhc3>;
        bus-width = <4>;
        cd-gpios = <&gpio4 10 0>;//sd3_cd
    //    wp-gpios = <&gpio2 1 0>;
        no-1-8-v;
        keep-power-in-suspend;
        enable-sdio-wakeup;
        status = "okay";
    };
    
    &usdhc4 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_usdhc4>;
        bus-width = <8>;
        non-removable;
        no-1-8-v;
        keep-power-in-suspend;
        status = "okay";
    };
    
    &wdog1 {
    //    status = "disabled";
        status = "okay";
    };
    
    &wdog2 {
    //    status = "okay";
        status = "disabled";
    };
```
