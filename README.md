# MOTION-ACTIVATED-SECURITY-ALARM
#A Motion Activated Security Alarm using an Arduino Uno and PIR sensor to detect movement and trigger an LED and buzzer for real-time security alerts.
// Motion Activated Security Alarm
// PIR OUT -> D2 | LED (220 ohm) -> D12 | Buzzer (+) -> D8

const int pirPin    = 2;
const int ledPin    = 12;
const int buzzerPin = 8;

const bool PASSIVE_BUZZER = false;        // passive buzzer na true
const unsigned long ALARM_TIME = 5000;    // alarm 5 sec
const unsigned long BEEP_GAP   = 200;     // beep speed

bool alarmOn = false;
unsigned long alarmStart = 0;
unsigned long lastBeep = 0;
bool beepState = false;
int lastPir = LOW;

void setBuzzer(bool on) {
  if (PASSIVE_BUZZER) {
    if (on) tone(buzzerPin, 1000);
    else    noTone(buzzerPin);
  } else {
    digitalWrite(buzzerPin, on ? HIGH : LOW);
  }
}

void setup() {
  pinMode(pirPin, INPUT);
  pinMode(ledPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
  Serial.begin(9600);

  Serial.println("PIR warming up... 30 sec wait");
  for (int i = 0; i < 30; i++) {          // warm-up la LED blink
    digitalWrite(ledPin, HIGH);
    delay(500);
    digitalWrite(ledPin, LOW);
    delay(500);
  }
  Serial.println("Alarm ARMED. Security ON.");
}

void loop() {
  int pir = digitalRead(pirPin);

  if (pir == HIGH && lastPir == LOW && !alarmOn) {
    alarmOn = true;
    alarmStart = millis();
    Serial.println("INTRUDER DETECTED! Alarm ON");
  }
  lastPir = pir;

  if (alarmOn) {
    unsigned long now = millis();

    if (now - lastBeep >= BEEP_GAP) {
      lastBeep = now;
      beepState = !beepState;
      setBuzzer(beepState);
      digitalWrite(ledPin, beepState);
    }

    if (now - alarmStart >= ALARM_TIME) {
      alarmOn = false;
      beepState = false;
      setBuzzer(false);
      digitalWrite(ledPin, LOW);
      Serial.println("Alarm OFF. Watching again...");
    }
  }
}
