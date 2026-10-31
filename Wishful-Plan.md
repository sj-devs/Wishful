# Wishful – Projektplan

> Native Mobile App in Swift (iOS zuerst, Android später) für Wunsch-Posts, bei denen andere den Wunsch erfüllen können.
> Team: Stefan Matteo Kauflin + Joel Felix · Stand: 31.10.2026

---

## 1. Anforderungen (zusammengefasst)

- **Account & Profil:** Registrierung/Login, Lieferadresse wird **verschlüsselt** gespeichert und ist nie öffentlich sichtbar. Öffentlich sind nur Ort/Kanton und Land.
- **Wunsch-Posts:** Man gibt einen Titel (z.B. «PS5-Controller») und einen Produktlink ein. Das System erkennt Preis, Bild und Shopnamen automatisch.
- **Feeds:** «For You» (alle bzw. empfohlene Posts) und «Friends» (nur markierte Freunde/Familie), umschaltbar wie bei TikTok.
- **Beziehungen:** Man kann Profile als Freund oder Familie markieren (mit gegenseitiger Bestätigung).
- **Schenken:** Andere Nutzer kaufen einen Wunsch, **ohne die Adresse des Empfängers je zu sehen**. Wishful bestellt das Produkt automatisch und lässt es direkt an den Empfänger liefern.
- **Geschäftsmodell:** Provision, die der Schenker zusätzlich bezahlt (siehe Abschnitt 2).
- **Distribution:** Zuerst Apple App Store (native Swift-App), danach Google Play Store.

---

## 2. Kernentscheidung: Wie funktioniert das Schenken?

### Übersicht der Optionen

| Option | Ablauf | Vorteile | Nachteile |
|---|---|---|---|
| **A – Geldgeschenk** | Schenker zahlt → Geld geht zweckgebunden an den Empfänger → Empfänger bestellt selbst | Technisch am einfachsten | Betrugsrisiko (Fake-Wünsche), eigentlich «Geld schenken», Geldwäscherei-Thematik (GwG) |
| **B – Wishful bestellt manuell** | Schenker zahlt an Wishful → jemand vom Team bestellt von Hand | Echte Ware, kein Technik-Risiko | Skaliert nicht, viel Handarbeit |
| **C – Adresse beim Kauf freigeben** | Empfänger gibt die Adresse pro Kauf frei | Simpel | Bricht das Datenschutz-Versprechen → **nicht empfohlen** |
| **D – Automatische Bestellung (Dropshipping-Prinzip)** ⭐ | Schenker zahlt in der App → **das System öffnet den Produktlink, bestellt und liefert an die hinterlegte Adresse des Empfängers** | Entspricht der Idee genau, Adresse bleibt komplett verborgen, Provision pro Bestellung, kein Geldtransfer an Private (kein GwG-Thema) | Technisch anspruchsvoll, ihr seid rechtlich Käufer bzw. Wiederverkäufer (Retouren, Garantie, MWST) |

**Entscheidung: Option D.** Sie wird **stufenweise** umgesetzt, weil eine vollautomatische Bestellung in *jedem* beliebigen Shop am Anfang nicht realistisch ist.

### Option D im Detail

#### Ablauf
1. Der Schenker tippt auf «Wunsch erfüllen».
2. Das System **prüft den Preis live** über den Link (er kann sich seit dem Posten geändert haben) und berechnet die Provision.
3. Der Schenker sieht den Gesamtbetrag (Artikel + Versand + Provision) und bezahlt.
4. Erst **nach erfolgreichem Zahlungseingang** löst das System die Bestellung im Shop aus. Lieferadresse ist die verschlüsselte Adresse des Empfängers, entschlüsselt nur in diesem Moment und serverseitig.
5. Der Empfänger bekommt eine Push-Nachricht und eine E-Mail, der Schenker eine Bestellbestätigung.
6. Wenn die Bestellung fehlschlägt (ausverkauft, Preis stark gestiegen, Shop liefert nicht in die Region), erhält der Schenker **automatisch eine volle Rückerstattung**.

#### Provisionsmodell

| Artikelpreis | Provision (zahlt der Schenker) |
|---|---|
| unter CHF 100.– | **CHF 5.– pauschal** |
| ab CHF 100.– | **5 % des Artikelpreises** |

Beispiele:

