#include <WiFi.h>
#include <Firebase_ESP_Client.h>
#include "addons/TokenHelper.h"
#include "addons/RTDBHelper.h"
#include <time.h>

// 1. ตั้งค่า Wi-Fi
#define WIFI_SSID "YOUR_WIFI_NAME"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"

// 2. ตั้งค่า Firebase (คัดลอกจาก Firebase Console)
#define API_KEY "YOUR_FIREBASE_WEB_API_KEY"
#define DATABASE_URL "https://YOUR_PROJECT-default-rtdb.asia-southeast1.firebasedatabase.app"
#define USER_EMAIL "esp32@yourdomain.com"
#define USER_PASSWORD "your_user_password"

// 3. กำหนดขา Relay ตามผังวงจร (ESP32-S3 IO15)
const int RELAY_PIN = 15;

FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

bool relayState = false;
bool lastRelayState = false;
float ratedWatt = 60.0;        // ค่าวัตต์เริ่มต้น (แก้ได้จากหน้าเว็บ)
float totalEnergyKwh = 0.0;    // หน่วยไฟสะสม (kWh)
unsigned long totalRuntimeSec = 0;

unsigned long lastSyncMs = 0;
unsigned long lastLogMs = 0;
unsigned long lastTickMs = 0;

// ดึงเวลาปัจจุบันจาก NTP Server (ประเทศไทย GMT+7)
String getTimeString() {
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) return "N/A";
  char buf[25];
  strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", &timeinfo);
  return String(buf);
}

// บันทึกประวัติการใช้ไฟลง /smart_plug/logs
void pushUsageLog(String eventName, float currentWatt) {
  FirebaseJson logJson;
  logJson.set("time", getTimeString());
  logJson.set("event", eventName);
  logJson.set("power_w", currentWatt);
  logJson.set("energy_kwh", totalEnergyKwh);

  Firebase.RTDB.pushJSON(&fbdo, "/smart_plug/logs", &logJson);
}

void setup() {
  Serial.begin(115200);
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW);

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(300);
    Serial.print(".");
  }
  Serial.println("\nWiFi Connected!");

  // ซิงค์เวลาประเทศไทย (UTC+7 = 25200 วินาที)
  configTime(25200, 0, "pool.ntp.org", "time.nist.gov");

  // ตั้งค่าการเชื่อมต่อ Firebase
  config.api_key = API_KEY;
  config.database_url = DATABASE_URL;
  auth.user.email = USER_EMAIL;
  auth.user.password = USER_PASSWORD;
  config.token_status_callback = tokenStatusCallback;

  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
  lastTickMs = millis();
}

void loop() {
  unsigned long now = millis();

  // คำนวณพลังงานสะสมทุก 1 วินาทีเมื่อรีเลย์เปิดอยู่
  if (now - lastTickMs >= 1000) {
    unsigned long elapsedSec = (now - lastTickMs) / 1000;
    lastTickMs = now;

    if (relayState) {
      totalRuntimeSec += elapsedSec;
      // หากเพิ่ม PZEM-004T ให้เปลี่ยน ratedWatt เป็นค่า pzem.power()
      float currentWatt = ratedWatt;
      // พลังงาน (kWh) = (วัตต์ * วินาที) / 3,600,000
      totalEnergyKwh += (currentWatt * elapsedSec) / 3600000.0;
    }
  }

  // ซิงค์คำสั่งและอัปเดตสถานะกับ Firebase ทุก 2 วินาที
  if (Firebase.ready() && (now - lastSyncMs >= 2000)) {
    lastSyncMs = now;

    // 1) อ่านคำสั่งเปิด-ปิดจากเว็บ
    if (Firebase.RTDB.getBool(&fbdo, "/smart_plug/control/relay")) {
      relayState = fbdo.boolData();
      digitalWrite(RELAY_PIN, relayState ? HIGH : LOW);
    }

    // 2) อ่านค่ากำลังวัตต์ที่ตั้งไว้บนเว็บ
    if (Firebase.RTDB.getFloat(&fbdo, "/smart_plug/control/rated_watt")) {
      float w = fbdo.floatData();
      if (w > 0) ratedWatt = w;
    }

    // 3) ตรวจสอบคำสั่งรีเซ็ตค่าหน่วยสะสม
    if (Firebase.RTDB.getBool(&fbdo, "/smart_plug/control/reset_energy")) {
      if (fbdo.boolData()) {
        totalEnergyKwh = 0.0;
        totalRuntimeSec = 0;
        Firebase.RTDB.setBool(&fbdo, "/smart_plug/control/reset_energy", false);
        pushUsageLog("RESET", 0.0);
      }
    }

    float activeWatt = relayState ? ratedWatt : 0.0;

    // 4) ส่งสถานะปัจจุบันขึ้น Firebase
    FirebaseJson statusJson;
    statusJson.set("state", relayState);
    statusJson.set("power_w", activeWatt);
    statusJson.set("energy_kwh", totalEnergyKwh);
    statusJson.set("runtime_sec", (int)totalRuntimeSec);
    statusJson.set("last_update", getTimeString());
    Firebase.RTDB.setJSON(&fbdo, "/smart_plug/status", &statusJson);

    // 5) บันทึก Log ทันทีเมื่อสถานะเปลี่ยน (ON/OFF)
    if (relayState != lastRelayState) {
      lastRelayState = relayState;
      pushUsageLog(relayState ? "ON" : "OFF", activeWatt);
    }
  }

  // บันทึก Log อัตโนมัติทุก 5 นาทีขณะเปิดไฟใช้งาน
  if (Firebase.ready() && relayState && (now - lastLogMs >= 300000)) {
    lastLogMs = now;
    pushUsageLog("LOG", ratedWatt);
  }
}
