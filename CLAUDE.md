# CLAUDE.md - GeoGuess

## Project Overview

GeoGuess is an open-source geography guessing game built with Vue 2 and Google Maps Street View. Players compete to guess random locations across five rounds, with support for both single-player and multiplayer (up to 5 players via Firebase Realtime Database). Custom maps via GeoJSON are supported.

## Tech Stack

- **Framework**: Vue 2.6 with Vue CLI 5
- **UI Library**: Vuetify 2 (Material Design)
- **State Management**: Vuex 3 (namespaced modules, auto-loaded)
- **Routing**: Vue Router 3 (history mode)
- **Backend/Realtime**: Firebase 8 (Realtime Database, Analytics)
- **Maps**: Google Maps API via `gmap-vue`
- **Geospatial**: Turf.js (bbox, distance, point-in-polygon)
- **HTTP**: Axios with caching (`axios-cache-adapter`)
- **i18n**: vue-i18n with 16 locales (Crowdin-managed)
- **Styling**: SCSS + Vuetify theming
- **PWA**: Vue CLI PWA plugin with service worker

## Commands

```bash
npm install          # Install dependencies
npm run serve        # Dev server at localhost:8080
npm run build        # Production build to dist/
npm run lint         # ESLint (auto-fix)
npm run test:unit    # Jest unit tests with coverage
npm run test:e2e     # Cypress end-to-end tests
npm test             # lint + test:unit + test:e2e
npm run test:mutation # Stryker mutation testing
```

### Lint in CI mode (no auto-fix)

```bash
NODE_ENV=production npm run lint -- --no-fix
```

## Project Structure

```
src/
  App.vue                    # Root component
  main.js                    # App entry - Firebase init, Vue plugins
  router.js                  # Route definitions (history mode)
  constants.js               # GAME_MODE, SCORE_MODE, AREA_MODE, STROKE_COLORS
  registerServiceWorker.js   # PWA service worker registration
  assets/                    # Static assets (images, SVGs)
  components/
    DialogMessage.vue        # Generic message dialog
    DialogSummary.vue        # Game summary dialog
    HeaderGame.vue           # In-game header bar
    Maps.vue                 # Maps display component
    dialogroom/              # Multiplayer room setup dialogs
    game/                    # In-game components (DetailsMap, Leaderboard)
    history/                 # Game history display
    home/                    # Home page components (SearchBox, MapsContainer, cards)
    map/                     # Map components with mixins
    page/                    # Layout components (Header, Footer, ContentPage, About)
    shared/                  # Reusable components (FlagIcon, SaveButton, TimePicker)
  lang/
    index.js                 # i18n setup, auto-loads locale JSON files
    locale/                  # Translation JSON files (en.json, fr.json, etc.)
  mixins/
    ConfirmExitMixin.js      # Navigation guard mixin
  models/
    GeoMap.js                # GeoMap data model class
  pages/
    Home.vue                 # Home/landing page
    HistoryPage.vue          # Game history page
    MedalsPage.vue           # Medals/achievements page
    StreetView.vue           # Main game page (Street View)
  plugins/
    axios.js                 # Axios with cache adapter
    countryNamePlugin.js     # Vue plugin for country name resolution
    IndexedDBService.js      # IndexedDB for local game history
    StreetViewService.js     # Google Street View API wrapper
    vuetify.js               # Vuetify configuration
  scss/
    variables.scss           # SCSS variables
  store/
    index.js                 # Vuex store - auto-loads modules from modules/
    mutation-types.js         # All mutation type constants
    modules/
      alert.store.js         # Alert/notification state
      area.store.js          # Game area/region state
      home.store.js          # Home page state (maps, GeoJSON, history)
      settings.store.js      # Game settings, room management, player state
  utils/
    index.js                 # Shared utilities (URL validation, GeoJSON, scoring helpers)
    game/
      medals.js              # Medal/achievement calculations
      score.js               # Score calculation logic
tests/
  unit/                      # Jest unit tests (mirrors src/ structure)
  e2e/                       # Cypress E2E tests
    specs/                   # E2E test specs
    support/                 # Cypress support files
    plugins/                 # Cypress plugins
```

## Architecture Patterns

### Vuex Store
- Modules are **auto-loaded** from `src/store/modules/` via `require.context`
- Module filenames follow `*.store.js` pattern; they are registered as `*Store` (e.g., `settings.store.js` -> `settingsStore`)
- All modules use **namespaced: true**
- Mutation types are centralized in `src/store/mutation-types.js` as exported constants (e.g., `HOME_SET_MAP`)
- Mutations are defined using computed property names: `[MutationTypes.CONSTANT_NAME](state, payload) { ... }`

