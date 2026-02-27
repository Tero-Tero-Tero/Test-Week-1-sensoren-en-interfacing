// Test-Week-1-sensoren-en-interfacing

#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BMP280.h> 
#include <DHT.h>
#include <LiquidCrystal_I2C.h>


#define DHTPIN 2            // DHT11 
#define DHTTYPE DHT11       

// I2C adressen 
#define BMP280_ADDRESS 0x76 // of 0x77 
#define LCD_ADDRESS    0x27 // of 0x3F


Adafruit_BMP280 bmp;        
DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal_I2C lcd(LCD_ADDRESS, 16, 2); // 16x2 LCD display

// Variabelen voor temperaturen
float tempBinnen = 0.0;
float tempBuiten = 0.0;
unsigned long vorigeMeting = 0;
const long interval = 2000; // elke 2 seconden

void setup() {
  Serial.begin(9600);
  Wire.begin();
  
  
  if (!bmp.begin(BMP280_ADDRESS)) {
    Serial.println("BMP280 niet gevonden!");
    Serial.println("Controleer de bedrading en het I2C-adres (0x76 of 0x77)"); //probleem oplossing in serial monitor
    while (1); // blijf hangen
  }
  
  // Start DHT
  dht.begin();
  
  // Start LCD
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Binnen: ");
  lcd.setCursor(0, 1);
  lcd.print("Buiten: ");
  
  Serial.println("BMP280 binnen thermometer gestart!");
}

void loop() {
  unsigned long nu = millis();
  if (nu - vorigeMeting >= interval) {
    vorigeMeting = nu;
    
    // Lees binnen temperatuur van BMP280
    tempBinnen = bmp.readTemperature(); // in graden Celsius
    
    // Lees buiten temperatuur van DHT11
    float t = dht.readTemperature();    // in graden Celsius
    
    // Controleer of de DHT11-meting geldig is
    if (!isnan(t)) {
      tempBuiten = t;
    } else {
      Serial.println("Fout bij uitlezen DHT11");
    }
    
    // Toon op LCD
    lcd.setCursor(9, 0); // na "Binnen: "
    lcd.print(tempBinnen, 1); // 1 decimaal
    lcd.print(" C  ");   // extra spaties om rest te wissen
    
    lcd.setCursor(9, 1); // na "Buiten: "
    lcd.print(tempBuiten, 1);
    lcd.print(" C  ");
    
    // Ook naar seriële monitor voor debugging
    Serial.print("Binnen (BMP280): "); Serial.print(tempBinnen); Serial.println(" C");
    Serial.print("Buiten (DHT11): "); Serial.print(tempBuiten); Serial.println(" C");
    Serial.println("-------------------");
  }
}
