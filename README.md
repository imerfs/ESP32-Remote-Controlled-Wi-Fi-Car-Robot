# 🤖 ESP32 Remote-Controlled Wi-Fi Car Robot

> **Arduino IDE** | **ESP32** | **L298N Motor Driver** | **Web Server**

---

## 🇬🇧 English

### 📋 Project Overview

Build a **Wi-Fi remote-controlled car robot** using the ESP32 microcontroller. The robot is controlled through a web-based interface accessible on any device within your local network. You can move the robot **forward, backward, left, right**, and **stop** it — with an adjustable speed slider.

---

### 🛒 Parts Required

| # | Component | Quantity |
|---|-----------|----------|
| 1 | 🧠 ESP32 DOIT DEVKIT V1 Board | 1x |
| 2 | 🚗 Smart Robot Chassis Kit (with 2x DC motors) | 1x |
| 3 | ⚡ L298N Motor Driver | 1x |
| 4 | 🔋 Power Bank (portable charger) | 1x |
| 5 | 🔋 AA 1.5V Batteries | 4x |
| 6 | 🔌 100nF Ceramic Capacitors | 2x |
| 7 | 🔘 SPDT Slide Switch | 1x |
| 8 | 🔗 Jumper Wires | — |
| 9 | 🧩 Breadboard or Stripboard | 1x |
| 10 | 🪄 Velcro Tape | — |

---

### ⚡ Wiring / Pin Connections

| L298N Motor Driver Pin | ESP32 GPIO |
|------------------------|------------|
| IN1 | GPIO 27 |
| IN2 | GPIO 26 |
| ENA (Enable Motor A) | GPIO 14 |
| IN3 | GPIO 33 |
| IN4 | GPIO 25 |
| ENB (Enable Motor B) | GPIO 32 |

> 💡 **Tip:** Solder a 100nF ceramic capacitor across each motor's terminals to reduce voltage spikes.

---

### 🗺️ Motor Direction Logic

| Direction | IN1 | IN2 | IN3 | IN4 |
|-----------|-----|-----|-----|-----|
| ⬆️ Forward | LOW | HIGH | LOW | HIGH |
| ⬇️ Backward | HIGH | LOW | HIGH | LOW |
| ➡️ Right | LOW | HIGH | LOW | LOW |
| ⬅️ Left | LOW | LOW | LOW | HIGH |
| 🛑 Stop | LOW | LOW | LOW | LOW |

---

### 🔌 PWM Settings

```cpp
const int freq       = 30000;
const int resolution = 8;
int dutyCycle        = 0;
```

> Speed range: duty cycle mapped from `200` (25%) to `255` (100%). Values below 200 may cause motor buzzing without movement.

---

### 💻 Arduino Code

