# FSR Matrix Printed Circuit Board

<p align="center">
These are the CAD files for the FSR printed circuit boards (PCB) using through-hole only components. This board is used for controlling a force sensitive resistor matrix, reading analog values from each cell.  
</p>

## 📖 Description

This design is based off of Sensitronics' MatrixArray products which are comprised of a multitude of force sensing resistor elements arranged in a grid.
Providing n-input multitouch + force sensing capabilities, MatrixArrays are the basis for musical instruments, PC input devices, and other innovative designs.
If you have measured single-element FSRs by looking at the voltage across a divider, we're doing the same thing here.

But in this case, we're choosing the point to measure by powering a single drive column, enabling one mux channel so that the ADC sees a single row.
In the next section, the Arduino code demonstrates a scan pattern to read all $\approx$ 160 elements in sequence.

### 🚨 Dependencies

* Software
  * KiCAD v6
* Hardware
  * ThruMode MatrixArray
  * Arduino Uno (or other)
  * 74HC595 Shift Register (x2)
  * 74HC4051 Analog Multiplexer (x2)
  * 20k resistor (x1)
  * Solderless Breadboard & Jumper Wire

### 💻 Installing

#### 🕹️ Software

Use a web browser to navigate to www.kicad.org the KiCad EDA website. Click the DOWNLOAD button on the KiCad home page. On the download page that opens, click the Windows button. Under Stable Release, click either Windows 64-bit or Windows 32-bit, depending on if you are going to install KiCad on a 32-bit or 64-bit version of the Windows operating system. Save the KiCad installation file. It may take a while to download, depending on your internet speed.

1. Click the Open button under File > Open (`Ctrl + O`) and navigate to this repository
2. Select the `force_sens_resistor.kicad_pro` file
3. Double click the `*.sch` file and begin to edit the board

#### 🤖 Hardware

Arrange the hardware connections folling this schematic:

<p align="center">
  <img src="https://user-images.githubusercontent.com/29158525/204196328-7055ace2-47ef-4152-ad91-9fc8b4088fba.png">
</p>


### ▶️ Executing program

Now that the hardware's connected, it's time to write some code.

To read a single intersection, or "cell", of the MatrixArray, we need to perform the following sequence:

* Drive the cell's row with +5V
* Enable the cell's column (multiplexer channel)
* Take an ADC reading on pin A0
* Output the reading via serial port

Simple enough, so from here, we will write a scan pattern to efficiently drive and read each cell of the Matrix. Recalling that we're calling shift register drive pins "rows" and multiplexed read pins "columns" in this example, we'll enable each column in order, and while each column is active, we'll drive each row high and take a reading.

```cpp
// Constant definitions
...

void setup() {
  Serial.begin(BAUD_RATE);

  pinMode(PIN_ADC_INPUT, INPUT);
  pinMode(PIN_SHIFT_REGISTER_DATA, OUTPUT);
  pinMode(PIN_SHIFT_REGISTER_CLOCK, OUTPUT);
  pinMode(PIN_MUX_CHANNEL_0, OUTPUT);
  pinMode(PIN_MUX_CHANNEL_1, OUTPUT);
  pinMode(PIN_MUX_CHANNEL_2, OUTPUT);
  pinMode(PIN_MUX_INHIBIT_0, OUTPUT);
  pinMode(PIN_MUX_INHIBIT_1, OUTPUT);
}
void loop() {
  for(int i = 0; i < ROW_COUNT; i ++) {
    setRow(i);
    shiftColumn(true);
    shiftColumn(false);                         //with SR clks tied, latched outputs are one clock behind

    for(int j = 0; j < COLUMN_COUNT; j ++) {
      int raw_reading = analogRead(PIN_ADC_INPUT);
      byte send_reading = (byte) (lowByte(raw_reading >> 2));
      shiftColumn(false);
      printFixed(send_reading);
      Serial.print(" ");
    }
    Serial.println();
  }
  Serial.println();
  delay(200);
}

...

// Function implementations for `setRow`, `shiftCoumn`, and `printFixed`

```

## 🧔🏽‍♂️Authors

Contributors names and contact info

Kyle Kochi
Tiffany Prayitino
Umar Ali  

## 🧾 Version History

* 1.0
    * Initial Release follwoing Sensitronics' MatrixArray tutorial
