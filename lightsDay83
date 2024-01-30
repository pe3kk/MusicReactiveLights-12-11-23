#include <Audio.h>
#include <FastLED.h>
#include <deque>
#include <AudioStream.h>

#define NUM_LEDS  120
#define NUM_LEDS_T_and_B 36
#define NUM_LEDS_MID 60

#define LED_PIN0   3
#define LED_PIN1   29
#define LED_PIN2   26
#define LED_PIN3   31
#define LED_PIN4   30
#define LED_PIN5   28

#define MODE_INTERVAL 7000
#define INTERVAL_POT 300

#define BUTTON 32
#define BUTTON1 33
#define potPin 24
#define potPin1 25

CRGB leds0[NUM_LEDS_MID];
CRGB leds4[NUM_LEDS];
CRGB leds5[NUM_LEDS];

CRGBArray<NUM_LEDS> leds1;
CRGBSet leds_11(leds1, 0,                                                                    NUM_LEDS/2);
CRGBSet leds_12(leds1, NUM_LEDS/2,                                                           NUM_LEDS);

CRGBArray<NUM_LEDS_T_and_B> leds2;
CRGBSet leds_21(leds2, 0,                                                                    NUM_LEDS_T_and_B/3);
CRGBSet leds_22(leds2, NUM_LEDS_T_and_B/3+1,                                                           (NUM_LEDS_T_and_B/3)*2);
CRGBSet leds_23(leds2, (NUM_LEDS_T_and_B/3)*2,                                                           NUM_LEDS_T_and_B);

CRGBArray<NUM_LEDS_T_and_B> leds3;
CRGBSet leds_31(leds3, 0,                                                                    NUM_LEDS_T_and_B/3);
CRGBSet leds_32(leds3, NUM_LEDS_T_and_B/3+1,                                                           (NUM_LEDS_T_and_B/3)*2);
CRGBSet leds_33(leds3, (NUM_LEDS_T_and_B/3)*2,                                                           NUM_LEDS_T_and_B);

const int colorSets = 4;
const int numArrays = 6;
const int arrayLengths[numArrays] = {5, 16, 41, 181, 278, 463}; // Set the lengths of the arrays
CHSV colorArrays[numArrays][463];
CHSV colorArrays1[colorSets][numArrays][463];

int setsRange = 224 / colorSets;

//const int myInput = AUDIO_INPUT_MIC;
const int myInput = AUDIO_INPUT_LINEIN;

AudioInputI2S          audioInput;
AudioMixer4            mixer;
AudioAnalyzeFFT1024    fft;
AudioOutputI2S         audioOutput;


// AudioConnection myConnection(source, sourcePort, destination, destinationPort);
AudioConnection patchCord1(audioInput, 0, mixer, 0);
AudioConnection patchCord2(audioInput, 1, mixer, 1);
AudioConnection patchCord3(mixer, fft);             // using mixer for stereo line-in connection

//AudioConnection patchCord1(sinewave, 0, myFFT, 0);
AudioControlSGTL5000 audioShield;


/*This isn't a problem in your code, because arraySize is never negative. But the compiler doesn't know that; 
it's warning you about the potential problem in case arraySize were negative. By changing arraySize to unsigned int, 
you're making it clear to the compiler that arraySize is always non-negative, so the comparison with buffer.size() 
is safe and the warning goes away. */

const unsigned int arraySize = 250;                         //related to deque container
const unsigned int arraySizeSh = 50;                        //related to deque container

int newVal = 0;                                     //related to potentiometers
int val = 0;                                        //related to potentiometers
int k = 0;                                          //variable switches subArray if colorArrays1 is in use

float subBassAvg, bassAvg, midAvg, highAvg, presenceAvg, brillianceAvg;
int subBassAvgCnt = 0, bassAvgCnt = 0, midAvgCnt = 0, highAvgCnt = 0, presenceAvgCnt = 0, brillianceAvgCnt = 0;
float subBassAvgOut = 0, bassAvgOut = 0, midAvgOut = 0, highAvgOut = 0, presenceAvgOut = 0, brillianceAvgOut = 0;

float subBassAvgSh, bassAvgSh, midAvgSh, highAvgSh, presenceAvgSh, brillianceAvgSh;
int subBassAvgCntSh = 0, bassAvgCntSh = 0, midAvgCntSh = 0, highAvgCntSh = 0, presenceAvgCntSh = 0, brillianceAvgCntSh = 0;
float subBassAvgOutSh = 0, bassAvgOutSh = 0, midAvgOutSh = 0, highAvgOutSh = 0, presenceAvgOutSh = 0, brillianceAvgOutSh = 0;

