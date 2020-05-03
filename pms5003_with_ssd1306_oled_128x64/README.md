![pms5003 serial nodemcu esp8266mod](https://raw.githubusercontent.com/hanandika/pms5003_serial_nodemcu_esp8266mod/master/images/pms5003_serial_nodemcu_ssd1306.png)

# pms5003_serial_nodemcu_esp8266mod
particle sensor pms5003, pms6003, pms7003 using Serial communication line rather than digital pin.

### CAUTION
Please remember **unplug** TX-cable(from sensor)/RX-cable(from nodemcu) when uploading program.
**Re-connect** TX-cable(from sensor)/RX-cable(from nodemcu) when succesfully upload, and **do RESET** your nodemcu
https://www.youtube.com/watch?v=ZFmDEGK6E74

### CODE
```c++
// I2C library
#include <Wire.h>
#include <Adafruit_SSD1306.h>

//
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels
#define OLED_RESET    -1 // Reset pin # (or -1 if sharing Arduino reset pin)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

struct pms5003data {
  uint16_t framelen;
  uint16_t pm10_standard, pm25_standard, pm100_standard;
  uint16_t pm10_env, pm25_env, pm100_env;
  uint16_t particles_03um, particles_05um, particles_10um, particles_25um, particles_50um, particles_100um;
  uint16_t unused;
  uint16_t checksum;
};
struct pms5003data data;

void setup(){
  Serial.begin(9600);

  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // Address 0x3D for 128x64
    Serial.println(F("SSD1306 allocation failed"));
//    for(;;); // Don't proceed, loop forever
  }
}


void loop(){
  if (readPMSdata(&Serial)) {
    // reading data was successful!
    Serial.println();
    Serial.println("---------------------------------------");
    Serial.println("Concentration Units (standard)");
    Serial.print("PM 1.0: "); Serial.print(data.pm10_standard);
    Serial.print("\t\tPM 2.5: "); Serial.print(data.pm25_standard);
    Serial.print("\t\tPM 10: "); Serial.println(data.pm100_standard);
    Serial.println("---------------------------------------");
    Serial.println("Concentration Units (environmental)");
    Serial.print("PM 1.0: "); Serial.print(data.pm10_env);
    Serial.print("\t\tPM 2.5: "); Serial.print(data.pm25_env);
    Serial.print("\t\tPM 10: "); Serial.println(data.pm100_env);
    Serial.println("---------------------------------------");
    Serial.print("Particles > 0.3um / 0.1L air:"); Serial.println(data.particles_03um);
    Serial.print("Particles > 0.5um / 0.1L air:"); Serial.println(data.particles_05um);
    Serial.print("Particles > 1.0um / 0.1L air:"); Serial.println(data.particles_10um);
    Serial.print("Particles > 2.5um / 0.1L air:"); Serial.println(data.particles_25um);
    Serial.print("Particles > 5.0um / 0.1L air:"); Serial.println(data.particles_50um);
    Serial.print("Particles > 10.0 um / 0.1L air:"); Serial.println(data.particles_100um);
    Serial.println("---------------------------------------");
    
    print_header();
    display.setTextSize(1);
    //display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
    display.setTextColor(SSD1306_WHITE);
    display.print(F("PM 1.0 : "));
    display.println(data.pm10_standard);
    display.print(F("PM 2.5 : "));
    display.println(data.pm25_standard);
    display.print(F("PM  10 : "));
    display.println(data.pm100_standard);
    display.display();
  } else {
    //Serial.println("cannot readPMSdata");
  }
}

boolean readPMSdata(Stream *s) {
  if (! s->available()) {
    return false;
  }
  // Read a byte at a time until we get to the special '0x42' start-byte
  if (s->peek() != 0x42) {
    s->read();
    return false;
  }
  // Now read all 32 bytes
  if (s->available() < 32) {
    return false;
  }
  uint8_t buffer[32];    
  uint16_t sum = 0;
  s->readBytes(buffer, 32);
  // get checksum ready
  for (uint8_t i=0; i<30; i++) {
    sum += buffer[i];
  }
  /* debugging
  for (uint8_t i=2; i<32; i++) {
    Serial.print("0x"); Serial.print(buffer[i], HEX); Serial.print(", ");
  }
  Serial.println();
  */
  // The data comes in endian'd, this solves it so it works on all platforms
  uint16_t buffer_u16[15];
  for (uint8_t i=0; i<15; i++) {
    buffer_u16[i] = buffer[2 + i*2 + 1];
    buffer_u16[i] += (buffer[2 + i*2] << 8);
  }
  // put it into a nice struct :)
  memcpy((void *)&data, (void *)buffer_u16, 30);
  if (sum != data.checksum) {
    Serial.println("Checksum failure");
    return false;
  }
  // success!
  return true;
}


void print_header(){
  display.clearDisplay();
  display.display();
  display.setTextSize(1);             // Normal 1:1 pixel scale
  display.setTextColor(SSD1306_WHITE);        // Draw white text
  display.setCursor(0,0);             // Start at top-left corner
  display.println(F("*** hanandika"));
  display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
  display.println(F("Pollution Sensor v.1 "));
}
void print_float(String names, float value, String satuan) {
  Serial.print(names);Serial.print(value);Serial.print(satuan);Serial.println();
  print_header();
  display.setTextSize(2);
//  display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
  display.setTextColor(SSD1306_WHITE);
  display.println(names);
  display.println();
  display.setTextSize(2); // Draw 2X-scale text
  display.setTextColor(SSD1306_WHITE);
  display.print(value);
  display.print(satuan);
  display.display();
  delay(4000);
}
void print_int(String names, int value, String satuan) {
  Serial.print(names);Serial.print(value);Serial.print(satuan);Serial.println();
  print_header();
  display.setTextSize(2);
//  display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
  display.setTextColor(SSD1306_WHITE);
  display.println(names);
  display.println();
  display.setTextSize(2); // Draw 2X-scale text
  display.setTextColor(SSD1306_WHITE);
  display.print(value);
  display.print(satuan);
  display.display();
  delay(4000);
}
```



![pms5003 serial nodemcu esp8266mod](https://raw.githubusercontent.com/hanandika/pms5003_serial_nodemcu_esp8266mod/master/images/image.jpeg)