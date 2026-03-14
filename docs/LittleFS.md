To build littlefs.bin from kettle-firmware/data, open PowerShell and enter:

$MKLFS = "C:\Users\Keith\AppData\Local\arduino15\packages\esp32\tools\mklittlefs\4.0.2-db0513a\mklittlefs.exe"
& $MKLFS -c .\data -s 0x0E0000 .\littlefs.bin


To flash littlefs.bin to the CYD, open PowerShell and enter:

# adjust COM port to your board
$PORT = "COM9"

# path to esptool inside Arduino ESP32 package (yours may differ slightly)
$ESPTOOL = "$env:LOCALAPPDATA\arduino15\packages\esp32\tools\esptool_py\5.1.0\esptool.exe"

# (optional but recommended) wipe just the LittleFS partition region
& $ESPTOOL --chip esp32s3 --port $PORT --baud 921600 erase_region 0x310000 0x0E0000

# flash the image you built
& $ESPTOOL --chip esp32s3 --port $PORT --baud 921600 write_flash -z 0x310000 .\littlefs.bin
