# Wishful – Projektplan

> Mobile App (iOS + Android) für Wunsch-Posts, bei denen andere den Wunsch erfüllen können.
> Team: Du + Joel · Stand: 31.10.2026

---

## 1. Anforderungen (zusammengefasst)

- **Account & Profil:** Registrierung/Login, Lieferadresse wird **verschlüsselt** gespeichert und ist nie öffentlich sichtbar. Öffentlich sind nur Ort/Kanton und Land.
- **Wunsch-Posts:** Man gibt einen Titel (z.B. «PS5-Controller») und einen Produktlink ein. Das System erkennt Preis, Bild und Shopnamen automatisch.
- **Feeds:** «For You» (alle bzw. empfohlene Posts) und «Friends» (nur markierte Freunde/Familie), umschaltbar wie bei TikTok.
- **Beziehungen:** Man kann Profile als Freund oder Familie markieren (mit gegenseitiger Bestätigung).
- **Schenken:** Andere Nutzer kaufen einen Wunsch, **ohne die Adresse des Empfängers je zu sehen**.
- **Distribution:** Apple App Store + Google Play Store.

---

## 2. Kernentscheidung: Wie funktioniert das Schenken?

Davon hängen Sicherheit, Recht und Aufwand am stärksten ab.

| Option | Ablauf | Vorteile | Nachteile |
|---|---|---|---|
| **A – Geldgeschenk via Stripe Connect** | Schenker zahlt in der App → Geld geht zweckgebunden an den Empfänger → Empfänger bestellt selbst | Einfachstes MVP, keine Adressweitergabe, Stripe übernimmt KYC/GwG | Betrugsrisiko (Fake-Wünsche), ist eigentlich «Geld schenken» |
| **B – Wishful bestellt selbst** | Schenker zahlt an Wishful → Wishful bestellt beim Shop → Lieferung an die hinterlegte Adresse (nur das System kennt sie) | Entspricht der Idee exakt, echte Ware | Ihr seid Händler/Vermittler: Retouren, MWST, Haftung, Shop-Automatisierung, viel Betriebsaufwand |
| **C – Adresse beim Kauf freigeben** | Empfänger bestätigt pro Kauf die Adressweitergabe | Technisch simpel | Bricht das Datenschutz-Versprechen, Stalking-Risiko → **nicht empfohlen** |

**Empfehlung:** **Option A** fürs MVP, **Option B** später als Ausbaustufe mit Partnershops (z.B. Galaxus/Digitec über eine Affiliate- oder Partner-API). Die Adresse wird trotzdem von Anfang an erfasst, damit B später möglich ist.
Referenz zum Anschauen: die App «Throne» (funktioniert nach Modell B).

---

## 3. Tech-Stack (Empfehlung)

| Bereich | Wahl | Begründung |
|---|---|---|
| App | **React Native + Expo** (TypeScript) | Eine Codebasis für iOS und Android, Builds/Updates via EAS |
| Backend | **Supabase** (Postgres, Auth, Storage, Edge Functions), Region **EU (Frankfurt) oder Zürich** | Row Level Security, schneller Start, Daten in Europa |
| Link-Scraper | Separater, isolierter Service (Cloud Run / Fly.io) | Wegen SSRF-Risiko vom Backend getrennt (siehe Abschnitt 6) |
| Zahlungen | **Stripe Connect** (TWINT, Karten, Apple Pay, Google Pay) | Keine Kartendaten bei euch |
| Push | Expo Notifications | – |
| Moderation | Meldefunktion, Blockieren, Admin-Dashboard | Pflicht bei nutzergenerierten Inhalten (Apple Guideline 1.2) |

> Hinweis: Physische Waren dürfen ohne Apples In-App-Kauf (und ohne die 30 % Gebühr) verkauft werden (Guideline 3.1.5). Bei Option A müsst ihr das Konzept gegenüber Apple klar als «Geschenk für ein physisches Produkt» darstellen.

---

## 4. Phasen

