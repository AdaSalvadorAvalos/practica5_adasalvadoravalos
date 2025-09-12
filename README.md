# Practice 5: I2C OLED Display with HTU21D Sensor on ESP32 (English Version)

## Materials
- ESP32  
- OLED Display (I2C)  
- KYYKA HTU21D I2C - Humidity & Temperature Sensor  

## Introduction
In this practice for the Digital Processors course, we explore the **I2C bus**.  
We see that it is simpler since it only requires **two wires**: one for the clock signal (SCL) and one for data transmission (SDA).  
It is synchronous and uses a master–slave architecture.  
We connect an OLED display to show the **current humidity and temperature** from the HTU21D sensor.  

## Code Explanation (with line-by-line comments)
```cpp
#include <Arduino.h>

void init_temp_hum_task(void);

// Include the correct display library for I2C
#include <Wire.h>              // Only required for Arduino 1.6.5 and earlier
#include "SSD1306Wire.h"       

#include "images.h"            // Custom XBM image file
#include "SparkFunHTU21D.h"    // HTU21D sensor library

HTU21D myHumidity;             // Create an HTU21D sensor object

// Initialize the OLED display using I2C (Wire)
SSD1306Wire display(0x3c, SDA, SCL);   // Address, SDA pin, SCL pin

#define DEMO_DURATION 3000
typedef void (*Demo)(void);

int demoMode = 0;
int counter = 1;

void setup() {
  Serial.begin(115200);        // Start serial communication
  Serial.println();
  Serial.println();

  Serial.println("HTU21D Example!");

  myHumidity.begin();          // Initialize the sensor

  display.init();              // Initialize OLED display
  display.flipScreenVertically(); // Flip screen vertically
  display.setFont(ArialMT_Plain_10); // Set default font
}

// Demo: display text in different font sizes
void drawFontFaceDemo() { 
  display.setTextAlignment(TEXT_ALIGN_LEFT);
  display.setFont(ArialMT_Plain_10);
  display.drawString(0, 0, "Hello world");
  display.setFont(ArialMT_Plain_16);
  display.drawString(0, 10, "Hello world");
  display.setFont(ArialMT_Plain_24);
  display.drawString(0, 26, "Hello world");
}

// Demo: text flowing and max width
void drawTextFlowDemo() {
  display.setFont(ArialMT_Plain_10);
  display.setTextAlignment(TEXT_ALIGN_LEFT);
  display.drawStringMaxWidth(0, 0, 128,
    "Lorem ipsum\n dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore.");
}

// Demo: text alignment (left, center, right)
void drawTextAlignmentDemo() {
  display.setFont(ArialMT_Plain_10);

  display.setTextAlignment(TEXT_ALIGN_LEFT);
  display.drawString(0, 10, "Left aligned (0,10)");

  display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.drawString(64, 22, "Center aligned (64,22)");

  display.setTextAlignment(TEXT_ALIGN_RIGHT);
  display.drawString(128, 33, "Right aligned (128,33)");
}

// Demo: rectangles and pixels
void drawRectDemo() {
  for (int i = 0; i < 10; i++) {
    display.setPixel(i, i);        // Draw pixel diagonal
    display.setPixel(10 - i, i);   // Draw mirrored pixel
  }
  display.drawRect(12, 12, 20, 20);   // Draw rectangle outline
  display.fillRect(14, 14, 17, 17);   // Draw filled rectangle
  display.drawHorizontalLine(0, 40, 20); // Horizontal line
  display.drawVerticalLine(40, 0, 20);   // Vertical line
}

// Demo: circles
void drawCircleDemo() {
  for (int i = 1; i < 8; i++) {
    display.setColor(WHITE); 
    display.drawCircle(32, 32, i * 3);   // Draw circle at (32,32) with increasing radius
    if (i % 2 == 0) {
      display.setColor(BLACK);           // Alternate colors
    }
    display.fillCircle(96, 32, 32 - i * 3); // Fill circle at (96,32)
  }
}

// Demo: progress bar
void drawProgressBarDemo() {
  int progress = (counter / 5) % 100;
  display.drawProgressBar(0, 32, 120, 10, progress); // Draw bar
  display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.drawString(64, 15, String(progress) + "%"); // Draw percentage
}

// Demo: display XBM image
void drawImageDemo() {
  display.drawXbm(34, 14, WiFi_Logo_width, WiFi_Logo_height, WiFi_Logo_bits);
}

// Array of demo functions
Demo demos[] = {drawFontFaceDemo, drawTextFlowDemo, drawTextAlignmentDemo, drawRectDemo, drawCircleDemo, drawProgressBarDemo, drawImageDemo};
int demoLength = (sizeof(demos) / sizeof(Demo));
long timeSinceLastModeSwitch = 0;

void loop() {
  float humd = myHumidity.readHumidity();      // Read humidity
  float temp = myHumidity.readTemperature();   // Read temperature

  // Print readings to serial monitor
  Serial.print("Time:");
  Serial.print(millis());
  Serial.print(" Temperature:");
  Serial.print(temp, 1);
  Serial.print("C");
  Serial.print(" Humidity:");
  Serial.print(humd, 1);
  Serial.print("%");
  Serial.println();

  display.clear();                             // Clear display

  display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.setFont(ArialMT_Plain_10);
  display.drawString(128/2, 0, "HUMIDITY");
  display.setFont(ArialMT_Plain_16);
  display.drawString(128/2, 11, String(humd)+ "%");

  display.setFont(ArialMT_Plain_10);
  display.drawString(128/2, 30, "TEMPERATURE");
  display.setFont(ArialMT_Plain_16);
  display.drawString(128/2, 41, String(temp)+ "ºC");

  display.setFont(ArialMT_Plain_10);
  display.setTextAlignment(TEXT_ALIGN_RIGHT);
  display.drawString(128, 54, String(millis()/3600000)+":"+String((millis()/60000)%60)+":"+String((millis()/1000)%(60));

  display.display(); // Write buffer to display

  delay(100); // Wait 100ms before next update
}

```

