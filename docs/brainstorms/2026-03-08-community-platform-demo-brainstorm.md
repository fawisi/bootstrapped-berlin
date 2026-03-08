# Community Platform Demo — Brainstorm

**Date**: 2026-03-08
**Status**: Complete
**Scope**: Clickthrough-Demo (kein Backend, keine echte Auth)

---

## What We're Building

Eine klickbare Demo der bootstrapped, berlin Community-Plattform. Die bestehende Landing Page bleibt als Einstieg erhalten. Der CTA ("Join waitlist") führt in die App-Demo, die den kompletten User-Flow zeigt — von der Bewerbung bis zum anonymen Chat.

**Ziel**: Look & Feel der Plattform erlebbar machen. Kein funktionierendes Backend, nur Dummy-Daten und Screen-Transitions.

---

## User Flow

```
Landing Page (index.html)
    └─ CTA Button ──►

Onboarding (linear, einmalig):
  1. Bewerbung (apply.html) — 3-Step Wizard
     Step 1: Basics (Name, E-Mail)
     Step 2: Business (Produkt, Branche, Revenue-Range)
     Step 3: Motivation (Warum diese Community?, größtes Problem)
     → Success-State: "Bewerbung eingegangen" (inline, kein extra Screen)

  2. Willkommen / Akzeptiert (welcome.html)

  3. Profil-Setup (profile.html)
     - Vorname, Branche (Dropdown), Challenge-Tags (Multi-Select)
     → Weiter zu Discover

App (Bottom-Tab-Bar Navigation):
  4. Discover (discover.html) — Challenge-Liste, filterbar
  5. Challenge-Detail (challenge.html) — Profil + "Let's connect"
  6. Chat (chat.html) — Minimaler Chat mit Dummy-Verlauf
```

### Navigation innerhalb der App
- **Bottom-Tab-Bar** (Mobile-first): 3 Tabs — Discover, Chats, Profil
- **Top-Bar**: Logo + Theme/Lang Toggle
- Onboarding-Screens (1-3) haben keine Tab-Bar — nur linearen Flow mit "Weiter"-Button

---

## Key Decisions

### Architektur: Multi-Page HTML
- Separate HTML-Dateien pro Screen
- Shared CSS wird in eine externe `shared.css` ausgelagert
- Shared JS (Theme, i18n) wird in eine externe `shared.js` ausgelagert
- Kein Framework, kein Build-Step, kein Routing-Library
- Links zwischen Seiten (einfache `<a href>`)

### Matching: Challenge-basiert (nicht Profil-basiert)
- Man sieht nicht Profile, sondern **Challenges/Probleme** anderer Founder
- "Ich kämpfe mit X" — man klickt auf eine Challenge die einen anspricht
- Differenzierung: Es geht um gemeinsame Probleme, nicht um Networking-Smalltalk

### Privacy: Halbwegs offen
- **Sichtbar für andere**: Vorname, Branche, Challenge-Tags
- **Nicht sichtbar**: Nachname, Firma, E-Mail, Kontaktdaten
- **Kommunikation**: Nur über anonyme E-Mail-Relay (z.B. `bootstrapped+abc123@bootstrapped-berlin.de`)
- Kontaktdaten tauscht man erst persönlich beim Coffee Date

### Chat: Minimal
- Einfaches Textfeld + Senden-Button
- Im echten Produkt: Nachrichten werden als anonyme E-Mails weitergeleitet (JOIN/Greenhouse-Modell)
- Kein Kalender, kein Scheduling — Verabredungen regeln die Leute selbst im Chat
- Demo zeigt Dummy-Nachrichtenverlauf

### Bewerbung: Mehrstufiger Wizard
- 3 Steps mit Fortschrittsanzeige
- Step 1: Name, E-Mail
- Step 2: Produkt/Firma, Branche, Revenue-Range
- Step 3: Warum diese Community?, größtes aktuelles Problem

