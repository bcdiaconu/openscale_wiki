<!-- 
<p align="center">
<a href="https://github.com/oliexdev/openScale/raw/master/doc/screens/screen_graph.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/screens/screen_graph.png' width='300px' alt='missing' /> </a> <br>
<sub>Caption</sub>
</p>
-->

# openScale wiki

*openScale is an Open Source App for Android to keep a log of your body weight, fat, water and muscle.*

<table border="0">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/scale_hacked_front.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/scale_hacked_front.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure A: Final result of the hacked bathroom scale (front)</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/scale_hacked_back.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/scale_hacked_back.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure B: Final result of the hacked bathroom scale (back)</sub>
</th>

  </tr>
</table>

## Reducing the power consumption

The goal of this step is reduce the power consumption as far as possible to extend the battery lifetime. The best way to do this is to bring the Arduino in sleep mode when it's not needed. To wake up the Arduino I used an external interrupt signal on pin 3 of the Arduino Pro Mini. It is connected to the first cycle clock (C0) of the display connector (see figure 2.4) because the cycle clock is only active if the scale is on. For bringing the Arduino into the sleep mode I used the [Low-Power library](https://github.com/rocketscream/Low-Power) by rocketscream. To detect when the scale is off I count all no active cycles. If it's exceed 32 cycles then I put the Arduino into sleep mode again and wait for the next interrupt event. The following code snippet implement this behaviour:

```C
#include <LowPower.h>

#define WAKEUP_PIN 3

#define MAX_NO_ACTIVITY_CYCLES 32

int no_activity_cycles = 0;
volatile boolean sleep_state = true;

void interrupt_handler()
{
  sleep_state = false;
}

void setup() 
{
  ...
  pinMode(WAKEUP_PIN, INPUT); 
  ...
}  

void check_display_activity()
{
  if (no_activity_cycles > MAX_NO_ACTIVITY_CYCLES)
  {
    sleep_state = true;
    no_activity_cycles = 0;
  }


  if (sleep_state == true)
  {        
    before_sleep_event();

    attachInterrupt(1, interrupt_handler, RISING);
    LowPower.powerDown(SLEEP_FOREVER, ADC_OFF, BOD_OFF); 
    detachInterrupt(1); 

    after_sleep_event();  
  } 
}

void loop() 
{   
  check_display_activity(); 
  ...
  else if (control_bit[0] == HIGH && control_bit[1] == HIGH && control_bit[2] == HIGH && control_bit[3] == HIGH)
  {      
    no_activity_cycles++;
  }
  ...
}
```

Make sure you detach the external interrupt with `detachInterrupt(1)` after you wake up the Arduino otherwise the interrupt handler will always be called if the clock cycle C0 is rising! I measured the power consumption of the Arduino Pro Mini board in active and sleep mode. I got the following results:

|  Mode  | Power Consumption |
|:------:|:-----------------:|
| Active |        50mA       |
|  Sleep |       1.5mA       |

To minimize the power consumption a bit more I soldered out the power LED on the Arduino Pro Mini board. It reduced the power consumption about 1.5mA. So the final measured results are:

|  Mode  | Power Consumption |
|:------:|:-----------------:|
| Active |        48mA       |
|  Sleep |       0.035mA     |

