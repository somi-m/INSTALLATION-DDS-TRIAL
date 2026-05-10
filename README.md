#include <AccelStepper.h>

#define DRIVER 1

// ---------------- MOTOR PINS ----------------

// Motor A
#define STEP_A 2
#define DIR_A 5

// Motor B
#define STEP_B 3
#define DIR_B 6

// Motor C
#define STEP_C 4
#define DIR_C 7

// ---------------- TRIGGER BUTTON ----------------

#define TRIGGER_PIN 8

// ---------------- MOTOR OBJECTS ----------------

AccelStepper motorA(DRIVER, STEP_A, DIR_A);
AccelStepper motorB(DRIVER, STEP_B, DIR_B);
AccelStepper motorC(DRIVER, STEP_C, DIR_C);

// ---------------- SETTINGS ----------------

// Motion values
long descendAB = 3200;
long descendC  = 2400;

// Speed settings
float maxSpeed = 1000;
float acceleration = 500;

// Hold time at bottom
unsigned long holdTime = 4000;

// ---------------- STATE VARIABLES ----------------

bool runningSequence = false;
bool movingDown = false;
bool waitingAtBottom = false;
bool movingUp = false;

unsigned long waitStart = 0;

// --------------------------------------------------

void setup() {

  pinMode(TRIGGER_PIN, INPUT_PULLUP);

  motorA.setMaxSpeed(maxSpeed);
  motorA.setAcceleration(acceleration);

  motorB.setMaxSpeed(maxSpeed);
  motorB.setAcceleration(acceleration);

  motorC.setMaxSpeed(maxSpeed);
  motorC.setAcceleration(acceleration);
}

// --------------------------------------------------

void loop() {

  motorA.run();
  motorB.run();
  motorC.run();

  // -------- START SEQUENCE --------

  if (!runningSequence && digitalRead(TRIGGER_PIN) == LOW) {

    delay(50);

    if (digitalRead(TRIGGER_PIN) == LOW) {

      runningSequence = true;
      movingDown = true;

      motorA.moveTo(descendAB);
      motorB.moveTo(descendAB);
      motorC.moveTo(descendC);

      while (digitalRead(TRIGGER_PIN) == LOW) {
      }
    }
  }

  // -------- DESCENDING COMPLETE --------

  if (movingDown &&
      motorA.distanceToGo() == 0 &&
      motorB.distanceToGo() == 0 &&
      motorC.distanceToGo() == 0) {

    movingDown = false;
    waitingAtBottom = true;
    waitStart = millis();
  }

  // -------- HOLD AT BOTTOM --------

  if (waitingAtBottom &&
      millis() - waitStart >= holdTime) {

    waitingAtBottom = false;
    movingUp = true;

    motorA.moveTo(0);
    motorB.moveTo(0);
    motorC.moveTo(0);
  }

  // -------- ASCENDING COMPLETE --------

  if (movingUp &&
      motorA.distanceToGo() == 0 &&
      motorB.distanceToGo() == 0 &&
      motorC.distanceToGo() == 0) {

    movingUp = false;
    runningSequence = false;
  }
}
