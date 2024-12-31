# Separate SM

## Introduction
A concise and independent state machine core that provides a clear code framework foundation for business logic, making it extremely suitable for complex state management in microcontroller projects. It can also be used within a single task or thread under an RTOS operating system.

## Why is it called: Separate SM?

The full name of Separate SM is Separate State Machine. As the name suggests, the core operates each state in its own isolated function, ensuring that each state is independent and unaffected by others. This makes state transitions simple, clear, and efficient. The highly flexible interfaces allow users to easily extend and focus on business implementation.

#### Portability Characteristics
Separate SM uses highly encapsulated macros, maintaining generality while compressing the core to a simple function. Users only need a simple call to run the core, enabling quick porting so they can focus on business code.

## Usage Example
Compared to traditional state machines, the biggest difference in design philosophy is that Separate SM aims for the most convenient usability in framework design.

### 1. Initialization and Definition of the Core
Separate SM manages states through context-switching variables. Each state corresponds to a function, which is called when the state machine transitions to that state, thereby achieving state transitions.

Users only need to perform the following initialization operations:
  ``````
1.Define an enumeration type for the context-switching variable.
2.(Optional) Declare a callback function for handling unregistered states.
3.Call the initialization macro for the core.  
 ``````
 Example Code:
  ``````
enum ENUM_WORKSTATE
{
    LED_INIT,
    LED_ON,
    LED_OFF,
};


ENUM_WORKSTATE WorkStat = LED_INIT;


const static SEPARATE_STATE BusinessList[] = 
{
    SEPARATE_DEFINE_STATE(LED_INIT, led_init),
    SEPARATE_DEFINE_STATE(LED_ON, led_on),
    SEPARATE_DEFINE_STATE(LED_OFF, led_off),
};


static void SEPARATE_DEFAULT_NOT_FOUND_CALLBACK() 
{
    WorkStat = LED_INIT;
}

// init kernel
SEPARATE_INIT_KERNEL(BusinessList,       \
                ENUM_WORKSTATE,        \
                &WorkStat,             \
                SEPARATE_DEFAULT_NOT_FOUND_CALLBACK); //option： func or null
  ``````
### 2. Core Invocation
In the main function, a very simple call can implement a complete state machine transition.

  ``````
void main()
{
    // run kernel
    SEPARATE_RUN_KERNEL(1); // manage the loop 0: No 1: Yes 
}
  ``````

  
### 3. Highly Separated Business Logic
Business logic is decoupled from the state machine core, with each state corresponding to a function. Users only need to focus on the business logic within each state function to invoke the state machine core.

#### 1. Simple State Machine
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
#### 2. Complex Business

For more complex state business applications, implement independent business logic in each state.
 ``````
int blick_count = 0;
void led_on()
{
    //init
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
            break; //jump out of the state function
        }
    }

    //deinit
    {
        WorkStat = LED_OFF;
    }

}
 ``````
 
### Code Structure
src
 - separate_core.h   Core header file
 - separate_core.cpp Core implementation file
