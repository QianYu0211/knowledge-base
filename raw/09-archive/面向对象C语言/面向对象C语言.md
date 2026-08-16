# 面向对象 C 语言学习笔记

> 本笔记由「面向对象C语言」系列课件整理而成,完整覆盖用 C 语言模拟面向对象编程的全部核心技法。

## 目录

- [第0章 面向过程 vs 面向对象](#第0章-面向过程-vs-面向对象)
- [第1章 函数指针基础](#第1章-函数指针基础)
- [第2章 结构体 + 函数指针 = 模拟面向对象](#第2章-结构体--函数指针--模拟面向对象)
- [第3章 C 语言 OOP 总览:LED 驱动架构](#第3章-c-语言-oop-总览led-驱动架构)
- [第4章 UART 设备类 OOP 实战](#第4章-uart-设备类-oop-实战)
- [第5章 封装:不透明指针](#第5章-封装不透明指针)
- [第6章 构造与析构函数](#第6章-构造与析构函数)
- [第7章 静态对象数组](#第7章-静态对象数组)
- [第8章 对象池](#第8章-对象池)
- [第9章 单例模式](#第9章-单例模式)
- [第10章 深拷贝与浅拷贝](#第10章-深拷贝与浅拷贝)
- [第11章 内存对齐](#第11章-内存对齐)
- [第12章 继承](#第12章-继承)
- [第13章 多态](#第13章-多态)

---

## 第0章 面向过程 vs 面向对象

**学习目标**:从思想层面理解面向过程与面向对象的本质区别,为后续用 C 模拟 OOP 打下认知基础。

### 0.1 本质区别

- **面向对象**:内聚数据和行为。在构建数据之间联系、封装对象的过程中,顺带就把问题解决了。
  - 例如:程序的目标是控制一个电机 → 面向对象就是在把"数据 + 行为"封装成一个个对象的过程中,自动把控制电机的问题解决了。
- **面向过程**:不在意数据和行为如何归类,只要拿到数据和行为就直接拿来用,能解决问题就算完成。

> 先忘掉"继承、封装、多态"这些概念——它们是面向对象思路已经掌握之后的具体做法。没有理解本质区别,直接搞继承封装多态,都是空中楼阁。

### 0.2 核心类比:从原始社会到城市

以"人的一生:种地 → 吃饭 → 睡觉 → 治病 → 生养孩子"为例:

| | 原始社会(面向过程) | 城市社会(面向对象) |
|---|---|---|
| 解决问题的方式 | 一个人包揽所有事 | 分工,每类人只解决自己的问题 |
| 质量 | 每件事都做不好、不可复用 | 专业化,质量高、可复用 |
| 数据与行为 | 全部嵌在一个人身上,没有明确特性 | 农场主=种地,厨师=烹饪,医生=治病…… |
| 复制 | 无法复制"医生"这个角色 | 复制医院时,"医生"这个对象可无限复制 |

**为什么会产生分工**:社会是逆熵增的。生命体、社会会逐渐归类、积累知识,人的欲望推动不停向前。种地可以种更多更好 → 于是有人专门种地(农场主),有人专门做菜(厨师),有人专门开酒店,有人专门治病(医生),有人专门教书(老师)。**每一个步骤越来越精细化,社会分工由此产生。**

**对象的内涵**:
- 每一个"场景建筑"(农场、食堂、酒店、医院、学校)里装的都是**拥有自己数据和行为的人**。
- 农场里的人掌握种地和粮食筛选;食堂的人掌握烹饪;酒店的人掌握酒店管理。
- 把酒店人拿去食堂他不会做食堂的事,把食堂人拿去医院他不会做医生 → **不同对象之间实现了隔离**。
- 每个对象解决自己对应的问题,他们彼此存在,就解决了我们要解决的一串问题中的每一个。

### 0.3 复制对象:类与实例

- 面向过程中,代码把一串事情全部嵌在一起,没有"医生"这种可以单独分配的明确角色。
- 面向对象中,"医生"可以变成一个对象;**复制医院时,这个对象可以无限复制**。
- 在代码层面:同一个"医生类" = 同一个 class,无限创建实例。

### 0.4 用类比理解三大特性

- **继承**:在北京创建农场积累的农业知识,去上海开农场时可以继承,再结合当地气候做调整;北京食堂的菜谱可以继承,到上海再开一家食堂,但开食堂的人(代码)变了——无需从 0 到 1 重写。
- **封装**:分酒店可以按自己的价格招待本地客户,但拿不到总酒店的优惠和授权;即使继承了总酒店的管理方案,也无法替总酒店做决定——权限被封装起来。
- **多态**:有了种粮食的技能和流程,同一个农场既可以种西瓜,也可以种小麦、大豆;无论种什么,都需要浇水——**同一个接口,不同的实现**。

### 要点总结

1. 面向对象 = 内聚数据和行为,在封装对象的过程中解决问题。
2. 面向过程 = 只在乎拿数据和行为解决问题,不管归类。
3. 类比主线:原始社会(一人干所有事) → 城市(每类人干自己的事,专业化、可复用)。
4. 类 = 可无限复制的角色模板,实例 = 具体的角色。
5. 继承、封装、多态都是围绕"具象每一个要解决的问题并复用它"这个目标服务的。

---

## 第1章 函数指针基础

**学习目标**:掌握函数指针的语法、赋值、调用,理解它是 C 语言实现"多态"与"回调"的基石。

### 1.1 什么是函数指针

- 函数编译后存在内存的**代码区**,有起始地址。
- **函数指针就是存放函数起始地址的变量**。
- 函数名即地址:`func_ptr = add;` 与 `func_ptr = &add;` 完全等价。

### 1.2 声明语法

```c
// 返回类型 (*指针名)(参数类型...)
int (*operation)(int, int);
```

> **括号至关重要**:`int (*p)(int)` 是"指向返回 int 的函数的指针";去掉括号的 `int* p(int)` 就变成了"返回 int 指针的函数"。

### 1.3 赋值与调用

```c
int add(int a, int b)
{
    return a + b;
}

int multiply(int a, int b)
{
    return a * b;
}

int main(void)
{
    int (*operation)(int, int);

    operation = add;            // 赋值:指向加法
    printf("%d\n", operation(3, 5));    // 8

    operation = multiply;       // 改变指向:指向乘法
    printf("%d\n", operation(3, 5));    // 15

    return 0;
}
```

- 两种调用方式等价:`(*operation)(3, 5)`(显式解引用)与 `operation(3, 5)`(直接调用,现代代码推荐)。
- 同一个函数指针通过改变指向调用不同函数 → **动态绑定**,这是"多态"的雏形。

### 1.4 生活类比:万能遥控器

遥控器像函数指针,可以指向不同的"电器"(函数)。按按钮控制当前指向的设备:
- 电视机 = `function_A()`,空调 = `function_B()`,按"开机"键时,执行的是当前遥控器指向的那个电器对应的函数。

### 1.5 应用场景

- 回调函数(如注册事件处理)
- 函数表 / 虚函数表
- 策略模式
- 状态机

### 要点总结

| 操作 | 语法 |
|---|---|
| 声明 | `int (*ptr)(int, int);` |
| 赋值 | `ptr = func;` 或 `ptr = &func;` |
| 调用 | `(*ptr)(args);` 或 `ptr(args);` |

1. 函数名就是函数地址。
2. 声明时括号决定是"函数指针"还是"返回指针的函数"。
3. 同一指针改变指向即可切换行为,为多态打基础。

---

## 第2章 结构体 + 函数指针 = 模拟面向对象

**学习目标**:掌握把函数指针放进结构体,实现"数据 + 行为"的封装,从而用 C 语言模拟"类"与"对象"。

### 2.1 核心思想

- 结构体是"容器",可装不同类型数据。
- 把**函数指针**作为结构体成员 → 函数指针成员就是对象的"行为/方法"。
- 数据(成员变量) + 行为(函数指针成员) 封装在一起 = 模拟一个类。

### 2.2 Dog 类示例

```c
struct Dog
{
    char name[20];
    int  age;
    void (*bark)(void);          // 无参函数指针成员:行为
    void (*eat)(char* food);     // 带参函数指针成员:行为
};

void dog_bark(void)
{
    printf("汪汪汪!\n");
}

void dog_eat(char* food)
{
    printf("吃%s\n", food);
}

int main(void)
{
    struct Dog myDog;

    strcpy(myDog.name, "旺财");
    myDog.age = 3;

    myDog.bark = dog_bark;       // 初始化函数指针
    myDog.eat  = dog_eat;

    myDog.bark();                // 旺财:汪汪汪!
    myDog.eat("骨头");            // 吃骨头
    return 0;
}
```

- 调用方式等价:`myDog.bark()` 与 `(*myDog.bark)()`。
- 本课为最基础的"直接成员赋值"方式(尚未引入 self 指针,第3章开始引入)。

### 2.3 生活类比:遥控器

- 遥控器 = 结构体;遥控器上的按钮 = 函数指针成员;按按钮 = 调用函数。
- 按钮指向哪个电器(函数),按下时就执行哪个行为。

### 2.4 传统 C vs 结构体+函数指针

| 维度 | 传统 C | 结构体 + 函数指针 |
|---|---|---|
| 数据 | 全局变量,分散 | 封装在结构体内 |
| 函数 | 独立,与数据分离 | 通过函数指针与数据关联 |
| 调用 | `function(data)` | `object.method()` |
| 关系 | 数据和行为分离 | 数据和行为封装 |

### 要点总结

- 口诀:**结构体 = 类,函数指针成员 = 方法,结构体实例 = 对象**。
- 函数指针成员的声明格式:`返回类型 (*指针名)(参数类型)`。

### 思考练习

定义 `struct Student`,包含姓名、学号、以及 `printInfo` 行为(函数指针),打印学生信息。

---

## 第3章 C 语言 OOP 总览:LED 驱动架构

**学习目标**:通过嵌入式 LED 驱动,掌握用纯 C 实现面向对象的核心手法:**封装(struct)、行为(函数指针)、self 指针、依赖注入、模拟构造函数**。

### 3.1 三层架构

```
┌─────────────────────────────┐
│   LED Handler 层(业务逻辑)   │
├─────────────────────────────┤
│   LED Driver 层(驱动类)      │
├─────────────────────────────┤
│   硬件抽象层(GPIO / 时间服务) │
└─────────────────────────────┘
```

目标:模块化、可复用、硬件无关。

### 3.2 LED 驱动"类"的设计

```c
typedef struct
{
    void (*pf_led_on)(void);    // LED 开启
    void (*pf_led_off)(void);   // LED 关闭
} led_drv_ops_t;                // 硬件操作接口(函数指针表)

typedef struct
{
    void (*pf_delay_ms)(uint32_t ms);
} led_drv_time_t;               // 时间服务接口

typedef struct bsp_led_drv
{
    led_drv_attr_t  attr;                       // 属性成员:闪烁次数、周期、占空比
    led_drv_ops_t*  p_ops;                      // 硬件操作接口(依赖注入)
    led_drv_time_t* p_time_base;                // 时间服务接口(依赖注入)
    led_drv_status_t (*pf_led_controler)(struct bsp_led_drv* const self,
                                         led_drv_attr_t* const attr);  // 成员方法
} bsp_led_drv_t;
```

### 3.3 self 指针(C 语言的 this)

- 每个"成员函数"的第一个参数显式传入 `self`,相当于 C++ 的 `this` 指针。
- 通过 `self->p_ops->pf_led_on()` 访问对象自己的硬件接口。

### 3.4 模拟构造函数

C 语言没有构造函数,用初始化函数 `bsp_led_drv_inst()` 模拟,共五步:

1. **参数检查**(防御性编程,`assert`)
2. **状态检查**(`is_inited` 标志,防止重复初始化)
3. **依赖注入**(绑定 `p_ops`、`p_time_base`)
4. **绑定成员方法**(`self->pf_led_controler = led_controler;`)
5. **初始化属性并置 `is_inited = true`**

### 3.5 使用示例

```c
bsp_led_drv_t led1, led2, led3;     // 创建多个 LED 对象
// 共享同一份 led_ops 和 time_service(依赖注入)

led1.pf_led_controler(&led1, &config);   // 显式传 self
led2.pf_led_controler(&led2, &config);
led3.pf_led_controler(&led3, &config);
```

### 要点总结

- **结构体 = 类,函数指针 = 成员方法,self 指针 = this,初始化函数 = 构造函数,依赖注入 = 灵活性**。
- 通过函数指针实现硬件抽象,驱动代码与具体硬件解耦。
- 防御性编程:参数校验、`is_inited` 防重复初始化。

---

## 第4章 UART 设备类 OOP 实战

**学习目标**:把前面几课落到真实单片机外设(UART 串口),实现工程级的"一个实现、无限对象"的设备类。

### 4.1 传统 C 写法的痛点

- 每个外设都要复制一套 `uart1_init` / `uart1_send` / `uart2_init` / `uart2_send`……
- 面向对象写法:一份实现,`UART_Device_Create(USART3, ...)` 一行就能创建新对象。

### 4.2 UART_Device 类

```c
typedef struct UART_Device UART_Device;

struct UART_Device
{
    // 数据成员
    USART_TypeDef* hw;              // 寄存器基地址
    uint32_t       baudrate;
    uint8_t        tx_buffer[256];
    uint8_t        rx_buffer[256];

    // 方法成员(函数指针)
    void (*init)(UART_Device* self);
    void (*send)(UART_Device* self, const char* data, int len);
    int  (*recv)(UART_Device* self, uint8_t* buf, int max);
    void (*deinit)(UART_Device* self);
};
```

> 每创建一个对象,就有一份独立的数据副本,互不干扰。

### 4.3 this 指针(对象的身份证)

- 每个函数指针的**第一个参数必须是 `UART_Device* self`**——这是方法的灵魂。
- 核心问题:`uart1->send()` 怎么知道操作的是 uart1 还是 uart2?答案就是 this 指针。
- 类比:ATM 取款必须先"插卡"(传入 this 指针)告诉机器"我是张三",ATM 才知道从张三的账户取钱。

### 4.4 构造函数 / 工厂

```c
UART_Device* UART_Device_Create(USART_TypeDef* hw, uint32_t baud)
{
    UART_Device* dev = malloc(sizeof(UART_Device));  // ① 分配内存
    dev->hw = hw;                                    // ② 初始化数据成员
    dev->baudrate = baud;
    dev->init = uart_init_impl;                      // ③ 绑定方法
    dev->send = uart_send_impl;
    dev->recv = uart_recv_impl;
    dev->deinit = uart_deinit_impl;
    return dev;
}
```

### 4.5 方法实现(真实寄存器操作)

```c
void uart_init_impl(UART_Device* self)
{
    self->hw->CR1 |= USART_CR1_UE;
    uint32_t div = 42000000 / (16 * self->baudrate);   // 波特率分频
    self->hw->BRR = div;
    self->hw->CR1 |= USART_CR1_TE | USART_CR1_RE;
}

void uart_send_impl(UART_Device* self, const char* data, int len)
{
    int i;

    for (i = 0; i < len; i++)
    {
        while (!(self->hw->SR & USART_SR_TXE))
        {
            ;    // 等待发送寄存器空
        }
        self->hw->DR = data[i];
    }
}
```

### 4.6 C++ 风格的最终调用

```c
UART_Device* uart1 = UART_Device_Create(USART1, 115200);
UART_Device* uart2 = UART_Device_Create(USART2, 9600);

uart1->init(uart1);
uart1->send(uart1, "Hello UART1", 11);
```

可再用宏简化调用:

```c
#define UART_INIT(obj)      ((obj)->init(obj))
#define UART_SEND(obj, d, l) ((obj)->send(obj, d, l))
```

### 要点总结

- 同一份函数实现,通过不同的 `self` 参数操作不同的对象/硬件 → **代码复用 + 数据隔离**,这就是 OOP 精髓。
- 构造函数三部曲:分配内存 → 初始化数据 → 绑定方法。
- 下一步:继承、多态、虚函数表(见第12、13章)。

---

## 第5章 封装:不透明指针

**学习目标**:通过不透明指针 + 前向声明 + .h/.c 文件分离,把结构体内部成员隐藏起来,只暴露操作接口——这是用 C 模拟"类"的第一步。

### 5.1 核心思想:菜单 vs 后厨配方

- 用户只看**菜单**(接口),看不到**后厨配方**(实现细节)。
- 公式:**隐藏实现 + 暴露接口 = 封装**。

### 5.2 反面教材:信息泄露

把结构体直接写在头文件里:

```c
// door.h(反面教材)
struct Door
{
    int  password;
    int  open_count;
    bool is_locked;
};
```

调用者可以直接 `door.password = 9999` 篡改内部数据 → 破坏封装、无法约束。

### 5.3 正确做法:.h/.c 分离 + 前向声明

`door.h`(头文件只放前向声明 + 接口原型):

```c
// door.h
#ifndef DOOR_H
#define DOOR_H

#include <stdbool.h>

typedef struct Door Door;   // 前向声明:告诉编译器存在这样一个类型

Door* door_create(int password);
bool  door_open(Door* d, int pwd);
void  door_close(Door* d);
void  door_destroy(Door* d);

#endif /* DOOR_H */
```

`door.c`(结构体完整定义藏在源文件里):

```c
// door.c
#include "door.h"
#include <stdlib.h>

struct Door
{
    int  password;
    int  open_count;
    bool is_locked;
};

Door* door_create(int password)
{
    Door* d = malloc(sizeof(Door));
    d->password   = password;
    d->open_count = 0;
    d->is_locked  = true;
    return d;
}

bool door_open(Door* d, int pwd)
{
    if (d->password == pwd)
    {
        d->is_locked = false;
        d->open_count++;
        return true;
    }
    return false;
}

void door_close(Door* d)
{
    d->is_locked = true;
}

void door_destroy(Door* d)
{
    free(d);
}
```

`main.c`(使用示例):

```c
#include "door.h"

int main(void)
{
    Door* d = door_create(1234);

    door_open(d, 1234);
    door_close(d);
    door_destroy(d);
    return 0;
}
```

### 5.4 为什么"指针就够了"

- 指针的大小固定(32 位下 4 字节,64 位下 8 字节)。
- 编译器无需知道结构体完整定义,就能操作指针 → 把结构体内部布局彻底隐藏。

### 5.5 接口三段式生命周期

```
Create(创建) → Operate(操作) → Destroy(销毁)
(构造函数)     (成员方法)      (析构函数)
```

### 附带收益

- 信息隐藏
- 灵活维护:改实现不用重新编译调用方
- 编译优化

### 要点总结

1. 前向声明 `typedef struct Door Door;` + 接口原型放 .h,完整定义藏 .c。
2. 调用方无法直接访问成员,只能通过接口函数操作。
3. 生命周期三段式:Create / Operate / Destroy。

### 思考练习

仿照门模块,编写**银行账户模块**:`bank_account.h` / `bank_account.c` / `main.c`,包含创建账户、存款、取款、查询余额、销毁账户。

---

## 第6章 构造与析构函数

**学习目标**:掌握对象生命周期管理:构造(Init)负责资源获取与初始化,析构(DeInit)负责资源清理,以及嵌入式 C 的标准实现模式。

### 6.1 对象生命周期三阶段

```
出生(构造) → 成长(使用) → 死亡(析构)
分配内存    执行功能    清理资源
初始化资源   处理数据    释放内存
```

### 6.2 构造函数 / Init 的四大职责

1. **内存分配**(像给新生儿准备房间)
2. **初始化成员**
3. **资源获取**(打开文件、建立连接、申请硬件)
4. **状态验证**(检查初始化是否成功)

### 6.3 析构函数 / DeInit 为什么重要

- 类比:人离开房间要"关灯锁门"。
- 善后不到位会导致:**内存泄漏、资源占用、数据丢失**。
- 析构四步:保存状态 → 关闭连接 → 释放资源 → 清理内存。

### 6.4 嵌入式 C 标准 Init / DeInit 设计模式

```c
typedef struct
{
    uint8_t   initialized;   // 状态标志:防止重复初始化
    void*     resource;      // 持有的资源
    uint32_t  config;
} Module_t;

// 返回错误码便于错误处理:
//   0 = 成功
//  -1 = 已初始化
//  -2 = 内存分配失败
int  Module_Init(Module_t* module, uint32_t config);

// 释放资源、重置状态、幂等(多次调用安全)
void Module_DeInit(Module_t* module);
```

设计要点:
- 用 `initialized` 状态标志**防止重复初始化**。
- `Init` 返回**错误码**。
- `DeInit` 保证**幂等性**(多次调用安全)。
- 两者**配对使用、责任对等**。

### 6.5 最佳实践

- **RAII 原则**:资源获取即初始化。
- **异常安全**:构造失败要回滚;析构不抛异常。
- **顺序原则**:初始化按 基类→成员→自身;析构按 自身→成员→基类。
- **防御编程**:检查空指针、验证状态、记录日志。

### 要点总结

1. Init = 准备资源(分配/初始化/获取/验证);DeInit = 清理资源(保存/关闭/释放/清理)。
2. 用状态标志防重复初始化,用错误码反馈失败,DeInit 必须幂等。
3. Init/DeInit 配对使用,责任对等。

---

## 第7章 静态对象数组

**学习目标**:用结构体数组(静态对象数组)批量管理多个同类型"对象",掌握其内存布局、访问方式和适用场景。

### 7.1 结构体 = 对象的"设计图纸"

```c
typedef struct
{
    uint8_t port;    // 端口号
    uint8_t pin;     // 引脚号
    uint8_t state;   // 状态
} LED_t;
```

### 7.2 为什么需要数组

- 10 个单独变量(`led0` ~ `led9`)→ 代码重复、无法循环批量操作、难以维护。
- 改用**对象数组** → 简洁、可循环、易扩展。

### 7.3 声明与初始化

```c
LED_t leds[4] = {
    {1, 0, 0},
    {1, 1, 0},
    {2, 0, 0},
    {2, 1, 0}
};
```

### 7.4 内存布局

- 数组元素在内存中**连续存储**。
- 每个元素是完整对象(例:3 个 `uint8_t` 字段 → 每对象 3 字节,地址依次 `0x1000 / 0x1003 / 0x1006 / 0x1009`)。

### 7.5 访问与操作

```c
leds[2].state = 1;          // 下标访问单个对象

for (i = 0; i < 4; i++)     // for 循环批量操作
{
    leds[i].state = 1;      // 全部点亮
}
```

### 7.6 优缺点对比

| 优点 | 限制 |
|---|---|
| 内存固定(编译期分配) | 大小固定,不可扩展 |
| 访问快速(无动态分配) | 即使不用也占内存 |
| 使用简单(无需手动 free) | 大数组可能栈溢出 |
| 可批量操作 | — |

**适用场景**:对象数量固定(8 个 GPIO、4 个 LED)、嵌入式系统(确定性内存分配)、配置表(传感器/通信端口)、小型数组(一般 < 100 个)。

### 7.7 实战案例:GPIO 端口管理

```c
typedef struct
{
    uint8_t port;    // A=0, B=1, C=2...
    uint8_t pin;     // 0-15
    uint8_t mode;    // 0=输入, 1=输出
    uint8_t state;   // 0=低, 1=高
} GPIO_Pin_t;

GPIO_Pin_t gpio_pins[8] = { ... };   // 8 引脚配置表

void GPIO_Init_All(void)
{
    int i;

    for (i = 0; i < 8; i++)
    {
        /* 逐个初始化 gpio_pins[i] */
    }
}

void LED_Control(uint8_t index, uint8_t state)
{
    if (index < 8 && gpio_pins[index].mode == 1)
    {
        gpio_pins[index].state = state;
    }
}
```

### 要点总结

1. 静态对象数组:批量管理、循环访问、确定性内存。
2. 数组元素连续存储,每个元素是一个完整对象。
3. 适用固定数量对象;注意大小固定和栈溢出的限制。

### 思考练习

用静态对象数组设计一个**停车场管理系统**:10 个车位,每个车位有编号、占用状态、车牌号。

---

## 第8章 对象池

**学习目标**:掌握对象池设计模式——预分配一批对象并缓存复用,避免频繁 malloc/free 的开销,是嵌入式/高性能场景的内存优化手段。

### 8.1 性能动机

- 传统 `malloc/free`(🐌)动态分配慢,比对象池慢 **10~100 倍**。
- 对象池(🚀)通过**预分配 + 复用**大幅提速。

### 8.2 生活类比:共享单车

- 对象池 = 共享单车停放点。
- **分配 = 借车,使用 = 骑车,回收 = 还车**。
- 还车不销毁单车,只是放回停放点等待下次借出。

### 8.3 核心思想

- **不真正 free 内存**。
- 通过 `is_free` 标志标记对象"空闲 / 使用中"。
- 回收只是把标志置回空闲,内存块持续复用。

### 8.4 数据结构与核心 API

```c
typedef struct
{
    int  is_free;    // 1 = 空闲, 0 = 使用中
    char data[256];  // 实际数据
} Object;

typedef struct
{
    Object* pool;       // 对象数组指针
    int     pool_size;  // 池容量
    int     used_count; // 已用数量
} ObjectPool;

ObjectPool* pool_init(int size);               // 预分配 + 全部置空闲
Object*     pool_alloc(ObjectPool* p);         // 找空闲块,标记占用,返回指针;池满返回 NULL
void        pool_free(ObjectPool* p, Object* obj);  // 指针范围校验 + 标记空闲
void        pool_destroy(ObjectPool* p);       // free 对象数组 + 池结构
```

```c
ObjectPool* pool_init(int size)
{
    ObjectPool* p = malloc(sizeof(ObjectPool));
    int         i;

    p->pool       = malloc(sizeof(Object) * size);
    p->pool_size  = size;
    p->used_count = 0;
    for (i = 0; i < size; i++)
    {
        p->pool[i].is_free = 1;
    }
    return p;
}

Object* pool_alloc(ObjectPool* p)
{
    int i;

    for (i = 0; i < p->pool_size; i++)
    {
        if (p->pool[i].is_free)
        {
            p->pool[i].is_free = 0;
            p->used_count++;
            return &p->pool[i];
        }
    }
    return NULL;    // 池已满
}

void pool_free(ObjectPool* p, Object* obj)
{
    // 指针范围校验:确保 obj 属于本池
    if (obj >= p->pool && obj < p->pool + p->pool_size)
    {
        obj->is_free = 1;
        p->used_count--;
    }
}
```

### 8.5 完整使用流程

```
pool_init() → pool_alloc() → 使用对象 → pool_free() → 复用!
```

- 同块内存反复分配复用 → 性能提升 10~100 倍。

### 要点总结

1. 预分配 + 复用,避免动态分配开销。
2. `is_free` 标志管理对象状态,回收不真正释放内存。
3. `pool_alloc` 池满返回 NULL;`pool_free` 做指针范围校验。

---

## 第9章 单例模式

**学习目标**:掌握单例模式——保证某个对象在程序运行期间只有一个实例,所有调用者共享访问它。

### 9.1 核心口号

**唯一实例 + 全局访问**。

### 9.2 生活类比

- 国家总统:全中国只有一位。
- 公司 CEO、唯一的硬件设备(打印机)。

### 9.3 static 关键字的作用

- 普通局部变量:生命周期短,函数结束即销毁。
- **static 变量**:只初始化一次,存在于程序整个生命周期,始终保持状态。

### 9.4 getInstance 四步流程

```
调用函数 → 检查标志 → 初始化(仅一次) → 返回地址
```

### 9.5 打印机单例实现

```c
typedef struct
{
    int port;        // 端口号(并行口)
    int pageCount;   // 已打印页数
} Printer;

Printer* getPrinterInstance(void)
{
    static Printer instance;      // 函数内静态局部变量:生命周期贯穿程序
    static int initialized = 0;   // 初始化保护标志

    if (!initialized)
    {
        instance.port = 0x378;
        instance.pageCount = 0;
        initialized = 1;
    }
    return &instance;
}
```

### 9.6 使用与验证

```c
Printer* p1 = getPrinterInstance();
Printer* p2 = getPrinterInstance();
// p1 == p2 ✅ 都指向 0x00401000
```

- 问题演示:3 个程序员各自创建打印机对象 → 混乱爆炸 💥。
- 单例解决:所有程序员通过箭头指向中心唯一实例。
- 初始化保护:第 1 次调用执行初始化,第 2 次及以后跳过。

### 9.7 应用场景

- 硬件管理、日志系统、配置管理、资源池。

### 要点总结

1. 单例 = 全局唯一 + 共享访问。
2. 用函数内 `static` 变量承载单例对象,用标志位控制一次性初始化。
3. 所有调用者拿到同一份实例(p1 == p2)。

---

## 第10章 深拷贝与浅拷贝

**学习目标**:理解结构体拷贝在含指针成员时的危险,掌握深拷贝与浅拷贝的区别、陷阱和选用原则。

### 10.1 拷贝的触发

```c
Person p2 = p1;    // 结构体整体赋值
```

关键问题:`p2.name` 指向新内存,还是与 `p1` 共享同一块内存?

### 10.2 浅拷贝:只复制地址

```c
typedef struct
{
    char* name;    // 指针是关键字!
    int   age;
} Person;

Person shallow_copy(Person* src)
{
    Person dest;

    dest.name = src->name;    // 只复制地址,不复制内容
    dest.age  = src->age;
    return dest;
}
```

- `p2.name` 与 `p1.name` 指向同一块堆内存 → **两个指针共享数据**。
- 类比:两人共用一个银行账户。

### 10.3 浅拷贝的三大陷阱

1. **意外修改**:改 `p1.name`,`p2.name` 也跟着变。
2. **重复释放**:对 `p1.name`、`p2.name` 两次 `free()` → 崩溃。
3. **悬空指针**:`free(p1.name)` 后,`p2.name` 指向非法内存,再 `printf` 就崩溃。

### 10.4 深拷贝:数据完全独立

```c
Person deep_copy(Person* src)
{
    Person dest;
    int   len;

    dest.age = src->age;
    len = strlen(src->name) + 1;
    dest.name = malloc(len);
    if (dest.name != NULL)
    {
        strcpy(dest.name, src->name);   // 复制内容
    }
    return dest;
}
```

深拷贝三步口诀:**malloc 分配新内存 → strcpy 复制数据 → 指向新地址**。

### 10.5 深拷贝 vs 浅拷贝

| 维度 | 浅拷贝 | 深拷贝 |
|---|---|---|
| 复制内容 | 只复制地址 | 复制完整数据 |
| 数据独立 | 否(共享) | 是 |
| 修改影响 | 互相影响 | 互不影响 |
| 速度 | 快 | 较慢 |
| 内存 | 少 | 多 |
| 安全性 | 低 | 高 |

### 10.6 选用原则与嵌入式场景

- **只读 / 临时 → 浅拷贝;修改 / 销毁 → 深拷贝**。
- 嵌入式应用场景(都用深拷贝):传感器数据缓冲、配置参数备份、RTOS 消息队列。
- 结论:**安全性 > 内存效率**。

### 要点总结

1. 含指针成员的结构体拷贝,浅拷贝只复制地址,共享数据。
2. 浅拷贝三大陷阱:意外修改、重复释放、悬空指针。
3. 深拷贝三步:malloc → strcpy → 指向新地址。

---

## 第11章 内存对齐

**学习目标**:理解为什么要内存对齐、未对齐的代价、不同数据类型的对齐规则,以及如何通过调整结构体成员顺序节省内存。(本章根据课件图片整理)

### 11.1 为什么需要对齐:CPU 一次读取

- CPU 通过**数据总线**(32 位 / 64 位)**一次读取 4 / 8 字节**。
- 内存以字节为单位编址:`0x00 0x01 0x02 0x03 ...`。

```
┌───────┐      数据总线(32/64位)      ┌──────────────────┐
│ CPU   │ ──────────────────────────> │ 内存 Memory      │
│ 处理器 │     一次读取 4/8 字节        │ 0x00 0x01 0x02 0x03 ... │
└───────┘                            │ D0  D1  D2  D3  ... │
                                     └──────────────────┘
```

- **对齐的 int**(从 `0x00` 开始,占 `0x00~0x03` 共 4 字节):CPU 一次读取完成 ✅。
- 若 int 数据**跨越两个读取边界**,CPU 需要读取两次才能取完整 → 效率下降。

```
0x00 0x01 0x02 0x03 0x04 0x05 0x06 0x07
┌────────────────────┬────────────────────┐
│ 第1次读取 (取头)     │ 第2次读取 (取尾)     │
│ DATA  ←─ 数据跨越边界 →  DATA            │
└────────────────────┴────────────────────┘
  未对齐的 int:需要两次读取
```

### 11.2 对齐规则一览

| 类型 | 占用大小 | 对齐边界 |
|---|---|---|
| `char` | 1B | 1 字节(任意地址) |
| `short` | 2B | 2 字节(偶数地址) |
| `int` / `float` | 4B | 4 字节(4 的倍数地址) |
| `double` | 8B | 8 字节(8 的倍数地址) |

### 11.3 反面教材:BadStudent

```c
struct BadStudent
{
    char grade;     // 1 字节
    int  id;        // 4 字节
    char gender;    // 1 字节
};
// 预期:1 + 4 + 1 = 6 字节
// 实际:12 字节!
```

内存布局(0~11 字节):

```
 0    1    2    3    4    5    6    7
[G ] [P ] [P ] [P ] [I ] [I ] [I ] [I ]
 8    9    10   11
[g ] [P ] [P ] [P ]

G = grade(1B)  P = Padding 填充(浪费)  I = id(4B)  g = gender(1B)
```

- `char grade` 占 1 字节后,int 需要 4 字节对齐,因此补 3 字节填充。
- `gender` 之后,为保证结构体总大小是最大对齐(4)的整数倍,再补 3 字节。
- 结果:**6 字节数据 + 6 字节填充 = 12 字节**。

### 11.4 正面教材:GoodStudent(调整成员顺序)

```c
struct GoodStudent
{
    int  id;        // 4 字节
    char grade;     // 1 字节
    char gender;    // 1 字节
};
// 实际:8 字节!
// 节省:4 字节(33%)
```

内存布局(0~7 字节):

```
 0    1    2    3    4    5    6    7
[I ] [I ] [I ] [I ] [G ] [g ] [P ] [P ]

I = id(4B)  G = grade(1B)  g = gender(1B)  P = Padding
```

- 把 4 字节的 `int` 放在最前面,两个 `char` 挨在一起,只留 2 字节尾部填充。
- 结果:**8 字节,节省 4 字节(33%)**。

### 11.5 对比总结

| | ❌ 未优化的排列(BadStudent) | ✅ 优化后的排列(GoodStudent) |
|---|---|---|
| 成员顺序 | `char, int, char` | `int, char, char` |
| 实际大小 | 12 字节 | 8 字节 |
| 填充浪费 | 6 字节 | 2 字节 |
| 结论 | 浪费空间 | 节省 4 字节(33%) |

### 11.6 补充知识

> 以下为课件之外的标准内存对齐补充,便于理解课件示例。

- **对齐规则**:结构体成员按其类型对齐边界对齐;结构体总大小为最大成员对齐边界的整数倍。
- **`#pragma pack(n)`**:可修改对齐方式(如 `#pragma pack(1)` 强制紧凑排列,节省空间但可能牺牲访问效率,嵌入式跨通信协议时常用)。
- **指针大小**:32 位平台 4 字节,64 位平台 8 字节,指针的对齐边界相应为 4 / 8。
- **对齐的意义**:对齐访问一次完成,未对齐访问需要拆分成多次,且部分架构(如某些 ARM)会直接产生异常。

### 要点总结

1. CPU 一次读取 4/8 字节,对齐的数据一次读取完成,未对齐的需要两次。
2. 对齐规则:char 1B、short 2B、int/float 4B、double 8B。
3. 结构体内存 = 数据 + 填充(Padding),总大小按最大成员对齐。
4. 优化技巧:**把大的成员放前面、小的成员聚在一起**,可显著节省内存。

### 思考练习

分析以下结构体在 32 位平台上的大小,并通过调整成员顺序优化:

```c
struct Data
{
    char  a;
    double b;
    char  c;
    int   d;
};
```

---

## 第12章 继承

**学习目标**:掌握用 C 语言实现继承的两种方法:结构体嵌套(父类作第一个成员)和函数指针(方法继承/重写)。

### 12.1 核心问题

- C 语言不是面向对象语言,但实际开发(Linux 内核、嵌入式)经常需要"继承"功能。
- 类比:跑车**继承**汽车的全部特性(品牌、颜色、行驶),再新增最高时速、加速。

### 12.2 方法一:结构体嵌套(父类作第一个成员)

```c
typedef struct
{
    char brand[50];
    char color[20];
    int  speed;
} Vehicle;

typedef struct
{
    Vehicle base;        // 关键!父类作为第一个成员
    int     maxSpeed;
    int     acceleration;
} SportsCar;
```

- 内存布局关键:父类字段排在子类对象内存的**最前面**,保证首地址对齐,可向上转型。
- `(Vehicle*)&sportsCar` 可安全当作父类指针使用。

### 12.3 方法二:函数指针实现方法继承(虚函数模拟)

```c
typedef struct Vehicle
{
    char brand[50];
    char color[20];
    int  speed;
    void (*drive)(struct Vehicle* self);     // 方法(传 self 模拟 this)
    void (*display)(struct Vehicle* self);
} Vehicle;

typedef struct
{
    Vehicle base;        // 可重写 base 里的函数指针
    int     turboMode;
} SportsCar;
```

- 父类方法:`vehicle->drive(vehicle)`。
- 子类重写:`sportsCar->base.drive = sportsCar_drive;`。
- 多态调用:同样的接口,不同的实现。

### 12.4 完整示例:动物继承体系

```c
typedef struct Animal
{
    char name[30];
    int  age;
    void (*makeSound)(struct Animal* self);
} Animal;

typedef struct
{
    Animal base;
    char   breed[30];
} Dog;                    // dog_sound():汪汪汪!

typedef struct
{
    Animal base;
    int   lives;
} Cat;                    // cat_sound():喵喵喵!
```

```c
Animal* animals[] = {
    (Animal*)&myDog,
    (Animal*)&myCat
};

for (i = 0; i < 2; i++)
{
    animals[i]->makeSound(animals[i]);   // 多态!
}
// 输出:旺财: 汪汪汪! / 咪咪: 喵喵喵!
```

### 12.5 优缺点与应用场景

**优点**:代码复用性高、结构清晰易维护、可实现多态、内存布局可控。

**注意事项**:类型转换需谨慎、函数指针初始化别忘记、**内存对齐要注意**(见第11章)、没有编译器的类型检查。

**应用场景**:Linux 内核(设备驱动模型)、嵌入式系统(硬件抽象层)、游戏引擎(实体组件系统)、网络协议栈(协议分层实现)。

### 要点总结

1. 方法一:父类结构体作子类第一个成员,实现数据继承与向上转型。
2. 方法二:函数指针成员实现方法继承与重写,传 self 模拟 this。
3. 父类字段前置保证首地址对齐,是内存布局可控的根基。

---

## 第13章 多态

**学习目标**:掌握用函数指针 + 工厂函数 + 虚函数表在 C 语言中实现多态。

### 13.1 什么是多态

**同一个行为,不同的对象有不同的表现方式**:都是"叫",狗汪汪 / 猫喵喵 / 鸟叽叽。

### 13.2 传统 C 的困境:if-else 按类型分发

```c
void make_sound(int type)
{
    switch (type)
    {
    case DOG: printf("汪汪汪\n"); break;
    case CAT: printf("喵喵喵\n"); break;
    case BIRD: printf("叽叽喳喳\n"); break;
    }
}
```

三大问题:
1. **扩展性差**:每加一种动物都要改函数。
2. **维护困难**:代码越来越臃肿。
3. **违反开闭原则**:对扩展开放、对修改关闭。

### 13.3 用函数指针 + 结构体实现多态

```c
typedef struct Animal
{
    char name[20];
    void (*makeSound)(struct Animal* self);   // 多态方法
    void (*move)(struct Animal* self);
} Animal;

void dogSound(struct Animal* self)
{
    printf("%s说:汪汪汪!\n", self->name);
}

void dogMove(struct Animal* self)
{
    printf("%s在快速奔跑!\n", self->name);
}

void catSound(struct Animal* self)
{
    printf("%s说:喵喵喵~\n", self->name);
}

void catMove(struct Animal* self)
{
    printf("%s在优雅地走路\n", self->name);
}

// 工厂函数:返回填好函数指针的 Animal 对象
Animal createDog(const char* name)
{
    Animal dog;

    strcpy(dog.name, name);
    dog.makeSound = dogSound;
    dog.move      = dogMove;
    return dog;
}

// 统一处理:接收"父类"指针,内部实现多态
void performActions(Animal* animal)
{
    animal->makeSound(animal);
    animal->move(animal);
}
```

```c
Animal dog = createDog("旺财");
Animal cat = createCat("小花");

performActions(&dog);   // 旺财说:汪汪汪! / 旺财在快速奔跑!
performActions(&cat);   // 小花说:喵喵喵~ / 小花在优雅地走路
```

### 13.4 进阶:虚函数表(vtable)

把所有函数指针集中管理,模拟 C++ 虚函数表:

```c
typedef struct
{
    void (*makeSound)(void* self);
    void (*move)(void* self);
    void (*eat)(void* self);
} VTable;

typedef struct
{
    const VTable* vtable;    // 虚函数表指针(放在对象开头)
    char name[20];
} Animal;

static const VTable dogVTable = {
    .makeSound = dogSound,
    .move      = dogMove,
    .eat       = dogEat
};

#define CALL(obj, method, ...) \
    ((obj)->vtable->method((obj), ##__VA_ARGS__))
```

- 父类字段 / 虚表指针放在对象开头,保证类型强转后首地址一致。
- 与第12章"父类字段前置"、第11章"内存对齐"直接相关。

### 要点总结

1. 多态 = 同一接口,不同对象不同表现。
2. 函数指针是运行时动态决定调用哪个函数的关键。
3. 结构体封装数据 + 函数指针,工厂函数创建对象,统一函数接收父类指针 → 多态。
4. 进阶:用 vtable 集中管理虚函数,更接近 C++ 的实现。

### 思考练习

在动物例子的基础上增加一个 `Bird`(鸟)类:叫声"叽叽喳喳"、动作"扑腾翅膀",并通过 `performActions` 统一调用。
