# Testing & Validation Guide

## Pre-Implementation Baseline

Before applying any fixes from CODE_FIXES_GUIDE.md, establish a baseline of current behavior:

### Current Dashboard State
1. **Load dashboard** at http://localhost:8000
2. **Verify current behavior**:
   - Machine selector dropdown shows F01-F12
   - Sidebar shows "{MACHINE} Live KPIs"
   - Overview stat cards show: 49.5%, 52.2%, 4, 11 (hardcoded values)
   - Alerts section populated with F06-specific content
   - Breakdown causes chart shows [8,1,1,1,1,1]

---

## Phase 1 Testing Checklist

### Fix #1: Dynamic Stat Cards (Critical)

**Before Implementation**:
- [ ] Machine selector set to F06
- [ ] Note stat values: 49.5%, 52.2%, 4, 11
- [ ] Switch to F09 (empty machine)
- [ ] Verify stat cards STILL show 49.5%, 52.2%, 4, 11 (proof they're hardcoded)

**After Implementation**:
1. [ ] Reload dashboard
2. [ ] Machine selector set to F06
3. [ ] Verify stat cards show calculated values matching F06 data:
   - OEE %: Should be average of 37 daily entries
   - Day OEE: Verify against day data
   - Breakdowns: Count of breakdown events in F06 data (should match array length)
   - Failures: Count of process failures in F06 data
4. [ ] Switch to F09 (empty machine)
5. [ ] Verify stat cards show: 0%, 0%, 0, 0 (or N/A)
6. [ ] Switch back to F06
7. [ ] Verify original values restored
8. [ ] Console check: No errors related to stat calculation

**Test Data**: F06 has 37 entries with OEE values ranging 30-75%, breakdowns, and failures logged.

---

### Fix #2: Machine-Aware Alerts (Critical)

**Before Implementation**:
- [ ] Machine selector set to F06
- [ ] Note alerts shown (mention F06 post-upgrade timeline)
- [ ] Switch to F02 (empty machine)
- [ ] Alerts STILL show F06 content (proof of hardcoding)

**After Implementation**:
1. [ ] Reload dashboard
2. [ ] Machine selector set to F06
3. [ ] Verify alerts reference F06 and show correct status:
   - Should mention "SVE3600WR - F06 Post-Upgrade"
   - Should show stabilization timeline (started Dec 20, now Mar 12)
   - Should highlight low OEE and breakdowns
4. [ ] Switch to F09 (empty machine)
5. [ ] Verify alerts show placeholder message:
   - "No data available for F09"
   - OR "F09 - Awaiting first data entry"
6. [ ] Switch to F07 (empty machine)
7. [ ] Same placeholder behavior
8. [ ] Switch back to F06
9. [ ] Verify F06-specific alerts restored
10. [ ] Check each machine with data shows appropriate alerts
11. [ ] Console check: No errors in alert rendering

**Expected Alert Types After Fix**:
- ✅ Machine-specific (mentions F06, F07, etc.)
- ✅ Data-aware (shows actual status: Low OEE, High Breakdowns, etc.)
- ✅ Timeline-aware (references machine upgrade dates)
- ✅ Graceful empty state (shows placeholder when no data)

---

### Fix #4: Form Validation (Critical)

**Before Implementation**:
- [ ] Navigate to "Log Data" view
- [ ] Try to enter OEE: 150 (should accept - no validation)
- [ ] Try to enter OEE: -50 (should accept - no validation)
- [ ] Try to enter date: 2020-01-01 (before all machines) should accept

**After Implementation**:
1. [ ] Reload dashboard
2. [ ] Navigate to "Log Data" view
3. [ ] **OEE Validation**:
   - [ ] Enter OEE: 101 → Alert: "OEE must be between 0-100%"
   - [ ] Enter OEE: -1 → Alert: "OEE must be between 0-100%"
   - [ ] Enter OEE: 75.5 → Accepted ✓
   - [ ] Enter OEE: 0 → Accepted ✓
   - [ ] Enter OEE: 100 → Accepted ✓
4. [ ] **Date Validation**:
   - [ ] Set machine to F06
   - [ ] Enter date: 2020-01-01 (before commission) → Alert: "Date must be after F06 commissioning date (2025-12-20)"
   - [ ] Enter date: 2025-12-19 → Alert: "Date must be after F06 commissioning date (2025-12-20)"
   - [ ] Enter date: 2025-12-21 → Accepted ✓
5. [ ] **Machine Default**:
   - [ ] Switch machine selector to F07
   - [ ] Navigate to Log Data
   - [ ] Form machine field defaults to F07 (currently defaults to F06)
6. [ ] **Volume Validation** (if implemented):
   - [ ] Enter negative volume → Alert
   - [ ] Enter 0 volume → Decision: Accept or warn?
   - [ ] Enter 10000 volume → Accepted
7. [ ] **Submission**:
   - [ ] Enter valid data: OEE 75%, Vol 1000, Date 2026-03-12, Machine F06
   - [ ] Click Submit → Entry added to userEntries array
   - [ ] Verify in browser console: `userEntries` array shows new entry
8. [ ] Console check: No validation errors for valid inputs

**Expected Behavior**:
- ✅ OEE limited to 0-100%
- ✅ Dates validated against machine commissioning dates
- ✅ Volume field validates (non-negative, reasonable upper bound)
- ✅ Form defaults to currentMachine
- ✅ Clear error messages explain validation failures
- ✅ Valid submissions logged to userEntries

---

## Phase 2 Testing Checklist

### Fix #3: Dynamic Breakdown Data

**Before Implementation**:
- [ ] Navigate to "Breakdowns" view
- [ ] Causes chart shows [8,1,1,1,1,1] for all machines
- [ ] Switch machines, chart data unchanged

**After Implementation**:
1. [ ] Reload dashboard
2. [ ] Machine selector set to F06
3. [ ] Navigate to "Breakdowns" view
4. [ ] Causes chart shows F06 data:
   - [ ] Count matches actual breakdown events in F06 data
   - [ ] Colors match categories (electrical, mechanical, etc.)
5. [ ] Switch to F09 (empty machine)
6. [ ] Causes chart shows empty state or placeholder
7. [ ] Breakdown events table shows no rows (or "No data" message)
8. [ ] Switch back to F06
9. [ ] Data restored correctly
10. [ ] Hover over chart elements shows correct values (not hardcoded 8,1,1,1,1,1)

**Test Verification**:
```javascript
// In browser console:
MACHINES_DATA.F06.map(day => day.breakdown).filter(e => e).length
// Should match breakdown event count in chart
```

---

### Fix #5: Empty Machine Indicators

**Before Implementation**:
- [ ] Machine dropdown shows F01 F02 F03... F12 uniformly
- [ ] No visual difference between populated (F06) and empty machines

**After Implementation**:
1. [ ] Reload dashboard
2. [ ] Machine dropdown now shows:
   - [ ] Populated machines: "F06 ✓" (with checkmark or badge)
   - [ ] Empty machines: "F09 (No Data)" or grayed out
   - [ ] OR separate sections: "Active Machines" / "Awaiting Data"
3. [ ] Hover over empty machine option shows tooltip: "No data available"
4. [ ] Select empty machine still works but clear placeholder message
5. [ ] Overview shows: "F09 - No data available. Add entries via Log Data view."

**Expected Styling**:
- ✅ Visual distinction between active/empty machines
- ✅ Helpful guidance when selecting empty machine
- ✅ Encourages users to populate data
- ✅ Prevents confusion about "broken" machine views

---

### Fix #6: Dynamic Impact Chart Data

**Before Implementation**:
- [ ] Navigate to "Impact" view
- [ ] Week data shows hardcoded array: weekData array
- [ ] Switching machines doesn't change chart data
- [ ] All machines show identical trend

**After Implementation**:
1. [ ] Reload dashboard
2. [ ] Machine selector set to F06
3. [ ] Navigate to "Impact" view
4. [ ] Chart shows F06's actual data:
   - [ ] X-axis: Weeks since commissioning (calculated from start date)
   - [ ] Y-axis: Rolling average stabilization trend
   - [ ] Data points: Actual OEE progression of F06
5. [ ] Switch to F01 (different commissioning date)
6. [ ] Chart redraws with F01 data:
   - [ ] Different timeline (F01 commissioned earlier)
   - [ ] Different trend line
7. [ ] Switch to F09 (empty machine)
8. [ ] Chart shows placeholder: "Insufficient data to plot trend"
9. [ ] Console verification:
   ```javascript
   // Should vary by machine
   console.log(MACHINES_DATA.F06.length)  // 37 entries
   console.log(MACHINES_DATA.F09.length)  // 0 entries
   ```

---

## Phase 3 Testing Checklist

### Fix #7: Color Variables for Charts

**Before Implementation**:
```javascript
// Colors hardcoded as hex:
let chartColors = ['#FF6384', '#36A2EB', '#FFCE56', ...];
```

**After Implementation**:
1. [ ] Colors now reference CSS variables:
   ```javascript
   // Colors use CSS variables:
   let chartColors = [
     'var(--color-danger)',
     'var(--color-primary)',
     'var(--color-warning)',
     ...
   ];
   ```
2. [ ] In browser inspector, verify colors computed correctly
3. [ ] Inspect element on chart → Colors match CSS values
4. [ ] Change CSS variable value, verify chart color updates
5. [ ] Test dark mode (if CSS has light/dark variants)

---

### Fix #8: Error Handling for Missing Data

**Before Implementation**:
- [ ] Switch to F09 (empty machine)
- [ ] Charts silently render empty
- [ ] No error message or guidance
- [ ] User confused whether system is broken

**After Implementation**:
1. [ ] Switch to F09 (empty machine)
2. [ ] Each view now shows graceful error state:
   - [ ] **Overview**: "No data for F09. Start logging data to view charts."
   - [ ] **Breakdowns**: "No equipment failures recorded for F09."
   - [ ] **Tracker**: Still shows status (not error state)
   - [ ] **Impact**: "Insufficient data to plot trend."
3. [ ] Error states don't crash or show console errors
4. [ ] Error messages are actionable (point to "Log Data")
5. [ ] Populate F09 with data, error states disappear

---

### Fix #9: Mobile Responsive Machine Selector

**Before Implementation** (on mobile/tablet):
- [ ] Machine selector dropdown cramped
- [ ] Text overflow or truncated
- [ ] Options list wider than screen
- [ ] Difficult to use on small screens

**After Implementation**:
1. [ ] Test on mobile-sized viewport (375px width):
   - [ ] Dropdown properly sized
   - [ ] Options display in scrollable list
   - [ ] Selection tap-friendly (minimum 44px height)
2. [ ] Test on tablet (768px width):
   - [ ] Dropdown full-width or appropriately sized
   - [ ] Text readable without overflow
3. [ ] Test on desktop (1920px width):
   - [ ] Original behavior unchanged
4. [ ] Browser inspector → Toggle device toolbar
   - [ ] Responsive design properly adapts

---

## Integration Testing

### Cross-Machine Switching Workflow

**Scenario**: User monitors multiple machines, switching between them

1. [ ] Start at F06 (has data)
2. [ ] Take note of:
   - Sidebar text shows "F06 Live KPIs"
   - Stat cards show F06 values (49.5%, etc.)
   - Charts display F06 data
   - Live date shows today's date
3. [ ] Switch to F02 (empty machine)
   - [ ] Sidebar updates: "F02 Live KPIs"
   - [ ] Stat cards show: 0%, 0%, 0, 0
   - [ ] Alerts show placeholder
   - [ ] Charts show empty state
   - [ ] Form defaults to F02
4. [ ] Switch to F07 (empty)
   - [ ] Same empty behavior as F02
5. [ ] Switch back to F06
   - [ ] All original data restored
   - [ ] No data loss or corruption
6. [ ] Rapid switching (F06 → F09 → F06 → F02 → F06):
   - [ ] No console errors
   - [ ] Each switch renders correct data
   - [ ] No memory leaks (Task Manager CPU/RAM stable)

---

### Data Entry Workflow

**Scenario**: User adds data for F06 via Log Data form

1. [ ] Machine selector: F06
2. [ ] Navigate to "Log Data"
3. [ ] Enter data:
   - Date: 2026-03-13 (after last F06 entry)
   - OEE: 68.5%
   - Day OEE: 70%
   - Night OEE: 67%
   - Volume: 1250
   - Breakdown: None
4. [ ] Click Submit
5. [ ] Verify:
   - [ ] Entry appears in userEntries array (browser console)
   - [ ] Stat cards update with new average OEE
   - [ ] Charts add new data point (visible when returning to Overview)
   - [ ] Heatmap shows new row at bottom
   - [ ] No errors in console

---

### Browser Console Validation

After each fix, check browser console (F12 → Console tab):

```javascript
// Should have NO errors, only optional warnings:
console.clear()  // Clear existing logs
// Then perform test scenario
// Result: Zero console errors, only info/debug messages okay
```

---

## Performance Validation

### Load Time Baseline
- [ ] First load: Measure time to interactive (dashboard usable)
- [ ] Expected: <2 seconds on modern connection
- [ ] After fixes: Should not increase

### Machine Switching Performance
- [ ] Time from clicking machine selector to charts rendering
- [ ] Expected: <500ms (feels instant to user)
- [ ] Measure with DevTools Performance tab

### Memory Usage
- [ ] Open dashboard, note Task Manager memory (for Python server)
- [ ] Switch between machines 20+ times
- [ ] Memory should remain stable (not increase linearly)
- [ ] After fix: Memory should still be stable

---

## Sign-Off Checklist

When all fixes complete, verify:

- [ ] ✅ All Phase 1 tests pass (stat cards, alerts, validation)
- [ ] ✅ All Phase 2 tests pass (breakdown data, empty indicators, impact chart)
- [ ] ✅ All Phase 3 tests pass (colors, error handling, mobile)
- [ ] ✅ Integration test: Cross-machine switching works flawlessly
- [ ] ✅ Data entry workflow: Form → Charts all connected
- [ ] ✅ Browser console: Zero errors
- [ ] ✅ Performance: Load time <2s, switching <500ms
- [ ] ✅ Mobile responsive: Works on 375px and larger
- [ ] ✅ Git commit: All changes committed with clear messages

---

## Regression Testing (After Each Fix)

After implementing each fix, regression test OTHER features:

- [ ] Navigation still works (all 8 views accessible)
- [ ] Print-to-PDF still works (Ctrl+P)
- [ ] Charts still render (no rendering errors)
- [ ] Data tables still display (proper formatting)
- [ ] Sidebar navigation functional
- [ ] Dropdown selectors working
- [ ] Form still accepts input
- [ ] Browser compatibility (Chrome, Firefox, Safari)

---

## Known Issues to Monitor

During testing, watch for these potential issues:

1. **Chart.js Memory Leaks**: When switching machines rapidly, JS could leak memory if not destroying old chart instances. Monitor: `console.log(Chart.helpers.core.instances)`

2. **Timezone Display**: Dates might differ based on browser timezone. Monitor: Compare `new Date()` with server time

3. **Decimal Precision**: OEE calculations might show 49.50000000001% due to floating point. Watch for rounding

4. **Empty Array Edge Cases**: Empty machines might cause `Math.max(...empty)` errors. Verify graceful handling.

---

## Testing Schedule

Recommended testing timeline:

- **Day 1**: Apply Phase 1 fixes (stat cards, alerts, validation)
  - Estimate: 2-3 hours coding + 1 hour testing
  
- **Day 2**: Apply Phase 2 fixes (breakdown data, indicators, impact chart)
  - Estimate: 2-3 hours coding + 1 hour testing
  
- **Day 3**: Apply Phase 3 fixes (colors, error handling, mobile)
  - Estimate: 1-2 hours coding + 45 min testing
  
- **Day 4**: Integration testing + regression testing
  - Estimate: 2 hours comprehensive validation

---

## Questions During Testing?

If a fix doesn't pass testing:

1. Check the CODE_FIXES_GUIDE.md implementation details
2. Verify you copied the exact code (sometimes typos hide in copy-paste)
3. Check browser console for error messages
4. Reload dashboard (sometimes cache issues)
5. Verify currentMachine variable is set correctly
6. Check MACHINES_DATA structure matches expected format
7. If still stuck: Check git diff to see exactly what changed

---

**Good luck with testing! This guide should cover all scenarios.** 🚀
