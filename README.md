# STM32 TB6612 电机调速与 OLED 实时显示

基于 STM32 HAL 库，使用 EC11 旋转编码器控制直流电机转向及转速，通过 TB6612 电机驱动模块实现电机控制，并使用 OLED 实时显示编码器 count 值和速度值。

本项目主要用于 STM32 定时器、PWM、编码器模式及 OLED 驱动库的学习与实践。



## 一、实现功能

1. **旋转编码器控制电机转速与方向**
   - 顺时针旋转旋钮，电机正转，速度逐渐加快；反向旋转旋钮，速度逐渐减慢。
   - 继续逆时针旋转旋钮，电机反转，速度逐渐加快；反向旋转旋钮，速度逐渐减慢。
2. **OLED 实时显示**
   - 显示编码器 count 值。
   - 显示电机速度值。
3. **TB6612 电机驱动**
   - 使用 TB6612 的 A 通道驱动直流电机。
   - 通过 PWM 调节电机速度，通过 GPIO 控制电机转向。

<img src="../../Note_Typora/figure/image-20261103100809331-1793693227568-1.png" style="zoom: 50%;" />



## 二、硬件接线

### 1、TB6612

+ **PWMA——PA2 【PWM信号】**

+ **AIN1——PA3**

+ **AIN2——PA4**

+ **STBY——面包板正极【STBY=L：芯片待机，电机不工作，一直给高电平电机才开始工作】**

+ **VM——STLINK 5V引脚 / 电机电源（ 如 12V）**

+ **VCC——3.3V，面包板正极**

+ **GND——面包板GND**

+ **AO1——正对电机的左侧引脚片（图中红夹子）**
+ **AO2——正对电机的右侧引脚片（图中黑夹子）**

<img src="../../Note_Typora/figure/image-20261103155514167-1793693246842-3.png" style="zoom:25%;" />

### 2、编码器EC11

+ **VCC——面包板正极**

+ **GND——面包板GND**
+ **A相——PA8**
+ **B相——PA9**

### 3. OLED 显示屏

OLED 使用 I2C1 进行通信，SCL、SDA 引脚根据 CubeMX 中 I2C1 的实际配置连接。



## 三、CubeMX 外设配置

1. **TIM1：** 配置为 Encoder Mode（编码器模式），用于读取 EC11 旋转编码器。
2. **TIM2：** 开启 CH3 PWM 输出，用于控制 TB6612 的 PWMA 引脚。
3. **I2C1：** 用于 OLED 显示屏通信。
4. **编码器输入滤波：** 建议设置为 15，以提高旋转编码器计数的稳定性。



## 四、其他

1、本代码OLED使用I2C1。

需要在oled.c中将

```\
void OLED_Send(uint8_t *data, uint8_t len)
{
  HAL_I2C_Master_Transmit(&hi2c2, OLED_ADDRESS, data, len, HAL_MAX_DELAY);
}
```

改为

```
void OLED_Send(uint8_t *data, uint8_t len)
{
  HAL_I2C_Master_Transmit(&hi2c1, OLED_ADDRESS, data, len, HAL_MAX_DELAY);
}
```

【也可以不改，直接在CubeMX中配置I2C2。只是本人每次习惯使用I2C1了。】

2、本例只使用了TB6612的通道A，所以在CubeMX上将两个空引脚PB12、PB13设置为GPIO_output，标签分别为TB6612_BIN1、TB6612_BIN。

3、在本人的面包板搭建过程中，编码器的四个引脚使用公对母杜邦线直接与对应位置连接，相比将编码器插在面包板上再使用跳线连接，连接更加稳定

4、需要在[波特律动](https://led.baud-dance.com/)取模助手中生成“值速度”这三字的代码，替代驱动库文件font.c中“波特律动”对应的代码。



## 五、参考项目与致谢

感谢以下作者提供的教程及开源驱动库：

- **Keysking（波特律动）**
  - 教程及 OLED 驱动库：[波特律动](https://led.baud-dance.com/)
- **Myqfeng**
  - TB6612 电机驱动库：[STM32 HAL 库 TB6612 电机驱动](https://gitee.com/myqfeng/stm32-hal-tb6612)
  - 本项目根据实际硬件配置，对原驱动库的 PWM 定时器及通道进行了适配。
  - 感谢 Keysking STM32 教程评论区用户「小黄人爱抄家」分享 TB6612 驱动库相关资源。