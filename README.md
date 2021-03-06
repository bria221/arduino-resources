# arduino-resources

## General info
This repository is needed to generate all resources, that are used to compile Arduino programs in the OpenRoberta Lab for

- Arduino Mega
- Arduino Nano
- Arduino Uno
- Arduino Uno Wifi Rev.2
- BOB3
- Bot'n Roll
- Festo Bionics4Education
- mBot
- senseBox

The resources generated are

- libraries (.a archive),
- header files and
- a script, that runs the compiler and linker in the Lab

## Details

Sketches that are located in their respective folders contain header includes and are processed by `arduino-cli` to
find all dependencies and compile the libraries. Once this lengthy process is done, the results can be reused.

The main reason for creating this repository is to to speed up the compilation in the Lab. To achieve this,
unnecessary (re-)compilations are avoided by compiling everything that could be referenced in the Lab once and
storing the object files in libraries.

For a stable build the resources are generated inside a docker container. `arduino-cli` is used to download most of
the compilers and resources.

When the container is built, a shell script generates the libraries and stores them together with the header files
in the directory `/tmp/arduino-release`.

unowifirev2 and sensebox need to link with the `variant.c(pp).o` in addition to the core, otherwise nothing will be executed on the board.

LTO support was turned off for uno, nano, mega, botnroll, mbot and unowifirev2 boards due to errors when using libraries on a Raspberry Pi.

## Adding new libraries

In order to add new libraries simply put the include directive with the needed header in the respective sketch.
Do not forget to provide the library itself through arduino-cli in the Dockerfile.
In some cases additional include directories may need to be added to the robot specific build scripts.

## Quickly testing libraries

In case a library needs to be included quickly for some initial tests and should not be included here directly, it can also be added manually to the wanted build script (replace `avr` with appropriate compiler for platform):

- Copy the library into the `RobotArdu/arduino-resources/includes` folder in `ora-cc-rsc`
- Duplicate the whole `avr-g++` compiler call and replace the last line of it with `$LIB_INCLUDE_DIR/<path-to-lib>/<lib-name>.cpp -o $BUILD_DIR/<lib-name>.o`
- Add `$BUILD_DIR/<lib-name>.o` before `$BUILD_DIR/$PROGRAM_NAME.o` in the `avr-gcc` linker call
- Add the appropriate include in the source code editor, e.g `#include <lib-name/src/lib-name.h>` and use it

## Automatic build

The resources are automatically built and released using GitHub Actions when creating a commit with this pattern: `v*`.
A zip file containing all of the libraries and scripts is created and available through the release.

## Manual build

To generate the docker image:

```bash
docker build -t arduino-resources -f Dockerfile .
```

To replace the arduino resources in ora-cc-rsc:

```bash
rm -rf ../ora-cc-rsc/RobotArdu/arduino-resources && mkdir ../ora-cc-rsc/RobotArdu/arduino-resources
docker rm arduinolibs
docker create --name arduinolibs arduino-resources
docker cp arduinolibs:/tmp/arduino-release/. ../ora-cc-rsc/RobotArdu/arduino-resources
```

Following warnings can be disregarded:

`cp: -r not specified; omitting directory './RobotArdu/libraries/ArduinoSTL/src/abi'`

abi folder is not needed at all.

The following:

```bash
In file included from /opt/ora-cc-rsc/RobotArdu/libraries/SparkFun_LSM6DS3_Breakout/src/SparkFunLSM6DS3.cpp:32:0:
/opt/ora-cc-rsc/RobotArdu/hardware/additional/arduino/hardware/megaavr/1.8.5/libraries/Wire/src/Wire.h: In member function 'status_t LSM6DS3Core::readRegisterRegion(uint8_t*, uint8_t, uint8_t)':
/opt/ora-cc-rsc/RobotArdu/hardware/additional/arduino/hardware/megaavr/1.8.5/libraries/Wire/src/Wire.h:62:13: note: candidate 1: uint8_t TwoWire::requestFrom(int, int)
     uint8_t requestFrom(int, int);
             ^~~~~~~~~~~
/opt/ora-cc-rsc/RobotArdu/hardware/additional/arduino/hardware/megaavr/1.8.5/libraries/Wire/src/Wire.h:60:13: note: candidate 2: virtual uint8_t TwoWire::requestFrom(uint8_t, size_t)
     uint8_t requestFrom(uint8_t, size_t);
             ^~~~~~~~~~~
/opt/ora-cc-rsc/RobotArdu/hardware/additional/arduino/hardware/megaavr/1.8.5/libraries/Wire/src/Wire.h: In member function 'status_t LSM6DS3Core::readRegister(uint8_t*, uint8_t)':
/opt/ora-cc-rsc/RobotArdu/hardware/additional/arduino/hardware/megaavr/1.8.5/libraries/Wire/src/Wire.h:62:13: note: candidate 1: uint8_t TwoWire::requestFrom(int, int)
     uint8_t requestFrom(int, int);
             ^~~~~~~~~~~
/opt/ora-cc-rsc/RobotArdu/hardware/additional/arduino/hardware/megaavr/1.8.5/libraries/Wire/src/Wire.h:60:13: note: candidate 2: virtual uint8_t TwoWire::requestFrom(uint8_t, size_t)
     uint8_t requestFrom(uint8_t, size_t);
```

Comes from a third party library, may be fixed with an update
