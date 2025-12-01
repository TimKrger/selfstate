# Technischer Umsetzungsplan: SelfState

**Status:** Ready for Dev / Vibecoding
**Repository-Strategie:** Monorepo (Turborepo)
**Hosting:** Self-Hosted fähig (Docker) / Cloud-Agnostic

## 1. Architektur-Übersicht

Wir verfolgen einen **Hybrid-Ansatz**:

1.  **Central Truth (Payload CMS):** Hält Daten, User-Management und die Pattern-Library.
2.  **Privacy AI Layer:** Die KI-Logik läuft entweder lokal auf dem Device oder als Stateless Edge Function, die mit dem "Bring Your Own Key" (BYOK) des Users gefüttert wird. Keine Speicherung von API-Keys in der Datenbank im Klartext.

### Der Stack (Vibecoding-Optimiert)

Dieser Stack ist besonders gut geeignet, um von KI-Agenten (Cursor, Windsurf, Cline) geschrieben zu werden, da er auf stark typisierten Standards basiert.

- **Paket-Manager:** `pnpm` (schnell, effizient für Monorepos).
- **Orchestrierung:** `Turborepo`.
- **Backend / Headless CMS:** **Payload 3.0** (Next.js native).
  - _Warum:_ Open Source, TypeScript-first, läuft Serverless oder im Docker-Container.
  - _Datenbank:_ **PostgreSQL** (skalierbar, relational).
- **Frontend Web:** **Next.js 15** (App Router).
  - _UI:_ **shadcn/ui** + TailwindCSS (Industrie-Standard, KI kann das perfekt generieren).
- **Frontend Mobile:** **React Native (Expo)**.
  - _UI:_ **NativeWind** (Tailwind für Mobile) – erlaubt Copy-Paste von Styles zwischen Web und Mobile.
- **AI Integration:** **Vercel AI SDK** (Core).
  - _Warum:_ Abstrahiert die LLM-Provider. Egal ob OpenAI, Anthropic oder lokale Modelle.

---

## 2. Datenmodell (Schema Design)

Hier sind die exakten Collection-Namen für Payload, angepasst an die "Quandes-Terminologie".

### A. Core Collections (aktualisiert)

1.  **`users`**  
    - Felder: `name`, `roles` (admin, user), `impactProfile` (Relation zu `identities`), `teamId` (optional).
2.  **`identities`**  
    - Global: `SELF`, `CREATE`, `CONNECT`.
    - Felder: `name`, `description`, `slug`, `icon`.
3.  **`micro_scripts`**  
    - Felder: `title`, `trigger`, `action`, `rewardHint`, `identity`, `owner`, `level` (L1–L5), `isActive`, `sourcePattern`, `contextTags`, `availabilityWindows`, `visibility` (`private` | `team` | `public`), `teamId` (optional).
    - `taxonomyIds` (denormalisiert, vom Pattern oder user-spezifisch abgeleitet).
    - Optional `intendedOutcome`, `metricHint`.
4.  **`state_syncs`**  
    - `microScript`: Relation zu `micro_scripts`.
    - `timestamp`: DateTime.
    - `status`: (`flow`, `friction`, `skipped`).
    - `flowRating` (1–5), `effortRating` (optional), `moodDelta` (optional).
    - `frictionTags`: Array aus leichter Friction-Taxonomie.
    - `frictionNote`, `aiAdjustment` (Text/JSON).

### B. Community Collections & Taxonomien

5.  **`pattern_library`**  
    - Felder: `title`, `trigger`, `action`, `rewardHint`, `identity`, `category`, `levels` (L1–L5), `clones`, `contributorName`, `featured` (optional), `complexity` (S/M/L), `prerequisites`, `intendedOutcome`, `metricHint`, `visibility` (`public` | `team` | `private`), `teamId` (optional).
    - Denormalisiert: `taxonomyIds`, `taxonomySlugs`, `identitySlug`, `featuredTaxonomyIds`.
6.  **`taxonomy_groups`**  
    - Filter-Kategorien: z. B. Kontext, Dauer, Energie, Identität, Modus, Zielgruppe, Ergebnis.
    - Felder: `name`, `slug`, `order`, `description`, `externalId` (für Import).
7.  **`taxonomies`**  
    - Filter-Werte, Relation zu `taxonomy_groups`.
    - Felder: `name`, `order`, `description`, `externalId`, `isFeatured`.
8.  **`pattern_taxonomies`** (Join)  
    - n:m zwischen `pattern_library` und `taxonomies`.
    - Optional denormalisiert als `taxonomyIds` direkt am Pattern.
9.  **`saved_patterns`** (User↔Pattern)  
    - Felder: `userId`, `patternId`, `pinned`, `rating`, `usageCount`, `lastUsedAt`.
10. **`micro_script_overrides`** (User-spezifische Level/Context)  
    - Felder: `microScriptId`, `userId`, `levelOverrides` (JSON für L1–L5), `contextTags` (userbezogen).

---

## 3. Die "Agentic" Logik (KI-Implementierung)

Wir nutzen das **Vercel AI SDK**, um die Logik des "State Sync" zu bauen.

