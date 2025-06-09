**EEC 172 Final Project**
**UC Davis**

## CC3200 Security System/Alarm

Hanna Shui  
Computer Engineering  
 UC Davis  
Davis CA USA  
 hshui@ucdavis.edu

Aditya Bhatia  
 Computer Science and Engineering  
UC Davis  
 Davis CA USA  
 htxbhatia@ucdavis.edu

**Implementation**  
**Brief Overview**  
The project involves connecting the CC3200 board with a PIR motion sensor, an AS608 Fingerprint sensor, the Adafruit OLED, a buzzer, the IR receiver, and AWS to send a receive updates from Discord and email. The system has 4 main alarm states that are displayed on the OLED and can be swapped between by using the AT\&T remote \+ IR Receiver or by sending signals using Discord slash commands. The alarm is triggered when the PIR motion sensor detects motion, and the alarm can be disabled by entering a password using the Remote ir by reading a valid fingerprint using the fingerprint sensor. The user can also change the alarm password and add or replace up to 9 fingerprints. The buzzer is used to send pulses to signify which state the alarm is in, and so is the onboard Red LED of the CC3200. The user can enter a Discord channel that alerts them whenever the state of the alarm has been changed or if the password has been changed. From the Discord channel, the user can send slash commands to enable or disable the alarm or set an alarm on and off timer. The user also receives an email when the alarm is triggered due to motion or incorrect fingerprint, or password attempts.

**Alarm States**  
The alarm has a total of 4 states that the user can switch between, and is offered various ways to switch between them.

**Alarm Off State**  
The program starts in the default state, which is the alarm off state. In this state, the OLED displays the “Alarm Off” banner in green along with the message to press the mute key on the remote to turn the alarm on, and from here, the user can switch to the alarm on state using the remote or by sending a Discord slash command. They can also switch to the password change or the new fingerprint screen by pressing the SW3 button on the CC3200 board. If the user presses the mute button on the remote, a confirmation message appears. Pressing the mute button again will cause the alarm to switch to the On state and send an HTTP post to update the AWS shadow, which will trigger the lambda function to send an Alarm On message to Discord. Pressing SW3 will cancel the dialog. On the other hand, when the local alarm state is off and the alarm state in the AWS shadow is on, caused either by sending an Alarm On Discord slash command, or if the device was reset and the last state it was in was On, then the alarm will turn on and send the http post that sends the Discord message. When the Alarm Off state is returned to from any other state, the Red LED turns off and the buzzer makes two short beeps to signify that it is in the Off state.

**Alarm On State**  
When entering the Alarm On state, the OLED screen displays “Alarm On” in red and the text telling the user that they can press mute to disable the alarm. Then the Red LED turns on and the buzzer does one long beep. When in this state, there are many possible paths. The user can either turn off the alarm by sending a Discord command or can press mute on the remote and enter the 6-digit password or use their fingerprint. The user can press SW3 at any time to disable the inputs. If the password or fingerprint is wrong 3 times, it sets the alarm into the Alarm Triggered state. Outside of that, if the PIR motion sensor detects movement, it automatically activates the password and fingerprint input and starts an approximately 30-second countdown. If the alarm is not turned off within this time, the alarm then switches to the triggered state. When switching to the Off state, an HTTP post and thus a Discord message is sent with the new state. When the alarm is switched to the Alarm Triggered state, it sends both a Discord message and an email to the user with the text ‘Intruder Alert\!’.

**Alarm Triggered State**  
In this state, the OLED displays “Triggered\!” in red, and both the Red LED and buzzer are constantly pulsing. In this state, the password input and fingerprint input are always on and will constantly provide attempts until the correct one is received. The user will receive an Alarm Triggered message and email when entering this state, and they cannot use any commands to leave this state. Once a valid password or fingerprint has been read, the alarm will switch to the Alarm Off state and send the appropriate message.

