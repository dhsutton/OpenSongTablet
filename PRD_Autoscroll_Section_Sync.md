# Product Requirements Document: Autoscroll Section Synchronization

## Document Information
- **Feature Name**: Autoscroll Section Synchronization
- **Version**: 1.0
- **Date**: 2025-11-06
- **Status**: Draft for Review

---

## 1. Executive Summary

This feature enhances the autoscroll functionality to automatically advance through song sections on the connected display (Android Presentation/Secondary Display) in synchronization with the scrolling content on the primary display. When a section scrolls off the visible area of the primary display, the connected display automatically advances to the next section, providing a synchronized viewing experience for audiences while maintaining full manual control for the performer.

---

## 2. Problem Statement

Currently, when using autoscroll mode on the primary display, the connected display (projector/secondary screen) shows the full song content and does not automatically advance through individual sections. This creates a disconnect:

- **For the performer**: They cannot easily track which section is currently visible on the connected display while autoscroll is running
- **For the audience**: They see the entire song at once rather than following section by section as the performer progresses
- **Manual control**: The performer must manually click sections while managing autoscroll, creating unnecessary cognitive load

### User Pain Points:
1. No visual feedback on which section is "active" during autoscroll
2. Connected display doesn't follow the scrolling progress automatically
3. Cannot pause autoscroll and have the connected display pause in sync
4. End-of-song sections remain invisible on connected display if they never scroll off the primary screen

---

## 3. Goals and Objectives

### Primary Goals:
1. **Automatic Section Advancement**: Sections on the connected display advance automatically as content scrolls on the primary display
2. **Visual Feedback**: Current section is highlighted on the primary display during autoscroll
3. **Manual Override**: User can manually navigate sections during autoscroll, with connected display tracking changes
4. **Pause Synchronization**: Pausing autoscroll also pauses section advancement on connected display
5. **End-of-Song Handling**: Sections that never scroll off screen still advance on the connected display

### Success Metrics:
- Section advancement on connected display occurs within 100ms of trigger condition
- No missed section transitions during normal autoscroll operation
- Manual section changes immediately reflected on connected display
- User can configure advancement timing to their preference

---

## 4. User Stories

### User Story 1: Basic Section Synchronization
**As a** worship leader using autoscroll
**I want** the connected display to automatically show the current section as it appears on my primary display
**So that** the audience sees sections in sync with my performance without manual intervention

**Acceptance Criteria:**
- When autoscroll is running, connected display advances to next section automatically
- Section advancement occurs when current section content scrolls off the primary display
- No timer-based advancement (must be position-based)

---

### User Story 2: Visual Feedback
**As a** performer using autoscroll
**I want** to see which section is currently displayed on the connected display
**So that** I know what the audience is seeing at any moment

**Acceptance Criteria:**
- Current section is highlighted in the section list on primary display
- Highlighting updates automatically as sections advance during autoscroll
- Highlight color/alpha follows existing app theming conventions

---

### User Story 3: Manual Control Override
**As a** performer running autoscroll
**I want** to manually skip forward or backward through sections while scrolling continues
**So that** I can adjust the connected display presentation without stopping autoscroll

**Acceptance Criteria:**
- Clicking next/previous section buttons works during autoscroll
- Connected display immediately shows the manually selected section
- Automatic advancement resumes from the manually selected section
- Manual section change does not affect autoscroll speed or position

---

### User Story 4: Pause Synchronization
**As a** performer
**I want** section advancement to pause when I pause autoscroll
**So that** the connected display stays in sync during pauses

**Acceptance Criteria:**
- Pausing autoscroll pauses automatic section advancement
- Connected display shows the current section until autoscroll resumes
- Resuming autoscroll resumes automatic section advancement
- Manual section navigation still works while paused

---

### User Story 5: End-of-Song Handling
**As a** performer
**I want** all remaining sections to display on the connected screen even when they don't scroll off the primary display
**So that** the audience sees all sections of the song

**Acceptance Criteria:**
- When scrolling reaches bottom of song, remaining sections still advance on connected display
- Advancement timing for end sections is configurable
- Final section remains displayed until song ends or user navigates away

