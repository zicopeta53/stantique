# M2 Marine App → Google Play: Schritt für Schritt

**Stand:** 18. August 2026 · geprüfter Projektstand: `m2-marine-letters-app`, App-Version 1.0.15, `versionCode` 16

---

## Das Wichtigste zuerst (bitte wirklich lesen)

**1. In deinem ZIP ist keine APK drin.**
Ich habe die Datei ausgepackt und durchsucht: es ist der **Quellcode** der App (Expo / React Native), keine fertige `.apk` und keine `.aab`.

**2. Google Play nimmt sowieso keine APK an.**
Für neue Apps verlangt Google eine **AAB-Datei** (`.aab`, "Android App Bundle"). Die APK von Manus ist nur zum Selbst-Testen auf dem Handy gut.

**3. Du musst also einmal selbst bauen.** Das ist halb so wild — dein Projekt ist dafür schon vorbereitet (`eas.json` hat bereits ein `production`-Profil, das eine AAB erzeugt).

**4. Der Zeitplan ist wahrscheinlich länger, als du denkst.**
Wenn dein Play-Console-Konto ein **persönliches Konto** ist, das **nach dem 13.11.2023** erstellt wurde, verlangt Google vor der öffentlichen Veröffentlichung:
> einen geschlossenen Test mit **mindestens 12 Testern**, die **14 Tage am Stück** angemeldet bleiben.

Das heißt: Selbst wenn heute alles perfekt wäre, bist du frühestens in **~3 Wochen** öffentlich live. Deshalb ist Schritt 0 unten der wichtigste Schritt von allen.

---

## Schritt 0 — Prüfe zuerst, welches Konto du hast (5 Minuten)

1. Öffne die **Google Play Console** → Zahnrad **Einstellungen** → **Entwicklerkonto** → **Kontodetails**.
2. Schau auf **Kontotyp**:
   - **"Persönlich" / "Personal"** → die 12-Tester-Regel gilt sehr wahrscheinlich für dich.
   - **"Organisation"** (mit D-U-N-S-Nummer) → die Regel gilt nicht, du kannst direkt in Produktion.
3. Prüfe außerdem: Sind die **25 USD** einmalige Gebühr bezahlt und ist deine **Identität verifiziert**? Ohne das geht gar nichts.

Wenn "Persönlich": Fang **sofort** an, 12 Leute zu sammeln (Familie, Freunde, Kunden, Mitarbeiter — jeder mit Google-Konto). Du brauchst ihre **Gmail-Adressen**. Das ist erfahrungsgemäß der Punkt, an dem Leute 2 Wochen verlieren.

---

## Schritt 1 — Deinen Computer vorbereiten (30 Minuten, einmalig)

Du brauchst einen normalen Windows- oder Mac-Rechner.

1. **Node.js** installieren: https://nodejs.org → die **LTS**-Version herunterladen und installieren, alles auf "Weiter" klicken.
2. **Terminal öffnen** (Windows: "PowerShell" im Startmenü suchen · Mac: "Terminal" in Spotlight suchen).
3. Diese Zeile eintippen und Enter drücken:
   ```bash
   npm install -g pnpm eas-cli
   ```
4. **Gratis-Konto bei Expo** anlegen: https://expo.dev/signup
5. Im Terminal anmelden:
   ```bash
   eas login
   ```

---

## Schritt 2 — Projekt entpacken und startklar machen (10 Minuten)

1. Entpacke `m2marinelettersappcurrent1.0_1.zip` z. B. auf den Desktop.
2. Im Terminal in den Ordner wechseln (Pfad anpassen):
   ```bash
   cd ~/Desktop/m2marinelettersappcurrent1.0_1/m2-marine-letters-app
   ```
3. Alle Bausteine herunterladen (dauert ein paar Minuten):
   ```bash
   pnpm install
   ```
4. Kurzer Selbsttest, ob alles gesund ist:
   ```bash
   pnpm run check
   pnpm test
   pnpm run verify:android
   ```
   Alle drei müssen "grün" durchlaufen. Wenn nicht: nicht weitermachen, sondern den Fehler beheben lassen.

---

## Schritt 3 — Die EINE Entscheidung, die du nie mehr rückgängig machen kannst

