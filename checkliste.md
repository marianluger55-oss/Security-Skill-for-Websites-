# Website Security-Checkliste

> Stand: Oktober 2026 · abgeglichen mit OWASP Top 10:2025, OWASP Next.js Security Cheat Sheet, PCI DSS v4.0.1 SAQ A (FAQ 1588), Stripe-Security-Empfehlungen, Shopify-Security-Checklisten 2026, Payload-Security-Advisories.
> Gilt für Websites und Web-Apps von der Landingpage bis zum Shop mit Abo-Zahlungen. Nur Security – Rechtliches, Performance und SEO gehören in eigene Checklisten.
> Werkzeuge und Anbieter sind **Beispiele**. Wer andere einsetzt, prüft den gleichen Punkt beim eigenen Anbieter.
> Platzhalter in [eckigen Klammern] = projektspezifisch festzulegen.

---

## 0. So wird die Checkliste benutzt

### Tags

Jeder Abschnitt trägt Tags. Ein Projekt arbeitet **alle Abschnitte ab, deren Tags zutreffen**. Technologie-Abschnitte (Next.js, Payload, WordPress, eigener Server) entfallen, wenn die Technologie nicht eingesetzt wird.

| Tag | Bedeutung |
|---|---|
| `BASIS` | Grundschutz für jedes Projekt |
| `CMS` | Content-Management-System / Admin-Bereich |
| `SHOP` | eigener Onlineshop mit Zahlungsanbieter |
| `SHOPIFY` | Shop auf Shopify (klassisch oder headless) |
| `ABO` | wiederkehrende Zahlungen, Mitgliedschaften, SaaS-Abrechnung |
| `APP` | Login für Endkunden, Rollen, APIs, Mandanten |
| `KI` | KI-Funktionen (Chatbot, Assistent, Automatisierung) |
| `TEAM` | Organisation: Accounts, Geräte, Zugänge des Teams/der Agentur |

### Projekttyp → Tags

| Projekttyp | Tags |
|---|---|
| Statische Seite / Landingpage | BASIS |
| Website mit CMS | BASIS, CMS |
| Website mit Login-Bereich | BASIS, CMS, APP |
| Eigener Onlineshop | BASIS, CMS, SHOP (+ ABO) |
| Shopify-Shop | BASIS, SHOPIFY (+ CMS bei headless, + ABO) |
| Web-App / SaaS | BASIS, APP (+ CMS, SHOP, ABO, KI je nach Umfang) |

### Prioritäten

- **M** = Muss – Go-live-Blocker. Nicht erfüllt = NO-GO.
- **S** = Soll – Standard; Abweichung nur begründet und dokumentiert.
- **K** = Kann – für höheres Risiko, sensible Daten oder wenn Budget da ist.

---

## 1. Team, Accounts & Geräte `TEAM`

### 1.1 Identitäten & Passwörter
- [ ] **M** Passwortmanager für alle Zugänge; keine Passwörter in Notizen, Mails, Docs oder Chats
- [ ] **M** Pro Projekt/Kunde ein eigener Tresor; Teilen nur über den Passwortmanager
- [ ] **M** Jedes Passwort einzigartig, generiert, ≥ 16 Zeichen
- [ ] **M** 2FA überall, wo möglich – bevorzugt Passkeys/Security-Key, sonst TOTP; SMS nur, wenn nichts anderes geht
- [ ] **M** Höchste Schutzstufe (Passkey/Security-Key) für: E-Mail, Code-Hosting (z. B. GitHub), Hosting, CDN/DNS, Domain-Registrar, Zahlungsanbieter, Shop-Plattform, Passwortmanager
- [ ] **M** Wiederherstellungscodes offline und getrennt vom Passwortmanager gesichert
- [ ] **S** Zweiter Security-Key als Backup registriert

### 1.2 E-Mail & Office-Konten
- [ ] **M** 2FA für alle Nutzer erzwungen; Admin-Konto getrennt vom Alltagskonto
- [ ] **M** SPF, DKIM, DMARC für die eigene Domain
- [ ] **S** Drittanbieter-App-Zugriffe (OAuth) eingeschränkt und regelmäßig geprüft
- [ ] **S** Freigaben von Ordnern/Dokumenten nie „Jeder mit dem Link“ für Kundendaten; quartalsweise prüfen
- [ ] **S** Warnungen bei verdächtigen Logins und neuen Weiterleitungsregeln aktiv

### 1.3 Geräte
- [ ] **M** Festplattenverschlüsselung auf allen Arbeitsgeräten
- [ ] **M** Betriebssystem und Browser automatisch aktualisiert, Bildschirmsperre ≤ 5 Min.
- [ ] **M** Keine Admin-Zugänge auf fremden oder geteilten Geräten
- [ ] **S** SSH-Keys mit Passphrase, idealerweise in Hardware-Key oder Passwortmanager-Agent
- [ ] **S** Eigenes Browserprofil für Admin-Arbeit

### 1.4 Zugangsverwaltung & Offboarding
- [ ] **M** Zugangsliste pro Projekt: Dienst, Person, Rolle, seit wann
- [ ] **M** Beim Ausscheiden einer Person: alle Zugänge am selben Tag entzogen, geteilte Secrets rotiert
- [ ] **M** Jede Rolle bekommt nur, was sie braucht (z. B. Vertrieb kein Zugriff auf Server, Repos, Zahlungsanbieter)
- [ ] **S** Quartalsweiser Access-Review über alle Dienste
- [ ] **S** Selbst gehostete interne Tools (Dashboards, Automatisierung, Social-Media-Tools) nicht ungeschützt öffentlich; Registrierung deaktiviert
- [ ] **S** Keine Secrets, Zugangsdaten oder personenbezogenen Daten in Prompts von KI-Tools

---

## 2. Domain, DNS & E-Mail `BASIS`

### 2.1 Registrar
- [ ] **M** Domain gehört dem Betreiber/Kunden; Dienstleister bekommen delegierten Zugang
- [ ] **M** Registrar-Konto mit 2FA; Transfer-Lock aktiv
- [ ] **M** Auto-Renew aktiv, Kontakt-Mail ist ein überwachtes Postfach
- [ ] **S** WHOIS-Privacy aktiv (soweit die Registry es erlaubt)