### Phase 0 – Entscheidungen & Setup (1–2 Wochen) · Komplexität: niedrig
- [ ] Option A oder B festlegen, Mindestalter festlegen (Empfehlung: **18+**)
- [ ] GitHub-Repo, CI, Expo-Projekt, Supabase-Projekt in der EU-Region
- [ ] Apple Developer Account (99 USD/Jahr), Google Play Account (25 USD einmalig), idealerweise auf eine **GmbH** (Haftung)
- [ ] Wireframes in Figma

### Phase 1 – Auth, Profil, Adresse (2–3 Wochen) · Komplexität: mittel
- [ ] Login mit E-Mail, **Sign in with Apple** (Pflicht, sobald es Google-Login gibt) und Google
- [ ] Profil: Name, Bild, Bio, öffentlicher Standort (Kanton/Ort und Land, frei wählbar)
- [ ] Adresse in separater, verschlüsselter Tabelle (siehe Abschnitt 6)
- [ ] Account löschen und Daten exportieren (verlangt von nDSG und Apple)

### Phase 2 – Posts & Link-Erkennung (3 Wochen) · Komplexität: hoch
- [ ] Post erstellen (Titel, Link, optionale Notiz) mit Vorschau
- [ ] Scraper versucht der Reihe nach: schema.org/JSON-LD `Product` → Open-Graph-Tags → shopspezifische Parser für die Top-Shops (Galaxus, Digitec, Amazon, Zalando …)
- [ ] Preis und Bild werden mit Zeitstempel gecacht, Preise periodisch aktualisiert
- [ ] Fallback: Preis und Bild manuell eingeben, falls die Erkennung scheitert
- [ ] Post-Status: offen → reserviert → geschenkt

### Phase 3 – Feed & Freunde (3 Wochen) · Komplexität: mittel
- [ ] Freundschafts- und Familienanfragen mit gegenseitiger Bestätigung
- [ ] Friends-Feed chronologisch
- [ ] «For You»-Feed im MVP mit einfachem Ranking (Aktualität, Region, Preisklasse). Echtes Profiling erst später und mit Transparenz (siehe Abschnitt 7)
- [ ] Sichtbarkeit pro Post: öffentlich oder nur Freunde

### Phase 4 – Schenken & Zahlung (3–4 Wochen) · Komplexität: hoch
- [ ] Stripe-Connect-Onboarding für Empfänger (KYC durch Stripe)
- [ ] Checkout für Schenker, Webhooks, Statuswechsel des Posts
- [ ] Benachrichtigung «Du wurdest beschenkt!», optional anonym schenken
- [ ] Rückerstattungen und Streitfälle

### Phase 5 – Sicherheit, Moderation, Recht (2–3 Wochen, teilweise parallel) · Komplexität: mittel
- [ ] Melden, Blockieren, Admin-Tool, Wortfilter
- [ ] Datenschutzerklärung, AGB, Impressum, Datenschutz-Folgenabschätzung (DSFA)
- [ ] Externer Security-Review bzw. Pentest vor dem Launch

### Phase 6 – Beta & Launch (3–4 Wochen)
- [ ] TestFlight und Google Play Closed Testing. **Achtung:** Neue private Play-Accounts brauchen **mind. 12 Tester während 14 Tagen**, bevor sie veröffentlichen dürfen.
- [ ] Store-Listing, Altersfreigabe, Store-Review

**Geschätzte Gesamtdauer:** ca. 4–6 Monate neben Schule oder Job.

---

## 5. Arbeitsaufteilung

*Vorschlag, an eure Stärken anpassen.*