int subbassBh = 0, bassBh = 0, midBh = 0, highBh = 0, presenceBh = 0, brillianceBh = 0;
int subbassCnt = 0, bassCnt = 0, midCnt = 0, highCnt = 0, presenceCnt = 0, brillianceCnt = 0;
int subBassBin = 0, bassBin = 0, midBin = 0, highBin = 0, presenceBin = 0, brillianceBin = 0;



unsigned long previousMillis = 0;
unsigned long previousMillisPot = 0;
unsigned long buttonPressTime = 0;

int preButtonState = 1;
int curButtonState = 0;
int preButtonState0 = 1;
int curButtonState0 = 0;
int preButtonState1 = 1;
int curButtonState1 = 0;

int fadeAmount = 3;

int color = 0;
int mode = 0;

int potentiometerBrigh = 0;
int potentiometerSens = 0;


float shortVsLong(float avgOutSh, float& avgOut)  {
  if (avgOutSh * 0.85 > avgOut || avgOutSh * 1.05 < avgOut) {
    avgOut = avgOutSh;
  }
  return avgOut;
}

void buttonColPress(void) {
  curButtonState0 = digitalRead(BUTTON1);
  if (curButtonState0 != preButtonState0) {
    if (curButtonState0 == LOW) {
      buttonPressTime = millis();
    }
    else {
      unsigned long buttonDuration = millis() - buttonPressTime;

      if (buttonDuration >= MODE_INTERVAL) {
        //Serial.println("Button held for 1 second");
        color = (color + 1) % 2;
        k -= 1;
      }
    }
    preButtonState0 = curButtonState0;
  }
}

void buttonPress(int& preButtonState, int& curButtonState, int& state, const int lastVal, const int BUTT) {
  int newButtonState = digitalRead(BUTT);
  if (preButtonState == HIGH && newButtonState == LOW) {
    curButtonState = newButtonState;
    state++;
  }
  else if (preButtonState == LOW && newButtonState == HIGH) {
    if (state) {
      //Serial.println("Button was pressed");
      if (state >= lastVal) {
        state = 0;
      }
    }
    curButtonState = newButtonState;
  }
  preButtonState = curButtonState;
}

/*void potBrigh() {
  int newVal = analogRead(potPin);
  if (newVal >= val + 5  || newVal <= val - 5 )  {
    FastLED.setBrightness((newVal / 4) -1);
    //Serial.println(newVal);
  }
  val = newVal;
}

void potSens() {
  int newVal = analogRead(potPin1);
  if (newVal >= val + 5  || newVal <= val - 5 )  {
    audioShield.lineInLevel((newVal / 100) + 2);
    //Serial.println(newVal);
  }
  val = newVal;
}*/

void potentiometer(int pin, int& currPotVal) {
  currPotVal = analogRead(pin);
  if (currPotVal >= val + 5  || currPotVal <= val - 5 )  {
    if (pin == 25) {
      FastLED.setBrightness((currPotVal / 4) -1);
    } else {
      audioShield.lineInLevel((currPotVal / 100) + 2);
    }
  }
  val = currPotVal;
}

class Leds {
private:
  int numLeds;
  CRGB* leds;
  int arrayIndex;
  int position;
  int moveBy;
  
public:
  Leds(int initialNumLeds, CRGB* initialLeds, int initialArrayIndex)
    : numLeds(initialNumLeds), leds(initialLeds), arrayIndex(initialArrayIndex)
  {
    position = 1;
    moveBy = 4;
  }

  void updateLEDs(float val, float valAvgOut, float maxVal, float threshold, int& brightness, int colorIndex) {
    switch (mode) {
      case 0:
        if ((val >= valAvgOut * 1.05 || maxVal * 0.9 >= valAvgOut || val > valAvgOut * 0.9) && maxVal > threshold) {
          brightness = 255;
          this->setColor(colorIndex);
        } else {
          this->fadeDown(brightness);
        }
        break;

      case 1:
        if ((val >= valAvgOut * 1.05 || maxVal * 0.9 >= valAvgOut || val > valAvgOut * 0.8) && maxVal > threshold) {
          this->setColorIrregular(colorIndex);
        } else {
          this->setBlack();
        }
        break;

      case 2:
        if ((val >= valAvgOut * 1.05 || maxVal * 0.9 >= valAvgOut || val > valAvgOut * 0.8) && maxVal > threshold) {
          this->setColor(colorIndex);
        } else {
          this->setBlack();
        }
        break;

      case 3:
        if ((val >= valAvgOut * 1.05 || maxVal * 0.9 >= valAvgOut || val > valAvgOut * 0.8) && maxVal > threshold) {
          brightness = 125;
          this->fadeUp(brightness);
          this->setColor(colorIndex);
        } else {
          this->fadeDown(brightness);
        }
        break;
    }
  }

