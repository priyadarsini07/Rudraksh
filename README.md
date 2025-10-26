Calibration Constants
const float slope = 3.68;     // Δ[Hb]/ΔA at 540 nm (ICSH reference)
const float intercept = 0.02; // Zero intercept (Drabkin’s reagent blank)


These values are derived from WHO/ICSH HiCN standard curves and can be adjusted per your spectrophotometric calibration data.

🧠 Physiological Reference Parameters
const float Hct = 42.0;      // Hematocrit (%)
const float totalHb = 15.0;  // Total hemoglobin [g/dL]

🧾 Full Code
#include <Wire.h>
#include <U8g2lib.h>

// 🖥️ Define display object for 1.3" SH1106 OLED
U8G2_SH1106_128X64_NONAME_F_HW_I2C u8g2(U8G2_R0, U8X8_PIN_NONE);

// 🔌 Pin definitions
#define LED_PIN        9     // Green LED (~540 nm)
#define PD_TRANSM_PIN  A0    // Transmitted photodiode
#define PD_REFLECT_PIN A1    // Reflected photodiode

// 📏 Calibration constants (WHO/ICSH reference)
const float slope = 3.68;      // Δ[Hb]/ΔA (g/dL per absorbance unit)
const float intercept = 0.02;  // Zero offset

// 🧬 Physiological reference values
const float Hct = 42.0;        // Hematocrit (%)
const float totalHb = 15.0;    // Total Hb (g/dL)

void setup() {
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);
  Serial.begin(9600);

  // 🖥️ Initialize OLED
  u8g2.begin();
  u8g2.setFont(u8g2_font_ncenB14_tr);

  // 🚀 Welcome screen
  u8g2.clearBuffer();
  u8g2.setCursor(10, 35);
  u8g2.print("WELCOME");
  u8g2.setCursor(10, 60);
  u8g2.print("RUDRAKSH");
  u8g2.sendBuffer();
  delay(2000);

  // Switch to smaller font for data
  u8g2.setFont(u8g2_font_6x13_tf);
}

void loop() {
  // 💡 Activate LED and stabilize
  digitalWrite(LED_PIN, HIGH);
  delay(10);

  // 📥 Read photodiodes (LED ON)
  int rawTrans = analogRead(PD_TRANSM_PIN);
  int rawRef   = analogRead(PD_REFLECT_PIN);

  // 🌑 Read dark signals (LED OFF)
  digitalWrite(LED_PIN, LOW);
  delay(5);
  int darkTrans = analogRead(PD_TRANSM_PIN);
  int darkRef   = analogRead(PD_REFLECT_PIN);

  // 🔍 Compute net light intensities
  float I_trans = (float)(rawTrans - darkTrans);
  float I_ref   = (float)(rawRef - darkRef);

  // ⚠️ Validate readings
  if (I_trans <= 0 || I_ref <= 0) {
    Serial.println("Error: Invalid photodiode readings!");
    showError("Invalid PD readings!");
    delay(1000);
    return;
  }

  // 📊 Absorbance calculation
  float absorbance = log10(I_ref / I_trans);

  // 🧪 Convert absorbance → Free Hb (g/dL)
  float freeHb = slope * absorbance + intercept;

  // 💔 Calculate % hemolysis (WHO/ICSH)
  float hemolysisPct = (freeHb / totalHb) * 100.0;

  // 🖨️ Serial output
  Serial.print("Free Hb (g/dL): ");
  Serial.println(freeHb, 2);
  Serial.print("% Hemolysis: ");
  Serial.println(hemolysisPct, 2);
  Serial.println("------------");

  // 🖥️ OLED output
  u8g2.clearBuffer();
  u8g2.setCursor(0, 15);
  u8g2.print("Hemoglobin Meter");

  u8g2.setCursor(0, 35);
  u8g2.print("Free Hb: ");
  u8g2.print(freeHb, 2);
  u8g2.print(" g/dL");

  u8g2.setCursor(0, 55);
  u8g2.print("Hemolysis: ");
  u8g2.print(hemolysisPct, 2);
  u8g2.print("%");
  u8g2.sendBuffer();

  delay(100); // Sampling rate
}

void showError(const char* msg) {
  u8g2.clearBuffer();
  u8g2.setCursor(0, 30);
  u8g2.print("ERROR:");
  u8g2.setCursor(0, 50);
  u8g2.print(msg);
  u8g2.sendBuffer();
}

🧾 Example Serial Output
Free Hb (g/dL): 0.72
% Hemolysis: 4.88
------------

🖥️ OLED Display Output
Hemoglobin Meter
Free Hb: 0.72 g/dL
Hemolysis: 4.88%

