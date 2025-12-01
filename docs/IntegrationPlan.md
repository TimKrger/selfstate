# SelfState: Implementierungsplan (Taxonomie-/Filter-System)

Ziel: Erst als Payload-Web-App, danach Expo-Wrapping.

## A. Datenmodell-Übernahme (Payload CMS)

1. **TaxonomyGroups**: `name`, `slug`, `order`, `description`, `externalId`, `isActive`.
2. **Taxonomies**: `name`, `order`, `description`, `externalId`, `taxonomyGroup`, `isFeatured`.
3. **Pattern Library**: `title`, `trigger`, `action`, `rewardHint`, `identity`, `levels (L1–L5)`, `clones`, `contributorName`, `featured`, `complexity`, `prerequisites`, `intendedOutcome`, `metricHint`, `visibility`, `teamId`, Relation `taxonomies` (hasMany), denorm: `taxonomyIds`, `taxonomySlugs`, `identitySlug`, `featuredTaxonomyIds`, optional `searchText`, `synonyms`.
4. **PatternTaxonomy**: n:m Join (alternativ: Relation + denorm).
5. **SavedPatterns**: `userId`, `patternId`, `pinned`, `rating`, `usageCount`, `lastUsedAt`.
6. **MicroScripts**: Relation zum Pattern, übernehmen `taxonomyIds`, user-spezifische Overrides (`micro_script_overrides`: Levels/Context), `contextTags`, `availabilityWindows`, `visibility`/`teamId`.
7. **StateSyncs**: `flowRating`, `effortRating`, `moodDelta`, `frictionTags` (Codes), `frictionNote`, `aiAdjustment`.

## B. API/Import

1. **Import**
   - Kombiniert: POST `/api/taxonomies/import` mit `taxonomyGroups`, `taxonomies`, optional Pattern-Taxonomy-Mapping; idempotent via `externalId`.
   - Einzel: POST `/api/taxonomyGroups/import`, `/api/taxonomies/import`.
2. **Queries/Filter**
   - OR innerhalb einer Gruppe, AND zwischen Gruppen: `where.and = [ { "taxonomies.id": { "in": [...] } }, ... ]`.
   - Freitext + Facetten (z. B. `title/action/rewardHint like ...` + `taxonomies.in`).
3. **Search/Ranking**
   - Boost auf Identität, Popularität (clones/saves), jüngste Flow-Raten, Featured-Taxonomies; Synonyme/Keywords optional.

## C. Frontend-Patterns

1. **Facetten-Sidebar**: Gruppen-Accordion, Checkboxen, Mehrfachauswahl, Suchfeld pro Gruppe, „Available“-Status pro Taxonomy.
2. **Filter-Chips**: Anzeige aktiver Filter, schnelles Entfernen pro Facette.
3. **Result-Liste**: Badges für Taxonomies (Kontext/Dauer/Energie/Identität), „Featured“-Hervorhebung.
4. **Creation/Editing**: Pattern-/MicroScript-Form mit Mehrfach-Auswahl; Clone-Flow übernimmt Taxonomies und denorm Felder.

## D. Schrittweiser Plan (Payload → Expo)

1. **Payload-Web-App**
   - Collections erweitern, Import-Endpoints portieren (idempotent).
   - Facetten-UI (Sidebar + Chips + Query-Builder).
   - Suche/Ranking (Identität, clones/saves, featured) simpel starten.
   - Saved Patterns, Featured/Editorial Notes.
   - Weekly Review Agent nutzt Friction-Tags/Effort/Mood.
2. **Expo-Wrapper**
   - Shared API-Client (REST/GraphQL) für Facetten-Filter/Patterns.
   - Mobile UI: Bottom-Sheet-Filter, Chips, Quick-Apply.
   - Caching von TaxonomyGroups/Taxonomies/Patterns; StateSync-Upload bei Reconnect.
   - Flow: Pattern-Browse → Clone → MicroScript → State Sync (Card Stack).

## E. Ergänzende Punkte

- Teams/Visibility: `teamId` + `visibility` (private/team/public) auf Pattern/MicroScript.
- Localization: optionale `translations` an Pattern/MicroScript.
- Analytics/Events: `pattern.saved`, `pattern.cloned`, `stateSync.frictionTags` für Empfehlungen.
- Denorm/Performance: `taxonomyIds`/`slugs` am Pattern/MicroScript; ggf. Suchfeld `searchText`.
- Admin-Tools: Management-UI für TaxonomyGroups/Taxonomies (Listen, Suche, „available“ Check).
- Testdaten: siehe `docs/OntologySamples.json` für Facetten-/Pattern-Beispiele.
