// Arduino code for hemolysis percentage via 540 nm absorbance

const int LED_PIN = 9;               // LED drive pin (green ~540 nm)
const int PD_REF_PIN = A0;           // Reference photodiode (incident light)
const int PD_SAMPLE_PIN = A1;        // Sample photodiode (transmitted light)
const float ABS_FULL = 1.0;          // Calibrated absorbance for 100% hemolysis (HiCN standard)
// (Set ABS_FULL = A_full measured during calibration)

void setup() {
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);        // Start with LED off
  Serial.begin(9600);
}

void loop() {
  // Measure dark (ambient) with LED off
  digitalWrite(LED_PIN, LOW);
  delay(50);
  int darkRef = analogRead(PD_REF_PIN);
  int darkSample = analogRead(PD_SAMPLE_PIN);

  // Turn LED on and measure light signals
  digitalWrite(LED_PIN, HIGH);
  delay(50);
  int rawRef = analogRead(PD_REF_PIN);
  int rawSample = analogRead(PD_SAMPLE_PIN);

  // Compute net signal by subtracting dark current
  int I_ref = max(rawRef - darkRef, 1);     // avoid division by zero
  int I_samp = max(rawSample - darkSample, 1);

  // Compute absorbance: A = log10(I_ref / I_samp)
  float ratio = float(I_ref) / float(I_samp);
  float absorbance = log10(ratio);

  // Compute hemolysis percentage using calibrated full-lysis absorbance
  float hemolysisPercent = (absorbance / ABS_FULL) * 100.0;
  if (hemolysisPercent > 100.0) hemolysisPercent = 100.0;
  if (hemolysisPercent < 0.0) hemolysisPercent = 0.0;

  // Output results
  Serial.print("Absorbance = ");
  Serial.print(absorbance, 3);
  Serial.print("   Hemolysis = ");
  Serial.print(hemolysisPercent, 1);
  Serial.println("%");

  delay(500);  // sample rate (~2 readings per second)
}
