# i.MX6 dts gpio-keys hacking

跟踪代码的目的如下：
* 了解设备中GPIO口的控制器是怎么注册的；
* 了解GPIO要怎么指定；
* 了解gpio-keys怎么注册设备；
* 了解gpio-keys怎么获取设备树中属性；

## 设备树跟踪
* GPIO控制器注册(imx6qdl.dtsi)： 
  * [Documentation/devicetree/bindings/gpio/fsl-imx-gpio.txt](http://lxr.free-electrons.com/source/Documentation/devicetree/bindings/gpio/fsl-imx-gpio.txt)

```
	gpio1: gpio@0209c000 {
		compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
		reg = <0x0209c000 0x4000>;
		interrupts = <0 66 IRQ_TYPE_LEVEL_HIGH>,
			     <0 67 IRQ_TYPE_LEVEL_HIGH>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
	};

	gpio2: gpio@020a0000 {
		compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
		reg = <0x020a0000 0x4000>;
		interrupts = <0 68 IRQ_TYPE_LEVEL_HIGH>,
			     <0 69 IRQ_TYPE_LEVEL_HIGH>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
	};

	gpio3: gpio@020a4000 {
		compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
		reg = <0x020a4000 0x4000>;
		interrupts = <0 70 IRQ_TYPE_LEVEL_HIGH>,
			     <0 71 IRQ_TYPE_LEVEL_HIGH>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
	};

	gpio4: gpio@020a8000 {
		compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
		reg = <0x020a8000 0x4000>;
		interrupts = <0 72 IRQ_TYPE_LEVEL_HIGH>,
			     <0 73 IRQ_TYPE_LEVEL_HIGH>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
	};

	gpio5: gpio@020ac000 {
		compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
		reg = <0x020ac000 0x4000>;
		interrupts = <0 74 IRQ_TYPE_LEVEL_HIGH>,
			     <0 75 IRQ_TYPE_LEVEL_HIGH>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
	};

	gpio6: gpio@020b0000 {
		compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
		reg = <0x020b0000 0x4000>;
		interrupts = <0 76 IRQ_TYPE_LEVEL_HIGH>,
			     <0 77 IRQ_TYPE_LEVEL_HIGH>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
	};

	gpio7: gpio@020b4000 {
		compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
		reg = <0x020b4000 0x4000>;
		interrupts = <0 78 IRQ_TYPE_LEVEL_HIGH>,
			     <0 79 IRQ_TYPE_LEVEL_HIGH>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
	};
```
* 重新命名(imx6qdl.dtsi)
```
	aliases {
		gpio0 = &gpio1;
		gpio1 = &gpio2;
		gpio2 = &gpio3;
		gpio3 = &gpio4;
		gpio4 = &gpio5;
		gpio5 = &gpio6;
		gpio6 = &gpio7;
        ...
    }
```
* 注册gpio-keys(imx6qdl-sabresd.dtsi)，ref：
  * [Documentation/devicetree/bindings/gpio/gpio_keys.txt](http://lxr.free-electrons.com/source/Documentation/devicetree/bindings/gpio/gpio_keys.txt)

```
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
```
* 配置GPIO口的iomux(imx6qdl-sabresd.dtsi)，ref：
  * [Documentation/devicetree/bindings/pinctrl/fsl,imx-pinctrl.txt](http://lxr.free-electrons.com/source/Documentation/devicetree/bindings/pinctrl/fsl,imx-pinctrl.txt)
  * [Documentation/devicetree/bindings/pinctrl/fsl,imx6dl-pinctrl.txt](http://lxr.free-electrons.com/source/Documentation/devicetree/bindings/pinctrl/fsl,imx6dl-pinctrl.txt)

```
    pinctrl_gpio_keys: gpio_keysgrp {
        fsl,pins = <
            MX6QDL_PAD_EIM_D29__GPIO3_IO29 0x80000000
            MX6QDL_PAD_GPIO_4__GPIO1_IO04  0x80000000
            MX6QDL_PAD_GPIO_5__GPIO1_IO05  0x80000000
        >;
    };
```

## gpio-key Linux driver跟踪
```
    late_initcall(gpio_keys_init);     --------------+
    module_exit(gpio_keys_exit);                     |
                                                     |
    static int __init gpio_keys_init(void)     <-----+
    {
        return platform_driver_register(&gpio_keys_device_driver);  -----+
    }                                                                    |
                                                                         |
    static void __exit gpio_keys_exit(void)                              |
    {                                                                    |
        platform_driver_unregister(&gpio_keys_device_driver);            |
    }                                                                    |
                                                                         |
    static struct platform_driver gpio_keys_device_driver = {      <-----+
        .probe        = gpio_keys_probe,                      -------------+
        .remove        = gpio_keys_remove,                                 |
        .driver        = {                                                 |
            .name    = "gpio-keys",                                        |
            .owner    = THIS_MODULE,                                       |
            .pm    = &gpio_keys_pm_ops,                                    |
            .of_match_table = of_match_ptr(gpio_keys_of_match), ----+      |
        }                                                           |      |
    };                                                              |      |
                                                                    |      |
    static struct of_device_id gpio_keys_of_match[] = {       <-----+      |
        // 和设备树中的compatible对应                                      |
        { .compatible = "gpio-keys", },                                    |
        { },                                                               |
    };                                                                     |
                                                                           |
    static int gpio_keys_probe(struct platform_device *pdev)  <------------+
    {
        struct device *dev = &pdev->dev;
        const struct gpio_keys_platform_data *pdata = dev_get_platdata(dev);
        struct gpio_keys_drvdata *ddata;
        struct input_dev *input;
        int i, error;
        int wakeup = 0;
    
        if (!pdata) {
            pdata = gpio_keys_get_devtree_pdata(dev);        ---------------+
            if (IS_ERR(pdata))                                              |
                return PTR_ERR(pdata);                                      |
        }                                                                   |
                                                                            |
        ddata = kzalloc(sizeof(struct gpio_keys_drvdata) +                  |
                pdata->nbuttons * sizeof(struct gpio_button_data),          |
                GFP_KERNEL);                                                |
        input = input_allocate_device();                                    |
        if (!ddata || !input) {                                             |
            dev_err(dev, "failed to allocate state\n");                     |
            error = -ENOMEM;                                                |
            goto fail1;                                                     |
        }                                                                   |
                                                                            |
        ddata->pdata = pdata;                                               |
        ddata->input = input;                                               |
        mutex_init(&ddata->disable_lock);                                   |
                                                                            |
        platform_set_drvdata(pdev, ddata);                                  |
        input_set_drvdata(input, ddata);                                    |
                                                                            |
        input->name = pdata->name ? : pdev->name;                           |
        input->phys = "gpio-keys/input0";                                   |
        input->dev.parent = &pdev->dev;                                     |
        input->open = gpio_keys_open;                                       |
        input->close = gpio_keys_close;                                     |
                                                                            |
        input->id.bustype = BUS_HOST;                                       |
        input->id.vendor = 0x0001;                                          |
        input->id.product = 0x0001;                                         |
        input->id.version = 0x0100;                                         |
                                                                            |
        /* Enable auto repeat feature of Linux input subsystem */           |
        if (pdata->rep)                                                     |
            __set_bit(EV_REP, input->evbit);                                |
                                                                            |
        for (i = 0; i < pdata->nbuttons; i++) {                             |
            const struct gpio_keys_button *button = &pdata->buttons[i];     |
            struct gpio_button_data *bdata = &ddata->data[i];               |
                                                                            |
            error = gpio_keys_setup_key(pdev, input, bdata, button);  ------*--+
            if (error)                                                      |  |
                goto fail2;                                                 |  |
                                                                            |  |
            if (button->wakeup)                                             |  |
                wakeup = 1;                                                 |  |
        }                                                                   |  |
                                                                            |  |
        error = sysfs_create_group(&pdev->dev.kobj, &gpio_keys_attr_group); |  |
        if (error) {                                                        |  |
            dev_err(dev, "Unable to export keys/switches, error: %d\n",     |  |
                error);                                                     |  |
            goto fail2;                                                     |  |
        }                                                                   |  |
                                                                            |  |
        error = input_register_device(input);                               |  |
        if (error) {                                                        |  |
            dev_err(dev, "Unable to register input device, error: %d\n",    |  |
                error);                                                     |  |
            goto fail3;                                                     |  |
        }                                                                   |  |
                                                                            |  |
        device_init_wakeup(&pdev->dev, wakeup);                             |  |
                                                                            |  |
        return 0;                                                           |  |
                                                                            |  |
     fail3:                                                                 |  |
        sysfs_remove_group(&pdev->dev.kobj, &gpio_keys_attr_group);         |  |
     fail2:                                                                 |  |
        while (--i >= 0)                                                    |  |
            gpio_remove_key(&ddata->data[i]);                               |  |
                                                                            |  |
     fail1:                                                                 |  |
        input_free_device(input);                                           |  |
        kfree(ddata);                                                       |  |
        /* If we have no platform data, we allocated pdata dynamically. */  |  |
        if (!dev_get_platdata(&pdev->dev))                                  |  |
            kfree(pdata);                                                   |  |
                                                                            |  |
        return error;                                                       |  |
    }                                                                       |  |
                                                                            |  |
    static struct gpio_keys_platform_data *                                 |  |
    gpio_keys_get_devtree_pdata(struct device *dev)           <-------------+  |
    {                                                                          |
        struct device_node *node, *pp;                                         |
        struct gpio_keys_platform_data *pdata;                                 |
        struct gpio_keys_button *button;                                       |
        int error;                                                             |
        int nbuttons;                                                          |
        int i;                                                                 |
                                                                               |
        node = dev->of_node;                                                   |
        if (!node) {                                                           |
            error = -ENODEV;                                                   |
            goto err_out;                                                      |
        }                                                                      |
                                                                               |
        // 获取当前gpio-keys注册了多少个按键，目前是3个                        |
        nbuttons = of_get_child_count(node);                                   |
        if (nbuttons == 0) {                                                   |
            error = -ENODEV;                                                   |
            goto err_out;                                                      |
        }                                                                      |
                                                                               |
        pdata = kzalloc(sizeof(*pdata) + nbuttons * (sizeof *button),          |
                GFP_KERNEL);                                                   |
        if (!pdata) {                                                          |
            error = -ENOMEM;                                                   |
            goto err_out;                                                      |
        }                                                                      |
                                                                               |
        pdata->buttons = (struct gpio_keys_button *)(pdata + 1);               |
        pdata->nbuttons = nbuttons;                                            |
                                                                               |
        pdata->rep = !!of_get_property(node, "autorepeat", NULL);              |
                                                                               |
        // foreach 迭代指针                                                    |
        i = 0;                                                                 |
        for_each_child_of_node(node, pp) {                                     |
            int gpio;                                                          |
            enum of_gpio_flags flags;                                          |
                                                                               |
            // 检查是否有gpios属性                                             |
            if (!of_find_property(pp, "gpios", NULL)) {                        |
                pdata->nbuttons--;                                             |
                dev_warn(dev, "Found button without gpios\n");                 |
                continue;                                                      |
            }                                                                  |
                                                                               |
            // 获取设备树的gpios属性值，这里是一个整数值，flags是触发有效值    |
            gpio = of_get_gpio_flags(pp, 0, &flags);                           |
            if (gpio < 0) {                                                    |
                error = gpio;                                                  |
                if (error != -EPROBE_DEFER)                                    |
                    dev_err(dev,                                               |
                        "Failed to get gpio flags, error: %d\n",               |
                        error);                                                |
                goto err_free_pdata;                                           |
            }                                                                  |
                                                                               |
            button = &pdata->buttons[i++];                                     |
                                                                               |
            button->gpio = gpio;                                               |
            button->active_low = flags & OF_GPIO_ACTIVE_LOW;                   |
                                                                               |
            // 获取设备树的linux,code属性                                      |
            if (of_property_read_u32(pp, "linux,code", &button->code)) {       |
                dev_err(dev, "Button without keycode: 0x%x\n",                 |
                    button->gpio);                                             |
                error = -EINVAL;                                               |
                goto err_free_pdata;                                           |
            }                                                                  |
                                                                               |
            // 获取设备树的label属性
            button->desc = of_get_property(pp, "label", NULL);                 |
                                                                               |
            if (of_property_read_u32(pp, "linux,input-type", &button->type))   |
                button->type = EV_KEY;                                         |
                                                                               |
            // 获取设备树的gpio-key,wakeup属性，这一般是power键专有功能        |
            button->wakeup = !!of_get_property(pp, "gpio-key,wakeup", NULL);   |
                                                                               |
            if (of_property_read_u32(pp, "debounce-interval",                  |
                         &button->debounce_interval))                          |
                button->debounce_interval = 5;                                 |
        }                                                                      |
                                                                               |
        if (pdata->nbuttons == 0) {                                            |
            error = -EINVAL;                                                   |
            goto err_free_pdata;                                               |
        }                                                                      |
                                                                               |
        return pdata;                                                          |
                                                                               |
    err_free_pdata:                                                            |
        kfree(pdata);                                                          |
    err_out:                                                                   |
        return ERR_PTR(error);                                                 |
    }                                                                          |
                                                                               |
    static int gpio_keys_setup_key(struct platform_device *pdev,    <----------+
                    struct input_dev *input,
                    struct gpio_button_data *bdata,
                    const struct gpio_keys_button *button)
    {
        const char *desc = button->desc ? button->desc : "gpio_keys";
        struct device *dev = &pdev->dev;
        irq_handler_t isr;
        unsigned long irqflags;
        int irq, error;
    
        bdata->input = input;
        bdata->button = button;
        spin_lock_init(&bdata->lock);
    
        if (gpio_is_valid(button->gpio)) {
    
            // 从前面看，其实是不知道这个gpio到底是什么值得，这个相当于以前的IMX_GPIO_NR()转换后的值
            error = gpio_request_one(button->gpio, GPIOF_IN, desc);
            if (error < 0) {
                dev_err(dev, "Failed to request GPIO %d, error %d\n",
                    button->gpio, error);
                return error;
            }
    
            if (button->debounce_interval) {
                error = gpio_set_debounce(button->gpio,
                        button->debounce_interval * 1000);
                /* use timer if gpiolib doesn't provide debounce */
                if (error < 0)
                    bdata->timer_debounce =
                            button->debounce_interval;
            }
    
            irq = gpio_to_irq(button->gpio);
            if (irq < 0) {
                error = irq;
                dev_err(dev,
                    "Unable to get irq number for GPIO %d, error %d\n",
                    button->gpio, error);
                goto fail;
            }
            bdata->irq = irq;
    
            INIT_WORK(&bdata->work, gpio_keys_gpio_work_func);
            setup_timer(&bdata->timer,
                    gpio_keys_gpio_timer, (unsigned long)bdata);
    
            isr = gpio_keys_gpio_isr;
            irqflags = IRQF_TRIGGER_RISING | IRQF_TRIGGER_FALLING;
            if (bdata->button->wakeup)
                irqflags |= IRQF_NO_SUSPEND;
    
        } else {
            if (!button->irq) {
                dev_err(dev, "No IRQ specified\n");
                return -EINVAL;
            }
            bdata->irq = button->irq;
    
            if (button->type && button->type != EV_KEY) {
                dev_err(dev, "Only EV_KEY allowed for IRQ buttons.\n");
                return -EINVAL;
            }
    
            bdata->timer_debounce = button->debounce_interval;
            setup_timer(&bdata->timer,
                    gpio_keys_irq_timer, (unsigned long)bdata);
    
            isr = gpio_keys_irq_isr;
            irqflags = 0;
        }
    
        input_set_capability(input, button->type ?: EV_KEY, button->code);
    
        /*
         * If platform has specified that the button can be disabled,
         * we don't want it to share the interrupt line.
         */
        if (!button->can_disable)
            irqflags |= IRQF_SHARED;
    
        // 申请按键中断
        error = request_any_context_irq(bdata->irq, isr, irqflags, desc, bdata);
        if (error < 0) {
            dev_err(dev, "Unable to claim irq %d; error %d\n",
                bdata->irq, error);
            goto fail;
        }
    
        return 0;
    
    fail:
        if (gpio_is_valid(button->gpio))
            gpio_free(button->gpio);
    
        return error;
    }
```
