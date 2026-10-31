# 🎁 Wishful

**Teile deine Wünsche – und lass sie wahr werden.**

Wishful ist eine Social-App für iOS und Android, in der man Wünsche als Posts teilt und andere sie erfüllen können. Wünsche erstellen, im Feed entdecken und verschenken, ohne dass dabei die Privatsphäre auf der Strecke bleibt.

> 🚧 **Status:** In Planung. Details im [Projektplan](Wishful-Plan.md).

---

## ✨ So funktioniert's

1. **Wunsch posten:** Titel eingeben (z.B. *«PS5-Controller»*) und den Link zum Shop einfügen.
2. **Automatische Vorschau:** Wishful liest Produktbild, Preis und Shop direkt aus dem Link.
3. **Entdecken:** Andere scrollen durch ihren Feed und stossen auf deinen Wunsch.
4. **Erfüllen:** Wer Lust hat, kauft dir deinen Wunsch. Deine Lieferadresse bleibt dabei verborgen.

## 📱 Features

- **Wunsch-Posts** mit automatischer Preis- und Bilderkennung aus dem Produktlink
- **Zwei Feeds:** *For You* zum Entdecken und *Friends* für Freunde und Familie
- **Freunde & Familie markieren:** Beziehungen werden gegenseitig bestätigt
- **Privatsphäre zuerst:** Öffentlich sichtbar sind nur Ort/Kanton und Land. Die Adresse ist verschlüsselt gespeichert und für niemanden einsehbar.
- **Sicher bezahlen:** Zahlungen laufen über einen zertifizierten Zahlungsanbieter, Kartendaten werden nie bei uns gespeichert

## 🔒 Sicherheit & Datenschutz

Wishful arbeitet mit sensiblen Daten wie Adressen und Zahlungen. Deshalb gilt:

- Verschlüsselung der Adressdaten auf Feldebene, Zugriff nur serverseitig und protokolliert
- Hosting in der Schweiz bzw. EU
- Umsetzung nach dem **Schweizer Datenschutzgesetz (nDSG)**: Privacy by Design, Auskunfts- und Löschrecht, Datenexport

## 🛠️ Geplanter Tech-Stack

| Bereich | Technologie |
|---|---|
| App | React Native + Expo (TypeScript) |
| Backend | Supabase (PostgreSQL, Auth, Storage), Region EU/CH |
| Zahlungen | Stripe (TWINT, Karte, Apple Pay, Google Pay) |
| Link-Erkennung | Isolierter Scraper-Service |

## 👥 Team

| Name | GitHub | Fokus |
|---|---|---|
| Stefan Matteo Kauflin | [@MrKringel76](https://github.com/MrKringel76) | App & Produkt |
| Joel Felix | [@JoelF09](https://github.com/JoelF09) | Backend & Sicherheit |

---

## 📄 Lizenz

Proprietär – © 2026 Stefan Matteo Kauflin und Joel Felix. **Alle Rechte vorbehalten.**

Kopieren, Forken, Weiterverbreiten, Nutzen oder Beitragen ist ohne schriftliche Zustimmung beider Rechteinhaber nicht gestattet. Details siehe [LICENSE](LICENSE).
