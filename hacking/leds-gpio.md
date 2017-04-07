# i.MX6 dts leds-gpio hacking

* 跟踪代码的目的如下：
  * 了解led在设备树上如何注册；
  * 了解led设备的设备节点名称是如何生成的；

## 设备树跟踪

* leds-gpio设备注册(imx6qdl-sabresd.dtsi)

```
    ...
	leds {
		compatible = "gpio-leds";

		charger-led {
			gpios = <&gpio1 2 0>;
			linux,default-trigger = "max8903-charger-charging";
			retain-state-suspended;
		};
	};
    ...
    &iomuxc {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_hog>;

        imx6qdl-sabresd {
            pinctrl_hog: hoggrp {
                fsl,pins = <
                    ...
                    MX6QDL_PAD_GPIO_2__GPIO1_IO02 0x1b0b0
                    ...
                >
                ...
            }
            ...
        }
        ...
    }
    ...
```

## leds-gpio驱动跟踪

```
    module_platform_driver(gpio_led_driver);     ----------------+
                                                                 |
    static struct platform_driver gpio_led_driver = {  <---------+
        .probe        = gpio_led_probe,                ---------------+
        .remove        = gpio_led_remove,                             |
        .driver        = {                                            |
            .name    = "leds-gpio",                                   |
            .owner    = THIS_MODULE,                                  |
            .of_match_table = of_match_ptr(of_gpio_leds_match),       |     --------+
        },                                                            |             |
    };                                                                |             |
                                                                      |             |
    static int gpio_led_probe(struct platform_device *pdev) <---------+             |
    {                                                                               |
        struct gpio_led_platform_data *pdata = dev_get_platdata(&pdev->dev);        |
        struct gpio_leds_priv *priv;                                                |
        int i, ret = 0;                                                             |
                                                                                    |
                                                                                    |
        // pdata->num_leds 应该是0，走else那条路                                    |
        if (pdata && pdata->num_leds) {                                             |
            priv = devm_kzalloc(&pdev->dev,                                         |
                    sizeof_gpio_leds_priv(pdata->num_leds),                         |
                        GFP_KERNEL);                                                |
            if (!priv)                                                              |
                return -ENOMEM;                                                     |
                                                                                    |
            priv->num_leds = pdata->num_leds;                                       |
            for (i = 0; i < priv->num_leds; i++) {                                  |
                ret = create_gpio_led(&pdata->leds[i],                              |
                              &priv->leds[i],                                       |
                              &pdev->dev, pdata->gpio_blink_set);                   |
                if (ret < 0) {                                                      |
                    /* On failure: unwind the led creations */                      |
                    for (i = i - 1; i >= 0; i--)                                    |
                        delete_gpio_led(&priv->leds[i]);                            |
                    return ret;                                                     |
                }                                                                   |
            }                                                                       |
        } else {                                                                    |
            priv = gpio_leds_create_of(pdev);   --------------------+               |
            if (IS_ERR(priv))                                       |               |
                return PTR_ERR(priv);                               |               |
        }                                                           |               |
                                                                    |               |
        platform_set_drvdata(pdev, priv);                           |               |
                                                                    |               |
        return 0;                                                   |               |
    }                                                               |               |
                                                                    |               |
    /* Code to create from OpenFirmware platform devices */         |               |
    #ifdef CONFIG_OF_GPIO                    v----------------------+               |
    static struct gpio_leds_priv *gpio_leds_create_of(struct platform_device *pdev) |
    {                                                                               |
        struct device_node *np = pdev->dev.of_node, *child;                         |
        struct gpio_leds_priv *priv;                                                |
        int count, ret;                                                             |
                                                                                    |
        /* count LEDs in this device, so we know how much to allocate */            |
        count = of_get_available_child_count(np);                                   |
        if (!count)                                                                 |
            return ERR_PTR(-ENODEV);                                                |
                                                                                    |
        for_each_available_child_of_node(np, child)                                 |
            if (of_get_gpio(child, 0) == -EPROBE_DEFER)                             |
                return ERR_PTR(-EPROBE_DEFER);                                      |
                                                                                    |
        priv = devm_kzalloc(&pdev->dev, sizeof_gpio_leds_priv(count),               |
                GFP_KERNEL);                                                        |
        if (!priv)                                                                  |
            return ERR_PTR(-ENOMEM);                                                |
                                                                                    |
        for_each_available_child_of_node(np, child) {                               |
            struct gpio_led led = {};                                               |
            enum of_gpio_flags flags;                                               |
            const char *state;                                                      |
                                                                                    |
            led.gpio = of_get_gpio_flags(child, 0, &flags);                         |
            led.active_low = flags & OF_GPIO_ACTIVE_LOW;                            |
            // 设备树上label属性就使用child->name作为名字，                         |
            // 后面设备节点会以这个名字为准                                         |
            led.name = of_get_property(child, "label", NULL) ? : child->name;       |
            // 获取设备树上的触发属性                                               |
            led.default_trigger =                                                   |
                of_get_property(child, "linux,default-trigger", NULL);              |
            // 获取设备树上的默认状态
            state = of_get_property(child, "default-state", NULL);                  |
            if (state) {                                                            |
                if (!strcmp(state, "keep"))                                         |
                    led.default_state = LEDS_GPIO_DEFSTATE_KEEP;                    |
                else if (!strcmp(state, "on"))                                      |
                    led.default_state = LEDS_GPIO_DEFSTATE_ON;                      |
                else                                                                |
                    led.default_state = LEDS_GPIO_DEFSTATE_OFF;                     |
            }                                                                       |
                                                                                    |
            // 获取设备树上的节点状态
            if (of_get_property(child, "retain-state-suspended", NULL))             |
                led.retain_state_suspended = 1;                                     |
                                                                                    |
            ret = create_gpio_led(&led, &priv->leds[priv->num_leds++],   -----------*-+
                          &pdev->dev, NULL);                                        | |
            if (ret < 0) {                                                          | |
                of_node_put(child);                                                 | |
                goto err;                                                           | |
            }                                                                       | |
        }                                                                           | |
                                                                                    | |
        return priv;                                                                | |
                                                                                    | |
    err:                                                                            | |
        for (count = priv->num_leds - 2; count >= 0; count--)                       | |
            delete_gpio_led(&priv->leds[count]);                                    | |
        return ERR_PTR(-ENODEV);                                                    | |
    }                                                                               | |
                                                                                    | |
    static const struct of_device_id of_gpio_leds_match[] = {         <-------------+ |
        { .compatible = "gpio-leds", },                                               |
        {},                                                                           |
    };                                                                                |
    #else /* CONFIG_OF_GPIO */                                                        |
    static struct gpio_leds_priv *gpio_leds_create_of(struct platform_device *pdev)   |
    {                                                                                 |
        return ERR_PTR(-ENODEV);                                                      |
    }                                                                                 |
    #endif /* CONFIG_OF_GPIO */                                                       |
                                                                                      |
    static int create_gpio_led(const struct gpio_led *template,       <---------------+
        struct gpio_led_data *led_dat, struct device *parent,
        int (*blink_set)(unsigned, int, unsigned long *, unsigned long *))
    {
        int ret, state;
    
        led_dat->gpio = -1;
    
        /* skip leds that aren't available */
        if (!gpio_is_valid(template->gpio)) {
            dev_info(parent, "Skipping unavailable LED gpio %d (%s)\n",
                    template->gpio, template->name);
            return 0;
        }
    
        ret = devm_gpio_request(parent, template->gpio, template->name);
        if (ret < 0)
            return ret;
    
        // 注意这里名字，后面生成设备节点会用到这个名字
        led_dat->cdev.name = template->name;
        led_dat->cdev.default_trigger = template->default_trigger;
        led_dat->gpio = template->gpio;
        led_dat->can_sleep = gpio_cansleep(template->gpio);
        led_dat->active_low = template->active_low;
        led_dat->blinking = 0;
        if (blink_set) {
            led_dat->platform_gpio_blink_set = blink_set;
            led_dat->cdev.blink_set = gpio_blink_set;
        }
        led_dat->cdev.brightness_set = gpio_led_set;
        if (template->default_state == LEDS_GPIO_DEFSTATE_KEEP)
            state = !!gpio_get_value_cansleep(led_dat->gpio) ^ led_dat->active_low;
        else
            state = (template->default_state == LEDS_GPIO_DEFSTATE_ON);
        led_dat->cdev.brightness = state ? LED_FULL : LED_OFF;
        if (!template->retain_state_suspended)
            led_dat->cdev.flags |= LED_CORE_SUSPENDRESUME;
    
        ret = gpio_direction_output(led_dat->gpio, led_dat->active_low ^ state);
        if (ret < 0)
            return ret;
    
        INIT_WORK(&led_dat->work, gpio_led_work);
    
        ret = led_classdev_register(parent, &led_dat->cdev);          ------------------+
        if (ret < 0)                                                                    |
            return ret;                                                                 |
                                                                                        |
        return 0;                                                                       |
    }                                                                                   |
                                                                                        |
    /**                                                                                 |
     * led_classdev_register - register a new object of led_classdev class.             |
     * @parent: The device to register.                                                 |
     * @led_cdev: the led_classdev structure for this device.                           |
     */                                                                                 |
    int led_classdev_register(struct device *parent, struct led_classdev *led_cdev) <---+
    {
        // 因为前面的设备树上没有label属性，用到前面的child->name来生成设备节点名
        led_cdev->dev = device_create(leds_class, parent, 0, led_cdev,
                          "%s", led_cdev->name);
        if (IS_ERR(led_cdev->dev))
            return PTR_ERR(led_cdev->dev);
    
    #ifdef CONFIG_LEDS_TRIGGERS
        init_rwsem(&led_cdev->trigger_lock);
    #endif
        /* add to the list of leds */
        down_write(&leds_list_lock);
        list_add_tail(&led_cdev->node, &leds_list);
        up_write(&leds_list_lock);
    
        if (!led_cdev->max_brightness)
            led_cdev->max_brightness = LED_FULL;
    
        led_update_brightness(led_cdev);
    
        INIT_WORK(&led_cdev->set_brightness_work, set_brightness_delayed);
    
        init_timer(&led_cdev->blink_timer);
        led_cdev->blink_timer.function = led_timer_function;
        led_cdev->blink_timer.data = (unsigned long)led_cdev;
    
    #ifdef CONFIG_LEDS_TRIGGERS
        led_trigger_set_default(led_cdev);
    #endif
    
        dev_dbg(parent, "Registered led device: %s\n",
                led_cdev->name);
    
        return 0;
    }
    EXPORT_SYMBOL_GPL(led_classdev_register);
    ```
