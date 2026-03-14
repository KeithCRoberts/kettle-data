{
  "schema": 1,
  "latest_fw": "1.1.0",
  "latest_fw_date": "01/21/2026",

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
        "mcu": "ESP32-S3",
        "flash_mb": 16,
        "psram": true,
        "psram_mb": 8,
        "touch": "capacitive",
        "sd": "sdmmc"
      }
    }
  },
  
  "devices": [
    { "device_id": "82F3ECBA", "owner_name": "Bob Taubert",   "profile": "CYD-USBC" },
    { "device_id": "C4CE1D88", "owner_name": "Keith Roberts", "profile": "CYD-USBC" },
    { "device_id": "84FFEBBC", "owner_name": "Ron Taubert",   "profile": "CYD-USBC" },
    { "device_id": "C3A6628F", "owner_name": "Tim Osborne",   "profile": "CYD-USBC" },
    { "device_id": "2D97092D", "owner_name": "Dave Fink",     "profile": "CYD-USBC" },
    { "device_id": "075CCD6B", "owner_name": "Keith Roberts", "profile": "CYD-USBM" }
  ]
}
