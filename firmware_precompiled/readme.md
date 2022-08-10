# Beschreibung
user_config_override.h ergänzen um die folgenden Zeilen:

```
#define ETHERNET

#ifndef USE_SCRIPT
#define USE_SCRIPT
#endif
#ifndef USE_SML_M
#define USE_SML_M
#endif
#ifdef USE_RULES
#undef USE_RULES
#endif
```

Zusatzfunktion für Scripting

```
// >W
// The lines in this section are displayed in the web UI main page. Requires compiling with #define USE_SCRIPT_WEB_DISPLAY.
#ifndef USE_SCRIPT_WEB_DISPLAY
#define USE_SCRIPT_WEB_DISPLAY
#endif

// >J
// The lines in this section are published via MQTT in a JSON payload on TelePeriod. Requires compiling with #define USE_SCRIPT_JSON_EXPORT.
#ifndef USE_SCRIPT_JSON_EXPORT
#define USE_SCRIPT_JSON_EXPORT
#endif
```

# Version

V12.0.2 --> Script+ bedeutet --> #define USE_SCRIPT_WEB_DISPLAY & #define USE_SCRIPT_JSON_EXPORT