| Artikel | Provision | Total für den Schenker (ohne Versand) |
|---|---|---|
| CHF 25.– | CHF 5.– | CHF 30.– |
| CHF 99.– | CHF 5.– | CHF 104.– |
| CHF 100.– | CHF 5.– | CHF 105.– |
| CHF 300.– | CHF 15.– | CHF 315.– |

> Bei genau CHF 100.– ergeben beide Regeln CHF 5.–, es gibt also keinen Preissprung an der Grenze. ✅
>
> **Achtung, Gebühren des Zahlungsanbieters:** Die Gebühren (je nach Anbieter ca. 1.3–3 % + Fixbetrag) gehen von eurer Provision ab. Bei einem Artikel für CHF 25.– bleiben euch von den CHF 5.– ca. CHF 4.–. Rechnet das je nach gewähltem Zahlungsanbieter durch und klärt, ob Versandkosten provisionspflichtig sind (Empfehlung: Provision nur auf den Artikelpreis).

#### Wie bestellt «das System» technisch? (Stufen)

| Stufe | Methode | Wann |
|---|---|---|
| **1 – Concierge-MVP** | Das System bereitet alles vor (Link, Menge, Variante, Adresse, Betrag) und erstellt einen Auftrag im Admin-Dashboard. **Ihr klickt im Shop selbst auf «Kaufen»**, Tracking-Nummer wird nachgetragen. | Launch, erste Nutzer. So lernt ihr die Abläufe kennen. |
| **2 – Shop-APIs & Partnerprogramme** | Direkte Bestellung über offizielle Schnittstellen (z.B. Shopify-basierte Shops, Partner-/Affiliate-APIs von grossen Händlern, Universal-Checkout-Dienste wie Rye; Verfügbarkeit in der Schweiz prüfen) | Sobald es genug Bestellungen gibt |
| **3 – Browser-Automatisierung** | Headless-Browser (z.B. Playwright) bestellt für ausgewählte Top-Shops | Nur gezielt. Fehleranfällig, oft gegen die AGB der Shops, Captchas und Bot-Schutz |

#### Was Option D zusätzlich braucht
- **Firmen-Zahlungsmittel für die Shop-Bestellungen** (z.B. Firmenkreditkarte oder virtuelle Karte pro Bestellung). Das Geld des Schenkers ist dabei immer schon eingegangen, bevor ihr bestellt, ihr müsst also nichts vorfinanzieren.
- **Kundenkonten bei den Shops** (Firmenkonto pro Shop) bzw. Gast-Checkout
- **Retouren & Garantie:** Prozess definieren (Empfänger meldet sich bei Wishful, ihr wickelt mit dem Shop ab)
- **MWST:** Der volle Artikelpreis kann als euer Umsatz zählen, nicht nur die Provision. Die Pflicht zur MWST-Registrierung beginnt bei **CHF 100'000 Jahresumsatz** und ist damit schnell erreicht. Mit Treuhand bzw. Fachperson klären, ob ihr als **Vermittler** (Provision = Umsatz) oder als **Wiederverkäufer** auftretet.
- **Ausland:** Bestellungen aus Shops ausserhalb der Schweiz verursachen Zoll bzw. Einfuhr-MWST. Für das MVP **nur Schweizer Shops** oder Shops, die in die Schweiz liefern.
- **Chargebacks:** Schenker ziehen Zahlungen zurück, nachdem die Ware schon bestellt wurde → Betrugserkennung, Limits für neue Accounts

---

## 3. Tech-Stack (Empfehlung)

