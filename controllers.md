## Controller

Einer der großen Vorteile der OUYA Konsole ist, dass Spieler einen *richtigen* Kontroller benutzen können! Der OUYA Controller hat:
- vier digitale Buttons (O, U, Y und A)
- ein Vierweg digital Pad (D-Pad)
- zwei analoge Joysticks (LS, RS)
- zwei digitale Buttons die aktiviert werden, wenn man die Joysticks nach unten (in den Controller) drückt (L3, R3)
- zwei digitale bumper Buttons (L1, R1)
- zwei analoge Trigger (L2, R2)
- ein Touchpad, welches so eingestellt ist, dass es sich wie eine Maus verhält (Touchpad)

**Beachte:** Die analogen Trigger funktionieren auch als digitale Buttons, mit einem Schwellenwert von 0.5 für den analogen Wert.

Da Controller Schnittstellen extrem wichtig sind, haben wir einiges an Arbeit geleistet um dir das Leben einfacher zu gestalten.

##### Konstanten

Die **OuyaController** Klasse enthält einige OUYA-spezifische Konstanten für Buttons und Achsen. Eine kleine Auswahl siehst du hier.
```java
public static final int BUTTON_O;
public static final int BUTTON_U;
public static final int BUTTON_Y;
public static final int BUTTON_A;
```

Mit diesen Konstanten ist es völlig Akzeptabel den Input über die standard **onKeyDown**, **onKeyUp**, oder **onGenericMotionEvent** Methoden zu verarbeiten. Wenn auf diese Weise mit dem Input umgegangen wird, kann die Controller ID durch **OuyaController.getPlayerNumByDeviceId()** auf folgende Weise abgefragt werden.

```java
@Override
public boolean onKeyDown(final int keyCode, KeyEvent event){
    //Hole die Spielernummer
    int player = OuyaController.getPlayerNumByDeviceId(event.getDeviceId());       
    boolean handled = false;
    
    //Bearbeite den Input
    switch(keyCode){
        case OuyaController.BUTTON_O:
			//Nun kennst du die gedrückte Taste sowie die Spieler Nummer des Auslösers
            //doSomethingWithKey();
            handled = true;
            break;
    }
    return handled || super.onKeyDown(keyCode, event);
}

@Override
public boolean onGenericMotionEvent(final MotionEvent event) {
    //Hole die Spielernummer
    int player = OuyaController.getPlayerNumByDeviceId(event.getDeviceId());    
    
	//Hole alle Achsen für dieses Event
    float LS_X = event.getAxisValue(OuyaController.AXIS_LS_X);
    float LS_Y = event.getAxisValue(OuyaController.AXIS_LS_Y);
    float RS_X = event.getAxisValue(OuyaController.AXIS_RS_X);
    float RS_Y = event.getAxisValue(OuyaController.AXIS_RS_Y);
    float L2 = event.getAxisValue(OuyaController.AXIS_L2);
    float R2 = event.getAxisValue(OuyaController.AXIS_R2);
    
    //Mach etwas mit dem Input
    //updatePlayerInput(player, LS_X, LS_Y, RS_X, RS_Y, L2, R2);
    
    return true;
}
```

##### Unterscheiden zwischen analogem Joystick und Touchpad

Sowohl der analoge Joystick als auch das Touchpad lassen ihren Status über **onGenericMotionEvent** erfragen. Um zwischen ihnen zu Unterscheiden kannst du die **MotionEvent* action abfragen, für die gilt:

* Analoger Joystick = MotionEvent.ACTION_MOVE
* Touchpad == MotionEvent.ACTION_HOVER_MOVE

Beispiel:

```java
@Override
public boolean onGenericMotionEvent(final MotionEvent event) {
    //Speicher die Spielernummer
    int player = OuyaController.getPlayerNumByDeviceId(event.getDeviceId());    
    
    switch(event.getActionMasked()){
        //Joystick
        case MotionEvent.ACTION_MOVE:
            float LS_X = event.getAxisValue(OuyaController.AXIS_LS_X);            
            //mach irgendwelche Dinge mit dem Joystick
            break;
            
        //Touchpad
        case MotionEvent.ACTION_HOVER_MOVE:
			//Gebe die Pixel Koordinaten des Cursors aus
            Log.i("Touchpad", "Cursor X: " + event.getX() + "Cursor Y: " + event.getY());
            break;
    }
    
    return true;
}
```

##### Jederzeit den Status abfragen

Wenn du die extra Flexibilität haben möchtest, den Controller Status jederzeit abzufragen, kannst du die restliche **OuyaController** Klasse verwenden. 
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

Sobald **OuyaController** die Events empfängt, kannst du eine Instanz der Klasse in einer von zwei Möglichkeiten erhalten: Geräte-ID, oder Spieler Nummer:

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

Damit kannst du dich ganz auf das entwickeln eines großartigen Spiels konzentrieren, anstatt auf das korrekte Umgehen mit Input.