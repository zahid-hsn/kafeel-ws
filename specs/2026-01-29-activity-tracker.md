# Feature: macOS Activity Tracker

## Summary
Native macOS app that monitors app usage, calculates focus/productivity scores, and displays analytics dashboards.

## Trigger
- App launches at startup (or manual launch)
- Runs in background, monitoring active applications
- User opens dashboard to view analytics

## Expected Result
- Real-time app usage tracking
- Dashboard with:
  - App usage chart (bar chart showing time per app)
  - Focus score (0-100 based on productive vs distracting time)
  - Time filters (Day/Week/Year views)
  - Browsing activity summary
  - Weekly hours total
- Settings panel with:
  - Pause/resume tracking
  - App categorization (productive/distracting/neutral)
  - Password-protected changes
- Persistent local storage (survives app restart)
- Password-protected data deletion

## Edge Cases
- Screen lock/unlock → pause/resume tracking
- App crash → data persists (SwiftData auto-saves)
- Unknown app → defaults to "neutral" category
- Very short app usage (<2 seconds) → ignored to reduce noise
- Multiple monitors → tracks frontmost app only
- Full-screen apps → tracked normally via NSWorkspace

## Technical Design

### macOS Client Changes

**New Models (SwiftData):**
```swift
@Model ActivityLog {
    id, appBundleIdentifier, appName, windowTitle?,
    startTime, endTime?, durationSeconds
}

@Model AppCategory {
    bundleIdentifier, appName, category (enum),
    isDefault, lastModified
}

@Model AppSettings {
    isTrackingEnabled, defaultCategoryForNewApps
}
```

**New Services:**
- `ActivityMonitor` - NSWorkspace.didActivateApplicationNotification
- `PersistenceService` - SwiftData CRUD
- `FocusScoreCalculator` - Weighted scoring algorithm

**New Views:**
- `DashboardView` - Main analytics display
- `AppUsageChart` - Swift Charts bar chart
- `FocusScoreCard` - Circular score indicator
- `SettingsView` - Preferences and category management
- `SecureActionButton` - LocalAuthentication wrapper

### API Changes (Phase 4 - Optional)
- Endpoint: `POST /activity/logs/batch`
- Request: `[{ app_bundle_identifier, app_name, start_time, duration_seconds }]`
- Response: `{ synced_count: int, last_sync: datetime }`

### Database Changes (Phase 4 - Optional)
- New tables: activity_logs, app_categories, daily_summaries

## Implementation Plan
1. [x] Create spec (this document)
2. [ ] Write SwiftData model tests (Red)
3. [ ] Implement SwiftData models (Green)
4. [ ] Write ActivityMonitor tests (Red)
5. [ ] Implement ActivityMonitor (Green)
6. [ ] Write FocusScoreCalculator tests (Red)
7. [ ] Implement FocusScoreCalculator (Green)
8. [ ] Build DashboardView with charts
9. [ ] Build SettingsView with categories
10. [ ] Add password protection (LocalAuthentication)
11. [ ] Integration test full workflow
12. [ ] Commit and create PR
