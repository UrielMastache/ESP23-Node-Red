# Practica ESP32 con DHT22 Y visualizando datos recabados del sensor en tiempo real en un Dashboard

Este repositorio muestra como podemos programar una ESP32 con el sensor DHT22 para saber la temperatura y la humedad, posteriormente se visualizara en un dashboard en tiempo real los datos simulados.

## Introducción

### Descripción

La ```Esp32``` la utilizamos en un entorno de adquision de datos, y un sensor (```DHT22```) Cabe aclarar que esta practica se usara un simulador llamado [WOKWI](https://https://wokwi.com/), y utilizando node red, para realizar la visualización en tiempo real de los datos simulados.


## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- Programa instalado Node Red
- Sensor DHT22
- Y muchas ganas de chambear :D


## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).

Tener instalado el programa de Node Red (en este proyecto no se muestra la instalación así que puede buscar un tutorial a parte y regresar para finalizar el proyecto una vez lo tenga listo).


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación de Wokwi y colocar la siguente programación:


```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="ElUriel";
String password_mqtt="230397";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("ElUriel/Programador", output.c_str());
  }
}

```

2. Hacer la conexion de **DHT22** con la **ESP32** como se muestra en la siguente imagen.



![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/conexiones.png?raw=true)

3. Posteriormente a instalar el programa de Node Red nos iremos a las 3 barritas en la parte superior derecha y seleccionaremos "Manage palette"

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/3barritas.png?raw=true)

4. Instalaremos los siguientes bloques.

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/bloques%20instalados.png?raw=true)

5. Agregaremos los siguientes bloques en nuestro espacio de trabajo de Nod Red.
![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/bloques%20en%20Node.png?raw=true)

6. Configuraremos el primer bloque mqtt de acuerdo al server que pusimos en el codigo.En este ejemplo usamos el server 44.195.202.69

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/server%20codigo.png?raw=true)

Aquí se ve en la configuración del bloque en la parte del server (dando doble clic al bloque).

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/server%20mqt.png?raw=true)

En la parte de TOPIC colocaremos lo que escribimos en la parte final del codigo como cliente.

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/cliente.png?raw=true)

7. Configuración del segundo bloque. Daremos doble clic al bloque Json, y cambiaremos la opcion a "always convert a javascript object".

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/json.png?raw=true).

8. En los dos siguientes bloques de funciones copiaremos el siguiente codigo uno para cada bloque.

Bloque 1 Temperatura
```
msg.payload = msg.payload.TEMPERATURA;
msg.topic = "TEMPERATURA";
return msg;
```
Bloque 2 Humedad
```
msg.payload = msg.payload.HUMEDAD;
msg.topic = "HUMEDAD";
return msg;
```
9. Para los ultimos bloques nos iremos a la pagina de Node Red, en la flecha del lado derecho nos iremos al apartado "Dashboard".

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/dash.png?raw=true).

Posteriormente le daremos al boton de "+Tab" y le pondremos el nombre que querramos.

Nos vamos de regreso a los ultimos bloques y comenzamos con su config.

10. Bloque gauge de temperatura.

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/config%20temperatura%20bloque.png?raw=true).

**Cabe aclarar que cuando le des a "Done" se agregara automaticamente a tu apartado de dashboard dentro de tu proyecto, en este caso como en una sub carpeta en "El programador"**

11. Bloque gauge de Humedad.

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/config%20humedad%20bloque.png?raw=true).

12. Bloque chart Grafico.

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/config%20char%20bloque.png?raw=true).

Asi te debe quedar la configuración de bloques y que te aparezca en tu dashboard en Node Red.

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/Node%20red%20bloques%20finales.png?raw=true).



### Instrucciónes de operación

1. Iniciar simulador.
2. Para poder ver los datos en tiempo real tienes que darle al cuadro superior derecho (cuadro con una flecha apuntando hacia arriba-derecha) para que te mande a tu dashboard.

## Resultados


Así tienes que ver tus resultados
Cambiando en tiempo real en el simulador

Temperatura y humedad 1

Simulador 

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/4%2037%20simulador.png?raw=true)

Dashboard

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/4%2037%20dash.png?raw=true)


Temperatura y humedad 2

Simulador 

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/73%20y%2055%20simulador.png?raw=true)

Dashboard

![](https://github.com/UrielMastache/ESP23-Node-Red/blob/main/73%20y%2055%20dashboard.png?raw=true)


## Comentarios

Recuerda que esto es demostrativo, y puedes modificar todo a como mejor te adaptes y lo necesites.


Gracias por ver :D 

Ciao.