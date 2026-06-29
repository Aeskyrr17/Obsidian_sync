RMUC最后采用G474板子，利用ADC读取左右两路力传感器

### RMUC26赛场标定
R
0.031 32871.1521
1.032 33090.6664
1.736 33250.28589814304
2.736 33486.99752586595
```
Calibration points used:
  force=    0.0310 kg, adc=  32871.1521, fit=    0.0503 kg, error=+0.0193 kg
  force=    1.0320 kg, adc=  33090.6664, fit=    1.0144 kg, error=-0.0176 kg
  force=    1.7360 kg, adc=  33250.2859, fit=    1.7154 kg, error=-0.0206 kg
  force=    2.7360 kg, adc=  33486.9975, fit=    2.7550 kg, error=+0.0190 kg

Result:
  force_kg = k * adc + b
  k = 0.0043918528f
  b = -144.3149817305f

Paste into main.cpp:
#define FORCE_SENSOR_L_LINEAR_K 0.0043918528f
#define FORCE_SENSOR_L_LINEAR_B -144.3149817305f

Fit quality:
  RMSE          = 0.019161 kg
  Max abs error = 0.020619 kg
  R^2           = 0.99962404
```
L
0.031 32858.8075
1.032 33088.2932
1.736 33320.38720362622
2.736 33557.035187580856
```
Calibration points used:
  force=    0.0310 kg, adc=  32858.8075, fit=    0.0673 kg, error=+0.0363 kg
  force=    1.0320 kg, adc=  33088.2932, fit=    0.9371 kg, error=-0.0949 kg
  force=    1.7360 kg, adc=  33320.3872, fit=    1.8168 kg, error=+0.0808 kg
  force=    2.7360 kg, adc=  33557.0352, fit=    2.7138 kg, error=-0.0222 kg

Result:
  force_kg = k * adc + b
  k = 0.0037902456f
  b = -124.4756404968f

Paste into main.cpp:
#define FORCE_SENSOR_L_LINEAR_K 0.0037902456f
#define FORCE_SENSOR_L_LINEAR_B -124.4756404968f

Fit quality:
  RMSE          = 0.065852 kg
  Max abs error = 0.094883 kg
  R^2           = 0.99555953
```