# Autosar Layers
AUTOSAR is an acronym for *AUTomotive Open System ARchitecture* . 
Automotive is the  of automotive cars. A company like **ELARABY** Group for home appliances has an embedded team but it is not an automotive industry. Another company is **BioBusiness** company for biomedical devices, It also has an embedded team but not automotive. 
MISRA C is a coding standard to avoid bugs, While AUTOSAR is a coding Methodology to help portability and reuse.. 

Normally a car contains dozens of ECUs (Electronic Control Unit - MCU with some other components), Each System in the car has its ECU, like the EBS (Emergency Braking System), ABS (Anti-Lock Braking System), Airbags, Doors sensors, Parking sensors and so on. 
These ECUs communictes through some communication protocols like CAN and LIN. 
If you are working in an embedded company like *Valeo*,  *Ejad*,  *Mentor Graphics*,  *EveLabs*, *BrightSkies* and so on, You are likely going to deal with these ECUs. 

AUTOSAR is a standard for the automotive industry, some guidelines or rules that sets your way writing a software for the automotive industry. 
It is preferred to follow it, as following it has advantages. 
Some companies follows it 100% and some other companies takes what it needs from it, and some companies do not follow it at all. 
When you see the job offers it is often a plus or recommended to have a knowledge with AUTOSAR. 

We are not going to take an AUTOSAR course, we will take a small part of it and apply it to all sessions. 
A recall to what to do after AVR course? What to advance into? 
If Embedded Linux:
- OS
- Embedded Linux
If Automotive:
- ARM Course (Most Imp)
- AUTOSAR Course



## Autosar Layers
Do you remember this code for toggling a pin? 
```c
#define F_CPU 8000000UL
#include <util/delay.h>
#include <dio.h>

int main(void) {
	DIO_SetPinDir('A', 2, 1);
	
	while(1) {
		DIO_WritePin('A', 2, 1);
		_delay_ms(1000);
		DIO_WritePin('A', 2, 0);
		_delay_ms(1000);
	}
}
```
That's the normal code for calling functions
But in Autosar states that functions calling must be in layers. 

### What are Autosar Layers (Layered Architecture)? 
Autosar Layers are three layers:
- Application Layer
- HAL (Hardware Abstraction Layer)
- MCAL (Micro-Controller Abstraction Layer)

Normally we module our project into include folder containing header files and source folder containing source files and main. 
The main file is the Application Layer. 
Th HAL is all the external hardware modules drivers like led module, button module, motors module, relay module, any external hardware module. 
The MCAL is all the internal hardware modules drivers like DIO, PWM, ADC, Timers and so on. 

Normally the sequence should be as follows:
- The main function (Application Layer) should be calling an external module function (HAL). 
- The External Module Function (HAL) should be calling an internal module function (MCAL). 

![AUTOSAR layers diagram showing three stacked software layers labeled Application Layer at the top, HAL in the middle, and MCAL at the bottom. Arrows indicate application code calling HAL functions, which call MCAL drivers for microcontroller hardware access. The illustration is a simple technical architecture diagram with a neutral academic tone. Text in the image reads Application Layer, HAL, and MCAL.](/Interfacing/Attachments/autosar-layers.png)

So the code above isn't following the Layered Autosar Architecture, as the main (Application Layer) is calling the internal module directly (MCAL) with no HAL in between.

How to make the code above follow the Layered Architecture? 
We are trying above to control a Led, This led is considered an external module, so the best move here is that we make a driver for the led. 
The main calls the led driver and the led driver calls the dio driver. 
It seems redundant but be patient. 

Instead of calling this in the main:
```c
DIO_vsetPinDir('A', 2, 1);
```
We will call this in the main:
```c
led_init('A', 2);
```
And this calls the first. 


Instead of this in the main:
```c
DIO_vwritePin('A', 2, 1);
```
We will call this in the main:
```c
led_on('A', 2);
```
And this will call the first. 

Instead of calling this in the main:
```c
DIO_vwritePin('A', 2, 0)
```
We will call this in the main:
```c
led_off('A', 2);
```
And this will call the first. 

Here I increased the overhead, and the executing time. 
Now it seems redundant and worse, but be patient. 

