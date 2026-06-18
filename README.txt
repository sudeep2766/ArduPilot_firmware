Mon Jun 15 16:33:14
Done on : ubuntu:latest     docker

From: https://github.com/ArduPilot/ardupilot

#  signing firmware worked only on python3.11
(venv-ardupilot) flash@d4e770def1ed:~/ardupilot$ sudo apt install -y software-properties-common
(venv-ardupilot) flash@d4e770def1ed:~/ardupilot$ sudo apt install python3.11 python3.11-venv
python3.11 -m venv venv-signing
source venv-signing/bin/activate
# and install pymonocipher for signing the fw
pip install pymonocypher==3.1.3.2

check:
(venv-signing) flash@d4e770def1ed:~/ardupilot$ python -c "import monocypher; print(monocypher.__version__)"


# keygen:
(venv-signing) flash@d4e770def1ed:~/ardupilot$ python Tools/scripts/signing/generate_keys.py mykey
Generated mykey_private_key.dat
Generated mykey_public_key.dat

(venv-signing) flash@d4e770def1ed:~/ardupilot$ sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi



# building secure bootloader(1161):
(venv-ardupilot) flash@be6c13119726:~/ardupilot$ ./Tools/scripts/build_bootloaders.py mini-pix --signing-key=mykey_public_key.dat


# didnt work, so changed FLASH_BOOTLOADER_LOAD_SIZE_KB from 16 to 32(1732):
(venv-signing) flash@d4e770def1ed:~/ardupilot$ nano libraries/AP_HAL_ChibiOS/hwdef/mini-pix/hwdef-bl.dat

# after running line31 again:
BUILD SUMMARY
Build directory: /home/flash/ardupilot/build/mini-pix
Target                    Text (B)  Data (B)  BSS (B)  Total Flash Used (B)  Free Flash (B)  External Flash Used (B)
--------------------------------------------------------------------------------------------------------------------
bootloader/AP_Bootloader     26652        64    11788                 26716            6048  Not Applicable         
'bootloader' finished successfully (14.567s)
Created Tools/bootloaders/mini-pix_bl.bin
Created Tools/bootloaders/mini-pix_bl.elf
Signing bootloader with mykey_public_key.dat
Running (./Tools/scripts/signing/make_secure_bl.py Tools/bootloaders/mini-pix_bl.bin mykey_public_key.dat)
Adding ArduPilot keys
Applying Public Key Tools/scripts/signing/ArduPilotKeys/ArduPilot_public_key2.dat
Applying Public Key Tools/scripts/signing/ArduPilotKeys/ArduPilot_public_key3.dat
Applying Public Key Tools/scripts/signing/ArduPilotKeys/ArduPilot_public_key1.dat
Applying Public Key mykey_public_key.dat
Running (./Tools/scripts/signing/make_secure_bl.py Tools/bootloaders/mini-pix_bl.elf mykey_public_key.dat)
Adding ArduPilot keys
Applying Public Key Tools/scripts/signing/ArduPilotKeys/ArduPilot_public_key2.dat
Applying Public Key Tools/scripts/signing/ArduPilotKeys/ArduPilot_public_key3.dat
Applying Public Key Tools/scripts/signing/ArduPilotKeys/ArduPilot_public_key1.dat
Applying Public Key mykey_public_key.dat
Running (/home/flash/venv-ardupilot/bin/python3 Tools/scripts/bin2hex.py --offset 0x08000000 Tools/bootloaders/mini-pix_bl.bin Tools/bootloaders/mini-pix_bl.hex)
Created Tools/bootloaders/mini-pix_bl.hex




# signing firmware finally:
(venv-signing) flash@be6c13119726:~/ardupilot$ ./Tools/scripts/signing/make_secure_fw.py build/mini-pix/bin/arducopter.apj mykey_private_key.dat
Applying signature
Wrote build/mini-pix/bin/arducopter.apj





in short:

Generate keys:
  python3 -m pip install pymonocypher==3.1.3.2

  Tools/scripts/signing/generate_keys.py mykey


Build secure bootloader:
 Tools/scripts/build_bootloaders.py RadiolinkF405 --signing-key=mykey_public_key.dat


Build firmware:
 ./waf configure --board RadiolinkF405 --signed-fw
 ./waf copter

Sign it:
 # requires pymonocypher
 source venv-signing/bin/activate

 ./Tools/scripts/signing/make_secure_fw.py build/RadiolinkF405/bin/arducopter.apj mykey_private_key.dat


Flash BL