 # 👋 Hey, 

Thanks for reaching over I’m @FidiasAlexopulos. A Professional Scrum Product Owner and Industrial Civil Engineer from the University of Santiago of Chile. I have complemented my skills with a Diploma from the University of Chile in Innovation Management and several online courses like for example Professional Scrum Product Owner at Udemy.

# Some Projects 🚀

## **1. Bronce, An app for Service teams**

This app prototype was made for making more efficient the Field service management for a individual or big number teams.

[Check the prototype here](https://www.figma.com/proto/Ap9ubnpQOThj4OwnVvCiNI/Bronce-APP-(Copy)?node-id=6%3A1248&scaling=scale-down&page-id=0%3A1&starting-point-node-id=1%3A2&show-proto-sidebar=1/)  


## **2. CORFO Transparency viewer**

This is a Public Viewer for transparent information about CORFO public finance saparated by local territory 

[Direct Link to Tableau Project](https://public.tableau.com/views/AdjudicacionesTotalCorfo2021/Dashboard1?:language=es-ES&publish=yes&:display_count=n&:origin=viz_share_link)  
 
<div class='tableauPlaceholder' id='viz1659989441204' style='position: relative'><noscript><a href='#'><img alt='Dashboard 1 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Ad&#47;AdjudicacionesTotalCorfo2021&#47;Dashboard1&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='AdjudicacionesTotalCorfo2021&#47;Dashboard1' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Ad&#47;AdjudicacionesTotalCorfo2021&#47;Dashboard1&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /><param name='filter' value='publish=yes' /></object></div>        
.


## **3. Arduino Water leak detecter**

This is wifi leak detetor that notifies a website that youre house has a water leak

<details>
  <summary>Click to check the code</summary>
  
```python

#include <ESP8266WiFi.h>
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <PubSubClient.h>
#include <Arduino_JSON.h>

 
#define SCREEN_WIDTH 128    // OLED display width, in pixels
#define SCREEN_HEIGHT 64    // OLED display height, in pixels
#define OLED_RESET -1       // Reset pin # (or -1 if sharing Arduino reset pin)
 
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);
 
const char *ssid = "xxxx";     // replace with your wifi ssid and wpa2 key
const char *pass = "xxxx";
const char* mqttServer = "103.123.8.25";
const int mqttPort = 1883;
const char* mqttUser = "fidiasalexopulos";
const char* mqttPassword = "fidiasalexopulos";

 
#define SENSOR  13
 
long currentMillis = 0;
long previousMillis = 0;
int interval = 1000;
int publishinterval = 60000;
long previousMillispublish = 0;
boolean ledState = LOW;
float calibrationFactor = 4.5;
volatile byte pulseCount;
byte pulse1Sec = 0;
float flowRate;
unsigned long flowMilliLitres;
unsigned int totalMilliLitres;
float flowLitres;
float totalLitres;
int flowcount=0;
int leakstatus=0;
 
void IRAM_ATTR pulseCounter()
{
  pulseCount++;
}
 
WiFiClient Leakdetector;
PubSubClient client(Leakdetector);

String UUID()
{
  return WiFi.macAddress();
}

void setup_wifi() {
  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.hostname("fidiasalexopulos"); 
  WiFi.begin(ssid, pass);
 
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
 
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect(UUID().c_str(),mqttUser,mqttPassword)) {
      Serial.println("connected"); 
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup()
{
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqttServer, mqttPort);
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C); //initialize with the I2C addr 0x3C (128x64)
  display.clearDisplay();
  delay(10); 
  pinMode(SENSOR, INPUT_PULLUP); 
  pulseCount = 0;
  flowRate = 0.0;
  flowMilliLitres = 0;
  totalMilliLitres = 0;
  previousMillis = 0; 
  attachInterrupt(digitalPinToInterrupt(SENSOR), pulseCounter, FALLING);
}
 
void publishleak(){
  JSONVar data;
  String uuid = String(UUID());
  data["MAC_Address"] = uuid.c_str();
  data["Flowtotal"] = String(totalLitres);
  data["Leakstatus"] = String(leakstatus);
  String data_json = JSON.stringify(data);
  client.publish("Leaksensor/fidiasalexopulos/",data_json.c_str());
}

void loop()
{
  if (WiFi.status() != WL_CONNECTED){
    setup_wifi();
  }

  if (!client.connected()) {
    reconnect();
  }
 
  currentMillis = millis();
  if (currentMillis - previousMillis > interval) 
  {
    
    pulse1Sec = pulseCount;
    pulseCount = 0;
 
    // Because this loop may not complete in exactly 1 second intervals we calculate
    // the number of milliseconds that have passed since the last execution and use
    // that to scale the output. We also apply the calibrationFactor to scale the output
    // based on the number of pulses per second per units of measure (litres/minute in
    // this case) coming from the sensor.
    flowRate = ((1000.0 / (millis() - previousMillis)) * pulse1Sec) / calibrationFactor;
    if (flowRate <=0){
      flowcount = 0;
      leakstatus = 0;
          }
    if (flowRate > 0){
      flowcount = flowcount+1;
      Serial.print(String(flowcount));
      if (flowcount > 10){
        leakstatus = 1;
      }
    }
    previousMillis = millis();
 
    // Divide the flow rate in litres/minute by 60 to determine how many litres have
    // passed through the sensor in this 1 second interval, then multiply by 1000 to
    // convert to millilitres.
    flowMilliLitres = (flowRate / 60) * 1000;
    flowLitres = (flowRate / 60);
 
    // Add the millilitres passed in this second to the cumulative total
    totalMilliLitres += flowMilliLitres;
    totalLitres += flowLitres;

    if (leakstatus == 1){
     display.clearDisplay(); 
     display.setCursor(0,25);  //oled display
     display.setTextSize(2);
     display.setTextColor(WHITE);
     display.print("!!!Leak!!!");
     display.display();
     publishleak();     
    }
     if (leakstatus == 0){    

    // Print the flow rate for this second in litres / minute
    Serial.print("Flow rate: ");
    Serial.print(float(flowRate));  // Print the integer part of the variable
    Serial.print("L/min");
    Serial.print("\t");       // Print tab space
 
    display.clearDisplay();
    
    display.setCursor(10,0);  //oled display
    display.setTextSize(1);
    display.setTextColor(WHITE);
    display.print("Water Flow Meter");
    
    display.setCursor(0,20);  //oled display
    display.setTextSize(2);
    display.setTextColor(WHITE);
    display.print("R:");
    display.print(float(flowRate));
    display.setCursor(100,28);  //oled display
    display.setTextSize(1);
    display.print("L/M");
 
    // Print the cumulative total of litres flowed since starting
    Serial.print("Output Liquid Quantity: ");
    Serial.print(totalMilliLitres);
    Serial.print("mL / ");
    Serial.print(totalLitres);
    Serial.println("L");
 
    display.setCursor(0,45);  //oled display
    display.setTextSize(2);
    display.setTextColor(WHITE);
    display.print("V:");
    display.print(totalLitres);
    display.setCursor(100,53);  //oled display
    display.setTextSize(1);
    display.print("L");
    display.display();
  }
  }
 
 if (currentMillis - previousMillispublish > publishinterval){
  
  previousMillispublish = millis();
  JSONVar data;
  String uuid = String(UUID());
  data["MAC_Address"] = uuid.c_str();
  data["Flowtotal"] = String(totalLitres);
  data["Leakstatus"] = String(leakstatus);
  String data_json = JSON.stringify(data);
  client.publish("Leaksensor/fidiasalexopulos/",data_json.c_str());
 }  
 client.loop(); 
 delay(500);
}

```
</details>

https://user-images.githubusercontent.com/109599051/183510748-36509534-749a-4e44-b90f-c6c77cbb17d3.mp4

![image](https://user-images.githubusercontent.com/109599051/183510998-65f8d1b4-907d-44b2-beed-9e0169a08e6f.png)


## **4. A Family buisness**
This is a leak detection company that i had with my family. It´s still goes on...
[Rescate Hogar Web Page](https://rescatehogar.cl/)

# Final Comments 👍

Very happy you made it to down here. I´m constantly publishing content on my  LinkedIn profile [https://www.linkedin.com/in/fidias-alexopulos]  and here is my complete resume ![CV Fidias Alexopulos Markar 2022](https://user-images.githubusercontent.com/109599051/183512687-1f416c28-059e-4d3c-b7a5-c69e4067f795.png)