  void setLeds(CRGB* leds, int start, int stop, CHSV color)  {
      for (int i = start; i < stop; i++) {
        this->leds[i] = color;
    }
  }
  
  void setColor(int colorIndex) {
    switch (color) {
      case 0:
        for (int i = 0; i < numLeds; i++) {
          this->leds[i] = CRGB::Red;
        }
      break;
      case 1:
        for (int i = 0; i < numLeds; i++) {
          this->leds[i] = CHSV(colorArrays1[k][arrayIndex][colorIndex].h, colorArrays1[k][arrayIndex][colorIndex].s, colorArrays1[k][arrayIndex][colorIndex].v);
        }
      break;
    }
  }

  void setColorIrregular(int colorIndex)  {
    switch (color) {
      case 0:
        for (int i = 0; i < numLeds; i += moveBy) {
          position++;
          int shiftedIndex = (i + position) % numLeds;
          this->leds[shiftedIndex] = CHSV(colorArrays[arrayIndex][colorIndex].h, colorArrays[arrayIndex][colorIndex].s, colorArrays[arrayIndex][colorIndex].v);
        }
      break;
      case 1:
        for (int i = 0; i < numLeds; i += moveBy) {
          position++;
          int shiftedIndex = (i + position) % numLeds;
          this->leds[shiftedIndex] = CHSV(colorArrays1[k][arrayIndex][colorIndex].h, colorArrays1[k][arrayIndex][colorIndex].s, colorArrays1[k][arrayIndex][colorIndex].v);
        }
      break;
    }
  }


  void updateLEDsMid(float val, float valAvgOut, float maxVal, float threshold, int& brightness, int colorIndex) {
    if ((val >= valAvgOut * 1.05 || maxVal * 0.9 >= valAvgOut) && maxVal > threshold) {
      brightness = 255;
      this->setColor(colorIndex);
    } else {
      this->fadeDownn(brightness);
    }
  }
  

  void setBlack() {
    for (int i = 0; i < numLeds; i++) {
      this->leds[i] = CRGB::Black;
    }
  }

  void fadeUp(int& brightness) {
    if (brightness <= 235) {
      brightness = brightness + fadeAmount;

      for (int i = 0; i < numLeds; ++i) {
        this->increaseBrightnessBy(leds[i], brightness);
      }
    }
  }

  void fadeDown(int& brightness) {
    if (brightness >= fadeAmount * 2 && brightness > 20) {
      brightness = brightness - fadeAmount;

      for (int i = 0; i < numLeds; i++ ) {
        this->leds[i].fadeLightBy(-brightness+255);
      }
    } else {
      brightness = 0;
      this->setBlack();
    }
  }

  void fadeDownn(int& brightness) {
    if (brightness >= fadeAmount * 2 && brightness > 20) {
      brightness = brightness - fadeAmount * 5;

      for (int i = 0; i < numLeds; i++ ) {
        this->leds[i].fadeLightBy(-brightness+255);
      }
    } else {
      brightness = 0;
      this->setBlack();
    }
  }

  void increaseBrightnessBy(CRGB& led, uint8_t amount) {
    uint8_t r = scale8(led.r, 255 - amount);
    uint8_t g = scale8(led.g, 255 - amount);
    uint8_t b = scale8(led.b, 255 - amount);

    led.r = qadd8(led.r, r);
    led.g = qadd8(led.g, g);
    led.b = qadd8(led.b, b);
  }
};


  int spinState = 0;
  bool isSpinning = false;
  
  void spin(CRGB* leds) {
    isSpinning = true;
  }


class Buffer {
private:
  const unsigned int arraySize = 250;
  const unsigned int arraySizeSh = 50;
  int avgOut;
  std::deque<float> buffer;            // Added deque for buffer

public:
  Buffer(int initialBufferSize)
    : arraySize(initialBufferSize),
      buffer(initialBufferSize) {  // Initialize buffer with initialBufferSize
    buffer.resize(arraySize);
  }