### System Prompt (Persona: The Impact Coach)

Dieser Prompt wird bei jedem API-Call mitgegeben:

> "Du bist der SelfState Impact Coach. Dein Ziel ist nicht Disziplin, sondern Flow und Identität. Du analysierst die Daten des Nutzers basierend auf den Prinzipien von Quandes Projects (Self, Create, Connect). Wenn ein Nutzer niedrigen Flow meldet, schlägst du IMMER eine Vereinfachung vor (Step-down), niemals mehr Anstrengung. Du sprichst kurz, empathisch und handlungsorientiert."

### Der BYOK-Flow (Privacy First)

1.  User geht in Einstellungen -> "AI Configuration".
2.  User gibt OpenAI/Anthropic Key ein.
3.  **Web:** Key wird im `localStorage` (verschlüsselt) gespeichert.
4.  **Mobile:** Key wird im `SecureStore` gespeichert.
5.  Beim Request (z.B. "Analysiere meine Woche") wird der Key im Header an die API-Route (Edge Function) gesendet. Der Server speichert den Key **nicht**, sondern nutzt ihn nur für den einen Call.

---

## 4. Entwicklungsphasen (Roadmap für Vibecoding)

Da wir im Social Impact Camp arbeiten, sind die Phasen so geschnitten, dass man in den montäglichen Sessions Ergebnisse sieht.

### Phase 1: Das Fundament (Setup & Daten)

- **Ziel:** Payload läuft lokal, Datenbank steht, man kann User und Skripte manuell anlegen.
- **Tasks:**
  - Turborepo init.
  - Payload Config schreiben (Collections `users`, `micro_scripts`).
  - Docker Compose für Postgres aufsetzen.
  - TaxonomyGroups/Taxonomies Collections anlegen (Facetten für Pattern-Browsing).
  - _Resultat:_ Ein funktionierendes Admin-Panel.

### Phase 2: Web-Interface (Dashboard & Planung)

- **Ziel:** Nutzer können ihre Identität definieren und Skripte planen.
- **Tasks:**
  - Next.js Setup mit Shadcn/UI.
  - Auth-Verbindung zu Payload.
  - "Script Builder": Ein Formular, das hilft, Trigger/Action zu definieren.
  - _Resultat:_ Web-App, in der man Skripte sehen und bearbeiten kann.

### Phase 3: Mobile App & State Sync (Der Kern)

- **Ziel:** Die tägliche Interaktion funktioniert (Basalganglien-Modus).
- **Tasks:**
  - Expo Setup mit NativeWind.
  - Login Screen.
  - **The Card Stack:** Das Herzstück der UI. Swipe-Interface für die täglichen Skripte.
    - Swipe Rechts: "Im Flow erledigt".
    - Swipe Links: "Nicht gemacht / Hürde".
  - Wenn Swipe Links -> Öffne kurzes Feedback-Modal ("Warum?").

### Phase 4: Der KI-Layer & Pattern Library

- **Ziel:** Intelligenz und Community mit facettiertem Pattern-Browsing.
- **Tasks:**
  - Integration Vercel AI SDK.
  - Bau des "Weekly Review" Agents (Zusammenfassung der State Syncs, Friction-Tags nutzen).
  - Öffentlicher Bereich für die Pattern Library mit Facetten-Filter (TaxonomyGroups/Taxonomies, OR/AND).
  - "Clone to my Scripts" (Pattern → MicroScript, Taxonomien denormalisieren).
  - Saved Patterns (Pin/Rating), Featured/Editorial Notes für Kuration.

---

## 5. Repository-Struktur (Verzeichnisbaum)

So sollte das Git-Repo aussehen, damit KI-Agenten sich zurechtfinden:

```text
/
├── apps/
│   ├── cms/            # Payload 3.0 (Server)
│   ├── web/            # Next.js 15 (Browser Dashboard)
│   └── mobile/         # Expo (iOS/Android App)
├── packages/
│   ├── ui/             # Shared UI Components (shadcn adaptiert)
│   ├── db/             # Prisma Schema / Payload Types
│   └── ai/             # Shared Prompts & AI Utilities
├── docker-compose.yml  # Für lokale DB
├── AGENTS.md           # Anweisungen für KI-Coding-Assistenten
└── README.md
```

---

## 6. Start-Anleitung (Für den Montag im Camp)

Um als Gruppe sofort loszulegen ("Vibecoding Session"):

1.  **Repo klonen:** `git clone https://github.com/Quandes-Projects/selfstate.git`
2.  **Abhängigkeiten:** `pnpm install`
3.  **Datenbank:** `docker-compose up -d` (Startet Postgres)
4.  **Dev Mode:** `pnpm dev` (Startet CMS, Web und Expo Server parallel).
5.  **Coding:**
    - _Team A (Web):_ Baut den "Script Builder" mit Hilfe von Cursor.
    - _Team B (Mobile):_ Baut den "Swipe-Mechanismus" für den State Sync.
    - _Team C (Content/Logic):_ Definiert die ersten "Starter-Patterns" (Quandes Best Practices) in Payload.
