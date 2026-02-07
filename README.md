# Strike Rate

**A habit tracker that rewards consistency, not perfection.**

Strike Rate ditches the fragile "streak" model and replaces it with a **batting average** — a percentage-based consistency score that doesn't punish you for one bad day. Miss Monday? Your 85% drops to 80%, not back to zero.

---

## How It Works

| Metric | Formula | What It Measures |
|--------|---------|-----------------|
| **Daily Strike Rate** | (Completed Tasks / Total Tasks) × 100 | Today's performance |
| **Consistency Score** | Average of all daily strike rates | Lifetime discipline |

> **Why average percentages instead of raw totals?** A day with 1/1 tasks completed (100%) and a day with 8/10 (80%) average to 90%. This ensures "light days" and "heavy days" carry equal weight — measuring *consistency of effort*, not volume.

---

##  Features

- **Local-First** — All data stored on-device via Room. No account required. Works 100% offline.
- **Smart Recurrence** — Daily, Weekly (pick days), Monthly (with overflow handling for short months), and One-Off tasks.
- **Calendar Scout** — Fetches Google Calendar events as suggestions you can optionally track.
- **Analytics** — Weekly/monthly trend charts and a 90-day color-coded heatmap.
- **Reminders** — Configurable daily notification with yesterday's score and today's task count.
- **Dark Mode** — Material 3 theming with system/light/dark toggle.
- **Accessible** — TalkBack support, 48dp touch targets, WCAG 2.1 AA contrast.

---

## 🏗️ Architecture

```
MVVM + Clean Architecture │ Kotlin │ Jetpack Compose │ Material 3
```

```
com.strikerate
├── data/
│   ├── local/          Room DB, DAOs, TypeConverters
│   ├── remote/         Google Calendar API
│   └── repository/     Repository implementations
├── domain/
│   ├── model/          Entities, RecurrenceRule (sealed class), Enums
│   ├── repository/     Repository interfaces
│   └── usecase/        StrikeRateUseCase, business logic
├── ui/
│   ├── dashboard/      "The Arena" — progress ring + task list
│   ├── addtask/        Create/Edit with recurrence selector
│   ├── history/        Trend charts + calendar heatmap
│   ├── settings/       Preferences, account, data export
│   └── theme/          Material 3 color scheme + typography
├── worker/             DailySchedulerWorker, DailyReminderWorker
└── di/                 Hilt modules
```

### Tech Stack

| Layer | Technology |
|-------|-----------|
| DI | Hilt |
| Async | Kotlin Coroutines & Flow |
| Database | Room (SQLite) |
| Background | WorkManager |
| Navigation | Navigation Compose |
| Calendar API | Google Calendar v3 + Credential Manager |
| Charting | Vico / MPAndroidChart |

### Data Model

The core insight is separating **Blueprints** (habit definitions) from **Instances** (daily occurrences):

```
TaskBlueprint (1) ──→ (many) TaskInstance
  "Gym, Mon/Wed/Fri"         "Gym, 2026-02-09, ✅"
                              "Gym, 2026-02-11, ☐"
```

Recurrence rules use a **sealed class** to make illegal states unrepresentable:

```kotlin
sealed class RecurrenceRule {
    object Daily : RecurrenceRule()
    data class Weekly(val days: Set<DayOfWeek>) : RecurrenceRule()
    data class Monthly(
        val dayOfMonth: Int,
        val overflowPolicy: OverflowPolicy = OverflowPolicy.LAST_DAY
    ) : RecurrenceRule()
    data class OneOff(val date: LocalDate) : RecurrenceRule()
}
```

---

## Getting Started

### Prerequisites

- Android Studio Ladybug (2024.2.1) or later
- JDK 17
- Min SDK 26 (Android 8.0), Target SDK 35

### Build & Run

```bash
git clone https://github.com/YOUR_USERNAME/strike-rate.git
cd strike-rate
```

1. Open in Android Studio
2. Sync Gradle
3. Run on emulator or device (API 26+)

### Google Calendar Setup (Optional)

Calendar integration requires a Google Cloud project:

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create an OAuth 2.0 Client ID (Android type) with your package name + SHA-1
3. Enable the **Google Calendar API**
4. Add your `GOOGLE_CLIENT_ID` to `local.properties`:

```properties
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
```

> The app works fully without Calendar integration. This is only needed for the "Calendar Scout" feature.

---

## Implementation Phases

| Phase | Name | Status | Scope |
|-------|------|--------|-------|
| 1 | **Foundation** | 🔲 | Hilt, Room DB, entities, DAOs, TypeConverters, RecurrenceRule |
| 2 | **Core Loop** | 🔲 | DashboardViewModel, Compose UI, progress ring, checkbox logic |
| 3 | **Engine** | 🔲 | WorkManager recurrence with backfill, Create/Edit Task screen |
| 4 | **Analytics** | 🔲 | History screen, trend charts, calendar heatmap |
| 5 | **Calendar** | 🔲 | Google OAuth (Credential Manager), CalendarRepository, Calendar Scout |
| 6 | **Notifications** | 🔲 | Daily reminder worker, notification channel, permission flow |
| 7 | **Polish** | 🔲 | Theming, Settings, blueprint management, data export, accessibility |

---

## Testing

```bash
# Unit tests
./gradlew testDebugUnitTest

# Instrumented tests (requires device/emulator)
./gradlew connectedDebugAndroidTest
```

### Test Coverage Priorities

| Component | What's Tested | Framework |
|-----------|--------------|-----------|
| `StrikeRateUseCase` | Calculation edge cases (0 tasks, partial, all complete) | JUnit 5 + MockK |
| `DailySchedulerWorker` | Backfill logic, monthly overflow, duplicate detection | WorkManager Testing |
| DAOs | Query correctness, DatabaseView accuracy | Room Testing |
| ViewModels | State transitions, UiState mapping | Turbine (Flow) |
| UI | Screen rendering, navigation, interactions | Compose UI Testing |

---

## Design Decisions

**Why "Consistency Score" over "Completion Rate"?**
Averaging daily percentages means a day where you complete 1/1 tasks weighs the same as a day where you complete 8/10. This rewards showing up consistently, not just doing more. A raw completion rate can be added as a secondary metric if needed.

**Why sealed class for RecurrenceRule?**
The v1 design used an overloaded `targetDays: List<DayOfWeek>` field for all recurrence types, which made monthly (needs an Int) and one-off (needs a LocalDate) awkward. The sealed class enforces that each variant only carries relevant data.

**Why DatabaseView instead of a stats table?**
`DailyStrikeStats` is computed on the fly from `TaskInstance` via a Room `@DatabaseView`. This eliminates sync issues between the stats cache and source data. If performance degrades at 1000+ days, it can be materialized.

**Why Credential Manager over GoogleSignInClient?**
`GoogleSignInClient` / `GoogleSignInOptions` are deprecated as of 2023. The Credential Manager API (`androidx.credentials`) is the modern replacement and handles account selection natively.

---

##  Contributing

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Please read the [TDD (Technical Design Document)](docs/StrikeRate_TDD_v2.docx) before contributing to understand the architecture and constraints.

---

##  License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  <i>Stop counting streaks. Start measuring consistency.</i>
</p>
