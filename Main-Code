#include "OV2640.h"
#include "TFT_eSPI.h"
#include "WiFi.h"
#include "WiFiClient.h"
#include "cvzones.h"

#define CAMERA_MODEL_AI_THINKER
#include "camera_pins.h"

#define SCREEN_WIDTH 240
#define SCREEN_HEIGHT 320

TFT_eSPI tft = TFT_eSPI();

WiFiClient client;

void setup() {
  Serial.begin(115200);

  tft.begin();
  tft.setRotation(1);

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = -1;
  config.pin_reset = -1;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;

  if (psramFound()) {
    config.frame_size = FRAMESIZE_UXGA;
    config.jpeg_quality = 10;
} else {
config.frame_size = FRAMESIZE_SVGA;
config.jpeg_quality = 12;
}

esp_err_t err = esp_camera_init(&config);
if (err != ESP_OK) {
Serial.printf("Camera init failed with error 0x%x", err);
return;
}

// Connect to Wi-Fi network
WiFi.begin("SSID", "PASSWORD");
while (WiFi.status() != WL_CONNECTED) {
delay(1000);
Serial.println("Connecting to WiFi...");
}
Serial.println("Connected to WiFi");

// Set up connection to Python server
if (!client.connect("SERVER_IP", 5000)) {
Serial.println("Connection to server failed");
return;
}
Serial.println("Connected to server");
}

void loop() {
// Capture an image
camera_fb_t *fb = esp_camera_fb_get();
if (!fb) {
Serial.println("Failed to capture image");
return;
}

// Send image to Python server
String imageData = "";
imageData += "imageStart\n";
imageData += String(fb->len) + "\n";
imageData += String(fb->width) + "\n";
imageData += String(fb->height) + "\n";
client.print(imageData);
client.write(fb->buf, fb->len);
client.println("imageEnd");

// Receive object detection results from Python server
String objectData = "";
while (!client.available()) {
delay(1);
}
while (client.available()) {
char c = client.read();
objectData += c;
}

// Display object detection results on screen
tft.fillScreen(TFT_BLACK);
if (objectData.indexOf("person") != -1) {
tft.drawString("Person detected!", 10, 10);
} else if (objectData.indexOf("cat") != -1) {
tft.drawString("Cat detected!", 10, 10);
} else if (objectData.indexOf("dog") != -1) {
tft.drawString("Dog detected!", 10, 10);
} else {
tft.drawString("No object detected", 10, 10);
}

// Clean up
esp_camera_fb_return(fb);
delay(500);
}
