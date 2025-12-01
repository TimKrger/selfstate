# Code-Elemente für SelfState

## 1) Collections (Payload-Schema als Typ-Skizze)

```ts
// taxonomyGroups.ts
const TaxonomyGroups = {
  slug: "taxonomyGroups",
  fields: [
    { name: "name", type: "text", required: true },
    { name: "slug", type: "text", unique: true },
    { name: "order", type: "number" },
    { name: "description", type: "text" },
    { name: "externalId", type: "text" }, // Import-Mapping
    { name: "isActive", type: "checkbox", defaultValue: true },
  ],
};

// taxonomies.ts
const Taxonomies = {
  slug: "taxonomies",
  admin: { defaultColumns: ["name", "taxonomyGroup", "order", "updatedAt"] },
  fields: [
    { name: "name", type: "text", required: true },
    { name: "order", type: "number" },
    {
      name: "taxonomyGroup",
      type: "relationship",
      relationTo: "taxonomyGroups",
      required: true,
    },
    { name: "externalId", type: "text" },
    { name: "description", type: "text" },
    { name: "isFeatured", type: "checkbox", defaultValue: false },
  ],
};
```

## 2) Pattern-Relation (n:m)

```ts
// patterns.ts (Auszug)
fields: [
  // ...
  {
    name: "taxonomies",
    type: "relationship",
    relationTo: "taxonomies",
    hasMany: true,
  },
  // Denorm zum schnellen Filtern
  {
    name: "taxonomyIds",
    type: "array",
    fields: [{ name: "id", type: "text" }],
  },
  {
    name: "taxonomySlugs",
    type: "array",
    fields: [{ name: "slug", type: "text" }],
  },
  {
    name: "featuredTaxonomyIds",
    type: "array",
    fields: [{ name: "id", type: "text" }],
  },
];
```

## 3) Import-Endpoints (idempotent, External IDs)

```ts
// POST /api/taxonomies/import
// Body: { taxonomyGroups?: [...], taxonomies?: [...] }
// Ablauf:
// 1) Groups importieren/updaten via externalId
// 2) Mapping externalId -> id aufbauen
// 3) Taxonomies importieren/updaten und Group-Mapping nutzen
// 4) Ergebnisliste (created/updated/error) zurückgeben
```

## 4) Filter-Query (OR innerhalb, AND zwischen Gruppen)

```json
{
  "where": {
    "and": [
      { "taxonomies.id": { "in": ["ctx-commute", "ctx-desk"] } },
      { "taxonomies.id": { "in": ["dur-2min"] } },
      { "taxonomies.id": { "in": ["idn-self"] } }
    ]
  }
}
```

## 5) Facetten-UI (Logik-Skizze)

```ts
// Datenaufbereitung
const groupedTaxonomies = groupBy(
  taxonomies,
  (t) => t.taxonomyGroup.externalId
);

// Auswahl-Status
type SelectedFilters = Record<
  string /*groupExternalId*/,
  string[] /*taxonomyIds*/
>;

// Available-Berechnung (optional):
// Query ohne bestimmte Gruppe ausführen, oder aus Result-Set ableiten, welche IDs vorkommen.

// UI-Verhalten
// - Checkbox pro Taxonomy (Mehrfachauswahl pro Gruppe)
// - Chips zeigen aktive Filter, Entfernung pro Chip
// - OR innerhalb einer Gruppe, AND zwischen Gruppen → Query-Builder wie oben
```

## 6) Pattern Library (erweiterte Felder)

```ts
fields: [
  { name: "title", type: "text", required: true },
  { name: "trigger", type: "text" },
  { name: "action", type: "textarea" },
  { name: "rewardHint", type: "text" },
  { name: "identity", type: "relationship", relationTo: "identities" },
  {
    name: "levels",
    type: "array",
    fields: [
      {
        name: "level",
        type: "select",
        options: ["L1", "L2", "L3", "L4", "L5"],
      },
      { name: "action", type: "text" },
    ],
  },
  { name: "clones", type: "number", defaultValue: 0 },
  { name: "contributorName", type: "text" },
  { name: "featured", type: "checkbox" },
  { name: "complexity", type: "select", options: ["S", "M", "L"] },
  { name: "prerequisites", type: "textarea" },
  {
    name: "intendedOutcome",
    type: "select",
    options: ["energy", "clarity", "connection", "output"],
  },
  { name: "metricHint", type: "text" },
  {
    name: "visibility",
    type: "select",
    options: ["public", "team", "private"],
    defaultValue: "public",
  },
  { name: "teamId", type: "text" },
  {
    name: "taxonomies",
    type: "relationship",
    relationTo: "taxonomies",
    hasMany: true,
  },
  // Denorm
  {
    name: "taxonomyIds",
    type: "array",
    fields: [{ name: "id", type: "text" }],
  },
  {
    name: "taxonomySlugs",
    type: "array",
    fields: [{ name: "slug", type: "text" }],
  },
  {
    name: "featuredTaxonomyIds",
    type: "array",
    fields: [{ name: "id", type: "text" }],
  },
  { name: "identitySlug", type: "text" },
  { name: "searchText", type: "textarea" },
  {
    name: "synonyms",
    type: "array",
    fields: [{ name: "value", type: "text" }],
  },
];
```

