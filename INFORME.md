# Práctica 5

## Materiales:
- ESP32
- OLED Display I2C
- KYYKA HTU21D I2C - Módulo de sensor de humedad

## Presentación: 
En esta práctica observamos el bus I2C, vemos que es más simple solo se usan dos cables(uno para la señal de reloj: CLK y otro para el envío de datos: SDA )  , es síncrono y tiene una arquitectura maestro-esclavo. Conectaremos un display donde podrá la humedad y temperatura actual.

## Explicación del código (con comentarios que explican el funcionamiento línea a línea):


```
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

Salida explicada con claridad en el vídeo.