---

### User Story 6: Configurable Advancement Timing
**As a** user
**I want** to configure when section advancement occurs relative to content scrolling off screen
**So that** I can adjust the timing to match my performance style

**Acceptance Criteria:**
- Setting available to adjust advancement threshold (e.g., "50% scrolled off", "fully scrolled off", "25% of next section visible")
- Setting persists across app sessions
- Default value provides sensible out-of-box experience

---

## 5. Functional Requirements

### 5.1 Core Functionality

#### FR-1: Scroll Position Monitoring
- **Description**: Monitor the vertical scroll position of the primary display during autoscroll
- **Priority**: P0 (Critical)
- **Implementation Details**:
  - Hook into existing `Autoscroll.scrollRunnable` execution (runs every 20ms)
  - Calculate current visible content boundaries based on `scrollPosition` and display height
  - Track which song sections are currently visible based on scroll position

#### FR-2: Section Boundary Detection
- **Description**: Detect when current section scrolls off the visible area
- **Priority**: P0 (Critical)
- **Implementation Details**:
  - Parse song content to determine Y-coordinate boundaries of each section
  - Compare scroll position to section boundaries
  - Trigger section advancement when current section scrolls beyond threshold
  - Use existing `Song.presoOrderSongSections` for section content

#### FR-3: Automatic Section Advancement
- **Description**: Advance to next section on connected display when trigger condition is met
- **Priority**: P0 (Critical)
- **Implementation Details**:
  - Call existing `displayInterface.performanceShowSection(position)` method
  - Update `Song.currentSection` to new section index
  - Use existing `DisplayInterface.updateDisplay("showSection")` for connected display update
  - Increment section position by 1 for each advancement

#### FR-4: Primary Display Section Highlighting
- **Description**: Highlight the current section in the primary display UI
- **Priority**: P0 (Critical)
- **Implementation Details**:
  - Update `SongSectionsAdapter.highlightedArray` when section changes
  - Call `notifyItemChanged()` for visual update
  - Use existing highlighting mechanism (color change for Presenter mode, alpha for Stage mode)
  - Ensure highlighting persists across screen rotations

#### FR-5: Manual Section Override
- **Description**: Allow manual section navigation during autoscroll
- **Priority**: P0 (Critical)
- **Implementation Details**:
  - Maintain existing section click behavior in `SongSectionsAdapter.itemSelected()`
  - Update internal "expected current section" to match manual selection
  - Automatic advancement continues from manually selected section
  - Connected display updates immediately via existing `presenterShowSection()` mechanism

#### FR-6: Pause Behavior
- **Description**: Pause section advancement when autoscroll is paused
- **Priority**: P0 (Critical)
- **Implementation Details**:
  - Check `Autoscroll.isPaused` flag before processing section advancement
  - No section advancement while `isPaused == true`
  - Resume advancement when `isPaused == false`
  - Manual section navigation remains functional while paused

#### FR-7: End-of-Song Section Handling
- **Description**: Continue advancing sections at end of song when scrolling has stopped
- **Priority**: P1 (High)
- **Implementation Details**:
  - Detect when autoscroll reaches bottom (`isContentAtBottom == true`)
  - Calculate remaining unadvanced sections
  - Advance through remaining sections at configured interval (e.g., every N seconds)
  - Use existing song duration or configurable "end section duration" setting

---

### 5.2 Configuration & Settings

#### FR-8: Advancement Threshold Setting
- **Description**: Configure when section advancement occurs relative to content position
- **Priority**: P1 (High)
- **Setting Location**: Autoscroll Settings Fragment
- **Options**:
  1. "When section starts to scroll off" (top of section reaches top of screen)
  2. "When section is 25% scrolled off" (25% of section is above screen top)
  3. "When section is 50% scrolled off" (50% of section is above screen top) **[Default]**
  4. "When section is 75% scrolled off" (75% of section is above screen top)
  5. "When section fully scrolled off" (bottom of section reaches top of screen)
  6. "When next section is 25% visible" (25% of next section is visible on screen)
