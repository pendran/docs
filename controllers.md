## Controller

Einer der gro�en Vorteile der OUYA Konsole ist, dass Spieler einen *richtigen* Kontroller benutzen k�nnen! Der OUYA Controller hat:
- vier digitale Buttons (O, U, Y und A)
- ein Vierweg digital Pad (D-Pad)
- zwei analoge Joysticks (LS, RS)
- zwei digitale Buttons die aktiviert werden, wenn man die Joysticks nach unten (in den Controller) dr�ckt (L3, R3)
- zwei digitale bumper Buttons (L1, R1)
- zwei analoge Trigger (L2, R2)
- ein Touchpad, welches so eingestellt ist, dass es sich wie eine Maus verh�lt (Touchpad)

**Beachte:** Die analogen Trigger funktionieren auch als digitale Buttons, mit einem Schwellenwert von 0.5 f�r den analogen Wert.

Da Controller Schnittstellen extrem wichtig sind, haben wir einiges an Arbeit geleistet um dir das Leben einfacher zu gestalten.

##### Konstanten

Die **OuyaController** Klasse enth�lt einige OUYA-spezifische Konstanten f�r Buttons und Achsen. Eine kleine Auswahl siehst du hier.
```java
public static final int BUTTON_O;
public static final int BUTTON_U;
public static final int BUTTON_Y;
public static final int BUTTON_A;
```

Mit diesen Konstanten ist es v�llig Akzeptabel den Input �ber die standard **onKeyDown**, **onKeyUp**, oder **onGenericMotionEvent** Methoden zu verarbeiten.

##### Jederzeit den Status abfragen

Wenn du die extra Flexibilit�t haben m�chtest, den Controller Status jederzeit abzufragen, kannst du die restliche **OuyaController** Klasse verwenden. 
Das bedeutet deine activity sollte alle **onKeyDown**, **onKeyUp**, oder **onGenericMotionEvent** Aufrufe weiterleiten an **OuyaController**:

```java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    boolean handled = OuyaController.onKeyDown(keyCode, event);
    return handled || super.onKeyDown(keyCode, event);
}

@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    boolean handled = OuyaController.onKeyUp(keyCode, event);
    return handled || super.onKeyUp(keyCode, event);
}

@Override
public boolean onGenericMotionEvent(MotionEvent event) {
    boolean handled = OuyaController.onGenericMotionEvent(event);
    return handled || super.onGenericMotionEvent(event);
}
```

Sobald **OuyaController** die Events empf�ngt, kannst du eine Instanz der Klasse in einer von zwei M�glichkeiten erhalten: Ger�te-ID, oder Spieler Nummer:

```java
OuyaController c = OuyaController.getControllerByDeviceId(deviceId);
OuyaController c = OuyaController.getControllerByPlayer(playerNum);
```

Jetzt ist es einfach den Button oder Achsen-Wert abzufragen:

```java
float axisX = c.getAxisValue(OuyaController.AXIS_LSTICK_X);
float axisY = c.getAxisValue(OuyaController.AXIS_LSTICK_Y);
boolean buttonPressed = c.getButton(OuyaController.BUTTON_O);
```

Damit kannst du dich ganz auf das entwickeln eines gro�artigen Spiels konzentrieren, anstatt auf das korrekte Umgehen mit Input.