# 💡 ESP32 HTTP LED Control

A simple IoT application using an **ESP32** to control an LED remotely through an **HTTP GET request**.

The ESP32 reads a value from a remote text file hosted on GitHub:

- `1` → LED ON 💡
- `0` → LED OFF ⚫

---

## 🎯 Project Objective

The objective of this task is to demonstrate how an ESP32 can retrieve data from the web using an **HTTP GET request**, store the received response in a variable, and use it to control a physical output.

In this project, the HTTP response is stored in the `payload` variable and used to control an LED.

---

## 🔧 Components

- WEMOS D1 Mini ESP32
- LED
- 220Ω or 330Ω resistor
- Breadboard
- Jumper wires
- USB cable

---

## 🔌 Wiring

| ESP32 | Connection |
|---|---|
| GPIO 26 | Resistor → LED Anode (+) |
| GND | LED Cathode (-) |

```text
GPIO 26
   │
Resistor
   │
 LED (+)
 LED (-)
   │
  GND
```

### Hardware Setup

<img width="600" alt="ESP32 LED Setup" src="https://github.com/user-attachments/assets/d3eb664a-323c-47d2-9c37-2f8a16f5fbe4" />

<img width="600" alt="ESP32 LED Wiring" src="https://github.com/user-attachments/assets/f5565b9d-692b-496d-94bf-005960ad0c09" />

---

## 🌐 How It Works

A remote file named `t.text` is stored in the same GitHub repository.

The file contains either:

```text
1
```

or:

```text
0
```

The ESP32 sends an **HTTP GET request** to the GitHub Raw URL of this file.

```text
GitHub Raw File
       ↓
    HTTP GET
       ↓
     ESP32
       ↓
http.getString()
       ↓
    payload
       ↓
 ┌─────────────┐
 │             │
"1"           "0"
 │             │
LED ON       LED OFF
```

The HTTP response is stored inside the `payload` variable:

```cpp
String payload = http.getString();
```

Then:

```cpp
payload.trim();
```

removes unnecessary spaces or newline characters.

Finally, an `if` condition checks the received value:

```cpp
if (payload == "1") {
  digitalWrite(LED_PIN, HIGH);
}
else if (payload == "0") {
  digitalWrite(LED_PIN, LOW);
}
```

---

## 📄 Remote Control File

The remote control file used in this project is `t.text`.

### 💡 LED ON

```text
1
```

### ⚫ LED OFF

```text
0
```

The ESP32 checks the file every **5 seconds**, allowing the LED state to change remotely after updating the value on GitHub.

---

## 💻 ESP32 Code

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <WiFiMulti.h>
#include <HTTPClient.h>

WiFiMulti wifiMulti;

#define LED_PIN 26

void setup() {
  Serial.begin(115200);

  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);

  wifiMulti.addAP("YOUR_WIFI_SSID", "YOUR_WIFI_PASSWORD");
}

void loop() {

  if (wifiMulti.run() == WL_CONNECTED) {

    HTTPClient http;

    Serial.print("[HTTP] begin...\n");

    http.begin(
      "https://raw.githubusercontent.com/Dooni0harbi/ESP32_HTTP_LED_Control_W6_T1/main/t.text"
    );

    Serial.print("[HTTP] GET...\n");

    int httpCode = http.GET();

    if (httpCode > 0) {

      Serial.printf("[HTTP] GET... code: %d\n", httpCode);

      if (httpCode == HTTP_CODE_OK) {

        String payload = http.getString();
        payload.trim();

        Serial.print("Received value: ");
        Serial.println(payload);

        if (payload == "1") {
          digitalWrite(LED_PIN, HIGH);
          Serial.println("LED ON");
        }
        else if (payload == "0") {
          digitalWrite(LED_PIN, LOW);
          Serial.println("LED OFF");
        }
      }

    } else {

      Serial.printf(
        "[HTTP] GET... failed, error: %s\n",
        http.errorToString(httpCode).c_str()
      );
    }

    http.end();
  }

  delay(5000);
}
```

> **Note:** Wi-Fi credentials are intentionally excluded from this repository. Replace `YOUR_WIFI_SSID` and `YOUR_WIFI_PASSWORD` locally before uploading the code to the ESP32.

---

## ⚠️ Challenge: Web Hosting

Initially, I created a remote text file on a **free web hosting service** to provide the ESP32 with the `1` or `0` value.

However, when the ESP32 sent an HTTP request to the hosted file, the server did not return the expected value.

Instead, the response contained an HTML page with a **JavaScript/AES browser verification challenge**.

The Serial Monitor returned content similar to:

```html
<script type="text/javascript" src="/aes.js"></script>
This site requires Javascript to work.
```

Because the ESP32 `HTTPClient` does not operate as a full web browser and does not execute the required JavaScript verification, the expected `1` or `0` value could not be retrieved directly.

---

## 💡 Solution

Instead of using the original web hosting service, I hosted the control file directly inside the project's **GitHub repository**.

The `t.text` file can be accessed through its **GitHub Raw URL**, which provides the raw file content directly without the additional HTML or JavaScript verification page.

The ESP32 can therefore successfully receive `1` or `0` and store the received value in the `payload` variable.

---

## 🖥️ Serial Monitor Result

When `t.text` contains `1`, the Serial Monitor displays:

```text
[HTTP] begin...
[HTTP] GET...
[HTTP] GET... code: 200
Received value: 1
LED ON
```

The HTTP status code `200` confirms that the request was successfully completed.

When the remote value is changed to `0`, the ESP32 receives the updated value and turns the LED OFF.

---

## 🎥 Demo Video

The demo demonstrates the complete HTTP-based LED control process:

- Remote value `1` → 💡 **LED ON**
- Remote value `0` → ⚫ **LED OFF**

The ESP32 periodically retrieves the updated value through an HTTP GET request and changes the LED state accordingly.

### ▶️ Demo

https://github.com/user-attachments/assets/b2a99bf7-05ec-492d-8814-b1deb1be48be

---

## ✅ Result

The project successfully demonstrates:

- Connecting the ESP32 to Wi-Fi
- Sending HTTP GET requests
- Retrieving remote data from the web
- Using `http.getString()` to read the HTTP response
- Storing the received value in a `payload` variable
- Using conditional statements with remote data
- Controlling an ESP32 GPIO output
- Turning an LED ON/OFF remotely using `1` and `0`
- Handling a web-hosting compatibility issue by using GitHub Raw content

---

##  Internship Task

This project was developed as a practical task for the **AI & Web Track** during my **Robotics Engineering Internship at Smart Methods (الأساليب الذكية)**.

It represents a simple practical application of connecting **web-based data with embedded hardware**, where an ESP32 retrieves remote data through HTTP and uses it to control a physical electronic component.