- **Storage**: `autoscrollSectionAdvanceThreshold` preference (String or int enum)
- **UI**: SeekBar or Spinner in `AutoscrollSettingsFragment`

#### FR-9: End-Section Duration Setting
- **Description**: Configure how long each end section displays when scrolling has stopped
- **Priority**: P2 (Medium)
- **Setting Location**: Autoscroll Settings Fragment
- **Default Value**: 5 seconds per section
- **Range**: 2-15 seconds
- **Storage**: `autoscrollEndSectionDuration` preference (int seconds)
- **UI**: NumberPicker or SeekBar with value label

#### FR-10: Feature Enable/Disable Toggle
- **Description**: Allow users to enable or disable section synchronization
- **Priority**: P1 (High)
- **Setting Location**: Autoscroll Settings Fragment
- **Default Value**: Enabled (true)
- **Storage**: `autoscrollSectionSyncEnabled` preference (boolean)
- **UI**: SwitchPreference with description
- **Behavior When Disabled**: Autoscroll works as before (no section advancement)

---

### 5.3 Edge Cases & Error Handling

#### FR-11: No Connected Display
- **Description**: Handle autoscroll when no connected display is active
- **Behavior**:
  - Section highlighting still occurs on primary display
  - No section advancement commands sent
  - No errors or crashes

#### FR-12: Song with Single Section
- **Description**: Handle songs that have only one section
- **Behavior**:
  - No section advancement (only 1 section exists)
  - Section remains highlighted throughout autoscroll
  - No errors

#### FR-13: Song with No Section Markers
- **Description**: Handle songs without explicit section headers
- **Behavior**:
  - Treat entire song as single section
  - Section sync feature effectively disabled for that song
  - Autoscroll works normally

#### FR-14: Rapid Manual Section Changes
- **Description**: Handle user rapidly clicking through sections during autoscroll
- **Behavior**:
  - Each manual change immediately updates connected display
  - Automatic advancement resumes from last manually selected section
  - No race conditions or missed updates

#### FR-15: Autoscroll Speed Changes
- **Description**: Handle speed up/slow down during autoscroll
- **Behavior**:
  - Section advancement timing adjusts automatically based on scroll position
  - No manual recalculation needed
  - Existing scroll speed mechanisms remain unchanged

---

## 6. Non-Functional Requirements

### NFR-1: Performance
- Section boundary calculations should not impact autoscroll frame rate (maintain 20ms cycle time)
- Section advancement should occur within 100ms of trigger condition
- Memory overhead should be minimal (<1MB additional for section tracking)

### NFR-2: Compatibility
- Feature must work with existing display modes (Presenter, Stage, Performance)
- Must work with existing connected display types (HDMI, wireless)
- Backward compatible with songs created in older app versions

### NFR-3: Reliability
- No missed section transitions during normal operation
- Graceful degradation if connected display disconnects during autoscroll
- No crashes or freezes under any user interaction pattern

### NFR-4: Usability
- Feature should work "out of the box" with sensible defaults
- Settings should be discoverable in Autoscroll Settings
- Visual feedback (highlighting) should be immediately obvious

---

## 7. Technical Architecture

### 7.1 Component Overview