```cpp
/*  
  Rui Santos & Sara Santos - Random Nerd Tutorials
  https://RandomNerdTutorials.com/esp32-wi-fi-car-robot-arduino/
  Permission is hereby granted, free of charge, to any person obtaining a copy
  of this software and associated documentation files.
*/

#include <WiFi.h>
#include <WebServer.h>

// ✏️ Replace with your network credentials
const char* ssid     = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";

WebServer server(80);

// Motor 1 Pins
int motor1Pin1 = 27;
int motor1Pin2 = 26;
int enable1Pin = 14;

// Motor 2 Pins
int motor2Pin1 = 33;
int motor2Pin2 = 25;
int enable2Pin = 32;

// PWM Properties
const int freq       = 30000;
const int resolution = 8;
int dutyCycle        = 0;

String valueString = String(0);

void handleRoot() {
  const char html[] PROGMEM = R"rawliteral(
  <!DOCTYPE HTML><html>
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" href="data:,">
    <style>
      html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center; }
      .button { -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none;
                user-select: none; background-color: #4CAF50; border: none; color: white;
                padding: 12px 28px; text-decoration: none; font-size: 26px;
                margin: 1px; cursor: pointer; }
      .button2 { background-color: #555555; }
    </style>
    <script>
      function moveForward()  { fetch('/forward'); }
      function moveLeft()     { fetch('/left'); }
      function stopRobot()    { fetch('/stop'); }
      function moveRight()    { fetch('/right'); }
      function moveReverse()  { fetch('/reverse'); }

      function updateMotorSpeed(pos) {
        document.getElementById('motorSpeed').innerHTML = pos;
        fetch(`/speed?value=${pos}`);
      }
    </script>
  </head>
  <body>
    <h1>ESP32 Motor Control</h1>
    <p><button class="button" onclick="moveForward()">FORWARD</button></p>
    <div style="clear: both;">
      <p>
        <button class="button" onclick="moveLeft()">LEFT</button>
        <button class="button button2" onclick="stopRobot()">STOP</button>
        <button class="button" onclick="moveRight()">RIGHT</button>
      </p>
    </div>
    <p><button class="button" onclick="moveReverse()">REVERSE</button></p>
    <p>Motor Speed: <span id="motorSpeed">0</span></p>
    <input type="range" min="0" max="100" step="25" id="motorSlider"
           oninput="updateMotorSpeed(this.value)" value="0"/>
  </body>
  </html>)rawliteral";
  server.send(200, "text/html", html);
}

void handleForward() {
  digitalWrite(motor1Pin1, LOW);  digitalWrite(motor1Pin2, HIGH);
  digitalWrite(motor2Pin1, LOW);  digitalWrite(motor2Pin2, HIGH);
  server.send(200);
}

void handleLeft() {
  digitalWrite(motor1Pin1, LOW);  digitalWrite(motor1Pin2, LOW);
  digitalWrite(motor2Pin1, LOW);  digitalWrite(motor2Pin2, HIGH);
  server.send(200);
}

void handleStop() {
  digitalWrite(motor1Pin1, LOW);  digitalWrite(motor1Pin2, LOW);
  digitalWrite(motor2Pin1, LOW);  digitalWrite(motor2Pin2, LOW);
  server.send(200);
}

void handleRight() {
  digitalWrite(motor1Pin1, LOW);  digitalWrite(motor1Pin2, HIGH);
  digitalWrite(motor2Pin1, LOW);  digitalWrite(motor2Pin2, LOW);
  server.send(200);
}

void handleReverse() {
  digitalWrite(motor1Pin1, HIGH); digitalWrite(motor1Pin2, LOW);
  digitalWrite(motor2Pin1, HIGH); digitalWrite(motor2Pin2, LOW);
  server.send(200);
}

void handleSpeed() {
  if (server.hasArg("value")) {
    valueString = server.arg("value");
    int value = valueString.toInt();
    if (value == 0) {
      ledcWrite(enable1Pin, 0);
      ledcWrite(enable2Pin, 0);
      digitalWrite(motor1Pin1, LOW); digitalWrite(motor1Pin2, LOW);
      digitalWrite(motor2Pin1, LOW); digitalWrite(motor2Pin2, LOW);
    } else {
      dutyCycle = map(value, 25, 100, 200, 255);
      ledcWrite(enable1Pin, dutyCycle);
      ledcWrite(enable2Pin, dutyCycle);
    }
  }
  server.send(200);
}

void setup() {
  Serial.begin(115200);

  pinMode(motor1Pin1, OUTPUT); pinMode(motor1Pin2, OUTPUT);
  pinMode(motor2Pin1, OUTPUT); pinMode(motor2Pin2, OUTPUT);

  ledcAttach(enable1Pin, freq, resolution);
  ledcAttach(enable2Pin, freq, resolution);
  ledcWrite(enable1Pin, 0);
  ledcWrite(enable2Pin, 0);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nWiFi connected. IP: " + WiFi.localIP().toString());

  server.on("/",        handleRoot);
  server.on("/forward", handleForward);
  server.on("/left",    handleLeft);
  server.on("/stop",    handleStop);
  server.on("/right",   handleRight);
  server.on("/reverse", handleReverse);
  server.on("/speed",   handleSpeed);

  server.begin();
}

void loop() {
  server.handleClient();
}
```

