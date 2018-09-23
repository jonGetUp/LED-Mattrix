/*  Autors: jonGetUp & Tonton
 *  Date: 20.09.2018
 *  Infos: Display the text present on "your...API" across a 128x8 Leds-Matrix.
 *	Equipment:
 *	- ArduinoMega with a Yunshield or an Arduino Yunshield
 * 		Warning: arduino Uno doesn't have enough RAM
 *	- Four 32x8 Adafruit_NeoMatrix
 */

#include <Adafruit_GFX.h>
#include <Adafruit_NeoMatrix.h>
#include <Adafruit_NeoPixel.h>
#include <Process.h>
#include <Wire.h>  // Comes with Arduino IDE
#include <LiquidCrystal_I2C.h>
#ifndef PSTR
 #define PSTR // Make Arduino MEGA happy
#endif

#define PIN 6

/*-----( MATTRIX declaration)-----*/
// Parameter 1 = width of NeoPixel matrix
// Parameter 2 = height of matrix
// Parameter 3 = tiles number of X axis
// Parameter 4 = tiles number of Y axis
// Parameter 5 = pin number (most are valid)
// Parameter 6 = matrix layout flags, add together as needed:
//  NEO_MATRIX_TOP, NEO_MATRIX_BOTTOM, NEO_MATRIX_LEFT, NEO_MATRIX_RIGHT:
//    Position of the FIRST LED in the matrix; pick two, e.g.
//    NEO_MATRIX_TOP + NEO_MATRIX_LEFT for the top-left corner.
//  NEO_MATRIX_ROWS, NEO_MATRIX_COLUMNS: LEDs are arranged in horizontal
//    rows or in vertical columns, respectively; pick one or the other.
//  NEO_MATRIX_PROGRESSIVE, NEO_MATRIX_ZIGZAG: all rows/columns proceed
//    in the same order, or alternate lines reverse direction; pick one.
//  NEO_TILE_TOP or NEO_TILE_BOTTOM and NEO_TILE_LEFT or NEO_TILE_RIGHT:
//    Position of the first tile; pick two
//  NEO_TILE_ROWS or NEO_TILE_COLUMNS:
//    Tiles can be arranged in horizontal rows or vertical columns
//  NEO_TILE_PROGRESSIVE or NEO_TILE_ZIGZAG:
//    Indicate the order
// Parameter 7 = pixel type flags, add together as needed:
//   NEO_KHZ800  800 KHz bitstream (most NeoPixel products w/WS2812 LEDs)
//   NEO_KHZ400  400 KHz (classic 'v1' (not v2) FLORA pixels, WS2811 drivers)
//   NEO_GRB     Pixels are wired for GRB bitstream (most NeoPixel products)
//   NEO_RGB     Pixels are wired for RGB bitstream (v1 FLORA pixels, not v2)

Adafruit_NeoMatrix matrix = Adafruit_NeoMatrix(32, 8, 4, 1, PIN,
  NEO_MATRIX_BOTTOM  + NEO_MATRIX_RIGHT + NEO_MATRIX_COLUMNS + NEO_MATRIX_ZIGZAG +
  NEO_TILE_BOTTOM    + NEO_TILE_RIGHT   + NEO_TILE_COLUMNS   + NEO_TILE_PROGRESSIVE,
  NEO_GRB            + NEO_KHZ800);

/*-----( Declare Variables )-----*/
String text = "";
int f = 0;  //index for loop
int x = matrix.width();
int sL = 0; //String length

void setup() {/*----( SETUP: RUNS ONCE )----*/
  Bridge.begin();
  matrix.begin();
  matrix.setTextWrap(false); // Allow text to run off right edge
  matrix.setBrightness(50);
  matrix.setTextColor(matrix.Color(255, 255, 255));

  }/*--( end setup )---*/

void loop() {/*----( LOOP )----*/
  matrix.fillScreen(0); 	//turn all pixel off
  if(text != runCurl()) {	//watch the network change
    text = runCurl();
    sL = text.length();
    if(sL <= 21) {  //short text
        matrix.setCursor(x, 0);
        matrix.print(text);
        matrix.show();
        delay(60000); //60 sec
    }
    else{  //long text
      for(f = 0; f < 10; f++) { //repeat 10 time the text
        matrix.fillScreen(0); //turn all pixel off
        matrix.setCursor(x, 0);
        matrix.print(text); 
        if(--x < -(sL*6)) { //decrement x each test, shift one matrix width and 6 time per char     
          x = matrix.width();}
        matrix.show();
        delay(80);
      }
    }
  }
}/*----( end LOOP )----*/

String runCurl() {/*----( runCurl: talk with the network )----*/
  String netText = "";  //local variable
  // Launch "curl" command and get the network text
  // curl is command line program for transferring data using different internet protocols
  Process p;        // Create a process and call it "p"
  p.begin("curl");  // Process that launch the "curl" command
  p.addParameter("http://..........."); // Add the URL parameter to "curl"
  p.run();  // Run the process and wait for its termination

  while (p.available()>0) {
    // A process output can be read with the stream methods
    netText += p.read();
  }
  return netText;
  }/*----( end runCurl )----*/