```
┌─────────────────────────────────────────────────────────────┐
│                   Autoscroll.java                           │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  scrollRunnable (existing, runs every 20ms)           │  │
│  │  ├─ Update scrollPosition                             │  │
│  │  ├─ Check isPaused                                    │  │
│  │  └─ NEW: Call sectionSyncManager.checkAndAdvance()   │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│          AutoscrollSectionSyncManager.java (NEW)            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  checkAndAdvance()                                    │  │
│  │  ├─ Calculate visible content boundaries             │  │
│  │  ├─ Determine current section based on scroll pos    │  │
│  │  ├─ Compare to lastAdvancedSection                   │  │
│  │  ├─ If threshold met: advanceToNextSection()         │  │
│  │  └─ Handle end-of-song sections                      │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  calculateSectionBoundaries()                         │  │
│  │  └─ Parse song content, build Y-position map         │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  advanceToNextSection()                               │  │
│  │  ├─ Update Song.currentSection                        │  │
│  │  ├─ Call displayInterface.performanceShowSection()   │  │
│  │  └─ Update section highlighting                      │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│              DisplayInterface.java (existing)               │
│  └─ performanceShowSection(position)                        │
│     └─ MainActivity.performanceShowSection()                │
│        └─ updateDisplay("showSection")                      │
│           └─ SecondaryDisplay.showSection()                 │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│          SongSectionsAdapter.java (existing)                │
│  └─ Update highlightedArray and call notifyItemChanged()   │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Key Classes to Modify

#### 7.2.1 Autoscroll.java (Existing - Modify)
**Changes:**
- Add `AutoscrollSectionSyncManager sectionSyncManager` field
- Initialize manager in `startAutoscroll()`
- Call `sectionSyncManager.checkAndAdvance(scrollPosition, isPaused, isContentAtBottom)` in `scrollRunnable`
- Pass necessary dependencies: `Song`, `DisplayInterface`, `MainActivityInterface`

#### 7.2.2 AutoscrollSectionSyncManager.java (New Class)
**Purpose**: Manages section synchronization logic

**Key Fields:**
- `Song song` - Reference to current song
- `DisplayInterface displayInterface` - For triggering section changes
- `MainActivityInterface mainActivityInterface` - For accessing preferences and adapters
- `int currentTrackedSection` - Last section we advanced to (may differ from Song.currentSection if user manually changed)
- `ArrayList<Integer> sectionYPositions` - Y-coordinate of each section's top
- `float advanceThreshold` - Percentage threshold for advancement (from preferences)
- `boolean isEnabled` - Feature enabled state (from preferences)
- `long lastEndSectionAdvanceTime` - Timestamp for end-section advancement timer

**Key Methods:**
- `void initialize(Song song)` - Calculate section boundaries when song loads
- `void checkAndAdvance(int scrollPosition, boolean isPaused, boolean isAtBottom)` - Main logic called every 20ms
- `void calculateSectionBoundaries()` - Parse song content to determine Y positions
- `int getCurrentVisibleSection(int scrollPosition)` - Determine which section is at top of screen
- `boolean shouldAdvanceSection(int visibleSection)` - Check if advancement threshold is met
- `void advanceToSection(int newSection)` - Execute section advancement
- `void handleEndOfSongSections(boolean isAtBottom)` - Advance through end sections on timer
- `void updateHighlighting(int section)` - Update section highlighting in adapters
- `void onManualSectionChange(int section)` - Called when user manually changes section

#### 7.2.3 AutoscrollSettingsFragment.java (Existing - Modify)
**Changes:**
- Add `autoscrollSectionSyncEnabled` SwitchPreference
- Add `autoscrollSectionAdvanceThreshold` SeekBar or Spinner preference
- Add `autoscrollEndSectionDuration` NumberPicker or SeekBar preference
- Group new settings under "Section Synchronization" category

#### 7.2.4 SongSectionsAdapter.java (Existing - Modify)
**Changes:**
- Add public method `void highlightSection(int position)` to allow external highlighting updates
- Ensure highlighting updates don't conflict with user clicks

#### 7.2.5 PerformanceFragment.java (Existing - Modify)
**Changes:**
- Pass `AutoscrollSectionSyncManager` instance when initializing autoscroll
- Call `sectionSyncManager.onManualSectionChange()` when user manually navigates sections

---

### 7.3 Section Boundary Calculation Algorithm

**Challenge**: Determine Y-position of each section in rendered song content

**Approach**:
1. Get the rendered song view (TextView or custom view in `MyZoomLayout`)
2. Parse song content string to find section markers (`§[SectionName]`)
3. For each section:
   - Calculate text height up to that section
   - Account for font size, line spacing, chord heights
   - Store Y-position in `sectionYPositions` array
4. Update boundaries whenever song loads or font size changes

**Pseudo-code**:
```java
void calculateSectionBoundaries() {
    ArrayList<String> sections = song.getPresoOrderSongSections();
    sectionYPositions = new ArrayList<>();

    int currentY = 0;
    Paint textPaint = new Paint();
    textPaint.setTextSize(preferences.getFontSize());

    for (int i = 0; i < sections.size(); i++) {
        sectionYPositions.add(currentY);

        String sectionContent = sections.get(i);
        String[] lines = sectionContent.split("\n");

        for (String line : lines) {
            // Calculate line height (text + chords if present)
            float lineHeight = getLineHeight(line, textPaint);
            currentY += lineHeight;
        }
    }
}

