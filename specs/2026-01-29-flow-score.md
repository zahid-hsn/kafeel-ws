# Feature: Flow Score - Personal Productivity Intelligence

## Summary
A gamified productivity system with streaks, achievements, self-comparison, calendar-aware meeting detection, and dual-personality coaching modes.

## Trigger
- Auto-starts when tracking is enabled
- Runs continuously in background
- Updates in real-time as apps change

## Expected Result
- Live score in menu bar (0-100, color-coded)
- Streak tracking with shields
- XP and leveling system
- Achievement gallery
- Daily summary with personal comparisons
- Encouraging OR brutally honest feedback

## Edge Cases
- Meetings vs distractions: Calendar events + video call detection
- Streak protection: Shields earned at milestones
- Bad days: Compare to personal average, not absolute

---

## Technical Design

### New SwiftData Models

#### DailyScore
```swift
@Model
public final class DailyScore {
    @Attribute(.unique) public var dateKey: String  // "YYYY-MM-DD"
    public var date: Date
    public var focusScore: Double  // 0-100
    public var productiveSeconds: Int
    public var distractingSeconds: Int
    public var neutralSeconds: Int
    public var meetingSeconds: Int
    public var deepWorkSeconds: Int  // Uninterrupted 25+ min blocks
    public var switchCount: Int
    public var peakHour: Int  // 0-23
    public var xpEarned: Int
    public var streakDay: Int  // Which day of streak (0 if broken)
}
```

#### Streak
```swift
@Model
public final class Streak {
    public var currentStreak: Int
    public var longestStreak: Int
    public var streakStartDate: Date?
    public var lastQualifyingDate: Date?
    public var streakShields: Int
    public var usedShieldDates: [Date]
    public var threshold: Double  // Default 60.0
}
```

#### Achievement
```swift
public enum AchievementType: String, Codable, CaseIterable {
    case earlyBird, nightOwl, marathon, streakStarter, streakMaster
    case streakLegend, meetingSurvivor, comebackKid, centurion
    case consistent, deepDiver, weekendWarrior
}

@Model
public final class Achievement {
    @Attribute(.unique) public var typeRaw: String
    public var unlockedDate: Date?
    public var isUnlocked: Bool
    public var progress: Double
    public var timesAchieved: Int
}
```

#### UserProfile
```swift
public enum PersonalityMode: String, Codable {
    case encouraging
    case brutallyHonest
}

@Model
public final class UserProfile {
    public var totalXP: Int
    public var level: Int
    public var personalityMode: PersonalityMode
    public var dailyAverageScore: Double  // Rolling 30-day
    public var weeklyAverages: [Int: Double]  // Day of week -> avg
    public var joinDate: Date
    public var totalFocusedHours: Double
}
```

#### PersonalRecord
```swift
public enum RecordType: String, Codable {
    case bestDayScore, bestWeekAverage, longestStreak
    case mostProductiveHour, bestDayOfWeek, longestFocusSession
}

@Model
public final class PersonalRecord {
    @Attribute(.unique) public var recordTypeRaw: String
    public var value: Double
    public var achievedDate: Date
    public var metadata: String?
}
```

### New Services

#### FlowScoreEngine
Central orchestrator integrating with existing FocusScoreCalculator.
- `calculateRealTimeScore()` - Live score with meeting adjustments
- `finalizeDailyScore()` - End-of-day aggregation
- `generateDailySummary()` - Stats for summary view
- `onActivityChanged()` - Hook for ActivityMonitor

#### StreakService
- `getCurrentStreak()` / `updateStreakForToday(score:)`
- `useStreakShield()` / `awardStreakShield()`
- `isStreakAtRisk(currentScore:)`

#### AchievementService
- `checkAllAchievements(for:)` - Check after daily finalization
- Individual checks: earlyBird, marathon, meetingSurvivor, etc.

#### PersonalityService
- `getMessage(for:mode:profile:)` - Context-aware messages
- Two message sets: encouraging vs brutally honest

#### MeetingDetector
- Detect: Google Meet via browser window title
- Calendar integration via EventKit

### UI Components

| Component | Purpose |
|-----------|---------|
| StreakCard | Flame icon, current streak, shields |
| LevelProgressBar | XP bar with tier badge |
| AchievementGalleryView | Grid of achievements |
| DailySummaryView | End-of-day stats popup |
| PersonalitySettingsView | Toggle encouraging/brutal |

### Menu Bar Changes
- Show live score number (e.g., "78")
- Color: green (80+), yellow (60-79), red (<60)
- Streak flame when active

---

## Implementation Steps

### Phase 1: Models
1. [ ] Create DailyScore model + tests
2. [ ] Create Streak model + tests
3. [ ] Create Achievement model + tests
4. [ ] Create UserProfile model + tests
5. [ ] Create PersonalRecord model + tests
6. [ ] Update PersistenceService schema

### Phase 2: Services
7. [ ] Implement StreakService + tests
8. [ ] Implement PersonalRecordService + tests
9. [ ] Implement FlowScoreEngine + tests
10. [ ] Implement MeetingDetector + tests
11. [ ] Implement AchievementService + tests
12. [ ] Implement PersonalityService + tests

### Phase 3: UI
13. [ ] Create StreakCard component
14. [ ] Create LevelProgressBar component
15. [ ] Create AchievementGalleryView
16. [ ] Create DailySummaryView
17. [ ] Enhance DashboardView
18. [ ] Update MenuBarManager with live score

### Phase 4: Integration
19. [ ] Hook FlowScoreEngine into ActivityMonitor
20. [ ] Add notifications for achievements/streaks
21. [ ] End-to-end testing
22. [ ] Create PR

---

## XP System

| Action | XP |
|--------|-----|
| Per focused hour | 100 (×1.0-2.0 streak multiplier) |
| Daily score ≥60 | +200 |
| Daily score ≥80 | +500 |
| Achievement unlock | 100-1000 |
| Streak milestone | 500-5000 |
| Personal record | +300 |

## Level Tiers

| Tier | Levels | Total XP |
|------|--------|----------|
| Apprentice | 1-10 | 0-10K |
| Journeyman | 11-25 | 10K-75K |
| Expert | 26-50 | 75K-350K |
| Master | 51+ | 350K+ |

---

## Tests

### Unit Tests
- `DailyScoreTests.swift` - Model validation
- `StreakServiceTests.swift` - Streak logic, shields
- `AchievementServiceTests.swift` - All achievement conditions
- `PersonalityServiceTests.swift` - Message generation
- `FlowScoreEngineTests.swift` - Score calculation, XP
- `MeetingDetectorTests.swift` - Google Meet window detection

### Integration Tests
- Full day simulation
- Streak across multiple days
- Achievement unlock flow

---

## Meeting Detection

**Google Meet Detection (via browser window title):**
- Chrome: `com.google.Chrome`
- Safari: `com.apple.Safari`
- Firefox: `org.mozilla.firefox`
- Arc: `company.thebrowser.Browser`

**Window Title Patterns:**
- `"Meet -"` → Google Meet active
- `"meet.google.com"` → Google Meet URL in tab

**Calendar Integration:**
- EventKit for macOS Calendar events
- Match detected meetings to calendar for context