Der **Paketname** deiner App ist aktuell:

```
com.app.m2marinelettersapp
```

Nach dem allerersten Upload ist dieser Name **für immer** festgelegt — er ist die Identität deiner App bei Google. `com.app.…` ist ein generischer Platzhalter von Manus.

**Empfehlung:** ändere ihn jetzt auf etwas Markenmäßiges, z. B. `com.m2marineletters.app`.

So geht's: Datei `app.config.ts` öffnen, diese Zeile suchen …

```ts
const rawBundleId = "com.app.m2marinelettersapp";
```

… und den Text in den Anführungszeichen ersetzen:

```ts
const rawBundleId = "com.m2marineletters.app";
```

Speichern. Fertig. (Nur Buchstaben, Zahlen und Punkte, jeder Abschnitt muss mit einem Buchstaben anfangen.)

---

## Schritt 4 — Projekt mit deinem Expo-Konto verbinden (5 Minuten)

```bash
eas init
```

Das legt eine Projekt-ID an. Falls der Befehl meckert, dass er `app.config.ts` nicht automatisch bearbeiten kann, trage es von Hand ein — in `app.config.ts`, direkt unter `slug`:

```ts
const config: ExpoConfig = {
  name: env.appName,
  slug: env.appSlug,
  owner: "DEIN-EXPO-BENUTZERNAME",
  extra: { eas: { projectId: "DIE-ID-DIE-EAS-INIT-ANZEIGT" } },
  version: "1.0.15",
  // ... Rest bleibt unverändert
```

---

## Schritt 5 — Erst testen, dann veröffentlichen (Pflicht!)

Bau dir zuerst eine **Test-APK** für dein eigenes Handy:

```bash
eas build --platform android --profile preview
```

Der Bau läuft auf Expos Servern (10–30 Minuten, je nach Warteschlange). Am Ende bekommst du einen Link + QR-Code. Den auf einem **echten Android-Handy** öffnen, APK herunterladen, installieren ("Installation aus unbekannter Quelle" erlauben).

**Diese Dinge musst du auf dem echten Gerät durchtesten:**

- [ ] Bootsfoto auswählen (System-Fotoauswahl öffnet sich)
- [ ] Kamera-Aufnahme funktioniert
- [ ] Schrift entwerfen → Vorschau/Mockup wird korrekt angezeigt
- [ ] **"Download your mockup"** → Bild landet wirklich in der Galerie
- [ ] **"Share your mockup"** → Android-Teilen-Menü öffnet sich mit dem Bild
- [ ] **Checkout** → Shopify öffnet sich im Browser mit dem richtigen Warenkorb
- [ ] Alle Seiten durchklicken: Shop, Design, Ideas, FAQ, Legal, Contact, About, Reseller
- [ ] "Contact" → dein E-Mail-Programm öffnet sich mit vorbereiteter Mail

Erst wenn das alles sitzt, geht es weiter.

---

## Schritt 6 — Deinen Signaturschlüssel sichern (SEHR wichtig)

Beim ersten Build erzeugt EAS automatisch einen **Keystore** (deinen Signaturschlüssel). Ohne ihn kannst du deine App später **nie wieder updaten**.

```bash
eas credentials
```
→ Android auswählen → Keystore → herunterladen.

Die Datei + das Passwort in deinen **Passwortmanager** und in ein **Backup** legen. Nicht auf dem Desktop liegen lassen.

Aktiviere in der Play Console zusätzlich **Play App Signing** (ist bei neuen Apps Standard) — das ist dein Sicherheitsnetz.

---

## Schritt 7 — Das echte Paket für Google bauen

```bash
eas build --platform android --profile production
```

Ergebnis: eine **`.aab`-Datei**. Herunterladen und merken, wo sie liegt.

> **Merke für später:** Bei *jedem* neuen Upload musst du in `app.config.ts` den `versionCode` erhöhen (aktuell `16` → nächster Upload `17`) und am besten auch `version` (`"1.0.15"` → `"1.0.16"`). Google lehnt sonst den Upload ab.

---

## Schritt 8 — App in der Play Console anlegen

