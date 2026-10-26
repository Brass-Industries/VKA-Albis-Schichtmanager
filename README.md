[README.md](https://github.com/user-attachments/files/33964502/README.md)
# VKA Dienstplan

Web-App zur Einsatzplanung für den Verband der Krankentransporte Albis (VKA). Ermöglicht das Erstellen und Verwalten von Anlässen, Personen, Posten und Schichtzuteilungen — live synchronisiert über alle Geräte.

---

## Stack

| Schicht | Technologie |
|---|---|
| Frontend | Vanilla HTML/CSS/JS (eine Datei, kein Build-Step) |
| Datenbank | [Supabase](https://supabase.com) (PostgreSQL + Realtime) |
| Hosting | Cloudflare Pages (automatisch via GitHub) |

---

## Features

- **Anlässe** erstellen, bearbeiten, löschen und duplizieren
- **Personen** mit Rang, Rolle und Funkrufname verwalten
- **Posten** mit Mindestrang und Mindestbesetzung definieren
- **Schichten** erstellen und Personen per Drag & Drop zuteilen
- **Echtzeit-Sync** — Änderungen erscheinen sofort auf allen Geräten
- **Funk-Modus** umschaltbar (Posten- oder Funkrufname-Ansicht)
- **Passwortschutz** (client-seitig, konfigurierbar)

---

## Setup

### 1. Supabase-Projekt erstellen

1. Account unter [supabase.com](https://supabase.com) erstellen
2. Neues Projekt anlegen (Region: West EU / Frankfurt)
3. Im **SQL Editor** das Datenbankschema ausführen (siehe unten)
4. Unter **Project Settings → API** folgende Werte kopieren:
   - **Project URL** → `SUPABASE_URL`
   - **anon public** Key → `SUPABASE_ANON_KEY`

### 2. Datenbankschema

Im Supabase SQL Editor ausführen:

```sql
create table anlaesse (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  datum date not null,
  funk_modus text default 'posten',
  created_at timestamptz default now()
);
create table schichten (
  id uuid primary key default gen_random_uuid(),
  anlass_id uuid references anlaesse(id),
  posten_name text,
  von text,
  bis text,
  personen jsonb default '[]',
  created_at timestamptz default now()
);
create table personen (
  id uuid primary key default gen_random_uuid(),
  anlass_id uuid references anlaesse(id),
  name text,
  rank text,
  rolle text,
  funkrufname text,
  color text,
  created_at timestamptz default now()
);
create table posten (
  id uuid primary key default gen_random_uuid(),
  anlass_id uuid references anlaesse(id),
  name text,
  beschreibung text,
  standort text,
  mindest_rang text,
  min_personen int default 0,
  created_at timestamptz default now()
);

-- Row Level Security
alter table anlaesse enable row level security;
alter table schichten enable row level security;
alter table personen  enable row level security;
alter table posten    enable row level security;

create policy "public" on anlaesse for all to anon using (true) with check (true);
create policy "public" on schichten for all to anon using (true) with check (true);
create policy "public" on personen  for all to anon using (true) with check (true);
create policy "public" on posten    for all to anon using (true) with check (true);

-- Realtime
alter publication supabase_realtime add table anlaesse;
alter publication supabase_realtime add table schichten;
alter publication supabase_realtime add table personen;
alter publication supabase_realtime add table posten;
```

### 3. Zugangsdaten in index.html eintragen

```javascript
const SUPABASE_URL      = "https://xxxx.supabase.co";
const SUPABASE_ANON_KEY = "eyJ...";
const APP_PASSWORT      = "vka-albis2026";
```

### 4. Auf GitHub pushen

Cloudflare Pages deployed die App automatisch bei jedem Push auf `main`.

---

## Deployment (Cloudflare Pages)

| Einstellung | Wert |
|---|---|
| Build command | *(leer lassen)* |
| Build output directory | `/` oder `.` |
| Root directory | *(leer lassen)* |

---

## Änderungen vornehmen

| Was | Wo |
|---|---|
| Aussehen, Logik, Texte | `index.html` bearbeiten → GitHub pushen |
| Passwort ändern | `APP_PASSWORT` in `index.html` |
| Neue Datenbankfelder | Supabase SQL Editor + `index.html` anpassen |

---

## Passwortschutz

Das Passwort wird client-seitig geprüft und als `base64`-String im `localStorage` des Browsers gespeichert (`vka_dp_auth`). Es schützt vor zufälligem Zugriff, ist aber kein Ersatz für echte Authentifizierung.