| Bereich | Wahl | Begründung |
|---|---|---|
| App (iOS) | **Swift + SwiftUI** (nativ), Xcode, Swift Package Manager | Beste Performance und Apple-Integration (Apple Pay, Keychain, Push, Sign in with Apple). **Braucht einen Mac mit Xcode.** |
| App (Android) | **Noch offen** (siehe unten) | Swift läuft nicht direkt auf Android |
| Backend-Code | Supabase Edge Functions (TypeScript/Deno) oder Swift auf dem Server (Vapor) | Betrifft nur den Server, die App bleibt Swift |
| Backend | **Supabase** (Postgres, Auth, Storage, Edge Functions), Region **EU (Frankfurt) oder Zürich** | Row Level Security, schneller Start, Daten in Europa |
| Captcha | **Cloudflare Turnstile** oder **hCaptcha** | Von Supabase Auth direkt unterstützt, datenschutzfreundlicher als Google reCAPTCHA |
| Link-Scraper | Separater, isolierter Service (Cloud Run / Fly.io) | Wegen SSRF-Risiko vom Backend getrennt (siehe Abschnitt 6) |
| Bestell-Service | Separater Service mit Auftrags-Queue (Stufe 1: Admin-Dashboard) | Bestellungen zuverlässig, wiederholbar und nachvollziehbar |
| Zahlungen | **Noch offen**. Kandidaten: Stripe, Datatrans, Payrexx, Wallee, Adyen | Muss TWINT, Karten, Apple Pay und Google Pay können. Keine Kartendaten bei euch. |
| E-Mail | Transaktions-E-Mail-Dienst (z.B. Resend, Postmark, Brevo), EU-Region bevorzugt | Automatische Benachrichtigungen und Bestellbestätigungen |
| Push | Apple Push Notification Service (APNs), für Android später Firebase Cloud Messaging | – |
| Moderation | Meldefunktion, Blockieren, Admin-Dashboard | Pflicht bei nutzergenerierten Inhalten (Apple Guideline 1.2) |

### Swift und Android

Swift-Apps laufen nur auf Apple-Geräten. Für Android gibt es drei Wege:

| Weg | Beschreibung | Aufwand |
|---|---|---|
| **1 – iOS zuerst** ⭐ | Zuerst nur die iOS-App in Swift bauen und launchen, Android erst nach erfolgreicher Beta | Am wenigsten, Fokus auf ein Produkt |
| **2 – Skip** | Framework, das Swift/SwiftUI-Code in eine native Android-App (Kotlin/Jetpack Compose) übersetzt. Eine Codebasis in Swift. | Mittel. Noch jung, nicht alle Bibliotheken werden unterstützt, aktuelle Lizenz und Kosten prüfen. |
| **3 – Separate Android-App** | Zweite App in Kotlin + Jetpack Compose, gleiches Backend | Hoch, alles doppelt bauen und pflegen |

**Empfehlung:** Weg 1. Die gesamte Logik (Preise, Provision, Bestellungen, Berechtigungen) liegt im Backend, nicht in der App. Damit bleibt eine spätere Android-App (Weg 2 oder 3) gut machbar.

**Voraussetzung:** iOS-Entwicklung mit Swift geht nur auf **macOS mit Xcode**. Wer am Code der App arbeitet, braucht einen Mac. Ohne Mac geht es nur über einen gemieteten Mac in der Cloud (z.B. MacinCloud) oder Cloud-Builds (Xcode Cloud, GitHub Actions mit macOS-Runnern). Für die tägliche Entwicklung ist das umständlich.

> Hinweis: Physische Waren dürfen ohne Apples In-App-Kauf (und ohne die 30 % Gebühr) verkauft werden (Guideline 3.1.5). Mit Option D wird tatsächlich ein physisches Produkt verkauft, das passt also gut.

---

## 4. Phasen

> Bis Phase 6 testet ihr die App im iOS-Simulator und auf euren eigenen iPhones. Dafür reicht eine **kostenlose Apple-ID** in Xcode (die App muss dann alle 7 Tage neu installiert werden). Ein bezahlter Developer Account wird erst für TestFlight und den App Store nötig.

### Phase 0 – Entscheidungen & Setup (1–2 Wochen) · Komplexität: niedrig
- [ ] Mindestalter festlegen (Empfehlung: **18+**)
- [ ] Klären, wer einen Mac mit Xcode hat (Voraussetzung für die Swift-Entwicklung)
- [ ] GitHub-Repo, CI (GitHub Actions mit macOS-Runner), Xcode-Projekt (SwiftUI, iOS 17+), Supabase-Projekt in der EU-Region
- [ ] Code-Stil festlegen: SwiftLint, SwiftFormat, Architektur (z.B. MVVM mit `@Observable`), Tests mit Swift Testing / XCTest
- [ ] Wireframes in Figma

### Phase 1 – Auth, Profil, Adresse (2–3 Wochen) · Komplexität: mittel
- [ ] Login mit E-Mail, Sign in with Apple (Pflicht, sobald es Google-Login gibt) und Google, **mit Captcha bei Registrierung, Login und Passwort-Reset** (Turnstile/hCaptcha, serverseitig in Supabase Auth geprüft)
- [ ] Profil: Name, Bild, Bio, öffentlicher Standort (Kanton/Ort und Land, frei wählbar)
- [ ] Adresse in separater, verschlüsselter Tabelle (siehe Abschnitt 6)
- [ ] Account löschen und Daten exportieren (verlangt von nDSG und Apple)