| Bereich | **Du** (App & Produkt) | **Joel** (Backend & Sicherheit) |
|---|---|---|
| Phase 0 | Wireframes, Design-System, Expo-Setup | Supabase, CI, Repo-Regeln, Secrets-Management |
| Phase 1 | Login-, Profil- und Onboarding-Screens | Auth-Konfiguration, DB-Schema, RLS, Adress-Verschlüsselung |
| Phase 2 | Post erstellen, Link-Vorschau, Post-Detailansicht | Scraper-Service, Shop-Parser, Preis-Refresh-Job |
| Phase 3 | Feed-UI mit Tab-Switch, Freundesliste | Feed-Queries, Ranking, Freundschafts-Logik |
| Phase 4 | Checkout-Flow, Statusanzeigen | Stripe Connect, Webhooks, Zahlungslogik |
| Phase 5 | Melde- und Blockier-UI, Store-Texte | Admin-Tool, Rate Limiting, Logging, Pentest-Fixes |
| Recht | Datenschutzerklärung und AGB formulieren | Bearbeitungsverzeichnis, DSFA, Auftragsverarbeitungsverträge (AVV) |

**Gemeinsam:**
- Jeder Pull Request wird von der anderen Person reviewt (Branch-Schutz auf `main`)
- Wöchentliches Planning (z.B. GitHub Projects)
- Eine Person ist offizielle **Datenschutz-Ansprechperson** (Vorschlag: Joel)

---

## 6. Sicherheitsmassnahmen

### Adressen (höchster Schutzbedarf)
- Separate Tabelle `addresses` ohne direkten Lesezugriff aus der App. Auch der Besitzer sieht die Adresse nach dem Speichern nur maskiert.
- **Verschlüsselung auf Feldebene** (pgsodium/Supabase Vault oder KMS), damit ein DB-Dump allein nicht reicht
- Entschlüsseln kann nur eine Server-Funktion mit eigener Rolle (z.B. für die Lieferung), und **jeder Zugriff wird im Audit-Log festgehalten**
- Den öffentlichen Ort wählt der Nutzer selbst. Bei kleinen Dörfern besser nur den Kanton zeigen, weil sonst Identifikation möglich ist.

### Zahlungsdaten
- **Keine Kartendaten bei euch.** Alles läuft über Stripe Elements bzw. das Payment Sheet (minimaler PCI-Aufwand, SAQ-A).
- Stripe-Webhooks per Signatur prüfen, Zahlungen idempotent verarbeiten, damit nichts doppelt gebucht wird

### Link-Scraper (oft unterschätzt)
- **SSRF-Schutz:** interne IPs sperren (127.x, 10.x, 172.16–31.x, 192.168.x, 169.254.x, IPv6-Pendants), und zwar **nach** der DNS-Auflösung prüfen
- Nur `https`, Timeout, maximale Antwortgrösse, Redirect-Limit, kein JavaScript ausführen
- Läuft isoliert und ohne Zugriff auf DB-Secrets
- Bilder serverseitig neu kodieren (entfernt EXIF-Daten, verhindert Malware)
- Links gegen Phishing- und Malware-Listen prüfen (z.B. Google Safe Browsing)

### Allgemein
- **Row Level Security** auf allen Tabellen, mit eigenen Tests dafür
- Rate Limiting (Login, Post-Erstellung, Scraper)
- Optionale 2FA oder Passkeys, Sessions sicher speichern (Expo SecureStore)
- Secrets nie ins Repo, Dependabot und `npm audit` in der CI
- Backups und ein Notfallplan für Datenlecks (wer macht was?)
- Gegen Missbrauch: Mindestalter, Meldefunktion, **keine Direktnachrichten** unter Fremden (vorerst), Limits für neue Accounts

---

## 7. Schweizer Datenschutzgesetz (nDSG, in Kraft seit 1.9.2023)

