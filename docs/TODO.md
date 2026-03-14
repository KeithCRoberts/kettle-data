Bugs:

Not saving eTag for system.json

Formating on end of day Pushover message truncates part of message (increase buffer)

Pushover API call failure - retry at least once on timeout


  
=====================================================

V1.1.0 - Enhancements:

UI Changes
  - Display all negative numbers in red (requested by Bob)
  
  
  
=====================================================

V1.2.0 - Enhancements:

SD card cleanup routine
  - Delete any CORE and/or log files after 30 days
  
Logging enhancements
  - Only log lock calc on startup and when a new lock is issued (currently logging every refresh)
  - Change Payoff: log message to debug only (or delete altogether)
  - Add DEBUG, INFO, WARN, ERROR to the start of all logged messages to enable filtering

Statistics enhancements
  - Make sure we're capturing sprite 8-bit failures as well as sprite 4-bit failures
  - Use colors in Pushover message to denote warnings/errors
  - Check/identify when memory heap logging is before/after an operation (particularly with sprite fallbacks)

Support configuration update on a web portal
  - Create the HTML to update the configuration file through the web server
  
Splits/Acquisitions
  - Handle cash acquisition
  - Handle stock acquisition (x tendered, y issued)
  - Handle cash & stock acquistion (x tendered, $ + y issued)
  - Handle complicated (x tendered, $ + y issued + z issued)

System Files
  - Reorder loading of system files to eliminate reboot requirement

ESP32-S3 support
  - improved performance

Battery support - enables portable use
  - Lithium battery
  - Onboard charging
  
RTC support
  - real time clock support, not reliant on NTP syncing
  
Touch support
  - Capacitive touch enabled
  - With rotation off, touch can be used to pull up a detail screen
  - Limited configuration update capability using touch
  
PSRAM support
  - improved memory / stabilization
  


=====================================================

V2.0.0 (Kettle Pro) - Enhancements:

Enhanced display support
  - 480 x 800 landscape mode supported
  
Charting support
  - performance charts for securities
  
  