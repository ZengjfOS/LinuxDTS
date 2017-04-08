# i.MX6 skeleton.dtsi hacking

## 原文解析

```
    / {
        /**
         * 前面的#表示number的意思，address-cells = <1> 表示用来描述sub node中的reg属性的地址域特性使用1个u32的cell空间来来描述。
         * 同理推断#size-cells的含义。
         */
    	#address-cells = <1>;
    	#size-cells = <1>;
        /**
         * 主要用于描述由系统firmware指定的runtime parameter。如果存在chosen这个node，其parent node必须是根节点。
         * 原来通过tag list传递的一些linux kernel的运行时参数可以通过Device Tree传递。
         */
    	chosen { };
        /**
         * 节点定义了一些别名，Device tree是树状结构，当然要引用一个node的时候要指明相对于root node的full path，比较麻烦，所有有了别名引用
         */
    	aliases { };
        /**
         * memory device node是所有设备树文件的必备节点，它定义了系统物理内存的layout。
         */
    	memory { device_type = "memory"; reg = <0 0>; };
    };
```
