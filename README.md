# Anycubic_i3-_BLTouch

Ziel: 

Anycubic i3+ OS mit Autolevelfunktion für BLTouch. 

Version 1.1.9 oder höher.

Hilfreiche Links:

Guide zu Marlin OS insgesamt:
https://marlinfw.org/docs/configuration/configuration.html

Beispiel für die Anpassung beim Nachfolgemodell (Anycubic i3 Mega):

http://www.3d-proto.de/index.php?p=tips_autobed

Daraus Code: 
 
 2.) Software Anpassung in der neuen Marlin Firmware (2.0.x) für beide Variaten mit kap. Sensor und 3D-Touch. Im Kommentar steht die Variante Anpassungen in der Configuration.h:

Pull-up Einstellungen:
//#define ENDSTOPPULLUPS // 3DP: Auskommentieren um Pull-Ups zu aktivieren
#if DISABLED(ENDSTOPPULLUPS)
// fine endstop settings: Individual pullups. will be ignored if ENDSTOPPULLUPS is defined
...
//#define ENDSTOPPULLUP_ZMIN //kap. Sensor braucht keinen Pull-Up
#define ENDSTOPPULLUP_ZMIN //3D-Touch braucht Pull-Up
//#define ENDSTOPPULLUP_ZMIN_PROBE //kap. Sensor
#define ENDSTOPPULLUP_ZMIN_PROBE //3D-Touch


Die nächsten zwei Codezeilen können das Ausgangssignal des Sensors invertieren.
#define Z_MIN_ENDSTOP_INVERTING false // Kapazitiver Sensor LJC18A3-BZ/AX ist ein NPN-Öffner 
#define Z_MIN_ENDSTOP_INVERTING false // 3D-Touch, BL-Touch
...
...
#define Z_MIN_PROBE_ENDSTOP_INVERTING false // Kapazitiver Sensor LJC18A3-BZ/AX ist ein NPN-Öffner 
#define Z_MIN_PROBE_ENDSTOP_INVERTING false // 3D-Touch, BL-Touch

Der Min-Endstop ist gleichzeitig auch der Abstandssensor oder 3D-Touch:
#define Z_MIN_PROBE_USES_Z_MIN_ENDSTOP_PIN //Beide Varianten verwednen den Zmin-Pin für das ABL
...
...
#define FIX_MOUNTED_PROBE //kap. Sensor: einkommentieren
...
#define BLTOUCH //BL-Touch oder 3D-Touch: einkommentieren

Abstand des Sensors zum Hotend abmessen und im folgenden eintragen.
Dabei kann für den Z-Offset die folgenden Prozedur angewendet werden:
1.) Mit "G28" alle Achsen zurücksetzen. (Home all axis)
2.) Mit "M211 S0" die Software Endstops deaktivieren. Damit kann die Z-Achse temporär auch unter die Endstop-Begrenzung bewegt werden
3.) Mit "G92 Z10" die Z-Achsen Position auf 10mm Höhe setzen ohne die Achse zu bewegen.
4.) In 0,1mm Schritten den Abstand vom Hotend zum Heizbett verringern bis nur noch ein Papierblatt dazwischen passt.
5.) Mit "M114" aktuelle Z-Position abfragen und das Delta als negativen Wert (XX) in NOZZLE_TO_PROBE_OFFSET { -25, 0, XX} eintragen:
z.B. Wert (XX) = -(10mm - 9,2mm):
#define NOZZLE_TO_PROBE_OFFSET { -25, 0, 0} //3D-proto: Default { 10, 10, 0 }, Anpassen an ABL Sensor

Nach dem Homing in das Bettzentrum fahren, um beim Absenken mit dem Sensor auf der Druckplatte zu bleiben.
#define Z_SAFE_HOMING //3D-Proto einkommentiert
...
#define Z_SAFE_HOMING_X_POINT ((X_BED_SIZE) / 2) 
#define Z_SAFE_HOMING_Y_POINT ((Y_BED_SIZE) / 2) 

XY-Geschwindigkeit sollte verringert werden, um keine Geschwindigkeitsbegrenzung zu erreichen.
#define XY_PROBE_SPEED 2000 //3D-proto, reduziert auf 2000 

Einschalten des 3-Punkte Autobed Levelings für ein planes Druckbett (Glasplatte oder präzisionsgefräste Platte).
#define AUTO_BED_LEVELING_3POINT //Einfachste Variante mit ebenem Druckbett
//#define AUTO_BED_LEVELING_LINEAR
//#define AUTO_BED_LEVELING_BILINEAR
//#define AUTO_BED_LEVELING_UBL
//#define MESH_BED_LEVELING

Anpassungen in der Configuration_adv.h: Definition der Testpunkte für das einfachste 3-Punkt ABL (in mm) für die Prozedur (passt für ein Druckbett mit 200x200mm Fläche). Diese werden angefahren und ausgemessen. Hier sollte man testweise mit kleinen Werten beginnen, damit der Sensor nicht aus dem Druckbett herausfährt.
#define PROBE_PT_1_X 15 //3DP: 200x200mm Heizbett (Sensor befindet sich 25mm rechts vom Hotend)
#define PROBE_PT_1_Y 160 //3DP: 200x200mm Heizbett (Sensor befindet sich 25mm rechts vom Hotend)
#define PROBE_PT_2_X 15 //3DP: 200x200mm Heizbett (Sensor befindet sich 25mm rechts vom Hotend)
#define PROBE_PT_2_Y 20 //3DP: 200x200mm Heizbett (Sensor befindet sich 25mm rechts vom Hotend)
#define PROBE_PT_3_X 130 //3DP: 200x200mm Heizbett (Sensor befindet sich 25mm rechts vom Hotend)
#define PROBE_PT_3_Y 20 //3DP: 200x200mm Heizbett (Sensor befindet sich 25mm rechts vom Hotend)

#define BLTOUCH_DELAY 500 //3D-Touch
...
#define BLTOUCH_FORCE_SW_MODE //3D-Touch


3.) Änderung des Start G-Codes in Slic3r für alle Varianten

Zu Beginn des G-Codes muss nun G28 (Home all axis) und G29 (Auto-Bed-Leveling) durchgeführt werden. Unter Slic3r kann dies über den Custom G-Code im Fenster Start G-Code umgesetzt werden. Und fertig!
G28; home all axes: Unbedingt vorher durchführen.
G29; Auto bed level