### How to Run

1. **Install PlatformIO**
   - Install [VS Code](https://code.visualstudio.com/)
   - Install the [PlatformIO extension](https://platformio.org/install/ide?install=vscode)

2. **Create a New Project**
   - Open PlatformIO in VS Code
   - Create a new project and select your board (**ESP32 Dev Module**)
   - Choose **Arduino framework**

3. **Install Required Libraries**
   - Install in PlatformIO:
     - `SparkFun HTU21D Humidity and Temperature Sensor`
     - `ESP8266 and ESP32 OLED driver for SSD1306 displays`

4. **Add the Code**
   - Replace the contents of `src/main.cpp` with the code provided

5. **Connect the Hardware**
   - HTU21D sensor → I2C:
     - SDA → GPIO 21
     - SCL → GPIO 22
   - OLED Display → Same I2C pins (SDA, SCL)
   - Power both with 3.3V and GND

6. **Build and Upload**
   - Click **Build (✓)** to compile the code
   - Click **Upload (→)** to flash the ESP32

7. **Observe the Result**
   - OLED will show **Humidity (%)**, **Temperature (ºC)**, and elapsed time
   - Serial Monitor will also log the readings

### Resources  
- **Video Demonstration:** [Watch video](assets/practica5.mp4)  
- **OLED Library GitHub:** [ESP8266 and ESP32 OLED driver for SSD1306 displays](https://github.com/ThingPulse/esp8266-oled-ssd1306)  
- **Temperature Sensor Library:** [SparkFun HTU21D](https://github.com/sparkfun/SparkFun_HTU21D_Arduino_Library)  
- **PlatformIO Dependencies (in `platformio.ini`):**  
  ```ini
  lib_deps = 
    sparkfun/SparkFun HTU21D Humidity and Temperature Sensor Breakout@^1.1.3
    thingpulse/ESP8266 and ESP32 OLED driver for SSD1306 displays@^4.2.1
    ;jmparatte/jm_LCM2004A_I2C@^1.0.2
  ```

# Práctica 5: Pantalla OLED I2C con Sensor HTU21D en ESP32 (Versión en Español)

## Materiales
- ESP32
- OLED Display (I2C)
- KYYKA HTU21D I2C - Módulo de sensor de humedad

## Introducción

En esta práctica del curso de Procesadores Digitales exploramos el **bus I2C**.
Vemos que es más simple porque solo requiere **dos cables**: uno para la señal de reloj (SCL) y otro para el envío de datos (SDA).
Es síncrono y tiene una arquitectura maestro–esclavo.
Conectaremos una pantalla OLED donde se mostrarán la **humedad y temperatura actuales** medidas por el sensor HTU21D.

## Explicación del código (con comentarios línea por línea):

```cpp
#include <Arduino.h>


void init_temp_hum_task(void);


// Incluir la biblioteca de visualización correcta

// Para una conexión a través de I2C utilizando Arduino Wire:
#include <Wire.h>               // solo requerido por Arduino 1.6.5 y antes
#include "SSD1306Wire.h"       

#include "images.h"

#include "SparkFunHTU21D.h"

//Crea una variable de HTU21D 
HTU21D myHumidity;



// inicializa el OLED display usando Arduino Wire:
SSD1306Wire display(0x3c, SDA, SCL);   // ADDRESS, SDA, SCL  - SDA and SCL generalmente se completan automaticamente en función de los pines de la placa

#define DEMO_DURATION 3000
typedef void (*Demo)(void);

int demoMode = 0;
int counter = 1;

void setup() {
  Serial.begin(115200); //Establece la velocidad de datos en bits por segundo para la transmisión de datos en serie
  Serial.println();
  Serial.println();

   Serial.println("HTU21D Example!");

   myHumidity.begin(); //inicializa la variable del sensor
//}


  // inicializa el display
  display.init();

  display.flipScreenVertically(); //cambia la pantalla a vertical
  display.setFont(ArialMT_Plain_10); //y cambia la fuente de la letra
 
}

void drawFontFaceDemo() { 
  display.setTextAlignment(TEXT_ALIGN_LEFT); //función para situar el texto donde se quiere
  display.setFont(ArialMT_Plain_10);
  display.drawString(0, 0, "Hello world"); //función para escribir
  display.setFont(ArialMT_Plain_16);
  display.drawString(0, 10, "Hello world");
  display.setFont(ArialMT_Plain_24);
  display.drawString(0, 26, "Hello world");
}

void drawTextFlowDemo() {
  display.setFont(ArialMT_Plain_10);
  display.setTextAlignment(TEXT_ALIGN_LEFT);
  display.drawStringMaxWidth(0, 0, 128,
                             "Lorem ipsum\n dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore." );
//escribir un string diciendo cuanto se quiere que ocupe
}

void drawTextAlignmentDemo() { //otra función de display para alinear(en este caso)
  // Text alignment demo
  display.setFont(ArialMT_Plain_10);

  // las coordenadas definen el punto en el que empieza el texto
  display.setTextAlignment(TEXT_ALIGN_LEFT);
  display.drawString(0, 10, "Left aligned (0,10)"); //columna y fila en el que se definen

  // las coordenadas definen el punto del centro del texto
  display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.drawString(64, 22, "Center aligned (64,22)");//columna y fila en el que se definen


  // las coordenadas definen el punto del el final del texto
  display.setTextAlignment(TEXT_ALIGN_RIGHT);
  display.drawString(128, 33, "Right aligned (128,33)");//columna y fila en el que se definen

}

void drawRectDemo() {
  // dibuja un pixel en la posición indicada
  for (int i = 0; i < 10; i++) {
    display.setPixel(i, i);
    display.setPixel(10 - i, i);
  }
  display.drawRect(12, 12, 20, 20);

  // rellena un rectangulo
  display.fillRect(14, 14, 17, 17);

  // dibuja una linea horizontal
  display.drawHorizontalLine(0, 40, 20);

  // dibuja una linea vertical
  display.drawVerticalLine(40, 0, 20);
}

void drawCircleDemo() { //dibuja un circulo
  for (int i = 1; i < 8; i++) {
    display.setColor(WHITE); //color del circulo
    display.drawCircle(32, 32, i * 3); dibuja el circulo, en los ejes de coordenadas x,y, y radio de i * 3
    if (i % 2 == 0) {
      display.setColor(BLACK);
    }
    display.fillCircle(96, 32, 32 - i * 3); //rellena el circulo, en los ejes de coordenadas x,y, y radio de 32 - i * 3
  }
}

void drawProgressBarDemo() {
  int progress = (counter / 5) % 100;
  // dibuja una barra de progreso
  display.drawProgressBar(0, 32, 120, 10, progress);

  // dibuja el percentaje en un string
  display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.drawString(64, 15, String(progress) + "%");
}

void drawImageDemo() { //dibuja una fotografía
  // necesitas un fichero xbm
  display.drawXbm(34, 14, WiFi_Logo_width, WiFi_Logo_height, WiFi_Logo_bits);
}

Demo demos[] = {drawFontFaceDemo, drawTextFlowDemo, drawTextAlignmentDemo, drawRectDemo, drawCircleDemo, drawProgressBarDemo, drawImageDemo};
int demoLength = (sizeof(demos) / sizeof(Demo));
long timeSinceLastModeSwitch = 0;

void loop() {

  float humd = myHumidity.readHumidity();
  float temp = myHumidity.readTemperature();
//crea dos variables una de tipo temperatura y otra de humedad 
  Serial.print("Time:");
  Serial.print(millis());
  Serial.print(" Temperature:");
  Serial.print(temp, 1);
  Serial.print("C");
  Serial.print(" Humidity:");
  Serial.print(humd, 1);
  Serial.print("%");
  Serial.println();

//serial.print lo escribe por pantalla

  // limpiamos el display
  display.clear();

  display.setTextAlignment(TEXT_ALIGN_CENTER);
  display.setFont(ArialMT_Plain_10);
  display.drawString(128/2, 0, "HUMEDAD");
  display.setFont(ArialMT_Plain_16);
  display.drawString(128/2, 11, String(humd)+ "%");
  display.setFont(ArialMT_Plain_10);
  display.drawString(128/2, 30, "TEMPERATURA");
  display.setFont(ArialMT_Plain_16);
  display.drawString(128/2, 41, String(temp)+ "ºC");

  display.setFont(ArialMT_Plain_10);
  display.setTextAlignment(TEXT_ALIGN_RIGHT);
  display.drawString(128, 54, String(millis()/3600000)+String(":")\
          +String((millis()/60000)%60)+String(":")\
          +String((millis()/1000)%(60)));
 
  //display.setTextAlignment,   display.setFont,   display.drawString se utilizan para dibujar en el display como en las funciones anteriores he hecho 

  // escribimos el buffer al display
  display.display();

  
  
  delay(100);
}


```


### Cómo Ejecutar

1. **Instalar PlatformIO**
   - Instala [VS Code](https://code.visualstudio.com/)
   - Instala la [extensión PlatformIO](https://platformio.org/install/ide?install=vscode)

2. **Crear un Nuevo Proyecto**
   - Abre PlatformIO en VS Code
   - Crea un nuevo proyecto y selecciona tu placa (**ESP32 Dev Module**)
   - Elige **framework Arduino**

3. **Instalar Librerías Requeridas**
   - Instala en PlatformIO:
     - `SparkFun HTU21D Humidity and Temperature Sensor`
     - `ESP8266 and ESP32 OLED driver for SSD1306 displays`

4. **Agregar el Código**
   - Reemplaza el contenido de `src/main.cpp` con el código proporcionado

5. **Conectar el Hardware**
   - Sensor HTU21D → I2C:
     - SDA → GPIO 21
     - SCL → GPIO 22
   - Pantalla OLED → mismos pines I2C (SDA, SCL)
   - Alimenta ambos con 3.3V y GND

6. **Compilar y Subir**
   - Haz clic en **Build (✓)** para compilar
   - Haz clic en **Upload (→)** para cargar la ESP32

7. **Observar el Resultado**
   - La pantalla OLED mostrará **Humedad (%)**, **Temperatura (ºC)** y tiempo transcurrido
   - El Monitor Serie también mostrará las lecturas

### Recursos  
- **Video demostrativo:** [Ver video](assets/practica5.mp4)  
- **Librería OLED GitHub:** [ESP8266 and ESP32 OLED driver for SSD1306 displays](https://github.com/ThingPulse/esp8266-oled-ssd1306)  
- **Librería del sensor de temperatura:** [SparkFun HTU21D](https://github.com/sparkfun/SparkFun_HTU21D_Arduino_Library)  
- **Dependencias de PlatformIO (en `platformio.ini`):**  
  ```ini
  lib_deps = 
    sparkfun/SparkFun HTU21D Humidity and Temperature Sensor Breakout@^1.1.3
    thingpulse/ESP8266 and ESP32 OLED driver for SSD1306 displays@^4.2.1
    ;jmparatte/jm_LCM2004A_I2C@^1.0.2
  ```
