# Tasmota für ETH & SML einrichten

1. extra Firware aufspielen.
2. ETH Settings
3. Script erstellen
4. Test
5. Hilfe Links

## Firmware einerichten

`user_config_override.h` ergänzen um die folgenden Zeilen:

```
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

Its only used by Scripting

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

[precompiled binaries](../firmware_precompiled)

## ETH Settings

Main --> Console --> folgende Befehle nacheinander eingeben:

```
Ethernet 1 (default)
EthType 0 (default)
EthAddress 0 (default)
EthClockMode 3 (3 = ETH_CLOCK_GPIO17_OUT)
```

## Script erstellen

ASCII ausgabe des Zählers eZH.....
```
\EMH5----eHZ-E0028E<\r><\n>
<\r><\n>
1-0:0.0.0*255(331210-5002855)<\r><\n>
1-0:1.8.1*255(075595.6918)<\r><\n>
1-0:96.5.5*255(82)<\r><\n>
0-0:96.1.255*255(0000952727)<\r><\n>
!
```
Script

```
>D
>B
->sensor53 r
>M 1  
+1,3,o,1,9600,OBIS
1,1-0:0.0.0*255(@1,Meter Nr,,Meter_number,0
1,1-0:1.8.1*255(@1,Total consumption,KWh,Total_in,4
1,1-0:96.5.5*255(@1, Status,,Status,0
1,0-0:96.1.255*255(@1, Herstellernummer,,Herstellernummer,0
#
```

Manche Zähler haben binäre Codierung: 
```
>D  
>B  
->sensor53 r
>M 1  
+1,3,s,0,9600,OBIS  
1,77070100010800ff@1000,Total consumption,KWh,Total_in,4  
1,77070100020800ff@1000,Total Feed,KWh,Total_out,4  
1,77070100100700ff@1,Current consumption,W,Power_curr,0  
1,77070100000009ff@#,Meter Nr,,Meter_number,0  
#
```


Will man ein paar Berechnungen machen und die Werte auch veröffentlichen (siehe Skript unten), dann muss man die beiden Zeilen: `#define USE_SCRIPT_WEB_DISPLAY` und `#define USE_SCRIPT_JSON_EXPORT` beim Compilieren einbinden.

nähere Beschreibung findet man hier: https://tasmota.github.io/docs/Scripting-Language/


```
>D
pin=0
pi_s=0
pi_m=0
pi_m_add=0
pi_h=0
sec=0
min=0

>B
->sensor53 r

>T
pin=OBIS#Total_in

>S
sec=sec+1

if sec>59
then
sec=0
min=min+1
pi_m=pin-pi_s
pi_s=pin
pi_m_add=pi_m_add+pi_m
print POWER per minute = %pi_m%
endif

if min>60
then
min=0
pi_h=pi_m_add
pi_m_add=0
print POWER per hour = %pi_h%
endif

>W
Minute_Consumption: {m} %pi_m% kW
Houtly_Consumption: {m} %pi_h% kW

>J
,"minute_consumption":%pi_m%
,"hourly_consumption":%pi_h%

>M 1
+1,16,o,1,9600,OBIS
1,1-0:0.0.0*255(@1,Meter Nr,,Meter_number,0
1,1-0:1.8.1*255(@1,Total consumption, KWh,Total_in,4
1,1-0:96.5.5*255(@1, Status,,Status,0
1,0-0:96.1.255*255(@1, Herstellernummer,,Herstellernummer,0
#
```

## Test

IN der Console folgenden Befehl eingeben:

- `Sensor53 d1` Hiermit werden Rohdaten in der Console ausgegeben
- `Sensor53 d0` Hiermit werden Rohdaten in der Console deaktiviert

1 Steht dabei für den ersten definierten Zähler
man kann bis zu 5 Zähler im Script definieren

## openHAB Beispiel

In openHAB muss ein Generic MQTT Thing angelegt werden.
Wobei das Topic und auch der JSONPATH auf die jeweiligen Einstellungen im Tasmota Modul angepasst werden muss.
Tasmota schickt die Werte über das `Tele` Topic. Das Interval steht standartmässig auf 60s. Dies kann mit dem Konsolen Befehl `TelePeriod` auf minimal 10sek herabgesetzt werden.

Der Code sollte ungefähr so aussehen:

```
UID: mqtt:topic:MQTTBroker:SmartMeter
label: SmartMeter
thingTypeUID: mqtt:topic
configuration:
  payloadNotAvailable: Offline
  availabilityTopic: tele/SmartMeter/LWT
  payloadAvailable: Online
bridgeUID: mqtt:broker:MQTTBroker
channels:
  - id: TotalConsumption
    channelTypeUID: mqtt:number
    label: TotalConsumption
    description: ""
    configuration:
      unit: kWh
      formatBeforePublish: "%.3f"
      stateTopic: tele/SmartMeter/SENSOR
      transformationPattern: JSONPATH:$.SML.Total_in
  - id: CurrentConsumption
    channelTypeUID: mqtt:number
    label: CurrentConsumption
    description: ""
    configuration:
      unit: W
      formatBeforePublish: "%.3f"
      stateTopic: tele/SmartMeter/SENSOR
      transformationPattern: JSONPATH:$.SML.Power_curr
```
## Hilfe Links

Tasmota (Smart Meter Interface) https://tasmota.github.io/docs/Smart-Meter-Interface/

Befehl `Sensor53` https://tasmota.github.io/docs/Commands/#sensor53

Hilfe Info:
 
HowTo: https://wvssiot.files.wordpress.com/2020/04/howto_-tasmota-sml.pdf

PinOut von NodeMCU esp8266: https://randomnerdtutorials.com/esp8266-pinout-reference-gpios/

https://www.msxfaq.de/sonst/bastelbude/smartmeter_d0_sml.htm
