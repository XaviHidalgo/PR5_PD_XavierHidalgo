# PR5_PD_XavierHidalgo

## PRACTICA 5:  Buses de comunicación I (introducción y I2c)

### Ejercicio Practico 1 ESCÁNER I2C

**PROGRAMA:**

``` cpp
#include <Arduino.h>
#include <Wire.h>

void setup()
{
Wire.begin();
Serial.begin(115200);
while (!Serial){
// Leonardo: wait for serial monitor
Serial.println("\nI2C Scanner");
}
}
void loop()
{
byte error, address;
int nDevices;
Serial.println("Scanning...");
nDevices = 0;
for(address = 1; address < 127; address++ )
{
// The i2c_scanner uses the return value of
// the Write.endTransmisstion to see if
// a device did acknowledge to the address.
Wire.beginTransmission(address);
error = Wire.endTransmission();
if (error == 0)
{
nDvices++;
Serial.print("I2C device found at address 0x");
if (address<16) {
Serial.print("0");
Serial.print(address,HEX);
Serial.println(" !");
}
}
}
}
```
**INFORME:**

El anterior programa es un escáner o programa de reconocimiento de periféricos conectados a un bus I2C. 

Para saber la cantidad de periféricos connectadosa nuestro bus (objetivo de un escáner), enviaremos a las 128 direcciones posibles (ya que el bus I2C guarda las direcciones de los dispositivos esclavos en 7 bits) una señal buscando otra señal de respuesta. Si es así, sabremos que hay un dispositivo connectado al bus y además su dirección para comunicarnos con él. Debido a las resistencias de pull-up que incluye el bus I2C, el circuito se mantiene en alta impedancia continuamente así que, si acertamos la dirección de un periférico, lo sabremos ya que como respuesta el periférico variará esta alta impedancia. Esta variación, en el programa la entenderemos como un '0'.

A continuación explico el funcionamiento del programa y la salida que tenemos por el pueto série tras su ejecucción:

Si nos fijamos en el inicio del código, vemos que, como casi siempre hacemos incluimos la libreria Arduino.h (ya que estamos programando con un esquema tipo Arduino y es posible que nos evite problemas) y, como nueva incorporación, incluimos Wire.h. Esta última libreria nos permitira comunicarnos sobre el bus I2C, permitiendonos iniciar y acabar transmisiones en distintas direcciones.

Más adelante, en el void_setup(), inicializamos el bus série (como siempre) y el bus I2C a partir de la instancia "Wire.begin;". Después nos encontramos con un while: "while (!Serial)". Como vemos, se ejecutará cuando el bus série este listo, tras eso escribe por él "\nI2C Scanner", diciéndonos que el bus série esta operativo y que se inicia el escáner.

EL escáner se inicia en el void_loop(). Como he escrito al principio, necesitamos enviar por el bus todas las direcciones de dispositivo y ver si conseguimos respuesta de algún periférico tras ello. Esto lo haremos con un bucle for. Para que funcione el bucle necesitamos declarar una serie de variables, declararemos: nDevices (tipo int, define la cantidad de periféricos connectados), error (tipo byte, nos indicará si ha encontrado en x dirección un dispositivo) y address (tipo byte, aqui guardaremos las direcciones). El bucle recorrerá acabará cuando haya comprobado todas las direcciones, es decir, empieza comprobando la dirección 00000000 y va sumando +1 en binario hasta llegar a la última, la **0**1111111 (el primer 0 se debe a que el tipo de variable byte son lógicamente 8 bits y las direcciones, como he escrito antes, son de 7).

Bien, enviaremos cada dirección por el bus I2C gracias a la función .beginTransmission() de Wire.h de la siguiente forma:
``` cpp
Wire.beginTransmission(address);
```
Tras iniciarla, la cerraremos con la función .endTransmission() de la misma libreria la cual nos devuelve un valor tipo byte que lo guardaremos en nuestra variable error:
``` cpp
error = Wire.endTransmission();
```
error valdrá 0 si la conexión ha sido satisfactoria, como he comentado en el segundo párrafo, la variación de alta impedancia y, por lo tanto una respuesta por parte del periférico, la entendemos como un 0. Entonces, tras tener un valor en error, con un if() preguntamos si error vale 0 para, si es así, enviar por el puerto série la dirección del dispositivo encontrado en **hexadecimal**:
``` cpp
if (error == 0)
{
nDvices++;
Serial.print("I2C device found at address 0x");
if (address<16) {
Serial.print("0");
Serial.print(address,HEX);
Serial.println(" !");
}
}
```

**CIRCUITO EN PRÁCTICA:**

Periféricos connectados : 2

Conexiones : cada periférico connectado a: PIN 21(representa el SDA del bus), PIN 22(representa el SCL del bus), GND, Vcc (3.3 V)

Imagen del montage:

