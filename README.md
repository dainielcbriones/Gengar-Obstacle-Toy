#include <SPI.h>
#include <math.h>

#include <Adafruit_GFX.h>
#include <Adafruit_GC9A01A.h>

#include <WiFi.h>
#include <Trioe.h>
#include <TRIOE_ROOT_CA.h>

// Replace with your actual Wi-Fi and TRIOE Hub credentials before uploading
const char* WIFI_SSID     = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";

const char* TELEMETRY_URL = "https://hub.trioe.dev/api/v1/devices/YOUR_DEVICE_ID/telemetry/";
const char* API_KEY       = "YOUR_TRIOE_API_KEY";

TrioeClient trioe(
  TELEMETRY_URL,
  API_KEY
);

float latitude  = 14.5995f;
float longitude = 120.9842f;

#define TRIG_PIN 1
#define ECHO_PIN 2

const float MAX_DETECTION_DISTANCE = 2.00f;

#define TFT_CS   7
#define TFT_DC   6
#define TFT_RST  8
#define TFT_MOSI 5
#define TFT_SCLK 4

Adafruit_GC9A01A tft(
  TFT_CS,
  TFT_DC,
  TFT_RST
);

GFXcanvas16 canvas(240, 240);

#define BLACK       0x0000
#define WHITE       0xFFFF
#define RED_EYE     0xF800
#define RED_LIGHT   0xFBE0
#define DARK_PURPLE 0x3008

enum FaceState {
  FACE_IDLE,
  FACE_CALM,
  FACE_ALERT,
  FACE_ANGRY,
  FACE_RAGE
};

FaceState faceState = FACE_IDLE;

unsigned long lastSensorRead    = 0;
unsigned long lastDisplayUpdate = 0;
unsigned long lastTelemetry     = 0;
unsigned long lastSerial        = 0;

const unsigned long SENSOR_INTERVAL    = 100;
const unsigned long DISPLAY_INTERVAL   = 30;
const unsigned long TELEMETRY_INTERVAL = 1000;
const unsigned long SERIAL_INTERVAL    = 500;

float distanceMeters = -1.0f;
bool objectDetected  = false;

float eyeMoveX = 0;
float eyeMoveY = 0;

float targetEyeX = 0;
float targetEyeY = 0;

float mouthOpen       = 0;
float targetMouthOpen = 0;

float faceOffset = 0;

unsigned long lastEyeTarget = 0;
unsigned long nextEyeTarget = 1500;

unsigned long lastBlink = 0;
unsigned long nextBlink = 4000;

bool blinking = false;

enum IdleEyeAction {
  IDLE_LOOK,
  IDLE_THINK,
  IDLE_CENTER,
  IDLE_QUICK_GLANCE
};

IdleEyeAction idleAction = IDLE_LOOK;

unsigned long idleActionStart    = 0;
unsigned long idleActionDuration = 1500;

int idleLookX = 0;
int idleLookY = 0;

bool idleLookingRight = false;

const uint8_t rageSymbolBitmap[] PROGMEM = {
  0x00, 0x7C, 0x00, 0x00,
  0x00, 0x7C, 0x3E, 0x00,
  0x00, 0x7C, 0x3E, 0x00,
  0x00, 0x7C, 0x3E, 0x00,
  0x00, 0x7C, 0x3E, 0x00,
  0x00, 0x7C, 0x3E, 0x00,
  0x00, 0xFC, 0x3E, 0x00,
  0x00, 0xFC, 0x3F, 0x00,
  0x01, 0xF8, 0x3F, 0x00,
  0x07, 0xF8, 0x1F, 0x80,
  0x1F, 0xF0, 0x1F, 0xE0,
  0xFF, 0xE0, 0x0F, 0xFC,
  0xFF, 0xC0, 0x07, 0xFF,
  0xFF, 0x80, 0x03, 0xFF,
  0xFE, 0x00, 0x00, 0xFF,
  0xF0, 0x00, 0x00, 0x3F,
  0x00, 0x00, 0x00, 0x07,
  0x00, 0x3F, 0x80, 0x00,
  0x01, 0xFF, 0xE0, 0x00,
  0x07, 0xFF, 0xF8, 0x00,
  0x0F, 0xFF, 0xFC, 0x00,
  0x0F, 0xFF, 0xFE, 0x00,
  0x07, 0xE1, 0xFE, 0x00,
  0x03, 0x00, 0x7F, 0x00,
  0x00, 0x00, 0x1F, 0x00,
  0x00, 0x00, 0x1E, 0x00,
  0x00, 0x00, 0x08, 0x00
};