---

### 🚀 Getting Started

1. ✅ Install the **ESP32 board** in Arduino IDE
2. ✏️ Replace `REPLACE_WITH_YOUR_SSID` and `REPLACE_WITH_YOUR_PASSWORD` with your Wi-Fi credentials
3. 🔌 Wire the circuit according to the pin table above
4. 📤 Upload the code to your ESP32
5. 📺 Open Serial Monitor at **115200 baud** — note the IP address shown
6. 🔋 Disconnect from PC, power ESP32 with the power bank
7. 🌐 Open your browser and navigate to the ESP32's IP address
8. 🎮 Control your robot!

---

### ⚠️ Troubleshooting

| Problem | Solution |
|---------|----------|
| 🔄 Motors spin in wrong direction | Swap `OUT1↔OUT2` or `OUT3↔OUT4` wires |
| 😵 Motors buzz but don't move | Increase duty cycle minimum (try > 200) |
| 🌐 Can't reach web page | Make sure your phone/PC is on the same Wi-Fi network |
| ⚡ Motors don't run | Check battery pack connection and slide switch position |

---

### 📦 Libraries Used

- `WiFi.h` — built-in with ESP32 Arduino core
- `WebServer.h` — built-in with ESP32 Arduino core

---
---

## 🇮🇷 فارسی

### 📋 معرفی پروژه

ساخت یک **ربات ماشینی کنترل از راه دور Wi-Fi** با استفاده از میکروکنترلر ESP32. ربات از طریق یک رابط وب که روی هر دستگاهی در شبکه محلی قابل دسترسی است کنترل می‌شود. می‌توانید ربات را به **جلو، عقب، چپ، راست** حرکت داده و **متوقف** کنید — همچنین یک اسلایدر برای تنظیم سرعت وجود دارد.

---

### 🛒 قطعات مورد نیاز

| # | قطعه | تعداد |
|---|------|-------|
| 1 | 🧠 برد ESP32 DOIT DEVKIT V1 | ۱ عدد |
| 2 | 🚗 کیت شاسی ربات هوشمند (با ۲ موتور DC) | ۱ عدد |
| 3 | ⚡ درایور موتور L298N | ۱ عدد |
| 4 | 🔋 پاوربانک (شارژر قابل حمل) | ۱ عدد |
| 5 | 🔋 باتری AA 1.5V | ۴ عدد |
| 6 | 🔌 خازن سرامیکی 100nF | ۲ عدد |
| 7 | 🔘 سوئیچ کشویی SPDT | ۱ عدد |
| 8 | 🔗 سیم جامپر | — |
| 9 | 🧩 برد بورد یا استریپ بورد | ۱ عدد |
| 10 | 🪄 نوار ولکرو | — |

---

### ⚡ سیم‌کشی / اتصال پین‌ها

| پین درایور موتور L298N | GPIO مربوطه در ESP32 |
|------------------------|----------------------|
| IN1 | GPIO 27 |
| IN2 | GPIO 26 |
| ENA (فعال‌ساز موتور A) | GPIO 14 |
| IN3 | GPIO 33 |
| IN4 | GPIO 25 |
| ENB (فعال‌ساز موتور B) | GPIO 32 |

> 💡 **نکته:** یک خازن سرامیکی 100nF را به ترمینال‌های هر موتور لحیم کنید تا نوسانات ولتاژ کاهش یابد.

---

### 🗺️ جهت‌های حرکت موتور

| جهت | IN1 | IN2 | IN3 | IN4 |
|-----|-----|-----|-----|-----|
| ⬆️ جلو | LOW | HIGH | LOW | HIGH |
| ⬇️ عقب | HIGH | LOW | HIGH | LOW |
| ➡️ راست | LOW | HIGH | LOW | LOW |
| ⬅️ چپ | LOW | LOW | LOW | HIGH |
| 🛑 توقف | LOW | LOW | LOW | LOW |

