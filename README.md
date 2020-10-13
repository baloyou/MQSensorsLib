<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->
[![DOI](https://zenodo.org/badge/170540207.svg)](https://zenodo.org/badge/latestdoi/170540207)
![Build Status][build-url]
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

# MQSensorsLib

We present a unified library for MQ sensors, this library allows to read MQ signals easily from Arduino, Genuino, ESP8266, ESP-32 boards whose references are MQ2, MQ3, MQ4, MQ5, MQ6, MQ7, MQ8, MQ9, MQ131, MQ135, MQ303A, MQ309A.

MQ3（酒精传感器） 利用二氧化锡在高温下遇到酒精气体时，电阻会急剧减小的原理制造，缺点：需要预热，稳定性较差，受环境温度、湿度等因素影响较大；每一种传感器的选择性都不是唯一的，输出参数也不能确定，不宜应用于计量准确要求的场所。

以PPM的输出为标准，PPM应该是 Parts per million的缩写，与酒精测试仪的兑换方法：

清洁空气 = 0.04/L

此时电压 = 0.5v，用 0.4/0.5 = 0.8

用 0.8 * 酒精电压 = 结果

我使用的几元钱的传感器，数据

```
规格参数
1. 探测范围：10~1000ppm酒精
2. 特征气体：125ppm酒精
3. 加热电压：5±0.2V（AC·DC）
4. 回路电压：≤24V    DC
5. 负载电阻：可调
6. 清洁空气中电压：≤1.5V 
7. 灵敏度：Rin air/Rin typical gas≥5
8. 响应时间：≤10S
9. 恢复时间：≤30S
10.元件功耗：≤0.9W
11.主要芯片：LM393、MQ-3气体传感器
12.使用寿命：5年

电气性能

主要芯片：LM393、MQ-3气体传感器

输入电压：DC5V 功耗（电流）：150mA

DO输出：TTL数字量0和1（0.1和5V）

AO输出：0.1-0.3V（相对无污染），最高浓度电压4V左右

特别提醒：传感器通电后，需要预热20S左右，测量的数据才稳定，传感器发热属于正常现象，因为内部有电热丝，如果烫手就不正常了。
```

我们提供一个统一的MQ系列传感器库，这个库允许简单的从arduino,genuino,esp8266,esp-32开发板上简单的读取mq传感器信号。包括：MQ2, MQ3, MQ4, MQ5, MQ6, MQ7, MQ8, MQ9, MQ131, MQ135, MQ303A, MQ309A.

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [Getting Started](#Getting-Started)
* [Wiring](#Wiring)
  * [Sensor](#Sensor)
  * [Arduino](#Arduino)
  * [ESP8266 or ESP-32](#ESP8266-ESP32)
* [User Manual](#Manuals)
* [Sensor manufacturers](#Sensor-manufacturers)
* [Contributing](#Contributing)
* [Authors](#Authors)

## Getting Started

```
//Include the library
#include <MQUnifiedsensor.h>
/************************Hardware Related Macros************************************/
#define         Board                   ("Arduino UNO")
#define         Pin                     (A4)  //Analog input 4 of your arduino
/***********************Software Related Macros************************************/
#define         Type                    ("MQ-4") //MQ4
#define         Voltage_Resolution      (5)
#define         ADC_Bit_Resolution      (10) // For arduino UNO/MEGA/NANO
#define         RatioMQ4CleanAir        (4.4) //RS / R0 = 60 ppm 
/*****************************Globals***********************************************/
//Declare Sensor
MQUnifiedsensor MQ4(Board, Voltage_Resolution, ADC_Bit_Resolution, Pin, Type);
// Setup
MQ4.setRegressionMethod("Exponential"); //_PPM =  a*ratio^b
MQ4.setA(1012.7); MQ4.setB(-2.786); // Configurate the ecuation values to get CH4 concentration
MQ4.setR0(3.86018237); // Value getted on calibration
// Loop
MQ4.init();
MQ4.update();
float ppmCH4 = MQ4.readSensor();
```

## Wiring
### Sensor
#### Important points:
##### Points you should identify （需要注意的几点）
* VCC -> 5V Power supply (+) wire
* GND -> GND Ground (-) wire
* AO -> Analog Output of the sensor
##### Data of board that you should have （你需要知道开发板中）
* RL Value in KOhms （阻抗中的RL值）
##### Graph
![Wiring_MQSensor](https://raw.githubusercontent.com/miguel5612/MQSensorsLib_Docs/master/static/img/Points_explanation.jpeg)
#### RS/R0 value (From datasheet of your sensor 来自传感器数据表)

RS = 传感器在不同浓度气体中的电阻值

R0 = 传感器在洁净传感器中的电阻值

RS/R0 = 计算电阻比

* RS/R0 (Clean air - English) -> (Aire puro - Spanish)
* **Note**: RS/R0 is equal to Ratio variable on the program 等于程序上的比率变量
![Graph from datasheet](https://raw.githubusercontent.com/miguel5612/MQSensorsLib_Docs/master/static/img/Graph_Explanation.jpeg)
### Arduino
![Arduino_Wiring_MQSensor](https://raw.githubusercontent.com/miguel5612/MQSensorsLib_Docs/master/static/img/MQ_Arduino.PNG)
#### MQ-7 / MQ-309A
** Note ** [issue](https://github.com/miguel5612/MQSensorsLib/issues/26): MQ-7 and MQ-309 needs two different voltages for heater, they can be supplied by PWM and DC Signal controlled by your controller, another option is to use two different power sources, you should use the best option for you, next i will show the PWM option and on the examples this will be the way .
![MQ-7_MQ-309](https://raw.githubusercontent.com/miguel5612/MQSensorsLib_Docs/master/static/img/MQ-309_MQ-7.PNG)
### ESP8266-ESP32
![ESP8266_Wiring_MQSensor](https://raw.githubusercontent.com/miguel5612/MQSensorsLib_Docs/master/static/img/MQ_ESP8266.PNG)

### ESP32 WROOM 32D
The ESP32 WROOM 32D does not need an external power supply. A0 goes to PIN36, Vcc to 3v3 and GND to any GND port on the board. Check the **ESP2/ESP32_WROOM_32** folder to fixing the measuring issue when connecting to wifi.

### Manuals
#### User Manual (v1.0) 12.2019
[Manual](https://drive.google.com/open?id=1BAFInlvqKR7h81zETtjz4_RC2EssvFWX)
#### User Manual (v2.0) 04.2020
[Manual](https://github.com/miguel5612/MQSensorsLib_Docs/blob/master/Docs/MQSensorLib_2.0.pdf)

### Serial debug (optional)
If your sensor is an **MQ2** (Same for others sensors):
* To enable on setup wrote
```
MQ2.serialDebug(true); 
```
* And on Loop Wrote
```
MQ2.serialDebug(); 
```
* Result:

![Serial debug output](https://github.com/miguel5612/MQSensorsLib_Docs/blob/master/static/img/Serial_Mon_Explanation.jpeg?raw=true)

**Note**: 
* ![#c5f015](https://placehold.it/15/c5f015/000000?text=+) `Yellow -> Calibration status.`
* ![#008000](https://placehold.it/15/008000/000000?text=+) `Green -> Hardware and software characteristics.`
* ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) `Red -> Headers of the library calculations.`
* Only valid for **1** gas sensor readings.

**Usage**
* Quick troubleshooting, since it shows everything the library does and the results of the calculations in each function.

### Prerequisites

You'll need Arduino desktop app 1.8.9 or later.

### Sensor manufacturers:
| Sensor | Manufacture | URL Datasheet |
|----------|----------|----------|
| MQ-2 | Pololulu| [datasheet](https://www.pololu.com/file/0J309/MQ2.pdf) |
| MQ-3 | Sparkfun | [datasheet](https://www.sparkfun.com/datasheets/Sensors/MQ-3.pdf) |
| MQ-4 | Sparkfun | [datasheet](https://www.sparkfun.com/datasheets/Sensors/Biometric/MQ-4.pdf) |
| MQ-5 | parallax | [datasheet](https://www.parallax.com/sites/default/files/downloads/605-00009-MQ-5-Datasheet.pdf) |
| MQ-6 | Sparkfun | [datasheet](https://www.sparkfun.com/datasheets/Sensors/Biometric/MQ-6.pdf) |
| MQ-7 | Sparkfun | [datasheet](https://www.sparkfun.com/datasheets/Sensors/Biometric/MQ-7.pdf) |
| MQ-8 | Sparkfun | [datasheet](https://dlnmh9ip6v2uc.cloudfront.net/datasheets/Sensors/Biometric/MQ-8.pdf) |
| MQ-9 | Haoyuelectronics | [datasheet](http://www.haoyuelectronics.com/Attachment/MQ-9/MQ9.pdf) |
| MQ-131 | Sensorsportal | [datasheet](http://www.sensorsportal.com/DOWNLOADS/MQ131.pdf) |
| MQ-135 | HANWEI Electronics | [datasheet](https://www.electronicoscaldas.com/datasheet/MQ-135_Hanwei.pdf) |
| MQ-303A | HANWEI Electronics | [datasheet](http://www.kosmodrom.com.ua/pdf/MQ303A.pdf) |
| MQ-309A | HANWEI Electronics | [datasheet](http://www.sensorica.ru/pdf/MQ-309A.pdf) |

### Info of datasheets 

Review WPDigitalizer [folder](https://github.com/miguel5612/MQSensorsLib_Docs/tree/master/WPDigitalizer) [website](https://automeris.io/WebPlotDigitizer/)

### Installing

Clone this repository into your desktop machine

```
git clone https://github.com/miguel5612/MQSensorsLib
```


## Running the tests

Use calibration systems if you have several sensors that read the same gas.

### Break down into end to end tests

These tests can re-adjust values defined previously and you can contribute to improve conditions or features obtained from particular scenes.

```
Examples/MQ-3
```

### And coding style tests

These tests may generate statistics validation using descriptive tools for quantitative variables.

```
Examples/MQ-board.ino
```

## Built With

* [Data sheets](https://github.com/miguel5612/MQSensorsLib_Docs/tree/master/Datasheets) - Curves and behavior for each sensor, using logarithmic graphs.
* [Main purpose](https://github.com/miguel5612/MQSensorsLib_Docs/blob/master/static/img/bg.jpg) - Every sensor has high sensibility for a specific gas or material.

## Contributing

Please read [CONTRIBUTING.md](https://github.com/miguel5612/MQSensorsLib/blob/master/CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Reviewers

* **PhD. Jacipt A Ramón V.** - [*GitHub*]() - [CV](https://scienti.minciencias.gov.co/cvlac/visualizador/generarCurriculoCv.do?cod_rh=0000512702)

## Authors

* **Miguel A. Califa U.** - [*GitHub*](https://github.com/miguel5612) - [CV](https://scienti.colciencias.gov.co/cvlac/visualizador/generarCurriculoCv.do?cod_rh=0000050477)
* **Ghiordy F. Contreras C.** - [*GitHub*](https://github.com/Ghiordy) - [CV](https://scienti.colciencias.gov.co/cvlac/visualizador/generarCurriculoCv.do?cod_rh=0000050476) 
* **Yersson R. Carrillo A.** - [*GitHub*](https://github.com/Yercar18/Dronefenix)  - [CV](https://scienti.colciencias.gov.co/cvlac/visualizador/generarCurriculoCv.do?cod_rh=0001637655)

## Collaborators

* **Andres A. Martinez.** - [*Github*](https://github.com/andresmacsi) - [CV](https://www.linkedin.com/in/andr%C3%A9s-acevedo-mart%C3%ADnez-73ab35185/?originalSubdomain=co)
* **Juan A. Rodríguez.** - [*Github*](https://github.com/Obiot24) - [CV]()
* **Mario A. Rodríguez O.** - [*GitHub*](https://github.com/MarioAndresR) - [CV](https://scienti.colciencias.gov.co/cvlac/visualizador/generarCurriculoCv.do?cod_rh=0000111304)

See also the list of [contributors](https://github.com/miguel5612/MQSensorsLib/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Cite as

* Plain text: Califa Urquiza, Miguel Angel, Contreras Contreras, Ghiordy, & Carrillo Amado, Yerson Ramiro. (2019, September 3). miguel5612/MQSensorsLib: Arduino Preview V1.03 (Version 1.0.3). Zenodo. http://doi.org/10.5281/zenodo.3384301
* CSL: {
  "publisher": "Zenodo", 
  "DOI": "10.5281/zenodo.3384301", 
  "title": "miguel5612/MQSensorsLib: Arduino Preview V1.03", 
  "issued": {
    "date-parts": [
      [
        2019, 
        9, 
        3
      ]
    ]
  }, 
  "abstract": "<p>Publishing on Zenodo platform as software in order to extend its applications for other works allowing to recognize MQSensorLib&#39;s Authors this work into scientific community using Digital Object Identifier System (DOI).</p>", 
  "author": [
    {
      "family": "Califa Urquiza, Miguel Angel"
    }, 
    {
      "family": "Contreras Contreras, Ghiordy"
    }, 
    {
      "family": "Carrillo Amado, Yerson Ramiro"
    }
  ], 
  "version": "1.0.3", 
  "type": "article", 
  "id": "3384301"
}
* BibTeX: 
@misc{califa_urquiza_miguel_angel_2019_3384301,
  author       = {Califa Urquiza, Miguel Angel and
                  Contreras Contreras, Ghiordy and
                  Carrillo Amado, Yerson Ramiro},
  title        = {miguel5612/MQSensorsLib: Arduino Preview V1.03},
  month        = sep,
  year         = 2019,
  doi          = {10.5281/zenodo.3384301},
  url          = {https://doi.org/10.5281/zenodo.3384301}
}



<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/miguel5612/MQSensorsLib.svg?style=flat-square
[contributors-url]: https://github.com/miguel5612/MQSensorsLib/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/miguel5612/MQSensorsLib.svg?style=flat-square
[forks-url]: https://github.com/miguel5612/MQSensorsLib/network/members
[stars-shield]: https://img.shields.io/github/stars/miguel5612/MQSensorsLib.svg?style=flat-square
[stars-url]: https://github.com/miguel5612/MQSensorsLib/stargazers
[issues-shield]: https://img.shields.io/github/issues/miguel5612/MQSensorsLib.svg?style=flat-square
[issues-url]: https://github.com/miguel5612/MQSensorsLib/issues
[license-shield]: https://img.shields.io/github/license/miguel5612/MQSensorsLib.svg?style=flat-square
[license-url]: https://github.com/miguel5612/MQSensorsLib/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=flat-square&logo=linkedin&colorB=555
[build-url]: https://travis-ci.org/dwyl/esta.svg?branch=master
[linkedin-url]: https://www.linkedin.com/in/miguel5612
[product-screenshot]: images/screenshot.png