## 7) Saved Patterns (Pins/Ratings)

```ts
// saved_patterns.ts
fields: [
  { name: "user", type: "relationship", relationTo: "users", required: true },
  {
    name: "pattern",
    type: "relationship",
    relationTo: "pattern_library",
    required: true,
  },
  { name: "pinned", type: "checkbox", defaultValue: false },
  { name: "rating", type: "number", min: 1, max: 5 },
  { name: "usageCount", type: "number", defaultValue: 0 },
  { name: "lastUsedAt", type: "date" },
];
```

## 8) MicroScripts (User-seitig)

```ts
fields: [
  { name: "title", type: "text", required: true },
  { name: "trigger", type: "text" },
  { name: "action", type: "textarea" },
  { name: "rewardHint", type: "text" },
  { name: "identity", type: "relationship", relationTo: "identities" },
  { name: "owner", type: "relationship", relationTo: "users" },
  { name: "level", type: "select", options: ["L1", "L2", "L3", "L4", "L5"] },
  { name: "isActive", type: "checkbox", defaultValue: true },
  {
    name: "sourcePattern",
    type: "relationship",
    relationTo: "pattern_library",
  },
  {
    name: "contextTags",
    type: "array",
    fields: [{ name: "value", type: "text" }],
  },
  {
    name: "availabilityWindows",
    type: "array",
    fields: [
      { name: "start", type: "text" },
      { name: "end", type: "text" },
    ],
  },
  {
    name: "visibility",
    type: "select",
    options: ["private", "team", "public"],
    defaultValue: "private",
  },
  { name: "teamId", type: "text" },
  // denorm Taxonomies vom Pattern
  {
    name: "taxonomyIds",
    type: "array",
    fields: [{ name: "id", type: "text" }],
  },
];
```

## 9) StateSyncs (Flow-Logs)

```ts
fields: [
  {
    name: "microScript",
    type: "relationship",
    relationTo: "micro_scripts",
    required: true,
  },
  { name: "timestamp", type: "date", required: true },
  {
    name: "status",
    type: "select",
    options: ["flow", "friction", "skipped"],
    required: true,
  },
  { name: "flowRating", type: "number", min: 1, max: 5 },
  { name: "effortRating", type: "number", min: 1, max: 5 },
  { name: "moodDelta", type: "number", min: -5, max: 5 },
  {
    name: "frictionTags",
    type: "array",
    fields: [{ name: "code", type: "text" }],
  },
  { name: "frictionNote", type: "textarea" },
  { name: "aiAdjustment", type: "textarea" },
];
```

## 10) MicroScript-Overrides (User-spezifisch)

```ts
fields: [
  {
    name: "microScript",
    type: "relationship",
    relationTo: "micro_scripts",
    required: true,
  },
  { name: "user", type: "relationship", relationTo: "users", required: true },
  { name: "levelOverrides", type: "json" }, // z. B. { L1: "...", L2: "..." }
  {
    name: "contextTags",
    type: "array",
    fields: [{ name: "value", type: "text" }],
  },
];
```

## 11) Suche/Ranking (einfach starten)

- Freitext über `title`, `action`, `rewardHint` + Facetten-Filter.
- Ranking-Boost: `identitySlug` Match, `clones`, `saves`, `featured`.
- Optional Synonyme/Keywords-Feld am Pattern für bessere Treffer.

## 12) Clone-Flow (Pattern → MicroScript)

- Pattern lesen, `taxonomyIds` denormalisieren, MicroScript anlegen mit diesen IDs.
- User-spezifische Overrides (Levels/Context) später ergänzen.

## 13) Mobile (Expo) Adaption

- Gleiche API-Queries nutzen; Filter-UI als Bottom-Sheet mit Gruppen/Chips.
- Lokales Caching: `taxonomyGroups`, `taxonomies`, `patterns` (Lightweight-Felder).
- Offline: StateSyncs puffern, beim Reconnect hochladen.

## 14) Admin-Tools

- Listen/CRUD für TaxonomyGroups/Taxonomies.
- Import-Button (CSV/JSON), zeigt created/updated/errors.
- Schnell-Suche innerhalb Gruppen (Client-seitig filtern).

## 15) Testdaten

- Verwende `docs/OntologySamples.json` als Seed für Gruppen/Taxonomien/Patterns/Joins/SavedPatterns.

