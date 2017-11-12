Following are body metric estimations that are implemented in openScale based on scientific publications

## Body fat estimation

<table>
<tr>
<td><b>Author</b></td>
<td><b>Publication</b></td>
<td><b>Formula</b></td>
</tr>
<tr>
<td>Deurenberg et. al (1991)</td>
<td>"Body mass index as a measure of body fatness: age- and sex-specific prediction formulas" in British Journal of Nutrition vol 65, Issue 2 pp. 105-114</td>
<td> 
Male: (BMI * 1.2) + (AGE * 0.23) - 16.2 <br>
Female: (BMI * 1.2) + (AGE * 0.23) - 5.4
</td>
</tr>
<tr>
<td>Deurenberg (1992)</td>
<td>"The  assessment  of  body  composition:  use  and
misuse" Annual report Nestle Foundation, 35 - 72</td>
<td> AGE >= 16: (1.2 * BMI) + (0.23*AGE) - (10.8*(isMale) - 5.4 <br>
AGE < 16: (1.294 * BMI) + (0.20*AGE) - (11.4*isMale) - 8.0</td>
</tr>
<tr>
<td>Eddy et. al (1976)</td>
<td>Found in "Research into Obesity" Compiled by James W, Research into Obesity, HMSO 1976, p9. ISBN 11 450034 7</td>
<td> Male: (1.281* BMI) - 10.13 <br>
Female: (1.48* BMI) - 7.0</td>
</tr>
<tr>
<td>Gallagher et. al [non-asian] (2000)</td>
<td>"Healthy percentage body fat ranges: an approach for developing guidelines based on body mass index" in American Society for Clinical Nutrition</td>
<td> non-asian Male: 64.5 - 848.0 * (1.0 / BMI) + 0.079 * AGE - 16.4 + 0.05 * AGE + 39.0 * (1.0f / BMI) <br>
non-asian Female: 64.5 - 848.0 * (1.0f / BMI) + 0.079 * AGE</td>
</tr>
<tr>
<td>Gallagher et. al [asian] (2000)</td>
<td>"Healthy percentage body fat ranges: an approach for developing guidelines based on body mass index" in American Society for Clinical Nutrition</td>
<td>asian Male: 51.9 - 740.0 * (1.0 / BMI) + 0.029 * AGE<br>
asian Female: 64.8 - 752.0 * (1.0f / BMI) + 0.016 * AGE
</tr>
</table>

## Body water estimation

<table>
<tr>
<td><b>Author</b></td>
<td><b>Publication</b></td>
<td><b>Formula</b></td>
</tr>
<tr>
<td>Behnke et. al (1963)</td>
<td>"ANTHROPOMETRIC EVALUATION OF BODY COMPOSITION THROUGHOUT LIFE" in Annals of the New York Academy of Sciences vol 110</td>
<td> 
Male: 0.72 * (0.204 * BODY_HEIGHT^2) / 100.0 <br>
Female: 0.72 * (0.18 * BODY_HEIGHT^2) / 100.0
</td>
</tr>
<tr>
<td>Delwaide-Crenier et. al (1973)</td>
<td>"Body potassium as related to lean body mass measured by total water determination and by anthropometric method" in Human Biology 45, 509-526</td>
<td> 0.72 * (-1.976 + 0.907 * WEIGHT)</td>
</tr>
<tr>
<td>Hume & Weyers (1971)</td>
<td>"Relationship between total body water and surface area in normal and obese subjects" in J Clin Path. 1971; 24: 234-238</td>
<td> Male: (0.194786 * BODY_HEIGHT) + (0.296785 * WEIGHT) - 14.012934<br>
Female: (0.34454 * BODY_HEIGHT) + (0.183809 * WEIGHT) - 35.270121</td>
</tr>
<tr>
<td>Lee, Song, Kim, Lee et. al (2001)</td>
<td>"Assessment of total body water from anthropometry‐based equations using bioelectrical impedance as reference in Korean adult control and haemodialysis subjects" in Nephrology Dialysis Transplantation, Volume 16, Issue 1, 1 January 2001, Pages 91–97</td>
<td> Male: -28.3497 + (0.243057 * BODY_HEIGHT) + (0.366248 * WEIGHT) <br>
Female: -26.6224 + (0.262513 * BODY_HEIGHT) + (0.232948 * WEIGHT)</td>
</tr>
</table>

## Lean body mass estimation

<table>
<tr>
<td><b>Author</b></td>
<td><b>Publication</b></td>
<td><b>Formula</b></td>
</tr>
<tr>
<td>Boer (1984)</td>
<td>"Estimated lean body mass as an index for normalization of body fluid volumes in humans" in Am J Physiol. 1984 Oct;247(4 Pt 2):F632-6.</td>
<td> 
Male: (0.4071 * WEIGHT) + (0.267 * BODY_HEIGHT) - 19.2 <br>
Female: (0.252 * WEIGHT) + (0.473 * BODY_HEIGHT) - 48.3
</td>
</tr>
<tr>
<td>Hume (1966)</td>
<td>"Prediction of lean body mass from height and weight" in J Clin Pathol. 1966 Jul;19(4):389-91.</td>
<td>
Male: (0.32810 * WEIGHT) + (0.33929 * BODY_HEIGHT) - 29.5336 <br>
Female: (0.29569 * WEIGHT) + (0.41813 * BODY_HEIGHT) - 43.2933</td>
</tr>
</table>