struct PointF {
  float x;
  float y;
};

PointF bezierPoint(
  PointF p0,
  PointF p1,
  PointF p2,
  float t
) {
  float u = 1.0f - t;
  PointF p;
  p.x = u * u * p0.x + 2.0f * u * t * p1.x + t * t * p2.x;
  p.y = u * u * p0.y + 2.0f * u * t * p1.y + t * t * p2.y;
  return p;
}

void smoothCurve(
  PointF p0,
  PointF p1,
  PointF p2,
  uint16_t color,
  int thickness
) {
  PointF previous = p0;

  for (int i = 1; i <= 30; i++) {
    float t = (float)i / 30.0f;
    PointF current = bezierPoint(p0, p1, p2, t);

    for (int k = 0; k < thickness; k++) {
      canvas.drawLine(
        previous.x,
        previous.y + k,
        current.x,
        current.y + k,
        color
      );
    }
    previous = current;
  }
}

void fillBezierBand(
  PointF a0,
  PointF a1,
  PointF a2,
  PointF b0,
  PointF b1,
  PointF b2,
  uint16_t color
) {
  const int steps = 40;

  for (int i = 0; i < steps; i++) {
    float t1 = (float)i / steps;
    float t2 = (float)(i + 1) / steps;

    PointF p1 = bezierPoint(a0, a1, a2, t1);
    PointF p2 = bezierPoint(a0, a1, a2, t2);
    PointF q1 = bezierPoint(b0, b1, b2, t1);
    PointF q2 = bezierPoint(b0, b1, b2, t2);

    canvas.fillTriangle(
      p1.x, p1.y,
      p2.x, p2.y,
      q1.x, q1.y,
      color
    );

    canvas.fillTriangle(
      p2.x, p2.y,
      q1.x, q1.y,
      q2.x, q2.y,
      color
    );
  }
}

void drawSmoothEllipse(
  int cx,
  int cy,
  int rx,
  int ry,
  uint16_t color
) {
  for (int y = -ry; y <= ry; y++) {
    float yy = (float)y / (float)ry;
    float xx = rx * sqrtf(fmaxf(0.0f, 1.0f - yy * yy));

    canvas.drawFastHLine(
      cx - (int)xx,
      cy + y,
      (int)(2.0f * xx),
      color
    );
  }
}

void drawLeftEye(int ox, int oy) {
  fillBezierBand(
    {20 + ox, 82 + oy},
    {43 + ox, 48 + oy},
    {96 + ox, 69 + oy},
    {24 + ox, 91 + oy},
    {48 + ox, 103 + oy},
    {96 + ox, 73 + oy},
    BLACK
  );

  fillBezierBand(
    {27 + ox, 83 + oy},
    {47 + ox, 57 + oy},
    {90 + ox, 70 + oy},
    {29 + ox, 89 + oy},
    {49 + ox, 97 + oy},
    {90 + ox, 73 + oy},
    RED_EYE
  );

  drawSmoothEllipse(61 + ox, 76 + oy, 22, 5, RED_LIGHT);
  drawSmoothEllipse(68 + ox, 78 + oy, 6, 16, BLACK);
  canvas.fillCircle(64 + ox, 73 + oy, 3, WHITE);

  smoothCurve(
    {20 + ox, 82 + oy},
    {43 + ox, 48 + oy},
    {96 + ox, 69 + oy},
    BLACK,
    6
  );

  smoothCurve(
    {24 + ox, 91 + oy},
    {48 + ox, 103 + oy},
    {96 + ox, 73 + oy},
    BLACK,
    4
  );
}