#### Why was it called Abstraction Layer? 
Abstraction means isolating like a remote control and a TV, When I press on a button on the remote I am waiting for some functionality to occurr without worrying how will it be implemented. 
We can say here that the functionality is isolated from the end user. 
The same rule applies on the Autosar Layers, 
The HAL is named as Hardware Abstraction Layer as the hardware is isolated from the end user in the main and also isolated from the MCAL, No one knows the real implementation except the HAL Developer. They only have the user interface layer (Function calls) to interact with the HAL with. 
The MCAL is named as Micro-Controller Abstraction Layer as the implementation is isolated from the end user in the main and from the HAL Developer, They only have the user interface layer (Function calls) to interact with the MCAL with. 

Imagine we are working on a big project and you used a specific types of LCDs, and you wanted to change it with another type that needs a new driver. If you didn't follow the autosar standards you will need to change the whole main, but if you are following the autosar standards you only replace the driver with the new one, you don't need to change the main file as the function prototypes are the same. 

#### Co-Operate on standards, Compete on the Implementation
This is a well known quote in Autosar architecture, as all the functions have the same prototypes (have the same name, return type, arguments), but the implementation differs across companies. 

### Callback Functions
Callback functions happen when you try to call a function from an upper layer. Like you are trying to call a function in *Application Layer* or *HAL* from a *MCAL*, or trying to call a function in *Application Layer* from a *HAL*.

---
# Led Driver

## `led_Init(uint8_t, uint8_t );`
This function sets the led to be output.
```c
// led.c

void led_Init(uint8_t port, uint8_t pinNum) {
	// Calling MCAL
	DIO_SetPinDir(port, pinNum, 1);
}
```

## `led_on(uint8_t, uint8_t);`
This function sets the pin of the led to High
```c
// led.c

void led_on(uint8_t port, uint8_t pinNim) {
	DIO_WrirePin(port, pinNum, 1);
}
```

## `led_off(uint8_t port, uint8_t pinNim);`
This function sets the pin of the led to Low
```c
// led.c

void led_off(uint8_t port, uint8_t pinNim) {
	DIO_WrirePin(port, pinNum, 0);
}
```

## `led_Toggle(uint8_t, uint8_t);`
This function toggles the pin that a led is connected to. 
```c
// led.c

void led_Toggle(uint8_t port, uint8_t pinNum) {
	DIO_TogglePin(port, pinNum);
}
```

## `uint8_t led_ReadStatus(uint8_t, uint8_t);`
This function returns the status of a led. It reads the PINX Register. 
```c
// led.c

uint8_t led_ReadStatus(uint8_t port, uint8_t pinNum) {
    return DIO_ReadPin(port, pinNum);
}

```

---
# Some Led Examples
## Example 1
In this example we want to control 8 leds connected on port, Led i is connected on Pin D i. 
We want to gradient the leds turning them from the first led to the last led one by one with a delay of 1 second, then turn them off one by one in a reversed order with a delay of 1 second. 

We first need to initialize the leds, we can do so by repeatedly call `led_Init('D', i);` (with i from 0 to 7).
```c
#define F_CPU 8000000UL
#include "led.h"
#include <util/delay.h>

int main(void) {
	
	/*
		I am a jerk
	*/
	
	led_Init('D', 0);
	led_Init('D', 1);
	led_Init('D', 2);
	led_Init('D', 3);
	led_Init('D', 4);
	led_Init('D', 5);
	led_Init('D', 6);
	led_Init('D', 7);
	
	while(1) {
		// Some Stunning Logic
	}
}
```


Or we can simply put them in a loop:
```c
#define F_CPU 8000000UL
#include "led.h"
#include <util/delay.h>

int main(void) {
	
	for(uint8_t i=0; i<=7; ++i) {
		led_Init('D', i);
	}
	
	while(1) {
		// Some Stunning Logic
	}
}
```
*I have used `uint8_t` to save memory*.

Now regarding the stunning logic we need to do. 
We can simply make 2 loops one of them js reversed. One for turning them on and the other for turning them off. 

```c
#define F_CPU 8000000UL
#include "led.h"
#include <util/delay.h>

int main(void) {
		
	for(uint8_t i=0; i<=7; ++i) {
		led_Init('D', i);
	}
		
	while(1) {
		
		for(uint8_t i=0; i<=7; ++i) {
			led_TurnOn('D', i);
			_delay_ms(1000);
		}
		
		_delay_ms(2000);
		
		for(uint8_t i=7; i>=0; --i) {
			led_TurnOff('D', i);
			_delay_ms(1000);
		}
		
		_delay_ms(2000);
	}
}
```
Wow, that is really a stunning implementation, but it has only 1 problem, it only runs for a once, it doesn't repeat. 
Try to Catch the problem, Take a 5 and look carefully. 

