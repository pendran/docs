## In-App Purchasing

Per In-App Purchasing (IAP) kann deine App Geld generieren. Das OUYA Developer Kit (ODK) wurde entworfen um sowohl einfach nutzbar, als auch sicher zu sein.

#### Definitionen

**Product (Produkt)** -- ist das was der Nutzer k�uft. Es hat drei Eigenschaften:
* *Identifier (Bezeichnung)* -- an den Entwickler gerichtete Bezeichnung (Einzigartig f�r jeden Entwickler)
* *Price (Preis)* -- die Kosten des Produkts
* *Name* -- an den Nutzer gerichteter Name

**Purchasable (Erwerbung)** -- bezeichnet ein Produkt w�hrend des Einkaufs

**Receipt (Quittung)** -- Informationen �ber einen get�tigten Kauf. Hat drei Attribute:
* *Product ID (Produkt ID)* -- die Produkt Bezeichnung
* *Price (Preis)* -- Der gezahlte Betrag
* *PurchaseDate (Erwerbsdatum)* -- wann wurde der Kauf get�tigt

**Entitlements (Erm�chtigung)** -- ein Produkt das einmalig erworben werden kann und auch dann noch dem Spiel zur Verf�gung steht, wenn dieses deinstalliert wurde

**Consumable (Verbrauchbar)** -- ein Produkt das wiederholt erworben werden kann

**Subscription (Abbonnement)** -- ein Produkt das automatisch nach gesetztem Zeitplan erworben wird [noch nicht implementiert]

#### Das ODK initialisieren

Die komplette IAP funktionalit�t l�uft �ber das **OuyaFacade** Objekt. Eines dieser Objekte sollte zu Beginn der Anwendung erzeugt und f�r alle IAP Anfragen genutzt werden.
```java
	// Deine Entwickler ID findest du im Entwickler Portal
	public static final String DEVELOPER_ID = "00000000-0000-0000-0000-000000000000";

	// Erzeuge ein OuyaFacade
	private OuyaFacade OuyaFacade = new OuyaFacade();

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		OuyaFacade.init(this, DEVELOPER_ID);
		super.onCreate(savedInstanceState);
	}
```
Wenn deine Anwendung beendet wird, ist es nat�rlich h�flich das **OuyaFacade** ebenfalls zu informieren:
```java
	@Override
	protected void onDestroy() {
		OuyaFacade.shutdown();
		super.onDestroy();
	}
```
Nun sind wir bereit f�r richtige Action!