### 2.2 DNS
- [ ] **M** Keine „hängenden“ Records (CNAME/A auf gelöschte Dienste → Subdomain-Takeover)
- [ ] **M** Bei CDN/WAF davor: Server-IP nicht über alte Records, Neben-Subdomains oder Mail-Header verraten
- [ ] **S** DNSSEC aktiv
- [ ] **S** CAA-Record erlaubt nur die tatsächlich genutzten Zertifizierungsstellen
- [ ] **K** Certificate-Transparency-Monitoring (Benachrichtigung bei neuen Zertifikaten)

### 2.3 E-Mail-Authentifizierung (auch wenn die Website „nur“ Formularmails schickt)
- [ ] **M** SPF korrekt, max. 10 DNS-Lookups, endet mit `-all` oder `~all`
- [ ] **M** DKIM für den Versanddienst und den Mailprovider
- [ ] **M** DMARC gesetzt; Start `p=none` mit Reports, dann `quarantine` → `reject`
- [ ] **M** Domains ohne Mailversand: `v=spf1 -all`, DMARC `p=reject`, Null-MX
- [ ] **S** Transaktionsmails aus Subdomain (z. B. `mail.beispiel.at`), damit die Reputation getrennt bleibt
- [ ] **K** MTA-STS + TLS-RPT

---

## 3. CDN / WAF (z. B. Cloudflare) `BASIS`

*Falls kein CDN/WAF eingesetzt wird: Rate-Limiting, TLS und Caching-Regeln am Server bzw. beim Hoster umsetzen.*

- [ ] **M** TLS zwischen CDN und Server verschlüsselt und validiert (Cloudflare: **Full (strict)**, nie „Flexible“)
- [ ] **M** HTTP → HTTPS-Weiterleitung aktiv; Minimum TLS 1.2, TLS 1.3 aktiv
- [ ] **M** Server nimmt Web-Traffic **nur vom CDN** an (Firewall auf CDN-IP-Bereiche) – sonst ist die WAF umgehbar
- [ ] **M** WAF-Regelwerk aktiv (Managed Rules / OWASP Core Ruleset)
- [ ] **M** Rate-Limiting für Login, Formulare, Checkout, API-Routen
- [ ] **M** Admin-, API-, Konto-, Warenkorb- und Checkout-Seiten werden **nie gecacht**
- [ ] **S** Authenticated Origin Pulls / mTLS zwischen CDN und Server
- [ ] **S** Bot-Schutz aktiv – auf Kompatibilität mit Webhooks getestet
- [ ] **S** Admin-Bereich zusätzlich geschützt (Zero-Trust-Zugang, IP- oder Länder-Einschränkung)
- [ ] **S** Captcha-Alternative (z. B. Turnstile, hCaptcha, Friendly Captcha) für öffentliche Formulare
- [ ] **S** HSTS nur an einer Stelle gesetzt (CDN **oder** App), nicht widersprüchlich
- [ ] **K** Notfallmodus (z. B. „Under Attack“) als dokumentierte Maßnahme bekannt
- [ ] **K** Script-Monitoring auf Checkout-Seiten

---

## 4. Eigener Server / VPS & Container `BASIS`

*Bei Managed Hosting (z. B. Vercel, Netlify): Abschnitt 4.1–4.4 entfällt; stattdessen 2FA, Rollen und Umgebungsvariablen beim Hoster prüfen. 4.5 gilt immer.*

### 4.1 Zugang
- [ ] **M** SSH nur mit Key, Passwort-Login deaktiviert
- [ ] **M** Root-Login per SSH deaktiviert, eigener Benutzer mit `sudo`
- [ ] **M** SSH nicht offen für alle – Firewall auf eigene IP(s) oder Zugang über VPN/Tailscale
- [ ] **S** Fail2ban oder CrowdSec gegen Brute-Force
- [ ] **S** Hosting-Konsole mit 2FA; Projekte/Kunden klar getrennt

### 4.2 Firewall & Netz
- [ ] **M** Eingehend nur 80/443 (bei CDN: nur von dessen IPs) + SSH (eingeschränkt)
- [ ] **M** Datenbank, Redis, Admin-Dashboards **nie** öffentlich erreichbar
- [ ] **M** Docker-Falle beachtet: veröffentlichte Ports umgehen `ufw` → Ports an `127.0.0.1` binden oder Firewall des Hosters nutzen
- [ ] **S** Private Netzwerke zwischen Servern statt öffentlicher IPs

### 4.3 System
- [ ] **M** Automatische Sicherheitsupdates (z. B. `unattended-upgrades`)
- [ ] **M** Reboot-Bedarf nach Kernel-Updates überwacht
- [ ] **M** Nur benötigte Dienste laufen; Standardpasswörter überall geändert
- [ ] **S** Zeit-Synchronisation aktiv (wichtig für TOTP, Logs, Webhook-Zeitstempel)
- [ ] **S** Log-Rotation und Speicherplatz-Monitoring

### 4.4 Container & Reverse Proxy
- [ ] **M** Container laufen nicht als root (`USER` im Dockerfile)
- [ ] **M** Offizielle, schlanke Base-Images, Versionen gepinnt, regelmäßig neu gebaut
- [ ] **M** Reverse Proxy (Caddy/Traefik/Nginx) terminiert TLS und leitet nur an interne Ports
- [ ] **S** Container mit `read_only`, `no-new-privileges`, gedroppten Capabilities, Ressourcenlimits
- [ ] **S** Image-Scan (z. B. Trivy) in der CI
- [ ] **S** Projekte voneinander isoliert (eigene Container, Netze, DB-User, Volumes)
- [ ] **K** Docker-Socket nie in Container mounten

### 4.5 Backups & Wiederherstellung
- [ ] **M** Automatische Backups von Datenbank **und** Uploads/Medien
- [ ] **M** 3-2-1-Prinzip: mindestens eine Kopie außerhalb des Servers und Anbieters
- [ ] **M** Backups verschlüsselt (z. B. restic, borg), Schlüssel getrennt gesichert
- [ ] **M** Wiederherstellung real getestet und dokumentiert (Datum, Dauer)
- [ ] **S** Backups unveränderlich/versioniert (Schutz vor Ransomware und versehentlichem Löschen)
- [ ] **S** RPO/RTO festgelegt: [wie viel Datenverlust, wie lange Ausfall ist akzeptabel]
- [ ] **S** Snapshot vor jedem größeren Update

---

## 5. Transport & HTTP-Security-Header `BASIS`

