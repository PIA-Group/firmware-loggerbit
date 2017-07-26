# LoggerBIT Firmware
Firmware to create a data logger for BITalino (r)evolution based on the OpenLog by SparkFun

## Programming the OpenLog with the LoggerBIT firmware
Loading the LoggerBIT firmware onto the OpenLog is the first thing to do. This process **must** be done **without making any physical changes** to the OpenLog board that is currently shipped by SparkFun. 

1. Download the [Arduino IDE version 1.6.5](https://www.arduino.cc/en/Main/OldSoftwareReleases), a verified known good version.
2. Download the [LoggerBIT firmware](https://github.com/BITalinoWorld/firmware-loggerbit/blob/master/LoggerBIT_BIN.ino).
3. Download the required libraries:
   * Bill Greiman's [SerialPort library](https://github.com/greiman/SerialPort)
   * Bill Greiman's [SdFat library](https://github.com/greiman/SdFat)
4. Install the libraries into Arduino. Check [here](https://www.arduino.cc/en/Guide/Libraries) for more detailed instructions on how to do it.
5. Modify the **SerialPort.h** file found in the **\Arduino\Libraries\SerialPort** directory. Change `BUFFERED_TX` to `0` and `ENABLE_RX_ERROR_CHECKING` to `0`.
6. Connect the OpenLog to the computer via an FTDI board. Check [here](https://learn.sparkfun.com/tutorials/openlog-hookup-guide#hardware-hookup) for more detailed instructions on how to make the connection between the two boards.
7. Open the LoggerBIT sketch with the Arduino IDE, select the **Arduino/Genuino Uno** board setting under **Tools>Board**, and select the proper COM port for the FTDI board under **Tools>Port**.
8. Upload the code and it's done! Proceed to the hardware changes in the OpenLog board.

## What to change in the OpenLog board for using hardware flow-control
Once the OpenLog is loaded with the LoggerBIT firmware, the board has to be physically adjusted, by including the necessary extra wiring, so the hardware flow-control mechanism can properly function.

You will need:
* A male header with 6 pins (right angle)
 <img src="https://github.com/BITalinoWorld/firmware-loggerbit/blob/master/docs/images/5-way-header.jpg" width="128">

* Thin wire (for the shunt)
* Soldering iron and thin solder

Firstly, **bend the leftmost pin of the header**, so that it stays in a straight horizontal direction instead of the original right angle.

*insert image*

Afterwards, place the header on the pads of OpenLog UART interface (at the bottom), so that the bent pin stays over the **GRN pad**. **Solder each of the 5 remaining pins** with the corresponding pad.

*insert image*

Solder one end of the shunt directly on the **2nd pin from the right, on the top side of the MCU** and the other end to the **header pin that hangs above the GRN pad**. Make sure that there is **no solder touching the GRN pad**, it has to remain not connected.

*insert image*

## How to configure the BITalino acquisition settings

To begin with, place the [configuration file](https://github.com/BITalinoWorld/firmware-loggerbit/blob/master/config.txt) on the **root of a freshly formatted microSD card**. For formatting, use the oficial [SD memory card formatter](https://www.sdcard.org/downloads/formatter_4/) tool from the SD Association.

This configuration file has has two lines: one for the settings and the other for making it user-friendly by indexing their definitions.

```
1000,live,123456,00,00
sampling rate,mode,channels,trigger,digital IO
```

The possible settings for the user to configure are defined
as follows:
* `sampling rate` - The sampling rate, in Hz, in which to acquire data, with possible values being `1`, `10`, `100` and `1000`.
* `mode` - The BITalino's state of operation, it can be either `live` or `simulated`. In live mode the device will acquire and stream data in real time, whereas in simulated mode, mostly aimed at developers, it streams synthetic data generated by the firmware (sine, saw tooth, square waves and pre-recorded electrocardiography time series).
* `channels` - Which analog channels to acquire, from the 6 available ones. This parameter is defined as number, in which each digit corresponds to the analog channel of equivalent numeral. For example, if you whish to acquire the first, third and fifth analog channels, then this option should be set as `135`. At most, you can choose to acquire all channels, by setting this parameter as `123456` , and at least, it is possible to not sample any analog channel, by leaving this setting empty. Considering this, possible values for this argument range all the combinations, without repetion, that can be made from the 6-element set.
* `trigger` - This setting is used to indicate whether the data logging to the SD card should initiate immediately or only when a variation is detected in one (`01` or `10`) or both (`11`) digital inputs of the BITalino. This parameter can be useful, for instance, if you wish to initiate the logging by pressing a push button. It is a 2-digit number, in which the tens and units digits correspond to
digital inputs *I1* and *I2*, respectively. It should be noted that both inputs are active-low. As such, for example, if this argument is set as `10` then the logging will only begin when a change is detected in digital input *I1*, i.e., when there is a transition from *HIGH* to *LOW* in that pin.
* `digital IO` - The initial state of the digital outputs pins *O1* and *O2*. Similarly to what occurs with the trigger command, this is also a 2-digit number, in which the tens and units digits correspond to digital outputs *O1* and *O2*, respectively. If, for instance, it is defined as `01`, the digital output pin *O2* is set *HIGH* and the other one is set *LOW*.

## Recommended microSD cards

## How to use the decoder

The [decoder](https://github.com/BITalinoWorld/firmware-loggerbit/blob/master/decoder.py) is a Python script that converts the `.BIN` files generated by the OpenLog into `.TXT` (ASCII) files that are compatible with the OpenSignals (r)evolution software. For each `.BIN` file, its corresponding `.TXT` file is created. At the end of execution, a single `.LOG` file is also created, that contains, per file, some useful informations about the decoding process, such as:
* number of channels configured for acquisiton
* sampling rate
* acquisition mode
* total sampling time
* total decoding time
* number of failed packet receptions
* total number of lost packets

### Arguments

The decoder can convert a single `.BIN` file at a time, or an entire folder (including its subdirectories) where multiple `.BIN` files may reside. As such, it accepts **1 out of 3** arguments:

```
-h, --help              show this help message and exit
-p PATHNAME, --pathname PATHNAME
                        the pathname of the folder to decode
-f FILENAME, --filename FILENAME
                        the filename of the single file to decode
```
One of the arguments -p/--pathname -f/--filename is **always required**.

### How to run the decoder?
The decoder can be executed as a script or treated as a module that can be imported. Below there are examples of how to run it for a single file, for both types of execution.  

* Script
```bash
python decoder.py -f "C:\Users\margarida\logs\LOG00000.BIN"
```

* Module
```c#
import decoder
decoder.main([r'-f C:\Users\margarida\logs\LOG00000.BIN'])
```

## Acknowledgments:
This work was partially supported by the IT – Instituto de Telecomunicações under the grant UID/EEA/50008/2013 "SmartHeart" (https://www.it.pt/Projects/Index/4465).
