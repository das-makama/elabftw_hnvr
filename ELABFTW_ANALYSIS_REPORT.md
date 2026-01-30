# elabFTW Repository Analyse & UX-Evaluierung

**Datum:** 2026-01-30
**Analysiert von:** Claude (Opus 4.5)
**Repository:** https://github.com/elabftw/elabftw
**Version analysiert:** Commit e2f9c0a (Hypernext Branch)

---

## Inhaltsverzeichnis

1. [Executive Summary](#1-executive-summary)
2. [Repository-Übersicht](#2-repository-übersicht)
3. [Architektur-Analyse](#3-architektur-analyse)
4. [Code-Qualität Assessment](#4-code-qualität-assessment)
5. [UI/UX-Analyse](#5-uiux-analyse)
6. [Customization-Potenzial](#6-customization-potenzial)
7. [Verbesserungsoptionen (A-D)](#7-verbesserungsoptionen-bewertung)
8. [Markdown-Protokoll Feature](#8-markdown-protokoll-feature)
9. [Equipment Booking System](#9-equipment-booking-system)
10. [Empfehlung & Nächste Schritte](#10-empfehlung--nächste-schritte)

---

## 1. Executive Summary

### A) Ist-Zustand: Die 5 Hauptprobleme

| # | Problem | Schweregrad | Betroffene Datei |
|---|---------|-------------|------------------|
| 1 | **Information Overload** in der Listenansicht: 40+ interaktive Elemente | KRITISCH | `src/templates/show.html` |
| 2 | **Icon-Only Navigation** ohne Labels reduziert Entdeckbarkeit | HOCH | `src/templates/view-edit-toolbar.html` |
| 3 | **Komplexes Berechtigungssystem** mit 4 separaten Read/Write-Paaren | HOCH | `src/templates/edit.html` |
| 4 | **Überwältigende Bulk-Operationen** mit 15+ plötzlich erscheinenden Controls | HOCH | `src/templates/show.html` |
| 5 | **Tiefe Seitenverschachtelung** mit 12+ Sections in der Edit-Ansicht | MITTEL | `src/templates/edit.html` |

### Warum ist die UX schlecht? Konkrete Code-Beispiele:

**1. Filter-Explosion (show.html, Zeilen 52-239):**
```html
<!-- 4 immer sichtbare Filter -->
<select name='category'>Filter category</select>
<select name='status'>Filter status</select>
<select name='owner'>Filter owner</select>
<input id='tagFilter' multiple name='tags[]' />

<!-- PLUS 9 versteckte Filter durch Toggle -->
<!-- PLUS 16 Sort-Optionen in einem Dropdown -->
<!-- PLUS Bulk-Operations mit 15 weiteren Controls -->
```

**2. Icon-Only Toolbar (view-edit-toolbar.html, Zeilen 154-323):**
```html
<!-- 15+ Icon-Buttons ohne Text-Labels -->
<button><i class='fas fa-copy'></i></button>      <!-- Duplizieren? -->
<button><i class='fas fa-calendar-check'></i></button>  <!-- Buchung oder Zeitstempel? -->
<button><i class='fas fa-link'></i></button>      <!-- Share oder Link? -->
```

### B) Empfohlener Weg: Quick Win Strategie

| Zeitrahmen | Ansatz | ROI |
|------------|--------|-----|
| **Quick Win** (< 1 Woche) | CSS-Overlay via Browser-Extension | HOCH |
| **Medium-term** (1-2 Monate) | Custom Landing Page mit Vue/React | MITTEL-HOCH |
| **Long-term** | API-basiertes Custom Frontend | NIEDRIG (zu aufwendig) |

**Pragmatischste Empfehlung:** Option A (CSS-Overlay) + Option B (Custom Landing Page)

### C) Nächste Schritte

1. **Sofort (Tag 1-2):** CSS-Overlay erstellen für:
   - Verstecken unnötiger Filter in `show.html`
   - Icon-Buttons mit Text-Labels versehen
   - Sidebar-Panels standardmäßig ausblenden

2. **Kurzfristig (Woche 1-2):** Custom Landing Page:
   - 3-Kachel-Layout (Geräte, Material, Protokolle)
   - Quick-Links zu häufigen Aktionen
   - Integration via Browser-Extension oder Reverse Proxy

3. **Mittelfristig (Monat 1-2):** Simplified Booking Widget:
   - Standalone React-App für Geräte-Buchung
   - Nutzt `/api/v2/events` Endpoints
   - Kann als iframe oder eigene Seite deployed werden

---

## 2. Repository-Übersicht

### Verzeichnisstruktur

```
elabftw/
├── src/                    # Haupt-Quellcode (PHP + TS + SCSS)
│   ├── Auth/              # Authentifizierung (SAML, LDAP, Local, Cookie)
│   ├── classes/           # Core-Klassen (App, Db, Auth, EntitySqlBuilder)
│   ├── controllers/       # MVC-Controller (20+ Klassen)
│   ├── models/            # Domain-Modelle (48 Dateien)
│   ├── services/          # Business Logic Services (42+ Dateien)
│   ├── templates/         # Twig-Templates (93 HTML-Dateien)
│   ├── ts/                # TypeScript Frontend (50+ Module)
│   ├── scss/              # Stylesheets (SCSS mit Bootstrap)
│   ├── Make/              # Export-Generatoren (PDF, ZIP, ELN)
│   ├── sql/               # Datenbankschema (181 Migrationen)
│   └── langs/             # Übersetzungen (21 Sprachen)
├── web/                   # Web Entry Points
│   ├── *.php              # Seiten-Endpoints (experiments.php, etc.)
│   ├── api/v2/            # REST API
│   └── assets/            # Kompilierte Assets (JS, CSS)
├── tests/                 # Test-Suite
│   ├── unit/              # 128 PHP Unit Tests
│   ├── apiv2/             # 4 API Integration Tests
│   └── cypress/           # 24 E2E Tests
├── apidoc/                # API-Dokumentation (OpenAPI 3.0)
└── composer.json          # PHP Dependencies
```

### Wichtigste Einstiegspunkte

| Datei | Zweck |
|-------|-------|
| `web/index.php` | Haupteingang (SAML ACS + Dashboard Redirect) |
| `web/experiments.php` | Experimente-Listenansicht |
| `web/database.php` | Ressourcen/Inventar-Listenansicht |
| `web/scheduler.php` | Geräte-Buchungskalender |
| `web/app/init.inc.php` | Bootstrap-Skript (Auth, Session, Config) |
| `src/controllers/Apiv2Controller.php` | REST API Handler |

---

## 3. Architektur-Analyse

### Backend-Architektur

```
┌─────────────────────────────────────────────────────────────────────┐
│                         ELABFTW ARCHITEKTUR                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────────┐    ┌──────────────────┐                      │
│  │   HTTP Request   │───▶│   web/*.php      │                      │
│  └──────────────────┘    │   (Entry Points) │                      │
│                          └────────┬─────────┘                      │
│                                   │                                 │
│                          ┌────────▼─────────┐                      │
│                          │  init.inc.php    │                      │
│                          │  ├─ Autoloader   │                      │
│                          │  ├─ Session      │                      │
│                          │  ├─ Config       │                      │
│                          │  ├─ CSRF Check   │                      │
│                          │  └─ Auth::try()  │                      │
│                          └────────┬─────────┘                      │
│                                   │                                 │
│         ┌─────────────────────────┼─────────────────────────┐      │
│         │                         │                         │      │
│  ┌──────▼──────┐          ┌───────▼───────┐         ┌───────▼────┐ │
│  │ Controllers │          │    Models     │         │  Services  │ │
│  │             │          │               │         │            │ │
│  │ Experiments │◀────────▶│ Experiments   │◀───────▶│ Filter     │ │
│  │ Database    │          │ Items         │         │ Email      │ │
│  │ Apiv2       │          │ Users         │         │ Transform  │ │
│  │ Scheduler   │          │ Scheduler     │         │ Search     │ │
│  └──────┬──────┘          └───────┬───────┘         └────────────┘ │
│         │                         │                                 │
│         │                  ┌──────▼──────┐                         │
│         │                  │     Db      │                         │
│         │                  │  (Singleton)│                         │
│         │                  └──────┬──────┘                         │
│         │                         │                                 │
│  ┌──────▼──────┐                  │                                │
│  │    Twig     │           ┌──────▼──────┐                         │
│  │  Templates  │           │   MySQL     │                         │
│  │  (93 files) │           │  Database   │                         │
│  └──────┬──────┘           └─────────────┘                         │
│         │                                                           │
│  ┌──────▼──────┐                                                   │
│  │   Response  │                                                   │
│  │   (HTML)    │                                                   │
│  └─────────────┘                                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Pattern:** Custom MVC-ähnlich mit File-per-Route

- **Kein Framework:** Selektive Symfony-Komponenten (http-foundation, console)
- **Template Engine:** Twig 3.x
- **Database:** MySQL/MariaDB mit PDO Singleton
- **Auth:** Multi-Strategy (Local, SAML, LDAP, Cookie, Anonymous)

### Frontend-Architektur

| Komponente | Technologie | Dateien |
|------------|-------------|---------|
| Templates | Twig 2 | 93 HTML-Dateien |
| JavaScript | TypeScript → ES5 | 50+ Module |
| CSS | SCSS → CSS | Bootstrap 4.6 + Custom |
| Bundler | Webpack 5 | `builder.js` |
| Editor | TinyMCE 6 + Bootstrap-Markdown | WYSIWYG + MD |
| Calendar | FullCalendar 5 | Scheduler |
| Grid | ag-Grid | Tabellen |

### Routing

**File-per-Route Pattern:**
```
/experiments.php?mode=view&id=123
       │              │      │
       │              │      └── Query Parameter
       │              └── Mode (view|edit|show)
       └── Direkte Datei-Zuordnung (kein Router)
```

**API Routing:**
```
/api/v2/experiments/123/uploads
   │  │      │       │     │
   │  │      │       │     └── Sub-Resource
   │  │      │       └── Resource ID
   │  │      └── Resource Type
   │  └── API Version
   └── API Prefix
```

### REST API Übersicht

| Endpoint | Methoden | Beschreibung |
|----------|----------|--------------|
| `/api/v2/experiments` | GET, POST | Experimente CRUD |
| `/api/v2/experiments/{id}` | GET, PATCH, DELETE | Einzelnes Experiment |
| `/api/v2/experiments/{id}/uploads` | GET, POST | Datei-Anhänge |
| `/api/v2/experiments/{id}/steps` | GET, POST | Prozess-Schritte |
| `/api/v2/experiments/{id}/comments` | GET, POST | Kommentare |
| `/api/v2/items` | GET, POST | Ressourcen/Inventar |
| `/api/v2/events` | GET | Buchungs-Events |
| `/api/v2/events/{itemId}` | POST | Buchung erstellen |
| `/api/v2/event/{id}` | GET, PATCH, DELETE | Buchung bearbeiten |
| `/api/v2/users` | GET, POST | Benutzer-Management |
| `/api/v2/teams` | GET, POST | Team-Management |

**API-Dokumentation:** `/apidoc/v2/openapi.yaml` (5,075 Zeilen, OpenAPI 3.0)

---

## 4. Code-Qualität Assessment

### Bewertung (1-10 Skala)

| Aspekt | Bewertung | Begründung |
|--------|-----------|------------|
| **Modularität** | 7/10 | Gute Trennung von Controllers/Models/Services, aber keine Plugin-Architektur |
| **Code-Komplexität** | 6/10 | Einige lange Dateien (Apiv2Controller: 18KB), aber klare Struktur |
| **Dokumentation** | 8/10 | Hervorragende API-Docs, inline-Kommentare gut, README qualitativ |
| **Test-Coverage** | 7/10 | 160 Test-Dateien, PHPStan Level 7, Psalm Level 3 |
| **Dependencies** | 8/10 | Gepflegte Pakete, Symfony-Komponenten aktuell |

### Test-Infrastruktur

```
Tests (160 Dateien)
├── Unit Tests (Codeception)      128 Dateien
│   ├── models/                    44 Tests
│   ├── services/                  36 Tests
│   ├── Make/                      13 Tests
│   └── classes/                   12 Tests
├── API Integration Tests           4 Dateien
└── E2E Tests (Cypress)            24 Specs
```

**Code Quality Tools:**
- PHPStan Level 7 (strikt)
- Psalm Level 3
- PHP-CS-Fixer (PER-CS2.0)
- ESLint + TypeScript
- StyleLint für SCSS

### Dependencies

**PHP (composer.json):** 45+ Pakete
- Symfony: http-foundation, console, mailer, yaml
- Security: HTMLPurifier, JWT
- Export: mPDF, PhpSpreadsheet
- Storage: AWS SDK, League Flysystem
- Auth: OneLogin SAML, LdapRecord

**JavaScript (package.json):** 80+ Pakete
- Framework: jQuery 3.7, Bootstrap 4.6
- Editor: TinyMCE 6.8, Marked 15
- Calendar: FullCalendar 6.1
- Grid: ag-Grid 32
- i18n: i18next 25

---

## 5. UI/UX-Analyse

### Navigation-Struktur

```
┌────────────────────────────────────────────────────────────────────┐
│ Logo  Dashboard  Experiments▼  Resources▼  Scheduler  Team  Tools▼│
├────────────────────────────────────────────────────────────────────┤
│                         Experiments Dropdown:                      │
│                         ├─ All experiments                        │
│                         ├─ My experiments                         │
│                         ├─ Templates                              │
│                         ├─ [5+ Kategorie-Links]                   │
│                         ├─ Experiments categories                 │
│                         └─ Experiments status                     │
└────────────────────────────────────────────────────────────────────┘
```

### Die 5 größten UX-Probleme (mit Code-Referenzen)

#### 1. Information Overload in Listenansicht
**Datei:** `src/templates/show.html` (Zeilen 52-428)

```html
<!-- PROBLEM: 40+ interaktive Elemente auf einer Seite -->

<!-- 4 immer sichtbare Filter -->
<select name='category'>...</select>
<select name='status'>...</select>
<select name='owner'>...</select>
<input id='tagFilter' multiple />

<!-- 9 versteckte Filter (Toggle) -->
<select name='metadataField'>...</select>
<input name='metadataValue' />
<select name='visibility'>...</select>
<!-- ... 6 weitere ... -->

<!-- 16 Sort-Optionen -->
<select id='orderSelect'>
  <option value='date'>newest</option>
  <option value='dateold'>oldest</option>
  <!-- ... 14 weitere ... -->
</select>

<!-- Bulk Operations (15 Controls) -->
<div id='withSelected'>
  <select data-action='export-selected-entities'>6 Optionen</select>
  <button>Lock</button>
  <button>Unlock</button>
  <button>Timestamp</button>
  <button>Archive</button>
  <button>Delete</button>
  <select name='category'>Change category</select>
  <select name='status'>Change status</select>
  <!-- ... 7 weitere ... -->
</div>
```

#### 2. Icon-Only Toolbar
**Datei:** `src/templates/view-edit-toolbar.html` (Zeilen 154-323)

```html
<!-- PROBLEM: 15+ Icons ohne Text-Labels -->
<button class='btn'><i class='fas fa-arrow-left'></i></button>
<button class='btn'><i class='fas fa-copy'></i></button>
<button class='btn'><i class='fas fa-calendar-check'></i></button>
<button class='btn'><i class='fas fa-link'></i></button>
<button class='btn'><i class='fas fa-pen-fancy'></i></button>
<!-- Benutzer muss raten was jedes Icon bedeutet -->
```

#### 3. Komplexes Berechtigungssystem
**Datei:** `src/templates/edit.html` (Zeilen 131-169)

```html
<!-- PROBLEM: 4 separate Read/Write-Paare, verwirrend angeordnet -->
<h3>Permissions</h3>
<!-- 1. CANREAD -->
<div>{% include('view-permissions.html') %}</div>
<!-- 2. CANWRITE -->
<div>{% include('view-permissions.html') %}</div>
<!-- 3. CANREAD_TARGET (nur für Templates) -->
<div>{% include('view-permissions.html') %}</div>
<input type='checkbox' data-target='canread_is_immutable' />
<!-- 4. CANWRITE_TARGET (nur für Templates) -->
<div>{% include('view-permissions.html') %}</div>
<input type='checkbox' data-target='canwrite_is_immutable' />
```

#### 4. Überwältigende Bulk-Operationen
**Datei:** `src/templates/show.html` (Zeilen 342-428)

- Section erscheint plötzlich wenn Items selektiert werden
- 15+ neue Controls ohne Übergang
- Gemischte Funktionen (Export + Edit + Delete)
- Horizontal Scrolling nötig auf Mobile

#### 5. Tiefe Edit-Page Struktur
**Datei:** `src/templates/edit.html` (223 Zeilen)

12+ collapsible Sections, alle standardmäßig geöffnet:
1. Title/Custom ID
2. Tags
3. Main Text (50+ Zeilen)
4. Extra Fields
5. Uploads
6. Steps
7. Links
8. Storage
9. Compounds
10. Permissions (40+ Zeilen)
11. Doodle/Draw
12. JSON Editor

### Template-Dateien Übersicht

| Kategorie | Anzahl | Pfad |
|-----------|--------|------|
| Base Templates | 5 | `src/templates/base.html`, `head.html`, `footer.html` |
| View Templates | 15 | `view.html`, `edit.html`, `show.html`, etc. |
| Modal Templates | 20+ | `*-modal.html` |
| Component Templates | 40+ | Partials für wiederverwendbare Elemente |
| Admin Templates | 10+ | `admin.html`, `sysconfig.html`, `team.html` |

### CSS-Struktur

**Hauptdatei:** `src/scss/main.scss` (29KB)

```scss
// Variablen
$elabblue: #29aeb9;  // Brand-Farbe
$dangerred: #e6614c;
$green: #54aa08;

// Imports
@import 'variables';
@import 'bootstrap';  // Bootstrap 4.6
@import 'fontawesome';
@import 'animations';
@import 'header';
@import 'tom-select';
// ...
```

**Pattern:** BEM-ähnlich mit Bootstrap-Basis

---

## 6. Customization-Potenzial

### Zusammenfassung

| Fähigkeit | Status | Details |
|-----------|--------|---------|
| Custom CSS | ❌ Nicht möglich | Kein Injection-Mechanismus, erfordert Fork |
| Theme System | ❌ Nicht vorhanden | Keine Theme-Engine |
| Plugin System | ❌ Nicht vorhanden | Keine Plugin-Architektur |
| Hook System | ❌ Begrenzt | Nur Audit-Events (read-only) |
| Custom Fields | ✅ Gut | JSON-Metadata für Extra-Felder |
| Admin Config | ✅ Gut | 94 Konfigurations-Optionen |
| Team Settings | ✅ Gut | Templates, Announcements, Onboarding |
| REST API | ✅ Sehr gut | Vollständige API v2 mit OpenAPI Docs |

### Konfigurations-Optionen (94 Einstellungen)

**Branding & Kommunikation:**
- `announcement` - Instanz-weite Ankündigung
- `login_announcement` - Login-Seiten-Nachricht
- `support_url` - Custom Support-Link
- `chat_url` - Chat-Raum-Link

**Team-Ebene:**
- `common_template` / `common_template_md` - Default-Templates
- `onboarding_email_*` - Willkommens-Emails
- `newcomer_banner` - Nachricht für neue Mitglieder

**Policies:**
- `privacy_policy` - Datenschutz-Text
- `terms_of_service` - Nutzungsbedingungen
- `legal_notice` - Impressum

### Was NICHT anpassbar ist (ohne Fork)

- Logo/Branding (hardcoded SVG)
- Farbschema (SCSS-Variablen)
- Navigation-Struktur
- Template-Layout
- Button-Anordnung
- Filter-Optionen

---

## 7. Verbesserungsoptionen Bewertung

### Option A: CSS-Overlay (kein Fork)

**Beschreibung:** Custom Stylesheet über Browser-Extension laden

**Machbarkeit:** ✅ EASY

**Implementierung:**
```css
/* Beispiel: Verstecke unnötige Filter */
#filtersDiv { display: none !important; }

/* Icon-Buttons mit Tooltip versehen */
.btn[data-action="duplicate"]::after {
  content: "Duplizieren";
  position: absolute;
}

/* Sidebar standardmäßig ausblenden */
#sidePanelContainer { display: none; }

/* Vereinfachte Bulk-Operations */
#withSelected .form-inline:nth-child(2) { display: none; }
```

**Pro:**
- Keine Server-Änderungen nötig
- Sofort deploybarer via Browser-Extension (Stylus/UserStyles)
- Rückgängig machbar

**Contra:**
- Kann durch Updates brechen
- Nur visuelle Änderungen, keine Funktionalität
- Nutzer müssen Extension installieren

**Geschätzter Aufwand:** 8-16 Stunden

---

### Option B: Custom Landing Page

**Beschreibung:** Neue Startseite mit 3 Haupt-Bereichen

**Machbarkeit:** ✅ MEDIUM

**Architektur:**
```
┌─────────────────────────────────────────────────────────┐
│                   Custom Landing Page                    │
│                  (React/Vue SPA)                        │
├──────────────────┬──────────────────┬──────────────────┤
│                  │                  │                  │
│   🔬 Protokolle  │   📦 Material    │   🔧 Geräte      │
│                  │                  │                  │
│   [Neu erstellen]│   [Suchen]       │   [Buchen]       │
│   [Letzte 5]     │   [Kategorien]   │   [Kalender]     │
│   [Vorlagen]     │   [Bestand]      │   [Verfügbar]    │
│                  │                  │                  │
└──────────────────┴──────────────────┴──────────────────┘
                           │
                           ▼
                    elabFTW API v2
```

**Implementierung:**
```javascript
// React/Vue Landing Page nutzt elabFTW API
const Dashboard = () => {
  const [protocols, setProtocols] = useState([]);
  const [equipment, setEquipment] = useState([]);

  useEffect(() => {
    fetch('/api/v2/experiments?limit=5&order=lastchange')
      .then(r => r.json())
      .then(setProtocols);

    fetch('/api/v2/items?cat=equipment')
      .then(r => r.json())
      .then(setEquipment);
  }, []);

  return (
    <div className="dashboard">
      <ProtocolsCard items={protocols} />
      <MaterialCard />
      <EquipmentCard items={equipment} />
    </div>
  );
};
```

**Deployment-Optionen:**
1. Reverse Proxy (nginx) mit Custom Route
2. Browser Extension mit Page Override
3. Iframe auf separater Domain

**Pro:**
- Vollständige UI-Kontrolle
- Nutzt vorhandene API
- Kann schrittweise erweitert werden

**Contra:**
- Erfordert separate Deployment-Infrastruktur
- Auth-Token-Handling nötig
- Doppelte Maintenance

**Geschätzter Aufwand:** 40-80 Stunden

---

### Option C: Frontend komplett ersetzen

**Beschreibung:** Neues React/Vue-Frontend, nur API nutzen

**Machbarkeit:** ⚠️ HARD

**API-Vollständigkeit:**

| Feature | API Support | Endpoint |
|---------|-------------|----------|
| Experimente CRUD | ✅ Vollständig | `/api/v2/experiments` |
| Items CRUD | ✅ Vollständig | `/api/v2/items` |
| File Uploads | ✅ Vollständig | `/{entity}/{id}/uploads` |
| Buchungen | ✅ Vollständig | `/api/v2/events` |
| User Management | ✅ Vollständig | `/api/v2/users` |
| Team Settings | ⚠️ Teilweise | `/api/v2/teams` |
| Sysadmin | ⚠️ Begrenzt | `/api/v2/config` |

**Fehlende Endpoints:**
- Dashboard-Statistiken (müssen berechnet werden)
- Notifications (nur lesend)
- Audit-Logs (read-only)
- Einige Admin-Funktionen

**Pro:**
- Vollständige UI-Freiheit
- Moderne Stack-Wahl möglich
- Bessere Mobile-Experience

**Contra:**
- Enormer Aufwand (6+ Monate)
- API-Lücken müssen gefüllt werden
- Maintenance-Last verdoppelt
- Feature-Parity schwer erreichbar

**Geschätzter Aufwand:** 500-1000+ Stunden

---

### Option D: Minimal-Fork mit UI-Patches

**Beschreibung:** Nur Template-Dateien ändern

**Machbarkeit:** ✅ MEDIUM

**Zu ändernde Dateien:**
```
src/templates/
├── show.html           # Filter vereinfachen
├── edit.html           # Sections reorganisieren
├── view-edit-toolbar.html  # Labels hinzufügen
├── head.html           # Navigation vereinfachen
└── dashboard.html      # Neues Layout
```

**Merge-Schwierigkeit bei Updates:**

| Datei | Update-Häufigkeit | Konflikt-Risiko |
|-------|-------------------|-----------------|
| `show.html` | Mittel (1-2x/Release) | HOCH |
| `edit.html` | Mittel | HOCH |
| `view-edit-toolbar.html` | Niedrig | MITTEL |
| `head.html` | Niedrig | MITTEL |
| `dashboard.html` | Niedrig | NIEDRIG |

**Pro:**
- Vollständige Kontrolle über UI
- Kann upstream-Changes übernehmen (mit Aufwand)
- Server-seitige Änderungen möglich

**Contra:**
- Merge-Konflikte bei jedem Update
- Erfordert PHP/Twig-Kenntnisse
- Maintenance-Overhead

**Geschätzter Aufwand:** 40-60 Stunden initial, 4-8 Stunden pro Update

---

### ROI-Vergleich

| Option | Aufwand | Nutzen | ROI | Empfehlung |
|--------|---------|--------|-----|------------|
| A: CSS-Overlay | 16h | Mittel | ⭐⭐⭐⭐ | ✅ EMPFOHLEN |
| B: Landing Page | 60h | Hoch | ⭐⭐⭐ | ✅ EMPFOHLEN |
| C: Neues Frontend | 800h | Sehr hoch | ⭐ | ❌ Zu aufwendig |
| D: Fork | 60h + Ongoing | Hoch | ⭐⭐ | ⚠️ Mit Vorsicht |

---

## 8. Markdown-Protokoll Feature

### Aktueller Support-Status

| Feature | Status | Details |
|---------|--------|---------|
| Markdown-Editor | ✅ Vorhanden | `bootstrap-markdown-fa5` + `marked.js` |
| Markdown-Speicherung | ✅ Vorhanden | `content_type = 2` in DB |
| Markdown-Vorschau | ✅ Vorhanden | Live-Preview im Editor |
| PDF-Export | ✅ Vorhanden | Via mPDF (MD → HTML → PDF) |
| Markdown-Export | ❌ Fehlt | Kein `.md` Export-Format |
| User-Präferenz | ✅ Vorhanden | `use_markdown` Einstellung |

### Technische Details

**Content-Type System:**
```php
// src/models/AbstractEntity.php
const int CONTENT_HTML = 1;
const int CONTENT_MD = 2;

// Speicherung in DB: body (MEDIUMTEXT) + content_type (TINYINT)
```

**Markdown-Konvertierung:**
```php
// src/classes/Tools.php
public static function md2html(string $md): string
{
    $converter = new GithubFlavoredMarkdownConverter([
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);
    return $converter->convert($md)->getContent();
}
```

**PDF-Export Pfad:**
```
Markdown → HTML (CommonMark) → mPDF → PDF
```

### Was fehlt für "echtes" Markdown

1. **Markdown-Export Format:**
   - Neue `MakeMarkdown.php` Klasse erstellen
   - `ExportFormat::Markdown` hinzufügen
   - Download als `.md` Datei

2. **Verbesserter Editor:**
   - Mehr Toolbar-Buttons (Tables, Code-Blocks)
   - Bessere Syntax-Highlighting
   - Markdown-Cheatsheet

3. **Direct PDF:**
   - Optional: Pandoc-Integration für native MD→PDF
   - Aktuell funktioniert aber HTML-Zwischenschritt gut

### Empfehlung

Der aktuelle Markdown-Support ist **ausreichend** (7/10). Für bessere Experience:

**Quick Win:**
- Markdown als User-Default setzen in Team-Settings
- Templates in Markdown erstellen

**Medium-term:**
- PR für Markdown-Export-Format upstream senden

---

## 9. Equipment Booking System

### Architektur

```
┌─────────────────────────────────────────────────────────────────┐
│                    BOOKING SYSTEM ARCHITEKTUR                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Frontend (src/ts/scheduler.ts)                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  FullCalendar 5.x                                        │   │
│  │  ├─ dayGrid (Monatsansicht)                              │   │
│  │  ├─ timeGrid (Wochen/Tagesansicht)                       │   │
│  │  ├─ timeline (Zeitleiste)                                │   │
│  │  └─ list (Listenansicht)                                 │   │
│  │                                                          │   │
│  │  Event Handlers:                                         │   │
│  │  ├─ select → Modal öffnen → Items wählen                │   │
│  │  ├─ eventClick → Edit Modal                             │   │
│  │  ├─ eventDrop → PATCH start/end                         │   │
│  │  └─ eventResize → PATCH end                             │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  API Layer                                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  GET  /api/v2/events       → List bookings               │   │
│  │  POST /api/v2/events/{id}  → Create booking              │   │
│  │  GET  /api/v2/event/{id}   → Get booking                 │   │
│  │  PATCH /api/v2/event/{id}  → Update booking              │   │
│  │  DELETE /api/v2/event/{id} → Cancel booking              │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  Backend (src/models/Scheduler.php, 624 Zeilen)                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Validierungen:                                          │   │
│  │  ├─ checkOverlap()      → Überlappungs-Check            │   │
│  │  ├─ checkSlotTime()     → Max-Dauer-Check               │   │
│  │  ├─ checkMaxSlots()     → Max-Buchungen pro User        │   │
│  │  ├─ isFutureOrExplode() → Vergangenheits-Check          │   │
│  │  └─ canWrite()          → Berechtigungs-Check           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  Database (team_events)                                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  id | team | item | start | end | title | userid |       │   │
│  │  experiment | item_link | created_at | modified_at       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### User Flow für Buchung

1. **Scheduler öffnen** → `/scheduler.php`
2. **Ressourcen filtern** → Scope, Kategorie, Items wählen
3. **Zeitslot klicken** → FullCalendar `select` Event
4. **Items bestätigen** → Modal mit Checkboxen
5. **"Create event(s)"** → POST `/api/v2/events/{itemId}`
6. **Kalender aktualisiert** → Event erscheint

### Vereinfachungs-Potenzial

**Aktuelles Problem:**
- Multi-Step Modal-Flow
- TomSelect für Item-Auswahl (komplex)
- Viele Filter-Optionen

**Simplified Widget möglich:**

```javascript
// Minimaler Booking-Widget (React)
const QuickBook = ({ itemId, itemName }) => {
  const [date, setDate] = useState(new Date());
  const [duration, setDuration] = useState(60); // Minuten

  const book = async () => {
    const start = date.toISOString();
    const end = new Date(date.getTime() + duration * 60000).toISOString();

    await fetch(`/api/v2/events/${itemId}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ start, end })
    });
  };

  return (
    <div className="quick-book">
      <h3>{itemName} buchen</h3>
      <input type="datetime-local" onChange={e => setDate(new Date(e.target.value))} />
      <select onChange={e => setDuration(e.target.value)}>
        <option value="30">30 Min</option>
        <option value="60">1 Stunde</option>
        <option value="120">2 Stunden</option>
      </select>
      <button onClick={book}>Jetzt buchen</button>
    </div>
  );
};
```

**Aufwand für Simplified Widget:** 20-30 Stunden

---

## 10. Empfehlung & Nächste Schritte

### Priorisierte Handlungsempfehlungen

#### Quick Wins (< 1 Woche)

| # | Aktion | Aufwand | Impact |
|---|--------|---------|--------|
| 1 | CSS-Overlay erstellen (Option A) | 16h | Sofortige visuelle Verbesserung |
| 2 | Team-Default auf Markdown setzen | 1h | Bessere Protokoll-Erstellung |
| 3 | Markdown-Templates erstellen | 4h | Standardisierte Protokolle |
| 4 | Browser-Extension für CSS deployen | 2h | Team-weiter Rollout |

#### Medium-term (1-2 Monate)

| # | Aktion | Aufwand | Impact |
|---|--------|---------|--------|
| 5 | Custom Landing Page (Option B) | 60h | Deutlich bessere UX |
| 6 | Simplified Booking Widget | 30h | Schnellere Geräte-Buchung |
| 7 | Quick-Create Protokoll-Button | 10h | Reduzierte Klicks |

#### Long-term (3+ Monate)

| # | Aktion | Aufwand | Impact |
|---|--------|---------|--------|
| 8 | Minimal-Fork evaluieren (Option D) | 60h initial | Volle Kontrolle |
| 9 | Upstream-Contributions | Variabel | Community-Nutzen |

### Benötigte Skills & Ressourcen

| Skill | Level | Für Optionen |
|-------|-------|--------------|
| CSS/SCSS | Mittel | A, B, D |
| JavaScript/TypeScript | Mittel | B |
| React oder Vue | Basis | B, Booking Widget |
| PHP/Twig | Fortgeschritten | D |
| Git/Merge-Konflikte | Fortgeschritten | D |

### Konkrete Action Items

**Woche 1:**
- [ ] CSS-Overlay Stylesheet erstellen
- [ ] Stylus/UserStyles Extension für Team bereitstellen
- [ ] Markdown als Team-Default konfigurieren

**Woche 2-3:**
- [ ] Custom Landing Page Prototyp (Vue/React)
- [ ] 3-Kachel-Layout implementieren
- [ ] API-Integration für häufige Aktionen

**Woche 4:**
- [ ] Simplified Booking Widget
- [ ] Integration in Landing Page
- [ ] Team-Feedback sammeln

**Monat 2:**
- [ ] Iterationen basierend auf Feedback
- [ ] Documentation für Custom Components
- [ ] Entscheidung über Fork ja/nein

### Finale Empfehlung

**Für euer Lab empfehle ich:**

1. **Sofort starten mit Option A** (CSS-Overlay)
   - Geringster Aufwand
   - Sofortige Wirkung
   - Risikolos

2. **Parallel Option B entwickeln** (Custom Landing Page)
   - Beste Balance aus Aufwand und Nutzen
   - Nutzt vorhandene API
   - Kann inkrementell wachsen

3. **Option C und D vermeiden**
   - Zu hoher Aufwand für begrenztes Budget
   - Maintenance-Last nicht tragbar

**Bottom Line:**
elabFTW ist ein solides, gut gepflegtes Open-Source-Projekt mit exzellenter API. Die UX-Probleme sind real, aber durch Overlay + Custom Landing Page lösbar ohne die Vorteile der aktiven Upstream-Entwicklung zu verlieren.

---

## Anhang: Wichtige Dateipfade

### Backend
- `web/app/init.inc.php` - Bootstrap
- `src/controllers/Apiv2Controller.php` - API Handler
- `src/models/Experiments.php` - Experiment-Logik
- `src/models/Scheduler.php` - Booking-Logik
- `src/classes/Tools.php` - Markdown-Konvertierung

### Frontend
- `src/templates/show.html` - Listenansicht (UX-Problem #1)
- `src/templates/edit.html` - Edit-Seite (UX-Problem #3, #5)
- `src/templates/view-edit-toolbar.html` - Toolbar (UX-Problem #2)
- `src/ts/scheduler.ts` - Booking-Kalender
- `src/scss/main.scss` - Haupt-Stylesheet

### API
- `apidoc/v2/openapi.yaml` - API-Dokumentation
- `apidoc/v2/README.md` - API-Anleitung

### Tests
- `tests/unit/` - PHP Unit Tests
- `tests/cypress/` - E2E Tests

---

*Report generiert am 2026-01-30*