int getCurrentVisibleSection(int scrollPosition) {
    for (int i = sectionYPositions.size() - 1; i >= 0; i--) {
        if (scrollPosition >= sectionYPositions.get(i)) {
            return i;
        }
    }
    return 0;
}
```

---

### 7.4 Advancement Threshold Logic

**Threshold Options** (as percentage of section height):
- 0% = Section top reaches screen top (advancement at section start)
- 25% = 25% of section scrolled off screen
- 50% = 50% of section scrolled off screen (default)
- 75% = 75% of section scrolled off screen
- 100% = Section bottom reaches screen top (advancement at section end)

**Implementation**:
```java
boolean shouldAdvanceSection(int scrollPosition) {
    int currentSection = getCurrentVisibleSection(scrollPosition);

    if (currentSection == currentTrackedSection) {
        return false; // Already advanced to this section
    }

    if (currentSection > currentTrackedSection) {
        // Scrolling forward
        int sectionTop = sectionYPositions.get(currentTrackedSection);
        int sectionBottom = (currentTrackedSection + 1 < sectionYPositions.size())
            ? sectionYPositions.get(currentTrackedSection + 1)
            : totalSongHeight;
        int sectionHeight = sectionBottom - sectionTop;

        int scrolledAmount = scrollPosition - sectionTop;
        float percentScrolled = (float) scrolledAmount / sectionHeight;

        return percentScrolled >= advanceThreshold;
    }

    return false;
}
```

---

### 7.5 End-of-Song Section Handling

**Problem**: Last 1-3 sections may never scroll off screen if song content < display height at end

**Solution**:
1. Detect when autoscroll reaches bottom (`isContentAtBottom == true`)
2. Calculate remaining unadvanced sections
3. Start timer-based advancement for remaining sections
4. Advance every `autoscrollEndSectionDuration` seconds until all sections shown

**Implementation**:
```java
void handleEndOfSongSections(boolean isAtBottom) {
    if (!isAtBottom || !isEnabled) {
        return;
    }

    int totalSections = song.getPresoOrderSongSections().size();
    int remainingSections = totalSections - currentTrackedSection - 1;

    if (remainingSections > 0) {
        long currentTime = System.currentTimeMillis();
        long timeSinceLastAdvance = currentTime - lastEndSectionAdvanceTime;
        long interval = preferences.getAutoscrollEndSectionDuration() * 1000;

        if (timeSinceLastAdvance >= interval) {
            advanceToSection(currentTrackedSection + 1);
            lastEndSectionAdvanceTime = currentTime;
        }
    }
}
```

---

## 8. User Interface Changes

### 8.1 Autoscroll Settings Screen

**New Section**: "Section Synchronization" category

**New Settings**:

1. **Enable Section Sync** (SwitchPreference)
   - Title: "Sync sections with connected display"
   - Summary: "Automatically advance sections on connected display during autoscroll"
   - Default: ON

2. **Advancement Timing** (SeekBar or Spinner)
   - Title: "Section advancement timing"
   - Summary: "When to advance to next section"
   - Options: 0%, 25%, 50%, 75%, 100% (displayed as descriptive text)
   - Default: 50%
   - Visual: SeekBar with labels showing current selection

3. **End Section Duration** (SeekBar with value)
   - Title: "End section display time"
   - Summary: "Seconds to show each end section"
   - Range: 2-15 seconds
   - Default: 5 seconds
   - Visual: SeekBar with current value displayed (e.g., "5 seconds")

**XML Addition** (settings_autoscroll.xml):
```xml
<PreferenceCategory
    android:title="Section Synchronization"
    android:key="autoscroll_section_sync_category">

    <SwitchPreference
        android:key="autoscrollSectionSyncEnabled"
        android:title="Sync sections with connected display"
        android:summary="Automatically advance sections on connected display during autoscroll"
        android:defaultValue="true" />

    <com.garethevans.church.opensongtablet.customviews.SeekBarPreference
        android:key="autoscrollSectionAdvanceThreshold"
        android:title="Section advancement timing"
        android:summary="When to advance to next section"
        android:dependency="autoscrollSectionSyncEnabled"
        android:defaultValue="50" />

    <com.garethevans.church.opensongtablet.customviews.SeekBarPreference
        android:key="autoscrollEndSectionDuration"
        android:title="End section display time"
        android:summary="Seconds to show each end section"
        android:dependency="autoscrollSectionSyncEnabled"
        android:defaultValue="5" />
