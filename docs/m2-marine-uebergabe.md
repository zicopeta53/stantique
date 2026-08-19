# Übergabe: M2 Marine → Google Play

**Diese Datei zuerst lesen.** Sie beschreibt den geprüften Ist-Zustand, die
vorgenommenen Änderungen und die offenen Punkte bis zur Veröffentlichung.

**Übergabedatum:** 18. August 2026
**Projektstand:** `m2-marine-letters-app`, App-Version 1.0.15, `versionCode` 16

---

## 1. Ausgangslage und wichtigste Klarstellung

Der Nutzer ging davon aus, eine fertige APK übergeben zu haben. Das ist nicht der Fall.
Zwei ZIP-Archive wurden geprüft (`…current1.0_1.zip` und `…VGRavXCrIQIFUjqZ.zip`) —
beide enthalten **ausschließlich Quellcode**, keine `.apk`, keine `.aab`, keinen
Keystore und keinen `android/`-Ordner. Die beiden Archive unterscheiden sich nur in
einer hinzugefügten `CLAUDE.md` und einer geänderten `todo.md`.

Die auf dem Handy des Nutzers installierte APK stammt aus einem Manus-Build. Sie ist
für die Veröffentlichung **unbrauchbar**, weil Google Play für neue Apps ein Android
App Bundle (`.aab`) verlangt und weil der zugehörige Signaturschlüssel bei Manus
liegt, nicht beim Nutzer. Ohne eigenen Schlüssel sind spätere Updates unmöglich.

**Konsequenz:** Es muss einmalig ein eigener Release-Build mit eigenem Schlüssel
erzeugt werden. Das Projekt ist dafür vorbereitet (siehe Abschnitt 3 und 4).

---

## 2. Verifizierter Zustand

Alle drei Prüfungen wurden in dieser Umgebung ausgeführt und sind **grün**:

| Prüfung | Befehl | Ergebnis |
|---|---|---|
| TypeScript | `pnpm run check` | 0 Fehler |
| Testsuite | `pnpm test` | 46 bestanden, 1 übersprungen (13 Dateien) |
| Android-Preflight | `pnpm run verify:android` | JS-Bundle exportiert, 4,81 MB |

`pnpm install` läuft sauber durch (1166 Pakete, ~12 s). Node 22, pnpm 9.12.0.

### Funktionale Architektur, die für die Store-Angaben relevant ist

Ich habe den Code daraufhin gelesen, weil davon die Play-Console-Antworten abhängen:

- **Kein Login-Bildschirm.** `app/oauth/callback.tsx` existiert, ist aber von keinem
  UI aus erreichbar. → In der Console: „Alle Funktionen ohne Zugang verfügbar",
  keine Kontolöschungs-URL nötig.
- **Kontaktformular** (`lib/contact-form.ts`) baut nur einen `mailto:`-Link und
  öffnet das E-Mail-Programm. Es überträgt selbst keine Daten.
- **Checkout** (`app/checkout.tsx` + `lib/shopify-cart-permalink.ts`) baut einen
  Shopify-Cart-Permalink auf `m2marineletters.com` und öffnet ihn im externen
  Browser. Name, E-Mail, Telefon und Adresse gehen dadurch **an Shopify** — das
  ist in der Data-Safety-Erklärung als „erhoben und geteilt" anzugeben.
- **Bootsfotos** bleiben lokal. Sie werden nur gespeichert oder geteilt, wenn der
  Nutzer aktiv darauf tippt. → nicht als „erhoben" deklarieren.
- **Backend wird nicht benötigt.** Einzige tRPC-Nutzung ist `app/ideas.tsx`
  (KI-Namensgenerator). Schlägt der Aufruf fehl, greift ein lokaler Fallback
  (`catch { setIdeas(createIdeas(...)) }`). Die App funktioniert also vollständig
  ohne laufenden Server; nur die KI-Vorschläge sind dann deterministisch statt generiert.
- **Keine Werbe-SDKs** im Code. → „Keine Anzeigen".
- Der Bereich „From the workshop" (`app/news.tsx`) ist eigener Firmentext, keine
  Nachrichtenaggregation. → **keine** News-App.

### Design-Assets: was im Projekt liegt und was nicht

| Asset | Ort | Status |
|---|---|---|
| 25 Schriftarten (.ttf) | `assets/fonts/` | im Projekt, 6,0 MB |
| 25 Schrift-Vorschaubilder | `assets/font-previews/` | im Projekt, 128 KB |
| App-Icon, Adaptive Icons, Splash, Header-Logo | `assets/images/` | im Projekt, 1,6 MB |
| **Produktfoto (Shop + Produktseite)** | `cdn.shopify.com` | **nicht im Projekt** |
| **Hero-Bild (Startseite)** | `cdn.shopify.com` | **nicht im Projekt** |