  float movingAverage(const std::deque<float>& buffer, int arrSz) {
    float sum = 0;
    for (float value : buffer) {
      sum += value;
    }
    return sum / arrSz;
  }

  void updateMovingAverages(float& output, int arrSz) {
    if (buffer.size() >= arrSz) {
      output = movingAverage(buffer, arrSz);
    }
  }

  void updateBuffer(float newestBufferVal, float& avgOut, int arrSz) {
    buffer.pop_front();
    buffer.push_back(newestBufferVal);
    updateMovingAverages(avgOut, arrSz);
  }
  
};

void updateHuesByK(int by) {
  for (int k = 0; k < colorSets; k++) {
    for (int j = 0; j < numArrays; j++) {
      for (int i = 0; i < arrayLengths[j]; i++) {
        if (i >= 0 && i <= 4 && j == 0) { // subbass color range
          colorArrays1[k][j][i].hue += by;
          if (colorArrays1[k][j][i].hue > 255) {
            colorArrays1[k][j][i].hue -= 255;
          }
        } else if (i >= 5 && i <= 15 && j == 1) { // bass color range
          colorArrays1[k][j][i].hue += by;
          if (colorArrays1[k][j][i].hue > 255) {
            colorArrays1[k][j][i].hue -= 255;
          }
        } else if (i >= 16 && i <= 40 && j == 2) { // midrange color range
          colorArrays1[k][j][i].hue += by;
          if (colorArrays1[k][j][i].hue > 255) {
            colorArrays1[k][j][i].hue -= 255;
          }
        } else if (i >= 41 && i <= 180 && j == 3) { // highfreq color range
          colorArrays1[k][j][i].hue += by;
          if (colorArrays1[k][j][i].hue > 255) {
            colorArrays1[k][j][i].hue -= 255;
          }
        } else if (i >= 181 && i <= 277 && j == 4) { // presence color range
          colorArrays1[k][j][i].hue += by;
          if (colorArrays1[k][j][i].hue > 255) {
            colorArrays1[k][j][i].hue -= 255;
          }
        } else if (i >= 278 && j == 5)  { // brilliance color range
          colorArrays1[k][j][i].hue += by;
          if (colorArrays1[k][j][i].hue > 255) {
            colorArrays1[k][j][i].hue -= 255;
          }
        }
      }
    }
  }
}

void updateHuesBy(int by) {
  for (int j = 0; j < numArrays; j++) {
    for (int i = 0; i < arrayLengths[j]; i++) {
      if (i >= 0 && i <= 4 && j == 0) { // subbass color range
        colorArrays[j][i].hue += by;
        if (colorArrays[j][i].hue > 255) {
          colorArrays[j][i].hue -= 255;
        }
      } else if (i >= 5 && i <= 15 && j == 1) { // bass color range
        colorArrays[j][i].hue += by;
        if (colorArrays[j][i].hue > 255) {
          colorArrays[j][i].hue -= 255;
        }
      } else if (i >= 16 && i <= 40 && j == 2) { // midrange color range
        colorArrays[j][i].hue += by;
        if (colorArrays[j][i].hue > 255) {
          colorArrays[j][i].hue -= 255;
        }
      } else if (i >= 41 && i <= 180 && j == 3) { // highfreq color range
        colorArrays[j][i].hue += by;
        if (colorArrays[j][i].hue > 255) {
          colorArrays[j][i].hue -= 255;
        }
      } else if (i >= 181 && i <= 277 && j == 4) { // presence color range
        colorArrays[j][i].hue += by;
        if (colorArrays[j][i].hue > 255) {
          colorArrays[j][i].hue -= 255;
        }
      } else if (i >= 278 && j == 5)  { // brilliance color range
        colorArrays[j][i].hue += by;
        if (colorArrays[j][i].hue > 255) {
          colorArrays[j][i].hue -= 255;
        }
      }
    }
  }
}

    Leds ledStrip0(NUM_LEDS_MID, leds0, 0); // leds0 is a CRGBArray, so you may need to wrap it in a CRGBSet if you want to pass it in the same way
    Leds ledStrip1(NUM_LEDS, leds1, 1);
    Leds ledStrip2(NUM_LEDS_T_and_B, leds2, 2);
    Leds ledStrip3(NUM_LEDS_T_and_B, leds3, 3);
    Leds ledStrip4(NUM_LEDS, leds4, 4); // same comment as for leds0
    Leds ledStrip5(NUM_LEDS, leds5, 5); // same comment as for leds0

    Buffer subbassBuffer(arraySize);
    Buffer bassBuffer(arraySize);
    Buffer midBuffer(arraySize);
    Buffer highBuffer(arraySize);
    Buffer presenceBuffer(arraySize);
    Buffer brillianceBuffer(arraySize);

    Buffer subbassBufferSh(arraySizeSh);
    Buffer bassBufferSh(arraySizeSh);
    Buffer midBufferSh(arraySizeSh);
    Buffer highBufferSh(arraySizeSh);
    Buffer presenceBufferSh(arraySizeSh);
    Buffer brillianceBufferSh(arraySizeSh);
    
    