void drawRightEye(int ox, int oy) {
  fillBezierBand(
    {144 + ox, 69 + oy},
    {197 + ox, 48 + oy},
    {220 + ox, 82 + oy},
    {144 + ox, 73 + oy},
    {192 + ox, 103 + oy},
    {216 + ox, 91 + oy},
    BLACK
  );

  fillBezierBand(
    {150 + ox, 70 + oy},
    {193 + ox, 57 + oy},
    {213 + ox, 83 + oy},
    {150 + ox, 73 + oy},
    {191 + ox, 97 + oy},
    {211 + ox, 89 + oy},
    RED_EYE
  );

  drawSmoothEllipse(179 + ox, 76 + oy, 22, 5, RED_LIGHT);
  drawSmoothEllipse(172 + ox, 78 + oy, 6, 16, BLACK);
  canvas.fillCircle(176 + ox, 73 + oy, 3, WHITE);

  smoothCurve(
    {144 + ox, 69 + oy},
    {197 + ox, 48 + oy},
    {220 + ox, 82 + oy},
    BLACK,
    6
  );

  smoothCurve(
    {144 + ox, 73 + oy},
    {192 + ox, 103 + oy},
    {216 + ox, 91 + oy},
    BLACK,
    4
  );
}

void drawGengarEyes(int ox, int oy) {
  if (blinking) {
    canvas.fillRect(27 + ox, 86 + oy, 66, 5, BLACK);
    canvas.fillRect(147 + ox, 86 + oy, 66, 5, BLACK);
    return;
  }

  drawLeftEye(ox, oy);
  drawRightEye(ox, oy);

  if (
    faceState == FACE_CALM ||
    faceState == FACE_ALERT ||
    faceState == FACE_ANGRY ||
    faceState == FACE_RAGE
  ) {
    int browThickness = 7;

    if (faceState == FACE_ALERT) {
      browThickness = 8;
    } else if (faceState == FACE_ANGRY) {
      browThickness = 10;
    } else if (faceState == FACE_RAGE) {
      browThickness = 12;
    }

    canvas.fillTriangle(
      18 + ox, 48 + oy,
      96 + ox, 61 + oy,
      94 + ox, 61 + oy + browThickness,
      BLACK
    );

    canvas.fillTriangle(
      18 + ox, 48 + oy,
      94 + ox, 61 + oy + browThickness,
      16 + ox, 56 + oy,
      BLACK
    );

    canvas.fillTriangle(
      144 + ox, 61 + oy,
      222 + ox, 48 + oy,
      224 + ox, 56 + oy,
      BLACK
    );

    canvas.fillTriangle(
      146 + ox, 61 + oy,
      224 + ox, 56 + oy,
      146 + ox, 61 + oy + browThickness,
      BLACK
    );

    if (faceState == FACE_ANGRY || faceState == FACE_RAGE) {
      canvas.fillTriangle(
        86 + ox, 58 + oy,
        108 + ox, 67 + oy,
        91 + ox, 71 + oy,
        BLACK
      );

      canvas.fillTriangle(
        154 + ox, 67 + oy,
        176 + ox, 58 + oy,
        169 + ox, 71 + oy,
        BLACK
      );
    }

    if (faceState == FACE_RAGE) {
      canvas.fillTriangle(
        28 + ox, 55 + oy,
        58 + ox, 42 + oy,
        91 + ox, 58 + oy,
        BLACK
      );

      canvas.fillTriangle(
        149 + ox, 58 + oy,
        182 + ox, 42 + oy,
        212 + ox, 55 + oy,
        BLACK
      );
    }
  }
}

void drawRageSymbol(int ox, int oy) {
  if (faceState != FACE_RAGE) {
    return;
  }

  const int symbolWidth  = 32;
  const int symbolHeight = 27;
  const int symbolX = 50;
  const int symbolY = 20;

  canvas.drawBitmap(
    symbolX + ox,
    symbolY + oy,
    rageSymbolBitmap,
    symbolWidth,
    symbolHeight,
    RED_EYE,
    DARK_PURPLE
  );
}