Okay I am bored, I will spoil it. 
In the second loop, when do we stop? 
When the counter is not greater than or equal to zero right? Doesn't this mean when we hit `-1`? 
We have declared the counter an `unsigned char` (`uint8_t`) which never reaches `-1`.

There are many solutions for this, we can increase the counter and the condition by 1 and reduce the led by 1.

```c
for(uint8_t i=8; i>=1; --i) {
	led_TurnOff('D', i-1);
}
```

Another solution is that we can cast it into `signed char` during the comparison. 

```c
for(uint8_t i=7; (int8_t)i>=0; --i) {
	led_TurnOff('D', i);
}
```

Another solution is to define it as `signed char`.

```c
for(int8_t i=7; i>=0; --i) {
	led_TurnOff('D', i);
}
```
## Example 2
Imagine we want to connect our MCU to 2 leds on different pins, the first led should toggle each second and the second led should toggle each two seconds. 

First Thoughts? 
We can trace all the cases we have. 

| Seconds   | Led 2 | Led 1 |
| --------- | ----- | ----- |
| Second 1  | OFF   | OFF   |
| Second 2  | OFF   | ON    |
| Second 3  | ON    | OFF   |
| Second 4  | ON    | ON    |
| Second 5  | OFF   | OFF   |
| Second 6  | OFF   | ON    |
| Second 7  | ON    | OFF   |
| Second 8  | ON    | ON    |
| Second 9  | OFF   | OFF   |
| Second 10 | OFF   | ON    |

Don't you see a pattern? 
If you look closely you can see that the first 4 seconds repeats, so we can take the first four states, hardcode them in a loop.

```c
#define F_CPU 8000000UL
#include "led.h"
#include <util/delay.h>

int main(void) {
	
	led_Init('D', 0);
	led_Init('D', 1);
	
	while(1) {	
		// First Second
		led_TurnOff('D', 0);
		led_TurnOff('D', 1);
		_delay_ms(1000);
		
		// Second Second
		led_TrunOn('D', 0);
		_delay_ms(1000);
		
		// Third Second
		led_TurnOn('D', 0);
		led_TurnOn('D', 1);
		_delay_ms(1000);
		
		// Fourth Second
		led_TurnOff('D', 0);
		_delay_ms(1000);
		
		// See you above
	}
} 
```

But What if we have 3 leds, now we have 2 states for each, using multiplication rule so we have $2^3$ distinct Conditions that needs to be hardcoded, this is complex and inconsistent. 
A better approach is to use Timers and Counters, but we don't know them yet, so we will try to think harder. 

If you took your time thinking (or sinking whatever) you will have a great conclusion. These states literally have this nature due to its binary nature, in the table above if we replace ON with 1 and OFF with 0 it is literally will look like a logic design problem. 
If we remember from Logic Design Course in college we can use the reminder to operate this circuit. 
This the circuit:

| Seconds   | Led 3 | Led 2 | Led 1 | Dec |
| --------- | ----- | ----- | ----- | --- |
| Second 1  | 0     | 0     | 0     | 0   |
| Second 2  | 0     | 0     | 1     | 1   |
| Second 3  | 0     | 1     | 0     | 2   |
| Second 4  | 0     | 1     | 1     | 3   |
| Second 5  | 1     | 0     | 0     | 4   |
| Second 6  | 1     | 0     | 1     | 5   |
| Second 7  | 1     | 1     | 0     | 6   |
| Second 8  | 1     | 1     | 1     | 7   |
| Second 9  | 0     | 0     | 0     | 8   |
| Second 10 | 0     | 0     | 1     | 9   |

When does Led 1 toggle? 
 At all numbers (Every Iteration). 

When does Led 2 toggle? 
At even numbers only (Every 2 Iterations). 

When does Led 3 toggle? 
At numbers divisible by 4 only (Every 4 iterations). 

So we simply can map this logic in a loop with a counter and some conditions on this counter. 

$Led_i$ toggles every $i$ iterations (i starts from 1).
We can check for this by modulo. 