## 16) Next.js FilterView (SelfState, aus mysuricate adaptiert) – Kochrezept

- **Server-Loader:** In der Pattern-Übersicht (`/patterns` o.ä.) `taxonomyGroups` (limit 100, sort name) und `taxonomies` (limit 1000, sort `[taxonomyGroup, name]`) via Payload laden und als Props an die Client-Komponente geben. Gleiches Schema für Pattern-Collections, falls es eine kombinierte Ansicht gibt.
- **State/Init (Client):**
  ```ts
  type SelectedFilters = Record<string, string[]>
  const [searchTerm, setSearchTerm] = useState(() => searchParams.get('search') || '')
  const [selectedFilters, setSelectedFilters] = useState<SelectedFilters>(() => {
    const filters: SelectedFilters = {}
    taxonomyGroups.forEach(group => {
      const param = searchParams.get(group.externalId)
      if (param) filters[group.externalId] = param.split(',')
    })
    return filters
  })
  const [showFavoritesOnly, setShowFavoritesOnly] = useState(false)
  const [items, setItems] = useState<Pattern[] | PatternCollection[]>([])
  const [page, setPage] = useState(1)
  const [hasMore, setHasMore] = useState(true)
  ```
- **Debounce + URL-Sync:** `useEffect` auf `searchTerm`, `selectedFilters`, `showFavoritesOnly` (skip wenn Favorites aktiv). Nach 300 ms:
  1. `page` auf 1 zurücksetzen, Params (`search`, jede Filtergruppe) bauen.
  2. `fetch` auf Endpoint (`/api/patterns` oder `/api/pattern-collections`) mit `page`/`limit` ausführen, `items` + `hasMore` setzen.
  3. URL mit `router.replace` aktualisieren, nur wenn Query-String sich ändert; bei Patterns optional `patternId` aus aktueller URL beibehalten. `searchParams` **nicht** als Dependency, damit kein Loop entsteht.
- **Filter-Handler (Toggle pro Checkbox):**
  ```ts
  const handleFilterChange = (groupExternalId: string, taxonomyId: string, checked: boolean) => {
    setSelectedFilters(prev => {
      const next = { ...prev }
      next[groupExternalId] = next[groupExternalId] || []
      if (checked) {
        if (!next[groupExternalId].includes(taxonomyId)) next[groupExternalId].push(taxonomyId)
      } else {
        next[groupExternalId] = next[groupExternalId].filter(id => id !== taxonomyId)
        if (next[groupExternalId].length === 0) delete next[groupExternalId]
      }
      return { ...next }
    })
  }
  const handleClearAll = () => { setSelectedFilters({}); setShowFavoritesOnly(false) }
  const handleRemoveFilter = (groupId, taxonomyId) => handleFilterChange(groupId, taxonomyId, false)
  ```
- **Favorites-Flow:** `showFavoritesOnly` Toggle räumt andere Filter leer, lädt Favoriten via POST (`/api/patterns/favorites` bzw. `/api/pattern-collections/favorites`) mit IDs aus den Favorites-Hooks (LocalStorage + Analytics). Während Favorites aktiv sind, wird der reguläre Fetch-Effekt übersprungen.
- **Pagination & Infinite Scroll:** `loadMore` baut `URLSearchParams` mit `page+1`, holt weitere Items, dedupliziert per `Set` auf `id`, setzt `hasMore`; Scroll-Listener triggert bei 1000 px vor Seitenende.
- **Verfügbare Facetten berechnen:**  
  ```ts
  const TAXONOMY_FIELDS = ['contextTags','identity','duration','location','mood','goal'] // an SelfState anpassen
  const availableTaxonomyIds = useMemo(() => {
    const ids = new Set<string>()
    items.forEach(item => TAXONOMY_FIELDS.forEach(field => {
      const value = (item as any)[field]
      const push = (v: any) => { if (v) ids.add(String(v?.id ?? v?.value?.id ?? v?.value ?? v)) }
      Array.isArray(value) ? value.forEach(push) : push(value)
    }))
    return ids
  }, [items])
  ```
  Wird an `FilterSidebar` gereicht, um Gruppen/Optionen ohne Treffer auszublenden bzw. zu disablen.
- **UI-Komposition:**
  - Desktop `FilterSidebar`; mobile Drawer nutzt dieselbe Komponente.
  - `FilterSidebar` Features: Favoriten-Toggle, expand/collapse pro Gruppe, optional Suchfeld innerhalb einer Gruppe (ab >5 Einträgen), Badge mit aktiven Filtern, Checkboxen; Pattern-Variante kann Gruppen einzeln öffnen, Collection-Variante erlaubt mehrere gleichzeitig.
  - `FilterChips`: Map aus `taxonomyGroups`/`taxonomies`, rendert Chips + Heart-Chip für Favoriten; Buttons rufen `onRemoveFilter` bzw. `onClearAll`.
  - `activeFilterCount` (`selectedFilters` + Favorites) steuert Badges/Drawer-Button. Modal-View nutzt `patternId` aus URL; beim Schließen wird der Param per `history.replaceState` entfernt.