- [ ] **M** HTTPS überall, kein Mixed Content
- [ ] **M** `Strict-Transport-Security: max-age=31536000; includeSubDomains` (Preload erst, wenn alle Subdomains HTTPS können)
- [ ] **M** `Content-Security-Policy` definiert und getestet; für Skripte kein pauschales `unsafe-inline`/`unsafe-eval` (Nonces oder Hashes)
- [ ] **M** `frame-ancestors 'none'` (bzw. `'self'`) in der CSP + `X-Frame-Options: DENY` als Fallback
- [ ] **M** `X-Content-Type-Options: nosniff`
- [ ] **M** `Referrer-Policy: strict-origin-when-cross-origin` (oder strenger)
- [ ] **M** `Permissions-Policy`: Kamera, Mikrofon, Geolocation, Payment usw. aus, wenn nicht gebraucht
- [ ] **M** Server-/Framework-Header entfernt (`X-Powered-By`, Versionsangaben)
- [ ] **S** `Cross-Origin-Opener-Policy: same-origin`
- [ ] **S** `Cross-Origin-Resource-Policy: same-origin` (wo keine Einbettung durch Dritte nötig)
- [ ] **S** CSP zuerst als `Report-Only` mit Reporting ausrollen, auswerten, dann scharf schalten
- [ ] **K** `/.well-known/security.txt` mit Kontaktadresse und Ablaufdatum

---

## 6. Framework: Next.js / React `BASIS` *(nur wenn eingesetzt)*

*Für andere Frameworks sinngemäß anwenden: jeder serverseitige Endpunkt prüft Auth, keine Secrets im Client, Framework aktuell.*

### 6.1 Versionen & Advisories
- [ ] **M** Next.js und React auf aktuellem Patch-Stand; Security-Advisories abonniert
- [ ] **M** Bekannte kritische Lücken geprüft (z. B. Middleware-Bypass CVE-2025-29927, React-Server-Components-RCE CVE-2025-55182, Fixes vom Oktober 2026 zu SVG-Bildoptimierung und `use cache`)
- [ ] **M** Auth-Prüfung **nie nur in Middleware/Proxy** (`middleware.ts`, ab Next.js 16 `proxy.ts`) – zusätzlich in Server Components, Route Handlers und Server Actions
- [ ] **M** Server-only Data Access Layer: prüft Berechtigungen und gibt nur minimale DTOs zurück

### 6.2 Server Actions & Route Handlers
- [ ] **M** Jede Server Action wie einen öffentlichen API-Endpunkt behandeln: Auth, Berechtigung, Validierung (z. B. zod)
- [ ] **M** Keine IDs, Preise oder Rollen aus dem Client ungeprüft übernehmen
- [ ] **M** Route Handlers: erlaubte Methoden explizit, Rate-Limiting für öffentliche Routen
- [ ] **S** `serverActions.allowedOrigins` nur setzen, wenn wirklich nötig

### 6.3 Daten- & Secret-Leaks
- [ ] **M** Nur `NEXT_PUBLIC_`-Variablen landen im Browser – geprüft, dass dort **kein** Secret steht
- [ ] **M** `server-only` in allen Modulen mit Secrets oder DB-Zugriff
- [ ] **M** An Client Components nur benötigte Felder übergeben (keine kompletten DB-Objekte)
- [ ] **M** Source Maps in Produktion nicht öffentlich
- [ ] **M** Fehlerseiten ohne Stacktraces, Pfade, Versionen
- [ ] **S** React Taint APIs als zusätzliche Absicherung, wo sinnvoll

### 6.4 Caching, Bilder, Redirects
- [ ] **M** `use cache` / `unstable_cache`: keine nutzerspezifischen Daten ohne Nutzer im Cache-Key
- [ ] **M** `next/image`: `remotePatterns` eng auf eigene Domains/Buckets beschränkt
- [ ] **M** Redirects mit Nutzereingaben (`?redirect=`, `?next=`) nur auf relative Pfade oder Allowlist
- [ ] **M** Revalidation-, Preview- und Draft-Mode-Endpunkte mit Secret geschützt
- [ ] **M** `dangerouslySetInnerHTML` nur mit sanitisiertem HTML (DOMPurify, sanitize-html)
- [ ] **S** Personalisierte Inhalte nie in geteilten Cache
- [ ] **S** Cache Poisoning: CDN-Cache hängt nicht an ungeprüften Headern; `X-Forwarded-Host` nur vom eigenen Proxy vertrauen
- [ ] **S** `dangerouslyAllowSVG` aus, sonst strikte `contentSecurityPolicy` für Bilder

---

## 7. CMS & Admin-Bereich `CMS`

### 7.1 Allgemein (jedes CMS)
- [ ] **M** 2FA für **alle** Admin- und Redakteurskonten
- [ ] **M** Login-Lockout bzw. Verzögerung nach Fehlversuchen
- [ ] **M** Keine öffentliche Registrierung im Admin-Bereich
- [ ] **M** Persönliche Konten statt Sammelkonten (`admin@`)
- [ ] **M** Rollen nach Least Privilege; Betreiber/Kunde nicht automatisch Superadmin
- [ ] **M** CMS, Plugins und Themes auf aktuellem Stand; Advisories beobachtet
- [ ] **M** Ungenutzte APIs/Endpunkte (GraphQL, Playground, XML-RPC o. Ä.) deaktiviert
- [ ] **S** Admin-Pfad zusätzlich geschützt (siehe 3)
- [ ] **S** Session-Laufzeit bewusst kurz

### 7.2 Payload CMS *(nur wenn eingesetzt)*
- [ ] **M** Für **jede** Collection und jedes Global `read`, `create`, `update`, `delete` explizit definiert
- [ ] **M** Feld-Level-Access für sensible Felder (Rolle, interne Notizen, Preise, Zahlungsstatus)
- [ ] **M** Nutzer können ihre eigene Rolle **nicht** ändern
- [ ] **M** Local API: `overrideAccess` ist standardmäßig `true` → im Namen eines Nutzers `overrideAccess: false` + `user` übergeben
- [ ] **M** In Hooks `req` an `payload.find/update/delete` weitergeben, damit Access-Funktionen greifen
- [ ] **M** `maxLoginAttempts` und `lockTime` in der Users-Collection gesetzt; `create` nur für Admins
- [ ] **M** Payload auf Patch-Version gepinnt, GitHub-Advisories beobachtet (2025/26 u. a. SSRF über Upload-Collections, Prüfung beim Konto-Entsperren)
- [ ] **M** `PAYLOAD_SECRET` lang, zufällig, je Umgebung unterschiedlich
- [ ] **M** `cors` und `csrf` auf eigene Domains beschränkt; `depth`-Limit gesetzt
- [ ] **S** Versteckte Felder nicht über `where`-Abfragen erratbar
- [ ] **S** Access-Funktionen mit Tests abgedeckt (anonym / Redakteur / Admin)