### Phase 2 – Posts & Link-Erkennung (3 Wochen) · Komplexität: hoch
- [ ] Post erstellen (Titel, Link, Variante wie Grösse oder Farbe, optionale Notiz) mit Vorschau
- [ ] Scraper versucht der Reihe nach: schema.org/JSON-LD `Product` → Open-Graph-Tags → shopspezifische Parser für die Top-Shops (Galaxus, Digitec, Zalando …)
- [ ] Preis und Bild werden mit Zeitstempel gecacht, Preise periodisch aktualisiert
- [ ] Fallback: Preis und Bild manuell eingeben, falls die Erkennung scheitert
- [ ] Prüfen, ob der Shop **unterstützt wird** (liefert in die Schweiz, bestellbar). Nicht unterstützte Shops werden im Post gekennzeichnet.
- [ ] Post-Status: offen → bezahlt → bestellt → versandt → geliefert

### Phase 3 – Feed & Freunde (3 Wochen) · Komplexität: mittel
- [ ] Freundschafts- und Familienanfragen mit gegenseitiger Bestätigung
- [ ] Friends-Feed chronologisch
- [ ] «For You»-Feed im MVP mit einfachem Ranking (Aktualität, Region, Preisklasse). Echtes Profiling erst später und mit Transparenz (siehe Abschnitt 7)
- [ ] Sichtbarkeit pro Post: öffentlich oder nur Freunde

### Phase 4 – Schenken, Zahlung & Bestellung (4–6 Wochen) · Komplexität: hoch
- [ ] **Zahlungsanbieter evaluieren und entscheiden** (noch offen, ob Stripe oder eine Alternative). Kriterien: TWINT, Gebühren, Betrugsschutz, Schweizer Firma ja/nein, Auszahlung, Aufwand für die Integration
- [ ] Checkout: Preis live neu prüfen, Provision berechnen (unter CHF 100.– → CHF 5.–, ab CHF 100.– → 5 %), Gesamtbetrag transparent anzeigen
- [ ] Benachrichtigung «Du wurdest beschenkt!» per Push **und automatische E-Mail an die hinterlegte E-Mail-Adresse des Empfängers**, zusätzlich Bestell- und Zahlungsbestätigung per E-Mail an den Schenker. Optional anonym schenken (dann taucht der Name des Schenkers in keiner E-Mail auf).
- [ ] Bestell-Service Stufe 1 (Concierge): nach Zahlungseingang Auftrag im Admin-Dashboard, Tracking-Nummer erfassen, Statusmeldungen per Push und E-Mail
- [ ] Automatische Rückerstattung, wenn die Bestellung fehlschlägt
- [ ] Retouren- und Garantie-Prozess, Umgang mit Chargebacks

### Phase 5 – Sicherheit, Moderation, Recht (2–3 Wochen, teilweise parallel) · Komplexität: mittel
- [ ] Melden, Blockieren, Admin-Tool, Wortfilter
- [ ] **GmbH gründen** (Haftung, Firmenkonto, Firmenkarte für Bestellungen)
- [ ] Datenschutzerklärung, AGB (inkl. Provision, Rückerstattung, Retouren), Impressum, Datenschutz-Folgenabschätzung (DSFA)
- [ ] MWST-Frage mit Treuhand bzw. Fachperson klären (Vermittler oder Wiederverkäufer)
- [ ] Externer Security-Review bzw. Pentest vor dem Launch

### Phase 6 – Store-Account, Beta & iOS-Launch (3–4 Wochen)
- [ ] **Apple Developer Account (99 USD/Jahr)**, idealerweise auf die **GmbH**. Für einen Firmen-Account bei Apple braucht es eine D-U-N-S-Nummer (kostenlos, Bearbeitung kann einige Tage dauern).
- [ ] Signing, Provisioning und Push-Zertifikate (APNs) einrichten
- [ ] TestFlight-Beta (intern und extern)
- [ ] App-Store-Listing, Altersfreigabe, App Review

### Phase 7 – Android (nach erfolgreichem iOS-Launch)
- [ ] Entscheid: Skip (Swift → Android) oder separate Kotlin-App
- [ ] **Google Play Account (25 USD einmalig)**. **Achtung:** Neue private Play-Accounts brauchen **mind. 12 Tester während 14 Tagen** im Closed Testing, bevor sie veröffentlichen dürfen (gilt nicht für Firmen-Accounts).
- [ ] Android-App bauen, Push via Firebase Cloud Messaging, Google Pay
- [ ] Play-Store-Listing und Review