void drawMouth(float openness, int ox, int oy) {
  int cx = 120 + ox;
  int cy = 160 + oy;

  int width  = 62 + (int)(35.0f * openness);
  int height = 20 + (int)(45.0f * openness);

  canvas.fillRoundRect(
    cx - width,
    cy - height / 2,
    width * 2,
    height,
    18,
    BLACK
  );

  canvas.fillRoundRect(
    cx - width + 7,
    cy - height / 2 + 7,
    width * 2 - 14,
    height - 14,
    13,
    DARK_PURPLE
  );

  int toothCount = 7;

  for (int i = 0; i < toothCount; i++) {
    float p = (float)i / (float)(toothCount - 1);
    int tx = cx - width + 13 + (int)(p * (width * 2 - 26));
    int ty = cy - height / 2 + 7;
    int toothHeight = 9 + (int)(13.0f * openness);

    canvas.fillTriangle(
      tx - 5, ty,
      tx + 5, ty,
      tx, ty + toothHeight,
      WHITE
    );
  }

  if (openness > 0.55f) {
    for (int i = 0; i < 5; i++) {
      float p = (float)i / 4.0f;
      int tx = cx - width + 18 + (int)(p * (width * 2 - 36));
      int ty = cy + height / 2 - 7;

      canvas.fillTriangle(
        tx - 4, ty,
        tx + 4, ty,
        tx, ty - 11,
        WHITE
      );
    }
  }
}

void drawFace() {
  canvas.fillScreen(DARK_PURPLE);

  int ox = (int)roundf(eyeMoveX + faceOffset);
  int oy = (int)roundf(eyeMoveY + sinf(millis() * 0.003f) * 2.0f);

  drawGengarEyes(ox, oy);
  drawRageSymbol(ox, oy);
  drawMouth(mouthOpen, ox, oy);

  canvas.fillCircle(120 + ox, 125 + oy, 4, BLACK);

  tft.drawRGBBitmap(0, 0, canvas.getBuffer(), 240, 240);
}

float readDistanceMeters() {
  const int samples = 5;
  float readings[samples];

  for (int i = 0; i < samples; i++) {
    digitalWrite(TRIG_PIN, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG_PIN, LOW);

    unsigned long duration = pulseIn(ECHO_PIN, HIGH, 30000UL);

    if (duration == 0) {
      readings[i] = -1.0f;
    } else {
      float cm = (duration * 0.0343f) / 2.0f;
      if (cm >= 2.0f && cm <= 400.0f) {
        readings[i] = cm / 100.0f;
      } else {
        readings[i] = -1.0f;
      }
    }
    delayMicroseconds(500);
  }

  for (int i = 0; i < samples - 1; i++) {
    for (int j = i + 1; j < samples; j++) {
      float ai = readings[i] < 0.0f ? 999.0f : readings[i];
      float aj = readings[j] < 0.0f ? 999.0f : readings[j];

      if (aj < ai) {
        float temp = readings[i];
        readings[i] = readings[j];
        readings[j] = temp;
      }
    }
  }

  int validCount = 0;
  for (int i = 0; i < samples; i++) {
    if (readings[i] >= 0.0f) {
      validCount++;
    }
  }

  if (validCount == 0) {
    objectDetected = false;
    return -1.0f;
  }

  float distance = readings[(validCount - 1) / 2];

  if (distance >= 0.02f && distance <= MAX_DETECTION_DISTANCE) {
    objectDetected = true;
    return distance;
  }

  objectDetected = false;
  return -1.0f;
}

void updateFaceState() {
  if (
    !objectDetected ||
    distanceMeters <= 0.0f ||
    distanceMeters > MAX_DETECTION_DISTANCE
  ) {
    faceState = FACE_IDLE;
    targetMouthOpen = 0.0f;
    return;
  }

  if (distanceMeters <= 0.15f) {
    faceState = FACE_RAGE;
    targetMouthOpen = 1.0f;
  } else if (distanceMeters <= 0.30f) {
    faceState = FACE_ANGRY;
    targetMouthOpen = 0.85f;
  } else if (distanceMeters <= 0.50f) {
    faceState = FACE_ALERT;
    targetMouthOpen = 0.65f;
  } else if (distanceMeters <= 1.50f) {
    faceState = FACE_CALM;
    targetMouthOpen = 0.35f;
  } else if (distanceMeters <= 2.00f) {
    faceState = FACE_CALM;
    targetMouthOpen = 0.20f;
  } else {
    faceState = FACE_IDLE;
    targetMouthOpen = 0.0f;
  }
}