## 17) API-Handler für Facetten-Filter (Payload, Next.js route.ts)

- **Patterns (analog zu `src/app/api/games/create/route.ts`):**
  - `where` startet mit `isPublished=true` (außer `includeDrafts=true` + Auth).
  - `search` baut `or` über Kernfelder (Titel, Kurzbeschreibung, Quelle/Owner).
  - Pro TaxonomyGroup: `searchParams.get(group.externalId)` → IDs splitten (String oder Zahl). Mapping `externalId → Field` (z. B. `contextNode` → `contextTags`, `identityNode` → `identity`, `durationNode` → `availabilityWindows`, …). Für jedes Mapping: `where.and.push({ [fieldName]: { in: taxonomyIds } })` → OR innerhalb der Gruppe, AND zwischen Gruppen.
  - Query: `payload.find({ collection: 'pattern_library', where, page, limit, sort: '-createdAt', depth: 2 })`.
- **Pattern-Collections (`src/app/api/game-chains/route.ts` als Vorlage):**
  - `where.and` startet mit `isPublished=true`.
  - `search` (optional) → `or` über `title`, `shortDescription`, `tags`.
  - Erlaubte Gruppen-Slugs SelfState-spezifisch wählen (z. B. `identities`, `contexts`, `effortLevel`); Query-Param pro Gruppe → `where.and.push({ [slug]: { in: ids } })`.
  - Query: `payload.find({ collection: 'pattern_collections', where, page, limit, sort: '-updatedAt', depth: 2 })`.
- **Semantik:** `in` liefert OR in einer Gruppe; mehrere Gruppen in `and` ergeben AND-Verknüpfung. Favorites laufen über eigene POST-Endpunkte (IDs aus LocalStorage), nicht über diese Filter-Queries.

## 18) Favorites-Layer (Client + API)

- **Client (Patterns):** `useFavorites` (`src/contexts/FavoritesContext.tsx`) hält ein `Set<string>` in LocalStorage (`selfstate_favorites` o.ä.), bietet `add/remove/toggle/isFavorite/getFavoriteIds/getFavoriteCount`. Optimistische Updates werden bei Tracking-Fehlern rückgängig gemacht. Analytics via `trackFavoriteToggle` (s.u.).
- **Client (Pattern-Collections):** `usePatternCollectionFavorites` (Analogon zur bisherigen Collection-Variante) speichert IDs unter `selfstate-favorite-patterncollections` (oder konsistentem Key), lädt beim Mount, optimistisch add/remove + revert bei Fehler, Analytics via `usePatternCollectionAnalytics`.
- **API:** POST `/api/patterns/favorites` bzw. `/api/pattern-collections/favorites` erwartet `{ favoriteIds: string[] }`; validiert leere Arrays, findet alle `isPublished=true` + `id in favoriteIds`, `limit` = Anzahl IDs, sort `-updatedAt`, `depth:2`. Reihenfolge der Antwort entspricht der übergebenen ID-Liste (`favoriteIds.map(id => docs.find(...))`). Liefert `success`, `docs`, `totalDocs`, `hasNextPage:false`.
- **UI-Hook:** Favorites-Filter setzt andere Filter zurück, nutzt POST-Endpunkte zum Laden aller Favoriten (kein Pagination), zeigt Heart-Chip, Badge im Sidebar-Header und Toggle (siehe Sidebar/Chips-Komponenten).

## 19) Analytics-Hooks (Click/Favorite)

- **Patterns (`src/hooks/useAnalytics.ts` als Vorlage):** `trackEvent(patternId, event)` POST `/api/patterns/{id}/analytics` mit `{ event: 'click' | 'favorite' | 'unfavorite' }`. Doppelaufrufe werden über `isTracking[patternId-event]` abgefangen. Helpers: `trackClick`, `trackFavoriteToggle(patternId, isFavorite)`, `isTrackingEvent`.
- **Pattern-Collections (ehem. Chain-Hook als Vorlage):** Single-flight-Guard (`isTracking`), Endpunkt `/api/pattern-collections/{id}/analytics`, Events `click|favorite|unfavorite`, Wrapper `trackFavoriteToggle(id, isFavorite)`.
- **Einsatz:** Klick auf Karte oder Modalkachel löst Tracking aus; Favoriten-Buttons in Cards/Modals rufen Toggle + Analytics, Favorites-Filter nutzt gespeicherte IDs und getrennte Fetch-Endpunkte (siehe oben).