### Design: Gleiches System wie Landing Page
- Dark Mode, Copper Accent (#c45a3c), gleiche Fonts
- JetBrains Mono (Headlines), Lora (Serif), Inter (Body)
- Gleiche Theming-Logik (Dark/Light Toggle, i18n EN/DE)
- **Volle Effekte überall**: Cursor Glow, Grain, Parallax auch in App-Screens
- i18n (EN/DE) in allen Screens, nicht nur Landing Page

### Revenue-Ranges (Bewerbung Step 2)
- Pre-Revenue
- 0–10k EUR/mo
- 10–50k EUR/mo
- 50–100k EUR/mo
- 100k+ EUR/mo

### Challenge-Tags (vordefinierte Liste, EN / DE)
Tags werden je nach Spracheinstellung angezeigt:

| EN | DE |
|----|-----|
| First Revenue | Erster Umsatz |
| Customer Acquisition | Kundengewinnung |
| Pricing & Monetization | Pricing & Monetarisierung |
| Product-Market Fit | Product-Market Fit |
| First Hire / Team | Erste Einstellung / Team |
| Cashflow / Runway | Cashflow / Runway |
| Marketing & Branding | Marketing & Branding |
| Sales Process | Vertriebsprozess |
| Retention / Churn | Kundenbindung / Churn |
| Scaling Operations | Skalierung |
| Tech / Infrastructure | Tech / Infrastruktur |
| Legal & Compliance | Recht & Compliance |
| Work-Life Balance | Work-Life Balance |
| Bootstrapping vs. Funding | Bootstrapping vs. Funding |
| Go-to-Market | Go-to-Market |
| Partnerships | Partnerschaften |

---

## Dateistruktur (geplant)

```
bootstrapped_berlin/
├── index.html              # Landing Page (besteht bereits)
├── shared.css              # Extracted design system
├── shared.js               # Extracted theme/i18n logic
├── app/
│   ├── apply.html          # Mehrstufige Bewerbung (inkl. Bestätigung als Success-State)
│   ├── welcome.html        # Akzeptiert-Bestätigung
│   ├── profile.html        # Profil-Setup (Name, Branche, Tags)
│   ├── discover.html       # Challenge-basiertes Matching (Liste)
│   ├── challenge.html      # Challenge-Detail + "Let's connect"
│   └── chat.html           # Anonymer Chat
└── docs/
    └── brainstorms/
        └── 2026-03-08-community-platform-demo-brainstorm.md
```

---

## Dummy-Daten für die Demo

### Beispiel-Founder (5-6 Profile)
- "Mika" — SaaS, kämpft mit Churn
- "Lena" — E-Commerce, First Hire
- "Jonas" — Dev Tools, Pricing
- "Ayla" — HealthTech, Customer Acquisition
- "Tom" — Marketplace, Product-Market Fit
- "Priya" — FinTech, Legal & Compliance

### Beispiel-Chat (Dummy-Verlauf)
```
Du: Hey! Ich hab gesehen du kämpfst auch mit Pricing. Was genau ist deine Situation?
Jonas: Hey! Ja, ich hab ein Dev-Tool und weiß nicht ob Freemium oder direkt Paid. Du?
Du: Ähnlich. SaaS, B2B. Hab Angst zu viel zu verlangen. Wollen wir uns auf einen Kaffee treffen?
Jonas: Klar! Nächste Woche? Ich bin flexibel.
```

---

## Why This Approach

- **Multi-Page HTML**: Konsistent mit dem "kein Build-Step"-Prinzip. Jede Seite ist eigenständig und kann einzeln entwickelt/getestet werden. Kein Routing-Overhead.
- **Challenge-basiert statt Profil-basiert**: Differenziert von LinkedIn/generischen Netzwerken. Founder verbinden sich über gemeinsame Probleme — das ist der Kern der Community.
- **Anonymer E-Mail-Relay**: Schützt die Privatsphäre, senkt die Hürde sich zu melden, und ist technisch einfach (kein Real-Time-Chat nötig).
- **Clickthrough-Demo**: Schnell gebaut, gut für Feedback, kein Backend-Overhead. Wenn das Feeling stimmt, kann man ein echtes MVP darauf aufbauen.