void setup() {
  FastLED.addLeds<WS2812B, LED_PIN0, GRB>(leds0, NUM_LEDS_MID);
  FastLED.addLeds<WS2812B, LED_PIN1, GRB>(leds1, NUM_LEDS);
  FastLED.addLeds<WS2812B, LED_PIN2, GRB>(leds2, NUM_LEDS_T_and_B);
  FastLED.addLeds<WS2812B, LED_PIN3, GRB>(leds3, NUM_LEDS_T_and_B);
  FastLED.addLeds<WS2812B, LED_PIN4, GRB>(leds4, NUM_LEDS);
  FastLED.addLeds<WS2812B, LED_PIN5, GRB>(leds5, NUM_LEDS);
  FastLED.setBrightness(200);

  pinMode(BUTTON, INPUT_PULLUP);
  pinMode(BUTTON1, INPUT_PULLUP);

  AudioMemory(12);
  audioShield.enable();
  audioShield.inputSelect(myInput);
  audioShield.lineInLevel(7);
  //audioShield.micGain(15);
  fft.windowFunction(AudioWindowBlackman1024);

  Serial.begin(115200);

  for (int k = 0; k < colorSets; k++) {
    for (int j = 0; j < numArrays; j++) {
      for (int i = 0; i < arrayLengths[j]; i++) {
        if (i >= 0 && i <= 4 && j == 0) { // subbass color range
          colorArrays1[k][j][i] = CHSV(map(i, 0, 4, k * setsRange, setsRange + k * setsRange), 255, 255);
        } else if (i >= 5 && i <= 15 && j == 1) { // bass color range
          colorArrays1[k][j][i] = CHSV(map(i, 5, 15, k * setsRange, setsRange + k * setsRange), 255, 255);
        } else if (i >= 16 && i <= 40 && j == 2) { // midrange color range
          colorArrays1[k][j][i] = CHSV(map(i, 16, 40, k * setsRange, setsRange + k * setsRange), 255, 255);
        } else if (i >= 41 && i <= 180 && j == 3) { // highfreq color range
          colorArrays1[k][j][i] = CHSV(map(i, 41, 180, k * setsRange, setsRange + k * setsRange), 255, 255);
        } else if (i >= 181 && i <= 277 && j == 4) { // presence color range
          colorArrays1[k][j][i] = CHSV(map(i, 181, 277, k * setsRange, setsRange + k * setsRange), 255, 255);
        } else if (i >= 278 && j == 5)  { // brilliance color range
          colorArrays1[k][j][i] = CHSV(map(i, 278, 463, k * setsRange, setsRange + k * setsRange), 255, 255);
        }
      }
    }
  }

  for (int j = 0; j < numArrays; j++) {
    for (int i = 0; i < arrayLengths[j]; i++) {
      if (i >= 0 && i <= 4 && j == 0) { // subbass color range
        colorArrays[j][i] = CHSV(map(i, 0, 4, 0, 255), 255, 255);
      } else if (i >= 5 && i <= 15 && j == 1) { // bass color range
        colorArrays[j][i] = CHSV(map(i, 4, 15, 0, 255), 255, 255);
      } else if (i >= 16 && i <= 40 && j == 2) { // midrange color range
        colorArrays[j][i] = CHSV(map(i, 16, 40, 128, 62), 255, 255);
      } else if (i >= 41 && i <= 180 && j == 3) { // highfreq color range
        colorArrays[j][i] = CHSV(map(i, 41, 180, 228, 94), 255, 255);
      } else if (i >= 181 && i <= 277 && j == 4) { // presence color range
        colorArrays[j][i] = CHSV(map(i, 181, 277, 0, 200), 255, 255);
      } else if (i >= 278 && j == 5)  { // brilliance color range
        colorArrays[j][i] = CHSV(map(i, 278, 463, 200, 180), 255, 255);
      }
    }
  }
}

