# 示例 · 第 1 周 GPIO 完整例子（范文）

> ⚠️ 这是一份**范文**，告诉你"写到什么程度算合格"。
> 代码要自己敲一遍（别复制），README 要写**你自己跑出来的现象、你自己踩的坑**。
> 直接把这份抄进你的仓库没有意义——面试官聊两句就能听出来是不是你做的。

---

## 一、代码长什么样（合格水平）

工程结构（Keil 标准库工程，江协同款 STM32F103C8T6）：

```
01-gpio/
├── README.md          # 下面第二节的范文就是它该有的样子
├── main.c
├── Delay.c
├── Delay.h
└── (Keil 工程文件，Objects/Listings 已被 .gitignore 自动忽略)
```

`main.c`：

```c
#include "stm32f10x.h"   // 设备头文件，标准库工程自带
#include "Delay.h"

int main(void)
{
    /* 1. 开时钟：STM32 每个外设用之前必须先在 RCC 里开时钟。
          新手第一大坑：忘了这行，后面配什么都没反应。 */
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);

    /* 2. 配置引脚：PA1、PA2 接了两颗 LED（低电平点亮）。
          引脚以你手上板子的原理图为准——查原理图本身就是练习。 */
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_Out_PP;   // 推挽输出：能主动输出强 0 和强 1
    GPIO_InitStructure.GPIO_Pin   = GPIO_Pin_1 | GPIO_Pin_2;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;   // 输出速度，点灯随便选
    GPIO_Init(GPIOA, &GPIO_InitStructure);              // 把上面的配置写进寄存器

    while (1)
    {
        /* 流水灯：两颗灯各亮 500ms 交替。
            注意写法：Bit_RESET = 输出 0 = LED 导通亮（低电平点亮的板子） */
        GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_RESET);    // PA1 亮
        Delay_ms(500);
        GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_SET);      // PA1 灭
        GPIO_WriteBit(GPIOA, GPIO_Pin_2, Bit_RESET);    // PA2 亮
        Delay_ms(500);
        GPIO_WriteBit(GPIOA, GPIO_Pin_2, Bit_SET);      // PA2 灭
    }
}
```

`Delay.c`（用 SysTick 定时，比空循环delay准）：

```c
#include "stm32f10x.h"

void Delay_us(uint32_t xus)
{
    SysTick->LOAD = 72 * xus;                // 主频 72MHz：1us = 72 个时钟周期
    SysTick->VAL  = 0x0000000;               // 清空计数值
    SysTick->CTRL = 0x0000005;               // bit2=1 用内核时钟，bit0=1 开始计数
    while (!(SysTick->CTRL & 0x00010000));   // 等计数到 0（COUNTFLAG 置 1）
    SysTick->CTRL = 0x0000000;               // 关闭 SysTick
}

void Delay_ms(uint32_t xms)
{
    while (xms--) Delay_us(1000);
}
```

`Delay.h`：

```c
#ifndef __DELAY_H
#define __DELAY_H
#include "stm32f10x.h"
void Delay_us(uint32_t xus);
void Delay_ms(uint32_t xms);
#endif
```

**注意看代码里什么最重要**：不是功能多复杂，而是——①分步骤注释（为什么开时钟、为什么选推挽）；②关键常量有解释（72×xus 是怎么来的）。面试官看的就是这些。

---

## 二、README 长什么样（范文）

```markdown
# 01 · GPIO 点灯（第 1 周）

## 目标
不抄例程，独立完成：LED 闪烁 + 两灯流水灯。
搞懂三步曲：开时钟 → 配模式 → 读写引脚。

## 现象
PA1、PA2 两颗 LED 每 500ms 交替亮灭。
万用表量引脚：亮 = 0V，灭 = 3.3V，和"低电平点亮"的原理图对上了。
（这里贴一张流水灯的图或短视频链接更好）

## 我做了什么
- 时钟：查手册确认 GPIOA 挂在 APB2 上，所以用 RCC_APB2PeriphClockCmd 开
- 模式：选推挽输出（GPIO_Mode_Out_PP），因为要主动输出高低电平驱动 LED；
  如果是按键输入就该换成上拉/下拉输入——这是两种场景的核心区别
- 延时：用 SysTick 写了 Delay_us，比空循环准，后面串口/定时器都要复用

## 踩坑
1. LED 一直不亮 → Keil 单步调试发现程序根本没进 GPIO_Init →
   原来忘了开 RCC 时钟，加上那行就好了。教训：外设不响应，先查时钟。
2. 两个灯同时亮半秒 → Delay_ms 没生效 → SysTick->VAL 拼错了，
   改对后正常。教训：延时不准先怀疑延时函数本身。
```

看到区别了吗：**"现象"写你亲眼看到的，"踩坑"写你真踩的**。每条踩坑都是"现象 → 怎么定位 → 怎么改"三段。

---

## 三、你的操作步骤（今晚就能做完）

1. Keil 新建工程（照江协 P3「新建工程」那一集），芯片选 STM32F103C8T6
2. 把上面三个文件自己敲进去（敲，不是复制——敲的过程就是学习）
3. 编译 → 烧录 → 看现象
4. 把工程整个文件夹放进 `D:\Code\stm32-learning\01-gpio\`（Objects、Listings 会自动被忽略）
5. 按范文格式写 `01-gpio/README.md`，写你自己的现象和坑
6. 双击 `每天提交.bat`，输入：`feat(gpio): LED 流水灯，学会开时钟三步曲`

完成。这就是一份合格的第 1 周提交。
