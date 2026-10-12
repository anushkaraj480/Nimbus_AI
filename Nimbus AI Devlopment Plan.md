# Nimbus AI Weather App: Development Plan and Architecture

**Project type:** Mobile app mini project
**Framework:** Flutter (Dart)
**Architecture:** Layered MVVM with Repository pattern
**Duration:** ~3 weeks (21 days)
**Target platform:** Android (iOS-ready, since Flutter is cross-platform)

---

## Table of Contents

1. [Project Summary](#1-project-summary)
2. [Requirements](#2-requirements)
3. [Technology Stack](#3-technology-stack)
4. [System Architecture](#4-system-architecture)
5. [Data Layer Design](#5-data-layer-design)
6. [API Design](#6-api-design)
7. [State Management Design](#7-state-management-design)
8. [Feature Logic Design](#8-feature-logic-design)
9. [UI and Navigation Design](#9-ui-and-navigation-design)
10. [Project Folder Structure](#10-project-folder-structure)
11. [Development Phases and Milestones](#11-development-phases-and-milestones)
12. [Permissions and Configuration](#12-permissions-and-configuration)
13. [Error Handling Strategy](#13-error-handling-strategy)
14. [Testing Plan](#14-testing-plan)
15. [Git and Version Control Plan](#15-git-and-version-control-plan)
16. [Build and Release Checklist](#16-build-and-release-checklist)
17. [Risks and Mitigation](#17-risks-and-mitigation)
18. [Definition of Done](#18-definition-of-done)
19. [AI Features (Nimbus AI)](#19-ai-features-nimbus-ai)

---

## 1. Project Summary

### 1.1 Problem statement
Most weather apps display raw data (temperature, humidity, wind) but leave users to work out what it means for their day. They also tend to look and behave the same.

### 1.2 Proposed solution
**Nimbus AI** is a weather app that combines accurate forecasts with **actionable guidance**: what to wear, the best time for outdoor activities, upcoming rain alerts, and air quality advice.

### 1.3 Objectives
- Display accurate current, hourly, and daily weather.
- Support search and GPS-based location.
- Provide smart, rule-based guidance (outfit, activity, rain, air quality).
- Work offline using cached data.
- Follow a clean, maintainable architecture.

### 1.4 Scope

| In scope | Out of scope (future) |
|---|---|
| Current, hourly, 7-day forecast | Weather radar maps |
| City search and GPS | User accounts / cloud sync |
| Favorites and offline cache | Multi-language support |
| Outfit, Activity, Rain, AQI advisors | Wearable app |
| Morning briefing notification | Paid features |

---

## 2. Requirements

### 2.1 Functional requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-01 | Show current weather (temp, feels-like, humidity, wind, pressure, UV) | Must |
| FR-02 | Show 24-hour hourly forecast | Must |
| FR-03 | Show 7-day daily forecast | Must |
| FR-04 | Search cities by name with suggestions | Must |
| FR-05 | Detect current location using GPS | Must |
| FR-06 | Toggle between °C/°F and km/h/mph | Must |
| FR-07 | Save and remove favorite cities | Should |
| FR-08 | Show cached data when offline | Should |
| FR-09 | Outfit Advisor suggestions | Must |
| FR-10 | Rain alert for the next 2 hours | Must |
| FR-11 | Activity Score Planner | Should |
| FR-12 | Air Quality Index and health advice | Should |
| FR-13 | Dynamic weather-based background | Should |
| FR-14 | Daily morning briefing notification | Could |
| FR-15 | Compare two cities | Could |
| FR-16 | Voice search | Could |
| FR-17 | Light / dark / system theme | Should |

### 2.2 Non-functional requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-01 | Home screen loads (with network) | Under 3 seconds |
| NFR-02 | Cached data displays | Under 1 second |
| NFR-03 | App does not crash on network failure | 0 crashes |
| NFR-04 | Minimum Android version | Android 7.0 (API 24) or as set by Flutter |
| NFR-05 | Code organized by layers with no API calls inside widgets | Enforced |
| NFR-06 | UI adapts to different screen sizes | Phones, and tablets if time allows |

---

## 3. Technology Stack

| Area | Choice | Purpose |
|---|---|---|
| Language | Dart | App code |
| Framework | Flutter | UI and cross-platform |
| IDE | Android Studio or VS Code | Development |
| HTTP client | `dio` | API calls, timeouts, interceptors |
| State management | `provider` (upgrade to `riverpod` optional) | Connect logic to UI |
| Location | `geolocator`, `geocoding` | GPS and reverse geocoding |
| Local database | `hive` (or `sqflite`) | Cache and favorites |
| Key-value storage | `shared_preferences` | Settings |
| Connectivity | `connectivity_plus` | Detect offline state |
| Notifications | `flutter_local_notifications`, `timezone` | Morning briefing |
| Background tasks | `workmanager` | Scheduled refresh |
| Charts | `fl_chart` | Hourly temperature chart |
| Animation | `lottie` | Weather animations |
| Voice | `speech_to_text` | Voice search (optional) |
| Utilities | `intl` | Date and number formatting |
| Testing | `flutter_test`, `mocktail` | Unit and widget tests |
| Version control | Git + GitHub | Source management |

> Package versions change often. Add each one with `flutter pub add <package>` so you get the current compatible version, and read its README for setup steps.

### Weather data sources (Open-Meteo, no API key required)
- Forecast API
- Geocoding API
- Air Quality API

Check the official docs (https://open-meteo.com/en/docs) when you begin, because parameter names and limits can change.

---

## 4. System Architecture

### 4.1 High-level architecture

```
+--------------------------------------------------------------+
|                       PRESENTATION LAYER                     |
|   Screens: Home | Search | Details | Activity | AQI | Settings|
|   Widgets: WeatherCard, HourlyTile, DailyTile, AqiGauge      |
+------------------------------+-------------------------------+
                               | watches / calls
+------------------------------v-------------------------------+
|                     STATE / LOGIC LAYER                      |
|   Providers: WeatherProvider, SearchProvider,                |
|              FavoritesProvider, SettingsProvider             |
|   Advisors:  OutfitAdvisor, ActivityScorer, RainAlertService,|
|              AqiAdvisor                                      |
+------------------------------+-------------------------------+
                               | requests data
+------------------------------v-------------------------------+
|                        DOMAIN / DATA ACCESS                  |
|   WeatherRepository: decides network vs cache                |
+------------------+-------------------------+-----------------+
                   |                         |
        +----------v---------+    +----------v-----------+
        |    REMOTE SOURCE   |    |     LOCAL SOURCE     |
        | WeatherApiService  |    | CacheStorage (Hive)  |
        | GeocodingService   |    | FavoritesStorage     |
        | AirQualityService  |    | SettingsStorage      |
        +----------+---------+    +----------------------+
                   |
          +--------v---------+
          |  Open-Meteo APIs |
          +------------------+
```

### 4.2 Layer responsibilities

| Layer | Responsibility | Must NOT do |
|---|---|---|
| **Presentation** | Draw UI, react to user input | Call APIs, contain business rules |
| **State / Logic** | Hold state, run advisors, expose data to UI | Know about HTTP or storage details |
| **Repository** | Choose between remote and cache, map errors | Contain UI code |
| **Data sources** | Talk to API or database, parse JSON | Hold UI state |

**Benefit:** you can change the API or database without touching the UI.

### 4.3 Data flow: loading weather (sequence)

```
User opens app
     |
     v
HomeScreen --> WeatherProvider.loadWeather(city)
                     |
                     v
              WeatherRepository.getWeather(city)
                     |
        +------------+-------------+
        | Is there internet?       |
        +--yes---------------no----+
        |                          |
        v                          v
 WeatherApiService.fetch()   CacheStorage.read(city)
        |                          |
   success? ---no--> fallback -----+
        |
       yes
        v
 Save to CacheStorage
        |
        v
 Return WeatherData  -->  Provider updates state  -->  UI rebuilds
        |
        v
 Advisors run on WeatherData (outfit, rain, activity)
```

### 4.4 Caching strategy

| Rule | Detail |
|---|---|
| Cache key | City ID or rounded latitude/longitude |
| Freshness window | Treat data as fresh for ~15 to 30 minutes |
| If fresh cache exists | Show immediately, skip network call |
| If stale cache + online | Show cache first, refresh in background, then update UI |
| If offline | Show cache with a "Last updated at HH:MM" label |
| If no cache + offline | Show a friendly offline message with a retry button |

This pattern is called **"stale-while-revalidate"** and keeps the app fast.

### 4.5 UI state model

Every screen that loads data should be in exactly one of these states:

```
Initial --> Loading --> Success(data)
                   \--> Error(message)
                   \--> Offline(cachedData)
```

Model it with a Dart `enum` or sealed class so the UI handles every case.

---

## 5. Data Layer Design

### 5.1 Core models

```
WeatherData
 |- location: LocationInfo
 |- current: CurrentWeather
 |- hourly: List<HourlyWeather>
 |- daily: List<DailyWeather>
 |- fetchedAt: DateTime

LocationInfo
 |- name, country, latitude, longitude, timezone

CurrentWeather
 |- temperature, feelsLike, humidity, windSpeed
 |- pressure, uvIndex, weatherCode, isDay

HourlyWeather
 |- time, temperature, rainProbability, weatherCode, uvIndex

DailyWeather
 |- date, tempMax, tempMin, rainProbability
 |- sunrise, sunset, uvMax, weatherCode

AirQualityData
 |- usAqi, pm25, pm10, category (Good / Moderate / ...)

AppSettings
 |- temperatureUnit, windUnit, themeMode
 |- briefingEnabled, briefingTime, healthSensitive
```

### 5.2 Local storage plan

| Storage | Holds | Tool |
|---|---|---|
| Weather cache | Last `WeatherData` per city (JSON) | Hive box `weather_cache` |
| Favorites | List of `LocationInfo` | Hive box `favorites` |
| Recent searches | Last 5 city names | Hive box `recent_searches` |
| Settings | Units, theme, notification options | `shared_preferences` |

### 5.3 Weather code mapping
Open-Meteo returns numeric weather codes (WMO codes). Create one mapping file that converts a code into:
- a text label ("Light rain")
- an icon
- a background theme (sunny / cloudy / rainy / storm / snow / night)

Keep it in one place (`core/utils/weather_code_mapper.dart`) so every screen stays consistent.

---

## 6. API Design

> Verify exact parameter names in the official docs before coding; the lists below show what to request.

### 6.1 Forecast
```
GET https://api.open-meteo.com/v1/forecast
  ?latitude={lat}&longitude={lon}
  &current=temperature_2m,apparent_temperature,relative_humidity_2m,
           wind_speed_10m,surface_pressure,weather_code,is_day
  &hourly=temperature_2m,precipitation_probability,weather_code,uv_index
  &daily=weather_code,temperature_2m_max,temperature_2m_min,
         sunrise,sunset,uv_index_max,precipitation_probability_max
  &forecast_days=7
  &timezone=auto
```

### 6.2 Geocoding (city search)
```
GET https://geocoding-api.open-meteo.com/v1/search
  ?name={query}&count=5&language=en
```

### 6.3 Air quality
```
GET https://air-quality-api.open-meteo.com/v1/air-quality
  ?latitude={lat}&longitude={lon}
  &current=us_aqi,pm2_5,pm10
```

### 6.4 Networking rules
- Set connect and receive **timeouts** (for example 10 seconds).
- Retry once on timeout.
- Fetch forecast and air quality **in parallel** (`Future.wait`) to save time.
- Convert all HTTP/JSON failures into your own error types in the repository.

---

## 7. State Management Design

### 7.1 Providers

| Provider | State it holds | Key methods |
|---|---|---|
| `WeatherProvider` | Current `WeatherData`, UI state, selected location | `loadWeather()`, `refresh()`, `useCurrentLocation()` |
| `SearchProvider` | Search results, recent searches | `search(query)`, `clear()` |
| `FavoritesProvider` | Saved cities | `add()`, `remove()`, `load()` |
| `SettingsProvider` | Units, theme, notification prefs | `setUnit()`, `setTheme()`, `toggleBriefing()` |
| `AirQualityProvider` | AQI data and state | `load(lat, lon)` |

### 7.2 Rules
- Providers call **repositories**, never services directly.
- Providers expose read-only state; the UI changes state only through methods.
- Advisor outputs are **computed** from `WeatherData` (not stored separately), so they stay in sync.

### 7.3 Learning path
Start with `setState` for the first screen to understand rebuilds, then move to `provider`. Consider `riverpod` only after the app works.

---

## 8. Feature Logic Design

### 8.1 Outfit Advisor
**Input:** feels-like temperature, rain probability, wind speed, UV index
**Output:** list of tips and an icon set

| Condition | Tip |
|---|---|
| Feels-like < 10°C | Heavy jacket, gloves |
| 10-17°C | Light jacket or sweater |
| 18-27°C | T-shirt, comfortable clothes |
| >= 28°C | Light cotton, stay hydrated |
| Rain probability > 50% | Carry an umbrella |
| UV index >= 6 | Sunglasses and sunscreen |
| Wind > 30 km/h | Windproof layer |

### 8.2 Rain Alert (next 2 hours)
1. Take the next 2 to 3 hourly entries from the current time.
2. If any has rain probability above ~50% (tune this threshold), mark it as likely rain.
3. Estimate the start time from the first matching hour and show a message such as "Rain likely around 4 PM."
4. If none match, show nothing, or a "No rain expected soon" message.

Note that hourly data gives hour-level accuracy only, so use soft wording like "around" and "likely."

### 8.3 Activity Score Planner
**Score = 100 minus penalties**, calculated for each hour, per activity.

| Factor | Penalty idea (example) |
|---|---|
| Temperature outside the comfort range | Up to -40 |
| Rain probability | Up to -40 |
| Wind speed | Up to -20 |
| Poor air quality (for outdoor exercise) | Up to -30 |
| UV index too high | Up to -15 |

- Define a separate comfort range for each activity (running, cycling, drying clothes, photography).
- Output: the top-scoring window and a rating (Excellent / Good / Fair / Poor).
- Keep weights in a config map so they can be tuned easily.

### 8.4 Air Quality Advisor

| US AQI | Category | Advice |
|---|---|---|
| 0-50 | Good | Great day for outdoor activity |
| 51-100 | Moderate | Fine for most; sensitive people take care |
| 101-150 | Unhealthy for sensitive groups | Limit long outdoor exertion if sensitive |
| 151-200 | Unhealthy | Reduce outdoor exercise |
| 201+ | Very unhealthy / Hazardous | Stay indoors if possible |

Add a "sensitive profile" setting that shifts thresholds to be stricter. This is general information, not medical advice.

### 8.5 Morning Briefing
- User chooses a time in Settings.
- Schedule a daily task that fetches the forecast for the saved/favorite location and posts a notification with a one-line summary (temperature range, rain chance, outfit tip).
- Request notification permission on Android 13+.
- Test thoroughly on a real device, because some phone brands restrict background tasks.

### 8.6 City Compare (optional)
- Select two cities, fetch both in parallel, and show metrics side by side.
- Highlight the better value per metric (for example, lower rain chance).

### 8.7 Dynamic Backgrounds
Choose the background from `weatherCode` + `isDay`:

`clear-day`, `clear-night`, `cloudy`, `rain`, `thunderstorm`, `snow`, `fog`

Use gradients first (simple and fast); add Lottie animation afterwards.

---

## 9. UI and Navigation Design

### 9.1 Screens

| Screen | Contents |
|---|---|
| Splash | Logo, permission check |
| Home | Location name, current temp, condition, rain alert banner, outfit tip card, hourly strip, 7-day list |
| Search | Search field, voice button, results, recent and favorite cities |
| Details | Humidity, wind, pressure, UV, sunrise/sunset, hourly temperature chart |
| Activity Planner | Cards per activity with best time and score |
| Air Quality | AQI gauge, pollutant values, advice |
| Compare | Two city columns |
| Settings | Units, theme, notification time, sensitivity profile |

### 9.2 Navigation map

```
Splash
  |
  v
Home <---------> Search ---> (select city) ---> Home
  |
  +--> Details
  +--> Activity Planner
  +--> Air Quality
  +--> Compare
  +--> Settings
```

Use a bottom navigation bar for Home / Search / Planner / Settings, or simple `Navigator.push`. Start simple; consider `go_router` later.

### 9.3 UI guidelines
- Use a consistent spacing scale (8, 16, 24).
- Define colors and text styles in the theme file, not inline.
- Support light and dark themes from the start.
- Keep text readable on top of dynamic backgrounds (use overlays or cards with some transparency).
- Add pull-to-refresh on Home.

---

## 10. Project Folder Structure

```
nimbus_ai/
|-- android/
|-- ios/
|-- assets/
|   |-- animations/            # Lottie JSON files
|   |-- icons/
|   |-- images/
|-- lib/
|   |-- main.dart              # entry point, provider setup
|   |-- app.dart               # MaterialApp, routes, theme
|   |-- core/
|   |   |-- constants/         # api_constants.dart, app_strings.dart
|   |   |-- theme/             # app_theme.dart, colors.dart
|   |   |-- utils/             # weather_code_mapper.dart, unit_converter.dart, date_utils.dart
|   |   |-- errors/            # app_exception.dart
|   |-- data/
|   |   |-- models/            # weather_data.dart, location_info.dart, air_quality.dart
|   |   |-- services/          # weather_api_service.dart, geocoding_service.dart,
|   |   |                      # air_quality_service.dart, location_service.dart
|   |   |-- local/             # cache_storage.dart, favorites_storage.dart, settings_storage.dart
|   |   |-- repositories/      # weather_repository.dart
|   |-- logic/
|   |   |-- providers/         # weather_provider.dart, search_provider.dart,
|   |   |                      # favorites_provider.dart, settings_provider.dart
|   |   |-- advisors/          # outfit_advisor.dart, activity_scorer.dart,
|   |                          # rain_alert_service.dart, aqi_advisor.dart
|   |-- notifications/         # notification_service.dart, briefing_task.dart
|   |-- presentation/
|       |-- screens/           # home/, search/, details/, planner/, aqi/, compare/, settings/
|       |-- widgets/           # weather_card.dart, hourly_tile.dart, daily_tile.dart,
|                              # aqi_gauge.dart, error_view.dart, loading_view.dart
|-- test/
|   |-- unit/                  # advisors, mappers, converters
|   |-- widget/                # screen state tests
|-- pubspec.yaml
|-- README.md
|-- .gitignore
```

---

## 11. Development Phases and Milestones

### Phase 0: Setup (Day 1)
- [ ] Install Flutter SDK, Android Studio / VS Code, and an emulator
- [ ] Run `flutter doctor` and fix any issues
- [ ] Create the project and run the default app
- [ ] Initialize Git and push to GitHub
- [ ] Create the folder structure

**Milestone M0:** Default app runs on emulator and is on GitHub.

### Phase 1: Foundation and Core Weather (Days 2-7)

| Day | Tasks |
|---|---|
| 2 | Learn widgets, layouts, Stateless vs Stateful; build small practice screens |
| 3 | Build the Home screen UI with dummy data |
| 4 | Create models; call the forecast API and print JSON; write `fromJson` |
| 5 | Build `WeatherApiService` and `WeatherRepository`; display real current weather |
| 6 | Add loading / error / offline states and `ErrorView`, `LoadingView` |
| 7 | Hourly strip; weather code mapper; refactor and commit |

**Milestone M1:** Home screen shows live current and hourly weather with error handling.

### Phase 2: Core Features (Days 8-14)

| Day | Tasks |
|---|---|
| 8 | 7-day forecast list |
| 9 | City search with geocoding API and recent searches |
| 10 | GPS location and permission flow |
| 11 | Settings screen (units, theme) saved with `shared_preferences` |
| 12 | Favorites using Hive |
| 13 | Offline cache with freshness rules and "last updated" label |
| 14 | Introduce `provider`; refactor logic out of widgets; commit |

**Milestone M2:** Fully functional standard weather app (all "Must" core features done).

### Phase 3: Exclusive Features (Days 15-20)

| Day | Tasks |
|---|---|
| 15 | Outfit Advisor + unit tests |
| 16 | Rain Alert banner + unit tests |
| 17 | Activity Score Planner screen |
| 18 | Air Quality service, provider, and screen |
| 19 | Dynamic backgrounds and animations |
| 20 | Morning Briefing notification (or City Compare if time is short) |

**Milestone M3:** Unique features working and visible on the Home screen.

### Phase 4: Polish and Release (Day 21 and buffer)
- [ ] Fix bugs and test edge cases
- [ ] App icon and splash screen
- [ ] Dark mode check on every screen
- [ ] Take screenshots
- [ ] Write README and project report
- [ ] Build release APK

**Milestone M4:** Submission-ready app and documentation.

### Priority if time runs short
Drop in this order: Voice Search, City Compare, Home widget, Morning Briefing, Activity Planner. **Never drop** error handling, offline cache, or the Outfit Advisor and Rain Alert.

---

## 12. Permissions and Configuration

### 12.1 Android permissions (`android/app/src/main/AndroidManifest.xml`)

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
<!-- Add microphone only if you build voice search -->
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
```

Some plugins need extra manifest entries or Gradle settings. Follow each package's README.

### 12.2 Runtime permission flow (location)

```
Tap "Use my location"
      |
      v
Is location service on? --no--> Ask user to enable GPS
      |yes
      v
Permission granted? --no--> Request --> denied? --> Show manual search
      |yes
      v
Get coordinates --> Reverse geocode --> Load weather
```

Always provide a manual-search fallback so the app works without location.

---

## 13. Error Handling Strategy

### 13.1 Custom error types

| Error | Cause | User message |
|---|---|---|
| `NetworkException` | No internet | "You're offline. Showing saved data." |
| `TimeoutException` | Slow server | "Taking too long. Please retry." |
| `ServerException` | Non-200 response | "Weather service is unavailable right now." |
| `ParseException` | Unexpected JSON | "Something went wrong reading the data." |
| `LocationException` | GPS / permission problem | "Couldn't get your location. Search for a city instead." |
| `CityNotFoundException` | Empty search result | "No city found. Try another name." |

### 13.2 Rules
- Services throw; repositories catch and convert; providers turn them into UI state.
- Never show raw exception text to users.
- Always give the user a next step (Retry, Search, Open settings).
- Log errors in debug mode for yourself.

---

## 14. Testing Plan

### 14.1 Unit tests (write alongside the feature)

| Target | Example cases |
|---|---|
| `OutfitAdvisor` | Cold + rain, hot + high UV, mild day |
| `RainAlertService` | Rain in next hour, none in 2 hours, boundary values |
| `ActivityScorer` | Ideal weather = high score; storm = low score |
| `AqiAdvisor` | Each AQI category boundary |
| `UnitConverter` | °C to °F, km/h to mph |
| `WeatherCodeMapper` | Known codes, unknown code fallback |
| Models | `fromJson` with valid and missing fields |

### 14.2 Widget tests
- Home shows a loader, then data, then an error view when the provider state changes.
- Settings toggles update the displayed units.

### 14.3 Manual test checklist
- [ ] Airplane mode (with cache and without cache)
- [ ] Deny location permission, then allow
- [ ] GPS off
- [ ] City not found
- [ ] Rotate screen
- [ ] Light and dark mode
- [ ] Large font size in device settings
- [ ] Small and large screens
- [ ] Low-end device or emulator performance
- [ ] Notification arrives at the chosen time (real device)

### 14.4 Bug log
Keep a table in your docs: Bug, Steps, Cause, Fix. It is useful material for your report's "Challenges faced" section.

---

## 15. Git and Version Control Plan

### 15.1 Branching (simple)
```
main            <- stable, working versions only
 |- dev         <- daily integration
     |- feature/home-ui
     |- feature/search
     |- feature/outfit-advisor
```

### 15.2 Commit habits
- Commit at least once per working session.
- Use clear messages: `feat: add hourly forecast strip`, `fix: handle empty search results`, `refactor: move API calls to repository`.
- Tag milestones: `v0.1-core`, `v0.2-features`, `v1.0`.

### 15.3 `.gitignore` essentials
Flutter's default `.gitignore` covers build output. Also make sure you never commit keystores, `key.properties`, or any secret keys.

---

## 16. Build and Release Checklist

- [ ] Remove debug prints and unused code
- [ ] Update app name, package ID, and version in `pubspec.yaml`
- [ ] Add app icon and splash screen
- [ ] Run `flutter analyze` and fix warnings
- [ ] Run `flutter test`
- [ ] Test on at least one real device
- [ ] Build release APK: `flutter build apk --release`
- [ ] Install the APK on a phone and run through the demo
- [ ] Save screenshots for the report
- [ ] Final README with setup instructions

---

## 17. Risks and Mitigation

| Risk | Impact | Mitigation |
|---|---|---|
| API changes or goes down | App shows no data | Cache data; keep parsing in one place; show clear error |
| Location permission denied | No GPS feature | Manual search fallback |
| Background tasks blocked by phone brand | Notification doesn't fire | Test on a real device; document the limitation |
| Scope creep | Project not finished | Strict priority list; finish core first |
| Falling behind schedule | Missed deadline | Buffer days; cut optional features |
| Environment setup problems | Lost days at the start | Run `flutter doctor` early; ask for help early |
| Confusing state bugs | Slow progress | Keep logic in providers; add small tests |

---

## 18. Definition of Done

The project is complete when:

1. All **Must** requirements (FR-01 to FR-06, FR-09, FR-10) work correctly.
2. The app never crashes in offline, denied-permission, or bad-data situations.
3. At least **three exclusive features** are working (Outfit Advisor, Rain Alert, plus one more).
4. Code follows the layered architecture with no API calls in widgets.
5. Unit tests exist for the advisors and utilities.
6. Release APK installs and runs on a real device.
7. The README and project report are written, with screenshots.
8. You can explain every layer and the data flow in your own words.

---

## 19. AI Features (Nimbus AI)

### 19.1 Rule-based vs AI: be clear in your report

| Feature | Type |
|---|---|
| Outfit Advisor, Rain Alert, Activity Score, AQI Advisor | **Rule-based** (if/else and scoring formulas) |
| Chat with Nimbus, AI Briefing, Explain-it | **AI (language model)** |
| Comfort Learning | **Adaptive logic** (simple learning from user feedback) |

Rule-based features stay in the app as the **offline fallback**. AI is an enhancement, not a dependency.

### 19.2 AI feature list

| ID | Feature | Description | Priority |
|---|---|---|---|
| AI-01 | **Chat with Nimbus** | Ask questions like "Should I run tomorrow evening?" and get answers based on real forecast data | Must (for AI scope) |
| AI-02 | **AI Daily Briefing** | A natural-language morning summary replacing the template text | Should |
| AI-03 | **Explain-it** | Tap any metric (UV, AQI, feels-like) for a plain-language explanation | Should |
| AI-04 | **Comfort Learning** | "Too hot / Just right / Too cold" feedback adjusts the outfit thresholds per user | Should |
| AI-05 | **Trip Packing Planner** | Enter a destination and dates, get a packing list from the forecast | Could |
| AI-06 | **Voice Chat** | Speech-to-text input for the chat | Could |

Recommended minimum: **AI-01, AI-02, AI-03, AI-04**.

### 19.3 AI architecture

The app must **never contain the AI API key**. Anyone can extract a key from an APK. Use a small backend proxy that holds the key.

```
+-------------------+        +----------------------+        +-----------------+
|   Flutter App     |  HTTPS |   Proxy Server       | HTTPS  |   AI API        |
|  (Nimbus AI)      +------->+ (Node/serverless)    +------->+ (LLM provider)  |
|                   |        |  - holds API key     |        |                 |
|  - builds context |<-------+  - rate limits       |<-------+                 |
|  - shows answer   |        |  - validates input   |        |                 |
+-------------------+        +----------------------+        +-----------------+
```

**Where AI fits in the existing layers**

```
Presentation:  ChatScreen, ExplainSheet, BriefingCard
Logic:         ChatProvider, ComfortProfileProvider
               WeatherContextBuilder  (turns WeatherData into prompt text)
Data:          AiChatService (HTTP client to your proxy)
               ComfortStorage (Hive, stores feedback)
Fallback:      Rule-based advisors (used when offline or AI fails)
```

### 19.4 Request flow: Chat with Nimbus

```
User types a question
        |
        v
ChatProvider.send(question)
        |
        v
WeatherContextBuilder.build(currentWeatherData, settings, comfortProfile)
        |  -> compact text: location, current, next 24h, next 3 days, AQI, preferences
        v
AiChatService.ask(question, context, history)
        |
        v
Proxy -> AI API (system instructions + context + question)
        |
        v
Answer returned -> ChatProvider adds message -> UI updates
        |
   (on failure) -> show friendly error + offer rule-based tip
```

### 19.5 Prompt design

**System instructions (kept on the proxy so users cannot alter them):**

```
You are Nimbus, a friendly weather assistant inside a mobile app.
Answer ONLY using the weather data provided in the context.
If the answer is not in the data, say you don't have that information.
Keep answers short (2 to 4 sentences) and practical.
Never give safety-critical advice (for example, whether to evacuate or
travel during a severe storm); suggest checking official local alerts instead.
Use the user's units. Do not invent numbers.
```

**Context block (built by the app):**

```
Location: Bengaluru, India (timezone Asia/Kolkata)
Now: 27°C (feels 29°C), humidity 70%, wind 12 km/h, UV 6
Next 12 hours: 4 PM 28°C rain 70%; 5 PM 26°C rain 80%; ...
Next 3 days: Sat 24-30°C rain 40%; Sun 23-29°C rain 60%; ...
Air quality: US AQI 82 (Moderate)
User prefs: units metric; feels cold easily; commutes by bike
```

**Tips**
- Send **compact** context (key hours and days only), not the full JSON. It is cheaper and faster.
- Keep the last few messages as history so follow-up questions work.
- Limit the question length (for example, 300 characters) in the app and the proxy.

### 19.6 Starter code (Flutter)

> Package names and APIs can change. Add dependencies with `flutter pub add dio provider` and check their docs.

**`lib/data/models/chat_message.dart`**

```dart
enum ChatRole { user, assistant }

class ChatMessage {
  final ChatRole role;
  final String text;
  final DateTime time;

  ChatMessage({required this.role, required this.text, DateTime? time})
      : time = time ?? DateTime.now();

  Map<String, String> toJson() => {
        'role': role == ChatRole.user ? 'user' : 'assistant',
        'content': text,
      };
}
```

**`lib/logic/advisors/weather_context_builder.dart`**

```dart
import '../../data/models/weather_data.dart';

class WeatherContextBuilder {
  static String build(WeatherData w, {String? userPrefs}) {
    final c = w.current;
    final buffer = StringBuffer()
      ..writeln('Location: ${w.location.name}, ${w.location.country} '
          '(timezone ${w.location.timezone})')
      ..writeln('Now: ${c.temperature.round()}°C '
          '(feels ${c.feelsLike.round()}°C), humidity ${c.humidity}%, '
          'wind ${c.windSpeed.round()} km/h, UV ${c.uvIndex}');

    buffer.writeln('Next 12 hours:');
    for (final h in w.hourly.take(12)) {
      buffer.writeln('  ${h.time.hour}:00 ${h.temperature.round()}°C '
          'rain ${h.rainProbability}%');
    }

    buffer.writeln('Next 3 days:');
    for (final d in w.daily.take(3)) {
      buffer.writeln('  ${d.date.month}/${d.date.day}: '
          '${d.tempMin.round()}-${d.tempMax.round()}°C '
          'rain ${d.rainProbability}%');
    }

    if (userPrefs != null && userPrefs.isNotEmpty) {
      buffer.writeln('User preferences: $userPrefs');
    }
    return buffer.toString();
  }
}
```

**`lib/data/services/ai_chat_service.dart`**

```dart
import 'package:dio/dio.dart';
import '../models/chat_message.dart';

class AiException implements Exception {
  final String message;
  AiException(this.message);
  @override
  String toString() => message;
}

class AiChatService {
  final Dio _dio;
  // Your own proxy URL, NOT the AI provider's URL, and NO API key here.
  static const String proxyUrl = 'https://your-proxy.example.com/chat';

  AiChatService([Dio? dio])
      : _dio = dio ??
            Dio(BaseOptions(
              connectTimeout: const Duration(seconds: 10),
              receiveTimeout: const Duration(seconds: 30),
            ));

  Future<String> ask({
    required String question,
    required String weatherContext,
    List<ChatMessage> history = const [],
  }) async {
    try {
      final response = await _dio.post(proxyUrl, data: {
        'question': question,
        'context': weatherContext,
        'history': history.map((m) => m.toJson()).toList(),
      });
      final answer = response.data['answer'];
      if (answer is! String || answer.trim().isEmpty) {
        throw AiException('Nimbus returned an empty answer.');
      }
      return answer.trim();
    } on DioException catch (e) {
      if (e.type == DioExceptionType.connectionError) {
        throw AiException('You appear to be offline.');
      }
      if (e.type == DioExceptionType.receiveTimeout ||
          e.type == DioExceptionType.connectionTimeout) {
        throw AiException('Nimbus is taking too long. Please try again.');
      }
      if (e.response?.statusCode == 429) {
        throw AiException('Too many questions. Please wait a moment.');
      }
      throw AiException('Nimbus is unavailable right now.');
    }
  }
}
```

**`lib/logic/providers/chat_provider.dart`**

```dart
import 'package:flutter/foundation.dart';
import '../../data/models/chat_message.dart';
import '../../data/models/weather_data.dart';
import '../../data/services/ai_chat_service.dart';
import '../advisors/weather_context_builder.dart';

class ChatProvider extends ChangeNotifier {
  final AiChatService _service;
  ChatProvider(this._service);

  final List<ChatMessage> _messages = [];
  bool _loading = false;
  String? _error;

  List<ChatMessage> get messages => List.unmodifiable(_messages);
  bool get isLoading => _loading;
  String? get error => _error;

  Future<void> send(String question, WeatherData weather,
      {String? userPrefs}) async {
    final text = question.trim();
    if (text.isEmpty || _loading) return;
    if (text.length > 300) {
      _error = 'Please keep your question under 300 characters.';
      notifyListeners();
      return;
    }

    _error = null;
    _messages.add(ChatMessage(role: ChatRole.user, text: text));
    _loading = true;
    notifyListeners();

    try {
      final context =
          WeatherContextBuilder.build(weather, userPrefs: userPrefs);
      // Send only the last few messages as history.
      final history = _messages.length > 7
          ? _messages.sublist(_messages.length - 7, _messages.length - 1)
          : _messages.sublist(0, _messages.length - 1);

      final answer = await _service.ask(
        question: text,
        weatherContext: context,
        history: history,
      );
      _messages.add(ChatMessage(role: ChatRole.assistant, text: answer));
    } on AiException catch (e) {
      _error = e.message;
    } catch (_) {
      _error = 'Something went wrong. Please try again.';
    } finally {
      _loading = false;
      notifyListeners();
    }
  }
}
```

**Minimal proxy (Node.js + Express), for your own server or a serverless function**

```js
// server.js  (run with: node server.js)
// Put your key in an environment variable, never in code or Git:
//   AI_API_KEY=...   AI_MODEL=<current model name from the provider's docs>
import express from "express";

const app = express();
app.use(express.json({ limit: "20kb" }));

const SYSTEM = `You are Nimbus, a friendly weather assistant inside a mobile app.
Answer ONLY using the weather data provided in the context.
If the answer is not in the data, say you don't have that information.
Keep answers short (2 to 4 sentences) and practical.
Never give safety-critical advice; suggest official local alerts instead.
Do not invent numbers.`;

// Very simple rate limit: 10 requests per minute per IP
const hits = new Map();
function limited(ip) {
  const now = Date.now();
  const recent = (hits.get(ip) || []).filter((t) => now - t < 60_000);
  recent.push(now);
  hits.set(ip, recent);
  return recent.length > 10;
}

app.post("/chat", async (req, res) => {
  if (limited(req.ip)) return res.status(429).json({ error: "rate_limited" });

  const { question, context, history = [] } = req.body || {};
  if (typeof question !== "string" || question.length > 300) {
    return res.status(400).json({ error: "bad_question" });
  }

  try {
    const r = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: {
        "content-type": "application/json",
        "x-api-key": process.env.AI_API_KEY,
        "anthropic-version": "2023-06-01",
      },
      body: JSON.stringify({
        model: process.env.AI_MODEL,
        max_tokens: 300,
        system: `${SYSTEM}\n\nWeather data:\n${String(context).slice(0, 4000)}`,
        messages: [...history.slice(-6), { role: "user", content: question }],
      }),
    });
    const data = await r.json();
    const answer = data?.content?.[0]?.text;
    if (!r.ok || !answer) return res.status(502).json({ error: "ai_failed" });
    res.json({ answer });
  } catch {
    res.status(502).json({ error: "ai_failed" });
  }
});

app.listen(3000, () => console.log("Proxy running on :3000"));
```

> The proxy example uses Anthropic's Messages API. Check the current docs for the correct model name and request format, and for the free-tier or pricing details of whichever provider you choose. Deploy options for the proxy include a small cloud function or a free-tier hosting service; verify current limits.

### 19.7 Comfort Learning (AI-04) design

**Idea:** the outfit thresholds start with defaults and shift based on user feedback.

```
Default thresholds:  cold < 10, cool 10-17, mild 18-27, hot >= 28

User taps feedback after following a suggestion:
   "Too cold" -> shift all thresholds +1°C (user feels cold more easily)
   "Too hot"  -> shift all thresholds -1°C
   "Just right" -> no change

Limit the total shift to a range of about -5 to +5°C.
Store the offset in Hive (ComfortStorage).
OutfitAdvisor uses: effectiveTemp = feelsLike - offset
```

This is a simple adaptive model. In your report, describe it as **personalization through feedback**, not deep learning.

### 19.8 Explain-it (AI-03) design

- Each metric card (UV, AQI, feels-like, humidity) has an info icon.
- On tap, open a bottom sheet and send a prompt like: "Explain UV index 6 in simple words and what to do about it today."
- **Cache explanations** by metric and value range, so repeated taps cost nothing and work offline.

### 19.9 AI Briefing (AI-02) design

- At the user's chosen time, the background task fetches the forecast, builds the context, and asks the AI for a 1 to 2 sentence summary.
- **If the AI call fails, send the template-based briefing instead** (the rule-based fallback), so the notification always arrives.

### 19.10 AI development schedule (add after Day 20)

| Day | Task |
|---|---|
| 22 | Set up the proxy locally; test it with a REST client; add the system prompt |
| 23 | `WeatherContextBuilder`, `AiChatService`, `ChatProvider` with unit tests on the context builder |
| 24 | Chat screen UI (message bubbles, input, loading indicator, error banner, suggestion chips) |
| 25 | Explain-it bottom sheet and caching |
| 26 | Comfort Learning (feedback buttons and offset logic) |
| 27 | AI Briefing with fallback; deploy the proxy; test on a real device |
| 28 | Polish, safety notices, update documentation and screenshots |

Suggestion chips for the chat screen: "What should I wear today?", "Will it rain this week?", "Best time for a walk?", "Is the air safe for exercise?".

### 19.11 Safety, privacy, and cost checklist

- [ ] No API key inside the app or the Git repository
- [ ] The proxy enforces rate limits and a maximum input length
- [ ] The system prompt lives on the proxy, not in the app
- [ ] Only weather data and basic preferences are sent (no personal identifiers)
- [ ] Location sent to the AI is the city name or rounded coordinates, not an exact position
- [ ] Disclaimer shown in the chat: "AI answers may be inaccurate. For severe weather, check official alerts."
- [ ] Every AI feature has a rule-based or friendly-error fallback
- [ ] You have checked the provider's current pricing, free-tier limits, and terms
- [ ] Privacy note added to the README describing what data is sent

### 19.12 Testing the AI features

| Test | What to check |
|---|---|
| Context builder (unit) | Output includes location, current values, and the right number of hours and days |
| Chat provider (unit, mock service) | Adds user message, shows loading, handles success and error |
| Empty or long input | Rejected with a clear message |
| Offline | Friendly error, rule-based tips still available |
| Rate limit (429) | "Please wait" message, no crash |
| Prompt behavior (manual) | Ask something outside the data, such as "Will it snow in Mumbai next year?" and confirm Nimbus declines to invent an answer |
| Follow-up questions | History is used ("What about Sunday?") |

### 19.13 Updated pitch

> **Nimbus AI** is a weather app you can talk to. It combines live forecasts with smart advisors and an AI assistant that tells you what to wear, when to go out, and what to expect, and it keeps working offline with built-in rule-based guidance.

---

*Build it in this order: make it work, make it correct, then make it beautiful.*