### 7.3 WordPress *(nur wenn eingesetzt)*
- [ ] **M** Nur gepflegte Plugins/Themes aus vertrauenswürdiger Quelle; ungenutzte gelöscht (nicht nur deaktiviert)
- [ ] **M** Automatische Updates für Core und Sicherheitsupdates von Plugins
- [ ] **M** `DISALLOW_FILE_EDIT` gesetzt (kein Code-Editor im Admin)
- [ ] **M** XML-RPC deaktiviert, wenn nicht gebraucht; Benutzer-Enumeration (`/?author=`, REST `/wp/v2/users`) unterbunden
- [ ] **M** `wp-config.php` und `.git`/Backups nicht über den Webserver abrufbar; PHP-Ausführung in `uploads/` gesperrt
- [ ] **S** Datenbankpräfix und DB-User mit minimalen Rechten

### 7.4 Uploads & Medien
- [ ] **M** Erlaubte Dateitypen per Allowlist, Dateigröße begrenzt
- [ ] **M** SVG-Upload nur, wenn nötig – dann sanitisiert (SVG kann JavaScript enthalten)
- [ ] **M** Keine ausführbaren Dateien; HTML-Uploads nur, wenn ausdrücklich gebraucht
- [ ] **M** Upload per URL deaktiviert oder auf Allowlist beschränkt (SSRF)
- [ ] **S** Medien mit `Content-Disposition`/`nosniff` ausgeliefert, idealerweise von eigener Subdomain
- [ ] **S** Private Dateien (Rechnungen, Verträge) getrennt mit Zugriffskontrolle, nicht im öffentlichen Medienordner
- [ ] **K** Virenscan für Uploads von Endkunden

---

## 8. Formulare, Spam & E-Mail-Versand `BASIS`

- [ ] **M** Serverseitige Validierung aller Felder (Typ, Länge, Format)
- [ ] **M** Bot-Schutz: Honeypot + Captcha-Alternative oder Zeitstempel-Prüfung
- [ ] **M** Rate-Limiting pro IP
- [ ] **M** Keine ungeprüften Nutzereingaben in `To`, `Cc`, `Subject`, `Reply-To` (Header-Injection)
- [ ] **M** Nutzereingaben in Benachrichtigungsmails escaped
- [ ] **M** API-Key des Mail-Versanddienstes nur serverseitig, mit minimalen Rechten, auf Domain beschränkt
- [ ] **M** Keine „Kopie an mich“-Funktion an beliebige Adressen
- [ ] **S** Newsletter: Double-Opt-in, Abmeldelink, Anmelde-Endpunkt rate-limited („Subscription Bombing“)
- [ ] **S** Rückmeldungen verraten nicht, ob eine E-Mail-Adresse schon existiert

---

## 9. Drittanbieter-Skripte & Einbettungen `BASIS`

- [ ] **M** Inventar aller externen Skripte und iFrames (wer, warum, welche Domain)
- [ ] **M** Externe Skripte nur nach Einwilligung, wo rechtlich nötig, und nur von vertrauenswürdigen Anbietern
- [ ] **M** Fonts, Icons und Libraries selbst hosten statt Fremd-CDN (Supply-Chain-Risiko, vgl. Polyfill.io 2024)
- [ ] **S** Subresource Integrity für unvermeidbare externe Skripte mit fester Version
- [ ] **S** iFrames (Karten, Videos, Buchungstools) mit `sandbox` und minimalen Rechten
- [ ] **S** Tag Manager nur, wenn nötig – Zugriff darauf ist faktisch Zugriff auf die Website
- [ ] **S** Analytics-Skript über eigene Domain/Proxy, CSP entsprechend

---

## 10. Abhängigkeiten, Repository & CI/CD `BASIS`

### 10.1 Abhängigkeiten
- [ ] **M** Lockfile committet; Installation in CI mit `npm ci` / `pnpm install --frozen-lockfile`
- [ ] **M** `npm audit` / `pnpm audit` ohne kritische oder hohe Findings vor dem Go-live
- [ ] **M** Dependabot oder Renovate aktiv
- [ ] **M** Neue Pakete vor Aufnahme geprüft (Downloads, Maintainer, Alter, Tippfehler-Namen)
- [ ] **S** Install-Skripte von Paketen eingeschränkt (z. B. pnpm `onlyBuiltDependencies`) – Schutz gegen npm-Wurm-Vorfälle wie 2025
- [ ] **S** Neue Versionen nicht am Erscheinungstag übernehmen (z. B. Renovate `minimumReleaseAge`)
- [ ] **S** SBOM pro Projekt (z. B. `npm sbom`, CycloneDX), damit bei neuen CVEs sofort klar ist, was betroffen ist
- [ ] **S** Automatischer Schwachstellen-Scan (z. B. OWASP ZAP Baseline) vor dem Go-live

### 10.2 Repository
- [ ] **M** `.env*` in `.gitignore`; Secret-Scanning + Push Protection aktiv
- [ ] **M** Repo-Historie einmal mit gitleaks/trufflehog geprüft
- [ ] **M** Branch Protection auf dem Hauptbranch: kein Force-Push, CI muss grün sein
- [ ] **S** Repositories privat, außer bewusst Open Source
- [ ] **K** Signierte Commits

### 10.3 CI/CD (z. B. GitHub Actions)
- [ ] **M** CI-Token mit minimalen Rechten (`permissions: contents: read` als Standard)
- [ ] **M** Fremde Actions auf Commit-SHA gepinnt, nicht auf `@v1`/`@main`
- [ ] **M** Deployment-Secrets in geschützten Environments (nur vom Hauptbranch)
- [ ] **M** Kein `pull_request_target` mit Checkout von fremdem Code
- [ ] **S** Deploy-Zugang zum Server nur mit Deployment-Rechten (eigener User, kein root)
- [ ] **S** SAST (z. B. Semgrep, CodeQL) und Image-Scan in der Pipeline