1. Play Console → **Alle Apps** → **App erstellen**
2. Ausfüllen:
   - **App-Name:** `M2 Marine` (max. 30 Zeichen)
   - **Standardsprache:** Englisch (US) — die App ist auf Englisch
   - **App oder Spiel:** App
   - **Kostenlos oder kostenpflichtig:** **Kostenlos** (der Verkauf läuft über Shopify, nicht über Google)
   - Erklärungen abhaken

---

## Schritt 9 — Store-Eintrag ("Store listing") vorbereiten

Das brauchst du an Material:

| Was | Format | Tipp |
|---|---|---|
| App-Icon | 512 × 512 px, PNG | ist im Projekt: `assets/images/icon.png` |
| Feature-Grafik | 1024 × 500 px | Bootsfoto + Logo + Claim, z. B. in Canva |
| Handy-Screenshots | mind. 2, besser 4–8 | einfach aus deiner Test-APK abfotografieren (Screenshot am Handy) |
| Kurzbeschreibung | max. 80 Zeichen | z. B. *"Design and order custom stainless steel boat lettering."* |
| Vollständige Beschreibung | max. 4000 Zeichen | Was die App kann: Namen finden, Schrift wählen, Mockup aufs eigene Bootsfoto, bestellen |
| Kategorie | — | **Shopping** |
| Kontakt-E-Mail | — | `msquare.marine.solutions@gmail.com` |

---

## Schritt 10 — "App-Inhalte" ausfüllen — hier sind deine konkreten Antworten

Play Console → **Richtlinien** → **App-Inhalte**. Ich habe den Code durchgesehen; so sieht die Realität deiner App aus:

| Frage in der Console | Deine Antwort | Warum |
|---|---|---|
| **Datenschutzerklärung (URL)** | `https://m2marineletters.com/policies/privacy-policy` | Diese URL ist auch in der App verlinkt. **Vorher im Browser prüfen, dass sie wirklich lädt.** |
| **App-Zugriff** | "Alle Funktionen sind ohne besonderen Zugang verfügbar" | Die App hat keinen Login-Bildschirm. |
| **Anzeigen** | **Nein**, keine Werbung | Im Code ist kein einziges Werbe-SDK. |
| **Inhaltseinstufung** | Fragebogen ausfüllen: Kategorie *Shopping / Utility*, alle Fragen zu Gewalt, Sex, Drogen, Glücksspiel → **Nein** | reine Shopping-App |
| **Zielgruppe** | **18+** (mindestens 16+), **nicht** für Kinder | Es ist ein Bestellshop für Bootsbesitzer. |
| **Ist es eine News-App?** | **Nein** | Der "From the workshop"-Bereich sind eigene Firmeninhalte, keine Nachrichten. |
| **Regierungs-App / Finanz-App / Gesundheits-App** | **Nein** | — |
| **Kontolöschung** | "Meine App bietet keine Kontoerstellung an" | stimmt — es gibt keine Nutzerkonten. |

### Datensicherheit ("Data safety") — so füllst du es aus

- **Erhebt oder teilt deine App Nutzerdaten?** → **Ja**
- Angeben als **erhoben UND geteilt**:
  - **Name**, **E-Mail-Adresse**, **Telefonnummer**, **Adresse**
  - Zweck: **App-Funktionalität** (Bestellabwicklung/Versand)
  - Grund: Beim Checkout gibst du diese Daten in der App ein, und sie werden an **Shopify** übergeben.
  - "Ist die Angabe optional?" → **Ja** (nur wenn der Nutzer bestellen will)
- **Fotos** → **nicht erhoben.** Das Bootsfoto bleibt auf dem Gerät; es wird nur gespeichert oder geteilt, wenn der Nutzer selbst darauf tippt.
- **Zahlungsdaten** → **nicht in der App erhoben** (Bezahlung passiert im externen Shopify-Checkout).
- **Verschlüsselt bei der Übertragung?** → **Ja** (alles läuft über HTTPS)
- **Können Nutzer Löschung beantragen?** → **Ja** (per E-Mail, steht so in deiner Privacy Policy)

> Wichtig: Diese Erklärung muss zur **fertigen** App passen. Wenn später doch noch Tracking oder Werbung dazukommt, musst du sie ändern.

---

## Schritt 11 — Hochladen und testen

