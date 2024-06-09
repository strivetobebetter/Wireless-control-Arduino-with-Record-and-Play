![Leanbot in competition.](/image/leanbot.jpg "Image of Leanbot.")


## Problem Statement
[Leanbot](https://leanbot.space/), a Arduino based robotkit from Vietnam, is used in competition to expose student the STEM concept like microcontroller, programming, engineering and etc. Every year different competition will come up with new and different task for leanbot to score point, so Leanbot code need to change to match the tasks every year. This is a very time consuming process. Measuring the Leanbot distance and orientation then modify a small section of the code for every iteration is a tideous trial and error process. 

## Solution
Rather than doing the measurement and modify code instruction manually, bluetooth communication can be added between PC and leanbot for data collection and send command to perform task then "record and play" feature saved a sequences of comamnd and leanbot will follow the command to complete the tasks in one go. As a result of this features, the trial and error process is **shorten from 2 weeks to 2 days**. 

## Leanbot Platform
Leanbot come with many function out of the box. It include stepper motor control, gripper, ultrasonic sensor, RGB Led, MotionTracking sensor, IR sensor array, buzzer, bluetooth module and more. Since its microcontroller is arduino, all of the electronic components are open source as well. Many resources are available to tinker the Arduino ecosystem. What make Leanbot unique is that its arduino come in ready to use along with beginner-friendly programming platform to start children age between 7 to 15 their learning journey in STEM. 

### Bluetooth Communication
To connect PC with Leanbot, follow the steps below.

1. Add leanbot to PC bluetooth device.
![PC Bluetooth.](/image/PC_bluetooth.jpg "Image of Blutooth setting.")
2. Key in the password '1234'.

3. Download Arduino IDE to use its Serial monitor

4. Connect IDE with leanbot
	- Find leanbot com **may be different for each PC 
	- Select port

5. Connection is successful when leanbot stop blinking and a short buzz is heard

### Built-in function
To program leanbot and utilise their custom made library function, go to their [online IDE website](https://ide.pythaverse.space/). 

## How to Use
Upload 

  
## Limitation