Die letzten beiden werden zur Laufzeit von der Shopify-CDN nachgeladen
(`app/shop.tsx:7`, `app/product/[slug].tsx:8`, `app/(tabs)/index.tsx:9`).
**Risiko:** Werden diese Dateien in Shopify gelöscht, umbenannt oder neu
hochgeladen, zeigt die veröffentlichte App leere Flächen — ein App-Update wäre
nötig. Vor Veröffentlichung entscheiden, ob die beiden Bilder ins Bundle wandern
sollen. Die URLs konnten hier nicht geprüft werden, weil der Egress-Proxy dieser
Umgebung `cdn.shopify.com` sperrt.

---

## 3. Vorgenommene Änderungen

Der Nutzer hat den Paketnamen auf die Marke „M2Marine" festgelegt. Da Google die
Umkehr-Domain-Schreibweise verlangt, wurde daraus `com.m2marine.letters`.

**Wichtiger Fund:** Der Paketname war **an zwei Stellen** hartkodiert. Eine Änderung
nur in `app.config.ts` hätte das Deep-Link-Schema der App von dem in
`constants/oauth.ts` erwarteten Schema getrennt. Beide Dateien wurden geändert:

| Datei | vorher | nachher |
|---|---|---|
| `app.config.ts` | `rawBundleId = "com.app.m2marinelettersapp"` | `rawBundleId = "com.m2marine.letters"` |
| `app.config.ts` | Schema aus Bundle-ID abgeleitet → `manusm2marinelettersapp` | `appScheme = "m2marine"` |
| `constants/oauth.ts` | eigene Kopie derselben Ableitung | `schemeFromBundleId = "m2marine"` |

Ergebnis von `npx expo config --type public`:

```
Name:         M2 Marine
Version:      1.0.15
Scheme:       m2marine
Android pkg:  com.m2marine.letters
versionCode:  16
iOS bundle:   com.m2marine.letters
```

Nach der Änderung wurden TypeScript, Testsuite und Android-Preflight erneut
ausgeführt — weiterhin alle grün.

> **Nebenwirkung, die dem Nutzer bekannt ist:** Durch den neuen Paketnamen ist der
> nächste Build für Android eine andere App. Er überschreibt die vorhandene
> Manus-APK auf dem Handy nicht, sondern erscheint als zweites Icon.

### Noch offen an der Konfiguration

`app.config.ts` enthält weder `owner` noch `extra.eas.projectId`. Beides ist nötig,
falls über EAS gebaut wird, und hängt vom Expo-Konto des Nutzers ab:

```ts
const config: ExpoConfig = {
  name: env.appName,
  slug: env.appSlug,
  owner: "<expo-benutzername>",
  extra: { eas: { projectId: "<id aus `eas init`>" } },
  // ...
```

Kleinigkeit ohne Blockerwirkung: Der Intent-Filter in `app.config.ts` setzt
`autoVerify: true` auf ein Custom-Scheme. App-Links-Verifizierung greift nur bei
`http`/`https`; das ist funktional harmlos, kann aber eine Notiz in der Play Console
erzeugen.

---

## 4. Vorbereitete, aber NIE AUSGEFÜHRTE Build-Automatik

Der Nutzer wollte ohne eigene Entwicklungsumgebung bauen. Deshalb liegen im Projekt:

- `.github/workflows/1-signaturschluessel-erzeugen.yml` — erzeugt einmalig einen
  4096-Bit-RSA-Keystore (27 Jahre gültig), legt ihn samt Passwörtern als
  Workflow-Artefakt zum Download ab und erklärt im Job-Summary, welche vier
  Repository-Secrets anzulegen sind.
- `.github/workflows/2-app-bauen.yml` — baut auf Knopfdruck `aab` (Play Store) oder
  `apk` (Handy-Test). Setzt Version und versionCode aus den Eingabefeldern, läuft
  durch `pnpm run check` und `pnpm test`, erzeugt via `expo prebuild` das
  Android-Projekt, signiert mit dem Upload-Schlüssel und **bricht ab, falls das
  Ergebnis mit dem Debug-Schlüssel signiert wäre**.
- `scripts/ci/patch-android-signing.py` — verdrahtet den Release-Build-Type mit dem
  Upload-Keystore. `expo prebuild` erzeugt eine `build.gradle`, deren Release-Variante
  den Debug-Schlüssel wiederverwendet; damit signierte Dateien lehnt Google ab. Das
  Skript scheitert laut, wenn es die erwartete Stelle nicht findet, statt still
  nichts zu tun.