void loop() {

  float n = 0;
  int i = 0;

  int k = 1;

  unsigned long currentMillisPot = millis();
  unsigned long currentMillis = millis();

  buttonColPress();                                                       //But1 hold - colorArray change
  buttonPress(preButtonState1, curButtonState1, k, colorSets, BUTTON1);   //But1 - color
  buttonPress(preButtonState, curButtonState, mode, 4, BUTTON);           //But0 - mode

  if (fft.available()) {
    float subbass = 0, bass = 0, midrange = 0, highfreq = 0, presence = 0, brilliance = 0;
    float maxSubBassVal = 0, maxBassVal = 0, maxMidVal = 0, maxHighVal = 0, maxPresenceVal = 0, maxBrillianceVal = 0;
    subBassBin = 0, bassBin = 0, midBin = 0, highBin = 0, presenceBin = 0, brillianceBin = 0;

    
    for (i = 0; i < 462; i++) {
      n = fft.read(i);

      if (i >= 0 && i <= 4) {
        if (n >= maxSubBassVal) {
          maxSubBassVal = n;
          subBassBin = i;
        }
        subbass += n;
      }
      else if (i >= 5 && i <= 15) {
        if (n >= maxBassVal) {
          maxBassVal = n;
          bassBin = i;
        }
        bass += n;
      }
      else if (i >= 16 && i <= 40) {
        if (n > maxMidVal) {
          maxMidVal = n;
          midBin = i;
        }
        midrange += n;
      }
      else if (i >= 41 && i <= 180) {
        if (n > maxHighVal) {
          maxHighVal = n;
          highBin = i;
        }
        highfreq += n;
      }
      else if (i >= 181 && i <= 277) {
        if (n > maxPresenceVal) {
          maxPresenceVal = n;
          presenceBin = i;
        }
        presence += n;
      }
      else if (i >= 278 && i <= 462) {
        if (n > maxBrillianceVal) {
          maxBrillianceVal = n;
          brillianceBin = i;
        }
        brilliance += n;
      }
    }

    subbassBuffer.updateBuffer(subbass, subBassAvgOut, arraySize);
    bassBuffer.updateBuffer(bass, bassAvgOut, arraySize);
    midBuffer.updateBuffer(midrange, midAvgOut, arraySize);
    highBuffer.updateBuffer(highfreq, highAvgOut, arraySize);
    presenceBuffer.updateBuffer(presence, presenceAvgOut, arraySize);
    brillianceBuffer.updateBuffer(brilliance, brillianceAvgOut, arraySize);

    subbassBufferSh.updateBuffer(subbass, subBassAvgOutSh, arraySizeSh);
    bassBufferSh.updateBuffer(bass, bassAvgOutSh, arraySizeSh);
    midBufferSh.updateBuffer(midrange, midAvgOutSh, arraySizeSh);
    highBufferSh.updateBuffer(highfreq, highAvgOutSh, arraySizeSh);
    presenceBufferSh.updateBuffer(presence, presenceAvgOutSh, arraySizeSh);
    brillianceBufferSh.updateBuffer(brilliance, brillianceAvgOutSh, arraySizeSh);
 

    if (currentMillisPot - previousMillisPot >= INTERVAL_POT) {                 
      previousMillisPot = currentMillisPot;
      //potentiometer(potPin, potentiometerBrigh);
      potentiometer(potPin1, potentiometerSens);
    }

    
   if (currentMillis - previousMillis >= MODE_INTERVAL) {
      previousMillis = currentMillis;
      //mode++;
      switch (color)  {
        case 0:
          updateHuesBy(5);
        break;
        
        case 1:
          updateHuesByK(10);
        break;
      }
    }
    /*else if (mode >= 4) {
      mode = 0;
    }*/

      Serial.print(subbass);
      Serial.print(",");
      Serial.print(bass);
      Serial.print(",");
      Serial.print(midrange);
      Serial.print(",");
      Serial.print(highfreq);
      Serial.print(",");
      Serial.print(presence);
      Serial.print(",");
      Serial.print(brilliance);
      Serial.println(",");

    shortVsLong(subBassAvgOutSh, subBassAvgOut);

    
    ledStrip0.updateLEDs(subbass, subBassAvgOut, maxSubBassVal, 0.1, subbassBh, subBassBin);
    //ledStrip33.updateLEDs(highfreq, highAvgOut, maxHighVal, 0.015, highBh, highBin);
    FastLED.setBrightness(10);
    FastLED.show();
  }
}