**Password Change or New Fingerprint State**  
When the user is in the base Alarm Off state, pressing SW3 will bring up a prompt that asks them to either enter ‘0’ to change the password or enter a number from 1-9, which sets the ID where the user wants to add a fingerprint. There is input handling at this prompt that confirms if the input is only one digit, shows an error message, and prevents the user from submitting if there are more or fewer digits. If the user enters 0, the screen resets the bottom half and shows the password input prompt and asks the user to enter a new password. This prompt has the same digit check as the earlier prompt, but for 6 digits instead of 1\. When the user submits the 6 digits, the screen rests at the bottom and asks the user to enter the password again. It once again does the 6-digit check, but also now checks if the passwords are the same. If they aren’t, it prints a mismatch message and asks the user to reset using SW3. If they are the same, it prints a success message, sends an HTTP post to Discord with the message “Password Reset”, and sets the new password. If the user inputs a number 1 \- 9, then the fingerprint sensor is turned on and allows the user to set a new fingerprint for that ID. It overwrites any existing fingerprint for that ID. The screen shows the same prompts as the password, but you cannot add any remote input. Instead, you tap your finger on the sensor, wait for the screen to change into the confirm stage, remove and then replace your finger. If the fingers match, there will be a success message and the new fingerprint will be added, and the screen will go back to the default Alarm Off state. If there is a mismatch, an error message will display and ask the user to press SW3 to reset. If there is any other error, a “Fingerprint Error” message will be displayed, but will continue to allow for fingerprint input. The user can escape this state at any time by pressing the SW3 button on the CC3200 board.

**The OLED Screen**  
**Pinout**  
MOSI (Data): Pin 7 (SPI MOSI)  
SCLK (Clock): Pin 5 (SPI CLK)  
DC (Data/Command): GPIO Pin 53  
CS (Chip Select): GPIO Pin 15  
RST (Reset): GPIO Pin 18  
VCC/GND: Connected to 3.3V and GND, respectively

**Description**  
The OLED display is a 128x128 Adafruit SSD1351 module connected via SPI. It is used to present the user interface, including alarm states, input prompts, error messages, and confirmation dialogs. The top half of the screen displays the current system state in large font and color-coded messages, while the bottom half is reserved for input feedback, error notifications, or additional instructions. Drawing functions from a modified Adafruit OLED library are used. These functions were defined as part of earlier labs.

