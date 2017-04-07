# i.MX6 dts gpio-reset hacking

* 跟踪代码的目的如下：
  * 了解reset按键在设备树上如何注册；
  * 了解reset按键的设备节点名称是如何生成的；

## 设备树跟踪

* leds-gpio设备注册(imx6qdl-sabresd.dtsi)
  * [Documentation/devicetree/bindings/reset/gpio-reset.txt](http://lxr.free-electrons.com/source/Documentation/devicetree/bindings/reset/gpio-reset.txt)

```
	mipi_dsi_reset: mipi-dsi-reset {
		compatible = "gpio-reset";
		reset-gpios = <&gpio6 11 GPIO_ACTIVE_LOW>;
		reset-delay-us = <50>;
		#reset-cells = <0>;
	};
```

## leds-gpio驱动跟踪
  * drivers/reset/gpio-reset.c

```
    arch_initcall(gpio_reset_init);                 --------------+
                                                                  |
    static int __init gpio_reset_init(void)         <-------------+
    {
        return platform_driver_register(&gpio_reset_driver); -----+
    }                                                             |
                                                                  |
    static struct platform_driver gpio_reset_driver = {    <------+
        .probe = gpio_reset_probe,                         ---------+
        .remove = gpio_reset_remove,                                |
        .driver = {                                                 |
            .name = "gpio-reset",                                   |
            .owner = THIS_MODULE,                                   |
            .of_match_table = of_match_ptr(gpio_reset_dt_ids), ---+ |
        },                                                        | |
    };                                                            | |
                                                                  | |
    static struct of_device_id gpio_reset_dt_ids[] = {     <------+ |
        { .compatible = "gpio-reset" },                             |
        { }                                                         |
    };                                                              |
                                                                    |
    static int gpio_reset_probe(struct platform_device *pdev)  <----+
    {
        struct device_node *np = pdev->dev.of_node;
        struct gpio_reset_data *drvdata;
        enum of_gpio_flags flags;
        unsigned long gpio_flags;
        bool initially_in_reset;
        int ret;
    
        drvdata = devm_kzalloc(&pdev->dev, sizeof(*drvdata), GFP_KERNEL);
        if (drvdata == NULL)
            return -ENOMEM;
    
        if (of_gpio_named_count(np, "reset-gpios") != 1) {
            dev_err(&pdev->dev,
                "reset-gpios property missing, or not a single gpio\n");
            return -EINVAL;
        }
    
        drvdata->gpio = of_get_named_gpio_flags(np, "reset-gpios", 0, &flags);
        if (drvdata->gpio == -EPROBE_DEFER) {
            return drvdata->gpio;
        } else if (!gpio_is_valid(drvdata->gpio)) {
            dev_err(&pdev->dev, "invalid reset gpio: %d\n", drvdata->gpio);
            return drvdata->gpio;
        }
    
        drvdata->active_low = flags & OF_GPIO_ACTIVE_LOW;
    
        ret = of_property_read_u32(np, "reset-delay-us", &drvdata->delay_us);
        if (ret < 0)
            drvdata->delay_us = -1;
        else if (drvdata->delay_us < 0)
            dev_warn(&pdev->dev, "reset delay too high\n");
    
        initially_in_reset = of_property_read_bool(np, "initially-in-reset");
        if (drvdata->active_low ^ initially_in_reset)
            gpio_flags = GPIOF_OUT_INIT_HIGH;
        else
            gpio_flags = GPIOF_OUT_INIT_LOW;
    
        ret = devm_gpio_request_one(&pdev->dev, drvdata->gpio, gpio_flags, NULL);
        if (ret < 0) {
            dev_err(&pdev->dev, "failed to request gpio %d: %d\n",
                drvdata->gpio, ret);
            return ret;
        }
    
        platform_set_drvdata(pdev, drvdata);
    
        drvdata->rcdev.of_node = np;
        drvdata->rcdev.owner = THIS_MODULE;
        drvdata->rcdev.nr_resets = 1;
        drvdata->rcdev.ops = &gpio_reset_ops;         ----------+
        drvdata->rcdev.of_xlate = of_gpio_reset_xlate;          |
        reset_controller_register(&drvdata->rcdev);             |       ----------+
                                                                |                 |
        return 0;                                               |                 |
    }                                                           |                 |
                                                                |                 |
    static struct reset_control_ops gpio_reset_ops = {   <------+                 |
        .reset = gpio_reset,                   --------------+                    |
        .assert = gpio_reset_assert,                         |                    |
        .deassert = gpio_reset_deassert,                     |                    |
    };                                                       |                    |
                     v---------------------------------------+                    |
    static int gpio_reset(struct reset_controller_dev *rcdev, unsigned long id)   |
    {                                                                             |
        struct gpio_reset_data *drvdata = container_of(rcdev,                     |
                struct gpio_reset_data, rcdev);                                   |
                                                                                  |
        if (drvdata->delay_us < 0)                                                |
            return -ENOSYS;                                                       |
                                                                                  |
        gpio_reset_set(rcdev, 1);                                                 |
        udelay(drvdata->delay_us);                                                |
        gpio_reset_set(rcdev, 0);                                                 |
                                                                                  |
        return 0;                                                                 |
    }                                                                             |
                                                                                  |
    /**                                                                           |
     * reset_controller_register - register a reset controller device             |
     * @rcdev: a pointer to the initialized reset controller device               |
     */                                                                           |
    int reset_controller_register(struct reset_controller_dev *rcdev)   <---------+
    {
        if (!rcdev->of_xlate) {
            rcdev->of_reset_n_cells = 1;
            rcdev->of_xlate = of_reset_simple_xlate;
        }
    
        mutex_lock(&reset_controller_list_mutex);
        list_add(&rcdev->list, &reset_controller_list);
        mutex_unlock(&reset_controller_list_mutex);
    
        return 0;
    }
    EXPORT_SYMBOL_GPL(reset_controller_register);
```