### Später – Ausbau
- [ ] Bestell-Service Stufe 2 (Shop-APIs, Partnerprogramme)
- [ ] Bestell-Service Stufe 3 (Browser-Automatisierung für Top-Shops)
- [ ] Ausländische Shops (Zoll, Einfuhr-MWST)

**Geschätzte Gesamtdauer:** ca. 5–7 Monate neben Schule oder Job.

---

## 5. Arbeitsaufteilung

*Vorschlag, an eure Stärken anpassen.*

| Bereich | **Stefan** (App & Produkt) | **Joel** (Backend & Sicherheit) |
|---|---|---|
| Phase 0 | Wireframes, Design-System, Xcode-Projekt, SwiftLint | Supabase, CI, Repo-Regeln, Secrets-Management |
| Phase 1 | Login-, Profil- und Onboarding-Screens, Captcha-Widget | Auth-Konfiguration, Captcha-Prüfung, DB-Schema, RLS, Adress-Verschlüsselung |
| Phase 2 | Post erstellen, Link-Vorschau, Post-Detailansicht | Scraper-Service, Shop-Parser, Preis-Refresh-Job, Shop-Unterstützung prüfen |
| Phase 3 | Feed-UI mit Tab-Switch, Freundesliste | Feed-Queries, Ranking, Freundschafts-Logik |
| Phase 4 | Checkout-Flow, Provisionsanzeige, Statusanzeigen, E-Mail-Templates | Zahlungsanbieter-Integration, Webhooks, Bestell-Service, Admin-Dashboard, E-Mail-Versand |
| Phase 5 | Melde- und Blockier-UI, AGB-Texte | Admin-Tool, Rate Limiting, Logging, Pentest-Fixes |
| Phase 6 | App-Store-Listing, Screenshots, TestFlight-Tester organisieren | Apple-Account, Signing, APNs, CI-Builds |
| Phase 7 | Android-UI (je nach Entscheid Skip oder Kotlin) | Firebase Cloud Messaging, Play-Account, Backend-Anpassungen |
| Recht | Datenschutzerklärung und AGB formulieren | Bearbeitungsverzeichnis, DSFA, Auftragsverarbeitungsverträge (AVV) |

**Gemeinsam:**
- **Evaluation des Zahlungsanbieters** (Phase 4, Schritt 1)
- Concierge-Bestellungen im Betrieb abwickeln (Stufe 1)
- GmbH-Gründung und MWST-Abklärung
- Jeder Pull Request wird von der anderen Person reviewt (Branch-Schutz auf `main`)
- Wöchentliches Planning (z.B. GitHub Projects)
- Eine Person ist offizielle **Datenschutz-Ansprechperson** (Vorschlag: Joel)

---

## 6. Sicherheitsmassnahmen

### Adressen (höchster Schutzbedarf)
- Separate Tabelle `addresses` ohne direkten Lesezugriff aus der App. Auch der Besitzer sieht die Adresse nach dem Speichern nur maskiert.
- **Verschlüsselung auf Feldebene** (pgsodium/Supabase Vault oder KMS), damit ein DB-Dump allein nicht reicht
- Entschlüsselt wird **nur durch den Bestell-Service im Moment der Bestellung**, und **jeder Zugriff wird im Audit-Log festgehalten**. Im Concierge-Betrieb (Stufe 1) sieht nur das Team die Adresse, nie der Schenker.
- Den öffentlichen Ort wählt der Nutzer selbst. Bei kleinen Dörfern besser nur den Kanton zeigen, weil sonst Identifikation möglich ist.

### Zahlungsdaten
- **Keine Kartendaten bei euch.** Alles läuft über die gehosteten Zahlungsformulare bzw. SDKs des gewählten Zahlungsanbieters (minimaler PCI-Aufwand, SAQ-A).
- Webhooks des Zahlungsanbieters per Signatur prüfen, Zahlungen idempotent verarbeiten, damit nichts doppelt gebucht wird
- **Bestellung erst nach bestätigtem Zahlungseingang** (Webhook), nie nur aufgrund einer Meldung aus der App
- Firmenkarte für Shop-Bestellungen: Limits setzen, wenn möglich virtuelle Karten pro Bestellung, Zugangsdaten nur im Secret Store

