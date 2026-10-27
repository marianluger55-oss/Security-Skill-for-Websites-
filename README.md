# website-security-check

Ein Claude-Skill für Security-Checks von Websites und Web-Apps – von der Landingpage über CMS-Seiten bis zu Onlineshops, Shopify, Abo-Zahlungen und SaaS.

Claude bestimmt anhand des Projekttyps, welche Teile der Checkliste gelten, sucht aktuelle Security-Advisories, prüft Code, Konfiguration und Live-Seite selbst und schreibt einen Bericht mit klarem **GO / NO-GO**, Fundstellen und konkreten Behebungsschritten.

## Inhalt

```
website-security-check/
├── SKILL.md                     # Ablauf und Regeln für Claude
└── references/
    ├── checkliste.md            # ~360 Prüfpunkte in 25 Abschnitten
    └── bericht-vorlage.md       # Aufbau des Prüfberichts
```

**Abgedeckt:** Accounts & 2FA, Domain/DNS/E-Mail (SPF, DKIM, DMARC), CDN/WAF, Server- und Container-Härtung, Backups, Security-Header & CSP, Next.js/React, CMS (allgemein, Payload, WordPress), Formulare, Drittanbieter-Skripte, Supply Chain & CI/CD, Secrets, Login & Sessions, Autorisierung/IDOR, Injection & „fail closed“, APIs, Logging & Incident Response, Onlineshop & PCI DSS, Webhooks, Abo-Zahlungen, Shopify, SaaS, KI-Funktionen, Übergabe & Betrieb.

Jeder Punkt hat eine Priorität:
- **M** – Muss (Go-live-Blocker)
- **S** – Soll (Standard, Abweichung begründen)
- **K** – Kann (für höheres Risiko)

Abgeglichen mit OWASP Top 10:2025, dem OWASP Next.js Security Cheat Sheet, PCI DSS v4.0.1 SAQ A, Stripe- und Shopify-Security-Empfehlungen (Stand Oktober 2026).

## Installation

**Claude (App/Web):** Ordner als ZIP packen und unter *Einstellungen → Skills* hochladen.

**Claude Code:** Ordner nach `~/.claude/skills/website-security-check/` (global) oder `.claude/skills/website-security-check/` (pro Projekt) kopieren.

```bash
git clone https://github.com/<user>/website-security-check.git ~/.claude/skills/website-security-check
```

## Benutzung

Beispiele:
- „Mach einen Security-Check für dieses Projekt, es geht nächste Woche live.“
- „Prüfe meinen Shopify-Shop auf Sicherheitslücken.“
- „Security-Review für unsere Next.js-App mit Stripe-Abos.“

Am meisten bringt der Check, wenn Claude Zugriff auf das Repository und eine erreichbare URL hat. Was sich nur in Dashboards prüfen lässt (Hosting, CDN, Zahlungsanbieter, Registrar), markiert Claude mit ❓ und sagt, wo du selbst nachsehen musst.

## Anpassen

- Werkzeuge und Anbieter in der Checkliste sind Beispiele – ersetze sie durch deinen Stack.
- Eigene Abschnitte mit eigenem Tag anlegen und in der Tabelle „Projekttyp → Tags“ eintragen.
- Platzhalter in `[eckigen Klammern]` (Backup-Ziele, Intervalle, Löschfristen) für dich festlegen.

## Hinweis

Der Skill ist ein strukturierter Check, **kein Ersatz für einen Penetrationstest oder Rechtsberatung**. Die Checkliste hat einen festen Stand; neue Schwachstellen kommen laufend dazu. Keine Gewähr für Vollständigkeit.