```c
#define F_CPU 8000000UL
#include "led.h"
#include <util/delay.h>

int main(void) {
	
	for(uint8_t i=0; i<3; ++i) {
		led_Init('D', i);
		led_TurnOff('D', i);
	}
	
	uint8_t TheMostPowerfulCounterEver = 0;
	
	while(1) {
		_delay_ms(1000);
		
		// Happens Each second
		led_Toggle('D', 0);
		
		if(TheMostPowerfulCounterEver % 2 == 0) {
			// Happens Each 2 Seconds
			led_Toggle('D', 1);
		}
		
		if(TheMostPowerfulCounterEver % 3 == 0) {
				// Happens Each 3 Seconds
			led_Toggle('D', 2);
		}
		
		TheMostPowerfulCounterEver++;
	}
}
```

---

# Buzzer Module
The buzzer is also an output component that is polarized, its anode pin is longer than its cathode pin, and also its anode pin is annotated with a positive sign at the top. 

The buzzer has many types depending on the voltage it needs to operate, when it operates it outputs a buzzing sound, that's why it is named like this. 

It's driver doesn't differ from the led's. They has literally the same logic and the same implementation. 

---
# Button Driver
Buttons has many types (DIP, Push Button, .. )

The Push button has 2 pins that are isolated (open circuit) as long as we are not pressing the button, when we press it, a short happens between the two pins closing the circuit. 

The switch button (spst) does the same but we don't need to keep pressing, we only press the switch once and it keeps conducting till we press it again. 

The DIP can be pushed and pulled, it is often packaged in many switches together. When it is pushed to ON side it closes the circuut, and when it is pulled to OFF side it opens the circuit. 

### Button Connection
If we tried to connect a button (any type of the above) to a MCU pin, The button should be an input component, I can connect it to PA2 and connect it into a 5V rail. 

In this connection, when we press the button we send a Logic High to the Pin, but when we don't press what happens?
This is known as the floating pin famous problem. 

MCUs and Normally Most of the Embedded Hardware Boards are made of TTL (Transistor-Transistor Logic) which means you are either getting your input from a transistor or you are giving your output to a transistor, or you yourself are a transistor. 

Transistors have only 2 modes: - Hungry or Horny (Kidding) - Logic Low or Logic High. 
They have nothing in between, they have no state to represent neither of them (floating) (HI-Z).

Acceptable TTL Gate Level:
- $[0V \space , \space 0.8V]$ considered Logic Low. 
- $[2V \space , \space 5V]$ considered Logic High. 

The gap between $0.8V$ and $2V$ is floating, we have no state for it, so it keeps flipping unexpectedly. 


This problem relates to our connection, when we don't press the button it is not pulled into $0V$. It's left noisy, due to electromagnetics around it keeps floating with no clue with its real logic level. So we need to pull it to $0V$ when we are not pressing the button, and pull it to $5V$ when we are pressing the button. 

This can be solved using the Pull Down Resistor. 
I am too lazy to explain why it is true, I have already explained it before in Hardware Article.
Go continue there if you don't get the logic and comeback here for the software part. 

[Microchip PIC18F452 hardware article link, a documentation page with the title PIC18F452 Hardware and a clean white layout with blue accents, providing a reference to the hardware article on this topic.](/Interfacing/Microchip's%20Processors/1.%20PIC18F452%20Hardware.md) 

### Software Part
Imagine we want to connect a button and a led to the MCU, when we press the button the led should turn on. 

This can be implemented using `button.h`
```c
#define F_CPU 8000000UL
#include <util/delay.h>
#include "led.h"
#include "button.h"

int main(void) {
	
	button_Init('D', 0);
	led_Init('D', 1);
	
	while(1) {
		if(button_Read() == 1) {
			led_TurnOn('D', 1);
		} else {
			led_TurnOff('D', 1);
		}			
	}
}
```

Now we need to know the implementation of those functions. 

### Button Driver

#### `Button_Init(uint8_t, uint8_t)`
This function initializes the button by setting the pin to it into an input pin (calling MCAL). 

```c
// Button.c

void Button_Init(uint8_t port, uint8_t pin) {
	DIO_SetPinDir(port, pin, 0);
}
```

#### `uint8_t Button_ReadState(uint8_t, uint8_t)`
Thus function returns the state of the pin the button is connected to (calling MCAL). 

```c
// button.c
uint8_t Button_ReadState(uint8_t port, uint8_t pin) {
	return DIO_ReadPin(port, pin);
}
```