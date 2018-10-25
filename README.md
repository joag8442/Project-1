
#include <AltSoftSerial.h>    // Arduino build environment requires this
#include <wavTrigger.h>

#define LED 13                // our LED

wavTrigger wTrig;             // Our WAV Trigger object

Metro gLedMetro(500);         // LED blink interval timer
Metro gSeqMetro(6000);        // Sequencer state machine interval timer

byte gLedState = 0;           // LED State
int  gSeqState = 0;           // Main program sequencer state
int  gRateOffset = 0;         // WAV Trigger sample-rate offset
int  gNumTracks;              // Number of tracks on SD card

char gWTrigVersion[VERSION_STRING_LEN];    // WAV Trigger version string



void setup() {
  // put your setup code here, to run once:
  // Serial monitor
  Serial.begin(9600);
 
  // Initialize the LED pin
  pinMode(LED,OUTPUT);
  digitalWrite(LED,gLedState);

  // If the Arduino is powering the WAV Trigger, we should wait for the WAV
  //  Trigger to finish reset before trying to send commands.
  delay(1000);

  // WAV Trigger startup at 57600
  wTrig.start();
  delay(10);
  
  // Send a stop-all command and reset the sample-rate offset, in case we have
  //  reset while the WAV Trigger was already playing.
  wTrig.stopAllTracks();
  wTrig.samplerateOffset(0);
  
  // Enable track reporting from the WAV Trigger
  wTrig.setReporting(true);
  
  // Allow time for the WAV Trigger to respond with the version string and
  //  number of tracks.
  delay(100); 
}

void loop() {
  // put your main code here, to run repeatedly:
if (gSeqMetro.check() == 1) {
      
      switch (gSeqState) {
  
          // State 0: Demonstrates how to fade in a music track 
          case 0:
              wTrig.samplerateOffset(0);            // Reset our sample rate offset
              wTrig.masterGain(0);                  // Reset the master gain to 0dB
              
              wTrig.trackGain(2, -40);              // Preset Track 2 gain to -40dB
              wTrig.trackPlayPoly(2);               // Start Track 2
              wTrig.trackFade(2, 0, 2000, 0);       // Fade Track 2 up to 0dB over 2 secs
          break;

          // State 1: Demonstrates how to cross-fade music tracks
          case 1:
              wTrig.trackGain(1, -40);              // Preset Track 1 gain to -40dB
              wTrig.trackPlayPoly(1);               // Start Track 1
              wTrig.trackFade(1, 0, 3000, false);   // Fade Track 1 up to 0db over 3 secs
              wTrig.update();
              delay(2000);                          // Wait 2 secs
              wTrig.trackFade(2, -40, 3000, true);  // Fade Track 2 down to -40dB over 3 secs and stop
          break;
                                 
          // State 2:  start looping track 4
          case 2:
             
              wTrig.trackPlayPoly(4);               // Start Track 4 poly
              wTrig.trackLoop(4, 1);                // Enable Track 4 looping
          break;
          // State 3: play chords 2 times
          case 3:
              wTrig.trackPlayPoly(3);               // Start Track 3 poly
              delay(500);
              wTrig.trackStop(3);                   // Stop Track 3
              delay(250);
              wTrig.trackPlayPoly(3);               // Start Track 3 poly
              delay(500);
              wTrig.trackStop(3);                   // Stop Track 3
          break;
}
