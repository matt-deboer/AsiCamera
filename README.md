# ASI Camera Boost

A wrapper for libASICamera2 library that improves performance at higher FPS.
This library wrapper is under development.
The library will be prepared for work with [INDI Library](https://github.com/indilib/indi) and [INDI 3rd Party Drivers](https://github.com/indilib/indi-3rdparty)

# What is the problem in the original library?
When running the library on Raspberry PI 4, the problem becomes reaching the declared maximum FPS, e.g. for the ASI178 camera.
The reasons are:
- copying the same frame several times
- synchronous communication between frames on USB
- problems with handling bad frames

# What has been improved?
After an in-depth analysis of the original library, frame copying has been completely removed in libASICamera2Boost. In addition, more bulks transfers to USB ensure more stable operation.
Additionally:
- you can use several cameras at once
- more frames per second
- more stable work
- correct first frame (long exposure)

# What is the difference in use?
Just swap the libraries!
If you want to get rid of unnecessary data copying from buffer to buffer, be sure to use ASIGetVideoDataPointer function.
With this function, you will get an address to the frame buffer. The data is available until the next function call. You can edit the data in the buffer for image transformations.

# Additional Features
The library offers additional features that increase performance and facilitate debugging. To use, you have to manipulate the source of the program that is using the library.
- ASIGetVideoDataPointer - this function gives direct access to frame data. Copying avoided
- gCameraBoostEnable - global bool variable - restore the original behavior of the library **(also see Control Caps!)**
- gCameraBoostDebug - global bool variable - print additional information useful for debugging to stderr **(also see Control Caps!)**

# Building

## Install Pre-requisites

On Debian/Ubuntu:

```
sudo apt-get -y install git cmake libusb-1.0-0-dev
```

## Get the code and build
```
cd ~/Projects
git clone https://github.com/pawel-soja/AsiCamera

mkdir -p ~/Projects/build/AsiCamera
cd ~/Projects/build/AsiCamera
cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DCMAKE_BUILD_TYPE=Release ~/Projects/AsiCamera
make -j4
```

# Test your camera

## INDI Library
You can run with a preloaded library
```
LD_PRELOAD=~/Projects/build/AsiCamera/lib/libASICamera2Boost.so indiserver indi_asi_ccd
```
![image](https://user-images.githubusercontent.com/73316926/113363285-c8c5b900-9350-11eb-9c1d-2ff4f3ef3e89.png)

## Example - performance
Sample program that tests all connected cameras for different BandWidth values.
In the arguments of the program, you can specify camera options, For example:
```
$ ./example/performance/AsiCameraPerformance Exposure 10000 HighSpeedMode 1 Gain 100 Format 8bit
Camera Found!
ASIGetVideoDataPointer: found

|-----------------------------------------------------------|
|        Parameter       |     Value                        |
|-----------------------------------------------------------|
| Name                   | ZWO ASI178MM                     |
| Camera Id              | 0                                |
| Max Height             | 2080                             |
| Max Width              | 3096                             |
| Is Color               | no                               |
| Bayer Pattern          | RG                               |
| Supported Bins         | 1, 2, 3, 4                       |
| Supported Video Format | Raw8, Raw16                      |
| Pixel Size             | 2.4                              |
| Mechanical Shutter     | no                               |
| ST4 Port               | yes                              |
| Is Cooled Camera       | no                               |
| Is USB3 Host           | yes                              |
| Is USB3 Camera         | yes                              |
| Elec Per Adu           | 0.289665                         |
| Bit Depth              | 14                               |
| Is Trigger Camera      | no                               |
|-----------------------------------------------------------|

Set 'Exposure' to 10000
Set 'HighSpeedMode' to 1
Set 'Gain' to 100
Set 'Format' to 8bit

Camera Options
|------------------------------------------------------------------------------------------------------------------------------|
|        Option  Name      |  Value |   Min  |     Max    | Default  | Auto |                Description                       |
|------------------------------------------------------------------------------------------------------------------------------|
| Gain                     | 100    | 0      | 510        | 210      | no   | Gain                                             |
| Exposure                 | 10000  | 32     | 2000000000 | 10000    | no   | Exposure Time(us)                                |
| Offset                   | 10     | 0      | 600        | 10       | no   | offset                                           |
| BandWidth                | 100    | 40     | 100        | 50       | no   | The total data transfer rate percentage          |
| Flip                     | 0      | 0      | 3          | 0        | no   | Flip: 0->None 1->Horiz 2->Vert 3->Both           |
| AutoExpMaxGain           | 255    | 0      | 510        | 255      | no   | Auto exposure maximum gain value                 |
| AutoExpMaxExpMS          | 30000  | 1      | 60000      | 100      | no   | Auto exposure maximum exposure value(unit ms)    |
| AutoExpTargetBrightness  | 100    | 50     | 160        | 100      | no   | Auto exposure target brightness value            |
| HardwareBin              | 0      | 0      | 1          | 0        | no   | Is hardware bin2:0->No 1->Yes                    |
| HighSpeedMode            | 1      | 0      | 1          | 0        | no   | Is high speed mode:0->No 1->Yes                  |
| Temperature              | 266    | -500   | 1000       | 20       | no   | Sensor temperature(degrees Celsius)              |
| CameraBoostDebug         | 0      | 0      | 1          | 0        | no   | Enable debug mode for Boost                      |
| CameraDebug              | 0      | 0      | 1          | 0        | no   | Enable debug mode for ASICamera2                 |
| MaxChunkSize             | 128    | 1      | 256        | 128      | no   | Size limit of a single USB transfer(MB)          |
| ChunkedTransfers         | 2      | 1      | 8          | 2        | no   | Number of chunked transfers                      |
| USBMemory                | 512    | 0      | 4096       | 512      | no   | /sys/module/usbcore/parameters/usbfs_memory_mb   |
|------------------------------------------------------------------------------------------------------------------------------|

Find best bandwidth
|---------------------------------------------------------------------------------------------------------------------------------|
| BW [%] |                                            Frame duration [ms]                                                         |
|---------------------------------------------------------------------------------------------------------------------------------|
|   40%  |  303    48    36    42    41    42    41    42    41    42    41    42    42    42    41    42    41    43    40    42 |
|   45%  |   23    37    37    37    37    37    37    36    37    37    37    38    37    37    37    37    37    37    37    37 |
|   50%  |   20    34    34    33    35    33    32    34    34    33    34    33    33    34    33    33    33    34    33    33 |
|   55%  |   17    30    31    31    30    31    30    30    31    30    30    30    30    30    31    30    30    30    30    31 |
|   60%  |   12    28    28    28    27    28    27    28    28    28    28    28    27    28    28    28    27    28    28    28 |
|   65%  |   10    26    26    25    26    25    25    26    25    26    26    26    26    26    26    25    26    26    26    26 |
|   70%  |   11    24    23    24    24    24    23    24    24    25    24    24    23    25    24    24    24    24    25    23 |
|   75%  |    8    22    24    23    22    22    22    22    23    23    22    22    24    21    22    22    23    22    22    23 |
|   80%  |    4    20    21    21    20    22    21    20    21    22    21    20    21    20    21    20    22    20    22    20 |
|   85%  |    2    19    20    20    20    20    18    20    19    20    20    20    19    21    17    20    19    20    19    21 |
|   90%  |    1    19    18    19    19    18    19    21    17    19    19    17    20    18    18    19    19    18    18    19 |
|   95%  |    3    17    18    17    18    18    17    18    18    17    18    18    17    19    18    18    18    17    16    19 |
|  100%  |    1    17    16    17    17    17    16    18    16    16    17    17    16    16    17    16    17    18    16    16 |
|---------------------------------------------------------------------------------------------------------------------------------|

Best bandwidth is 100%, it has 60.1 FPS
```
The table shows the waiting time for the frame for the given BandWidth value. Based on these values, the FPS value is calculated.

## Example - simply
A simplified program that measures time frames for specific parameters.
If you used the original libASICamera2 library, nothing will surprise you here.
The program is very useful for debugging a library.
```
$ ./example/simply/AsiCameraSimply Exposure 10000 HighSpeedMode 1 Gain 100 Format 8bit
ASIGetVideoDataPointer: found
[ASIOpenCamera]: grab CameraID 0
[CameraBoost]: created
[__wrap_libusb_open]: grab libusb_device_handle 0x557a57792090
[__wrap__ZN11CCameraBase12InitVariableEv]: grab CCameraBase 0x557a577ab8d0
Set 'Exposure' to 10000
Set 'HighSpeedMode' to 1
Set 'Gain' to 100
Set 'Format' to 8bit
[ResetDevice]: catched
[initAsyncXfer]: device 0x557a57792090, endpoint 0x81, buffer size 6439680
timeframe:   258ms, status: 0
timeframe:    20ms, status: 0
timeframe:    13ms, status: 0
timeframe:    17ms, status: 0
timeframe:    17ms, status: 0
timeframe:    19ms, status: 0
timeframe:    14ms, status: 0
timeframe:    20ms, status: 0
timeframe:    14ms, status: 0
timeframe:    19ms, status: 0
timeframe:    14ms, status: 0
timeframe:    19ms, status: 0
timeframe:    15ms, status: 0
timeframe:    19ms, status: 0
timeframe:    14ms, status: 0
timeframe:    19ms, status: 0
timeframe:    14ms, status: 0
timeframe:    19ms, status: 0
timeframe:    15ms, status: 0
timeframe:    19ms, status: 0
[releaseAsyncXfer]: catched
```