![Im1](https://github.com/XaviHidalgo/PR5_PD_XavierHidalgo/blob/main/20240318_194215.jpg)


### Ejercicio Practico 2

Este ejercicio nos pedía utilizar dos dispositivos, connectarlos al bus I2C y hacerlos funcionar. En mi caso he utilizado un sensor de temperatura y humedad y un display OLED. El objetivo es hacer que el sensor capte la temperatura y humedad y la muestre por el display.

Después connectar los dos periféricos al bus, utilicé el programa del ejercicio práctico anterior para comprobar que se establecia comunicación con el MP correctamente. Tras ello y ver que los dos estan correctamente connectados, cargué el siguiente programa:

**PROGRAMA:**

``` cpp
#include <Adafruit_AHTX0.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Wire.h>

#define SCREEN_WIDTH 128 // Ancho de la pantalla OLED, en píxeles
#define SCREEN_HEIGHT 32 // Alto de la pantalla OLED, en píxeles

#define OLED_RESET -1 // Pin de reset (o -1 si se comparte el pin de reset del Arduino)
#define SCREEN_ADDRESS 0x3C ///< Ver la hoja de datos para la dirección; 0x3D para 128x64, 0x3C para 128x32

Adafruit_AHTX0 aht;
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

void setup() {
 Wire.begin();
 // Wire1.begin(OLED_SDA, OLED_SCL);
 Serial.begin(115200);
 Serial.println("Adafruit AHT10/AHT20 demo!");

 if (!aht.begin()) {
   Serial.println("¡No se pudo encontrar el AHT! ¡Verifique la conexión!");
   while (1) delay(10);
 }
 Serial.println("AHT10 o AHT20 encontrado");

 if(!display.begin(SSD1306_SWITCHCAPVCC, SCREEN_ADDRESS)) {
   Serial.println(F("¡Error al asignar memoria para SSD1306!"));
   for(;;);
 }

 display.display();
 delay(2000);
 display.clearDisplay();
}

void loop() {
 sensors_event_t humidity, temp;
 aht.getEvent(&humidity, &temp); // Obtener los objetos de temperatura y humedad con datos actualizados

 display.clearDisplay();
 display.setTextSize(1);
 display.setTextColor(SSD1306_WHITE);
 display.setCursor(0, 0);
 display.print("Temperatura: ");
 display.println(temp.temperature);
 display.print("Humedad: ");
 display.println(humidity.relative_humidity);

 display.display();
 Serial.print("Temperatura: ");
 Serial.print(temp.temperature);
 Serial.println(" grados C");
 Serial.print("Humedad: ");
 Serial.print(humidity.relative_humidity);
 Serial.println("% HR");

 delay(500);
}
```

Este programa ha sido realizado con ChatGPT y lo hemos ido retocando hasta que todo ha funcionado correctamente.

Para cargar este programa al MP, es necessario instalar varias librerias en Platformio, estas son las primeras que salen en el código provinientes todas de Adafruit. Estas librerias nos permitiran programar los dispositivos, concreatmente Adafruit_AHTX0.h el sensor y Adafruit_SSD1306.h el display. Evidentemente, incuimos Wire.h como en la parte anterior para poder establecer el bus I2C. Vemos también una série de constantes definidas, estas son solamente **especificaciones**, vemos por ejemplo el ancho y alto del display, etc (ChatGPT ha decidido crearlas a parte con "defines"). Después de los "defines" creamos 2 variables más pero las definimos a partir de clases de las librerias de Adafruit, definimos así "aht" que será la variable que guardará los valores del sensor y "display" que guardará los del display.

El void_setup() de este programa no tiene mucho a comentar, inicializamos bus série y I2C y, después, nos encontramos con una série de condicionales que comprueban la comunicación con los dispositivos. Para comprobar esto se usan unas funciones de las librerías bastante interesantes ya que, todo lo que representava la parte anterior de la práctica, aqui lo hacen con un solo if():

``` cpp
 if (!aht.begin()) {
   Serial.println("¡No se pudo encontrar el AHT! ¡Verifique la conexión!"); //COMPROBAR SI EL **SENSOR** ESTA DISPONIBLE
   while (1) delay(10);
 }
 Serial.println("AHT10 o AHT20 encontrado");

 if(!display.begin(SSD1306_SWITCHCAPVCC, SCREEN_ADDRESS)) {
   Serial.println(F("¡Error al asignar memoria para SSD1306!")); //COMPROBAR SI EL **DISPLAY** ESTA DISPONIBLE
   for(;;);
 }
```

En el void_loop(), creamos dos variables para guardar la temperatura y la humedad que capta el dispositivo en ese momento, consigue sus valores a partir de la función .getEvent(), función que forma parte de nustra variable "ath" tipo Adafruit_AHTX0 proviniente de la libreria Adafruit_AHTX0.h. Después sacaremos los valores obtenidos por el display y, ya que estamos, también por el puerto série. Utilizamos diferentes funciones de nuestra variable "display" tipo Adafruit_SSD1306 proviniente de la libreria Adafruit_SSD1306.h para mostrar los valores por el dispositivo.

Finalmente y, punto muy importante, tras mostrar todos los datos hacemos un delay de 0,5 segundos. Esto significa que cada 0,5 segundos se actualizara la pantalla (sin tener en cuanta los tiempos de procesado que son mínimo) y obtendremos nuevos valores.

Adjunto una imagen del montaje y resultado:

![Im1](https://github.com/XaviHidalgo/PR5_PD_XavierHidalgo/blob/main/20240318_195001.jpg)
