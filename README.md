# esp32-smart-home
it automatically controls your home or commercial appliances or machine via instructing only one , controlled by mobile phone , by your voice and simply touch command also
#include <WiFi.h>
#include "SinricPro.h"
#include "SinricProSwitch.h"

#define WIFI_SSID         "WIFI_NAME"
#define WIFI_PASS         "WIFI_PASSWORD"
#define APP_KEY           "APP_KEY"
#define APP_SECRET        "APP_SECRET"

#define SWITCH_ID_1       "DEVICE_ID_1"
#define SWITCH_ID_2       "DEVICE_ID_2"
#define SWITCH_ID_3       "DEVICE_ID_3"

// Pins for the Relays
#define RELAYPIN_1        26
#define RELAYPIN_2        27
#define RELAYPIN_3        32

// Pins for the Physical Wall Switches
#define WALL_SWITCH_1     13
#define WALL_SWITCH_2     14
#define WALL_SWITCH_3     25

// Variables to track the physical switch positions
bool lastState1 = false;
bool lastState2 = false;
bool lastState3 = false;

bool onPowerState1(const String &deviceId, bool &state) {
  digitalWrite(RELAYPIN_1, state ? LOW : HIGH);
  return true;
}
bool onPowerState2(const String &deviceId, bool &state) {
  digitalWrite(RELAYPIN_2, state ? LOW : HIGH);
  return true;
}
bool onPowerState3(const String &deviceId, bool &state) {
  digitalWrite(RELAYPIN_3, state ? LOW : HIGH);
  return true;
}

void handleWallSwitches() {
  // Check Switch 1
  bool currentState1 = digitalRead(WALL_SWITCH_1);
  if (currentState1 != lastState1) {
    lastState1 = currentState1;
    bool isRelayOn = (digitalRead(RELAYPIN_1) == LOW);
    digitalWrite(RELAYPIN_1, isRelayOn ? HIGH : LOW); // Flip the light
    
    // Explicitly declare as a Switch before sending the event
    SinricProSwitch& mySwitch1 = SinricPro[SWITCH_ID_1];
    mySwitch1.sendPowerStateEvent(!isRelayOn); 
    
    delay(150); 
  }

  // Check Switch 2
  bool currentState2 = digitalRead(WALL_SWITCH_2);
  if (currentState2 != lastState2) {
    lastState2 = currentState2;
    bool isRelayOn = (digitalRead(RELAYPIN_2) == LOW);
    digitalWrite(RELAYPIN_2, isRelayOn ? HIGH : LOW);
    
    SinricProSwitch& mySwitch2 = SinricPro[SWITCH_ID_2];
    mySwitch2.sendPowerStateEvent(!isRelayOn);
    
    delay(150);
  }

  // Check Switch 3
  bool currentState3 = digitalRead(WALL_SWITCH_3);
  if (currentState3 != lastState3) {
    lastState3 = currentState3;
    bool isRelayOn = (digitalRead(RELAYPIN_3) == LOW);
    digitalWrite(RELAYPIN_3, isRelayOn ? HIGH : LOW);
    
    SinricProSwitch& mySwitch3 = SinricPro[SWITCH_ID_3];
    mySwitch3.sendPowerStateEvent(!isRelayOn);
    
    delay(150);
  }
}

void setup() {
  pinMode(RELAYPIN_1, OUTPUT);
  pinMode(RELAYPIN_2, OUTPUT);
  pinMode(RELAYPIN_3, OUTPUT);

  // Set switch pins to use internal pull-up resistors
  pinMode(WALL_SWITCH_1, INPUT_PULLUP);
  pinMode(WALL_SWITCH_2, INPUT_PULLUP);
  pinMode(WALL_SWITCH_3, INPUT_PULLUP);

  digitalWrite(RELAYPIN_1, HIGH);
  digitalWrite(RELAYPIN_2, HIGH);
  digitalWrite(RELAYPIN_3, HIGH);

  lastState1 = digitalRead(WALL_SWITCH_1);
  lastState2 = digitalRead(WALL_SWITCH_2);
  lastState3 = digitalRead(WALL_SWITCH_3);

  WiFi.begin(WIFI_SSID, WIFI_PASS);

  SinricProSwitch& mySwitch1 = SinricPro[SWITCH_ID_1];
  mySwitch1.onPowerState(onPowerState1);

  SinricProSwitch& mySwitch2 = SinricPro[SWITCH_ID_2];
  mySwitch2.onPowerState(onPowerState2);

  SinricProSwitch& mySwitch3 = SinricPro[SWITCH_ID_3];
  mySwitch3.onPowerState(onPowerState3);

  SinricPro.begin(APP_KEY, APP_SECRET);
}

void loop() {
  SinricPro.handle();
  handleWallSwitches();
}
