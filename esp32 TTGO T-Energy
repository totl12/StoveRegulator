#include <ESP32Servo.h>
#include <GyverPID.h>
#include <OneWire.h>
#include <DallasTemperature.h>
#include <GyverPortal.h>
#include "Adafruit_SSD1306.h"
GyverPortal portal;

uint16_t v1 = analogRead(35);
Adafruit_SSD1306 display(128, 64, &Wire, 4); // указываем размер экрана в пикселях

// GPIO where the DS18B20 is connected to
const int oneWireBus = 5;   
float temperatureC;
OneWire oneWire(oneWireBus);
DallasTemperature sensors(&oneWire);

Servo servo;  // create servo object to control a servo
GyverPID regulator(20, 0, 0, 10000);  // коэф. П, коэф. И, коэф. Д, период дискретизации dt (мс)
String valTemp = "23";
String p = "20";
String i = "0";
String d = "0";

bool valSwitch;

// конструктор страницы
void build() {
  BUILD_BEGIN();
  GP.AJAX_UPDATE("val");
  GP.THEME(GP_DARK);
  GP.TITLE("PID", "t1");
  GP.HR();
  GP.LABEL("Температура: ");
  GP.LABEL("NAN","val");      GP.BREAK();
  GP.HR();
  GP.LABEL("Удерживать:");
  GP.TEXT("tem", "°C" , valTemp);      GP.BREAK();
  GP.HR();
  GP.TEXT("p", "П:default 20",p);      GP.BREAK();
  GP.TEXT("i", "И:default 0",i);      GP.BREAK();
  GP.TEXT("d", "Д:default 0",d);      GP.BREAK();
  GP.HR();  
  GP.AJAX_PLOT_DARK("plot", 3, 20000, 1000);
  GP.BUILD_END();
  BUILD_END();
}

const char* ssid = "tt";
const char* password = "123456789";

void setup() {
  Serial.begin(115200);  
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // Address 0x3D for 128x64
    Serial.println(F("SSD1306 allocation failed"));
    for(;;);
  }
  delay(1000);
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(WHITE);
  display.setCursor(0, 0);
  WiFi.mode(WIFI_STA); //Optional
  Serial.println("**Start Scan**");
  int network = WiFi.scanNetworks();
  Serial.print("Number of Network : ");
  Serial.println(network);
  if (network == 0) {
    Serial.println("No Networks Found");
  }  else {
    Serial.print(network);
    Serial.println(" Networks Found"); 
    String ssid;
    for (int i = 0; i < network; ++i) {
       ssid = WiFi.SSID(i);
       char _ssid[ssid.length()];
       ssid.toCharArray(_ssid, ssid.length()+1);
       WiFi.begin(_ssid,password);
       Serial.println("\nConnecting: " + WiFi.SSID(i) );
       display.setCursor(0, 0);
       display.clearDisplay();
       display.println(WiFi.SSID(i));
       display.display(); 
       delay(100); 
       int n = 0;
       while(WiFi.status() != WL_CONNECTED){
           n=n+1;
           Serial.print(".");
           delay(1000);
           if (n > 20){
            break;  
           }
       }
       if (WiFi.status() == WL_CONNECTED){
         break; 
       }     
    }
  }     
   Serial.println(WiFi.localIP());

  // подключаем конструктор и запускаем
  portal.attachBuild(build);
  portal.enableAuth("1", "1");
  portal.attach(action);
  portal.start();
  
  sensors.begin();
  servo.attach(4);
  regulator.setDirection(NORMAL);   
  regulator.setpoint = 24;        // сообщаем регулятору температуру, которую он должен поддерживать
  regulator.setLimits(0,90);      // пределы (ставим для 8 битного ШИМ). ПО УМОЛЧАНИЮ СТОЯТ 0 И 255 

  delay(1000);
  display.clearDisplay();
  // Display static text
  display.println(WiFi.localIP());
  display.display(); 
  delay(5000);
  display.clearDisplay();
  delay(1000);
}

void action() {
   if (portal.update()) {
    if (portal.update("val")) portal.answer(temperatureC);
   }
 
   if(portal.click()){
     if (portal.click("p")) {
       p=portal.getInt("p");
       regulator.Kp += p.toFloat();
     }
     if (portal.click("i")) {
       i=portal.getInt("i");
       regulator.Ki += i.toFloat();
     }
     if (portal.click("d")) {
       d=portal.getInt("d"); 
       regulator.Kd += d.toFloat();
     }
    if (portal.click("tem")) {
      valTemp= portal.getInt("tem");
      regulator.setpoint = valTemp.toFloat();
    }
   }

  if (portal.update("plot")) {
    int answ[] = {valTemp.toFloat(), temperatureC, regulator.getResult()};
    portal.answer(answ, 3);
  }  
}

void loop() {
  float battery_voltage = ((float)v1 / 4095.0) * 2.0 * 3.3 * (1100 / 1000.0);
  Serial.print(battery_voltage);
  portal.tick(); 
  sensors.requestTemperatures(); 
  temperatureC = sensors.getTempCByIndex(0);
  Serial.print(temperatureC);
  Serial.println("---");
  regulator.input =  temperatureC;
  regulator.getResult();
  servo.write(regulator.output); // отправляем на серву
  Serial.println(p);
  Serial.println(i);
  Serial.println(d);
  Serial.println(valTemp);
  display.setTextSize(3); // указываем размер шрифта
  display.setTextColor(SSD1306_WHITE); // указываем цвет надписи
  display.setCursor(20, 10);
  display.println(temperatureC);
  display.display();
  delay(100);
  display.clearDisplay(); // очищаем экран
  
}