const char* getReaction() {
  if (
    !objectDetected ||
    distanceMeters <= 0.0f ||
    distanceMeters > MAX_DETECTION_DISTANCE
  ) {
    return "IDLE";
  }

  if (distanceMeters <= 0.15f) {
    return "RAGE";
  }

  if (distanceMeters <= 0.30f) {
    return "ANGRY";
  }

  if (distanceMeters <= 0.50f) {
    return "ALERT";
  }

  return "CALM";
}

const char* getObjectStatus() {
  if (
    objectDetected &&
    distanceMeters >= 0.02f &&
    distanceMeters <= MAX_DETECTION_DISTANCE
  ) {
    return "OBJECT DETECTED";
  }

  return "NO OBJECT";
}

void startIdleEyeAction(unsigned long now) {
  int action = random(0, 100);

  if (action < 55) {
    idleAction = IDLE_LOOK;
    int direction = random(0, 4);

    if (direction == 0) {
      idleLookX = random(-13, -6);
      idleLookY = random(-4, 5);
    } else if (direction == 1) {
      idleLookX = random(6, 14);
      idleLookY = random(-4, 5);
    } else if (direction == 2) {
      idleLookX = random(-5, 6);
      idleLookY = random(-8, -3);
    } else {
      idleLookX = random(-5, 6);
      idleLookY = random(3, 8);
    }

    idleActionDuration = random(700, 1800);
  } else if (action < 78) {
    idleAction = IDLE_THINK;

    if (idleLookingRight) {
      idleLookX = random(7, 14);
      idleLookY = random(-8, -2);
    } else {
      idleLookX = random(-14, -6);
      idleLookY = random(-8, -2);
    }

    idleLookingRight = !idleLookingRight;
    idleActionDuration = random(1000, 2200);
  } else if (action < 93) {
    idleAction = IDLE_CENTER;
    idleLookX = 0;
    idleLookY = 0;
    idleActionDuration = random(500, 1200);
  } else {
    idleAction = IDLE_QUICK_GLANCE;

    if (random(0, 2) == 0) {
      idleLookX = random(-16, -10);
    } else {
      idleLookX = random(10, 17);
    }

    idleLookY = random(-3, 4);
    idleActionDuration = random(250, 550);
  }

  idleActionStart = now;
}

void updateAnimation() {
  unsigned long now = millis();

  mouthOpen += (targetMouthOpen - mouthOpen) * 0.12f;

  if (objectDetected) {
    if (now - lastEyeTarget > nextEyeTarget) {
      lastEyeTarget = now;
      nextEyeTarget = random(800, 2200);

      if (faceState == FACE_RAGE) {
        targetEyeX = random(-14, 15);
        targetEyeY = random(-8, 9);
      } else if (faceState == FACE_ANGRY) {
        targetEyeX = random(-10, 11);
        targetEyeY = random(-6, 7);
      } else {
        targetEyeX = random(-8, 9);
        targetEyeY = random(-5, 6);
      }
    }

    if (faceState == FACE_RAGE) {
      faceOffset = sinf(now * 0.012f) * 4.0f;
    } else if (faceState == FACE_ANGRY) {
      faceOffset = sinf(now * 0.008f) * 3.2f;
    } else {
      faceOffset = sinf(now * 0.004f) * 2.5f;
    }
  } else {
    faceOffset = sinf(now * 0.0015f) * 1.0f;

    if (now - idleActionStart >= idleActionDuration) {
      startIdleEyeAction(now);
    }

    targetEyeX = idleLookX;
    targetEyeY = idleLookY;
  }

  float eyeSpeed = objectDetected ? 0.08f : 0.055f;

  eyeMoveX += (targetEyeX - eyeMoveX) * eyeSpeed;
  eyeMoveY += (targetEyeY - eyeMoveY) * eyeSpeed;

  if (!blinking && now - lastBlink > nextBlink) {
    blinking = true;
    lastBlink = now;
    nextBlink = random(2500, 6000);
  }

  if (blinking && now - lastBlink > 120) {
    blinking = false;
  }
}

