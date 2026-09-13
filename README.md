# NutriTrack

A cross-platform nutrition and meal-tracking app built with Flutter for
ICT725 Assessment 4. Targets health-conscious adults aged 25–45, follows
the Assessment 3 high-fidelity prototype (teal / navy / white, high
contrast, ≥48dp touch targets), and implements the 7 screens and
functional requirements from the assignment brief.

Figma prototype: https://misty-diary-02745217.figma.site

## Tech stack

- **Flutter (Dart)**, Material 3
- **State management:** Provider
- **Navigation:** go_router
- **Local storage:** shared_preferences (offline-first mock backend —
  see "Backend notes" below)
- **Charts:** fl_chart
- **Barcode scanning:** mobile_scanner

## Getting the project running

This zip contains only the Flutter/Dart source (`lib/`), assets,
`pubspec.yaml`, and tests — not the native `android/` and `ios/`
platform folders, since those are large, auto-generated, and need to
match the exact Flutter SDK version installed on your machine. Generate
them locally in one step:

```bash
# 1. Unzip, then from the project root:
flutter create . --platforms=android,ios --org com.nutritrack --project-name nutritrack

# 2. Install dependencies
flutter pub get

# 3. Run on the Android Studio Emulator (start the emulator first)
flutter run
```

`flutter create .` will not overwrite the existing `lib/`, `pubspec.yaml`,
or `test/` files — it only adds the missing `android/`, `ios/`, and
related platform scaffolding.

### Run tests

```bash
flutter test
```

## Backend notes (Firebase vs local)

The assignment allows **"Firebase (Auth, Firestore, Storage) OR local
SQLite/Hive for offline-first."** This build uses the local/offline-first
option: `AuthService` and `DatabaseService` (in `lib/services/`) persist
users, meals, goals, and recent foods to `shared_preferences`, structured
so they can be swapped for `firebase_auth` / `cloud_firestore` later —
the providers (`lib/providers/`) never talk to storage directly, only to
these two service classes. This was the pragmatic choice for a build
that needs to run immediately in the emulator without a Firebase
project needing to be provisioned with API keys first.

To switch to Firebase: add `firebase_core`, `firebase_auth`, and
`cloud_firestore` to `pubspec.yaml`, run `flutterfire configure`, and
reimplement the methods in `AuthService`/`DatabaseService` against those
SDKs — the rest of the app is unaffected.

## Implemented functionality (major features)

1. **Meal Logging + Food Search** (`meal_logging_screen.dart`,
   `food_search_screen.dart`, `barcode_scan_screen.dart`): search or
   manually enter a food, pick a meal type and portion size, and save —
   with barcode scanning as the secondary input method added after user
   testing.
2. **Progress/Statistics** (`progress_screen.dart`,
   `widgets/progress_chart.dart`): fl_chart line chart of calorie trend
   and bar chart of average macros, with a 7/30/90-day range selector.

Also fully implemented: Login/Signup with validation
(`login_screen.dart`, `signup_screen.dart`), Dashboard with nutrition
summary rings and Recent Meals one-tap logging (`dashboard_screen.dart`),
Goal Setting with sliders (`goal_setting_screen.dart`), and Profile with
health info, diet preferences, and settings (`profile_screen.dart`).

## Not implemented / simplified

- **Live nutrition API for barcode lookups**: `ApiService.lookupBarcode`
  is wired to Open Food Facts but untested against a real network in
  this environment — needs a live device/emulator with internet access
  to verify end to end.
- **Firebase Auth/Firestore**: intentionally deferred in favor of the
  local-storage option (see "Backend notes"), since setting up a live
  Firebase project needs your own Google account and API keys.
- **Push notifications**: the Profile screen's "Notifications" toggle
  is stored as a preference but doesn't yet trigger local notifications.

## Project structure

```
lib/
├── main.dart              # entry point, provider wiring, session sync
├── app_router.dart         # go_router routes + auth redirect logic
├── models/                 # AppUser, FoodItem, Meal, NutritionGoal
├── providers/               # AuthProvider, MealProvider, GoalProvider
├── screens/                 # 7 screens + shell + barcode scanner
├── widgets/                  # NutritionCard, MealListItem, charts, buttons
├── services/                  # AuthService, DatabaseService, ApiService
└── utils/                      # theme, constants, validators/helpers
```

## Accessibility & responsiveness

- All interactive elements use the theme's 48dp minimum touch target.
- Text theme is built for high contrast against the teal/navy/white
  palette.
- Screens use `LayoutBuilder`/`MediaQuery`-aware layouts (e.g. the
  Dashboard's nutrition grid switches from 2 to 4 columns on wide
  screens) rather than fixed pixel sizing.
