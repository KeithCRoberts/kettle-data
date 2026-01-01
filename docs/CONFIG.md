{
  "update_interval": 10,
  "timezone": "America/Chicago",
  "muted": false,
  "wifi": [
	{ "ssid": "YOUR-SSID", "pass": "YOUR-SSID-PW" },
	{ "ssid": "ALT-SSID", "pass": "ALT-SSID-PW" }
  ],
  "rotation": {
	  "enabled": true,
	  "grid_delay": 60,
	  "view_delay": 15
  },
  "notifications": {
	  "wifi_connect": true,
	  "new_lock": true,
	  "lock_daily_change": {
	    "send": true,
		"threshold_pct": 7.5
	  },
	  "midday_update": true,
	  "after_hours_update": true,
	  "eod_stats": false
  },
  "ntp_server": "pool.ntp.org",
  "api_keys": {
    "finnhub": "API-KEY-VALUE",
    "fmp": "API-KEY-VALUE",
	"pushover": "PUSHOVER_USER_KEY"
  }
}