---

## 11. Secrets & Konfiguration `BASIS`

- [ ] **M** Keine Secrets im Code, in Commits, im Client-Bundle, in Logs oder Tickets
- [ ] **M** Getrennte Secrets für Entwicklung, Staging, Produktion; Test-Keys nie in Produktion, Live-Keys nie lokal
- [ ] **M** Jeder API-Key mit minimalen Rechten (Restricted Keys, scoped Tokens)
- [ ] **M** Secrets-Inventar pro Projekt: Name, Zweck, Ort, Rotationsdatum
- [ ] **M** Rotation sofort bei Verdacht, nach Personalwechsel und bei Projektende
- [ ] **S** Secrets auf dem Server mit restriktiven Dateirechten (`600`), nicht in Images gebacken
- [ ] **S** Regelmäßige Rotation [Intervall festlegen]

---

## 12. Authentifizierung für Endkunden `APP` `SHOP`

### 12.1 Passwörter
- [ ] **M** Hashing mit Argon2id (oder bcrypt mit ausreichendem Cost-Faktor) – nie selbst gebaut
- [ ] **M** Mindestlänge ≥ 12, lange Passphrasen erlaubt, keine Zwangs-Sonderzeichen (NIST-Ansatz)
- [ ] **M** Abgleich gegen geleakte Passwörter (z. B. Have I Been Pwned, k-Anonymity)
- [ ] **M** Login-Fehlermeldung verrät nicht, ob die E-Mail existiert
- [ ] **M** Rate-Limiting + Lockout/Verzögerung pro Account **und** pro IP

### 12.2 Passwort-Reset & E-Mail-Änderung
- [ ] **M** Reset-Token zufällig, gehasht gespeichert, einmalig, Ablauf ≤ 60 Min.
- [ ] **M** Nach Reset alle Sessions invalidiert
- [ ] **M** Reset-Link nicht aus dem `Host`-Header gebaut (Host-Header-Injection)
- [ ] **M** E-Mail-Änderung mit Bestätigung an alte **und** neue Adresse
- [ ] **S** Benachrichtigung bei Passwort-/E-Mail-Änderung und neuem Login

### 12.3 Sessions & Tokens
- [ ] **M** Cookies `Secure`, `HttpOnly`, `SameSite=Lax` (oder `Strict`), möglichst `__Host-`-Präfix
- [ ] **M** Neue Session-ID nach Login (Session Fixation)
- [ ] **M** Logout invalidiert serverseitig
- [ ] **M** Absolute und Idle-Timeouts gesetzt
- [ ] **M** JWT (falls genutzt): fester Algorithmus (kein `none`), kurze Laufzeit, Refresh-Rotation, Widerruf möglich
- [ ] **M** Tokens nie in `localStorage` und nie in URLs
- [ ] **S** Bewährte Auth-Bibliothek statt Eigenbau

### 12.4 2FA & Social Login
- [ ] **M** OAuth/Social Login: `state` + PKCE, Redirect-URIs exakt registriert, E-Mail nur übernehmen, wenn vom Provider verifiziert
- [ ] **S** 2FA (TOTP/Passkeys) angeboten; für Admin- und Verwaltungsrollen erzwungen
- [ ] **S** Wiederherstellungscodes einmalig, gehasht gespeichert
- [ ] **K** Passkeys als primäre Login-Methode

---

## 13. Autorisierung & Mandantentrennung `APP` `SHOP`

- [ ] **M** Jede Anfrage prüft, ob das Objekt dem Nutzer gehört (IDOR – Bestellungen, Rechnungen, Profile, Dateien)
- [ ] **M** Rollenprüfung serverseitig; ausgeblendete Buttons sind kein Schutz
- [ ] **M** Mass Assignment verhindert: nie `role`, `isAdmin`, `price`, `status` aus Requests übernehmen
- [ ] **M** Mehrere Mandanten: Mandanten-ID in **jeder** Abfrage erzwungen, nie aus dem Request
- [ ] **S** Nicht erratbare IDs für öffentlich sichtbare Ressourcen – zusätzlich zur Prüfung, nicht stattdessen
- [ ] **S** Funktionen wie „Als Nutzer einloggen“ protokolliert und auf Admins beschränkt
- [ ] **S** Autorisierungstests für jede Rolle

---

## 14. Eingaben, Injection & Fehlerbehandlung `BASIS`

- [ ] **M** SQL/NoSQL: nur ORM oder parametrisierte Queries; keine Nutzerobjekte direkt als Filter (`$ne`, `$gt`)
- [ ] **M** XSS: Framework-Escaping nicht umgehen; Rich-Text über sicheren Renderer
- [ ] **M** SSRF: keine vom Nutzer gelieferten URLs abrufen – oder nur per Allowlist, interne IP-Bereiche und Metadaten-Endpunkte gesperrt
- [ ] **M** Path Traversal: Dateinamen aus Eingaben nie direkt in Pfade
- [ ] **M** Keine Shell-Befehle mit Nutzereingaben (auch bei PDF-/Bild-Erzeugung)
- [ ] **S** Regex ohne katastrophales Backtracking (ReDoS)
- [ ] **S** Request-Body-Größe begrenzt
- [ ] **S** CSRF-Schutz (SameSite-Cookies + Origin-Prüfung) für zustandsändernde Requests

### 14.1 Fehlerbehandlung – „fail closed“ (OWASP A10:2025)
- [ ] **M** Bei Fehlern wird Zugriff **verweigert**, nie erlaubt (Auth-Dienst, DB oder Zahlungsanbieter nicht erreichbar → keine Freischaltung)
- [ ] **M** Kein `try/catch`, der Fehler verschluckt und mit „erlaubt“/„bezahlt“/„admin“ weitermacht
- [ ] **M** Unbehandelte Exceptions/Promise-Rejections bringen den Server nicht zum Absturz
- [ ] **S** Mehrstufige Vorgänge (Bestellung, Abo-Wechsel) transaktional – kein halber Zustand bei Abbruch
- [ ] **S** Timeouts und Limits für externe Aufrufe

---

## 15. API-Sicherheit `APP`

