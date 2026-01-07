Bugs:

When no Pushover user key, exit routine (don't log that a notification was sent)

Not saving eTag for system.json

BootCfg in kettle-firmware.ino when logged invert = 1 (s/b true)

Something seems to be wiping out LittleFS - needs more investigation


=====================================================

Enhancements:

SD card cleanup routine
  - Delete any CORE and/or log files after 30 days
  
Logging enhancements
  - Only log lock calc on startup and when a new lock is issued (currently logging every refresh)
  - Change Payoff: log message to debug only (or delete altogether)
  - Add DEBUG, INFO, WARN, ERROR to the start of all logged messages to enable filtering

Statistics enhancements
  - Make sure we're capturing sprite 8-bit failures as well as sprite 4-bit failures
  - Improve formatting on all statistics  --  Done v1.1.0
  - Use colors in Pushover message to denote warnings/errors
  - Persist statistics to LittleFS - minimally for a day, but potentially for a week (rolling)
  - Establish a configuration parameter to dump the persisted statistics to the log
  - Establish warning levels for memory heap  --  Done v1.1.0
  - Check/identify when memory heap logging is before/after an operation (particularly with sprite fallbacks)

Support configuration update on a web portal
  - Add a Wi-Fi station mode if Wi-Fi can't connect  --  Done v1.1.0
  - Create a web server when in station mode  --  Done v1.1.0
  - Create the HTML to update the configuration file through the web server

Pushover
  - Add a Pushover message when a Stop & Lock gains x% during the day  --  Done v1.1.0

Device Profiles
  - Add device profile support to system.json
  - Create profile variables for screen inversion & rotation
  - Cache screen inversion & rotation on startup & use if cached
  
Splits/Acquisitions
  - Handle splits (and reverse splits) for watchlist securities (x tendered, y issued)
  - Handle cash acquisition
  - Handle stock acquisition (x tendered, y issued)
  - Handle cash & stock acquistion (x tendered, $ + y issued)
  - Handle complicated (x tendered, $ + y issued + z issued)

System Files
  - Reorder loading of system files to eliminate reboot requirement