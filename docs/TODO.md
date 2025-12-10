Bugs:

When Wi-Fi disconnects, switch to demo mode

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
  - Improve formatting on all statistics
  - Use colors in Pushover message to denote warnings/errors
  - Persist statistics to LittleFS - minimally for a day, but potentially for a week (rolling)
  - Establish a configuration parameter to dump the persisted statistics to the log
  - Establish warning levels for memory heap
  - Check/identify when memory heap logging is before/after an operation (particularly with sprite fallbacks)

Support configuration update on a web portal
  - Add a Wi-Fi station mode if Wi-Fi can't connect
  - Create a web server when in station mode
  - Create the HTML to update the configuration file through the web server

Add a Pushover message when a Stop & Lock gains x% during the day