- [ ] **M** Jeder Endpunkt: Authentifizierung, Autorisierung, Validierung, Rate-Limit
- [ ] **M** CORS nur für eigene Origins; nie `*` mit Credentials
- [ ] **M** Antworten enthalten nur benötigte Felder
- [ ] **M** Pagination mit Maximalgröße
- [ ] **S** API-Keys für Integrationen: gehasht, pro Kunde, widerrufbar, mit Ablaufdatum
- [ ] **S** Ausgehende Webhooks signiert (HMAC); Ziel-URLs gegen SSRF geprüft
- [ ] **S** Teure Operationen (Exporte, Reports, KI) mit Kontingent pro Nutzer
- [ ] **K** OWASP API Security Top 10 vollständig durchgegangen
- [ ] **K** Bei sensiblen Daten Abgleich mit OWASP ASVS (Level 1 Minimum, Level 2 für Zahlungen/Gesundheitsdaten)

---

## 16. Datenschutz-relevante Sicherheit `BASIS`

*Nur der technische Teil. Rechtliche Anforderungen (z. B. DSGVO in der EU) mit Fachleuten klären.*

- [ ] **M** Datenminimierung: nur erheben und speichern, was gebraucht wird
- [ ] **M** Personenbezogene Daten verschlüsselt übertragen und auf verschlüsselten Datenträgern/Backups gespeichert
- [ ] **M** Formulareinsendungen und Leads mit Löschfrist [festlegen]
- [ ] **S** Besonders sensible Daten (Gesundheit, Ausweise) zusätzlich feldverschlüsselt oder gar nicht über die Website erhoben
- [ ] **S** Datenexport und Löschung von Nutzerkonten technisch umsetzbar
- [ ] **S** Verträge zur Auftragsverarbeitung mit allen Diensten, die personenbezogene Daten verarbeiten

---

## 17. Logging, Monitoring & Incident Response `BASIS`

### 17.1 Logging
- [ ] **M** Keine Passwörter, Tokens, Kartendaten oder Reset-Links in Logs
- [ ] **M** Sicherheitsereignisse protokolliert: Logins, Rollenänderungen, Passwort-Resets, Admin-Aktionen, Zahlungs-Webhooks
- [ ] **S** Aufbewahrung begrenzt [Dauer festlegen], IP-Adressen ggf. gekürzt

### 17.2 Monitoring
- [ ] **M** Uptime-Monitoring für Website, Admin, API, Checkout
- [ ] **M** Ablauf von Zertifikat und Domain überwacht
- [ ] **S** Alarm bei vielen fehlgeschlagenen Logins, 5xx-Spitzen, ungewöhnlichen Bestellungen/Zahlungsfehlern
- [ ] **S** Webhook-Fehler überwacht – fehlgeschlagene Events gehen sonst still verloren

### 17.3 Incident Response
- [ ] **M** Notfallplan (1 Seite): erkennen → eindämmen (Wartungsmodus, Keys rotieren, Zugänge sperren) → informieren → wiederherstellen → nachbereiten
- [ ] **M** Kontaktliste: Betreiber, Hoster, CDN, Zahlungsanbieter, Rechtsberatung
- [ ] **M** Meldepflichten bei Datenpannen bekannt (EU: 72 Stunden an die Aufsichtsbehörde)
- [ ] **S** Nach jedem Vorfall: kurzer Bericht, Checkliste ergänzen

---

## 18. Eigener Onlineshop `SHOP`

### 18.1 Zahlungsdaten & PCI DSS
- [ ] **M** Kartendaten berühren nie das eigene System: nur Hosted Checkout (Redirect) oder eingebettete Felder des Zahlungsanbieters (iFrame)
- [ ] **M** Ziel PCI DSS v4.0.1 SAQ A; Fragebogen beim Zahlungsanbieter ausgefüllt
- [ ] **M** SAQ-A-Voraussetzung seit 31.03.2025: die Website ist **nicht anfällig für Skript-Angriffe** – nachgewiesen über die Vorgaben des Zahlungsanbieters (PCI-FAQ 1588) oder eigene Maßnahmen nach 6.4.3/11.6.1
- [ ] **M** Integration exakt nach Anleitung des Zahlungsanbieters, keine eigenen Kartenfelder
- [ ] **M** Strikte CSP, Skript-Inventar, keine unnötigen Drittskripte – auf der **ganzen** Website
- [ ] **S** Script-Monitoring auf Seiten mit Zahlungsformular

### 18.2 Preise, Warenkorb, Bestellung
- [ ] **M** Preise, Rabatte, Versand, Steuern **nur serverseitig** berechnet; Client sendet nur Produkt-ID + Menge
- [ ] **M** Mengen validiert (keine negativen, keine Dezimalzahlen, Maximum)
- [ ] **M** Gutscheine serverseitig geprüft, Einlöselimit atomar (Race Conditions), nicht erratbar, nicht beliebig kombinierbar
- [ ] **M** Lagerbestand atomar reserviert/abgebucht – kein Überverkauf durch parallele Bestellungen
- [ ] **M** Bestellung gilt **erst nach verifiziertem Webhook** als bezahlt, nie aufgrund der Erfolgs-URL
- [ ] **M** Bestell- und Rechnungsseiten nur für Besitzer (IDOR); Gastbestellungen über langen Zufallstoken
- [ ] **S** Bestellnummern nicht fortlaufend erratbar; Bestätigungen ohne vollständige Zahlungsdaten

### 18.3 Webhooks
- [ ] **M** Signatur jedes Webhooks mit dem **rohen** Request-Body verifiziert
- [ ] **M** Zeitstempel-Toleranz geprüft (Replay-Schutz)
- [ ] **M** Idempotenz: jede Event-ID nur einmal verarbeiten
- [ ] **M** Events können in falscher Reihenfolge kommen → aktuellen Zustand beim Anbieter nachladen
- [ ] **M** Webhook-Secret je Umgebung, nicht im Code
- [ ] **S** Webhook-Endpunkt von Bot-Schutz und Caching ausgenommen, aber rate-limited
- [ ] **S** Nur benötigte Event-Typen abonniert

### 18.4 Betrug & Missbrauch
- [ ] **M** 3-D Secure / SCA aktiv (im EWR Pflicht nach PSD2)
- [ ] **M** Schutz gegen Card Testing: Rate-Limit auf Zahlungsversuche, Captcha vor Checkout, Betrugserkennung des Anbieters (z. B. Stripe Radar) aktiv
- [ ] **S** Betrugsregeln geprüft (z. B. Block bei CVC-/PLZ-Fehlschlag)
- [ ] **S** Benachrichtigung bei Chargebacks; Erstattungen nur durch berechtigte Rollen, protokolliert
- [ ] **K** Limits für Bestellwert/-anzahl bei neuen Kunden

