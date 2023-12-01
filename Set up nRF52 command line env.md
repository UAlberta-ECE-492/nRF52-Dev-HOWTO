This HOWTO uses the following references:

* https://embeddedexplorer.com/nrf52-gcc-tutorial/
* https://content.u-blox.com/sites/default/files/NINA-B3_DataSheet_UBX-17052099.pdf

We assume we're using the NINA-B306 family and according to the datasheet softdevice S140 is recommended. While that information is not useful here, it good to keep in mind.

Here we simply want to set things up for the nRF52xxx in general, so will not consider incorporating BLE in our simple test.

1. I have a main `Development` folder I like to work in, so have created subfolders under which all the of steps below happen. That is, everything is rooted at the folder below where "project name" is to be replaced with your project.
```
~/Development/<project name>
```
2. I like to keep resources in one place, so make a directory
```
~/Development/<project name>/nRFResources
```
3. Download [Nordic command line tools](https://www.nordicsemi.com/Products/Development-tools/nrf-command-line-tools/download) into `nRFResources` 
	1. Selected version 10.23.0 Linux x86 64
	3. For Ubuntu, choose the `.deb` file and install by double-clicking
	   `nrf-command-line-tools_10.23.0_amd64.deb`
	3. Double-click the `.deb` file and let the installer install it.
	4. This will actually fail because of circular dependency on a SEGGER package. To fix, open a terminal and execute the command 
	   `sudo dpkg -i --force-overwrite /opt/nrf-command-line-tools/share/JLink_Linux_V788j_x86_64.deb`
	5. Check that `nrfjprog` has been installed by executing
```
nrfjprog --version
```
4. Download the [nRF52 SDK from Nordic](https://www.nordicsemi.com/Products/Development-software/nrf5-sdk/download) into `nRFResources`. For this HOWTO, 17.1.0 is the latest. The various soft devices and the SDK are bundled into a single zip file, `DeviceDownload.zip`
5. From inside  `~/Development/<project name>/nRFResources`, unzip the `DeviceDownload.zip` file. There should be one SDK archive and some soft device archives, e.g.
```
ls -l    
total 1112512
-rw-rw-r-- 1 knud 131838843 Aug  9 16:14 nRF5_SDK_17.1.0_ddde560.zip
-rw-rw-r-- 1 knud    278071 Aug  9 16:14 s112nrf52720.zip
-rw-rw-r-- 1 knud    273973 Aug  9 16:14 s113nrf52720.zip
-rw-rw-r-- 1 knud    261560 Aug  9 16:14 s122nrf52800.zip
-rw-rw-r-- 1 knud    494754 Aug  9 16:14 s132nrf52720.zip
-rw-rw-r-- 1 knud    448582 Aug  9 16:14 s140nrf52720.zip
```
6. Download the GCC ARM toolchain from [here](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain) into `nRFResources`.
	1. At this time the latest is 12.3Rel1; however, there appears to be a bug with the compiler and array indexing, so we will use release 11.3.rel1. For Linux the file to download is `arm-gnu-toolchain-11.3.rel1-x86_64-arm-none-eabi.tar.xz`
	2. Even the 11.3.rel1 release has issues with a missing library. While harmless, it's kind of stupid to have to ignore the linker error. So, instead, we will look at using release 10.3-2021.10 that can be found [on the old, deprecated download site](https://developer.arm.com/downloads/-/gnu-rm). The desired file is named `gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2`
	   
	   It's fine to use as the SDR and softdevice came out well before it was released, so they are compatible
	3. Download the file into `~/Development/<project name>/nRFResources`
7. We will follow the guidelines and use `make`, but hope to change to Meson someday.
8. Under `~/Development/<project name>` make a new dir, `nRF52` and change into it  by executing `cd ~/Development/<project name>/nRF52`
9. Extract the nRF5 SDK zip file (here it's `nRF5_SDK_17.1.0_ddde560.zip`) under `nRF52`. There will be a folder named `nRF5_SDK_17.1.0_ddde560`.
```
unzip ../nRFResources/nRF5_SDK_17.1.0_ddde560.zip
```
10. Extract the ARM gcc tools under `nRF52`
```
tar jxf ../nRFResources/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2
```
11. So we don't have to use the release-specific SDK name, make a soft-link in `~/Development/Spartan` to the SDK and confirm it by executing
```
cd ~/Development/Spartan
ln -s nRF52/nRF5_SDK_17.1.0_ddde560 sdk
ls -l sdk 
lrwxrwxrwx 1 knud knud 29 Aug 14 13:20 sdk -> nRF52/nRF5_SDK_17.1.0_ddde560
```
12. Last, we modify the toolchain path by updating the `Makefile.posix` file. It can be found under `nRF52/nRF5_SDK_17.1.0_ddde560/components/toolchain/gcc/`. In your favourite editor, change the line that is 
```
GNU_INSTALL_ROOT ?= /usr/local/gcc-arm-none-eabi-9-2020-q2-update/bin/
```
   to
```
GNU_INSTALL_ROOT ?= ${HOME}/Development/<project name>/nRF52/gcc-arm-none-eabi-10.3-2021.10/bin/
```
12. Confirm this works by re-making the peripheral blinky example.
13. `cd nRF52/nRF5_SDK_17.1.0_ddde560/examples/peripheral/blinky`
14. We are using the nRF52840-dk, so choose the `pca10056` directory
```
cd pca10056/blank/armgcc
```
15. Make the hex and bin files.
```
make clean
make
```
16. On my Linux box the output looks like;
```
pca10056/blank/armgcc via 🅒 base took 3.5s …
➜ make clean 
rm -rf _build

pca10056/blank/armgcc via 🅒 base …
➜ make      
mkdir _build
cd _build && mkdir nrf52840_xxaa
Assembling file: gcc_startup_nrf52840.S
Compiling file: nrf_log_frontend.c
Compiling file: nrf_log_str_formatter.c
Compiling file: boards.c
Compiling file: app_error.c
Compiling file: app_error_handler_gcc.c
Compiling file: app_error_weak.c
Compiling file: app_util_platform.c
Compiling file: nrf_assert.c
Compiling file: nrf_atomic.c
Compiling file: nrf_balloc.c
Compiling file: nrf_fprintf.c
Compiling file: nrf_fprintf_format.c
Compiling file: nrf_memobj.c
Compiling file: nrf_ringbuf.c
Compiling file: nrf_strerror.c
Compiling file: nrfx_atomic.c
Compiling file: main.c
Compiling file: system_nrf52840.c
Linking target: _build/nrf52840_xxaa.out
   text	   data	    bss	    dec	    hex	filename
   1768	    108	     28	   1904	    770	_build/nrf52840_xxaa.out
Preparing: _build/nrf52840_xxaa.hex
Preparing: _build/nrf52840_xxaa.bin
DONE nrf52840_xxaa
```
17. Now we try programming the nRF52840-DK with the J-Link Plus connected to the debug in header (main USB connector provides power, switch near LEDs is set to default)
```
nrfjprog -f nrf52 --program _build/nrf52840_xxaa.hex --sectorerase --verify --reset
[ ######               ]   0.000s | Erase file - Check image
[ #####                ]   0.000s | Check image validity - Initialize device info
[ ##########           ]   0.000s | Check image validity - Check region 0 settings
[ ###############      ]   0.000s | Check image validity - block 1 of 2
[ #################### ]   0.010s | Check image validity - Finished
[ #############        ]   0.000s | Erase file - Erasing
[ ##########           ]   0.000s | Erasing non-volatile memory - block 1 of 1
[ #################### ]   0.000s | Erasing non-volatile memory - Erase
successful
[ #################### ]   0.114s | Erase file - Done erasing 

[ ######               ]   0.000s | Program file - Checking image
[ #####                ]   0.000s | Check image validity - Initialize device info
[ ##########           ]   0.000s | Check image validity - Check region 0 settings
[ ###############      ]   0.063s | Check image validity - block 1 of 2
[ #################### ]   0.007s | Check image validity - Finished
[ #############        ]   0.000s | Program file - Programming
[ ##########           ]   0.000s | Programming image - block 1 of 1
[ #################### ]   0.000s | Programming image - Write successful
[ #################### ]   0.023s | Program file - Done programming

[ ######               ]   0.000s | Verify file - Check image
[ #####                ]   0.000s | Check image validity - Initialize device info
[ ##########           ]   0.000s | Check image validity - Check region 0
settings
[ ###############      ]   0.096s | Check image validity - block 1 of 2
[ #################### ]   0.006s | Check image validity - Finished
[ #############        ]   0.000s | Verify file - Verifying
[ ##########           ]   0.000s | Verifying image - block 1 of 1
[ #################### ]   0.000s | Verifying image - Verify successful
[ #################### ]   0.012s | Verify file - Done verifying                                                       
Applying system reset.
Run.

```
18. The blinky light demo appears to work.  All 4 green LEDs flash in sequence with 100 ms on/off
