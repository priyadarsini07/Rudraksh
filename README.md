Photodiode Signal Processing & Hemolysis Calculation

This code segment handles the core signal acquisition and computation for hemolysis analysis by reading transmitted and reflected light intensities from photodiodes, correcting ambient noise, and converting the optical data into hemoglobin concentration (g/dL) and percentage hemolysis (%).

🧩 Code Overview
// 📥 Read photodiodes with LED ON
int rawTrans = analogRead(PD_TRANSM_PIN);
int rawRef   = analogRead(PD_REFLECT_PIN);

// 🌑 Turn OFF LED and capture ambient (dark) readings
digitalWrite(LED_PIN, LOW);
delay(5);
int darkTrans = analogRead(PD_TRANSM_PIN);
int darkRef   = analogRead(PD_REFLECT_PIN);

// 🧮 Calculate net light intensities (subtract ambient)
float I_trans = (float)(rawTrans - darkTrans);
float I_ref   = (float)(rawRef   - darkRef);

// 🚫 Validate readings to prevent division errors
if (I_trans <= 0 || I_ref <= 0) {
  Serial.println("Error: Invalid photodiode readings!");
  delay(1000);
  return;
}

// 📊 Compute absorbance (Beer–Lambert Law)
float absorbance = log10(I_ref / I_trans);

// 🧪 Convert absorbance → Free Hemoglobin (g/dL)
float freeHb = slope * absorbance + intercept;  // Calibration line

// 💔 Compute percentage hemolysis using total Hb
float hemolysisPct = (freeHb / totalHb) * 100.0;

// 🖨️ Display results via Serial Monitor
Serial.print("Free Hb (g/dL): ");
Serial.println(freeHb, 2);
Serial.print("% Hemolysis: ");
Serial.println(hemolysisPct, 2);
Serial.println("------------");

// ⏱️ Repeat measurement every 100 ms
delay(100);

🧠 Working Principle
Step	Process	Description
1️⃣	LED Activation	LED emits green light (~540 nm) through the blood sample.
2️⃣	Signal Capture	Two photodiodes measure transmitted and reflected intensities.
3️⃣	Ambient Correction	LED is turned off; dark readings are subtracted to remove background noise.
4️⃣	Absorbance Calculation	Based on Beer–Lambert law → 𝐴 = log₁₀(𝐼₀ / 𝐼).
5️⃣	Free Hb Estimation	Calibrated linear relation converts absorbance to hemoglobin concentration.
6️⃣	Hemolysis Percentage	Expressed as the ratio of free Hb to total Hb × 100%.
🧾 Example Output
Free Hb (g/dL): 0.68
% Hemolysis: 4.53
------------
