# Separate SM

## 简介
简洁且独立的状态机内核，为业务逻辑提供及其清晰的代码框架基础，极度适合状态管理复杂的单片机项目，也可用于RTOS操作系统下的单任务或单线程内的代码框架。

## 为什么叫做：Separate SM ？

Separte SM全称 Separte State Machine；如同其名，内核将每个状态都分离的放在了独立的函数中运作，从而使得每个状态都独立，互不影响，从而使得状态之间的切换也变得简单，清晰，高效。并且高度自由的接口，使得用户可以轻松的扩展，专注于业务实现。

#### 易于移植的特性
Separte SM使用了高度封装的宏，在保留通用性的同时，将内核极致的压缩到了简单的一个函数，用户只需要简单的一句调用，就可以轻松的实现内核的运行，快速的移植可让用户专注于业务代码。

## 使用示例
与传统的状态机相比设计思路上最大的不同：Separte SM是以最便捷的使用性为目标来完成的框架设计。

### 1.内核的初始化与定义
Separte SM 是通过上下文切换变量来管理状态，每个状态都对应一个函数，当状态机切换到该状态时，就会调用该函数，从而实现状态的切换。

用户只需要做以下初始化操作：
  ``````
 1.定义一个枚举类型的上下文状态切换量
 2.（可选）申明未找到状态时的回调函数
 3.调用初始化内核宏
  ``````
示例代码：
  ``````
enum ENUM_WORKSTATE
{
    LED_INIT,
    LED_ON,
    LED_OFF,
};

//状态机上下文切换量
ENUM_WORKSTATE WorkStat = LED_INIT;

//定义每个状态下执行的函数
const static SEPARATE_STATE BusinessList[] = 
{
    SEPARATE_DEFINE_STATE(LED_INIT, led_init),
    SEPARATE_DEFINE_STATE(LED_ON, led_on),
    SEPARATE_DEFINE_STATE(LED_OFF, led_off),
};

//默认未找到状态的回调函数（WorkStat非法值的容错处理-可选）
static void SEPARATE_DEFAULT_NOT_FOUND_CALLBACK() 
{
    WorkStat = LED_INIT;
}

// 初始化内核
SEPARATE_INIT_KERNEL(BusinessList,       \
                ENUM_WORKSTATE,        \
                &WorkStat,             \
                SEPARATE_DEFAULT_NOT_FOUND_CALLBACK); //可选 func or null
  ``````
### 2.内核调用
 在主函数中非常简单的调用即可实现一个完整的状态机切换。
  ``````
void main()
{
    // 运行内核 
    SEPARATE_RUN_KERNEL(1); //是否自行管理循环 0：否 1：是
}
  ``````


### 3.高度分离的业务逻辑
业务逻辑与状态机内核解耦，每个状态都对应一个函数，用户只需要专注于每个状态函数中的业务逻辑，即可完成状态机内核的调用。
#### 1.简单状态机
 ``````
void led_init()
{
    led.init();
    WorkStat = LED_ON;
}

void led_on()
{
    led.on();
    delay(500);
    WorkStat = LED_OFF;
}

void led_off()
{
    led.off();
    delay(500);
    WorkStat = LED_ON;
}

 ``````
##### 1.复杂业务

针对更复杂的状态业务应用，在每个状态下实现独立的业务逻辑。
 ``````
int blick_count = 0;
void led_on()
{
    //进入led_on的业务初始化代码块
    {
        blick_count = 0;
    }

    while(1)
    {
        blick_count++;

        if(blick_count <20)
        {
            led.onGreen();
            delay(500);
            led.onRed();
            delay(500);
        }
        else
        {
            break; //跳出状态函数
        }
    }

    {
        //退出led_on业务的状态切换与资源清理代码块
        WorkStat = LED_OFF;
    }

}
 ``````



### 代码结构
src
  - separate_core.h   内核头文件
  - separate_core.c   内核实现文件