### Bestell-Service
- Shop-Logins und Kartendaten nur in einem Secret Store (z.B. Supabase Vault), nie im Code
- Jede Bestellung mit eindeutiger ID, Zustandsmaschine und Audit-Log
- Plausibilitätsprüfung vor jeder Bestellung: Ist der Preis gleich wie beim Checkout? Ist die Adresse in der Schweiz?

### Login & Konten
- **Captcha** bei Registrierung, Login und Passwort-Reset (Schutz vor Bots, Fake-Accounts und Credential Stuffing)
- Optionale 2FA oder Passkeys, Sessions und Tokens sicher in der **iOS Keychain** speichern, App Transport Security aktiv lassen (nur HTTPS)
- Rate Limiting (Login, Post-Erstellung, Scraper, Checkout)

### Link-Scraper (oft unterschätzt)
- **SSRF-Schutz:** interne IPs sperren (127.x, 10.x, 172.16–31.x, 192.168.x, 169.254.x, IPv6-Pendants), und zwar **nach** der DNS-Auflösung prüfen
- Nur `https`, Timeout, maximale Antwortgrösse, Redirect-Limit, kein JavaScript ausführen
- Läuft isoliert und ohne Zugriff auf DB-Secrets
- Bilder serverseitig neu kodieren (entfernt EXIF-Daten, verhindert Malware)
- Links gegen Phishing- und Malware-Listen prüfen (z.B. Google Safe Browsing)

### E-Mails
- SPF, DKIM und DMARC für die Absender-Domain einrichten (damit E-Mails nicht im Spam landen und nicht gefälscht werden können)
- Keine Adressen und keine sensiblen Daten in E-Mails, nur Status und Link in die App

### Allgemein
- **Row Level Security** auf allen Tabellen, mit eigenen Tests dafür
- Secrets nie ins Repo, Dependabot für Swift-Packages und Backend-Abhängigkeiten, keine API-Secrets in der App (nur der öffentliche Supabase-Key, alles Weitere serverseitig)
- Backups und ein Notfallplan für Datenlecks (wer macht was?)
- Gegen Missbrauch: Mindestalter, Meldefunktion, **keine Direktnachrichten** unter Fremden (vorerst), Limits für neue Accounts

---

## 7. Schweizer Datenschutzgesetz (nDSG, in Kraft seit 1.9.2023)

| Pflicht | Artikel | Umsetzung in Wishful |
|---|---|---|
| Grundsätze: verhältnismässig, zweckgebunden, transparent | Art. 6 | Nur nötige Daten erheben, Adresse ausschliesslich für die Lieferung verwenden |
| **Privacy by Design und by Default** | Art. 7 | Datenschutzfreundliche Standardeinstellungen, grober Standort, verschlüsselte Adresse |
| Datensicherheit | Art. 8 | Massnahmen aus Abschnitt 6 |
| Auftragsbearbeiter | Art. 9 | Auftragsverarbeitungsverträge (AVV/DPA) mit Supabase, Zahlungsanbieter, E-Mail-Dienst, Captcha-Anbieter, Hosting, Push-Dienst |
| Verzeichnis der Bearbeitungstätigkeiten | Art. 12 | Für KMU mit < 250 MA nur bei hohem Risiko Pflicht, **trotzdem führen** |
| Bekanntgabe ins Ausland | Art. 16 | EU-Hosting ist unproblematisch, bei US-Diensten auf Zertifizierung unter dem Swiss-US Data Privacy Framework achten |
| **Informationspflicht / Datenschutzerklärung** | Art. 19 | In der App und im Store verlinkt: welche Daten, Zweck, Empfänger (**auch: Adresse geht an den jeweiligen Shop zur Lieferung**), Länder, Rechte |
| Datenschutz-Folgenabschätzung (DSFA) | Art. 22 | **Empfohlen** wegen Adressen, Zahlungen und Feed-Profiling |
| Meldung von Datenlecks an den EDÖB | Art. 24 | «So rasch als möglich» bei hohem Risiko, Ablauf vorab festlegen |
| Auskunftsrecht | Art. 25 | Datenexport in der App, Antwort innert 30 Tagen |
| Datenherausgabe / Portabilität | Art. 28 | Export als JSON |
| Löschung | Art. 32 | Account-Löschung mit echter Löschung (Backups mit Frist; Bestell- und Buchhaltungsdaten müssen **10 Jahre aufbewahrt** werden, OR Art. 958f) |