void updateTelemetry() {
  float d = distanceMeters;

  if (
    !objectDetected ||
    d < 0.0f ||
    d > MAX_DETECTION_DISTANCE
  ) {
    d = 0.0f;
  }

  trioe.addReading("distance", d, "meters");
  trioe.addReading("reaction", getReaction());
  trioe.addReading("object", getObjectStatus());
}

void printStatus() {
  Serial.println("==============================");
  Serial.print("Distance: ");

  if (
    objectDetected &&
    distanceMeters >= 0.02f &&
    distanceMeters <= MAX_DETECTION_DISTANCE
  ) {
    Serial.print(distanceMeters, 2);
    Serial.println(" m");
  } else {
    Serial.println("NO OBJECT");
  }

  Serial.print("Reaction: ");
  Serial.println(getReaction());

  Serial.print("Object: ");
  Serial.println(getObjectStatus());

  Serial.print("WiFi: ");
  Serial.println(WiFi.status() == WL_CONNECTED ? "CONNECTED" : "DISCONNECTED");

  Serial.println("TRIOE: distance + reaction + object");
  Serial.println("Detection limit: 2.00 meters");
  Serial.println("==============================");
}

void setup() {
  Serial.begin(115200);
  delay(500);

  Serial.println();
  Serial.println("Starting Gengar TRIOE System...");

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  digitalWrite(TRIG_PIN, LOW);

  SPI.begin(TFT_SCLK, -1, TFT_MOSI, TFT_CS);

  tft.begin();
  tft.setRotation(0);
  tft.fillScreen(DARK_PURPLE);

  canvas.fillScreen(DARK_PURPLE);

  randomSeed(analogRead(0));

  idleAction = IDLE_THINK;
  idleActionStart = millis();
  idleActionDuration = 1200;

  idleLookX = -8;
  idleLookY = -5;

  targetEyeX = idleLookX;
  targetEyeY = idleLookY;

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to WiFi");

  unsigned long start = millis();

  while (WiFi.status() != WL_CONNECTED && millis() - start < 15000) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("WiFi connected");
    Serial.print("IP: ");
    Serial.println(WiFi.localIP());
  } else {
    Serial.println("WiFi connection failed");
  }

  trioe.setCACert(TRIOE_ROOT_CA);
  trioe.setReportingInterval(1000);
  trioe.setChangeThreshold(0.2f);

  if (!trioe.begin()) {
    Serial.println("TRIOE setup failed.");
  } else {
    Serial.println("TRIOE initialized.");
  }

  objectDetected = false;
  distanceMeters = -1.0f;
  faceState = FACE_IDLE;

  drawFace();

  Serial.println("Gengar system ready.");
  Serial.println("Detection range: 0.02m - 2.00m");
  Serial.println("RAGE <= 0.15m");
  Serial.println("RAGE SYMBOL: RED + SHAKE");
}

void loop() {
  unsigned long now = millis();

  if (now - lastSensorRead >= SENSOR_INTERVAL) {
    lastSensorRead = now;
    distanceMeters = readDistanceMeters();
    updateFaceState();
  }

  if (now - lastDisplayUpdate >= DISPLAY_INTERVAL) {
    lastDisplayUpdate = now;
    updateAnimation();
    drawFace();
  }

  if (now - lastTelemetry >= TELEMETRY_INTERVAL) {
    lastTelemetry = now;

    if (WiFi.status() == WL_CONNECTED) {
      updateTelemetry();
    }
  }

  trioe.loop();

  if (now - lastSerial >= SERIAL_INTERVAL) {
    lastSerial = now;
    printStatus();
  }
}
