{
  "schema": 1,
  "latest_fw": "1.2.0",
  "latest_fw_date": "MM/DD/YYYY",
  "latest_fw_url": "https://github.com/KeithCRoberts/kettle-flasher/releases/download/vX.Y.Z/firmware_s3.bin",

  "coredump_max_age_days": 5,
  "coredump_max_files": 10,

  "profiles": {
    "CYD-USBM": {
      "display": {
        "invert": true,
        "rotation": 0
      },
      "capabilities": {
        "mcu": "ESP32",
        "flash_mb": 4,
        "psram": false,
        "touch": "resistive",
        "sd": "spi"
      }
    },
    "CYD-USBC": {
      "display": {
        "invert": false,
        "rotation": 0
      },
      "capabilities": {
        "mcu": "ESP32",
        "flash_mb": 4,
        "psram": false,
        "touch": "resistive",
        "sd": "spi"
      }
    },
    "CYD-USBH": {
      "display": {
        "invert": true,
        "rotation": 0
      },
      "capabilities": {
        "battery": false,
        "mcu": "ESP32-S3",
        "flash_mb": 16,
        "psram": true,
        "psram_mb": 8,
        "touch": "capacitive",
        "touch_enabled": true,
        "sd": "sdmmc"
      }
    },
    "CYD-USBHB": {
      "display": {
        "invert": true,
        "rotation": 0
      },
      "capabilities": {
        "battery": true,
        "mcu": "ESP32-S3",
        "flash_mb": 16,
        "psram": true,
        "psram_mb": 8,
        "touch": "capacitive",
        "touch_enabled": true,
        "sd": "sdmmc"
      }
    }
  },

  "devices": [
    { "device_id": "XXXXXXXX", "owner_name": "Member Name", "profile": "CYD-USBC",  "comment": "Description" },
    { "device_id": "XXXXXXXX", "owner_name": "Member Name", "profile": "CYD-USBH",  "comment": "Description" },
    { "device_id": "XXXXXXXX", "owner_name": "Member Name", "profile": "CYD-USBHB", "comment": "Description" }
  ]
}