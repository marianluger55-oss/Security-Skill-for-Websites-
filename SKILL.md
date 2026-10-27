---
name: website-security-check
description: Security-Prüfung für Websites und Web-Apps – von der Landingpage über CMS-Seiten bis zu Onlineshops, Shopify, Abo-Zahlungen und SaaS. Wendet eine gestaffelte Checkliste nach Projekttyp an, prüft Code, Konfiguration und Live-Seite und erstellt einen GO/NO-GO-Bericht. Verwenden vor dem Go-live, bei Security-Reviews oder Audits bestehender Projekte.
---

# Website Security-Check

Dieser Skill prüft ein konkretes Web-Projekt gegen die Checkliste in `references/checkliste.md` und erstellt einen Bericht nach `references/bericht-vorlage.md`. Er deckt nur Security ab – keine Rechtstexte, Performance oder SEO.

## Ablauf

### 1. Kontext klären
Nur fragen, was nicht aus den Projektdateien oder dem Gespräch hervorgeht:
- **Projekttyp:** Statische Seite / Landingpage, Website mit CMS, eigener Onlineshop, Shop-Plattform (z. B. Shopify), Web-App / SaaS
- **Module:** CMS (welches?), Login für Endkunden, Zahlungen, Abos, KI-Funktionen, Headless-Shop
- **Stack:** Framework (z. B. Next.js), Hosting (eigener Server/VPS, Container, Managed Hosting wie Vercel/Netlify), CDN/WAF, Zahlungsanbieter
- **Anlass:** Go-live, regelmäßiges Review, Audit, Vorfall
- **Zugang:** Repository/Projektordner, Live- oder Staging-URL, nur Beschreibung?

### 2. Tags bestimmen
Mit den Tabellen in Abschnitt 0 der Checkliste festlegen, welche Abschnitte gelten. Nur diese prüfen. Abschnitte zu Technologien, die das Projekt nicht nutzt (z. B. Next.js, Payload, WordPress, eigener Server), als „nicht zutreffend“ überspringen. `TEAM` nur, wenn auch die Organisation/das Team geprüft werden soll.

### 3. Aktuelle Advisories nachschlagen
Die Checkliste hat einen festen Stand. Vor der Prüfung per Websuche nachsehen:
- Security-Advisories der eingesetzten Frameworks, CMS und Plugins (GitHub Security Advisories, Hersteller-Blogs)
- Änderungen bei Zahlungsanbieter oder Shop-Plattform
- Eingesetzte Versionen gegen gepatchte Versionen abgleichen.

### 4. Aktiv prüfen statt nur abfragen
Wo Zugriff besteht, selbst nachsehen:
- **Code & Konfiguration:** Security-Header, Auth-Prüfungen in allen serverseitigen Endpunkten, Eingabevalidierung, Rate-Limits, öffentlich exponierte Umgebungsvariablen, Caching personalisierter Daten, Fehlerbehandlung („fail closed“)
- **CMS:** Zugriffsrechte je Inhaltstyp und Feld, Admin-Login (Lockout, 2FA, keine offene Registrierung), Upload-Beschränkungen, deaktivierte ungenutzte APIs
- **Repository & CI:** `.gitignore`, Lockfile, `npm audit`/`pnpm audit`, Secret-Scan (z. B. gitleaks), Dockerfile (`USER`), veröffentlichte Ports, CI-Workflows (Token-Rechte, gepinnte Actions)
- **Shop & Abos:** serverseitige Preisberechnung, Webhook-Verarbeitung (Signatur, Idempotenz, Reihenfolge), Abo-Status-Prüfung bei jeder Anfrage, Allowlist für Preise/Pläne
- **Live-URL:** Header, TLS, Cookies, offene Pfade (Admin, API-Playgrounds, Source Maps, `.env`, `.git`), `security.txt`
- **Nicht prüfbar** (Dashboards von Hosting, CDN, Zahlungsanbieter, Domain-Registrar, Geräte): als ❓ markieren und genau sagen, wo und was der Nutzer selbst nachsehen muss.

### 5. Bericht erstellen
Nach `references/bericht-vorlage.md` als Markdown-Datei `security-check-<projekt>.md` (Kleinbuchstaben, Bindestriche):
- Zuerst **GO / NO-GO** mit Begründung. NO-GO, sobald ein **M**-Punkt ❌ ist.
- Dann Findings: offene **M** → offene **S** → **K**-Empfehlungen, jeweils mit Punkt-Nummer, Fundstelle (Datei:Zeile oder URL) und konkreter Behebung.
- Danach die gefilterte Checkliste mit Status je Punkt: ✅ erfüllt (mit Beleg) · ❌ offen · ⚠️ bewusst abweichend (Begründung) · ❓ nicht prüfbar · – nicht zutreffend.

### 6. Beheben (optional)
Code nur ändern, wenn der Nutzer das ausdrücklich will. Dann **M**-Punkte zuerst und jede Änderung im Bericht nachtragen.

## Regeln

- Einen **M**-Punkt nie ohne Beleg als ✅ markieren. Im Zweifel ❓.
- Gefundene Secrets, Tokens oder Passwörter nie im Bericht, Chat oder Log wiedergeben – nur Fundstelle nennen und Rotation empfehlen.
- Keine Rechtsberatung. Rechtliche Fragen (Datenschutz, Impressum, Verträge) nur als Hinweis an Fachleute weitergeben.
- Der Bericht ersetzt keinen Penetrationstest. Bei Zahlungen, Gesundheitsdaten oder anderen sensiblen Daten einen externen Test empfehlen.
- Neue Erkenntnisse aus Advisories am Ende des Berichts als „Vorschlag: Checkliste ergänzen“ aufführen.
