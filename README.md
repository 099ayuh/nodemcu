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
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>

// Replace with your network credentials
const char* ssid     = "replace with your WiFi ssid";
const char* password = "replace with your WiFi password";

// REPLACE with your Domain name and URL path or IP address with path
const char* serverName = "http://ip_address/sensordata/post-esp-data.php";

// Keep this API Key value to be compatible with the PHP code provided in the project page. 
// If you change the apiKeyValue value, the PHP file /post-esp-data.php also needs to have the same key 
String apiKeyValue = "tPmAT5Ab3j7F9";

String sensorName = "BME280";
String sensorLocation = "My Room";

#define SEALEVELPRESSURE_HPA (1013.25)
Adafruit_BME280 bme;  // I2C
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

  // (you can also pass in a Wire library object like &Wire2)
  bool status = bme.begin(0x76);
  if (!status) {
    Serial.println("Could not find a valid BME280 sensor, check wiring or change I2C address!");
    while (1);
  }
}
void loop() {
  //Check WiFi connection status
  if(WiFi.status()== WL_CONNECTED){
    HTTPClient http;
    
    // Your Domain name with URL path or IP address with path
    http.begin(serverName);
    
    // Specify content-type header
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");
    
    // Prepare your HTTP POST request data
    String httpRequestData = "api_key=" + apiKeyValue + "&sensor=" + sensorName
                          + "&location=" + sensorLocation + "&value1=" + String(bme.readTemperature())
+ "&value2=" + String(bme.readHumidity()) + "&value3=" + String(bme.readPressure()/100.0F) + "";
    Serial.print("httpRequestData: ");
    Serial.println(httpRequestData);
    
    // You can comment the httpRequestData variable above
    // then, use the httpRequestData variable below (for testing purposes without the BME280 sensor)
    //String httpRequestData = "api_key=tPmAT5Ab3j7F9&sensor=BME280&location=Office&value1=24.75&value2=49.54&value3=1005.14";
 // Send HTTP POST request
    int httpResponseCode = http.POST(httpRequestData);
     
    // If you need an HTTP request with a content type: text/plain
    //http.addHeader("Content-Type", "text/plain");
    //int httpResponseCode = http.POST("Hello, World!");
    
    // If you need an HTTP request with a content type: application/json, use the following:
    //http.addHeader("Content-Type", "application/json");
    //int httpResponseCode = http.POST("{\"value1\":\"19\",\"value2\":\"67\",\"value3\":\"78\"}");
  if (httpResponseCode>0) {
      Serial.print("HTTP Response code: ");
      Serial.println(httpResponseCode);
    }
    else {
      Serial.print("Error code: ");
      Serial.println(httpResponseCode);
    }
    // Free resources
    http.end();
  }
  else {
    Serial.println("WiFi Disconnected");
  }
  //Send an HTTP POST request every 15 seconds
  delay(15000);  
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