---

### 🔌 تنظیمات PWM

```cpp
const int freq       = 30000;
const int resolution = 8;
int dutyCycle        = 0;
```

> بازه سرعت: duty cycle از `200` (25%) تا `255` (100%) تنظیم می‌شود. مقادیر زیر 200 ممکن است فقط باعث وزوز موتور شوند بدون اینکه حرکتی صورت گیرد.

---

### 💻 کد Arduino

```cpp
/*  
  Rui Santos & Sara Santos - Random Nerd Tutorials
  https://RandomNerdTutorials.com/esp32-wi-fi-car-robot-arduino/
  Permission is hereby granted, free of charge, to any person obtaining a copy
  of this software and associated documentation files.
*/

#include <WiFi.h>
#include <WebServer.h>

// ✏️ اطلاعات شبکه خود را جایگزین کنید
const char* ssid     = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";

WebServer server(80);

// پین‌های موتور ۱
int motor1Pin1 = 27;
int motor1Pin2 = 26;
int enable1Pin = 14;

// پین‌های موتور ۲
int motor2Pin1 = 33;
int motor2Pin2 = 25;
int enable2Pin = 32;

// تنظیمات PWM
const int freq       = 30000;
const int resolution = 8;
int dutyCycle        = 0;

String valueString = String(0);

void handleRoot() {
  const char html[] PROGMEM = R"rawliteral(
  <!DOCTYPE HTML><html>
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" href="data:,">
    <style>
      html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center; }
      .button { -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none;
                user-select: none; background-color: #4CAF50; border: none; color: white;
                padding: 12px 28px; text-decoration: none; font-size: 26px;
                margin: 1px; cursor: pointer; }
      .button2 { background-color: #555555; }
    </style>
    <script>
      function moveForward()  { fetch('/forward'); }
      function moveLeft()     { fetch('/left'); }
      function stopRobot()    { fetch('/stop'); }
      function moveRight()    { fetch('/right'); }
      function moveReverse()  { fetch('/reverse'); }

      function updateMotorSpeed(pos) {
        document.getElementById('motorSpeed').innerHTML = pos;
        fetch(`/speed?value=${pos}`);
      }
    </script>
  </head>
  <body>
    <h1>ESP32 Motor Control</h1>
    <p><button class="button" onclick="moveForward()">FORWARD</button></p>
    <div style="clear: both;">
      <p>
        <button class="button" onclick="moveLeft()">LEFT</button>
        <button class="button button2" onclick="stopRobot()">STOP</button>
        <button class="button" onclick="moveRight()">RIGHT</button>
      </p>
    </div>
    <p><button class="button" onclick="moveReverse()">REVERSE</button></p>
    <p>Motor Speed: <span id="motorSpeed">0</span></p>
    <input type="range" min="0" max="100" step="25" id="motorSlider"
           oninput="updateMotorSpeed(this.value)" value="0"/>
  </body>
  </html>)rawliteral";
  server.send(200, "text/html", html);
}

void handleForward() {
  digitalWrite(motor1Pin1, LOW);  digitalWrite(motor1Pin2, HIGH);
  digitalWrite(motor2Pin1, LOW);  digitalWrite(motor2Pin2, HIGH);
  server.send(200);
}

void handleLeft() {
  digitalWrite(motor1Pin1, LOW);  digitalWrite(motor1Pin2, LOW);
  digitalWrite(motor2Pin1, LOW);  digitalWrite(motor2Pin2, HIGH);
  server.send(200);
}

void handleStop() {
  digitalWrite(motor1Pin1, LOW);  digitalWrite(motor1Pin2, LOW);
  digitalWrite(motor2Pin1, LOW);  digitalWrite(motor2Pin2, LOW);
  server.send(200);
}

void handleRight() {
  digitalWrite(motor1Pin1, LOW);  digitalWrite(motor1Pin2, HIGH);
  digitalWrite(motor2Pin1, LOW);  digitalWrite(motor2Pin2, LOW);
  server.send(200);
}

void handleReverse() {
  digitalWrite(motor1Pin1, HIGH); digitalWrite(motor1Pin2, LOW);
  digitalWrite(motor2Pin1, HIGH); digitalWrite(motor2Pin2, LOW);
  server.send(200);
}

void handleSpeed() {
  if (server.hasArg("value")) {
    valueString = server.arg("value");
    int value = valueString.toInt();
    if (value == 0) {
      ledcWrite(enable1Pin, 0);
      ledcWrite(enable2Pin, 0);
      digitalWrite(motor1Pin1, LOW); digitalWrite(motor1Pin2, LOW);
      digitalWrite(motor2Pin1, LOW); digitalWrite(motor2Pin2, LOW);
    } else {
      dutyCycle = map(value, 25, 100, 200, 255);
      ledcWrite(enable1Pin, dutyCycle);
      ledcWrite(enable2Pin, dutyCycle);
    }
  }
  server.send(200);
}

void setup() {
  Serial.begin(115200);

  pinMode(motor1Pin1, OUTPUT); pinMode(motor1Pin2, OUTPUT);
  pinMode(motor2Pin1, OUTPUT); pinMode(motor2Pin2, OUTPUT);

  ledcAttach(enable1Pin, freq, resolution);
  ledcAttach(enable2Pin, freq, resolution);
  ledcWrite(enable1Pin, 0);
  ledcWrite(enable2Pin, 0);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nWiFi connected. IP: " + WiFi.localIP().toString());

  server.on("/",        handleRoot);
  server.on("/forward", handleForward);
  server.on("/left",    handleLeft);
  server.on("/stop",    handleStop);
  server.on("/right",   handleRight);
  server.on("/reverse", handleReverse);
  server.on("/speed",   handleSpeed);

  server.begin();
}

void loop() {
  server.handleClient();
}
```