**The IR Sensor \+ Remote**  
**Pinout**  
OUT: GPIO 3 (IR Receiver output)  
VCC/GND: Connected to 3.3V and GND, respectively  
**Description**  
An AT\&T Universal Remote communicates with the system via the provided IR receiver from earlier labs. The IR signal is connected to GPIO 3 and processed using GPIO interrupt handlers and pulse-width measurement via SysTick. Each button on the remote corresponds to a unique 32-bit code, decoded in real-time. Shorter pulse widths are represented as 0, and longer pulse widths are represented as 1\. The code has an array of known codes for each button and checks the code to know which button has been pressed. The mute ('m) and last ('l') buttons are used for system state control and submitting, and character deletion, respectively, while the numeric buttons ('0' to '9') are used for password and ID input. The code used for this is the same as the one for the previous two labs.

**The PIR Sensor**  
**Pinout**  
OUT: GPIO Pin 62 (PIR digital signal)  
VCC/GND: Connected to 5V and GND, respectively  
**Description**  
The PIR (Passive Infrared) sensor detects human motion and is connected to GPIO Pin 62 (GPIOA0, pin 0x80). When motion is detected, it triggers a rising-edge GPIO interrupt. If the system is in the Alarm On state, the motion event activates the input mode and starts a \~30-second countdown during which the user must disable the alarm via password or fingerprint. If no valid input is received within this grace period, the system transitions to the Alarm Triggered state.

**The Buzzer**  
**Pinout**  
Signal (+): Timer A0 PWM output (GPIO Pin 17\)  
GND: Connected to GND  
**Description**  
A piezoelectric buzzer is connected to a GPIO pin configured for PWM using Timer A0. It provides audible feedback to indicate state transitions: Two short beeps: Alarm Off. One long beep: Alarm On. Continuous pulsing: Alarm Triggered. The PWM duty cycle is modified with UpdateDutyCycle() to produce these tones.

**The Fingeprint Sensor**  
**Pinout**  
TX: Connected to UART1 RX (Pin 02\)  
RX: Connected to UART1 TX (Pin 01\)  
VCC/GND: Connected to 5V and GND, respectively  
**Description**  
The AS608 Fingerprint Sensor communicates via UART1 and is initialized using the [Adafruit\_Fingerprint library](https://github.com/adafruit/Adafruit-Fingerprint-Sensor-Library). The sensor has been implemented with the CC3200 using a modified version of Adafruit\_Fingerprint.cpp and Adafruit\_Fingerprint.h. The modules have been modified to replace all calls to Arduino-based serial read and write functions with UartCharPut and UartCharGet functions of the CC3200. The class’s constructor has also been modified to accept the macro for UARTA1 so that it can enable the correct UART interface to use to transmit and receive data. The appropriate includes were also added to the library code. The module is used for two main functions:  
Authentication: Disarm the alarm by matching a scanned fingerprint against stored templates.  
Enrollment: Add or replace fingerprints by assigning them to IDs (1–9). The user must place the same finger twice for confirmation before it is stored.  
The fingerprint process includes detailed prompts and feedback via the OLED screen and can recover from various error states, such as image mismatch or poor quality. The authentication and enrollment code has been adapted from the [fingerprint](https://github.com/adafruit/Adafruit-Fingerprint-Sensor-Library/blob/master/examples/fingerprint/fingerprint.ino) and [enroll](https://github.com/adafruit/Adafruit-Fingerprint-Sensor-Library/blob/master/examples/enroll/enroll.ino) Arduino examples, respectively. The main functions from both examples had to be slightly modified to work properly in the project. More details about the modifications have been described in the [Challenges section](#2.-small-issues-with-the-adafruit-fingerprint-sensor-library). The fingerprint sensor is only turned on when the code needs input from the sensor. The code to turn on and begin communication with the sensor is also from the example code by Adafruit.

**Communication with AWS**  
**Get Request**  
The system sends periodic HTTP GET requests to AWS IoT to fetch the current shadow state of the device. This ensures that the local state remains synchronized with any changes made remotely via Discord. The state is in the format {“state” : {“desired” : {“Message” : {“email” : …}}}}. If the shadow reports that the email value is "Alarm On" and the local state is "Off", the device will update its state, turn on the buzzer and LED, update the OLED, and send a confirmation POST to AWS. If the email value is “Alarm Off” and the current local state is On, the local state is switched to Alarm On, and the appropriate LED, buzzer, and HTTP requests events take place. These requests use the secure TLS socket established with AWS and are scheduled to run approximately every 25 seconds using a timer. The timer setup is the same as the one that is used to check for the motion grace period.

**Post Request**  
Whenever the alarm state changes (e.g., from Off to On, On to Triggered, etc.) or when the password is changed, the system sends an HTTP POST to the AWS IoT endpoint with a JSON payload describing the new state. This payload is written manually in the code and includes state-specific messages in the email field like: "Alarm On", "Alarm Off", "Intruder Alert\!", "Password Reset". These updates modify the shadow state and trigger either one or two SNS topics. One of the topics is just for Discord messages, and that triggers a Lambda function on AWS to send the state as a message on Discord. If the state is “Intruder Alert\!”, it also triggers another AWS topic, which sends an email to the address set in AWS with the same message as the state. The post is usually triggered when the state is switched, and the code block to start rendering the new state begins. The function is sent with a flag to decide which message goes in the JSON body for the request. The password reset message is sent right when the code successfully matches both of the password inputs.

**Discord Communication**  
**Messaging Lambda**  
An SNS topic is set up so that it triggers an AWS Lambda function whenever the shadow state changes. When a new state is posted (e.g., "Alarm On"), the function sends a message to a designated Discord channel using a pre-configured webhook. The webhook URL is taken from the channel created by the user in their own Discord server. The lambda grabs the text from the email field in the state and formats it into a message, which is then sent to Discord as an HTTP post to the webhook URL. Discord then sends that message to the channel using a bot. Currently, the lambda sends messages for the alarm state “Alarm On/Off”, “Intruder Alert”, password changes “Password Reset”, and can also show the time set by the user for the on and off timer for the alarm converted to Pacific Daylight Time and report them as “Start: \<time\> Stop: \<time\>”.

**Slash Commands**  
Configuring a slash command is a bit more complicated. It first involves creating a public bot for your account using the Discord Developer Portal. Once the bot is created, it generates a bot URL that you can use to invite the bot to your server. Once that is done, you can create the slash commands you want using JSON files. We used ChatGPT to generate the JSON files for our two slash commands. You can then post these JSON files using curl to the bot’s URL while providing your Discord public key to add the slash command to the bot. In AWS, you have to create your own API endpoint URL with the format of \<name\>/interactions. The bot will use this endpoint to send and receive data from the AWS Lambda function. The next step involves creating the lambda function. The function needs to include the pynacl library, which needs to be installed, zipped, and then uploaded for Python 3.9 to AWS as a layer that needs to be added to the lambda. The lambda function needs to verify the post signature from the bot using this library, as can be seen in the Discord-provided instructions. This verification is required for Discord to let you use the bot. Outside of that, the lambda must reply with a 200 code and body response of ‘1’ when receiving a request from Discord that is of type ‘1’. Discord uses this ping to validate your API’s endpoint. Once this functionality of the lambda is implemented, you can add your AWS API endpoint into the interactions URL section in the Discord Dev Portal for your bot. Now, it is possible to handle the slash commands. Our code handles Discord slash commands to remotely control a smart alarm system by updating its state in AWS IoT.

Slash Command: /alarm  
Purpose: Turns the alarm on or off.  
How it works:  
Reads the value (on or off) from the command. Sends an update to the AWS IoT device shadow with an "email" field like "Alarm on" or "Alarm off". Sends a confirmation reply in Discord: ✅ Alarm set to on (though this ends up not being visible to the user)

Slash Command: /timer  
Purpose: Schedules the alarm to turn on/off at specific times.  
How it works:  
Parses ISO timestamps (start\_str and stop\_str) and localizes them to PDT (Pacific Daylight Time). Converts these times to UNIX timestamps. Sends the timestamps to AWS IoT under a "timer" key in the device shadow. Sends a confirmation reply in Discord showing the scheduled times.

In both cases, AWS IoT triggers updates to the connected CC3200 device, which syncs its local alarm state accordingly.  
Unfortunately, due to time constraints, even though the timer slash command works and properly updates the device state, we were unable to finish implementing the code to grab the current time, parse these values, and then update the local state. More details have been provided in the Future Work section.  
   
**Challenges**  
During the development of the project, we faced the following challenges:

**1\. Unusability of the RC522 RFID sensor**  
We had initially planned for the user to be able to disable the alarm using an NFC tag or card that they would tap against an RC522 RFID sensor connected to our CC3200 board. The product descriptions of the sensor showed that the sensor was compatible with SPI, I2C, and UART. Since the OLED display used SPI, we planned on using either I2C or UART for the sensor. However, after ordering, receiving, and then sitting down with the part to implement it into the project, we realized that the sensor was hard-wired to use the SPI communication protocol. After looking online and at the schematics, we realized that we would either have to drill a hole in the sensor board to enable I2C or cut a trace and tie a contact pin of the chip on the sensor board to ground to use UART. We tried to get the chip into UART mode since it seemed easier, but alas, we ended up damaging the board in the process, rendering it unusable. This happened on the Saturday of the week our project was due on the Wednesday lab session. We quickly had to order a replacement module and decided to go with the fingerprint sensor since it communicated natively through UART. However, that module arrived on Monday, giving us less than 2 full days to get it working with the project. Thankfully, the availability of the Adafruit library for the fingerprint sensor made it possible to do so.  
[Product information for RC522](https://www.amazon.com/HiLetgo-RFID-Kit-Arduino-Raspberry/dp/B01CSTW0IA)  
[Drilling a hole for I2C communication](https://www.teachmemicro.com/arduino-rfid-rc522-tutorial/#Using_I2C_to_Communicate_with_RFID_Reader)  
[Cutting a trace and grounding a pin for UART communication](https://raw.githubusercontent.com/mfdogalindo/MFRC522-UART/master/wiring.jpg)

**2\. Small issues with the Adafruit Fingerprint Sensor library**  
In the example code provided by Adafruit for enrolling and reading a fingerprint, the code called for the fingerprint sensor to first take an image of the fingerprint and then calls a function on the sensor that converts the image into some kind of hash/key that it uses to save or check with its database to confirm the fingerprint. It seems like the adfruit library expected this code to execute on the chip instantly, but it actually takes some time for the chip to respond. This was causing the code to error out since it kept reading communication errors. This problem was fixed by [allowing the code to loop until it received an OK](https://github.com/Aditya-Bhatia/EEC172-Project/blob/11dce6461d8eaba020ef1b2429f40234951015ea/main.cpp#L1014) or some other non-communication-related error code from the sensor’s chip. The example code [already did this for taking an image of the fingerprint](https://github.com/Aditya-Bhatia/EEC172-Project/blob/11dce6461d8eaba020ef1b2429f40234951015ea/main.cpp#L988), but we also adapted it for the conversion command.

**3\. Inconsistent behavior of the PIR sensor with the CC3200 board**  
We have a GPIO interrupt enabled to detect when the data pin of the sensor goes high when it detects movement, but for some reason, it initially didn’t work on some pins on both of our boards. It took us a while to figure out whether the issue was with the code, sensor, or pin, but eventually we found a pin where the interrupt worked, and we got the sensor communicating with the board properly.

**Future Work**  
Here are some ideas we had for some things that could be done in the future:

**1\. Complete the timer stretch goal**  
The timer slash command is set up in Discord and posts to the shadow, but there is no code to handle it. It would be great if we had more time to figure out a way to get the current UTC time on the CC3200 board and compare it with the set start and stop times for the alarm in the shadow through the Discord slash command, and then appropriately switch the Alarm’s state at those times.

**2\. Presence detector instead of a Motion sensor**  
We could make it so that we use a 24GHz mm wave motion \+ presence detector instead of a PIR sensor. These sensors are more accurate than PIR sensors and will be able to detect if the user is standing still in front of the device to enter their password or fingerprint, or if they’ve walked away. We could also make it so that the fingerprint sensor is now outside/on the door, and the alarm is inside, and the presence detector can detect if someone is at the door and can turn on the fingerprint sensor so that they can deactivate the alarm before walking in.

**3\. Store fingerprints in DynamoDB**  
This feature is not that important. The fingerprint module only allows for the bitmap images of the fingerprint to be exported, and not the hash/key it stores to confirm the fingerprint. So it would only act as a possible backup in case the sensor is lost or damaged, and not as a way to store extra fingerprints.

**4\. Remove fingerprint option**  
Currently, the code does not allow the user to remove a fingerprint from the sensor, they can only overwrite an existing fingerprint. The sensor module allows for deletion of a particular fingerprint using its index, so it would be possible to add a feature to remove a fingerprint through both the display \+ remote and through a Discord slash command.

**5\. Use Discord slash command to change password**  
It would be beneficial for the user to be able to change the password of the alarm using a slash command on Discord. To implement this, we would have to use encryption to store and share the password instead of the plain text storage that it currently uses, so that an attacker can’t just packet sniff the post request bodies being sent to the alarm and grab the password from there.