So in the end I could power supply the Arduino for around 3 years with the 4 AAA batteries (960mAh). To power down all external modules, like the Bluetooth, EEPROM, RTC module, while the Arduino is in sleeping mode I used an external [BC546 NPN Transistor](https://github.com/oliexdev/openScale/raw/master/doc/bc546_transistor/bc546_datasheetpdf) as a switch. I connected the transistor as in figure 6.1 or figure 2.4.

<p align="center">
<a href="https://github.com/oliexdev/openScale/raw/master/doc/bc546_transistor/bc546_schematic.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/bc546_transistor/bc546_schematic.png' width='300px' alt='missing' /> </a> <br>
<sub><b>Figure 6.1: Schematic of the BC546 NPN Transistor</b></sub>
</p>

So before the Arduino went to sleep mode I disable all external modules by setting pin 13 to low. I also writing (if a valid measurement exist) the measured values into the I²C EEPROM and sending all measured scale data entries via Bluetooth to my tablet with the following code snippet:

```C
#define EXT_SWITCH_PIN 13

void setup()
{
 ...
 pinMode(EXT_SWITCH_PIN, OUTPUT);
  
 digitalWrite(EXT_SWITCH_PIN, HIGH);
}

void before_sleep_event()
{
   Serial.println("$I$ going to sleep in 3 seconds!");
  
  if (measured_weight != -1 && measured_fat != -1 && measured_water != -1 && measured_muscle != -1) {
    write_scale_data(measured_weight, measured_fat, measured_water, measured_muscle);
    delay(100);
  }
  
  send_scale_data();
  
  delay(3000);
  
  digitalWrite(EXT_SWITCH_PIN, LOW);   
}
```

After an interrupt event occurred I enable all external modules by setting pin 13 to high. I also initializing the measured values to a defined value.

```C
#define UP 12

void setup()
{
 ...
  pinMode(UP, OUTPUT);
 ...
}

void after_sleep_event()
{
  digitalWrite(EXT_SWITCH_PIN, HIGH);

  measured_weight = -1;
  measured_fat = -1;
  measured_water = -1;
  measured_muscle = -1;

  delay(4000);
  digitalWrite(UP, HIGH);
  delay(500);
  digitalWrite(UP, LOW);
  
...
  
  Serial.println("$I$ openScale MCU ready!");
}
```

The impulse on pin 12 is for bringing the SBF 12 bathroom scale automatically into the "body measuring mode" otherwise I had to manually press the UP button on the bathroom scale.

Note in figure 2.4 that I power supply the bathroom scale with the 4 AAA batteries. So remove the bathroom scale coin cell!

## Adding external I²C EEPROM

For temporally storing the measured values, even if the power supply is disconnected, I used an external [512 Kbit I²C EEPROM 24LC512](https://github.com/oliexdev/openScale/raw/master/doc/eeprom_24lc512/24lc512_datasheet.pdf), see figure 5.2-5.3.

<table border="0">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/eeprom_24lc512/24lc512_schematic.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/eeprom_24lc512/24lc512_schematic.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 5.1: Schematic of the 24LC512 I²C EEPROM</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/eeprom_front.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/eeprom_front.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 5.2: I²C EEPROM module (front)</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/eeprom_back.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/eeprom_back.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 5.3: I²C EEPROM module  (back)</sub>
</th>

  </tr>
</table>

I soldered a breakout board like in figure 5.1 and connected it with the Arduino like in figure 2.4. Note because I set A0-A2 to GND the EEPROM had the I²C address 0x50. For the Arduino library I used the [LibraryForI2CEEPROM](http://arduino.cc/playground/Main/LibraryForI2CEEPROM) by robtillaart. 

For storing the measured values I first defined a struct "scale_data".

```C
typedef struct scale_data{
  int year;
  byte month;
  byte day;
  byte hour;
  byte minute;
  int weight;
  int fat;
  int water;
  int muscle;
  int checksum;
} __attribute__ ((packed));
```

The attribute `__attribute__ ((packed))` is important to avoid byte padding in this struct otherwise the contiguous writing or reading to the I²C EEPROM will not work! To reduce the amount of the required bytes I used integers (2 bytes) and not floats (4 bytes). The conversion between integers and float would I done when I send the data to the tablet. To know how many data entries are in the I²C EEPROM stored I always write/read the current data size at the EEPROM address 0. So the writing function to the I²C EEPROM looks like:

```C
#include <Wire.h>
#include <I2C_eeprom.h>

I2C_eeprom eeprom(0x50);

void write_scale_data(int weight, int fat, int water, int muscle)
{
  int data_size = 0;
  struct scale_data wdata;

  eeprom.readBlock(0, (uint8_t*)&data_size, sizeof(data_size));

  wdata.year = year();
  wdata.month = month();
  wdata.day = day();
  wdata.hour = hour();
  wdata.minute = minute();
  wdata.weight = weight;
  wdata.fat = fat;
  wdata.water = water;
  wdata.muscle = muscle;
  wdata.checksum = calc_checksum(&wdata);

  if (eeprom.writeBlock(sizeof(data_size)+data_size*sizeof(wdata), (uint8_t*)&wdata, sizeof(wdata)) != 0) {
    Serial.println("$E$ Error writing data to eeprom");
  }
  
  delay(100);
  data_size++;
    
  if (eeprom.writeBlock(0, (uint8_t*)&data_size, sizeof(data_size)) != 0) {
    Serial.println("$E$ Error writing data to eeprom");
  }
}
```

To check the integrity of the data I calculated a XOR checksum with the following function:

```C
int calc_checksum(struct scale_data* wdata)
{
  int checksum = 0;
  
  checksum ^= wdata->year;
  checksum ^= wdata->month;
  checksum ^= wdata->day;
  checksum ^= wdata->hour;
  checksum ^= wdata->minute;
  checksum ^= (int)((float)wdata->weight / 10.0f);
  checksum ^= (int)((float)wdata->fat / 10.0f);
  checksum ^= (int)((float)wdata->water / 10.0f);
  checksum ^= (int)((float)wdata->muscle / 10.0f);

  return checksum;
}
```

The reason why I did the wired conversion back to float and then again to integer was that I send the data to the tablet with the correct floating point value. In this way the Android app could also calculate the correct XOR checksum.

To send the stored values in the I²C EEPROM I used following function:
```C
void send_scale_data()
{
  int data_size = 0;  
  struct scale_data wdata;

  eeprom.readBlock(0, (uint8_t*)&data_size, sizeof(data_size));
  
  Serial.print("$S$");
  Serial.println(data_size);
  
  for (int i=0; i < data_size; i++)
  {
    eeprom.readBlock(sizeof(data_size)+i*sizeof(wdata), (uint8_t*)&wdata, sizeof(wdata));

    if (wdata.checksum != calc_checksum(&wdata)) {
      Serial.print("$E$ Wrong Checksum for data ");
      Serial.print(i);
      Serial.println();
    }
    
      Serial.print("$D$");
      Serial.print(i);
      Serial.print(',');
      Serial.print(wdata.year);
      Serial.print(',');
      Serial.print(wdata.month);
      Serial.print(',');
      Serial.print(wdata.day);
      Serial.print(',');
      Serial.print(wdata.hour);
      Serial.print(',');
      Serial.print(wdata.minute);
      Serial.print(',');
      Serial.print((float)wdata.weight / 10.0f);
      Serial.print(',');
      Serial.print((float)wdata.fat  / 10.0f);
      Serial.print(',');
      Serial.print((float)wdata.water / 10.0f);
      Serial.print(',');
      Serial.print((float)wdata.muscle  / 10.0f);
      Serial.print(',');
      Serial.print(wdata.checksum);
      Serial.print('\n');
  } 
}
```

For clear all stored values just set the data size at I²C EEPROM address 0 to zero, like in the following code snippet:

```C
void clear_scale_data()
{
  int data_size = 0;
  eeprom.writeBlock(0, (uint8_t*)&data_size, sizeof(data_size));
}
```

New measured values will now just overwriting the old stored values.

## Setting up a RTC module

I need a real time clock (RTC) module to get always the correct time and date, even if the Arduino is in deep sleep mode. First I used the [Tiny RTC I²C module](https://www.google.com/search?q=tiny+rtc+i2c+module) that has the DS1307 chip set on but somehow my Tiny RTC board lost always the time if the power supply was disconnected. I measured the battery voltage of the Tiny RTC but it was okay. I guess I ordered a faulty module. 

This time I bought a different [I²C RTC module](http://www.roboter-bausatz.de/31/rtc-ds3231-echtzeituhr-modul) which has a DS3231 chip set on, see figure 4.1 and 4.2. I connected the module like in the schematic in figure 2.4.

<table border="0">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/rtc_front.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/rtc_front.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 4.1: I²C RTC module (front)</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/rtc_back.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/rtc_back.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 4.2: I²C RTC module  (back)</sub>
</th>

  </tr>
</table>

I used the [DS3232RTC library](https://github.com/JChristensen/DS3232RTC) by JChristensen to get the current time and date. At first I setted up the current time and date by using the example sketch "SetSerial" which comes with the library. I just send a string with the format `yy,m,d,h,m,s` via the serial monitor of the Arduino IDE.  

Now I could read the current time and date with the [Arduino Time library functions](http://playground.arduino.cc/Code/Time) like in the following code snippet:

```C
#include <Wire.h>
#include <Time.h>
#include <DS3232RTC.h> 

void after_sleep_event()
{
...

  setSyncProvider(RTC.get);
  if (timeStatus() != timeSet) {
    Serial.println("$E$ Can't sync to RTC clock!");
  } else {
    Serial.println("$I$ Successful sync to RTC clock");
  }

  Serial.print("$I$ Time: ");
  Serial.print(hour());
  Serial.write(':');
  Serial.print(minute());
  Serial.write(':');
  Serial.print(second());
  Serial.print(" Date: ");
  Serial.print(day());
  Serial.write('/');
  Serial.print(month());
  Serial.write('/');
  Serial.print(year());
  Serial.println();

...
}
```

The function `setSyncProvider` will sync with the I²C RTC module and needs only called once when the Arduino woke up from the sleep mode.

## Connecting a HC-05 Bluetooth module to a tablet

For a wireless connection from my scale to my Nexus 10 tablet I am using a HC-05 Bluetooth 3.0 module, see figure 3.1 and figure 3.2. The HC-05 module can be operate as a master or slave device. Note the HC-06 Bluetooth module can only operate as a slave device.

<table border="0">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/bluetooth_front.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/bluetooth_front.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 3.1: HC-05 Bluetooth module (front)</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/bluetooth_back.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/bluetooth_back.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 3.2: HC-05 Bluetooth module  (back)</sub>
</th>

  </tr>
</table>

As I connected the module to the Arduino Pro Mini as the schematic in figure 2.4 then I already found the default device name in the paring list of my tablet. As default the device operates as a slave device with a paring code "1234" and a baud rate of 9600 bits/s. First I tested if I could send a test message from the Arduino with the following code:

```C
#include <Wire.h>

void setup()
{
  pinMode(13, OUTPUT);
  digitalWrite(13, HIGH);
  Serial.begin(9600);
}

void loop()
{
  Serial.println("Hello World!");
  delay(1000);
}
```

On my Nexus 10 tablet I installed the [Bluetooth Terminal](https://play.google.com/store/apps/details?id=Qwerty.BluetoothTerminal) app by Qwerty to display the test messages. I just had to couple the two devices. Please make sure that the CP2102 is not together with the Bluetooth module to the Arduino Pro Mini connected if you are using for both devices the TX and RX pin! Both devices could be confused. Sometimes I couldn't upload a new sketch to the Arduino Pro Mini or I couldn't send a Bluetooth message. To avoid these problem I would use two different digital pins (which I don't have left) for the HC-05 Bluetooth TX and RX pin. 

For changing the default parameters I had to send AT commands to the Bluetooth module. So in this case I had to connect together the Bluetooth HC-05 and the CP2101 device to the Arduino Pro Mini. To avoid the mentioned trouble I connected the Bluetooth HC-05 module for the configuration process as follow:

| HC-05 | Arduino Pro Mini |
|:-----:|:----------------:|
|  VCC  |        VCC       |
|  GND  |        GND       |
|  TXD  |         9        |
|  RXD  |         8        |
|  KEY  |         6        |

The KEY pin has to be pulled high to enter AT mode of the HC-05 Bluetooth module. Sometimes the KEY pin is not on the breakout board. If that the case you have to solder a wire directly to pin 34 on the HC-05 module, see the [HC-05 Manual](https://github.com/oliexdev/openScale/raw/master/doc/hc_05/HC_05_Manual.pdf) for more details. The default baud rate in AT command mode is 38400 bits/s. Use the following sketch to configure the HC-05 Bluetooth module:

```C
#include <Wire.h>
#include <SoftwareSerial.h>

SoftwareSerial BTSerial(9, 8); // RX | TX

void setup()
{
  pinMode(13, OUTPUT);
  digitalWrite(13, HIGH);

  pinMode(6, OUTPUT);
  digitalWrite(6, HIGH);

  Serial.begin(9600);
  Serial.println("Enter AT commands:");

  BTSerial.begin(38400);
}

void loop()
{
  if (BTSerial.available()) {
    Serial.write(BTSerial.read());
  }

  if (Serial.available()) {
    BTSerial.write(Serial.read());
  }
}
```

If you successfully in the AT mode the HC-05 Bluetooth module LED blinks every 2 seconds. Open the Serial Monitor in the Arduino IDE under Tools->Serial Monitor. Change "no line ending " to "both NL & CR" found just beside the baud rate. Now you can send a test command with `AT` if everything works fine you should receive a `OK`. Following is a short summary of the most important AT commands of the available [HC-05 AT Commands](https://github.com/oliexdev/openScale/raw/master/doc/hc_05/HC_05_AT_Commands.pdf):

| AT command | Response | Parameter | Comment |
|:----------:|:--------:|:---------:|:-------:|
| AT | OK | None | Test command |
| AT+VERSION? | +VERSION: <param>  OK | Version | Shows the version |
| AT+ORGL | OK | None | Restore module to default parameters |
| AT+NAME | OK | Bluetooth device name | Set device name |
| AT+ROLE | OK | 0 - slave mode 1- master mode 2 - slave-loop mode | Set the device role as slave or master |
| AT+PSWD | OK | Pairing code | Set the pairing code of the device |
| AT+BIND | OK | Bind device address | Set the device address which is automatically connected (works only if the device role is in master mode) |

Note if you want to set the bind device address to `12:34:56:ab:cd:ef`. The AT command should be `AT+BIND=1234,56,abcdef`. First I set the HC-05 Bluetooth module in the master mode and set my tablet device address with AT+BIND command. Everything works fine, the HC-05 automatically paired with my tablet, but after some reconnection’s (without a reproducible action) my tablet couldn't find any more the HC-05 Bluetooth device. Only after a full reset on the Bluetooth module with `AT+ORGL` and a full deletion in the pairing list on my tablet it works again. I couldn't figure out why this happened. But it's only occurred when the HC-05 Bluetooth module is in master mode. Therefore I switched back to the slave mode. In this mode everything works perfectly but my tablet had to act as a master device now.

## Reading the measured values with Arduino

In the previous step we had successful reverse engineered the scale's display. The next step will be to read and decode the signals with a microcontroller. I used the [Arduino Pro Mini board](http://arduino.cc/en/pmwiki.php?n=Main/ArduinoBoardProMini) (8 MHz, 3.3V) with an ATmega 328 microcontroller, see figure 2.1. For reducing the size and the power consumption the Arduino Pro Mini don't have USB on board, so I need an external USB to Serial converter for writing sketches. I bought a CP2102 USB to Serial converter, see figure 2.2 and figure 2.3. 

<table border="0">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/arduino_front.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/arduino_front.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 2.1: Arduino Pro Mini board</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/cp2102_front.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/cp2102_front.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 2.2: CP2102 USB to Serial converter (front)</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/cp2102_back.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/cp2102_back.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 2.3: CP2102 USB to Serial converter (back)</sub>
</th>

  </tr>
</table>

In the [Arduino IDE](http://arduino.cc/en/pmwiki.php?n=Main/Software) I navigated to Tools->Board and selected the "Arduino Pro or Pro Mini (3.3V, 8MHz) w/ ATmega328" and also selected under Tools->Programmer the "AVR ISP" programmer. Now I could connect the CP2102 board with the Arduino Pro Mini as follow:

| CP2102 | Arduino Pro Mini |
|:------:|:----------------:|
|   DTR  |        DTR       |
|   RXI  |        TXO       |
|   TXO  |        RXI       |
|   3V3  |        VCC       |
|   GND  |        GND       |

Note that I used 3.3V (see pin 3V3 on figure 2.3) from the CP2102 converter board and not the 5V! If I did it right I was able to program the microcontroller, for example with the blink sketch, using File->Upload. 

I connected the scale's display connector to the Arduino Pro Mini as the following schematic:

<p align="center">
<a href="https://github.com/oliexdev/openScale/raw/master/doc/parts/schematic.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/parts/schematic.png' width='300px' alt='missing' /> </a> <br>
<sub><b>Figure 2.4: Schematic overview of the openScale project</b></sub>
</p>

Next step is to program the Arduino Pro Mini to read the signal of the scale's display. First I set some defines in C to make the programming more clear:

```C
...
#define SEG_1_1 4
#define SEG_1_2 5
#define SEG_2_1 6
#define SEG_2_2 7
#define SEG_3_1 8
#define SEG_3_2 9
#define SEG_4_1 10
#define SEG_4_2 11
#define UP 12
#define C0 A0
#define C1 A1
#define C2 A2
#define C3 A3
...
```

In the `setup()` function I set the pin modes as follow:
```C
void setup() 
{
...
  pinMode(SEG_1_1, INPUT);
  pinMode(SEG_1_2, INPUT);
  pinMode(SEG_2_1, INPUT);
  pinMode(SEG_2_2, INPUT);
  pinMode(SEG_3_1, INPUT);
  pinMode(SEG_3_2, INPUT);
  pinMode(SEG_4_1, INPUT);
  pinMode(SEG_4_2, INPUT);
  pinMode(UP, OUTPUT);
  pinMode(C0, INPUT);
  pinMode(C1, INPUT);
  pinMode(C2, INPUT);
  pinMode(C3, INPUT);
...
}
```

After I finished the initializing I had to read the signals in the `loop()` function. As you can see in figure 1.12-1.23 the signal width is 400kHz (2.5ms). It's much slower than the Arduino Pro Mini with 8MHz (0.000125ms). But beware using the `digitalRead` command in the Arduino library is relative slow. For this reason I made a own function that reads directly the ports from the microprocessor. This also makes sure I read definitely all signals at (almost) the same time.

```C
...
int seg_raw_1_1[4];
int seg_raw_1_2[4];
int seg_raw_2_1[4];
int seg_raw_2_2[4];
int seg_raw_3_1[4];
int seg_raw_3_2[4];
int seg_raw_4_1[4];
int seg_raw_4_2[4];
...

void set_seg_raw(int cycle_n)
{
  seg_raw_1_1[cycle_n] = (port_digital_pinA & (1 << 4)) ? 1 : 0;
  seg_raw_1_2[cycle_n] = (port_digital_pinA & (1 << 5)) ? 1 : 0;
  seg_raw_2_1[cycle_n] = (port_digital_pinA & (1 << 6)) ? 1 : 0;
  seg_raw_2_2[cycle_n] = (port_digital_pinA & (1 << 7)) ? 1 : 0;
  seg_raw_3_1[cycle_n] = (port_digital_pinB & (1 << 0)) ? 1 : 0;
  seg_raw_3_2[cycle_n] = (port_digital_pinB & (1 << 1)) ? 1 : 0;
  seg_raw_4_1[cycle_n] = (port_digital_pinB & (1 << 2)) ? 1 : 0;
  seg_raw_4_2[cycle_n] = (port_digital_pinB & (1 << 3)) ? 1 : 0;
}
``` 

And in the `loop()` function I read the ports and then store the signals depending on the cycle number with:

```C
char port_control;
char port_digital_pinA;
char port_digital_pinB;

int control_bit[4];

void loop()
{
...
  port_control = PINC;
  port_digital_pinA = PIND;
  port_digital_pinB = PINB;

  control_bit[0] = (port_control & (1 << 0)) ? 1 : 0;
  control_bit[1] = (port_control & (1 << 1)) ? 1 : 0;
  control_bit[2] = (port_control & (1 << 2)) ? 1 : 0;
  control_bit[3] = (port_control & (1 << 3)) ? 1 : 0;

  if (control_bit[0] == LOW && control_bit[1] == HIGH && control_bit[2] == HIGH && control_bit[3] == HIGH)
  {
    set_seg_raw(0);

  } 
  else if (control_bit[0] == HIGH && control_bit[1] == LOW && control_bit[2] == HIGH && control_bit[3] == HIGH)
  {
    set_seg_raw(1);

  } 
  else if (control_bit[0] == HIGH && control_bit[1] == HIGH && control_bit[2] == LOW && control_bit[3] == HIGH)
  {
    set_seg_raw(2);

  } 
  else if (control_bit[0] == HIGH && control_bit[1] == HIGH && control_bit[2] == HIGH && control_bit[3] == LOW)
  { 
    set_seg_raw(3);

  } 
  else if (control_bit[0] == HIGH && control_bit[1] == HIGH && control_bit[2] == HIGH && control_bit[3] == HIGH)
  {      
    no_activity_cycles++;
  }
...
}
```

You can find the port/pin mapping of the ATmega 328 on the [Arduino website](http://arduino.cc/en/Reference/Atmega168Hardware). For the decoding process I used the reverse engineered 8-Bit decoding table:
```C
char decode_seg(int seg_x[4], int seg_y[4])
{
  boolean b = seg_x[0];
  boolean c = seg_x[1];
  boolean e = seg_x[2];
  boolean f = seg_x[3];
  boolean a = seg_y[0];
  boolean d = seg_y[1];
  boolean g = seg_y[2];
  boolean x = seg_y[3];

  if (!e && !c && !b && !f &&
    !g &&  !d && !a)
    return ' ';

  if (e && !c && b && f &&
    g &&  d && a)
    return '0';

  if (e && !c && b && !f &&
    !g &&  !d && !a)
    return '1';

  if (!e && c && b && f &&
    g &&  !d && a)
    return '2';

  if (e && c && b && f &&
    !g &&  !d && a)
    return '3';

  if (e && c && b && !f &&
    !g &&  d && !a)
    return '4';

  if (e && c && !b && f &&
    !g &&  d && a)
    return '5';

  if (e && c && !b && f &&
    g &&  d && a)
    return '6';

  if (e && !c && b && !f &&
    !g &&  !d && a)
    return '7';

  if (e && c && b && f &&
    g &&  d && a)
    return '8';

  if (e && c && b && f &&
    !g &&  d && a)
    return '9';

  if (!e && c && !b && !f &&
    !g &&  !d && !a)
    return '-';

  if (!e && c && b && !f &&
    g &&  d && a)
    return 'P';

  if (e && !c && b && !f &&
    g &&  d && a)
    return 'M';

  if (!e && c && !b && f &&
    g &&  d && a)
    return 'E';   

  if (!e && c && !b && !f &&
    g &&  d && a)
    return 'F';   

  if (e && c && b && !f &&
    g &&  d && !a)
    return 'H';   

  return -1;
}
``` 

As you may notice I did a right shift in the decoding table, so now the used decoding table looks like:

<th>
<table>
  <tr>
    <th>Cycle No.<br></th>
    <th>0<br></th>
    <th>1<br></th>
    <th>2</th>
    <th>3</th>
  </tr>
  <tr>
    <td>Signal B<br></td>
    <td>a</td>
    <td>d</td>
    <td>g</td>
    <td>x</td>
  </tr>
  <tr>
    <td>Signal A<br></td>
    <td>b</td>
    <td>c</td>
    <td>e</td>
    <td>f</td>
  </tr>
</table>

I don't know why I needed this shift. I am assuming that I exchanged the C1 and C3 pin on the Arduino Pro Mini!? Regardless, I could successful decoded the measured values of the scale with:

```C
char seg_value_1;
char seg_value_2;
char seg_value_3;
char seg_value_4;

void loop()
{
...
  seg_value_1 = decode_seg(seg_raw_1_1, seg_raw_1_2);
  seg_value_2 = decode_seg(seg_raw_2_1, seg_raw_2_2);
  seg_value_3 = decode_seg(seg_raw_3_1, seg_raw_3_2);
  seg_value_4 = decode_seg(seg_raw_4_1, seg_raw_4_2);
...
}
```
To make the measured values more stable against glitches I added a running median. I used the [running median library](http://playground.arduino.cc/Main/RunningMedian) by robtillaart and added following code to the loop, so that every sixth measure a median is used as a valid value.

```C
#include <RunningMedian.h>

#define MAX_SAMPLE_SIZE 6

RunningMedian<char, MAX_SAMPLE_SIZE> seg_samples_1;
RunningMedian<char, MAX_SAMPLE_SIZE> seg_samples_2;
RunningMedian<char, MAX_SAMPLE_SIZE> seg_samples_3;
RunningMedian<char, MAX_SAMPLE_SIZE> seg_samples_4;

int sample_count = 0;

int measured_weight = -1;
int measured_fat = -1;
int measured_water = -1;
int measured_muscle = -1;

void loop()
{
...
  if (seg_value_1 != -1 && seg_value_2 != -1 && seg_value_3 != -1 && seg_value_4 != -1)
  {
    seg_samples_1.add(seg_value_1);
    seg_samples_2.add(seg_value_2);
    seg_samples_3.add(seg_value_3);
    seg_samples_4.add(seg_value_4);

    sample_count++;
  }


  if (sample_count > MAX_SAMPLE_SIZE)
  {
    seg_samples_1.getMedian(seg_value_1);
    seg_samples_2.getMedian(seg_value_2);
    seg_samples_3.getMedian(seg_value_3);
    seg_samples_4.getMedian(seg_value_4);
   
     if (seg_value_4 == ' ') {
       measured_weight = char_to_int(seg_value_1) + char_to_int(seg_value_2)*10 + char_to_int(seg_value_3)*100;
     }
   
     if (seg_value_4 == 'F') {
       measured_fat = char_to_int(seg_value_1) + char_to_int(seg_value_2)*10 + char_to_int(seg_value_3)*100;
     }
     
     if (seg_value_4 == 'H') {
       measured_water = char_to_int(seg_value_1) + char_to_int(seg_value_2)*10 + char_to_int(seg_value_3)*100;
     }
     
     if (seg_value_4 == 'M') {
       measured_muscle = char_to_int(seg_value_1) + char_to_int(seg_value_2)*10 + char_to_int(seg_value_3)*100;
     }

    sample_count = 0; 
  }
...
}
```
Note to save memory usage the measured values are stored as integers and not as floating points. You can see the full source code in the [openScale repository](https://raw.githubusercontent.com/oliexdev/openScale/master/arduino_mcu/openScale_MCU/openScale_MCU.ino).

## Reverse Engineering of the scale

First of all I had to find a suitable bathroom scale that I wanted to reverse engineer. I was searching for a cheap bathroom scale that can analyse not only my weight but also my body fat, water and muscle. The scale design should be clear and the display of the scale should have some kind of a simple seven segment display (I hoped that a simple display would be easier to reverse engineer). The [Sanitas SBF12 scale](http://www.sanitas-online.de/web/en/products/weight/SBF12.php) that I found in a department store seemed to be right for my purpose. So I bought one for only 20€ (around 25$), see figure 1.1.

<p align="center">
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/package.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/package.JPG' width='300px' alt='image missing' /> </a> <br>
<sub><b>Figure 1.1: Sanitas SBF 12 scale</b></sub>
</p>

To gain access to the inside of the scale I had to unscrew only a few screws on the backside of the scale, see figure 1.2. Surprisingly the circuit board is well labelled and structured. As you can see on figure 1.3 the pins for GND, VDD, UP, DOWN, SET and the pressure sensors pins are easy to find. On the left side on figure 1.4 you can see the vibration sensor and the two outgoing wires going to a 3.3V coin cell battery. On the top you can see the connector pins for the scale's display. But unfortunately the microprocessor of the scale was hidden under a big black blob of some kind of plastic on the middle of the circuit board.

<table border="0">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scale_overview_raw.jpg" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scale_overview_raw.jpg' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.2: Scale overview</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/circuit_board_back_raw.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/circuit_board_back_raw.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.3: Back side of the circuit board</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/circuit_board_front_raw.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/circuit_board_front_raw.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.4: Front side of the circuit board</sub>
</th>

  </tr>
</table>


Because I couldn't identify the name of the used microprocessor and I also didn't have access to the microprocessor’s pins, I had only two options left to read the measurements from the scale. One option was to read the raw values from the sensors and do the weight calculation and the impedance calculation on my own. The other option was to decode the scale's display. The idea of the second option was that all calculations of the body analyses are done by the scale as usually and you only had to decode the scale's display with an attached microprocessor. The first option was too complicated and the change that I am doing something wrong with the calculation was too high, so I tried the second option.

The first step was to connect all pins with wires that are routed to the display connector. Luckily on the circuit board are testing points for the display, which you can perfectly use for soldering a wire on. I recommend using enamelled coper wire because it's very thin and you can solder it very easily. You can see my result in the figures 1.5-1.9.

<table border="0">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/circuit_board_back_wired_unorderd.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/circuit_board_back_wired_unorderd.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.5: Connected wires unordered</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/circuit_board_back_wired_ordered.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/circuit_board_back_wired_ordered.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.6: Connected wires ordered</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scale_overview_back_wired.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scale_overview_back_wired.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.7: Back side of the scale wired</sub>
</th>

  </tr>

  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scale_overview_front_wired.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scale_overview_front_wired.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.8: Front side of the scale wired</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/wired_with_notes.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/wired_with_notes.JPG' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.9: Attached notes to the wires</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/display_connector_pin_layout.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/display_connector_pin_layout.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.10: Pin layout of the display connector</sub>
</th>

  </tr>
</table>

Next step is to analyse the signals that are controlling the display. I had the opportunity to use a 16  digital channels oscilloscope ([Agilent Technologies MSO7014B](http://www.keysight.com/en/pd-1788165-pn-MSO7014B/mixed-signal-oscilloscope-100-mhz-4-analog-plus-16-digital-channels))  for this step, see figure 1.11. Alternative you can use a microcontroller like [Arduino](http://www.arduino.cc/) to read the signals. 

<p align="center">
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/oscilloscope.JPG" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/oscilloscope.JPG' width='300px' alt='image missing' /> </a> <br>
<sub><b>Figure 1.11: The 16  digital channels oscilloscope for analysing the signals</b></sub>
</p>

Because I couldn’t connected all display connector pins to the oscilloscope I left pin 1 and pin 2 unconnected. So I had the following pin mapping:

| display pin | oscilliscope | label      |
|:-----------:|:------------:|:----------:|
|      1      | not connected| Sym1:1     |
|      2      | not connected| Sym1:2     |
|      3      |      D0      | Seg1:1     |
|      4      |      D1      | Seg1:2     |
|      5      |      D2      | Seg2:1     |
|      6      |      D3      | Seg2:2     |
|      7      |      D4      | Seg3:1     |
|      8      |      D5      | Seg3:2     |
|      9      |      D6      | Seg4:1     |
|      10     |      D7      | Seg4:2     |
|      11     |      D8      | Sym2:1     |
|      12     |      D9      | Sym2:2     |
|      13     |      D10     | not used   |
|      14     |      D11     | not used   |
|      15     |      D12     | C3         | 
|      16     |      D13     | C2         |
|      17     |      D14     | C1         |
|      18     |      D15     | C0         |

The question for the reverse engineering process was how the signals would respond when a value of the display had changed? Could I find some logic behind the signal changes? As I turned on the scale the first observation was that not all pins are active and that four pins (D12-D15) had always the same repeating pattern, no matter which value was displaying. So these four pins must be the cycle pins for a serial communication. The second observation was if the first digit of the four digit scale was displaying a value D0 and D1 were active. If two digits are displaying some values then the signals D0-D3, for three digits D0-D5 and for four digits D0-D7 were active. The obvious conclusion was to display a digit on the scale a pair of two signals are used. In short I knew a command is four cycles long. I also knew that pin D0 and D1 are used for displaying the first digit. So the display controller uses 2x4 Bit = 8 Bit for representing a digit. If I can decode these 8 Bit word, I would be able decode the other signals too. The next step was to see the behaviour of the two signals D0 and D1 when different digits (0-9) are shown on the scale’s display. The best way to display each digit on the display was by pushing the up or down button of the scale to get into the selection mode of the scale. In this mode I was able to click through the values “P-01” to “P-09”. The correspondent recorded signals are shown in figure 1.12-1.20. Note that valid signals are near the right side of the signal diagram because on the left side old signals could be still visible. 

<table border="0">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P01.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P01.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.12: signals of value "P-01"</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P02.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P02.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.13: signals of value "P-02"</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P03.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P03.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.14: signals of value "P-03"</sub>
</th>

  </tr>

  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P04.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P04.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.15: signals of value "P-04"</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P05.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P05.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.16: signals of value "P-05"</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P06.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P06.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.17: signals of value "P-06"</sub>
</th>

  </tr>

  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P07.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P07.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.18: signals of value "P-07"</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P08.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P08.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.19: signals of value "P-08"</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P09.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_P09.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.20: signals of value "P-09"</sub>
</th>

  </tr>

  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_0_0kg.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_0_0kg.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.21: signals of value "0.0 kg"</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_person.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_person.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.22: signals while displaying the person symbol, see signal on D8 and D9</sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_person_age.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/scope_person_age.png' width='250px' alt='image missing' /> </a> <br>
<sub>Figure 1.23: signals while displaying the person and age symbol, see signal D8 and D9</sub>
</th>

  </tr>
</table>

For decoding the 8 Bit word I compared the recorded signals. I first chose two numbers that differs only in one segment of the seven segment display. For example the digit 5 and 6 differs only in segment g, see figure 1.24. If you compare the corresponded signals in figure 1.16 and 1.17 you will see that only one bit on pin D1 has changed. So I knew that cycle no. 1 bit on pin D1 was for segment g of the seven segment display. Another example was the signal difference between digit 6 (figure 1.17) and 8 (figure 1.19). Only cycle no. 3 bit on pin D0 has changed. Therefore this bit was responsible for turning segment b on or off. After I did this kind of comparison with all digits (it’s like playing Sudoku) I finally got the following decoding table:

<table>
  <tr>
<th>
<a href="https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/seven_segement.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/sanitas_sbf12/seven_segement.png' width='200px' alt='image missing' /> </a> <br>
<sub>Figure 1.24: seven segment display</sub>
</th>

<th>
<table>
  <tr>
    <th>Cycle No.<br></th>
    <th>0<br></th>
    <th>1<br></th>
    <th>2</th>
    <th>3</th>
  </tr>
  <tr>
    <td>Signal B<br></td>
    <td>d</td>
    <td>g</td>
    <td>x</td>
    <td>a</td>
  </tr>
  <tr>
    <td>Signal A<br></td>
    <td>c</td>
    <td>e</td>
    <td>f</td>
    <td>b</td>
  </tr>
</table>
8-Bit word decoding table
</th>

</table>

## How its started ...
I gained a little weight a while ago, not overweighted, but I decided to do something about it. So I started my first diet. I bought a diet book in a book shop and followed it strictly about 2 weeks. At the beginning it was really tough, but in the end my diet worked out. I lost around 10kg (around 22lb). In that time I checked my weight every two or three days with my normal digital bathroom scale. That's helped me to stay motivated because I was happy to see that I gradually lost some weight. The disappointment was that my diet progress was not tracked in any way. Of course I could create a spreadsheet and wrote down my weight values but that would be troublesome and not seasonable in a world of smartphones and tablets.

My first intention was to buy a ready to use wireless scale. So I searched on the internet for a wireless bathroom scale that can track my weight and display my diet progress on a smartphone or tablet. I found a couple of start-ups and companies which have wireless scales, for example [FitBit Aria](http://www.fitbit.com/de/aria) , [Withings Smart Body Analyzer](http://www.withings.com/de/smart-body-analyzer.html) , [Runstatic libra](https://www.runtastic.com/shop/de/runtastic-libra-scale) , [Medisana BS-430](http://www.medisana.de/Gesundheitskontrolle/Personenwaagen/BS-430-connect-Koerperanalysewaage-HausMed.html) and [Beurer BF-800](http://www.beurer.com/web/en/products/weight/diagnostic_scales/BF-800-white). Most of the design of the wireless scales and the app design are really good but the big drawback is that (except the Beurer BF-800 scale) sends all your data directly into the cloud of each company. Beside that you lost your privacy and the high prices of the scales, you need always an internet connection. You are heavily dependent to the company servers. For example some Withings users reported that they have always a delay between weighting and seeing the result in the app. All in all I don't want to give any health data to a company. The Beurer BF-800 was the only scale that doesn't send automatically all your data to a cloud (of course it's possible with Beurer-Connect). I decided to give Beurer BF-800 a try, so I bought one. I installed the [Beurer Health Manager](https://play.google.com/store/apps/details?id=com.beurer.connect.healthmanager) on my Nexus 10 tablet and tried to connect the scale via Bluetooth with my tablet. It didn't work. My tablet has Bluetooth 3.0 and the scale works with Bluetooth 4.0 (also named as Bluetooth Smart) and it is not downwards compatible (didn't know that before). To be fair on the Beurer website is a [list of compatible smartphones](http://www.beurer.com/web/we-dokumente/gebrauchsanweisungen/list-compatible-smartphones.pdf). But the Beurer Health Manager lacks also in design and many users reported on Google Play that they are unsatisfied with the app. Furthermore I don't want to buy extra a new tablet or smartphone that has Bluetooth 4.0 for only logging my weight. So I sent the scale back. 

Frustrated about the available commercial scales and the idea to have a new little project, I decided to make my own wireless bathroom scale. First of all I searched for information of similar projects. The resources that I have found are [SdCardBathroomScale](http://code.google.com/p/casainho-projects/wiki/SdCardBathroomScale) , [SmartScale](http://code.google.com/p/casainho-projects/wiki/SmartScale), [Bluetooth wireless bathroom scale](http://www.keyboardmods.com/2010/05/bluetooth-wireless-bathroom-scale-with.html), [hacking a digital bathroom scale](http://scanlime.org/2010/01/hacking-a-digital-bathroom-scale/) and the book [Arduino and Kinect Projects](http://www.apress.com/9781430241676). Both projects SdCardBathroomScale and SmartScale by casainho are really good. I wanted something similar. My goal was to have an appealing wireless scale that works with Bluetooth 3.0. All modifications of the scale should be hidden, so it appears like a normal scale. Also the constant connection to a Bluetooth device should not be required for the scale. Measured values should be temporary stored in the scale. Only if a Bluetooth device is around the scale, new measured values would transferred. Of course the power consumption of the scale and the microcontroller should be low as possible. Furthermore all data should be locally stored in the app or on the scale itself, so that the user has full control of their data. The app design should be light and simple. In addition I wanted to learn how to write an Android app from scratch. To fulfil all my requirements the  project "openScale" was born.