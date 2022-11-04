# Planck-pi fbtft 驱动移植ST7735屏
## 修改设备树

1.  在suniv-f1c100s.dtsi添加Planck-pi的SPI0管脚（PC0->CLK, PC1->CS, PC2->MISO, PC3->MOSI）

```cpp
spi0_pins: spi0-pins{
               	pins = "PC0", "PC1", "PC2", "PC3";
                function = "spi0";
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/70b2fb31399e4591bfa6c3d15490aba3.jpeg#pic_center)

```cpp
                 spi0:spi@1c05000 {
                        compatible = "allwinner,suniv-spi", "allwinner,sun8i-h3-spi";
                         reg = <0x1c05000 0x1000>;
                        interrupts =<0xa>;
                         clocks = <&ccu CLK_BUS_SPI0>, <&ccu CLK_BUS_SPI0>;
                        clock-names = "ahb", "mod";
                        resets = <&ccu RST_BUS_SPI0>;
                        status = "okay";
                        #address-cells =<1>;
                        #size-cells =<0>;
                        pinctrl-names = "default";
                        pinctrl-0 = <&spi0_pins>;

                };
```
2.  在suniv-f1c100s-licheepi-nano.dts中创建st7735s@0节点（PD0->DC, PD1->reset）
 

```cpp
&spi0 {
        //status ="okay";
        st7735r@0 {
            status ="okay";
            compatible = "sitronix,st7735r";
               reg = <0>;
               spi-max-frequency =<48000000>;        //SPI时钟50M
               rotate =<90>;                    //屏幕旋转90度
               spi-cpol;
               spi-cpha;
               rgb;                           //颜色格式RGB
               fps =<30>;                      //刷新30帧率
               buswidth =<8>;                   //总线宽度8
               dc-gpios  =<&pio 3 0 GPIO_ACTIVE_LOW>;   //GPIOD0
               reset-gpios=<&pio 3 1 GPIO_ACTIVE_LOW>;   //GPIOD1 暂时测试后面dc板子会改成pb4
               debug =<0>;                     //不开启调试
        };
```
## 修改fbtft-core.c文件

1. 添加头文件

```cpp
#include "linux/gpio.h"
#include "linux/of_gpio.h"
```
2. 修改函数

    fbtft_request_one_gpio，   fbtft_request_gpios， fbtft_reset    三个函数 
## make menuconfig 编译选项

1.SPI驱动的时候必须选择A31（Device Drivers -> SPI support）

2. Device Drivers  --->　　

　　　　[*] Staging drivers  --->　　

　　　　　　　　<*>   Support for small TFT LCD display modules  --->

　　　　　　　　　　　　　　<*>   FB driver for the ST7735R LCD Controller 
​
## 测试

1. ST7735S屏接线说明：(BL->3.3V, SDA->MOSI。其他的对应即可)  。SPI0的MISO脚不接，因为只向屏幕写数据。

2. 将编译的内核zImage 和dtb文件替换tf卡内的文件即可。

3. cat /dev/urandom > /dev/fb0  屏幕有雪花点 说明驱动成功。

4. 与终端交互   ls / > /dev/tty0

## 参考：

[SPI LCD驱动移植（基于fbtft)](https://www.bilibili.com/read/cv9947785)

[记录为Linux配置spi屏幕（st7735s）](https://blog.csdn.net/qq_46604211/article/details/116449891?ops_request_misc=&request_id=&biz_id=102&utm_term=linux%20st7735%E6%BA%90%E7%A0%81&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduweb~default-1-116449891.nonecase&spm=1018.2226.3001.4450)

​
