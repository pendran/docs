## Installationanleitung für das Ouya Development Kit

##### MacOS
Lade dir das [Android SDK und Tools](http://developer.android.com/sdk/index.html) hertuner und installiere es auf deinem Mac oder PC, indem du die beiligende Anleitung befolgst.

Starte den Android SDK Manager durch ausführen von ([ausführliche Anleitung](http://developer.android.com/sdk/installing/adding-packages.html)):
```bash 
./android sdk
```

Installiere die folgenden Packete:

- **Tools**: Beinhaltet sowohl das Android SDK als auch die Android SDK Plattform Tools
- **Android 4.1 (API 16)**: SDK Plattform
- **Android 4.0 (API 14)**: SDK Plattform
- **Extras**: Android Support Library


Installiere die Java-runtime, wenn du dazu aufgefordert wirst.

Du musst einige Pfade zur PATH adden. Angenommen du hast den SDK Ordner an der folgenden Stelle `~/android/android-sdk-macosx`, dann öffne ein Terminal und füge die folgenden drei Zeilen zu deiner `.bashrc` hinzu:

```bash
export PATH=$PATH:~/android/android-sdk-macosx/tools
export PATH=$PATH:~/android/android-sdk-macosx/platform-tools
export ANDROID_HOME=~/android/android-sdk-macosx
```

Möglicherweise musst du diese `.bashrc` Einträge anpassen, sofern du eine eigene SDK Ordner Position genutzt hast. 


##### Windows
Folgt noch.

##### Eclipse
Folgt noch.

##### Entwickeln mit der ODK
Nutze die Android API Level 16 (Android 4.1 "Jelly Bean"), wenn du für die OUYA Konsole entwickelst.

Um die OUYA API nutzen zu können, musst du die `ouya-sdk.jar` in deine Projekt Libraries einbinden, ebenso die `guava-r09.jar` und `commons-lang-2.6.jar`. Diese findest du im `libs` Ordner.

Für Informationen bezüglich der API Befehle, wirf bitte einen Blick in die OUYA API Referenzdokumentation.

Um den Beispielcode laufen zu lassen, öffne das Projekt in `iap-sample-app` und folge den Schritten in der `README.txt` Datei.

Damit erkannt wird, dass deine Anwendung oder dein Spiel für die OUYA gemacht ist, musst du eine OUYA intent Kategorie im Manifest Eintrag deiner Hauptaktivität (main activity) hinzufügen.
Nutze dafür “ouya.intent.category.GAME” oder “ouya.intent.category.APP”.

```xml
<activity android:name=".GameActivity" android:label="@string/app_name">
  <intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
    <category android:name="ouya.intent.category.GAME"/>
  </intent-filter>
</activity>
```

Das Anwendungsbild, welches im Launcher angezeigt wird, ist in der APK selbst eingebettet. Die erwartete Datei befindet sich in `res/drawable-xhdpi/ouya_icon.png` und die Bildgröße muss 732x412 für Spiele oder 412x412 für Anwendungen sein.

#### Hardware Einstellungen

Um mit der Entwicklung von Software anzufangen, bevor du in Besitz der OUYA Konsole bist, kannst du einen Android Emulator, oder ein standard Android Tablet benutzen.

##### Software

Die OUYA Konsolenhardware beinhaltet bereits den OUYA Launcher, aber wenn du einen Emulator, oder Android Tablet verwendest, musst du Launcher manuell installieren. Diese Datei ist ebenfalls im OUYA ODK Packet enthalten:

Um den Launcher zu installieren, führe folgendes aus:
```bash
adb install -r ouya-framework.apk
adb install -r ouya-launcher.apk
```

**Beachte**: Wenn der OUYA Launcher nicht installiert ist, werden einige ODK Features nicht korrekt funktionieren.

##### Emulator

Wenn du einen Emulator nutzt, dann konfiguriere das Android Virutal Device folgendermaßen:

- **Resolution (Auflösung)**: 1920x1080 oder 1280x720, wie gewünscht
- **Hardware Back/Home keys (Zurück/Home Tasten)**: ja (du musst dies zu den Hardwareparametern hinzufügen)
- **D-Pad support**: ja (du musst dies zu den Hardwareparametern hinzufügen)
- **Target (Zielversion)**: Android 4.1 - API Level 16
- **CPU/ABI**: Intel Atom x86
- **Device RAM size (Arbeitsspeicher des Gerätes)**: 1024

Wir empfehlen die Benutzung des Intel Atom x86 CPU/ABI und Intel's HAXM Erweiterung um sicherzustellen, dass die Emulator Leistungen ausreichend für die Spieleentwicklung sind. Wenn du low-level Code entwickelst beachte, dass das Gerät auf ARM basiert. Dementsprechend solltest du für die ARM Architektur entwickeln und ein Emulator AVD nutzen, deren CPU/ABI auf ARM Architektur eingestellt ist.

Die OUYA Konsole hat keine Hardware Knöpfe für Zurück oder Menü, also sollten deine Spiele sich auch nicht auf diese beziehen. Das setzen der Hardware Tasten Emulator Eigenschaft versteckt die Android Navigationsbar, woduch der Emulator die kompletten 1920x1080, beziehungsweise 1280x720 nutzen kann, so wie es die OUYA Konsole macht.

**Beachte**: Wenn du mit dem Emulator entwickelst, ist es nicht möglich die kompletten OUYA Kontroller Buttons und Feature zu simulieren.

##### Android Tablet

Wenn du ein standard Android Tablet verwendest, dann empfehlen wir ein Tablet zu nutzen, dessen Display Auflösung so nah wie möglich an 1920x1080 oder 1280x720 heran reicht.

**Beachte**: Die Android Navigationsbar wird einen Teil des Bildschirms auf standard Tablets einen Teil des Bildschirms in Anspruch nehmen, was bei der OUYA Konsole nicht der Fall ist.

Der OUYA Controller kombiniert einen standard Controller (zwei Joysticks, ein D-Pad, vier Spiel Buttons, zwei Schulter Buttons und zwei Trigger) mit einem Touchpad. Zum Testen empfehlen wir einen Xbox 360 (Kabel) USB Controller in zusammenarbeit mit einer Maus oder einem Touchpad zu verwenden.

##### Bildschirmauflösung

Die OUYA Konsole unterstützt ausschließlich 720p oder 1080p.

Für OpenGL-basierte Spiele empfehlen wir einen Render-Puffer von 1920x1080 (wenn 1080p angestrebt sind), oder 1280x720 (wenn 720p angestrebt sind) zu erzeugen. Wenn dies nicht der Display-Auflösung deines Gerätes entspricht, wird der Spiel Bildschirm vielleicht als Fenster oder skaliert dargestellt, je nach Gerätehersteller definiertem Verhalten.

Fpr Spiele die das Android UI Framework nutzen, befolge die Android bewährten Methoden zur Entwicklung von Applikationen für xhdpi-large und tvdpi-large Displays. Besuche [developer.android.com](http://developer.android.com) für weitere Informationen.