---

### 🚀 مراحل راه‌اندازی

1. ✅ نصب **برد ESP32** در Arduino IDE
2. ✏️ جایگزین کردن `REPLACE_WITH_YOUR_SSID` و `REPLACE_WITH_YOUR_PASSWORD` با اطلاعات Wi-Fi خود
3. 🔌 سیم‌کشی مدار طبق جدول پین‌ها
4. 📤 آپلود کد روی ESP32
5. 📺 باز کردن Serial Monitor با **baud rate 115200** — آدرس IP نمایش داده‌شده را یادداشت کنید
6. 🔋 جدا کردن از کامپیوتر و تغذیه ESP32 با پاوربانک
7. 🌐 باز کردن مرورگر و رفتن به آدرس IP نمایش‌داده‌شده
8. 🎮 کنترل ربات!

---

### ⚠️ عیب‌یابی

| مشکل | راه‌حل |
|------|--------|
| 🔄 موتورها در جهت اشتباه می‌چرخند | سیم‌های `OUT1↔OUT2` یا `OUT3↔OUT4` را جابجا کنید |
| 😵 موتورها وزوز می‌کنند اما حرکت نمی‌کنند | حداقل duty cycle را افزایش دهید (بیشتر از 200 امتحان کنید) |
| 🌐 نمی‌توان به صفحه وب دسترسی داشت | مطمئن شوید گوشی/لپ‌تاپ شما در همان شبکه Wi-Fi است |
| ⚡ موتورها کار نمی‌کنند | اتصال باتری و وضعیت سوئیچ کشویی را بررسی کنید |

---

### 📦 کتابخانه‌های استفاده‌شده

- `WiFi.h` — به‌صورت پیش‌فرض در هسته Arduino برای ESP32 موجود است
- `WebServer.h` — به‌صورت پیش‌فرض در هسته Arduino برای ESP32 موجود است

---

### 🔗 منبع اصلی

[Random Nerd Tutorials — ESP32 Wi-Fi Car Robot](https://RandomNerdTutorials.com/esp32-wi-fi-car-robot-arduino/)

---

*Made with ❤️ using ESP32 + Arduino IDE*