### i18n / Translations
- Locale JSON files in `src/lang/locale/` are auto-loaded via `require.context`
- Translations managed via Crowdin (https://translate.geoguess.games/)
- Fallback locale is `en`
- RTL languages defined in `src/lang/index.js`
- To add a language: create JSON file in `src/lang/locale/`, then add vuetify locale import in `src/plugins/vuetify.js`

### Components
- Single-file components (`.vue`) with `<template>`, `<script>`, `<style>` sections
- BEM-style CSS class naming (e.g., `home-page__main__container`)
- Vuetify components used extensively (`v-container`, `v-layout`, `v-btn`, `v-icon`, etc.)
- Path alias: `@/` maps to `src/`

### Routing
- History mode (no hash)
- Lazy-loaded routes: `StreetView` page uses dynamic import
- Route guards validate game mode and time params before entering game

## Environment Variables

Copy `.env.dist` to `.env` and fill in values:

| Variable | Description |
|---|---|
| `VUE_APP_API_KEY` | Google Maps API key |
| `VUE_APP_FIREBASE_API_KEY` | Firebase API key |
| `VUE_APP_FIREBASE_AUTH_DOMAIN` | Firebase auth domain |
| `VUE_APP_FIREBASE_DATABASE_URL` | Firebase Realtime Database URL |
| `VUE_APP_FIREBASE_PROJECT_ID` | Firebase project ID |
| `VUE_APP_STORAGE_BUCKET` | Firebase storage bucket |
| `VUE_APP_FIREBASE_MESSAGING_SENDER_ID` | Firebase messaging sender ID |
| `VUE_APP_FIREBASE_APP_ID` | Firebase app ID |
| `VUE_APP_FIREBASE_MEASUREMENT_ID` | Firebase measurement ID |
| `VUE_APP_LIST_MAPS_JSON_URL` | URL for list of available maps |
| `VUE_APP_PUBLIC_PATH` | Public path for production deployment |

## Testing

### Unit Tests (Jest)
- Config: `jest.config.js` (preset `@vue/cli-plugin-unit-jest`)
- Test files: `tests/unit/**/*.spec.js` - mirror the `src/` directory structure
- Coverage collected from `src/**/*.{js,vue}` excluding plugins, lang, router, main, registerServiceWorker
- Mock: `jest-google-maps-mock` for Google Maps API
- Run: `npm run test:unit`

### E2E Tests (Cypress)
- Config: `cypress.config.js`
- Specs: `tests/e2e/specs/`
- Base URL: `http://localhost:8080/`
- Run: `npm run test:e2e` (requires dev server running)

### Mutation Testing (Stryker)
- Config: `stryker.conf.json`
- Run: `npm run test:mutation`

## Code Style

- **ESLint** with `plugin:vue/essential` and Prettier
- Semicolons required (`semi: ['error', 'always']`)
- 4-space HTML indentation in Vue templates (`vue/html-indent: ['warn', 4]`)
- `no-console` is error in production, warning in dev
- `no-debugger` is error in production, off in dev
- Multi-word component names rule is disabled
- Prettier handles formatting

## CI/CD

### GitHub Actions (`.github/workflows/ci.yml`)
Pipeline runs on push to `master` and all PRs:
1. **Linter** - `NODE_ENV=production npm run lint -- --no-fix`
2. **Unit tests** - `npm run test:unit`
3. **Build** - `npm run build`
4. **E2E tests** - Cypress on Chrome (conditional on API key availability)
5. **Code coverage** - Codecov integration

### Docker
- Multi-stage build: Node LTS Alpine (build) + Nginx Alpine (serve)
- `entrypoint.sh` handles runtime environment variable substitution
- `docker-compose.yml` available for local Docker deployment

### Deployment
- Netlify (`netlify.toml`) and Vercel (`vercel.json`) configs present
- Production build outputs to `dist/`

## Key Game Concepts

- **Game Modes**: `CLASSIC` (lat/lng guess), `COUNTRY` (country guess), `CUSTOM_AREA` (area-based guess)
- **Score Modes**: `NORMAL` (distance only), `TIME` (distance + time factor)
- **Multiplayer**: Uses Firebase Realtime Database for room creation/joining and game state sync
- **Custom Maps**: GeoJSON-based (FeatureCollection or Feature with Point/Polygon/MultiPolygon geometries)
- **Scoring**: Max 5000 points per round, exponential decay based on distance from target, with difficulty scaling based on map bbox size