#### Erzeugung von Produkten
=======================================================
Damit Nutzer dir Geld zukommen lassen k�nnen, musst du zun�chst Produkte die sie kaufen k�nnen erzeugen. Dies geschieht auf der OUYA Website �ber das [Entwickler Portal](https://devs.ouya.tv/developers).
Nach dem einloggen, klicke auf das **Products** Men� und dann auf den **New Product** link. Dies wird dich zur Seite weiterleiten, auf der du die Produkt Bezeichnung, Preis und Name eintragen kannst.

#### Zugriff auf Produkt Informationen

Nachdem du Produkte erzeugt hast, m�chtest du vermutlich, dass deine Anwendung die Server nach aktuellem Namen und Preis abfragt. Dies machst du, indem du eine Produkt-Liste abfragst, in der Produkte, an denen du interessiert bist, mit ihrer Produkt Bezeichnung aufgelistet sind.
```java
	// Dies ist ein Satz an Produkt Bezeichnungen, welche deine Anwendung kennt
	public static final List<Purchasable> PRODUCT_ID_LIST =
		Arrays.asList(new Purchasable("sharp_sword"));
```
Da die Anfrage �ber das Internet bis zu den OUYA Server l�uft, erfolgt die Antwort �ber einen R�ckrufmechanismus. Es liegt an deiner Anwendung das entsprechende Listener-Objekt daf�r zu erzeugen. Listener erweitern die **OuyaResponseListener** Klasse und haben drei R�ckruf-Methoden:

* **onSuccess**	-- die Anfrage war erfolgreich und die angeforderten Daten liegen vor
* **onFailure**	-- die Anfrage ist fehlgeschlagen und der Fehler liegt vor
* **onCancel**	-- der Nutzer hat die Anfrage abgebrochen (beispielsweise hat er "Abbrechen" gedr�ckt, als nach einem Passwort gefragt wurde)

Nun werden wir unseren eigenen Listener erzeugen! In diesem Beispiel erweitern wir die Klasse **CancelIgnoringOuyaResponseListener** welche Abbr�che ignoriert. Demnach m�ssen wir nur die **onSuccess** und **onFailure** methoden implementieren:
```java
	OuyaResponseListener<ArrayList<Product>> productListListener =
		new CancelIgnoringOuyaResponseListener<ArrayList<Product>>() {
			@Override
			public void onSuccess(ArrayList<Product> products) {
				for(Product p : products) {
					Log.d("Product", p.getName() + " costs " + p.getPriceInCents());
				}
			}

			@Override
			public void onFailure(int errorCode, String errorMessage) {
				Log.d("Error", errorMessage);
			}
		};
```
Wie bekommen wir nun die eigentlichen Daten die wir brauchen? Indem wir eine Anfrage �ber **OuyaFacade** starten:
```java
	OuyaFacade.requestProductList(PRODUCT_ID_LIST, productListListener);
```
So einfach!

#### Einen Kauf t�tigen

Wenn Nutzer ersteinmal deine Anwendung erlebt haben, dann werden sie s�chtig nach ihr und erpicht darauf alles was du anbietest zu kaufen! Damit das funktioniert, m�ssen wir zun�chst einen neuen Listener erzeugen:
```java
	CancelIgnoringOuyaResponseListener<Product> purchaseListener =
		new CancelIgnoringOuyaResponseListener<Product>() {
			@Override
			public void onSuccess(Product product) {
				Log.d("Purchase", "Congrats you bought: " + product.getName());
			}

			@Override
			public void onFailure(int errorCode, String errorMessage) {
				Log.d("Error", errorMessage);
			}
		};
```
Mit diesem Listener ist der eigentliche Kauf nur noch eine Zeile:
```java
	Purchasable productToBuy = PRODUCT_ID_LIST.get(0);
	OuyaFacade.requestPurchase(productToBuy, purchaseListener);
```
Nun warten wir darauf, dass das Geld anf�ngt zu flie�en...

#### Gekaufte Produkte abfragen

Bis jetzt k�nnen wir Informationen zu Produkten abfragen und diese kaufen, aber was wenn ein Nutzer etwas in einer vorherigen Spielsitzung gekauft hat? Das ODK bietet uns eine M�glichkeit gekaufte Produkte aufzulisten. Richtig, daf�r ben�tigen wir ein weiteres Listener Objekt!

_Bitte beachte, dass nur Produkte vom Typ entitlement (Erm�chtigung) zur�ckgegeben werden. Dies geschieht um zu verhindern, dass bereits verbrauchte Produkte des Typs Verbrauchbar erneut gutgeschrieben werden._

Aus Sicherheitsgr�nden werden K�ufe verschl�sselt zur�ckgegeben und m�ssen in der Anwendung selbst entschl�sselt werden. Um dies zu erleichtern, kannst du die Methoden **OuyaEncryptionHelper**, sowie **decryptReceiptResponse** nutzen.

Lass uns einen Blick auf den Listener werfen:
```java
	CancelIgnoringOuyaResponseListener<String> receiptListListener =
		new CancelIgnoringOuyaResponseListener<String>() {
			@Override
			public void onSuccess(String receiptResponse) {
				OuyaEncryptionHelper helper = new OuyaEncryptionHelper();
				List<Receipt> receipts = null;
				try {
					receipts = helper.decryptReceiptResponse(receiptResponse);
				} catch (IOException e) {
					throw new RuntimeException(e);
				}
				for (Receipt r : receipts) {
					Log.d("Receipt", "You have purchased: " + r.getIdentifier())
				}
			}

			@Override
			public void onFailure(int errorCode, String errorMessage) {
				Log.d("Error", errorMessage);
			}
		};
```
Wie immer gilt, die eigentliche Anfrage ist recht simpel:
```java
	OuyaFacade.requestReceipts(receiptListListener);
```
Das entschl�sseln des Kaufes geschieht innerhalb der Anwendung um hacking zu verhindern. Durch das entschl�sseln in jeder Anwendung gibt es keinen "besonderen st�ck Code", den ein Hacker angreifen kann um die Verschl�sselung aller Anwendungen zu brechen. In Zukunft werden wir Entwickler dazu aufrufen die **decryptReceiptResponse** Methode zu vermeiden. Sie werden diese Methode in ihre Anwendungen verschieben m�ssen und ihr Verhalten *verzerren* (for-Schleifen in while-Schleifen �ndern, und so weiter) um dabei zu Helfen die Dinge noch sicherer zu machen.
Im Moment ist das ODK noch stark in der Entwicklung, daher wird die helper Methode dabei helfen dich von den �nderungen "unter der Haube" zu isolieren.

#### Den Nutzer identifizieren

Wenn deine Anwendungen mit einem externen Server spricht, ist es oftmals notwendig eine einzigarte Bezeichnung f�r einen aktuellen Nutzer zu bekommen (zum Beispiel um High-Scores auf einer Website zu speichern). Dies geschieht in der normalen Vorlage einer Listener Erzeugung:
```java
	CancelIgnoringOuyaResponseListener<String> gamerUuidListener =
		new CancelIgnoringOuyaResponseListener<String>() {
			@Override
			public void onSuccess(String result) {
				Log.d("UUID", "UUID is: " + result);
			}

			@Override
			public void onFailure(int errorCode, String errorMessage) {
				Log.d("Error", errorMessage);
			}
		};
```
Dann die Anfrage:
```java
	OuyaFacade.requestGamerUuid(gamerUuidListener);
```
**Beachte**: Diese Spiel-UUIDs sind unterschiedlichen von Entwickler zu Entwickler; zwei Apps von unterschiedlichen Entwicklern, welche die UUID des gleichen Nutzers abfragen werden unterschiedliche Ergebnisse erhalten.