</PreferenceCategory>
```

---

### 8.2 Primary Display Visual Feedback

**No UI Changes Required** - Use existing section highlighting mechanisms:

- **Presenter Mode**: `SongSectionsAdapter` already supports highlighting via `highlightedArray`
- **Stage Mode**: `StageSectionAdapter` already uses alpha values (1.0 = bright, 0.4 = dimmed)
- **Performance Mode**: Scrolls to position in RecyclerView

**Behavior**:
- Highlighted section updates automatically as sections advance during autoscroll
- Highlighting follows existing app theme colors
- No additional UI elements needed

---

### 8.3 Connected Display

**No Visual Changes** - Connected display already shows individual sections via `SecondaryDisplay.showSection()`

**Behavior**:
- Section content updates automatically as sections advance
- Uses existing transition animations (if configured)
- Matches existing display settings (rotation, scaling, margins)

---

## 9. Testing Requirements

### 9.1 Unit Tests

**AutoscrollSectionSyncManagerTest.java**:
- Test section boundary calculation for various song structures
- Test threshold advancement logic with different thresholds
- Test end-of-song section handling
- Test pause behavior
- Test manual section override
- Test edge cases (single section, no sections, empty song)

### 9.2 Integration Tests

- Test autoscroll + section sync working together
- Test connected display receives section changes
- Test manual section changes during autoscroll
- Test pause/resume behavior
- Test speed changes during autoscroll

### 9.3 Manual Testing Scenarios

1. **Basic Functionality**:
   - Start autoscroll with connected display attached
   - Verify sections advance automatically on connected display
   - Verify current section is highlighted on primary display

2. **Manual Override**:
   - Start autoscroll
   - Click next/previous section buttons during scroll
   - Verify connected display updates immediately
   - Verify automatic advancement resumes from new section

3. **Pause Behavior**:
   - Start autoscroll
   - Pause autoscroll
   - Verify section advancement stops
   - Verify manual section navigation still works while paused
   - Resume autoscroll
   - Verify section advancement resumes

4. **End of Song**:
   - Use short song with 5-6 sections
   - Configure autoscroll to end before all sections shown
   - Verify remaining sections advance on timer
   - Verify final section remains displayed

5. **Configuration**:
   - Test each advancement threshold setting
   - Verify section advances at correct scroll position
   - Test different end section durations

6. **Edge Cases**:
   - Test song with single section
   - Test song with no section markers
   - Test disconnecting display during autoscroll
   - Test rapid speed changes
   - Test rapid manual section changes

### 9.4 Performance Testing

- Monitor autoscroll frame rate with section sync enabled (should maintain 20ms cycle)
- Test with long songs (20+ sections)
- Test with complex formatting (chords, inline pauses)
- Measure memory usage impact

---

## 10. Implementation Phases

### Phase 1: Core Infrastructure (Week 1)
**Deliverables**:
- Create `AutoscrollSectionSyncManager.java` class
- Implement section boundary calculation algorithm
- Integrate manager into `Autoscroll.java` scroll loop
- Add basic advancement logic (50% threshold only)
- Basic unit tests

### Phase 2: Display Integration (Week 1-2)
**Deliverables**:
- Connect section advancement to `DisplayInterface`
- Update `SongSectionsAdapter` for external highlighting control
- Test with real connected display
- Handle edge cases (no display, disconnect during autoscroll)

### Phase 3: Manual Override & Pause (Week 2)
**Deliverables**:
- Implement manual section override handling
- Implement pause synchronization
- Update `PerformanceFragment` integration
- Integration tests

### Phase 4: Settings & Configuration (Week 2-3)
**Deliverables**:
- Add preferences to `AutoscrollSettingsFragment`
- Implement threshold percentage options (0%, 25%, 50%, 75%, 100%)
- Add end section duration setting
- Add enable/disable toggle
- Settings UI tests

### Phase 5: End-of-Song Handling (Week 3)
**Deliverables**:
- Implement timer-based end section advancement
- Test with various song lengths and section counts
- Edge case testing

### Phase 6: Polish & Testing (Week 3-4)
**Deliverables**:
- Complete unit test suite
- Performance testing and optimization
- Manual testing across all scenarios
- Bug fixes and refinements
- Documentation

---

## 11. Dependencies & Risks

### Dependencies:
1. Existing `Autoscroll` class functionality must remain unchanged
2. Existing `DisplayInterface` and `SecondaryDisplay` must support section display
3. `Song` object must contain section data
4. `SongSectionsAdapter` must support external highlighting updates

### Risks:

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Section boundary calculation inaccurate | High | Medium | Extensive testing with various song formats; fallback to simpler calculation |
| Performance impact on autoscroll | High | Low | Optimize calculations; cache boundaries; only calculate when song changes |
| Conflicts with manual section navigation | Medium | Low | Clear precedence: manual changes override automatic; test thoroughly |
| Connected display disconnect during autoscroll | Low | Medium | Graceful handling; check display status before sending commands |
| End-of-song section timing feels wrong | Medium | High | Make configurable; provide sensible defaults; gather user feedback |

---

## 12. Open Questions & Decisions Needed

### Question 1: Section Boundary Calculation Method
**Question**: How accurately can we calculate Y-positions of sections in rendered content?

**Options**:
- A) Parse text and calculate heights programmatically (complex but accurate)
- B) Use View.getLocationOnScreen() for each section TextView (requires section views)
- C) Approximate based on line count and average line height (simple but less accurate)

**Recommendation**: Start with Option C for MVP, refine to Option A if needed

---

### Question 2: Advancement Threshold Default Value
**Question**: What should the default advancement threshold be?

**Options**:
- 0%: Section starts scrolling (very early, audience sees section before it's fully scrolled away on primary)
- 50%: Section half scrolled off (balanced, recommended)
- 100%: Section fully scrolled off (late, audience may lag behind performer)

**Recommendation**: 50% as default (balanced), with easy configuration

---

### Question 3: End Section Advancement Strategy
**Question**: How should end sections advance when autoscroll has stopped?

**Options**:
- A) Timer-based (every N seconds) - simple, predictable
- B) Based on remaining song time - more dynamic but complex
- C) Immediate advancement of all remaining sections - jarring for audience
- D) Don't advance - leave remaining sections unshown (current behavior)

**Recommendation**: Option A (timer-based) for MVP, configurable duration

---

### Question 4: Highlighting During Performance Mode
**Question**: In Performance mode (full song view), how should we show current section?

**Options**:
- A) No highlighting (Performance mode doesn't show sections as cards)
- B) Scroll the section list sidebar to current section (if visible)
- C) Add visual indicator (e.g., line/marker) on the Performance view itself

**Recommendation**: Option B if section list is visible in Performance mode; otherwise Option A

---

### Question 5: Behavior with Inline Pauses
**Question**: What happens to section advancement during inline pauses (`;D:` markers)?

**Options**:
- A) Section advancement pauses during inline pauses (like autoscroll pause)
- B) Section advancement continues based on scroll position regardless
- C) Inline pauses extend the time for current section (prevent advancement)

**Recommendation**: Option A (pause advancement during inline pauses) - most consistent behavior

---

### Question 6: Nearby Connections Integration
**Question**: Should section sync state propagate to Nearby-connected devices?

**Current State**: Nearby connections already sync autoscroll start/stop/pause and section changes

**Options**:
- A) Automatic section advancement only happens on host device (clients follow via existing section sync)
- B) Both host and clients run section sync independently
- C) Add new Nearby command specifically for section sync state

**Recommendation**: Option A - leverage existing section sync mechanism, no new Nearby commands needed

---

## 13. Success Criteria

### MVP Success Criteria:
- ✅ Sections advance automatically on connected display during autoscroll
- ✅ Current section is highlighted on primary display
- ✅ Manual section changes work during autoscroll
- ✅ Pause behavior synchronizes section advancement
- ✅ Feature can be enabled/disabled via settings
- ✅ Advancement threshold is configurable
- ✅ No performance degradation (maintains 20ms autoscroll cycle time)
- ✅ No crashes or errors under normal use

### Post-MVP Enhancements:
- Advanced section boundary calculation (more accurate positioning)
- Per-song override for section sync settings
- Visual indicator in Performance mode showing current section
- Analytics/telemetry for advancement timing optimization
- User preference learning (automatically adjust threshold based on usage patterns)

---

## 14. Documentation Requirements

### User Documentation:
- Update autoscroll help documentation with section sync feature
- Add screenshots showing section highlighting
- Explain advancement timing settings
- Provide troubleshooting guide for sync issues

### Developer Documentation:
- Add inline code comments in `AutoscrollSectionSyncManager`
- Document section boundary calculation algorithm
- Update autoscroll architecture documentation
- Create API documentation for new public methods

---

## 15. Appendix

### A. Relevant Existing Code Locations

**Autoscroll**:
- `/home/user/OpenSongTablet/app/src/main/java/com/garethevans/church/opensongtablet/autoscroll/Autoscroll.java` (lines 1-800)
- `scrollRunnable` execution: lines 400-600

**Connected Display**:
- `/home/user/OpenSongTablet/app/src/main/java/com/garethevans/church/opensongtablet/secondarydisplay/SecondaryDisplay.java` (lines 1-1502)
- `showSection()` method: implements section display logic

**Sections**:
- `/home/user/OpenSongTablet/app/src/main/java/com/garethevans/church/opensongtablet/presenter/SongSectionsAdapter.java` (lines 1-400)
- `itemSelected()` method: line 259
- `highlightedArray`: line ~50

**Display Interface**:
- `/home/user/OpenSongTablet/app/src/main/java/com/garethevans/church/opensongtablet/interfaces/DisplayInterface.java`
- `performanceShowSection(int position)`: line 12

---

### B. Glossary

- **Primary Display**: The tablet/device screen used by the performer
- **Connected Display**: Secondary display (projector, TV, etc.) viewed by the audience
- **Section**: A logical division of a song (verse, chorus, bridge, etc.)
- **Autoscroll**: Automatic scrolling of song content at a configured speed
- **Section Advancement**: Moving to display the next section on the connected display
- **Advancement Threshold**: The scroll position percentage that triggers section advancement
- **End-of-Song Sections**: Sections that remain visible at the bottom of the song (never scroll off screen)
- **Inline Pause**: A pause marker in song lyrics (`;D:` syntax) that temporarily halts autoscroll

---

### C. References

- Android Presentation API: https://developer.android.com/reference/android/app/Presentation
- DisplayManager: https://developer.android.com/reference/android/hardware/display/DisplayManager
- RecyclerView Adapter Notifications: https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.Adapter#notifyItemChanged(int)

---

## Document Approval

**Author**: Claude (AI Assistant)
**Reviewer**: [To be assigned]
**Approver**: [Project Owner]

**Status**: Draft - Awaiting Review & Questions

---

## Next Steps

1. **Review this PRD** and provide feedback
2. **Answer open questions** (Section 12)
3. **Approve or request changes** to requirements
4. **Prioritize features** (if phased delivery is needed)
5. **Begin implementation** (Phase 1: Core Infrastructure)

---

**End of PRD**
