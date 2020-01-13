# CURRENTLY UNDER DEVELOPMENT

# BMP388_DEV
An Arduino compatible, non-blocking, I2C/SPI library for the Bosch BMP388 barometer.

![alt text](https://cdn-learn.adafruit.com/assets/assets/000/072/428/small360/sensors_BMP388_Top_Angle.jpg?1551997243 "Adafruit BMP388 Breakout Board")

© Copyright, image courtesy of [Adafruit Industries](https://www.adafruit.com/product/3966) lisensed under the terms of the [Create Commons Attribution-ShareAlike 3.0 Unported](https://creativecommons.org/licenses/by-sa/3.0/legalcode). 

This BMP388_DEV library offers the following features:

- Returns temperature in degrees celius (**°C**), pressure in hectoPascals/millibar (**hPa**) and altitude in metres (**m**)
- NORMAL or FORCED modes of operation
- I2C or hardware SPI communications with configurable clock rates
- Non-blocking operation 
- In NORMAL mode barometer returns results at the specified standby time interval
- Highly configurable, allows for changes to pressure and temperature oversampling, IIR filter and standby time
- Polling or interrupt driven measurements (using the BMP388's external INT pin)
- Storage and burst reading of up to 72 temperature and pressure measurements using the BMP388's internal 512 byte FIFO memory

##__Contents__

Version

## __Version__

- Version 1.0.0 -- Intial version

## __Arduino Compatibility__

- All Arduino boards, but for 5V Arduino boards (such as the Uno, Nano, Mega, Leonardo, etc...), please check if the BMP388 breakout board requires a 5V to +3.3V voltage level shifter

## __Installation__

The BMP388_DEV library can be installed using the Arduino IDE's Library Manager. To access the Library Manager, in the Arduino IDE's menu select _Sketch->Include Library->Manage Libraries..._. In the Library Manager's search bar type BMP388 then select the "Install" button in the BMP388_DEV entry.

Alternatively simply download BMP388_DEV from this Github repository, un-zip or extract the files and place the BMP388_DEV directory in your _.../Arduino/libraries/..._ folder. The _.../Arduino/..._ folder is the one where your Arduino IDE sketches are usually located.

## __Usage__

### __BMP388_DEV Library__

Simply include the BMP388_DEV.h file at the beginning of your sketch:

```
#include <BMP388_DEV.h>
```

For I2C communication the BMP388_DEV object is created (instantiated) without parameters:

```
BMP388_DEV bmp388;	// Set up I2C communications
```

By default the library uses the BMP388's I2C address 0x77. (To use the alternate I2C address: 0x76, see the begin() function below.

For SPI communication the chip select (CS) Arduino digital output pin is specified as an argument, for example digital pin 10:

```
BMP388_dev bmp388(10);	// Set up SPI communications on digital pin D10
```

The library also supports the ESP32 HSPI operation on pins: SCK 14, MOSI 13, MISO 27 and user defined SS (CS):

```
SPIClass SPI1(HSPI);							    // Create the SPI1 HSPI object
BMP388_DEV bmp388(21, HSPI, SPI1);		// Set up HSPI port communications on the ESP32
```

By default the I2C runs in fast mode at 400kHz and SPI at 1MHz. However it is possible to change the clock speed using the set clock function:

```
bmp388.setClock(4000000);			// Set the SPI clock to 4MHz
```

---
### __Device Initialisation__

To initialise the BMP388 it is necessary to call the begin() function with or without arguments. The parameters specify the starting mode, pressure/temperature oversampling, IIR filter and standby time options respectively:

```
bmp388.begin(SLEEP_MODE, OVERSAMPLING_X16, OVERSAMPLING_X2, IIR_FILTER_4, TIME_STANDBY_5MS);
```

Alternatively simply call the begin function without any arguments, this sets up the default configuration: SLEEP_MODE, pressure oversampling X16, temperature oversampling X2, IIR filter OFF and a standby time of 5ms:

```
bmp388.begin();	// Initialise the BMP388 with default configuration
```

Another alternative is to pass the BMP388's mode as an argument:

```
bmp388.begin(NORMAL_MODE);	// Initialise the BMP388 in NORMAL_MODE with default configuration
```

Or, specifying mode and alternate I2C address:

```
bmp388.begin(FORCED_MODE, BMP388_I2C_ALT_ADDR);	// Initialise the BMP388 in FORCED_MODE with the alternate I2C address (0x76)
```

Or even just the alternate I2C address, (BMP388 initialised in SLEEP_MODE by default):

```
bmp388.begin(BMP388_I2C_ALT_ADDR);	// Initialise the BMP388 with the alternate I2C address (0x76)
```

Note that the begin functions return the value 1 upon successful initialisation, otherwise it returns 0 for failure.

---
### __Device Configuration__

After initialisation it is possible to change the BMP388 configuration with the following functions:

```
bmp388.setPresOversamping(OVERSAMPING_X4);	// Options are OVERSAMPLING_SKIP, _X1, _X2, _X4, _X8, _X16, _32
```

```
bmp388.setTempOversamping(OVERSAMPING_X4);	// Options are OVERSAMPLING_SKIP, _X1, _X2, _X4, _X8, _X16, _X32
```

```
bmp388.setIIRFilter(IIR_FILTER_16);	// Options are IIR_FILTER_OFF, _2, _4, _8, _16, _32
```

```
bmp388.setTimeStandby(TIME_STANDBY_2000MS);	// Options are TIME_STANDBY_05MS, _62MS, _125MS, _250MS, _500MS, _1000MS, 2000MS, 4000MS
```
---
### __Modes Of Operation__

The BMP388 has 3 modes of operation: **SLEEP_MODE**, **NORMAL_MODE** and **FORCED_MODE**: 

- **SLEEP_MODE**: puts the device into an inactive standby state 

- **NORMAL_MODE**: performs continuous conversions, separated by the standby time

- **FORCED_MODE**: performs a single conversion, returning to **SLEEP_MODE** upon completion

To kick-off conversions in **NORMAL_MODE**:

```
bmp388.startNormalConversion();	// Start continuous conversions, separated by the standby time
```

To perform a single oneshot conversion in **FORCED_MODE**:

```
bmp388.startForcedConversion();	// Start a single oneshot conversion
```

To stop the conversion at anytime and return to **SLEEP_MODE**:

```
bmp388.stopConversion();	// Stop conversion and return to SLEEP_MODE
```
---
### __Results Acquisition__

The BMP388 barometer library acquires temperature in degrees celius (**°C**), pressure in hectoPascals/millibar (**hPa**) and altitude in metres (**m**). The acquisition functions scan the BMP388's status register and return 1 if the barometer results are ready and have been successfully read, 0 if they are not; this allows for non-blocking code implementation. The temperature, pressure and altitude results themselves are _float_ variables by passed reference to the function and are updated upon a successful read.

Here are the results acquisition functions:

```
bmp388.getMeasurements(temperature, pressure, altitude);	// Acquire temperature, pressue and altitude measurements
```

```
bmp388.getTempPres(temperature, pressure);	// Acquire both the temperature and pressure
```

```
bmp388.getTemperature(temperature);	// Acquire the temperature only
```

```
bmp388.getPressure(pressure);	// Acquire the pressure only, (also calculates temperature, but doesn't return it)
```

```
bmp388.getAltitude(altitude);	// Acquire the altitude only
```
---
### __Code Implementation__

Here is an example sketch of how to use the BMP388 library for non-blocking I2C operation, default configuration with continuous conversion in NORMAL_MODE, but with a standby sampling time of 1 second:

```
#include <BMP388_DEV.h>                           // Include the BMP388_DEV.h library

float temperature, pressure, altitude;            // Create the temperature, pressure and altitude variables
BMP388_DEV bmp388;                                // Instantiate (create) a BMP388_DEV object and set-up for I2C operation (address 0x77)

void setup() 
{
  Serial.begin(115200);                           // Initialise the serial port
  bmp388.begin();                                 // Default initialisation, place the BMP388 into SLEEP_MODE 
  bmp388.setTimeStandby(TIME_STANDBY_1000MS);     // Set the standby time to 1s
  bmp388.startNormalConversion();                 // Start NORMAL conversion mode
}

void loop() 
{
  if (bmp388.getMeasurements(temperature, pressure, altitude))    // Check if the measurement is complete
  {
    Serial.print(temperature);                    // Display the results    
    Serial.print(F("*C   "));
    Serial.print(pressure);    
    Serial.print(F("hPa   "));
    Serial.print(altitude);
    Serial.println(F(m"));
  }
}
```

A second sketch example for I2C operation, default configuration in FORCED conversion mode:

```
#include <BMP388_DEV.h>                           // Include the BMP388_DEV.h library

float temperature, pressure, altitude;            // Create the temperature, pressure and altitude variables
BMP388_DEV bmp388;                                // Instantiate (create) a BMP388_DEV object and set-up for I2C operation (address 0x77)

void setup() 
{
  Serial.begin(115200);                           // Initialise the serial port
  bmp388.begin();                                 // Default initialisation, place the BMP388 into SLEEP_MODE 
}

void loop() 
{
  bmp388.startForcedConversion();                 // Start a forced conversion (if in SLEEP_MODE)
  if (bmp388.getMeasurements(temperature, pressure, altitude))    // Check if the measurement is complete
  {
    Serial.print(temperature);                    // Display the results    
    Serial.print(F("*C   "));
    Serial.print(pressure);    
    Serial.print(F("hPa   "));
    Serial.print(altitude);
    Serial.println(F("m"));
  }
}
```

The sketches for SPI operation are identical except that the line:

```
BMP388_DEV bmp388;	// Instantiate (create) a BMP388_DEV object and set-up for I2C operation (address 0x77)
```

...should be replaced with the line:

```
BMP388_DEV bmp388(10);	// Instantiate (create) a BMP388_DEV object and set-up for SPI operation with chip select on D10
```

For more details see code examples provided in the _.../examples/..._ directory.

---
### __Interrupts__

The BMP388 barometer has an INT output pin that enables measurements to be interrupt driven instead of using polling. Interrupts function in both in NORMAL and FORCED modes of operation.

Interrupts are configured by calling the enable interrupt function with or without arguments. The parameters specify whether the INT pin output drive is: PUSH_PULL or OPEN_DRAIN, the signal is: ACTIVE_LOW or ACTIVE_HIGH and interrupt itself is: UNLATCHED or LATCHED. In UNLATCHED mode the interrupt signal automatically clears after 2.5ms, while in LATCHED mode the interrupt signal remains active until the data is read.

The default settings are PUSH_PULL, ACTIVE_HIGH and UNLATCHED:

```
bmp388.enableInterrupt(PUSH_PULL, ACTIVE_HIGH, UNLATCHED);		// Enable interrupts with default settings
```

Alternatively the enable interrupt function with default settings can be called without any arguments:

```
bmp388.enableInterrupt();		// Enable interrupts with default settings
```
The interrupts can also be disabled by calling the disable interrupt function:

```
bmp388.disableInterrupt();		// Enable interrupts with default settings
```
The interrupt settings can also be changed independently:

```
bmp388.setIntOutputDrive(OPEN_DRAIN);		// Set the interrupt pin's output drive to open drain
```

```
bmp388.setIntActiveLevel(ACTIVE_LOW);		// Set the interrupt signal's active level to LOW
```

```
bmp388.setIntLatchConfig(LATCHED);		// Set the interrupt signal to latch until cleared
```

Attaching the Arduino microcontroller the BMP388's INT pin is performed using the standard Arduino attachInterrupt() function:

```
attachInterrupt(digitalPinToInterrupt(2), interruptHandler, RISING);   // Set interrupt to call interruptHandler function on D2
```

If the SPI interface is being used and happens to be shared with other devices, then it is also necessary to call the SPI usingInterrupt function as well:

```
bmp388.usingInterrupt(digitalPinToInterrupt(2));     // Invoke the SPI usingInterrupt() function
```

The I2C interface uses the Arduino Wire library. However as the Wire library generates interrupts itself during operation, it is unfortunately not possible to call the results acqusition functions (such as getTemperature(), getPressure() and getMeasurements()) from within the Interrupt Service Routine (ISR). Instead a data ready flag is set within the ISR that allows the barometer data to be read in the main loop() function.

Here is an example sketch using I2C in NORMAL_MODE, default configuration with interrupts:

```
//////////////////////////////////////////////////////////////////////////////////////////
// BMP388_DEV - I2C Communications, Default Configuration, Normal Conversion, Interrupt
//////////////////////////////////////////////////////////////////////////////////////////

#include <BMP388_DEV.h>                           // Include the BMP388_DEV.h library

volatile boolean dataReady = false;               // Define the data ready flag
float temperature, pressure, altitude;            // Declare the measurement variables

BMP388_DEV bmp388;                                // Instantiate (create) a BMP388_DEV object and set-up for I2C operation (address 0x77)

void setup() 
{
  Serial.begin(115200);                           // Initialise the serial port
  bmp388.begin();                                 // Default initialisation, place the BMP388 into SLEEP_MODE 
  bmp388.enableInterrupt();                       // Enable the BMP388's interrupt (INT) pin
  attachInterrupt(digitalPinToInterrupt(2), interruptHandler, RISING);   // Set interrupt to call interruptHandler function on D2
  bmp388.setTimeStandby(TIME_STANDBY_1280MS);         // Set the standby time to 1.3 seconds * 10 = measurement every 13 seconds
  bmp388.startNormalConversion();                     // Start BMP388 continuous conversion in NORMAL_MODE 
}

void loop() 
{
  if (dataReady)                                      // Check if data is ready
  {  
    bmp388.getMeasurements(temperature, pressure, altitude);      // Read the measurements
    Serial.print(temperature);                    // Display the results    
    Serial.print(F("*C   "));
    Serial.print(pressure);    
    Serial.print(F("hPa   "));
    Serial.print(altitude);
    Serial.println(F("m")); 
    dataReady = false;                            // Clear the dataReady flag
  }   
}

void interruptHandler()                           // Interrupt handler function
{
  dataReady = true;                               // Set the dataReady flag
}
```

The SPI interface on the other hand, does allow for the results acquisition functions to be called from within the ISR.

Here is an example sketch using SPI in NORMAL_MODE, default configuration with interrupts:

```
///////////////////////////////////////////////////////////////////////////////////////////
// BMP388_DEV - SPI Communications, Default Configuration, Normal Conversion, Interrupts
///////////////////////////////////////////////////////////////////////////////////////////

#include <BMP388_DEV.h>                             // Include the BMP388_DEV.h library

volatile boolean dataReady = false;									// Define the data ready flag
volatile float temperature, pressure, altitude;			// Declare the measurement variables

BMP388_DEV bmp388(10);                              // Instantiate (create) a BMP388_DEV object and set-up for SPI operation on digital pin D10

void setup() 
{
  Serial.begin(115200);                             // Initialise the serial port
  bmp388.begin();                                   // Default initialisation, place the BMP388 into SLEEP_MODE 
  bmp388.enableInterrupt();                         // Enable the BMP388's interrupt (INT) pin
  bmp388.usingInterrupt(digitalPinToInterrupt(2));  // Invoke the SPI usingInterrupt() function
  attachInterrupt(digitalPinToInterrupt(2), interruptHandler, RISING);   // Set interrupt to call interruptHandler function on D2
  bmp388.setTimeStandby(TIME_STANDBY_1280MS);       // Set the standby time to 1.3 seconds
  bmp388.startNormalConversion();                   // Start BMP388 continuous conversion in NORMAL_MODE 
}

void loop() 
{
  if (dataReady)                                    // Check if the measurement is complete
  {   
    dataReady = false;                              // Clear the data ready flag
    Serial.print(temperature);                      // Display the results    
    Serial.print(F("*C   "));
    Serial.print(pressure);    
    Serial.print(F("hPa   "));
    Serial.print(altitude);
    Serial.println(F("m"));  
  }
}

void interruptHandler()                             // Interrupt handler function
{
  bmp388.getMeasurements(temperature, pressure, altitude);    // Read the measurement data
  dataReady = true;                                 // Set the data ready flag
}
```
---
## __FIFO (First In First Out) Operation__ 

The BMP388 barometer contains a 512 byte FIFO memory, capable of storing and burst reading up to 72 temperature and pressure measurements in NORMAL_MODE.

By default the BMP388_DEV library always enables temperature, pressure, altitude and sensor time, however this function allows these and other parameters to be changed. The parameters include pressure enable, altitude enable, sensor time enable, subsampling rate and data select: 

```
bmp388.enableFIFO(PRES_ENABLE, ALT_ENABLE, SUBSAMPLETIME_OFF, FILTERED);
```

Alternatively, to enable the FIFO with default settings and without arguments: 

```
bmp388.enableFIFO();		// Enable the BMP388's FIFO operation
```

To disable the FIFO:

```
bmp388.disableFIFO();		// Disable the BMP388's FIFO
```

It is also possible to change the FIFO settings independently:

```
bmp388.setFIFOPressEnable(PRESS_ENABLED);		// Enable FIFO pressure measurements, options: PRESS_DISABLED, PRESS_ENABLED
```

```
bmp388.setFIFOAltEnable(ALT_ENABLED);		// Enable FIFO altitude measurements, options: ALT_DISABLED, ALT_ENABLED
```

```
bmp388.setFIFOTimeEnable(TIME_ENABLED);		// Enable FIFO sensor time, options: IME_DISABLED, TIME_ENABLED
```

```
bmp388.SetFIFOSubSampling(SUBSAMPING_OFF);	 // Enable FIFO sub-sampling, options: SUBSAMPING_OFF, DIV2, DIV4, DIV8, DIV16, DIV32, DIV64, DIV128
```

```
bmp388.SetDataSelect(FILTERED);		// Set FIFO to store unfiltered or filtered data, options UNFILITERED, FILTERED
```

```
bmp388.StopFIFOOnFull(STOP_ON_FULL_ENABLED);	// Set FIFO to stop when full, options: UNFILTERED, FILTERED
```

To specify the number of measurements required:

```
bmp388.setFIFONoOfMeasurements(10);		// Calculate the size of the FIFO required to store 10 measurements
```

This function above calculates the size in bytes required to store the specified number of either temperature or temperature and pressure meaurements. If the allocation 


In order to check if 
```


bmp388.getFifoData(
```

The FIFO also allows measurements to the FIFO to be sub-sampled at a lower rate than the sample rate. The sub-sample rate is a division of the barometer standard sample rate and can be set using the subSample.

The data select option allows the FIFO to store data that UNFILTERED or FILTERED


---
## __FIFO Operation with Interrupts__ 

In NORMAL_MODE the BMP388 barometer also allows FIFO operation to be integrated with interrupts, using its INT pin to indicate to the microcontroller that batch of measurements are ready to be read. This is extremely useful for ultra low power applications, since it allows the barometer to independently collect data over a long duration, while the microcontroller remains asleep.

To enable FIFO interrupts simply call the FIFO interrupt function, the parameters are identical to the enable interrupt function with the same default argurments:

```
bmp388.enableFIFOInterrupt(PUSH_PULL, ACTIVE_HIGH, UNLATCHED);		// Enable FIFO interrupts
```
Alternatively, this function can also be called with default arguments without specifying any parameters:

```
bmp388.enableFIFOInterrupt(PUSH_PULL, ACTIVE_HIGH, UNLATCHED);		// Enable FIFO interrupts
```

To disable FIFO interrupts:

```
bmp388.enableFIFOInterrupt();		// Disable FIFO interrupts
```

It is also possible to change the FIFO interrupt settings independently:

```

```

```

```

```

```

---
## __Example Code__

Here is the list of the Arduino example sketches:

- __BMP388_I2C_Normal.ino__ : I2C Interface, Normal Mode, Standard I2C Address (0x77)

- __BMP388_I2C_Alt_Normal.ino__ : I2C Interface, Normal Mode, Alternative I2C Address (0x76)

- __BMP388_I2C_Forced.ino__ : I2C Interface, Forced Mode

- __BMP388_I2C_Normal_Interrupt.ino__ : I2C Interface, Normal Mode, Interrupts

- __BMP388_I2C_Forced_Interrupt.ino__ : I2C Interface, Forced Mode, Interrupts

- __BMP388_I2C_Normal_FIFO.ino__ : I2C Interface, Normal Mode, FIFO Operation

- __BMP388_I2C_Normal_Interrupt_FIFO.ino__ : I2C Interface, Normal Mode, Interrupts, FIFO Operation

- __BMP388_SPI_Normal.ino__ : SPI Interface, Normal Mode

- __BMP388_SPI_Forced.ino__ : SPI Interface, Forced Mode

- __BMP388_SPI_Normal_Interrupt.ino__ : SPI Interface, Normal Mode, Interrupts

- __BMP388_SPI_Forced_Interrupt.ino__ : SPI Interface, Forced Mode, Interrupts

- __BMP388_SPI_Normal_FIFO.ino__ : SPI Interface, Normal Mode, FIFO Operation

- __BMP388_SPI_Normal_Interrupt_FIFO.ino__ : SPI Interface, Normal Mode, Interrupts, FIFO Operation

- __BMP388_SPI_Normal_Multiple.ino__ : SPI Interface, Normal Mode, Multiple Devices

- __BMP388_ESP32_HSPI_Normal.ino__ : ESP32 HSPI Interface, Normal Mode
