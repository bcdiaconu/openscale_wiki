## How to support your scale

If you want to help to support your Bluetooth scale please see [here](https://github.com/oliexdev/openScale/wiki/How-to-reverse-engineer-a-Bluetooth-4.x-scale) for further information.

## Overview

<table>
<tr>
<td><b>Scale</b></td>
<td align="center"><b>Initialisation process</b></td>
<td align="center"><b>History data</b></td>
<td align="center"><b>Body metrics</b></td>
<td><b>Remarks / Known Issues</b></td>
</tr>
<tr>
<td>Custom Bluetooth scale</td>
<td align="center">&#10003;</td>
<td align="center">&#10003;</td>
<td align="center">&#10003;</td>
<td>Works only with Bluetooth 3.x</td>
</tr>
<tr>
<td>Xiaomi Mi scale v1</td>
<td align="center">&#10003;</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td>No data on the scale is ever deleted</td>
</tr>
<tr>
<td>Medisana BS444</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">&#10003;</td>
<td>-</td>
</tr>
<tr>
<td>Sanitas SBF70</td>
<td align="center">&#10003;</td>
<td align="center">&#10003;</td>
<td align="center">&#10003;</td>
<td>assigned user name on the scale and in openScale have to be the same, see 
<a href="https://github.com/oliexdev/openScale/wiki/FAQ/_edit#how-do-i-get-on-my-sbf70-scale-the-body-metrics-water-muscle-fat-percentage">FAQ</a></td>
</tr>
<tr>
<td>Digoo DG-S038H</td>
<td align="center">o</td>
<td align="center">n/a</td>
<td align="center">&#10003;</td>
<td>Initialisation process needs to be reverse engineered. Currently you have to use one time the vendors app to initialise the scale.</td>
</tr>
<tr>
<td>Yunmai Mini</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">o</td>
<td>Only the weight and body fat are supported by openScale. The rest of the body metrics are currently not supported because of the not transparent impedance calculation by the company, see issue <a href="https://github.com/oliexdev/openScale/issues/71">#71</a></td>
</tr>
<tr>
<td>Excelvan CF369BLE</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">&#10003;</td>
<td>-</td>
</tr>
<tr>
<td>Yunmai SE</td>
<td align="center">o</td>
<td align="center">n/a</td>
<td align="center">n/a</td>
<td>Weight unit is currently not set via openScale</td>
</tr>
<tr>
<td>MGB</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">&#10003;</td>
<td>-</td>
</tr>
<tr>
<td>Xiaomi Mi scale v2</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">o</td>
<td>Only the weight are supported by openScale. The rest of the body metrics are currently not supported because of the not transparent impedance calculation by the company</td>
</tr>
<tr>
<td>Exingtech Y1</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">&#10003;</td>
<td>-</td>
</tr>
<tr>
<td>Beurer BF700/800</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">&#10003;</td>
<td>-</td>
</tr>
<tr>
<td>Silvercrest SBF75</td>
<td align="center">&#10003;</td>
<td align="center">&#10003;</td>
<td align="center">&#10003;</td>
<td>assigned user name on the scale and in openScale have to be the same, see 
<a href="https://github.com/oliexdev/openScale/wiki/FAQ/_edit#how-do-i-get-on-my-sbf70-scale-the-body-metrics-water-muscle-fat-percentage">FAQ</a></td>
</tr>
<tr>
<td>Runtastic Libra</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">&#10003;</td>
<td>-</td>
</tr>
<tr>
<td>Hesley (Yunchen)</td>
<td align="center">&#10003;</td>
<td align="center">n/a</td>
<td align="center">&#10003;</td>
<td>-</td>
</tr>
</table>
&#10003; : supported in openScale <br>
o : supported by the scale but still needs to be reverse engineered<br>
n/a : not available on the scale 

<!-- DivTable.com -->

## Detailed information

_Please select a scale for further information_

<table border="0" align="center">
  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/wiki/Custom-Bluetooth-Scale">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/custom_scale/parts/scale_hacked_front.JPG' width='270px' alt='image missing' /> </a> <br>
<sub><a href="https://github.com/oliexdev/openScale/wiki/Custom-Bluetooth-Scale">Custom Bluetooth scale</a></sub>
</th>

<th>
<a href="https://github.com/oliexdev/openScale/wiki/Xiaomi-Bluetooth-Mi-Scale">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/scales/miscale.jpg' width='200px' alt='image missing' /> </a> <br>
<sub><a href="https://github.com/oliexdev/openScale/wiki/Xiaomi-Bluetooth-Mi-Scale">Xiaomi Mi scale v1</a></sub>
</th>

  </tr>

  <tr>

<th>
<a href="https://github.com/oliexdev/openScale/wiki/Medisana-BS444">
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/medisana_bs444.png' width='250px' alt='image missing' /> </a> <br>
<sub><a href="https://github.com/oliexdev/openScale/wiki/Medisana-BS444">Medisana BS444 scale</a></sub>
</th>

<th>
<img src='https://github.com/oliexdev/openScale/raw/master/doc/scales/sanitas_sbf70.jpg' width='280px' alt='image missing' /> <br>
<sub>Sanitas SBF70</sub>
</th>

  </tr>

  <tr>

<th>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/Digoo%20DG-S038H.jpeg' width='250px' alt='image missing' /> <br>
<sub>Digoo DG-S038H</sub>
</th>

<th>
<img src='https://github.com/oliexdev/openScale/raw/master/doc/scales/yunmai_mini.jpg' width='250px' alt='image missing' /> <br>
<sub>Yunmai Mini</sub>
</th>
</tr>

<tr>
<th>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/excelvan_cf369ble.jpg' width='250px' alt='image missing' /> <br>
<sub>Excelvan CF369BLE</sub>
</th>

<th>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/yunmai_se.jpeg' width='250px' alt='image missing' /> <br>
<sub>Yunmai SE</sub>
</th>

  </tr>
</tr>

<tr>
<th>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/mgb.jpg' width='250px' alt='image missing' /> <br>
<sub>MGB</sub>
</th>

<th>
<br>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/mi_scalev2.jpg' width='250px' alt='image missing' /> <br>
<sub>Xiaomi Mi scale v2</sub>
</th>

  </tr>
<tr>
<th>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/exingtech_y1.jpg' width='250px' alt='image missing' /> <br>
<sub>Exingtech Y1</sub>
</th>

<th>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/beurer_bf700.jpg' width='300px' alt='image missing' /> <br>
<sub>Beurer BF700/800</sub>
</th>

  </tr>

<tr>
<th>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/silvercrest_sbf75.jpg' width='250px' alt='image missing' /> <br>
<sub>Silvercrest SBF75</sub>
</th>

<th>
<br>
<sub>Runtastic Libra</sub>
</th>

  </tr>

<tr>
<th>
<img src='https://github.com/oliexdev/openScale/blob/master/doc/scales/hesley.jpg' width='250px' alt='image missing' /> <br>
<sub>Hesley (Yunchen) scale</sub>
</th>

<th>
<br>
<sub></sub>
</th>

  </tr>
</table>