<!-- 
<p align="center">
<a href="https://github.com/oliexdev/openScale/raw/master/doc/screens/screen_graph.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/screens/screen_graph.png' width='300px' alt='missing' /> </a> <br>
<sub>Caption</sub>
</p>
-->

# openScale wiki
openScale is an Open Source App for Android to keep a log of your body weight, fat, water and muscle.

## Writing an Android app

## Reducing the power consumption

## Adding external I²C EEPROM

## Setting up a RTC module

## Connecting a HC-05 Bluetooth module to a tablet

## Reading the measured values with Arduino

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
|:-----------:|:------------:|:-----------|
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

The question for the reverse engineering process was how the signals will respond when a value of the display had changed? Could I find some logic behind the signal changes? As I turned on the scale the first observation was that not all pins are active and that four pins (D12-D15) had always the same repeating pattern, no matter which value was displaying. So these four pins must be the cycle pins for a serial communication. The second observation was if the first digit of the four digit scale was displaying a value D0 and D1 were active. If two digits are displaying some values then the signals D0-D3, for three digits D0-D5 and for four digits D0-D7 were active. The obvious conclusion was to display a digit on the scale a pair of two signals are used. In short I knew a command is four cycles long. I also knew that pin D0 and D1 are used for displaying the first digit. So the display controller uses 2x4 Bit = 8 Bit for representing a digit. If I can decode these 8 Bit word, I would be able decode the other signals too. The next step was to see the behaviour of the two signals D0 and D1 when different digits (0-9) are shown on the scale’s display. The best way to display each digit on the display was by pushing the up or down button of the scale to get into the selection mode of the scale. In this mode I was able to click through the values “P-01” to “P-09”. The correspondent recorded signals are shown in figure 1.12-1.20. Note that valid signals are near the right side of the signal diagram because on the left side old signals could be still visible. 

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

Frustrated about the available commercial scales and the idea to have a new little project, I decided to make my own wireless bathroom scale. First of all I searched for information of similar projects. The resources that I have found are [SdCardBathroomScale](http://code.google.com/p/casainho-projects/wiki/SdCardBathroomScale) , [SmartScale](http://code.google.com/p/casainho-projects/wiki/SmartScale), [Bluetooth wireless bathroom scale](http://www.keyboardmods.com/2010/05/bluetooth-wireless-bathroom-scale-with.html), [hacking a digital bathroom scale](http://scanlime.org/2010/01/hacking-a-digital-bathroom-scale/) and the book [Arduino and Kinect Projects](http://www.apress.com/9781430241676). Both projects SdCardBathroomScale and SmartScale by casainho are really good. I wanted something similar. My goal was to have an appealing wireless scale that works with Bluetooth 3.0. All modifications of the scale should be hidden, so it appears like a normal scale. Also the constant connection to a Bluetooth device should not be required for the scale. Measured values should be temporary stored in the scale. Only if a Bluetooth device is around the scale, new measured values will transferred. Of course the power consumption of the scale and the microcontroller should be low as possible. Furthermore all data should be locally stored in the app or on the scale itself, so that the user has full control of their data. The app design should be light and simple. In addition I wanted to learn how to write an Android app from scratch. To fulfil all my requirements the  project "openScale" was born.