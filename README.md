#define LED_PIN        9    // Digital pin driving the green LED (~540 nm)
#define PD_TRANSM_PIN  A0   // Analog pin for transmitted light photodiode
#define PD_REFLECT_PIN A1   // Analog pin for reflected (reference) photodiode

// Calibration constants (determined from HiCN standard curve)
const float slope = 1.23;      // example value: (Δ[Hb]/ΔA)
const float intercept = 0.05;  // example offset [g/dL]

// Blood sample parameters (input known values)
const float Hct = 45.0;    // Hematocrit in percent (e.g., 45%)
const float totalHb = 15.0; // Total hemoglobin in g/dL (patient’s total Hb)

void setup() {
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);   // Ensure LED starts off
  Serial.begin(9600);
}

void loop() {
  // Turn on LED and let it stabilize
  digitalWrite(LED_PIN, HIGH);
  delay(10);

  // Read photodiodes with LED on
  int rawTrans = analogRead(PD_TRANSM_PIN);
  int rawRef   = analogRead(PD_REFLECT_PIN);

  // Turn off LED and read ambient (dark) values immediately
  digitalWrite(LED_PIN, LOW);
  delay(5);
  int darkTrans = analogRead(PD_TRANSM_PIN);
  int darkRef   = analogRead(PD_REFLECT_PIN);

  // Net signals (subtract ambient)
  float I_trans = (float)(rawTrans - darkTrans);
  float I_ref   = (float)(rawRef   - darkRef);

  // Prevent division by zero
  if (I_trans <= 0 || I_ref <= 0) {
    Serial.println("Error: Invalid photodiode readings!");
    delay(1000);
    return;
  }

  // Compute absorbance A = log10(I_ref / I_trans):contentReference[oaicite:3]{index=3}
  float absorbance = log10(I_ref / I_trans);

  // Convert absorbance to free hemoglobin concentration (g/dL)
  float freeHb = slope * absorbance + intercept; // Calibration line
  
  // Compute percentage hemolysis using known Hct and total Hb:contentReference[oaicite:4]{index=4}
  float hemolysisPct = ((100.0 - Hct) * freeHb) / totalHb;

  // Output the results
  Serial.print("Free Hb (g/dL): ");
  Serial.println(freeHb, 2);
  Serial.print("% Hemolysis: ");
  Serial.println(hemolysisPct * 100.0, 2); // if desired, multiply by 100
  Serial.println("------------");
  
  // Repeat measurement every 100 ms (adjust as needed up to 2 s total)
  delay(100);
}