### 18.5 Konto beim Zahlungsanbieter
- [ ] **M** Konto gehört dem Betreiber; Dienstleister nur als Teammitglied mit passender Rolle
- [ ] **M** 2FA für alle; jede Person mit eigenem Login
- [ ] **M** Restricted Keys mit minimalen Rechten, **ein Key pro Integration**
- [ ] **M** Publishable Key im Client, Secret/Restricted Key nur serverseitig
- [ ] **M** Keys nie in Frontend-Code, CI-Logs, Chats oder KI-Prompts; bei Verdacht sofort rotieren
- [ ] **S** Ungenutzte Keys und Webhook-Endpunkte gelöscht
- [ ] **S** Benachrichtigung bei Änderung der Auszahlungsdaten (klassischer Übernahme-Angriff)
- [ ] **S** Zugänge von Dienstleistern nach Projektende entfernt

---

## 19. Abo-Zahlungen & Mitgliedschaften `ABO`

### 19.1 Abo-Status & Berechtigungen
- [ ] **M** Der Abo-Status gilt beim Zahlungsanbieter; die lokale Kopie wird **nur über verifizierte Webhooks** aktualisiert
- [ ] **M** Zugriff auf bezahlte Inhalte/Funktionen bei **jeder** Anfrage serverseitig geprüft
- [ ] **M** Alle Zustände behandelt (z. B. Stripe: `active`, `trialing`, `past_due`, `unpaid`, `canceled`, `incomplete`, `incomplete_expired`, `paused`)
- [ ] **M** Bezahlte Downloads über kurzlebige, signierte URLs

### 19.2 Planwechsel & Preise
- [ ] **M** Erlaubte Preise/Pläne serverseitig per Allowlist – der Client kann keinen beliebigen Preis übergeben
- [ ] **M** Up-/Downgrade nur über den Server oder ein Kundenportal mit konfigurierten erlaubten Wechseln
- [ ] **M** Anteilige Verrechnung und Gutscheine nicht zu Null-/Negativbeträgen missbrauchbar
- [ ] **S** Kundenportal des Zahlungsanbieters statt eigener Kartenverwaltung
- [ ] **S** Portal-Sitzungen nur für eingeloggte Besitzer, feste Return-URL

### 19.3 Testphasen & Missbrauch
- [ ] **S** Trial-Missbrauch begrenzt (ein Trial pro Kunde/Zahlungsmittel)
- [ ] **S** Abo-Gutscheine mit Laufzeit- und Einlösebegrenzung
- [ ] **K** Erkennung geteilter Accounts (gleichzeitige Sessions)

### 19.4 Fehlgeschlagene Zahlungen & Kündigung
- [ ] **M** Mahnlauf (automatische Wiederholung, Mails) aktiv; Zugriff nach Kulanzfrist automatisch beschränkt
- [ ] **M** Kündigung technisch so umgesetzt wie vereinbart (Periodenende oder sofort)
- [ ] **M** SEPA-Lastschrift: Mandat sauber erfasst, Rücklastschriften behandelt
- [ ] **S** Wiederholte Zahlungsfehler lösen Benachrichtigung an Kunde **und** Betreiber aus
- [ ] **S** Rechnungen nur für Besitzer abrufbar
- [ ] **S** Regelmäßiger Abgleich: aktive Abos ↔ freigeschaltete Leistungen

---

## 20. Shopify-Shop `SHOPIFY`

### 20.1 Konto & Zugänge
- [ ] **M** Shop gehört dem Betreiber; Agenturen/Freelancer über **Collaborator-Zugang** mit eingeschränkten Rechten
- [ ] **M** Collaborator-Anfragecode aktiviert
- [ ] **M** 2FA für **alle** Mitarbeiter- und Collaborator-Konten – bei Shopify Plus auf Organisationsebene erzwingen; **ohne Plus nicht erzwingbar** → bei Einrichtung aktivieren lassen und monatlich kontrollieren
- [ ] **M** Least Privilege; kein geteiltes Owner-Login; Berechtigung „Apps“ nur für wenige Personen
- [ ] **M** Ehemalige Mitarbeiter und Agenturen entfernt; Liste monatlich (mind. quartalsweise) geprüft
- [ ] **M** E-Mail-Postfach des Owner-Kontos mit 2FA (Übernahme des Postfachs = Übernahme des Shops)
- [ ] **K** Shopify Plus: SSO, kürzere Admin-Sessions, IP-Einschränkung

#### Notfall-Ablauf
- [ ] **S** Bei Verdacht: Passwörter von Shopify **und** zugehöriger E-Mail ändern, 2FA neu einrichten, Aktivitätsprotokoll prüfen, unbekannte Konten/Apps entfernen, App-Tokens rotieren, Shopify Support kontaktieren, Meldepflichten prüfen

### 20.2 Apps
- [ ] **M** Nur Apps mit guten Bewertungen und aktiver Pflege; angefragte Scopes vor Installation geprüft
- [ ] **M** Ungenutzte Apps deinstalliert, Reste im Theme entfernt
- [ ] **M** Custom Apps: Admin-API-Token nur serverseitig, minimale Scopes, bei Personalwechsel rotiert – diese Tokens umgehen 2FA und sind gefährlicher als das Admin-Passwort
- [ ] **S** App-Inventar mit Zweck, Scopes, Kosten, Verantwortlichem
- [ ] **S** Abo-Apps: Kundenportal, Webhooks und Berechtigungen wie in Abschnitt 19 geprüft

### 20.3 Theme & Code
- [ ] **M** Keine Secrets oder privaten Tokens in Liquid, Theme-JS oder öffentlichen Metafields
- [ ] **M** Fremdskripte inventarisiert; Tracking über Customer Events / Web Pixels statt direkt ins Theme
- [ ] **M** Checkout-Anpassungen nur über Checkout Extensibility – kein Legacy `checkout.liquid`
- [ ] **S** Theme versioniert (GitHub-Integration); Backup vor größeren Änderungen

