# Main-C
#include <Bela.h>
#include <cmath>
#include <vector>
// Definiera portnumren för LED:erna
#define BLUE_LED_PIN 0   // Blå LED på digital pin 0
#define RED_LED_PIN 1    // Röd LED på digital pin 1
// Variabler för att spåra frekvenser och ackord
std::vector<float> frequencyBins;
float sampleRate = 44100.0; // Bela's default sample rate
int fftSize = 1024; // FFT-storlek för frekvensanalys
// Variabler för att spåra om det är moll eller dur
bool isMinor = false;
bool isMajor = false;
// FFT-bibliotek (använd Bela's inbyggda eller ett externt)
#include "FFT.h"
FFT fft(fftSize);
void setup() {
    pinMode(BLUE_LED_PIN, OUTPUT);
    pinMode(RED_LED_PIN, OUTPUT);
    digitalWrite(BLUE_LED_PIN, LOW);
    digitalWrite(RED_LED_PIN, LOW);
    // Initiera FFT
    fft.init();
}
void loop() {
    // Läs in ljuddata (antag att audioIn är en global buffert)
    for (int i = 0; i < fftSize; i++) {
        fft.input[i] = audioIn[i]; // audioIn är en global array med ljudsamples
    }
    // Utför FFT
    fft.execute();
    // Analysera frekvenser för att detektera ackord
    // (Förenklat exempel: detektera grundton och ters)
    float fundamentalFreq = detectFundamentalFrequency();
    float thirdFreq = detectThirdFrequency(fundamentalFreq);
    // Bestäm om det är moll eller dur
    isMinor = isMinorChord(fundamentalFreq, thirdFreq);
    isMajor = isMajorChord(fundamentalFreq, thirdFreq);
    // Uppdatera LED:erna
    if (isMinor) {
        digitalWrite(BLUE_LED_PIN, HIGH);
        digitalWrite(RED_LED_PIN, LOW);
    } else if (isMajor) {
        digitalWrite(BLUE_LED_PIN, LOW);
        digitalWrite(RED_LED_PIN, HIGH);
    } else {
        // Ingen tydlig ton detekterad
        digitalWrite(BLUE_LED_PIN, LOW);
        digitalWrite(RED_LED_PIN, LOW);
    }
}
// Förenklade funktioner för att detektera frekvenser och ackord
float detectFundamentalFrequency() {
    // Implementera en algoritm för att detektera grundfrekvensen
    // (t.ex. genom att hitta den starkaste frekvensen i FFT-resultatet)
    float maxMagnitude = 0;
    float fundamentalFreq = 0;
    for (int i = 0; i < fftSize / 2; i++) {
        float magnitude = fft.output[2 * i] * fft.output[2 * i] + fft.output[2 * i + 1] * fft.output[2 * i + 1];
        if (magnitude > maxMagnitude) {
            maxMagnitude = magnitude;
            fundamentalFreq = (i * sampleRate) / fftSize;
        }
    }
    return fundamentalFreq;
}
float detectThirdFrequency(float fundamentalFreq) {
    // Detektera tersfrekvensen (ca 1.25x grundfrekvensen för dur, 1.2x för moll)
    // (Förenklat: sök efter en frekvens nära 1.25x eller 1.2x grundfrekvensen)
    float thirdFreqDur = fundamentalFreq * 1.25; // Dur-ters
    float thirdFreqMinor = fundamentalFreq * 1.2; // Moll-ters
    // Jämför magnituden för dessa frekvenser i FFT-resultatet
    // och returnera den som är starkast
    float magnitudeDur = getMagnitudeAtFrequency(thirdFreqDur);
    float magnitudeMinor = getMagnitudeAtFrequency(thirdFreqMinor);
    return (magnitudeDur > magnitudeMinor) ? thirdFreqDur : thirdFreqMinor;
}
float getMagnitudeAtFrequency(float freq) {
    int bin = static_cast<int>(freq / sampleRate * fftSize);
    if (bin >= 0 && bin < fftSize / 2) {
        return fft.output[2 * bin] * fft.output[2 * bin] + fft.output[2 * bin + 1] * fft.output[2 * bin + 1];
    }
    return 0;
}
bool isMinorChord(float fundamentalFreq, float thirdFreq) {
    // Moll-ters är ca 1.2x grundfrekvensen
    return (std::abs(thirdFreq - fundamentalFreq * 1.2) < 10.0); // Tolerans för detektering
}
bool isMajorChord(float fundamentalFreq, float thirdFreq) {
    // Dur-ters är ca 1.25x grundfrekvensen
    return (std::abs(thirdFreq - fundamentalFreq * 1.25) < 10.0); // Tolerans för detektering
}