> ⚠️ **Diese Automatik wurde nie ausgeführt.** Weder die Workflows noch das
> Signing-Skript sind gegen einen echten Gradle-Build getestet. Nur die YAML-Syntax
> und die Python-Syntax wurden geprüft. Ein Repository konnte nicht angelegt werden
> (das GitHub-Token dieser Sitzung darf das nicht: HTTP 403). Erster Lauf daher mit
> `paket_typ = apk` starten, nicht direkt mit `aab`.

### Warum hier nicht direkt gebaut wurde

Java 21 und Gradle waren vorhanden, aber der Egress-Proxy dieser Umgebung sperrt
`dl.google.com` (HTTP 403, Organisations-Policy). Ohne diesen Host gibt es weder
Android SDK noch Googles Maven-Repository. Ein Build ist hier technisch unmöglich —
er muss auf dem Rechner des Nutzers, auf EAS Build oder auf GitHub-Runnern laufen.

---

## 5. Offene Blocker vor der Veröffentlichung

Aus `notes/google-play-readiness.md` des Projekts, von mir nachgeprüft und weiterhin gültig:

| Prio | Punkt | Was konkret zu tun ist |
|---|---|---|
| **P0** | Shopify-Richtlinien enthalten Platzhalter: `[INSERT RETURN ADDRESS]` in der Refund-Policy, `My Store` und `[LINK TO … POLICY]` in den Terms | In Shopify Admin → Einstellungen → Richtlinien bereinigen. Google-Prüfer öffnen diese Links; Platzhalter sind ein klassischer Ablehnungsgrund. |
| **P0** | Versandangaben widersprechen sich: App verspricht weltweiten Gratisversand und USA-DDP inklusive, die Shopify-Versandseite nennt Aufpreise und nicht enthaltene Abgaben | Einen verbindlichen Text festlegen und App, Website und Shopify angleichen. |
| **P0** | Kein Produktions-AAB existiert | Abschnitt 4 ausführen. |
| **P1** | Realgerätetest im Release-Build ausstehend | Foto wählen, Kamera, Mockup-Download in die Galerie, Share-Sheet, Shopify-Checkout inklusive Rückkehr in die App. |
| **P1** | Play-Console-Angaben | Data Safety, Content Rating, Zielgruppe, Store-Assets. Die konkreten Antworten stehen in Abschnitt 2. |
| **P1** | Kontotyp | Persönliche Entwicklerkonten, die nach dem 13.11.2023 erstellt wurden, brauchen vor der Produktionsfreigabe einen 14-tägigen Closed Test mit mindestens 12 dauerhaft angemeldeten Testern. **Das ist der zeitbestimmende Faktor** und muss als Erstes geklärt werden. |

Unkritisch: Der Verkauf **physischer** Buchstaben über einen externen Shopify-Checkout
ist zulässig. Google Play Billing wäre nur für digitale Güter verpflichtend.

---

## 6. Empfohlene nächste Schritte

1. Kontotyp in der Play Console prüfen (persönlich vs. Organisation). Bei „persönlich"
   sofort 12 Tester mit Google-Konten sammeln — das bestimmt den Zeitplan.
2. Entscheiden, ob Produktfoto und Hero-Bild ins App-Bundle wandern sollen,
   statt von der Shopify-CDN geladen zu werden.
3. Repository anlegen, Projekt hochladen, Workflow 1 einmalig laufen lassen,
   Keystore sichern, vier Secrets anlegen.
4. Workflow 2 mit `paket_typ = apk` starten und die Automatik erstmals verifizieren.
   Ergebnis auf einem echten Android-Gerät durchtesten (Liste in Abschnitt 5, P1).
5. Erst danach `paket_typ = aab` bauen und in den internen Test-Track hochladen.
6. Parallel die beiden P0-Punkte in Shopify abarbeiten.

---

## 7. Was in diesem Paket liegt

Der komplette Quellcode inklusive aller Assets, plus:

```
UEBERGABE-AN-M2-CLAUDE.md                         diese Datei
.github/workflows/1-signaturschluessel-erzeugen.yml
.github/workflows/2-app-bauen.yml
scripts/ci/patch-android-signing.py
```

Nicht enthalten: `node_modules/`, `dist-web/`, `.expo/` — alles reproduzierbar über
`pnpm install`.

*Hinweis: technische Übergabe, keine Rechtsberatung. Shop-Richtlinien, Versand- und
Steuerangaben vor dem öffentlichen Verkauf rechtlich prüfen lassen.*
