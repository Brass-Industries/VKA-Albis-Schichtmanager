# VKA Dienstplan

Web-App zur Einsatzplanung für den Verband der Krankentransporte Albis (VKA). Ermöglicht das Erstellen und Verwalten von Anlässen, Personen, Posten und Schichtzuteilungen — live synchronisiert über alle Geräte.

---

## Benutzungsanleitung

### Übersicht der Tabs

| Tab | Inhalt |
|---|---|
| 📅 Anlässe | Alle Einsätze und Veranstaltungen verwalten |
| Posten | Posten und Standorte für den gewählten Anlass |
| Personen | Helfer mit Rang und Rolle für den gewählten Anlass |
| Plan | Schichten erstellen und Personen zuteilen |
| Übersicht | Auswertung nach Person oder Posten |

---

### Schritt 1 — Anlass erstellen

1. Tab **Anlässe** öffnen
2. Auf **+ Neuer Anlass** klicken
3. Name, Datum und Funk-Modus eingeben
4. Speichern → die App wechselt automatisch zum Tab **Posten**

> Mit dem 📋-Symbol lässt sich ein bestehender Anlass duplizieren (inkl. Personen, Posten und Schichten).

---

### Schritt 2 — Posten erfassen

Posten sind die Stationen oder Aufgabenbereiche des Anlasses (z.B. «Sanitätsposten Haupteingang»).

1. Tab **Posten** → **+ Neuer Posten**
2. Felder ausfüllen:
   - **Name** — Bezeichnung des Postens
   - **Beschreibung** — optionale Zusatzinfos
   - **Standort** — Adresse (wird als Google-Maps-Link angezeigt)
   - **Mindestrang** — welcher Rang mindestens eingeteilt sein muss
   - **Mindestbesetzung** — wie viele Personen mindestens eingeteilt sein müssen
3. Speichern

---

### Schritt 3 — Personen erfassen

1. Tab **Personen** → **+ Neue Person**
2. Felder ausfüllen:
   - **Name**
   - **Rang** — z.B. RS, San, Gruppenführer
   - **Rolle** — z.B. Fahrer, Sanitäter
   - **Funkrufname** — wird im Funk-Modus statt des Postens angezeigt
   - **Farbe** — zur visuellen Unterscheidung im Plan
3. Speichern

---

### Schritt 4 — Plan erstellen

1. Tab **Plan** öffnen
2. **+ Neue Schicht** klicken
3. Posten, Von-Zeit und Bis-Zeit eingeben
4. Person zuteilen: auf **+ Person** in der Schicht klicken und aus der Liste wählen
5. Personen können per Klick auf das **×** wieder entfernt werden

> Alle Änderungen werden sofort auf allen Geräten angezeigt (Echtzeit-Sync).

---

### Schritt 5 — Übersicht

Der Tab **Übersicht** zeigt die gesamte Einteilung:

- **Nach Person** — wer ist wann wo eingeteilt
- **Nach Posten** — welcher Posten ist wann besetzt und ob die Mindestbesetzung erreicht ist

---

### Funk-Modus

Der Funk-Modus steuert, was im Plan angezeigt wird:

| Modus | Anzeige |
|---|---|
| Posten | Postenname (z.B. «Sanitätsposten Ost») |
| Funk | Funkrufname der eingeteilten Person |

Der Modus wird pro Anlass gesetzt und kann beim Erstellen oder Bearbeiten eines Anlasses geändert werden.

---

### Anlass bearbeiten und löschen

- **✏️** — Anlass umbenennen oder Datum/Modus ändern
- **📋** — Anlass duplizieren (Kopie mit neuen Daten erstellen)
- **🗑️** — Anlass löschen (löscht auch alle Personen, Posten und Schichten)

---

### Sync-Punkt (oben rechts)

Das kleine Symbol neben dem Datum zeigt den Verbindungsstatus:

| Farbe | Bedeutung |
|---|---|
| Grün | Verbunden, alle Daten aktuell |
| Drehend | Daten werden gespeichert |
| Grau | Keine Verbindung |

---

## Setup (technisch)

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

## Stack

| Schicht | Technologie |
|---|---|
| Frontend | Vanilla HTML/CSS/JS (eine Datei, kein Build-Step) |
| Datenbank | [Supabase](https://supabase.com) (PostgreSQL + Realtime) |
| Hosting | Cloudflare Pages (automatisch via GitHub) |

---

## Passwortschutz

Das Passwort wird client-seitig geprüft und als `base64`-String im `localStorage` des Browsers gespeichert (`vka_dp_auth`). Es schützt vor zufälligem Zugriff, ist aber kein Ersatz für echte Authentifizierung.
