#include "DHT.h"

#define DHTPIN 0
#include "ESP8266WiFi.h"
const char* ssid = "iot" ;
const char* password = "123456789";

const char* host = "www.iotwebserver.in";
String line,rfid;


int cur=0,count=0,voltage=0,power=0,t,h;
float energy,pwr;
#define DHTTYPE DHT11 
DHT dht(DHTPIN, DHTTYPE);

void setup() {
Serial.begin(9600);
  pinMode(A0, INPUT);
  
  Serial.println();
  Serial.begin(9600);
Serial.println();
Serial.println("Connecting to ");
Serial.println(ssid);
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.println("Wifi Not connected");
}
 Serial.println("");
 Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
dht.begin();
pinMode(A0, INPUT);
}

void loop() {
count++;
cur=analogRead(A0)*1.3;
voltage=230;
if(cur < 10)
{
  cur=0;
}
power=cur/1.4;
//power=cur*0.8; 
pwr=pwr+power;
energy=pwr/1000;
  h = dht.readHumidity();
  // Read temperature as Celsius (the default)
  t = dht.readTemperature();
Serial.println(String(cur)+","+String(voltage)+","+String(power)+","+String(pwr)+","+String(energy));

if(count >=20)
{
  String link="/upload.php?id=rlev8eb0&data1="+String(cur)+"&data2="+String(voltage)+"&data3="+String(power)+"&data4="+String(energy)+"&data5="+String(t)+"&data6="+String(h);
  Serial.println(link);
  getdata(link);
  count=0;
}
}


void getdata(String url) 
{

  WiFiClient client;
  const int httpPort = 80;
  if (!client.connect(host, httpPort)) {
   Serial.println("connection failed");
   return;
  }
   client.print(String("GET ") + url + " HTTP/1.1\r\n" +
 "Host: " + host + "\r\n" +
               "Connection: close\r\n\r\n");
   unsigned long timeout = millis();
   while (client.available() == 0) {
       if (millis() - timeout > 5000) {
        Serial.println(">>> Client Timeout !");
        client.stop();
        return;
       }
     }
     // Read all the lines of the reply from server and print them to Serial
      while(client.available()){
        line = client.readStringUntil('\n');
        Serial.println(line);
      }
 }