### 20.4 Headless (Hydrogen oder eigenes Frontend)
- [ ] **M** Öffentlicher Storefront-Token mit minimalen Rechten; privater Storefront- und Admin-Token nur serverseitig
- [ ] **M** Shopify-Webhooks per HMAC (`X-Shopify-Hmac-Sha256`) mit rohem Body verifiziert
- [ ] **M** Kunden-Login über die Customer Account API (OAuth/PKCE)
- [ ] **S** Abschnitte 5, 6, 10, 11 gelten zusätzlich

### 20.5 Zahlungen & Betrug
- [ ] **M** Betrugsanalyse aktiv; risikoreiche Bestellungen nicht automatisch erfüllt
- [ ] **S** Shopify Flow: Hochrisiko-Bestellungen markieren oder stornieren
- [ ] **S** Bot-Schutz/Captcha für Konto- und Kontaktformulare
- [ ] **S** Geschenkkarten nur durch berechtigte Personen, Codes nicht erratbar
- [ ] **S** Benachrichtigung bei Änderung der Auszahlungsdaten

---

## 21. Web-App / SaaS – zusätzliche Tiefe `APP`

- [ ] **M** Bedrohungsmodell (1 Seite): Was ist wertvoll, wer greift an, wo sind Vertrauensgrenzen?
- [ ] **M** Rollen- und Rechtematrix dokumentiert und getestet
- [ ] **M** Private Dateien in nicht öffentlichem Storage; Ausgabe über signierte, kurzlebige URLs nach Prüfung
- [ ] **M** Hintergrundjobs und Cron-Endpunkte mit eigenem, minimal berechtigtem Zugang bzw. Secret geschützt
- [ ] **M** WebSockets/Realtime: Authentifizierung beim Verbindungsaufbau, Berechtigung pro Kanal
- [ ] **S** Audit-Log für sicherheitsrelevante Aktionen
- [ ] **S** Sensible Felder (z. B. Drittanbieter-Tokens) verschlüsselt in der DB
- [ ] **S** DB-User der App ohne DDL-Rechte; Migrationen mit separatem User
- [ ] **S** Feature-Flags und Admin-Schalter nicht über den Client manipulierbar
- [ ] **K** Externer Penetrationstest vor dem Launch
- [ ] **K** Responsible-Disclosure-Seite + `security.txt`

---

## 22. KI-Funktionen `KI`

- [ ] **M** API-Keys der KI-Anbieter nur serverseitig, Ausgabenlimit beim Anbieter gesetzt
- [ ] **M** Rate-Limit und Tageskontingent pro Nutzer/IP
- [ ] **M** Prompt Injection einkalkuliert: KI-Ausgaben nie ungeprüft als Code, SQL, HTML oder Befehl ausführen
- [ ] **M** KI hat nur Zugriff auf Daten, die der aktuelle Nutzer sehen darf
- [ ] **M** Aktionen mit Folgen (Mail senden, Daten ändern, Zahlungen) nur nach Bestätigung durch den Nutzer
- [ ] **S** Keine personenbezogenen Daten an Anbieter ohne passenden Vertrag und Einstellungen
- [ ] **S** Keine Secrets im System-Prompt (kann ausgelesen werden)
- [ ] **S** KI-Ausgaben im Frontend wie Nutzereingaben escaped

---

## 23. Übergabe & laufender Betrieb `BASIS`

### 23.1 Übergabe an den Betreiber
- [ ] **M** Betreiber ist Eigentümer von Domain, Hosting, Shop-Plattform, Zahlungsanbieter-Konto
- [ ] **M** Zugangsdaten nur über Passwortmanager oder einmalige verschlüsselte Links, nie per E-Mail im Klartext
- [ ] **M** Betreiber hat 2FA auf den eigenen Konten aktiviert
- [ ] **S** Kurze Sicherheitsanleitung (Phishing, Rollen im CMS, wer hat Zugriff)

### 23.2 Laufender Betrieb
| Aufgabe | Intervall |
|---|---|
| Security-Updates (Abhängigkeiten, CMS, Server) | [z. B. wöchentlich / sofort bei kritisch] |
| Backup-Prüfung / Restore-Test | [z. B. quartalsweise] |
| Access-Review | [z. B. quartalsweise] |
| Header-/TLS-/Schwachstellen-Scan | [z. B. monatlich] |
| Log- und Alarm-Auswertung | [z. B. wöchentlich] |
| Secret-Rotation | [z. B. jährlich] |
| Penetrationstest | [z. B. jährlich, bei sensiblen Daten] |

### 23.3 Projektende / Dienstleisterwechsel
- [ ] **M** Zugänge entfernt bzw. übertragen, Secrets rotiert
- [ ] **M** Daten übergeben und beim bisherigen Dienstleister gelöscht (inkl. Backups nach Frist)
- [ ] **M** Monitoring, Webhooks und Deploy-Keys abgebaut

---

## 24. Test-Tools

| Zweck | Beispiele |
|---|---|
| Security-Header | securityheaders.com, Mozilla HTTP Observatory |
| TLS/Zertifikat | SSL Labs (Ziel: A/A+) |
| Mail/DNS | internet.nl, MXToolbox, DMARC-Report-Auswertung |
| Dynamischer Scan | OWASP ZAP Baseline |
| Abhängigkeiten | npm/pnpm audit, Dependabot, Renovate |
| Container | Trivy |
| Secrets im Repo | gitleaks, trufflehog, GitHub Secret Scanning |
| Code-Analyse | Semgrep, CodeQL |
| CSP | CSP Evaluator |
| WordPress | WPScan |
| Offene Ports | `nmap` von extern gegen die Server-IP |

---

## 25. Go-live-Gate (GO / NO-GO)

- [ ] Alle **M**-Punkte der zutreffenden Tags erfüllt
- [ ] Alle **S**-Abweichungen begründet und dokumentiert
- [ ] Scans aus Abschnitt 24 ohne kritische oder hohe Findings
- [ ] Restore-Test erfolgreich
- [ ] Zugangsliste und Secrets-Inventar vollständig
- [ ] Notfallplan und Kontaktliste hinterlegt
- [ ] Ehrlicher Check: Würde ein automatischer Scanner oder ein neugieriger Nutzer mit Browser-DevTools in 15 Minuten etwas Offensichtliches finden?

**Ergebnis:** ☐ GO ☐ NO-GO  Datum: ______  Geprüft von: ______