1. **Testen und veröffentlichen** → **Interner Test** → **Neuen Release erstellen** → deine `.aab` hochladen.
2. Dich selbst als Tester eintragen, App über den Play-Store-Link installieren → prüfen, dass die Store-Version wirklich läuft.
3. Den **Pre-Launch-Report** abwarten (Google lässt die App automatisch auf echten Geräten laufen) und Abstürze/Warnungen anschauen.
4. Dann **Geschlossener Test** → 12+ Tester per E-Mail-Liste einladen → **14 Tage durchlaufen lassen** (falls persönliches Konto).
5. Danach: **Produktion** → Release erstellen → **Zur Prüfung einreichen**. Prüfung dauert meist 1–7 Tage.

---

## Schritt 12 — Die echten Blocker, die NICHT im Code liegen

Deine eigenen Projektnotizen (`notes/google-play-readiness.md`) listen drei Dinge, die vor dem Live-Gang erledigt sein müssen. Ich habe sie geprüft, sie sind weiterhin gültig:

### 🔴 1. Shopify-Richtlinien enthalten sichtbare Platzhalter
- Refund-Policy enthält `[INSERT RETURN ADDRESS]`
- Terms enthalten `My Store` und `[LINK TO … POLICY]`

**Google-Prüfer klicken diese Links an.** Platzhaltertexte in den Shop-Richtlinien sind ein klassischer Ablehnungsgrund und wirken auf Kunden unseriös.
→ Shopify Admin → **Einstellungen** → **Richtlinien** → alle vier Texte sauber ausformulieren, echte Rücksendeadresse eintragen.

### 🔴 2. Versandangaben widersprechen sich
Die App zeigt "weltweiter Gratisversand" und "USA DDP inklusive". Die Shopify-Versandseite spricht von Aufpreisen und nicht enthaltenen Abgaben.
→ Einen Text festlegen und App, Website und Shopify-Seite angleichen. Widersprüchliche Preisangaben sind ein Verbraucherschutz-Thema, kein Schönheitsfehler.

### 🟡 3. Shopify-Checkout einmal komplett echt durchspielen
Kaufe dir selbst ein günstiges Testprodukt bis zur Bestätigung durch — inklusive Rückkehr aus dem Checkout in die App.

### ✅ Was gut ist
Der Verkauf **physischer** Buchstaben über einen externen Shopify-Checkout ist bei Google ausdrücklich erlaubt. Google Play Billing wäre nur für digitale Güter Pflicht. Da bist du auf der sicheren Seite.

---

## Dein Zeitplan, realistisch

| Wann | Was |
|---|---|
| **Heute** | Schritt 0 (Kontotyp prüfen!) + Schritt 1 & 2 + 12 Tester-Gmails sammeln |
| **Tag 2** | Paketname entscheiden, Test-APK bauen, auf echtem Handy durchtesten |
| **Tag 3** | Shopify-Richtlinien bereinigen, Versandtexte angleichen, Testkauf |
| **Tag 4** | Screenshots + Grafiken erstellen, Store-Eintrag + App-Inhalte ausfüllen |
| **Tag 5** | Produktions-AAB bauen, Internen Test hochladen, Pre-Launch-Report prüfen |
| **Tag 6** | Geschlossenen Test mit 12 Testern starten |
| **Tag 6 + 14** | Produktion beantragen |
| **+ 1–7 Tage** | Google-Prüfung → **live** 🎉 |

---

## Wenn etwas klemmt

| Problem | Lösung |
|---|---|
| `eas build` sagt "project not configured" | Schritt 4 wiederholen, `owner` und `projectId` von Hand eintragen |
| Build schlägt fehl | Den Fehler-Log-Link öffnen, den EAS ausgibt — dort steht die echte Ursache |
| Warteschlange dauert ewig | Gratis-Kontingent von Expo ist begrenzt; entweder warten oder einen Monat Expo-Plan buchen |
| Google lehnt ab | Der Ablehnungsgrund steht immer konkret in der Console — meist Datenschutz-URL, Data Safety oder Richtlinien-Platzhalter |

---

*Hinweis: Dies ist eine technische Anleitung, keine Rechtsberatung. Die Shop-Richtlinien, Versand- und Steuerangaben solltest du vor dem öffentlichen Verkauf rechtlich prüfen lassen.*
