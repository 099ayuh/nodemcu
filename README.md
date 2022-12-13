# Connecting the ESP8266 Nodemcu with BME 280 sensor.
### To insert data into the database, we are going to use the ESP8266 Nodemcu and BME280 sensor so that we can insert temperature, humidity, pressure readings into our database every 30 seconds. The setup is as shown in the schematic below.

![image](https://user-images.githubusercontent.com/83543768/207245733-6873c8ae-05a2-42b9-8439-c35db1a51185.png)



```js
#ifdef ESP32
  #include <WiFi.h>
  #include <HTTPClient.h>
#else
  #include <ESP8266WiFi.h>
  #include <ESP8266HTTPClient.h>
  #include <WiFiClient.h>
#endif
#include <Wire.h>
#define ln D2
const char* ssid     = "Wifi name";
const char* password = "Wifi password";
WiFiClient wifiClient;
String serverName = "http://ipv4 address:3000/POST";
String apiKeyValue = "tPmAT5Ab3j7F9";
String sensorName = "BME280";
String sensorLocation = "My Room";
#define SEALEVELPRESSURE_HPA (1013.25)
void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  Serial.println("Connecting");
  while(WiFi.status() != WL_CONNECTED) { 
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to WiFi network with IP Address: ");
  Serial.println(WiFi.localIP());
  pinMode(ln, INPUT);  
}
void loop() {
  if(WiFi.status()== WL_CONNECTED){
    HTTPClient http;
    http.begin(wifiClient, serverName);
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");
    Serial.println(random(100));
    String httpRequestData = "&sensor=" + sensorName + "&location=" + sensorLocation + "&value1=" + digitalRead(ln);
    Serial.print("httpRequestData: ");
    Serial.println(httpRequestData);
    int httpResponseCode = http.POST(httpRequestData);
  if (httpResponseCode>0) {
      Serial.print("HTTP Response code: ");
      Serial.println(httpResponseCode);
    }
    else {
      Serial.print("Error code: ");
      Serial.println(httpResponseCode);
    }
    http.begin(wifiClient, "http://192.168.1.3:3000/");
    int httpCode = http.GET(); 
    if (httpCode > 0) {
      String payload = http.getString(); 

      Serial.println(payload);
    }
    http.end();
  }
  else {
    Serial.println("WiFi Disconnected");
  }
  delay(11500);  
}
```

### code-with-dht11
```js
#ifdef ESP32
  #include <WiFi.h>
  #include <HTTPClient.h>
#else
  #include <ESP8266WiFi.h>
  #include <ESP8266HTTPClient.h>
  #include <WiFiClient.h>
#endif
#include <Wire.h>
#include "DHT.h"
#define DHTPIN D2 
#define DHTTYPE DHT11
const char* ssid     = "Subhamoy";
const char* password = "PASS020302";
WiFiClient wifiClient;
String serverName = "http://192.168.1.3:3000/POST";
DHT dht(DHTPIN, DHTTYPE);
void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  Serial.println("Connecting");
  while(WiFi.status() != WL_CONNECTED) { 
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to WiFi network with IP Address: ");
  Serial.println(WiFi.localIP());
  dht.begin();  
}
void loop() {
  if(WiFi.status()== WL_CONNECTED){
    HTTPClient http;
    http.begin(wifiClient, serverName);
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");
    delay(1500);
    float h = dht.readHumidity();
    float t = dht.readTemperature();
    float f = dht.readTemperature(true);
    if (isnan(h) || isnan(t) || isnan(f)) {
      Serial.println(F("Failed to read from DHT sensor!"));
      return;
  }
  float hif = dht.computeHeatIndex(f, h);
  float hic = dht.computeHeatIndex(t, h, false);
  String httpRequestData = "&Temperature=" + String(t) + "&Humidity=" + String(h) + "&Heat Index=" + String(hic)+" "+String(hif);
  int httpResponseCode = http.POST(httpRequestData);
  if (httpResponseCode>0) {
     // Serial.print("HTTP Response code: ");
      //Serial.println(httpResponseCode);
    }
    else {
      Serial.print("Error code: ");
      Serial.println(httpResponseCode);
    }
    http.begin(wifiClient, "http://192.168.1.3:3000/");
    int httpCode = http.GET(); 
    if (httpCode > 0) {
      String payload = http.getString(); 
      Serial.println(payload);
    }
    http.end();
  }
  else {
    Serial.println("WiFi Disconnected");
  }
  delay(1500);  
}
```


Don’t forget to replace the SSID and password corresponding to your network. Also make sure you input the correct domain name and URL path or IP address, so the ESP publishes the sensor readings to your own server.
const char* serverName = "http://ip_address/sensordata/post-esp-data.php";

After uploading the above code to the ESP82665 Nodemcu you can be able to access the sensor reading via a web page using the URL; https://localhost/sensordata

![image](https://user-images.githubusercontent.com/83543768/207245984-89bf4766-3152-4a32-af71-bd0f54193a62.png)

You can also access this webpage using other devices like mobile phone and other computers using the URL: http://ip_address/sensordata

You can also view the readings in the database in the sensordata table.

![image](https://user-images.githubusercontent.com/83543768/207246033-551d511e-2247-4e84-85e2-d0bbe4de9867.png)

In the above setup we were using a local host with the help of XAMPP web server to host MySQL database locally on your Windows PC. However you can use this knowledge for a real website in case you have your own domain name and hosting account as demonstrated here.