| Pflicht | Artikel | Umsetzung in Wishful |
|---|---|---|
| Grundsätze: verhältnismässig, zweckgebunden, transparent | Art. 6 | Nur nötige Daten erheben, Adresse ausschliesslich für die Lieferung verwenden |
| **Privacy by Design und by Default** | Art. 7 | Datenschutzfreundliche Standardeinstellungen, grober Standort, verschlüsselte Adresse |
| Datensicherheit | Art. 8 | Massnahmen aus Abschnitt 6 |
| Auftragsbearbeiter | Art. 9 | Auftragsverarbeitungsverträge (AVV/DPA) mit Supabase, Stripe, Hosting, Push-Dienst |
| Verzeichnis der Bearbeitungstätigkeiten | Art. 12 | Für KMU mit < 250 MA nur bei hohem Risiko Pflicht, **trotzdem führen** |
| Bekanntgabe ins Ausland | Art. 16 | EU-Hosting ist unproblematisch, bei US-Diensten auf Zertifizierung unter dem Swiss-US Data Privacy Framework achten |
| **Informationspflicht / Datenschutzerklärung** | Art. 19 | In der App und im Store verlinkt: welche Daten, Zweck, Empfänger, Länder, Rechte |
| Datenschutz-Folgenabschätzung (DSFA) | Art. 22 | **Empfohlen** wegen Adressen, Zahlungen und Feed-Profiling |
| Meldung von Datenlecks an den EDÖB | Art. 24 | «So rasch als möglich» bei hohem Risiko, Ablauf vorab festlegen |
| Auskunftsrecht | Art. 25 | Datenexport in der App, Antwort innert 30 Tagen |
| Datenherausgabe / Portabilität | Art. 28 | Export als JSON |
| Löschung | Art. 32 | Account-Löschung mit echter Löschung (Backups mit Frist) |

> ⚠️ **Wichtig:** Das nDSG büsst **natürliche Personen** (also euch persönlich, nicht die Firma) mit bis zu **CHF 250'000**. Deshalb die Pflichten ernst nehmen und eine **GmbH gründen**.

### Weitere rechtliche Punkte
- **DSGVO:** gilt zusätzlich, sobald ihr Nutzer in der EU ansprecht. Dann braucht es eventuell eine EU-Vertretung (Art. 27 DSGVO).
- **Impressum und AGB:** Pflicht für Online-Angebote (UWG Art. 3 Abs. 1 lit. s)
- **Geldwäschereigesetz (GwG):** Wer Geld für Dritte hält, ist Finanzintermediär (FINMA). Mit Stripe Connect liegt das bei Stripe, das muss aber vertraglich sauber geregelt sein.
- **Affiliate-Links:** müssen als Werbung gekennzeichnet sein
- Für DSFA und AGB eine kurze **Rechtsberatung** einplanen (Startup-Angebote, z.B. über Hochschulen oder Startup-Programme)

---

## 8. Risiken

| Stufe | Risiko | Gegenmassnahme |
|---|---|---|
| **HOCH** | **Minderjährige erhalten Geschenke von Fremden** (Grooming-Gefahr) | 18+ im MVP, keine Direktnachrichten, Meldefunktion, Moderation |
| **HOCH** | Betrug (Fake-Wünsche, gestohlene Karten) | Stripe Radar, KYC über Connect, Limits für neue Accounts, Auszahlung verzögert |
| **HOCH** | Shops blockieren den Scraper oder Preise sind falsch | Mehrstufige Erkennung, manueller Fallback, Partner-/Affiliate-APIs, «Preis vom …» anzeigen |
| **MITTEL** | Ablehnung im App-Store-Review (Zahlungsmodell, Moderation) | Guidelines 1.2, 3.1.5 und 5.1.1 vor dem Bau prüfen |
| **MITTEL** | Datenleck bei den Adressen | Verschlüsselung, Audit-Log, Pentest |
| **NIEDRIG** | Urheberrecht an Produktbildern | Nur als Vorschau mit Quellenlink, nicht dauerhaft weiterverwenden |

---

## 9. Offene Fragen

1. Schenken über **Option A** (Geld, zweckgebunden) oder direkt **Option B** (Wishful bestellt)?
2. Mindestalter 18+, einverstanden?
3. Wer kann was? (Erfahrung mit React Native oder Backend?) → Aufteilung anpassen
4. Start nur in der Schweiz oder gleich in DACH/EU?
5. Wie verdient Wishful Geld? Gebühr pro Geschenk, Affiliate-Provision oder Premium-Abo?