> ⚠️ **Wichtig:** Das nDSG büsst **natürliche Personen** (also euch persönlich, nicht die Firma) mit bis zu **CHF 250'000**. Deshalb die Pflichten ernst nehmen und eine **GmbH gründen**.

### Weitere rechtliche Punkte
- **DSGVO:** gilt zusätzlich, sobald ihr Nutzer in der EU ansprecht. Dann braucht es eventuell eine EU-Vertretung (Art. 27 DSGVO).
- **Impressum und AGB:** Pflicht für Online-Angebote (UWG Art. 3 Abs. 1 lit. s). Die Provision muss **vor der Zahlung klar ausgewiesen** sein (Preisbekanntgabeverordnung).
- **AGB der Shops:** Viele Shops verbieten automatisierte Bestellungen (Bots) oder Wiederverkauf. Vor Stufe 3 pro Shop prüfen, Partnerschaften bevorzugen.
- **MWST:** Registrierungspflicht ab CHF 100'000 Umsatz. Ob der Artikelpreis oder nur die Provision zählt, hängt vom Modell ab (Vermittler oder Wiederverkäufer), deshalb mit Treuhand klären.
- **Geldwäschereigesetz (GwG):** Mit Option D bezahlt der Schenker ein Produkt und schickt kein Geld an Private. Das reduziert das Thema stark. Trotzdem mit dem Zahlungsanbieter abklären.
- **Affiliate-Links:** falls genutzt, als Werbung kennzeichnen
- Für DSFA, AGB und MWST eine kurze **Rechtsberatung** einplanen (Startup-Angebote, z.B. über Hochschulen oder Startup-Programme)

---

## 8. Risiken

| Stufe | Risiko | Gegenmassnahme |
|---|---|---|
| **HOCH** | **Minderjährige erhalten Geschenke von Fremden** (Grooming-Gefahr) | 18+ im MVP, keine Direktnachrichten, Meldefunktion, Moderation |
| **HOCH** | **Automatische Bestellung scheitert** (Bot-Schutz, Shop-Änderungen, ausverkauft) | Stufenweise Umsetzung (Concierge zuerst), automatische Rückerstattung, Shop-Positivliste |
| **HOCH** | Betrug (gestohlene Karten, Chargebacks nach der Bestellung) | Betrugserkennung des Zahlungsanbieters, 3-D Secure, Limits für neue Accounts |
| **HOCH** | Preis ändert sich zwischen Post und Kauf | Live-Preisprüfung im Checkout, Bestellung abbrechen und erstatten bei Abweichung |
| **MITTEL** | Provision deckt die Kosten nicht (Gebühren, Retouren) | Kalkulation pro Zahlungsanbieter, Provision nur auf den Artikelpreis, Modell nach der Beta überprüfen |
| **MITTEL** | MWST-Pflicht früher als erwartet | Früh mit Treuhand klären |
| **MITTEL** | Ablehnung im App-Store-Review (Moderation, Zahlungsablauf) | Guidelines 1.2, 3.1.5 und 5.1.1 vor dem Bau prüfen |
| **MITTEL** | Datenleck bei den Adressen | Verschlüsselung, Audit-Log, Pentest |
| **NIEDRIG** | Urheberrecht an Produktbildern | Nur als Vorschau mit Quellenlink, nicht dauerhaft weiterverwenden |

---

## 9. Offene Fragen

1. **Welcher Zahlungsanbieter?** Stripe, Datatrans, Payrexx, Wallee oder Adyen (Vergleich in Phase 4)
2. Mindestalter 18+, einverstanden?
3. Wer kann was? (Erfahrung mit Swift/SwiftUI oder Backend?) → Aufteilung anpassen
4. **Wer hat einen Mac mit Xcode?** Ohne Mac ist Swift-Entwicklung kaum möglich.
5. **Android:** iOS zuerst (Empfehlung), Skip oder separate Kotlin-App?
6. Start nur mit Schweizer Shops, einverstanden?
7. Wer trägt die Versandkosten? (Vorschlag: Schenker, ohne Provision darauf)
8. Was passiert bei Retouren bzw. wenn der Empfänger das Geschenk nicht will?
9. Treten wir rechtlich als **Vermittler** oder als **Wiederverkäufer** auf? (Auswirkung auf MWST und Haftung)
